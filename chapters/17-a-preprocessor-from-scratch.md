# Chapter 17 — A preprocessor from scratch

> Commits covered: `1e1ea39`, `146c7b3`, `d367510`, `ec149f6`, `d138864`, `bf6ff92`, `aa570f3`, `c6e81d2`, `e7a1857`, `97d33ad`, `9ad60e4`, `2651448`, `acce002`, `1f80f58`, `dec3b3f`, `b9ad3e4`, `dd4306c`, `c7d7ce0`, `1313fc6`, `8f6f792`, `8f561ae`, `769b5a0`, `5cb2f89`, `a8d76ad`, `8075582`, `b33fe0e`, `d85fc4f`, `a1dd621`, `a939a7a`, `e7fdc2e`, `5f5a850`, `6f17071`, `dc01f94`, `ba6b4b6`, `82ba010`, `ab4f1e1`, `7746e4e`, `7cbfd11`, `5322ea8`, `12a9e75`. Forty commits — the longest single arc in the book — building chibicc's own C preprocessor from a do-nothing pass through `#include`, `#if`, `#define`, function-like macros with stringizing and pasting, conditional-inclusion, predefined macros, and the standard headers, ending in self-hosting at the fortieth commit.

Through Chapter 16, the C source `chibicc input.c` flows through is *preprocessed C* — the result of running the host `cc -E` over the input file. The Makefile pipes test files through `cc -o- -E -P -C` before handing them to chibicc, and the stage-2 build runs `self.py`, a Python regex script, over chibicc's own source. The reason is the same in both cases: chibicc has no preprocessor of its own. It can't see `#include`. It can't see `#define`. The hash sign at column zero is, to chibicc, an unknown punctuation token, and any source file that starts with `#include <stdio.h>` is — to chibicc, raw — a parse error.

Chapter 17 closes that gap. By the end, chibicc handles the full preprocessor surface that ordinary C programs lean on: file inclusion (with both `"..."` and `<...>` forms, search paths, and recursive includes), conditional compilation (`#if`/`#ifdef`/`#ifndef`/`#elif`/`#else`/`#endif` with token-level constexpr evaluation), macros (object-like and function-like, with empty arguments, parenthesized arguments, the no-rescan rule for both kinds, stringizing `#`, pasting `##`, and variadics through `__VA_ARGS__`), the predefined macros (`__STDC__`, `__FILE__`, `__LINE__`, the `__x86_64__`/`__linux__`/`__amd64__` family), `#error`, line continuations, adjacent-string-literal concatenation, the standard headers (`stdarg.h`, `stdbool.h`, `stddef.h`, `stdalign.h`, `float.h`, `stdnoreturn.h`), and the `va_arg()` macro that finally turns Chapter 14's magic `__va_area__` into ordinary user-side code. The fortieth commit deletes `self.py`, retires the `cc -E` pipeline, and points the stage-2 Makefile target at chibicc's own source. Chibicc compiles itself.

This is a long chapter, and it has to be. Forty commits — in the chapter mapping's grouping, the largest arc by a wide margin. Section-per-commit treatment would inflate the chapter past thirty thousand words. Instead, the forty commits group naturally into six sections, each carrying a topic that the C preprocessor itself splits along.

- **§17.1 — The skeleton** (commits 158–162). Five commits. A do-nothing `preprocess()` pass between `tokenize_file` and `parse`. The null directive (`#`-only line). `#include "..."`. Skipping extra tokens after `#include`. The `-E` flag.
- **§17.2 — Conditionals** (commits 163–166). Four commits. `#if`/`#endif` with a token-level constexpr evaluator separate from the AST-level `eval` quartet. Skipping nested `#if` in a skipped clause. `#else`. `#elif`.
- **§17.3 — Object-like macros, the hideset, and `#ifdef`** (commits 167–171). Five commits. `#define` with an object-like body. `#undef`. Macro expansion in `#if`/`#elif` arguments. The no-rescan rule, implemented per-token through a *hideset*. `#ifdef`/`#ifndef`.
- **§17.4 — Function-like macros, stringizing, and pasting** (commits 172–178). Seven commits. Zero-arity, multi-arity, empty arguments, parenthesized arguments, the no-rescan rule's function-like-macro variant (using hideset *intersection* at the closing paren), stringizing `#`, pasting `##`.
- **§17.5 — Polish: the rest of the preprocessor** (commits 179–196). Eighteen commits — the longest stretch. The test pipeline switches off `cc -E`. `defined()`. Identifier-to-zero in constexpr. Whitespace and newline preservation through expansion. Line continuation. `<...>` includes. `-I`. Default include paths. `#error`. Predefined macros (the `__STDC__`/`__x86_64__` block). `__FILE__`/`__LINE__` (with the `Token->origin` chain). `__VA_ARGS__`. `__func__` and `__FUNCTION__` (which land in *parse.c*, not *preprocess.c*). Adjacent-string concatenation. Wide character literal. The five standard headers. `va_arg()` as a real macro.
- **§17.6 — Self-host** (commit 197). One commit. `self.py` deleted. Stage-2 points at chibicc's own source. Chibicc compiles itself.

The `main` order across these forty commits is even more scattered by calendar date than Chapter 16. The dates jump between March 2020 and September 2020 with no apparent rhythm — Rui clearly worked on the preprocessor in bursts, sometimes laying skeleton then immediately filling it in, sometimes branching off to a different feature for weeks before returning. The chapter follows `main` order without comment and without trying to reorder.

The macro-expansion algorithm chibicc implements is *Prosser's algorithm*, sometimes called the "blue paint" algorithm in pedagogical writing. The C standard's wording for how macros expand was directly drawn from a 1986 technical document by Dave Prosser, and Rui's source code links to a copy of that document in the chibicc wiki. Rui's commit 170 (the no-rescan rule, §17.3) is the first commit to mention it by name, and his `preprocess.c` opens with a comment block summarizing the algorithm. We'll walk that algorithm in §17.3 and §17.4 in detail; for now, the headline is that the algorithm is designed to *terminate*, even on `#define X X`, even on `#define f() f()`, by tracking — for each token — the set of macro names that have already been expanded into that token's neighborhood. The set is called the *hideset*. Once a macro's name is in a token's hideset, that token is invisible to that macro. Rescans walk through it as ordinary text.

The split between *what the preprocessor does* and *what cc1 does* lands cleanly: `preprocess()` is called inside `cc1()` between `tokenize_file()` and `parse()`. There's no separate `cpp` binary, no second fork-exec round-trip beyond Chapter 16's driver-cc1 split. The cc1 process tokenizes its input file, runs `preprocess()` over the resulting token list, then parses the (now-expanded) tokens. The `-E` flag short-circuits the parse step and prints the post-`preprocess` token list as text. This is the same shape modern GCC uses internally — preprocessing is a *phase* of the compiler, not a separate process — although GCC has historically supported the option of running `cpp0` as a separate stage too.

Forty commits, six sections. The chapter's punchline is two-line: in the fortieth commit, `chibicc compiles chibicc`, and `self.py` — the Python regex preprocessor that has stood in for chibicc's missing preprocessor since Chapter 8 — gets deleted. Chibicc is, at that point, a self-hosting compiler. The book's stated destination, since the preface, has been self-hosting; this chapter is where chibicc arrives.

---

## 17.1 — The skeleton

Five commits. The first does literally nothing — `preprocess()` returns its input list unchanged, after running `convert_keywords` once. The next four fill in the smallest things a preprocessor can recognize: a `#`-only line, a quoted-include directive, error recovery for stray tokens after that directive, and the `-E` driver flag that lets a user see what the preprocessor produced. By the end of the section, the preprocessor's *shape* is in place: a directive recognizer that treats a hash-at-column-zero as a token marker, a token-list-rewriting model where directives splice new tokens into the stream, and a do-nothing default that passes other tokens through.

### 17.1.1 — A do-nothing preprocessor

> `git checkout 1e1ea39dadd0035443f1d15c651deaf979341879` — *Add a do-nothing preprocessor*

Nothing is preprocessed in this commit. What changes is the *shape* of the cc1 pipeline. A new file `preprocess.c` is added with a single function, `preprocess()`, that takes a token list and returns one. The body runs `convert_keywords` over the input and returns it unchanged:

```c
#include "chibicc.h"

// Entry point function of the preprocessor.
Token *preprocess(Token *tok) {
  convert_keywords(tok);
  return tok;
}
```

`convert_keywords` was a `static` helper inside `tokenize.c` (it walked the token list and rewrote `TK_IDENT` to `TK_KEYWORD` for the language's keywords). It loses its `static` qualifier and gains a forward declaration in `chibicc.h`:

```diff
+void convert_keywords(Token *tok);
```

The `tokenize` function stops calling it. `preprocess()` calls it instead. The reason is *ordering*: keyword conversion happens after macro expansion, not before, because a macro can expand to text containing a keyword (`#define X int` is supposed to make `X foo;` declare an `int`). With the preprocessor in place, the keyword-conversion step has to run after all macro expansion is done. In this commit the preprocessor doesn't expand anything, but the ordering is set up correctly so the next commits don't have to re-shuffle.

The `cc1()` driver function gets one new line:

```diff
 static void cc1(void) {
   // Tokenize and parse.
   Token *tok = tokenize_file(base_file);
+  tok = preprocess(tok);
   Obj *prog = parse(tok);
```

The pipeline is now `tokenize → preprocess → parse`. Both arrows are fully typed `Token *` — chibicc's preprocessor takes a token list and returns a token list, never reading from raw text and never producing raw text. The preprocessor manipulates tokens that the tokenizer already produced. This is the *single-pass tokenize-then-preprocess* model, and it's a deliberate simplification over what the C standard literally describes (the standard frames preprocessing as operating on *preprocessing tokens*, then converting them to ordinary tokens). Chibicc collapses the two: every token is an ordinary token from the moment it leaves the tokenizer.

There's a quiet downstream consequence. Pre-Chapter-17, an idiom like `0xa` for-an-`a`-suffix-on-a-number didn't matter because chibicc's tokenizer handled it. Post-Chapter-17, when *macros* can synthesize numbers like `2##0xff`, the tokenizer has to handle the same shapes consistently. Rui's commits don't explicitly call this out, but the regression net (`make test`) catches it.

The chapter's *real* preprocessor work doesn't start until the next commit. This one commits the *seam*: a function called `preprocess` that lives in `preprocess.c` and runs at the right point in the pipeline. Once the seam exists, every subsequent commit fills in a feature, and the diff stays local to `preprocess.c` (with rare exceptions when a feature genuinely needs the tokenizer's help — line continuations, the `##` punctuation, wide character literals).

This first commit is the chapter's *pre-factor entry*. Chibicc has been growing pre-factors before features for nearly the whole book — the canonicalization-at-parse-time pattern, the type-system passes, the cc1-vs-driver split. The do-nothing preprocessor is the same move, applied to the largest single feature still missing: lay the seam first, fill it in over forty commits. With this commit the *pre-factor-before-feature* count goes from seven to eight.

#### Where we are

`preprocess.c` exists and is wired into the cc1 pipeline. The function does nothing — its only effect is to call `convert_keywords` (which it took over from `tokenize`). The very next commit gives it work to do.

---

### 17.1.2 — The null directive

> `git checkout 146c7b3dd47bb65da2da86cce7f4d75d8efa157d` — *Add the null directive*

The smallest piece of preprocessor syntax that exists is the *null directive* — a line containing only `#` and nothing else. It's legal C and means nothing; it's the kind of thing that shows up in machine-generated code. This commit makes the preprocessor recognize and skip it, and along the way installs the directive-recognizer that every subsequent commit reuses.

The new field on `Token` is a single `bool`:

```diff
   int line_no;    // Line number
+  bool at_bol;    // True if this token is at beginning of line
 };
```

`at_bol` (at beginning of line) is set by the tokenizer when it sees a newline:

```c
static bool at_bol;
// ...
// Skip newline.
if (*p == '\n') {
  p++;
  at_bol = true;
  continue;
}
```

Every newly-emitted token reads the flag and clears it:

```c
tok->at_bol = at_bol;
at_bol = false;
```

So exactly the *first* token after a newline carries `at_bol = true`. Whitespace and comments don't reset it; only a non-trivia token does.

`is_hash` is the directive recognizer:

```c
static bool is_hash(Token *tok) {
  return tok->at_bol && equal(tok, "#");
}
```

A hash sign in the middle of a line is *not* a directive (in chibicc's lexer, `#` is just a punctuation token; it doesn't have an operator meaning yet — `##` and stringizing arrive later in the chapter). Only a hash at the beginning of a line counts.

The preprocessor's main loop is `preprocess2`, separated from the entry-point `preprocess` so it can be called recursively later:

```c
static Token *preprocess2(Token *tok) {
  Token head = {};
  Token *cur = &head;

  while (tok->kind != TK_EOF) {
    // Pass through if it is not a "#".
    if (!is_hash(tok)) {
      cur = cur->next = tok;
      tok = tok->next;
      continue;
    }

    tok = tok->next;

    // `#`-only line is legal. It's called a null directive.
    if (tok->at_bol)
      continue;

    error_tok(tok, "invalid preprocessor directive");
  }

  cur->next = tok;
  return head.next;
}
```

The structure is the *chained-list rewriter* pattern: walk the input, splice non-directive tokens onto the output (`cur = cur->next = tok`), and consume directive tokens specially. Tokens flow through the loop one at a time. When a directive is recognized, it's consumed without producing output (the null directive case here); when it's not a directive, it's spliced through unchanged.

The two key reads are `is_hash` (do we have a directive?) and `tok->at_bol` (have we hit the next line yet?). The loop after `tok = tok->next` reads the *first token after the hash*. If that token is itself at beginning of line — meaning the hash had nothing after it — we have a null directive, and we skip it by `continue`. Otherwise (in this commit) we error out, because no actual directive exists yet.

The test file `test/macro.c` is created in this commit (rather than reusing an existing test file), and lands with two null-directive lines:

```c
#

/* */ #
```

The second case verifies that a comment between newline-and-hash doesn't break recognition. The tokenizer eats the comment (`has_space` doesn't exist yet; comments simply don't produce a token), and `at_bol` carries through to the `#`.

#### Where we are

The directive recognizer is in place. The preprocessor consumes one well-formed directive (the null directive) and rejects everything else. The next commit gives it a real directive to handle.

---

### 17.1.3 — `#include "..."`

> `git checkout d367510fcc1396fa252c4b87439c2f9fcd0abbe7` — *Add `#include "..."`*

`#include` is the preprocessor's most user-visible feature — for many C programmers, *the* preprocessor feature. It's also where the token-list-rewriting model earns its keep: the preprocessor reads a filename token, calls `tokenize_file` on the named file to produce a fresh token list, and *splices that list* into the position where the directive was. The body of the included file flows through `preprocess2` in the next pass over the spliced stream, recursively, with no special handling.

