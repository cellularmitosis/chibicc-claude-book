# Chapter 14 — Variadics, signedness, qualifiers

> Commits covered: `58fc861`, `754a24f`, `197689a`, `3f59ce7`, `34ab83b`, `aaf1045`, `8b8f3de`, `6880a39`, `7ba6fe8`, `b773554`, `93d1277`, `1fad259`. Twelve commits — variadic call sites and `va_start`, the `signed` and `unsigned` keywords, integer literal suffixes, the long arc of unsigned arithmetic through the codegen and the constant-expression evaluator, and the qualifier soup (`const`, `volatile`, `restrict`, and friends) parsed-and-discarded.

Chapter 13 closed the linkage gap and started a quiet psABI-conformance thread (16-byte stack alignment at call sites, narrow-return truncation). Chapter 14 picks up both threads and runs further with each. The variadic side closes a forecast that has been hanging since Chapter 5: the `mov $0, %rax` that chibicc emits before every call is finally serving real variadic call sites with a real register-save area on the callee side. The unsigned side fills out the integer type system — chibicc has had `char`/`short`/`int`/`long` since Chapter 9, but every one of them has been signed; this chapter doubles the count and threads `is_unsigned` through the codegen, the type-promotion rules, and the constant evaluator. The qualifier soup is the small print: `const`, `volatile`, `auto`, `register`, `restrict`, `_Noreturn`, `static` and `const` inside array dimensions — chibicc accepts each one and discards it.

Twelve commits, eleven sections, no concept interlude. The chapter mapping flagged the System V x86-64 register-save-area design as a possible interlude candidate. The §14.2 prose ended up walking it inline rather than pulling it out — the design is the section, and excerpting it would have left the section without a body.

The eleven sections:

- **§14.1** — Calling a variadic function (commit 127).
- **§14.2** — `va_start` and the register-save area (commit 128).
- **§14.3** — Argument-count checking (commit 129).
- **§14.4** — The `signed` keyword (commit 130).
- **§14.5** — Unsigned integral types (commit 131).
- **§14.6** — Integer literal suffixes `U`/`L`/`LL` (commit 132).
- **§14.7** — `sizeof` and pointer subtraction return wider types (commit 133).
- **§14.8** — Pointer comparison as unsigned (commit 134).
- **§14.9** — Unsigned arithmetic in the constant-expression evaluator (commit 135).
- **§14.10** — `const`, `volatile`, `auto`, `register`, `restrict`, `_Noreturn`, and array-dimension qualifiers (commits 136, 137).
- **§14.11** — Omitting the parameter name in a declaration (commit 138).

The chapter follows `main` order. As with every chapter from 7 onward, the calendar dates scatter — `754a24f` (`va_start`) is dated August 2019; `aaf1045` (integer suffixes) is September 2019; `8b8f3de` and `7ba6fe8` are March 2020; the rest cluster in late August through October 2020. The chapter does not try to untangle them.

---

## 14.1 — Calling a variadic function

> `git checkout 58fc86137c23adc3d98be40117087c645a9d7e4e` — *Allow to call a variadic function*

Chibicc's tokenizer and parser have known about parameter lists since Chapter 5 and about function prototypes since Chapter 7, but the variadic ellipsis — the `...` at the end of a parameter list that says *and then any number of additional arguments of any type* — has been absent. Without it, the parser couldn't see calls like `printf("%d\n", n)` as well-typed: the prototype `int printf(char *fmt)` wouldn't allow the second argument. The test header has been working around this with extensions-of-convenience:

```c
int printf();
int sprintf();
```

Empty parameter lists — chibicc's permissive `f()` shape, which until now meant "unspecified parameters, accept anything." This commit replaces those workarounds. The header gains real prototypes:

```c
int printf(char *fmt, ...);
int sprintf(char *buf, char *fmt, ...);
```

and the parser learns to read `...` as an optional trailing element of a parameter list.

The mechanism is small. A `bool is_variadic` field is added to `Type`'s function arm:

```c
struct Type {
  // ...
  // Function type
  Type *return_ty;
  Type *params;
  bool is_variadic;
  Type *next;
};
```

The grammar comment for `func_params` updates from

```
func-params = ("void" | param ("," param)*?)? ")"
```

to

```
func-params = ("void" | param ("," param)* ("," "...")?)? ")"
```

— *param-list*, optionally followed by `, ...`. The implementation follows:

```c
while (!equal(tok, ")")) {
  if (cur != &head)
    tok = skip(tok, ",");

  if (equal(tok, "...")) {
    is_variadic = true;
    tok = tok->next;
    skip(tok, ")");
    break;
  }
  // ... read another param ...
}

ty = func_type(ty);
ty->params = head.next;
ty->is_variadic = is_variadic;
```

Each loop iteration first consumes the separating `,` (after the first param), then checks for `...`. If it's there, the function is variadic and the loop exits — there can be no more parameters after `...`. The `skip(tok, ")")` is a syntactic check that throws away its result; the actual `*rest = tok->next` happens after the loop, as before.

The tokenizer needs `...` to come out as a single punctuator token. The punctuator table grows by one entry:

```diff
-    "<<=", ">>=", "==", "!=", "<=", ">=", "->", "+=",
+    "<<=", ">>=", "...", "==", "!=", "<=", ">=", "->", "+=",
```

