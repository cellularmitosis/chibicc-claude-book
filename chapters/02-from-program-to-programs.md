# Chapter 2 — From program to programs

> Commit covered: `725badf`. One commit, dated 2020-10-07.

Chapter 1 ended with a 395-line `main.c` divided by three section comments — `// Tokenizer`, `// Parser`, `// Code generator`. The compiler did three things in sequence and said as much in its source. Chapter 2 is the one-commit chapter that promotes those comments to file boundaries: `// Tokenizer` becomes `tokenize.c`, `// Parser` becomes `parse.c`, `// Code generator` becomes `codegen.c`, and a new header `chibicc.h` declares what each module exports to the others. `main.c` shrinks to eleven lines.

That is the entire commit. No new language feature, no new test case. The reader can be forgiven for asking why we're stopping a whole chapter on a refactor. The answer is that this is the architecture chibicc keeps for the rest of the book — the file boundaries laid down here will outlast every other organizational decision in the project. From the very next commit onward, "add a feature" will mean "edit two of these files and leave the third alone." That's the property the split exists to create, and it's worth a chapter to make sure we see it.

> `git checkout 725badfb494544b7c7f1d4c4690b9bc033c6d051` — *Split main.c into multiple small files*

---

## What changed at a glance

The commit's stat line is small enough to read whole:

```
 Makefile   |   8 +-
 chibicc.h  |  68 ++++++++++
 codegen.c  |  75 +++++++++++
 main.c     | 411 +------------------------------------------------------------
 parse.c    | 165 +++++++++++++++++++++++++
 tokenize.c | 107 ++++++++++++++++
 6 files changed, 425 insertions(+), 409 deletions(-)
```

Four hundred and nine lines deleted from `main.c`, four hundred and twenty-five added across four new files. The numbers are nearly equal because almost nothing here is *new* code — it's the same code, redistributed. The two genuinely new things are the header `chibicc.h` and a pair of entry-point functions, `parse()` and `codegen()`, that didn't exist before. The rest is rearrangement.

A small note on the commit's date that's worth getting out of the way. This commit is dated 2020-10-07, but the next commit on `main` (the one Chapter 3 starts from) is dated 2020-09-26 — eleven days *earlier*. The repository's history isn't strictly chronological. Rui has stated the policy explicitly in the README: "When I find a bug in this compiler, I go back to the original commit that introduced the bug and rewrite the commit history as if there were no such bug from the beginning." The same policy applies to pedagogical reordering — if the curriculum reads better with the file-split here, before the language gains state, that's where the file-split goes, regardless of when the work was originally typed. We mention this once and then trust the order. The book follows the curriculum order Rui chose, not the wall-clock order he actually wrote in.

## The header

`chibicc.h` is sixty-eight lines and falls into four parts: a block of standard-library includes, then three sections — one per `.c` file — declaring exactly what that file lets the rest of the compiler see.

```c
#include <assert.h>
#include <ctype.h>
#include <stdarg.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
```

These are the seven standard headers `main.c` had been including all along. Pulling them into `chibicc.h` means every `.c` file in the project gets them by including a single project header. We will not see `#include <stdio.h>` again at the top of any chibicc source file — only `#include "chibicc.h"`, which transitively pulls everything in. This is unusual style for production C, where you'd typically include only what each file needs, but it's deliberate here. Rui's README defends inefficiency-for-readability over and over; this is one more application of that principle, applied to compile time.

Then the public API of the tokenizer:

```c
//
// tokenize.c
//

typedef enum {
  TK_PUNCT, // Keywords or punctuators
  TK_NUM,   // Numeric literals
  TK_EOF,   // End-of-file markers
} TokenKind;

typedef struct Token Token;
struct Token {
  TokenKind kind;
  Token *next;
  int val;
  char *loc;
  int len;
};

void error(char *fmt, ...);
void error_at(char *loc, char *fmt, ...);
void error_tok(Token *tok, char *fmt, ...);
bool equal(Token *tok, char *op);
Token *skip(Token *tok, char *op);
Token *tokenize(char *input);
```

