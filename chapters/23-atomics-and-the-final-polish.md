# Chapter 23 — Atomics and the final polish

> Commits covered: `ca27455`, `80ea9d4`, `d69a11d`, `0a5d08c`, `2ed3fda`, `395308c`, `44bea4c`, `b35d148`, `982041f`, `90d1f7f`. Ten commits — the last chapter, closing the project. Two atomics-shaped commits add `__builtin_compare_and_swap` and `__builtin_atomic_exchange`. A larger third commit teaches the type system about `_Atomic` and rewrites compound-assignment to a CAS loop when the destination is atomically qualified. `stdatomic.h` gets fleshed out. A shell script tries to compile cpython. A small parse.c cleanup converts the function-definition path into one that detects redeclarations. Two `__attribute__` parsers land — `packed` and `aligned`. The README is finally written. And the project's very last commit, on December 7, 2020, is a seven-line codegen patch that lets the address-of operator look through `=` and `?:` for struct values.

This is the chapter where the story ends. After it, Rui's `main` branch goes quiet. There are no further commits to walk because there are no further commits, period. That makes the section grouping easier than usual — the commits cluster naturally into atomics, attribute-driven struct layout, and the small odds and ends — and it makes the closing recap a different shape than the previous twenty-two. The recap surveys the whole compiler, not just this chapter.

Eight sections from ten commits.

- **§23.1** — `__builtin_compare_and_swap` and `__builtin_atomic_exchange` (commits 307–308).
- **§23.2** — `_Atomic` and the CAS loop for atomic compound-assignment (commit 309).
- **§23.3** — `stdatomic.h` (commit 310).
- **§23.4** — The cpython script (commit 311).
- **§23.5** — Redeclaration of functions (commit 312).
- **§23.6** — `__attribute__((packed))` and `__attribute__((aligned))` (commits 313–314).
- **§23.7** — The README (commit 315).
- **§23.8** — Address-of through `=` and `?:` (commit 316).

The chapter follows `main` order. The dates are scattered across mid-September through early December 2020 — the atomics arc lands in two days, the attribute commits stretch over two weeks, and the very last commit (the seven-line `gen_addr` patch) is dated almost three months after the README was written. The prose ignores the dates except where they matter.

---

## 23.1 — Atomic builtins: compare-and-swap and exchange

> `git checkout ca27455b92be2ffbfe58c7ffda623cf6ec112632` — *Add atomic_compare_exchange*
> `git checkout 80ea9d427c5041415b014a0a97193f1f7e0a871b` — *Add atomic_exchange*

The two atomics in this section are the building blocks of every higher-level lock-free pattern: a compare-and-swap that atomically updates a memory location only if its current value matches an expected one, and an exchange that atomically swaps a register with a memory location and returns the old value. Both are single x86 instructions with a `lock` prefix; chibicc's job is to get the operands into the right registers and emit them.

### `__builtin_compare_and_swap`

The first commit adds a node kind, a parser arm, an `add_type` arm, and a codegen arm. The library-side name is `atomic_compare_exchange_weak` (and `_strong`); both expand via `stdatomic.h` to the chibicc-internal builtin:

```c
#define atomic_compare_exchange_weak(p, old, new) \
  __builtin_compare_and_swap((p), (old), (new))
```

`__builtin_compare_and_swap(p, old, new)` takes a pointer to the destination, a pointer to the expected old value, and the new value to install. It returns `1` if the swap happened (`*p` was equal to `*old`) and `0` if it didn't (in which case `*old` is overwritten with the actual current value). That's the C11 contract.

The parser recognizes the builtin name in `primary` and reads three argument expressions:

```c
if (equal(tok, "__builtin_compare_and_swap")) {
  Node *node = new_node(ND_CAS, tok);
  tok = skip(tok->next, "(");
  node->cas_addr = assign(&tok, tok);
  tok = skip(tok, ",");
  node->cas_old = assign(&tok, tok);
  tok = skip(tok, ",");
  node->cas_new = assign(&tok, tok);
  *rest = skip(tok, ")");
  return node;
}
```

The three arguments live on dedicated `Node` fields — `cas_addr`, `cas_old`, `cas_new` — rather than reusing `lhs`/`rhs`/some-third-slot. `add_type` sets the result type to `ty_bool` and checks that the first two arguments are pointers.

The codegen is the interesting part. It needs to put four things in four specific registers (because `cmpxchg` is implicit about which registers it reads):

```c
case ND_CAS: {
  gen_expr(node->cas_addr);
  push();
  gen_expr(node->cas_new);
  push();
  gen_expr(node->cas_old);
  println("  mov %%rax, %%r8");
  load(node->cas_old->ty->base);
  pop("%rdx"); // new
  pop("%rdi"); // addr
```