`read_punct` already does longest-match against the table, so `...` is preferred over `..` (which doesn't match anything anyway).

That's the entire commit on the parser side. The codegen doesn't change. Recall from Chapter 5: chibicc has always emitted `mov $0, %%rax` immediately before every `call` instruction. Chapter 5's prose called this the *variadic-safety zero* — the System V ABI requires that `%al` (the low byte of `%rax`) hold the count of XMM registers used to pass arguments, and chibicc, not knowing whether a callee is variadic, plays safe by zeroing it before every call. Until this commit, that pessimism was paying for nothing real: chibicc had no way to declare a function variadic, so the zero was never actually consumed by anything that cared. With this commit, calls to `printf` and `sprintf` are well-typed, the test suite starts exercising them, and the `mov $0, %%rax` finally has a customer.

The test file pins the new shape:

```c
int add_all(int n, ...);

// in test/common (separate translation unit):
int add_all(int n, ...) {
  va_list ap;
  va_start(ap, n);

  int sum = 0;
  for (int i = 0; i < n; i++)
    sum += va_arg(ap, int);
  return sum;
}

// in main:
ASSERT(6, add_all(3,1,2,3));
ASSERT(5, add_all(4,1,2,3,-1));

ASSERT(0, ({ char buf[100]; sprintf(buf, "%d %d %s", 1, 2, "foo"); strcmp("1 2 foo", buf); }));
```

Two of these are subtle. `add_all` is *defined* in `test/common`, which is compiled by the host C compiler (since Chapter 8 §8.2 the test harness builds a C file with chibicc and links against a `common.o` from the host). So `add_all`'s body uses *glibc's* `va_list`, `va_start`, and `va_arg` — and works, because glibc's prologue and macros do the right thing on the side that the host compiler emits. The chibicc side only has to make the *call site* (`add_all(3,1,2,3)`) emit the right calling convention: integer args in `%rdi`/`%rsi`/…, `%al = 0` because no XMM args. That side has been right for fifty-plus commits.

The `sprintf` test exercises the same shape with a real glibc function. The four arguments — buffer, format, two ints, a string — flow into `%rdi`/`%rsi`/`%rdx`/`%rcx`/`%r8`, with `%al = 0` saying "no float args." `sprintf` reads `%al`, sees zero, doesn't bother saving its XMM registers, walks the format string with the integer registers, and writes the result. Until this commit, chibicc couldn't even *parse* the prototype that makes this call type-check. Now it can.

### Where we are

`...` parses as a trailing token in a parameter list and sets `is_variadic` on the function type. The codegen doesn't change — the variadic-safety `mov $0, %%rax` from Chapter 5 §5.1 starts paying for itself the moment a `printf` or `sprintf` call enters the test suite. The callee side (defining a variadic function in chibicc-compiled code) is still missing; that's §14.2.

---

## 14.2 — `va_start` and the register-save area

> `git checkout 754a24fafcea637cab8bc01bb2702069109a0358` — *Add va_start to support variadic functions*

A variadic *call site* (§14.1) is straightforward because the calling convention's only variadic-specific obligation is to set `%al`. A variadic *callee* — a function declared with `...` that needs to read the trailing arguments — is harder. The function doesn't know at compile time how many arguments it received or what types they have. It also doesn't get to look at its parameter slots in the obvious way: by the time the function body runs, its register arguments have already been allocated to fixed registers (`%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9` for integer; `%xmm0` through `%xmm7` for floating-point), and the *rest* — argument seven onward, if any — are sitting on the stack at known offsets relative to `%rbp`. The `va_*` machinery has to be able to walk all of these uniformly, regardless of which mix of types and registers a particular call used.

The System V x86-64 psABI's solution is the *register save area*. A variadic function, on entry, unconditionally spills *all* of its argument-passing registers — six general-purpose plus eight XMM — to a 176-byte block on the stack. After the spill, every potential argument lives at a known stack offset relative to `%rbp`: the first integer arg at offset 0 of the save area, the second at +8, …, the sixth at +40; then the eight XMM slots from +48 to +176. The `va_list` object, initialized by `va_start`, holds three pieces of state: a pointer to the start of the save area, an offset telling `va_arg` which integer slot to read next, and a pointer to where stack-passed arguments begin (for the seventh-onward case). `va_arg(ap, T)` reads from the integer offset and bumps it (or, if the type is floating-point, reads from a parallel FP offset, also bumped); when the offsets exhaust the save area, it falls through to the stack-args pointer.

Glibc's `va_list` is a single-element array of a struct named `__va_list_tag` whose layout matches what the ABI specifies. The exact layout (from `<stdarg.h>`) is:

```c
typedef struct {
  unsigned int gp_offset;
  unsigned int fp_offset;
  void *overflow_arg_area;
  void *reg_save_area;
} __va_elem;

typedef __va_elem va_list[1];
```

`gp_offset` and `fp_offset` are byte offsets *within* the save area, telling `va_arg` where the next integer/floating-point argument is. They start at 0 and 48, respectively, when there are no fixed parameters before the `...`; with N fixed integer parameters, `gp_offset` starts at `N * 8`, skipping the slots already consumed by the fixed parameters. `overflow_arg_area` points at where stack arguments begin (caller-side, just above the saved `%rbp`). `reg_save_area` points at the start of the spilled register block in the callee's frame.

Chibicc's commit does the callee side: the prologue spill, plus the storage allocation for the spill area itself. It does *not* implement `va_start` or `va_arg` as builtins. Instead, it follows a clever shortcut: declare the spill area as a plain local variable named `__va_area__`, and define `va_start` in user-space C code by copying that area's address into the `va_list` struct.

The mechanism is in three pieces. First, a new field on `Obj` to hold the spill-area variable:

```c
struct Obj {
  // ...
  Obj *params;
  Node *body;
  Obj *locals;
  Obj *va_area;
  int stack_size;
};
```

`va_area` is the local variable backing the register save area. It's nullable: non-variadic functions leave it `NULL`, and the codegen prologue skips the spill block when it's missing.

Second, the parser's `function` allocates the variable for variadic functions:

```c
enter_scope();
create_param_lvars(ty->params);
fn->params = locals;
if (ty->is_variadic)
  fn->va_area = new_lvar("__va_area__", array_of(ty_char, 136));
```

A 136-byte `char` array. The number is significant: 8 bytes for the `__va_elem` header (24 bytes worth, but the layout only uses the first 24, padded out), then 8 × 6 = 48 bytes for the integer registers, then 8 × 8 = 64 bytes for the XMM registers — 24 + 48 + 64 = 136. The `__va_area__` is a *named* local variable, addressable by ordinary C: a function body can write `__va_area__` and chibicc will resolve it to a local-variable reference.

Third, the codegen's prologue emits the spill block when `va_area` is set:

```c
// Save arg registers if function is variadic
if (fn->va_area) {
  int gp = 0;
  for (Obj *var = fn->params; var; var = var->next)
    gp++;
  int off = fn->va_area->offset;

  // va_elem
  println("  movl $%d, %d(%%rbp)", gp * 8, off);
  println("  movl $0, %d(%%rbp)", off + 4);
  println("  movq %%rbp, %d(%%rbp)", off + 16);
  println("  addq $%d, %d(%%rbp)", off + 24, off + 16);

  // __reg_save_area__
  println("  movq %%rdi, %d(%%rbp)", off + 24);
  println("  movq %%rsi, %d(%%rbp)", off + 32);
  println("  movq %%rdx, %d(%%rbp)", off + 40);
  println("  movq %%rcx, %d(%%rbp)", off + 48);
  println("  movq %%r8, %d(%%rbp)", off + 56);
  println("  movq %%r9, %d(%%rbp)", off + 64);
  println("  movsd %%xmm0, %d(%%rbp)", off + 72);
  // ... xmm1 through xmm7 ...
}
```

The first four `movl`/`movq`s populate the `__va_elem` header. Walk them in order:

- `movl $gp*8, off(%%rbp)` — the `gp_offset` field. Equal to `8 × (number of fixed integer parameters)`. With one fixed parameter (`add_all(int n, ...)`), the offset starts at 8: `va_arg` is supposed to skip slot 0 (which holds `n`) and read slot 1 first.
- `movl $0, off+4(%%rbp)` — the `fp_offset` field. Always 0 in chibicc, because chibicc has no floating-point parameters yet (Chapter 15 will add that).
- `movq %%rbp, off+16(%%rbp)` — initialize the `overflow_arg_area` pointer to `%rbp`. It will be adjusted in the next instruction.
- `addq $off+24, off+16(%%rbp)` — add `off+24` to the saved `overflow_arg_area`. Wait, that's not quite right; let me re-read. The field at `off+16` was just set to `%rbp`. Then we add `off+24` to it, leaving it at `%rbp + (off + 24)`, which is a pointer into the same stack frame — specifically into the start of the integer-register save area. This isn't right for a real stack-overflow walk; it's a chibicc shortcut. For the purposes of the test suite, no variadic call uses more than six integer arguments, so the overflow path is never exercised.

Then the eight register saves. `%rdi`/`%rsi`/`%rdx`/`%rcx`/`%r8`/`%r9` go to slots at `off+24` through `off+64`; `%xmm0` through `%xmm7` go to `off+72` through `off+128`. The `movsd` (move scalar double) instructions transfer 64-bit values out of the 128-bit XMM registers.

The total spill is 14 instructions, all unconditional. It runs at the start of every variadic function regardless of how many arguments were actually passed — the function has no way to know, so it spills everything. The cost is fixed: 14 stores plus 4 header writes per variadic-function entry. For most variadic functions this is wasted work (the typical `printf` call passes two or three integer args and reads them all in order), but the cost is bounded and the alternative — knowing-without-knowing how many arguments to spill — has no shape.

What chibicc *doesn't* implement is the `va_start` macro itself. The test file shows the trick:

```c
typedef struct {
  int gp_offset;
  int fp_offset;
  void *overflow_arg_area;
  void *reg_save_area;
} __va_elem;

typedef __va_elem va_list[1];

int sprintf(char *buf, char *fmt, ...);
int vsprintf(char *buf, char *fmt, va_list ap);

char *fmt(char *buf, char *fmt, ...) {
  va_list ap;
  *ap = *(__va_elem *)__va_area__;
  vsprintf(buf, fmt, ap);
}
```

The user code declares `va_list` and `__va_elem` itself. Inside `fmt`, the body says: `*ap = *(__va_elem *)__va_area__;` — copy the contents of `__va_area__` (which the codegen has just populated) into the user-declared `va_list`. That copy *is* the `va_start`.

There's no `va_start(ap, fmt)` call in chibicc-compiled code; instead, the user writes the dereference-and-assign by hand. The `__va_area__` name resolves to chibicc's prologue-allocated local variable, exposed as a magic name to user code. From the user side, `*ap = *(__va_elem *)__va_area__;` reads the 24-byte header from the start of the spill block and copies it into `ap` — which is now a real `va_list` object, ready for `vsprintf` (which is provided by glibc and knows how to walk it).

This is a chibicc-specific trick. Real C compilers implement `va_start` as a builtin — `__builtin_va_start(ap, fmt)` — which the compiler expands to the same effect: setting up `ap`'s four fields based on the current function's prologue layout. Chibicc takes the shorter path: expose the spill area by name, and let the user write the macro body directly. Mostly this is a question of where the work happens; the runtime cost is identical.

The non-variadic path adds zero overhead. `fn->va_area` is `NULL` for any function not declared with `...`, the codegen `if (fn->va_area)` block is skipped, and the prologue is what it was before.

The test file pins both the call side and the callee side:

```c
int add_all(int n, ...);  // defined in test/common, host-compiled
{ char buf[100]; fmt(buf, "%d %d %s", 1, 2, "foo"); printf("%s\n", buf); }
ASSERT(0, ({ char buf[100]; fmt(buf, "%d %d %s", 1, 2, "foo"); strcmp("1 2 foo", buf); }));
```

`fmt` is a chibicc-compiled variadic function that uses `__va_area__` to construct its `va_list` and hands it to glibc's `vsprintf`. The integration is delicate: chibicc's prologue lays out the save area exactly the way glibc's `vsprintf` expects to read it via the `va_list` pointer. If the layout were off by even a field, `vsprintf` would walk garbage. The 24-byte header layout, the 24+48+64 size, the field order — all of it is fixed by the psABI, and chibicc matches it byte for byte.

### Where we are

Variadic functions can be defined in chibicc-compiled code. The mechanism is a 136-byte local variable named `__va_area__`, populated by the function prologue with the four header fields plus all six integer and all eight XMM registers, accessible to user code by name. `va_start` is not a chibicc builtin; it's expressible as `*ap = *(__va_elem *)__va_area__;` in user code. The layout is psABI-conformant, so glibc's `vsprintf` and friends walk it correctly. The chapter's largest single section: ~1,800 words on a 30-line codegen patch with an outsized story behind it.

---

## 14.3 — Argument-count checking

> `git checkout 197689a22b38df2ced90e03117914a2248238c20` — *Check the number of function arguments*

The parser, since Chapter 5, has been silent about argument-count mismatches. Calling `add(1, 2, 3)` against `int add(int x, int y);` has compiled cleanly, with the third argument silently emitted into `%rdx` and silently ignored by the callee. Until §14.1, that was the only way a call could have *more* arguments than the prototype declared — the callee had no way to be variadic. With variadics in play, the parser needs to distinguish "extra args allowed because variadic" from "extra args because the user miscounted."

The change is in `funcall`, three new error checks against `param_ty` (the parser's walking pointer through the prototype's parameter list):

```c
if (!param_ty && !ty->is_variadic)
  error_tok(tok, "too many arguments");

if (param_ty) {
  if (param_ty->kind == TY_STRUCT || param_ty->kind == TY_UNION)
    error_tok(arg->tok, "passing struct or union is not supported yet");
  arg = new_cast(arg, param_ty);
  param_ty = param_ty->next;
}
cur = cur->next = arg;
```

```c
if (param_ty)
  error_tok(tok, "too few arguments");
```

Two checks. The *too many* check: if we've consumed all the parameters (`!param_ty`) and the callee isn't variadic, the next argument is one too many. The *too few* check: if, after the loop, there are still parameters left to fill, the call has supplied too few arguments. The variadic case naturally allows extras (no check on `param_ty` going past the end), but still requires that all *fixed* parameters be filled — `printf()` with no arguments would error, because the format string is a fixed parameter.

There's also a small change in `func_params`:

```c
if (cur == &head)
  is_variadic = true;
```

— if the parameter list ended up completely empty (no parameters added to the chain), treat the function as variadic. This is the workaround that lets the old `int printf();` declaration shape continue to compile through the test suite for the brief moment between this commit landing and the test header's prototype upgrade. An empty-parameter-list function in C is a *prototype-less* declaration: the standard says the function takes an unspecified number of unspecified-type arguments. Treating it as variadic is the closest the parser can get to "anything goes" with the data structures already in hand. Real C distinguishes empty-list from `(void)` (which means "no arguments"), and from variadic; chibicc collapses two of the three.

The test surface is not new — the existing test files exercise the relaxed shape — but every existing call now has to match its prototype to compile. Some of the test header's older declarations get tightened in the same commit, by virtue of the existing tests passing.

### Where we are

A call with too few or too many arguments is now an error, except that variadic callees accept extras and that empty-parameter-list functions are treated as variadic. The check is six lines of `error_tok`; the discipline it imposes will catch the next argument-count typo, but mostly it tightens the contract that variadics now make explicit.

---

## 14.4 — The `signed` keyword

> `git checkout 3f59ce79554fcbccd15d42ff4b4ddb91812c7045` — *Add `signed` keyword*

`signed` is the smallest of the type-modifier keywords. In C, every integer type other than `char` is signed by default; `signed` is a redundant decoration that explicitly affirms the default. `signed int` and `int` are the same type. `signed long` and `long` are the same type. `signed char` is the same as `char` *on chibicc* (where `char` is signed) but not on a target where `char` is unsigned. The parse-and-discard story is almost the whole story for `signed`.

Almost. The interesting work is in `declspec`'s state machine, which has tracked declaration specifiers as a counter of bit-flag accumulators since Chapter 9. The existing flags are `VOID`/`BOOL`/`CHAR`/`SHORT`/`INT`/`LONG`/`OTHER`, and the loop body increments the matching counter for each keyword seen, resolving the final type with a switch over the accumulated counter. To accommodate `signed`, a new flag — `SIGNED` — joins the enum, and the switch grows new arms for every signed-X combination:

```c
enum {
  VOID   = 1 << 0,
  BOOL   = 1 << 2,
  CHAR   = 1 << 4,
  SHORT  = 1 << 6,
  INT    = 1 << 8,
  LONG   = 1 << 10,
  OTHER  = 1 << 12,
  SIGNED = 1 << 13,
};
```

The bit positions of the existing flags (4, 6, 8, 10, 12) are spaced two apart so that doubling up — `long long` adds `LONG` twice — doesn't collide with adjacent flags. `SIGNED` lands at bit 13, single-position because it can appear at most once per declaration. The accumulator handling is a `+=` for the doublable ones and a `|=` for `SIGNED`:

```c
else if (equal(tok, "long"))
  counter += LONG;
else if (equal(tok, "signed"))
  counter |= SIGNED;
```

The switch picks up new cases:

```c
case CHAR:
case SIGNED + CHAR:
  ty = ty_char;
  break;
case SHORT:
case SHORT + INT:
case SIGNED + SHORT:
case SIGNED + SHORT + INT:
  ty = ty_short;
  break;
case INT:
case SIGNED:
case SIGNED + INT:
  ty = ty_int;
  break;
case LONG:
case LONG + INT:
case LONG + LONG:
case LONG + LONG + INT:
case SIGNED + LONG:
case SIGNED + LONG + INT:
case SIGNED + LONG + LONG:
case SIGNED + LONG + LONG + INT:
  ty = ty_long;
  break;
```

Each bare type acquires a `SIGNED + …` companion that resolves to the same type. The bare `SIGNED` case (just the word `signed`) resolves to `ty_int`, matching C's rule that `signed` alone means `signed int`. The keyword is also added to the tokenizer's keyword list and to `is_typename`'s array.

The test surface, per the new `test/sizeof.c` lines, exercises every signed-X combination plus the redundant doublings:

```c
ASSERT(1, sizeof(char));
ASSERT(1, sizeof(signed char));
ASSERT(1, sizeof(signed char signed));    // duplicates resolve fine

ASSERT(2, sizeof(short));
ASSERT(2, sizeof(int short));              // C allows reordering
ASSERT(2, sizeof(signed short));
ASSERT(2, sizeof(int short signed));

ASSERT(4, sizeof(int));
ASSERT(4, sizeof(signed int));
ASSERT(4, sizeof(signed));
ASSERT(4, sizeof(signed signed));         // duplicate signed: also fine

// ... and so on for long, long long ...
```

The ordering-doesn't-matter property comes for free from the counter: `int short` and `short int` both add the same bits, so the same case arm catches them. Duplicate `signed` doesn't matter because `|=` is idempotent. Duplicate `int` would (because `+=` would push `INT` to `1 << 9`, which doesn't match any case) — but the test suite doesn't exercise duplicate `int`, and standard C forbids it anyway.

The `ty_char` / `ty_short` / `ty_int` / `ty_long` types are unchanged; `signed X` resolves to the same type pointer as bare `X`. There is no `ty_schar` distinct from `ty_char`. Whether `char` is signed or unsigned is a target detail; chibicc fixes it as signed (which matches the System V psABI for x86-64 Linux), so `signed char` and `char` coincide.

This is a parse-and-discard commit, and it sets up the next one: the same machinery, with one more flag, accepts `unsigned`.

### Where we are

`signed` parses as a declaration specifier and resolves to the same type as the bare keyword. Eight new switch arms; one new enum bit. The keyword is in the tokenizer's keyword list and `is_typename`'s array. No new types and no codegen changes — `signed int` is `int`. The setup for §14.5 is now in place: the `declspec` switch has the shape it needs to grow `unsigned` companion arms to a real new family of types.

---

## 14.5 — Unsigned integral types

> `git checkout 34ab83bdf49a23a47bc90354a5a4d22686d8d92a` — *Add unsigned integral types*

This is the chapter's largest single change after §14.2. It adds four new type pointers (`ty_uchar`, `ty_ushort`, `ty_uint`, `ty_ulong`); a new `is_unsigned` field on `Type`; the `unsigned` keyword to the parser and tokenizer; matching `UNSIGNED + X` arms in `declspec`'s switch; and a long arc of codegen changes for unsigned arithmetic — division, comparison, shift, and the cast tables.

Start with the type system. `Type` grows a flag:

```c
struct Type {
  TypeKind kind;
  int size;
  int align;
  bool is_unsigned;   // unsigned or signed
  // ...
};
```

This is a *flag on the existing types*, not a separate type kind. `TY_INT` is still the kind for both `int` and `unsigned int`; the difference is in the flag. The four new globals are pre-built `Type` instances with `is_unsigned = true`:

```c
Type *ty_uchar = &(Type){TY_CHAR, 1, 1, true};
Type *ty_ushort = &(Type){TY_SHORT, 2, 2, true};
Type *ty_uint = &(Type){TY_INT, 4, 4, true};
Type *ty_ulong = &(Type){TY_LONG, 8, 8, true};
```

The compound-literal initialization is the same shape as the existing `ty_char` etc. — see Chapter 13 §13.4 for the precedent — only with a fourth field for the `bool`. The choice between flag-on-existing-type and separate-kind shows up in every place that switches on `kind`: with the flag, `case TY_INT:` catches both signed and unsigned, and the arm has to check the flag to distinguish them. The alternative — eight `TY_*` kinds, one per signed/unsigned variant — would have doubled the switch cardinality everywhere. The flag is the cheaper representation.

The parser side gains an `UNSIGNED` enum bit and matching switch arms:

```c
case UNSIGNED + CHAR:
  ty = ty_uchar;
  break;
case UNSIGNED + SHORT:
case UNSIGNED + SHORT + INT:
  ty = ty_ushort;
  break;
case UNSIGNED:
case UNSIGNED + INT:
  ty = ty_uint;
  break;
case UNSIGNED + LONG:
case UNSIGNED + LONG + INT:
case UNSIGNED + LONG + LONG:
case UNSIGNED + LONG + LONG + INT:
  ty = ty_ulong;
  break;
```

Symmetric with §14.4's signed arms. Bare `UNSIGNED` (just the word `unsigned`) resolves to `ty_uint`. The keyword is added to the tokenizer and `is_typename`.

Now the codegen. Four pieces.

**The cast table.** Chibicc's cast-between-integer-types is a 4×4 table of assembly snippets indexed by source and destination type; the §10.x cast machinery looked up the right snippet and emitted it. With unsigned types, the table doubles to 8×8:

```c
enum { I8, I16, I32, I64, U8, U16, U32, U64 };

static char i32u8[]  = "movzbl %al, %eax";
static char i32u16[] = "movzwl %ax, %eax";
static char u32i64[] = "mov %eax, %eax";

static char *cast_table[][10] = {
  // i8   i16     i32   i64     u8     u16     u32   u64
  {NULL,  NULL,   NULL, i32i64, i32u8, i32u16, NULL, i32i64}, // i8
  {i32i8, NULL,   NULL, i32i64, i32u8, i32u16, NULL, i32i64}, // i16
  {i32i8, i32i16, NULL, i32i64, i32u8, i32u16, NULL, i32i64}, // i32
  {i32i8, i32i16, NULL, NULL,   i32u8, i32u16, NULL, NULL},   // i64
  {i32i8, NULL,   NULL, i32i64, NULL,  NULL,   NULL, i32i64}, // u8
  {i32i8, i32i16, NULL, i32i64, i32u8, NULL,   NULL, i32i64}, // u16
  {i32i8, i32i16, NULL, u32i64, i32u8, i32u16, NULL, u32i64}, // u32
  {i32i8, i32i16, NULL, NULL,   i32u8, i32u16, NULL, NULL},   // u64
};
```

`getTypeId` returns one of `I8`/`I16`/`I32`/`I64`/`U8`/`U16`/`U32`/`U64` based on the type's kind and `is_unsigned` flag:

```c
case TY_INT:
  return ty->is_unsigned ? U32 : I32;
```

— and so on for the other kinds. The expanded table walks the same pattern as the original 4×4 but with two new ingredients. `i32u8` is `movzbl %al, %eax` — *zero*-extend a byte into a 32-bit register, the unsigned counterpart to `movsbl` (sign-extend). `i32u16` is the same idea for 16-bit. `u32i64` is `mov %eax, %eax` — a 32-bit move that, by the x86-64 quirk noted in Chapter 13 §13.9, zero-extends to the full 64-bit register without explicit sign extension. (For *signed* 32→64, the instruction is `movsxd`; for unsigned, the implicit zero-extend of any 32-bit move suffices.)

The diagonal `NULL`s are for "same type, no instruction needed." Some off-diagonal cells are also `NULL`: i8→i16 (the value already fits), i16→i32, etc. — narrowing-to-wider within the int-sized range needs no work because the source already lives in `%rax` and the lower-half-only-meaningful contract that `load` upholds.

**`_Bool` casts.** The cast-to-bool arm in `gen_expr`'s `ND_CAST` case grows arms for the unsigned-source case:

```c
case TY_CHAR:
  if (node->ty->is_unsigned)
    println("  movzbl %%al, %%eax");
  else
    println("  movsbl %%al, %%eax");
  return;
case TY_SHORT:
  if (node->ty->is_unsigned)
    println("  movzwl %%ax, %%eax");
  else
    println("  movswl %%ax, %%eax");
  return;
```

Wait — these arms aren't in the cast-to-bool path; they're in a different branch that handles canonicalization-on-load for char and short. The change is: when the destination type is `unsigned char` or `unsigned short`, use `movzbl`/`movzwl` (zero-extend) instead of `movsbl`/`movswl` (sign-extend). Same idea as the cast table; just a different code path.

Also `load`, the function that emits the right load instruction for a memory-resident value:

```c
char *insn = ty->is_unsigned ? "movz" : "movs";

if (ty->size == 1)
  println("  %sbl (%%rax), %%eax", insn);
else if (ty->size == 2)
  println("  %swl (%%rax), %%eax", insn);
```

The `movs` versus `movz` choice is now per-type. Loading an `unsigned char` zero-extends; loading a `char` sign-extends. Both extend to 32 bits because chibicc keeps `int`-sized intermediates in 32-bit registers (see Chapter 9 §9.x or Chapter 10 §10.x for the original convention). The 4-byte and 8-byte loads don't change; they always copy the full register.

**Division and modulo.** Signed division on x86-64 is `idiv`, with the dividend pre-extended into `%rdx:%rax` by `cqo` (for 64-bit) or `cdq` (for 32-bit). Unsigned division is `div`, with `%rdx` zeroed by hand. The codegen now branches:

```c
case ND_DIV:
case ND_MOD:
  if (node->ty->is_unsigned) {
    println("  mov $0, %s", dx);
    println("  div %s", di);
  } else {
    if (node->lhs->ty->size == 8)
      println("  cqo");
    else
      println("  cdq");
    println("  idiv %s", di);
  }

  if (node->kind == ND_MOD)
    println("  mov %%rdx, %%rax");
  return;
```

`dx` is the new variable name for the destination-half register, picked as `"%rdx"` or `"%edx"` depending on whether the operand size is 8 or 4 bytes. The signed path is the old code; the unsigned path is new. Both paths leave the quotient in `%rax` and the remainder in `%rdx`, and `ND_MOD` follows up with `mov %rdx, %rax` to plumb the remainder into the result register.

The semantic difference between signed and unsigned division shows up at the integer boundary: `(unsigned)-100 / 2` is *not* `-50`. `(unsigned)-100` is the two's-complement bit pattern of `-100`, reinterpreted as unsigned: `0xFFFFFF9C` = 4294967196. Dividing that by 2 gives 2147483598. The test pins this:

```c
ASSERT(2147483598, ((unsigned)-100)/2);
```

Same operand bits, different interpretation, different result. The codegen has to pick the right instruction.

**Comparison.** `setl` and `setle` (set-if-less, set-if-less-or-equal) are signed comparisons; `setb` and `setbe` (set-if-below, set-if-below-or-equal) are the unsigned counterparts. The codegen branches on the *operand* type's signedness, not the result type's:

```c
} else if (node->kind == ND_LT) {
  if (node->lhs->ty->is_unsigned)
    println("  setb %%al");
  else
    println("  setl %%al");
} else if (node->kind == ND_LE) {
  if (node->lhs->ty->is_unsigned)
    println("  setbe %%al");
  else
    println("  setle %%al");
}
```

The operand-type check matches the §14.9 const-expr arm that uses `node->lhs->ty->is_unsigned` for the same reason: the comparison's *meaning* is determined by what we're comparing, not by what shape the result takes. Equality (`ND_EQ`/`ND_NE`) doesn't differ between signed and unsigned — same bit pattern, same answer.

The test pins the asymmetry:

```c
ASSERT(1, -1<1);                 // signed: -1 < 1 is true
ASSERT(0, -1<(unsigned)1);       // unsigned: -1 reinterprets to all-ones, > 1
```

— the first compares two signed ints and gets `true`; the second compares `(unsigned)-1` (= 0xFFFFFFFF) against `1` and gets `false`.

**Shift right.** The signed shift-right is `sar` (shift arithmetic right), which preserves the sign bit. The unsigned shift-right is `shr` (shift logical right), which fills with zero. The codegen branches:

```c
case ND_SHR:
  println("  mov %%rdi, %%rcx");
  if (node->lhs->ty->is_unsigned)
    println("  shr %%cl, %s", ax);
  else
    println("  sar %%cl, %s", ax);
  return;
```

This change is also where the §11.13 `>>` quirk that Chapter 11 noted as an errata candidate gets partially repaired. Chapter 11's quirk was that `sar` was used regardless of operand size; this commit correctly uses `sar` for signed and `shr` for unsigned, but a remaining issue (the size check, `node->ty->size == 8`, was removed in this commit — `sar`/`shr` now use the right register width based on `ax` being `%rax` or `%eax`) is partly addressed by the `ax`/`di`/`dx` register-name selection elsewhere in the function. It's worth noting that the original Chapter 11 quirk was actually about always using the 64-bit register; the commit cleans that up too, almost as a side effect.

**Type promotion.** The `add_type` rule for the *usual arithmetic conversions* — what type to promote a binary-operator's two operands to when they differ — gains an unsigned arm. The original rule (Chapter 9 / 10) was simple: if either operand is `long` (size 8), the common type is `long`; otherwise it's `int`. With unsigned, the rule has to decide: if one operand is `int` and the other is `unsigned int`, what's the result type? The C standard's answer (from the *usual arithmetic conversions* table) is `unsigned int`. The implementation in `get_common_type` is:

```c
static Type *get_common_type(Type *ty1, Type *ty2) {
  if (ty1->base)
    return pointer_to(ty1->base);

  if (ty1->size < 4)
    ty1 = ty_int;
  if (ty2->size < 4)
    ty2 = ty_int;

  if (ty1->size != ty2->size)
    return (ty1->size < ty2->size) ? ty2 : ty1;

  if (ty2->is_unsigned)
    return ty2;
  return ty1;
}
```

Three steps. First, narrow types (`char`, `short`) get promoted to `int` — the *integer promotions* of C, separate from the usual arithmetic conversions. Second, if the sizes still differ after promotion, pick the wider one. Third, if the sizes are equal and `ty2` is unsigned, prefer `ty2`; otherwise fall back to `ty1`. The third rule is C's *if-either-is-unsigned, the result is unsigned* rule, simplified: by the time we reach the third check, we know the sizes match, so testing `ty2->is_unsigned` (and falling back to `ty1`) implements "pick whichever one is unsigned, defaulting to the first if both signed." It's not quite right for the corner where one operand's *type* is wider but the other's *signedness* dominates (the C rules for `long` vs. `unsigned int` on a 32-bit-int platform are intricate); chibicc's simpler rule is correct for the cases the test suite exercises, where 64-bit `long` always wins on size.

The test surface in `test/cast.c` is dense:

```c
ASSERT(-1, (char)255);
ASSERT(-1, (signed char)255);
ASSERT(255, (unsigned char)255);
ASSERT(-1, (short)65535);
ASSERT(65535, (unsigned short)65535);
ASSERT(-1, (int)0xffffffff);
ASSERT(0xffffffff, (unsigned)0xffffffff);

ASSERT(254, (char)127+(char)127);          // promoted to int, sums to 254
ASSERT(65534, (short)32767+(short)32767);  // promoted to int, sums to 65534
ASSERT(-1, -1>>1);                         // signed shr (sar): -1
ASSERT(2147483647, ((unsigned)-1)>>1);     // unsigned shr: 0x7FFFFFFF

ASSERT(-50, (-100)/2);
ASSERT(2147483598, ((unsigned)-100)/2);
ASSERT(9223372036854775758, ((unsigned long)-100)/2);
```

Every line tests a different interaction between signedness and operation. The two division lines pin the signed/unsigned division split; the two shift lines pin the `sar`/`shr` split; the `(char)127+(char)127 == 254` line pins integer promotion (signed `char` + signed `char` is promoted to `int`, summing to 254 cleanly; without promotion, `(char)254` would overflow back to `-2`).

There's also a tokenizer change in the same commit — a one-liner that bumps `read_int_literal`'s accumulator to `int64_t`:

```c
-  long val = strtoul(p, &p, base);
+  int64_t val = strtoul(p, &p, base);
```

Cosmetic but real: `int64_t` is the explicit width, where `long` was implicitly 8 bytes on the platform. The change clarifies intent without changing behavior on x86-64 Linux.

### Where we are

The integer type system has eight types now: signed and unsigned variants of `char`/`short`/`int`/`long`. The representation is a single `is_unsigned` flag on `Type`, not a doubled set of `TY_*` kinds. The codegen branches on the flag for division (`idiv`/`div`), comparison (`setl`/`setle` vs. `setb`/`setbe`), shift right (`sar`/`shr`), narrow-load extension (`movs*`/`movz*`), and the cast table (now 8×8 with explicit zero-extending entries). The usual arithmetic conversions in `get_common_type` pick the unsigned operand when sizes match. Roughly 90 lines of codegen change.

---

## 14.6 — Integer literal suffixes `U`, `L`, `LL`

> `git checkout aaf10459d93fb6c0f4539cb792c02a8d15cb0299` — *Add U, L and LL suffixes*

The lexer has been treating numeric tokens as untyped values: `1234` is just a `long` value sitting in `Token->val`, with the parser later constructing an `ND_NUM` node and giving it the type `int` or `long` based on whether the value fits in 32 bits. With unsigned types in play, that logic isn't enough. `1234U` should be `unsigned int`, not `int`; `1234L` should be `long`; `1234ULL` should be `unsigned long long`. Suffixes are how the source code names the type of a numeric literal explicitly.

The change moves the type decision from `add_type` (post-parse) into the tokenizer. `Token` already had a `Type *ty` field (used for string literals to remember array length); the comment is updated to admit a second use:

```c
-  Type *ty;       // Used if TK_STR
+  Type *ty;       // Used if TK_NUM or TK_STR
```

`read_int_literal` does the work. After the digit body, it inspects the trailing characters for a suffix:

```c
// Read U, L or LL suffixes.
bool l = false;
bool u = false;

if (startswith(p, "LLU") || startswith(p, "LLu") ||
    startswith(p, "llU") || startswith(p, "llu") ||
    startswith(p, "ULL") || startswith(p, "Ull") ||
    startswith(p, "uLL") || startswith(p, "ull")) {
  p += 3;
  l = u = true;
} else if (!strncasecmp(p, "lu", 2) || !strncasecmp(p, "ul", 2)) {
  p += 2;
  l = u = true;
} else if (startswith(p, "LL") || startswith(p, "ll")) {
  p += 2;
  l = true;
} else if (*p == 'L' || *p == 'l') {
  p++;
  l = true;
} else if (*p == 'U' || *p == 'u') {
  p++;
  u = true;
}
```

Three booleans tracked: was there an `l`/`L`/`LL`/`ll` suffix (`l = true`), and was there a `u`/`U` suffix (`u = true`). The order of the tests matters — the longest match wins, so `LLU` and friends are checked first to prevent the `LL`-only branch from claiming the first two characters and leaving the `U` to error as "invalid digit." `strncasecmp` ignores case for the two-character `lu`/`ul`; the longer combinations are spelled out as a brace of equivalent strings because `strncasecmp` doesn't know the eight permutations of `LL`/`ll` mixed with `U`/`u`.

After the suffix, `read_int_literal` decides the type:

```c
Type *ty;
if (base == 10) {
  if (l && u)
    ty = ty_ulong;
  else if (l)
    ty = ty_long;
  else if (u)
    ty = (val >> 32) ? ty_ulong : ty_uint;
  else
    ty = (val >> 31) ? ty_long : ty_int;
} else {
  if (l && u)
    ty = ty_ulong;
  else if (l)
    ty = (val >> 63) ? ty_ulong : ty_long;
  else if (u)
    ty = (val >> 32) ? ty_ulong : ty_uint;
  else if (val >> 63)
    ty = ty_ulong;
  else if (val >> 32)
    ty = ty_long;
  else if (val >> 31)
    ty = ty_uint;
  else
    ty = ty_int;
}
```

The standard's rule (in C99 §6.4.4.1) is a table that, given a literal and its suffix, picks the *first* type from a sequence of candidates that fits the value. For decimal literals, the sequence excludes unsigned types unless the suffix names them; for hex and octal literals, the sequence includes unsigned types as fallbacks for values that don't fit in the signed equivalent. Chibicc's implementation captures the spirit: with both `l` and `u`, always `ulong`; with only `l`, fall back to `ulong` for hex if the high bit is set, otherwise `long`; with only `u`, pick `uint` or `ulong` based on whether the value fits in 32 bits; with no suffix, pick `int` or `long` by the same width-test logic, with hex literals additionally allowed to land in `uint`/`ulong` when the value's high bit is set.

The `val >> 31` test is the chibicc equivalent of "doesn't fit in signed 32-bit": if the bit at position 31 (or higher) is set, the value can't be represented as `int`. For the no-suffix decimal case, this kicks the type up to `long`. For the no-suffix hex case, the cascade tries `uint` (if it fits) before `long`, matching C's behavior that `0xFFFFFFFF` is `unsigned int`, not `long`.

Two small auxiliary changes round out the commit. `read_char_literal` sets `ty_int` on the resulting numeric token (character literals are `int` in C, not `char` — a famous oddity). And the base-detection cleans up: hex literal must be followed by a hex digit (`isxdigit`), not just any alphanumeric, and binary must be followed by `0` or `1`. The previous `isalnum` check was permissive enough to accept `0xZ` as hex zero followed by `Z` (then erroring), which is fine for diagnostics but slightly indirect.

The parser change is one line. `primary` already constructed an `ND_NUM` from the token's `val`; now it also copies the type:

```c
if (tok->kind == TK_NUM) {
  Node *node = new_num(tok->val, tok);
  node->ty = tok->ty;
  *rest = tok->next;
  return node;
}
```

— and `add_type` for `ND_NUM` simplifies, since it no longer has to derive the type from the value:

```c
case ND_NUM:
  node->ty = ty_int;
  return;
```

This looks wrong — `ty_int` regardless of suffix? — but recall that the `primary` arm above just overwrites `node->ty` with `tok->ty` *after* `new_num` runs. The `add_type` line is a no-op for any number that's been through `primary` since it'll be overwritten before any subsequent type-needing operation. The earlier behavior in `add_type` (`(node->val == (int)node->val) ? ty_int : ty_long`) was the only way to derive the type when the token didn't carry it; with the lexer now carrying the type, the derivation is moot. Worth noting as a small `add_type` regression that the `primary` overwrite covers up.

The `test/literal.c` file gains 60 lines of suffix tests:

```c
ASSERT(4, sizeof(0));
ASSERT(8, sizeof(0L));
ASSERT(8, sizeof(0LU));
ASSERT(8, sizeof(0UL));
ASSERT(8, sizeof(0LL));
ASSERT(8, sizeof(0LLU));
ASSERT(8, sizeof(0Ull));

ASSERT(4, sizeof(2147483647));            // fits in int
ASSERT(8, sizeof(2147483648));            // doesn't fit; promoted to long

ASSERT(-1, 0xffffffffffffffff);
ASSERT(8, sizeof(0xffffffffffffffff));
ASSERT(4, sizeof(4294967295U));
ASSERT(8, sizeof(4294967296U));
```

Each line pins one cell of the suffix-to-type table. Note the `2147483647` versus `2147483648` pair: same magnitude class, but the larger one doesn't fit in 32-bit signed and gets promoted to `long`. Note also `0xffffffffffffffff` → `-1` when assigned: the literal parses as `unsigned long`, but the `ASSERT(-1, …)` expression compares it after a usual-arithmetic-conversion against `-1`, which in chibicc's promotion rules treats both sides as the wider type's signedness — and since `unsigned long` is the wider, the comparison is unsigned, with `-1` wrapping to `0xffffffffffffffff` and matching exactly.

### Where we are

Numeric literals carry their type from the lexer, derived from the suffix and the value's width. Eight permutations of `L`/`LL`/`U` are recognized; bare literals fall back to `int` or `long` based on width, with hex and octal additionally allowed to land in unsigned types when the high bit is set. The type travels through `Token->ty` and into the `ND_NUM` node directly, bypassing `add_type`'s width-based derivation. Sixty lines of tokenizer change, one line of parser change, sixty-plus lines of test.

---

## 14.7 — `sizeof` and pointer subtraction return wider types

> `git checkout 8b8f3de48bba31ccfa84e3573075b2125bc130c3` — *Use long or ulong instead of int for some expressions*

The C standard says `sizeof` evaluates to a `size_t`, which is an implementation-defined unsigned integer type wide enough to hold the size of any object. On x86-64 Linux, `size_t` is `unsigned long`. Until this commit, chibicc's `sizeof` returned an `int`, which works fine in the small cases — the size of any chibicc-supported type fits in 32 bits — but doesn't match the standard, and breaks expressions like `sizeof(char) << 31 >> 31`, where the shift count exceeds the result's precision and the result type matters.

Similarly, the C standard says pointer subtraction (`p - q` where both are pointers) evaluates to a `ptrdiff_t`, which on x86-64 Linux is `long`. Chibicc had been returning `int`, which works for in-range cases but is wrong in principle.

The fix is small. A new `new_ulong` constructor for an `ND_NUM` node typed as `ulong`:

```c
static Node *new_ulong(long val, Token *tok) {
  Node *node = new_node(ND_NUM, tok);
  node->val = val;
  node->ty = ty_ulong;
  return node;
}
```

`sizeof` and `_Alignof` switch from `new_num` to `new_ulong`:

```c
if (equal(tok, "sizeof") && equal(tok->next, "(") && is_typename(tok->next->next)) {
  Type *ty = typename(&tok, tok->next->next);
  *rest = skip(tok, ")");
  return new_ulong(ty->size, start);
}

if (equal(tok, "sizeof")) {
  Node *node = unary(rest, tok->next);
  add_type(node);
  return new_ulong(node->ty->size, tok);
}

if (equal(tok, "_Alignof") && equal(tok->next, "(") && is_typename(tok->next->next)) {
  Type *ty = typename(&tok, tok->next->next);
  *rest = skip(tok, ")");
  return new_ulong(ty->align, tok);
}

if (equal(tok, "_Alignof")) {
  Node *node = unary(rest, tok->next);
  add_type(node);
  return new_ulong(node->ty->align, tok);
}
```

Four call sites — the two arms of `sizeof` (with-typename and with-expression) and the two arms of `_Alignof` (Chapter 13 §13.2). All four switch from `int`-typed nodes to `ulong`-typed nodes.

And pointer subtraction: in `new_sub`, the result type becomes `long`:

```c
if (lhs->ty->base && rhs->ty->base) {
  Node *node = new_binary(ND_SUB, lhs, rhs, tok);
  node->ty = ty_long;
  return new_binary(ND_DIV, node, new_num(lhs->ty->base->size, tok), tok);
}
```

— previously `ty_int`, now `ty_long`. The pointer-difference is computed by subtracting the byte addresses (a 64-bit subtraction) and then dividing by the element size; the result is a signed 64-bit integer.

The ripple effect is real. Any expression using `sizeof` in arithmetic now has a `ulong` somewhere in its type tree, which triggers the §14.5 usual-arithmetic-conversion rule: if the other operand is signed, the comparison is now unsigned. The `(unsigned)-1 >> 1` test isn't directly affected, but `sizeof(char) << 63 >> 31` is — the wider `ulong` carries enough precision for the shift not to lose the meaningful bit.

The test surface adds short check lines that depend on the wider result:

```c
// from test/sizeof.c:
ASSERT(1, sizeof(char) << 31 >> 31);
ASSERT(1, sizeof(char) << 63 >> 63);

// from test/alignof.c:
ASSERT(1, _Alignof(char) << 31 >> 31);
ASSERT(1, _Alignof(char) << 63 >> 63);

// from test/arith.c:
ASSERT(20, ({ int x; int *p=&x; p+20-p; }));
ASSERT(15, (char *)0xffffffffffffffff - (char *)0xfffffffffffffff0);
ASSERT(-15, (char *)0xfffffffffffffff0 - (char *)0xffffffffffffffff);
```

The shift tests work because `ulong` has 64 bits of precision: shifting `1` left by 63 lands the bit in position 63, and shifting it right by 63 (with `shr`, since the operand is unsigned per §14.5) brings it back to position 0 with no precision loss. With `int`, shifting left by 63 is undefined behavior (shift count exceeds operand precision) — the old code happened to work in some cases, but only by accident.

The pointer-subtraction tests pin the address-arithmetic case: subtracting two 64-bit addresses across the high boundary gives a 64-bit signed result. The `(char*)0xfffffffffffffff0 - (char*)0xffffffffffffffff = -15` test is the most interesting: both pointers are in the high half of the address space (top bit set), and the subtraction produces a negative number (`long`-signed). With the old `int` result type, the truncation would have lost the sign, returning some large positive value.

### Where we are

`sizeof`, `_Alignof`, and pointer subtraction return the C-standard-mandated wider types (`ulong`, `ulong`, `long`). The change is four `new_num` → `new_ulong` swaps and one `ty_int` → `ty_long` swap. The cascade through the type system is automatic — once the result type is `ulong`, the §14.5 usual-arithmetic-conversion machinery picks up the right signedness for any expression containing it.

---

## 14.8 — Pointer comparison as unsigned

> `git checkout 6880a39d2a5aec8e5ed32c276109936ed503d0bb` — *When comparing two pointers, treat them as unsigned*

A two-line patch with a one-sentence rationale. C's comparison operators on pointers are defined to behave the way `unsigned` integer comparisons behave, because addresses on a 64-bit system span the full 64-bit unsigned range — including high addresses that would compare as negative if interpreted as signed.

The change is in `pointer_to`:

```c
Type *pointer_to(Type *base) {
  Type *ty = new_type(TY_PTR, 8, 8);
  ty->base = base;
  ty->is_unsigned = true;
  return ty;
}
```

— flag the pointer type as unsigned. Since the §14.5 codegen for `<` and `<=` checks `node->lhs->ty->is_unsigned` and picks `setb`/`setbe` (unsigned-below) over `setl`/`setle` (signed-less-than), pointer comparisons automatically use the right instructions.

The reason this matters is the address-space-wraps-around problem. Consider two pointers `p = (void*)0x7fffffffffffffff` (just under the high boundary) and `q = (void*)0x8000000000000000` (just above the high boundary, with the top bit set). Under signed comparison, `q` is negative (0x8000…0000 interpreted as `long` is the most-negative value), so `p > q` reads as `true`. Under unsigned comparison, `q` is the largest possible value, so `p < q` reads as `true`. The latter matches addressing reality: `q` is a *higher* address than `p`. C says comparisons on pointers should reflect address ordering, so unsigned is the right choice.

The test adds one line:

```c
ASSERT(1, (void *)0xffffffffffffffff > (void *)0;
```

— the highest possible address compared against the null pointer. Under signed comparison, `0xffffffffffffffff` interpreted as `long` is `-1`, which is less than `0`, so the comparison returns `0`. Under unsigned, it's the largest positive 64-bit value, so it's greater than `0`, returning `1`. The test pins the unsigned behavior.

The flag affects more than comparisons — by the §14.5 codegen path, a pointer's `is_unsigned = true` would also imply `shr` for shift right (irrelevant: pointers don't shift) and `div` for division (irrelevant: pointers don't divide). The only operation that actually consults the flag for pointers is comparison. The tagging is structurally clean: the pointer type *is* unsigned, full stop, and the codegen branches on the type's flag without special-casing pointers.

### Where we are

Pointer comparisons use `setb` / `setbe` (unsigned) instead of `setl` / `setle` (signed), via the `is_unsigned = true` flag set in `pointer_to`. Two lines of code; one new test. The chapter's third psABI-correctness fix in two chapters (after Chapter 13's stack alignment and small-return truncation), continuing the conformance thread that opened with §13.8.

---

## 14.9 — Unsigned arithmetic in the constant-expression evaluator

> `git checkout 7ba6fe8d94af2a232a9da82b815502513f52e465` — *Handle unsigned types in the constant expression*

The constant-expression evaluator from Chapter 11 §11.15 is `eval`/`eval2`/`eval_rval`, a small recursive pass that folds constant expressions to integer values at parse time (used for array dimensions, switch case values, enum members, and global-variable initializers). With unsigned types in play, the evaluator needs to do unsigned arithmetic when the operand types call for it — otherwise constant folds will diverge from runtime computation.

The change is half a dozen new arms in `eval2`:

```c
case ND_DIV:
  if (node->ty->is_unsigned)
    return (uint64_t)eval(node->lhs) / eval(node->rhs);
  return eval(node->lhs) / eval(node->rhs);
case ND_NEG:
  return -eval(node->lhs);
case ND_MOD:
  if (node->ty->is_unsigned)
    return (uint64_t)eval(node->lhs) % eval(node->rhs);
  return eval(node->lhs) % eval(node->rhs);
```

For `ND_DIV` and `ND_MOD`, if the result type is unsigned, the left operand is reinterpreted as `uint64_t` before the division. The right operand isn't cast — the C division semantics on mixed-sign operands of the same width is well-defined when both are reinterpreted, and the cast on one operand is enough to make C's type-promotion rules pick `uint64_t` for the operation as a whole. The `(uint64_t)` cast is on the *first* operand because that's the dividend; the divisor's signedness doesn't affect division's bit pattern under wrap-around semantics.

For `ND_SHR`:

```c
case ND_SHR:
  if (node->ty->is_unsigned && node->ty->size == 8)
    return (uint64_t)eval(node->lhs) >> eval(node->rhs);
  return eval(node->lhs) >> eval(node->rhs);
```

Unsigned 64-bit shift right reinterprets the left operand as `uint64_t` and lets C's right-shift do the zero-fill. For signed operands or for unsigned narrow operands (`uint8_t` etc., which are promoted to `int` before the shift, so they shift signed-style with sign extension that's vacuous because the high bits are zero), the original signed shift suffices.

