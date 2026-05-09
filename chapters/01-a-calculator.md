# Chapter 1 — A calculator

> Commits covered: `0522e2d`, `bf7081f`, `a1ab0ff`, `cc5a6d9`, `84cfcaf`, `bf9ab52`, `25b4b85`. Seven commits, all written on a single Saturday in August 2019.

Most compiler books begin by laying out the theory — finite automata, grammars, abstract syntax trees, intermediate representations — and only after a chapter or two of preliminaries do they let you write any code. Rui Ueyama's chibicc takes the opposite approach. Its very first commit is a working program that compiles the digit `42` to an executable that exits with status `42`. The compiler is fifteen lines of C. It cannot do anything else. It is, however, a *real* compiler: source code goes in, machine code comes out, the result runs.

This chapter walks through the seven commits Rui wrote on the morning and afternoon of August 3, 2019. By the end of those seven commits, chibicc accepts arithmetic expressions like `(3 + 5) * -2` and `1 < 42`, with all the precedence and associativity you'd expect from C. The compiler is still small — a single file, about 350 lines — but the architecture it ends with (tokenizer, recursive-descent parser, AST, code generator) is the same architecture chibicc has on its very last commit. Everything we do for the next 22 chapters is, in some sense, just adding features to this chapter's compiler.

If you're going to read along, now is the time to clone the repo:

```sh
git clone https://github.com/rui314/chibicc.git
cd chibicc
git checkout 0522e2d
```

We will work forward through commits one by one. At each section heading you'll see a `git checkout` command — run it, and the working tree will match what we're discussing.

---

## 1.1 — One integer in, one integer out

> `git checkout 0522e2d77e3ab82d3b80a5be8dbbdc8d4180561c` — *Compile an integer to an executable that exits with the given number*

The first commit creates four files: `.gitignore`, `Makefile`, `main.c`, and `test.sh`. The compiler itself is `main.c`:

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
  if (argc != 2) {
    fprintf(stderr, "%s: invalid number of arguments\n", argv[0]);
    return 1;
  }

  printf("  .globl main\n");
  printf("main:\n");
  printf("  mov $%d, %%rax\n", atoi(argv[1]));
  printf("  ret\n");
  return 0;
}
```

This is the entire compiler. It takes one command-line argument — a number, written in decimal — and prints four lines of x86-64 assembly to standard output. To turn that into a runnable program, the test driver pipes the output through GCC, which calls the system assembler and linker:

```sh
./chibicc 42 > tmp.s
gcc -static -o tmp tmp.s
./tmp
echo $?         # prints "42"
```

That last step deserves some attention. We did not write a `main` that calls `exit`. We wrote a `main` that puts the integer 42 into the `%rax` register and then issues a `ret` instruction. Why does that cause the process to exit with status 42?

### A first look at what we're emitting

C's `main` is an ordinary function as far as the machine is concerned. The C runtime — the small bit of code between the kernel handing control to your binary and your `main` being called — invokes `main` like any other function call, then takes its return value and passes it to the `exit` syscall. On x86-64 (specifically, on the System V AMD64 ABI used by Linux and macOS for user code), the return value of a function lives in the `%rax` register. So our four-line assembly program:

| Line | Meaning |
|---|---|
| `.globl main` | Make the symbol `main` visible to the linker, so the C runtime can find it. |
| `main:` | The label `main` — execution begins here when the runtime calls our function. |
| `mov $42, %rax` | Put the immediate value 42 into `%rax`. The `$` is AT&T-syntax for "immediate", the `%` prefix marks a register. |
| `ret` | Return from the function. The CPU pops a return address off the stack and jumps to it. |

We never wrote a prologue (`push %rbp; mov %rsp, %rbp`) or epilogue (`pop %rbp`) because this function doesn't use any stack space — it doesn't need a frame pointer. We never wrote `xor %rax, %rax` because we don't want zero in `%rax`, we want 42. The four lines we generated are the smallest possible function that returns a specific integer on x86-64.

### A note on syntax: AT&T vs Intel

There are two common ways to write x86 assembly. Most published Intel manuals and Microsoft's documentation use *Intel syntax* (`mov rax, 42`). The GNU toolchain — `as`, `gcc`, `objdump` — uses *AT&T syntax* by default (`mov $42, %rax`). They differ in three superficial ways: source and destination are reversed, registers are prefixed with `%`, and immediates are prefixed with `$`. They differ in zero substantial ways: the same machine code comes out either way.

chibicc emits AT&T syntax because that's what GNU `as` (the assembler we'll be feeding) accepts without flags. We are stuck with this choice for the rest of the book.

### Why output text and not machine code?

A compiler that emits assembly text and lets a separate assembler turn it into machine code is doing more I/O than strictly necessary. A compiler could, in principle, write the encoded x86-64 bytes directly into an object file. Some do. But there are good reasons to take the longer route:

- The system assembler is already written, well-tested, and aware of all the addressing modes, prefixes, and encoding quirks of the instruction set. We don't have to reimplement any of that.
- Assembly text is human-readable. When the compiler is misbehaving, you can `cat` its output and see what went wrong.
- The text format is portable across small variations of the same architecture — we don't have to track which encoding bits change between, say, Sandy Bridge and Skylake.

Almost every C compiler outside of LLVM uses this approach. GCC does. lcc does. tcc actually goes the *other* way — it writes machine code directly — but Rui notes in the README that he and Bellard made different design choices, and chibicc's "emit assembly, let `as` link it" model is the chibicc model.

### The build and test scripts

The `Makefile` is short:

```make
CFLAGS=-std=c11 -g -fno-common

chibicc: main.o
	$(CC) -o chibicc main.o $(LDFLAGS)

test: chibicc
	./test.sh

clean:
	rm -f chibicc *.o *~ tmp*

.PHONY: test clean
```

`-std=c11` because chibicc is written in modern C. `-g` for debug info. `-fno-common` is interesting: it tells GCC not to put uninitialized globals in COMMON symbols, so each `.o` has only one definition of any given variable name. It will save us a debugging headache much later when chibicc's own object files get linked together. We'll come back to it.

`test.sh` defines an `assert` helper that runs the compiler, assembles+links its output, runs the resulting program, and checks the exit code:

```sh
#!/bin/bash
assert() {
  expected="$1"
  input="$2"

  ./chibicc "$input" > tmp.s || exit
  gcc -static -o tmp tmp.s
  ./tmp
  actual="$?"

  if [ "$actual" = "$expected" ]; then
    echo "$input => $actual"
  else
    echo "$input => $expected expected, but got $actual"
    exit 1
  fi
}

