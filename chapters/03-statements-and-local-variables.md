# Chapter 3 — Statements and local variables

> Commits covered: `76cae0a`, `1f9f3ad`, `482c26b`, `6cc1c1f`, `18ac283`, `ff8912c`, `72b8415`, `f5d480f`, `1f3eb34`. Nine feature commits plus one administrative commit (`5b142b1`) that adds a LICENSE file.

After Chapter 2 we had a calculator. The compiler accepted one expression on its command line, evaluated it, and exited with the value as the process status. Chapter 3 turns that into a small imperative language. By the end of it chibicc compiles programs that look like this:

```c
{ a = 3; b = 5; if (a < b) return a; else return b; }
```

— programs that have variables, blocks, conditional branches, and loops; programs that look, from a careful enough distance, like real C. The architecture stays the three-pass shape settled in Chapter 2, and almost every commit in this chapter expands all three passes by a small amount each. That's the cadence Rui has been working toward, and Chapter 3 is the first chapter where we feel it: ten consecutive commits, none of them particularly big, each one moving the language forward by one carefully-chosen feature.

The chapter has one concept interlude, on the System V AMD64 stack frame. It's placed between §3.1 and §3.2, right before chibicc starts emitting prologue and epilogue code for the first time. Everything from §3.2 onward depends on understanding what `%rbp`, `%rsp`, push, pop, and "negative offset" mean to the machine, and the interlude exists to give us that vocabulary.

---

## 3.1 — Many statements

> `git checkout 76cae0ad05b6ba3e3e927b2b749ccddda23f0c51` — *Accept multiple statements separated by semicolons*

The first commit of the chapter promotes the language from one expression to a sequence of expressions, each terminated by a semicolon. Programs like `1; 2; 3;` now parse, run, and exit with the value of the last expression — `3`.

Three small grammar additions and a change to the codegen entry point are all it takes.

### Grammar

```
program    = stmt*
stmt       = expr-stmt
expr-stmt  = expr ";"
```

Three new non-terminals, but only one of them is doing real work. `program` is a `stmt*`, which means "zero or more statements." `stmt` is just an alias for `expr-stmt`, an indirection we'll start filling in immediately. `expr-stmt` is the actual new construct: an expression followed by a semicolon.

In C, *most* statements are expression statements. Function calls, assignments, increments, even bare expressions like `42;` (legal; computes 42 and discards it) — all of these are `expr ";"`. By naming the form now, before we have anything else, we set up the spot where future statement forms (`if`, `for`, `return`) will plug in.

### The new node kind and a `next` pointer

```diff
 typedef enum {
   ND_ADD,       // +
   ...
   ND_LE,        // <=
+  ND_EXPR_STMT, // Expression statement
   ND_NUM,       // Integer
 } NodeKind;
 
 struct Node {
   NodeKind kind;
+  Node *next;
   Node *lhs;
   Node *rhs;
   int val;
 };
```

`ND_EXPR_STMT` wraps an expression to mark it as "evaluate this and throw the result away." The wrapped expression goes in `lhs`. Why bother with the wrapper? Because code generation wants to know *kind of node* this is — an expression that contributes to a larger expression, or a statement-level expression whose value we discard. The wrapper makes that explicit, and it'll make even more sense in a few commits when we add other statement kinds.

The `Node *next` field is the second piece. Statements are chained via `next`, exactly the way tokens are chained. A program is a head-to-tail list of statement nodes.

### Parsing

Two new functions:

```c
// stmt = expr-stmt
static Node *stmt(Token **rest, Token *tok) {
  return expr_stmt(rest, tok);
}

// expr-stmt = expr ";"
static Node *expr_stmt(Token **rest, Token *tok) {
  Node *node = new_unary(ND_EXPR_STMT, expr(&tok, tok));
  *rest = skip(tok, ";");
  return node;
}
```

`stmt` delegates to `expr_stmt`. `expr_stmt` parses an expression, wraps it in `ND_EXPR_STMT`, and consumes the trailing semicolon via `skip`. If there's no semicolon, `skip` reports the error and exits.

`parse` is rewritten to be a list-builder rather than a single-expression parser:

```c
// program = stmt*
Node *parse(Token *tok) {
  Node head = {};
  Node *cur = &head;
  while (tok->kind != TK_EOF)
    cur = cur->next = stmt(&tok, tok);
  return head.next;
}
```

Same dummy-head pattern we saw in the tokenizer back in §1.3. Each statement we parse becomes the next link in the chain. The function returns `head.next` — the first real statement.

### Code generation

```c
static void gen_stmt(Node *node) {
  if (node->kind == ND_EXPR_STMT) {
    gen_expr(node->lhs);
    return;
  }
  error("invalid statement");
}

void codegen(Node *node) {
  printf("  .globl main\n");
  printf("main:\n");

  for (Node *n = node; n; n = n->next) {
    gen_stmt(n);
    assert(depth == 0);
  }

  printf("  ret\n");
}
```

`gen_stmt` is the new dispatch for statement-level nodes. Right now it knows exactly one kind — `ND_EXPR_STMT`, which it generates by running its `lhs` expression. The result of `gen_expr` lives in `%rax`, where it gets overwritten by the next statement's evaluation.

`codegen` walks the linked list of statements, calling `gen_stmt` on each. The `assert(depth == 0)` invariant moves *inside* the loop — we now check it after every statement, not just at the very end. Since each statement is supposed to leave the stack balanced, any leak shows up in the statement that caused it rather than at the end of the whole program.

### What ends up in `%rax`

The compiled output for `1; 2; 3;` is:

```
  .globl main
main:
  mov $1, %rax
  mov $2, %rax
  mov $3, %rax
  ret
```

Each statement evaluates and leaves its value in `%rax`. Since the only thing we do between statements is overwrite `%rax`, the last statement's value is what survives. The program exits with status 3.

This is not how real C works — real C treats `1; 2; 3;` as three statements that compute and discard three values, with no notion of "last value becomes the function's return." But the test suite is built around exit codes, and "the last expression's value becomes the exit code" is a useful informal convention while we don't have an actual `return` keyword. We'll get one in §3.4, and at that point the implicit "last expression wins" behavior will become a fallback rather than the primary mechanism.

### Where we are

Programs are now sequences of statements. The existing tests are rewritten to append a `;` to every expression, plus one new test for multi-statement programs. The grammar has a `stmt` slot that's about to absorb a long line of new statement forms. The next commit fills the slot with the language's first stateful feature: variables.

---

## Concept interlude — The System V AMD64 stack frame