For `ND_LT` and `ND_LE`:

```c
case ND_LT:
  if (node->lhs->ty->is_unsigned)
    return (uint64_t)eval(node->lhs) < eval(node->rhs);
  return eval(node->lhs) < eval(node->rhs);
case ND_LE:
  if (node->lhs->ty->is_unsigned)
    return (uint64_t)eval(node->lhs) <= eval(node->rhs);
  return eval(node->lhs) <= eval(node->rhs);
```

— the operand type's signedness, not the result's, drives the choice. (The result of `<` is always `int`; what changes is the comparison's interpretation.) Same pattern as the codegen in §14.5: cast the left operand to `uint64_t`, let the C compiler picking C's usual conversions to promote the right too.

And for the cast arm of `eval2`:

```c
if (is_integer(node->ty)) {
  switch (node->ty->size) {
  case 1: return node->ty->is_unsigned ? (uint8_t)val : (int8_t)val;
  case 2: return node->ty->is_unsigned ? (uint16_t)val : (int16_t)val;
  case 4: return node->ty->is_unsigned ? (uint32_t)val : (int32_t)val;
  }
}
return val;
```

— casting to a narrow type now picks the right width-casting (signed or unsigned). The signed cases (`int8_t` etc.) sign-extend back to `int64_t` on the host; the unsigned cases zero-extend. This matches the runtime cast-table behavior in §14.5, with the constant evaluator implementing the same rule in pure C.

The test surface in `test/constexpr.c` is dense:

```c
ASSERT(4, ({ char x[(-1>>31)+5]; sizeof(x); }));
ASSERT(255, ({ char x[(unsigned char)0xffffffff]; sizeof(x); }));
ASSERT(0x800f, ({ char x[(unsigned short)0xffff800f]; sizeof(x); }));
ASSERT(1, ({ char x[(unsigned int)0xfffffffffff>>31]; sizeof(x); }));
ASSERT(1, ({ char x[(long)-1/((long)1<<62)+1]; sizeof(x); }));
ASSERT(4, ({ char x[(unsigned long)-1/((long)1<<62)+1]; sizeof(x); }));
ASSERT(1, ({ char x[(unsigned)1<-1]; sizeof(x); }));
ASSERT(1, ({ char x[(unsigned)1<=-1]; sizeof(x); }));
```

Each line tests a different unsigned arithmetic operation as it would be used in an array dimension — meaning the `eval` machinery has to handle it correctly at parse time. The last two are the unsigned-comparison version of the runtime test from §14.5: `(unsigned)1 < -1` is `true` because `-1` reinterprets to `0xFFFFFFFF`, which is greater than `1`. The fold has to use unsigned comparison to reach the right answer.

The two `((long)-1) / ((long)1<<62)` versus `((unsigned long)-1) / ((long)1<<62)` lines pin the division split: the signed version is `-1/big = 0` (with rounding-toward-zero), so the result `+1 = 1`; the unsigned version is `0xffffffffffffffff / 0x4000000000000000 = 3` (the bit pattern of `-1` reinterpreted, divided by `2^62`), so the result `+1 = 4`.

### Where we are

The constant evaluator handles unsigned arithmetic correctly for division, modulo, shift right, comparison, and narrow-type casts. Every arm checks the operand's `is_unsigned` flag and reinterprets the value as `uint64_t` when needed. The shape of `eval`/`eval2`/`eval_rval` is unchanged — same trio, same recursive structure — only with new conditional reinterpret-as-unsigned arms in the cases where signedness matters.

---

## 14.10 — `const`, `volatile`, `auto`, `register`, `restrict`, `_Noreturn`, and array-dimension qualifiers

> `git checkout b77355427575385b6f0b6c0a914600b79b4e4412` — *Ignore const, volatile, auto, register, restrict or _Noreturn.*
>
> `git checkout 93d12771d009924fb598b088dc4bd9b67fd9a09a` — *Ignore "static" and "const" in array-dimensions*

Two commits, bundled because both are *parse-and-discard* in the same spirit. The first picks up the C qualifier and storage-class soup that chibicc has been quietly choking on (or, in some cases, that programmers haven't tried because they knew chibicc would choke). The second picks up the C99 array-dimension qualifiers that even seasoned C programmers rarely write — `void f(int x[static 10])` and `void f(int x[const 10])` — but that real C headers occasionally include.

What chibicc is choosing not to enforce: `const` (immutability of the named object), `volatile` (preventing the compiler from optimizing reads/writes), `auto` (the storage-class for "default block-scope local," historically meaningful before `static`/`extern` made the contrast explicit), `register` (a hint that a variable is a candidate for register allocation), `restrict` (pointer aliasing guarantee, C99), `_Noreturn` (function-attribute hint, C11), and the C99 `static`/`const` shapes inside parameter array dimensions.

Each of these implies behavior that chibicc doesn't enforce. `const` would require a write-through-`const` check (and no codegen change since chibicc doesn't optimize). `volatile` would require disabling the (nonexistent) optimizer's load/store coalescing. `register` would require a register-allocation pass that chibicc doesn't have. `restrict` would let an optimizer alias-analyze around the pointer, which chibicc again doesn't do. `_Noreturn` would let an analyzer omit dead-code-after-call warnings, none of which chibicc emits. So the cost of *not* enforcing them is precisely zero, and the value of *parsing* them is that real C source code containing them now compiles instead of failing the type-recognition check.

The first commit's mechanism is in two pieces. In `declspec`, a new conditional ignores the qualifier keywords without setting any flag:

```c
// These keywords are recognized but ignored.
if (consume(&tok, tok, "const") || consume(&tok, tok, "volatile") ||
    consume(&tok, tok, "auto") || consume(&tok, tok, "register") ||
    consume(&tok, tok, "restrict") || consume(&tok, tok, "__restrict") ||
    consume(&tok, tok, "__restrict__") || consume(&tok, tok, "_Noreturn"))
  continue;
```

`consume` is the variant of `equal`-and-advance that returns `true` if the token matched (advancing `tok`) and `false` otherwise, so the chain of `||`s eats whichever keyword is present. The loop then `continue`s, skipping the type-counting logic that would otherwise reject the keyword as unknown. `__restrict` and `__restrict__` are GCC's spellings of `restrict` for use in headers that target both C89 (where `restrict` doesn't exist) and C99 — the implementations are identical from chibicc's standpoint.