assert 0 0
assert 42 42

echo OK
```

Two test cases. They both pass. `make test` is happy.

This is not a sophisticated test framework. There is no test runner, no setup or teardown, no fixtures, no parallelism. There's a function called `assert` that takes an expected value and an input string. Adding a test case takes one line. As we add features over the next 316 commits, this file will grow to thousands of lines — but it will never get fancier than this. Rui's README explicitly defends this: "Slow algorithms are fine if we know that n isn't too big." A shell script that invokes the compiler several thousand times is fast enough on a modern laptop. Nothing in chibicc's testing apparatus is more elaborate than it needs to be.

### Where we are

We have a compiler. It has one job — compile a single decimal integer — and it does that job. We have a test suite that runs end-to-end: parse, assemble, link, run, check exit code. The infrastructure is in place. From here on, every commit is going to be of the form *the compiler can do everything it could do before, plus one new thing*.

---

## 1.2 — Two operators in 22 lines

> `git checkout bf7081fba7d8c6b1cd8a12eb329697a5481c604e` — *Add `+` and `-` operators*

```diff
--- a/main.c
+++ b/main.c
@@ -7,9 +7,29 @@ int main(int argc, char **argv) {
     return 1;
   }
 
+  char *p = argv[1];
+
   printf("  .globl main\n");
   printf("main:\n");
-  printf("  mov $%d, %%rax\n", atoi(argv[1]));
+  printf("  mov $%ld, %%rax\n", strtol(p, &p, 10));
+
+  while (*p) {
+    if (*p == '+') {
+      p++;
+      printf("  add $%ld, %%rax\n", strtol(p, &p, 10));
+      continue;
+    }
+
+    if (*p == '-') {
+      p++;
+      printf("  sub $%ld, %%rax\n", strtol(p, &p, 10));
+      continue;
+    }
+
+    fprintf(stderr, "unexpected character: '%c'\n", *p);
+    return 1;
+  }
+
   printf("  ret\n");
   return 0;
 }
```

Everything important about this commit is in the swap of `atoi` for `strtol`. `strtol` returns the parsed long *and* writes a pointer past the last character it consumed into its second argument. So a single call moves both the value (into `%rax`'s initial mov) and the cursor (`p`) forward in lockstep.

After that initial number, we walk forward one character at a time. If we see `+`, we advance past it, parse the next number, and emit `add $N, %rax`. If we see `-`, we emit `sub`. If we see anything else, we error out.

The new test cases:

```sh
assert 21 '5+20-4'
```

Five plus twenty is twenty-five, minus four is twenty-one. The compiler emits:

```
  .globl main
main:
  mov $5, %rax
  add $20, %rax
  sub $4, %rax
  ret
```

Run that program and `%rax` ends up holding 21, which becomes the process's exit code.

### What this commit teaches by what it leaves out

There is no parser, no AST, no operator precedence, no tokenizer. There are also no whitespace tokens — `5+20-4` works, but `5 + 20 - 4` would crash on the first space (`p++` eventually reaches a space, the `if (*p == '+')` and `if (*p == '-')` branches both fail, and we fall into the error case). The commit message acknowledges this implicitly by *not* claiming to support spaces. Spaces will arrive in the very next commit.

The compiler also relies on the order it sees characters in. There's no lookahead, no backtracking. Each iteration of the `while` loop consumes a single operator and a single number, in that order, and emits one instruction per iteration. This works because we only have left-associative operators of equal precedence — `5+20-4` happens to mean `((5+20)-4)`, which is exactly what you get from emitting one `add`/`sub` after another. If we tried to add multiplication this way, we'd compute `5+20*4` as `(5+20)*4 = 100` instead of the correct `5+(20*4) = 85`. We can't.

This is not a bug. It's deliberately as simple as it can be while still being interesting. Rui is teaching by ratcheting: each commit takes the absolute smallest viable step. When we hit the limit of "absolute smallest", we'll add infrastructure — but only when we have to.

### Where we are

The compiler now handles `+` and `-`, but only on tightly-packed input with no whitespace, no parentheses, and no precedence above what naturally falls out of left-to-right evaluation. We've outgrown one-character-at-a-time parsing in our heads, even if the code hasn't yet. The next commit fixes that.

---

## 1.3 — A tokenizer

> `git checkout a1ab0ff26f23c82f15180051204eeb6279747c9a` — *Add a tokenizer to allow space characters between tokens*

This is the biggest commit so far — `main.c` more than doubles. It introduces three new ideas, all in one go: a token type, a tokenizer pass, and a separation between *parsing* (consuming the token stream) and *codegen* (emitting assembly). The codegen is still inline with the parsing, but the structural seed has been planted.

The token type:

```c
typedef enum {
  TK_PUNCT, // Punctuators
  TK_NUM,   // Numeric literals
  TK_EOF,   // End-of-file markers
} TokenKind;