From this point forward, almost every codegen change in the chapter touches the stack. Variables live there. Function prologues set it up. Returns tear it down. Loops don't use it directly, but their conditions and increments do. A reader who isn't comfortable with what `push %rbp`, `mov %rsp, %rbp`, and `lea -8(%rbp), %rax` mean to the machine will struggle with the next commit. So we pause for vocabulary.

Two pieces of background. The first is the *call stack* itself: a region of memory the operating system gives every thread, used to hold function-call state. On x86-64 Linux and macOS, the stack starts near the top of the address space and grows *downward* — pushing onto it decrements the stack pointer; popping increments it. There is one stack pointer register, `%rsp`, that always holds the address of the most recent push.

The second is the *calling convention* that all C compilers on this platform agree to follow. The version chibicc targets is the System V AMD64 ABI — the same one GCC, Clang, and the Linux kernel use — and chibicc happens to be a strict subset of it. Two of its rules matter for us right now:

1. **The function's return value lives in `%rax`.** We knew this from Chapter 1. It's what made `mov $42, %rax; ret` exit a process with status 42.
2. **The frame pointer, `%rbp`, points at a stable anchor inside the current function's frame.** While `%rsp` moves as the function pushes and pops, `%rbp` stays put. Local variables are addressed as fixed offsets from `%rbp`, not from `%rsp`, so the offsets don't shift when the stack pointer moves around.

A function's *frame* — the chunk of stack belonging to one in-flight call — has a fixed shape. On entry to a function, the stack looks like this (addresses high at the top, low at the bottom — the way the stack grew):

```
  ┌────────────────────────────┐  high addresses
  │ caller's frame             │
  ├────────────────────────────┤
  │ return address             │  ← pushed by `call`
  ├────────────────────────────┤
  │ saved %rbp                 │  ← we push this in the prologue
  ├────────────────────────────┤
  │ local variables...         │  ← addressed as -8(%rbp), -16(%rbp), ...
  ├────────────────────────────┤
  │                            │  ← %rsp after `sub $N, %rsp`
  └────────────────────────────┘  low addresses
```

The standard prologue, the three instructions every chibicc-emitted function will start with from §3.2 onward, builds this layout:

```
  push %rbp           # save the caller's %rbp on the stack
  mov %rsp, %rbp      # set our %rbp to point at the saved value
  sub $N, %rsp        # reserve N bytes of stack space for our locals
```

After those three instructions, `%rbp` points exactly at the saved-`%rbp` slot. Any local variable at offset `-k` from `%rbp` lives in the chunk we just reserved. The negative offsets are not arbitrary — they reflect the stack's downward growth. The first local goes to `-8(%rbp)`, the second to `-16(%rbp)`, and so on, eight bytes apart because everything chibicc stores is the size of a 64-bit integer.

The standard epilogue undoes all three steps in reverse:

```
  mov %rbp, %rsp      # discard the locals by restoring %rsp
  pop %rbp            # restore the caller's %rbp
  ret                 # pop the return address and jump to it
```

After the epilogue, the stack looks the way it did at the moment of the `call`. The caller resumes as if nothing happened — except `%rax` now holds our return value.

Two questions are worth answering before we go on.

**Why do we need both `%rbp` and `%rsp`?** A clever compiler can drop `%rbp` and address locals as offsets from `%rsp` directly — this is what GCC does at higher optimization levels under `-fomit-frame-pointer`. The savings are one register and three instructions per function. The cost is that the offsets shift every time the function pushes or pops, so the compiler needs to track stack depth precisely at every point in the function. chibicc keeps `%rbp` for the same reason it does everything else: simplicity. The frame pointer is a fixed reference, so a local's address is a constant displacement, regardless of what the stack pointer is doing right now.