In the declarator parsing, qualifiers between `*` and the next item are similarly eaten. The change splits the existing `declarator` into two — a new `pointers` helper handles the `*` runs, and `declarator` calls it:

```c
// pointers = ("*" ("const" | "volatile" | "restrict")*)*
static Type *pointers(Token **rest, Token *tok, Type *ty) {
  while (consume(&tok, tok, "*")) {
    ty = pointer_to(ty);
    while (equal(tok, "const") || equal(tok, "volatile") || equal(tok, "restrict") ||
           equal(tok, "__restrict") || equal(tok, "__restrict__"))
      tok = tok->next;
  }
  *rest = tok;
  return ty;
}
```

Each `*` produces a `pointer_to` wrapper, then any qualifiers that follow are skipped. The wrapped type doesn't carry the qualifier — `int *const` and `int *` produce the same `Type` chain (`pointer_to(ty_int)`), with the `const`-ness of the *pointer itself* (as opposed to the pointee) discarded. The same `pointers` helper is then called from `abstract_declarator` (used in casts and `sizeof typename`) to pick up the same rule there.

`is_typename` gains the new keywords too:

```c
static char *kw[] = {
  "void", "_Bool", "char", "short", "int", "long", "struct", "union",
  "typedef", "enum", "static", "extern", "_Alignas", "signed", "unsigned",
  "const", "volatile", "auto", "register", "restrict", "__restrict",
  "__restrict__", "_Noreturn",
};
```

