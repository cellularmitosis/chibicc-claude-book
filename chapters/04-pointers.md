# Chapter 4 — Pointers

> Commits covered: `3d86277`, `863e2b8`, `a6bc4ab`, `b4e82cf`. Four commits, each smaller than its neighbours in Chapter 3, but the third one introduces a new file (`type.c`) and quietly rewires the parser to know what kind of value every expression produces.

Chapter 3 left chibicc with a small imperative language: blocks, locals, assignment, `if`, `for`, `while`, `return`. The compiler could turn programs like `{ i=0; while (i<10) i=i+1; return i; }` into running x86-64 binaries. What it couldn't do was *talk about memory*. There was no way to take a variable's address, no way to dereference a pointer, and — because there were no pointers — no need for the parser to think about what type any given expression had. Every expression was just "an integer."

Chapter 4 corrects that, and in doing so it forces the compiler to grow its first real type system. The four commits are:

1. A bookkeeping refactor that attaches a representative token to every AST node, so error messages can point at source locations.
2. The unary `&` and `*` operators — chibicc's first taste of pointer-shaped expressions.
3. Pointer arithmetic. This is the commit that makes the parser care about types: `p + 1` and `1 + 1` have to compile differently, and the compiler can't tell them apart without knowing what `p` is.
4. The keyword `int`, real variable declarations, and an end to the auto-create-on-first-use scheme that has carried us this far.

The chapter has one concept interlude, on what "type" means inside a compiler and why chibicc has to start tracking types now. It's placed between §4.2 and §4.3 — after we've seen what `&` and `*` *do*, but before pointer arithmetic forces us to give names to the things they operate on.

---

## 4.1 — Representative tokens for error messages

> `git checkout 3d8627719be00e39070eaca0ee5b599f2a877c5c` — *Add a representative node to each Node to improve error messages*

The first commit of the chapter doesn't add a feature. It adds a field. Every AST node gains a `Token *tok` member, and every node-constructor takes a token argument so it can fill the field in. The point is to give codegen something useful to report when it encounters an AST it can't handle.

```diff
 struct Node {
   NodeKind kind; // Node kind
   Node *next;    // Next node
+  Token *tok;    // Representative token

   Node *lhs;     // Left-hand side
   ...
 };
```

The token a node carries is its "representative" — usually the operator token of a compound expression, or the first token of a statement. For `1 + 2`, it's the `+`. For `if (cond) ...`, it's the `if`. For an assignment, it's the `=`. Nothing in the AST changes structurally; the token is purely metadata for diagnostics.

### Threading the token through the constructors

Every constructor in `parse.c` learns to take a `Token *tok` and stash it. The change rolls through the four helpers we've been using since Chapter 3:

```diff
-static Node *new_node(NodeKind kind) {
+static Node *new_node(NodeKind kind, Token *tok) {
   Node *node = calloc(1, sizeof(Node));
   node->kind = kind;
+  node->tok = tok;
   return node;
 }

-static Node *new_binary(NodeKind kind, Node *lhs, Node *rhs) {
-  Node *node = new_node(kind);
+static Node *new_binary(NodeKind kind, Node *lhs, Node *rhs, Token *tok) {
+  Node *node = new_node(kind, tok);
   node->lhs = lhs;
   node->rhs = rhs;
   return node;
 }
```

`new_unary`, `new_num`, and `new_var_node` get the same treatment. All twenty-some call sites get updated to pass the right token.

The interesting part is *which* token each call site picks. In the binary-operator loops, the pattern is:

```diff
 static Node *equality(Token **rest, Token *tok) {
   Node *node = relational(&tok, tok);

   for (;;) {
+    Token *start = tok;
+
     if (equal(tok, "==")) {
-      node = new_binary(ND_EQ, node, relational(&tok, tok->next));
+      node = new_binary(ND_EQ, node, relational(&tok, tok->next), start);
       continue;
     }
     ...
   }
 }
```

`start` is captured *before* the operator is consumed, so it points at the `==` (or `+`, `<`, etc.) — the natural thing for an error to underline if the binary op turns out to be malformed. The same shape repeats in `relational`, `add`, `mul`. Unary operators capture `tok` directly (the operator is the current token), and `primary`'s identifier and number cases pass the identifier or number token itself.

Statement constructors do the obvious thing — `stmt`'s `if`/`for`/`while`/`return` branches each grab the keyword token before advancing.

### Codegen's three error paths

The whole reason to do this is sitting in `codegen.c`:

```diff
 static void gen_addr(Node *node) {
   ...
-  error("not an lvalue");
+  error_tok(node->tok, "not an lvalue");
 }

 static void gen_expr(Node *node) {
   ...
-  error("invalid expression");
+  error_tok(node->tok, "invalid expression");
 }

 static void gen_stmt(Node *node) {
   ...
-  error("invalid statement");
+  error_tok(node->tok, "invalid statement");
 }
```

Three call sites, three swaps. `error_tok` is the function we built back in Chapter 1: it prints the source line, points a caret at the offending column, and exits. Before this commit, codegen errors said only "not an lvalue" with no indication of *which* lvalue. Now they print:

```
foo.c:3: not an lvalue
  *(1+2) = 3;
   ^
```

That's the difference between a useful error and a frustrating one. It's also the reason the rest of Chapter 4 can afford to be terser about diagnostics — every node we add for the next three commits will inherit the representative-token convention, so when (say) the dereference operator gets misused, the error already knows where to point.

### A small parser cleanup that snuck in

The forward declarations at the top of `parse.c` get reordered:

```diff
 static Node *compound_stmt(Token **rest, Token *tok);
-static Node *expr(Token **rest, Token *tok);
 static Node *expr_stmt(Token **rest, Token *tok);
+static Node *expr(Token **rest, Token *tok);
 static Node *assign(Token **rest, Token *tok);
```

Cosmetic only — the file's call graph runs `expr_stmt` → `expr` → `assign`, and reordering the declarations to match makes the file easier to read top-to-bottom. The kind of thing you only do when you're already touching every line of the file.

