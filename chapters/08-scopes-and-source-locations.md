# Chapter 8 — Scopes and source locations

> Commits covered: `ca8b243`, `cd832a3`, `6647ad9`, `1c91d19`, `e6307ad`. Five commits — block scope, a wholesale rewrite of the test suite from shell into C, a precomputed per-token line-number table, the `.file` and `.loc` assembler directives that make compiled programs debuggable in GDB, and the comma operator.

Chapter 7 closed with a mature compiler driver, a tokenizer that handled comments and strings, and an `Obj` table that distinguished locals from globals but treated every "local" as function-scoped. Even inside a `({...})` statement-expression body — which Chapter 7 §7.5 introduced — a variable named `x` lived for the entire enclosing function, with no opportunity to be shadowed or to disappear at a closing brace. That gap is what Chapter 8 begins by closing.

The five commits split fairly cleanly into three concerns. The first (block scope) is a language-feature commit that replaces the flat local-variable table with a stack of scopes. The next pair (precomputed line numbers, `.file`/`.loc`) is debug-info infrastructure: the per-token line numbers introduced in commit 46 are the data that the `.loc` directives in commit 47 emit into the assembly output. The middle commit (tests in C) is a one-shot infrastructure change that retires the shell-script test harness in favor of C-language test files compiled by chibicc itself. The last commit (the comma operator) is a small, late-arriving expression-level addition.

The five sections:

- **§8.1** — Block scope (commit 44).
- **§8.2** — Tests in C (commit 45).
- **§8.3** — Precomputed line numbers (commit 46).
- **§8.4** — `.file` and `.loc` (commit 47).
- **§8.5** — The comma operator (commit 48).

There's no concept interlude. The chapter mapping doesn't call for one, and none of the five commits develops enough conceptual weight on its own to need a stand-alone digression. The closest candidate would be a treatment of DWARF line-number information at §8.4, but chibicc's use of `.loc` is shallow enough that an in-prose paragraph covers what a reader needs.

As with several earlier chapters, the chronological dates of these commits do not match their position on `main`. Two of them (`6647ad9` and `1c91d19`) are dated April 2020; the comma operator (`e6307ad`) is dated August 2019; and the first two (`ca8b243`, `cd832a3`) are dated September 2020. The order here is the canonical-history order from `chapter-mapping.md`, which is what the book follows throughout.

---

## 8.1 — Block scope

> `git checkout ca8b2434c97fc37c14eddcb3a4e831d030ebb041` — *Handle block scope*

Until this commit, chibicc had two name-resolution lists: `locals` (rebuilt per-function) and `globals` (per-program). `find_var` walked `locals` first, then `globals`, and that was the entire scoping mechanism. A program like

```c
int main() {
  int x = 2;
  { int x = 3; }
  return x;
}
```

would have produced 3, not 2 — both `x`'s would be in the same flat `locals` list, the second `int x = 3` would have created a new `Obj` named `x`, and the lookup at `return x` would have found whichever one was at the head of the list. C says the answer is 2: the inner `x` lives only inside the inner braces, and after the closing `}` it's gone, so `return x` refers to the outer one.

This commit gives chibicc a real notion of block scope. Two new struct types, two new functions, one new module-level variable, and three call sites get the mechanism wired up.

### A stack of scopes

```c
// Scope for local or global variables.
typedef struct VarScope VarScope;
struct VarScope {
  VarScope *next;
  char *name;
  Obj *var;
};

// Represents a block scope.
typedef struct Scope Scope;
struct Scope {
  Scope *next;
  VarScope *vars;
};
```

Two linked lists, nested. A `Scope` is one block. Each `Scope` holds a `VarScope` list — the names introduced inside that block and the `Obj` each name binds to. Scopes are themselves linked through their own `next` field, oldest-first being deepest in the chain — actually the other way around. The module-level cursor

```c
static Scope *scope = &(Scope){};
```

points at the *innermost* scope, and `scope->next` is the enclosing one. The initializer `&(Scope){}` is a compound literal — a C99 construct that produces an unnamed `Scope` initialized to all zeros and yields its address. Because it's used as a static initializer, the compound literal has static storage duration, which is exactly what's wanted: an empty outermost scope that exists for the whole program lifetime.

That outermost scope is where global declarations end up. Recall that in Chapter 7's commit `a4d3223`, `new_gvar` registered globals into a `globals` list. After this commit, every variable creation also registers into the current `Scope`, and at program-parse time the current scope is the outermost one — so globals naturally accumulate there. The explicit `globals` list is still there, because codegen needs it (`emit_data` walks it to lay down the `.data` section), but the *lookup* path no longer treats globals as a separate fallback.

### `enter_scope` and `leave_scope`

```c
static void enter_scope(void) {
  Scope *sc = calloc(1, sizeof(Scope));
  sc->next = scope;
  scope = sc;
}

static void leave_scope(void) {
  scope = scope->next;
}
```

Push and pop. `enter_scope` allocates a fresh `Scope`, links it to the current top, and slides the cursor onto the new one. `leave_scope` just unlinks. Note that the popped `Scope` and its `VarScope` chain are not freed — chibicc never frees anything during compilation, by deliberate choice. The Trusting-Trust quote from Chapter 7 §7.4 isn't the only place chibicc trades simplicity for behavior a production compiler would be careful about; the parser leaks every allocation, on the theory that the OS will reclaim it when the process exits and that compilation is short-lived enough not to matter. For a program of any reasonable size, it doesn't.