// Token type
typedef struct Token Token;
struct Token {
  TokenKind kind; // Token kind
  Token *next;    // Next token
  int val;        // If kind is TK_NUM, its value
  char *loc;      // Token location
  int len;        // Token length
};
```

A token is a tagged record. It knows its kind (punctuator, number, end-of-file), its numeric value if applicable, and where in the input it came from (a pointer into the original source string, plus a length). Tokens form a singly linked list via `next`.

Two design choices in this struct deserve flagging because we'll see them again everywhere:

**A `Token` carries every field, even ones it doesn't use.** A `TK_NUM` token uses `val`. A `TK_PUNCT` token doesn't — but the `val` field is still there. Rui addresses this directly in the README: "We could save memory using unions, but I decided to simply put everything in the same struct instead. I believe the inefficiency is negligible." Trying to be clever about which fields belong to which kinds would make the code harder to read, and chibicc isn't trying to win benchmarks. This is going to be the pattern for the AST nodes too.

**`loc` is a pointer into the input source, not a copy.** When we want to report an error pointing at a specific token, we'll use this pointer to compute an offset into the input string. Since chibicc reads the entire source file into memory and never frees it (more on that in a moment), this pointer stays valid for the whole compilation.

### `calloc` and never `free`

The tokenizer uses `calloc` to allocate each token:

```c
static Token *new_token(TokenKind kind, char *start, char *end) {
  Token *tok = calloc(1, sizeof(Token));
  tok->kind = kind;
  tok->loc = start;
  tok->len = end - start;
  return tok;
}
```

Nothing here ever calls `free`. It will not, ever, in any commit. Rui's README:

> chibicc allocates memory using `calloc` but never calls `free`. Allocated heap memory is not freed until the process exits. I'm sure that this memory management policy (or lack thereof) looks very odd, but it makes sense for short-lived programs such as compilers.

This is not laziness — it's the same policy DMD uses, and the same policy gcc uses for many of its short-lived data structures. A compiler runs, builds an in-memory representation of the source program, emits output, and dies. The OS reclaims everything when the process exits. Adding `free` calls along the way would slow the compiler down, complicate ownership reasoning, and add bugs. They wouldn't fix anything, because nothing was leaking in any meaningful sense.

`calloc` zero-initializes, so every field starts at zero (or NULL). The unused fields in a `TK_PUNCT` token are zero rather than garbage; this doesn't matter for correctness but it makes debugging easier.

### The tokenizer itself

```c
static Token *tokenize(char *p) {
  Token head = {};
  Token *cur = &head;

  while (*p) {
    // Skip whitespace characters.
    if (isspace(*p)) {
      p++;
      continue;
    }

    // Numeric literal
    if (isdigit(*p)) {
      cur = cur->next = new_token(TK_NUM, p, p);
      char *q = p;
      cur->val = strtoul(p, &p, 10);
      cur->len = p - q;
      continue;
    }

    // Punctuator
    if (*p == '+' || *p == '-') {
      cur = cur->next = new_token(TK_PUNCT, p, p + 1);
      p++;
      continue;
    }

    error("invalid token");
  }

  cur = cur->next = new_token(TK_EOF, p, p);
  return head.next;
}
```

The classic linked-list-builder pattern, with two small touches worth pointing out.

`Token head = {};` declares a stack-allocated dummy. We never use it as a token — we just take its address as a sentinel `cur` so we don't need a special case for the very first node. At the end we return `head.next`, which is the first real token. This pattern shows up all over chibicc and saves an `if (head == NULL)` check in every list builder we'll write.

`cur = cur->next = new_token(...)` is a chained assignment. It both stores the new token in the previous node's `next` field and updates `cur` to point at the new node, in one line. C's right-to-left evaluation of `=` makes this work.

### Helper functions

Three helpers convert "what the parser knows about tokens" into "what the parser wants to ask":

```c
static bool equal(Token *tok, char *op) {
  return memcmp(tok->loc, op, tok->len) == 0 && op[tok->len] == '\0';
}
```

`equal(tok, "+")` returns true if the token's text is exactly `+`. The `memcmp` compares the token's `len` characters to the candidate string; the second clause makes sure the candidate is no longer than the token. Note that we compare against the token's location in the *original input* — we never copied the token text out, so we have to look at the source buffer.

```c
static Token *skip(Token *tok, char *s) {
  if (!equal(tok, s))
    error("expected '%s'", s);
  return tok->next;
}
```

`skip` is the "must consume this exact punctuator or die" version. It will be the parser's `assert`-like primitive.

```c
static int get_number(Token *tok) {
  if (tok->kind != TK_NUM)
    error("expected a number");
  return tok->val;
}
```

`get_number` insists on a numeric token and returns its value.

### `main`, rewritten

```c
int main(int argc, char **argv) {
  if (argc != 2)
    error("%s: invalid number of arguments", argv[0]);

  Token *tok = tokenize(argv[1]);

  printf("  .globl main\n");
  printf("main:\n");

  // The first token must be a number
  printf("  mov $%d, %%rax\n", get_number(tok));
  tok = tok->next;

  // ... followed by either `+ <number>` or `- <number>`.
  while (tok->kind != TK_EOF) {
    if (equal(tok, "+")) {
      printf("  add $%d, %%rax\n", get_number(tok->next));
      tok = tok->next->next;
      continue;
    }

    tok = skip(tok, "-");
    printf("  sub $%d, %%rax\n", get_number(tok));
    tok = tok->next;
  }

  printf("  ret\n");
  return 0;
}
```

Structurally this is identical to the previous commit's `main`. We mov the first number into `%rax`, then walk a sequence of (operator, number) pairs and emit `add` or `sub` for each. The only difference is what we walk over: a token list rather than a character buffer. Whitespace was stripped during tokenization, so we don't have to think about it here.

The `tok = skip(tok, "-")` pattern in the `else` branch is interesting: we don't bother checking explicitly for `-`, we just *demand* it. If the token isn't `+` and isn't `-`, `skip` will report `expected '-'` and exit. This is slightly misleading (the token might actually be valid in some other context that we just don't handle yet) but for now it's good enough — and chibicc's diagnostics will get more careful as the language grows.

### A single point of authority

There's one more change worth marking. The earlier `main` exited with status 1 on a parse error and printed to stderr inline. The new code uses a single `error(fmt, ...)` function that does the same thing in one place:

```c
static void error(char *fmt, ...) {
  va_list ap;
  va_start(ap, fmt);
  vfprintf(stderr, fmt, ap);
  fprintf(stderr, "\n");
  exit(1);
}
```

This is the kind of small consolidation that pays off enormously. By the end of the book, *every* error in chibicc — typos, type mismatches, undefined symbols, preprocessor failures — funnels through this same function (and a couple of variants we'll add in the next commit). Errors get printed and the compiler dies. There's no exception machinery, no error recovery, no "let's try to keep going to find more errors". When chibicc finds one problem, it stops. The simplicity of that policy is what makes the rest of the code so terse: no function ever has to think about what to return when its child failed, because by the time control gets back to a caller, the child has already exited the process.

### Where we are

We can now compile expressions like `12 + 34 - 5` with arbitrary whitespace. The compiler is in three pieces — tokenizer, ad-hoc parser, ad-hoc codegen — except the parser and codegen are still entangled in `main`. Untangling them is two commits away.

---

## 1.4 — Pointing at errors

> `git checkout cc5a6d978144bda90220bd10866c4fd908d07546` — *Improve error message*

The commit message is unusually long for chibicc — it includes the actual user-visible improvement:

```
$ ./chibicc 1+foo
1+foo
  ^ expected a number