And one more small change: `assign` is rewritten to thread `rest` directly through the recursive call, dropping a temporary:

```diff
 static Node *assign(Token **rest, Token *tok) {
   Node *node = equality(&tok, tok);
+
   if (equal(tok, "="))
-    node = new_binary(ND_ASSIGN, node, assign(&tok, tok->next));
+    return new_binary(ND_ASSIGN, node, assign(rest, tok->next), tok);
+
   *rest = tok;
   return node;
 }
```

Same behavior; one fewer assignment to `*rest`. The `tok` token passed to `new_binary` is the `=`, captured before the recursive call advances.

### Where we are

The AST now carries source locations. Every node remembers a token; every error in codegen can name a place. Nothing user-visible changed unless a program triggers a codegen error, but the next three commits are about to add four new node kinds and several new error paths, and we've paid the bookkeeping cost up front so each of them gets a useful diagnostic for free.

---

## 4.2 — Unary `&` and `*`

> `git checkout 863e2b8de25fdf43a4a63b93d0f57718e9edaa47` — *Add unary & and ***

Now the language gets pointers. After this commit, `&x` produces the address of `x`, and `*p` reads through `p`. The tests look like this:

```c
{ x=3; return *&x; }                  // exits with 3
{ x=3; y=&x; z=&y; return **z; }      // exits with 3
{ x=3; y=&x; *y=5; return x; }        // exits with 5
{ x=3; y=5; return *(&x+8); }         // exits with 5
```

The last one is worth a second look. It computes the address of `x`, adds *8 bytes* to it, and dereferences. Since `y` is the next slot down the stack from `x` — at offset `-16(%rbp)`, eight bytes lower than `x`'s `-8(%rbp)` — `&x + 8` happens to land on `y`. The program is reading `y` through pointer arithmetic on `x`'s address.

This works because at this commit chibicc has no idea what type `&x` is. To the compiler it's just an integer that came out of a `lea` instruction, and `+8` is just integer addition. Adding `8` to the address of `x` does what you'd expect on a machine where ints are eight bytes apart, but the compiler has no notion that "8" means "one int away." That ad-hoc-ness is exactly what the next commit will fix; in the meantime, the test suite leans into it. Tests like `*(&x+8)=7` are deliberately exercising the raw, untyped pointer arithmetic, with hand-computed byte offsets. They'll all be rewritten in the next commit, when `+1` starts meaning "one int forward."

### Two new node kinds

```diff
 typedef enum {
   ...
   ND_ASSIGN,    // =
+  ND_ADDR,      // unary &
+  ND_DEREF,     // unary *
   ND_RETURN,    // "return"
   ...
 } NodeKind;
```

`ND_ADDR` represents a `&` expression; its operand goes in `lhs`. `ND_DEREF` represents `*`; same shape. No new fields on `Node` — both node kinds reuse the existing `lhs`.

### Parser: two more cases in `unary`

The grammar comment grows:

```c
// unary = ("+" | "-" | "*" | "&") unary
//       | primary
```

The function gains two branches:

```c
if (equal(tok, "&"))
  return new_unary(ND_ADDR, unary(rest, tok->next), tok);

if (equal(tok, "*"))
  return new_unary(ND_DEREF, unary(rest, tok->next), tok);
```

Same shape as the existing unary `-` branch. The recursion is on `unary`, not `primary`, so chains like `**p` or `&*&x` parse the natural right-associative way. `**p` becomes `ND_DEREF(ND_DEREF(p))`; `&*x` becomes `ND_ADDR(ND_DEREF(x))`.

A subtlety the grammar doesn't make obvious: chibicc's tokenizer already handled `*` and `&` as punctuators long before this commit. `*` was `ND_MUL`'s operator; `&` had no use yet. The same `*` token now has two parser-side meanings — multiplication when it appears in `mul`, dereference when it appears in `unary`. Recursive descent disambiguates by context: `mul` calls `unary` for each operand, and `unary` decides whether the `*` it sees is a unary prefix (because it's at the start of a unary expression) or has already been past as a binary `*` (in which case `mul` consumed it before `unary` ever saw it).

### Parser comment block

This commit also drops a block comment at the top of `parse.c`:

```c
// This file contains a recursive descent parser for C.
//
// Most functions in this file are named after the symbols they are
// supposed to read from an input token list. ...
//
// Each function conceptually returns two values, an AST node and
// remaining part of the input tokens. Since C doesn't support
// multiple return values, the remaining tokens are returned to the
// caller via a pointer argument.
//
// Input tokens are represented by a linked list. Unlike many recursive
// descent parsers, we don't have the notion of the "input token stream".
// Most parsing functions don't change the global state of the parser.
// So it is very easy to lookahead arbitrary number of tokens in this
// parser.
```

This is the kind of orientation comment that pays off on a months-later read. Nothing it says is news to a reader who has been with us since Chapter 2 — the `Token **rest, Token *tok` convention has been with chibicc since the modular split — but committing it as a header is the moment Rui makes that convention an explicit part of the parser's contract. Future chapters will lean on the "no input stream, no global state, easy lookahead" claim several times; the comment is a quiet promise to keep it true.

### Codegen: `gen_expr` and `gen_addr` each grow a case

```c
case ND_DEREF:
  gen_expr(node->lhs);
  printf("  mov (%%rax), %%rax\n");
  return;
case ND_ADDR:
  gen_addr(node->lhs);
  return;
```

`ND_DEREF` evaluates its operand to get an address, then dereferences it: `mov (%rax), %rax` reads the eight bytes at the address in `%rax` and replaces `%rax` with them. This is the same instruction the existing `ND_VAR` case uses — a variable read is conceptually `*&v`, and the codegen pattern reflects that.

`ND_ADDR` is simpler. Its operand has to be an lvalue (`&3` is a syntax error), and `gen_addr` already knows how to compute the address of an lvalue into `%rax`. So `&x` is just "compute the address; that's the result."

The other change is in `gen_addr` itself:

```c
static void gen_addr(Node *node) {
  switch (node->kind) {
  case ND_VAR:
    printf("  lea %d(%%rbp), %%rax\n", node->var->offset);
    return;
  case ND_DEREF:
    gen_expr(node->lhs);
    return;
  }

  error_tok(node->tok, "not an lvalue");
}
```

The function flips from an `if` to a `switch`, and gains a `ND_DEREF` case. This is the bit that lets `*p = 5` work. The lvalue isn't `p` — that's a variable, and the value we want to write is `5`, not a new pointer. The lvalue is the location *that `p` points at*. Computing the address of `*p` is the same as computing the value of `p`: in both cases we want the address `p` holds, in `%rax`. So the `ND_DEREF` arm of `gen_addr` calls `gen_expr` on the operand, exactly like the rvalue case, except it stops there — no second `mov (%rax), %rax`.

This is a small but elegant piece of code-gen plumbing. In rvalue position (inside `gen_expr`), `*p` does *two* loads: one to get `p`'s value, another to read through it. In lvalue position (inside `gen_addr`), `*p` does *one* load: just `p`'s value, which already is the address we want to write to. The difference between an lvalue dereference and an rvalue dereference is one `mov`.

### Codegen forward-declares `gen_expr`

```diff
 static int depth;

+static void gen_expr(Node *node);
+
 static int count(void) {
   ...
 }
```

`gen_addr` now calls `gen_expr` (in the `ND_DEREF` arm), and `gen_expr` calls `gen_addr` (in the `ND_VAR`, `ND_ADDR`, and `ND_ASSIGN` arms). With the two functions mutually recursive, the one defined first needs a forward declaration. Tiny housekeeping; the kind of thing C makes you do whenever functions form a cycle.

### Where we are