The shape of the splice is a helper called `append`:

```c
// Append tok2 to the end of tok1.
static Token *append(Token *tok1, Token *tok2) {
  if (!tok1 || tok1->kind == TK_EOF)
    return tok2;

  Token head = {};
  Token *cur = &head;

  for (; tok1 && tok1->kind != TK_EOF; tok1 = tok1->next)
    cur = cur->next = copy_token(tok1);
  cur->next = tok2;
  return head.next;
}
```

`append(tok1, tok2)` produces a new list that's the concatenation of `tok1` (truncated at its EOF) and `tok2`. The truncation matters because each `tokenize_file` call returns a list ending in its own EOF token; `append` strips that EOF and stitches the next list onto its tail. `copy_token` is a shallow-copy helper:

```c
static Token *copy_token(Token *tok) {
  Token *t = calloc(1, sizeof(Token));
  *t = *tok;
  t->next = NULL;
  return t;
}
```

The directive case in `preprocess2` is short:

```c
if (equal(tok, "include")) {
  tok = tok->next;

  if (tok->kind != TK_STR)
    error_tok(tok, "expected a filename");

  char *path = format("%s/%s", dirname(strdup(tok->file->name)), tok->str);
  Token *tok2 = tokenize_file(path);
  if (!tok2)
    error_tok(tok, "%s", strerror(errno));
  tok = append(tok2, tok->next);
  continue;
}
```