The `TokenKind` enum and `Token` struct are now public types, because the parser needs to read tokens. Six functions are exported: the three error helpers (`error`, `error_at`, `error_tok`), the two token-stream helpers (`equal`, `skip`), and the tokenizer entry point (`tokenize`). Notice what *isn't* exported: `verror_at`, `new_token`, `startswith`, `read_punct`, and the static variable `current_input` are all implementation details of the tokenizer — they remain `static` inside `tokenize.c` and don't appear in the header.

The comment `// Keywords or punctuators` next to `TK_PUNCT` is a small spoiler. Up through Chapter 1 the only punctuators were the operators we'd added; there were no keywords. The comment tells us that `TK_PUNCT` is going to absorb keywords too (`return`, `if`, `for`, …) when those arrive. We'll see this in Chapter 3. For now the comment is aspirational — there is still nothing in the language that's a keyword.

The parser's section is shorter:

```c
//
// parse.c
//

typedef enum {
  ND_ADD, ND_SUB, ND_MUL, ND_DIV, ND_NEG,
  ND_EQ,  ND_NE,  ND_LT,  ND_LE,
  ND_NUM,
} NodeKind;

typedef struct Node Node;
struct Node {
  NodeKind kind;
  Node *lhs;
  Node *rhs;
  int val;
};

Node *parse(Token *tok);
```

`NodeKind` and `Node` become public so that `codegen.c` can walk the tree, but only one function escapes the parser: `parse(Token *tok)`. We will come back to that in a moment.

The code generator's section is the smallest of all:

```c
//
// codegen.c
//

void codegen(Node *node);
```

One function. No types. The codegen has no public types because nothing downstream of it consumes typed C values — its output is plain text on `stdout`, a side effect that doesn't return anything to its caller.

Three sections. The order matches the order in which the modules will be used by `main`. A reader scrolling through `chibicc.h` from top to bottom is reading a one-page outline of the whole compilation pipeline.

## The new module boundaries

Splitting one file into three demands a question for every name in the original file: who needs to see this? Each answer becomes either an `extern` declaration in the header or a `static` keyword on the definition. The split is the moment we draw a line between API and implementation, function by function.

### `tokenize.c`

```c
#include "chibicc.h"

static char *current_input;

void error(char *fmt, ...) { ... }
static void verror_at(char *loc, char *fmt, va_list ap) { ... }
void error_at(char *loc, char *fmt, ...) { ... }
void error_tok(Token *tok, char *fmt, ...) { ... }

bool equal(Token *tok, char *op) { ... }
Token *skip(Token *tok, char *op) { ... }

static Token *new_token(TokenKind kind, char *start, char *end) { ... }
static bool startswith(char *p, char *q) { ... }
static int read_punct(char *p) { ... }

Token *tokenize(char *p) {
  current_input = p;
  ...
}
```

`current_input` was a file-static in the old `main.c`. It stays a file-static here — it's exactly the kind of state that should be private to the module that uses it. The error helpers (`verror_at`, `error_at`, `error_tok`) read `current_input` to compute column positions, so they live in the same file. The error helpers are *public* (they're called from the parser and elsewhere), but the variable they read is *private*. That's a clean design: the rest of the compiler can report errors without ever touching the input pointer directly.

The one signature change worth noticing: `tokenize` used to take no arguments and read from the global `current_input`; it now takes the input string as a parameter and assigns it to the global at the top of the function. This pushes the global-variable assignment from `main` (which used to write `current_input = argv[1];`) into the tokenizer itself. `main` no longer needs to know that there *is* a `current_input`. The public API is now `Token *tokenize(char *input)`, and the rest is private.

### `parse.c`