The compiler can now talk about memory. Tests pass programs like `int x; x=3; *&x = 5;` (give or take real declarations, which we don't have yet) and produce the right behavior. But the four `*(&x+8)` tests in the suite are a tell: pointer arithmetic in chibicc is a fiction. The compiler isn't really doing pointer arithmetic; it's doing integer arithmetic on values that happen to be addresses. To make `+1` mean "one int forward" instead of "one byte forward," we need the compiler to know when an expression is a pointer and when it's an integer. That means types.

---

## Concept interlude — What a type is, and why the parser needs them

Chibicc has gotten by for three chapters without a type system. Every expression has been an integer; every variable has held an integer; every operation has been an integer operation. The compiler hasn't had to ask "what kind of value is this?" because there was only one kind.

Pointer arithmetic ends that. The expression `p + 1` needs to do something different from `n + 1`, and the only way to tell them apart is to know that `p` is a pointer and `n` isn't. So we have to introduce, however minimally, the idea that expressions have *types*.

### Types in a compiler

A *type* in a compiler is a static label attached to an expression that says what kind of value it produces. "Static" means the label is computed at compile time, not run time — by the time the program runs, the types have done their job and disappeared. The label is computed by walking the AST: the type of `1 + 2` is determined by the types of `1` and `2`; the type of `*p` is determined by the type of `p`; and so on, recursively, up from the leaves.

Types matter because operators care. Some operators only make sense on certain types (`*p` requires `p` to be a pointer). Some operators do different things depending on the types of their operands (`p + 1` is pointer arithmetic if `p` is a pointer, integer addition if it isn't). Some operators are restricted to specific combinations (`p - q` makes sense if both are pointers, or if `p` is a pointer and `q` is an integer, but not if `p` is an integer and `q` is a pointer). Without types, none of those distinctions can be made at compile time, and the compiler can't generate the right code.

A type system is the bookkeeping the compiler does to make all of this work. At minimum it needs three things: a representation for types (so the compiler can compare them and pass them around), a way to attach types to AST nodes (so each expression's type is available to the code that needs it), and a procedure for *deriving* types — figuring out the type of a compound expression from the types of its parts.

### What chibicc starts with

Chibicc's first type system is the smallest one that solves the immediate problem. It has exactly two type kinds:

```c
typedef enum {
  TY_INT,
  TY_PTR,
} TypeKind;

struct Type {
  TypeKind kind;
  Type *base;
};
```

`TY_INT` represents the (one and only) integer type. `TY_PTR` represents "pointer to something." The "something" is recorded in the `base` field — itself a pointer to a `Type`. So "pointer to int" is a `Type` whose `kind` is `TY_PTR` and whose `base` points at the singleton `int` type. "Pointer to pointer to int" is a `Type` whose `kind` is `TY_PTR` and whose `base` points at "pointer to int." Arbitrary nesting falls out of the recursive structure.

This is a *very* limited type system. It has no `char`, no `long`, no `short`, no `float`, no `void`, no struct, no array, no function type. All those will arrive in later chapters: `char` in Chapter 7, arrays in Chapter 6, structs in Chapter 9, function types in Chapter 10. For now, every value is either `int` or a pointer to (a pointer to (a pointer to (...))) `int`.

That sounds restrictive, and it is. But it's enough to express the type discriminations chibicc currently needs, because every operator we have can be classified along exactly one axis: does this operand need to be an integer, a pointer, or either? Pointer arithmetic is the only operator family that has to look at types at all, and even there, the rules collapse to "scale by 8 if one side is a pointer." Everything else — arithmetic, comparisons, assignment — works without caring whether the operands are pointers. (We'll see in §4.3 that some of those operators inherit a pointer type for their *result* even though they treat their operands uniformly. That's a small quirk of the implementation, not a deep feature.)

### A separate file: `type.c`

Chibicc gets a fourth source file in this chapter's third commit:

```
chibicc.h
codegen.c
parse.c
tokenize.c
type.c       ← new
```

Why a separate file? Two reasons. First, the module split from Chapter 2 was deliberately along compilation phases (lex/parse/emit), and the type system is awkward to slot into that split — types are computed during parsing but used by both the parser (for pointer arithmetic) and, eventually, by codegen (for sizing loads and stores). Putting types in their own file makes them a service that other files call into rather than a piece of one phase's internals. Second, the type system is going to grow. Adding `char`, structs, arrays, function types, and the conversion rules between them will eventually balloon `type.c` to several hundred lines. Starting it as its own file keeps the eventual growth contained.

The file at this commit is small — fewer than sixty lines — but its shape is what matters. It exports three things: `ty_int` (a singleton for the int type), `is_integer` (a predicate), `pointer_to` (a constructor that wraps a base type in `TY_PTR`), and `add_type` (a tree-walker that recursively assigns types to every node in an AST subtree). The parser will call `add_type` whenever it builds a piece of AST whose type it needs to consult. We'll see exactly how in the next section.

### The shape of "types attached to AST"

`Node` gains one field:

```diff
 struct Node {
   NodeKind kind; // Node kind
   Node *next;    // Next node
+  Type *ty;      // Type, e.g. int or pointer to int
   Token *tok;    // Representative token
   ...
 };
```

`ty` is a pointer, not an embedded `Type`, for the obvious reason: types are shared. Every `int`-typed expression points at the same `ty_int` singleton; every "pointer to int"-typed expression can point at the same allocated `pointer_to(ty_int)` `Type` (in practice each call to `pointer_to` allocates a fresh one — chibicc doesn't intern types — but they're equivalent).

`ty` starts out NULL. It's filled in by `add_type`. The order is important: a node's type depends on its children's types, so `add_type` computes children first, then the parent. The recursion bottoms out at leaf nodes — `ND_NUM` and `ND_VAR` — whose types come from their stored data directly (`int` for numbers, `var->ty` for variables — though for §4.3 every variable's type is still `int`, so they all end up `int`).

That's the model. The next section puts it to work.

---

## 4.3 — Pointer arithmetic

> `git checkout a6bc4ab101c20b6398fd6bbfe124665bb7db5d25` — *Make pointer arithmetic work*

This is the commit where `+` and `-` learn that pointers exist. After it, `&x + 1` advances by `sizeof(int)` bytes — eight, since chibicc treats every value as eight bytes — and the test suite is rewritten to use `+1` instead of `+8`:

```diff
-assert 5 '{ x=3; y=5; return *(&x+8); }'
-assert 3 '{ x=3; y=5; return *(&y-8); }'
+assert 5 '{ x=3; y=5; return *(&x+1); }'
+assert 3 '{ x=3; y=5; return *(&y-1); }'
+assert 5 '{ x=3; y=5; return *(&x-(-1)); }'
```

A new test exercises pointer subtraction:

```sh
assert 5 '{ x=3; return (&x+2)-&x+3; }'
```

`(&x+2) - &x` should be 2 (two ints forward minus zero ints forward), and `2 + 3` is 5. The compiler has to produce that, which means it has to handle three different `-` cases: integer minus integer, pointer minus integer, and pointer minus pointer.

### The trick: desugaring at parse time

The whole change happens in the parser. `codegen.c` is untouched — not one line. That's because chibicc handles pointer arithmetic by *rewriting it into integer arithmetic* in the AST itself. By the time codegen sees the tree, `p + 1` has already been turned into `p + (1 * 8)`, an ordinary `ND_ADD` of two integer-typed expressions. Codegen has no idea pointer arithmetic happened; it just adds two values.

This is the same canonicalize-at-parse-time discipline we saw with `>` (rewritten as `<` with operands swapped) and `while` (rewritten as a degenerate `for`). The codegen stays simple at the cost of slightly more work in the parser. Over the course of the book this pattern will compound: by the time chibicc compiles itself, the codegen handles a much smaller language than the parser accepts.

### The new helpers: `new_add` and `new_sub`

The `add` function used to call `new_binary(ND_ADD, ...)` and `new_binary(ND_SUB, ...)` directly. Now it calls into two new wrappers:

```diff
 static Node *add(Token **rest, Token *tok) {
   Node *node = mul(&tok, tok);

   for (;;) {
     Token *start = tok;

     if (equal(tok, "+")) {
-      node = new_binary(ND_ADD, node, mul(&tok, tok->next), start);
+      node = new_add(node, mul(&tok, tok->next), start);
       continue;
     }

     if (equal(tok, "-")) {
-      node = new_binary(ND_SUB, node, mul(&tok, tok->next), start);
+      node = new_sub(node, mul(&tok, tok->next), start);
       continue;
     }
     ...
   }
 }
```

`new_add` and `new_sub` are where the type-aware logic lives.

### `new_add`: four cases collapse to three

```c
// In C, `+` operator is overloaded to perform the pointer arithmetic.
// If p is a pointer, p+n adds not n but sizeof(*p)*n to the value of p,
// so that p+n points to the location n elements (not bytes) ahead of p.
// In other words, we need to scale an integer value before adding to a
// pointer value. This function takes care of the scaling.
static Node *new_add(Node *lhs, Node *rhs, Token *tok) {
  add_type(lhs);
  add_type(rhs);

  // num + num
  if (is_integer(lhs->ty) && is_integer(rhs->ty))
    return new_binary(ND_ADD, lhs, rhs, tok);

  if (lhs->ty->base && rhs->ty->base)
    error_tok(tok, "invalid operands");

  // Canonicalize `num + ptr` to `ptr + num`.
  if (!lhs->ty->base && rhs->ty->base) {
    Node *tmp = lhs;
    lhs = rhs;
    rhs = tmp;
  }

  // ptr + num
  rhs = new_binary(ND_MUL, rhs, new_num(8, tok), tok);
  return new_binary(ND_ADD, lhs, rhs, tok);
}
```

Let's walk it. The first two lines compute the types of both operands — we have to know what they are before we can decide what to emit. (`add_type` is idempotent: if a node already has a `ty`, the call is a no-op.)

Then four cases:

1. **`num + num`.** Both operands are integers; emit a plain `ND_ADD`. This is the path that compiles `1 + 2` exactly the way it always has.
2. **`ptr + ptr`.** Both operands are pointers. C declares this an error (you can subtract two pointers, but not add them — adding two addresses doesn't mean anything geometrically). `error_tok` reports it at the `+`.
3. **`num + ptr`.** Swap the operands so the pointer is on the left, then fall into case 4.
4. **`ptr + num`.** Multiply the integer by 8, emit `ND_ADD` with the pointer on the left and the scaled integer on the right.

The check for "is this a pointer?" is `lhs->ty->base`. Recall the `Type` struct: `base` is non-NULL if and only if the type is a pointer. (`int`'s `base` is NULL; "pointer to int" has `base` pointing at `ty_int`.) This is a slick test — it doesn't even mention `TY_PTR` — and it'll keep working once arrays arrive in Chapter 6, because arrays will also use the `base` field. The condition really means "is this a thing that points at something?"

The "swap to canonicalize" step deserves a moment. `2 + p` and `p + 2` mean the same thing in C, but the codegen pattern for "scale the int and add to the pointer" is easier to write if the pointer is always on the left. So the parser swaps when it has to, and from that point on every pointer-arithmetic AST has the pointer on the left. That's the canonicalization: the surface form `2 + p` is allowed, but the AST never reflects it. By the time codegen runs, the asymmetry is gone.

The hardcoded `8` is the giveaway that chibicc doesn't yet have a real notion of size. C's actual rule is "scale by `sizeof(*p)`" — by the size of the pointee. Right now the pointee is always `int`, and `int` is always eight bytes (chibicc uses 64-bit ints for now), so `8` happens to be correct. Chapter 6 will introduce arrays and `sizeof`, and Chapter 7 will add `char` (a one-byte type), and the literal `8` will eventually have to become a per-type lookup. But the simplest possible code today is one literal, and that's what Rui writes.

### `new_sub`: three cases, three different result types

Subtraction has more shapes than addition because pointer-minus-pointer is legal:

```c
static Node *new_sub(Node *lhs, Node *rhs, Token *tok) {
  add_type(lhs);
  add_type(rhs);

  // num - num
  if (is_integer(lhs->ty) && is_integer(rhs->ty))
    return new_binary(ND_SUB, lhs, rhs, tok);

  // ptr - num
  if (lhs->ty->base && is_integer(rhs->ty)) {
    rhs = new_binary(ND_MUL, rhs, new_num(8, tok), tok);
    add_type(rhs);
    Node *node = new_binary(ND_SUB, lhs, rhs, tok);
    node->ty = lhs->ty;
    return node;
  }

  // ptr - ptr, which returns how many elements are between the two.
  if (lhs->ty->base && rhs->ty->base) {
    Node *node = new_binary(ND_SUB, lhs, rhs, tok);
    node->ty = ty_int;
    return new_binary(ND_DIV, node, new_num(8, tok), tok);
  }

  error_tok(tok, "invalid operands");
}
```

Three legal cases:

1. **`num - num`.** Plain integer subtraction.
2. **`ptr - num`.** Scale the integer by 8, subtract. The result is a pointer (you've moved backwards by `n` elements, but you're still pointing at something), so the resulting node's `ty` is set to `lhs->ty`. This is the first time we see a node with a manually-overridden type.
3. **`ptr - ptr`.** Subtract the two addresses; the result is a byte distance. Divide by 8 to get the element distance. The resulting node is `int`-typed — `add_type` would have given it `lhs->ty`, but this case explicitly sets it to `ty_int` *before* the division wraps it, because the division is what produces the final `int` value.

The fourth case — `num - ptr` — is illegal (you can't subtract a pointer from an integer) and falls through to the error.

The little dance with manually setting `node->ty` is necessary because `add_type`'s rule for `ND_ADD`/`ND_SUB`/`ND_MUL`/`ND_DIV` is "the result has the same type as `lhs`." That's correct for `ptr - num` (pointer in, pointer out), but it's *wrong* for `ptr - ptr`, where both operands are pointers but the result is an integer count. Setting `node->ty = ty_int` overrides the default. The intermediate `ND_SUB` node we build is never directly returned — it's wrapped in an `ND_DIV` — so the override is essentially a hint to whoever else might inspect it later (right now, no one does, but the discipline is good).

### `add_type`: the type-deriver itself

Stored in the new file `type.c`, just shy of sixty lines. Its job is to walk an AST subtree and fill in `node->ty` for every node that doesn't have one yet:

```c
void add_type(Node *node) {
  if (!node || node->ty)
    return;

  add_type(node->lhs);
  add_type(node->rhs);
  add_type(node->cond);
  add_type(node->then);
  add_type(node->els);
  add_type(node->init);
  add_type(node->inc);

  for (Node *n = node->body; n; n = n->next)
    add_type(n);

  switch (node->kind) {
  case ND_ADD:
  case ND_SUB:
  case ND_MUL:
  case ND_DIV:
  case ND_NEG:
  case ND_ASSIGN:
    node->ty = node->lhs->ty;
    return;
  case ND_EQ:
  case ND_NE:
  case ND_LT:
  case ND_LE:
  case ND_VAR:
  case ND_NUM:
    node->ty = ty_int;
    return;
  case ND_ADDR:
    node->ty = pointer_to(node->lhs->ty);
    return;
  case ND_DEREF:
    if (node->lhs->ty->kind == TY_PTR)
      node->ty = node->lhs->ty->base;
    else
      node->ty = ty_int;
    return;
  }
}
```

Three things to notice.

**The early-out.** `if (!node || node->ty) return;` handles two cases: a NULL node (which can happen for any of the optional fields like `cond` or `els`) and a node whose type is already filled in. The latter is what makes `add_type` cheap to call multiple times — `new_add` calls it on every operand, and many of those operands have been visited before. The check is the cache; calling `add_type` on an already-typed tree is essentially free.

**The recursion is unconditional.** `add_type` walks every pointer the `Node` struct has — `lhs`, `rhs`, `cond`, `then`, `els`, `init`, `inc`, and the `body` list. Most of those will be NULL for any given node, so the walks bottom out quickly. The point is that one `add_type` call is enough to type a whole subtree; the parser doesn't have to remember which fields are populated for which node kinds.

**The type rules are mostly local.** Each node kind's type is computed from its immediate children's types, in one step:
- Arithmetic and assignment inherit `lhs`'s type.
- Comparisons, variables, and numbers are `int`.
- `&x` is "pointer to whatever `x` is."
- `*p` is "whatever `p` points at," with a graceful fallback to `int` if `p` happens not to be a pointer.

The `ND_DEREF` fallback is interesting. In real C, `*1` is an error — you can't dereference a non-pointer. Chibicc accepts it and types the result as `int`. The reason is the test suite at this point: programs like `x = 3; y = *&x;` work because the *codegen* doesn't care about types, even when the parser does. Saying "if it isn't a pointer, pretend the dereference produces an int" is a way to keep `add_type` from spuriously rejecting programs that the codegen would handle fine. This permissive behavior won't last; the next commit replaces it with a real error.

### A small parser cleanup that comes with the territory

`compound_stmt` now calls `add_type` on each statement as it's parsed:

```diff
   while (!equal(tok, "}")) {
-    cur = cur->next = stmt(&tok, tok);
+    cur = cur->next = stmt(&tok, tok);
+    add_type(cur);
+  }
```

This guarantees every statement-level subtree has its types filled in by the time the parser is done. It's slightly more than the *minimum* needed — `new_add` and `new_sub` already call `add_type` on the operands they care about — but it's cheap (`add_type` early-outs on already-typed nodes) and it sets up future commits that will want types available throughout the AST.

`stmt` also gets a forward declaration that was missing:

```diff
 static Node *compound_stmt(Token **rest, Token *tok);
+static Node *stmt(Token **rest, Token *tok);
 static Node *expr_stmt(Token **rest, Token *tok);
```

This was actually a latent ordering problem — `compound_stmt` calls `stmt`, which is defined later in the file. The compilers we're using haven't complained, but the declaration makes the order explicit.

### A look at what gets emitted

To make all of this concrete, take the program `*(&x+1)`. After parsing and type-attaching, the AST looks like this (with types in brackets):

```
ND_DEREF [int]
└── ND_ADD [pointer-to-int]
    ├── ND_ADDR [pointer-to-int]
    │   └── ND_VAR x [int]
    └── ND_MUL [int]
        ├── ND_NUM 1 [int]
        └── ND_NUM 8 [int]
```

The `*1` has been multiplied to `*8` already. The codegen sees only the `ND_DEREF`, the `ND_ADD`, the `ND_ADDR`, and two `ND_NUM`s plus an `ND_MUL` — all node kinds it has known how to handle since long before this commit. From its point of view nothing has changed; the typed parse already did the hard work.

### Where we are

Pointer arithmetic works the way C says it should. The compiler now has a notion of types, however small — two kinds and a one-level recursion. Codegen still doesn't look at types, but the parser does, and that's enough for `+` and `-` to do the right thing on pointers. The next commit ties up the loose end: we still don't have the keyword `int`, and `x = 3` still implicitly creates `x`. Now that the type system can express "pointer to int," it's time to make declarations real.

---

## 4.4 — `int` and mandatory declarations

> `git checkout b4e82cf7ce1cbfff8dd30f20fdad73fd3f1d5ccb` — *Add keyword "int" and make variable definition mandatory*

Up to this point chibicc has been auto-creating variables. The first time `primary` saw an identifier, it called `new_lvar` and made one up, type unspecified. After this commit, every variable has to be declared with `int` (and optionally `*`s in front), and the auto-create path is replaced with an outright error.

Programs that compiled in the previous commit no longer compile. The test suite is rewritten to add `int` declarations to every variable use:

```diff
-assert 3 '{ a=3; return a; }'
-assert 8 '{ a=3; z=5; return a+z; }'
+assert 3 '{ int a; a=3; return a; }'
+assert 3 '{ int a=3; return a; }'
+assert 8 '{ int a=3; int z=5; return a+z; }'
```

The new tests also exercise initializers and multi-declaration syntax:

```sh
assert 8 '{ int x, y; x=3; y=5; return x+y; }'
assert 8 '{ int x=3, y=5; return x+y; }'
```

And the pointer tests get explicit pointer types:

```diff
-assert 3 '{ x=3; y=&x; z=&y; return **z; }'
+assert 3 '{ int x=3; int *y=&x; int **z=&y; return **z; }'
-assert 5 '{ x=3; y=&x; *y=5; return x; }'
+assert 5 '{ int x=3; int *y=&x; *y=5; return x; }'
```

`int **z` gets parsed as "pointer to pointer to int," and the type system handles the rest.

### A new helper: `consume`

Tokenize.c gets a small utility:

```c
bool consume(Token **rest, Token *tok, char *str) {
  if (equal(tok, str)) {
    *rest = tok->next;
    return true;
  }
  *rest = tok;
  return false;
}
```

`consume` is `equal` plus an advance-on-match. It returns `true` if the current token matches `str` (consuming it) and `false` otherwise (leaving the cursor where it was). The "did it match? advance if so" idiom shows up everywhere in the parser; bundling it lets the call sites read more like English. We'll use it almost immediately, in the declarator's pointer loop.

### The keyword list grows by one

```diff
-  static char *kw[] = {"return", "if", "else", "for", "while"};
+  static char *kw[] = {"return", "if", "else", "for", "while", "int"};
```

Same single-line change as the keyword additions in §3.7 and §3.9. The reclassification pass in `convert_keywords` does the rest.

### `Type` gains a `name` field

```diff
 struct Type {
   TypeKind kind;
+
+  // Pointer
   Type *base;
+
+  // Declaration
+  Token *name;
 };
```

A `Type` is starting to do double duty. In an expression context it represents the type of an expression — `int`, "pointer to int," etc. In a *declaration* context, the parser also wants to track the *name being declared*. Putting the name on the `Type` is the path of least resistance: the declarator parser builds up a `Type` as it walks left-to-right through `*`s and the identifier, and the identifier ends up stored on the type itself.

Chibicc could have used a separate "declaration" struct that holds a `Type *` and a `Token *name` side by side. Rui chooses to fold the two together. The same pattern as `Node` having every field anybody could want: keep the data model wide and cheap rather than narrow and proliferating.

### `Obj` gains a type

```diff
 struct Obj {
   Obj *next;
   char *name; // Variable name
+  Type *ty;   // Type
   int offset; // Offset from RBP
 };
```

Every local variable now has a recorded type. It's filled in by `new_lvar`, which gains a `Type *` parameter:

```c
static Obj *new_lvar(char *name, Type *ty) {
  Obj *var = calloc(1, sizeof(Obj));
  var->name = name;
  var->ty = ty;
  var->next = locals;
  locals = var;
  return var;
}
```

This is the link in the chain that makes `add_type` work for `ND_VAR`. Recall the previous commit's case: `ND_VAR`'s type came from `var->ty`, which was always NULL. After this commit the field is populated, and `add_type` for `ND_VAR` returns whatever the variable's declared type was — usually `int`, sometimes a pointer.

Speaking of which, `type.c` updates the `ND_VAR` case:

```diff
   case ND_NE:
   case ND_LT:
   case ND_LE:
-  case ND_VAR:
   case ND_NUM:
     node->ty = ty_int;
     return;
+  case ND_VAR:
+    node->ty = node->var->ty;
+    return;
```

`ND_VAR` is no longer always `int`; it inherits the declared type of the variable.

And `ND_DEREF` becomes strict:

```diff
   case ND_DEREF:
-    if (node->lhs->ty->kind == TY_PTR)
-      node->ty = node->lhs->ty->base;
-    else
-      node->ty = ty_int;
+    if (node->lhs->ty->kind != TY_PTR)
+      error_tok(node->tok, "invalid pointer dereference");
+    node->ty = node->lhs->ty->base;
     return;
```

The "fallback to int" branch goes away. Once the type system is reliable — every variable is declared, every type is known — dereferencing a non-pointer is a real error worth reporting.

### Parsing declarations

This is the bulk of the diff. The grammar grows three new productions:

```
declaration = declspec (declarator ("=" expr)? ("," declarator ("=" expr)?)*)? ";"
declspec    = "int"
declarator  = "*"* ident
```

`declspec` parses a "declaration specifier" — for now, only the keyword `int`. It returns the type that specifier names:

```c
// declspec = "int"
static Type *declspec(Token **rest, Token *tok) {
  *rest = skip(tok, "int");
  return ty_int;
}
```

Three lines of code; over the rest of the book this function will accumulate every type-naming construct in C. By Chapter 17 it'll handle `void`, `_Bool`, `char`, `short`, `long`, `signed`, `unsigned`, `float`, `double`, `enum`, `struct`, `union`, `typedef`, and the various arrangement rules ("any order: `unsigned long int` and `int unsigned long` mean the same thing"). Today it's a one-keyword stub.

`declarator` parses the rest of a single variable's declaration: zero or more `*`s, followed by an identifier. It takes a base type (the result of `declspec`) and returns the full type, with pointers wrapped on as needed:

```c
// declarator = "*"* ident
static Type *declarator(Token **rest, Token *tok, Type *ty) {
  while (consume(&tok, tok, "*"))
    ty = pointer_to(ty);

  if (tok->kind != TK_IDENT)
    error_tok(tok, "expected a variable name");

  ty->name = tok;
  *rest = tok->next;
  return ty;
}
```

The `consume` we just defined is doing the loop work. Each `*` in the input wraps the current type in another `pointer_to`. So `int **x` walks: start with `int`; consume `*` → "pointer to int"; consume `*` → "pointer to pointer to int"; bind name `x`. The token of the identifier is stored on the type's `name` field.

C's declarator syntax is famously knotty — the part that makes pointer-to-array different from array-of-pointers, the part that makes function declarators do their right-to-left dance — and chibicc isn't yet wading into any of that. The current declarator handles only `*`s and a name. By Chapter 6, arrays will appear; by Chapter 5, function definitions will reuse this parser; by Chapter 10, the full C declarator zoo (including the parenthesized-name trick) will land. For now: `*`s, then a name.

`declaration` is the top-level entry, handling commas and optional initializers:

```c
// declaration = declspec (declarator ("=" expr)? ("," declarator ("=" expr)?)*)? ";"
static Node *declaration(Token **rest, Token *tok) {
  Type *basety = declspec(&tok, tok);

  Node head = {};
  Node *cur = &head;
  int i = 0;

  while (!equal(tok, ";")) {
    if (i++ > 0)
      tok = skip(tok, ",");

    Type *ty = declarator(&tok, tok, basety);
    Obj *var = new_lvar(get_ident(ty->name), ty);

    if (!equal(tok, "="))
      continue;

    Node *lhs = new_var_node(var, ty->name);
    Node *rhs = assign(&tok, tok->next);
    Node *node = new_binary(ND_ASSIGN, lhs, rhs, tok);
    cur = cur->next = new_unary(ND_EXPR_STMT, node, tok);
  }

  Node *node = new_node(ND_BLOCK, tok);
  node->body = head.next;
  *rest = tok->next;
  return node;
}
```

Read it like this. We parse the `int` once, into `basety`. Then we loop until we see `;`, each iteration handling one declarator (after the first, we expect a comma to separate it from the previous one). Each declarator gives us a full `Type`, and we register a new local with `new_lvar`. If the declarator is followed by an `=`, we parse an initializer expression and emit an assignment; otherwise the variable is declared but not initialized in the AST. We collect the assignment statements into a block and return it.

Two things worth pausing on.

**Declarations with no initializer emit no code.** `int x;` registers a new local in the symbol table but doesn't generate any AST. It's a parse-time-only construct. The only run-time effect is that the function's frame ends up bigger because there's an extra `Obj` in the locals list and `assign_lvar_offsets` will reserve a slot for it.

**Initializers are syntactic sugar for assignments.** `int x = 3;` is parsed as if the user had written `int x; x = 3;` — declare `x`, then assign `3` to it. The `=` in the declaration produces an `ND_ASSIGN` wrapped in an `ND_EXPR_STMT`, exactly the AST we'd build for the explicit assignment form. The result is that codegen has nothing new to learn: it sees an `ND_BLOCK` containing zero or more `ND_EXPR_STMT`s (one per initialized declarator), and it already knows how to walk a block.

This is canonicalization-at-parse-time again, and it's a particularly nice instance because it's *invisible* — there's no source form `int x; x = 3;` that's preferred over `int x = 3;` or vice versa, and a reader of the AST can't tell which the user wrote. The parser collapses the two surface syntaxes into one tree.

There's one more small wrinkle. When a declaration list has no initializers — say `int x, y;` — every declarator falls into the `if (!equal(tok, "=")) continue;` branch and the loop doesn't add any statements to the head list. The function returns an empty `ND_BLOCK`. That's fine — `gen_stmt`'s `ND_BLOCK` case loops over an empty body and emits nothing — but it's the same trick we used for the null statement in §3.6. Nodes that mean "do nothing" are blocks with no body.

### Wiring it into `compound_stmt`

The compound-statement parser learns to recognize declarations:

```diff
-// compound-stmt = stmt* "}"
+// compound-stmt = (declaration | stmt)* "}"
 static Node *compound_stmt(Token **rest, Token *tok) {
   Node *node = new_node(ND_BLOCK, tok);

   Node head = {};
   Node *cur = &head;
   while (!equal(tok, "}")) {
-    cur = cur->next = stmt(&tok, tok);
+    if (equal(tok, "int"))
+      cur = cur->next = declaration(&tok, tok);
+    else
+      cur = cur->next = stmt(&tok, tok);
     add_type(cur);
   }
```

If the next token is `int`, we have a declaration; otherwise it's an ordinary statement. The two branches are co-equal — declarations and statements can be freely intermixed inside a block, the way they can be in C99 and later. (C89 required declarations to come before statements; chibicc doesn't enforce that.)

### The auto-create path becomes an error

```diff
   if (tok->kind == TK_IDENT) {
     Obj *var = find_var(tok);
     if (!var)
-      var = new_lvar(strndup(tok->loc, tok->len));
+      error_tok(tok, "undefined variable");
     *rest = tok->next;
     return new_var_node(var, tok);
   }
```

Three lines change in `primary`. The "if you haven't seen it, make one up" branch is replaced with an error. From this commit onward, every variable use is checked against the symbol table, and an unrecognized identifier is a hard error pointing at the misspelled name.

The shift is a conceptually large one even though the code change is tiny. Through Chapters 3 and 4.1–4.3, chibicc has had a frontier-style approach to symbol management: see a name, register a name. That's only possible because there was nothing to declare — every variable was an `int` of unspecified type, and a declaration would have been redundant. Now that declarations are mandatory, the symbol table is built explicitly, and the parser can give a useful error when something goes wrong. This is the language taking on more of C's discipline as it gains the type system to support it.

### A small helper: `get_ident`

```c
static char *get_ident(Token *tok) {
  if (tok->kind != TK_IDENT)
    error_tok(tok, "expected an identifier");
  return strndup(tok->loc, tok->len);
}
```

The `strndup` to allocate a name out of a token used to be inlined in `primary`. With declarations, we now allocate names from two places (`primary` for variable references, `declaration` for new declarations), and the duplication-plus-validation idiom is worth a name. It's a tiny refactor, but the kind that the chapter's other changes make natural — once we're touching the symbol table from a new place, there's value in unifying how identifiers are extracted.

The `pointer_to` declaration also gets exported in `chibicc.h`:

```diff
 bool is_integer(Type *ty);
+Type *pointer_to(Type *base);
 void add_type(Node *node);
```

It was static-to-`type.c` in the previous commit; now `parse.c` calls it (in `declarator`'s `*` loop), so it has to be visible across files.

### Where we are

The compiler is at the end of Chapter 4. Programs declare their variables; pointer types are real; pointer arithmetic scales correctly; and every undeclared identifier produces a useful error. The type system is two kinds wide and recursively deep. It's enough to express what chibicc currently understands, and the framework is in place for everything that comes next: arrays will piggyback on the `base` field, `char` will ride into `declspec` alongside `int`, and structs will eventually get their own kind alongside `TY_INT` and `TY_PTR`.

The architecture has earned its keep. `tokenize.c` handles `int` by adding one entry to a list. `parse.c` handles declarations entirely on its own — the AST grew no new node kinds for this commit. `codegen.c` doesn't change at all, three commits in a row now: pointer-aware code generation is an emergent property of the parser desugaring and the type system, not anything the emitter has to know about.

---

## Recap

| Commit | What it added |
|---|---|
| `3d86277` | `Token *tok` on every `Node`; `error_tok` in codegen's three error paths |
| `863e2b8` | `&` and `*`; `ND_ADDR`, `ND_DEREF`; lvalue-deref via `gen_addr` |
| `a6bc4ab` | `Type`, `ty_int`, `pointer_to`, `add_type`; `new_add`/`new_sub` desugar pointer arithmetic |
| `b4e82cf` | `int` keyword; `declspec`/`declarator`/`declaration`; mandatory declarations; `consume` helper |

Four commits, three modules touched, one new file (`type.c`). The chapter's center of gravity is the third commit — the one that introduces the type system — and it's the commit where the parser does the most subtle work, rewriting `+` and `-` into different shapes depending on what they're operating on. The other three commits are small. The first puts diagnostic plumbing in place; the second adds two unary operators that are mostly straightforward at the assembly level; the fourth adds declarations on top of the type system the third commit built.

Chapter 5 turns to functions. Chibicc currently has exactly one function in every program — `main`, implicit, with no formal definition. The next four commits introduce zero-argument calls, calls with up to six arguments, and finally function *definitions*, so a chibicc program can have more than one function. The frame layout from the §3 interlude will get a sequel about argument-passing registers, and we'll see how the SysV calling convention's "first six args in registers, rest on the stack" rule gets enforced (chibicc never enforces "rest on the stack" — six arguments is the cap — but the principle's the same). It's a chapter where the calling convention from the earlier interlude finally gets its second use: not just for `main`'s entry/exit, but for actual call instructions inside the body of a function.
