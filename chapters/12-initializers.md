# Chapter 12 — Initializers

> Commits covered: `22dd560`, `ae0a37d`, `a754732`, `0d71737`, `5b95533`, `e9d2c46`, `aca19dd`, `483b194`, `bbfe3f4`, `eeb62b6`, `1eae5ae`, `efa0f33`, `a58958c`, `fde464c`, `3d216e3`, `824543b`, `cd688a8`, `7a1f816`, `157356c`. Nineteen commits — the densest single arc in the compiler — taking chibicc's initializers from "scalar at definition" to the full C grammar.

Through Chapter 11 chibicc grew most of its operator surface and a real constant-expression evaluator. What it still couldn't do was the one feature C makes inescapable for serious programs: it couldn't write `int x[3] = {1, 2, 3};`, or `struct point p = {1, 2};`, or `char buf[] = "hello";`, or `int *table[] = {a, b, c};`. The parser accepted exactly one shape for an initializer — a single scalar expression on the right of `=`, lowered to one assignment node. Nineteen commits replace that with the C99 initializer grammar.

The chapter is the densest arc in the chibicc commit log. It runs from August 2019 (the first local initializer commit) through October 2020 (the last polish), with most of the work clustered in the September 2020 weekend that also produced the bulk of Chapter 11. Bundled aggressively, the nineteen commits fit into eleven sections plus a closer.

The eleven sections:

- **§12.1** — Local scalar and array initializers (commit 97).
- **§12.2** — Excess and missing array elements (commits 98, 99).
- **§12.3** — String literal initializers (commit 100).
- **§12.4** — Length omitted from the array (commit 101).
- **§12.5** — Struct and union local initializers (commits 102, 103, 104).
- **§12.6** — Global scalar and string initializers (commit 105).
- **§12.7** — Global struct, union, and pointer initializers (commits 106, 107).
- **§12.8** — Initializer-list ergonomics: braces, parens, trailing commas (commits 108, 109, 110).
- **§12.9** — Uninitialized globals to `.bss` (commit 111).
- **§12.10** — Flexible array members (commits 112, 113).
- **§12.11** — `void` as a parameter list, and `.align` for globals (commits 114, 115).

Two structural ideas anchor the chapter. The first is the *Initializer tree*: a small data structure, parallel in shape to a `Type`, that holds the parsed contents of a brace-enclosed initializer before any code is generated. Every commit in the chapter is in some sense a change to how the Initializer tree is filled in or consumed. §12.1 builds the structure; the rest of the chapter extends it.

The second is the *local-versus-global split*. A local initializer doesn't really need its own representation — it can be lowered to a sequence of assignments at the start of the function. A global initializer can't, because globals are placed in static storage and have to be filled in by data directives in the assembler output. The two paths share the front end (the `Initializer` tree, the parser predicate) and diverge at the back. The split shows up explicitly in §12.6, where chibicc grows a separate `gvar_initializer` mirroring `lvar_initializer`. From there, the chapter alternates between extending one side and then the other.

The chapter follows `main` order. As with Chapters 7 through 11, that order doesn't match calendar dates — the August-2019 commits and the September-2020 commits are interleaved on `main` — but it is the order Rui pinned and the order the chapter mapping uses.

---

## 12.1 — Local scalar and array initializers

> `git checkout 22dd560ecf06e9ac4a4c1be33be74bac7924f06a` — *Support local variable initializers*

This is the chapter's anchor commit. The `Initializer` data structure lives here, the recursive initializer parser lives here, and the *initializer lowering* — the conversion from a parsed initializer tree to a sequence of assignment expressions — lives here. Every later section in the chapter builds on top.

Before the commit, `declaration` handled the `=` after a declarator inline:

```c
Obj *var = new_lvar(get_ident(ty->name), ty);

if (!equal(tok, "="))
  continue;

Node *lhs = new_var_node(var, ty->name);
Node *rhs = assign(&tok, tok->next);
Node *node = new_binary(ND_ASSIGN, lhs, rhs, tok);
cur = cur->next = new_unary(ND_EXPR_STMT, node, tok);
```

That generates one `ND_ASSIGN` per declarator and that's it. It can't see a brace. After the commit, the same site delegates:

```c
Obj *var = new_lvar(get_ident(ty->name), ty);
if (equal(tok, "=")) {
  Node *expr = lvar_initializer(&tok, tok->next, var);
  cur = cur->next = new_unary(ND_EXPR_STMT, expr, tok);
}
```

`lvar_initializer` is the new piece. The work it does has three parts: parse the source into an `Initializer` tree, walk that tree to produce a chain of `ND_ASSIGN` nodes glued together with `ND_COMMA`, and return the chain as one expression statement. The compiler that runs after sees a flat sequence of stores — exactly what the pre-commit code generated, just written by the front end instead of the programmer.

The `Initializer` struct itself:

```c
typedef struct Initializer Initializer;
struct Initializer {
  Initializer *next;
  Type *ty;
  Token *tok;

  // If it's not an aggregate type and has an initializer,
  // `expr` has an initialization expression.
  Node *expr;

  // If it's an initializer for an aggregate type (e.g. array or struct),
  // `children` has initializers for its children.
  Initializer **children;
};
```

Two payloads, mutually exclusive in practice. For a scalar initializer (`int x = 5;`), `expr` holds the parsed AST for the right-hand side and `children` is null. For an aggregate initializer (`int x[3] = {1, 2, 3};`), `children` is an array of pointers to nested `Initializer` objects — one per element of the type — and `expr` is null at this level. The structure is a tree, with the same shape as the type that it's an initializer for. An `int x[2][3]` initializer is an `Initializer` with two children, each of which has three children, each of which carries a scalar `expr`. That parallel — *the Initializer tree mirrors the type* — is the structural idea the rest of the chapter extends.

The constructor reflects this:

```c
static Initializer *new_initializer(Type *ty) {
  Initializer *init = calloc(1, sizeof(Initializer));
  init->ty = ty;

  if (ty->kind == TY_ARRAY) {
    init->children = calloc(ty->array_len, sizeof(Initializer *));
    for (int i = 0; i < ty->array_len; i++)
      init->children[i] = new_initializer(ty->base);
  }

  return init;
}
```