**What about 16-byte alignment?** The System V AMD64 ABI requires `%rsp` to be a multiple of 16 at the moment of a `call` instruction (the callee then sees `%rsp` shifted by 8 because the call pushed a return address — the callee's first instruction sees `%rsp ≡ 8 (mod 16)`). chibicc doesn't make any function calls yet, so it doesn't have to align anything. But §3.3 will round the stack-allocation size up to a multiple of 16 anyway, in anticipation of the day chibicc starts generating call instructions. We'll note it when we get there. The takeaway for now is that 16-byte alignment is the rule on x86-64 SysV, even when it doesn't yet matter.

That's the model. The next commit puts it to work.

---

## 3.2 — Single-letter local variables

> `git checkout 1f9f3adf324af1432a380b41c7690834e649e346` — *Support single-letter local variables*

Programs gain identifiers. Every lowercase letter `a` through `z` becomes a usable local variable, and assignment becomes a real expression form. After this commit, the test suite includes programs like:

```c
a = 3; a;            // exits with 3
a = 3; b = 5; a + b; // exits with 8
```

The change spreads across all three modules — tokenizer, parser, codegen — but it's small in each one.

### Tokenizer: a new kind, a single character

```diff
 typedef enum {
-  TK_PUNCT, // Keywords or punctuators
+  TK_IDENT, // Identifiers
+  TK_PUNCT, // Punctuators
   TK_NUM,
   TK_EOF,
 } TokenKind;
```

```c
// Identifier
if ('a' <= *p && *p <= 'z') {
  cur = cur->next = new_token(TK_IDENT, p, p + 1);
  p++;
  continue;
}
```

A single lowercase letter becomes a one-character `TK_IDENT` token. No multi-letter handling, no underscores, no digits — that arrives in the next commit. For now, every variable is one letter long, which means the language has at most twenty-six variables. We'll see in a moment why that's exactly the right number for the codegen trick this commit pulls.

### AST: two new node kinds, one new field

```diff
 typedef enum {
   ...
   ND_LE,
+  ND_ASSIGN,    // =
   ND_EXPR_STMT,
+  ND_VAR,       // Variable
   ND_NUM,
 } NodeKind;
 
 struct Node {
   NodeKind kind;
   Node *next;
   Node *lhs;
   Node *rhs;
+  char name;     // Used if kind == ND_VAR
   int val;
 };
```

`ND_VAR` represents a use of a variable. The `name` field stores its single-character identifier as a `char`. `ND_ASSIGN` represents `lhs = rhs` — a binary operator like the others, but with a different code-gen pattern (more on that in a moment).

### Parser: assignment, and a new bottom for the precedence chain

```c
static Node *new_var_node(char name) {
  Node *node = new_node(ND_VAR);
  node->name = name;
  return node;
}
```

Standard new-node helper.

The grammar changes at two points. First, `expr` no longer goes directly to `equality`; it goes through `assign`:

```c
// expr = assign
static Node *expr(Token **rest, Token *tok) {
  return assign(rest, tok);
}

// assign = equality ("=" assign)?
static Node *assign(Token **rest, Token *tok) {
  Node *node = equality(&tok, tok);
  if (equal(tok, "="))
    node = new_binary(ND_ASSIGN, node, assign(&tok, tok->next));
  *rest = tok;
  return node;
}
```

`assign` parses an `equality` expression, then optionally a `=` followed by another `assign`. The recursion is on the right — that's how we get *right associativity*. `a = b = 3` parses as `a = (b = 3)`, not `(a = b) = 3`. The recursive call on the RHS keeps consuming as long as it sees `=`s, so a long chain `a = b = c = 3` builds nested ND_ASSIGN nodes from the right.

Why does `=` belong below equality in the precedence chain? Because in C, `a = b == c` means `a = (b == c)`, not `(a = b) == c`. Equality binds tighter than assignment. By placing `assign` *above* `equality` in the call chain, we get that precedence for free — when `assign` calls `equality(&tok, tok)`, equality has already consumed `b == c` before `assign` ever sees the `=`.

Second, `primary` learns about identifiers:

```c
// primary = "(" expr ")" | ident | num
static Node *primary(Token **rest, Token *tok) {
  if (equal(tok, "(")) { ... }

  if (tok->kind == TK_IDENT) {
    Node *node = new_var_node(*tok->loc);
    *rest = tok->next;
    return node;
  }

  if (tok->kind == TK_NUM) { ... }

  error_tok(tok, "expected an expression");
}
```

If we see an identifier, we make an `ND_VAR` node holding its character. `*tok->loc` reads the single character at the token's source position. We don't validate that the character is `a..z` here — the tokenizer has already enforced that.

### Code generation: lvalues and rvalues

This is where the language's first interesting code-gen idea appears: the distinction between *lvalues* and *rvalues*. The same `ND_VAR` node has two completely different meanings depending on context. On the right of an `=`, `a` means "the value stored at `a`'s address." On the left of an `=`, `a` means "`a`'s address itself, so I can store something there." Code generation has to handle these two senses with two different emission patterns.

`gen_addr` is the new helper that handles the address sense:

```c
static void gen_addr(Node *node) {
  if (node->kind == ND_VAR) {
    int offset = (node->name - 'a' + 1) * 8;
    printf("  lea %d(%%rbp), %%rax\n", -offset);
    return;
  }

  error("not an lvalue");
}
```

`lea` ("load effective address") computes an address and stores it in a register without dereferencing. So `lea -8(%rbp), %rax` puts the address `%rbp - 8` into `%rax` — useful when you want to *write* to that location later.

The offset formula deserves a moment. `(node->name - 'a' + 1) * 8` maps `'a'` to `1`, `'b'` to `2`, …, `'z'` to `26`, multiplied by 8 to get bytes. Then we negate to get a negative offset from `%rbp`. So `a` lives at `-8(%rbp)`, `b` at `-16(%rbp)`, …, `z` at `-208(%rbp)`. Twenty-six locals, eight bytes each, two hundred and eight bytes total — which is exactly the number we'll see in the prologue:

```c
printf("  push %%rbp\n");
printf("  mov %%rsp, %%rbp\n");
printf("  sub $208, %%rsp\n");
```

Every program reserves stack space for all twenty-six possible locals, used or not. This is grossly wasteful — the actual programs in the test suite use one or two — but it's the simplest possible scheme: no symbol table, no scoping, no allocation logic. The address of any variable is computable from its name alone.

This trick won't survive the next commit (which introduces real names) but it's a beautiful illustration of the chibicc design philosophy. Rui's README defends slow algorithms when n is small: when `n` is twenty-six, there's no need for cleverness. Reserve the space and move on.

The `gen_expr` switch grows two cases:

```c
case ND_VAR:
  gen_addr(node);
  printf("  mov (%%rax), %%rax\n");
  return;

case ND_ASSIGN:
  gen_addr(node->lhs);
  push();
  gen_expr(node->rhs);
  pop("%rdi");
  printf("  mov %%rax, (%%rdi)\n");
  return;
```

The `ND_VAR` case is the rvalue path: compute the address into `%rax`, then dereference it (`mov (%rax), %rax`) to load the stored value. After this sequence `%rax` holds the variable's value.

The `ND_ASSIGN` case is more delicate. We want the side effect *and* a result — `a = 3` both stores `3` into `a` and produces `3` as the expression's value (so `b = a = 3` works). The pattern: compute the lhs's *address* (not value), push it; compute the rhs's value into `%rax`; pop the address into `%rdi`; store `%rax` to `(%rdi)`. After this sequence, the assignment's stored, and the rhs's value is still in `%rax` — exactly what the next operator up the tree expects.

A subtle ordering point: we evaluate the lhs's address *before* the rhs's value. C's standard says the order of evaluation around `=` is unspecified, so chibicc's choice is conformant. But when chibicc later compiles itself, that choice silently becomes part of chibicc's observable behavior. Worth knowing about.

Finally, `codegen` gains the prologue and epilogue from the interlude:

```c
void codegen(Node *node) {
  printf("  .globl main\n");
  printf("main:\n");

  // Prologue
  printf("  push %%rbp\n");
  printf("  mov %%rsp, %%rbp\n");
  printf("  sub $208, %%rsp\n");

  for (Node *n = node; n; n = n->next) { ... }

  printf("  mov %%rbp, %%rsp\n");
  printf("  pop %%rbp\n");
  printf("  ret\n");
}
```

The prologue runs before any statement; the epilogue runs after all statements. We reserve 208 bytes of stack on entry and release them on exit, regardless of how many variables the program actually uses.

### Where we are

The compiler now has variables. Programs can store and read intermediate results. The implementation is short on cleverness — every program reserves all 26 local-variable slots whether they're used or not, and variable names are restricted to a single character — but the *shape* of the codegen is correct. `gen_addr` for lvalues, `mov (%rax), %rax` for reading, push-and-store for writing. None of that pattern will change when the next commit makes variables less primitive.

---

## 3.3 — Multi-letter local variables

> `git checkout 482c26b536f8e5c998af6210470cd3d97a47ee9a` — *Support multi-letter local variables*

The single-character-identifier shortcut is up. This commit replaces it with a real symbol-table mechanism: identifiers can be any length, the parser tracks declared locals in a list, the codegen assigns offsets based on actual usage, and the stack frame is sized to hold exactly the locals each program uses.

Three structural changes happen at once.

### A new type for variables

```c
// Local variable
typedef struct Obj Obj;
struct Obj {
  Obj *next;
  char *name;
  int offset;  // Offset from RBP
};
```

`Obj` is the symbol-table entry for a local variable. It has a `name` (the identifier as a heap-allocated string), an `offset` (from `%rbp`), and a `next` pointer threading it into a list.

The name `Obj` is a hint at where this is going. For now `Obj` represents only local variables, but the same struct will eventually represent globals, functions, and string literals — anything that has a name and lives somewhere in memory or in the program. The first time we use a name that's slightly more general than what we currently need is in this commit. The generality will pay off across many later chapters.

### A new type for functions

```c
typedef struct Function Function;
struct Function {
  Node *body;
  Obj *locals;
  int stack_size;
};
```

A `Function` is a parsed program: a body (a chain of statements), a list of locals, and a frame size that codegen will fill in. `parse` now returns a `Function *` instead of a `Node *`, and `codegen` consumes one. There's still only ever one function in a program (we won't get function definitions until Chapter 5), but the structure is reaching for that future.