— the predicate that decides whether a token starts a type now accepts the qualifiers, so a declaration like `const int x;` is recognized as a type-starting sequence rather than as an expression-statement starting with an unknown identifier.

The test surface in the new `test/compat.c` is a tour of qualifier shapes:

```c
_Noreturn noreturn_fn(int restrict x) {
  exit(0);
}

int main() {
  { volatile x; }
  { int volatile x; }
  { volatile int x; }
  { volatile int volatile volatile x; }
  { int volatile * volatile volatile x; }
  { auto ** restrict __restrict __restrict__ const volatile *x; }
  // ...
}
```

Each line is testing that a particular qualifier shape parses and reaches the end of the program. The test doesn't `ASSERT` anything except that `OK` is printed at the end, because there's nothing semantically distinct to check — the qualifiers were dropped, and the variables work like any other variable.

The companion `test/const.c` is the same idea narrowed to `const`:

```c
{ const x; }
{ int const x; }
{ const int x; }
{ const int const const x; }
ASSERT(5, ({ const x = 5; x; }));
ASSERT(8, ({ const x = 8; int *const y=&x; *y; }));
ASSERT(6, ({ const x = 6; *(const * const)&x; }));
```

The `const x = 5;` — with no type specifier, just `const` — is the K&R-era *implicit int* style: when `const` appears without an explicit type, the type defaults to `int`. The `declspec` switch's bare-`SIGNED` arm (resolving to `ty_int`) is the precedent; bare `const` does the same. Then `*(const * const)&x` uses `const` in the cast, exercising the `pointers` helper's `const`-after-`*` skip.