### `find_var` walks the scope chain

```c
// Find a variable by name.
static Obj *find_var(Token *tok) {
  for (Scope *sc = scope; sc; sc = sc->next)
    for (VarScope *sc2 = sc->vars; sc2; sc2 = sc2->next)
      if (equal(tok, sc2->name))
        return sc2->var;
  return NULL;
}
```

Two nested loops. The outer one walks scopes from innermost out. The inner one walks the `VarScope` list of each scope. The first match wins. Because each scope's `VarScope` list is populated by prepending (`sc->next = scope->vars; scope->vars = sc;`), the most recently declared name in a scope is found first — but in C this can't matter, because two declarations of the same name in the same scope would be a redeclaration error. (Chibicc doesn't actually check for that yet; declaring `int x; int x;` inside one block just produces two `x` entries, and the lookup finds the second. This is a wart for the errata appendix.)

The two-loop structure also handles shadowing for free. Searching innermost first means an inner declaration wins over an outer one; only if no inner scope has the name do we keep walking out.

The previous `find_var` had a separate global-fallback loop:

```diff
-// Find a local variable by name.
-static Obj *find_var(Token *tok) {
-  for (Obj *var = locals; var; var = var->next)
-    if (strlen(var->name) == tok->len && !strncmp(tok->loc, var->name, tok->len))
-      return var;
-
-  for (Obj *var = globals; var; var = var->next)
-    if (strlen(var->name) == tok->len && !strncmp(tok->loc, var->name, tok->len))
-      return var;
-
-  return NULL;
-}
+// Find a variable by name.
+static Obj *find_var(Token *tok) {
+  for (Scope *sc = scope; sc; sc = sc->next)
+    for (VarScope *sc2 = sc->vars; sc2; sc2 = sc2->next)
+      if (equal(tok, sc2->name))
+        return sc2->var;
+  return NULL;
+}
```

The locals/globals split is gone from lookup. The `locals` and `globals` lists themselves still exist — codegen's `assign_lvar_offsets` walks the function's `locals` list to assign stack offsets, and `emit_data` walks `globals` to emit the `.data` section — but they're no longer the data structure consulted at name-resolution time. The new mechanism subsumes both: a global lives in the outermost scope, a function parameter lives one level in, a function body's locals live one or more levels deeper.

The diff also picks up a small idiomatic improvement. The old comparison was `strlen(var->name) == tok->len && !strncmp(tok->loc, var->name, tok->len)` — manual byte-length-then-bytes. The new one is `equal(tok, sc2->name)`, the helper from Chapter 1 that already does the same comparison. This is a nice cleanup that the rewrite enables in passing.

### `push_scope` and `new_var`

Every variable creation now registers into the current scope:

```c
static VarScope *push_scope(char *name, Obj *var) {
  VarScope *sc = calloc(1, sizeof(VarScope));
  sc->name = name;
  sc->var = var;
  sc->next = scope->vars;
  scope->vars = sc;
  return sc;
}

static Obj *new_var(char *name, Type *ty) {
  Obj *var = calloc(1, sizeof(Obj));
  var->name = name;
  var->ty = ty;
  push_scope(name, var);
  return var;
}
```

`new_var` is the shared base used by `new_lvar` and `new_gvar`. Centralizing the `push_scope` call here means *every* variable creation, local or global, gets a `VarScope` entry in whichever `Scope` is current. This is what lets globals end up in the outermost scope without any special case at the call site — `new_gvar` just calls `new_var`, which calls `push_scope`, and at the time `new_gvar` runs there's only one scope.

`new_lvar` and `new_gvar` retain their existing list-prepending behavior on top:

```c
static Obj *new_lvar(char *name, Type *ty) {
  Obj *var = new_var(name, ty);
  var->is_local = true;
  var->next = locals;
  locals = var;
  return var;
}

static Obj *new_gvar(char *name, Type *ty) {
  Obj *var = new_var(name, ty);
  var->next = globals;
  globals = var;
  return var;
}
```

Two layers of bookkeeping per variable: the `VarScope` entry (for lookup) and the `Obj` chain (for codegen). Not redundant — they serve different consumers.

### Where the calls happen

`compound_stmt` brackets its body with enter/leave:

```diff
 static Node *compound_stmt(Token **rest, Token *tok) {
   Node *node = new_node(ND_BLOCK, tok);
-
   Node head = {};
   Node *cur = &head;
+
+  enter_scope();
+
   while (!equal(tok, "}")) {
     if (is_typename(tok))
       cur = cur->next = declaration(&tok, tok);
@@
     add_type(cur);
   }
+
+  leave_scope();
+
   node->body = head.next;
   *rest = tok->next;
   return node;
 }
```

Every `{...}` block — whether it's a function body, a `for`-loop body, an `if`-then branch with braces, or a free-standing block of the form `{ int x = 2; }` — goes through `compound_stmt`, which now opens a new scope on entry and closes it on exit. The Chapter 1 grammar has `compound_stmt` as the body of every braced statement form, so this single place suffices for all braced scopes.

The `function` parser also enters a scope, *before* calling `create_param_lvars`:

```diff
 static Token *function(Token *tok, Type *basety) {
   ...
   fn->is_function = true;

   locals = NULL;
+  enter_scope();
   create_param_lvars(ty->params);
   fn->params = locals;

   tok = skip(tok, "{");
   fn->body = compound_stmt(&tok, tok);
   fn->locals = locals;
+  leave_scope();
   return tok;
 }
```

This is subtle. `function` opens a scope, registers parameters into it, then calls `compound_stmt` to parse the body — and `compound_stmt` opens *another* scope. The result is two nested levels for every function: an outer one holding parameters, and an inner one holding the body's first level of locals.

Why two? Because in C, parameters and body locals are separate scopes. The standard says a parameter and a top-level body local with the same name *would* be a redeclaration in the same scope and therefore an error, but chibicc isn't checking that yet. What the two-scope structure does buy is the right behavior for a body local that *shadows* a parameter:

```c
int f(int x) {
  int x;  // would be an error in real C, but if allowed, shadows the param
  return x;
}
```

With two scopes, the body's `x` lookup finds the inner `x` first; with one scope, the parser would prepend the inner `x` to the same `VarScope` list and lookup would still find it first by accident. Functionally similar, semantically distinct. The two-scope structure is the right shape for what C actually says.

### Back-reference: statement expressions

Chapter 7 §7.5 introduced statement expressions, the GNU-extension `({...})` form. The §7.5 prose noted that any locals declared inside a `({...})` body lived for the whole enclosing function, not just for the body — because every "local" was function-scoped at that point. This commit changes that. A statement expression's body is a `compound_stmt`, which now opens its own scope, so

```c
int main() {
  int x = 1;
  int y = ({ int x = 2; x; });
  return x + y;
}
```

returns 3, not 4. The inner `x = 2` lives only inside the `({...})` body and disappears at the closing brace; the outer `x` is unshadowed for the `return x + y` line that follows. Chapter 7 flagged this as a behavior that would change here, and it has.

### The tests

Three new tests at the bottom of `test.sh`:

```sh
assert 2 'int main() { int x=2; { int x=3; } return x; }'
assert 2 'int main() { int x=2; { int x=3; } { int y=4; return x; }}'
assert 3 'int main() { int x=2; { x=3; } return x; }'
```

The first is the canonical shadowing case: an inner `int x = 3` doesn't disturb the outer `x`. The second mixes a shadowing block and a non-shadowing one to make sure the outer `x` remains visible after both inner scopes have closed. The third is the assignment counterpart: `{ x = 3; }` does *not* declare a new `x`, so the assignment writes to the outer `x` — which can only happen if the bare expression statement inside the block resolves `x` to the outer scope. Together they exercise three distinct paths through `find_var`.

### Where we are

Chibicc has block scope. A nested `{...}` introduces a fresh `Scope`, its declarations live in that `Scope`'s `VarScope` list, and at the closing brace the `Scope` is popped — leaving the outer scope's bindings intact. Lookup is innermost-first, which gives shadowing for free. Functions get an outer scope for parameters and an inner one for the body, matching C's semantic split. Globals quietly inherit the same machinery by living in the program's outermost `Scope`.

The implementation is small — under fifty net lines of `parse.c` — and the data structures are linked lists of linked lists, walked linearly. Real compilers use hash tables keyed on symbol names; chibicc will eventually too, in Chapter 22, but not yet. For program sizes that fit in chibicc's existing ambitions (and for chibicc's own source code, which will be the self-host target in Chapter 17), linear scope walking is fast enough that performance never shows up as a problem.