### `Node`'s `var` field

```diff
 struct Node {
   NodeKind kind;
   Node *next;
   Node *lhs;
   Node *rhs;
-  char name;     // Used if kind == ND_VAR
+  Obj *var;      // Used if kind == ND_VAR
   int val;
 };
```

An `ND_VAR` node now points at its `Obj` rather than carrying a single character. The variable's name and offset live on the `Obj`, where they can be shared by every reference to the same variable.

### Tokenizer: real identifier rules

Two character classifiers replace the single-letter test:

```c
// Returns true if c is valid as the first character of an identifier.
static bool is_ident1(char c) {
  return ('a' <= c && c <= 'z') || ('A' <= c && c <= 'Z') || c == '_';
}

// Returns true if c is valid as a non-first character of an identifier.
static bool is_ident2(char c) {
  return is_ident1(c) || ('0' <= c && c <= '9');
}
```

`is_ident1` accepts letters and underscore — the legal first characters. `is_ident2` accepts those plus digits — the legal continuation characters. The split is C's rule: an identifier can't *start* with a digit, but can contain them after the first position.

The tokenizer's identifier branch becomes:

```c
if (is_ident1(*p)) {
  char *start = p;
  do {
    p++;
  } while (is_ident2(*p));
  cur = cur->next = new_token(TK_IDENT, start, p);
  continue;
}
```

Read one character that satisfies `is_ident1`, then keep reading as long as `is_ident2` holds. The token's `loc` and `len` cover the whole identifier.

### Parser: a symbol table

A new file-static list:

```c
// All local variable instances created during parsing are
// accumulated to this list.
Obj *locals;
```

Same pattern as `current_input` in the tokenizer: a global variable that holds parser state. When we eventually have multiple functions, this will be reset per function. For now it's just a single list growing as the parser sees new identifiers.

Two new helpers:

```c
static Obj *find_var(Token *tok) {
  for (Obj *var = locals; var; var = var->next)
    if (strlen(var->name) == tok->len && !strncmp(tok->loc, var->name, tok->len))
      return var;
  return NULL;
}

static Obj *new_lvar(char *name) {
  Obj *var = calloc(1, sizeof(Obj));
  var->name = name;
  var->next = locals;
  locals = var;
  return var;
}
```

`find_var` is a linear scan: walk the locals list looking for one whose name matches the token's source text. The match has to compare *both* the length and the contents — `var->name` is a null-terminated string, but `tok->loc` is a pointer into the source buffer with no terminator. Comparing length first short-circuits the common case where names obviously differ.

Linear scan? Yes. The README again: "Slow algorithms are fine if we know that n isn't too big." A function with 50 locals doing a 50-step scan on every identifier reference is fine on a modern laptop. By Chapter 22 chibicc will introduce a hash table for situations where `n` actually is big, but it isn't yet.

`new_lvar` creates an `Obj`, sets its name, and pushes it onto the front of the `locals` list. The push-onto-front choice makes appending O(1), but it means the list ends up in *reverse* declaration order — the most recently created lvar is the head. This will matter for offset assignment in a moment.

`primary`'s identifier case becomes:

```c
if (tok->kind == TK_IDENT) {
  Obj *var = find_var(tok);
  if (!var)
    var = new_lvar(strndup(tok->loc, tok->len));
  *rest = tok->next;
  return new_var_node(var);
}
```

Find the variable; if it doesn't exist, create it. `strndup(tok->loc, tok->len)` allocates a fresh null-terminated copy of the identifier, which the `Obj` then owns.

There are no declarations yet — variables are auto-created on first use. `a = 3` "declares" `a` by referring to it. This is a conscious simplification; real C requires `int a;` first. We'll get declarations in Chapter 7.

`parse`'s top-level changes to return a `Function`:

```c
// program = stmt*
Function *parse(Token *tok) {
  Node head = {};
  Node *cur = &head;
  while (tok->kind != TK_EOF)
    cur = cur->next = stmt(&tok, tok);

  Function *prog = calloc(1, sizeof(Function));
  prog->body = head.next;
  prog->locals = locals;
  return prog;
}
```

After parsing, the `locals` list is captured into the new `Function`, alongside the statement chain.

### Code generation: real offsets, real frame size

```c
static int align_to(int n, int align) {
  return (n + align - 1) / align * align;
}
```

A bit of integer arithmetic worth pausing on. `(n + align - 1) / align * align` rounds `n` up to the next multiple of `align`, assuming `align` is a power of two. The math: we add `align - 1` to push past the next multiple, then integer-divide by `align` (which floors), then multiply back. So `align_to(5, 8)` is `(5+7)/8*8 = 12/8*8 = 1*8 = 8`. And `align_to(11, 8)` is `(11+7)/8*8 = 18/8*8 = 2*8 = 16`. We'll use this everywhere alignment matters.

The new offset-assignment pass:

```c
static void assign_lvar_offsets(Function *prog) {
  int offset = 0;
  for (Obj *var = prog->locals; var; var = var->next) {
    offset += 8;
    var->offset = -offset;
  }
  prog->stack_size = align_to(offset, 16);
}
```

Walk the locals list (in head-first order, which is reverse declaration order), assign each one an offset of `-8`, `-16`, `-24`, and so on, then round the total up to the nearest 16. The function records its stack size on the `Function`, where codegen reads it for the prologue.

Two things worth noticing.

