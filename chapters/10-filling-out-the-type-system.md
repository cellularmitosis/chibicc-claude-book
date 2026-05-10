# Chapter 10 — Filling out the type system

> Commits covered: `5831eda`, `43c2f08`, `9d48eef`, `a817b23`, `74e3acc`, `8c3503b`, `287906a`, `f46370e`, `a6b82da`, `67543ea`, `cb81a37`, `cfc4fa9`, `8b430a6`, `9e211cb`, `818352a`, `fdc80bc`, `44bba96`, `aa0accc`, `48ba265`, `736232f`. Twenty commits — the largest single chapter in the book — turning chibicc's three-type universe (`char`, `int`, `pointer`) into something a working C programmer would recognize.

Through Chapter 9 chibicc has known three integer types and one pointer type. `char` was 1 byte, `int` was 8, every pointer was 8, and arrays were the same shape as pointers when it mattered. Codegen took those sizes for granted: `mov`, `lea`, `add %rdi, %rax` — one register width, one set of instructions, no narrowing or widening anywhere.

That was always a placeholder. Real C programs declare `long` and `short` and `void` and `_Bool`. They write `(int) x` to convert. They put types behind `typedef` names. They write `static` on file-scope functions and they expect `int` to be 32 bits, not 64. Twenty commits in `main`'s middle stretch take chibicc from the placeholder to something close to right.

The chapter is long because the work is broad. New keywords; new `Type` constants; a `declspec` rewrite that handles `long int` and `int long` and `long long int` interchangeably; a parser hook for nested declarators like `int (*x)[3]`; a parser hook for `typedef`; a parser hook for `enum`; a `cast` AST node; a `cast_table` in codegen; the *usual arithmetic conversion*; argument and return-value conversion at function boundaries; 32-bit register variants. Each commit is small. Their interleaving is the chapter's main pedagogical challenge.

The fifteen sections (plus one concept interlude):

- **§10.1** — `int` becomes 32 bits (commit 56).
- **§10.2** — `long`, `short`, `void` (commits 57, 58, 61).
- **Concept interlude** — How to read a C type declaration.
- **§10.3** — Nested declarators (commit 59).
- **§10.4** — Function declarations (commit 60).
- **§10.5** — Complex type declarations (commits 62, 63).
- **§10.6** — `typedef` (commit 64).
- **§10.7** — `sizeof(typename)` (commit 65).
- **§10.8** — 32-bit register arithmetic (commit 66).
- **§10.9** — Casts (commit 67).
- **§10.10** — The usual arithmetic conversion (commit 68).
- **§10.11** — Function-call type conversions (commits 69, 70, 71).
- **§10.12** — `_Bool` (commit 72).
- **§10.13** — Character literals (commit 73).
- **§10.14** — `enum` (commit 74).
- **§10.15** — File-scope `static` functions (commit 75).

The dates of these commits do not match their position on `main`. Roughly seven of them (`a817b23`, `8c3503b`, `287906a`, `67543ea`, `cfc4fa9`, `aa0accc`, `48ba265`) are dated August 2019; the rest fall between March and September 2020. Notably the `int` size change (`5831eda`, dated 2020-09-06) sits in `main` order *before* a string of August-2019 commits, which means in chronological terms the August-2019 commits were originally written when `int` was still 8 bytes and were rewritten — or at least re-tested — once Rui changed the size. The chapter follows `main` order, which is the order the chapter mapping pins.

A note on the concept interlude. Rui's existing Japanese book has a dedicated chapter (#11) on reading C type declarations. The chapter mapping calls for an interlude in this position drawing on that material. Our interlude is shorter than a full chapter and sits between §10.2 and §10.3 — directly before the nested-declarator commit, which is where a reader first needs to read declarations like `char (*x)[3]` and not just `int x`.

---

## 10.1 — `int` becomes 32 bits

> `git checkout 5831edaab3eb6d56126c08f01f5639222602f7e5` — *Change size of int from 8 to 4*

The smallest commit of the chapter, and the one that pre-conditions all the rest. One number changes in `type.c`:

```c
-Type *ty_int = &(Type){TY_INT, 8, 8};
+Type *ty_int = &(Type){TY_INT, 4, 4};
```

`int` is now 4 bytes wide and 4-byte aligned. Everything chibicc does with `int` immediately gets narrower: a `sizeof(int)` returns 4, an `int x[4]` is 16 bytes on the stack, the offsets `assign_lvar_offsets` hands out tighten up, the `align_to` calls in `struct_decl` start producing different layouts. The test suite catches all of it, and the diff is full of test edits — `sizeof(x)` going from 8 to 4, struct sizes going from 16 to 8, the gap between `&y` and `&z` in `int x; int y; char z;` changing from 15 to 7.

But the change isn't only a constant. Until this commit, every value chibicc loaded or stored fit in a 64-bit register, full stop. With `int` at 4 bytes, that's no longer true. A value at an `int`-typed address is 4 bytes wide, and reading it with `mov (%rax), %rax` would pull 4 bytes of garbage along with the 4 bytes that matter. Storing back with `mov %rax, (%rdi)` would overwrite 4 bytes of whatever lives next to the variable.

So `load` and `store` grow a 4-byte branch:

```diff
   if (ty->size == 1)
     println("  movsbq (%%rax), %%rax");
+  else if (ty->size == 4)
+    println("  movsxd (%%rax), %%rax");
   else
     println("  mov (%%rax), %%rax");
```

```diff
   if (ty->size == 1)
     println("  mov %%al, (%%rdi)");
+  else if (ty->size == 4)
+    println("  mov %%eax, (%%rdi)");
   else
     println("  mov %%rax, (%%rdi)");
```

`movsxd` ("move sign-extending double to quad") reads 4 bytes from memory, sign-extends them into 64 bits, and writes the result into `%rax`. `mov %eax, (%rdi)` writes the low 4 bytes of `%rax` to memory. Both are exactly what the C standard's "load an `int`" and "store an `int`" should compile to on x86-64.

The commit also generalizes a small bit of the function prologue. Until now the parameter-saving loop hard-coded a two-way branch on size — 1 byte (use `argreg8`) versus everything else (use `argreg64`):

```c
for (Obj *var = fn->params; var; var = var->next) {
  if (var->ty->size == 1)
    println("  mov %s, %d(%%rbp)", argreg8[i++], var->offset);
  else
    println("  mov %s, %d(%%rbp)", argreg64[i++], var->offset);
}
```

Adding a 4-byte case to that branch would have made the loop start to look ugly. Rui factors it out into a helper, `store_gp`, that switches on size and picks the right register from the right table:

```c
static char *argreg32[] = {"%edi", "%esi", "%edx", "%ecx", "%r8d", "%r9d"};

static void store_gp(int r, int offset, int sz) {
  switch (sz) {
  case 1:
    println("  mov %s, %d(%%rbp)", argreg8[r], offset);
    return;
  case 4:
    println("  mov %s, %d(%%rbp)", argreg32[r], offset);
    return;
  case 8:
    println("  mov %s, %d(%%rbp)", argreg64[r], offset);
    return;
  }
  unreachable();
}
```

The prologue becomes a one-liner:

```c
int i = 0;
for (Obj *var = fn->params; var; var = var->next)
  store_gp(i++, var->offset, var->ty->size);
```

Two things are worth pausing on. First, the `unreachable()` macro is new. It's defined in `chibicc.h` and exists for exactly this purpose — to mark a switch arm that the surrounding logic guarantees is never taken:

```c
#define unreachable() \
  error("internal error at %s:%d", __FILE__, __LINE__)
```

Calling `unreachable()` aborts the compile with a file-and-line internal-error message. It documents *why* the function only handles three sizes (chibicc only has 1-, 4-, and 8-byte parameter types right now), and gives the future a noisy crash if a fourth size sneaks in without `store_gp` being updated. The `unreachable()` helper will get more callers as the type system grows.

Second, the new `argreg32` table foreshadows §10.8. The lower-32-bit System V argument registers are `%edi`, `%esi`, `%edx`, `%ecx`, `%r8d`, `%r9d` — the same six slots as `argreg64`, named by their 32-bit aliases. With `argreg32` declared, all the codegen paths that need to write a 32-bit argument to memory have what they need; the broader 32-bit register usage in `gen_expr` arrives ten commits later.

A subtlety: this commit changes the *size* but not the *codegen* of int arithmetic. After this commit, `int x = 1; int y = 2; x + y;` still computes through 64-bit registers — `add %rdi, %rax` adds full quadwords. The narrow load and narrow store mean the `int` operand is correctly read and written, but the arithmetic in between uses the high bits of `%rax`, where sign-extension from `movsxd` makes the answer agree with what a 32-bit `add` would have produced anyway. The test for that property is in `test/usualconv.c`, which won't land for another twelve commits — but the property already holds.