```

The caret is the point of the commit. When a user makes a mistake, the compiler should not just say `expected a number` and leave the user wondering *where*. It should print the source line, then a caret pointing to the offending token. Every modern compiler does this. chibicc gets it on day one of having errors at all.

```diff
+// Input string
+static char *current_input;
+
 // Reports an error and exit.
 static void error(char *fmt, ...) {
   ...
 }
 
+// Reports an error location and exit.
+static void verror_at(char *loc, char *fmt, va_list ap) {
+  int pos = loc - current_input;
+  fprintf(stderr, "%s\n", current_input);
+  fprintf(stderr, "%*s", pos, ""); // print pos spaces.
+  fprintf(stderr, "^ ");
+  vfprintf(stderr, fmt, ap);
+  fprintf(stderr, "\n");
+  exit(1);
+}
+
+static void error_at(char *loc, char *fmt, ...) {
+  va_list ap;
+  va_start(ap, fmt);
+  verror_at(loc, fmt, ap);
+}
+
+static void error_tok(Token *tok, char *fmt, ...) {
+  va_list ap;
+  va_start(ap, fmt);
+  verror_at(tok->loc, fmt, ap);
+}
```

Three functions now exist where there was one:

- `error(...)` — the original. Prints a message and dies. No source location. Used for things that aren't tied to a place in the source, like "wrong number of arguments to chibicc".
- `error_at(loc, ...)` — takes a `char *` pointing into the input. Used by the tokenizer, which doesn't have a `Token` yet at the moment it discovers a bad character.
- `error_tok(tok, ...)` — takes a `Token *`. Used by everything downstream of the tokenizer.

Both location-aware variants share `verror_at`, which does the work: figure out the column by subtracting the location pointer from `current_input`, print the source line, print the right number of spaces followed by a caret, then print the message.

The `%*s` format specifier is the trick that makes the spacing concise. `printf("%*s", n, "")` prints an empty string padded to width `n`, which is just `n` spaces. It's a C idiom that's worth knowing.

A new file-static variable `current_input` holds the source string. `tokenize` is rewritten to read from it (instead of taking the source as a parameter), and `main` sets it before calling `tokenize`. This is also a pattern that's going to recur — when a piece of state is needed by half a dozen functions across the compilation, chibicc tends to make it a file-static rather than thread it through every function as a parameter. Code clarity beats encapsulation here. We'll see this again with the parser's current input position, with the codegen's output stream, and with the preprocessor's macro table.

### Why `error_at` and `error_tok` both exist

The tokenizer produces tokens. Until it produces them, there are no tokens to point at. So any error inside `tokenize` (e.g., "this character isn't a digit, a punctuator, or whitespace") needs to identify the bad spot using a raw `char *`, not a `Token *`. Hence `error_at`. Once we're past the tokenizer, errors come from the parser or codegen, which work on tokens — so `error_tok` is more convenient.

The two functions could be one. We could pass `tok->loc` to `error_at` directly. But two thin wrappers reads better at the call site than one wrapper that requires you to remember which field to pass.

### Where we are

The compiler now produces user-facing errors with a column-accurate caret. Functionally, nothing changed — the compiler accepts and rejects exactly the same inputs. But the error messages went from useless to good. This is a recurring theme in chibicc: invest in diagnostics early, before there's much to diagnose, because the same `verror_at` machinery will serve us until the very end.

---

## Interlude — Grammar, BNF, and recursive descent

Before the next commit we need a vocabulary. So far we've gotten away with parsing by hand because the language is essentially a list: a number, then a sequence of (operator, number) pairs. The grammar is flat. A flat grammar can be walked with a `while` loop and a switch statement — no recursion, no precedence, no nesting.

The next commit, which adds `*`, `/`, and `()`, breaks that flatness. With `*` and `/` we have *precedence* — `1 + 2 * 3` should compute the multiplication first. With `()` we have *recursion* — `(1 + 2) * 3` requires the parser to handle `1 + 2` while it's already in the middle of handling the outer expression.

The standard apparatus for thinking about this is *context-free grammars*, usually written in *Backus–Naur Form* (BNF) or one of its variants. The version we'll see in chibicc looks like this:

```
expr    = mul ("+" mul | "-" mul)*
mul     = primary ("*" primary | "/" primary)*
primary = "(" expr ")" | num
```

Each line is a *production rule*. The left side names a non-terminal; the right side describes how that non-terminal can be made. `|` means "or". `*` means "zero or more times". Quoted strings are literal terminal symbols (in our case, the characters of an operator or a parenthesis). `num` is shorthand for "any numeric token".

Read the rules out loud:

- An *expr* is a *mul*, optionally followed by any number of `+ mul` or `- mul` pairs.
- A *mul* is a *primary*, optionally followed by any number of `* primary` or `/ primary` pairs.
- A *primary* is either a parenthesized expression, or a number.

Two things are encoded in this grammar that aren't immediately obvious.

**Precedence.** `mul` is below `expr` in the call chain, so multiplication binds tighter than addition. To parse `1 + 2 * 3`, the parser enters `expr`, calls `mul`, which calls `primary` and gets `1`, then returns to `expr`. `expr` sees a `+`, so it consumes it and calls `mul` again. This time `mul` sees `2`, then a `*`, so it loops and consumes `* 3`, returning `2 * 3` to `expr`. The result is `1 + (2 * 3)`. Precedence falls out for free from the order of the call chain.

**Left associativity.** The `*` after the parentheses on each line means "repeat zero or more times", and the parser implements it with a loop that *folds left*: each new operator+operand pair becomes the right child of a new node whose left child is whatever we've built so far. So `1 - 2 - 3` parses as `(1 - 2) - 3`, not `1 - (2 - 3)` — which matches what C does.

A grammar of this shape — one where each non-terminal maps to one parsing function, and each function decides which alternative to take by looking at the current token — is parsed by *recursive descent*. The function `expr` will descend into `mul`, which will descend into `primary`, which will descend into `expr` if it sees a parenthesis. Recursion in the grammar becomes recursion in the C code. Each non-terminal becomes a function. Each `|` becomes an `if/else`. Each `*` repetition becomes a `for(;;)` loop with a `break` when the next token doesn't match.

There are alternative parsing strategies — table-driven LR parsers (yacc, bison), parser combinators, Pratt parsing — and they have real advantages. Recursive descent has one advantage that matters more than all of those for chibicc's purposes: the C code looks exactly like the grammar. Anyone who can read a BNF rule can read the corresponding parser function, and vice versa. The whole compiler is going to lean on this readability.

That's the theory. Let's see chibicc do it.

---

## 1.5 — A real parser, a real code generator

> `git checkout 84cfcaf98f3d19c8f0f316e22a61725ad201f0f6` — *Add `*`, `/` and `()`*

This is the largest commit in the chapter — about 180 lines added. It does three things at once: adds multiplication, division, and parentheses; introduces an abstract syntax tree; and splits the compiler into three labeled passes. The file gains three section comments:

```c
//
// Tokenizer
//
...
//
// Parser
//
...
//
// Code generator
//
```

These comments will survive almost unchanged for the life of chibicc. They're how Rui orients the reader on first read of the source — three top-to-bottom passes, each one cleanly doing its job.

### The AST node type

```c
typedef enum {
  ND_ADD, // +
  ND_SUB, // -
  ND_MUL, // *
  ND_DIV, // /
  ND_NUM, // Integer
} NodeKind;