The chapter's other four commits leave scoping alone and turn to debugging, testing, and one more expression-level operator.

---

## 8.2 — Tests in C

> `git checkout cd832a311e56bda981c9c957ba45f1bc1f6cc737` — *Rewrite tests in shell script in C*

The 224-line shell script `test.sh` has done duty as chibicc's test harness since Chapter 1. Each test is a one-line shell invocation:

```sh
assert 8 'int main() { int x=3; int z=5; return x+z; }'
```

The `assert` shell function pipes the C source into chibicc, links the assembly with a small helper object, runs the resulting executable, and compares its exit status against the expected value. This works, but it has friction. Test sources have to be quoted as shell strings, which means embedded `'`s can't appear naturally and multi-line tests need awkward heredocs. Every test is exactly one C function (`main`), so there's no easy way to test multi-function behavior except by writing the whole thing as one long line. And every assertion has to be readable in a shell-quoted form, which limits what can be tested.

This commit retires `test.sh` and replaces it with a directory of `.c` files compiled by chibicc itself, then linked against a small helper, then executed. The C sources can be as long as they need to be, can use any C syntax chibicc understands, and most importantly can use macros to compress what shell quoting was making expensive.

### The directory

```
test/
  arith.c
  control.c
  function.c
  pointer.c
  string.c
  variable.c
  test.h
  common
  driver.sh
```

Six C files for the language tests, one header (`test.h`), one common helper (`common`), and the renamed driver script (`driver.sh`, formerly `test-driver.sh`). The C files are organized by what they test — arithmetic, control flow, functions, pointers, strings, variables — which is a natural grouping by feature. Each one is a stand-alone program with its own `main` that returns 0 on success.

### `test.h` and `ASSERT`

```c
#define ASSERT(x, y) assert(x, y, #y)
```