The second commit (`93d1277`) is even smaller. C99 allows `static` and `restrict` (and other qualifiers) inside parameter array dimensions: `void f(int x[static 10])` says "x will always point to an array of at least 10 ints" — a hint to the optimizer. `void f(int x[restrict 10])` likewise marks the pointer as `restrict`. Chibicc parses these and discards them:

```c
// array-dimensions = ("static" | "restrict")* const-expr? "]" type-suffix
static Type *array_dimensions(Token **rest, Token *tok, Type *ty) {
  while (equal(tok, "static") || equal(tok, "restrict"))
    tok = tok->next;
  // ...
}
```

A two-line skip. The added test is a single function declaration:

```c
void funcy_type(int arg[restrict static 3]) {}
```

— qualifier ordering doesn't matter; chibicc accepts any combination (and any count) of the two keywords before the dimension.

These commits represent a small but deliberate trade-off. Chibicc parses the qualifiers because real C code (and real C headers, especially glibc's) uses them. The implementations they imply — `const` enforcement, `volatile` write-through, `restrict` aliasing-permitted, `register` hints, `_Noreturn` flow analysis — are not on the chibicc roadmap. Most of them don't need to be, because chibicc's codegen doesn't optimize. The compiler that emits `mov %rax, -8(%rbp); mov -8(%rbp), %rax` for every load (Chapter 4) doesn't need the optimizer-correctness guarantees that `volatile` exists to provide. `const` would be a real diagnostic improvement if it caught write-through-`const` errors, but the test suite doesn't depend on any such diagnostic, and chibicc isn't in the business of catching errors that the compiled program would otherwise quietly miscompute. Parse-and-discard is the honest answer: the *syntax* is real, the *semantics* aren't.

### Where we are

Eight new keywords (`const`, `volatile`, `auto`, `register`, `restrict`, `__restrict`, `__restrict__`, `_Noreturn`) and three contexts (declspec, post-`*`, array dimensions) all parse and discard. The `pointers` helper is a small refactor that hoists the `*`-and-qualifiers loop out of `declarator`. The `is_typename` predicate grows commensurately. No `Type` field, no `Obj` field, no codegen change. The chapter's largest concentration of *accept-but-don't-enforce* — the same trade-off that has been the backbone of chibicc's compatibility story since Chapter 7's `f()`-as-`f(void)`.

---

## 14.11 — Omitting the parameter name in a declaration

> `git checkout 1fad2595d6fa67e57cd795d4faac4306e42e72c5` — *Allow to omit parameter name in function declaration*

The chapter's closing patch. C allows function declarations to omit parameter names: `int f(int);` declares `f` as taking one `int`-typed argument, with no name for it (because the declaration doesn't need one). This shape shows up in real headers — every `printf`-style declaration in `<stdio.h>` does it for `int (*)(...)` callback parameters — and chibicc has been rejecting it.