// AST node type
typedef struct Node Node;
struct Node {
  NodeKind kind; // Node kind
  Node *lhs;     // Left-hand side
  Node *rhs;     // Right-hand side
  int val;       // Used if kind == ND_NUM
};
```

A `Node` is a tree node for an expression. It has a kind (one of five), left and right children, and an integer value. Three constructors create the three patterns of node we need:

```c
static Node *new_node(NodeKind kind) {
  Node *node = calloc(1, sizeof(Node));
  node->kind = kind;
  return node;
}

static Node *new_binary(NodeKind kind, Node *lhs, Node *rhs) {
  Node *node = new_node(kind);
  node->lhs = lhs;
  node->rhs = rhs;
  return node;
}

static Node *new_num(int val) {
  Node *node = new_node(ND_NUM);
  node->val = val;
  return node;
}
```

Same `calloc`-and-never-free policy as the tokens. Same "all fields live in the struct, even unused ones" choice as the tokens. A `ND_NUM` node has `val` set; its `lhs` and `rhs` are zero. A `ND_ADD` node has `lhs` and `rhs` set; its `val` is zero. We don't use a union for the discriminated parts, even though a textbook compiler often does. Rui's principle holds: this is small enough that the inefficiency doesn't matter, and reading two fewer tagged-union accesses per visit is worth the savings in mental overhead.

### The parser functions

```c
static Node *expr(Token **rest, Token *tok);
static Node *mul(Token **rest, Token *tok);
static Node *primary(Token **rest, Token *tok);
```

Three forward declarations. One per non-terminal. Each takes a `Token **rest` out-parameter and a `Token *tok` in-parameter. This is the most chibicc-specific piece of style in the parser, and once you see it the whole codebase becomes easier to read.

The convention: a parser function consumes some number of tokens starting from `tok`, builds a `Node` representing them, and writes the *next* unconsumed token into `*rest` before returning. The caller passes `&tok` for `rest` and the current token for `tok`, then uses the updated `tok` to continue. So a typical caller looks like:

```c
Node *node = mul(&tok, tok);
```

That looks weird at first. Why pass `&tok` and `tok` in the same call? Because they're playing different roles. The right-hand `tok` is the *input position* — where to start parsing. The left-hand `&tok` is the *output slot* — where to write the position to resume from. After the call, `tok` has been updated to point past whatever `mul` consumed. The function body confirms it:

```c
static Node *mul(Token **rest, Token *tok) {
  Node *node = primary(&tok, tok);

  for (;;) {
    if (equal(tok, "*")) {
      node = new_binary(ND_MUL, node, primary(&tok, tok->next));
      continue;
    }

    if (equal(tok, "/")) {
      node = new_binary(ND_DIV, node, primary(&tok, tok->next));
      continue;
    }

    *rest = tok;
    return node;
  }
}
```

`primary(&tok, tok)` parses a primary and updates `tok`. The loop checks the new `tok`, and if it's `*` or `/`, parses another primary starting at `tok->next` — note we pass `&tok` again so the next call also updates our local `tok`. When neither operator matches, we write our final position to `*rest` and return.

Could we have used a global `Token *current` instead? Yes. The C compiler `tcc` does exactly that — one global cursor, no rest-pointers. chibicc could too. But the explicit-cursor style has one big virtue: when you read a parser function, every input and output is right there in the signature. There's no shared state to track. When something goes wrong, you can run the function in isolation. When you're modifying one parser function, you don't have to wonder which other function might also be reading or writing the cursor.

The cost is verbosity — every call passes two arguments where one would do. Rui takes that trade-off, and the same `(Token **rest, Token *tok)` signature appears on essentially every parser function for the rest of the book.

### `expr` and `primary`

```c
// expr = mul ("+" mul | "-" mul)*
static Node *expr(Token **rest, Token *tok) {
  Node *node = mul(&tok, tok);

  for (;;) {
    if (equal(tok, "+")) {
      node = new_binary(ND_ADD, node, mul(&tok, tok->next));
      continue;
    }

    if (equal(tok, "-")) {
      node = new_binary(ND_SUB, node, mul(&tok, tok->next));
      continue;
    }

    *rest = tok;
    return node;
  }
}