One line. The macro takes an expected value `x` and an expression `y`, calls a C function `assert` with both plus the *stringification* of `y`. The `#y` operator is the C preprocessor's stringify operator: it converts the macro argument's source text into a string literal. So

```c
ASSERT(21, 5+20-4);
```

expands to

```c
assert(21, 5+20-4, "5+20-4");
```

The `assert` function in `test/common` then prints the source text alongside the value for readable failure messages:

```c
void assert(int expected, int actual, char *code) {
  if (expected == actual) {
    printf("%s => %d\n", code, actual);
  } else {
    printf("%s => %d expected but got %d\n", code, expected, actual);
    exit(1);
  }
}
```

The pattern is the C-equivalent of what the shell `assert` function did with `$input`, just with the source text captured at preprocessor time rather than passed as a string parameter.

There's a notable absence in chibicc here: it has no preprocessor of its own. The `#define ASSERT` and `#include "test.h"` lines in the test files can't be processed by chibicc, because chibicc's tokenizer doesn't yet understand `#`. The Makefile gets around this by running the tests through the host `cc` as a preprocessor before handing them to chibicc:

```makefile
test/%.exe: chibicc test/%.c
	$(CC) -o- -E -P -C test/$*.c | ./chibicc -o test/$*.s -
	$(CC) -o $@ test/$*.s -xc test/common
```