The walk evaluates the three arguments in order, parking the first two on the stack. After the third (`cas_old`, a pointer) is in `%rax`, that pointer is copied into `%r8` (we'll need it again in a moment) and then dereferenced — `load` reads the pointed-to value into `%rax`. The two stacked operands come back into `%rdx` (the new value) and `%rdi` (the destination address). At this point `%rax` holds the *expected* old value, `%rdx` holds the new value, `%rdi` points at the destination, and `%r8` points at the slot we'd write the actual-old-value back to on failure.

That register layout is what `cmpxchg` wants. The instruction compares `%rax` against `(%rdi)`; if equal, writes `%rdx` into `(%rdi)` and sets the zero flag; if not equal, writes `(%rdi)` into `%rax` and clears the zero flag. The `lock` prefix makes the read-compare-write a single bus-locked transaction.

```c
  int sz = node->cas_addr->ty->base->size;
  println("  lock cmpxchg %s, (%%rdi)", reg_dx(sz));
  println("  sete %%cl");
  println("  je 1f");
  println("  mov %s, (%%r8)", reg_ax(sz));
  println("1:");
  println("  movzbl %%cl, %%eax");
  return;
}
```

`sete %cl` sets `%cl` to 1 if the swap succeeded (zero flag set), 0 otherwise. On the failure path (`jne` would follow but here `je` jumps over the store), `%rax` holds the actual current value of `(%rdi)`, which the C11 contract says must be written back through the `cas_old` pointer — that's the `mov %eax, (%r8)`. On success, the store is skipped (`je 1f`). Either way, the boolean result in `%cl` is zero-extended into `%eax` and that's the function's return value.

Two helper functions are added at the top of `codegen.c` to pick the right register name for a given operand size:

```c
static char *reg_dx(int sz) { switch (sz) { case 1: return "%dl"; ... } }
static char *reg_ax(int sz) { switch (sz) { case 1: return "%al"; ... } }
```

These let the same code template work for `_Atomic char`, `_Atomic short`, `_Atomic int`, and `_Atomic long`. The size comes from the pointee type of `cas_addr`.

### `__builtin_atomic_exchange`

The second commit is a smaller version of the same pattern. `atomic_exchange(p, val)` atomically writes `val` into `*p` and returns the old `*p`. The x86 implementation is one instruction: `xchg`. Unlike `cmpxchg`, the `xchg` instruction has an *implicit* `lock` prefix when its operand is a memory location — Intel's manual is explicit that the operand becomes locked for the duration of the swap, regardless of whether `lock` was written.

```c
case ND_EXCH: {
  gen_expr(node->lhs);
  push();
  gen_expr(node->rhs);
  pop("%rdi");
  int sz = node->lhs->ty->base->size;
  println("  xchg %s, (%%rdi)", reg_ax(sz));
  return;
}
```

Two arguments — destination pointer and new value — fit in `lhs` and `rhs`, so no dedicated fields. After the walk, `%rax` holds the new value and `%rdi` points at the destination. `xchg %rax, (%rdi)` swaps them, leaving the *old* value in `%rax` (the function's return value).

The header gets two more entries:

```c
#define atomic_exchange(obj, val)                  __builtin_atomic_exchange(obj, val)
#define atomic_exchange_explicit(obj, val, order)  __builtin_atomic_exchange(obj, val)
```

The `_explicit` variant takes a `memory_order` argument that chibicc silently discards. C11 lets you ask for `memory_order_relaxed`, `memory_order_acquire`, `memory_order_seq_cst`, etc.; the standard guarantees seq-cst as the strongest, and that's what x86's locked instructions deliver unconditionally. Asking for relaxed and getting seq-cst is correct (it's stronger than required); the code that depends on exactly-relaxed semantics for performance will not see the optimization, but won't break either. Rui doesn't try to honor the order parameter; on x86 with a small compiler, the easy answer is also a correct one.

### The test

`test/atomic.c` is new in this commit. It spawns three threads that all hammer the same `int` with `atomic_compare_exchange_weak`-driven increments, plus a main thread that does the same:

```c
static int incr(int *p) {
  int oldval = *p;
  int newval;
  do {
    newval = oldval + 1;
  } while (!atomic_compare_exchange_weak(p, &oldval, newval));
  return newval;
}
```

This is the canonical CAS retry pattern. Read the current value into `oldval`, compute the desired new value, attempt the swap; if it fails, the C11 contract has already updated `oldval` to the *real* current value (that's why the third argument is `&oldval`, not `oldval`), so the loop retries with the fresh data. After three threads each do 1,000,000 increments, the value should be 3,000,000 — and the test asserts exactly that.

**Where we are.** Two atomic primitives are in. `__builtin_compare_and_swap` lowers to `lock cmpxchg`; `__builtin_atomic_exchange` lowers to `xchg` (implicitly locked). The result types and operand layouts conform to the C11 atomics spec, modulo the discarded memory-order parameter. That parameter is moot on x86 because the locked-instruction paths are seq-cst regardless. The next commit makes the `_Atomic` qualifier real and arranges for ordinary `++`, `--`, `+=`, etc. on atomic-qualified destinations to expand into a CAS loop.

---

## 23.2 — `_Atomic` and the CAS-loop expansion

> `git checkout d69a11dd25a77c2b9390e54c9f9e8967456cb642` — *Add _Atomic and atomic ++, -- and op= operators*

The previous section gave us two builtins. This one teaches chibicc the `_Atomic` *qualifier*, so a programmer can write `_Atomic int x = 0;` and have the compiler track that `x` requires atomic access. More importantly, it teaches the parser to rewrite `x++`, `x--`, `x += 5` and so on into a CAS-loop when `x` is `_Atomic`-qualified. The user gets ordinary C syntax; the compiler issues correct lock-free code.

This is the most substantive commit in the chapter. About 80 lines of new parse.c code, a new `Type` flag, and a tokenizer keyword.

### The `is_atomic` flag

`Type` gains one new bool:

```c
struct Type {
  ...
  bool is_unsigned;
  bool is_atomic;     // true if _Atomic
  Type *origin;
  ...
};
```

`_Atomic` is a *qualifier*, not a type — like `const` or `volatile`, it attaches to a base type. The flag-on-`Type` representation matches the way Rui has handled qualifiers throughout: `const` is silently ignored, `volatile` is silently ignored, and now `_Atomic` is a flag whose only effect is at the assignment-rewrite site.

`_Atomic` joins the keyword tables in both `tokenize.c` and `parse.c`. In `declspec`, the keyword can appear in two syntactic positions. The plain `_Atomic int x;` form is just another type-modifier in the declspec loop. The function-style `_Atomic(int) x;` form is a way to qualify a type without writing it inline — the parenthesized form parses a fresh type-name and then sets the atomic flag on it:

```c
if (equal(tok, "_Atomic")) {
  tok = tok->next;
  if (equal(tok , "(")) {
    ty = typename(&tok, tok->next);
    tok = skip(tok, ")");
  }
  is_atomic = true;
  continue;
}
```

The local `is_atomic` flag is consolidated with the result `Type` at the bottom of `declspec`, after all other modifiers are applied:

```c
if (is_atomic) {
  ty = copy_type(ty);
  ty->is_atomic = true;
}
```

The `copy_type` is necessary because `ty_int`, `ty_long`, etc. are shared singleton types. Marking the singleton would mark every other declaration of the same plain type. The copy-then-flag pattern is what chibicc already uses for other per-variable type properties.

### The CAS-loop expansion

The interesting work is in `to_assign`, the helper that converts `A op= B` into an executable form. Until this commit, `to_assign` had a single path: bind `&A` to a temporary `tmp`, then evaluate `*tmp = *tmp op B`. The double-bind avoids re-evaluating `A`'s side effects.

For atomic destinations, that simple form isn't lock-free — between the load (`*tmp`) and the store (`*tmp = ...`), another thread could update the location, and the store would silently overwrite the update. The lock-free fix is the CAS retry pattern: load the current value, compute the desired result, attempt the swap; on failure, retry with whatever value the location now holds.

The new branch in `to_assign` runs first:

```c
if (binary->lhs->ty->is_atomic) {
  Node head = {};
  Node *cur = &head;

  Obj *addr = new_lvar("", pointer_to(binary->lhs->ty));
  Obj *val  = new_lvar("", binary->rhs->ty);
  Obj *old  = new_lvar("", binary->lhs->ty);
  Obj *new  = new_lvar("", binary->lhs->ty);
  ...
}
```

Four anonymous local variables — a pointer to the destination, the right-hand-side value, a slot for the loaded current value, and a slot for the computed new value. The empty-string `new_lvar("", ty)` pattern is the same one chibicc uses for other compiler-introduced temporaries.

The first three statements bind these locals:

```c
cur = cur->next = new_unary(ND_EXPR_STMT,
    new_binary(ND_ASSIGN, new_var_node(addr, tok),
               new_unary(ND_ADDR, binary->lhs, tok), tok), tok);

cur = cur->next = new_unary(ND_EXPR_STMT,
    new_binary(ND_ASSIGN, new_var_node(val, tok), binary->rhs, tok), tok);

cur = cur->next = new_unary(ND_EXPR_STMT,
    new_binary(ND_ASSIGN, new_var_node(old, tok),
               new_unary(ND_DEREF, new_var_node(addr, tok), tok), tok), tok);
```

In source-equivalent form: `addr = &A; val = B; old = *addr;`. Capturing the address and value first guarantees that side effects in `A` and `B` happen exactly once — the loop body that follows reads from the locals, not from the original expressions.

The retry loop is built next, as an `ND_DO`:

```c
Node *loop = new_node(ND_DO, tok);
loop->brk_label = new_unique_name();
loop->cont_label = new_unique_name();

Node *body = new_binary(ND_ASSIGN,
    new_var_node(new, tok),
    new_binary(binary->kind, new_var_node(old, tok),
               new_var_node(val, tok), tok), tok);

loop->then = new_node(ND_BLOCK, tok);
loop->then->body = new_unary(ND_EXPR_STMT, body, tok);

Node *cas = new_node(ND_CAS, tok);
cas->cas_addr = new_var_node(addr, tok);
cas->cas_old  = new_unary(ND_ADDR, new_var_node(old, tok), tok);
cas->cas_new  = new_var_node(new, tok);
loop->cond = new_unary(ND_NOT, cas, tok);
```

The body computes `new = old op val` — `binary->kind` is `ND_ADD`, `ND_SUB`, `ND_BITAND`, etc., whichever operator the source `op=` was. The condition is `!__builtin_compare_and_swap(addr, &old, new)`. Recall from §23.1 that CAS returns true on success and updates `*addr` only when the expected-old matched; on failure it overwrites `old` with the actual current value, which is exactly what the retry needs. So the `do { ... } while (!cas)` re-runs the body with the freshly-updated `old` until the swap succeeds.

The whole thing is wrapped in a `ND_STMT_EXPR` whose tail expression is `new`:

```c
cur = cur->next = loop;
cur = cur->next = new_unary(ND_EXPR_STMT, new_var_node(new, tok), tok);

Node *node = new_node(ND_STMT_EXPR, tok);
node->body = head.next;
return node;
```

So `x += 5`, with `x` of type `_Atomic int`, parses to roughly:

```c
({ _Atomic int *addr = &x; int val = 5; int old = *addr; int new;
   do { new = old + val; } while (!__builtin_compare_and_swap(addr, &old, new));
   new; })
```

That's the lock-free atomic add pattern, written by hand many times in concurrent code, here generated automatically by the parser.

### `++`, `--`, op= all share the path

`to_assign` is reached from the parser arms that expand `++`, `--`, `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=`. All of them route through this single function, which now means: if the destination is `_Atomic`, you get a CAS loop; otherwise you get the straight load-op-store. No further per-operator code is needed.

The test exercises all three. Three threads run `(*x)++`, `*x += 5`, and the explicit-CAS `incr` from §23.1; main runs `x--`. After each thread does 1,000,000 iterations, the expected value is `1*1M + 5*1M + 1*1M - 1*1M = 6,000,000`. The test asserts exactly that. The whole thing is portable race-free C; chibicc's atomic-aware codegen is what makes it work.

### A note on dead fields

The same commit adds two fields to `Node` that are never read or written:

```c
// Atomic op= operators
Obj *atomic_addr;
Node *atomic_expr;
```

A grep across the source after this commit (and at the end of `main`) finds no producers and no consumers. The CAS-loop expansion synthesizes its temporaries via `new_lvar` and threads them through `new_var_node`; it doesn't touch these fields. Most likely Rui sketched a different representation for the atomic-op= node (maybe a single `ND_ATOMIC_OP` kind that stored the destination address and the right-hand expression), then switched to the AST-construction approach above and forgot to delete the abandoned fields. They are dead weight in `chibicc.h`. Errata candidate.

**Where we are.** `_Atomic int x;` declares an atomic-qualified variable. `x++`, `x--`, and every compound-assignment operator rewrite to a CAS loop in the parser. The codegen path runs the loop body without further atomicity work — only the trailing `cmpxchg` inside the loop needs the lock prefix. The op-assigns, the increment/decrement, and the explicit `atomic_compare_exchange_*` calls all funnel through the single `ND_CAS` codegen arm. Plain assignment (`x = 5;`) on an atomic-qualified destination doesn't go through `to_assign` at all — it parses to a plain `ND_ASSIGN` and codegen emits a straight store. That's a single x86 word-sized store, which is atomic on x86 by hardware fiat; chibicc gets atomicity for free in that case.

---

## 23.3 — `stdatomic.h`

> `git checkout 0a5d08c8f8a72e39828e7b1910c55174e6c8dd5e` — *Complete stdatomic.h*

The header was a four-line stub after §23.1; this commit fills it out. The header now defines the lock-free constants, the `memory_order` enum, the load/store/fetch macros, the flag operations, the per-base-type `atomic_*` typedefs, and a few "do nothing" macros that satisfy the C11 spec without needing real codegen.

A single line vanishes from `preprocess.c`:

```c
-  define_macro("__STDC_NO_ATOMICS__", "1");
```

That predefined macro told user code (and library headers) that `<stdatomic.h>` was not available. Removing it asserts the opposite: chibicc supports C11 atomics. Library code that gates on `#ifndef __STDC_NO_ATOMICS__` will now reach the atomic codepath.

### Macros that defer to plain ops

The bulk of the header is macros that look like atomic operations but expand to plain pointer-dereferences:

```c
#define atomic_load(addr)            (*(addr))
#define atomic_store(addr, val)      (*(addr) = (val))

#define atomic_fetch_add(obj, val)   (*(obj) += (val))
#define atomic_fetch_sub(obj, val)   (*(obj) -= (val))
#define atomic_fetch_or(obj, val)    (*(obj) |= (val))
#define atomic_fetch_xor(obj, val)   (*(obj) ^= (val))
#define atomic_fetch_and(obj, val)   (*(obj) &= (val))
```

Why does this work? Because the *type* of `addr` is what makes the access atomic, not the macro. If the user wrote `_Atomic int x; ... atomic_fetch_add(&x, 5);`, the macro expands to `*(&x) += 5;`, the parser sees `+=` on an atomic-qualified destination, and §23.2's CAS-loop fires. The atomic semantics fall out of the type system; the header doesn't need to do anything clever.

The plain `atomic_load(addr)` and `atomic_store(addr, val)` cases similarly delegate to the language. A load from an `_Atomic int *` is a single word read; on x86 that's atomic. A store to one is a single word write; same story. The macros are presentation, not implementation.

The `_explicit` variants drop their `memory_order` parameter — same answer as §23.1's `atomic_exchange_explicit`. The strongest order is what x86's `cmpxchg`/`xchg`/`mov` deliver and what the C11 spec allows the compiler to substitute when asked for anything weaker.

### `atomic_flag`

```c
typedef _Atomic _Bool atomic_flag;

#define atomic_flag_test_and_set(obj)            atomic_exchange((obj), 1)
#define atomic_flag_test_and_set_explicit(obj, order) atomic_exchange((obj), 1)
#define atomic_flag_clear(obj)                   (*(obj) = 0)
#define atomic_flag_clear_explicit(obj, order)   (*(obj) = 0)
```

The C11 `atomic_flag` is the simplest possible synchronization primitive — a one-bit lock. Test-and-set is `xchg(&flag, 1)`, returning the previous value (zero if you got the lock). Clear is a plain store. Both fall through to the §23.1 builtin and to plain assignment respectively.

### Lock-free constants and the no-op macros

```c
#define ATOMIC_BOOL_LOCK_FREE   1
#define ATOMIC_CHAR_LOCK_FREE   1
...
#define ATOMIC_POINTER_LOCK_FREE 1
```

The C11 spec lets each implementation tell the user whether atomic operations on a given type are lock-free. The values are 0 (never lock-free), 1 (sometimes — depends on alignment), or 2 (always). Chibicc reports 1 for all of them, which is technically the most defensive answer; on x86-64 with the natural alignments chibicc gives, the actual answer is 2 for everything up through long, but 1 is also correct.

```c
#define ATOMIC_FLAG_INIT(x)            (x)
#define atomic_init(addr, val)         (*(addr) = (val))
#define kill_dependency(x)             (x)
#define atomic_thread_fence(order)
#define atomic_signal_fence(order)
#define atomic_is_lock_free(x)         1
```

Two macros expand to nothing — `atomic_thread_fence(order)` and `atomic_signal_fence(order)` are statement-position no-ops. C11 fences exist to constrain reordering across the fence; on x86, locked instructions are themselves full fences, and chibicc emits no out-of-order optimizations anyway, so the fences have nothing to do. `kill_dependency` is a `consume`-ordering helper; chibicc treats consume as seq-cst, so it's an identity macro.

### The typedefs

About forty `typedef _Atomic <base> atomic_<name>;` lines cover every C11-required atomic typedef (`atomic_int`, `atomic_long`, `atomic_uint_least32_t`, etc.). With the `_Atomic` qualifier from §23.2 in place, these are one-liners that exist purely to give the user the spec-canonical names.

**Where we are.** `<stdatomic.h>` is the C11-compliant face on top of the §23.1/§23.2 machinery. The header is mostly macros that fall through to the language; the heavy lifting is in `to_assign`'s atomic branch and the two `lock`-prefixed codegen arms. With this commit, `__STDC_NO_ATOMICS__` is no longer predefined, telling library code that atomics work. Atomics is now feature-complete enough that a reasonably-written concurrent program will compile and run correctly.

---

## 23.4 — The cpython script

> `git checkout 2ed3fdafa3d2f60bd1bcdb2bc5df6c1e58c357f7` — *Add test/thirdparty/cpython.sh*

A fifteen-line shell script joins the four third-party scripts from §22.7. It tries to build cpython with chibicc:

```bash
#!/bin/bash
repo='git@github.com:python/cpython.git'
. test/thirdparty/common
git reset --hard c75330605d4795850ec74fdc4d69aa5d92f76c00

# Python's './configure' command misidentifies chibicc as icc
# (Intel C Compiler) because icc is a substring of chibicc.
# Modify the configure file as a workaround.
sed -i -e 1996,2011d configure.ac
autoreconf

CC=$chibicc ./configure
$make clean
$make
$make test
```

The script follows the pattern from §22.7.6 — clone the upstream repo, pin to a specific commit, then drive the project's own build with `CC=$chibicc`. What's interesting is the workaround in the middle. Python's `configure` does compiler-vendor detection by string-matching the compiler name; the substring "icc" appears inside "chibicc", and `configure` concludes it's looking at Intel's compiler. The Intel-compiler branch then sets flags chibicc doesn't recognize, and the build collapses. The fix is to delete the offending sixteen lines from `configure.ac` and re-run `autoreconf` to regenerate `configure` without them.

Cpython is an order of magnitude larger than git, libpng, sqlite, or tinycc — hundreds of thousands of lines, dozens of subsystems, a custom build that's notorious for its idiosyncrasies. That chibicc can build it (the README in §23.7 is honest that it requires patches to the build, not to the C source) is the chapter's quiet milestone. The compiler can produce a working Python interpreter from the upstream source.

The script isn't part of `make test`; like the other third-party scripts, it requires network access and many minutes to run, so it's invoked manually. But its existence rounds out the third-party harness from four pinned codebases to five.

**Where we are.** The third-party harness has five scripts: git, libpng, sqlite, tinycc (all from §22.7) and now cpython. Each pins an upstream commit and drives the upstream build with `CC=chibicc`. The cpython script is the only one that needs a workaround for compiler-vendor detection.

---

## 23.5 — Redeclaration of functions

> `git checkout 395308c77b94fc16b146c01cc1316b9a07635686` — *redefinition*

The commit message is a single word. The diff is twenty-six lines in `parse.c`'s `function` helper. The change closes a long-standing gap — the function-definition path used to silently accept multiple definitions of the same function and it shouldn't have.

Before this commit, `function` always allocated a fresh `Obj`:

```c
Obj *fn = new_gvar(get_ident(ty->name), ty);
fn->is_function = true;
fn->is_definition = !consume(&tok, tok, ";");
fn->is_static = attr->is_static || (attr->is_inline && !attr->is_extern);
fn->is_inline = attr->is_inline;
```

Two definitions of the same name would push two `Obj`s onto `globals`. The output assembler would have two `.globl foo` directives and two `foo:` labels — the GNU assembler would reject the second `foo:` with a hard error, but the diagnostic came from the assembler, not the compiler. From the user's point of view, chibicc's output was simply broken assembly.

The new path consults the existing global scope first:

```c
Obj *fn = find_func(name_str);
if (fn) {
  // Redeclaration
  if (!fn->is_function)
    error_tok(tok, "redeclared as a different kind of symbol");
  if (fn->is_definition && equal(tok, "{"))
    error_tok(tok, "redefinition of %s", name_str);
  if (!fn->is_static && attr->is_static)
    error_tok(tok, "static declaration follows a non-static declaration");
  fn->is_definition = fn->is_definition || equal(tok, "{");
} else {
  fn = new_gvar(name_str, ty);
  fn->is_function = true;
  fn->is_definition = equal(tok, "{");
  fn->is_static = attr->is_static || (attr->is_inline && !attr->is_extern);
  fn->is_inline = attr->is_inline;
}
```

`find_func` is a small new helper that walks to the global scope and asks the hashmap for an existing function with the given name. Three diagnostics are added:

- *redeclared as a different kind of symbol* — fires when the existing global is a variable, not a function.
- *redefinition of `<name>`* — fires when there's already a body and we're seeing another `{`.
- *static declaration follows a non-static declaration* — the linkage-mismatch diagnostic, matching gcc and clang.

If the redeclaration is benign (a forward declaration followed by a definition, or two forward declarations), the new declaration is folded into the existing `Obj` — `is_definition` is OR'd with the new commit. The `ty` from the new declarator is silently discarded; the existing `Obj`'s type wins. That's a simplification — a real compiler would check the two types for compatibility (parameters match, return type matches) — but it's enough to catch the more common errors.

Note what this commit does *not* do. Variables, tags, typedef names, labels, and struct members all still lack their corresponding redeclaration check. The standing notes from prior chapters list these as five separate errata candidates; this commit closes only the function half of the variable/function pair. The four other redeclaration gaps remain open.

**Where we are.** Function definitions are now properly checked for redeclaration. Two `void f(void) { return; }` blocks in the same translation unit produce a clean *redefinition of f* error rather than producing ambiguous assembly. Forward declarations followed by a definition still work. Variables with multiple definitions still aren't checked — that's the next half of the same fix and Rui doesn't reach it before the project ends. Errata candidates: closed one (function redefinition); four still remaining (variable, tag, typedef, label, struct-member redeclarations).

---

## 23.6 — `__attribute__((packed))` and `__attribute__((aligned))`

> `git checkout 44bea4c85a48d440bc0f704abe64eac80e9165dc` — *Add __attribute__((packed))*
> `git checkout b35d148a8d8f7d9237173c70f18cd42d20f299ff` — *Add __attribute__((aligned(N)) for struct declaration*

Two commits add real `__attribute__` parsing. Until now, `__attribute__(...)` was a macro stub that vanished during preprocessing — `chibicc.h` defines `__attribute__(x)` to nothing when `__GNUC__` isn't defined, and chibicc never defines `__GNUC__`, so user code that relied on the stub was silent. That works for the common cases where the attribute is documentation or a hint the compiler can ignore. But `packed` and `aligned` *change struct layout*, and ignoring them produces wrong code. These two commits make them real.

### `packed`: byte-aligned member packing

The first commit adds the keyword `__attribute__` to the tokenizer (so the preprocessor stops eating it) and a small parser that recognizes the single attribute `packed`:

```c
// attribute = ("__attribute__" "(" "(" "packed" ")" ")")?
static Token *attribute(Token *tok, Type *ty) {
  if (!equal(tok, "__attribute__"))
    return tok;
  tok = tok->next;
  tok = skip(tok, "(");
  tok = skip(tok, "(");
  tok = skip(tok, "packed");
  tok = skip(tok, ")");
  tok = skip(tok, ")");
  ty->is_packed = true;
  return tok;
}
```

The double parenthesis is the GCC convention — `__attribute__((name))`. The parser is called from `struct_union_decl` at two positions: before the tag name (`struct __attribute__((packed)) S { ... }`) and after the closing brace (`struct S { ... } __attribute__((packed))`). Both forms are accepted; both set `is_packed` on the struct's `Type`.

`Type` gains a `bool is_packed` field next to `is_flexible`. The struct-layout function honors it:

```c
} else {
  if (!ty->is_packed)
    bits = align_to(bits, mem->align * 8);
  mem->offset = bits / 8;
  bits += mem->ty->size * 8;
}

if (!ty->is_packed && ty->align < mem->align)
  ty->align = mem->align;
```

Two things change. The per-member alignment skip (`align_to(bits, mem->align * 8)`) is suppressed — without it, an `int` follows a `char` directly, with no padding. And the struct's overall alignment doesn't get bumped by member alignments — without that, a packed `struct { char a; int b; }` has alignment 1, not 4. Together these make `sizeof(packed{char;int;})` equal 5 and `_Alignof` equal 1. Without packing, the same struct is 8 bytes with alignment 4.

The bitfield path (the `if (mem->is_bitfield)` arm just above) is untouched — bitfield packing has its own rules that don't interact with `packed`. The implication is that packed structs with bitfield members may not produce the layouts a programmer expects. In practice gcc has its own subtleties here too; chibicc's silence on the corner case is consistent with the rest of the chapter's "ship the common path" approach.

The test exercises all four parser positions plus offsetof and `_Alignof`:

```c
ASSERT(5, ({ struct { char a; int b; } __attribute__((packed)) x; sizeof(x); }));
ASSERT(1, offsetof(struct __attribute__((packed)) { char a; int b; }, b));
ASSERT(1, _Alignof(struct __attribute__((packed)) { char a; int b[2]; }));
```

### `aligned(N)`: member-comma-list and per-attribute parsing

The second commit broadens the attribute parser to handle multiple attributes per `__attribute__` — like `__attribute__((aligned(8), packed))` — and multiple `__attribute__` clauses at the same site, like `__attribute__((aligned(8))) __attribute__((packed))`. The single-attribute helper becomes `attribute_list`:

```c
// attribute = ("__attribute__" "(" "(" "packed" ")" ")")*
static Token *attribute_list(Token *tok, Type *ty) {
  while (consume(&tok, tok, "__attribute__")) {
    tok = skip(tok, "(");
    tok = skip(tok, "(");

    bool first = true;
    while (!consume(&tok, tok, ")")) {
      if (!first)
        tok = skip(tok, ",");
      first = false;

      if (consume(&tok, tok, "packed")) {
        ty->is_packed = true;
        continue;
      }

      if (consume(&tok, tok, "aligned")) {
        tok = skip(tok, "(");
        ty->align = const_expr(&tok, tok);
        tok = skip(tok, ")");
        continue;
      }

      error_tok(tok, "unknown attribute");
    }
    tok = skip(tok, ")");
  }
  return tok;
}
```

Three nested loops handle the three nested syntactic levels: outer `while` for repeated `__attribute__((...))`, inner `while` for comma-separated attributes inside a single `((...))`, and the per-attribute switch on the attribute name. The `first` flag distinguishes "first attribute, no leading comma" from "subsequent attributes, must consume a comma." Unknown attributes raise an error rather than being silently ignored — a reversal of the old `__attribute__` macro-stub policy. With the parser real, anything inside `__attribute__((...))` that the parser doesn't recognize is now a hard error.

`aligned(N)` evaluates `N` as a constant expression and overwrites `ty->align`. The expression goes through `const_expr`, so `aligned(8+8)` is honored; the test exercises that. The interaction with `packed` is that `aligned` *raises* the alignment, while `packed` keeps the layout function from raising it via member alignments — they're orthogonal. A struct with both `aligned(8)` and `packed` has byte-packed members but is itself 8-byte-aligned; the test exercises this combination.

The `if (!ty->is_packed && ty->align < mem->align) ty->align = mem->align;` line from the previous commit means `packed` *prevents* the layout function from clobbering an `aligned`-supplied alignment with a smaller member alignment. Without `packed`, the layout function would set `ty->align = max(ty->align, mem->align)`, which is fine when the user hasn't specified alignment (member alignments win) but would override an `aligned(8)` request if some member happened to have alignment 16. In practice, the tests don't exercise that corner.

The test for `aligned` covers the single-attribute case, the comma-separated `packed` combination, the multi-`__attribute__` form, and a position case where the attribute follows the closing brace:

```c
ASSERT(8,  ({ struct __attribute__((aligned(8))) { int a; } x; _Alignof(x); }));
ASSERT(8,  ({ struct __attribute__((aligned(8), packed)) { char a; int b; } x; _Alignof(x); }));
ASSERT(8,  ({ struct __attribute__((aligned(8))) __attribute__((packed)) { char a; int b; } x; _Alignof(x); }));
ASSERT(16, ({ struct __attribute__((aligned(8+8))) { char a; int b; } x; _Alignof(x); }));
```

Note what this attribute parser does *not* cover. There's no support for `aligned(N)` on a *member* (only on the struct itself), no support for the `noreturn`, `unused`, `deprecated`, `format`, `weak`, or `visibility` attributes, and no support for `__attribute__` in any position other than struct-decl. The two parsed attributes are the two that change layout; the rest stay invisible to the compiler the same way they did before this commit.

**Where we are.** `__attribute__((packed))` and `__attribute__((aligned(N)))` are parsed and honored on struct declarations. The attribute parser handles comma-lists and multiple `__attribute__` clauses at the same position. Unknown attribute names are now hard errors (a change from the old macro-stub silent-pass-through). All other `__attribute__` uses still go through `chibicc.h`'s no-op macro — which means user code that was relying on the stub now has a problem only if it sits at a struct-declaration position with an unrecognized attribute name. The change is conservative enough to not break the existing third-party-build harness.

---

## 23.7 — The README

> `git checkout 982041fb1c78147951e73050a6c87059f92ea4e6` — *Update README*

The repo's README was a single-line placeholder for the entire history of the project until this commit. Rui replaces it with 209 lines that introduce chibicc to a first-time visitor: what it is, what it supports, what it doesn't, how to read the commit history, and a section called *Design principles* that retroactively explains a number of source patterns that have come up in earlier chapters.

The introduction is straightforward:

> chibicc is yet another small C compiler that implements most C11 features. Even though it still probably falls into the "toy compilers" category just like other small compilers do, chibicc can compile several real-world programs, including Git, SQLite, libpng and chibicc itself, without making modifications to the compiled programs.

The supported-feature list overlaps with what we've been tracking — the preprocessor, float/double/long-double, bit-fields, alloca, VLAs, compound literals, thread-local variables, atomic variables, common symbols, designated initializers, the L/u/U/u8 string-literal prefixes, and ABI-conformant struct-by-value calls. The unsupported list is short: complex numbers, K&R prototypes, GCC inline assembly, and digraphs/trigraphs. Optimization is also explicitly out of scope:

> There's no optimization pass. chibicc emits terrible code which is probably twice or more slower than GCC's output. I have a plan to add an optimization pass once the frontend is done.

That sentence has aged interestingly — the project ended without that optimization pass arriving. From the standpoint of the book, "the frontend is done" is the subject of the next paragraph; what comes after is left to a hypothetical phase that didn't happen.

The *Design principles* section is the part most worth quoting. Several of the patterns we've spent chapters explaining are stated outright here:

> chibicc doesn't try too hard to save memory. An entire input source file is read to memory first before the tokenizer kicks in, for example.

> Slow algorithms are fine if we know that n isn't too big. For example, we use a linked list as a set in the preprocessor, so the membership check takes O(n) where n is the size of the set. But that's fine because we know n is usually very small. And even if n can be very big, I stick with a simple slow algorithm until it is proved by benchmarks that that's a bottleneck.

> Each AST node type uses only a few members of the `Node` struct members. Other unused `Node` members are just a waste of memory at runtime. We could save memory using unions, but I decided to simply put everything in the same struct instead. I believe the inefficiency is negligible.

> chibicc allocates memory using `calloc` but never calls `free`. Allocated heap memory is not freed until the process exits. I'm sure that this memory management policy (or lack thereof) looks very odd, but it makes sense for short-lived programs such as compilers. DMD, a compiler for the D programming language, uses the same memory management scheme for the same reason, for example.

These are all observations the book has made along the way. The hideset is a bitset on `Token` because `Token` already pays for the membership; `Initializer` is a giant fan-out of fields because each kind uses only a few; the `Macro`-into-hashmap migration in §22.3 was driven by exactly the "until benchmarks prove a bottleneck" criterion (the macro-list scan *did* show up in a profile). What this commit does is name the design principles explicitly. They've been there all along, and now they're in writing.

A section on *Contributing* explains the no-pull-request policy: Rui rewrites history as needed, so pull requests get rebased onto a re-rolled commit chain by hand. A section on *About the Author* mentions the LLVM lld linker. A *References* list points at tcc, lcc, the Ghuloum incremental-construction paper, and Pike's five rules.

The README is dated September 30, 2020 — the project at this point has 22 of its eventual 23 chapters' worth of code, and Rui is treating the repo as nearly publishable. The very last commit (in §23.8) is two months later.

**Where we are.** The README is real. Anyone arriving at the GitHub repo cold gets a coherent introduction, a feature list that's accurate as of this commit, and a design-principles section that explains why the source code looks the way it does. The book uses that design-principles vocabulary (slow-but-simple, calloc-and-leak, fields-not-unions) as authoritative — these are Rui's own framings, not the book's interpretation.

---

## 23.8 — Address-of through `=` and `?:`

> `git checkout 90d1f7f199cc55b13c7fdb5839d1409806633fdb` — *Make struct member access to work with `=` and `?:`*

This is the project's last commit. Seven lines of codegen, three lines of test:

```c
ASSERT(2, ({ struct {int a;} x={1}, y={2}; (x=y).a; }));
ASSERT(1, ({ struct {int a;} x={1}, y={2}; (1?x:y).a; }));
ASSERT(2, ({ struct {int a;} x={1}, y={2}; (0?x:y).a; }));
```

The patterns being tested are GCC extensions — assignment of a struct value, used as if it were itself an lvalue with a `.a` selector applied to the result; and the conditional operator returning a struct value, again subscripted. Standard C doesn't allow either: the result of `=` on a struct is an rvalue, and `?:` between two struct lvalues isn't itself an lvalue.

The codegen change is in `gen_addr`, which exists to compute the *address* of an expression when one is needed (for assignment, for `&`, for member access on the left of `.`):

```c
case ND_ASSIGN:
case ND_COND:
  if (node->ty->kind == TY_STRUCT || node->ty->kind == TY_UNION) {
    gen_expr(node);
    return;
  }
  break;
```

The trick is that `gen_expr` on a struct-typed `ND_ASSIGN` already does the right thing — it evaluates the right-hand struct, copies it into the destination, and leaves a pointer to the destination in `%rax`. Same for `ND_COND` — the codegen for `?:` on a struct picks one of the two struct lvalues and leaves a pointer to it in `%rax`. So `gen_addr` for these cases just needs to forward to `gen_expr`; the result is already an address.

For non-struct `ND_ASSIGN` and `ND_COND`, this codepath isn't taken. The `break` falls through to the default error path (*not an lvalue*), preserving standard C's prohibition on `(x = 5) = 7` and similar.

Why does it work? Trace through `(x=y).a`. The parser builds an `ND_MEMBER` whose `lhs` is the `ND_ASSIGN` node. To codegen the member access, `gen_addr` runs on the `ND_ASSIGN` to get the address of the assignment's result. With this commit, that recursion calls `gen_expr` on the assignment, which performs the struct copy and leaves the destination address in `%rax`. The `.a` selector is then a fixed offset added to that address. So `(x=y).a` reads `x.a` — *after* `y` has been copied into `x`.

Same for `(1?x:y).a`. The `ND_COND` codegen for a struct picks `x` or `y` depending on the condition and leaves a pointer to the chosen struct in `%rax`. The `.a` selector reads the `a` field of whichever struct was chosen. With the test's `1?x:y`, `x.a` (which is 1) is read; with `0?x:y`, `y.a` (which is 2) is read.

The change is small enough that it's tempting to see it as cleanup rather than feature work. But it closes a real gap — gcc and clang both accept these patterns as extensions, and code in real codebases (not just the test) uses them. With this seven-line commit, chibicc's parser-and-codegen handling of struct lvalues becomes uniform across all three sources of struct-typed expressions: variable references, struct-returning function calls, and now assignments and conditional expressions.

**Where we are.** Struct lvalues from `=` and `?:` work. Member access through these expressions is now valid. The compiler is — as far as Rui's `main` branch is concerned — done.

---

## Recap

Ten commits. Two atomic primitives, one `_Atomic` qualifier with CAS-loop expansion, a stdatomic.h header, a cpython script, a function-redeclaration check, two struct-attribute parsers, a 209-line README, and a seven-line codegen patch. The chapter is mostly small commits — the §23.2 `_Atomic` walk is the only one that adds real structural complexity, and even that one funnels through the existing `to_assign` helper rather than introducing a new code generation pattern.

The chapter's ten-row summary, in `main` order:

| # | Hash | Subject | Section |
|---|---|---|---|
| 307 | `ca27455` | Add atomic_compare_exchange | §23.1 |
| 308 | `80ea9d4` | Add atomic_exchange | §23.1 |
| 309 | `d69a11d` | Add _Atomic and atomic ++, -- and op= operators | §23.2 |
| 310 | `0a5d08c` | Complete stdatomic.h | §23.3 |
| 311 | `2ed3fda` | Add test/thirdparty/cpython.sh | §23.4 |
| 312 | `395308c` | redefinition | §23.5 |
| 313 | `44bea4c` | Add `__attribute__((packed))` | §23.6 |
| 314 | `b35d148` | Add `__attribute__((aligned(N))` for struct declaration | §23.6 |
| 315 | `982041f` | Update README | §23.7 |
| 316 | `90d1f7f` | Make struct member access to work with `=` and `?:` | §23.8 |

Errata candidates added this chapter:

- The `Obj *atomic_addr` and `Node *atomic_expr` fields added to `Node` in §23.2 are never read or written. Dead fields in `chibicc.h`.

Errata candidates closed this chapter:

- Function redeclaration silent-acceptance (§23.5). Variable, tag, typedef, label, and struct-member redeclaration checks remain.

The psABI conformance count grows by one — the locked `cmpxchg` and `xchg` patterns plus the `__sync`-style atomic codegen are the standard psABI atomic forms. New count: twenty.

The canonicalization-at-parse-time count grows by one for §23.2's CAS-loop expansion (the parser turns `x += 5` on an `_Atomic int` into a fully-built `ND_STMT_EXPR` with a `do { ... } while (...)` loop and four anonymous locals — all at parse time, with no codegen-side awareness of "atomic op-assign" as a node kind). New count: twelve.

The pre-factor-before-feature count is unchanged at nine. None of this chapter's commits introduced a refactor before its dependent feature; the §23.6 `attribute` → `attribute_list` extension in commit 314 is the closest, but the first attribute commit (313) was already a feature (`packed`), so this is a feature-extending-a-feature rather than a refactor-before-feature.

---

## And the project

This is the last chapter. After commit 316, Rui's `main` branch goes silent — no more commits, no further changes. What follows is a brief survey of what chibicc, at its end state, is and isn't.

What it is. A complete C11 compiler — preprocessor, parser, type checker, and code generator — for x86-64 Linux. About 10,000 lines of C in seven source files plus a header and a hashmap. It compiles itself (as of Ch 17 §17.6, end-to-end). It compiles real software: the third-party harness builds git, libpng, sqlite, tinycc, and now cpython, all from upstream sources with at most a one-line `libtool` `sed` patch and (for cpython) a configure-script edit. The driver speaks gcc's command-line vocabulary closely enough to drop into existing build systems. The preprocessor handles `__VA_OPT__`, GCC-style variadic macros, `#pragma once`, `#include_next`, and include-guard optimization. The type system covers integer promotions, the usual arithmetic conversions, function-call ABI conformance for structs-by-value, the four char-literal prefixes, the four string-literal prefixes, real 80-bit `long double`, real bit-fields, real flexible array members, real designated initializers (with GNU array range designators), and now the C11 atomics. The codegen emits debuggable assembly with per-token line numbers, GDB-walkable stack frames, and conformant SystemV variadic argument layout including the register save area for `va_arg` against floats.

What it isn't. There's no optimization pass. The codegen is a tree walk that emits a register-poor, stack-heavy instruction stream with ample redundant loads and stores. Rui's README estimates the output is roughly 2× slower than gcc's; in practice on real workloads the multiplier varies, but the point stands — chibicc's output runs, but it doesn't run fast. There's no register allocator beyond "everything goes through `%rax` and gets spilled to the stack." There's no instruction selection beyond a single per-AST-node template. There's a single backend, x86-64; no ARM, no RISC-V, no x86-32. The platform is Linux only — the driver paths are Ubuntu-specific in places, and the runtime depends on glibc's `__tls_get_addr`. K&R-style function prototypes aren't supported. GCC inline assembly isn't supported. The complex types (`_Complex`) aren't supported. Digraphs and trigraphs are intentionally absent.

The redeclaration checks for variables, tags, typedef names, labels, and struct members are missing — only functions are checked (§23.5). The Makefile-escape function (`quote_makefile` from §22.4.6) is one-sided. The `is_compatible` array arm (§20) compares element types incorrectly for incomplete arrays. The default include paths are Linux/glibc-specific. Eleven errata candidates remain open at project end.

The pedagogical structure that Rui designed — one feature per commit, each commit independently readable — held up across 316 commits. A reader who picks any commit on `main` can checkout that hash, read the diff, run the tests at that revision, and understand a single feature in isolation. The book has tried to follow that structure, walking each commit at the granularity Rui chose and grouping into chapters where the natural section structure suggested it. Some chapters cover one feature in depth (Ch 17's preprocessor); others cover many small features at once (this chapter, and Ch 22). The granularity follows the source.

What Rui didn't do — the optimization pass, the register allocator, the second backend — is a chapter someone else could write. The codebase is small enough and clean enough to extend; the design-principles section of the README (§23.7) makes that more inviting than discouraging. But that's work after this book ends. What this book has covered is the journey from a single-number "language" (Ch 1's three-line `main`) to a self-hosting C11 compiler that builds Python (this chapter's §23.4). Three hundred and sixteen commits. Twenty-three chapters. One small C compiler.