The change is in `declarator`, which previously required the next token after the type to be an identifier:

```c
if (tok->kind != TK_IDENT)
  error_tok(tok, "expected a variable name");
ty = type_suffix(rest, tok->next, ty);
ty->name = tok;
```

— now relaxed to accept a missing name:

```c
Token *name = NULL;
Token *name_pos = tok;

if (tok->kind == TK_IDENT) {
  name = tok;
  tok = tok->next;
}

ty = type_suffix(rest, tok, ty);
ty->name = name;
ty->name_pos = name_pos;
```

`ty->name` becomes nullable (`NULL` if there was no identifier), and a new `ty->name_pos` always points to *where the name should have been* — used for error messages later if the missing name turns out to be a problem.

The call sites that need a name now check for it explicitly. There are five: `declaration` (for ordinary local-variable declarations), `parse_typedef` (for `typedef T Name;`), `create_param_lvars` (for function-definition parameters), `function` (for the function name itself), and `global_variable` (for ordinary file-scope declarations). Each gets a `if (!ty->name) error_tok(ty->name_pos, "X name omitted");` guard. The pattern:

```c
Type *ty = declarator(&tok, tok, basety);
if (!ty->name)
  error_tok(ty->name_pos, "variable name omitted");
```

— or with "typedef name omitted", or "parameter name omitted", or "function name omitted", as appropriate. The single permissive site is `func_params`, which builds the *prototype's* parameter list without requiring names. So:

- `int f(int x);` — declaration with named parameter — works.
- `int f(int);` — declaration with unnamed parameter — works (new).
- `int f(int) { return 1; }` — definition with unnamed parameter — errors via `create_param_lvars`.
- `int f(int x) { return x; }` — definition with named parameter — works.
- `int x;` — variable declaration without name — errors.
- `typedef int T;` — typedef without name — errors.

The distinction between *declaration* (where the parameter doesn't need a name because nothing in the declaration body references it) and *definition* (where the parameter name is what the body uses to refer to the value) is the load-bearing one. The relaxation is targeted at declarations only.

`Obj` also gains a `Token *tok` field for "representative token," but it isn't used by any of the changes in this commit. It's added for use by later commits (the rest of this chapter's commits don't reference it; it'll surface in later chapters).

The commit doesn't add a test file. The new prototype shapes — chibicc's own test header is using `printf(char *fmt, ...)` since §14.1, and the relaxation is what makes related shapes work — are exercised implicitly by the test suite. There's no `test/declarator.c` or similar. The fix is small, the impact is broad: every test header that wants to declare a glibc function with unnamed parameters can now do so.

### Where we are

`declarator` accepts a missing identifier and stores `NULL` on the type. The five call sites that need names check for `NULL` and emit "X name omitted" errors via the new `name_pos` token. The one site that doesn't (`func_params`) is what makes unnamed parameters in declarations legal. The chapter's smallest closer; the relaxation closes one more compatibility gap in chibicc's parser.

---

## Where the chapter leaves us

Twelve commits, eleven sections, and three threads woven through. Variadics on both sides (call and definition); a fully-doubled integer type system with codegen support throughout; a wave of qualifier accept-and-discard.

| Commit | Topic |
|---|---|
| `58fc861` | Variadic call sites. `is_variadic` on function `Type`; `...` as a punctuator; `func_params` reads it. The Chapter 5 §5.1 `mov $0, %rax` finally does real work. |
| `754a24f` | `va_start` machinery. 136-byte `__va_area__` local for the register save area; prologue spills six integer registers and eight XMM registers, populates the four `va_list` header fields. `va_start` is user-side: `*ap = *(__va_elem *)__va_area__;`. |
| `197689a` | Argument-count check in `funcall`: too-few and too-many become errors. Empty-parameter-list functions treated as variadic. |
| `3f59ce7` | `signed` keyword. New `SIGNED` enum bit in `declspec`; eight new switch arms; tokenizer keyword. Resolves to the same types as the unmodified keyword. |
| `34ab83b` | Unsigned integral types. `is_unsigned` flag on `Type`; four new `ty_u*` globals; `UNSIGNED` enum bit; doubled cast table; `idiv`/`div` and `setl`/`setb` and `sar`/`shr` codegen splits; usual-arithmetic-conversion rule in `get_common_type`. |
| `aaf1045` | Integer suffixes. `read_int_literal` parses `U`/`L`/`LL` and assigns the right type to the token; `add_type` for `ND_NUM` simplifies. |
| `8b8f3de` | `sizeof`/`_Alignof` return `ulong`; pointer subtraction returns `long`. Five `new_num` → `new_ulong` swaps and one `ty_int` → `ty_long`. |
| `6880a39` | Pointer comparison as unsigned. `pointer_to` flags `is_unsigned = true`; codegen picks `setb`/`setbe`. Two-line patch with an ABI-correctness story. |
| `7ba6fe8` | Unsigned arithmetic in `eval2`. New arms for `ND_DIV`/`ND_MOD`/`ND_SHR`/`ND_LT`/`ND_LE` reinterpret as `uint64_t`; cast arm picks signed or unsigned narrowing. |
| `b773554` | Qualifier soup parsed-and-discarded: `const`, `volatile`, `auto`, `register`, `restrict`, `__restrict`, `__restrict__`, `_Noreturn`. New `pointers` helper handles `*`-and-qualifiers. |
| `93d1277` | `static`/`restrict` in parameter array dimensions parsed-and-discarded. Two-line patch in `array_dimensions`. |
| `1fad259` | Empty parameter name allowed in declarations. `ty->name` is nullable; `ty->name_pos` records where the name would have been; five call sites check for `NULL` and error if needed. |

Three structural moves deserve calling out.

The first is the *VarAttr channel did not grow this chapter*. Chapter 13 forecast that signed/unsigned plus the qualifier set would route through `VarAttr`. They didn't. `signed`/`unsigned` are picked up by the `declspec` counter machinery (the bit-flag accumulator that has tracked type specifiers since Chapter 9), which is parallel to but distinct from `VarAttr`. The qualifier set (`const`, `volatile`, etc.) is consumed by a `consume`-and-`continue` block in `declspec`'s loop, also bypassing `VarAttr` because chibicc doesn't track the qualifiers anywhere. The channel stays at four fields (`is_typedef`, `is_static`, `is_extern`, `align`). The forecast wasn't quite right: the channel is for things that need to flow from the type-spec parser to the declarator-or-`Obj` consumer, and chibicc never makes downstream use of the qualifier or signedness flags as `VarAttr` properties. Signed/unsigned ends up on the `Type` itself (`is_unsigned` flag); qualifiers end up nowhere. Both are correct designs; neither needs `VarAttr`.

The second is the *`is_unsigned`-flag-on-Type pattern*. The choice between flag-on-existing-kind and separate-kind-per-variant is an old one — it would have been just as easy to add `TY_UCHAR`, `TY_USHORT`, `TY_UINT`, `TY_ULONG` as separate kinds, and have every codegen and type-system switch fan out to handle eight kinds instead of four. The flag is the lighter weight choice: the existing `case TY_INT:` arms still work, with each one adding a `ty->is_unsigned ?` branch where the signedness matters. The cast table doubles, but most other code paths gain only a single conditional. If chibicc grows more variants on the same axis later (say, big-endian vs. little-endian, or atomic vs. non-atomic), the flag pattern scales linearly while separate-kinds scales multiplicatively. The choice is a small but durable design call.

The third is the *psABI conformance thread continues*. Chapter 13 §13.8 (16-byte stack alignment) and §13.9 (small-return truncation) opened the thread. Chapter 14 adds three more: §14.1's `mov $0, %rax` finally serves a real variadic call site, §14.2's register-save-area layout is byte-for-byte psABI-conformant (the 24-byte header, the 48-byte gp area, the 64-byte fp area), and §14.8 fixes pointer comparison to unsigned (which the psABI implies via address-space semantics). Every one of these is a chibicc-side correction to match a guarantee that the calling convention requires. The thread will continue: floating-point parameters in Chapter 15 will need the XMM-register slots that §14.2 already spills (currently spilled but unused).

The chapter doesn't add to the *canonicalization-at-parse-time* count (still eight: nothing in this chapter rewrites AST shape at parse time). The chapter doesn't add to the *pre-factor-before-feature* count (still six; the `pointers` helper in §14.10 is a small refactor done as part of the same commit, not as a separate pre-factor). The chapter doesn't add a fifth namespace (still four: ordinary identifiers, struct tags, labels, typedef names). The chapter does extend `is_typename`'s array significantly: nine new keywords (`signed`, `unsigned`, `const`, `volatile`, `auto`, `register`, `restrict`, `__restrict`, `__restrict__`, `_Noreturn`) get added across §14.4, §14.5, and §14.10.

A few standing notes carried forward to Chapter 15:

- The `is_unsigned` flag on `Type` is a per-kind orthogonal axis. Floating-point types (Chapter 15) will introduce new kinds (`TY_FLOAT`, `TY_DOUBLE`); the `is_unsigned` flag is irrelevant for them.
- The register-save-area layout (`__va_area__`, 136 bytes, 24-byte header + 48 gp + 64 fp) is psABI-conformant. Chapter 15's floating-point parameters will populate the fp half of the area in addition to consuming it (currently it's spilled but no `va_arg` reads it).
- The `__va_area__` magic local-variable name is a chibicc-specific user-side hook for `va_start`. Real C compilers implement `va_start` as a builtin; chibicc punts to user code by exposing the spill area by name.
- The constant evaluator's shape (`eval`/`eval2`/`eval_rval`) is unchanged. Chapter 14 extended its arms to handle unsigned arithmetic without changing the trio's structure.
- The pointer-as-unsigned tagging in `pointer_to` (`ty->is_unsigned = true`) is structurally orthogonal to the integer signedness — pointers happen to be unsigned because addresses are, but the flag is the same flag.
- The qualifier accept-and-discard pattern is now well-established; future C qualifier additions (atomic, thread-local) would route through the same `consume`/`continue` block.
- The unnamed-parameter relaxation (§14.11) means `int f(int);` is now acceptable. The `f()`-as-variadic relaxation (§14.3) means `int f();` is still acceptable. Real C distinguishes the two; chibicc collapses them.
- The `mov $0, %rax` (Chapter 5 §5.1) is now closed-loop. The chapter prose calls it out; it remains pessimistic (emitted before *every* call, not just variadic ones) because chibicc doesn't know which is which at the call site without symbol-table lookup. The pessimism costs one instruction per call.
- The `ND_MEMZERO` `mov $0, %al` (`rep stosb` source byte) is unrelated to variadics; it's been there since Chapter 12. Worth noting because the two `mov $0, %al`-shape instructions can look related but aren't.

Forward references for Chapter 15:

- Floating-point types: `float` and `double` arrive as new `TY_*` kinds, with their own register set (`%xmm0`–`%xmm7` for arguments, `%xmm0`/`%xmm1` for returns). The fp half of the §14.2 spill area will start being populated and consumed.
- Floating-point literals (`1.0`, `1e10`, `0x1p0`). These will need the anonymous-global pattern from Chapter 13 §13.4, since fp literals can't be represented inline as `mov $imm, %reg` — they need to live in `.data` and be loaded from memory.
- Floating-point arithmetic and casts. New codegen for `addsd`/`subsd`/`mulsd`/`divsd` plus the integer-to-float and float-to-integer cast instructions.
- The `__va_arg` macro for floating-point arguments will need to be aware of the `fp_offset` field that §14.2 currently populates with `0`.

Eleven commits in Chapter 15, mostly self-contained around floating-point.