**The first-declared variable gets the largest negative offset.** Because `new_lvar` pushes onto the front and `assign_lvar_offsets` walks from the head, the most recently declared local gets `-8(%rbp)`. This is fine — there's no semantic meaning to which slot a variable lives in, and "more recent locals get smaller negative offsets" is no worse a rule than "earlier locals get smaller offsets." It is, however, the kind of detail you'd want to know if you were debugging chibicc's output by hand.

**The 16-byte alignment.** It pays for itself only when chibicc starts emitting `call` instructions. Right now, no `call` ever happens in our compiled programs, so the alignment is wasted. Rui adds it anyway because the cost is at most 8 wasted bytes per program. Cheaper than coming back later and remembering to add the `align_to`.

The codegen's `gen_addr` and prologue update accordingly:

```c
static void gen_addr(Node *node) {
  if (node->kind == ND_VAR) {
    printf("  lea %d(%%rbp), %%rax\n", node->var->offset);
    return;
  }
  error("not an lvalue");
}
```

Read the offset off the `Obj` rather than computing it from a character.

```c
void codegen(Function *prog) {
  assign_lvar_offsets(prog);

  printf("  .globl main\n");
  printf("main:\n");

  printf("  push %%rbp\n");
  printf("  mov %%rsp, %%rbp\n");
  printf("  sub $%d, %%rsp\n", prog->stack_size);

  for (Node *n = prog->body; n; n = n->next) { ... }

  printf("  mov %%rbp, %%rsp\n");
  printf("  pop %%rbp\n");
  printf("  ret\n");
}
```

The prologue's `sub` size is now per-program, not the hardcoded 208.

### One more thing: `_POSIX_C_SOURCE`

The very top of `chibicc.h` gains:

```c
#define _POSIX_C_SOURCE 200809L
```

`strndup` is a POSIX function, not standard C, and on glibc it's hidden behind a feature-test macro. Defining `_POSIX_C_SOURCE` to `200809L` (POSIX.1-2008) makes glibc reveal it. Without this define, building chibicc would warn about an implicit declaration of `strndup`, and on a strict C compiler would fail. The chosen value `200809L` is the standard way to ask for POSIX 2008 features.

`chibicc.h` also gains:

```c
typedef struct Node Node;
```

A forward declaration moved up so that the new `parse.c` section (which now references `Node *body` inside `struct Function`) can compile cleanly even before `struct Node` is defined further down. This is the kind of small bookkeeping that the modular split from Chapter 2 requires us to do consistently.

### Where we are

Programs can use as many locals as they want, with names of any length. The generated assembly reserves exactly the right amount of stack, and the symbol-table mechanism is in place for everything we'll add downstream — function parameters, globals, arrays, structs. The list-and-linear-scan approach is fast enough that we won't replace it for nineteen chapters.

---

## 3.4 — The `return` statement

> `git checkout 6cc1c1f0643ce0f1af0857e024a0a438ddb45853` — *Add "return" statement*

A program can now exit early with a chosen value. The grammar gains:

```
stmt = "return" expr ";"
     | expr-stmt
```

And the implementation introduces something we'll come to rely on: a token category called `TK_KEYWORD`, distinct from `TK_IDENT`, used for words that are reserved in the language even though they look like identifiers.

### Tokenizer: a second pass that reclassifies keywords

```diff
 typedef enum {
-  TK_IDENT,
-  TK_PUNCT,
-  TK_NUM,
-  TK_EOF,
+  TK_IDENT,
+  TK_PUNCT,
+  TK_KEYWORD,
+  TK_NUM,
+  TK_EOF,
 } TokenKind;
```

```c
static void convert_keywords(Token *tok) {
  for (Token *t = tok; t->kind != TK_EOF; t = t->next)
    if (equal(t, "return"))
      t->kind = TK_KEYWORD;
}

Token *tokenize(char *p) {
  ...
  cur = cur->next = new_token(TK_EOF, p, p);
  convert_keywords(head.next);
  return head.next;
}
```

The tokenizer doesn't know about keywords during its main loop. Every word that satisfies `is_ident1`/`is_ident2` becomes a `TK_IDENT`. After tokenization is done, a second pass walks the token list and reclassifies any token whose text is `"return"` as `TK_KEYWORD`.

Why a separate pass? Three reasons. First, keywords use the *exact same* lexical rules as identifiers — they look like identifiers to the eye, and the only thing that distinguishes them is being on a list. Special-casing them inside the main tokenizer loop would mean checking every `TK_IDENT` against the keyword list at lex time. That's fine; it's just less factored. Second, the keyword list is destined to grow — `if`, `else`, `for`, `while`, `int`, `return`, `sizeof`, and dozens more by the end of the book. Putting that list in one named function (`convert_keywords`) means there's one obvious place to consult to answer "what are chibicc's keywords?" Third, the parser doesn't strictly need `TK_KEYWORD` to be a different kind from `TK_IDENT` — `equal(tok, "return")` works regardless. The kind distinction matters in `primary`, which only treats `TK_IDENT` as a variable use. Without the reclassification, a stray `return` in expression position would be treated as a variable named "return," which is at best confusing and at worst silently wrong.

### Parser: one new branch in `stmt`

```c
// stmt = "return" expr ";"
//      | expr-stmt
static Node *stmt(Token **rest, Token *tok) {
  if (equal(tok, "return")) {
    Node *node = new_unary(ND_RETURN, expr(&tok, tok->next));
    *rest = skip(tok, ";");
    return node;
  }

  return expr_stmt(rest, tok);
}
```

If the current token is `"return"`, build an `ND_RETURN` node wrapping the expression after it, then demand a trailing semicolon. The expression goes into `lhs` (per the unary-operand convention from §1.6).

A small but important comment-grammar update: the `stmt` non-terminal's BNF comment now lists multiple alternatives separated by `|`. Every future statement form will add a line here. The discipline of keeping the grammar comment synchronized with the function continues.

### Codegen: a single common exit point

```c
static void gen_stmt(Node *node) {
  switch (node->kind) {
  case ND_RETURN:
    gen_expr(node->lhs);
    printf("  jmp .L.return\n");
    return;
  case ND_EXPR_STMT:
    ...
  }
  error("invalid statement");
}
```

`gen_stmt` becomes a switch (it was an `if` until now — promoted because we now have two kinds of statement). For `ND_RETURN`, evaluate the expression into `%rax`, then jump to `.L.return` — a label we emit just before the epilogue:

```diff
   for (Node *n = prog->body; n; n = n->next) { ... }
 
+  printf(".L.return:\n");
   printf("  mov %%rbp, %%rsp\n");
   printf("  pop %%rbp\n");
   printf("  ret\n");
 }
```

This is a useful pattern. Every `return` statement, no matter how deeply nested in the function, jumps to the same label. The epilogue is written once, not duplicated. The price is one unconditional jump per `return`, which the CPU's branch predictor handles well. The benefit is that adding new statement forms (like `if` and `for`, coming up) doesn't have to think about where the function exits — every exit goes through the same place.

### Where we are

Programs can return early. The keyword infrastructure is built, and the upcoming `if`, `for`, and `while` statements will plug into it without needing to extend the tokenizer beyond a one-line list change.

---

## 3.5 — Compound statements

> `git checkout 18ac283a5d19c19f1e1a7020a50fe34c2160a0f8` — *Add `{ ... }`*

Curly braces become a real statement form. After this commit, programs are required to start with `{`, and any statement context can contain a brace-delimited block. The grammar:

```
stmt          = "return" expr ";"
              | "{" compound-stmt
              | expr-stmt
compound-stmt = stmt* "}"
```

### A new node kind, a new field

```diff
 typedef enum {
   ...
   ND_RETURN,
+  ND_BLOCK,     // { ... }
   ND_EXPR_STMT,
   ...
 } NodeKind;
 
 struct Node {
   NodeKind kind;
   Node *next;
   Node *lhs;
   Node *rhs;
+
+  // Block
+  Node *body;
+
   Obj *var;
   int val;
 };
```

`ND_BLOCK` represents a brace-delimited sequence of statements. The new `body` field points at the first statement in the block; subsequent statements are reached via the existing `next` chain. So a block is a node whose `body` is the head of a stmt list.

### Parser: a new function and a top-level requirement

```c
// compound-stmt = stmt* "}"
static Node *compound_stmt(Token **rest, Token *tok) {
  Node head = {};
  Node *cur = &head;
  while (!equal(tok, "}"))
    cur = cur->next = stmt(&tok, tok);

  Node *node = new_node(ND_BLOCK);
  node->body = head.next;
  *rest = tok->next;
  return node;
}
```

Same dummy-head pattern as `parse` had before. Loop reading statements until we see `}`, then wrap the chain in an `ND_BLOCK`.

`stmt` gains a `{` branch:

```c
if (equal(tok, "{"))
  return compound_stmt(rest, tok->next);
```

— and `parse` is rewritten:

```c
Function *parse(Token *tok) {
  tok = skip(tok, "{");

  Function *prog = calloc(1, sizeof(Function));
  prog->body = compound_stmt(&tok, tok);
  prog->locals = locals;
  return prog;
}
```

Programs are now `{` followed by zero or more statements followed by `}`. The whole program body is one `ND_BLOCK`. This is starting to look like a function body, which is exactly the direction we're heading — when we add real function definitions in Chapter 5, the function body will be precisely a compound statement.

The test suite gets updated wholesale to wrap every existing program in `{}`. `assert 3 'a=3; a;'` becomes `assert 3 '{ a=3; a; }'`. Every test in the suite is rewritten in this commit.

### Codegen: a tiny new case

```c
case ND_BLOCK:
  for (Node *n = node->body; n; n = n->next)
    gen_stmt(n);
  return;
```

For a block, walk its body and generate each statement. That's it.

The top-level `codegen` simplifies because the program body is a single `ND_BLOCK` now:

```diff
-  for (Node *n = prog->body; n; n = n->next) {
-    gen_stmt(n);
-    assert(depth == 0);
-  }
+  gen_stmt(prog->body);
+  assert(depth == 0);
```

We no longer iterate top-level statements — we generate the one block, and recursion inside `gen_stmt(ND_BLOCK)` handles the iteration. The `assert(depth == 0)` moves out of the loop and runs once after the whole body, which is a slight regression in error-locality (a leak inside one statement no longer points at that statement). The trade is a simpler control flow at the top of `codegen`.

### Where we are

The language has nestable blocks. The grammar's recursive shape lets a block contain statements, including other blocks, to any depth. From here, statement forms that *contain* a substatement (like `if`'s body, or `while`'s body) get blocks for free — the substatement can be a single statement *or* a `{}` block, and the parser doesn't have to know which.

---

## 3.6 — The null statement

> `git checkout ff8912c68e877744f8b15070e098af786e7bd296` — *Add null statement*

The smallest commit in this chapter. Seven lines of parser, one line of test.

A "null statement" is just a semicolon by itself: a statement that does nothing. The grammar:

```
expr-stmt = expr? ";"
```

The `?` makes the expression optional. If we see `;` immediately, we have a null statement.

The implementation:

```c
// expr-stmt = expr? ";"
static Node *expr_stmt(Token **rest, Token *tok) {
  if (equal(tok, ";")) {
    *rest = tok->next;
    return new_node(ND_BLOCK);
  }

  Node *node = new_unary(ND_EXPR_STMT, expr(&tok, tok));
  *rest = skip(tok, ";");
  return node;
}
```

If the next token is `;`, return an empty `ND_BLOCK` (a block with no body). Why an empty block instead of, say, an `ND_NULL`? Because an empty block already means "do nothing" — `gen_stmt` for `ND_BLOCK` runs `for (Node *n = node->body; n; n = n->next)`, and when `body` is `NULL`, the loop runs zero times. No new node kind, no new codegen case, no new logic. Reusing `ND_BLOCK` is the small-duplication choice that Rui explicitly defends in the README — and here the duplication is so small there's almost nothing to feel cheap about.

The test suite gains a single test:

```sh
assert 5 '{ ;;; return 5; }'
```

— three null statements followed by a real return. They emit zero instructions each.

There's a second reason the null statement matters that won't pay off until §3.8. The `for` statement's grammar will be `"for" "(" expr-stmt expr? ";" expr? ")" stmt`. Note that the initializer is an `expr-stmt`, which now allows a bare semicolon. That's how `for (;;)` becomes legal — the empty initializer slot is a null statement.

### Where we are

Statement, do nothing. One line of test.

---

## 3.7 — The `if` statement

> `git checkout 72b841508f562c65b427a502fe6b270c3717319b` — *Add "if" statement*

Conditional execution. The grammar:

```
stmt = "if" "(" expr ")" stmt ("else" stmt)?
     | ...
```

### AST: three new pointers