// primary = "(" expr ")" | num
static Node *primary(Token **rest, Token *tok) {
  if (equal(tok, "(")) {
    Node *node = expr(&tok, tok->next);
    *rest = skip(tok, ")");
    return node;
  }

  if (tok->kind == TK_NUM) {
    Node *node = new_num(tok->val);
    *rest = tok->next;
    return node;
  }

  error_tok(tok, "expected an expression");
}
```

Each function is the BNF rule of the previous interlude, transcribed into C. The grammar comment on the line above each function definition documents this. Whenever Rui adds a feature to the grammar, he updates that comment first, then makes the function match. We will see this discipline kept up rigidly across all 316 commits.

`primary` is where the recursion gets tied — if it sees an open paren, it descends back into `expr` for the contents, then demands a closing paren via `skip`. If it sees a number, it builds a leaf. If it sees anything else, it dies with `expected an expression`.

A thing worth noticing: `error_tok` is reachable but the function never returns from that call — `error_tok` calls `verror_at` calls `exit`. The compiler has no `[[noreturn]]` annotation here, but the function body has no `return` statement after `error_tok`, and that's fine because we *won't* reach the end. A picky compiler might warn about a missing return; chibicc passes `-Wall` later in the book, so apparently GCC is content with this on the platforms Rui targets.

### Stack-based code generation

```c
static int depth;

static void push(void) {
  printf("  push %%rax\n");
  depth++;
}

static void pop(char *arg) {
  printf("  pop %s\n", arg);
  depth--;
}

static void gen_expr(Node *node) {
  if (node->kind == ND_NUM) {
    printf("  mov $%d, %%rax\n", node->val);
    return;
  }

  gen_expr(node->rhs);
  push();
  gen_expr(node->lhs);
  pop("%rdi");

  switch (node->kind) {
  case ND_ADD:
    printf("  add %%rdi, %%rax\n");
    return;
  case ND_SUB:
    printf("  sub %%rdi, %%rax\n");
    return;
  case ND_MUL:
    printf("  imul %%rdi, %%rax\n");
    return;
  case ND_DIV:
    printf("  cqo\n");
    printf("  idiv %%rdi\n");
    return;
  }

  error("invalid expression");
}
```

This is the second-most chibicc-specific idiom in the book, after the parser's rest-pointer style. Code generation walks the AST, uses `%rax` as an accumulator for "the current expression result", and uses the machine stack as a scratch area.

The recursive plan: to compute `lhs OP rhs`, generate code that:

1. Computes `rhs` and leaves the result in `%rax`.
2. Pushes `%rax` onto the stack.
3. Computes `lhs`, leaves the result in `%rax`.
4. Pops the saved `rhs` value into `%rdi`.
5. Performs the operation `%rax OP %rdi`, leaving the result in `%rax`.

After this sequence, `%rax` holds the value of the whole subtree. So generating code for the parent is the same recursive procedure — `%rax` is the implicit return slot of `gen_expr`.

The right child is computed first because we want the *left* child's result in `%rax` at the moment of the `add`/`sub`/`imul`/`idiv` instruction. The left value goes on the destination side (`%rax`), the right value comes off the stack (into `%rdi`), and the `OP %rdi, %rax` form puts the result in `%rax`.

This is hilariously inefficient. `1 + 2` becomes:

```
  mov $2, %rax
  push %rax
  mov $1, %rax
  pop %rdi
  add %rdi, %rax
```

A halfway-decent optimizer would emit `mov $3, %rax`. A bad optimizer would emit `mov $1, %rax; add $2, %rax`. chibicc emits five instructions, including a memory write and a memory read, where one or two would do. It will keep doing things like this for the rest of the book. Per the README: "There's no optimization pass. chibicc emits terrible code which is probably twice or more slower than GCC's output." The point of chibicc is not to compete with GCC. It's to be readable.

The `cqo` / `idiv` pair for division is a quirk of x86-64. `idiv` divides a 128-bit dividend (held in `%rdx:%rax`) by its operand, putting the quotient in `%rax` and the remainder in `%rdx`. To divide a single 64-bit number by another, you have to first sign-extend it from `%rax` into `%rdx:%rax`. The `cqo` instruction does exactly that — it copies the sign bit of `%rax` into all bits of `%rdx`. Without `cqo`, dividing `-6 / 2` would give nonsense because `%rdx` would still hold whatever it had before. This is the kind of tiny ABI/ISA detail that the book has to surface every now and then; we'll let the comments in the code stand for most of them.

### The `depth` invariant

The `depth` variable counts pushes minus pops. After `gen_expr` is done with the whole AST, `main` asserts that `depth == 0`:

```c
gen_expr(node);
printf("  ret\n");

assert(depth == 0);
```

If we ever generate code that pushes more than it pops (or vice versa), the stack pointer will end up at the wrong place when we hit `ret`, and the program will probably crash or jump somewhere insane. The `depth == 0` check is a static-style invariant maintained dynamically: at compile time, after `gen_expr` returns for the root, the count must be back to zero. If a future commit ever introduces a push without a matching pop, this assertion will catch it before we've produced a broken executable. It's the kind of cheap insurance that costs almost nothing and pays for itself the first time it fires.

### `main` becomes a coordinator

```c
int main(int argc, char **argv) {
  if (argc != 2)
    error("%s: invalid number of arguments", argv[0]);

  // Tokenize and parse.
  current_input = argv[1];
  Token *tok = tokenize();
  Node *node = expr(&tok, tok);

  if (tok->kind != TK_EOF)
    error_tok(tok, "extra token");

  printf("  .globl main\n");
  printf("main:\n");

  // Traverse the AST to emit assembly.
  gen_expr(node);
  printf("  ret\n");

  assert(depth == 0);
  return 0;
}
```

Three lines do the real work. `tokenize()` produces a token list. `expr()` produces an AST. `gen_expr()` emits assembly. The `if (tok->kind != TK_EOF)` check makes sure `expr` consumed the entire input — without it, the input `1+2 3` would parse `1+2` and silently ignore the `3`.

If you squint, this is a very small Lisp interpreter: read, evaluate, print. The phases align: tokenize is read, expr is parse-into-tree, gen_expr is emit. The shape never changes. The remaining 309 commits add features to the tokenizer, the parser, and the code generator — but `main` stays at this rough size for a long time, just stitching the three together.

### The new tests

```sh
assert 47 '5+6*7'
assert 15 '5*(9-6)'
assert 4 '(3+5)/2'
```

Three lines, but they cover precedence (`5+6*7 = 47`, not `77`), parenthesized grouping (`5*(9-6) = 15`), and integer division (`(3+5)/2 = 4`).

### Where we are

The compiler now has the architecture it will keep for the rest of the book. The tokenizer is a string-to-list-of-tokens function. The parser is a list-of-tokens-to-tree function, written in recursive-descent, producing nodes that mirror C operators. The code generator is a tree-to-assembly function, using the stack as a scratch area and `%rax` as the implicit return slot. Each pass is small, each pass is named, and each pass is independent of the others. From here on, "add a feature" almost always means "extend each of the three passes a little bit".

---

## 1.6 — Unary plus and minus

> `git checkout bf9ab52860c1cbbeeca40df515468f42300ff429` — *Add unary plus and minus*

A small commit that introduces a new layer in the grammar:

```diff
+// unary = ("+" | "-") unary
+//       | primary
+static Node *unary(Token **rest, Token *tok) {
+  if (equal(tok, "+"))
+    return unary(rest, tok->next);
+
+  if (equal(tok, "-"))
+    return new_unary(ND_NEG, unary(rest, tok->next));
+
+  return primary(rest, tok);
+}
```

`mul`'s grammar changes from `primary ("*" primary | "/" primary)*` to `unary ("*" unary | "/" unary)*`, and `unary` itself sits between `mul` and `primary`. A new node kind `ND_NEG` represents unary negation; a new constructor `new_unary` creates a node with only a left child; codegen emits `neg %rax` after generating the operand:

```c
case ND_NEG:
  gen_expr(node->lhs);
  printf("  neg %%rax\n");
  return;
