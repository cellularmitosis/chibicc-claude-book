# Chapter 18 — The full ABI

> Commits covered: `b29f052`, `9021f7f`, `5e0f8c4`, `d63b1f4`, `c72df1c`, `d7bad96`, `b6d3cd0`, `603de50`, `e0b5da3`, `3f2c2d5`, `fc69f5c`, `be8b6f6`, `cc852fe`, `441a89b`, `54c2b3b`, `17ea802`, `c302a96`, `2bdc6b8`, `b1fdddf`, `2c91da5`, `5257ee0`, `9c36dd7`, `c3075b3`. Twenty-three commits — the full SysV AMD64 calling convention (stack-passed args and parameters, struct-by-value in both directions, variadic functions whose fixed parameters spill onto the stack, `va_copy`), bitfields, pp-numbers, command-line `-D`/`-U`, and a handful of polish commits.

Through Chapter 17, chibicc compiles itself. Stage-2 builds. The preprocessor is real. But the compiler still has gaps — the kind of gaps that are *quiet* under the existing test suite but loud the moment a real C program walks across them. Pass a function more than six integer arguments and chibicc miscompiles. Pass a struct by value and chibicc errors out at parse. Return a struct by value and there's no codegen path. Declare a struct member as `int x:3;` and chibicc thinks the colon is a syntax error. The features that close those gaps are scattered across twenty-three commits, dated from May 2020 through September 2020, and they fall into six topics that the chapter takes section by section.

The largest topic is the SysV AMD64 calling convention. Chibicc has been doing the easy parts of that ABI since Chapter 5: integer arguments in `%rdi`/`%rsi`/`%rdx`/`%rcx`/`%r8`/`%r9`, floating-point arguments in `%xmm0` through `%xmm7`. The hard parts — the parts that come up the moment a function takes more than six integer args, or an argument is a struct, or a return value is a struct — have been deferred. Two of those gaps appeared as named errata items at the close of Chapter 5 and again at Chapter 15. This chapter closes them.

The other topics are independent of the ABI work but Rui interleaves them on `main`: bitfields (five commits, the chapter's longest single arc after struct-by-value), pp-numbers (one commit, but with an invasive lexer change), command-line `-D` and `-U`, an in-memory output buffer in the driver, a list of GCC-compatible flags that get accepted-and-ignored, and small late corrections — `main` implicitly returning zero, arrays of at least sixteen bytes getting at-least-sixteen-byte alignment, anonymous struct and union members.

Six sections from twenty-three commits.

- **§18.1** — Stack-passed arguments and parameters (commits 198–199).
- **§18.2** — Struct-by-value (commits 200–203).
- **§18.3** — Variadic with stack-resident fixed parameters, and `va_copy` (commits 204–205).
- **§18.4** — Small completions: function-deref, pp-numbers, `-D`, `-U` (commits 206–209).
- **§18.5** — Bitfields (commits 210–214).
- **§18.6** — Polish and tail (commits 215–220).

The chapter follows `main` order. Calendar dates scatter — the bitfield commits cluster around late August and early September 2020, the small completions are spread across April through September, and the polish commits drift back into May. The function-deref commit (`e0b5da3`) is dated April 23, 2020 — well before the preprocessor work of Chapter 17 — but lands in `main` order at the position Rui chose for it.

---

## 18.1 — Stack-passed arguments and parameters

> `git checkout b29f0521025c95ff331ddb58258b1083f8efd9ff` — *Support passed-on-stack arguments*
>
> `git checkout 9021f7f5decea3e7954f138e9bac4cfea26292be` — *Support passed-on-stack parameters*

Two commits. The first teaches the caller side of a call to push arguments past the sixth integer or eighth floating-point onto the stack. The second teaches the callee side to read those arguments back out of the caller's stack frame. Both commits are pure codegen.

The chapter mapping had this section closing two errata items: at Chapter 5 §5.4, "more than 6 integer args silently miscompiles," and at Chapter 15 §15.6, "more than 8 FP args silently miscompiles." Both items meant the same thing — when a call had more arguments of one class than there were registers in that class, chibicc emitted code that wrote the seventh-and-beyond into nonexistent registers. The errata candidates are closed by the two commits below.

### The caller side

The pre-`b29f052` `push_args` is short:

```c
static void push_args(Node *args) {
  if (args) {
    push_args(args->next);

    gen_expr(args);
    if (is_flonum(args->ty))
      pushf();
    else
      push();
  }
}
```

Recursive walk to the end of the argument list, then evaluate and push each one on the way back. The popping into registers happens in `gen_expr`'s `ND_FUNCALL` case after this returns:

```c
int gp = 0, fp = 0;
for (Node *arg = node->args; arg; arg = arg->next) {
  if (is_flonum(arg->ty))
    popf(fp++);
  else
    pop(argreg64[gp++]);
}
```

If `gp` runs past 5 or `fp` runs past 7, `argreg64[gp]` and `popf(fp)` reach off the end of their arrays. Undefined behavior in chibicc the host program; broken codegen for chibicc the compiler.

The post-commit shape is two-pass. `push_args` itself is renamed to count: it walks the argument list and decides which arguments will go in registers and which will go on the stack, marking the latter with a new `pass_by_stack` bool on the `Node`:

```c
struct Node {
  // ...
  Node *args;
  bool pass_by_stack;
  // ...
};
```

The counting walk:

```c
static int push_args(Node *args) {
  int stack = 0, gp = 0, fp = 0;

  for (Node *arg = args; arg; arg = arg->next) {
    if (is_flonum(arg->ty)) {
      if (fp++ >= FP_MAX) {
        arg->pass_by_stack = true;
        stack++;
      }
    } else {
      if (gp++ >= GP_MAX) {
        arg->pass_by_stack = true;
        stack++;
      }
    }
  }

  if ((depth + stack) % 2 == 1) {
    println("  sub $8, %%rsp");
    depth++;
    stack++;
  }

  push_args2(args, true);
  push_args2(args, false);
  return stack;
}
```

`GP_MAX` is `6`, `FP_MAX` is `8` — defined as macros at the top of the file. The `(depth + stack) % 2 == 1` test ensures `%rsp` is sixteen-byte aligned at the call instruction; if not, an extra eight bytes is reserved. The new helper `push_args2` does the actual evaluation in two passes:

```c
static void push_args2(Node *args, bool first_pass) {
  if (!args)
    return;

  push_args2(args->next, first_pass);

  if ((first_pass && !args->pass_by_stack) || (!first_pass && args->pass_by_stack))
    return;

  gen_expr(args);

  if (is_flonum(args->ty))
    pushf();
  else
    push();
}
```

Pass one handles arguments that *won't* be stack-passed (they get evaluated and pushed onto the stack as a temporary holding area, in right-to-left order). Pass two handles arguments that *will* be stack-passed (they get evaluated and pushed in the same right-to-left order, but their final position is the call's stack-argument area). The reason for two passes — rather than one walk that knows what to do for each argument — is that the order of evaluation matters: C does not specify argument evaluation order, but chibicc evaluates them right-to-left, and the stack-arg area must end up immediately above `%rsp` at the call instruction. The two passes are how Rui maintains both invariants.

Inside `gen_expr`'s `ND_FUNCALL` case, the loop that pops arguments into registers gains a guard:

```c
int gp = 0, fp = 0;
for (Node *arg = node->args; arg; arg = arg->next) {
  if (is_flonum(arg->ty)) {
    if (fp < FP_MAX)
      popf(fp++);
  } else {
    if (gp < GP_MAX)
      pop(argreg64[gp++]);
  }
}
```

Stack-passed arguments aren't popped — they stay on the stack, where the callee will read them. After the call, the stack space is reclaimed:

```c
println("  mov %%rax, %%r10");
println("  mov $%d, %%rax", fp);
println("  call *%%r10");
println("  add $%d, %%rsp", stack_args * 8);

depth -= stack_args;
```

There are three small but important changes here. The first is that the callee is now called via `%r10`, not `%rax`. The reason: `%rax`, by ABI, holds the count of floating-point arguments at call time when the callee is variadic, and after the `mov $%d, %%rax` instruction it does. The callee address has to be moved out of `%rax` first, into a register that is *not* `%rax` and *not* one of the argument registers; `%r10` is the canonical scratch register for this purpose. The second change is the `mov $%d, %%rax` with the floating-point count — required for variadic functions, harmless for non-variadic ones, and chibicc emits it unconditionally. The third is the `add $%d, %%rsp` after the call, which reclaims the stack-passed-arg area.

The pre-commit alignment kludge — `if (depth % 2 == 0) { call ... } else { sub $8, %rsp; call ...; add $8, %rsp; }` — is gone. `push_args` now reserves the right amount of space up front, so the call site is uniform.

### The callee side