The argument has to be a string literal (the `"..."` form; the `<...>` form arrives in §17.5). The path is computed relative to *the directory of the file the directive appears in* — `dirname(tok->file->name)` — joined with the filename. That's the C standard's specified search behavior for the double-quoted form. (The standard also says compilers may search additional directories; chibicc's `-I` and default-paths work arrives in §17.5.)

The included file is tokenized into `tok2`. `append(tok2, tok->next)` produces a list with `tok2` at the front and the *rest of the current input* (the tokens after `#include "foo.h"`) at the back. The directive is consumed; the included file's tokens take its place in the stream. The `continue` re-enters the `while` loop, which now reads from the spliced list — meaning the included file is preprocessed by the same `preprocess2` machinery, recursively, including any `#include` directives *it* contains (which is what makes `include1.h` → `include2.h` work, the test exercises both directions).

A subtlety: the path is computed using `tok->file->name`. That requires `Token` to know which file it was tokenized from. The `File` struct is new in this commit:

```c
typedef struct {
  char *name;
  int file_no;
  char *contents;
} File;
```

— and `Token` gains a `File *file` field that the tokenizer populates. The `file_no` field is an integer the assembler's `.file` directive uses (the codegen change in this commit emits `  .file 1 "foo.c"`/`.file 2 "foo.h"` headers based on it; before this commit the codegen hardcoded `.file 1`).

The `tokenize_file` function gains a small bit of bookkeeping that preserves all input files' metadata across translation-unit boundaries:

```c
static File **input_files;
```

Every file that's tokenized is appended to this list. `get_input_files()` returns it for the codegen's `.file` emission step. The numbering matters for GDB: when the debugger sees a `.loc N L` instruction, `N` is the file number, and `.file N "path"` at the top of the assembly file teaches the debugger how to map. After this commit, errors inside an included file produce the right filename, and stepping through stops at the right file.

The error machinery also adapts: pre-this-commit, `error_at` and `error_tok` both reached for a static `current_input` pointer in `tokenize.c`. Post-this-commit, those helpers take a `filename` and `input` argument explicitly, and the variants pull from `tok->file->name` and `tok->file->contents`. Errors inside `include1.h` — the test exercises this — print `test/include1.h:N: ...`, not `test/macro.c:N: ...`.

`read_file` also softens its error handling: where it once errored out on `fopen` failure, it now returns `NULL`, and `tokenize_file` (and its callers) decide what to do:

```diff
-      error("cannot open %s: %s", path, strerror(errno));
+      return NULL;
```

— so `#include` can produce a *located* error (using `error_tok` to point at the directive) rather than a no-location bare `error`.

The included-file tests are minimal:

```c
// test/include1.h
#include "include2.h"
int include1 = 5;

// test/include2.h
int include2 = 7;

// test/macro.c
#include "include1.h"
// ...
assert(5, include1, "include1");
assert(7, include2, "include2");
```

`include1.h` includes `include2.h`. The recursion is implicit: `include1.h`'s tokens flow through `preprocess2`, which sees its `#include` directive, splices `include2.h`'s tokens in, continues the loop. Two levels deep. The Chapter 17 test pipeline never hits any deep stress test of recursion in a single file (chibicc doesn't have re-include guards yet — those are `#pragma once` and the include-guard optimization, which arrive in Chapter 22), but `include1.h`/`include2.h` is the basic shape.

#### Where we are

`#include "..."` works for relative paths (computed from the including file's directory) and absolute paths (the `tok->str[0] == '/'` branch arrives in §17.1.5 with the `-E` commit, but the directive itself works). Each input file gets a `File *` with a unique `file_no`. The codegen emits `.file` directives for all input files, and `.loc` directives reference the right file number, so debug-info-aware tools (gdb, addr2line) can map lines back to the right file. Errors carry the correct filename through include levels.

---

### 17.1.4 — Skip extra tokens after `#include`

> `git checkout ec149f64d2f5c41a2080c0b4e42e4ef64444b382` — *Skip extra tokens after `#include "..."`*

A short cleanup commit. The `#include "foo.h"` directive accepts the filename token and then *should* see end-of-line. Real-world C sometimes puts trailing tokens on directive lines — most often by accident, occasionally by `#include "foo.h" /* comment */` style; the standard says it's a constraint violation, but compilers in practice issue a warning and skip them. Chibicc opts for the skip-with-warning behavior:

```c
// Some preprocessor directives such as #include allow extraneous
// tokens before newline. This function skips such tokens.
static Token *skip_line(Token *tok) {
  if (tok->at_bol)
    return tok;
  warn_tok(tok, "extra token");
  while (tok->at_bol)
    tok = tok->next;
  return tok;
}
```

The first line returns immediately if we're already at the start of the next line. The second issues a warning. The loop walks forward until the next at-beginning-of-line token. Note the loop's condition: `while (tok->at_bol)` *would* spin forever on a token at-beginning-of-line — but the function only enters the loop having already passed the early-return, so `tok` is not at-bol when the loop starts. The first iteration moves to `tok->next`, and the loop re-checks `tok->at_bol`. Reading carefully: this is `while (!tok->at_bol)`, except spelled inverted because the loop *starts* on a known-not-at-bol token. The body increments the pointer and re-checks. It's a small piece of code that took the author at least a moment to read correctly; the comment helps.

`warn_tok` is also new in this commit, a sibling of `error_tok` that prints the same `filename:line: message` format to stderr but doesn't `exit`:

```c
void warn_tok(Token *tok, char *fmt, ...) {
  va_list ap;
  va_start(ap, fmt);
  verror_at(tok->file->name, tok->file->contents, tok->line_no, tok->loc, fmt, ap);
  va_end(ap);
}
```

The `#include` directive is updated to call `skip_line` after consuming the filename:

```diff
-      tok = append(tok2, tok->next);
+      tok = skip_line(tok->next);
+      tok = append(tok2, tok);
```

— the `tok->next` after the filename is the start of the *trailing* tokens (or the newline), `skip_line` advances past them with a warning if any, and the result is what gets stitched after the included file's body.

This commit also adds the `warn_tok` declaration to `chibicc.h` and is otherwise tiny. It's useful as a preview of the chapter's larger pattern: `skip_line` is going to be reused by every directive. Object-like `#define`, function-like `#define`, `#undef`, `#if`, `#elif`, `#else`, `#endif`, `#ifdef`, `#ifndef`, `#error`, `#include` (both forms) all end with `skip_line` to consume trailing junk.

#### Where we are

`#include "foo.h"` extra tokens emit a warning and are skipped. The `skip_line` helper is the basic discipline that every later directive uses to clean up its line.

---

### 17.1.5 — `-E`

> `git checkout d138864a2a99849e43d81ca071b7a799edc0e65a` — *Add `-E` option*

`-E` is the GCC-style flag that runs only the preprocessor stage and prints the result to stdout (or the `-o` path). With `-E`, the tokens aren't parsed; they're rendered back to text and the process exits. This commit adds `-E` to chibicc's argument parser, adds a `print_tokens` function that renders the post-`preprocess` token list, and adjusts the driver to dispatch `-E` to a cc1 invocation that produces preprocessed output instead of assembly.

The driver-level change is a new `opt_E` flag and a per-input dispatch:

```c
// Just preprocess
if (opt_E) {
  run_cc1(argc, argv, input, NULL);
  continue;
}
```

The cc1 callee handles `-E` by branching after `preprocess()`:

```c
static void cc1(void) {
  // Tokenize and parse.
  Token *tok = tokenize_file(base_file);
  if (!tok)
    error("%s: %s", base_file, strerror(errno));

  tok = preprocess(tok);

  // If -E is given, print out preprocessed C code as a result.
  if (opt_E) {
    print_tokens(tok);
    return;
  }

  Obj *prog = parse(tok);
  // ...
}
```

`print_tokens` walks the token list and emits each token's text, separating them with spaces and newlines based on `at_bol`:

```c
static void print_tokens(Token *tok) {
  FILE *out = open_file(opt_o ? opt_o : "-");

  int line = 1;
  for (; tok->kind != TK_EOF; tok = tok->next) {
    if (line > 1 && tok->at_bol)
      fprintf(out, "\n");
    fprintf(out, " %.*s", tok->len, tok->loc);
    line++;
  }
  fprintf(out, "\n");
}
```

Each token's *text* is its slice of the original file's contents — `tok->loc` and `tok->len` point into the source. For tokens that came from `#include`d files, `tok->loc` points into *that* file's contents, which is what we want: the printed form of an included file's contents reproduces the included file's text.

The `at_bol` flag drives the newlines: every time we hit a token marked at-beginning-of-line (after the first token), we print a `\n` first, so the output preserves the source's vertical structure. Adjacent tokens in the same line get separated by a single leading space (the `" %.*s"` format string). Whitespace *within* a line isn't preserved — the next commit (in §17.4) will add a `has_space` flag to fix that — but the line structure is preserved.

There's a subtle bug-or-feature in this output format. The `" %.*s"` always prints a leading space. For the first token of a line, that's an indent. For the very first token of the whole file, it's an indent that probably shouldn't be there. The `-E` output is intended to be human-inspectable rather than re-feedable to the compiler, so the leading space doesn't break anything — and the stage-2 build doesn't use `-E` (it'll feed `chibicc` directly without `-E` interposition once the preprocessor handles `#include` itself). It's fine.

The driver-side validation grows: `-o` with multiple inputs is now an error if any of `-c`, `-S`, `-E` is set:

```diff
-  if (input_paths.len > 1 && opt_o && (opt_c || opt_S))
-    error("cannot specify '-o' with '-c' or '-S' with multiple files");
+  if (input_paths.len > 1 && opt_o && (opt_c || opt_S | opt_E))
+    error("cannot specify '-o' with '-c,' '-S' or '-E' with multiple files");
```

(The bitwise `|` between `opt_S` and `opt_E` is a typo — should be `||`. It compiles, and the values are 0/1 so `|` happens to do the same thing as `||`, but it's the kind of thing a reviewer would flag. Errata candidate. We'll catalog with the rest of Chapter 17's small slips at the chapter's close.)

A second small adjustment in this commit handles absolute include paths:

```c
char *path;
if (tok->str[0] == '/')
  path = tok->str;
else
  path = format("%s/%s", dirname(strdup(tok->file->name)), tok->str);
```

— the `#include "/path/to/file"` form bypasses the directory-relative resolution. (This is non-standard for the double-quoted form; the standard says implementation-defined search applies. Chibicc accepts the absolute path verbatim.)

The driver tests for `-E`:

```bash
echo foo > $tmp/out
echo "#include \"$tmp/out\"" | $chibicc -E - | grep -q foo
check -E

echo foo > $tmp/out1
echo "#include \"$tmp/out1\"" | $chibicc -E -o $tmp/out2 -
cat $tmp/out2 | grep -q foo
check '-E and -o'
```

— a `#include`-via-stdin smoke test, with both stdout-output and `-o` redirection.

#### Where we are

The preprocessor's user-visible surface starts to take shape. `-E` lets users (and the test pipeline, eventually) see what the preprocessor produced. The `print_tokens` formatter is intentionally lossy — it doesn't preserve whitespace, doesn't preserve all separators perfectly — but it produces output that's recognizable as the input minus the `#include` lines and with their bodies inlined.

#### Where the section leaves us

Five commits, and chibicc has the *shape* of a preprocessor:

- `preprocess()` is wired between `tokenize_file()` and `parse()`. Tokens flow through, get rewritten or passed through, and feed the parser.
- The `at_bol` flag on `Token` distinguishes column-zero hashes from in-line hashes.
- The directive recognizer (`is_hash`) and the line-skipper (`skip_line`) are the building blocks every later directive uses.
- `#include "..."` works recursively, with proper file numbering for `.loc` debug info.
- `-E` runs the preprocessor and prints the result.

The preprocessor knows two directives so far: the null directive and `#include`. The next section adds four more: `#if`, `#endif`, `#else`, `#elif`. With those, conditional compilation works.

---

## 17.2 — Conditionals

Four commits. Conditional compilation is the C preprocessor's other core feature, alongside macros — `#if expression`/`#endif`, with `#else` and `#elif` for branches. Unlike `#include`, `#if` doesn't *splice* tokens into the stream; it *deletes* whole regions of source. The preprocessor reads `#if`'s argument as a constant expression, evaluates it, and either preserves or skips the body until the matching `#endif`.

The interesting piece is the constexpr evaluator. `#if` runs at preprocess time, before the AST exists, so it can't use the parser's `eval`/`eval2`/`eval_double` quartet — those operate on AST nodes. Instead, the preprocessor builds a *temporary* token list from the directive's argument, hands it to `const_expr` (which is now exposed from `parse.c`), gets back a `long`, and uses that as the truth value. The token-level evaluator is, in effect, a small parser-and-evaluator running in front of the main parser.

### 17.2.1 — `#if` and `#endif`

> `git checkout bf6ff928ad17d98d07f68f619e6cbe29829d0a20` — *Add `#if` and `#endif`*

The constexpr evaluator first. `parse.c`'s `const_expr` is promoted from `static` to externally visible, with a new declaration in `chibicc.h`:

```c
int64_t const_expr(Token **rest, Token *tok);
```

The function itself is unchanged: it parses a conditional expression and runs `eval` on the resulting AST node. What's new is that it can be called from `preprocess.c`. The preprocessor builds a synthetic token list that contains just the directive's argument, hands it to `const_expr`, and uses the returned `int64_t`.

The synthetic-list machinery is `copy_line` and `new_eof`:

```c
static Token *new_eof(Token *tok) {
  Token *t = copy_token(tok);
  t->kind = TK_EOF;
  t->len = 0;
  return t;
}

static Token *copy_line(Token **rest, Token *tok) {
  Token head = {};
  Token *cur = &head;

  for (; !tok->at_bol; tok = tok->next)
    cur = cur->next = copy_token(tok);

  cur->next = new_eof(tok);
  *rest = tok;
  return head.next;
}
```

`copy_line` reads tokens until the next at-bol (the next line's first token), copies each one into a fresh list, and terminates that list with a synthetic EOF. The `*rest` output is set to the at-bol token — i.e., the *next* line's first token, which is where the preprocessor will resume after the directive. `new_eof` makes a copy of an existing token and changes its kind to `TK_EOF`; the copy is used so the synthetic EOF carries a real source location for error reporting.

`eval_const_expr` wraps `const_expr` in this synthetic-list discipline:

```c
static long eval_const_expr(Token **rest, Token *tok) {
  Token *start = tok;
  Token *expr = copy_line(rest, tok->next);

  if (expr->kind == TK_EOF)
    error_tok(start, "no expression");

  Token *rest2;
  long val = const_expr(&rest2, expr);
  if (rest2->kind != TK_EOF)
    error_tok(rest2, "extra token");
  return val;
}
```

The argument-line tokens are copied (with `tok->next` because `tok` itself is the `if` keyword), passed to `const_expr`, and the result is checked for "no leftover tokens." If the expression doesn't consume the whole synthetic line, that's an error: `#if 1 +` (with no second operand) lands here.

Note the duplication. Chibicc has, post this commit, *two* constant-expression evaluation paths:

- `parse.c`'s `eval`/`eval2`/`eval_rval`/`eval_double` quartet, which runs over AST nodes and is used by initializers, array sizes, case labels, and bitfield widths.
- `preprocess.c`'s `eval_const_expr`, which calls `const_expr` (which calls `eval`) but operates on a synthesized token list and runs *before* the main parse.

The two share `eval` itself — the evaluator at the AST-node level — but go through different parsers (the preprocessor builds the AST through `const_expr` → `conditional`, which is the same recursive-descent stack as the main parser's). The duplication is mostly *temporary* token-list machinery; the *evaluation* is unified through `eval`.

The conditional state is held on a stack:

```c
typedef struct CondIncl CondIncl;
struct CondIncl {
  CondIncl *next;
  Token *tok;
};

static CondIncl *cond_incl;
```

`cond_incl` is a singly-linked list (used as a stack) that grows on `#if`/`#ifdef`/`#ifndef` and shrinks on `#endif`. The `tok` field stores the directive's `#` token, which is what `error_tok` reports if the conditional is unterminated.

The `if` and `endif` directive cases:

```c
if (equal(tok, "if")) {
  long val = eval_const_expr(&tok, tok);
  push_cond_incl(start, val);
  if (!val)
    tok = skip_cond_incl(tok);
  continue;
}

if (equal(tok, "endif")) {
  if (!cond_incl)
    error_tok(start, "stray #endif");
  cond_incl = cond_incl->next;
  tok = skip_line(tok->next);
  continue;
}
```

`#if` evaluates the constant, pushes a new entry on the stack (the `included` field arrives in §17.2.3), and — if the value is zero — calls `skip_cond_incl` to fast-forward past the conditional body to the matching `#endif`. `#endif` pops the stack.

`skip_cond_incl` is the simplest version of itself:

```c
// Skip until next `#endif`.
static Token *skip_cond_incl(Token *tok) {
  while (tok->kind != TK_EOF) {
    if (is_hash(tok) && equal(tok->next, "endif"))
      return tok;
    tok = tok->next;
  }
  return tok;
}
```

It scans forward looking for `#endif`. The next commit adds nesting; this one doesn't handle nested `#if` correctly. The test deliberately constructs an `#if 0`-skipping case where the body contains `#include "/no/such/file"` — the test passes because the include directive is *inside* the skipped region and never gets executed.

A small fix in this commit: a one-character correction to `tokenize.c`'s error-line printer that addresses an EOF-without-newline case introduced by `read_file`'s changed handling:

```diff
-  while (*end != '\n')
+  while (*end && *end != '\n')
```

— the loop now stops at the file's terminator, not just at a newline. This is what makes `#if 1\n#endif\n` (without trailing data) work cleanly through `error_at`'s line-printer when an error happens to fall on the last line.

The terminating check at the end of `preprocess()` is added:

```c
Token *preprocess(Token *tok) {
  tok = preprocess2(tok);
  if (cond_incl)
    error_tok(cond_incl->tok, "unterminated conditional directive");
  convert_keywords(tok);
  return tok;
}
```

— a missing `#endif` at end-of-file gets caught here.

### 17.2.2 — Skip nested `#if` in a skipped clause

> `git checkout aa570f3086ce3e2c5ac8bf6107c051fed5aabf89` — *Skip nested `#if` in a skipped `#if`-clause*

A six-line correction. The previous commit's `skip_cond_incl` walks forward to the *first* `#endif` it finds, regardless of nesting. That breaks on:

```c
#if 0
#if nested
#endif
#endif
```

— the `#endif` matching `#if nested` is found first, leaving the outer `#if 0` open. The fix:

```c
static Token *skip_cond_incl(Token *tok) {
  while (tok->kind != TK_EOF) {
    if (is_hash(tok) && equal(tok->next, "if")) {
      tok = skip_cond_incl(tok->next->next);
      tok = tok->next;
      continue;
    }
    if (is_hash(tok) && equal(tok->next, "endif"))
      break;
    tok = tok->next;
  }
  return tok;
}
```

— recursive: when `#if` is found inside the skipped region, recurse to find *its* matching `#endif`, then continue the outer scan. The recursion's depth bounds the deepest `#if` nesting; chibicc doesn't enforce a limit, but the C standard's minimum-implementation requirement is fifteen levels of conditional inclusion, which a recursive scan handles trivially.

### 17.2.3 — `#else`

> `git checkout c6e81d22f8189cd7bfcfcc33e4ac462529418192` — *Add `#else`*

`#else` is a substantial commit — forty lines in `preprocess.c`. The structure decision: when `#if 1`'s body is *included*, the matching `#else`'s body must be *excluded*, and vice versa. The conditional state needs to carry which case applies.

`CondIncl` grows two fields:

```c
struct CondIncl {
  CondIncl *next;
  enum { IN_THEN, IN_ELSE } ctx;
  Token *tok;
  bool included;
};
```

`included` records whether the `#if`/`#elif` body was kept (i.e., the constant was true). `ctx` tracks which clause we're currently in (`IN_THEN` for the body of `#if`, `IN_ELSE` for the body of `#else`). The `ctx` field guards against duplicate `#else` (`#else ... #else` is a stray-else error).

`push_cond_incl` updates to take the included-flag:

```c
static CondIncl *push_cond_incl(Token *tok, bool included) {
  CondIncl *ci = calloc(1, sizeof(CondIncl));
  ci->next = cond_incl;
  ci->ctx = IN_THEN;
  ci->tok = tok;
  ci->included = included;
  cond_incl = ci;
  return ci;
}
```

— and `#if` calls `push_cond_incl(start, val)`, recording whether the value was non-zero.

The skipper splits in two. `skip_cond_incl2` is the inner (nested-only) scanner used during nested skip; it runs to `#endif` only and ignores `#else`/`#elif`. `skip_cond_incl` (renamed to mean *outer* skip) runs to `#else`, `#elif`, or `#endif` — whichever comes first at the current nesting level:

```c
static Token *skip_cond_incl2(Token *tok) {
  while (tok->kind != TK_EOF) {
    if (is_hash(tok) && equal(tok->next, "if")) {
      tok = skip_cond_incl2(tok->next->next);
      continue;
    }
    if (is_hash(tok) && equal(tok->next, "endif"))
      return tok->next->next;
    tok = tok->next;
  }
  return tok;
}

static Token *skip_cond_incl(Token *tok) {
  while (tok->kind != TK_EOF) {
    if (is_hash(tok) && equal(tok->next, "if")) {
      tok = skip_cond_incl2(tok->next->next);
      continue;
    }
    if (is_hash(tok) &&
        (equal(tok->next, "else") || equal(tok->next, "endif")))
      break;
    tok = tok->next;
  }
  return tok;
}
```

The two-function split is necessary because at the *outermost* level we want to *stop* at `#else`/`#elif` to give the conditional machinery a chance to evaluate them, but at *inner* nested levels we want to *skip past* `#else`/`#elif` because they belong to the inner conditional.

The `else` directive case:

```c
if (equal(tok, "else")) {
  if (!cond_incl || cond_incl->ctx == IN_ELSE)
    error_tok(start, "stray #else");
  cond_incl->ctx = IN_ELSE;
  tok = skip_line(tok->next);

  if (cond_incl->included)
    tok = skip_cond_incl(tok);
  continue;
}
```

— if the `#if` clause was included, we're now in the `#else` clause and need to *skip* it (and the skip will land at the matching `#endif`). If the `#if` clause was excluded, we'd already arrived at the `#else` via the previous `skip_cond_incl`, and now we just continue as if `#else`'s body is the current included region.

The state machine: `included = false, ctx = IN_THEN` means we're currently in a `#else` body (we got skipped here because `#if 0`); `included = true, ctx = IN_THEN` means we're currently in a `#if` body, and any `#else` we encounter should be skipped; etc. The `included` flag plus the `ctx` flag together specify "what to do at the next `#else`/`#elif`."

The test exercises the four combinations and a deeply-nested case:

```c
#if 1
# if 0
#  if 1
    foo bar
#  endif
# endif
      m = 3;
#endif
    assert(3, m, "m");
```

— the `foo bar` line is in a doubly-skipped region; if the nested-skipper miscounted, the assignment `m = 3` would be skipped too.

### 17.2.4 — `#elif`

> `git checkout e7a1857a31fc0c0012773c021639a6297f5b208f` — *Add `#elif`*

`#elif` is `#else if` collapsed into one directive. The semantics: when `#if`'s body was excluded (the constant was false), the *first* `#elif` whose constant evaluates true takes over; subsequent `#elif`s and the final `#else` are skipped. The `included` flag tracks whether *any* clause has fired yet.

`CondIncl::ctx` gains a third state, `IN_ELIF`:

```c
enum { IN_THEN, IN_ELIF, IN_ELSE } ctx;
```

The skipper functions update to recognize `#elif`:

```c
if (is_hash(tok) &&
    (equal(tok->next, "elif") || equal(tok->next, "else") ||
     equal(tok->next, "endif")))
  break;
```

The `elif` directive case:

```c
if (equal(tok, "elif")) {
  if (!cond_incl || cond_incl->ctx == IN_ELSE)
    error_tok(start, "stray #elif");
  cond_incl->ctx = IN_ELIF;

  if (!cond_incl->included && eval_const_expr(&tok, tok))
    cond_incl->included = true;
  else
    tok = skip_cond_incl(tok);
  continue;
}
```

— if no clause has fired yet (`!cond_incl->included`) and *this* clause's expression is true, fire this clause (set `included = true`, fall through to process the body normally). Otherwise skip to the next `#elif`/`#else`/`#endif`.

Reading the test:

```c
#if 0
  m = 1;
#elif 0
  m = 2;
#elif 3+5
  m = 3;
#elif 1*5
  m = 4;
#endif
  assert(3, m, "m");
```

— `#if 0` is excluded, `#elif 0` is excluded, `#elif 3+5` (= 8, true) fires and sets `m = 3`. The fourth `#elif 1*5` (also true) is skipped because `included` is now true.

`#elif` and `#else` interact correctly because `#else`'s test (`if (!cond_incl->included)` is implicit in the order — `#else` only fires if no prior `#if`/`#elif` set `included = true`), and the `IN_ELSE` ctx prevents a `#elif` from following an `#else`.

#### Where the section leaves us

Four conditional directives (`#if`/`#elif`/`#else`/`#endif`) plus their `#ifdef`/`#ifndef` cousins (which arrive in §17.3). The constexpr evaluator runs on synthetic token lists, calling into `parse.c`'s `const_expr`, with no AST. Stack-based state lets conditionals nest.

The chapter's *eval-quartet duplication* note: chibicc has, after §17.2, two constant-expression entry points — the AST-level one (used by initializers, array sizes, case labels, bitfield widths) and the token-level one (used by `#if`/`#elif`). They share `eval` itself but go through different parsers. This is the kind of duplication Rui's design notes (Chapter 16's `quotes-rui.md`: "It's better to allow small duplications instead") explicitly endorses; the alternative would be a unified token-or-AST evaluator that's harder to read.

---

## 17.3 — Object-like macros, the hideset, and `#ifdef`

Five commits. `#define X 5` and `X` becomes `5` everywhere, with the wrinkle that `#define X X + 1` should expand `X` to `X + 1` once and stop, rather than infinitely. The infinite-recursion guard is the hideset machinery — a *set of macro names* attached to each token, recording which macros have already had a shot at expanding into this token's neighborhood. Once a macro's name is in a token's hideset, that token is invisible to that macro.

### 17.3.1 — Object-like `#define`

> `git checkout 97d33ad3bdc21c26356253046902d4b166bd115b` — *Add objlike `#define`*

Macros are stored as a singly-linked list:

```c
typedef struct Macro Macro;
struct Macro {
  Macro *next;
  char *name;
  Token *body;
};

static Macro *macros;
```

The body is a token list, terminated by EOF. `add_macro` cons-prepends a new entry:

```c
static Macro *add_macro(char *name, Token *body) {
  Macro *m = calloc(1, sizeof(Macro));
  m->next = macros;
  m->name = name;
  m->body = body;
  macros = m;
  return m;
}
```

— and because lookups walk the list in order, a redefinition (a second `#define` of the same name) shadows the first. `find_macro` is the linear-scan lookup keyed on `Token`'s text:

```c
static Macro *find_macro(Token *tok) {
  if (tok->kind != TK_IDENT)
    return NULL;

  for (Macro *m = macros; m; m = m->next)
    if (strlen(m->name) == tok->len && !strncmp(m->name, tok->loc, tok->len))
      return m;
  return NULL;
}
```

`expand_macro` is the simplest possible expander:

```c
static bool expand_macro(Token **rest, Token *tok) {
  Macro *m = find_macro(tok);
  if (!m)
    return false;
  *rest = append(m->body, tok->next);
  return true;
}
```

— append the macro's body to the rest of the input, return true (signaling the caller to re-run the loop on the spliced result). The body's tokens become part of the input stream. The next iteration through `preprocess2`'s loop reads the body's first token and processes it normally (passing it through, recognizing it as a directive, *or* expanding it as another macro if it matches).

The directive case in `preprocess2`:

```c
if (equal(tok, "define")) {
  tok = tok->next;
  if (tok->kind != TK_IDENT)
    error_tok(tok, "macro name must be an identifier");
  char *name = strndup(tok->loc, tok->len);
  add_macro(name, copy_line(&tok, tok->next));
  continue;
}
```

The body is `copy_line(&tok, tok->next)` — the rest of the line, copied (so the macro body has its own token storage, decoupled from the source's), terminated with the synthetic EOF.

The expand check is added to the top of `preprocess2`:

```c
while (tok->kind != TK_EOF) {
  // If it is a macro, expand it.
  if (expand_macro(&tok, tok))
    continue;
  // ... rest of the loop ...
}
```

Now ordinary identifiers get checked against the macro table before they're treated as identifiers. If they match, the body splices in and the loop re-enters.

The infinite-recursion problem doesn't surface yet because the test cases this commit ships with don't construct a self-referential macro. Tests like `#define M1 3` followed by `M1` work fine: the body is `3`, which isn't a macro, so the next loop iteration passes it through.

But consider `#define M2 M2 + 3`. The body is `M2 + 3`; the first token of the body is `M2` itself; the next loop iteration sees `M2`, looks it up in the macro table, finds itself, and splices `M2 + 3` again. Infinite expansion. Three commits below, the no-rescan rule fixes this.

### 17.3.2 — `#undef`

> `git checkout 9ad60e41d512158d942d1bf3808682ede6ef5118` — *Add `#undef`*

`#undef` removes a macro definition. Because chibicc's macro table is a stack-of-redefinitions where the most recent wins, `#undef` works by adding a *deleted* entry that shadows the (possibly multiple) earlier entries:

```c
struct Macro {
  Macro *next;
  char *name;
  Token *body;
  bool deleted;
};
```

`find_macro` skips deleted entries:

```c
for (Macro *m = macros; m; m = m->next)
  if (strlen(m->name) == tok->len && !strncmp(m->name, tok->loc, tok->len))
    return m->deleted ? NULL : m;
```

— the lookup *finds* the deleted entry (it's the most-recently-pushed, so it's seen first), and returns NULL because of the `deleted` flag. The earlier (non-deleted) entries are never reached.

The directive case:

```c
if (equal(tok, "undef")) {
  tok = tok->next;
  if (tok->kind != TK_IDENT)
    error_tok(tok, "macro name must be an identifier");
  char *name = strndup(tok->loc, tok->len);
  tok = skip_line(tok->next);

  Macro *m = add_macro(name, NULL);
  m->deleted = true;
  continue;
}
```

`#undef`-of-an-undefined-macro is a no-op (the deleted entry is added, but no earlier definition exists; subsequent lookups find it, see deleted, and return NULL — same as if it wasn't there).

The test exercises a redefinition case where the macro replaces a *keyword*:

```c
#define if 5
// ...
ASSERT_ 5, if, five END;

#undef if
// ...
if (0);
```

— while `if` is `#define`d to `5`, the source `if (0);` would expand to `5 (0);` and miscompile. After `#undef if`, the keyword reasserts itself. (Recall: `convert_keywords` runs *after* `preprocess`, so a macro with a keyword name actually wins during preprocessing — the macro lookup happens before keyword classification.)

### 17.3.3 — Expand macros in `#if` and `#elif` arguments

> `git checkout 2651448084a56dd0b960989798772e71e12e6c30` — *Expand macros in the `#if` and `#elif` argument context*

Three lines and a forward declaration. The `eval_const_expr` function gets a recursive call to `preprocess2` over its synthetic line:

```c
static Token *preprocess2(Token *tok);
```

```diff
 static long eval_const_expr(Token **rest, Token *tok) {
   Token *start = tok;
   Token *expr = copy_line(rest, tok->next);
+  expr = preprocess2(expr);

   if (expr->kind == TK_EOF)
     error_tok(start, "no expression");
```

The synthetic line is now run through the preprocessor before being handed to `const_expr`. That means `#if M` expands `M` to its body, then evaluates the body as a constant expression.

A subtle effect: `preprocess2` runs the *full* directive recognizer, which would treat a `#if` inside `#if`'s argument as nested. In practice, directive arguments don't contain hashes (the C standard prohibits it), so the `is_hash` check inside `preprocess2`'s recursive call doesn't fire. The recursion is, in practice, *only* macro expansion plus `#if`-style passes-through.

Test:

```c
#define M 5
#if M
  m = 5;
#else
  m = 6;
#endif
  assert(5, m, "m");
```

— `#if M` becomes `#if 5`, which is true.

### 17.3.4 — The no-rescan rule and the hideset

> `git checkout acce00228b842af35df5af8c97398765a386ab1e` — *Do not expand a token more than once for the same objlike macro*

This is the most subtle commit in the chapter, and possibly in the book. The `#define X X + 1` problem — what stops `X` from expanding to `X + 1` infinitely? The C standard's answer is: *each token, in the result of macro expansion, carries a record of which macros have already been expanded at it.* The set of those macro names is called a *hideset*. When a token's hideset contains macro `M`, the preprocessor will not expand that token via `M`. Self-reference terminates: the expansion `X` → `X + 1` paints the result token's hideset with `{X}`, so the rescan sees `X` (with hideset `{X}`) and doesn't try to expand it.

The data structure is a singly-linked list of macro names per token:

```c
typedef struct Hideset Hideset;
struct Hideset {
  Hideset *next;
  char *name;
};
```

— and `Token` gains a `Hideset *hideset` field. (No relation to `at_bol` / `has_space` — the hideset is per-token state used purely by the preprocessor.)

The hideset operations are union, intersection (added in §17.4 for function-like macros), and contains:

```c
static Hideset *new_hideset(char *name) { /* ... */ }

static Hideset *hideset_union(Hideset *hs1, Hideset *hs2) {
  Hideset head = {};
  Hideset *cur = &head;

  for (; hs1; hs1 = hs1->next)
    cur = cur->next = new_hideset(hs1->name);
  cur->next = hs2;
  return head.next;
}

static bool hideset_contains(Hideset *hs, char *s, int len) {
  for (; hs; hs = hs->next)
    if (strlen(hs->name) == len && !strncmp(hs->name, s, len))
      return true;
  return false;
}
```

`hideset_union` is destructive on hs2 (it appends hs1's elements as fresh links and stitches hs2 onto the end); for the in-pass rewriter pattern this is fine because the set's tail is never rewound.

`add_hideset` paints the union onto every token in a list:

```c
static Token *add_hideset(Token *tok, Hideset *hs) {
  Token head = {};
  Token *cur = &head;

  for (; tok; tok = tok->next) {
    Token *t = copy_token(tok);
    t->hideset = hideset_union(t->hideset, hs);
    cur = cur->next = t;
  }
  return head.next;
}
```

— each result token is a fresh copy so the original `Macro->body` is never mutated (the same body might be expanded many times, in many contexts, with many different hidesets).

The expansion hook:

```c
static bool expand_macro(Token **rest, Token *tok) {
  if (hideset_contains(tok->hideset, tok->loc, tok->len))
    return false;

  Macro *m = find_macro(tok);
  if (!m)
    return false;

  Hideset *hs = hideset_union(tok->hideset, new_hideset(m->name));
  Token *body = add_hideset(m->body, hs);
  *rest = append(body, tok->next);
  return true;
}
```

Read top-down. *If this token's hideset contains a macro of its own name, do not expand.* That's the gate — the test that makes recursion terminate. Otherwise, look up the macro. The replacement's hideset is the **union** of the source token's hideset (whatever painted *it*) and a singleton `{m->name}` (this expansion's own contribution). The replacement tokens all get that union painted on them via `add_hideset`. Then splice and return.

The walked example for `#define M2 M2 + 3` followed by `M2`:

1. The source `M2` token has empty hideset. `expand_macro` is called with that token.
2. The hideset doesn't contain `M2`, so we don't bail.
3. `find_macro` returns the `M2` macro.
4. New hideset: union of `{}` and `{M2}` is `{M2}`.
5. The body `M2 + 3` has each of its three tokens copied, with hideset `{M2}` painted on each.
6. `*rest` now starts with the painted `M2`, then `+`, then `3`, then the rest of the input.
7. The outer `preprocess2` loop iterates. It calls `expand_macro` on the painted `M2`.
8. The hideset contains `M2`. Bail.
9. The painted `M2` is then passed through as an ordinary identifier (it's not a directive, not another expandable macro).
10. After parsing, `M2` is an undefined identifier — a compile error if the source actually references it as a value. The test `#define M2 M2 + 3` followed by `int M2 = 6;` followed by `assert(9, M2, ...)` works because the *first* `M2` (in the assertion) has empty hideset, expands to `M2 + 3` (with `{M2}` hideset on each), then the painted `M2` reaches the parser as the variable reference (which has value 6), so the result is `6 + 3 = 9`.

The walked test for two-macro mutual recursion:

```c
int M4 = 3;
#define M4 M5 * 5
#define M5 M4 + 2
ASSERT(13, M4);
```

Expansion of the source `M4` (empty hideset):

1. Painted body of `M4`: `M5 * 5`, each with hideset `{M4}`.
2. Splice. Loop reads `M5` (hideset `{M4}`).
3. `M5`'s hideset doesn't contain `M5`. Look up; find `M5`. New hideset: `{M4} ∪ {M5}` = `{M4, M5}`.
4. Painted body of `M5`: `M4 + 2`, each with hideset `{M4, M5}`.
5. Splice. Loop reads `M4` (hideset `{M4, M5}`).
6. `M4`'s hideset contains `M4`. Bail. `M4` passes through.
7. Loop reads `+`, `2`, `*` (from the original `M4` body after the painted `M5`), `5`. All non-macro. All pass through.
8. Final token stream: `M4 + 2 * 5`, where the bare `M4` is the variable reference (value 3). Result: `3 + 2*5 = 13`.

The bookend asserts the value: `assert(13, M4, "M4")`.

The hideset's elegance is twofold. First, it's *correct*: the rule terminates regardless of how complicated the macro graph is, because no token can be expanded by the same macro twice. Second, it's *local*: the rule operates on the token, not on global counters or recursion-depth limits. A token painted with `{M4, M5}` carries that paint with it through subsequent expansions, even into completely unrelated regions of the program.

The algorithm's name is *Prosser's algorithm*. Rui's `preprocess.c` opens with a 23-line comment block explaining the rule, and links to a copy of Prosser's 1986 document in the chibicc wiki. The C standard's wording on macro expansion is, in the standards committee's own footnote, "based on" Prosser's algorithm. (Some informal C-implementation literature calls it the "blue paint" algorithm — the same metaphor: a macro's name "paints" the result tokens in its color, and a painted token can never be re-expanded by the painter.)

#### A concept interlude — why the paint must terminate

It's worth pausing on the *why* of the rule, separately from the implementation.

The C preprocessor is a token-rewriting system. A naive token-rewriting system can diverge: if rule R rewrites X to a phrase containing X, and we re-apply R to the result, we get an infinite expansion. Real-world rewrite systems handle this through various means — confluence proofs, termination measures, depth limits. The C preprocessor uses a finer-grained device: it records, *per token*, which rewrite rules have already applied at that position, and forbids the same rule from applying twice.

Why per-token? Because the alternative — a global "we already expanded X once in this file" rule — is too coarse. The user might write:

```c
#define X 1
int a = X;
#undef X
#define X 2
int b = X;
```

— and expect `a = 1, b = 2`. A global rule that disabled `X` after its first expansion would give `a = 1, b = X`. The per-token rule lets `X` be expanded once at the first reference and once at the second; the *result tokens* carry the paint, not the *macro definition*.

Why a *set* per token, rather than a single name? Because macros nest. If `X` expands to `Y X`, the result `Y X` should have `{X}` painted on its tokens (so the inner `X` doesn't re-expand). If `Y` then expands to `Z Y`, the result of *that* should have `{X, Y}` painted (so neither `X` nor `Y` re-expands). The set grows as expansions nest. Each expansion's contribution is its own name; the set is the cumulative paint.

The alternative implementation — a recursion-depth limit, "stop after N nested expansions" — is what some non-standard preprocessors use. It's *almost* correct for normal code (no real macro nests deeper than 100), but it fails the same `#define X X` test for principled reasons: depth N+1 *does* equal depth N, which is finite, so the recursion would stop, but for the wrong reason. The depth limit is a safety net, not a semantic rule. The hideset is a semantic rule.

The chibicc implementation lifts the algorithm directly from Prosser. The function-like-macro variant in §17.4 introduces hideset *intersection* (rather than union) at the closing-paren of an argument list, because the tokens spanning a function-like-macro invocation may have *different* hidesets, and the standard requires the hideset of the result to be only those macros *all* of the spanning tokens agreed on. We'll work through that in §17.4.

### 17.3.5 — `#ifdef` and `#ifndef`

> `git checkout 1f80f581e517ae4a5df6ab38af48a0d2a1089c73` — *Add `#ifdef` and `#ifndef`*

Two more conditional directives. `#ifdef X` is shorthand for `#if defined(X)` (which chibicc doesn't have yet — `defined()` is in §17.5). The implementation reads the next token as an identifier and tests `find_macro`:

```c
if (equal(tok, "ifdef")) {
  bool defined = find_macro(tok->next);
  push_cond_incl(tok, defined);
  tok = skip_line(tok->next->next);
  if (!defined)
    tok = skip_cond_incl(tok);
  continue;
}

if (equal(tok, "ifndef")) {
  bool defined = find_macro(tok->next);
  push_cond_incl(tok, !defined);
  tok = skip_line(tok->next->next);
  if (defined)
    tok = skip_cond_incl(tok);
  continue;
}
```

The two skip-functions update to recognize `#ifdef` and `#ifndef` as starting-tokens for nested skipping:

```c
if (is_hash(tok) &&
    (equal(tok->next, "if") || equal(tok->next, "ifdef") ||
     equal(tok->next, "ifndef"))) {
```

— so `#if 0\n#ifdef X\n#endif\n#endif` is handled correctly.

`#define M6` (with no body) followed by `#ifdef M6` works because `#define M6` adds a macro with empty body; `find_macro(M6)` returns non-NULL; `defined = true`; the body is included.

#### Where the section leaves us

The full object-like-macro surface: definitions, redefinitions, undefinitions, expansion-in-conditional, expansion-elsewhere, the no-rescan rule, and `#ifdef`/`#ifndef`. The hideset is per-token state, painted on every result token by `add_hideset`, gated on by `expand_macro`'s `hideset_contains` check. The data structure is a small linked list; the operations are union and contains for the object-like case (intersection arrives in §17.4).

The duplication-and-redundancy note: chibicc has, after §17.3, *six* directives that all end with `skip_line(tok)` to consume trailing junk. They're each five-to-seven lines of code, repeating the same pattern: read, validate, dispatch, skip_line. Rui doesn't factor out a `read_directive` helper because the dispatch differs across them (`#define` reads an identifier and a body; `#if` reads a constexpr; `#ifdef` reads an identifier and tests it). The repetition is what `quotes-rui.md` calls "small duplications" — tolerated rather than abstracted.

---

## 17.4 — Function-like macros, stringizing, and pasting

Seven commits. Function-like macros — `#define MAX(a, b) ((a) < (b) ? (b) : (a))` — are the C preprocessor's most code-heavy feature. They split into a definition stage (read parameter names), an invocation stage (read argument tokens, one per parameter, balancing parentheses), a substitution stage (walk the body, replace parameter names with their arguments), and an expansion stage (the substituted body becomes the replacement, with the no-rescan rule applied via the closing-paren hideset). Stringizing (`#x`) and pasting (`##`) are operators that act inside the substitution stage on parameter references.

### 17.4.1 — Zero-arity function-like `#define`

> `git checkout dec3b3fa02ffb343c37f82d36ae02be6bb30eb03` — *Add zero-arity funclike `#define`*

The first function-like-macro commit handles the simplest case: `#define X() 1` — a zero-parameter function-like macro, invoked as `X()`. The interesting wrinkle is *distinguishing* a function-like definition from an object-like one with a parenthesized body: `#define X (1)` is object-like (X expands to `(1)`), while `#define X() 1` is function-like (X expands to `1` only when invoked as `X()`). The disambiguator is *whitespace*: a `(` *immediately after* the macro name (no space between) introduces parameters; a `(` *with* whitespace before it is the start of the body.

The tokenizer gains a `has_space` flag on `Token`, mirroring `at_bol`:

```c
static bool has_space;
// ...
tok->has_space = has_space;
at_bol = has_space = false;
```

— set to true when the tokenizer skips whitespace, comments, or block comments, cleared when a token is emitted.

`Macro` gains `is_objlike`:

```c
struct Macro {
  Macro *next;
  char *name;
  bool is_objlike;
  Token *body;
  bool deleted;
};
```

`read_macro_definition` is the new shared definition-reader. It branches on `!tok->has_space && equal(tok, "(")`:

```c
static void read_macro_definition(Token **rest, Token *tok) {
  if (tok->kind != TK_IDENT)
    error_tok(tok, "macro name must be an identifier");
  char *name = strndup(tok->loc, tok->len);
  tok = tok->next;

  if (!tok->has_space && equal(tok, "(")) {
    // Function-like macro
    tok = skip(tok->next, ")");
    add_macro(name, false, copy_line(rest, tok));
  } else {
    // Object-like macro
    add_macro(name, true, copy_line(rest, tok));
  }
}
```

`expand_macro` branches on `is_objlike`:

```c
// Object-like macro application
if (m->is_objlike) {
  Hideset *hs = hideset_union(tok->hideset, new_hideset(m->name));
  Token *body = add_hideset(m->body, hs);
  *rest = append(body, tok->next);
  return true;
}

// If a funclike macro token is not followed by an argument list,
// treat it as a normal identifier.
if (!equal(tok->next, "("))
  return false;

// Function-like macro application
tok = skip(tok->next->next, ")");
*rest = append(m->body, tok);
return true;
```

The function-like branch returns `false` if the next token isn't `(` — that's the case where `M7` is defined as `#define M7() 1` but used as just `M7` (no parens). Such a use is a *non-invocation*; the identifier passes through as itself. The test:

```c
#define M7() 1
int M7 = 5;
ASSERT(1, M7());
ASSERT(5, M7);
```

— an identifier `M7` is accepted both as a function-like macro invocation (with parens) and as the variable reference (without). The same name lives in both namespaces. (This is also why pure macros redefining keywords work: `#define if 5` followed by `if (0);` after `#undef if` works because the macro is gone by the time the code is parsed.)

The zero-arity case skips the body's argument processing entirely (`m->body` is appended directly to `tok` after the `)`, with no substitution). That's why §17.4.2 (multi-arity) is the substantial commit, not this one.

The print-tokens function in `main.c` updates to honor `has_space`:

```diff
-    fprintf(out, " %.*s", tok->len, tok->loc);
+    if (tok->has_space && !tok->at_bol)
+      fprintf(out, " ");
+    fprintf(out, "%.*s", tok->len, tok->loc);
```

— the `-E` output now reproduces *only* the spaces that were in the source (or that were preserved through expansion).

### 17.4.2 — Multi-arity function-like `#define`

> `git checkout b9ad3e43cf7479712972514aa3f2c55a0f650f76` — *Add multi-arity funclike `#define`*

The biggest single commit in §17.4. It introduces parameter parsing, argument parsing, and substitution.

`MacroParam` is a parameter list:

```c
typedef struct MacroParam MacroParam;
struct MacroParam {
  MacroParam *next;
  char *name;
};
```

`MacroArg` is a parsed argument:

```c
typedef struct MacroArg MacroArg;
struct MacroArg {
  MacroArg *next;
  char *name;
  Token *tok;
};
```

Note the *parallel structure*: arguments carry parameter names (the `name` field), letting `find_arg(args, tok)` look up an argument by parameter name during substitution.

`Macro` gains `MacroParam *params`. `read_macro_params` parses the comma-separated parameter list:

```c
static MacroParam *read_macro_params(Token **rest, Token *tok) {
  MacroParam head = {};
  MacroParam *cur = &head;

  while (!equal(tok, ")")) {
    if (cur != &head)
      tok = skip(tok, ",");

    if (tok->kind != TK_IDENT)
      error_tok(tok, "expected an identifier");
    MacroParam *m = calloc(1, sizeof(MacroParam));
    m->name = strndup(tok->loc, tok->len);
    cur = cur->next = m;
    tok = tok->next;
  }
  *rest = tok->next;
  return head.next;
}
```

`read_macro_arg_one` reads a single argument's tokens up to a `,` or `)`:

```c
static MacroArg *read_macro_arg_one(Token **rest, Token *tok) {
  Token head = {};
  Token *cur = &head;

  while (!equal(tok, ",") && !equal(tok, ")")) {
    if (tok->kind == TK_EOF)
      error_tok(tok, "premature end of input");
    cur = cur->next = copy_token(tok);
    tok = tok->next;
  }

  cur->next = new_eof(tok);

  MacroArg *arg = calloc(1, sizeof(MacroArg));
  arg->tok = head.next;
  *rest = tok;
  return arg;
}
```

— the argument's tokens are copied into a fresh list terminated by EOF, so `subst` can iterate over it as a small token stream. (Argument tokens are *not yet* expanded; expansion happens lazily inside `subst`.)

`read_macro_args` zips parameters and arguments:

```c
static MacroArg *read_macro_args(Token **rest, Token *tok, MacroParam *params) {
  Token *start = tok;
  tok = tok->next->next;  // skip past macro name and '('

  MacroArg head = {};
  MacroArg *cur = &head;

  MacroParam *pp = params;
  for (; pp; pp = pp->next) {
    if (cur != &head)
      tok = skip(tok, ",");
    cur = cur->next = read_macro_arg_one(&tok, tok);
    cur->name = pp->name;
  }

  if (pp)
    error_tok(start, "too many arguments");
  *rest = skip(tok, ")");
  return head.next;
}
```

The loop walks `params` in lockstep with the comma-separated argument tokens. Each argument inherits its parameter's name. Too few or too many arguments is an error.

`subst` is the substitution pass:

```c
static Token *subst(Token *tok, MacroArg *args) {
  Token head = {};
  Token *cur = &head;

  while (tok->kind != TK_EOF) {
    MacroArg *arg = find_arg(args, tok);

    // Macro arguments are completely macro-expanded
    // before they are substituted into a macro body.
    if (arg) {
      Token *t = preprocess2(arg->tok);
      for (; t->kind != TK_EOF; t = t->next)
        cur = cur->next = copy_token(t);
      tok = tok->next;
      continue;
    }

    // Handle a non-macro token.
    cur = cur->next = copy_token(tok);
    tok = tok->next;
    continue;
  }

  cur->next = tok;
  return head.next;
}
```

Read carefully. Each token of the body is either a parameter reference (in which case its argument's tokens are pre-expanded via `preprocess2` and spliced in) or a literal token (in which case it's copied through). The pre-expansion of arguments is a key piece of the C standard's specified order: `f(g(x))` — where `f` and `g` are both macros — has `g(x)` fully expanded before being substituted into `f`'s body. This matters for the no-rescan rule: arguments are expanded *with their own hidesets*, separately from the function-like macro's expansion paint.

`expand_macro`'s function-like branch becomes:

```c
// Function-like macro application
MacroArg *args = read_macro_args(&tok, tok, m->params);
*rest = append(subst(m->body, args), tok);
return true;
```

The hideset machinery isn't fully integrated yet — that's §17.4.5. This commit's expansion *can* infinite-loop on a self-referential function-like macro. It happens to not, in the tests, because the tests don't construct one.

The two-line tests cover redefinition and basic substitution:

```c
#define M8(x,y) x+y
ASSERT(7, M8(3, 4));

#define M8(x,y) x*y
ASSERT(24, M8(3+4, 4+5));     // (3+4) * (4+5) but parsed as 3+4*4+5 = 24

#define M8(x,y) (x)*(y)
ASSERT(63, M8(3+4, 4+5));     // (3+4)*(4+5) = 63
```

— the second case demonstrates the classic argument-grouping pitfall: `M8(3+4, 4+5)` with `x*y` expands to `3+4*4+5 = 3 + 16 + 5 = 24`, not `63`. The `(x)*(y)` definition fixes it.

### 17.4.3 — Allow empty macro arguments

> `git checkout dd4306cdd8158f76f094fc699530311228536adb` — *Allow empty macro arguments*

Three test lines, no preprocess.c change. The previous commit's `read_macro_arg_one` already handles empty arguments (its `while` loop terminates at `,` or `)`, which produces an empty token list if `,` is the very first token). The test:

```c
#define M8(x,y) x y
ASSERT(9, M8(, 4+5));       // empty x, y = 4+5; result is " 4+5" → 4+5 = 9
```

— `M8(, 4+5)`'s first argument is empty; the substitution `x y` becomes `(empty) (4+5)` = ` 4+5` = `4+5`. Three lines, but worth its own commit because it's a behavior assertion.

### 17.4.4 — Allow parenthesized expressions as macro arguments

> `git checkout c7d7ce0f0cbd5869259a3365211ab92126a27ff6` — *Allow parenthesized expressions as macro arguments*

`read_macro_arg_one` learns to balance parentheses. Pre-this-commit, `M8((2+3), 4)` would tokenize the first argument as `(2+3` (stopping at the comma inside, even though it's nested). Post-this-commit, it tracks a `level` counter:

```c
static MacroArg *read_macro_arg_one(Token **rest, Token *tok) {
  Token head = {};
  Token *cur = &head;
  int level = 0;

  while (level > 0 || (!equal(tok, ",") && !equal(tok, ")"))) {
    if (tok->kind == TK_EOF)
      error_tok(tok, "premature end of input");

    if (equal(tok, "("))
      level++;
    else if (equal(tok, ")"))
      level--;

    cur = cur->next = copy_token(tok);
    tok = tok->next;
  }
  // ...
}
```

— `level > 0` keeps the loop going past commas and right-parens that are nested inside an opened paren. `M8((2,3), 4)` now lexes as `(2,3)` and `4`. Two-token nested expressions like `M8((2+3), 4)` work too.

### 17.4.5 — The function-like no-rescan rule and hideset intersection

> `git checkout 1313fc6d3a77cedbca18fa0ffee1a86d0903ad7f` — *Do not expand a token more than once for the same funclike macro*

The function-like-macro's no-rescan rule is subtler than the object-like one. Object-like: take the source token's hideset, union with `{macroname}`, paint the result. Function-like: the *invocation* spans multiple tokens (the macro name, the open paren, the arguments, the close paren). Those tokens may have *different* hidesets — they came from different expansions. The standard says: the result's hideset is the **intersection** of the hidesets at the *first* token (the macro name) and at the *last* token (the closing paren), unioned with `{macroname}`.

The intersection helper:

```c
static Hideset *hideset_intersection(Hideset *hs1, Hideset *hs2) {
  Hideset head = {};
  Hideset *cur = &head;

  for (; hs1; hs1 = hs1->next)
    if (hideset_contains(hs2, hs1->name, strlen(hs1->name)))
      cur = cur->next = new_hideset(hs1->name);
  return head.next;
}
```

The expansion's function-like branch updates:

```c
// Function-like macro application
Token *macro_token = tok;
MacroArg *args = read_macro_args(&tok, tok, m->params);
Token *rparen = tok;

// Tokens that consist a func-like macro invocation may have different
// hidesets, and if that's the case, it's not clear what the hideset
// for the new tokens should be. We take the interesection of the
// macro token and the closing parenthesis and use it as a new hideset
// as explained in the Dave Prossor's algorithm.
Hideset *hs = hideset_intersection(macro_token->hideset, rparen->hideset);
hs = hideset_union(hs, new_hideset(m->name));

Token *body = subst(m->body, args);
body = add_hideset(body, hs);
*rest = append(body, tok->next);
return true;
```

`read_macro_args` also has its `*rest = skip(tok, ")")` changed to leave `tok` at the close-paren rather than advancing past it (so that `rparen = tok` captures the close-paren's hideset before we consume it):

```diff
-  *rest = skip(tok, ")");
+  skip(tok, ")");
+  *rest = tok;
```

— `skip` here is purely an assertion (it errors out if the token isn't `)`); the *advance* happens at the bottom of `expand_macro` via `tok->next`.

The walked example for the test case:

```c
#define dbl(x) M10(x) * x
#define M10(x) dbl(x) + 3
ASSERT(10, dbl(2));
```

Source: `dbl(2)` — both tokens have empty hideset.

1. `expand_macro` on `dbl`. Hideset of `dbl` is `{}`. Hideset of `)` is `{}`. Intersection: `{}`. Union with `{dbl}` = `{dbl}`.
2. Substitute `x = 2` in `M10(x) * x`: result is `M10(2) * 2`. Paint `{dbl}` on every token: `M10₍dbl₎(₍dbl₎2₍dbl₎)₍dbl₎ *₍dbl₎ 2₍dbl₎`.
3. Splice. Loop reads `M10` (hideset `{dbl}`).
4. `M10`'s hideset doesn't contain `M10`. Look up. `(` after has hideset `{dbl}`. `)` after `2` has hideset `{dbl}`. Intersection of `{dbl}` and `{dbl}` is `{dbl}`. Union with `{M10}` = `{dbl, M10}`.
5. Substitute `x = 2` in `dbl(x) + 3`: result is `dbl(2) + 3`. Paint `{dbl, M10}`: `dbl₍dbl,M10₎(2) + 3`.
6. Splice. Loop reads `dbl` (hideset `{dbl, M10}`).
7. `dbl`'s hideset *contains* `dbl`. Bail. `dbl` passes through as an ordinary identifier.
8. Loop reads `(`, `2`, `)`, `+`, `3`, `*`, `2`. All non-macro. All pass through.
9. Final: `dbl(2) + 3 * 2`.
10. The leftover `dbl` matches the source-level *function* `int dbl(int x) { return x*x; }` defined earlier in the test. So `dbl(2) = 4`. Result: `4 + 3*2 = 10`. Asserted.

Two macros chase each other recursively but the hideset terminates expansion after one round of each. The *function* `dbl` (lower-case identifier shadowing the macro at the call site after the macro is hidden) catches the residual call.

The intersection rule's *why*: when an invocation's macro name has hideset `H1` and its close-paren has hideset `H2`, the only macros that have *definitely* already painted across this whole invocation are those in `H1 ∩ H2`. A macro in `H1 \ H2` painted only the start; a macro in `H2 \ H1` painted only the end. The result's tokens haven't been protected against either — a future rescan should be able to expand them.

### 17.4.6 — Stringizing (`#`)

> `git checkout 8f6f7925a04ca070167a38b8952a1a0bb7b63d23` — *Add macro stringizing operator (`#`)*

`#x` (where `x` is a macro parameter) substitutes a *string literal* containing the textual form of the argument. `#define M11(x) #x; M11(a + b)` produces `"a + b"`.

The implementation has three pieces. `quote_string` produces the C-source-level form of a string with `\` and `"` escaped:

```c
static char *quote_string(char *str) { /* doubles every \ and " */ }
```

`new_str_token` runs the result through the tokenizer to produce a string-literal token:

```c
static Token *new_str_token(char *str, Token *tmpl) {
  char *buf = quote_string(str);
  return tokenize(new_file(tmpl->file->name, tmpl->file->file_no, buf));
}
```

This is the kind of move that's pragmatic but worth pausing on: rather than constructing a `TK_STR` token by hand (with all its `Type *ty`, `char *str`, etc.), Rui *runs the tokenizer on the synthesized text*. The text contains a quoted string; the tokenizer produces the right `TK_STR` token. The `tmpl` parameter is the source-position template for error reporting.

`tokenize` and `new_file` get exposed in `chibicc.h`:

```c
File *new_file(char *name, int file_no, char *contents);
Token *tokenize(File *file);
```

`join_tokens` concatenates the textual forms of a token list:

```c
static char *join_tokens(Token *tok) {
  // Compute the length, copy in token texts, separate by ' ' on has_space.
}
```

`stringize` ties the two together:

```c
static Token *stringize(Token *hash, Token *arg) {
  char *s = join_tokens(arg);
  return new_str_token(s, hash);
}
```

And the substitution pass gains a `#`-handling branch:

```c
// "#" followed by a parameter is replaced with stringized actuals.
if (equal(tok, "#")) {
  MacroArg *arg = find_arg(args, tok->next);
  if (!arg)
    error_tok(tok->next, "'#' is not followed by a macro parameter");
  cur = cur->next = stringize(tok, arg->tok);
  tok = tok->next->next;
  continue;
}
```

— note the `find_arg` is called on `tok->next` (the parameter name after the `#`); the result is the argument's token list, which `stringize` joins and quotes.

The test exercises preserved spaces:

```c
#define M11(x) #x
ASSERT('a', M11( a!b  `""c)[0]);  // string is "a!b  `\"\"c"
ASSERT('!', M11( a!b  `""c)[1]);
ASSERT(' ', M11( a!b  `""c)[3]);
ASSERT('`', M11( a!b  `""c)[4]);
ASSERT('"', M11( a!b  `""c)[5]);  // escaped \" round-trips
```

The argument `a!b  \`""c` (eight tokens, lots of whitespace) stringizes to `"a!b  \`\"\"c"` (the spaces are preserved because `has_space` was set on the inner tokens; the `\"` are escaped because `quote_string` doubles them). The C-source-level result is a single string literal that, when read by the C string-literal tokenizer, produces the expected character sequence.

### 17.4.7 — Pasting (`##`)

> `git checkout 8f561aed9b7a47c38afd8c1cc75bc9a700ae97b5` — *Add macro token-pasting operator (`##`)*

The token-pasting operator: `a ## b` produces the *single* token `ab`. Unlike stringizing, it doesn't quote — it concatenates the source text and re-tokenizes:

```c
static Token *paste(Token *lhs, Token *rhs) {
  char *buf = format("%.*s%.*s", lhs->len, lhs->loc, rhs->len, rhs->loc);

  Token *tok = tokenize(new_file(lhs->file->name, lhs->file->file_no, buf));
  if (tok->next->kind != TK_EOF)
    error_tok(lhs, "pasting forms '%s', an invalid token", buf);
  return tok;
}
```

The *invalid token* check: pasting must produce a *single* token. `1 ## 5` → `15` (one token). `1 ## +` → `1+` would tokenize as two tokens (`1` and `+`); error.

`##` becomes a punctuator in `tokenize.c`:

```diff
-    "||", "<<", ">>",
+    "||", "<<", ">>", "##",
```

— so the tokenizer emits `##` as one token rather than two `#`s.

The substitution pass gains two branches. First, `##` directly:

```c
if (equal(tok, "##")) {
  if (cur == &head)
    error_tok(tok, "'##' cannot appear at start of macro expansion");
  if (tok->next->kind == TK_EOF)
    error_tok(tok, "'##' cannot appear at end of macro expansion");

  MacroArg *arg = find_arg(args, tok->next);
  if (arg) {
    if (arg->tok->kind != TK_EOF) {
      *cur = *paste(cur, arg->tok);
      for (Token *t = arg->tok->next; t->kind != TK_EOF; t = t->next)
        cur = cur->next = copy_token(t);
    }
    tok = tok->next->next;
    continue;
  }

  *cur = *paste(cur, tok->next);
  tok = tok->next->next;
  continue;
}
```

— after `##`, the next token is either a parameter (in which case the argument's first token is pasted to the previous output token, and the rest of the argument's tokens copy through) or a literal (in which case it's pasted directly). The empty-argument case (`arg->tok->kind == TK_EOF`) is handled by skipping.

Second, `arg ## ...` — a parameter followed by `##`:

```c
MacroArg *arg = find_arg(args, tok);

if (arg && equal(tok->next, "##")) {
  Token *rhs = tok->next->next;

  if (arg->tok->kind == TK_EOF) {
    MacroArg *arg2 = find_arg(args, rhs);
    if (arg2) {
      for (Token *t = arg2->tok; t->kind != TK_EOF; t = t->next)
        cur = cur->next = copy_token(t);
    } else {
      cur = cur->next = copy_token(rhs);
    }
    tok = rhs->next;
    continue;
  }

  for (Token *t = arg->tok; t->kind != TK_EOF; t = t->next)
    cur = cur->next = copy_token(t);
  tok = tok->next;
  continue;
}
```

— if the parameter is followed by `##`, *don't* expand it (`paste` is non-expanding by design; the C standard says macros aren't expanded across `##`'s operands), copy the argument's tokens through, and let the next loop iteration handle the `##`. The empty-argument case (`paste5,` with the empty side) defers to the rhs handling.

The cleanest tests:

```c
#define paste(x,y) x##y
ASSERT(15, paste(1,5));       // 1##5 → 15
ASSERT(255, paste(0,xff));    // 0##xff → 0xff (a hex literal)
ASSERT(3, ({ int foobar=3; paste(foo,bar); }));  // foo##bar → foobar
ASSERT(5, paste(5,));         // 5## → 5 (empty rhs)
ASSERT(5, paste(,5));         // ##5 → 5 (empty lhs)
```

— and the most twisted:

```c
#define i 5
ASSERT(101, ({ int i3=100; paste(1+i,3); }));
#undef i
```

— `paste(1+i, 3)` should *not* expand `i` while pasting (the second argument's first token is `3`, but the first argument's last token is `i`, and we paste `i` to `3` to form `i3`, which is a valid identifier and references the local variable `i3 = 100`). The result is `1 + i3 = 1 + 100 = 101`. Pasting suppresses argument expansion on the side adjacent to `##`.

#### Where the section leaves us

Function-like macros work end-to-end: definitions with parameters, invocations with comma-separated arguments, balanced-paren argument scanning, empty arguments, the no-rescan rule via hideset intersection, stringizing `#x`, pasting `a##b`. The substitution pass in `subst` is the chapter's most code-heavy single function — about 80 lines after this section, with branches for parameter substitution, stringizing, and pasting. The expansion driver in `expand_macro` is shorter but more subtle, doing the hideset bookkeeping that makes Prosser's algorithm terminate.

The non-trivial file-touch for the section: the tokenizer gains `##` as a punctuator, `has_space` tracking, and (in §17.4.6) is exposed via `chibicc.h`. The header gets a `Hideset` forward declaration. Otherwise, all the code is in `preprocess.c`.

---

## 17.5 — Polish: the rest of the preprocessor

Eighteen commits. The longest stretch of the chapter, but the *easiest* to bundle, because each commit is small and topic-coherent. The eighteen group into four sub-bundles by topic:

- **Tests, `defined()`, identifier-to-zero, whitespace** (commits 179–182): switching the test pipeline off `cc -E`, adding `defined()`, the constexpr-identifier-to-zero rule, and whitespace preservation through expansion.
- **Line continuation, `<...>` and search paths** (commits 183–186): the trailing-`\`-newline handling, `#include <...>`, `-I<dir>`, default include paths.
- **`#error` and predefined macros** (commits 187–192): `#error`, the `__STDC__`/`__x86_64__`/`__linux__` block, `__FILE__` and `__LINE__` (with the `Token->origin` chain), `__VA_ARGS__`, `__func__`, `__FUNCTION__`.
- **Adjacent-string concat, wide chars, headers, `va_arg`** (commits 193–196): adjacent-string-literal concatenation, wide character literal `L'\0'`, the standard headers, `va_arg()` as a real macro.

Each sub-bundle is one or two paragraphs.

### 17.5.1 — Tests switch off `cc -E`; `defined()`; identifier-to-zero; whitespace

> Commits `769b5a0` (use chibicc's preprocessor for all tests), `5cb2f89` (`defined()`), `a8d76ad` (identifier-to-zero), `8075582` (preserve newline and space).

Commit 179 (`769b5a0`) is the chapter's first user-visible behavioral milestone: the test Makefile stops piping through `cc -E`. Pre-this-commit, the test pipeline was:

```makefile
test/%.exe: chibicc test/%.c
	$(CC) -o- -E -P -C test/$*.c | ./chibicc -c -o test/$*.o -
```

Post-this-commit:

```makefile
test/%.exe: chibicc test/%.c
	./chibicc -c -o test/$*.o test/$*.c
```

`test/macro.c` updates from `assert(...)` calls (which require its own forward declarations) to `ASSERT(...)` calls (a macro from `test/test.h`, included via the now-working `#include "test.h"`). This is the moment the chapter's progress *cuts over*. From here, every test runs through chibicc's own preprocessor. The host `cc` is still invoked by the test Makefile for the final link step (with `test/common`), but it's no longer the preprocessor.

Commit 180 (`5cb2f89`, `defined()`) gives `#if` access to "is this macro defined?" The implementation is a substitution pass over the constexpr's synthetic line, inserting `0` or `1` for each `defined(X)` or `defined X`:

```c
static Token *read_const_expr(Token **rest, Token *tok) {
  // walk tokens; replace defined(X) with new_num_token(m ? 1 : 0, start)
}
```

— and the constexpr evaluator runs `read_const_expr` first, *before* `preprocess2`'s recursive macro-expansion pass. The order matters: `defined(X)` must be evaluated *without* X being expanded, even if X is defined as something else. Substituting first produces a clean `0` or `1` that survives the subsequent macro pass.

`new_num_token(int val, Token *tmpl)` is the integer counterpart of `new_str_token` — synthesize a number's textual form, run it through the tokenizer:

```c
static Token *new_num_token(int val, Token *tmpl) {
  char *buf = format("%d\n", val);
  return tokenize(new_file(tmpl->file->name, tmpl->file->file_no, buf));
}
```

Commit 181 (`a8d76ad`, identifier-to-zero) addresses: what should `#if foo` mean when `foo` is undefined? The C standard answers (6.10.1p4): replace remaining identifiers with `0`. So `#if foo` becomes `#if 0` (false). Chibicc's evaluator walks the post-expansion synthetic line and rewrites lingering `TK_IDENT` tokens to `0`-numbers:

```c
for (Token *t = expr; t->kind != TK_EOF; t = t->next) {
  if (t->kind == TK_IDENT) {
    Token *next = t->next;
    *t = *new_num_token(0, t);
    t->next = next;
  }
}
```

— the in-place mutation preserves the list's chain.

Commit 182 (`8075582`, preserve newline and space) is twelve lines in `subst` and `expand_macro`. After substitution, the *first* token of the expanded body inherits the `at_bol` and `has_space` flags of the source token at the call site. Without this, the `-E` output would lose the blank line or space that preceded the macro reference, breaking visual alignment. The `Token->at_bol` and `has_space` propagate through expansion as if the call site's whitespace had been carried.

### 17.5.2 — Line continuation; `<...>`; `-I`; default include paths

> Commits `b33fe0e` (line continuation), `d85fc4f` (`#include <...>`), `a1dd621` (`-I<dir>`), `a939a7a` (default include paths).

Commit 183 (`b33fe0e`) handles `\<newline>` by deleting the pair before tokenization. The function `remove_backslash_newline` walks the file's contents and removes each occurrence, taking care to *preserve the line count* (the deleted newlines are appended at the end so `__LINE__` stays accurate):

```c
static void remove_backslash_newline(char *p) {
  int i = 0, j = 0, n = 0;

  while (p[i]) {
    if (p[i] == '\\' && p[i + 1] == '\n') { i += 2; n++; }
    else if (p[i] == '\n') {
      p[j++] = p[i++];
      for (; n > 0; n--) p[j++] = '\n';
    }
    else { p[j++] = p[i++]; }
  }
  for (; n > 0; n--) p[j++] = '\n';
  p[j] = '\0';
}
```

— the `n` counter tracks deleted newlines and re-inserts them at the next physical newline (or end-of-file), so the column offsets are wrong after a continuation but the line counts match. This is important for `#define` bodies that span continuation lines and for `__LINE__` accuracy in continuation regions.

Commit 184 (`d85fc4f`, `#include <...>`) adds the angle-bracket form. The argument lexes as a sequence of tokens between `<` and `>` because the tokenizer never special-cased `<...>`. The `read_include_filename` function reconstructs the filename text from the token sequence:

```c
if (equal(tok, "<")) {
  Token *start = tok;
  for (; !equal(tok, ">"); tok = tok->next)
    if (tok->at_bol || tok->kind == TK_EOF)
      error_tok(tok, "expected '>'");
  *is_dquote = false;
  *rest = skip_line(tok->next);
  return join_tokens(start->next, tok);
}
```

— `join_tokens` (the same helper used for stringizing) concatenates the tokens between the brackets, preserving spaces. The test cases `#include <foo.h>` and `#include <some/path/foo.h>` both work because `<` and `/` and `.` all tokenize as their own tokens that get joined back.

The `is_dquote` flag distinguishes the two forms: the `"..."` form first searches the *including file's directory* (Chapter 17.1.3); the `<...>` form does not. Both forms then fall through to the `-I`-path search and the default-include-path search.

Commit 185 (`a1dd621`, `-I<dir>`) adds the `StringArray include_paths`. Each `-I<dir>` argument pushes a path. `search_include_paths` iterates:

```c
static char *search_include_paths(char *filename) {
  if (filename[0] == '/') return filename;
  for (int i = 0; i < include_paths.len; i++) {
    char *path = format("%s/%s", include_paths.data[i], filename);
    if (file_exists(path)) return path;
  }
  return NULL;
}
```

— first match wins. The `StringArray` type from Chapter 16's driver work gets its first §17 reuse here.

Commit 186 (`a939a7a`, default include paths) adds three system paths and one chibicc-specific path:

```c
static void add_default_include_paths(char *argv0) {
  strarray_push(&include_paths, format("%s/include", dirname(strdup(argv0))));
  strarray_push(&include_paths, "/usr/local/include");
  strarray_push(&include_paths, "/usr/include/x86_64-linux-gnu");
  strarray_push(&include_paths, "/usr/include");
}
```

— called from `cc1()` at startup. The `${argv0}/include` path is for chibicc's own header bundle (added in §17.5.4 — `stdarg.h`, `stdbool.h`, etc.). The three system paths are Linux/glibc-specific (the same kind of brittleness Chapter 16's `find_libpath` showed). On a non-Linux system, the paths are wrong, and `#include <stdio.h>` would only succeed if the user supplied a working `-I`. Errata candidate, lower priority — same category as Chapter 16's link-path probing.

### 17.5.3 — `#error`, predefined macros, `__FILE__`/`__LINE__`, `__VA_ARGS__`, `__func__`

> Commits `e7fdc2e` (`#error`), `5f5a850` (predefined macros), `6f17071` (`__FILE__`/`__LINE__`), `dc01f94` (`__VA_ARGS__`), `ba6b4b6` (`__func__`), `82ba010` (`__FUNCTION__`).

Commit 187 (`e7fdc2e`, `#error`) is *three lines*:

```c
if (equal(tok, "error"))
  error_tok(tok, "error");
```

— `#error` aborts the preprocessor with an error message at the directive's location. The directive line's *content* (the diagnostic message the user wrote) isn't reported; chibicc just says "error". Errata candidate (real `#error` should print the rest of the directive line as the message).

Commit 188 (`5f5a850`, predefined macros) adds 41 macros via `define_macro`:

```c
static void define_macro(char *name, char *buf) {
  Token *tok = tokenize(new_file("<built-in>", 1, buf));
  add_macro(name, true, tok);
}

static void init_macros(void) {
  define_macro("_LP64", "1");
  define_macro("__C99_MACRO_WITH_VA_ARGS", "1");
  define_macro("__ELF__", "1");
  // ... 38 more ...
  define_macro("__x86_64__", "1");
  define_macro("linux", "1");
  define_macro("unix", "1");
}
```

— a Linux/x86-64-specific block that lets standard library headers detect chibicc's target. `__chibicc__` is also defined (`1`), letting consumer code detect the compiler. `__alignof__`, `__inline__`, etc., are alias macros — they expand to the proper C11 keyword (`_Alignof`, `inline`, etc.). `__STDC_VERSION__` is `201112L` (claiming C11 support, which chibicc imperfectly approximates).

Commit 189 (`6f17071`, `__FILE__` and `__LINE__`) is the chapter's most *invasive* change to the existing `Token` struct after the hideset:

```diff
 struct Token {
   // ...
   Hideset *hideset;
+  Token *origin;    // If this is expanded from a macro, the original token
 };
```

The `Token->origin` field forms a chain: an expanded token's `origin` points at its source token (the one in the macro invocation, before substitution); that source token's `origin` points at *its* origin if it was itself an expansion result; and so on. The chain ends at a token whose `origin` is `NULL` — the "real" source token, the one in the program's original `.c` file.

`__FILE__` and `__LINE__` walk this chain to find the *real* source location:

```c
static Token *file_macro(Token *tmpl) {
  while (tmpl->origin) tmpl = tmpl->origin;
  return new_str_token(tmpl->file->name, tmpl);
}

static Token *line_macro(Token *tmpl) {
  while (tmpl->origin) tmpl = tmpl->origin;
  return new_num_token(tmpl->line_no, tmpl);
}
```

— so `#define LINE() __LINE__` followed by `int x = LINE();` produces the line number where `LINE()` was *invoked*, not the line where `__LINE__` appears in the macro's body.

The `Macro` struct gains a `handler` field that lets a macro be evaluated by C code rather than by token substitution:

```c
typedef Token *macro_handler_fn(Token *);
struct Macro {
  // ...
  macro_handler_fn *handler;
};
```

`add_builtin("__FILE__", file_macro)` registers a builtin. `expand_macro` checks:

```c
// Built-in dynamic macro application such as __LINE__
if (m->handler) {
  *rest = m->handler(tok);
  (*rest)->next = tok->next;
  return true;
}
```

The expansion paths for object-like and function-like macros also gain `Token->origin` painting:

```c
for (Token *t = body; t->kind != TK_EOF; t = t->next)
  t->origin = tok;
```

— so every token in an expanded body knows its source token, and `__FILE__`/`__LINE__` can walk back through nested expansions to the original source.

Commit 190 (`dc01f94`, `__VA_ARGS__`) adds variadic function-like macros — `#define LOG(...) printf(__VA_ARGS__)`. The implementation:

- `Macro` gains `bool is_variadic`.
- `read_macro_params` recognizes `...` as the trailing parameter and sets `*is_variadic`.
- `read_macro_arg_one` gains a `read_rest` flag that disables comma-stops for the variadic argument.
- `read_macro_args` collects extra arguments (after fixed parameters) into a single `__VA_ARGS__` argument.

The variadic argument's `name` is hardcoded to `"__VA_ARGS__"`, which `find_arg` matches against in `subst`. So the body's `__VA_ARGS__` references look up the synthetic argument and substitute its (comma-separated) contents.

```c
#define M14(...) __VA_ARGS__
ASSERT(2, M14() 2);
ASSERT(5, M14(5));

#define M14(x, ...) add6(1,2,x,__VA_ARGS__,6)
ASSERT(21, M14(3,4,5));   // add6(1,2,3,4,5,6) = 21
```

Commits 191 and 192 (`ba6b4b6` and `82ba010`, `__func__` and `__FUNCTION__`) land in `parse.c`, not `preprocess.c`:

```c
push_scope("__func__")->var =
  new_string_literal(fn->name, array_of(ty_char, strlen(fn->name) + 1));

// [GNU] __FUNCTION__ is yet another name of __func__.
push_scope("__FUNCTION__")->var =
  new_string_literal(fn->name, array_of(ty_char, strlen(fn->name) + 1));
```

— at function-entry, a variable scope named `__func__` (and the GNU-extension alias `__FUNCTION__`) is pushed, with a string-literal value containing the function's name. References to `__func__` in the function body resolve to that string literal through the ordinary variable-name lookup. Note this is *not* a preprocessor macro; the C standard specifies `__func__` as a *predefined identifier*, not a macro, and chibicc implements it that way. (If it were a macro, it would expand at preprocessing time, before the parser knows what function it's in.)

### 17.5.4 — Adjacent-string concat; wide char; standard headers; `va_arg`

> Commits `ab4f1e1` (concatenate strings), `7746e4e` (wide char), `7cbfd11` (standard headers), `5322ea8` (`va_arg`).

Commit 193 (`ab4f1e1`, adjacent-string concatenation) handles `"foo" "bar"` → `"foobar"`. The post-expansion pass (called from the entry point of `preprocess`) walks the token list looking for runs of adjacent `TK_STR` tokens and merges them:

```c
static void join_adjacent_string_literals(Token *tok1) {
  while (tok1->kind != TK_EOF) {
    if (tok1->kind != TK_STR || tok1->next->kind != TK_STR) {
      tok1 = tok1->next; continue;
    }
    // ... compute total length, allocate buffer, copy contents,
    // mutate tok1 to be the joined token, splice past the consumed ones ...
  }
}
```

— the merged token's `ty` becomes a longer `array_of(ty->base, len)`, so `sizeof("foo" "bar") = 7` (six characters plus terminator). Called at the very end of `preprocess`:

```c
Token *preprocess(Token *tok) {
  init_macros();
  tok = preprocess2(tok);
  if (cond_incl) error_tok(cond_incl->tok, "unterminated conditional directive");
  convert_keywords(tok);
  join_adjacent_string_literals(tok);
  return tok;
}
```

— the order is: macro expansion, keyword conversion, string concatenation. The standard puts string concatenation in translation phase 6 (after preprocessing in phase 4 and lexical conversion in phase 5). Chibicc collapses phases but keeps the pipeline order.

Commit 194 (`7746e4e`, wide character literal) recognizes `L'a'` as a character literal, treats it identically to `'a'` for now. The commit message: "For now, L'' is equivalent to ''." A real wide-char implementation would produce a `wchar_t` (32-bit on Linux) value; chibicc punts. The test:

```c
ASSERT(4, sizeof(L'\0'));
ASSERT(97, L'a');
```

— `sizeof(L'\0')` is `4` only because `'a'` itself has type `int` in C, and `int` is 4 bytes. The `L` prefix doesn't change the type yet. Errata candidate; the proper fix arrives with full wide-character work in Chapter 19.

Commit 195 (`7cbfd11`) adds the `include/` directory with five small headers: `stdarg.h` (with `va_start`, `va_end`, the `__va_elem` and `va_list` typedefs, but *not* `va_arg` yet — that's the next commit), `stdbool.h` (defining `bool`/`true`/`false` to `_Bool`/`1`/`0`), `stddef.h` (`NULL`, `size_t`, `ptrdiff_t`), `stdalign.h` (`alignas`/`alignof` aliases for `_Alignas`/`_Alignof`), `float.h` (the `FLT_*`/`DBL_*` constants), and `stdnoreturn.h` (`noreturn` alias for `_Noreturn`). The headers are short — they're not full implementations, just enough to make `#include <stdarg.h>` work the way real C programs expect.

The Makefile gains `-Iinclude` for the test build. The chibicc-specific include path (the one rooted at `${argv0}/include`) finds these headers at runtime when the binary is invoked from its source directory.

Commit 196 (`5322ea8`, `va_arg()`) — the long-awaited finishing move on Chapter 14's variadic-function magic. Pre-this-commit, accessing variadic arguments required the magic-name `__va_area__` plus hand-rolled arithmetic (or `self.py`'s regex substitution). Post-this-commit, `<stdarg.h>` provides:

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

#define va_arg(ap, type)                        \
  ({                                            \
    int klass = __builtin_reg_class(type);      \
    *(type *)(klass == 0 ? __va_arg_gp(ap) :    \
              klass == 1 ? __va_arg_fp(ap) :    \
              __va_arg_mem(ap));                \
  })
```

— `__builtin_reg_class(type)` is a chibicc-specific intrinsic added in `parse.c`'s `primary` function:

```c
if (equal(tok, "__builtin_reg_class")) {
  tok = skip(tok->next, "(");
  Type *ty = typename(&tok, tok);
  *rest = skip(tok, ")");
  if (is_integer(ty) || ty->kind == TY_PTR) return new_num(0, start);
  if (is_flonum(ty)) return new_num(1, start);
  return new_num(2, start);
}
```

— it returns `0` for integer/pointer types (they go through `gp_offset`), `1` for floats/doubles (`fp_offset`), `2` for "memory" types (the `__va_arg_mem` path, which `1/0`s — divides-by-zero — because chibicc doesn't yet support struct-passing). The `va_arg(ap, type)` macro picks one of three offset-advancers at compile time based on the type-classifier intrinsic, advances the appropriate offset, and dereferences.

The result is that *user code* that uses `va_arg(ap, int)` no longer goes through `self.py`'s regex substitution — chibicc's preprocessor substitutes the macro's body, and the body's `__builtin_reg_class` is parsed by chibicc itself. The whole variadic surface is now real C, no special-cased Python in sight.

#### Where the section leaves us

Eighteen commits, four sub-bundles. The preprocessor's surface is now feature-complete for ordinary C code: include paths and search rules, line continuation, `#error`, predefined macros, `__FILE__`/`__LINE__`/`__VA_ARGS__`/`__func__`, adjacent-string concatenation, wide char. Standard headers exist as small bundles. `va_arg()` retires the last hand-coded magic-name access pattern.

The chapter's standing notes:

- **The `Token->origin` chain** is the chapter's quiet structural addition. Every expanded token knows its source. `__FILE__`/`__LINE__` walk the chain. After Chapter 17, `.loc` directives keep tracking the original source line, not the expanded line — which is what GDB needs to step through macro-using code.
- **The host-cc-as-preprocessor pipeline collapses at commit 179.** The test Makefile no longer pipes through `cc -E`; chibicc's preprocessor handles `#include "test.h"` directly.
- **Tests are still in C** (since Chapter 8 §8.2). After commit 179, they're tests in C-with-real-#include.
- **The `StringArray` type** has its first §17 user (`include_paths`).
- **The `unreachable()` macro** still has no readers in `preprocess.c` — the preprocessor's error paths use `error_tok` and `error` directly, never `unreachable()`.
- **Errata candidates added in §17.5:** `#error` doesn't print the directive's message text; `L''` is equivalent to `''`; `__va_arg_mem` divides by zero rather than failing cleanly; the `opt_S | opt_E` bitwise-`|` typo from §17.1.5; the default include paths are Linux/glibc-specific.

The §17.5 sub-bundles are *long* in commit count but *short* in interesting structural change — most of the work is feature-by-feature, with the `Token->origin` chain (commit 189) being the chapter's only deeply structural §17.5 addition. The chapter mapping called this section's commits "polish" with good reason.

---

## 17.6 — Self-host

> `git checkout 12a9e7506c092fcbab8852db85c3aebefc8a8c81` — *Self-host: including preprocessor, chibicc can compile itself*

One commit. The chapter's, and arguably the book's, punchline.

The Makefile change is small:

```diff
-stage2/%.o: chibicc self.py %.c
+stage2/%.o: chibicc %.c
 	mkdir -p stage2/test
-	./self.py chibicc.h $*.c > stage2/$*.c
-	./chibicc -c -o stage2/$*.o stage2/$*.c
+	./chibicc -c -o $(@D)/$*.o $*.c
```

— the stage-2 build no longer runs `self.py` to preprocess chibicc's source. It compiles chibicc's source directly, with chibicc, end-to-end. The Python regex preprocessor that has stood in for chibicc's missing C preprocessor since Chapter 16 §16.1 is now obsolete, because chibicc's *own* preprocessor handles every directive `self.py` was working around.

Two consequences fall out immediately. First, `self.py` itself is *deleted* — 127 lines of Python, gone. The script that prepended hand-typed `<stdio.h>`-equivalent declarations, stripped `\<newline>` continuations, deleted `#include` and `#define` lines, substituted `bool`/`true`/`false`/`NULL`/`MIN`/`va_start`/`unreachable`, and otherwise massaged chibicc's source into something the pre-Chapter-17 chibicc could parse — that whole apparatus is no longer needed because the post-Chapter-17 chibicc handles real C.

Second, the test Makefile target gains `-Iinclude`:

```diff
-	./chibicc -Itest -c -o test/$*.o test/$*.c
+	./chibicc -Iinclude -Itest -c -o test/$*.o test/$*.c
```

— pointing at the chibicc-shipped `include/` directory so test files that `#include <stdarg.h>` resolve to the bundled header. This is symmetric with what stage-2 already did; the regular test pipeline catches up.

The `make test-stage2` target now does what the Chapter 16 §16.1 prose forecast it would: chibicc compiles chibicc, the resulting `stage2/chibicc` runs the test suite, every test passes. *Stage 2 is bug-equivalent to stage 1.* Which means stage 2 *is* a correct C compiler — because it was produced by a process that, at every step, used a tool we trust (the host `cc` for stage 1, then stage 1 chibicc for stage 2). And because stage 2 produces the same outputs as stage 1 on the test suite, we can trust that future stage-3 builds (compiling chibicc's source with stage-2 chibicc) would also work.

That's the self-hosting milestone: the compiler is its own host. The chibicc README's stated destination, the JP-book pedagogy's stated destination, the book's preface — all converge here. Commit 197 is the moment.

A note on what self-hosting *isn't*. Chibicc compiling chibicc means chibicc is a *complete-enough* C compiler to handle one specific C program (its own source). It's not yet a *general* C compiler — there's plenty of standard C still missing (most prominently: variadic argument passing on the stack with more than 8 register-eligible args, struct passing by value, full bitfields, designated initializers, generic selection, etc.). Chapters 18 through 23 fill those in. But the *capability gap* between "a C compiler good enough for chibicc's source" and "a C compiler good enough for typical real-world C" is now an incremental matter: each remaining feature is a self-contained addition, not a structural rebuild. The chapter's progression has been: *fill in the gaps, not redesign anything.* That continues in Chapter 18 and beyond.

#### Where we are

Chibicc compiles itself. `self.py` is gone. The chapter has done its work.

---

## Where the chapter leaves us

Forty commits, six sections, the longest single arc in chibicc's history. The chapter built a C preprocessor from a do-nothing seam to a full preprocessor with all the directives, macros, builtins, and supporting infrastructure that real-world C code uses, and ended with chibicc compiling itself.

| Commit | Topic |
|---|---|
| `1e1ea39` | Do-nothing preprocessor (the seam). `convert_keywords` moves to post-`preprocess`. |
| `146c7b3` | Null directive. `at_bol` flag on `Token`. `is_hash`, `preprocess2`. |
| `d367510` | `#include "..."`. `File` struct. Per-file `.file`/`.loc` numbering. |
| `ec149f6` | Skip extra tokens after `#include`. `warn_tok`, `skip_line`. |
| `d138864` | `-E` flag. `print_tokens`. Absolute-path `#include`. |
| `bf6ff92` | `#if`/`#endif`. `const_expr` exposed. `CondIncl` stack. `eval_const_expr`. |
| `aa570f3` | Skip nested `#if` in skipped clause. Recursive `skip_cond_incl`. |
| `c6e81d2` | `#else`. `IN_THEN`/`IN_ELSE` ctx. Two-tier skipper. |
| `e7a1857` | `#elif`. `IN_ELIF` ctx. `included` flag governs first-match-wins. |
| `97d33ad` | Object-like `#define`. `Macro` struct. `expand_macro`. |
| `9ad60e4` | `#undef`. `deleted` flag on `Macro`. |
| `2651448` | Expand macros in `#if`/`#elif` arguments. Recursive `preprocess2` call. |
| `acce002` | No-rescan via hideset. `Hideset` struct. Prosser's algorithm cited. |
| `1f80f58` | `#ifdef` and `#ifndef`. Skipper recognizes them as nesting-starters. |
| `dec3b3f` | Zero-arity function-like `#define`. `is_objlike` flag. `has_space` on `Token`. |
| `b9ad3e4` | Multi-arity function-like `#define`. `MacroParam`/`MacroArg`. `subst`. |
| `dd4306c` | Empty arguments. (Test-only.) |
| `c7d7ce0` | Parenthesized arguments. `level` counter in `read_macro_arg_one`. |
| `1313fc6` | Function-like no-rescan. `hideset_intersection` at the closing paren. |
| `8f6f792` | Stringizing `#x`. `quote_string`, `new_str_token`, `stringize`. |
| `8f561ae` | Pasting `a##b`. `paste`. `##` becomes a punctuator. |
| `769b5a0` | Tests use chibicc's preprocessor. `cc -E` retires from the test pipeline. |
| `5cb2f89` | `defined()`. `read_const_expr`. |
| `a8d76ad` | Identifier-to-zero in constexpr. C standard 6.10.1p4. |
| `8075582` | Whitespace and newline preservation through expansion. |
| `b33fe0e` | Line continuation. `remove_backslash_newline` in tokenizer. |
| `d85fc4f` | `#include <...>`. `is_dquote` flag. `join_tokens` reuses stringizer. |
| `a1dd621` | `-I<dir>`. `include_paths` `StringArray`. |
| `a939a7a` | Default include paths. `add_default_include_paths`. |
| `e7fdc2e` | `#error`. Three lines. (Errata: doesn't print the message.) |
| `5f5a850` | Predefined macros. 41 entries: `__STDC__`, `__x86_64__`, `__linux__`, etc. |
| `6f17071` | `__FILE__`/`__LINE__`. `Token->origin` chain. `macro_handler_fn`. |
| `dc01f94` | `__VA_ARGS__`. `is_variadic`. Variadic argument `name = "__VA_ARGS__"`. |
| `ba6b4b6` | `__func__`. Implemented in `parse.c`, not `preprocess.c`. |
| `82ba010` | `__FUNCTION__` (GNU alias for `__func__`). |
| `ab4f1e1` | Adjacent-string concatenation. `join_adjacent_string_literals`. |
| `7746e4e` | Wide character literal. `L''` ≡ `''`. (Errata candidate.) |
| `7cbfd11` | `stdarg.h`/`stdbool.h`/`stddef.h`/`stdalign.h`/`float.h`/`stdnoreturn.h`. |
| `5322ea8` | `va_arg()` as a real macro. `__builtin_reg_class`. Magic name retired user-side. |
| `12a9e75` | **Self-host.** `self.py` deleted. `chibicc compiles chibicc`. |

The structural moves carried forward.

The first is the *hideset-on-Token* mechanism. Chibicc's preprocessor implements Prosser's algorithm directly: a per-token set of macro names that have already been expanded "into" this token, gated on at expansion time, painted on at expansion time, intersected at function-like-macro close-parens to handle invocations whose start and end tokens have different paint. The algorithm terminates on `#define X X`, `#define f() f()`, mutually-recursive macros, and any other shape the C language permits, because no token is ever expanded by the same macro twice. The data structure is a small linked list; the operations are union, intersection, contains. Forty lines of `preprocess.c` implement the whole machinery.

The second is the *token-list-rewriting model*. The preprocessor takes a `Token *` and returns a `Token *`. It never reads from text after the initial `tokenize_file`, never writes to text except for `-E`. Directives splice (`#include`), delete regions (`#if 0`), or define rewrite rules (`#define`). Macros are rewrite rules whose application is gated by hidesets. The model is *uniform* — there's no special-case code path that bypasses the rewriter. It's the same shape modern preprocessors use; chibicc's implementation is small enough to fit in one `preprocess.c` of about 900 lines.

The third is the *Token->origin chain*. Every expanded token knows its source. The chain is what makes `__FILE__` and `__LINE__` work *inside* macros (they walk the chain to find the original source location). It's also what makes error reporting accurate when an error happens in expanded code: `error_tok(t, ...)` reports `t->file->name`, but if `t` is an expansion result, the chain can be walked to find where the *user* wrote the offending construct, not where the macro body wrote it. Chibicc's error messages don't fully exploit this yet — they generally point at expansion results — but the infrastructure is there for future use.

The fourth is *chibicc compiling chibicc*. The chapter's destination. By commit 197, the stage-2 Makefile target produces a chibicc binary that's been compiled by the stage-1 chibicc, with no `self.py` interposition, with chibicc's own preprocessor handling all `#include` and `#define` work in chibicc's source. The binary passes the test suite. Self-hosting is real.

The chapter's other notes worth tracking forward.

- **The cc1-vs-driver split** is unchanged. The preprocessor lives inside cc1, not as a third process. The `-E` flag short-circuits `cc1()` after `preprocess()`.
- **The eval-quartet duplication** is real but principled: AST-level constexpr (`eval`/`eval2`/`eval_rval`/`eval_double`) handles initializers, array sizes, case labels, bitfield widths; token-level constexpr (`eval_const_expr` → `const_expr` → `eval`) handles `#if`/`#elif`. They share `eval` itself but go through different parsers.
- **The `Initializer` tree** is unchanged.
- **The `Relocation` mechanism** is unchanged.
- **The `is_static` default in `new_gvar`** is unchanged.
- **The `is_definition` flag on `Obj`** is unchanged.
- **The `is_unsigned` flag on `Type`** is unchanged.
- **The `__va_area__` magic name** survives in chibicc's source, but the user-side hook (`va_start`, `va_arg`, `va_end`) is now expressed as ordinary `<stdarg.h>` macros. The magic-name access pattern is still emitted by `va_start`, but `va_arg(ap, type)` no longer needs hand-rolling.
- **The register-save-area layout** is unchanged.
- **The argreg integer/FP split** is unchanged.
- **Canonicalization-at-parse-time count is unchanged at eight.** The preprocessor runs *before* parse, so its work is upstream of canonicalization.
- **Pre-factor-before-feature count ticks to eight.** §17.1's do-nothing preprocessor is the seam-first move; the next 39 commits fill it in. (Chapter 16 was at seven; this chapter's first commit makes eight.)
- **The fourth namespace (labels)** is unchanged.
- **The `is_typename` predicate** is unchanged.
- **The VarAttr channel** is unchanged.
- **The `unreachable()` macro** has no readers in `preprocess.c` after Chapter 17 — the preprocessor's error paths use `error_tok` and `error` directly.
- **Per-token line numbers** (Chapter 8 §8.3) survive macro expansion via the `Token->origin` chain. `.loc` directives reference the original source line. GDB-debuggable output (Chapter 8 §8.4) keeps working through macro-using code.
- **Tests in C** (Chapter 8 §8.2) are now C with real `#include` (commit 179). The driver tests are still shell.
- **The `Obj->tok` field** still has no readers.
- **The `Type->name_pos` field** is unchanged.
- **The `>>` codegen quirk** is unchanged.
- **psABI conformance thread is unchanged at nine.** The preprocessor doesn't touch ABI.
- **Errata catalog grows by five small items in Chapter 17:** `#error` doesn't print the directive's message text (commit 187); `L''` is equivalent to `''` (commit 194); `__va_arg_mem` divides by zero rather than aborting cleanly (commit 196); the `opt_S | opt_E` bitwise-`|` typo in `main.c` (commit 162); the default include paths are Linux/glibc-specific (commit 186).

Forward references for Chapter 18 (the full ABI, commits 198–220):

- **More than 6 integer args, more than 8 FP args**: Chapter 5 §5.4 and Chapter 15 §15.6 introduced silent miscompilation when a function has more args than registers can pass. Chapter 18 fills in stack-passed args and parameters, closing the silent-miscompile errata items.
- **Struct-by-value**: Chapter 12 introduced struct types and member access, but passing a struct as a function argument or returning one as a value still doesn't work. Chapter 18 lands the SysV ABI's struct-passing rules — the eightbyte classification, the `MEMORY`/`INTEGER`/`SSE` class assignments, the register-and-stack hybrid passing.
- **Variadic register-spilling**: Chapter 14 §14.1 set up the register-save area for variadic functions, but variadic functions whose fixed parameters spill onto the stack don't work yet. Chapter 18 fixes this with the `va_copy` work and full register/stack mixed handling.
- **Bitfields**: Chapter 12 introduced struct members but not bitfields. Chapter 18 lands `int x:3;` syntax, the storage-allocation rules, the access codegen, and the related restrictions.
- **Pp-numbers**: Chapter 18 retokenizes preprocessor numbers (the C-spec category that lets `0x1.5p-3` lex as a single token before number parsing) — affecting how `#define X 0xff` and `#define X 1e10` work.
- **`-D`/`-U`**: Command-line macro definition, finally.

Chapter 18 is somewhere between Chapter 16's eight commits and Chapter 17's forty — twenty-three commits, the chapter mapping forecasts. Chapter 17's sweeping preprocessor work is behind us; Chapter 18 returns to the ABI grind, filling in what Chapters 5, 12, 14, and 15 left silently broken at the edges. The book's path from here is steady ABI-conformance until Chapter 19's Unicode arc and Chapters 20–23's GCC-extensions and standard-library rounding-out.

For now, Chapter 17 closes with chibicc compiling chibicc. The book's stated destination has been reached.