```

Three subtle points worth flagging.

**Unary `+` produces no node.** The grammar rule is `("+" | "-") unary`, and the implementation for `+` is just `return unary(rest, tok->next)` — a recursive call that throws away the `+`. The plus sign has no effect on its operand, and producing an `ND_POS` node would just create work for the code generator to no purpose. C's semantics actually do mandate one tiny effect — unary `+` triggers integer promotion — but at this stage of chibicc there is no type system at all, so there's nothing to promote. Skipping the node is correct.

**Unary `-` recurses on `unary`, not on `primary`.** That's why the test case `'- -10'` works — the outer `-` consumes itself, then calls `unary` again, which sees the inner `-` and recurses again, which finally sees `10` and produces a primary. The chain `- - +10` works the same way: three nested unary operators, the `+` and one of the `-`s short-circuit, and `10` is what comes out the bottom. If `unary` had recursed on `primary` instead, `-(-10)` would still work (because of the parentheses) but `- -10` would fail at parse time.

**`new_unary` is a sibling of `new_binary`, not a special case of it.** Looking at the new constructor:

```c
static Node *new_unary(NodeKind kind, Node *expr) {
  Node *node = new_node(kind);
  node->lhs = expr;
  return node;
}
```

The unary operand goes into `lhs`, not into a separate `child` field. The convention chibicc adopts: unary operators put their single operand in `lhs`. We could have used `rhs`. Either is fine. Picking `lhs` and being consistent about it is what matters. Code generation for `ND_NEG` follows the same convention: it generates `node->lhs` and then negates.

The `gen_expr` switch is also restructured: where it previously had an `if (node->kind == ND_NUM)` followed by binary handling, it now has a `switch` with `ND_NUM` and `ND_NEG` cases at the top, followed by the binary handling for everything else. This switch will keep growing for the next 50 commits before Rui splits things up further.

### Where we are

The grammar has gained a layer. We can express things like `-x + y` and `--x` with the right precedence — unary binds tighter than `*`, which binds tighter than `+`, which is exactly C. The same pattern — adding a new precedence level by adding a new function between two existing ones — is going to repeat at least seven more times before we're done with operators.

---

## 1.7 — Equality and ordering

> `git checkout 25b4b85b887c643e337a9fbcd1b0220b413952bf` — *Add `==`, `!=`, `<=` and `>=` operators*

The last commit of the chapter. It adds two new precedence levels (equality and relational), four new operators (`==`, `!=`, `<=`, `>=`, plus `<` and `>` which are below `<=` and `>=` in the parser's mind), multi-character punctuator support to the tokenizer, and the `cmp` / `setX` / `movzb` idiom to the code generator.

### Multi-character punctuators

The tokenizer used to look at one character and make a token of length one. Now some punctuators are two characters:

```c
static bool startswith(char *p, char *q) {
  return strncmp(p, q, strlen(q)) == 0;
}

// Read a punctuator token from p and returns its length.
static int read_punct(char *p) {
  if (startswith(p, "==") || startswith(p, "!=") ||
      startswith(p, "<=") || startswith(p, ">="))
    return 2;

  return ispunct(*p) ? 1 : 0;
}
```

`read_punct` returns the length of the punctuator at `p`, or `0` if there isn't one. The two-character cases are checked first — order matters, because `==` starts with `=`, which is a one-character punctuator (well, it isn't yet, but it will be in a few chapters), and we want to match the longer one.

The tokenizer loop is updated to use it:

```c
int punct_len = read_punct(p);
if (punct_len) {
  cur = cur->next = new_token(TK_PUNCT, p, p + punct_len);
  p += cur->len;
  continue;
}
```

Notice that `p += cur->len` rather than `p += punct_len` — both would work, but reading from the just-built token is slightly more direct and saves a temporary. Stylistic; not load-bearing.

### Two new precedence levels

The grammar now reads:

```
expr       = equality
equality   = relational ("==" relational | "!=" relational)*
relational = add ("<" add | "<=" add | ">" add | ">=" add)*
add        = mul ("+" mul | "-" mul)*
mul        = unary ("*" unary | "/" unary)*
unary      = ("+" | "-") unary | primary
primary    = "(" expr ")" | num
```

`expr` used to handle `+ -`; that role gets renamed to `add` and `expr` becomes a thin alias for `equality`. Two new functions, `equality` and `relational`, sit between them. The rename is the kind of forward-looking move Rui makes consistently: the new top-level `expr` is going to keep growing a lower-precedence layer above it (assignment, comma) for the next several commits, and pinning the topmost name to `expr` early avoids a series of churn-y renames.

```c
// expr = equality
static Node *expr(Token **rest, Token *tok) {
  return equality(rest, tok);
}
```

A one-line function that just delegates. Looks redundant; isn't. Keeping `expr` as the entry point lets the rest of the parser keep saying `expr(&tok, tok)` while the actual top-of-grammar gets richer underneath.

`equality` and `relational` follow the same template as `add` and `mul`:

```c
static Node *equality(Token **rest, Token *tok) {
  Node *node = relational(&tok, tok);

  for (;;) {
    if (equal(tok, "==")) {
      node = new_binary(ND_EQ, node, relational(&tok, tok->next));
      continue;
    }

    if (equal(tok, "!=")) {
      node = new_binary(ND_NE, node, relational(&tok, tok->next));
      continue;
    }

    *rest = tok;
    return node;
  }
}
```

The README's design principles section mentioned this directly: *"The recursive descendent parser contains many similar-looking functions for similar-looking generative grammar rules. You might be tempted to improve it to reduce the duplication using higher-order functions or macros, but I thought that that's too complicated. It's better to allow small duplications instead."* This is the start of that. There will be a lot more of it. We will not abstract it out.

### `<` is not a node kind

Look closely at `relational`:

```c
if (equal(tok, ">")) {
  node = new_binary(ND_LT, add(&tok, tok->next), node);
  continue;
}