`9021f7f` is the mirror commit on the callee side. The pre-commit `assign_lvar_offsets` walks every local variable (including parameters, since parameters are local variables in chibicc's `Obj` model) and stacks them at negative offsets from `%rbp`:

```c
int offset = 0;
for (Obj *var = fn->locals; var; var = var->next) {
  offset += var->ty->size;
  offset = align_to(offset, var->align);
  var->offset = -offset;
}
fn->stack_size = align_to(offset, 16);
```

The post-commit version recognizes that parameters past the register limit live at *positive* offsets from `%rbp`, in the caller's stack frame. Two loops, one for each direction:

```c
int top = 16;
int bottom = 0;
int gp = 0, fp = 0;

// Assign offsets to pass-by-stack parameters.
for (Obj *var = fn->params; var; var = var->next) {
  if (is_flonum(var->ty)) {
    if (fp++ < FP_MAX)
      continue;
  } else {
    if (gp++ < GP_MAX)
      continue;
  }

  top = align_to(top, 8);
  var->offset = top;
  top += var->ty->size;
}

// Assign offsets to pass-by-register parameters and local variables.
for (Obj *var = fn->locals; var; var = var->next) {
  if (var->offset)
    continue;

  bottom += var->ty->size;
  bottom = align_to(bottom, var->align);
  var->offset = -bottom;
}

fn->stack_size = align_to(bottom, 16);
```

`top` starts at 16 because the bytes above `%rbp` are, in order: the saved `%rbp` of the caller (eight bytes), then the return address (eight bytes), then the first stack-passed argument. The first such argument is therefore at `%rbp + 16`. Each subsequent argument is rounded up to eight-byte alignment and placed.

The local-variable loop now skips variables whose offset has already been set — `if (var->offset) continue;` — because the parameter loop wrote positive offsets for stack-passed parameters. Pass-by-register parameters fall through to the local-variable loop and get negative offsets like before; their `offset` field is still zero when the second loop reaches them.

The other callee-side change is in `emit_text`, where the prologue copies pass-by-register parameters from their argument registers into their stack slots:

```c
for (Obj *var = fn->params; var; var = var->next) {
  if (var->offset > 0)
    continue;

  if (is_flonum(var->ty))
    store_fp(fp++, var->offset, var->ty->size);
  else
    store_gp(gp++, var->offset, var->ty->size);
}
```

The `if (var->offset > 0) continue;` skips stack-passed parameters; they don't need to be copied because they're already at the right place in the stack frame. The argument-register-to-stack-slot copy applies only to the parameters that came in registers.

### Where we are

The two commits close the "more than 6 integer args silently miscompiles" errata from Chapter 5 §5.4 and the "more than 8 FP args silently miscompiles" errata from Chapter 15 §15.6. Caller-side, `push_args` returns a stack-cell count that the call site uses to reserve space and reclaim it; argument evaluation runs in two passes (register-bound first, then stack-bound) to keep right-to-left evaluation order while landing the stack-bound args in their final position. Callee-side, `assign_lvar_offsets` learns to place pass-by-stack parameters at positive offsets above `%rbp + 16`, and the prologue's register-copy loop skips them. With this, chibicc handles arbitrarily long argument lists — at least, lists of scalar arguments. Struct arguments are §18.2.

---

## 18.2 — Struct-by-value

> `git checkout 5e0f8c47e3bd91f589710a28f09b718d4a0ec6f3` — *Allow struct parameter*
>
> `git checkout d63b1f410a7aa3d308d0620d640f417a87b0c838` — *Allow struct argument*
>
> `git checkout c72df1c9be535bdfd5b46609996bf1eaf540aced` — *Allow to call a fucntion returning a struct* (Rui's typo)
>
> `git checkout d7bad961146b9f2fd918f05fd59a50f3f65bf325` — *Allow to define a function returning a struct*

Four commits. The deepest section in the chapter, and the section where the SysV AMD64 ABI's central trick — *eightbyte classification* — finally lands in chibicc's codegen.

The ABI's rule, abridged to chibicc's level of fidelity: structs and unions of size at most sixteen bytes split into one or two eight-byte chunks. Each chunk is classified independently. If a chunk contains only floating-point members in its byte range, it travels in an SSE register (`%xmm0`/`%xmm1`/...); otherwise it travels in an integer register (`%rdi`/`%rsi`/...). If a struct's classification would consume more registers of either class than remain available, the entire struct goes to the stack instead. Structs of size greater than sixteen bytes go to memory unconditionally — passed on the stack as arguments, and returned via a hidden first-pointer parameter that the caller allocates and passes in.

The full ABI specification is more elaborate than that — it has rules for unions, for misaligned chunks, for `__m256` and `__float128` and `_Decimal*`, for the special "X87" and "complex X87" classifications, and for nested struct flattening. Chibicc implements the small slice it needs: TY_STRUCT/TY_UNION/TY_ARRAY recursive walks, INTEGER vs. SSE classification, and the sixteen-byte size cutoff. That slice is enough to handle every struct chibicc itself uses, every struct in chibicc's test suite, and every struct in the C programs Rui's been compiling against. The "memory" classification — for things larger than sixteen bytes — turns into the hidden-pointer path.

Rui leaves a comment at the top of the section that summarizes the rule:

```c
// Structs or unions equal or smaller than 16 bytes are passed
// using up to two registers.
//
// If the first 8 bytes contains only floating-point type members,
// they are passed in an XMM register. Otherwise, they are passed
// in a general-purpose register.
//
// If a struct/union is larger than 8 bytes, the same rule is
// applied to the the next 8 byte chunk.
//
// This function returns true if `ty` has only floating-point
// members in its byte range [lo, hi).
```

The implementation is a recursive walk:

```c
static bool has_flonum(Type *ty, int lo, int hi, int offset) {
  if (ty->kind == TY_STRUCT || ty->kind == TY_UNION) {
    for (Member *mem = ty->members; mem; mem = mem->next)
      if (!has_flonum(mem->ty, lo, hi, offset + mem->offset))
        return false;
    return true;
  }

  if (ty->kind == TY_ARRAY) {
    for (int i = 0; i < ty->array_len; i++)
      if (!has_flonum(ty->base, lo, hi, offset + ty->base->size * i))
        return false;
    return true;
  }

  return offset < lo || hi <= offset || is_flonum(ty);
}

static bool has_flonum1(Type *ty) {
  return has_flonum(ty, 0, 8, 0);
}

static bool has_flonum2(Type *ty) {
  return has_flonum(ty, 8, 16, 0);
}
```

`has_flonum(ty, lo, hi, offset)` returns true if every scalar member of `ty` whose byte range overlaps `[lo, hi)` (when `ty` is placed at byte offset `offset` from the start of an outer struct) is itself a floating-point type. Members that fall outside `[lo, hi)` count as "vacuously floating-point" — they don't disqualify the chunk because they don't live in the chunk. The function returns true exactly when the chunk *can* go in an SSE register; if any non-floating-point scalar overlaps the chunk, the chunk has to go in an integer register.

`has_flonum1` checks the first chunk (bytes 0–7), `has_flonum2` checks the second (bytes 8–15). Together with the size check (`ty->size > 16`), they're the inputs to the classification.

### Struct as parameter (commit 5e0f8c4)

The first commit handles the parameter side of struct-by-value, on the *caller*. Specifically: at a call site, when an argument is a struct of at most sixteen bytes, decide whether the struct fits in available registers, and if so split it into one or two register loads; if not, push it onto the stack.

The pre-commit parser rejected the case at parse time:

```c
if (param_ty->kind == TY_STRUCT || param_ty->kind == TY_UNION)
  error_tok(arg->tok, "passing struct or union is not supported yet");
```

The post-commit parser drops the rejection and skips the cast:

```c
if (param_ty->kind != TY_STRUCT && param_ty->kind != TY_UNION)
  arg = new_cast(arg, param_ty);
```

Casts are skipped because there is no cast for struct types — the argument already has the right type, and chibicc's cast machinery doesn't know what to do with structs.

`push_args` learns the struct case in two places. First, the counting walk decides whether each struct argument fits in registers:

```c
case TY_STRUCT:
case TY_UNION:
  if (ty->size > 16) {
    arg->pass_by_stack = true;
    stack += align_to(ty->size, 8) / 8;
  } else {
    bool fp1 = has_flonum1(ty);
    bool fp2 = has_flonum2(ty);

    if (fp + fp1 + fp2 < FP_MAX && gp + !fp1 + !fp2 < GP_MAX) {
      fp = fp + fp1 + fp2;
      gp = gp + !fp1 + !fp2;
    } else {
      arg->pass_by_stack = true;
      stack += align_to(ty->size, 8) / 8;
    }
  }
  break;
```

The arithmetic — `fp + fp1 + fp2`, `gp + !fp1 + !fp2` — accounts for both chunks at once. `fp1` is true if chunk one is floating-point (so it consumes an FP register); `!fp1` is true if it isn't (so it consumes a GP register). Likewise for `fp2`. If a struct only has one chunk (size ≤ 8), `has_flonum2` returns true vacuously (every non-existent member is "floating-point"), so `fp2 = 1` and `!fp2 = 0`; the chunk-two terms drop out *exactly* when the size cutoff hits. The classification implicitly handles the one-chunk case through this vacuous-truth mechanism.

If either register class would overflow, the whole struct goes on the stack. The `align_to(ty->size, 8) / 8` is the number of eight-byte stack cells needed.

`push_args2` learns to push the bytes of a struct when the struct is stack-passed:

```c
static void push_struct(Type *ty) {
  int sz = align_to(ty->size, 8);
  println("  sub $%d, %%rsp", sz);
  depth += sz / 8;

  for (int i = 0; i < ty->size; i++) {
    println("  mov %d(%%rax), %%r10b", i);
    println("  mov %%r10b, %d(%%rsp)", i);
  }
}
```

Byte-wise copy from wherever the struct's address is (`%rax`, set by `gen_expr` evaluating the argument) onto the stack. One byte at a time through `%r10b`. Slow but correct, and correct is enough — the test suite doesn't depend on the speed of struct-passing.

The pop loop in `gen_expr`'s `ND_FUNCALL` case learns the struct case symmetrically:

```c
case TY_STRUCT:
case TY_UNION:
  if (ty->size > 16)
    continue;

  bool fp1 = has_flonum1(ty);
  bool fp2 = has_flonum2(ty);

  if (fp + fp1 + fp2 < FP_MAX && gp + !fp1 + !fp2 < GP_MAX) {
    if (fp1)
      popf(fp++);
    else
      pop(argreg64[gp++]);

    if (ty->size > 8) {
      if (fp2)
        popf(fp++);
      else
        pop(argreg64[gp++]);
    }
  }
  break;
```

The first chunk pops into either `%xmm` or `%r` register depending on `fp1`; the second chunk does the same for `fp2`. Stack-passed structs are skipped (they remain on the stack, to be reclaimed after the call).

### Struct as parameter, on the callee (commit d63b1f4)

The second commit is the callee's mirror. When a function takes a struct parameter, `assign_lvar_offsets` and `emit_text`'s prologue both have to know about it.

Offset assignment learns to consume registers for struct-by-register parameters:

```c
case TY_STRUCT:
case TY_UNION:
  if (ty->size <= 16) {
    bool fp1 = has_flonum(ty, 0, 8, 0);
    bool fp2 = has_flonum(ty, 8, 16, 8);
    if (fp + fp1 + fp2 < FP_MAX && gp + !fp1 + !fp2 < GP_MAX) {
      fp = fp + fp1 + fp2;
      gp = gp + !fp1 + !fp2;
      continue;
    }
  }
  break;
```

(Note the `fp2` here is `has_flonum(ty, 8, 16, 8)` — the third argument is `8`, not `0` as in the `has_flonum2` helper used on the caller. This is a minor inconsistency in chibicc's source: on the callee side, the offset accumulator starts at `8` for chunk two, while on the caller side the helper passes `0`. Both call sites land at the right answer because the recursive walk treats `offset` as a base, but the asymmetry is the kind of small thing that catches a maintainer's eye.)

The prologue's parameter-storing loop learns the struct case too:

```c
case TY_STRUCT:
case TY_UNION:
  assert(ty->size <= 16);
  if (has_flonum(ty, 0, 8, 0))
    store_fp(fp++, var->offset, MIN(8, ty->size));
  else
    store_gp(gp++, var->offset, MIN(8, ty->size));

  if (ty->size > 8) {
    if (has_flonum(ty, 8, 16, 0))
      store_fp(fp++, var->offset + 8, ty->size - 8);
    else
      store_gp(gp++, var->offset + 8, ty->size - 8);
  }
  break;
```

`store_gp` gains a `default` case to handle non-power-of-two sizes (a struct chunk might be three bytes, or seven bytes, depending on the struct's layout), copying byte-by-byte through `%al` and shift-rights to peel off bytes:

```c
default:
  for (int i = 0; i < sz; i++) {
    println("  mov %s, %d(%%rbp)", argreg8[r], offset + i);
    println("  shr $8, %s", argreg64[r]);
  }
  return;
```

The byte-by-byte store-and-shift is the inverse of the byte-by-byte load-and-or that the return-side codegen will need. Both shapes recur in the next two commits.

### Struct as return type, caller side (commit c72df1c)

The third commit handles a *call* whose return type is a struct. The ABI splits along the same sixteen-byte cutoff: small structs return through registers (the caller picks them up out of `%rax`/`%rdx` and `%xmm0`/`%xmm1`); large structs return through a hidden first-parameter pointer that the caller passes in and the callee fills.

The parser allocates a return buffer at parse time:

```c
// If a function returns a struct, it is caller's responsibility
// to allocate a space for the return value.
if (node->ty->kind == TY_STRUCT || node->ty->kind == TY_UNION)
  node->ret_buffer = new_lvar("", node->ty);
```

`Node` gains a `ret_buffer` field — an `Obj *` for the local variable that will hold the returned struct.

In codegen, `gen_addr` learns to take the address of a struct-returning call:

```c
case ND_FUNCALL:
  if (node->ret_buffer) {
    gen_expr(node);
    return;
  }
  break;
```

This means that `f().member` is now valid. The address-of is the buffer.

`push_args` gains two pieces — one for large returns (the hidden-pointer first argument), one for small returns (no first-argument tweak, but a buffer-fill after the call):

```c
// If the return type is a large struct/union, the caller passes
// a pointer to a buffer as if it were the first argument.
if (node->ret_buffer && node->ty->size > 16)
  gp++;

// ... [counting walk runs as before] ...

push_args2(node->args, true);
push_args2(node->args, false);

// If the return type is a large struct/union, the caller passes
// a pointer to a buffer as if it were the first argument.
if (node->ret_buffer && node->ty->size > 16) {
  println("  lea %d(%%rbp), %%rax", node->ret_buffer->offset);
  push();
}

return stack;
```

The `gp++` at the top reserves `%rdi` as the hidden buffer pointer. The `lea` near the bottom loads the buffer address and pushes it onto the stack so the post-evaluation pop loop will pick it up into `%rdi` first.

In `gen_expr`'s pop loop:

```c
// If the return type is a large struct/union, the caller passes
// a pointer to a buffer as if it were the first argument.
if (node->ret_buffer && node->ty->size > 16)
  pop(argreg64[gp++]);
```

The buffer pointer pops into `%rdi`, slotting in before the user-visible arguments.

After the call returns, the small-struct case copies the register-borne return value into the buffer:

```c
// If the return type is a small struct, a value is returned
// using up to two registers.
if (node->ret_buffer && node->ty->size <= 16) {
  copy_ret_buffer(node->ret_buffer);
  println("  lea %d(%%rbp), %%rax", node->ret_buffer->offset);
}
```

`copy_ret_buffer` is the helper that disassembles `%rax`/`%rdx` (or `%xmm0`/`%xmm1`) into the buffer's memory:

```c
static void copy_ret_buffer(Obj *var) {
  Type *ty = var->ty;
  int gp = 0, fp = 0;

  if (has_flonum1(ty)) {
    assert(ty->size == 4 || 8 <= ty->size);
    if (ty->size == 4)
      println("  movss %%xmm0, %d(%%rbp)", var->offset);
    else
      println("  movsd %%xmm0, %d(%%rbp)", var->offset);
    fp++;
  } else {
    for (int i = 0; i < MIN(8, ty->size); i++) {
      println("  mov %%al, %d(%%rbp)", var->offset + i);
      println("  shr $8, %%rax");
    }
    gp++;
  }
  // ... [chunk two follows the same pattern, with %xmm1/%rdx] ...
}
```

Floats go through `movss`/`movsd`. Integers go through the byte-and-shift sequence.

After `copy_ret_buffer`, the second `lea` reloads the buffer's address into `%rax`, so the value of the call expression *is* the buffer's address. This way, `f().a` and `f().a` both work — `gen_addr` saw `ND_FUNCALL` with a `ret_buffer` and called `gen_expr`, which left `%rax` holding the buffer address, which is exactly what a member-access expects from `gen_addr`.

The single typo in commit message — "fucntion" — survived to release. (Errata-of-history rather than errata-of-implementation.)

### Struct as return type, callee side (commit d7bad96)

The fourth commit closes the loop on the *function definition* side. The function being defined now has to know how to return a struct.

In `function`, a hidden first parameter is allocated for large-struct returns:

```c
// A buffer for a struct/union return value is passed
// as the hidden first parameter.
Type *rty = ty->return_ty;
if ((rty->kind == TY_STRUCT || rty->kind == TY_UNION) && rty->size > 16)
  new_lvar("", pointer_to(rty));
```

The hidden parameter is a pointer to the struct's type, named with the empty string `""`. It's just a regular local variable from there on out — `assign_lvar_offsets` places it, the prologue stores the incoming `%rdi` into it, and codegen reads it when the return statement runs.

In `stmt`, the return statement learns to skip the cast for struct returns:

```c
Type *ty = current_fn->ty->return_ty;
if (ty->kind != TY_STRUCT && ty->kind != TY_UNION)
  exp = new_cast(exp, current_fn->ty->return_ty);

node->lhs = exp;
```

Casts are skipped because there is no cast for struct types — same reasoning as for the caller-side parameter handling.

In `gen_stmt`, the `ND_RETURN` case adds a struct-copy after evaluating the return expression:

```c
case ND_RETURN:
  if (node->lhs) {
    gen_expr(node->lhs);

    Type *ty = node->lhs->ty;
    if (ty->kind == TY_STRUCT || ty->kind == TY_UNION) {
      if (ty->size <= 16)
        copy_struct_reg();
      else
        copy_struct_mem();
    }
  }

  println("  jmp .L.return.%s", current_fn->name);
  return;
```

`copy_struct_reg` is the inverse of `copy_ret_buffer`. The callee's return expression has put the address of the struct in `%rax`; `copy_struct_reg` reads bytes from that address into `%rax` (and `%rdx`, and possibly `%xmm0`/`%xmm1`) so the caller's `copy_ret_buffer` can disassemble them back out:

```c
static void copy_struct_reg(void) {
  Type *ty = current_fn->ty->return_ty;
  int gp = 0, fp = 0;

  println("  mov %%rax, %%rdi");

  if (has_flonum(ty, 0, 8, 0)) {
    assert(ty->size == 4 || 8 <= ty->size);
    if (ty->size == 4)
      println("  movss (%%rdi), %%xmm0");
    else
      println("  movsd (%%rdi), %%xmm0");
    fp++;
  } else {
    println("  mov $0, %%rax");
    for (int i = MIN(8, ty->size) - 1; i >= 0; i--) {
      println("  shl $8, %%rax");
      println("  mov %d(%%rdi), %%al", i);
    }
    gp++;
  }
  // ... [chunk two with %xmm1/%rdx] ...
}
```

The integer path zeros `%rax`, then loops high-to-low: shift left eight, mov in the new low byte, repeat. The result is a properly packed integer in `%rax`. The float path uses `movss`/`movsd` from the in-memory struct.

`copy_struct_mem` handles the large-struct path. The callee reads the hidden buffer pointer (which lives in the first parameter's stack slot), then byte-copies the struct into it:

```c
static void copy_struct_mem(void) {
  Type *ty = current_fn->ty->return_ty;
  Obj *var = current_fn->params;

  println("  mov %d(%%rbp), %%rdi", var->offset);

  for (int i = 0; i < ty->size; i++) {
    println("  mov %d(%%rax), %%dl", i);
    println("  mov %%dl, %d(%%rdi)", i);
  }
}
```

Byte-by-byte copy. `%rax` holds the local struct's address; `%rdi` holds the caller's buffer address; one byte at a time goes from `%rax`'s pointee to `%rdi`'s pointee through `%dl`.

After this commit, chibicc's struct-by-value handling is symmetric. A function `Ty4 struct_test34(void) { return (Ty4){10, 20, 30, 40}; }` produces correct register-borne return values for sixteen-byte-or-smaller structs and correct buffer-fill behavior for larger ones, on both the caller and the callee.

### Where we are

Four commits, the SysV AMD64 eightbyte classification arc. `has_flonum` is the chapter's central new helper — it walks struct/union/array members recursively, classifies each chunk by whether the chunk holds only floating-point scalars, and feeds the classification into the register-allocation logic. The split between `<= 16 bytes` and `> 16 bytes` controls the entire chapter's ABI work: small structs travel through up to two registers per chunk; large structs travel through a hidden caller-allocated buffer. The `Node->ret_buffer` field carries the buffer's `Obj *` from parse time through codegen. `copy_ret_buffer` and `copy_struct_reg` are mirror helpers; `push_struct` and `copy_struct_mem` likewise. The cost of all this is ~250 lines of new code in `codegen.c` and a few small additions in `parse.c`. The benefit: chibicc now compiles the kind of C programs that real C programmers write, where structs travel by value into and out of functions.

The psABI conformance count ticks up by four — one for each of the four commits. New count: thirteen.

The `unreachable()` macro gains a new caller in `store_gp`'s `default` case (now reachable for non-power-of-two struct chunk sizes; the pre-commit unreachable was a real unreachable for the old fixed-size cases).

---

## 18.3 — Variadic with stack-resident fixed parameters; `va_copy`

> `git checkout b6d3cd00df7d0496fca2af2c34e72ab3e6af4028` — *Allow variadic function to take more than 6 parameters*
>
> `git checkout 603de502fd8bad750d48aaf9a66c547e5ca04c2a` — *Add va_copy()*

Two commits. The first is the variadic completion that pairs with §18.1: a variadic function whose fixed parameters spill onto the stack, plus the `va_arg` macro learning to pull stack-resident variadic arguments out of the caller's overflow area. The second adds `va_copy`.

### Variadic with stack-resident fixed parameters

Through Chapter 14, chibicc's `va_arg` ran from the *register-save area* — a 176-byte block at the bottom of every variadic function's stack frame, where the prologue eagerly saved all six general-purpose argument registers and all eight SSE argument registers. That made variadic-function startup expensive — every variadic function paid for fourteen register saves whether it used them or not — but the implementation was simple, and `va_arg` only had to look in one place.

The pre-commit `va_arg` macro:

```c
static void *__va_arg_gp(__va_elem *ap) {
  void *r = (char *)ap->reg_save_area + ap->gp_offset;
  ap->gp_offset += 8;
  return r;
}

static void *__va_arg_fp(__va_elem *ap) {
  void *r = (char *)ap->reg_save_area + ap->fp_offset;
  ap->fp_offset += 8;
  return r;
}

static void *__va_arg_mem(__va_elem *ap) {
  1 / 0; // not implemented
}
```

`__va_arg_mem` was a placeholder. It divides by zero — a runtime trap if reached — and Chapter 17's §17.5 prose flagged it as an errata candidate. This commit closes the errata.

The post-commit `__va_arg_mem` reads from the overflow argument area, which is the part of the caller's stack frame above the saved registers:

```c
static void *__va_arg_mem(__va_elem *ap, int sz, int align) {
  void *p = ap->overflow_arg_area;
  if (align > 8)
    p = (p + 15) / 16 * 16;
  ap->overflow_arg_area = ((unsigned long)p + sz + 7) / 8 * 8;
  return p;
}
```

The `overflow_arg_area` field on `__va_elem` is set up by the function's prologue. The pre-commit prologue set `reg_save_area` and the offsets but didn't bother with `overflow_arg_area`. The post-commit prologue does:

```c
println("  movl $%d, %d(%%rbp)", gp * 8, off);          // gp_offset
println("  movl $%d, %d(%%rbp)", fp * 8 + 48, off + 4); // fp_offset
println("  movq %%rbp, %d(%%rbp)", off + 8);            // overflow_arg_area
println("  addq $16, %d(%%rbp)", off + 8);
println("  movq %%rbp, %d(%%rbp)", off + 16);           // reg_save_area
println("  addq $%d, %d(%%rbp)", off + 24, off + 16);
```

`overflow_arg_area` is initialized to `%rbp + 16` — the address just past the saved frame pointer and return address, where stack-passed arguments begin. As `__va_arg_mem` walks through them, it advances the field.

The `__va_arg_gp` and `__va_arg_fp` functions learn to fall back to `__va_arg_mem` when they've exhausted their register-save areas:

```c
static void *__va_arg_gp(__va_elem *ap, int sz, int align) {
  if (ap->gp_offset >= 48)
    return __va_arg_mem(ap, sz, align);

  void *r = ap->reg_save_area + ap->gp_offset;
  ap->gp_offset += 8;
  return r;
}

static void *__va_arg_fp(__va_elem *ap, int sz, int align) {
  if (ap->fp_offset >= 112)
    return __va_arg_mem(ap, sz, align);

  void *r = ap->reg_save_area + ap->fp_offset;
  ap->fp_offset += 8;
  return r;
}
```

The thresholds — `48` for general-purpose (six registers × eight bytes) and `112` for floating-point (six registers × sixteen bytes, with `fp_offset` starting at `48`) — match the size of the register-save area for each class.

The `va_arg` macro now passes `sizeof(ty)` and `_Alignof(ty)` through to the helpers, so the memory path can advance `overflow_arg_area` correctly:

```c
#define va_arg(ap, ty)                                                  \
  ({                                                                    \
    int klass = __builtin_reg_class(ty);                                \
    *(ty *)(klass == 0 ? __va_arg_gp(ap, sizeof(ty), _Alignof(ty)) :    \
            klass == 1 ? __va_arg_fp(ap, sizeof(ty), _Alignof(ty)) :    \
            __va_arg_mem(ap, sizeof(ty), _Alignof(ty)));                \
  })
```

The chapter mapping forecast that this commit might close the `fp_offset = fp * 8 + 48` non-conforming stride from Chapter 15 §15.9, where `fp_offset` was being initialized with an eight-byte stride instead of the ABI's mandated sixteen-byte stride. Reading the diff: no, the stride is unchanged (`fp * 8 + 48` stays). The errata candidate is *not* closed by this commit. It remains an errata candidate.

What *is* closed by this commit is the larger gap — variadic functions whose fixed parameters spill onto the stack now work end-to-end, because the `overflow_arg_area` is wired up correctly *and* because §18.1 already handled the stack-passed-parameter case in `assign_lvar_offsets`. The two commits compose. The test suite gains a `vsprintf` test that drives the path:

```c
char *fmt(char *buf, char *fmt, ...) {
  va_list ap;
  va_start(ap, fmt);
  vsprintf(buf, fmt, ap);
  va_end(ap);
  return buf;
}
```

— and a stress test that calls `sum2` with twenty arguments, exercising the overflow path:

```c
ASSERT(210, sum2(1, 2.0, 3, 4.0, 5, 6.0, 7, 8.0, 9, 10.0, 11, 12.0, 13, 14.0, 15, 16.0, 17, 18.0, 19, 20.0, 0));
```

### `va_copy`

`va_copy(dst, src)` is a one-line macro. The C standard requires it to act as if `dst` were initialized to a copy of `src`'s state. Chibicc's `va_list` is `__va_elem[1]` (an array of one element); array-decay makes the array name behave as a pointer; the macro indexes into the (decayed) pointer:

```c
#define va_copy(dest, src) ((dest)[0] = (src)[0])
```

That's the entire commit, on the chibicc side. The interesting part is *why* it's a one-liner. A real C compiler implements `va_copy` as a builtin (`__builtin_va_copy`) because `va_list` is allowed to be an array, a pointer, or a struct, and the assignment semantics differ across those — arrays can't be assigned, but you can index-and-assign, and pointers and structs can be assigned directly. Chibicc's `va_list` is one specific shape (an array of `__va_elem`), and the index-and-assign idiom works because `__va_elem` is a struct (so `(dest)[0] = (src)[0]` is a struct assignment, which chibicc handles). The macro relies on the array-decay equivalence: `dest` is a pointer in expression context, so `(dest)[0]` is `*dest`, which is a struct lvalue; assignment to a struct lvalue is legal C and chibicc's codegen handles it as a memcpy.

This is exactly the kind of move that justifies why chibicc's `va_list` is an array rather than a pointer or a struct — the array shape lets `va_copy` work as a macro without a builtin, and lets `va_start(ap, fmt)` invoke `*(ap) = *(__va_elem *)__va_area__` (which is what Chapter 14 set up). The whole `<stdarg.h>` is glued together by the array-decay equivalence.

### Where we are

The variadic completion makes `va_arg` work for arguments past the register limit and for fixed parameters that spill onto the stack. `__va_arg_mem` is no longer a divide-by-zero placeholder; it walks the overflow area. The `overflow_arg_area` field on `__va_elem` finally has a writer (the prologue) and a reader (`__va_arg_mem`). `va_copy` is a one-line macro, made possible by the array shape of `va_list`. The "more than 6 fixed parameters in a variadic function" errata candidate is closed; the `fp_offset` non-conforming stride is *not*.

The psABI conformance count ticks up by one. New count: fourteen.

---

## 18.4 — Small completion commits: function-deref, pp-numbers, `-D`, `-U`

> `git checkout e0b5da3b395e46bbc2e377a59d5cba33206288a9` — *Dereferencing a function shouldn't do anything*
>
> `git checkout 3f2c2d5bca4f4506e0ab0b03959d96be427fa672` — *Tokenize numeric tokens as pp-numbers*
>
> `git checkout fc69f5c6f9b3aeb5d6ee61353f0ed0df28f954c5` — *Add -D option*
>
> `git checkout be8b6f6d31f0c73c2aabffdf2794f20c69567cdb` — *Add -U option*

Four small-to-medium completions. None alone deserves a section; together, they fill a few of chibicc's remaining oddities.

### Dereferencing a function (commit e0b5da3)

The C standard makes `*f` and `f` equivalent when `f` is a function. So `(*add2)(2, 3)` calls `add2`, and `(***add2)(2, 3)` also calls `add2` — every dereference of a function-typed expression yields the same function-typed expression. Rui's commit message quotes the spec's relevant paragraph (n1570 §6.5.3.2p4) and his code adds the four-line check:

```c
if (equal(tok, "*")) {
  // [https://www.sigbus.info/n1570#6.5.3.2p4] This is an oddity
  // in the C spec, but dereferencing a function shouldn't do
  // anything. If foo is a function, `*foo`, `**foo` or `*****foo`
  // are all equivalent to just `foo`.
  Node *node = cast(rest, tok->next);
  add_type(node);
  if (node->ty->kind == TY_FUNC)
    return node;
  return new_unary(ND_DEREF, node, tok);
}
```

The test that exercises it is `ASSERT(5, (***add2)(2,3));`. Three asterisks, one call.

The reason this matters at all is that a function name in expression context — `add2`, without parentheses — already decays to a function pointer by the existing rules. So `add2` is a function pointer, and `*add2` would, under the ordinary `ND_DEREF` codegen, dereference the pointer. The result *type* would be a function (not a function pointer), and chibicc's load-skip rule (which Chapter 16 §16.2 added for `TY_FUNC`) would mean `*add2` and `add2` produce identical assembly anyway. So the four-line check above is a parser-level shortcut that avoids constructing the intermediate `ND_DEREF` node, but the codegen would have produced the right answer regardless. Rui makes the parser explicit because the C spec is explicit.

### Pp-numbers (commit 3f2c2d5)

The most invasive of the four small commits. The C spec defines a *preprocessor number* as a lexical category broader than the integer/float syntax the parser ultimately accepts. A pp-number can be `0x1.5p-3`, `1e+10`, or `1.5e-3.foo` — anything that starts with a digit (or `.` followed by a digit) and continues with digits, letters, periods, and `+`/`-` after `e`/`E`/`p`/`P`. The preprocessor handles them as opaque tokens. The parser later interprets the text.

Chibicc through Chapter 17 had the tokenizer parse numbers directly into `TK_NUM` tokens with a `val` (or `fval`) and a `Type *`. That worked for ordinary integer and floating-point literals, but it had a subtle bug for the `##` operator: when a macro pasted `2` and `0xff` into `20xff`, the result needed to be a single number token. Chibicc's tokenizer couldn't parse `20xff` as a single number — it would parse `20` and then stop at the `x`. The pasting commit (Chapter 17 §17.4.7) worked around this by re-tokenizing the pasted text, but the workaround was fragile: any `##` whose result wasn't accepted by `read_number` would have failed.

The fix is to introduce a new token kind, `TK_PP_NUM`, that the tokenizer produces for *anything* matching the broad pp-number lexical pattern. The parser later — through `convert_pp_tokens` — re-interprets each `TK_PP_NUM` as either an integer or floating-point `TK_NUM`:

```c
typedef enum {
  TK_IDENT,
  TK_PUNCT,
  TK_KEYWORD,
  TK_STR,
  TK_NUM,
  TK_PP_NUM,  // Preprocessing numbers
  TK_EOF,
} TokenKind;
```

The tokenizer's number arm gets simpler:

```c
// Numeric literal
if (isdigit(*p) || (*p == '.' && isdigit(p[1]))) {
  char *q = p++;
  for (;;) {
    if (p[0] && p[1] && strchr("eEpP", p[0]) && strchr("+-", p[1]))
      p += 2;
    else if (isalnum(*p) || *p == '.')
      p++;
    else
      break;
  }
  cur = cur->next = new_token(TK_PP_NUM, q, p);
  continue;
}
```

The lexer accepts the broadest pp-number shape: digits, letters, periods, and `e+` / `E-` / `p+` / `P-` two-char sequences. The full text is captured as a `TK_PP_NUM`. No interpretation happens here.

Two new functions handle the interpretation later. `convert_pp_int` tries to parse the token as an integer literal:

```c
static bool convert_pp_int(Token *tok) {
  char *p = tok->loc;
  // ... [base, suffix, value parsing — unchanged from old read_int_literal] ...
  if (p != tok->loc + tok->len)
    return false;

  tok->kind = TK_NUM;
  tok->val = val;
  tok->ty = ty;
  return true;
}
```

If the parse consumes the whole token, the token is rewritten in place to a `TK_NUM`. If not — say, because the token is `1.5e-3` and the integer parse stops at the `.` — the function returns false.

`convert_pp_number` falls back to `strtod`:

```c
static void convert_pp_number(Token *tok) {
  if (convert_pp_int(tok))
    return;

  char *end;
  double val = strtod(tok->loc, &end);
  // ... [type inference — unchanged from old read_number] ...

  if (tok->loc + tok->len != end)
    error_tok(tok, "invalid numeric constant");

  tok->kind = TK_NUM;
  tok->fval = val;
  tok->ty = ty;
}
```

`strtod` consumes as many characters as it can; if it stops short of the token's end, it's an invalid numeric constant. Otherwise, the token is rewritten to `TK_NUM`.

`convert_keywords` becomes `convert_pp_tokens`, picking up the second responsibility:

```c
void convert_pp_tokens(Token *tok) {
  for (Token *t = tok; t->kind != TK_EOF; t = t->next) {
    if (is_keyword(t))
      t->kind = TK_KEYWORD;
    else if (t->kind == TK_PP_NUM)
      convert_pp_number(t);
  }
}
```

The function runs once after preprocessing in the cc1 pipeline (in `preprocess()`'s tail) and once before the constant-expression evaluator in `eval_const_expr` (so `#if 1+2` works on `TK_PP_NUM` tokens).

The new test in `test/macro.c` exercises the paste-then-tokenize case:

```c
#define CONCAT(x,y) x##y
ASSERT(5, ({ int f0zz=5; CONCAT(f,0zz); }));
ASSERT(5, ({ CONCAT(4,.57) + 0.5; }));
```

`CONCAT(f, 0zz)` produces the identifier `f0zz` (which is then looked up as a variable). `CONCAT(4, .57)` produces the floating-point literal `4.57`. Pre-commit, `CONCAT(4, .57)` would have failed because `4.57` couldn't be parsed as a number through the old `read_number` after re-tokenization. Post-commit, the pp-number lexer accepts the text wholesale and `convert_pp_number` parses it later.

This is, in effect, a *canonicalization-at-parse-time* move — the kind chibicc has been doing throughout. The tokenizer's job is simpler (just match the broad pattern); the parser's job is the same as before (interpret the text). The simplification flows backward from the `##` operator, which needed a more permissive number lexer to work cleanly, and forward into a small clean-up of the lexer itself. The canonicalization-at-parse-time count remains at eight, because this is a different shape of move (canonicalization-at-pp-token-conversion-time, not canonicalization-at-parse-time), but it's in the same family.

### `-D` and `-U` (commits fc69f5c, be8b6f6)

Two adjacent commits. The first adds `-D macro=value` (and `-Dmacro=value`) to the driver, defining a macro from the command line. The second adds `-U macro` (and `-Umacro`), undefining one.

The driver's argument-parsing loop gains:

```c
if (!strcmp(argv[i], "-D")) {
  define(argv[++i]);
  continue;
}

if (!strncmp(argv[i], "-D", 2)) {
  define(argv[i] + 2);
  continue;
}

// Same shape for -U.
```

Both `-D foo` (with separator) and `-Dfoo` (joined) are accepted, mirroring GCC. The `define` helper splits on `=`:

```c
static void define(char *str) {
  char *eq = strchr(str, '=');
  if (eq)
    define_macro(strndup(str, eq - str), eq + 1);
  else
    define_macro(str, "1");
}
```

`define_macro` and `undef_macro` were already present in `preprocess.c` as `static` helpers; this commit unstatics them and exposes them through `chibicc.h`. The `-D` commit also moves the `init_macros()` call from inside `preprocess()` to the top of `main` — before argument parsing — so that `-D` arguments override any default predefined macros if the user re-defines one.

The `-U` commit replaces the inline two-liner inside `preprocess2`'s `#undef` handler with a call to the now-public `undef_macro`:

```c
// Before:
char *name = strndup(tok->loc, tok->len);
tok = skip_line(tok->next);
Macro *m = add_macro(name, true, NULL);
m->deleted = true;

// After:
undef_macro(strndup(tok->loc, tok->len));
tok = skip_line(tok->next);
```

The driver tests in `test/driver.sh` exercise the new flags:

```sh
echo foo | $chibicc -Dfoo -E - | grep -q 1
echo foo | $chibicc -Dfoo=bar -E - | grep -q bar
echo foo | $chibicc -Dfoo=bar -Ufoo -E - | grep -q foo
```

`-Dfoo` defines `foo` to `1`; `-Dfoo=bar` defines it to `bar`; `-Dfoo=bar -Ufoo` then undefines it again (and the input passes through unchanged).

The `StringArray` type from Chapter 16 §16.4 doesn't grow new users — `-D` and `-U` apply their effects at parse time directly, rather than queueing the strings.

### Where we are

Four small completions. Function-dereference is a four-line parser shortcut that reflects a specific oddity in the C spec. Pp-numbers replace the tokenizer's old direct-to-`TK_NUM` path with a permissive `TK_PP_NUM` lexer plus a separate interpretation pass; this closes the `##`-pasting gap and aligns chibicc's lexer with the C standard's preprocessor-tokens-vs-tokens phase distinction. `-D` and `-U` extend the driver and unstatic two helpers in `preprocess.c`. None of these is large; together, they tidy up four small incongruities.

---

## 18.5 — Bitfields

> `git checkout cc852fe99d0acfc6d547b36c75ff85e90975ad36` — *Add bitfield*
>
> `git checkout 441a89b80babf98d3feb13e4594ee01eb6cc4dd5` — *Support global struct bitfield initializer*
>
> `git checkout 54c2b3b18fb80235ad9ee53cac3966e8aad9e12a` — *Handle op=-style assignments to bitfields*
>
> `git checkout 17ea802ceaa76f55726488379959a983f891f631` — *Handle zero-width bitfield member*
>
> `git checkout c302a969d8217ab46113d494b8cd773cf057193d` — *Do not allow to obtain an address of a bitfield*

Five commits. The chapter's most subtle arc after struct-by-value. Bitfields are a corner of C that most C programmers ignore until they have to read a hardware register layout, and then suddenly need every detail. Chibicc implements the full bitfield surface — declaration syntax, layout, read codegen, write codegen, op-assign, zero-width, and address-of-restriction — across these five commits.

A bitfield is a struct member declared with a bit count: `int x : 3;` declares a member named `x` that occupies three bits. The C standard's layout rules are partially implementation-defined, but the standard does require: bitfields share storage when consecutive (so `struct { int a:3; int b:5; }` packs both into one byte if alignment permits); an unnamed bitfield with a zero width forces alignment to the next storage unit; the address operator `&` is illegal on a bitfield (because a bitfield doesn't necessarily have a byte address).

### The base bitfield commit (commit cc852fe)

`Member` gains three fields:

```c
struct Member {
  // ...
  int idx;
  int align;
  int offset;

  // Bitfield
  bool is_bitfield;
  int bit_offset;
  int bit_width;
};
```

`is_bitfield` distinguishes bitfield members from ordinary ones. `bit_offset` is the offset of the bitfield within its storage unit (0–63 for an `int` or `long` storage unit). `bit_width` is the number of bits the bitfield occupies.

`struct_members` learns to parse the `: width` suffix:

```c
if (consume(&tok, tok, ":")) {
  mem->is_bitfield = true;
  mem->bit_width = const_expr(&tok, tok);
}
```

`struct_decl`'s offset-assignment loop is rewritten in terms of *bits* rather than bytes:

```c
int bits = 0;

for (Member *mem = ty->members; mem; mem = mem->next) {
  if (mem->is_bitfield) {
    int sz = mem->ty->size;
    if (bits / (sz * 8) != (bits + mem->bit_width - 1) / (sz * 8))
      bits = align_to(bits, sz * 8);

    mem->offset = align_down(bits / 8, sz);
    mem->bit_offset = bits % (sz * 8);
    bits += mem->bit_width;
  } else {
    bits = align_to(bits, mem->align * 8);
    mem->offset = bits / 8;
    bits += mem->ty->size * 8;
  }

  if (ty->align < mem->align)
    ty->align = mem->align;
}

ty->size = align_to(bits, ty->align * 8) / 8;
```

`bits` tracks the running offset in bits. For an ordinary member, `bits` rounds up to the member's alignment (in bits, hence `mem->align * 8`), the member's byte offset is `bits / 8`, and `bits` advances by the member's size (in bits). For a bitfield, the rule is more subtle. The check `bits / (sz * 8) != (bits + mem->bit_width - 1) / (sz * 8)` tests whether the bitfield, if placed at the current `bits`, would *cross* a storage-unit boundary. If yes, `bits` rounds up to the next storage-unit boundary so the bitfield doesn't span across. The bitfield's `offset` is the storage unit's byte address (rounded *down*), and the `bit_offset` is the bitfield's position within that storage unit.

`align_down(n, align)` is a new one-line helper:

```c
static int align_down(int n, int align) {
  return align_to(n - align + 1, align);
}
```

It rounds `n` down to a multiple of `align`. Used here because the bitfield's storage unit is anchored at the largest aligned byte address ≤ `bits / 8`.

Read codegen, in `gen_expr`'s `ND_MEMBER` case:

```c
case ND_MEMBER: {
  gen_addr(node);
  load(node->ty);

  Member *mem = node->member;
  if (mem->is_bitfield) {
    println("  shl $%d, %%rax", 64 - mem->bit_width - mem->bit_offset);
    if (mem->ty->is_unsigned)
      println("  shr $%d, %%rax", 64 - mem->bit_width);
    else
      println("  sar $%d, %%rax", 64 - mem->bit_width);
  }
  return;
}
```

The trick is the shift-left then arithmetic-or-logical-shift-right sequence. After `load`, `%rax` holds the storage unit (sign-extended or zero-extended to 64 bits, per the member's type). To extract the bitfield's value:

1. Shift left by `64 - bit_width - bit_offset`. This puts the bitfield's most significant bit at bit 63 (the sign bit of `%rax`).
2. Shift right by `64 - bit_width`, either logically (`shr`, for unsigned) or arithmetically (`sar`, for signed). This places the bitfield at the low bits of `%rax`, sign-extending or zero-extending appropriately.

Two shifts; sign extension or zero extension comes for free. This is the canonical bitfield-extract pattern in compilers with arithmetic-right-shift.

Write codegen, in `gen_expr`'s `ND_ASSIGN` case:

```c
if (node->lhs->kind == ND_MEMBER && node->lhs->member->is_bitfield) {
  // If the lhs is a bitfield, we need to read the current value
  // from memory and merge it with a new value.
  Member *mem = node->lhs->member;
  println("  mov %%rax, %%rdi");
  println("  and $%ld, %%rdi", (1L << mem->bit_width) - 1);
  println("  shl $%d, %%rdi", mem->bit_offset);

  println("  mov (%%rsp), %%rax");
  load(mem->ty);

  long mask = ((1L << mem->bit_width) - 1) << mem->bit_offset;
  println("  mov $%ld, %%r9", ~mask);
  println("  and %%r9, %%rax");
  println("  or %%rdi, %%rax");
}

store(node->ty);
```

Read-modify-write through registers. `%rax` holds the new value; mask it to `bit_width` bits and shift it into `bit_offset` position, leaving the prepared insert-bits in `%rdi`. Then load the *current* storage unit through `%rsp` (which still holds the address of the storage unit, set by an earlier `gen_addr; push;` pair). Mask out the bitfield's bits in the current value (using `~mask` in `%r9`). Or in the prepared bits. Store the merged result.

The new test exercises packing and signed/unsigned reads:

```c
struct bit1 {
  short a;
  char b;
  int c : 2;
  int d : 3;
  int e : 3;
};

ASSERT(4, sizeof(struct bit1));
ASSERT(-1, ({ struct bit1 x={1,2,3,4,5}; x.c; }));   // 3 in 2 signed bits = -1
ASSERT(-4, ({ struct bit1 x={1,2,3,4,5}; x.d; }));   // 4 in 3 signed bits = -4
```

`struct bit1` packs `short a; char b;` (three bytes), then bitfields `c:2; d:3; e:3` totaling eight bits. The eight bits fit in the fourth byte of an `int` storage unit; `sizeof(struct bit1)` is four bytes. The bitfields are signed (`int`, not `unsigned int`), so the high bit sets the sign — `3` in two signed bits is `-1`, `4` in three signed bits is `-4`.

### Global bitfield initializer (commit 441a89b)

The base commit handles bitfield reads and writes at runtime. Initializing a bitfield in a *global* struct happens at compile time, not runtime — `write_gvar_data` has to compute the merged storage-unit value as a constant, then emit it as data.

A new helper `read_buf` is the inverse of the existing `write_buf`:

```c
static uint64_t read_buf(char *buf, int sz) {
  if (sz == 1)
    return *buf;
  if (sz == 2)
    return *(uint16_t *)buf;
  if (sz == 4)
    return *(uint32_t *)buf;
  if (sz == 8)
    return *(uint64_t *)buf;
  unreachable();
}
```

`write_gvar_data` for `TY_STRUCT` learns the bitfield case:

```c
for (Member *mem = ty->members; mem; mem = mem->next) {
  if (mem->is_bitfield) {
    Node *expr = init->children[mem->idx]->expr;
    if (!expr)
      break;

    char *loc = buf + offset + mem->offset;
    uint64_t oldval = read_buf(loc, mem->ty->size);
    uint64_t newval = eval(expr);
    uint64_t mask = (1L << mem->bit_width) - 1;
    uint64_t combined = oldval | ((newval & mask) << mem->bit_offset);
    write_buf(loc, combined, mem->ty->size);
  } else {
    cur = write_gvar_data(cur, init->children[mem->idx], mem->ty, buf,
                          offset + mem->offset);
  }
}
```

Read the current storage-unit value out of the buffer, evaluate the initializer expression as a constant, mask the new value to `bit_width` bits, shift it into position, OR with the existing bits, write back. Same shape as the runtime read-modify-write, but operating on the gvar's data buffer rather than memory.

`if (!expr) break;` handles the case where the initializer doesn't supply a value for the bitfield (common with `= {}` or short-form initializers); subsequent bitfields in the same storage unit don't get processed. This is a small simplification — a fully correct implementation would zero them — but `write_gvar_data` is called on a buffer that was zeroed by `calloc`, so the omitted bitfields are already zero by construction.

The test:

```c
struct {
  char a;
  int b : 5;
  int c : 10;
} g45 = {1, 2, 3}, g46={};

ASSERT(1, g45.a);
ASSERT(2, g45.b);
ASSERT(3, g45.c);
ASSERT(0, g46.a);
ASSERT(0, g46.b);
ASSERT(0, g46.c);
```

### Op-assign on bitfields (commit 54c2b3b)

The hardest commit in the bitfield arc. `x.bf += 1` is, by C's rules, equivalent to `x.bf = x.bf + 1`. The pre-commit `to_assign` rewrote `A op= C` into `tmp = &A, *tmp = *tmp op C` — this *almost* works for bitfields but breaks because `&A` is illegal for bitfields, and even if it weren't, `*tmp` would yield the storage unit, not the bitfield value.

Rui's solution is a different rewrite for bitfield op-assigns. `A.x op= C` becomes `tmp = &A, (*tmp).x = (*tmp).x op C`:

```c
// Convert `A.x op= C` to `tmp = &A, (*tmp).x = (*tmp).x op C`.
if (binary->lhs->kind == ND_MEMBER) {
  Obj *var = new_lvar("", pointer_to(binary->lhs->lhs->ty));

  Node *expr1 = new_binary(ND_ASSIGN, new_var_node(var, tok),
                           new_unary(ND_ADDR, binary->lhs->lhs, tok), tok);

  Node *expr2 = new_unary(ND_MEMBER,
                          new_unary(ND_DEREF, new_var_node(var, tok), tok),
                          tok);
  expr2->member = binary->lhs->member;

  Node *expr3 = new_unary(ND_MEMBER,
                          new_unary(ND_DEREF, new_var_node(var, tok), tok),
                          tok);
  expr3->member = binary->lhs->member;

  Node *expr4 = new_binary(ND_ASSIGN, expr2,
                           new_binary(binary->kind, expr3, binary->rhs, tok),
                           tok);

  return new_binary(ND_COMMA, expr1, expr4, tok);
}
```

`tmp` is a pointer to the *outer* struct (not the bitfield). `expr1` assigns `&A` into `tmp`. `expr2` and `expr3` are two separate `ND_MEMBER` nodes, both `(*tmp).x` (with `.member` set to the bitfield member). `expr4` is the assignment: `(*tmp).x = (*tmp).x op C`. The whole thing wraps in a comma.

The reason for *two* `ND_MEMBER` nodes (not one shared) is that codegen will walk each independently — `expr2` becomes the lvalue of the assignment, and `expr3` becomes the rvalue. They share the bitfield member descriptor but not the AST node.

The rewrite applies to *all* op-assigns on `ND_MEMBER` lvalues, not just bitfield ones. For ordinary struct members the new rewrite is functionally equivalent to the old one (both compute `&A` once and then read-modify-write through it); for bitfields, the new rewrite is the only one that works because it routes through the bitfield-aware read and write codegen rather than through ordinary `*tmp`.

The codegen side of this commit is small. The bitfield write path saves and restores `%rax`, so the *value* of the assignment expression (which is the new value of the bitfield, per C's rules) survives:

```c
if (node->lhs->kind == ND_MEMBER && node->lhs->member->is_bitfield) {
  println("  mov %%rax, %%r8");
  // ... [existing read-modify-write] ...
  store(node->ty);
  println("  mov %%r8, %%rax");
  return;
}

store(node->ty);
```

Without the save-and-restore, the assignment expression's value would be the *storage unit's* value (since `store` returns the merged storage unit), which is wrong — `++x.bf` would yield the post-increment storage unit, not the post-increment bitfield. The `mov %%rax, %%r8` saves the new bitfield value before the read-modify-write clobbers `%rax`; the `mov %%r8, %%rax` restores it after the store.

The test:

```c
ASSERT(1, ({ T3 x={1,2,3}; x.a++; }));   // post-increment yields old value
ASSERT(2, ({ T3 x={1,2,3}; x.b++; }));
ASSERT(3, ({ T3 x={1,2,3}; x.c++; }));

ASSERT(2, ({ T3 x={1,2,3}; ++x.a; }));   // pre-increment yields new value
ASSERT(3, ({ T3 x={1,2,3}; ++x.b; }));
ASSERT(4, ({ T3 x={1,2,3}; ++x.c; }));
```

### Zero-width bitfield (commit 17ea802)

Tiny commit. A zero-width unnamed bitfield (`int :0;`) means "force alignment to the next storage unit boundary." It contributes no member, but it does shift `bits` to the next aligned position.

The parser change is six lines in `struct_decl`:

```c
if (mem->is_bitfield && mem->bit_width == 0) {
  // Zero-width anonymous bitfield has a special meaning.
  // It affects only alignment.
  bits = align_to(bits, mem->ty->size * 8);
} else if (mem->is_bitfield) {
  // ... [existing bitfield path] ...
} else {
  // ... [existing ordinary-member path] ...
}
```

The test:

```c
ASSERT(4, sizeof(struct {int a:3; int c:1; int c:5;}));   // packs into 4 bytes
ASSERT(8, sizeof(struct {int a:3; int:0; int c:5;}));     // :0 forces 8 bytes
ASSERT(4, sizeof(struct {int a:3; int:0;}));              // :0 still leaves 4 bytes
```

The first case packs three bitfields totaling nine bits into one `int` storage unit — sizeof is four. The second case has `int:0` between `a:3` and `c:5`, which forces `c:5` into a *new* `int` storage unit — sizeof is eight. The third case is `a:3` followed by `int:0`; the `:0` aligns to four bytes but doesn't add anything past it, so sizeof remains four.

The two `c` members in the first test (`int c:1; int c:5;`) are a quiet incongruity — they're two members with the same name, which the C standard rejects. Chibicc accepts it because chibicc's struct-member-redeclaration check is missing (one of the redeclaration errata candidates flagged through the book). The test still passes because chibicc never tries to *read* `c` in this case — it only queries `sizeof`. But if the test read `c`, it would silently pick whichever member won the lookup.

### Address-of restriction (commit c302a96)

The C standard prohibits `&` on a bitfield. The reason is that bitfields don't have byte-aligned addresses — a bitfield in the middle of a storage unit isn't addressable through an ordinary pointer.

Three lines:

```c
if (equal(tok, "&")) {
  Node *lhs = cast(rest, tok->next);
  add_type(lhs);
  if (lhs->kind == ND_MEMBER && lhs->member->is_bitfield)
    error_tok(tok, "cannot take address of bitfield");
  return new_unary(ND_ADDR, lhs, tok);
}
```

The error fires before `ND_ADDR` is constructed. Pre-commit, `&s.bf` would have produced an `ND_ADDR` node whose codegen would have generated `lea offset(%rbp), %rax` for the storage unit's address — confusing and wrong. Post-commit, the parser rejects the construct outright.

### Where we are

Five commits. The five-line `Member` extension carries the bitfield's metadata (`is_bitfield`, `bit_offset`, `bit_width`). The `struct_decl` layout loop is rewritten in terms of bits, with a storage-unit-crossing rule and the zero-width alignment override. Reads use a shift-left-then-arithmetic-shift-right pair to extract the value. Writes use a register-resident read-modify-write. Op-assign uses a parser-side rewrite that re-derives the bitfield member through a `(*tmp).x` aliasing trick. Address-of is a one-line parser rejection.

The bitfield arc adds one to the canonicalization-at-parse-time count — the op-assign rewrite from `A.x op= C` to `tmp = &A, (*tmp).x = (*tmp).x op C` is canonicalized in `to_assign` before codegen sees it. New count: nine.

The psABI conformance count ticks up by one. New count: fifteen.

The pre-factor-before-feature count gains one — the `Member` field extension in commit `cc852fe` lays the data structure that the next four bitfield commits all build on. New count: nine.

---

## 18.6 — Polish and tail

> `git checkout 2bdc6b800c1dbe6db584b91046785d4c48c41fb2` — *Write to an in-memory buffer before writing to an actual output file*
>
> `git checkout b1fdddff1523d2ca7bab4050434499d3a5ac39a1` — *Ignore -O, -W and -g and other flags*
>
> `git checkout 2c91da54dff93a365feec5a34f8eaeccca3e3a70` — *Turn on -Wall compiler flag and fix compiler warnings*
>
> `git checkout 5257ee0f202a5f9c4e5bcb576646cefe70f3ae91` — *Make an array of at least 16 bytes long to have alignment of at least 16 bytes*
>
> `git checkout 9c36dd727c736dc3a3ffa6ce7ce473966d802068` — *Make "main" to implicitly return 0*
>
> `git checkout c3075b3030c0488df1e7aa9f600da0f66072186b` — *Add anonymous struct and union*

Six small commits. None deserves a section of its own. Together, they tidy up a handful of remaining loose ends.

### Buffered output (commit 2bdc6b8)

The driver's `cc1` function used to write assembly directly to the output file. This commit redirects through an `open_memstream` buffer:

```c
char *buf;
size_t buflen;
FILE *output_buf = open_memstream(&buf, &buflen);

codegen(prog, output_buf);
fclose(output_buf);

FILE *out = open_file(output_file);
fwrite(buf, buflen, 1, out);
fclose(out);
```

The reason — quoted from Rui's commit message: "We don't want to leave a partial assembly output if the compiler fails during compilation." If `codegen` aborts midway (through `error_tok` or any other failure path), `output_buf`'s memory buffer disappears with the process; the on-disk output file is never opened in the first place. Compare the pre-commit behavior: a half-written `.s` file sits on disk after a compiler error, the next `make` step (the assembler) sees it and fails on parse, and the user is confused about whether the cc1 error or the as error is the real one.

The commit message also notes a remaining hole: "there's still a risk of leaving a partially-written output file if the compiler dies during file copy." A fully robust approach would write to a temporary file in the same filesystem, then `rename` it to atomically replace the output file. Rui leaves that for future work. The half-step still catches the common failure mode (compiler error before output-file open), which is what matters.

`open_memstream` is a POSIX function that returns a `FILE *` writing to an in-memory buffer; the buffer pointer and length are made available through pointer-to-pointer and pointer-to-size out-parameters. The buffer is `malloc`-allocated and the caller takes ownership. (Chibicc doesn't `free` the buffer here — the process is exiting after `cc1` returns successfully, so the memory leak is harmless.)

### Ignored flags (commit b1fdddf)

`gcc` accepts a long tail of flags that affect optimization, warnings, debug info, language standard, and runtime model. Real Makefiles and configure-detected build systems set them by default. A driver that errors on unknown flags can't be a drop-in replacement.

This commit adds a list of flags chibicc accepts and ignores:

```c
if (!strncmp(argv[i], "-O", 2) ||
    !strncmp(argv[i], "-W", 2) ||
    !strncmp(argv[i], "-g", 2) ||
    !strncmp(argv[i], "-std=", 5) ||
    !strcmp(argv[i], "-ffreestanding") ||
    !strcmp(argv[i], "-fno-builtin") ||
    !strcmp(argv[i], "-fno-omit-frame-pointer") ||
    !strcmp(argv[i], "-fno-stack-protector") ||
    !strcmp(argv[i], "-fno-strict-aliasing") ||
    !strcmp(argv[i], "-m64") ||
    !strcmp(argv[i], "-mno-red-zone") ||
    !strcmp(argv[i], "-w"))
  continue;
```

`-O*`, `-W*`, `-g*`, `-std=*` cover the prefix flags. The rest are individual flags chibicc has been observed to receive from real build invocations. None of them changes chibicc's behavior — chibicc has no optimizer, no warning system beyond what its own checks emit, no debug info beyond `.loc` directives, no notion of language standard variants, and no runtime model that would care about red zones or stack protectors. Accepting and ignoring them is the right move: the user gets predictable behavior (the flag is silently accepted) and chibicc doesn't pretend to honor what it can't.

The TODO is implicit: a future commit could honor each flag, or some, or warn that unrecognized ones are ignored. Chibicc never gets there; the ignored-list is the final shape.

The driver test exercises the new flags, with `/dev/null` as the output to discard the produced object:

```sh
$chibicc -c -O -Wall -g -std=c11 -ffreestanding -fno-builtin \
         -fno-omit-frame-pointer -fno-stack-protector -fno-strict-aliasing \
         -m64 -mno-red-zone -w -o /dev/null $tmp/empty.c
```

### `-Wall`-clean self-build (commit 2c91da5)

The Makefile's `CFLAGS` gains `-Wall -Wno-switch`, turning on most GCC warnings while suppressing the warning about non-exhaustive `switch` statements (which chibicc's source has a lot of, intentionally — many `switch` statements only handle the cases of interest and let the default flow fall through).

The actual warnings that `-Wall` catches are addressed in `chibicc.h`: three `error*` declarations get the `noreturn` attribute:

```c
noreturn void error(char *fmt, ...);
noreturn void error_at(char *loc, char *fmt, ...);
noreturn void error_tok(Token *tok, char *fmt, ...);
```

The `noreturn` annotation tells GCC that these functions don't return, so that callers don't need to handle the no-return-value case after them. Pre-annotation, GCC was warning about lots of code paths like:

```c
case TY_FOO:
  // ...
case TY_BAR:
  // ...
default:
  error_tok(tok, "unexpected type");
}

return result;   // GCC: warning: no return after default — but error_tok doesn't return
```

— at every place where `error_*` was the last statement of a path that "should" return. After the `noreturn` annotation, GCC understands that the path doesn't actually fall through, and the warnings disappear.

The `noreturn` keyword is from `<stdnoreturn.h>` (C11), which is added to chibicc's `#include` list. Its definition (post-Chapter-17) is in chibicc's own `<stdnoreturn.h>`, a small alias header.

What's interesting about this commit is what it *implies* about chibicc's source quality. `-Wall` catches the common bugs — uninitialized local variables, signed/unsigned compares, missing parens, unused variables. None of those show up in this commit's diff. The only thing `-Wall` flagged was the missing `noreturn` annotations. That means chibicc's source was already (mostly) `-Wall`-clean before this commit. The commit isn't fixing latent bugs; it's tightening a static-check configuration that would catch them if they appeared in the future.

### 16-byte array alignment (commit 5257ee0)

The SysV AMD64 psABI specifies that arrays of length at least sixteen bytes get at least sixteen-byte alignment. Rui's commit message quotes the spec directly: "An array uses the same alignment as its elements, except that a local or global array variable of length at least 16 bytes or a C99 variable-length array variable always has alignment of at least 16 bytes." This is an SSE-friendliness rule — sixteen-byte alignment is the hardware's preferred boundary for `movaps` and friends.

Two places in `codegen.c`:

```c
// In assign_lvar_offsets:
int align = (var->ty->kind == TY_ARRAY && var->ty->size >= 16)
  ? MAX(16, var->align) : var->align;

bottom += var->ty->size;
bottom = align_to(bottom, align);
var->offset = -bottom;

// In emit_data:
int align = (var->ty->kind == TY_ARRAY && var->ty->size >= 16)
  ? MAX(16, var->align) : var->align;
println("  .align %d", align);
```

The condition is the same in both: array type, size at least sixteen bytes. The alignment is the maximum of sixteen and whatever the array would have asked for otherwise (which might be larger if the element type itself wants more, e.g. a `long double` array — though chibicc's `long double` is `double`, so this case doesn't arise in practice).

The test in `test/alignof.c`:

```c
ASSERT(0, ({ char x[16]; (unsigned long)&x % 16; }));
ASSERT(0, ({ char x[17]; (unsigned long)&x % 16; }));
ASSERT(0, ({ char x[100]; (unsigned long)&x % 16; }));
ASSERT(0, ({ char x[101]; (unsigned long)&x % 16; }));
```

Every array of at least sixteen bytes is sixteen-byte-aligned. (The `char` array's natural alignment is one byte; without this rule, the array would have been one-byte-aligned.)

The rule applies to arrays specifically; struct alignment is unchanged. A struct of sixteen bytes whose members are all `char` would still have one-byte alignment under the existing rules — which is the correct behavior, since the psABI's sixteen-byte rule is array-specific.

The psABI conformance count ticks up by one. New count: sixteen.

### `main` implicitly returns 0 (commit 9c36dd7)

The C standard has a special case for `main`: reaching the end of `main` without an explicit `return` is equivalent to `return 0`. Other functions that fall off the end have undefined behavior; `main` is the lone exception.

Chibicc's implementation is seven lines in `emit_text`:

```c
gen_stmt(fn->body);
assert(depth == 0);

// [https://www.sigbus.info/n1570#5.1.2.2.3p1] The C spec defines
// a special rule for the main function. Reaching the end of the
// main function is equivalent to returning 0, even though the
// behavior is undefined for the other functions.
if (strcmp(fn->name, "main") == 0)
  println("  mov $0, %%rax");

// Epilogue
```

A `mov $0, %%rax` immediately before the function's epilogue. If `main` reaches that point (because no `return` statement jumped to `.L.return.main` first), `%rax` is zero, and the epilogue's `ret` returns zero. If a `return` statement *did* fire, the `jmp .L.return.main` skips over this `mov`, so the explicit return value is preserved.

The test suite shrinks slightly — `test/function.c`'s closing `return 0;` is removed, since `main` now implicitly returns zero.

### Anonymous struct and union (commit c3075b3)

The chapter's last commit. Anonymous struct/union members let an inner struct's fields be addressed directly through the outer struct:

```c
struct {
  struct { int a, b; };
  int c;
} s;

s.a = 1;     // reaches the inner struct's `a`
s.b = 2;     // reaches the inner struct's `b`
s.c = 3;     // reaches the outer struct's `c`
```

The inner `struct { int a, b; }` declaration has no name (no member name, not a tag; tags are separate). The members of that inner struct flatten into the outer struct's lookup namespace.

`struct_members` learns the anonymous-member case:

```c
// Anonymous struct member
if ((basety->kind == TY_STRUCT || basety->kind == TY_UNION) &&
    consume(&tok, tok, ";")) {
  Member *mem = calloc(1, sizeof(Member));
  mem->ty = basety;
  mem->idx = idx++;
  mem->align = attr.align ? attr.align : mem->ty->align;
  cur = cur->next = mem;
  continue;
}
```

The detection: a struct or union type immediately followed by `;` (no declarator). The member is added with no `name` field — `mem->name` is `NULL`. That's the marker.

`get_struct_member` and `struct_ref` together implement the lookup rule. `get_struct_member` returns the *member* whose name matches, or — for an anonymous member — the anonymous member itself if a recursive search through it finds the name:

```c
static Member *get_struct_member(Type *ty, Token *tok) {
  for (Member *mem = ty->members; mem; mem = mem->next) {
    // Anonymous struct member
    if ((mem->ty->kind == TY_STRUCT || mem->ty->kind == TY_UNION) &&
        !mem->name) {
      if (get_struct_member(mem->ty, tok))
        return mem;
      continue;
    }

    // Regular struct member
    if (mem->name->len == tok->len &&
        !strncmp(mem->name->loc, tok->loc, tok->len))
      return mem;
  }
  return NULL;
}
```

The recursion mirrors C's lookup rule: look at this layer's regular members; if not found, descend into each anonymous member and search there. If found at any depth, the *outermost* anonymous member is returned, not the deeply-nested final member. The reason: `s.a` accesses `s.[anon].a`, so the AST needs to model two member-accesses (one through the anonymous member, one to `a`).

`struct_ref` builds the chain:

```c
static Node *struct_ref(Node *node, Token *tok) {
  add_type(node);
  if (node->ty->kind != TY_STRUCT && node->ty->kind != TY_UNION)
    error_tok(node->tok, "not a struct nor a union");

  Type *ty = node->ty;

  for (;;) {
    Member *mem = get_struct_member(ty, tok);
    if (!mem)
      error_tok(tok, "no such member");
    node = new_unary(ND_MEMBER, node, tok);
    node->member = mem;
    if (mem->name)
      break;
    ty = mem->ty;
  }
  return node;
}
```

A loop that builds one `ND_MEMBER` node per layer until it reaches a named member. Each iteration: look up the member at this layer, add an `ND_MEMBER` node, and — if the member is anonymous — descend into its type and look up the same name at the next layer.

The test exercises both forms:

```c
ASSERT(0xef, ({ union { struct { unsigned char a,b,c,d; }; long e; } x; x.e=0xdeadbeef; x.a; }));
ASSERT(0xbe, ({ union { struct { unsigned char a,b,c,d; }; long e; } x; x.e=0xdeadbeef; x.b; }));

ASSERT(3, ({struct { union { int a,b; }; union { int c,d; }; } x; x.a=3; x.b; }));
ASSERT(5, ({struct { union { int a,b; }; union { int c,d; }; } x; x.d=5; x.c; }));
```

The first two cases overlay an anonymous struct of four `unsigned char`s with a `long`, then read the bytes back individually after assigning the long. The result is little-endian — `x.a` is the least significant byte (`0xef`), `x.d` is the most significant (`0xde`).

The last two cases use anonymous union members. Two anonymous unions side by side: the first contains `a` and `b` (sharing storage), the second contains `c` and `d` (sharing storage but distinct from the first union). Assigning `x.a = 3` makes `x.b == 3` (same storage). Assigning `x.d = 5` makes `x.c == 5` (same storage). The two unions are at different offsets within the outer struct.

### Where we are

Six small commits, six small tidyings.

Buffered output prevents partial `.s` files on compile errors. Ignored flags make chibicc accept the GCC vocabulary that real build systems use. `-Wall`-clean self-build catches future bugs in chibicc's source by failing the build if any creep in. Sixteen-byte array alignment matches the psABI's array rule. Implicit `return 0` for `main` matches the C standard's special case. Anonymous structs and unions add the last small struct-system feature missing through Chapter 17.

The cast table at 10×10 doesn't grow. The fourth namespace (labels) is unchanged. The `Initializer` tree is unchanged. The `Relocation` mechanism is unchanged. The `Member->idx` field is unchanged (the bitfield extension added new fields but `idx` keeps its old role). The `is_unsigned` flag picks up a new reader in the bitfield read codegen (the `shr`-vs-`sar` choice).

---

## Where the chapter leaves us

Twenty-three commits. Six sections. The chapter's spine is the SysV AMD64 calling convention's full implementation — stack-passed args and parameters, struct-by-value with eightbyte classification on both caller and callee, variadic-with-overflow, `va_copy`. The bitfield arc is the chapter's longest single subsection after struct-by-value, with five commits that implement declaration syntax, layout, read codegen, write codegen, op-assign, zero-width, and address-of-restriction. The remaining commits are smaller corrections: the function-deref oddity, the pp-number lexical category, command-line `-D`/`-U`, the buffered output, the ignored-flags list, the `-Wall`-clean self-build, sixteen-byte array alignment, `main`'s implicit `return 0`, and anonymous struct/union members.

| Commit | Topic |
|---|---|
| `b29f052` | Stack-passed arguments. `push_args` becomes a counting walk that marks each over-the-limit argument with `pass_by_stack`. `push_args2` runs in two passes (register-bound first, then stack-bound) to keep right-to-left evaluation. Call site moves to `call *%r10` (with `mov $%d, %%rax` for the FP count). Sixteen-byte alignment of `%rsp` is now enforced up front by `push_args`, replacing the per-call kludge. |
| `9021f7f` | Stack-passed parameters. `assign_lvar_offsets` runs two passes — one for over-the-limit parameters at positive offsets above `%rbp + 16`, one for everything else at negative offsets. The prologue's register-copy loop skips parameters with positive offsets. |
| `5e0f8c4` | Struct as parameter, caller side. New `has_flonum`, `has_flonum1`, `has_flonum2` for SysV eightbyte classification. New `push_struct` for stack-passed structs. `push_args` and the pop loop both learn the struct case with the `fp + fp1 + fp2 < FP_MAX && gp + !fp1 + !fp2 < GP_MAX` rule. The pre-commit error for "passing struct or union is not supported yet" is removed. |
| `d63b1f4` | Struct as parameter, callee side. `assign_lvar_offsets` learns the struct case with the same classification logic. The prologue's parameter-store loop learns to split structs across one or two register stores. `store_gp` gains a `default` case for non-power-of-two sizes (byte-by-byte through `%al` with shift-rights). |
| `c72df1c` | Function returning a struct, caller side. `Node` gains a `ret_buffer` field; `funcall` allocates one for struct-returning calls. `gen_addr` handles `ND_FUNCALL` with `ret_buffer`. `push_args` reserves `%rdi` for the hidden buffer pointer when `ty->size > 16`. New `copy_ret_buffer` disassembles `%rax`/`%rdx` and `%xmm0`/`%xmm1` into the buffer. The call site reloads the buffer's address into `%rax` so member-access works on `f().x`. |
| `d7bad96` | Function returning a struct, callee side. `function` allocates a hidden first-parameter pointer for `> 16`-byte returns. The return statement skips the cast for struct returns. New `copy_struct_reg` (for `<= 16`) packs the return into `%rax`/`%rdx`/`%xmm0`/`%xmm1`. New `copy_struct_mem` (for `> 16`) byte-copies into the caller's buffer. |
| `b6d3cd0` | Variadic with stack-resident fixed parameters. `__va_arg_mem` learns to walk `overflow_arg_area` (replacing the `1/0; // not implemented` placeholder). `__va_arg_gp` and `__va_arg_fp` fall back to `__va_arg_mem` when their register-save areas are exhausted. The prologue initializes `overflow_arg_area`. The `va_arg` macro passes `sizeof(ty)` and `_Alignof(ty)` through. |
| `603de50` | `va_copy`. Single-line macro: `((dest)[0] = (src)[0])`. Relies on `va_list` being an array, so array-decay makes the index-and-assign pattern work as an ordinary struct assignment. |
| `e0b5da3` | Function dereference. Four-line parser shortcut: `*f`, `**f`, `*****f` are all equivalent to `f` when `f` is function-typed. Codegen's load-skip for `TY_FUNC` (Chapter 16 §16.2) would have produced the right result anyway, but the parser-side handling matches the C spec wording. |
| `3f2c2d5` | Pp-numbers. New `TK_PP_NUM` token kind. The tokenizer captures the broad pp-number lexical pattern (digits, letters, periods, plus `e+`/`E-`/`p+`/`P-`). `convert_pp_int` and `convert_pp_number` reinterpret each `TK_PP_NUM` after preprocessing. `convert_keywords` becomes `convert_pp_tokens`, picking up the second responsibility. Closes the `##`-pasting gap for non-trivial number constructions like `CONCAT(4, .57)`. |
| `fc69f5c` | `-D`. Driver accepts both `-Dfoo=bar` (joined) and `-D foo=bar` (separated). `define_macro` is unstaticed and called from the driver. `init_macros` moves to before argument parsing so command-line `-D` can override predefineds. |
| `be8b6f6` | `-U`. Driver accepts both `-Ufoo` and `-U foo`. `undef_macro` is the public version of the inline `#undef`-handler that already existed. |
| `cc852fe` | Bitfield (base). `Member` gains `is_bitfield`, `bit_offset`, `bit_width`. `struct_members` parses `: width`. `struct_decl`'s layout loop is rewritten in bits with a storage-unit-crossing rule. New `align_down` helper. Read codegen uses `shl` then `shr`/`sar` to extract. Write codegen does a register-resident read-modify-write. |
| `441a89b` | Global bitfield initializer. New `read_buf` (inverse of `write_buf`). `write_gvar_data`'s struct case learns to merge bitfield initializers into the storage unit at compile time. |
| `54c2b3b` | Bitfield op-assign. `to_assign` rewrites `A.x op= C` to `tmp = &A, (*tmp).x = (*tmp).x op C` — separately from the ordinary `tmp = &A, *tmp = *tmp op C` rewrite — so the bitfield's read and write codegen both fire. The codegen save-and-restores `%rax` through `%r8` so the assignment expression's value is the bitfield's new value, not the storage unit's. |
| `17ea802` | Zero-width bitfield. Six-line parser rule: `int :0;` aligns `bits` to the next storage unit and contributes no member. |
| `c302a96` | Address-of bitfield rejected. Three-line parser rule: `&` on a bitfield member is a parse error. |
| `2bdc6b8` | Buffered output. `cc1` writes to an `open_memstream` buffer first, then dumps to the output file. Avoids partial `.s` files on compile errors. |
| `b1fdddf` | Ignored flags. Driver accepts and ignores `-O*`, `-W*`, `-g*`, `-std=*`, plus a list of `-f*`/`-m*` flags that real build systems set. |
| `2c91da5` | `-Wall`-clean self-build. Makefile's `CFLAGS` adds `-Wall -Wno-switch`. `error*` declarations gain `noreturn`. `<stdnoreturn.h>` added to chibicc's includes. |
| `5257ee0` | Sixteen-byte array alignment. Arrays of size at least sixteen bytes get at-least-sixteen-byte alignment. Applied in `assign_lvar_offsets` and `emit_data`. Per the SysV AMD64 psABI's array rule. |
| `9c36dd7` | `main` implicitly returns 0. Seven-line addition in `emit_text`: a `mov $0, %%rax` immediately before the epilogue when the function name is `main`. |
| `c3075b3` | Anonymous struct and union. `struct_members` recognizes a typed-but-not-named member as anonymous. `get_struct_member` recurses through anonymous-member subtrees. `struct_ref` builds a chain of `ND_MEMBER` nodes — one per anonymous layer plus the final named member. |

Five structural moves carry forward.

The first is the *full SysV AMD64 calling convention*. Through Chapter 17, chibicc handled the easy cases: scalar arguments in registers, scalar returns in `%rax`/`%xmm0`, no overflow. After Chapter 18, the convention is fully implemented at chibicc's level of fidelity — `has_flonum`-driven eightbyte classification on both sides, hidden-pointer protocol for large struct returns, overflow-aware variadic fall-through. The cost is roughly ~250 lines of new codegen, distributed across `push_args`, `push_args2`, `gen_expr`'s `ND_FUNCALL` case, `assign_lvar_offsets`, `emit_text`'s prologue, `gen_stmt`'s `ND_RETURN` case, and the four mirror helpers (`copy_ret_buffer`, `copy_struct_reg`, `copy_struct_mem`, `push_struct`).

The second is *the bitfield system*. Five commits, ~120 lines, distributed across `parse.c` (declaration syntax, layout, op-assign rewrite, address-of restriction), `codegen.c` (read codegen, write codegen, op-assign value preservation), and `chibicc.h` (the three new `Member` fields). Rui pre-factored the data-structure extension — adding `is_bitfield`/`bit_offset`/`bit_width` to `Member` was one move, the layout rewrite was another, the codegen was a third — so each subsequent commit could fill in a corner without disturbing the others. The op-assign commit is the chapter's most subtle parser-level rewrite, requiring two separate `ND_MEMBER` nodes for the lvalue and rvalue sides because the bitfield codegen reads each independently.

The third is *the pp-number lexical category*. The tokenizer is now permissive in what it accepts as a numeric token (TK_PP_NUM), and the parser interprets the text in a separate pass (`convert_pp_number`). Pre-Chapter-18, the tokenizer was a *parser* of numbers — it produced `TK_NUM` directly with `val`/`fval` filled in. Post-Chapter-18, the tokenizer is a *recognizer* — it produces `TK_PP_NUM` with text and lets a later pass do the interpretation. The semantic change is small but the lexer-vs-parser split is now clean enough that future macro-paste shapes (anything that produces a number-shaped token) work without per-shape special cases.

The fourth is *the driver's GCC-compatibility surface*. With `-D`, `-U`, and the ignored-flags list, chibicc accepts the most common GCC flags real build systems set. The unsupported ones (anything not in the recognize-or-ignore list) still error out, which catches typos. The driver remains brittle in the link-path-discovery area (Chapter 16 §16.6's `find_libpath`/`find_gcc_libpath` paths are still hardcoded for Linux/glibc), but the argument-parsing surface is now broad enough that `make` invocations that work for `gcc` also work for `chibicc` in most cases.

The fifth is *small late corrections* — the `2bdc6b8` buffered output, the `5257ee0` array alignment, the `9c36dd7` `main` implicit return, and the `c3075b3` anonymous struct/union. None individually changes chibicc's character; together they push chibicc's ABI conformance and C-spec adherence up by a small but visible amount. The psABI conformance count stands at sixteen after Chapter 18, up from nine at Chapter 17's close. The pre-factor-before-feature count goes from eight to nine (the `Member` field extension in `cc852fe` is the latest entry). The canonicalization-at-parse-time count goes from eight to nine (the bitfield op-assign rewrite in `to_assign` is the latest entry).

Two errata items are *closed* by Chapter 18.

- "More than 6 integer args silently miscompiles" (Chapter 5 §5.4) — closed by `b29f052` and `9021f7f`.
- "More than 8 FP args silently miscompiles" (Chapter 15 §15.6) — closed by `b29f052` and `9021f7f`.
- "Variadic functions whose fixed parameters spill onto the stack don't work yet" (Chapter 14 §14.13) — closed by `b6d3cd0`.

Two errata candidates were *expected* to close but didn't.

- The `fp_offset = fp * 8 + 48` non-conforming stride from Chapter 15 §15.9 is *not* closed. The prologue's `movl $%d, %d(%%rbp)` for `fp_offset` keeps the eight-byte stride, where the ABI mandates a sixteen-byte stride. Errata candidate, still open.
- The `__va_arg_mem` divide-by-zero from Chapter 17 §17.5.3 *is* closed by `b6d3cd0`. (This errata was added at the close of Chapter 17 with the explicit forecast that it would close in Chapter 18; the forecast holds.)

Standing notes worth carrying forward.

- The hideset on Token, the Token->origin chain, the eval-quartet duplication — all unchanged.
- The cc1-vs-driver split — unchanged.
- The Initializer tree gains a small bitfield-related extension in `441a89b` (the `expr` field on bitfield-member initializers is now the bitfield's value rather than a synthetic store node). The `Relocation` mechanism is unchanged because global bitfield initializers are merged into the storage unit's data and don't generate relocations.
- The local-vs-global split is stable.
- The `is_static` default in `new_gvar` — unchanged.
- The `is_definition` flag on `Obj` — unchanged.
- The `is_unsigned` flag on `Type` picks up a new reader (the bitfield read codegen's `shr`-vs-`sar` choice).
- The `__va_area__` magic name — unchanged in chibicc's source.
- The register-save-area layout is unchanged in shape; only the `overflow_arg_area` field's initialization is new.
- The argreg integer/FP split — unchanged.
- The `Member->idx` field gains its bitfield-related siblings (`is_bitfield`/`bit_offset`/`bit_width`) without changing role.
- The `is_flexible` flag — unchanged.
- `copy_struct_type` — unchanged.
- `MIN`/`MAX` macros — `MIN` picks up new uses in `copy_ret_buffer`, `copy_struct_reg`. `MAX` picks up a new use in the sixteen-byte-array-alignment commit.
- `is_numeric` predicate — unchanged.
- `unreachable()` macro — gains new callers in the bitfield buffer-readback (`read_buf`'s default case), and in `store_gp`'s default case (which is now reachable for non-power-of-two struct chunk sizes).
- Per-token line numbers (Chapter 8 §8.3) — unchanged.
- GDB-debuggable output (Chapter 8 §8.4) — unchanged.
- Tests are in C (Chapter 8 §8.2). Driver tests in shell (`test/driver.sh`). Bitfield tests in `test/bitfield.c` (new file, `cc852fe`). Anonymous struct/union tests added to `test/union.c` (`c3075b3`).
- The `Obj->tok` field added in Chapter 14 §14.11 still has no readers.
- The `Type->name_pos` field — no new uses.
- The `>>` codegen quirk — unchanged.
- The `add_type` rule for `ND_STMT_EXPR` — errata candidate, unchanged.
- The hex-escape silent truncation — errata candidate, unchanged.
- The redeclaration-in-same-scope check missing for variables, tags, typedef names, labels — four errata candidates, unchanged. (The bitfield zero-width test case, `struct {int a:3; int c:1; int c:5;}`, exercises the missing struct-member-name redeclaration check — which is in fact a fifth instance, since two members named `c` should produce a parse error.)
- `f()` and `f(void)` are accepted as identical — errata candidate, unchanged.
- Empty brace initializer (`int x[3] = {};`) — chibicc-specific extension, still in use.
- `.bss` is the third assembly section — unchanged.
- `.align` — for arrays of size at least sixteen bytes, now `MAX(16, var->align)`.
- The `mov $0, %rax` for variadic FP-count is still pessimistic for non-variadic calls — errata candidate, unchanged.
- The `fp_offset = fp * 8 + 48` non-conforming stride — errata candidate, *not* closed by Chapter 18 despite forecast.
- `long double` is `double` — errata candidate, unchanged.
- The default-argument-promotion gap for chars and shorts — errata candidate, unchanged.
- Float literals are inlined as integer-immediate-bit-cast — unchanged.
- The cast/compound-literal disambiguator — unchanged.
- The cast table is 10×10 — unchanged. (Struct-by-value didn't add cast cells; struct casts are skipped at parse time.)
- Driver brittleness in `find_libpath`/`find_gcc_libpath` — unchanged.
- The link command's hardcoded distro list — unchanged.
- `Node->funcname` is gone (Chapter 16 §16.2) — still gone.
- `call *%rax` is uniform across all calls — through Chapter 18, the new `mov %rax, %r10; call *%r10` shape is uniform across all calls (the indirect-call shape changed — not from `*%rax` to `*%r10`, but the variadic-FP-count register dance forced moving the callee out of `%rax` first).
- The `StringArray` type — unchanged. `-D` and `-U` apply at parse time directly rather than queueing strings.
- `atexit(cleanup)` — unchanged.
- The `run_subprocess` helper — unchanged.
- Errata candidates added in Chapter 17 — `#error` doesn't print message text, `L''` ≡ `''`, `__va_arg_mem` divides by zero (closed by `b6d3cd0`), `opt_S | opt_E` typo, default include paths Linux/glibc-specific. One closed; four remaining.
- `self.py` is gone (Chapter 17 §17.6) — still gone.
- Stage-2 build is end-to-end chibicc as of Chapter 17 §17.6 — still is.
- Chibicc compiles itself — still does, plus now compiles itself with `-Wall` cleanly.

Forward references for Chapter 19 (Unicode and designated initializers, commits 221–244):

- The next chapter's largest single arc is full Unicode support — UTF-8 source files, UTF-16 and UTF-32 character literals, UTF-8 string literals, the wide-character conversion that this chapter punted at `L''` ≡ `''`. Multibyte sequences are decoded with the corresponding library calls; the encodings are interconverted at compile time.
- Designated initializers — `struct foo x = {.a = 1, .b = 2};` and array forms like `int x[10] = {[3] = 1, [7] = 2};` — land across several commits. Chapter 12's `Initializer` tree gains the `.member` and `[index]` parsing, with a recovery rule for sparse initializers and a precedence rule for nested designators.
- A few smaller features in the same arc: `__DATE__` and `__TIME__` (the two predefined macros chibicc deferred from Chapter 17), `__COUNTER__`, `_Generic`, `_Static_assert`, `\u`/`\U` escape sequences in string literals, and the `?:` ternary's GCC extension where the middle operand is omitted.
- The `<stdarg.h>`, `<stdbool.h>`, `<stddef.h>`, `<stdalign.h>`, `<stdnoreturn.h>`, and `<float.h>` headers are unchanged in Chapter 19.

Twenty-three commits. The chapter brings chibicc from "compiles itself but with quiet ABI gaps" to "compiles itself, conforms to most of the SysV AMD64 calling convention, supports bitfields, and recognizes the GCC flag vocabulary real build systems use." The next chapter takes on Unicode and designated initializers — the last two large topics on the chibicc roadmap before Chapter 20's series of small completions.