Recursing through arrays. In this commit, struct and union types fall through (they're not handled yet) and only arrays get a `children` array. §12.5 will extend the constructor.

The parser is `initializer2`:

```c
// initializer = "{" initializer ("," initializer)* "}"
//             | assign
static void initializer2(Token **rest, Token *tok, Initializer *init) {
  if (init->ty->kind == TY_ARRAY) {
    tok = skip(tok, "{");

    for (int i = 0; i < init->ty->array_len; i++) {
      if (i > 0)
        tok = skip(tok, ",");
      initializer2(&tok, tok, init->children[i]);
    }
    *rest = skip(tok, "}");
    return;
  }

  init->expr = assign(rest, tok);
}
```

A two-arm dispatch on the *type*, not the token. If the slot to fill is an array, expect a `{ … }` and recurse through children. Otherwise, parse one `assign`-level expression and stash it in `init->expr`. The decision is type-driven because that's how C initializer-list parsing actually works: the same source token sequence means different things depending on the type being initialized. Same character `1` is a scalar initializer for `int x = 1;` but an element of an aggregate initializer for `int x[3] = {1, 2, 3};`.

Two helpers convert the parsed tree into AST:

```c
static Node *init_desg_expr(InitDesg *desg, Token *tok) {
  if (desg->var)
    return new_var_node(desg->var, tok);

  Node *lhs = init_desg_expr(desg->next, tok);
  Node *rhs = new_num(desg->idx, tok);
  return new_unary(ND_DEREF, new_add(lhs, rhs, tok), tok);
}

static Node *create_lvar_init(Initializer *init, Type *ty, InitDesg *desg, Token *tok) {
  if (ty->kind == TY_ARRAY) {
    Node *node = new_node(ND_NULL_EXPR, tok);
    for (int i = 0; i < ty->array_len; i++) {
      InitDesg desg2 = {desg, i};
      Node *rhs = create_lvar_init(init->children[i], ty->base, &desg2, tok);
      node = new_binary(ND_COMMA, node, rhs, tok);
    }
    return node;
  }

  Node *lhs = init_desg_expr(desg, tok);
  Node *rhs = init->expr;
  return new_binary(ND_ASSIGN, lhs, rhs, tok);
}
```

`InitDesg` ("init designator") is a stack of "where am I in the variable" markers:

```c
typedef struct InitDesg InitDesg;
struct InitDesg {
  InitDesg *next;
  int idx;
  Obj *var;
};
```

The base of the stack carries the variable being initialized; each frame above carries an array index. `init_desg_expr` walks the stack from the bottom up, building `*(var + idx)` style AST — which then `add_type` will resolve into `var[idx]` after the existing scalar-add lowering applies. By the time the recursion finishes, a designator like `{var=x, idx=1, idx=0}` has produced the AST equivalent of `x[1][0]`.

`create_lvar_init` walks the type and the Initializer tree together. For arrays, it threads `ND_COMMA` between per-element assignments. For scalars, it emits one `ND_ASSIGN` of the parsed `expr` into the designated lvalue. The result is a left-leaning comma chain of assignments — `((((null, x[0][0]=…), x[0][1]=…), x[0][2]=…), x[1][0]=…)` and so on — that the existing codegen already handles.

The leftmost leaf is `ND_NULL_EXPR`, a new node kind:

```c
typedef enum {
  ND_NULL_EXPR, // Do nothing
  ...
```

with codegen of one return statement:

```c
case ND_NULL_EXPR:
  return;
```

The reason it exists is the comma-chain construction. `create_lvar_init` for an array starts with *some* node and accumulates assignments by wrapping each in another comma. Without `ND_NULL_EXPR`, the loop would need a special case for the first element — the construction would be `if (i == 0) node = rhs; else node = comma(node, rhs);`. With `ND_NULL_EXPR` as the seed, the loop is uniform: every iteration is `node = comma(node, rhs)`. The cost is one no-op store at the start of every initialized array; the codegen for `ND_NULL_EXPR` is literally a `return`, and the existing dead-code-elimination is whatever the assembler can do. The win is uniformity.

The wrapping function `lvar_initializer`:

```c
// A variable definition with an initializer is a shorthand notation
// for a variable definition followed by assignments. This function
// generates assignment expressions for an initializer. For example,
// `int x[2][2] = {{6, 7}, {8, 9}}` is converted to the following
// expressions:
//
//   x[0][0] = 6;
//   x[0][1] = 7;
//   x[1][0] = 8;
//   x[1][1] = 9;
static Node *lvar_initializer(Token **rest, Token *tok, Obj *var) {
  Initializer *init = initializer(rest, tok, var->ty);
  InitDesg desg = {NULL, 0, var};
  return create_lvar_init(init, var->ty, &desg, tok);
}
```

Three lines. Parse the initializer into a tree; build a designator pointing at the variable; walk the tree producing a comma chain. The comment makes the lowering explicit. The chapter will return to this comment frequently; it pins what an initializer *is*, in chibicc's model, to "shorthand for a series of assignments."

The test that pins it down:

```c
ASSERT(2, ({ int x[2][3]={{1,2,3},{4,5,6}}; x[0][1]; }));
ASSERT(4, ({ int x[2][3]={{1,2,3},{4,5,6}}; x[1][0]; }));
ASSERT(6, ({ int x[2][3]={{1,2,3},{4,5,6}}; x[1][2]; }));
```

The two-dimensional case — a brace-list of brace-lists — is the recursive arm exercised twice deep.

A note on what *isn't* a canonicalization. Chapters 6 through 11 named eight features that desugar at parse time into existing AST shapes — `*`/`&` for arrays, `+=`/etc. via `to_assign`, `++`/`--` via the cast-and-subtract trick. Initializer lowering does the same kind of thing — `int x[3] = {1,2,3};` desugars into a comma chain of three assignments — but the Chapter 12 prose doesn't add it to the canonicalization count. The mechanism is conventionally called "initializer lowering" rather than "canonicalization at parse time" because it goes through a separate intermediate representation (the Initializer tree) before producing the lowered AST. The count stays at eight.

### Where we are

Local scalar initializers and local array initializers (including nested arrays) work. The `Initializer` tree, `InitDesg` designator stack, and `lvar_initializer` lowering pipeline are in place. Struct and union types fall through and don't yet accept brace-list initializers. Globals don't accept anything beyond the single-scalar shape from Chapter 7.

---

## 12.2 — Excess and missing array elements

> `git checkout ae0a37dc4b39018a95616836ae4aaf4c8bfd779b` — *Initialize excess array elements with zero*
>
> `git checkout a754732c046939cd87ac9fc8e9483ae9b3369449` — *Skip excess initializer elements*

Two commits handle the size mismatches between an array's declared length and the number of elements its initializer supplies. C's rule is asymmetric: too few initializers leaves the rest zero, too many is permissible (in chibicc, anyway — the standard says it's an error) and the excess is silently ignored.

The first commit handles "too few." The parser needs to stop consuming elements when it hits the closing brace, even if it hasn't filled the type's array length. One condition added to the `for` loop:

```diff
-    for (int i = 0; i < init->ty->array_len; i++) {
+    for (int i = 0; i < init->ty->array_len && !equal(tok, "}"); i++) {
       if (i > 0)
         tok = skip(tok, ",");
       initializer2(&tok, tok, init->children[i]);
     }
```

The codegen side does the actual zeroing. A new `ND_MEMZERO` node:

```c
ND_MEMZERO,   // Zero-clear a stack variable
```

with codegen that calls `rep stosb` to clear the variable's full footprint:

```c
case ND_MEMZERO:
  // `rep stosb` is equivalent to `memset(%rdi, %al, %rcx)`.
  println("  mov $%d, %%rcx", node->var->ty->size);
  println("  lea %d(%%rbp), %%rdi", node->var->offset);
  println("  mov $0, %%al");
  println("  rep stosb");
  return;
```

`rep stosb` is the x86 string-store loop: it writes `%al` to `[%rdi]`, increments `%rdi`, and decrements `%rcx`, repeating until `%rcx` reaches zero. Three setup instructions, one repeated store, one variable cleared.

The `lvar_initializer` wrapper grows a `MEMZERO` prefix:

```c
static Node *lvar_initializer(Token **rest, Token *tok, Obj *var) {
  Initializer *init = initializer(rest, tok, var->ty);
  InitDesg desg = {NULL, 0, var};

  // If a partial initializer list is given, the standard requires
  // that unspecified elements are set to 0. Here, we simply
  // zero-initialize the entire memory region of a variable before
  // initializing it with user-supplied values.
  Node *lhs = new_node(ND_MEMZERO, tok);
  lhs->var = var;

  Node *rhs = create_lvar_init(init, var->ty, &desg, tok);
  return new_binary(ND_COMMA, lhs, rhs, tok);
}
```

The strategy is *zero everything first, then store the values that were given*. It's wasteful in the common case where the initializer supplies every element, but it's a one-line optimization to skip later (and chibicc never bothers). The comment makes it explicit. The simpler pre-state — every element gets a store — would have required tracking which children of each `Initializer` were actually populated, and emitting zero stores for the rest. Memzero-first sidesteps that entirely.

`create_lvar_init` also has to handle the case where an `Initializer` has no `expr`. Children that the parser never walked into have null `expr` fields; the comma chain skips them with another `ND_NULL_EXPR`:

```diff
+  if (!init->expr)
+    return new_node(ND_NULL_EXPR, tok);
+
   Node *lhs = init_desg_expr(desg, tok);
-  Node *rhs = init->expr;
-  return new_binary(ND_ASSIGN, lhs, rhs, tok);
+  return new_binary(ND_ASSIGN, lhs, init->expr, tok);
```

The combination — `MEMZERO` first, then comma-chain of stores for the populated children — produces correct values whether or not the user supplied every element. Tests:

```c
ASSERT(0, ({ int x[3]={}; x[0]; }));
ASSERT(2, ({ int x[2][3]={{1,2}}; x[0][1]; }));
ASSERT(0, ({ int x[2][3]={{1,2}}; x[1][0]; }));
```

The empty-brace initializer (`int x[3] = {};`) is a chibicc-specific extension — C99 doesn't allow it, but GCC accepts it as a common idiom and chibicc follows. The other two tests pin the partial-row case: `{{1,2}}` fills `x[0][0]` and `x[0][1]`; everything else, including the entirety of `x[1]`, is zero from the `MEMZERO`.

The second commit handles "too many." The previous loop bounded `i` by `init->ty->array_len`, which means an initializer with more elements than the type holds would have left tokens unconsumed and the closing brace would have failed to parse. Rui rewrites the loop to consume *all* elements, but only call `initializer2` on the in-range ones:

```c
static Token *skip_excess_element(Token *tok) {
  if (equal(tok, "{")) {
    tok = skip_excess_element(tok->next);
    return skip(tok, "}");
  }

  assign(&tok, tok);
  return tok;
}
```

`skip_excess_element` parses one full initializer worth of tokens — recursing through brace-pairs — and discards the result. The loop:

```c
for (int i = 0; !consume(rest, tok, "}"); i++) {
  if (i > 0)
    tok = skip(tok, ",");

  if (i < init->ty->array_len)
    initializer2(&tok, tok, init->children[i]);
  else
    tok = skip_excess_element(tok);
}
```

The bound has moved from the for-condition to an `if` inside the body. Tokens past the array's length still get parsed (so they balance braces correctly) but the parsed AST is dropped. The behavior matches GCC's: `int x[2] = {1, 2, 3, 4}` compiles, and `x` ends up as `{1, 2}`.

### Where we are

Array initializers handle both directions of size mismatch. Too few elements: the rest are zeroed via `ND_MEMZERO`. Too many: the excess is parsed for syntactic balance and discarded. The `ND_MEMZERO`-prefix-then-stores pattern is a small but worth-naming optimization-deferral: the sub-optimal "always zero everything, then store some of it" approach is taken because writing a more selective initializer would require tracking which children are populated, and the cost of a `rep stosb` of a few bytes is negligible.

---

## 12.3 — String literal initializers

> `git checkout 0d717373cc9e247fc6f6a0e02b0bbd424f0d70b0` — *Add string literal initializer*

C lets a `char` array be initialized by a string literal: `char s[5] = "hello";` is the same as `char s[5] = {'h', 'e', 'l', 'l', 'o'};` (with no trailing zero, in this case, because the array is too short for one). When the array is longer than the string, the rest is zeroed. This is one of C's stranger surface conveniences — the same syntactic token (a string literal) means an array of bytes in this context and a pointer-to-static-storage in any other context.

Rui handles the case by re-routing inside `initializer2`. Before this commit, an initializer for an array was always a brace-list. After:

```c
// initializer = string-initializer | array-initializer | assign
static void initializer2(Token **rest, Token *tok, Initializer *init) {
  if (init->ty->kind == TY_ARRAY && tok->kind == TK_STR) {
    string_initializer(rest, tok, init);
    return;
  }

  if (init->ty->kind == TY_ARRAY) {
    array_initializer(rest, tok, init);
    return;
  }

  init->expr = assign(rest, tok);
}
```

The dispatch is now a three-way: array-with-string-token, array-without-string-token, and scalar. `array_initializer` is the renamed and refactored version of the previous `if (init->ty->kind == TY_ARRAY)` body. `string_initializer` is new:

```c
// string-initializer = string-literal
static void string_initializer(Token **rest, Token *tok, Initializer *init) {
  int len = MIN(init->ty->array_len, tok->ty->array_len);
  for (int i = 0; i < len; i++)
    init->children[i]->expr = new_num(tok->str[i], tok);
  *rest = tok->next;
}
```

The function takes the per-byte source data from the token (`tok->str` is the parsed string contents, including the null terminator; `tok->ty->array_len` is its length) and writes one `ND_NUM` per byte into the corresponding child Initializer's `expr`. The `MIN` is the asymmetric truncation: if the destination array is shorter than the string, only the destination's length matters (the trailing characters of the string are discarded); if the destination is longer, only the string's length matters (and the §12.2 zero-prefix handles the excess slots).

`MIN` (and `MAX`) are macro definitions added to `chibicc.h`:

```c
#define MAX(x, y) ((x) < (y) ? (y) : (x))
#define MIN(x, y) ((x) < (y) ? (x) : (y))
```

The first uses in the codebase. They will reappear; this is the introduction.

The expansion is, in effect, a syntactic rewrite from string-literal-as-initializer to brace-list-of-bytes — except the rewrite happens inside `string_initializer` rather than at the token level. The user wrote `char x[5] = "abc";`. The parser builds an `Initializer` with five children. Three of the children get scalar `ND_NUM` initializers (`'a'`, `'b'`, `'c'`); the other two stay null and the §12.2 `MEMZERO` clears them to zero. The lowered AST is the comma chain `MEMZERO, x[0]='a', x[1]='b', x[2]='c'` (the unset children's stores are skipped by the §12.2 `if (!init->expr)` arm).

The tests cover the three relevant cases:

```c
ASSERT('a', ({ char x[4]="abc"; x[0]; }));
ASSERT('c', ({ char x[4]="abc"; x[2]; }));
ASSERT(0, ({ char x[4]="abc"; x[3]; }));
ASSERT('a', ({ char x[2][4]={"abc","def"}; x[0][0]; }));
```

The third pins the null terminator: `"abc"` has four bytes (`a`, `b`, `c`, `\0`), the destination has four slots, the `MIN` is four, all four bytes are stored. The fourth pins the nested case — a brace-list of strings, where each child is a `char[4]` initialized by one string. The recursion into `array_initializer` produces two children of type `char[4]`; each child is dispatched on type, sees a `TK_STR` token, and routes through `string_initializer`.

### Where we are

String literals as initializers for `char[]` work, including the truncate-or-zero-fill rules. The dispatch in `initializer2` now distinguishes array-with-string-token from array-with-brace-list at the type-and-token level rather than purely on type. The pattern — desugaring a syntactic shorthand by populating the existing tree, rather than by rewriting tokens — is the mechanism the chapter will reuse for length-omitted arrays in §12.4 and for omitted braces in §12.8.

---

## 12.4 — Length omitted from the array

> `git checkout 5b955336032881edf835a50fb63f9581af1efd73` — *Allow to omit array length if an initializer is given*

This commit closes the §11.9 prediction. Chapter 11 introduced incomplete arrays: `int x[]` parses, the type carries `array_len = -1` and `size = -1` (the §11.9 sentinel), and the variable is rejected at declarator time *unless* an initializer is going to fill in the size. §11.9 left the rejection unconditional, with a forward-reference to "the place this gets relaxed." That place is here.

The rejection used to live inside the per-declarator loop in `declaration`, before the initializer was parsed:

```diff
-    if (ty->size < 0)
-      error_tok(tok, "variable has incomplete type");
     if (ty->kind == TY_VOID)
       error_tok(tok, "variable declared void");

     Obj *var = new_lvar(get_ident(ty->name), ty);
     if (equal(tok, "=")) {
       Node *expr = lvar_initializer(&tok, tok->next, var);
       cur = cur->next = new_unary(ND_EXPR_STMT, expr, tok);
     }
+
+    if (var->ty->size < 0)
+      error_tok(ty->name, "variable has incomplete type");
+    if (var->ty->kind == TY_VOID)
+      error_tok(ty->name, "variable declared void");
   }
```

The check moves from "before the initializer" to "after the initializer." If the initializer ran and patched the variable's type to a now-complete one, the check passes. If it didn't, the check fires and the original error stands. The same structural pattern as Chapter 11's incomplete-struct mutation in §11.9: the type is filled in *after* parsing more tokens, and whoever holds the reference (`var->ty`) now points at the completed type. Chibicc mutates the variable's `ty` pointer rather than the type itself, and the check reads the (possibly-new) `ty` after the initializer has had its chance.

The patching happens in two places. `string_initializer` and `array_initializer1` (renamed from `array_initializer` in §12.8 — at this commit it's still `array_initializer`) both check whether the Initializer was created with a flexible flag, and if so, replace it in place:

```c
static void string_initializer(Token **rest, Token *tok, Initializer *init) {
  if (init->is_flexible)
    *init = *new_initializer(array_of(init->ty->base, tok->ty->array_len), false);
  ...
}
```

For strings, the size is the string's length. For brace-lists, a separate pass counts the elements first:

```c
static int count_array_init_elements(Token *tok, Type *ty) {
  Initializer *dummy = new_initializer(ty->base, false);
  int i = 0;

  for (; !equal(tok, "}"); i++) {
    if (i > 0)
      tok = skip(tok, ",");
    initializer2(&tok, tok, dummy);
  }
  return i;
}
```

The dummy-init pre-pass parses each element into a throwaway Initializer just to advance the token cursor, then returns the count. The real parse follows. That's a deliberate choice: the parser needs to know the array length to allocate the children array, but it can't know the length without scanning the elements. Two passes are simpler than a growable children array.

The `is_flexible` flag itself is added to the `Initializer` struct:

```diff
 struct Initializer {
   Initializer *next;
   Type *ty;
   Token *tok;
+  bool is_flexible;
   ...
```

It's set by `new_initializer` when the type has `size < 0`:

```c
static Initializer *new_initializer(Type *ty, bool is_flexible) {
  Initializer *init = calloc(1, sizeof(Initializer));
  init->ty = ty;

  if (ty->kind == TY_ARRAY) {
    if (is_flexible && ty->size < 0) {
      init->is_flexible = true;
      return init;
    }
    ...
  }
  ...
}
```

Note the early return: a flexible Initializer doesn't get a children array allocated. It's a placeholder, waiting for the parser to discover the real length and replace it via `*init = *new_initializer(real_type, false)`.

The top-level `initializer` function grows a `new_ty` out-parameter so the caller can read out the (possibly-patched) type:

```c
static Initializer *initializer(Token **rest, Token *tok, Type *ty, Type **new_ty) {
  Initializer *init = new_initializer(ty, true);
  initializer2(rest, tok, init);
  *new_ty = init->ty;
  return init;
}
```

`lvar_initializer` passes `&var->ty` so the variable's type pointer gets updated in place:

```c
static Node *lvar_initializer(Token **rest, Token *tok, Obj *var) {
  Initializer *init = initializer(rest, tok, var->ty, &var->ty);
  ...
}
```

The chain is: `declaration` makes a `var` with `ty->size = -1`; `lvar_initializer` calls `initializer` and passes `&var->ty`; `initializer` sets `*new_ty = init->ty`, which is the patched-in-place type with the real size; `var->ty` now points at the completed type; the `var->ty->size < 0` check after `lvar_initializer` returns sees a positive size and doesn't fire.

The tests:

```c
ASSERT(4, ({ int x[]={1,2,3,4}; x[3]; }));
ASSERT(16, ({ int x[]={1,2,3,4}; sizeof(x); }));
ASSERT(4, ({ char x[]="foo"; sizeof(x); }));

ASSERT(4, ({ typedef char T[]; T x="foo"; T y="x"; sizeof(x); }));
ASSERT(2, ({ typedef char T[]; T x="foo"; T y="x"; sizeof(y); }));
```

The last two are subtle. They typedef an incomplete-array type, then declare two variables with the same typedef but different initializers. Each variable's `ty` is independently patched; the typedef is a *template*, not a shared type. The chibicc copy-on-patch behavior — the variable's `ty` pointer gets a fresh `Type` allocated via `array_of`, rather than mutating the type in place — is what makes that work. (If chibicc had mutated the typedef's underlying `Type`, both variables would share the most-recently-patched size. The §11.9 struct-forward-decl mutation pattern, which *does* mutate in place, would have been the wrong shape here.)

### Where we are

The §11.9 `array_of(ty, -1)` sentinel has its first real consumer. Incomplete-array variables with initializers are now valid; the size is patched in from the initializer. The `is_flexible` Initializer flag is the channel that carries "this is a placeholder; please replace me" through the recursive parser. The two-pass count-then-parse strategy in `count_array_init_elements` is small but explicit; chibicc never bothers with a growable child array.

---

## 12.5 — Struct and union local initializers

> `git checkout e9d2c46ab3cc8b8518df289a4fc24a9e3fc9b3fe` — *Handle struct initializers for local variables*
>
> `git checkout aca19dd35027a12e245bfa52e6a98968e0cd2a9c` — *Allow to initialize a struct with other struct*
>
> `git checkout 483b194a80e904c11c5c6d855303596145adacee` — *Handle union initializers for local variables*

Three commits extend the Initializer machinery to aggregates. The structure is the same as §12.1 — parse into a tree, lower to a comma chain — but the parallel between Initializer shape and type shape now has to handle non-array aggregates.

The first commit handles `struct point p = {1, 2};`. `new_initializer` grows a `TY_STRUCT` arm:

```c
if (ty->kind == TY_STRUCT) {
  // Count the number of struct members.
  int len = 0;
  for (Member *mem = ty->members; mem; mem = mem->next)
    len++;

  init->children = calloc(len, sizeof(Initializer *));

  for (Member *mem = ty->members; mem; mem = mem->next)
    init->children[mem->idx] = new_initializer(mem->ty, false);
  return init;
}
```

The shape is the same as the array arm: allocate a children array, populate one Initializer per child. The wrinkle is that struct members are stored as a linked list (`ty->members` chains forward via `next`) but the Initializer's children are an array. The bridge is a new `idx` field on `Member`:

```diff
 struct Member {
   Type *ty;
   Token *tok;
   Token *name;
+  int idx;
   int offset;
 };
```

The `idx` is set when struct members are parsed:

```c
static void struct_members(Token **rest, Token *tok, Type *ty) {
  Member head = {};
  Member *cur = &head;
  int idx = 0;

  while (!equal(tok, "}")) {
    Type *basety = declspec(&tok, tok, NULL);
    bool first = true;

    while (!consume(&tok, tok, ";")) {
      if (!first)
        tok = skip(tok, ",");
      first = false;

      Member *mem = calloc(1, sizeof(Member));
      mem->ty = declarator(&tok, tok, basety);
      mem->name = mem->ty->name;
      mem->idx = idx++;
      cur = cur->next = mem;
    }
  }
```

A linear counter, assigned in declaration order. The same `idx` is used to look up the member's slot in the Initializer's children array.

The struct-shaped parser:

```c
// struct-initializer = "{" initializer ("," initializer)* "}"
static void struct_initializer(Token **rest, Token *tok, Initializer *init) {
  tok = skip(tok, "{");

  Member *mem = init->ty->members;

  while (!consume(rest, tok, "}")) {
    if (mem != init->ty->members)
      tok = skip(tok, ",");

    if (mem) {
      initializer2(&tok, tok, init->children[mem->idx]);
      mem = mem->next;
    } else {
      tok = skip_excess_element(tok);
    }
  }
}
```

A walk down the member list in lockstep with the brace-list. Each iteration consumes one element. If the member chain ran out before the `}`, the excess is skipped via the §12.2 helper.

`initializer2` gets one more arm:

```c
if (init->ty->kind == TY_STRUCT) {
  struct_initializer(rest, tok, init);
  return;
}
```

`init_desg_expr` and `create_lvar_init` get parallel struct arms. The designator stack grows a `member` field:

```diff
 struct InitDesg {
   InitDesg *next;
   int idx;
+  Member *member;
   Obj *var;
 };
```

The member-aware `init_desg_expr`:

```c
static Node *init_desg_expr(InitDesg *desg, Token *tok) {
  if (desg->var)
    return new_var_node(desg->var, tok);

  if (desg->member) {
    Node *node = new_unary(ND_MEMBER, init_desg_expr(desg->next, tok), tok);
    node->member = desg->member;
    return node;
  }

  Node *lhs = init_desg_expr(desg->next, tok);
  Node *rhs = new_num(desg->idx, tok);
  return new_unary(ND_DEREF, new_add(lhs, rhs, tok), tok);
}
```

Three frames: variable (the base), member (`p.x`), array index (`p.x[0]`). The recursion builds the expression bottom-up. A struct-of-array initializer like `struct s p = { {1,2}, 3 };` produces designators like `{var=p, member=arr, idx=0}` and lowers to `*((p.arr) + 0) = 1`, which subsequent type-checking resolves to `p.arr[0] = 1`.

`create_lvar_init` adds a struct arm:

```c
if (ty->kind == TY_STRUCT) {
  Node *node = new_node(ND_NULL_EXPR, tok);

  for (Member *mem = ty->members; mem; mem = mem->next) {
    InitDesg desg2 = {desg, 0, mem};
    Node *rhs = create_lvar_init(init->children[mem->idx], mem->ty, &desg2, tok);
    node = new_binary(ND_COMMA, node, rhs, tok);
  }
  return node;
}
```

Same shape as the array arm: an `ND_NULL_EXPR` seed, and a comma chain accumulated by walking the children. The base case at the leaf is one `ND_ASSIGN` per scalar member.

The second commit (`aca19dd`) adds one specific case the brace-list grammar doesn't cover: `struct T x = y;`, where `y` is an existing variable of struct type. There's no brace, so the type-driven dispatch in `initializer2` shouldn't have routed it to `struct_initializer`. The fix is to peek at the lookahead before recursing:

```c
if (init->ty->kind == TY_STRUCT) {
  // A struct can be initialized with another struct. E.g.
  // `struct T x = y;` where y is a variable of type `struct T`.
  // Handle that case first.
  if (!equal(tok, "{")) {
    Node *expr = assign(rest, tok);
    add_type(expr);
    if (expr->ty->kind == TY_STRUCT) {
      init->expr = expr;
      return;
    }
  }

  struct_initializer(rest, tok, init);
  return;
}
```

If the next token isn't `{`, parse the right-hand side as a regular expression and (if it has struct type) save it as a top-level `expr` instead of recursing into children. The lowering side has to learn to use `init->expr` as a struct-valued right-hand side rather than walking children:

```diff
-  if (ty->kind == TY_STRUCT) {
+  if (ty->kind == TY_STRUCT && !init->expr) {
     Node *node = new_node(ND_NULL_EXPR, tok);
     ...
```

When `init->expr` is set, the existing scalar fall-through emits `ND_ASSIGN` of the whole struct — which works because chibicc's struct assignment, since Chapter 9, is one node kind that copies the whole struct. The mechanism the chapter inherits.

The third commit (`483b194`) adds union initializers. Unions take only one initializer (the first member), and the parser is correspondingly simpler:

```c
static void union_initializer(Token **rest, Token *tok, Initializer *init) {
  // Unlike structs, union initializers take only one initializer,
  // and that initializes the first union member.
  tok = skip(tok, "{");
  initializer2(&tok, tok, init->children[0]);
  *rest = skip(tok, "}");
}
```

`new_initializer` extends to handle unions identically to structs (a children array, one slot per member, but only `children[0]` will be consulted):

```diff
-  if (ty->kind == TY_STRUCT) {
+  if (ty->kind == TY_STRUCT || ty->kind == TY_UNION) {
```

`create_lvar_init` adds a union arm that walks only the first member:

```c
if (ty->kind == TY_UNION) {
  InitDesg desg2 = {desg, 0, ty->members};
  return create_lvar_init(init->children[0], ty->members->ty, &desg2, tok);
}
```

Tests cover the type-puns that unions enable:

```c
ASSERT(4, ({ union { int a; char b[4]; } x={0x01020304}; x.b[0]; }));
ASSERT(3, ({ union { int a; char b[4]; } x={0x01020304}; x.b[1]; }));
ASSERT(0x01020304, ({ union { struct { char a,b,c,d; } e; int f; } x={{4,3,2,1}}; x.f; }));
```

The first two pin the byte order: storing `0x01020304` as an `int` lays out bytes `04 03 02 01` on little-endian x86, which is what `b[0]` and `b[1]` read back. The third pins the recursive case — a union whose first member is a struct, initialized with a brace-list that opens that struct.

### Where we are

Local initializers handle scalars, arrays, strings, structs (including struct-from-struct), and unions. The `Initializer` tree mirrors the type for all four aggregate shapes; `create_lvar_init` walks the type and Initializer in lockstep producing a comma chain. The designator stack tracks "where in the variable" via three frame kinds (variable base, member, array index). Globals still don't accept anything beyond the single-scalar shape from Chapter 7.

---

## 12.6 — Global scalar and string initializers

> `git checkout bbfe3f4369e1dd2266b827c81d7d9078ab1d301f` — *Add global initializer for scalar and string*

This commit names *the chapter's central tension*. A local initializer can be lowered to a sequence of assignments because the variable lives in stack memory and stack memory isn't initialized at program load time. A global initializer can't be — globals are placed in static storage at link time, and their initial values have to be encoded in the assembler output as `.byte` directives, not as assignments. The two paths share the front end (the `Initializer` tree, the parser predicate) and diverge at the back.

The split adds a `gvar_initializer`, mirroring `lvar_initializer`:

```c
// Initializers for global variables are evaluated at compile-time and
// embedded to .data section. This function serializes Initializer
// objects to a flat byte array. It is a compile error if an
// initializer list contains a non-constant expression.
static void gvar_initializer(Token **rest, Token *tok, Obj *var) {
  Initializer *init = initializer(rest, tok, var->ty, &var->ty);

  char *buf = calloc(1, var->ty->size);
  write_gvar_data(init, var->ty, buf, 0);
  var->init_data = buf;
}
```

The structure is the same: parse into an Initializer tree, then walk the tree. The walk is different. Where `create_lvar_init` produces an AST node, `write_gvar_data` writes bytes directly into a buffer. The buffer is stored as `var->init_data`, which the existing Chapter 7 codegen already emits one `.byte` at a time.

The walker:

```c
static void write_gvar_data(Initializer *init, Type *ty, char *buf, int offset) {
  if (ty->kind == TY_ARRAY) {
    int sz = ty->base->size;
    for (int i = 0; i < ty->array_len; i++)
      write_gvar_data(init->children[i], ty->base, buf, offset + sz * i);
    return;
  }

  if (init->expr)
    write_buf(buf + offset, eval(init->expr), ty->size);
}
```

Two arms. Arrays recurse, advancing `offset` by the per-element size. Scalars evaluate the initializer expression to a constant and write the bytes via `write_buf`:

```c
static void write_buf(char *buf, uint64_t val, int sz) {
  if (sz == 1)
    *buf = val;
  else if (sz == 2)
    *(uint16_t *)buf = val;
  else if (sz == 4)
    *(uint32_t *)buf = val;
  else if (sz == 8)
    *(uint64_t *)buf = val;
  else
    unreachable();
}
```

Casting through pointer-to-sized-integer to write a value of the right width. A `char` initialization writes one byte; a `long` writes eight. This is *the* place in the chapter where the host machine's byte order leaks into chibicc's output: `*(uint32_t *)buf = val` lays bytes down in host-endian order, and chibicc and its target both happen to be little-endian x86-64. There is no portability story here; chibicc compiles x86-64 on x86-64.

The constant-folding side is the §11.15 `eval`, called for the first time outside of `const_expr`. The forward-reference from §11.15 closes here. `eval(init->expr)` for `int x = 1+2;` walks the AST and returns `3`; `write_buf` writes that `3` into the four bytes for the global. `eval` for `int x = i;` (where `i` is a non-constant expression) crashes through the `error_tok(node->tok, "not a compile-time constant")` arm, which is the standard's required behavior for a global initializer.

The wiring at `global_variable`:

```diff
     Type *ty = declarator(&tok, tok, basety);
-    new_gvar(get_ident(ty->name), ty);
+    Obj *var = new_gvar(get_ident(ty->name), ty);
+    if (equal(tok, "="))
+      gvar_initializer(&tok, tok->next, var);
   }
```

Same shape as the local case in `declaration`: declarator, then optional `=` then initializer.

The strings-as-array-initializers case from §12.3 also works for globals, because the `Initializer` tree shape is the same. `string_initializer` populates the children with `ND_NUM` nodes; `write_gvar_data` recurses into the array and `eval` on each `ND_NUM` returns the byte. The same Initializer tree feeds both ends of the local-versus-global split.

A note on what isn't handled yet. This commit supports scalars, arrays, and strings. Structs and unions don't have a global path until §12.7. Pointer-valued initializers (`int *p = &x;`) don't work either; `eval` will reject them as non-constant. Both are added in §12.7.

The tests are minimal — four global scalars at file scope:

```c
char g3 = 3;
short g4 = 4;
int g5 = 5;
long g6 = 6;
```

with corresponding `ASSERT(3, g3);` etc. inside `main`. The minimal coverage matches the minimal commit.

### Where we are

Globals can have initializers for scalars, arrays, and strings. The initializer pipeline forks at `lvar_initializer`-vs-`gvar_initializer`; everything before the fork (parsing into the Initializer tree, type-driven dispatch in `initializer2`) is shared. The `eval` function from §11.15 has its first non-test caller. The split is named: locals lower to assignments at runtime; globals serialize to a byte buffer at compile time.

---

## 12.7 — Global struct, union, and pointer initializers

> `git checkout eeb62b6dd547da5742f3ed74f8c8ae534d883dd9` — *Add struct initializer for global variable*
>
> `git checkout 1eae5ae3678d079efc7d2807f10439e53932f811` — *Handle union initializers for global variable*

Two commits extend the global path to aggregates. The first is small (six lines): a `TY_STRUCT` arm in `write_gvar_data`, parallel to the array arm:

```c
if (ty->kind == TY_STRUCT) {
  for (Member *mem = ty->members; mem; mem = mem->next)
    write_gvar_data(init->children[mem->idx], mem->ty, buf, offset + mem->offset);
  return;
}
```

Walk the member list, recurse into each child, advance `offset` by the member's offset (which the §9.x struct-layout pass already filled in). Six lines because the only real work is "use `mem->offset` for the offset," and the surrounding shape was already established in §12.6.

Tests pin the layout:

```c
int g9[3] = {0, 1, 2};
struct {char a; int b;} g11[2] = {{1, 2}, {3, 4}};
struct {int a[2];} g12[2] = {{{1, 2}}};
```

The second test is interesting: `{char a; int b;}` is eight bytes wide because `b` is at offset 4 (the int requires four-byte alignment). `g11` is sixteen bytes total. The byte buffer that `write_gvar_data` produces has `01 00 00 00 02 00 00 00 03 00 00 00 04 00 00 00` — `01` for `g11[0].a` at offset 0, `02` for `g11[0].b` at offset 4 (with three padding bytes), and so on. The layout is correct because `mem->offset` is correct.

The second commit (`1eae5ae`) handles union initializers and, more substantively, *pointer* initializers. The union arm is small:

```c
if (ty->kind == TY_UNION)
  return write_gvar_data(cur, init->children[0], ty->members->ty, buf, offset);
```

Parallel to the local union case: write only the first member. The pointer arm is the chapter's most involved single addition.

A global initializer can name another global by address: `int *p = &x;`. The address of `x` isn't known at compile time — it's a link-time constant the assembler emits as a symbol reference. The `.data` section can't carry a literal byte sequence for that; it has to carry a relocatable directive (`.quad x`) that the assembler resolves later. Chibicc adds a `Relocation` struct to `Obj` and a parallel relocation list to the byte buffer:

```c
// Global variable can be initialized either by a constant expression
// or a pointer to another global variable. This struct represents the
// latter.
typedef struct Relocation Relocation;
struct Relocation {
  Relocation *next;
  int offset;
  char *label;
  long addend;
};
```

Each `Relocation` says: at byte `offset` in this global's data, instead of the literal bytes from `init_data`, emit a reference to symbol `label` plus `addend`. The codegen emits these alongside the data:

```c
if (var->init_data) {
  Relocation *rel = var->rel;
  int pos = 0;
  while (pos < var->ty->size) {
    if (rel && rel->offset == pos) {
      println("  .quad %s%+ld", rel->label, rel->addend);
      rel = rel->next;
      pos += 8;
    } else {
      println("  .byte %d", var->init_data[pos++]);
    }
  }
}
```

The walk is by byte; relocations punch eight-byte holes that get a `.quad symbol+addend` instead of `.byte`s. The relocation list is sorted by `offset` (because it's built in source order, which is the same order `write_gvar_data` visits memory in), so a single linear walk suffices.

The constant-folder is what produces relocations. `eval` was, before this commit, "evaluate to a 64-bit integer or fail." It needs to extend to "evaluate to a 64-bit integer plus optionally a symbol reference." Rui splits the function in two:

```c
static int64_t eval(Node *node) {
  return eval2(node, NULL);
}

// Evaluate a given node as a constant expression.
//
// A constant expression is either just a number or ptr+n where ptr
// is a pointer to a global variable and n is a postiive/negative
// number. The latter form is accepted only as an initialization
// expression for a global variable.
static int64_t eval2(Node *node, char **label) {
```

`eval2` takes an out-parameter `label` that the relocatable cases write to. The integer-returning cases — `ND_ADD`, `ND_SUB`, etc. — pass `label` through to one operand only (the left), so a relocation can flow up the AST through `ptr + offset`-shaped expressions but not through `n * ptr`-shaped ones (a multiply by a pointer is undefined). The pointer-producing cases — `ND_ADDR`, `ND_VAR`, `ND_MEMBER` — write to `*label`:

```c
case ND_ADDR:
  return eval_rval(node->lhs, label);
case ND_MEMBER:
  if (!label)
    error_tok(node->tok, "not a compile-time constant");
  if (node->ty->kind != TY_ARRAY)
    error_tok(node->tok, "invalid initializer");
  return eval_rval(node->lhs, label) + node->member->offset;
case ND_VAR:
  if (!label)
    error_tok(node->tok, "not a compile-time constant");
  if (node->var->ty->kind != TY_ARRAY && node->var->ty->kind != TY_FUNC)
    error_tok(node->tok, "invalid initializer");
  *label = node->var->name;
  return 0;
```

`eval_rval` is the lvalue-evaluator — it handles the `&x` and `&x.member` cases by walking variables, dereferences, and member accesses:

```c
static int64_t eval_rval(Node *node, char **label) {
  switch (node->kind) {
  case ND_VAR:
    if (node->var->is_local)
      error_tok(node->tok, "not a compile-time constant");
    *label = node->var->name;
    return 0;
  case ND_DEREF:
    return eval2(node->lhs, label);
  case ND_MEMBER:
    return eval_rval(node->lhs, label) + node->member->offset;
  }

  error_tok(node->tok, "invalid initializer");
}
```

A local variable's address isn't a link-time constant (locals don't have static storage), so `is_local` is rejected. A struct member's address is the base address plus the member's offset. A dereferenced address (`*&x`) is just the value of the inner expression.

The arms in `eval2` that pass `label` through to the left operand are the relocation-flow cases:

```c
case ND_ADD:
  return eval2(node->lhs, label) + eval(node->rhs);
case ND_SUB:
  return eval2(node->lhs, label) - eval(node->rhs);
```

`a + b` where `a` is a relocatable address and `b` is a constant integer produces a relocation with `addend = b`. The integer-arms use plain `eval` (which passes `NULL` and rejects pointer expressions), so `&x * 3` is a compile error.

The `gvar_initializer` wrapper threads a relocation list through:

```c
static void gvar_initializer(Token **rest, Token *tok, Obj *var) {
  Initializer *init = initializer(rest, tok, var->ty, &var->ty);

  Relocation head = {};
  char *buf = calloc(1, var->ty->size);
  write_gvar_data(&head, init, var->ty, buf, 0);
  var->init_data = buf;
  var->rel = head.next;
}
```

A linked list, dummy-head pattern, threaded through `write_gvar_data`'s scalar arm:

```c
char *label = NULL;
uint64_t val = eval2(init->expr, &label);

if (!label) {
  write_buf(buf + offset, val, ty->size);
  return cur;
}

Relocation *rel = calloc(1, sizeof(Relocation));
rel->offset = offset;
rel->label = label;
rel->addend = val;
cur->next = rel;
return cur->next;
```

If the initializer evaluated to a plain integer, write the bytes. If it evaluated to a label-plus-addend, append a relocation instead. Either way, the output is a byte buffer plus a list of relocations; the codegen merges them at emit time.

Tests for the pointer-initializer side are extensive:

```c
char g17[] = "foobar";
char *g20 = g17+0;
char *g21 = g17+3;
char *g22 = &g17-3;
int g26[3] = {1, 2, 3};
int *g27 = g26 + 1;
int *g28 = &g11[1].a;
long g29 = (long)(long)g26;
```

`g20` through `g23` exercise the `array + integer → relocatable pointer` path. `g28` exercises the `address-of-struct-member` path through `eval_rval`. `g29` exercises the cast-then-cast-back path (the result is a relocatable address, even after two `(long)` casts that the standard says don't change anything).

### Where we are

Globals can be initialized with anything an initializer-list can hold: scalars, arrays, strings, structs, unions, and addresses of other globals. The constant-expression evaluator splits into `eval` (integer-only) and `eval2` (integer-plus-relocation). Relocations are emitted as `.quad symbol+addend` directives instead of `.byte`s. The split between local and global initializers is now complete on both sides; the parser is shared, the lowering diverges.

---

## 12.8 — Initializer-list ergonomics

> `git checkout efa0f3366ddb914cc29f96fcdf10f99ded61775c` — *Allow parentheses in initializers to be omitted*
>
> `git checkout a58958ccb40a127a83e3383ef3887e4721352238` — *Allow extraneous braces for scalar initializer*
>
> `git checkout fde464c47cb69e030b58d8d204a508d6babd3e09` — *Allow extraneous comma at the end of enum or initializer list*

Three small commits that smooth out C's initializer-list grammar. None of them adds new functionality in any deep sense; they let the parser accept syntactic shapes that were previously rejected but that real C programs use.

The first commit — *omit parentheses in initializers* — is more accurately about omitting *braces*. C lets `int x[2][3] = {1, 2, 3, 4, 5, 6}` mean the same as `int x[2][3] = {{1, 2, 3}, {4, 5, 6}}`: the inner braces are optional, and the parser is supposed to greedily fill the elements left-to-right by walking the type. Same for structs: `struct {int a; int b;} x[2] = {1, 2, 3, 4}` works.

The implementation splits each aggregate parser into two variants, *with-braces* and *without-braces*:

```c
// array-initializer1 = "{" initializer ("," initializer)* "}"
static void array_initializer1(Token **rest, Token *tok, Initializer *init) {
  tok = skip(tok, "{");
  ...
}

// array-initializer2 = initializer ("," initializer)*
static void array_initializer2(Token **rest, Token *tok, Initializer *init) {
  ...
  for (int i = 0; i < init->ty->array_len && !equal(tok, "}"); i++) {
    if (i > 0)
      tok = skip(tok, ",");
    initializer2(&tok, tok, init->children[i]);
  }
  *rest = tok;
}
```

The `2` variant doesn't expect or skip `{` or `}`. It just consumes `array_len` initializers separated by commas, stopping when it hits a `}` (which belongs to an enclosing initializer-list it shouldn't consume) or runs out of room.

The dispatch in `initializer2` peeks at the next token to choose:

```c
if (init->ty->kind == TY_ARRAY) {
  if (equal(tok, "{"))
    array_initializer1(rest, tok, init);
  else
    array_initializer2(rest, tok, init);
  return;
}
```

The same split for `struct_initializer1`/`struct_initializer2` and a similar conditional for `union_initializer`. The result: an outer `{ … }` brackets the whole initializer for the outermost aggregate; inner braces are optional and the parser greedily consumes elements when they're absent.

A test that pins the recursive case:

```c
struct {int a[2];} g40[2] = {{1, 2}, {3, 4}};
struct {int a[2];} g41[2] = {1, 2, 3, 4};
```

Both produce the same layout. `g40` has braces all the way down; `g41` has only the outer brace. Each access (`g40[0].a[0]` through `g41[1].a[1]`) returns the same value across the two definitions.

The second commit — *extraneous braces for scalar initializer* — handles the converse: `int x = {3};`. C lets a scalar initializer be wrapped in superfluous braces, equivalent to writing `int x = 3;`. The fix is one stanza in `initializer2`:

```c
if (equal(tok, "{")) {
  // An initializer for a scalar variable can be surrounded by
  // braces. E.g. `int x = {3};`. Handle that case.
  initializer2(&tok, tok->next, init);
  *rest = skip(tok, "}");
  return;
}
```

Recurse into the same `initializer2` (now with the brace consumed), then expect a closing `}`. Subtle interaction with the `{}` empty-init case from §12.2: an empty brace pair on a scalar would now recurse with no expression to parse, which would be an error — but the §12.2 code doesn't actually allow `int x = {};` (the empty-list case only fires inside an aggregate). The combination is consistent.

The test is small:

```c
char *g44 = {"foo"};
```

A pointer initialized to a string literal, wrapped in braces. The chibicc-specific case — string literal as scalar initializer for a pointer — works because `initializer2` for a pointer falls through to the scalar arm, which now strips the braces and parses the string as an expression.

The third commit — *extraneous comma at the end of enum or initializer list* — is the small ergonomic detail that C allows `{1, 2, 3,}` (trailing comma) and `enum { A, B, C, }` (trailing comma in an enum). Real C source uses this constantly to keep diffs clean. The fix is two helpers:

```c
static bool is_end(Token *tok) {
  return equal(tok, "}") || (equal(tok, ",") && equal(tok->next, "}"));
}

static bool consume_end(Token **rest, Token *tok) {
  if (equal(tok, "}")) {
    *rest = tok->next;
    return true;
  }

  if (equal(tok, ",") && equal(tok->next, "}")) {
    *rest = tok->next->next;
    return true;
  }

  return false;
}
```

`is_end` is a peek; `consume_end` is a peek-and-skip. Each loop in `array_initializer1`, `struct_initializer1`, `count_array_init_elements`, and the enum-list parser swaps `equal(tok, "}")` for `is_end(tok)` and `consume(rest, tok, "}")` for `consume_end(rest, tok)`. The trailing comma is now silently absorbed by the loop's exit condition.

Tests are minimal — one each:

```c
ASSERT(1, ({ int x[3]={1,2,3,}; x[0]; }));
ASSERT(1, ({ union {int a; char b;} x={1,}; x.a; }));
ASSERT(2, ({ enum {x,y,z,}; z; }));
```

### Where we are

Initializer-list parsing handles the three corner-cases that real C source uses: optional inner braces in nested aggregates, redundant outer braces around scalars, and trailing commas. The pattern of *splitting a parser into two variants and dispatching on the lookahead token* repeats from §12.5's struct-from-struct case — peek, dispatch, recurse. None of it changes what initializers do, only what spellings the parser will accept.

---

## 12.9 — Uninitialized globals to `.bss`

> `git checkout 3d216e3e06eee7ea3679503867a619c28458e8a7` — *Emit uninitialized global data to .bss instead of .data*

A small commit, structurally isolated from the rest of the chapter. Until now, every global variable — initialized or not — was emitted into `.data`:

```c
println("  .data");
println("  .globl %s", var->name);
println("%s:", var->name);

if (var->init_data) {
  for (int i = 0; i < var->ty->size; i++)
    println("  .byte %d", var->init_data[i]);
} else {
  println("  .zero %d", var->ty->size);
}
```

The `.zero N` directive in `.data` reserves N bytes initialized to zero. That's correct — but wasteful. An uninitialized global doesn't need to occupy bytes in the object file at all; it can be declared in `.bss` (the *block started by symbol* section), which the loader maps to zero-filled memory at process start. The size of the program's binary shrinks by exactly the size of all uninitialized globals.

The fix splits the emission by whether the global has data:

```c
println("  .globl %s", var->name);

if (var->init_data) {
  println("  .data");
  println("%s:", var->name);

  Relocation *rel = var->rel;
  int pos = 0;
  while (pos < var->ty->size) {
    if (rel && rel->offset == pos) {
      println("  .quad %s%+ld", rel->label, rel->addend);
      rel = rel->next;
      pos += 8;
    } else {
      println("  .byte %d", var->init_data[pos++]);
    }
  }
  continue;
}

println("  .bss");
println("%s:", var->name);
println("  .zero %d", var->ty->size);
```

The `.globl` directive moves above the `.data`/`.bss` decision; everything else follows the dispatch. There's no test — the change is invisible from C, only visible in the assembler output — but the behavior is what every real C compiler does.

This is the first time chibicc emits `.bss`. Until this commit, the symbol-section list was just `.text` and `.data`.

### Where we are

Uninitialized globals occupy zero bytes in the object file. The dispatch in `emit_data` is now three-way: `.data` for initialized globals, `.bss` for uninitialized globals, `.text` for functions (handled elsewhere). The change is purely an output-format optimization; nothing in the front end changes.

---

## 12.10 — Flexible array members

> `git checkout 824543bb2f2b2e4f445d8c58b32f53bf1eec63ce` — *Add flexible array member*
>
> `git checkout cd688a89b8a57e9614f278e29a9267709494d236` — *Allow to initialize struct flexible array member*

C99 added the *flexible array member*: a struct's last member can be declared as an array of unspecified length, and `sizeof(struct s)` ignores it. The construct is `struct s { int n; int data[]; };` — a header with a count and a trailing array, allocated with one `malloc` of `sizeof(struct s) + n * sizeof(int)`. Common in protocols and in containers that want to keep the data inline.

The first commit detects the construct in `struct_members` and rewrites the trailing array to size zero:

```c
// If the last element is an array of incomplete type, it's
// called a "flexible array member". It should behave as if
// if were a zero-sized array.
if (cur != &head && cur->ty->kind == TY_ARRAY && cur->ty->array_len < 0)
  cur->ty = array_of(cur->ty->base, 0);
```

The §11.9 incomplete-array sentinel (`array_len < 0`) is the trigger. The §11.9 mechanism rejected such types as variable types unless an initializer rescued them; in struct-member position, instead, the type is silently coerced to a zero-length array. From there, the rest of the struct's size calculation works unchanged: a zero-length array adds zero to the struct's size, and the trailing member can be indexed at runtime to access whatever follows.

The test:

```c
ASSERT(4, sizeof(struct { int x, y[]; }));
```

`x` is four bytes; `y[]` is zero bytes; the struct is four bytes. Standard C says it's *implementation-defined* whether the trailing array's size is included in `sizeof`; chibicc and GCC both say no.

The second commit lets a flexible array be initialized along with the rest of the struct. The trick is that the struct's size *grows* when initialized — `T65 g65 = {'f','o','o',0};` for `typedef struct { char a, b[]; } T65` produces a four-byte struct, and `T65 g66 = {'f','o','o','b','a','r',0};` produces a seven-byte struct. The initializer determines the size of the trailing array, and so the size of the whole struct.

`Type` grows an `is_flexible` flag (distinct from the `Initializer`'s `is_flexible` flag from §12.4):

```diff
   // Struct
   Member *members;
+  bool is_flexible;
```

set in `struct_members`:

```diff
-  if (cur != &head && cur->ty->kind == TY_ARRAY && cur->ty->array_len < 0)
+  if (cur != &head && cur->ty->kind == TY_ARRAY && cur->ty->array_len < 0) {
     cur->ty = array_of(cur->ty->base, 0);
+    ty->is_flexible = true;
+  }
```

`new_initializer` learns to mark the last member of a flexible struct as a flexible Initializer (the same mechanism §12.4 uses for top-level flexible arrays):

```c
for (Member *mem = ty->members; mem; mem = mem->next) {
  if (is_flexible && ty->is_flexible && !mem->next) {
    Initializer *child = calloc(1, sizeof(Initializer));
    child->ty = mem->ty;
    child->is_flexible = true;
    init->children[mem->idx] = child;
  } else {
    init->children[mem->idx] = new_initializer(mem->ty, false);
  }
}
```

The `!mem->next` check identifies the last member. The flexibility propagates: the struct itself is flexible (`ty->is_flexible`), the user passed `is_flexible = true` to `new_initializer`, and the last member's Initializer becomes flexible. When `string_initializer` or `array_initializer` walks into that child, it patches the child's `ty` to the discovered length — the same mechanism §12.4 uses for top-level flexible arrays, now reused.

The struct's outer size is patched after the initializer is parsed:

```c
if ((ty->kind == TY_STRUCT || ty->kind == TY_UNION) && ty->is_flexible) {
  ty = copy_struct_type(ty);

  Member *mem = ty->members;
  while (mem->next)
    mem = mem->next;
  mem->ty = init->children[mem->idx]->ty;
  ty->size += mem->ty->size;

  *new_ty = ty;
  return init;
}
```

Two operations: walk the member chain to the last one, copy in the patched type from the corresponding Initializer child, and add the new size. The `copy_struct_type` helper deep-copies the struct type and its member list:

```c
static Type *copy_struct_type(Type *ty) {
  ty = copy_type(ty);

  Member head = {};
  Member *cur = &head;
  for (Member *mem = ty->members; mem; mem = mem->next) {
    Member *m = calloc(1, sizeof(Member));
    *m = *mem;
    cur = cur->next = m;
  }

  ty->members = head.next;
  return ty;
}
```

The deep copy is necessary because two different variables of the same flexible-struct typedef can have different trailing-array sizes. The same reason §12.4's typedef-shared-template case worked: the patched type belongs to the variable, not to the typedef.

Tests:

```c
typedef char T60[];
T60 g60 = {1, 2, 3};
T60 g61 = {1, 2, 3, 4, 5, 6};

typedef struct { char a, b[]; } T65;
T65 g65 = {'f','o','o',0};
T65 g66 = {'f','o','o','b','a','r',0};
```

The first pair re-tests the §12.4 typedef-of-incomplete-array case. The second pair tests the new flexible-struct case: `g65` is four bytes (`a` + three-byte `b`), `g66` is seven bytes (`a` + six-byte `b`).

### Where we are

Flexible array members work in struct definitions and in struct initializations. The §11.9 incomplete-array machinery (the `array_len < 0` sentinel) is reused once more — its third consumer, after §12.4's length-omitted arrays and the existing struct-forward-declaration mechanism. The `is_flexible` flag lives on both `Type` (the struct as declared) and `Initializer` (the slot waiting for its size). The mechanism mirrors §12.4 in shape; the trailing-member-only restriction is enforced by the `!mem->next` check.

---

## 12.11 — `void` as a parameter list, and `.align` for globals

> `git checkout 7a1f816783064a12156807fe0a4d760c2e212d4e` — *Accept `void` as a parameter list*
>
> `git checkout 157356c769d777b1721da8218724608081137fe2` — *Align global variables*

Two small commits at the chapter's end, unrelated to the initializer arc, bundled because they're each too small for their own section.

The first lets a function declare zero parameters as `f(void)` rather than `f()`. The two are not the same in C: `f()` declares a function with an *unspecified* parameter list, accepting any arguments (a K&R-era ambiguity that the standard preserved for compatibility). `f(void)` declares a function with *no* parameters, an explicit zero-argument signature. The check is one line in `func_params`:

```c
if (equal(tok, "void") && equal(tok->next, ")")) {
  *rest = tok->next->next;
  return func_type(ty);
}
```

A two-token lookahead — `void` followed by `)` — distinguishes the no-parameter case from a single `void` *parameter* (which would be `f(void x)` and is illegal but for different reasons). If the pattern matches, skip both tokens and return a function type with no parameters. Otherwise fall through to the existing parameter-parsing loop.

Chibicc doesn't actually distinguish the two semantics — `f()` and `f(void)` both produce the same `Type` and both accept any arguments at the call site, because chibicc doesn't validate argument counts against declarations. The test changes are stylistic — Rui converts the test suite's previously-`int ret3()` declarations to `int ret3(void)` to match standard C — but no semantic check is added. The commit is there to accept the syntax, not to enforce its meaning.

The second commit emits `.align N` before each global. The codegen change is one line:

```c
println("  .globl %s", var->name);
println("  .align %d", var->ty->align);
```

`var->ty->align` is the type's alignment requirement, set when the type was constructed (a `long` is eight, a `char` is one, a struct is the largest of its members'). The `.align` directive tells the assembler to pad the preceding data so the next symbol starts at a multiple of N.

Without the directive, the assembler doesn't know the global's alignment requirement and may place it at an offset that triggers a misaligned access at runtime. On x86-64, misaligned accesses to most types work but are slower; misaligned accesses to certain SSE/AVX instructions trap. Chibicc is generous with `.align` here — it emits one for every global, even one-byte `char`s where alignment is already guaranteed.

This commit interacts with §12.6 and §12.9: `.align` is now emitted for both `.data` globals and `.bss` globals, and a relocation-target's address is the post-aligned address (which matters for `&g` initializer values). No test — the alignment is only visible in the assembler output, and any runtime misalignment would have shown up as nondeterministic crashes.

### Where we are

Functions can declare zero parameters explicitly with `(void)`. Global variables get explicit `.align` directives. Both commits are small; both are the kind of polish that a real C compiler accumulates. Neither extends the initializer machinery, but both belong to chibicc's approach-to-completeness arc that the chapter as a whole occupies.

---

## Where the chapter leaves us

Eleven sections, nineteen commits, and a stack of new pieces:

| Commit | Topic |
|---|---|
| `22dd560` | Local scalar and array initializers. The `Initializer` tree, `InitDesg` designator stack, `lvar_initializer` lowering pipeline, `ND_NULL_EXPR` no-op node. |
| `ae0a37d` | Excess-element zero-fill via `ND_MEMZERO` and `rep stosb`. Memzero-first-then-store strategy. |
| `a754732` | Excess-initializer skip via `skip_excess_element`. Brace balance preserved; AST discarded. |
| `0d71737` | String-literal initializers. `MIN(dest, src)` per-byte copy into the array's children. `MIN`/`MAX` macros enter the codebase. |
| `5b95533` | Length omitted from array. `is_flexible` flag on `Initializer`; type patched in place after parse. Closes §11.9's `array_of(ty, -1)` sentinel. |
| `e9d2c46` | Struct local initializers. `Member->idx` field bridges linked-list members to indexed children. Designator stack grows a `member` frame. |
| `aca19dd` | `struct T x = y;` — struct-from-struct initialization. Parser peeks for `{` to dispatch. |
| `483b194` | Union initializers. First-member-only; same shape as struct, simpler walker. |

| Commit | Topic |
|---|---|
| `bbfe3f4` | Global scalar/string initializers. `gvar_initializer` mirrors `lvar_initializer`. `eval` from §11.15 gets its first non-test caller. Local-vs-global split named. |
| `eeb62b6` | Global struct initializers. `mem->offset` drives byte placement. |
| `1eae5ae` | Global union and pointer initializers. `Relocation` struct. `eval` splits into `eval`/`eval2`/`eval_rval` to thread a `label` out-parameter through pointer-arithmetic-shaped expressions. `.quad symbol+addend` directive. |
| `efa0f33` | Optional inner braces in nested aggregates. Each aggregate parser splits into `_1` (with-braces) and `_2` (without). |
| `a58958c` | Extraneous braces around a scalar initializer. One stanza in `initializer2`. |
| `fde464c` | Trailing comma in enum lists and initializer lists. `is_end`/`consume_end` helpers. |
| `3d216e3` | Uninitialized globals to `.bss` instead of `.data`. First `.bss` use. |
| `824543b` | Flexible array members: trailing array of unspecified length in struct. Coerced to size-0 in `struct_members`. |
| `cd688a8` | Flexible-array initialization. `Type->is_flexible` flag. Struct size patched after parse via `copy_struct_type`. |
| `7a1f816` | `f(void)` accepted as zero-parameter declaration. Two-token lookahead in `func_params`. |
| `157356c` | `.align` directive emitted for every global. |

Three structural shifts deserve calling out.

The first is the *Initializer tree as the chapter's central object*. §12.1 builds it. §12.2 through §12.5 extend its construction to arrays-with-gaps, strings, length-omitted arrays, structs, and unions. §12.6 and §12.7 add a second consumer (`gvar_initializer` alongside `lvar_initializer`). §12.8 extends the parser dispatch. §12.10 extends both ends for flexible arrays. The tree is small — three pointers, a type, and two flags — but it's the structure every commit in the chapter touches.

The second is the *local-versus-global split*. The local path lowers an initializer to a comma chain of assignments at parse time and lets the existing codegen generate stores. The global path serializes an initializer to a byte buffer (plus a relocation list) at parse time and lets the existing codegen emit `.byte`/`.quad` directives. Both paths share the front end. Both paths share the type-driven dispatch. The split comes after the Initializer tree is built. It's the chapter's clearest example of a *shared front end with diverging back ends* — a pattern Chapter 13 will return to with linkage.

The third is the *re-use of the §11.9 incomplete-type machinery*. The `array_of(ty, -1)` sentinel from §11.9 has three consumers in this chapter: §12.4 (length-from-initializer), §12.10 (flexible array members), and the existing struct-forward-declaration case. The §11.9 commit's worth — small at the time — is now load-bearing for three independent features. The pre-factor-before-feature pattern — code in commit N supports features in commits M > N — adds two new instances (§12.4 and §12.10 each consume the §11.9 sentinel). The count is now six.

A few standing notes carried forward to Chapter 13:

- The *Initializer tree* shape is final. Chapter 13 doesn't extend it. New initializer features (compound literals, `_Alignas`) reuse the existing infrastructure.
- The *local-versus-global split* is a stable separation. Chapter 13 adds `extern`, which routes through `VarAttr` like Chapter 10's `static`/`typedef`, and `_Alignof`/`_Alignas`, which interact with the global path's alignment emission.
- The *`eval`/`eval2`/`eval_rval`* trio is the constant-expression evaluator's final shape. Chapter 13's `_Alignof` adds an arm.
- The *`Relocation` mechanism* is the only place chibicc emits link-time-relocatable directives. Chapter 13's `extern` and static locals interact with it through the symbol table, not the relocation list.
- The *canonicalization-at-parse-time* count is unchanged at eight. Initializer lowering is a separate mechanism (an Initializer tree intermediate, not a direct AST rewrite) and the chapter doesn't add it to the canonicalization family.
- The *pre-factor-before-feature* count is now six. §12.4 and §12.10 each consume the §11.9 incomplete-array sentinel.
- The *fourth namespace* (labels, from §11.10) is unchanged. Chapter 12 doesn't introduce a fifth.
- The *`is_typename` predicate* is unchanged in shape. Chapter 12 doesn't extend it.
- The *struct-mutation-in-place* pattern from §11.9 (`*sc->ty = *ty;`) does *not* repeat for flexible arrays. §12.10 uses `copy_struct_type` instead — because two variables with the same flexible-struct type can have different trailing-array sizes, the patched type belongs to the variable, not to the type. Same reason §12.4's typedef-of-incomplete-array test passes.
- The *VarAttr channel* (`is_typedef` and `is_static`) is unchanged. Chapter 13 will add `is_extern` as a third flag.
- The *`ND_NULL_EXPR` seed pattern* in `create_lvar_init` is a small structural choice worth remembering: a no-op node lets a comma-chain accumulator avoid a special case for the first element. Chapter 13 doesn't reuse it, but the same shape will likely recur.

Forward references for Chapter 13:

- `extern int x;` will declare `x` without allocating storage; `gvar_initializer` won't run. The interaction with `VarAttr` is in §10.6's mechanism.
- `_Alignof(T)` and `_Alignas(N)` interact with the global-alignment emission from §12.11.
- Compound literals (`(struct s){1, 2}`) reuse the `Initializer` tree but build it inline at expression position, not at declarator position.
- Static locals get `gvar_initializer`-shaped initialization and `.data`/`.bss` storage. The local-vs-global split distinction blurs at the static-local boundary; watch for the routing in `declaration`.