if (equal(tok, ">=")) {
  node = new_binary(ND_LE, add(&tok, tok->next), node);
  continue;
}
```

`a > b` produces an `ND_LT` node with the operands swapped. `a >= b` produces an `ND_LE` node, also swapped. There are no `ND_GT` or `ND_GE` node kinds, because `a > b` is precisely `b < a` and `a >= b` is precisely `b <= a`. By doing the rewrite at parse time, we save the code generator from having to handle four cases — it only has to know how to emit `<` and `<=`.

This is a small but real piece of design wisdom: make canonicalizations early. If a feature has a cleaner internal form, store the cleaner form. The parser is the right place to do this — it's where source-level forms become internal forms, and it's the only pass that has to know about the source-level redundancy.

### Code generation for comparisons

```c
case ND_EQ:
case ND_NE:
case ND_LT:
case ND_LE:
  printf("  cmp %%rdi, %%rax\n");

  if (node->kind == ND_EQ)
    printf("  sete %%al\n");
  else if (node->kind == ND_NE)
    printf("  setne %%al\n");
  else if (node->kind == ND_LT)
    printf("  setl %%al\n");
  else if (node->kind == ND_LE)
    printf("  setle %%al\n");

  printf("  movzb %%al, %%rax\n");
  return;
```

A comparison evaluates its operands the same way as arithmetic — right operand pushed, left operand in `%rax`, right operand popped to `%rdi`. Then:

1. `cmp %rdi, %rax` does the comparison. This is one of the small mind-twisters of x86 AT&T syntax: `cmp %rdi, %rax` computes `%rax - %rdi` and sets the flags accordingly. The first operand is the *subtrahend* (right operand of the subtraction), the second is the *minuend*. So `cmp %rdi, %rax` followed by `setl` (set if less) is a test for `%rax < %rdi`, which is `lhs < rhs`. This is what we want.

2. `sete`, `setne`, `setl`, `setle` are the `set`-on-condition family. They write 1 or 0 into the low byte of a register based on the flags. We use `%al`, the low byte of `%rax`.

3. `movzb %al, %rax` zero-extends the low byte into the full register. Without this step, the high bits of `%rax` would still hold leftover data from the comparison or the previous arithmetic, and a subsequent `add` or `cmp` against the boolean would see garbage in the upper bits. The result of every comparison is canonically a clean 64-bit 0 or 1.

### Tests

The new tests exercise the new operators thoroughly:

```sh
assert 0 '0==1'
assert 1 '42==42'
...
assert 1 '1>=0'
assert 1 '1>=1'
assert 0 '1>=2'
```

Eighteen cases for six operators. `>` and `>=` get the same coverage as `<` and `<=` despite not having their own node kinds — because the language-level test doesn't care about internal representation, it only cares that `1>0` evaluates to `1`. The parse-time canonicalization is invisible to the test suite, which is exactly what makes it a good canonicalization.

### Where we are

Chapter 1 ends here. Run `make test` and 28 cases pass. The compiler:

- Tokenizes input, skipping whitespace and producing a linked list of tokens with kind, location, length, and (for numbers) value.
- Parses the token list with a recursive-descent parser whose function names mirror BNF non-terminals, returning a tree of nodes.
- Walks the tree with a code generator that uses `%rax` as a scratch accumulator and the machine stack as scratch space, emitting AT&T-syntax x86-64 assembly.
- Pipes its output through `gcc -static`, which assembles and statically links a binary that exits with the integer value of the input expression.

The architecture is final. The language is a calculator. Over the next chapter we'll add stateful infrastructure (a `main`-as-orchestrator that splits parse and codegen properly into named files), and then we'll start adding actual C: variables, statements, control flow.

---

## Recap of the calculator

| Commit | What it added | Cumulative size of `main.c` |
|---|---|---:|
| `0522e2d` | Single-integer compilation | 15 lines |
| `bf7081f` | `+` and `-` | ~35 lines |
| `a1ab0ff` | Tokenizer, error helper, token-list interface | ~120 lines |
| `cc5a6d9` | Caret-pointed error messages | ~145 lines |
| `84cfcaf` | AST, recursive-descent parser, `*` `/` `()`, codegen pass | ~290 lines |
| `bf9ab52` | Unary `+ -` | ~310 lines |
| `25b4b85` | Multi-char punctuators, `==` `!=` `<` `<=` `>` `>=` | ~395 lines |

The compiler grew from 15 lines of "print four lines of assembly" into 395 lines of three-pass compilation in seven commits over the course of one Saturday. None of the commits is more than ~180 lines of diff; most are under 50. Each one passes its own tests *and* every test from the previous commits — that's the value of the bug-free-history policy. From here on out, that property holds for all 309 remaining commits.

The next chapter is just one commit long. It splits `main.c` into multiple files. After eight commits the calculator will be a small library, ready to grow into a real language.