```diff
 typedef enum {
   ...
   ND_RETURN,
+  ND_IF,        // "if"
   ND_BLOCK,
   ...
 } NodeKind;
 
 struct Node {
   NodeKind kind;
   Node *next;
+
   Node *lhs;
   Node *rhs;
 
+  // "if" statement
+  Node *cond;
+  Node *then;
+  Node *els;
+
   // Block
   Node *body;
   ...
 };
```

`ND_IF` uses three pointers: `cond` for the condition expression, `then` for the body of the if-branch, and `els` for the optional else-branch. (The field is named `els` rather than `else` because `else` is a C keyword.)

This is the moment Rui's "everything in one struct" policy starts paying off (or starts to feel a little wide, depending on your taste). A `Node` now has `lhs`, `rhs`, `cond`, `then`, `els`, `body`, `var`, `val`, `name` (well, actually `name` is gone now; `var` replaced it) — eight fields, and we'll add more in the next two commits. Most of them are zero for any given node. From the README:

> We could save memory using unions, but I decided to simply put everything in the same struct instead. I believe the inefficiency is negligible.

The cost is real: an `ND_NUM` node has six unused pointer fields. The benefit is that *reading* the AST never requires figuring out which fields are valid. Every field is always there. The check `node->cond == NULL` is just a check, not an undefined operation.

### Parser: standard recursive-descent for `if`

```c
if (equal(tok, "if")) {
  Node *node = new_node(ND_IF);
  tok = skip(tok->next, "(");
  node->cond = expr(&tok, tok);
  tok = skip(tok, ")");
  node->then = stmt(&tok, tok);
  if (equal(tok, "else"))
    node->els = stmt(&tok, tok->next);
  *rest = tok;
  return node;
}
```

Skip the `if`, demand `(`, parse the condition, demand `)`, parse the body. Then optionally consume `else` and parse another statement. Because `then` and `els` both call `stmt`, an `if` body can be any single statement, including a `{}` block. That's how `if (cond) { ... }` works without the parser knowing anything special about brace-delimited if-bodies.

The dangling-else problem (`if (a) if (b) x; else y;` — does `else` go with the inner `if` or the outer?) doesn't need explicit handling here. Because `node->then = stmt(...)` recursively descends into the inner `if`, the inner `if` greedily consumes the `else`, attaching it to itself. That matches C's rule: `else` binds to the nearest unmatched `if`. We get the right behavior for free from how recursive descent works.

### Tokenizer: a small refactor of the keyword list

```c
static bool is_keyword(Token *tok) {
  static char *kw[] = {"return", "if", "else"};

  for (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)
    if (equal(tok, kw[i]))
      return true;
  return false;
}

static void convert_keywords(Token *tok) {
  for (Token *t = tok; t->kind != TK_EOF; t = t->next)
    if (is_keyword(t))
      t->kind = TK_KEYWORD;
}
```

The previous version of `convert_keywords` checked `equal(t, "return")` inline. Now there's an `is_keyword` helper backed by a static `kw[]` array. `sizeof(kw) / sizeof(*kw)` is the canonical C idiom for "number of elements in a fixed-size array" — it computes the array's size in bytes divided by the size of one element. We'll see this exact pattern dozens of times throughout chibicc.

Adding a keyword now just means appending to `kw[]`. The next two commits do exactly that.

### Codegen: a counter for unique labels

```c
static int count(void) {
  static int i = 1;
  return i++;
}
```

This little helper returns 1, 2, 3, … on successive calls. The static-local-int pattern is the standard C way to give a function a private counter. We'll use it whenever we need a new unique label number.

The `if` codegen:

```c
case ND_IF: {
  int c = count();
  gen_expr(node->cond);
  printf("  cmp $0, %%rax\n");
  printf("  je  .L.else.%d\n", c);
  gen_stmt(node->then);
  printf("  jmp .L.end.%d\n", c);
  printf(".L.else.%d:\n", c);
  if (node->els)
    gen_stmt(node->els);
  printf(".L.end.%d:\n", c);
  return;
}
```

Standard pattern. Compute the condition. Compare to zero. Jump to `.L.else` if equal (false). Otherwise execute the then-body and unconditional-jump to `.L.end`. At `.L.else`, execute the else-body if there is one. End at `.L.end`.

The labels are unique to this `if` thanks to `count()`. A program with two `if`s gets `.L.else.1`/`.L.end.1` and `.L.else.2`/`.L.end.2`. Without the counter, every `if` would emit `.L.else`/`.L.end`, and the assembler would reject the duplicate labels.

A small note on layout: the `.L.else` label is emitted even when there's no else-body. That's fine — an unused label is just a comment as far as the running program is concerned. It costs zero instructions.

### Where we are

Programs can branch. The codegen pattern for control flow — emit unique labels, use `cmp` and conditional jumps — is in place, and we'll see slight variations of it for every loop form coming up.

---

## 3.8 — The `for` statement

> `git checkout f5d480f139592cc2670c2b05076c39b2fd6fe9b3` — *Add "for" statement*

A counted loop. Grammar:

```
stmt = "for" "(" expr-stmt expr? ";" expr? ")" stmt
     | ...
```

### AST: two more pointers

```diff
 typedef enum {
   ...
   ND_IF,
+  ND_FOR,       // "for"
   ND_BLOCK,
   ...
 } NodeKind;
 
 struct Node {
   ...
-  // "if" statement
+  // "if" or "for" statement
   Node *cond;
   Node *then;
   Node *els;
+  Node *init;
+  Node *inc;
   ...
 };
```

`ND_FOR` reuses `cond` and `then` from `ND_IF`, and adds `init` (the initializer, run once at loop entry) and `inc` (the increment, run after each iteration). The struct's comment hints at the sharing: `// "if" or "for" statement`.

### Parser: subtleties in the three slots

```c
if (equal(tok, "for")) {
  Node *node = new_node(ND_FOR);
  tok = skip(tok->next, "(");

  node->init = expr_stmt(&tok, tok);

  if (!equal(tok, ";"))
    node->cond = expr(&tok, tok);
  tok = skip(tok, ";");

  if (!equal(tok, ")"))
    node->inc = expr(&tok, tok);
  tok = skip(tok, ")");

  node->then = stmt(rest, tok);
  return node;
}
```

The three slots between the parentheses behave differently from each other.

The **init** slot is parsed as `expr_stmt`. This is the trick we set up in §3.6: an `expr_stmt` can be a bare `;`, which makes the empty initializer (`for (;;)`) parse cleanly. The parsed init is either an `ND_EXPR_STMT` wrapping a real expression or an empty `ND_BLOCK`.