```c
#include "chibicc.h"

static Node *expr(Token **rest, Token *tok);
// ... six more forward declarations ...

static Node *new_node(NodeKind kind) { ... }
static Node *new_binary(NodeKind kind, Node *lhs, Node *rhs) { ... }
static Node *new_unary(NodeKind kind, Node *expr) { ... }
static Node *new_num(int val) { ... }

// expr = equality
static Node *expr(Token **rest, Token *tok) { ... }
// ... and so on for equality, relational, add, mul, unary, primary ...

Node *parse(Token *tok) {
  Node *node = expr(&tok, tok);
  if (tok->kind != TK_EOF)
    error_tok(tok, "extra token");
  return node;
}
```

Every parser non-terminal is `static`. Every node constructor is `static`. The forward declarations stay inside the file because they're internal — the parser's mutual recursion is its own business.

The new function is `parse(Token *tok)`. It's a four-line wrapper that does what `main` used to do inline:

1. Run the top-level grammar rule (`expr`).
2. Confirm the entire token stream was consumed (the `extra token` check).
3. Return the AST.

That's the whole parser API. The caller hands in a token list, gets back a tree. There's no way for `main` to accidentally call `relational` or build a `Node` directly — those names aren't visible outside `parse.c`. If the grammar grows a new top-level construct (statements, declarations, function definitions), `parse` is the function that changes; everyone else keeps calling `parse(tok)` the same way they always did.

### `codegen.c`

```c
#include "chibicc.h"

static int depth;

static void push(void) { ... }
static void pop(char *arg) { ... }
static void gen_expr(Node *node) { ... }

void codegen(Node *node) {
  printf("  .globl main\n");
  printf("main:\n");

  gen_expr(node);
  printf("  ret\n");

  assert(depth == 0);
}
```

Same shape as the parser. Everything is private except a single public entry point, `codegen(Node *node)`. It does the prologue (`.globl main` and the `main:` label), runs the recursive walk, emits the `ret`, and asserts the push/pop balance invariant introduced in Chapter 1. All four of those lived in `main.c` before; now they live with the rest of code generation.

The `depth` invariant is worth pausing on for one sentence. In Chapter 1 we saw it as a sanity check at the bottom of `main`. It's still doing the same job — pushes minus pops should be zero by the time we're done with the AST — but the assertion now sits next to the code that maintains the invariant rather than in a different file. If a future commit ever introduces a leak, the failure surfaces inside the module that owns it. That's good encapsulation.

### `main.c`

After everything is moved out, `main.c` becomes:

```c
#include "chibicc.h"

int main(int argc, char **argv) {
  if (argc != 2)
    error("%s: invalid number of arguments", argv[0]);

  Token *tok = tokenize(argv[1]);
  Node *node = parse(tok);
  codegen(node);
  return 0;
}
```

Eleven lines. Three of them are the pipeline: tokenize, parse, codegen. If you want a one-page mental model of what the chibicc compiler does, this is it.

It's worth dwelling on what this `main` *doesn't* know. It doesn't know there's a global `current_input`. It doesn't know the tokenizer uses linked lists or that the parser uses recursive descent. It doesn't know which assembly mnemonics codegen emits, or that codegen tracks a `depth` counter, or that the AST has nine kinds of nodes. It doesn't even know that any of those passes might fail — the error helpers don't return; they `exit(1)`, and the compiler dies before control gets back to `main`.

This is the loose-coupling property the file split was for. Each pass is a black box from the outside. The only contract between passes is the data type at the boundary: a `Token *` linked list flows from tokenizer to parser, a `Node *` tree flows from parser to codegen. Those types are public (declared in `chibicc.h`) and immutable from `main`'s perspective — `main` doesn't construct them and doesn't inspect them, it just passes them along.

## The Makefile

The Makefile gains four lines:

```diff
 CFLAGS=-std=c11 -g -fno-common
+SRCS=$(wildcard *.c)
+OBJS=$(SRCS:.c=.o)
 
-chibicc: main.o
-	$(CC) -o chibicc main.o $(LDFLAGS)
+chibicc: $(OBJS)
+	$(CC) $(CFLAGS) -o $@ $^ $(LDFLAGS)
+
+$(OBJS): chibicc.h
```

Three small things are happening.

`SRCS=$(wildcard *.c)` enumerates every `.c` file in the directory. Right now that's four; tomorrow it might be five. The Makefile won't need to be edited when files get added — `wildcard` re-evaluates each time `make` runs.

`OBJS=$(SRCS:.c=.o)` is the substitution-reference form of "for each `.c`, the matching `.o`." `$(SRCS:.c=.o)` says "take `$(SRCS)` and replace the `.c` suffix with `.o`." The OBJS list is now derived, not hand-maintained.

`$(OBJS): chibicc.h` declares a dependency: every object file depends on `chibicc.h`. If we edit the header, every `.c` file gets recompiled. This is coarse — strictly, only the `.c` files that actually use the changed declaration need to recompile — but for a four-file project the conservatism costs nothing and prevents the classic "I edited the header but `make` only rebuilt one file" stale-binary bug.

The recipe `$(CC) $(CFLAGS) -o $@ $^ $(LDFLAGS)` uses the automatic variables `$@` (the target, here `chibicc`) and `$^` (the prerequisites, here all the `.o` files). The old recipe hard-coded `main.o` because there was only one object; the new one is generic. Like the `wildcard`, this will not need to change as the project grows.

Make's *implicit* rules handle the `.c → .o` step. There's no rule in the Makefile that says how to compile `tokenize.c` into `tokenize.o`. Make has a built-in rule for that, parameterized by `$(CC)` and `$(CFLAGS)`, which is why setting those two variables is enough.

## Why now

This is the eighth commit in chibicc's history and the *first* refactor — every commit before it added something. So why does the refactor land here, before any of the heavy features?

Three reasons line up.

**The architecture is finished.** Chapter 1 ended with three labeled passes inside one file. The labels are the file boundaries we're drawing now; we couldn't have drawn them this cleanly before commit 5, because the parser and codegen were entangled. Now that the labels are accurate, promoting them to files is mechanical.

**The interfaces between phases are simple.** Tokenize hands a token list to parse. Parse hands an AST to codegen. There are no shared mutable structures, no callbacks, no cross-phase queries. When the boundaries are this thin, splitting is cheap; when boundaries are thick (lots of shared state, lots of crosstalk), splitting is expensive. Catching it before the boundaries thicken matters.

**The next several features expand all three phases.** Local variables (Chapter 3) need a new token role for identifiers, a new AST node kind, and a new codegen pattern for stack slots. Function calls (Chapter 5) need new token kinds, new AST nodes, and a new ABI dance in codegen. Doing this in one growing `main.c` would mean each commit's diff spans the whole file; doing it across three files means each diff is local to one or two of them. The reader's eye doesn't have to track changes across hundreds of lines per commit — it tracks changes within a single ~200-line file, and Rui's "every commit is a teachable unit" property is preserved.

The split is preventative maintenance done at the cheapest possible moment.

## Where we are

Eight commits in, and chibicc looks like a real compiler from the outside: `main.c` is a pipeline, three modules each do one job, the Makefile auto-discovers sources, and the public API of each phase fits on one line of `chibicc.h`. The language hasn't grown — `make test` still runs the same 28 cases from Chapter 1. But the codebase is now ready to grow.

Chapter 3 picks up the next ten commits. They add semicolons, statements, single-letter local variables, multi-character identifiers, `return`, blocks, the null statement, `if`, `for`, and `while`. After those ten commits the file we're editing the most will be `parse.c`; `tokenize.c` will gain a few cases; `codegen.c` will learn how to allocate stack slots and emit a function prologue that actually reserves space. Three files, three pieces of work per commit. That cadence starts here.