`$(CC) -o- -E -P -C` runs the host compiler in preprocess-only mode (`-E`), without `#line` markers (`-P`), preserving comments (`-C`), writing to stdout (`-o-`). The result is a fully preprocessed C file with no `#`-directives, which chibicc can then tokenize. The chibicc output is assembled and linked against `test/common` (told `-xc` to make sure it's treated as C source rather than guessed from extension).

This arrangement neatly separates concerns: preprocessing is *not* something chibicc has to handle yet, but tests still get to use macros for ergonomics. When chibicc grows its own preprocessor in Chapter 17, this pipeline will collapse — chibicc will run on the unprocessed source directly, and the host-`cc`-as-preprocessor trick will become unnecessary.

### Statement expressions everywhere

The shell-form tests had to fit into one C function each:

```sh
assert 3 'int main() { int x=3; return *&x; }'
```

The C-form tests don't have that constraint, but they do have a different one: each `.c` file is one program with one `main`, and there are dozens of assertions per file. The natural way to write each assertion as a self-contained expression is with a statement expression:

```c
ASSERT(3, ({ int x=3; *&x; }));
```

The `({...})` body declares `x`, dereferences `&x`, and yields the result; `ASSERT` compares it against 3. Crucially, the `int x` declared inside the `({...})` is now scoped to the statement-expression body — that's the §8.1 mechanism cashing in immediately. Without block scope, every `({ int x=...; })` in a test file would have polluted `main`'s local scope, with the second one redefining `x` and the third one redefining it again. With block scope, each `({...})` is a clean little world, and the test files can use the same throwaway local names dozens of times without interference.

This is the chapter's moment of "two commits ago we built a thing, and now we use it." `test.sh` got away with shadow-free tests because every test was in its own `assert` invocation with its own freshly forked chibicc process. The C-form tests put dozens of assertions in one function and need real scope to work.

### Three representative tests

A trivial case from `arith.c`:

```c
ASSERT(21, 5+20-4);
```

Identical to the shell form except for the macro shape. No statement expression needed; it's a single expression.

A statement-expression case from `pointer.c`:

```c
ASSERT(3, ({ int x=3; *&x; }));
```

Equivalent to the shell-form `int main() { int x=3; return *&x; }`. The body sets `x`, dereferences `&x`, and the final expression is the value of the statement expression. Block scope makes the local `x` invisible outside.

A multi-statement case from `variable.c`:

```c
ASSERT(2, ({ int x=2; { int x=3; } x; }));
```

This one *tests* block scope itself, in the same form chibicc's `test.sh` introduced for it three commits ago. Two `x`'s, one in the outer statement-expression body and one in a nested block. The expected value of the whole statement-expression is 2: the inner `x` shadows but doesn't replace.

### The Makefile

The new Makefile target shape:

```makefile
TEST_SRCS=$(wildcard test/*.c)
TESTS=$(TEST_SRCS:.c=.exe)

test/%.exe: chibicc test/%.c
	$(CC) -o- -E -P -C test/$*.c | ./chibicc -o test/$*.s -
	$(CC) -o $@ test/$*.s -xc test/common

test: $(TESTS)
	for i in $^; do echo $$i; ./$$i || exit 1; echo; done
	test/driver.sh
```

A wildcard pattern picks up every `.c` file in `test/`, derives a corresponding `.exe` target, and the per-file rule preprocesses, compiles with chibicc, and links. The top-level `test` target runs every `.exe` in turn, exiting on the first failure, then runs the renamed driver script for the option-parsing tests.

Adding a new test file is now a matter of dropping it into `test/`. The Makefile picks it up automatically. `test.sh` required editing the central script, which was tolerable but not as natural.

The driver script (`driver.sh`, formerly `test-driver.sh` from Chapter 7 §7.6) moves into `test/` and gets a one-line update to point at its new path. Its contents — testing `-o`, `--help`, and stdin input — are unchanged.

### Where we are

Chibicc tests itself in C. The host `cc` handles preprocessing; chibicc compiles the preprocessed result; the host `cc` again links the assembly with a small C helper; the resulting executable runs and exits 0 on pass, 1 on fail. The `Makefile` discovers test files by glob, runs them in order, and falls through to a shell-script driver for the option tests.

Beyond convenience, the rewrite is a low-key milestone: it's the first time chibicc is meaningfully *used* on programs of more than a few lines. Each test file is a hundred or so lines of real C, with multiple functions, statement expressions, globals, strings, and arrays. Bugs in chibicc that would have been hard to construct in a one-line shell test surface naturally here, and the effective surface area of the test suite grows substantially even though the line count of the test corpus is roughly the same.

---

## 8.3 — Precomputed line numbers

> `git checkout 6647ad9b843768968db0a331ff7077904c6f58ee` — *Precompute line number for each token*

Chapter 7 §7.6 introduced `verror_at`, the source-aware error reporter that prints the offending line with a caret pointing at the column. To find the line number for a given offset into the source buffer, `verror_at` walked from `current_input` to the error location counting newlines:

```c
int line_no = 1;
for (char *p = current_input; p < line; p++)
  if (*p == '\n')
    line_no++;
```

That's O(n) per error. For a single-error-and-exit pattern, it doesn't matter — the cost is paid once. But the next commit (§8.4) will emit a `.loc` directive at every statement and expression, which means asking for a token's line number hundreds or thousands of times per compilation. The walk-from-the-start approach scales badly for that, and more importantly, the right place to put the answer is on the `Token` itself.

This commit precomputes a line number for every token and caches it on the token struct.

### A new field

```diff
 struct Token {
   ...
   Type *ty;       // Used if TK_STR
   char *str;      // String literal contents including terminating '\0'
+
+  int line_no;    // Line number
 };
```

`int line_no`, set once at tokenize time, read everywhere thereafter.

### A single-pass annotator

```c
// Initialize line info for all tokens.
static void add_line_numbers(Token *tok) {
  char *p = current_input;
  int n = 1;

  do {
    if (p == tok->loc) {
      tok->line_no = n;
      tok = tok->next;
    }
    if (*p == '\n')
      n++;
  } while (*p++);
}
```

One pass over the input buffer, advancing both the source pointer `p` and the token pointer `tok`. Each iteration: if the source pointer matches the current token's location, stamp the line number onto the token and advance to the next token; if the source byte is a newline, bump the line counter; then advance the source pointer.

This works because the token list is in source order — each subsequent `tok->loc` is at a position later in the buffer than the previous one. So a single forward scan can stamp them all in a single pass over the source.

The function is called from `tokenize`, after the EOF token is added but before keyword conversion:

```diff
   cur = cur->next = new_token(TK_EOF, p, p);
+  add_line_numbers(head.next);
   convert_keywords(head.next);
```

The EOF token has its `loc` set to the input's terminating null, so `add_line_numbers` annotates it too. All in one O(n) sweep, where n is the input size — and once it's done, every token's line number is a constant-time field access.

### `verror_at` becomes scope-narrower

The line-counting loop comes out of `verror_at` and moves to its callers:

```diff
-static void verror_at(char *loc, char *fmt, va_list ap) {
+static void verror_at(int line_no, char *loc, char *fmt, va_list ap) {
   ...
-  // Get a line number.
-  int line_no = 1;
-  for (char *p = current_input; p < line; p++)
-    if (*p == '\n')
-      line_no++;
-
   // Print out the line.
   int indent = fprintf(stderr, "%s:%d: ", current_filename, line_no);
   ...
 }
```

The function now takes the line number as a parameter rather than computing it. Each caller supplies it from whichever source it has. `error_tok` reads it directly from the token:

```c
void error_tok(Token *tok, char *fmt, ...) {
  va_list ap;
  va_start(ap, fmt);
  verror_at(tok->line_no, tok->loc, fmt, ap);
}
```

`error_at`, which takes a raw `char *loc` and not a token, has to compute the line number itself — there's no token to read it from. So the old loop survives in one place:

```c
void error_at(char *loc, char *fmt, ...) {
  int line_no = 1;
  for (char *p = current_input; p < loc; p++)
    if (*p == '\n')
      line_no++;

  va_list ap;
  va_start(ap, fmt);
  verror_at(line_no, loc, fmt, ap);
}
```

This is fine — `error_at` is called only from the tokenizer (for things like "invalid token" or "unclosed block comment"), where there isn't a `Token` yet. The cost is paid once per tokenizer error, which is at most once per compilation.

### Why now

The commit message says "No functionality change." That's true at the user-visible level — error messages still print the same text and the same line numbers. The motivation isn't visible until the next commit. `.loc` directives in §8.4 are emitted from `gen_expr` and `gen_stmt`, both of which take `Node *` — and every `Node` carries its `Token *tok` (set at AST-construction time so `error_tok` can point at the source on type errors and the like). With per-token line numbers cached, emitting `.loc` becomes one printf with a struct field access; without them, every `.loc` would re-walk the source buffer.

The pre-factor-before-feature pattern from Chapter 6 §6.5 and Chapter 7 §7.6 fits here too. This is a structural change that doesn't add behavior on its own; the next commit is the feature that depends on it, and is small because of it. The commit message's "No functionality change" is the same flag chibicc has used twice before for a pre-factor.

### Where we are

Every `Token` carries its line number as a cached field. Error reporting is unchanged in behavior but no longer pays a per-error scan. The data needed by the next commit is in place.

---

## 8.4 — `.file` and `.loc`

> `git checkout 1c91d1943a8ee07034224dd950412c3c87ef3276` — *Emit `.file` and `.loc` assembler directives*

This is the smallest commit in the chapter — five lines of code added across two files — and the largest user-visible change. After this commit, programs compiled by chibicc are debuggable in GDB at the source level. A breakpoint on a line of C source halts at the right instruction. Stepping advances by source lines, not by individual machine instructions. A backtrace from a crash names the function and the source line where the crash happened.

None of this requires chibicc to emit DWARF debug info itself. The GNU assembler does it, given two directives that chibicc now produces.

### The directives

```diff
 // Generate code for a given node.
 static void gen_expr(Node *node) {
+  println("  .loc 1 %d", node->tok->line_no);
+
   switch (node->kind) {
   ...
 }

 static void gen_stmt(Node *node) {
+  println("  .loc 1 %d", node->tok->line_no);
+
   switch (node->kind) {
   ...
 }
```

Two lines. Every `gen_expr` and `gen_stmt` call now emits a `.loc 1 <line>` at the start of its output. The `1` is a file ID; the `<line>` is the cached `line_no` from the token that started this AST node.

And in `main.c`, one more line:

```diff
 FILE *out = open_file(opt_o);
+fprintf(out, ".file 1 \"%s\"\n", input_path);
 codegen(prog, out);
```

A `.file 1 "<path>"` at the top of every output, declaring that file ID 1 corresponds to the input source file.

That's the whole change: one `.file` at the top of the output, two `.loc` directives at the start of every code-generating function.

### What the assembler does

`.file` and `.loc` are pseudo-directives understood by the GNU assembler. The assembler doesn't put them into the executable as-is; it consumes them and emits a DWARF `.debug_line` section in the resulting object file. DWARF (Debugging With Attributed Record Formats) is the standard debug-info format on Unix-family systems. The `.debug_line` section is a state machine that maps program-counter ranges to source file and line tuples — given a runtime instruction address, the debugger can look up which source line generated it.

Chibicc doesn't have to know any of this. The assembler does the encoding. The link-time and runtime parts — embedding the DWARF section in the executable, exposing it to the debugger — happen automatically. Chibicc's contribution is the directive stream.

The `1` in `.file 1` and `.loc 1` is the file index. DWARF supports many input files per compilation unit (for things like `#include`d headers contributing line entries), so each `.loc` references the file by numeric ID. Chibicc compiles single-file translation units, so file ID 1 is always the input file and that's the only file ID it ever uses. When chibicc grows multi-file support — which it doesn't, in any rich way, even by Chapter 22 — it could in principle emit multiple `.file` declarations. For now, one is enough.

### Per-statement granularity is enough

The `.loc` directive at the top of `gen_expr` is emitted *every time* `gen_expr` is called recursively. For an expression like `a + b * c`, `gen_expr` is called for the whole expression, then for `b * c`, then for `b` and `c`. Each call emits a `.loc`. They're all on the same line (the source line of the whole expression), so the redundant directives don't change the resulting line table — the assembler deduplicates entries that share file/line/column/etc.

Chibicc could be smarter about this — emit `.loc` only at statement boundaries — but the brute-force approach is simpler and produces the same observable result. The cost is a few extra bytes of `.loc` directives in the assembly text, which the assembler discards. The benefit is the tiny code change visible in the diff: just two lines.

### A worked example

Compiling

```c
int main() {
  int x = 1;
  int y = 2;
  return x + y;
}
```

with the `-o -` form (write to stdout) shows the directives in the generated assembly:

```
.file 1 "input.c"
  .text
  .globl main
main:
  push %rbp
  mov %rsp, %rbp
  sub $16, %rsp
  .loc 1 2
  ... initialize x ...
  .loc 1 3
  ... initialize y ...
  .loc 1 4
  ... compute return ...
```

(Actual output has more directives because every nested `gen_expr` call emits one, but the line numbers are the noteworthy bit.) Each statement's machine instructions are preceded by a `.loc` pointing at the statement's source line.

When `gcc` (or `cc`) assembles and links this, the resulting executable carries DWARF line-number information. In GDB:

```
(gdb) break main
(gdb) run
(gdb) list
   1 int main() {
   2   int x = 1;
   3   int y = 2;
   4   return x + y;
   5 }
(gdb) step
... stops at line 2 ...
(gdb) step
... stops at line 3 ...
```

This is what the commit message promises: "with these directives, gdb can print out an error location when a compiled program crashes." It can also do everything else GDB can do at source level — set breakpoints by line, step by line, print backtraces with file/line info.

### Trade-offs left on the table

There's no `.debug_info` section. DWARF's `.debug_line` (line-number table) is one piece of debug info; `.debug_info` is the much larger one that encodes types, variable names, function signatures, and structure layout. Without `.debug_info`, GDB can navigate the source by line but can't print the value of a local variable by name or display a stack frame's arguments. Chibicc's `print x` in GDB will fail.

This is a deliberate stopping point. Full `.debug_info` emission would require chibicc to model and emit type descriptions, variable scopes, location expressions for where each variable lives at each point in the function, and a variety of other DWARF tags. That's many hundreds of lines of code. The line-number table alone is the cheapest 80%-payoff and Rui takes it. Stepping through chibicc-compiled code in GDB is enough of a debugging experience that the missing variable-printing is tolerable; it's not enough to make a production compiler, but it's enough for chibicc.

A real C compiler can be invoked with `-g0` (no debug info), `-g` (line numbers + variable info), or `-g3` (everything including macros). Chibicc has no flag to control any of this — `.file` and `.loc` are always emitted. The cost is small (debug info is a separate section that the loader doesn't read at runtime) and the benefit is universal, so there's no need for a flag.

### Where we are

Chibicc-compiled programs are debuggable. A user can step through their source, set breakpoints by line, and get meaningful backtraces. The implementation is three lines of code resting on top of the precomputed `line_no` field from the previous commit. The pair (commits 46 and 47) is one of chibicc's clearest examples of how a small enabling commit can make the feature commit trivial.

---

## 8.5 — The comma operator

> `git checkout e6307ad374eeecd6474286b1b6fda5b3dda89d9a` — *Add comma operator*

The C comma operator evaluates its left operand for side effects, discards the result, and yields its right operand:

```c
int a = (foo(), bar());
```

`foo()` runs, its return value is thrown away, `bar()` runs, and its return value is assigned to `a`. As a binary operator, comma sits at the very bottom of C's precedence table — lower than assignment — so

```c
a = 1, b = 2;
```

means "assign 1 to `a` and 2 to `b`," not "assign the comma expression `(1, b = 2)` to `a`."

This commit adds it to chibicc. The implementation is small: a new node kind, a one-line grammar production, three lines of codegen, and a one-line type rule.

### Grammar

```diff
-// expr = assign
+// expr = assign ("," expr)?
 static Node *expr(Token **rest, Token *tok) {
-  return assign(rest, tok);
+  Node *node = assign(&tok, tok);
+
+  if (equal(tok, ","))
+    return new_binary(ND_COMMA, node, expr(rest, tok->next), tok);
+
+  *rest = tok;
+  return node;
 }
```

Two-line grammar production. `expr` parses an `assign`; if a `,` follows, it recurses into another `expr` and wraps both sides in an `ND_COMMA` node. The right-recursive form means `1, 2, 3` parses as `1, (2, 3)` — but because comma is left-associative *in evaluation* and the codegen evaluates lhs-then-rhs, the visible behavior is identical to left-associative parsing. Chibicc could equally have written this as a `while`-loop that builds a left-leaning tree; the recursive form is shorter.

The grammar's placement matters. `expr` sits at the top of the expression hierarchy — it's what parses the expression in `for (init; cond; inc)`, what `expr_stmt` consumes, what's inside a `return`, and what `primary` consumes inside `( ... )`. Putting comma at `expr` and not at `assign` is correct: it's lower-precedence than assignment, so `a = 1, b` means `(a = 1), b` (assign 1 to a, then yield b), not `a = (1, b)`.

This is the first time `expr` itself has been more than a thin wrapper around `assign`. Until now, `expr` was a one-line forwarder. With the comma production, the expression hierarchy gains a top level distinct from assignment, matching C's full precedence table:

```
expr     = assign ("," expr)?
assign   = equality ("=" assign)?
equality = relational ("==" relational | "!=" relational)*
... and so on down through unary and primary
```

There's a subtle consequence. Some C constructs use comma for *something other* than the comma operator — function arguments are comma-separated, declarator lists are comma-separated, initializer lists (when those arrive) are comma-separated. The grammar for those constructs has to call `assign` rather than `expr` to avoid eating the separator commas as comma operators. Chibicc already does this — `funcall` (in `parse.c`) has called `assign` for each argument since Chapter 5, not `expr`. The discipline pays off here: nothing has to change to keep comma operators out of argument lists. They'd already been excluded by virtue of the grammar level call.

### A new node kind

```diff
   ND_ASSIGN,    // =
+  ND_COMMA,     // ,
```

`ND_COMMA` joins the binary-operator family. Its `lhs` is the left operand, its `rhs` is the right.

### Codegen

Two cases get a new branch. First, in `gen_expr`:

```c
case ND_COMMA:
  gen_expr(node->lhs);
  gen_expr(node->rhs);
  return;
```

Evaluate the lhs (its result is left in `%rax` and then immediately overwritten), then evaluate the rhs (its result remains in `%rax`, becoming the value of the comma expression). The lhs's value is discarded by the simple expedient of doing nothing with it before computing the rhs. This is the trick of single-register codegen — `%rax` is the value-of-the-last-thing register, and the comma operator's "yields the rhs" semantics drop out for free.

The second case is in `gen_addr`:

```c
case ND_COMMA:
  gen_expr(node->lhs);
  gen_addr(node->rhs);
  return;
```

This is what the commit message means by "generalized lvalue." In standard C, the comma expression `(a, b)` is *not* an lvalue, so `(a, b) = c` is a syntax error. GCC has long extended this — the so-called *generalized lvalue* extension lets certain non-standard expressions act as assignment targets, including parenthesized comma expressions. Rui's commit message:

> This patch allows writing a comma expression on the left-hand side of an assignment expression. This is called the "generalized lvalue" which is a deprecated GCC language extension. I'm implementing it anyway because it's useful to implement other features.

The "other features" Rui has in mind aren't spelled out, but the most likely beneficiary is compound assignment. `a += b` (which arrives in Chapter 11) is conventionally rewritten as `a = a + b`, but that double-evaluates `a` — bad if `a` has side effects, like `*p++` or `arr[i++]`. The trick to evaluate `a` once is to introduce a temporary pointer: `(tmp = &a, *tmp = *tmp + b)`. The temporary's address is computed once, and the comma expression yields the address as the lvalue of the assignment. That lowering only works if the comma can be on the left of `=`.

So the `gen_addr` case for `ND_COMMA` is forward-looking: chibicc isn't using it yet (the test `(i=5,j)=6` is the only direct exercise), but it makes the `+=` lowering in Chapter 11 expressible. The codegen is symmetrical with `gen_expr`'s comma case: evaluate the lhs for side effects, then take the address of the rhs.

The branch lives in `gen_addr`, which is the "compute the address of an lvalue into `%rax`" side of codegen. `ND_VAR` and `ND_DEREF` were already there; `ND_COMMA` joins them as a third lvalue-shaped kind.

### Type

```diff
+  case ND_COMMA:
+    node->ty = node->rhs->ty;
+    return;
```

A comma expression has the type of its rhs. The lhs's type is irrelevant — the value is discarded — and chibicc's `add_type` makes this explicit.

### Tests

Three new tests in `test/control.c`:

```c
ASSERT(3, (1,2,3));
ASSERT(5, ({ int i=2, j=3; (i=5,j)=6; i; }));
ASSERT(6, ({ int i=2, j=3; (i=5,j)=6; j; }));
```

The first is the classical use: `(1,2,3)` evaluates to 3, the rightmost. The second and third exercise the generalized-lvalue extension. `(i=5, j) = 6` does three things: it assigns 5 to `i` (the lhs of the comma), it determines that `j` is the lvalue (the rhs), and it assigns 6 to `j`. So after the statement, `i == 5` and `j == 6`. The two tests check both.

Note that `int i=2, j=3` in those tests uses a different comma: the *declarator-list comma* from Chapter 6, which is a grammar-level separator inside `declaration`, not the comma operator. The two commas live in different grammar productions and never confuse the parser.

### Where we are

The comma operator works, both as an expression and as an lvalue (via the generalized-lvalue extension Rui flagged in the commit message). The expression hierarchy now has its full top: `expr` is comma-separated assignments, `assign` is equality with optional assignment, and so on down the precedence ladder. The implementation is twenty-four lines across four files, and the bulk of the work is in the parser one-liner — the codegen and type rules drop out trivially because chibicc's single-register evaluation model already does what the comma operator's semantics require.

---

## Recap

| Commit | What it added |
|---|---|
| `ca8b243` | Block scope: `Scope`/`VarScope` structs; `enter_scope`/`leave_scope`; module-level `scope` cursor; `push_scope`; scope-aware `find_var`; `compound_stmt` brackets its body with enter/leave; `function` wraps params in their own scope before the body opens an inner one |
| `cd832a3` | Tests rewritten in C: `test/*.c` files, `test/test.h`'s `ASSERT(x,y)` macro using `#y` stringification, `test/common` host-language helper, host `cc -E -P -C` as preprocessor, Makefile glob-based test discovery |
| `6647ad9` | Per-token `line_no` field; `add_line_numbers` single-pass annotator; `verror_at` takes the line number as a parameter; `error_tok` reads it from the token; `error_at` keeps the on-the-fly walk |
| `1c91d19` | `.file 1 "..."` at the top of the assembly output; `.loc 1 <line>` at the start of every `gen_expr` and `gen_stmt` call; per-statement DWARF line-number information; programs become source-level debuggable in GDB |
| `e6307ad` | `ND_COMMA` node kind; `expr = assign ("," expr)?` grammar production; codegen for comma in `gen_expr` (evaluate lhs, evaluate rhs); generalized-lvalue support in `gen_addr` (evaluate lhs, take address of rhs); `add_type` rule (type of rhs); first time `expr` is more than a thin wrapper |

Five commits, two of them substantive (block scope, `.loc` directives), one large but mechanical (tests in C), and two small (precompute, comma). The chapter's center of gravity is split between §8.1 (block scope, the biggest single behavioral change) and the §8.3/§8.4 pair (the smallest combined diff with the largest debugging-experience payoff).

Two threads from earlier chapters tied off here. The pre-factor-before-feature pattern named in Chapter 6 §6.5 and Chapter 7 §7.6 has its third clean instance: commit `6647ad9` is the structural change ("No functionality change"), and commit `1c91d19` is the feature that depends on it. Three instances now make this clearly part of chibicc's commit-style vocabulary, and future "no functionality change" commits should be read as setup for what follows. And the Chapter 7 §7.5 note about statement-expression locals living at function scope is now obsolete — block scope means each `({...})` body is its own little world, and the C-form test files in §8.2 lean on that constantly.

The compiler now has real lexical scope, debuggable output, an in-language test suite, and a complete expression hierarchy. The next chapter will take advantage of all four to introduce the first compound type — structs and unions — which need scoped tag names, statement-expression-using tests, line-numbered error messages for malformed members, and one new corner of the expression grammar (the `.` and `->` operators) on top of what's now in place.