The **cond** and **inc** slots are bare `expr`, with explicit handling of the optional case. If the next token isn't `;` (for cond) or `)` (for inc), we parse an expression; otherwise we leave the field `NULL`. The terminator (`;` or `)`) gets consumed unconditionally afterward.

Why parse init differently from cond and inc? A historical answer would be "C grammar puts a *declaration or expression-statement* in the init slot," but chibicc doesn't have declarations yet, so the only thing init can hold is an expression-statement. The difference is really just where the trailing semicolon goes — `expr_stmt` consumes its own `;`, so the parser doesn't have a separate `skip(tok, ";")` after init. For cond and inc the parser handles the terminator explicitly. It's a tiny inconsistency, but it keeps the code readable.

### Codegen: another counter, another label dance

```c
case ND_FOR: {
  int c = count();
  gen_stmt(node->init);
  printf(".L.begin.%d:\n", c);
  if (node->cond) {
    gen_expr(node->cond);
    printf("  cmp $0, %%rax\n");
    printf("  je  .L.end.%d\n", c);
  }
  gen_stmt(node->then);
  if (node->inc)
    gen_expr(node->inc);
  printf("  jmp .L.begin.%d\n", c);
  printf(".L.end.%d:\n", c);
  return;
}
```

Run the init once. Emit `.L.begin`. If there's a condition, evaluate and bail to `.L.end` if false. Run the body. Run the increment. Jump back to `.L.begin`. Emit `.L.end`.

The init is generated via `gen_stmt`, not `gen_expr`, because we parsed it as an `expr-stmt` (which might be either an expression-statement or an empty block). The cond and inc are generated via `gen_expr` because we parsed them as bare `expr` and they have values we don't care about saving.

If `cond` is `NULL`, we emit no test — the loop runs forever. `for (;;)` becomes:

```
.L.begin.1:
  <body>
  jmp .L.begin.1
.L.end.1:
```

— with `.L.end.1` left orphaned but harmless.

### Where we are

We have C's main loop form. The next commit shows that this is enough.

---

## 3.9 — The `while` statement

> `git checkout 1f3eb34f637520b01e6b8cd10a9026d05036db6d` — *Add "while" statement*

The smallest non-trivial commit in the chapter. `while` reuses `ND_FOR`.

### Parser: one new branch, no new node kind

```c
if (equal(tok, "while")) {
  Node *node = new_node(ND_FOR);
  tok = skip(tok->next, "(");
  node->cond = expr(&tok, tok);
  tok = skip(tok, ")");
  node->then = stmt(rest, tok);
  return node;
}
```

Parse `while (cond) stmt`. Build an `ND_FOR` node. Set `cond` and `then`. Leave `init` and `inc` as `NULL`. Done.

The codegen is unchanged in spirit — same `ND_FOR` case as before — but it gains one defensive check:

```diff
   case ND_FOR: {
     int c = count();
-    gen_stmt(node->init);
+    if (node->init)
+      gen_stmt(node->init);
     printf(".L.begin.%d:\n", c);
```

`while` produces an `ND_FOR` with `init == NULL`, so the codegen has to guard against the missing init. (The cond and inc were already guarded.)

### A short note on canonicalization

This is the second time we've seen Rui collapse two grammar forms into one AST node. The first was `>` and `>=` becoming swapped `<` and `<=` back in §1.7. Now `while` becomes a degenerate `for`. The pattern: when two source forms have the same semantics, pick one as canonical and rewrite the other into it at parse time. The codegen sees fewer cases and writes less code.

It's a small choice, but it's a cumulative one. Over the next twenty chapters, chibicc will collapse `do-while` into a `while`-shaped node, `+=` into an explicit `=` plus `+` rewrite, `[]` indexing into pointer arithmetic plus dereference, and so on. Each canonicalization keeps codegen simple at a small parse-time cost. By the time we get to the preprocessor, the AST will represent a much smaller language than C's surface syntax suggests.

The keyword list grows by one:

```diff
-  static char *kw[] = {"return", "if", "else", "for"};
+  static char *kw[] = {"return", "if", "else", "for", "while"};
```

### Where we are

Chapter 3 ends here, plus an administrative commit that adds a LICENSE and README to the repo (`5b142b1`) — no source changes, just project hygiene that has to land somewhere. We mention it for the recap and move on.

The compiler now compiles small imperative programs:

```c
{
  i = 0;
  sum = 0;
  for (i = 0; i < 10; i = i + 1) sum = sum + i;
  return sum;
}
```

— and produces something that runs and exits with 45.

The architecture is exactly what Chapter 2 set up. `tokenize.c` knows how to make tokens; `parse.c` knows how to build an AST; `codegen.c` knows how to walk the AST and emit assembly. None of the three files ballooned. The data types — `Token`, `Node`, `Obj`, `Function` — that we'll be using for the rest of the book are all in place, and from here, growing the language mostly means adding cases to existing switches and entries to existing lists.

---

## Recap

| Commit | What it added |
|---|---|
| `76cae0a` | Multi-statement programs, `;`, `ND_EXPR_STMT`, `Node *next` |
| `1f9f3ad` | Single-letter locals, `=`, `ND_VAR`, `ND_ASSIGN`, prologue/epilogue |
| `482c26b` | Multi-letter locals, `Obj`, `Function`, real symbol table, dynamic stack size |
| `6cc1c1f` | `return`, `TK_KEYWORD`, the `.L.return` exit point |
| `18ac283` | `{ ... }`, `ND_BLOCK`, programs become `{ stmt* }` |
| `ff8912c` | Empty `;` as a no-op statement (reuses `ND_BLOCK`) |
| `72b8415` | `if`/`else`, `ND_IF`, `count()` for unique labels |
| `f5d480f` | `for`, `ND_FOR`, optional init/cond/inc |
| `1f3eb34` | `while` (reuses `ND_FOR`) |
| `5b142b1` | LICENSE and README.md (administrative) |

In Chapter 1 we added 7 commits' worth of features and the file `main.c` grew from 15 lines to about 395. In this chapter we added 9 commits' worth of features, but the per-file growth is much smaller — `parse.c` ends around 300 lines, `codegen.c` and `tokenize.c` around 180 and 140 respectively. Spreading the work across three files keeps any one of them comprehensible. That's the dividend the Chapter 2 split was paying for; this is the chapter where it earns its keep.

The next chapter is a short one — four commits — but it introduces an idea that reshapes everything from here forward: types. Pointers come first, with `&` and `*`, and as soon as we have pointer arithmetic the language has to know what type things are. Chapter 4 starts that work.