The chapter calls this kind of thing *pre-factor before feature*, a pattern named in §6.5 (`new_add` / `new_sub`) and used in §7.6 (`read_file`'s trailing-newline guarantee) and §8.3 (per-token line numbers). This is the fourth instance, and it's unusually long-running: the codegen catch-up that finally exploits the smaller-than-rax `int` doesn't fully arrive until commit 66, ten commits later. In the meantime, every commit between here and there can assume `sizeof(int) == 4`.

### Where we are

`int` has its standard width. Loads and stores narrow correctly. The parameter-saving prologue uses a size-dispatch helper. The arithmetic codegen still pretends the world is 64-bit, and will keep pretending until §10.8.

---

## 10.2 — `long`, `short`, `void`

> `git checkout 43c2f0829f7d4ec3b96132b9964a778ff816b2eb` — *Add long type*
> `git checkout 9d48eef58b964551350fe0c1f641a57f5da40529` — *Add short type*
> `git checkout 8c3503bb94bd6b2d57e1f979d9fc1d84383b2961` — *Add void type*

Three commits, the same shape repeated three times. Each one adds a new keyword, a new `TypeKind` enum entry, a new global `ty_X` constant with the right size and alignment, a new `is_keyword` line, a new `is_typename` line, a new `declspec` branch, a new `is_integer` line where applicable, and a test or two. Bundled as one section because once you've seen `long`, you've effectively seen `short` and `void`.

### `long` (commit 57)

`long` is the largest of the three. Eight bytes wide, eight-byte aligned, and a kind of "default" return type for anything `add_type` doesn't have a better answer for:

```c
Type *ty_long = &(Type){TY_LONG, 8, 8};
```

```diff
 typedef enum {
   TY_CHAR,
   TY_INT,
+  TY_LONG,
   TY_PTR,
   TY_FUNC,
   TY_ARRAY,
```

The parser's `declspec` grows a branch matching the existing `int` and `char` branches:

```c
if (equal(tok, "long")) {
  *rest = tok->next;
  return ty_long;
}
```

`is_typename` learns the new keyword. The tokenizer's `is_keyword` table picks up both `short` and `long` in one go, even though `short`'s parser support won't land until the next commit:

```diff
   static char *kw[] = {
     "return", "if", "else", "for", "while", "int", "sizeof", "char",
-    "struct", "union",
+    "struct", "union", "short", "long",
   };
```

Adding `short` to the lexer one commit early is a small inconsistency that doesn't break anything: until the next commit, `short` will tokenize as a keyword and `declspec` will fail to match it, producing a parse error that talks about an unexpected token rather than an unrecognized identifier. Probably faster than coming back to the lexer twice.

`long` is also the first commit where chibicc lifts the width of literal integer values to 64 bits. The token's `val` field, the AST's `val` field, and the parser's `new_num` parameter all change from `int` to `int64_t`:

```diff
 struct Token {
   ...
-  int val;        // If kind is TK_NUM, its value
+  int64_t val;    // If kind is TK_NUM, its value
   ...
 };
```

```diff
-static Node *new_num(int val, Token *tok) {
+static Node *new_num(int64_t val, Token *tok) {
```

The codegen emits `mov $%ld` instead of `mov $%d`. None of chibicc's existing programs use 64-bit literals, but since pointer arithmetic, `sizeof`, and `long`-typed expressions are about to start producing them, the widening is preventive.

`add_type` makes `long` the type of function calls and a few other "we don't know what else this would be" cases:

```diff
   case ND_NE:
   case ND_LT:
   case ND_LE:
   case ND_NUM:
   case ND_FUNCALL:
-    node->ty = ty_int;
+    node->ty = ty_long;
     return;
```

That's a coarse rule and several of these arms are revisited in §10.10 (`ND_EQ`/`ND_NE`/`ND_LT`/`ND_LE` get split out and given `ty_int`, `ND_NUM` gets an `int`-or-`long` decision based on the value). For now, `long` is doubling as both the new explicit type and the catch-all category for any expression whose type chibicc doesn't bother to compute.

### `short` (commit 58)

Two bytes wide, two-byte aligned. The `TY_SHORT` enum entry, the `ty_short` global, the `declspec` branch, the `is_integer` extension. Codegen grows a 2-byte branch in `load`, `store`, and `store_gp`, mirroring the 4-byte branches that `int` introduced two commits ago:

```c
static char *argreg16[] = {"%di", "%si", "%dx", "%cx", "%r8w", "%r9w"};
```

```diff
   if (ty->size == 1)
     println("  movsbq (%%rax), %%rax");
+  else if (ty->size == 2)
+    println("  movswq (%%rax), %%rax");
   else if (ty->size == 4)
     println("  movsxd (%%rax), %%rax");
```

`movswq` is "move sign-extending word to quad" — read 2 bytes, sign-extend to 8, deposit in `%rax`. Symmetric with `movsbq` for `char` and `movsxd` for `int`.

After this commit, `argreg8`, `argreg16`, `argreg32`, `argreg64` exist as a complete set, and the eight/sixteen/thirty-two/sixty-four split foreshadowed in §7.2 is now the actual structure of the parameter-passing code. (At the time of §7.2, only `argreg8` and `argreg64` existed; we predicted an `argreg32` would arrive when `int` got smaller. It did, in commit 56. The `argreg16` for `short` closes the loop.)

### `void` (commit 61)

Set aside that `void` is one byte wide and one-byte aligned (`Type *ty_void = &(Type){TY_VOID, 1, 1};`). That field setting is meaningless because `sizeof(void)` is a compile error in standard C, and chibicc's parser refuses `void` variables outright. The `1, 1` is just to make sure `void` doesn't accidentally crash something that asks for its size. The commit adds an explicit error in `declaration` for `void`-typed variables:

```c
Type *ty = declarator(&tok, tok, basety);
if (ty->kind == TY_VOID)
  error_tok(tok, "variable declared void");
```

And one in `add_type` for void-pointer dereferences:

```c
case ND_DEREF:
  if (!node->lhs->ty->base)
    error_tok(node->tok, "invalid pointer dereference");
  if (node->lhs->ty->base->kind == TY_VOID)
    error_tok(node->tok, "dereferencing a void pointer");
  node->ty = node->lhs->ty->base;
  return;
```

The void-pointer rule is what makes `void *` useful: you can declare it, you can take addresses with it, you can cast it back to a usable pointer type, but you can't dereference it directly. The test for the commit is one bracketed line in `test/variable.c`:

```c
{ void *x; }
```

A declaration that exists only to confirm `void *` parses and doesn't trip the "variable declared void" error.

The `void` commit also restructures `is_typename`. After three additions it had become a long disjunctive `equal(tok, ...)` chain; Rui replaces it with a small static table and a loop:

```diff
 static bool is_typename(Token *tok) {
-  return equal(tok, "char") || equal(tok, "short") || equal(tok, "int") ||
-         equal(tok, "long") || equal(tok, "struct") || equal(tok, "union");
+  static char *kw[] = {
+    "void", "char", "short", "int", "long", "struct", "union",
+  };
+
+  for (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)
+    if (equal(tok, kw[i]))
+      return true;
+  return false;
 }
```

The change is mechanical, but it pays off: every subsequent commit in this chapter that adds a new type-keyword adds it as one entry to `kw[]`, instead of as one more `||`-clause to a growing chain. `typedef`, `_Bool`, `enum`, `static` will all walk in this way. (Eventually `is_typename` grows past pure keyword matching into a symbol-table lookup — that's commit 64.)

### Where we are

`long`, `short`, and `void` exist. The four-way size split (1, 2, 4, 8) is fully covered by `load`, `store`, and `store_gp`. The parameter-passing tables are complete. The `is_typename` helper has been refactored into a table-and-loop form that the rest of the chapter will extend. `void` exists as a non-storable type that can only appear behind a pointer or as a return type; the parser blocks `void x;` and the type-checker blocks `*p` for `void *p`.

What's still wrong: `int x; int y;` at this point compiles to add operands using `add %rdi, %rax`, even though both operands are 4 bytes. The cleanup is in §10.8.

---

## Concept interlude — How to read a C type declaration

C's declaration syntax is famous for being hard to read. `int (*x)(int, int)` and `int *x[10]` and `int (*x)[10]` all *look* alike, and they all mean different things, and they all use the same five symbols (`*`, `[`, `]`, `(`, `)`) in slightly different positions. The hard part isn't the symbols, it's that the symbols don't compose left-to-right. They compose by following a path through the declaration that goes outward from the identifier.

The mnemonic for this is sometimes called *the spiral rule* or *the right-to-left-with-precedence rule* or *declaration follows use*. They're all the same observation: a C declaration is a copy of the *expression that uses the variable*, with the type written in front of it. If you can write the expression, you can write the declaration.

Take `int *x[10]`. The expression that uses `x` is `*x[10]` — index `x` at position 10 (or thereabouts), then dereference. So `x` is a thing you can index, and the result of that indexing is a thing you can dereference. Indexing means it's an array. Dereferencing means the array's elements are pointers. Working out from the identifier:

- `x` — an identifier
- `x[10]` — an array of 10 of them
- `*x[10]` — and each one is a pointer
- `int *x[10]` — a pointer to `int`

So `x` is an array of 10 pointers to `int`.

Now `int (*x)[10]`. The parentheses around `*x` change the precedence: the `*` binds before the `[10]`. The expression that uses `x` is `(*x)[10]` — dereference, then index. Working out:

- `x` — an identifier
- `*x` — dereference it (so `x` is a pointer)
- `(*x)[10]` — index the result (so what `x` points to is an array)
- `int (*x)[10]` — and each element is `int`

So `x` is a pointer to an array of 10 `int`s. Different from the previous declaration in every way that matters; the pointer is one level up, the array is one level down, and at the leaves there are integers, not pointers.

The third member of this family is `int (*x)(int, int)`. The expression is `(*x)(int, int)` — dereference, then call with two integer arguments. So `x` is a pointer to a function. Working out:

- `x` — an identifier
- `*x` — dereference it (so `x` is a pointer)
- `(*x)(int, int)` — call with two `int`s (so what `x` points to is a function)
- `int (*x)(int, int)` — and the function returns `int`

So `x` is a pointer to a function of two `int`s returning `int`. A function pointer.

The general procedure: *find the identifier, then read outward, applying operators in the order that an expression would*. Postfix operators (`[]` and `()`) bind tighter than the prefix `*`, and parentheses group things to override the default. When you reach the outermost layer, the leftmost type-specifier (`int`, `char`, etc.) tells you what the leaves are.

The classic Unix utility `cdecl` translates declarations to English using exactly this procedure, and reading its output is a perfectly reasonable way to learn the rules:

```
$ cdecl
> explain int (*x)(int, int)
declare x as pointer to function (int, int) returning int
> explain char *(*foo[5])(int)
declare foo as array 5 of pointer to function (int) returning pointer to char
```

For chibicc, the rule simplifies because the parser builds the type *as it walks the declarator*, returning the final `Type *` once it reaches the closing edge. The walking is recursive: the declarator `(*x)[10]` is "a `*` applied to whatever the parenthesized declarator parses to, with a `[10]` suffix on the result." That recursion is exactly the spiral, expressed as code. The next section is where the recursion appears in the parser.

A short table of the worst offenders, with parses:

| Declaration | Reading |
|---|---|
| `int x` | `x` is an `int`. |
| `int *x` | `x` is a pointer to `int`. |
| `int x[10]` | `x` is an array of 10 `int`s. |
| `int *x[10]` | `x` is an array of 10 pointers to `int`. |
| `int (*x)[10]` | `x` is a pointer to an array of 10 `int`s. |
| `int (*x)(int)` | `x` is a pointer to a function of `int` returning `int`. |
| `int *(*x)(int)` | `x` is a pointer to a function of `int` returning a pointer to `int`. |
| `int (*x[5])(int)` | `x` is an array of 5 pointers to functions of `int` returning `int`. |
| `int *(*(*x)(int))(char)` | `x` is a pointer to a function of `int` returning a pointer to a function of `char` returning a pointer to `int`. |

The last one is where most readers reach for `cdecl`.

---

## 10.3 — Nested declarators

> `git checkout a817b23da3c6f39f22bc57c0a53169978d97d7fa` — *Add nested type declarators*

Eleven lines of parser change support every declaration form in the table above except the function-pointer rows. The key insight is recursive descent: when a declarator opens with `(`, parse a declarator inside it, then keep going.

Through commit 58, `declarator` looked like:

```c
// declarator = "*"* ident type-suffix
static Type *declarator(Token **rest, Token *tok, Type *ty) {
  while (consume(&tok, tok, "*"))
    ty = pointer_to(ty);

  if (tok->kind != TK_IDENT)
    error_tok(tok, "expected a variable name");
  ty = type_suffix(rest, tok->next, ty);
  ty->name = tok;
  return ty;
}
```

A pile of `*`s, then an identifier, then a `type-suffix` (the optional array-size and function-parameters part). That handles `int x`, `int *x`, `int x[10]`, `int *x[10]`, `int x(int, int)` — but not anything with a parenthesized inner declarator.

The new version inserts one `if` arm:

```diff
 // declarator = "*"* ("(" ident ")" | "(" declarator ")" | ident) type-suffix
 static Type *declarator(Token **rest, Token *tok, Type *ty) {
   while (consume(&tok, tok, "*"))
     ty = pointer_to(ty);

+  if (equal(tok, "(")) {
+    Token *start = tok;
+    Type dummy = {};
+    declarator(&tok, start->next, &dummy);
+    tok = skip(tok, ")");
+    ty = type_suffix(rest, tok, ty);
+    return declarator(&tok, start->next, ty);
+  }
+
   if (tok->kind != TK_IDENT)
     error_tok(tok, "expected a variable name");
   ty = type_suffix(rest, tok->next, ty);
   ty->name = tok;
   return ty;
 }
```

This is the recursion that makes the spiral rule work. Read it carefully because the structure is unusual: the parenthesized inner declarator is parsed *twice*, with a dummy type the first pass and the real type the second pass.

To see why, take `char (*x)[3]`. The base type passed in is `char`. Skip past the `(` and we're at `*x)[3]`. We *can't* directly call `declarator(... ty)` here, because that would build "pointer to char" first, then apply `[3]` to it — producing pointer-to-char-array, which is wrong. We want array-of-3-char first (apply the suffix that's *outside* the parens), then pointer (apply what's *inside*).

So the parser does it in two passes. Pass one: walk the inside of the parens with a dummy `Type` just to get the `tok` pointer past it. After pass one, `tok` points at `)`. Skip the `)`, run `type_suffix` (which sees `[3]` and wraps the base type into `array of 3 char`), then re-walk the inside of the parens with that array type — building the `*` to produce "pointer to array of 3 char". Done.

The double-walk is unusual but it's the smallest code change that gets the spiral right. An alternative would be to change `type_suffix` into something that returns a *function* (in the sense of "an action to apply later"), but C doesn't compose closures cleanly enough to make that idiomatic. Two passes with a dummy type is the chibicc-style answer.

The tests cover the main cases:

```c
ASSERT(24, ({ char *x[3]; sizeof(x); }));
ASSERT(8, ({ char (*x)[3]; sizeof(x); }));
ASSERT(1, ({ char (x); sizeof(x); }));
ASSERT(3, ({ char (x)[3]; sizeof(x); }));
ASSERT(12, ({ char (x[3])[4]; sizeof(x); }));
ASSERT(4, ({ char (x[3])[4]; sizeof(x[0]); }));
ASSERT(3, ({ char *x[3]; char y; x[0]=&y; y=3; x[0][0]; }));
ASSERT(4, ({ char x[3]; char (*y)[3]=x; y[0][0]=4; y[0][0]; }));
```

`char *x[3]` is an array of 3 pointers (24 bytes — three 8-byte pointers). `char (*x)[3]` is a pointer to an array of 3 chars (8 bytes — one pointer). The first reading wraps pointer outside array; the second wraps array inside pointer. The parser disagrees with itself by exactly the amount the parentheses ask for.

`char (x[3])[4]` is the spiral one: `x` is an identifier, `x[3]` is an array of 3, `(x[3])[4]` is an array of 3 of arrays of 4 — `char[3][4]` total, 12 bytes. Reading the outer `[4]` first (because of the parens-then-suffix pattern) and the inner `[3]` second (because that's what's *inside* the parens) gets the same answer as reading `char x[3][4]` straight.

A note on what's still missing. This commit doesn't handle function-pointer declarators like `int (*f)(int, int)`. The parenthesized declarator in this commit handles "pointer to (something)" and "array of (something)", but not "pointer to function (something)" because `type_suffix` only recognizes `[`, not `(`. The complete handling — including function pointers — arrives in commit 62 (§10.5), which rewrites `declspec` to handle `int long` and friends and tightens `declarator`'s suffix logic. For now, function-pointer declarations work only if you write them with a non-parenthesized identifier, which excludes the C-canonical form.

### Where we are

Pointers can be wrapped around arrays, arrays around arrays, and the parentheses around parts of a declarator are honored. The double-walk-with-dummy-type trick will be reused two commits from now (in `abstract_declarator`, the version of `declarator` that takes no name) and is the chapter's main parser-side recursion idiom.

---

## 10.4 — Function declarations

> `git checkout 74e3acc296d90d6d16ae70803196e967564fb16a` — *Add function declaration*

Eight lines, distributed across `chibicc.h`, `parse.c`, and `codegen.c`. The change splits the existing notion of "function" into two: a *function declaration* (`int foo(int, int);` — a name and a type, no body) and a *function definition* (the same, with a body).

Until now, `function()` in `parse.c` always read a body after parsing the prototype. The change introduces an `is_definition` flag on `Obj`:

```diff
 // Global variable or function
 bool is_function;
+bool is_definition;
```

And a small early-return in `function()`:

```diff
 Obj *fn = new_gvar(get_ident(ty->name), ty);
 fn->is_function = true;
+fn->is_definition = !consume(&tok, tok, ";");
+
+if (!fn->is_definition)
+  return tok;

 locals = NULL;
 enter_scope();
```

The `consume` on `";"` is a semicolon-or-not check: if the prototype is followed by `;`, it's a declaration and we stop there. If not, the next token must be `{`, which means it's a definition and we proceed to parse the body as before.

Codegen has to skip declarations:

```diff
 for (Obj *fn = prog; fn; fn = fn->next) {
-  if (!fn->is_function)
+  if (!fn->is_function || !fn->is_definition)
     continue;
```

This is what makes `printf();` valid at the top of `test/test.h`:

```diff
 #define ASSERT(x, y) assert(x, y, #y)
+
+int printf();
```

It also makes the test harness `assert()` declarable without being defined in the same file (it's defined later in `test/common`). Until this commit, the test files leaned on the host C compiler to swallow undeclared `printf` and `assert` calls — chibicc would parse them, leaving the linker to pick up the implementations at link time, but the type information would be missing. With function declarations, those identifiers can be introduced with their types and chibicc has something to type-check against.

Two pieces don't land here. First, the parser doesn't yet *check* that calls match declarations — that arrives in §10.11. Second, the parser doesn't yet *use* the declared parameter types to convert arguments — also §10.11. For now, a forward declaration is little more than a placeholder; its value is mostly that it parses without erroring and that it gives the parser an `Obj` to point function calls at.

### Where we are

`int foo(int, int);` is valid syntax. The declaration creates an `Obj` with `is_function == true` and `is_definition == false`; codegen ignores it; the parser will be able to look up its type when commit 71 starts converting arguments. The split between declaration and definition is the precondition for everything that follows about function-call type-checking.

---

## 10.5 — Complex type declarations

> `git checkout 287906abb85081b961e118bb80b30decb93fba6f` — *Handle complex type declarations correctly*
> `git checkout f46370ef98adec5d3a840d69a6b34a03d80b0699` — *Add `long long` as an alias for `long`*

Two commits, the first of which is a substantial rework of `declspec`. The C standard says that the order of type-specifier keywords doesn't matter: `int long static const` and `static const long int` declare the same thing, and so does `long`, and so does `long int`, and so does `int long`. They're all aliases for the same type.

Through commit 61, `declspec` was a series of `equal(tok, "...")` arms that each consumed one keyword and returned. That's correct for one-keyword forms (`int`, `char`, `short`, `long`, `void`) but wrong for any combination — `long int x` would consume the `long` and return `ty_long`, leaving `int x` on the input, which `declarator` would then misparse as an `int` variable named `x`. The test suite gets away with not noticing because no test up to this point uses multi-keyword type specifiers.

Rui rewrites `declspec` to *count* keyword occurrences and look up the resulting tuple in a switch:

```c
static Type *declspec(Token **rest, Token *tok) {
  enum {
    VOID  = 1 << 0,
    CHAR  = 1 << 2,
    SHORT = 1 << 4,
    INT   = 1 << 6,
    LONG  = 1 << 8,
    OTHER = 1 << 10,
  };

  Type *ty = ty_int;
  int counter = 0;

  while (is_typename(tok)) {
    if (equal(tok, "struct") || equal(tok, "union")) {
      if (equal(tok, "struct"))
        ty = struct_decl(&tok, tok->next);
      else
        ty = union_decl(&tok, tok->next);
      counter += OTHER;
      continue;
    }

    if (equal(tok, "void"))       counter += VOID;
    else if (equal(tok, "char"))  counter += CHAR;
    else if (equal(tok, "short")) counter += SHORT;
    else if (equal(tok, "int"))   counter += INT;
    else if (equal(tok, "long"))  counter += LONG;
    else unreachable();

    switch (counter) {
    case VOID:                ty = ty_void;  break;
    case CHAR:                ty = ty_char;  break;
    case SHORT:
    case SHORT + INT:         ty = ty_short; break;
    case INT:                 ty = ty_int;   break;
    case LONG:
    case LONG + INT:          ty = ty_long;  break;
    default:
      error_tok(tok, "invalid type");
    }

    tok = tok->next;
  }

  *rest = tok;
  return ty;
}
```

The trick is the bit-field encoding. Each keyword gets two bits of counter (bits 0–1 for `VOID`, bits 2–3 for `CHAR`, etc.), so `LONG + INT` adds disjoint bits and produces a unique sum. The switch arms list every legal sum: `VOID` alone, `CHAR` alone, `SHORT` or `SHORT+INT`, `INT` alone, `LONG` or `LONG+INT`. Anything else — `LONG+CHAR`, `INT+CHAR`, two `SHORT`s — falls into the `default` arm and produces an error. Two bits per keyword leaves room for the second occurrence (`LONG+LONG` overflows into the third bit-pair, which the switch rejects as invalid for now and which the *next* commit explicitly accepts).

The Rui-style commentary is unusually expansive here — Rui is taking the time to explain what's going on:

> The order of typenames in a type-specifier doesn't matter. For example, `int long static` means the same as `static long int`. That can also be written as `static long` because you can omit `int` if `long` or `short` are specified. However, something like `char int` is not a valid type specifier. We have to accept only a limited combinations of the typenames.

> In this function, we count the number of occurrences of each typename while keeping the "current" type object that the typenames up until that point represent.

The `OTHER` counter — an artifact of the bit-field — gets added when a `struct` or `union` is found, and the bookkeeping ensures `struct` and a built-in type don't end up in the same `declspec`. The `is_typename` forward declaration at the top of the file is new because the new `declspec` calls it, and the existing `is_typename` is defined later.

A new test file lands:

```c
// test/decl.c
ASSERT(1, ({ char x; sizeof(x); }));
ASSERT(2, ({ short int x; sizeof(x); }));
ASSERT(2, ({ int short x; sizeof(x); }));
ASSERT(4, ({ int x; sizeof(x); }));
ASSERT(8, ({ long int x; sizeof(x); }));
ASSERT(8, ({ int long x; sizeof(x); }));
```

Order-insensitivity. `short int` and `int short` and `long int` and `int long` all parse and produce the right size.

### `long long` (commit 63)

Two new switch arms accept `LONG + LONG` and `LONG + LONG + INT`:

```diff
 case LONG:
 case LONG + INT:
+case LONG + LONG:
+case LONG + LONG + INT:
   ty = ty_long;
   break;
```

`long long` and `long long int` both alias `long`. That's not quite right by the standard — `long long` is supposed to be a distinct type guaranteed at least 64 bits — but on x86-64 Linux, `long` is already 64 bits and `long long` is also 64 bits, so the alias works. The test:

```c
ASSERT(8, ({ long long x; sizeof(x); }));
```

The bit-field encoding (two bits per keyword) is exactly large enough to distinguish `LONG+LONG` (which overflows to bit 9 set, bit 8 clear) from `INT+INT` (rejected) or `LONG+INT` (accepted, different type). If `long long long` were valid, the encoding would have to widen to three bits per keyword, but standard C tops out at `long long`.

### Where we are

The parser handles every legal combination of type specifier keywords. `long int`, `int long`, `long long`, `long long int` all work. The `declspec` function is now a small state machine instead of a chain of one-keyword branches. Any new built-in keyword can be added by allocating two more bits and adding switch arms for its legal combinations. The next two type-keyword commits — `_Bool` (§10.12) and `static` as a storage-class (§10.15) — both use this hook.

---

## 10.6 — `typedef`

> `git checkout a6b82da1ae9eefa44dada0baa885c283823ad59a` — *Add typedef*

`typedef` introduces a new name for an existing type. The implementation is the chapter's first interesting symbol-table extension, and it forces a small structural change in how the parser knows whether a token starts a type.

Until now, `is_typename` answered the question "is this a keyword that starts a type" by comparing the token against a fixed list. With typedef, the answer is "is this a keyword *or* is this an identifier that's been previously typedef'd in scope," and the second clause is a symbol-table lookup.

### Same scope, two roles

In §8.1 we built `Scope` as a chain with one variable list per frame; in §9.4 we added a parallel `tags` list for struct tags. `VarScope` records "this name is a variable; here's its `Obj`." With typedef, that record needs a second possible role: "this name is a type alias; here's its `Type`."

The C standard says typedef names live in the same namespace as ordinary identifiers — variables, function names, enum constants. That means `typedef int t; int t = 3;` is *valid* C: the typedef name `t` and the `int` variable named `t` shadow each other, with the variable winning in the inner scope (because the variable's declaration redefines the binding). chibicc gets this right by sharing a single `VarScope` slot:

```diff
 typedef struct VarScope VarScope;
 struct VarScope {
   VarScope *next;
   char *name;
   Obj *var;
+  Type *type_def;
 };
```

A `VarScope` entry now has three possible shapes: variable (`var` set, `type_def` null), typedef (`var` null, `type_def` set), or — much later, when enum constants land — enum constant. Lookup walks the chain and returns the entry; the caller decides which field to read.

`find_var` changes return type from `Obj *` to `VarScope *` and stops dereferencing the inner `var`:

```diff
-static Obj *find_var(Token *tok) {
+static VarScope *find_var(Token *tok) {
   for (Scope *sc = scope; sc; sc = sc->next)
     for (VarScope *sc2 = sc->vars; sc2; sc2 = sc2->next)
       if (equal(tok, sc2->name))
-        return sc2->var;
+        return sc2;
   return NULL;
 }
```

Callers are correspondingly updated. `primary` now distinguishes the variable case from the not-found-or-typedef case:

```c
VarScope *sc = find_var(tok);
if (!sc || !sc->var)
  error_tok(tok, "undefined variable");
*rest = tok->next;
return new_var_node(sc->var, tok);
```

### Storage-class specifiers

`typedef` is a *storage-class specifier* in C grammar — it appears in the same position as `static`, `extern`, `auto`, `register` would, mixed in with type-specifier keywords. It can come before or after `int`, before or after `*`, anywhere in the `declspec` part of a declaration. To express that, Rui passes a new optional argument through `declspec`:

```c
typedef struct {
  bool is_typedef;
} VarAttr;

static Type *declspec(Token **rest, Token *tok, VarAttr *attr);
```

Most call sites pass `NULL` (struct member declarations, function parameter declarations — places where a storage-class specifier doesn't make sense). The two top-level call sites — `compound_stmt` and `parse` — pass a real `VarAttr` and read out `attr.is_typedef` after `declspec` returns:

```c
while (!equal(tok, "}")) {
  if (is_typename(tok)) {
    VarAttr attr = {};
    Type *basety = declspec(&tok, tok, &attr);

    if (attr.is_typedef) {
      tok = parse_typedef(tok, basety);
      continue;
    }

    cur = cur->next = declaration(&tok, tok, basety);
  } else {
    cur = cur->next = stmt(&tok, tok);
  }
  add_type(cur);
}
```

The `if (attr.is_typedef)` branch routes to `parse_typedef`, which reads one or more declarators and pushes each as a typedef name:

```c
static Token *parse_typedef(Token *tok, Type *basety) {
  bool first = true;

  while (!consume(&tok, tok, ";")) {
    if (!first)
      tok = skip(tok, ",");
    first = false;

    Type *ty = declarator(&tok, tok, basety);
    push_scope(get_ident(ty->name))->type_def = ty;
  }
  return tok;
}
```

`push_scope` was previously reading the variable into the new entry; that's now split. `push_scope` returns the `VarScope *`, and the caller fills in either `var` or `type_def`. `new_var` is updated:

```diff
-static VarScope *push_scope(char *name, Obj *var) {
+static VarScope *push_scope(char *name) {
   VarScope *sc = calloc(1, sizeof(VarScope));
   sc->name = name;
-  sc->var = var;
   sc->next = scope->vars;
   scope->vars = sc;
   return sc;
 }
```

```diff
 static Obj *new_var(char *name, Type *ty) {
   ...
-  push_scope(name, var);
+  push_scope(name)->var = var;
   return var;
 }
```

### `is_typename` becomes a symbol-table lookup

Inside `declspec`, when we see an identifier (not a keyword), we have to ask: is this a typedef name? `find_typedef` is the helper:

```c
static Type *find_typedef(Token *tok) {
  if (tok->kind == TK_IDENT) {
    VarScope *sc = find_var(tok);
    if (sc)
      return sc->type_def;
  }
  return NULL;
}
```

And `is_typename` extends to:

```diff
 static bool is_typename(Token *tok) {
   static char *kw[] = {
     "void", "char", "short", "int", "long", "struct", "union",
+    "typedef",
   };

   for (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)
     if (equal(tok, kw[i]))
       return true;
-  return false;
+  return find_typedef(tok);
 }
```

The "return `find_typedef(tok)`" line is the structural shift. Through commit 63, `is_typename` was a pure syntactic predicate — it looked at the token text and said yes or no. Now, whether a token starts a type *depends on context*: the same identifier can be a type name in one scope and a variable name in another. Every place the parser asks "is this a declaration or a statement," it now asks the symbol table.

The `declspec` body grows a typedef-handling arm right at the top:

```c
while (is_typename(tok)) {
  if (equal(tok, "typedef")) {
    if (!attr)
      error_tok(tok, "storage class specifier is not allowed in this context");
    attr->is_typedef = true;
    tok = tok->next;
    continue;
  }

  Type *ty2 = find_typedef(tok);
  if (equal(tok, "struct") || equal(tok, "union") || ty2) {
    if (counter)
      break;
    if (equal(tok, "struct")) {
      ty = struct_decl(&tok, tok->next);
    } else if (equal(tok, "union")) {
      ty = union_decl(&tok, tok->next);
    } else {
      ty = ty2;
      tok = tok->next;
    }
    counter += OTHER;
    continue;
  }
  ...
}
```

A typedef name is treated like a `struct` or `union`: it's a "user-defined type," it consumes the entire `declspec` slot (the `if (counter) break` early-out makes sure we don't try to combine `MyInt int`), and it bumps the `OTHER` counter so other type-specifier keywords get rejected.

### Tests

`test/typedef.c` is new:

```c
typedef int MyInt, MyInt2[4];
typedef int;

int main() {
  ASSERT(1, ({ typedef int t; t x=1; x; }));
  ASSERT(1, ({ typedef struct {int a;} t; t x; x.a=1; x.a; }));
  ASSERT(1, ({ typedef int t; t t=1; t; }));
  ASSERT(2, ({ typedef struct {int a;} t; { typedef int t; } t x; x.a=2; x.a; }));
  ASSERT(4, ({ typedef t; t x; sizeof(x); }));
  ASSERT(3, ({ MyInt x=3; x; }));
  ASSERT(16, ({ MyInt2 x; sizeof(x); }));

  printf("OK\n");
  return 0;
}
```

The third test (`typedef int t; t t=1; t;`) is the C-grammar curiosity: `t` as a typedef and `t` as a variable name both exist, the variable shadows the typedef in its own scope, and reading `t` resolves to the variable. The fourth test is the block-scope version: an inner-scope typedef (`typedef int t;`) shadows an outer-scope typedef-of-struct, and the shadowing disappears at the closing brace, restoring the outer struct typedef just in time for `t x; x.a=2;`.

The two top-of-file declarations are also intentional. `typedef int MyInt, MyInt2[4];` shows that a single `typedef` can introduce multiple aliases (separated by commas), each with its own declarator on top of the same base type. `MyInt` is `int`, `MyInt2` is `int[4]`. And `typedef int;` — a typedef with no declarator at all — is allowed, vacuously, by the C grammar; it does nothing but parses cleanly.

`test/common` and `test/test.h` files come along with `int printf();` style declarations, now that those work.

### Where we are

`typedef` works at file scope and inside blocks. Typedef names share the variable namespace, with shadowing handled by the existing scope chain. `is_typename` is now a context-sensitive predicate that consults the symbol table. The `VarScope` entry has gained a third field, and `VarAttr` is the new vehicle for storage-class specifiers; `static` will land in §10.15 by walking through the same channel.

This is the chapter's biggest single structural change after `declspec`'s rewrite. Everything that follows can lean on it.

---

## 10.7 — `sizeof(typename)`

> `git checkout 67543ea113c5cc2b15881e2bbb85ffd44feaef1f` — *Make sizeof to accept not only an expression but also a typename*

C's `sizeof` operator takes either an expression or a parenthesized type name: `sizeof x` and `sizeof(int)` are both legal. Until now, chibicc supported only the expression form — `sizeof(int)` would fail because `int` isn't an expression. The fix is to introduce a *type-name* grammar rule (a declaration without an identifier) and add an alternative production for `sizeof`.

The C standard calls a name-less declaration an *abstract declarator*. It looks like a declarator with the identifier removed:

| Concrete | Abstract |
|---|---|
| `int x` | `int` |
| `int *x` | `int *` |
| `int x[4]` | `int [4]` |
| `int (*x)[4]` | `int (*)[4]` |

`abstract_declarator` is a copy of `declarator` with the identifier-required check removed:

```c
// abstract-declarator = "*"* ("(" abstract-declarator ")")? type-suffix
static Type *abstract_declarator(Token **rest, Token *tok, Type *ty) {
  while (equal(tok, "*")) {
    ty = pointer_to(ty);
    tok = tok->next;
  }

  if (equal(tok, "(")) {
    Token *start = tok;
    Type dummy = {};
    abstract_declarator(&tok, start->next, &dummy);
    tok = skip(tok, ")");
    ty = type_suffix(rest, tok, ty);
    return abstract_declarator(&tok, start->next, ty);
  }

  return type_suffix(rest, tok, ty);
}

// type-name = declspec abstract-declarator
static Type *typename(Token **rest, Token *tok) {
  Type *ty = declspec(&tok, tok, NULL);
  return abstract_declarator(rest, tok, ty);
}
```

The double-walk recursion is the same idiom as §10.3, just without an identifier at the leaf.

`primary` grows one new alternative:

```diff
 // primary = "(" "{" stmt+ "}" ")"
 //         | "(" expr ")"
+//         | "sizeof" "(" type-name ")"
 //         | "sizeof" unary
 //         | ident func-args?
 //         | str
 //         | num
```

```c
if (equal(tok, "sizeof") && equal(tok->next, "(") && is_typename(tok->next->next)) {
  Type *ty = typename(&tok, tok->next->next);
  *rest = skip(tok, ")");
  return new_num(ty->size, start);
}
```

The two-token lookahead — checking that the next two tokens are `(` and a typename — picks the parenthesized-type-name form. If those don't both match, the existing `sizeof unary` form kicks in, which still handles `sizeof x` and `sizeof(x)` (as a parenthesized expression).

`test/sizeof.c` is new and exercises everything:

```c
ASSERT(1, sizeof(char));
ASSERT(2, sizeof(short));
ASSERT(4, sizeof(int));
ASSERT(8, sizeof(long));
ASSERT(8, sizeof(char *));
ASSERT(8, sizeof(int **));
ASSERT(8, sizeof(int(*)[4]));
ASSERT(32, sizeof(int*[4]));
ASSERT(16, sizeof(int[4]));
ASSERT(48, sizeof(int[3][4]));
ASSERT(8, sizeof(struct {int a; int b;}));
```

`int(*)[4]` and `int*[4]` differ by exactly the parenthesization — pointer to array of 4 ints (8 bytes) versus array of 4 pointers to int (32 bytes). The spiral rule in action.

### Where we are

`sizeof` accepts both expressions and type names. The `abstract_declarator` and `typename` helpers exist; they will be reused immediately, in the next section, for cast expressions.

---

## 10.8 — 32-bit register arithmetic

> `git checkout cb81a379d9f7aef32fb1bbebd18f8618e1617a3f` — *Use 32 bit registers for char, short and int*

The codegen catch-up that §10.1 set up. After this commit, when chibicc compiles `int a + int b`, the `add` instruction uses 32-bit registers: `add %edi, %eax`, not `add %rdi, %rax`. For `long`, the existing 64-bit forms remain.

The change is tightly local to one function. `gen_expr`'s arithmetic tail used to read `%rax` and `%rdi` as fixed strings:

```c
case ND_ADD:
  println("  add %%rdi, %%rax");
  return;
case ND_SUB:
  println("  sub %%rdi, %%rax");
  return;
```

Now the operand widths are computed from the operand types and emitted as variable strings:

```c
char *ax, *di;

if (node->lhs->ty->kind == TY_LONG || node->lhs->ty->base) {
  ax = "%rax";
  di = "%rdi";
} else {
  ax = "%eax";
  di = "%edi";
}

switch (node->kind) {
case ND_ADD:
  println("  add %s, %s", di, ax);
  return;
case ND_SUB:
  println("  sub %s, %s", di, ax);
  return;
...
```

The condition is "operand is `long` *or* a pointer (`base != NULL`)." Pointers are 8 bytes wide and need 64-bit arithmetic. Everything else — `char`, `short`, `int`, `_Bool`, `enum` — fits in 32 bits, and 32-bit `add` is the right answer.

Why not 8- and 16-bit register arithmetic? Because the comment in `load` from §10.10 says the upper half of `%eax` may contain garbage, but the *lower* half always contains a valid value, sign-extended from whatever width the source had. That is: when chibicc loads a `char`, it issues `movsbl`, which sign-extends the 8-bit value to 32 bits in `%eax`. When it loads a `short`, `movswl` sign-extends 16 to 32. And for `int`, the value is already 32 bits. So at the point of the `add`, the operand always occupies at least the full lower 32 bits of the register, and 32-bit `add` produces the correct sum-modulo-2^32. (The full sign-extension to 64 bits — what `movsxd` would do — isn't required for correctness inside the expression and would just be wasted work.)

`ND_DIV` gets a special branch because the sign-extension required before `idiv` is different at 32 vs. 64 bits:

```c
case ND_DIV:
  if (node->lhs->ty->size == 8)
    println("  cqo");
  else
    println("  cdq");
  println("  idiv %s", di);
  return;
```

`cqo` ("convert quad to oct") sign-extends `%rax` into `%rdx:%rax` before `idiv`. `cdq` ("convert double to quad") sign-extends `%eax` into `%edx:%eax` before the 32-bit `idiv`. Picking the right one based on operand size keeps the dividend correctly extended.

### What this commit *doesn't* do

It doesn't insert sign-extending casts when the two operands of a binary expression have different types. `int + long` still happens with the 64-bit code path because `node->lhs->ty->kind == TY_LONG` is true (assuming `add_type` set the type already). `short + long` works because `add_type` will eventually narrow `short` to the bottom 16 bits of `%eax` via `movswl` during `load`, and then commit 68's `usual_arith_conv` will insert an explicit `(long)short` cast, making the operand reach the `add` as a properly-extended long. The two pieces — *load with sign-extension* and *cast inserted by `add_type`* — combine to make mixed-width arithmetic work.

`load` is unchanged here. It still emits `movsbq`/`movswq`/`movsxd` to load a 1-, 2-, or 4-byte value into `%rax` with sign-extension to 64 bits. The next codegen change (in commit 68) adjusts the `char`/`short` loads to extend only to 32 bits, which is the form we want now that the arithmetic code path uses `%eax`/`%edi`.

### Where we are

`int + int` finally compiles to `add %edi, %eax`, and `long + long` to `add %rdi, %rax`. Pointers stay on the 64-bit path. Division uses the right sign-extension for its operand width. The argreg-split foreshadowed in §7.2 is now the operand-width split running through the entire arithmetic core of the compiler.

---

## 10.9 — Casts

> `git checkout cfc4fa94c1eb17f37466571f74bbdfae03a6e11f` — *Add type cast*

The `(type) expr` syntax. Adds a new node kind, a new parser arm, a new codegen path, and an instruction-table for the narrowing/widening cases.

### The AST node

A cast is a unary operator: it takes one expression and produces a same-or-different-typed value. `ND_CAST` joins `NodeKind`:

```diff
   ND_STMT_EXPR, // Statement expression
   ND_VAR,       // Variable
   ND_NUM,       // Integer
+  ND_CAST,      // Type cast
 } NodeKind;
```

A small constructor:

```c
static Node *new_cast(Node *expr, Type *ty) {
  add_type(expr);

  Node *node = calloc(1, sizeof(Node));
  node->kind = ND_CAST;
  node->tok = expr->tok;
  node->lhs = expr;
  node->ty = copy_type(ty);
  return node;
}
```

`ty` is copied because the cast's destination type may be modified independently of the source `Type` object — for example, by being assigned a different size or aligning differently. The `copy_type` call exists to make sure the cast doesn't share storage with whatever `Type *` was passed in.

### Grammar

A new precedence level wedges in between `unary` and `mul`. The grammar comment says it nicely:

```c
// cast = "(" type-name ")" cast | unary
```

A cast is "an opening paren followed by a type name followed by a closing paren followed by another cast" — recursive, so `(int)(char)x` works — or, falling through, just a unary expression. The recursion is right-associative, which matches how casts compose in C.

The parser:

```c
static Node *cast(Token **rest, Token *tok) {
  if (equal(tok, "(") && is_typename(tok->next)) {
    Token *start = tok;
    Type *ty = typename(&tok, tok->next);
    tok = skip(tok, ")");
    Node *node = new_cast(cast(rest, tok), ty);
    node->tok = start;
    return node;
  }

  return unary(rest, tok);
}
```

`is_typename(tok->next)` is the lookahead: a `(` followed by a typename means cast; a `(` followed by anything else means a parenthesized expression. Because `is_typename` consults the symbol table, `(t)x` parses as a cast iff `t` is a typedef in scope, and as a function call `(t)(x)` — wait, no, function calls in chibicc parse via `primary`'s `ident "("` arm, not as `(expr)`. The ambiguity here is exactly the one C has: `(t)x` where `t` is an unknown identifier could in principle be either a cast or a parenthesized expression with a misplaced `x`. chibicc resolves it the standard way: if `t` is a typedef, cast; otherwise, fail at parse time when the inner expression doesn't end with `)`.

The grammar threads through every spot that previously called `unary`. Multiplication and `+`, `-`, `&`, `*` (deref) all now go through `cast`:

```diff
-// mul = unary ("*" unary | "/" unary)*
+// mul = cast ("*" cast | "/" cast)*
```

```diff
-// unary = ("+" | "-" | "*" | "&") unary
+// unary = ("+" | "-" | "*" | "&") cast
 //       | postfix
```

This is the standard C precedence: `(int)*p` is `(int)(*p)`, not `*((int)p)` — a cast binds *less* tightly than postfix and unary operators on its right side, but tighter than multiplicative. Adding a level between `unary` and `mul` and routing the unary-operator recursive calls through `cast` produces exactly that.

### Codegen

The narrowing-and-widening table:

```c
enum { I8, I16, I32, I64 };

static int getTypeId(Type *ty) {
  switch (ty->kind) {
  case TY_CHAR:  return I8;
  case TY_SHORT: return I16;
  case TY_INT:   return I32;
  }
  return I64;
}

static char i32i8[]  = "movsbl %al, %eax";
static char i32i16[] = "movswl %ax, %eax";
static char i32i64[] = "movsxd %eax, %rax";

static char *cast_table[][10] = {
  {NULL,  NULL,   NULL, i32i64}, // i8
  {i32i8, NULL,   NULL, i32i64}, // i16
  {i32i8, i32i16, NULL, i32i64}, // i32
  {i32i8, i32i16, NULL, NULL},   // i64
};

static void cast(Type *from, Type *to) {
  if (to->kind == TY_VOID)
    return;

  int t1 = getTypeId(from);
  int t2 = getTypeId(to);
  if (cast_table[t1][t2])
    println("  %s", cast_table[t1][t2]);
}
```

Read this as a 4×4 table indexed by (source size, destination size). The diagonal (i8→i8, i16→i16, etc.) is `NULL` — no instruction needed, the value is already the right shape. The columns "down" (going to a smaller type) are also `NULL` for sizes ≤ 4 because of the upper-bits-may-contain-garbage convention in `load`: chibicc never *guarantees* the upper 56 bits of an `%eax` holding a `char` are zero, but it also never *reads* them, because every consumer that wants 8 bits will use `%al` (not `%rax`). Truncation in chibicc is implicit.

The columns "up" (going to a larger type) need explicit sign-extension for char-or-short → 32, and 32 → 64. But the chart actually elides char-or-short → 64: `t2 == 64` is reached via `i32i64` (`movsxd %eax, %rax`), and the surrounding load convention guarantees that when a `char` or `short` reaches a register, it's already been sign-extended to 32 bits via `movsbl` / `movswl` in `load`. So the 32 → 64 step is the only widening explicitly emitted, and it composes with the loaded-as-int convention to produce the right answer for char → long and short → long without a special table entry.

A subtlety: `cast_table[I64][I32]` is `NULL`, meaning "cast from `long` to `int` emits nothing." That's because chibicc never zeroes the upper bits — the value remains in `%rax`, and a later `int`-sized load will read only the low 4 bytes from memory. For an `int x = (int) someLongVar;` the low 32 bits are written to `x`, and a subsequent read of `x` will sign-extend back to 64 bits via `movsxd`. Truncation followed by sign-extension is exactly the C semantics for narrowing-then-widening.

`gen_expr` grows one switch arm:

```c
case ND_CAST:
  gen_expr(node->lhs);
  cast(node->lhs->ty, node->ty);
  return;
```

### Tests

`test/cast.c` exercises the table:

```c
ASSERT(131585, (int)8590066177);
ASSERT(513, (short)8590066177);
ASSERT(1, (char)8590066177);
ASSERT(1, (long)1);
ASSERT(0, (long)&*(int *)0);
ASSERT(513, ({ int x=512; *(char *)&x=1; x; }));
ASSERT(5, ({ int x=5; long y=(long)&x; *(int*)y; }));
(void)1;
```

`8590066177` is `2^33 + 2`. Casting to `int` keeps the low 32 bits — `2 + (2^33 mod 2^32) = 2 + 0 = 131585`? Let me check: `2^33 = 8589934592`. `8590066177 - 8589934592 = 131585`. So `8590066177` low 32 bits are 131585. Casting to `short` keeps the low 16 bits: `131585 mod 65536 = 513`. Casting to `char` keeps the low 8 bits: `513 mod 256 = 1`. The diagonal arithmetic checks out.

`(void)1` is the standard "discard a value" idiom. The codegen for `cast(int, void)` emits nothing — the table's `to->kind == TY_VOID` check is a fast exit at the top.

### Where we are

Casts are first-class. The narrowing-and-widening table is small (one helper per non-trivial transition); the implicit-truncation-on-narrowing convention saves several entries. The next commit uses casts as a building block for the *implicit* type conversions C inserts at every binary operator and assignment.

---

## 10.10 — The usual arithmetic conversion

> `git checkout 8b430a6c5fd6d33a637f2c615f8e5ec59e7be30e` — *Implement usual arithmetic conversion*

The C standard's name for the rule that says "if you add an `int` and a `long`, the `int` is first promoted to `long`, and the result is `long`." More generally: when a binary operator has two operands of different arithmetic types, the operands are converted to a common type *before* the operator runs, and the result has that common type.

chibicc implements this by inserting `ND_CAST` nodes at parse-and-typecheck time. The mechanism is `add_type`: when the type-checker walks a `ND_ADD`, `ND_SUB`, `ND_MUL`, `ND_DIV`, `ND_EQ`, `ND_NE`, `ND_LT`, `ND_LE`, it computes a *common type* for the two operands and rewrites both children to be casts to that type.

### `get_common_type` and `usual_arith_conv`

```c
static Type *get_common_type(Type *ty1, Type *ty2) {
  if (ty1->base)
    return pointer_to(ty1->base);
  if (ty1->size == 8 || ty2->size == 8)
    return ty_long;
  return ty_int;
}

static void usual_arith_conv(Node **lhs, Node **rhs) {
  Type *ty = get_common_type((*lhs)->ty, (*rhs)->ty);
  *lhs = new_cast(*lhs, ty);
  *rhs = new_cast(*rhs, ty);
}
```

The common-type rules, in chibicc's simplified form:
- If the left operand is a pointer (`base != NULL`), the common type is "pointer to that base." This is the asymmetry that makes pointer arithmetic work — `ptr + int` keeps the pointer's type, doesn't try to promote to common-numeric.
- Otherwise, if either operand is 8 bytes, the common type is `long`.
- Otherwise (both ≤ 4 bytes), the common type is `int`.

The third rule is the C standard's *integer promotion*: `char` and `short` are always promoted to at least `int` before any binary operator runs. chibicc's table makes this happen because `get_common_type` returns `ty_int` for any pair of small integers, and `usual_arith_conv` casts both operands to `int`.

The new `add_type` arms for the binary-arithmetic operators:

```c
case ND_ADD:
case ND_SUB:
case ND_MUL:
case ND_DIV:
  usual_arith_conv(&node->lhs, &node->rhs);
  node->ty = node->lhs->ty;
  return;
case ND_NEG: {
  Type *ty = get_common_type(ty_int, node->lhs->ty);
  node->lhs = new_cast(node->lhs, ty);
  node->ty = ty;
  return;
}
case ND_EQ:
case ND_NE:
case ND_LT:
case ND_LE:
  usual_arith_conv(&node->lhs, &node->rhs);
  node->ty = ty_int;
  return;
```

`ND_NEG` (unary minus) uses the same machinery with a synthetic left operand of `ty_int`, which forces the integer-promotion rule. `-x` for `char x` produces a value of type `int`, not `char`.

The comparison operators (`==`, `!=`, `<`, `<=`) get `usual_arith_conv` to convert the operands but then explicitly set the result type to `ty_int`. Comparisons return `int` in C even if their operands are `long`.

### `ND_NUM`'s type becomes value-dependent

Previously, every `ND_NUM` had type `ty_long` (the catch-all from §10.2). That's wasteful — `1 + 2` shouldn't produce a `long` result on a machine where `int` arithmetic is available. After this commit:

```c
case ND_NUM:
  node->ty = (node->val == (int)node->val) ? ty_int : ty_long;
  return;
```

If the literal value fits in 32 bits, its type is `int`; otherwise `long`. This means `1 + 2` at the AST level has both operands as `int`, `usual_arith_conv` keeps both as `int`, and the result is `int`. `1 + 2147483648` (which doesn't fit in `int`) types the right operand as `long`, and the rule promotes the left to `long`, with the result as `long`.

### Pointer arithmetic gets `new_long`

Two existing callers of `new_num` in pointer arithmetic — the multiplications-by-element-size in `new_add` and `new_sub` — switch to a new helper that always emits a `long`-typed literal:

```c
static Node *new_long(int64_t val, Token *tok) {
  Node *node = new_node(ND_NUM, tok);
  node->val = val;
  node->ty = ty_long;
  return node;
}
```

```diff
   // ptr + num
-  rhs = new_binary(ND_MUL, rhs, new_num(lhs->ty->base->size, tok), tok);
+  rhs = new_binary(ND_MUL, rhs, new_long(lhs->ty->base->size, tok), tok);
```

The reason: pointer arithmetic should happen in 64-bit because pointers are 64 bits. If the size were a 32-bit `ND_NUM` (because most struct sizes fit in `int`), `usual_arith_conv` of `(int) * (long)` would correctly promote it, but it's cleaner — and one fewer cast in the AST — to make the size literal a `long` from the start.

### `ND_ASSIGN` gets implicit conversion

```c
case ND_ASSIGN:
  if (node->lhs->ty->kind == TY_ARRAY)
    error_tok(node->lhs->tok, "not an lvalue");
  if (node->lhs->ty->kind != TY_STRUCT)
    node->rhs = new_cast(node->rhs, node->lhs->ty);
  node->ty = node->lhs->ty;
  return;
```

`int x = 1L;` now inserts an `ND_CAST` to `int` on the right-hand side. The `if (node->lhs->ty->kind != TY_STRUCT)` exclusion preserves the byte-loop struct assignment from §9.7 — struct assignment doesn't go through scalar casts.

### Codegen tweak: char/short load extends to int

`load` changes its small-type widths from "extend to quad" to "extend to long":

```diff
   if (ty->size == 1)
-    println("  movsbq (%%rax), %%rax");
+    println("  movsbl (%%rax), %%eax");
   else if (ty->size == 2)
-    println("  movswq (%%rax), %%rax");
+    println("  movswl (%%rax), %%eax");
   else if (ty->size == 4)
     println("  movsxd (%%rax), %%rax");
   else
     println("  mov (%%rax), %%rax");
```

The change matches the convention §10.8 set up: 8- and 16-bit values arrive in `%eax`, sign-extended to 32 bits, with the upper 32 bits of `%rax` undefined. The 32-bit arithmetic in `gen_expr` reads from `%eax` and ignores `%rax`'s upper half. When a wider type is needed, `add_type` will already have inserted a `(long)x` cast, which `cast()` codegens as `movsxd %eax, %rax` — extending to 64 bits exactly when needed.

The new comment in `load` documents the contract:

```c
// When we load a char or a short value to a register, we always
// extend them to the size of int, so we can assume the lower half of
// a register always contains a valid value. The upper half of a
// register for char, short and int may contain garbage. When we load
// a long value to a register, it simply occupies the entire register.
```

### Tests

`test/usualconv.c` is new:

```c
ASSERT((long)-5, -10 + (long)5);
ASSERT((long)-15, -10 - (long)5);
ASSERT((long)-50, -10 * (long)5);
ASSERT((long)-2, -10 / (long)5);

ASSERT(1, -2 < (long)-1);
ASSERT(1, -2 <= (long)-1);
ASSERT(0, -2 > (long)-1);
ASSERT(0, -2 >= (long)-1);

ASSERT(0, 2147483647 + 2147483647 + 2);
ASSERT((long)-1, ({ long x; x=-1; x; }));
```

`-2 < (long)-1` is the signedness test. With `int` and `long` operands, the rule promotes the `int` to `long` (sign-extending), and `(long)-2 < (long)-1` is true. Without the rule, the comparison would happen at 32 bits, where `-2` and `-1` are both negative and the signed comparison still works — but the *result type* would be wrong. The comparison result is `int` regardless.

`2147483647 + 2147483647 + 2` is the overflow test. `2147483647` is `INT_MAX`. Both operands are `int`, so the addition runs at 32 bits, overflows, and `INT_MAX + INT_MAX + 2 == 0` (modulo 2^32, accounting for `INT_MIN` and the `+2`). The test asserts that.

`({ long x; x=-1; x; })` reads `x` after assigning `-1` (an `int` literal) to it. The implicit conversion in `ND_ASSIGN` casts `-1` to `long`, sign-extending to all-1s, and reading back gets `(long)-1`.

A new test is also added to `test/sizeof.c`:

```c
ASSERT(8, sizeof(-10 + (long)5));
ASSERT(8, sizeof(-10 - (long)5));
ASSERT(8, sizeof(-10 * (long)5));
ASSERT(8, sizeof(-10 / (long)5));
```

`sizeof` of an arithmetic expression returns the size of the operator's *result type*. After this commit, `int + long` has type `long`, so `sizeof` is 8.

### Where we are

Mixed-width integer arithmetic produces the right type and the right value. Comparisons return `int`. Assignment narrows or widens implicitly. The `usual_arith_conv` machinery is used in five places (the four binary arithmetic operators and the four comparison operators), and `ND_NEG` and `ND_ASSIGN` use the underlying `get_common_type` and `new_cast` helpers directly.

A small interpretive note: chibicc doesn't implement the full C99 *integer promotion* rules. The standard's full rule promotes any `_Bool`, `char`, or `short` to `int` (or to `unsigned int` if the value won't fit in `int`); it also has rules for `unsigned int` versus `long`. chibicc's `get_common_type` covers the most common case (the `size == 8 ? long : int` ladder), but doesn't engage with signedness or with the `unsigned int` versus `long` rule because chibicc doesn't have unsigned integers yet. Those arrive in Chapter 14.

---

## 10.11 — Function-call type conversions

> `git checkout 9e211cbf1d459babf035fd6b3407c2bd184cb639` — *Report an error on undefined/undeclared functions*
> `git checkout 818352acc07d0a982076b4b49345b42be706f5e1` — *Handle return type conversion*
> `git checkout fdc80bc6b5faa058b88d838332c71b7101712896` — *Handle function argument type conversion*

Three commits, in `main` order. The first wires up the lookup; the second uses it to convert return values; the third uses it to convert arguments.

### Looking up the function (commit 69)

Before this commit, `funcall` parsed an identifier and a comma-separated argument list, never asking what the identifier referred to. The result was that calling an undefined function produced a link-time error rather than a parse-time one, and there was no way to type-check arguments because the parser had no `Type` for the function.

The fix consults the symbol table:

```c
VarScope *sc = find_var(start);
if (!sc)
  error_tok(start, "implicit declaration of a function");
if (!sc->var || sc->var->ty->kind != TY_FUNC)
  error_tok(start, "not a function");

Type *ty = sc->var->ty->return_ty;
```

Two error cases: `sc == NULL` (the name isn't in scope at all) and `sc->var == NULL || not a function type` (the name is in scope but isn't a function — for example, `int x; x();`). Both produce parse errors.

The return type is read from the looked-up function and stored on the `ND_FUNCALL` node:

```c
*rest = skip(tok, ")");

Node *node = new_node(ND_FUNCALL, start);
node->funcname = strndup(start->loc, start->len);
node->ty = ty;
node->args = head.next;
return node;
```

This is what makes calls into a function returning `long` actually have type `long` at the call site, instead of the §10.2 default of `ty_long` for *all* calls. (At this point the default still happens to match — `ty` is taken from `return_ty`, but `add_type`'s `ND_FUNCALL` case still hard-codes `ty_long`, and that wins because it runs after this assignment. Commit 71 will change that.)

The arguments also get `add_type` called on them, eagerly:

```c
while (!equal(tok, ")")) {
  if (cur != &head)
    tok = skip(tok, ",");
  cur = cur->next = assign(&tok, tok);
  add_type(cur);
}
```

Without `add_type`, the next commit's argument-conversion code would have no types to read from. Calling `add_type` here is preemptive.

A small but important tokenizer/runtime change: `error_at` and `error_tok` no longer call `exit(1)` from within `verror_at`. The exit moves to the wrappers:

```diff
 static void verror_at(int line_no, char *loc, char *fmt, va_list ap) {
   ...
-  exit(1);
 }

 void error_at(char *loc, char *fmt, ...) {
   ...
   verror_at(line_no, loc, fmt, ap);
+  exit(1);
 }

 void error_tok(Token *tok, char *fmt, ...) {
   ...
   verror_at(tok->line_no, tok->loc, fmt, ap);
+  exit(1);
 }
```

The motivation is to allow `verror_at` to be called for warnings (or non-fatal errors) without aborting the compile. Nothing in this commit calls it that way, but the structural separation is there. The `// Reports an error message in the following format and exit.` comment also loses its "and exit" because `verror_at` no longer does that.

The test harness's `test/test.h` gets the `assert` declaration:

```diff
 #define ASSERT(x, y) assert(x, y, #y)

+void assert(int expected, int actual, char *code);
 int printf();
```

Without this, every test file that uses `ASSERT` would now fail with "implicit declaration of a function" because the harness lookup would miss `assert`.

### Return type conversion (commit 70)

`return e;` previously emitted code for `e` and jumped to the function epilogue, ignoring whether `e`'s type matched the function's return type. After this commit, the return value is implicitly cast to the return type:

```c
if (equal(tok, "return")) {
  Node *node = new_node(ND_RETURN, tok);
  Node *exp = expr(&tok, tok->next);
  *rest = skip(tok, ";");

  add_type(exp);
  node->lhs = new_cast(exp, current_fn->ty->return_ty);
  return node;
}
```

The conversion uses `new_cast`, which means the same instruction-table from §10.9 generates the right narrowing/widening. Inside `char int_to_char(int x) { return x; }`, the `return x;` becomes `return (char)x;` — the parser inserts an `ND_CAST(int → char)`, codegen does nothing (no instruction needed for narrowing), and the function returns the low 8 bits in `%al`.

A static `current_fn` tracks which function is being parsed, so the return type is reachable from inside `stmt()`:

```c
static Obj *current_fn;
```

It's set in `function()` after the prototype is parsed:

```c
current_fn = fn;
```

There's already a `current_fn` in `codegen.c` for emitting the right return-label name. The parser's `current_fn` is a separate variable with the same name, in a different translation unit. (Whether to merge them is a code-cleanliness question Rui doesn't engage with here.)

The test exercises both narrowing and address returns:

```c
int g1;
int *g1_ptr() { return &g1; }
char int_to_char(int x) { return x; }

ASSERT(3, *g1_ptr());
ASSERT(5, int_to_char(261));
```

`int_to_char(261)` truncates: `261 mod 256 = 5`. `g1_ptr()` returns the address of a global, which the caller dereferences.

### Argument type conversion (commit 71)

The third commit walks the function's parameter list (now reachable via `sc->var->ty->params`) in lock-step with the argument list, casting each argument to the declared parameter type:

```c
Type *ty = sc->var->ty;
Type *param_ty = ty->params;

Node head = {};
Node *cur = &head;

while (!equal(tok, ")")) {
  if (cur != &head)
    tok = skip(tok, ",");

  Node *arg = assign(&tok, tok);
  add_type(arg);

  if (param_ty) {
    if (param_ty->kind == TY_STRUCT || param_ty->kind == TY_UNION)
      error_tok(arg->tok, "passing struct or union is not supported yet");
    arg = new_cast(arg, param_ty);
    param_ty = param_ty->next;
  }

  cur = cur->next = arg;
}
```

The `if (param_ty)` guards against argument-list overflow — calling `printf("hello", "world", 1, 2, 3)` with `int printf()` has no declared parameters, so the cast loop is skipped for the extra arguments. (This is the silent miscompile of "more than 6 arguments to a variadic function" we noted in §5.4 as an errata candidate; it's not addressed here.)

The `TY_STRUCT`/`TY_UNION` check is a polite refusal: chibicc still doesn't handle struct or union arguments. Errors at parse time rather than producing wrong code.

A `func_ty` field is added to `Node` so that codegen can also see the function type at the call site, in addition to the return type:

```diff
   // Function call
   char *funcname;
+  Type *func_ty;
   Node *args;
```

```c
node->func_ty = ty;
node->ty = ty->return_ty;
```

`func_ty` isn't used yet in codegen. It will become important when chibicc starts caring about variadic functions and floating-point arguments — for now it's a forward-reference plant.

A new test:

```c
int div_long(long a, long b) {
  return a / b;
}

ASSERT(-5, div_long(-10, 2));
```

`div_long(-10, 2)` is a call where `-10` and `2` are both `int` literals (their types are `ty_int` per §10.10's `ND_NUM` rule), and the parameters are `long`. The argument cast inserts `(long)-10` and `(long)2`. Without the cast, `gen_expr` for the arguments would push 32-bit values onto the stack, the function would read 64-bit values from `%rdi` and `%rsi`, and the upper 32 bits would be garbage — producing wrong arithmetic when divided.

### Where we are

Function calls are type-checked. Arguments are cast to parameter types; return values are cast to the function's declared return type. Calling an undeclared function is now an error. Struct and union arguments are explicitly rejected. The `func_ty` plant is laid for later codegen work.

What's still missing: chibicc still doesn't *check* that the argument count matches the parameter count. Calling `int_to_char(1, 2, 3)` silently drops the extra arguments. A real C compiler would error.

---

## 10.12 — `_Bool`

> `git checkout 44bba965cbe3827be2b68651e541b33fa040bb72` — *Add _Bool type*

`_Bool` is an integer type with a special conversion rule: assigning any non-zero value to a `_Bool` produces 1, not the low bit. `_Bool x = 256;` yields `x == 1`, in contrast to `char x = 256;` where `x == 0` (because `256 mod 256 == 0`). The rule is what makes `_Bool` a real boolean rather than a tiny integer.

The commit message lays it out:

> _Bool isn't just a 1-bit integer because when you convert a value to bool, the result is 1 if the original value is non-zero. This is contrary to the other small integral types, e.g. char, as you can see below:
>
>   char x  = 256; // x is 0
>   _Bool y = 256; // y is 1

`TY_BOOL` joins the type enum, sized 1 byte and aligned 1 byte (matching the C standard, which says `sizeof(_Bool) == 1`):

```c
Type *ty_bool = &(Type){TY_BOOL, 1, 1};
```

`is_integer` adds it to the list. The keyword is added to the lexer, to `is_typename`, and to `declspec`. The `declspec` switch adds one new arm:

```c
case BOOL:
  ty = ty_bool;
  break;
```

The two-bit-per-keyword encoding gets a small renumbering — `BOOL` slots in between `VOID` and `CHAR`, and the existing constants shift. That's fine because nothing outside `declspec` looks at the values.

The interesting part is the codegen. `cast` gets a special arm:

```c
if (to->kind == TY_BOOL) {
  cmp_zero(from);
  println("  setne %%al");
  println("  movzx %%al, %%eax");
  return;
}
```

`cmp_zero` is a new helper that compares `%rax` (or `%eax`, depending on width) against zero:

```c
static void cmp_zero(Type *ty) {
  if (is_integer(ty) && ty->size <= 4)
    println("  cmp $0, %%eax");
  else
    println("  cmp $0, %%rax");
}
```

Then `setne %al` sets `%al` to 1 if the comparison was non-zero (sign and zero flags), 0 otherwise — exactly the `_Bool` conversion. `movzx %al, %eax` zero-extends `%al` to `%eax`, producing a clean 32-bit `0` or `1` value (not sign-extended garbage).

That's the whole `_Bool` conversion: compare to zero, set on not-equal, zero-extend. Three instructions. The result lives in the low byte of `%eax`, gets stored as a 1-byte value (because `_Bool` is 1 byte), and reads back as 0 or 1.

### Tests

```c
ASSERT(0, ({ _Bool x=0; x; }));
ASSERT(1, ({ _Bool x=1; x; }));
ASSERT(1, ({ _Bool x=2; x; }));
ASSERT(1, (_Bool)1);
ASSERT(1, (_Bool)2);
ASSERT(0, (_Bool)(char)256);
```

The third line is the canonical case: `_Bool x = 2` assigns `2`, the `ND_ASSIGN` from §10.10 inserts a cast to `_Bool`, codegen runs `cmp $0; setne; movzx`, and `1` is what gets stored. `(_Bool)(char)256` is the composition: `(char)256 = 0`, then `(_Bool)0 = 0`. Two casts, two table lookups, the composition produces `0`.

Function tests for parameter and return types:

```c
_Bool bool_fn_add(_Bool x) { return x + 1; }
_Bool bool_fn_sub(_Bool x) { return x - 1; }

ASSERT(1, bool_fn_add(3));
ASSERT(0, bool_fn_sub(3));
```

`bool_fn_add(3)` casts `3` to `_Bool` (yielding `1`), then `1 + 1 = 2` happens at `int` width (per the integer-promotion rule from §10.10), then the return cast back to `_Bool` turns `2` into `1`. Two `_Bool` conversions in a single call.

### Where we are

`_Bool` is a real type. Casts to `_Bool` produce 0 or 1, never any other value. The conversion is three instructions in codegen. Everything else — parameter passing, return values, comparisons — falls out of the existing machinery from §10.9 and §10.10.

---

## 10.13 — Character literals

> `git checkout aa0accc75e9358d313fef0a6d4005103e2ce25f5` — *Add character literal*

`'a'` becomes a tokenizer change. The token's `kind` is `TK_NUM`, its `val` is the integer value of the character, and the parser doesn't know the difference between `'a'` and `97`. That's exactly what the C standard says.

`read_char_literal`:

```c
static Token *read_char_literal(char *start) {
  char *p = start + 1;
  if (*p == '\0')
    error_at(start, "unclosed char literal");

  char c;
  if (*p == '\\')
    c = read_escaped_char(&p, p + 1);
  else
    c = *p++;

  char *end = strchr(p, '\'');
  if (!end)
    error_at(p, "unclosed char literal");

  Token *tok = new_token(TK_NUM, start, end + 1);
  tok->val = c;
  return tok;
}
```

The tokenizer recognizes `'` and dispatches:

```c
if (*p == '\'') {
  cur = cur->next = read_char_literal(p);
  p += cur->len;
  continue;
}
```

`read_escaped_char` was added in §7.4 for string literals. It already handles `\n`, `\t`, `\\`, `\'`, `\"`, octal escapes, and hex escapes — which means the same machinery now powers character escapes inside `'...'`. `'\n'` gets handed to `read_escaped_char(&p, p + 1)` and comes back as `10`.

The token's `val` is a `char`, which on x86-64 Linux is signed 8-bit. So `'\x80'` (the byte `0x80`) is stored as `-128`, sign-extended through `int64_t val`, which tests:

```c
ASSERT(97, 'a');
ASSERT(10, '\n');
ASSERT(-128, '\x80');
```

In a strict C, the type of a character constant is `int`, not `char` — but the *value* is the integer value of the character interpreted as `char`. chibicc represents the value through the existing `TK_NUM` path and never has an opportunity to disagree.

### Where we are

Character literals work. They tokenize as `TK_NUM`, leveraging the existing escape-sequence handling from §7.4. The parser is unchanged.

---

## 10.14 — `enum`

> `git checkout 48ba2656fecc646ec4eb7f943fa94b02ed9725c7` — *Add enum*

`enum` introduces a new kind of type and a way to define named integer constants. The implementation is the chapter's third interesting symbol-table extension (after typedef and after struct tags), and it puts every namespace question chibicc has ever asked into focus.

### Type

`TY_ENUM` joins the type enum:

```diff
 typedef enum {
   TY_VOID,
   TY_BOOL,
   TY_CHAR,
   TY_SHORT,
   TY_INT,
   TY_LONG,
+  TY_ENUM,
   TY_PTR,
   ...
```

A constructor:

```c
Type *enum_type(void) {
  return new_type(TY_ENUM, 4, 4);
}
```

Sized 4, aligned 4 — chibicc represents enum values as `int`s. Strict C says implementations may choose any integer type that fits all the values; chibicc just picks `int`. (`is_integer` is updated to include `TY_ENUM`, so enum values can be used in arithmetic and comparisons.)

### Storage in the symbol table

An enum constant is bound to a name and a value. `VarScope` already has slots for variable, typedef name; it grows two more for enum:

```diff
 struct VarScope {
   VarScope *next;
   char *name;
   Obj *var;
   Type *type_def;
+  Type *enum_ty;
+  int enum_val;
 };
```

`enum_ty` is the enum type the constant belongs to (so `enum Color { RED, GREEN, BLUE };` lets `RED` know its type is `enum Color`). `enum_val` is the integer value.

Enum *tags* (the `Color` in `enum Color`) live in the `tags` namespace from §9.4 — the same one that holds struct and union tags. C99 says struct, union, and enum tags share a single namespace per scope (§6.2.3), and chibicc gets this right because all three reach the tag scope through `push_tag_scope` and `find_tag`. (The Ch 9 prose treated struct/union tag-sharing as an errata candidate; that was a misreading on our part — sharing the tag namespace across struct/union/enum is exactly what the standard requires. We'll quietly skip restating it as a wart in this section.)

### `enum_specifier`

```c
// enum-specifier = ident? "{" enum-list? "}"
//                | ident ("{" enum-list? "}")?
//
// enum-list      = ident ("=" num)? ("," ident ("=" num)?)*
static Type *enum_specifier(Token **rest, Token *tok) {
  Type *ty = enum_type();

  Token *tag = NULL;
  if (tok->kind == TK_IDENT) {
    tag = tok;
    tok = tok->next;
  }

  if (tag && !equal(tok, "{")) {
    Type *ty = find_tag(tag);
    if (!ty)
      error_tok(tag, "unknown enum type");
    if (ty->kind != TY_ENUM)
      error_tok(tag, "not an enum tag");
    *rest = tok;
    return ty;
  }

  tok = skip(tok, "{");

  int i = 0;
  int val = 0;
  while (!equal(tok, "}")) {
    if (i++ > 0)
      tok = skip(tok, ",");

    char *name = get_ident(tok);
    tok = tok->next;

    if (equal(tok, "=")) {
      val = get_number(tok->next);
      tok = tok->next->next;
    }

    VarScope *sc = push_scope(name);
    sc->enum_ty = ty;
    sc->enum_val = val++;
  }

  *rest = tok->next;

  if (tag)
    push_tag_scope(tag, ty);
  return ty;
}
```

Two cases. With a `{` body, parse the enum list, push each name into the variable scope (with `enum_ty` and `enum_val` set), and optionally bind the tag name to the new type. Without a `{` body — just `enum Foo` — look up `Foo` in the tag scope, expect a `TY_ENUM`, return that. The `tag && !equal(tok, "{")` early-exit is the "use existing tag" path.

`val` starts at zero and increments after each entry, with `=` overriding it for the next entry. So:

```c
enum { zero, five=5, three=3, four };
```

binds `zero=0` (initial), `five=5` (override), `three=3` (override), `four=4` (incrementing past `three`). The test:

```c
ASSERT(0, ({ enum { zero, five=5, three=3, four }; zero; }));
ASSERT(5, ({ enum { zero, five=5, three=3, four }; five; }));
ASSERT(3, ({ enum { zero, five=5, three=3, four }; three; }));
ASSERT(4, ({ enum { zero, five=5, three=3, four }; four; }));
```

### Wiring to `declspec` and `is_typename`

`declspec` learns to dispatch on `enum`:

```diff
     Type *ty2 = find_typedef(tok);
-    if (equal(tok, "struct") || equal(tok, "union") || ty2) {
+    if (equal(tok, "struct") || equal(tok, "union") || equal(tok, "enum") || ty2) {
       if (counter)
         break;

       if (equal(tok, "struct")) {
         ty = struct_decl(&tok, tok->next);
       } else if (equal(tok, "union")) {
         ty = union_decl(&tok, tok->next);
+      } else if (equal(tok, "enum")) {
+        ty = enum_specifier(&tok, tok->next);
       } else {
         ty = ty2;
         tok = tok->next;
       }
       ...
```

`is_typename` adds `"enum"` to its keyword list. The lexer adds it. Same as `typedef` (§10.6), nothing structurally new.

### Reading an enum constant

In `primary`, when the parser sees an identifier, it looks it up in `find_var`. The lookup may now return a `VarScope` whose `var` is null but whose `enum_ty` is set, meaning "this is an enum constant." The new arm:

```c
// Variable or enum constant
VarScope *sc = find_var(tok);
if (!sc || (!sc->var && !sc->enum_ty))
  error_tok(tok, "undefined variable");

Node *node;
if (sc->var)
  node = new_var_node(sc->var, tok);
else
  node = new_num(sc->enum_val, tok);

*rest = tok->next;
return node;
```

If `sc->var` is set, the name is a variable — make an `ND_VAR` node. If `sc->enum_ty` is set, the name is an enum constant — make an `ND_NUM` node with the constant's value. Either way, the AST that comes out is one chibicc has been compiling for nine chapters.

### Tests

```c
ASSERT(0, ({ enum { zero, one, two }; zero; }));
ASSERT(1, ({ enum { zero, one, two }; one; }));
ASSERT(2, ({ enum { zero, one, two }; two; }));
ASSERT(5, ({ enum { five=5, six, seven }; five; }));
ASSERT(6, ({ enum { five=5, six, seven }; six; }));
ASSERT(4, ({ enum { zero, one, two } x; sizeof(x); }));
ASSERT(4, ({ enum t { zero, one, two }; enum t y; sizeof(y); }));
```

`sizeof(x)` for `enum { zero, one, two } x;` is 4 because `enum_type()` builds a 4-byte type. The two-statement `enum t { ... }; enum t y;` declares the tag in the first statement and uses it in the second.

### Where we are

`enum` works. Tags live with struct and union tags (correctly — all three share the C-standard's tag namespace). Constants live in the variable namespace, alongside ordinary identifiers and typedef names (also correctly). The `VarScope` entry has gained two more fields. The `declspec` rewrite from §10.5 is the framework that enum slots into.

---

## 10.15 — File-scope `static` functions

> `git checkout 736232f3d672dae9a1ddae800909204c17fbe37c` — *Support file-scope functions*

The `static` keyword on a function. In C, file-scope `static` means *internal linkage* — the function's name is not visible outside its translation unit, so the linker won't see it and won't conflict with similarly-named functions in other files.

The implementation rides through the same `VarAttr` channel as `typedef`:

```diff
 typedef struct {
   bool is_typedef;
+  bool is_static;
 } VarAttr;
```

`declspec` handles both keywords symmetrically:

```c
if (equal(tok, "typedef") || equal(tok, "static")) {
  if (!attr)
    error_tok(tok, "storage class specifier is not allowed in this context");

  if (equal(tok, "typedef"))
    attr->is_typedef = true;
  else
    attr->is_static = true;

  if (attr->is_typedef + attr->is_static > 1)
    error_tok(tok, "typedef and static may not be used together");
  tok = tok->next;
  continue;
}
```

The `(is_typedef + is_static) > 1` check is a polite refusal: both flags can't be set at the same time, because the C standard says a single declaration can have at most one storage-class specifier. (`extern` is a separate flag added in Chapter 13; with three flags, the same check generalizes.) The keyword is added to the lexer and to `is_typename`.

`function()` reads the static flag from the attribute:

```diff
-static Token *function(Token *tok, Type *basety) {
+static Token *function(Token *tok, Type *basety, VarAttr *attr) {
   Type *ty = declarator(&tok, tok, basety);

   Obj *fn = new_gvar(get_ident(ty->name), ty);
   fn->is_function = true;
   fn->is_definition = !consume(&tok, tok, ";");
+  fn->is_static = attr->is_static;
```

`Obj` gets a third boolean:

```diff
   bool is_function;
   bool is_definition;
+  bool is_static;
```

And codegen emits `.local` instead of `.globl` for static functions:

```c
if (fn->is_static)
  println("  .local %s", fn->name);
else
  println("  .globl %s", fn->name);
```

`.globl` makes the symbol visible to the linker; `.local` does not. After this commit, `static int foo(){...}` produces an object file whose `foo` symbol is internal and won't collide with another `foo` defined in another file.

### Tests

`test/common` (the harness, already in C as of §8.2) gets a static helper:

```c
static int static_fn() { return 5; }
```

And `test/function.c` gets a colliding one:

```c
static int static_fn() { return 3; }

ASSERT(3, static_fn());
```

Both files compile and link cleanly. Each file's `static_fn` is internal; the names don't collide. The test calls the local one and expects `3`.

### Where we are

`static` works on file-scope functions. The `VarAttr` machinery generalizes one flag further. Linkage — which is mostly Chapter 13's domain — gets its first chibicc encounter here.

---

## Recap

Twenty commits, fifteen sections, one concept interlude. The net change: chibicc's three integer types and one pointer type become eight integer types (`_Bool`, `char`, `short`, `int`, `long`, plus the `long long` alias and the `long int`/`int long` alternative spellings, plus `enum`). Function types acquire return and parameter conversion. A cast operator and an instruction-level cast table get the work right at runtime. A `typedef` machinery puts type-name aliasing in the symbol table. `static` on functions handles linkage. The parser handles every C declaration syntax a working programmer is likely to write.

Twenty commits is too many to summarize as a single table — the chapter has too much ground per commit. Instead, two tables, by theme:

| Commit | Topic |
|---|---|
| `5831eda` | `int` width changes from 8 to 4. Pre-factor for everything else; ships `argreg32`, `movsxd`, `mov %eax`, and `store_gp`. |
| `43c2f08` | `long` type — 8 bytes. Token/AST `val` widens to `int64_t`. |
| `9d48eef` | `short` type — 2 bytes. `argreg16` and the `movswq`/`mov %ax` paths. |
| `a817b23` | Nested declarators: `int (*x)[3]` and friends. The double-walk-with-dummy-type idiom. |
| `74e3acc` | Function declaration as separate from definition. `is_definition` flag on `Obj`. |
| `8c3503b` | `void` type. Variable-declared-`void` and void-pointer-deref errors. `is_typename` becomes a table-and-loop. |
| `287906a` | `declspec` rewrite handling `int long`, `long int`, etc. Bit-field counter + switch. |
| `f46370e` | `long long` and `long long int` as aliases for `long`. Two new switch arms. |
| `a6b82da` | `typedef`. `VarScope.type_def`, `VarAttr.is_typedef`, `parse_typedef`. `is_typename` becomes context-sensitive. |
| `67543ea` | `sizeof(typename)`. `abstract_declarator` and `typename` helpers. |

| Commit | Topic |
|---|---|
| `cb81a37` | 32-bit register arithmetic for non-`long`, non-pointer operands. `cqo`/`cdq` split for `idiv`. |
| `cfc4fa9` | `(type) expr` casts. `ND_CAST` node, `cast_table` codegen, `enum { I8, I16, I32, I64 }`. |
| `8b430a6` | Usual arithmetic conversion. `get_common_type`, `usual_arith_conv`, integer promotion at every binary operator. `ND_NUM` typed by value range. `load` extends to 32 bits. |
| `9e211cb` | Function-call lookup. Errors for "implicit declaration" and "not a function." `error_at` no longer exits from inside `verror_at`. |
| `818352a` | Return type conversion. `current_fn` in the parser. |
| `fdc80bc` | Argument type conversion. `Node.func_ty`. Struct/union arguments rejected. |
| `44bba96` | `_Bool` type. Cast-to-bool emits `cmp $0; setne; movzx`. |
| `aa0accc` | Character literal `'a'` via the existing escape-sequence machinery. |
| `48ba265` | `enum` type. Tag namespace shared with struct/union; constants in the variable namespace. |
| `736232f` | File-scope `static`. `VarAttr.is_static`, `Obj.is_static`, `.local` directive. |

Three structural shifts deserve repeating:

The first is the *bit-field-counter `declspec`*. Between commits 62 and 63 the parser's type-specifier handler became a small state machine that accepts every legal combination of C type-specifier and storage-class keywords. Every type or storage-class addition for the rest of the book — `_Bool`, `enum`, `static`, `extern`, `signed`, `unsigned`, `const`, `volatile` — slots into this framework by allocating two more bits and adding switch arms.

The second is the *context-sensitive `is_typename`*. After `typedef`, the question "does this token start a type" depends on the symbol table. The parser's distinction between declaration and statement now consults scope. This is the standard C lexer-versus-parser hack — sometimes known as "the lexer hack" — except that chibicc puts it in the parser by making `is_typename` look up the identifier in the current scope chain. (Real C compilers usually do the same thing; the alternative, marking typedef tokens at lex time, requires the lexer to know about scope, which is uglier.)

The third is the *cast machinery*. Once `ND_CAST` and `new_cast` exist, the type-checker uses them to insert implicit conversions everywhere: usual-arithmetic-conversion at binary operators, return-value conversion, argument-conversion, assignment-conversion. The codegen side is a 4×4 table plus a `_Bool` special case. Chapter 14's signed/unsigned types and Chapter 15's floating-point types both extend the table.

A few standing notes carried forward to Chapter 11:

- The *canonicalization-at-parse-time* count from §9.5 was six (five strict desugarings plus the `({...})` delegation). This chapter adds none directly. The cast insertions in §10.10 and §10.11 are *type-level* rewrites — not desugarings into more-primitive AST shapes — so we don't count them as canonicalizations. Chapter 11's `+=` family is the next canonicalization candidate.
- The *pre-factor-before-feature* count was three. This chapter's §10.1 (`int` → 32-bit) is the fourth, and is a long-running case: the codegen catch-up in §10.8 is ten commits later. The §10.6 `parse_typedef` and §10.6 `VarAttr` are arguably pre-factors for §10.15's `static` handling, but they ship together with their first feature so they don't count as separate.
- The *everything-fits-in-rax* invariant (broken in §9.7 for struct/union) gets refined here. After §10.10, the rule is: scalar values smaller than 8 bytes live in `%eax` with the low bits valid and the upper bits undefined; scalar values of size 8 live in `%rax`; struct and union "values" live at addresses pointed to by `%rax`. Three regimes, with the cast machinery moving values between them.
- The *`is_typename` symbol-table coupling* (§10.6) is the chapter's structural shift. Watch for further interactions with scope when Chapter 11 adds `goto` (which has its own label namespace) and Chapter 13 adds `extern` (which adds another `VarAttr` flag).
- The struct/union/enum *tag namespace* is one namespace per the C standard. The Ch 9 framing of struct/union tag-sharing as an errata candidate was wrong; quietly replaced by the correct C99 rule in §10.14.

Forward references for Chapter 11:

- `+=`, `-=`, `*=`, etc. are likely to canonicalize at parse time using the *generalized-lvalue comma operator* extension from §8.5.
- `++` and `--` have similar canonicalization-or-lowering questions.
- `?:` (the conditional operator) will use `usual_arith_conv` from §10.10.
- `goto` and labels introduce a fourth namespace (labels are function-scoped, not block-scoped, in C) and may interact with typedef names.
- `switch` builds on the `enum` plumbing only loosely — a `case` value is a constant expression, which is a separate parsing concept from enum constants.
