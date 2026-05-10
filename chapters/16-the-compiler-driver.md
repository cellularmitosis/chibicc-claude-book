# Chapter 16 — The compiler driver

> Commits covered: `5d15431`, `d06a8ac`, `c5953ba`, `53e8103`, `f3d9613`, `140b433`, `b833cd0`, `8b726b5`. Eight commits — the stage-2 Makefile target, the function-pointer trio (type, decay, arithmetic conversion), the cc1-vs-driver split, the `as` invocation, multiple input files, and the `ld` invocation.

Through Chapter 15, `chibicc` has been a single binary that takes one C source file on its command line and emits an assembly file on its standard output (or to a `-o` path). To turn that assembly into an executable, the test Makefile pipes it through the host `cc` for assembly and linking. The compiler is real, but the *front end* — the program a user types `cc input.c` at — has been borrowed from the host system.

Chapter 16 builds chibicc's own front end. By the end of the chapter, `chibicc input.c` produces an executable. The single binary becomes two: `cc1` (the compiler proper, the existing program with a renamed entry point) and `chibicc` (a driver that handles command-line arguments, runs `cc1` for each input, runs `as` to assemble cc1's output, and runs `ld` to link the assembled objects into a binary). The driver also handles the small but important detail that real compilers accept multiple input files at once, dispatch on file extension (`.c`, `.s`, `.o`), and observe the conventional flags `-S`, `-c`, and `-o`.

Eight commits. The chapter mapping flagged this as a bundling-feasible chapter and the bundling lands as predicted: the three function-pointer commits (the type, the decay rule, the arithmetic-conversion rule) collapse into one section. The other five commits each get their own. No concept interlude — the GCC-driver-vs-cc1 split is the substance of §16.3 and lives there inline.

The six sections:

- **§16.1** — Stage-2 build (commit 150).
- **§16.2** — Function pointers: type, decay, and arithmetic conversion (commits 151, 152, 153).
- **§16.3** — Splitting cc1 from the driver (commit 154).
- **§16.4** — Running the assembler (commit 155).
- **§16.5** — Multiple input files (commit 156).
- **§16.6** — Running the linker (commit 157).

The chapter follows `main` order. The calendar dates scatter further than usual: commit 150 (stage-2) is dated *August 2019* — a full year before the rest of the chapter — and commits 151–157 cluster across August, September, and October 2020. Commit 150's calendar date predates the entirety of Chapters 12 through 15. Reading the diff explains why: it's a Makefile change, untouched by the type-system work that came later, and Rui parked it in `main` order at a point that made sense as a *structural* prelude to self-hosting rather than as a calendar-time milestone. The chapter follows `main` order without untangling.

---

## 16.1 — Stage-2 build

> `git checkout 5d15431df1abab3a5cf596fabe0a77c030a10791` — *Add stage2 build*

The first commit of the chapter is a Makefile change, with one auxiliary script and no source-code change to chibicc itself. It introduces a `stage2` build target — a build of chibicc that uses the just-built chibicc to compile chibicc's own source code. This isn't self-hosting yet. Self-hosting will not arrive until Chapter 17, when chibicc grows a preprocessor of its own. What this commit does is *prepare the ground*: set up the Makefile pipeline that, when the preprocessor lands, will start producing a chibicc compiled by chibicc.

The Makefile gains four new targets and a comment-banner organization:

```makefile
# Stage 1

chibicc: $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^ $(LDFLAGS)
# ... existing test target ...

# Stage 2

stage2/chibicc: $(OBJS:%=stage2/%)
	$(CC) $(CFLAGS) -o $@ $^ $(LDFLAGS)

stage2/%.s: chibicc self.py %.c
	mkdir -p stage2/test
	./self.py chibicc.h $*.c > stage2/$*.c
	./chibicc -o stage2/$*.s stage2/$*.c

stage2/test/%.exe: stage2/chibicc test/%.c
	mkdir -p stage2/test
	$(CC) -o- -E -P -C test/$*.c | ./stage2/chibicc -o stage2/test/$*.s -
	$(CC) -o $@ stage2/test/$*.s -xc test/common

test-stage2: $(TESTS:test/%=stage2/test/%)
	for i in $^; do echo $$i; ./$$i || exit 1; echo; done
	test/driver.sh ./stage2/chibicc
```

The pipeline reads top-down: `stage2/chibicc` is an executable built by linking `stage2/*.o` files with the host `cc`. Each `stage2/*.s` is produced by running the just-built `./chibicc` on a *preprocessed* version of one of chibicc's own source files. The preprocessing is done by `self.py`, a Python script that — because chibicc has no preprocessor — does the preprocessing job by hand.

`self.py` is the trick that makes the whole pipeline work. It reads chibicc's own `.c` files and edits them into something chibicc can parse: it strips `\` line continuations, deletes any line starting with `#` (which throws away `#include` directives, the entire C preprocessor's job), removes adjacent string-literal concatenation (`"foo"\n  "bar"` becomes `"foobar"`), substitutes `bool` with `_Bool`, `errno` with `*__errno_location()`, `true` and `false` with `1` and `0`, `NULL` with `0`, `va_start(ap, n)` with `*(ap)=*(__va_elem*)__va_area__`, `unreachable()` with a call to `error("unreachable")`, and `MIN(x, y)` with the open-coded ternary form. Then, because the original file had `#include <stdio.h>` and friends stripped, the script *prepends* a hand-written block of typedefs and function declarations (`int8_t`, `size_t`, `FILE *fopen(...)`, etc.) so chibicc has type information for the standard library functions the source code calls.

This is a preprocessor implemented as a Python regex pipeline — pragmatic, brittle, and exactly enough for chibicc's source code in its present shape. It's the placeholder Chapter 17 will replace with a real preprocessor. Until then, it bridges the gap.

The compile step itself is unremarkable:

```
./chibicc -o stage2/$*.s stage2/$*.c
```

The just-built chibicc reads the preprocessed source, emits assembly, and writes it to a stage-2 directory. The stage-2 binary is then linked together by the host `cc` using its assembler and linker — chibicc still doesn't run those tools itself in this commit (that's §16.4 and §16.6).

The `test-stage2` target reuses the existing test-Makefile machinery. Each test C file is preprocessed by the host `cc -E`, fed through `./stage2/chibicc` instead of `./chibicc`, then linked by host `cc`, then executed. If `stage2/chibicc` produces correct code for every test, the chapter's "stage-2" milestone is met.

The driver-shell test (`test/driver.sh`) is also adjusted to take the chibicc binary path as its first argument, so it can be pointed at either `./chibicc` or `./stage2/chibicc`:

```diff
-./chibicc --help 2>&1 | grep -q chibicc
+chibicc=$1
+...
+$chibicc --help 2>&1 | grep -q chibicc
```

Two small consequences. The first is that *chibicc can compile the parts of its own source that don't use the C preprocessor* — meaning, anything `self.py` can handle. That's most of `parse.c`, `codegen.c`, `tokenize.c`, `type.c`, `strings.c`, and `main.c`. The exclusion list (`#include`, `#define`, conditional compilation, function-like macros that aren't `MIN`) is large in principle but small for chibicc's actual code: chibicc's source is written in a deliberately preprocessor-light style, exactly so that `self.py` can be a thin Python script rather than a full preprocessor. Rui has been writing chibicc to be self-hostable since the beginning; this commit is where that style decision starts paying dividends.

The second consequence is that *the stage-2 build is the canary for self-hosting*. If a future commit breaks the stage-2 build — by introducing a feature `self.py` can't handle, or by making chibicc emit code that miscompiles its own source — `make test-stage2` catches it. This becomes the regression net for Chapter 17's preprocessor: as the preprocessor lands, `self.py`'s job shrinks, and eventually the stage-2 target stops needing `self.py` at all.

The new `.gitignore` entry is a one-liner:

```
+/stage2
```

— so `make clean` and `git status` ignore the build artifacts.

### Where we are

The Makefile has a stage-2 target. Chibicc compiles a Python-preprocessed copy of its own source code into assembly, which the host `cc` then assembles and links into `stage2/chibicc`. The stage-2 binary runs the same test suite as the stage-1 binary. None of this is self-hosting — `self.py` is doing the preprocessor's job. But the pipeline's *shape* is the shape self-hosting will take. Chapter 17 will replace `self.py` with a real preprocessor.

---

## 16.2 — Function pointers: type, decay, and arithmetic conversion

> `git checkout d06a8ac6e6120861c9c79acb15b9a18693e4ee47` — *Add function pointer*
>
> `git checkout c5953ba1328fa86f906406843eb9f23cd596ef04` — *Decay a function to a pointer in the func param context*
>
> `git checkout 53e81033ce18fd94fcdcde9010b7c9d41f30aa2c` — *Add usual arithmetic conversion for function pointer*

Three commits in a row, all on function pointers. Bundled here because the story they tell is one story — *type → decay → arithmetic conversion*, the same three-step shape Chapter 6 walked through for arrays. Function pointers are the C type that fits exactly the same mold: a function name in expression context decays to a pointer-to-function (just as an array name decays to a pointer-to-element), and the resulting pointer participates in the usual arithmetic conversions (just as the array-decayed pointer does in `arr1 == arr2 + 1` style expressions). Rui's three commits land the three pieces in order.

The first commit is the largest, and it does most of the work: it makes `int (*fp)(int)` a valid declaration, makes `fp(x)` a valid call, and makes `&add2` and `add2` (in expression context) both yield a function pointer. Function pointers had partial support before — the parser already accepted parenthesized declarators like `int (*fp)(int)` from Chapter 10's full declarator syntax, and the function-pointer *type* could be constructed — but the codegen and the call path only handled named function calls. After this commit, the call path handles indirect calls through any expression of function-pointer type.

The change has three corners. The codegen's `gen_addr`, the `funcall` parsing logic, and the `funcall` codegen emission.

`gen_addr` learns to compute the address of a function. The old code had a simple split: local variable → `lea offset(%rbp), %rax`; global variable → `lea name(%rip), %rax`. Functions take the global-variable path (a function is, in chibicc's `Obj` model, just a global with `is_function` set), but the addressing mode is more complicated, and Rui takes the opportunity to write the longest comment block in `codegen.c`:

```c
case ND_VAR:
  // Local variable
  if (node->var->is_local) {
    println("  lea %d(%%rbp), %%rax", node->var->offset);
    return;
  }

  // Here, we generate an absolute address of a function or a global
  // variable. Even though they exist at a certain address at runtime,
  // their addresses are not known at link-time for the following
  // two reasons.
  //
  //  - Address randomization: ...
  //  - Dynamic linking: ...
  //
  // In order to deal with the former case, we use RIP-relative
  // addressing, denoted by `(%rip)`. For the latter, we obtain an
  // address of a stuff that may be in a shared object file from the
  // Global Offset Table using `@GOTPCREL(%rip)` notation.

  // Function
  if (node->ty->kind == TY_FUNC) {
    if (node->var->is_definition)
      println("  lea %s(%%rip), %%rax", node->var->name);
    else
      println("  mov %s@GOTPCREL(%%rip), %%rax", node->var->name);
    return;
  }

  // Global variable
  println("  lea %s(%%rip), %%rax", node->var->name);
  return;
```

The branch is by `is_definition`: if the function is defined in this translation unit, its address is known at link time within the executable, and `lea name(%rip), %rax` (RIP-relative load-effective-address) produces it directly. If the function is *not* defined here — `extern int printf(char *, ...);` style — its address may live in a shared object loaded at runtime, and the address has to be looked up via the Global Offset Table at runtime. The `name@GOTPCREL(%rip)` syntax is the assembler's way of asking the linker to fill in a GOT entry; the `mov` then loads through that entry to retrieve the actual function address.

The comment block is unusual for chibicc — Rui rarely writes prose this long in the codegen — and the reason is that the GOT/PIC story is the kind of thing that confuses every C programmer the first time they meet it. Rui spends fifteen lines making sure the reader doesn't have to know *why* `lea` versus `mov @GOTPCREL` to follow the rest of the chapter. The accompanying `load` change is one extra `case`:

```c
case TY_FUNC:
  // If it is an array, do not attempt to load a value to the
  // register because in general we can't load an entire array to a
  // register. ...
```

— grouped with `TY_ARRAY`, `TY_STRUCT`, `TY_UNION`. Loading a function-typed expression into a register is the same kind of category-error as loading an array into a register: the function-typed expression's *value* is its address, and `gen_addr` already put that address in `%rax`, so `load` should not emit anything further. This is the function-pointer analog of the array-decay rule from Chapter 6: function names appear in expression context, and the natural code-gen path (compute address, optionally load) skips the load step for `TY_FUNC` exactly as it skips it for `TY_ARRAY`.

The parser change is in `postfix`. The old shape was:

```
postfix = "(" type-name ")" "{" initializer-list "}"
        | primary ("[" expr "]" | "." ident | "->" ident | "++" | "--")*
```

— and the function-call case was handled separately, in `primary`, by looking at an identifier and peeking ahead for `(`:

```c
if (tok->kind == TK_IDENT) {
  // Function call
  if (equal(tok->next, "("))
    return funcall(rest, tok);
  // ...
}
```

This worked when *every* call was through a named function. It doesn't work when the callee is `(*fp)`, or `&add2`, or `(1 ? f : g)`, or any expression of function-pointer type. The rewrite moves the call detection into `postfix`'s loop:

```c
postfix = ident "(" func-args ")" postfix-tail*
        | primary postfix-tail*

postfix-tail = "[" expr "]"
             | "(" func-args ")"
             | "." ident
             | "->" ident
             | "++"
             | "--"
```

```c
Node *node = primary(&tok, tok);

for (;;) {
  if (equal(tok, "(")) {
    node = funcall(&tok, tok->next, node);
    continue;
  }

  if (equal(tok, "[")) { /* ... */ }
  // ...
}
```

A `(` in postfix position now means *call whatever expression we just parsed* — whether that expression is a named function, a function pointer dereference, a conditional expression that evaluates to a function pointer, anything. The callee node is passed into `funcall` as a parameter rather than reconstructed from a token.

`funcall` itself loses its "look up the name in the variable scope" prelude:

```c
static Node *funcall(Token **rest, Token *tok, Node *fn) {
  add_type(fn);

  if (fn->ty->kind != TY_FUNC &&
      (fn->ty->kind != TY_PTR || fn->ty->base->kind != TY_FUNC))
    error_tok(fn->tok, "not a function");

  Type *ty = (fn->ty->kind == TY_FUNC) ? fn->ty : fn->ty->base;
  Type *param_ty = ty->params;
  // ... rest unchanged ...

  Node *node = new_unary(ND_FUNCALL, fn, tok);
  node->func_ty = ty;
  node->ty = ty->return_ty;
  node->args = head.next;
  return node;
}
```

The callee expression is the new `lhs` of `ND_FUNCALL` (constructed via `new_unary`). The type test accepts either a direct function (`fn->ty->kind == TY_FUNC` — the unusual case where the callee is, syntactically, a function name without decay yet applied) or a pointer-to-function (`fn->ty->kind == TY_PTR && fn->ty->base->kind == TY_FUNC` — the common case after function-decay). The callee's type, dereferenced if necessary, becomes the `func_ty` for the call (used for parameter-type lookup). The old `funcname` field on `Node` is gone:

```diff
-  // Function call
-  char *funcname;
   Type *func_ty;
   Node *args;
```

— since the call site no longer cares about the callee's name; it only cares about the callee's address, which is now computed by walking `node->lhs`.

The codegen for `ND_FUNCALL` adjusts to match. The old code emitted `call %s` with the callee name. The new code emits `call *%rax`:

```c
case ND_FUNCALL: {
  push_args(node->args);
  gen_expr(node->lhs);

  // ... gp/fp argument loads ...

  if (depth % 2 == 0) {
    println("  call *%%rax");
  } else {
    println("  sub $8, %%rsp");
    println("  call *%%rax");
    println("  add $8, %%rsp");
  }
  // ...
}
```

`gen_expr(node->lhs)` walks the callee expression, which leaves the callee's address in `%rax`. Then `call *%rax` is the indirect call instruction — the `*` prefix is AT&T-syntax for "the operand is the address to call, not a relative displacement to a name." For `add2(2, 3)`, `node->lhs` is an `ND_VAR` for `add2`, and `gen_expr` walks it through `gen_addr` to produce `lea add2(%rip), %rax`. For `fp(2, 3)` where `fp` is a local function pointer, `node->lhs` is an `ND_VAR` for `fp`, and `gen_expr` walks it to produce `lea offset(%rbp), %rax; mov (%rax), %rax` (load the pointer's value). Either way, `%rax` holds the function's address by the time `call *%rax` runs.

Note one thing: every function call now goes through `call *%rax`, even direct named ones. There's no longer a fast path that emits `call name` for a known-by-name callee. This is a small pessimization (the indirect call adds a few cycles compared to a relative direct call, and on modern x86 the relative `call name` can be patched to the GOT entry by the linker if needed) but it's a worthwhile simplification: one code path, one instruction sequence, no dispatch on whether the callee is direct or indirect. Rui's "do the simple thing" principle continues.

The first commit's tests exercise the new shapes:

```c
ASSERT(5, (add2)(2,3));
ASSERT(5, (&add2)(2,3));
ASSERT(7, ({ int (*fn)(int,int) = add2; fn(2,5); }));
ASSERT(6, fnptr(add_all)(3, 1, 2, 3));
```

Four cases. `(add2)(2,3)` is `add2` in parentheses called with arguments — the parentheses force `add2` into expression context, where it decays to a function pointer, which is then called. `(&add2)(2,3)` is the explicit address-of, then a call through that pointer. The third assigns `add2` (decayed to pointer) to a local function pointer variable, then calls through it. The fourth calls `fnptr` (which returns a function pointer) and immediately calls the returned pointer — `fnptr(add_all)(3, 1, 2, 3)` — exercising the most twisted possible expression shape, since `fnptr` itself takes a function pointer as an argument and returns a function pointer.

The `fnptr` declaration in the test file is the chapter's most baroque piece of C syntax:

```c
int (*fnptr(int (*fn)(int n, ...)))(int, ...) {
  return fn;
}
```

Read outward: `fnptr` is a function that takes one parameter `fn` of type `int (*)(int n, ...)` (function-pointer-to-variadic-int-returner) and returns a value of type `int (*)(int, ...)` (also function-pointer-to-variadic-int-returner). Chibicc's declarator parser, since Chapter 10, handles this without modification — the parenthesized declarator can be arbitrarily nested. Adding *function pointers as a thing one can call* exposes that this declarator parser was already doing the right thing.

The second commit (decay in parameter context) is six lines:

```c
Token *name = ty2->name;

if (ty2->kind == TY_ARRAY) {
  // "array of T" is converted to "pointer to T" only in the parameter
  // context. For example, *argv[] is converted to **argv by this.
  ty2 = pointer_to(ty2->base);
  ty2->name = name;
} else if (ty2->kind == TY_FUNC) {
  // Likewise, a function is converted to a pointer to a function
  // only in the parameter context.
  ty2 = pointer_to(ty2);
  ty2->name = name;
}
```

This is in `func_params` — the function that parses function parameter lists. Chapter 6 added the array-to-pointer rule for parameters; this commit adds the corresponding rule for function-typed parameters. C says: in a function parameter list, a parameter declared as a function (e.g., `int x(int)`) is silently rewritten to a pointer-to-function (`int (*x)(int)`). It's the same syntactic rule that says `void f(int a[])` actually means `void f(int *a)`, applied to the function-type axis instead of the array-type axis.

The test:

```c
int param_decay2(int x()) { return x(); }
// ...
ASSERT(3, param_decay2(ret3));
```

`param_decay2`'s parameter is declared `int x()` — without an asterisk. Without the decay rule, this would be a parameter of *function type*, which is meaningless (you can't pass a function by value; functions aren't values). With the decay rule, the parameter is silently rewritten to `int (*x)()` — a function pointer — and the call site `param_decay2(ret3)` passes the address of `ret3` (which itself decayed at the call site, by the rule from the first commit). Inside `param_decay2`, `x()` calls through the pointer.

The third commit (usual arithmetic conversion) is five lines in `get_common_type`:

```c
if (ty1->kind == TY_FUNC)
  return pointer_to(ty1);
if (ty2->kind == TY_FUNC)
  return pointer_to(ty2);
```

Above the existing float/double cascade, below the array-decay arm. `get_common_type` is the function that decides what type two operands of a binary operator (or the two arms of a conditional expression, or any other context that demands "the common type") share. For function-typed operands — which can only show up in expression contexts where decay applies — the common type is "pointer to that function type." With this rule in place, `1 ? ret10 : (void *)0` works: the conditional's two arms are a function-typed expression (`ret10`) and a void-pointer expression (`(void *)0`). `get_common_type` decays `ret10` to `int (*)()`, the pointer comparison logic kicks in, and the conditional has type `int (*)()`. The test follows immediately:

```c
ASSERT(10, (1 ? ret10 : (void *)0)());
```

Three commits, three corners of the function-pointer story. The first introduces the type and the indirect call path. The second decays function-typed parameters. The third decays function-typed conditional-expression arms. The pattern mirrors Chapter 6's array story: the type came first, then the parameter-context rule, then the integration with the rest of the type system.

### Where we are

Function pointers exist as a C construct chibicc accepts, type-checks, and compiles. `fp(x)` works whether `fp` is a named function, a function pointer, an address-of-a-function, or any expression that evaluates to a function pointer. The `ND_FUNCALL` codegen emits `call *%rax`, with the callee address computed by `gen_expr(node->lhs)` — uniform across direct and indirect calls. The function-decay rule (function name in expression context → pointer to function) is implemented at three sites: parameter lists (where the parameter type decays at parse time), `gen_addr` and `load` (where the codegen treats `TY_FUNC` like `TY_ARRAY` and skips the value-load), and `get_common_type` (where the type-system unification rule decays a function to a pointer). The `Node->funcname` field is gone. Function pointers are now first-class.

---

## 16.3 — Splitting cc1 from the driver

> `git checkout f3d96136f292dea83fd760098d189a6884f59eb0` — *Split cc1 from compiler driver*

This is the chapter's structural anchor. Through the previous fifteen chapters, "chibicc" has named one binary that does everything: parses arguments, reads a C source file, tokenizes it, parses it, runs codegen, writes assembly. After this commit, "chibicc" is two binaries — `chibicc` (the driver, which dispatches to other tools) and the same `chibicc` binary running with the `-cc1` flag (which acts as cc1, the compiler proper). Real GCC works the same way: `gcc` is the driver; `cc1` (and `cpp`, `as`, `ld`, etc.) are the actual tools `gcc` shells out to.

The split lands as `f3d9613`, and it's the largest single commit in the chapter (after the first function-pointer commit). The trick chibicc plays — and this is a small but elegant one — is that *the cc1 binary and the driver binary are the same executable*, distinguished only by whether `-cc1` is on the argument list. There's no separate `cc1.c`, no separate `make` target. The `chibicc` binary, when invoked normally, becomes the driver and re-execs itself with `-cc1` prepended to compile the input. When invoked with `-cc1`, it becomes the compiler.

The change concentrates in `main.c`. Before, the file's `main` function did the entire compile job inline. After, `main` dispatches by flag:

```c
int main(int argc, char **argv) {
  parse_args(argc, argv);

  if (opt_cc1) {
    cc1();
    return 0;
  }

  run_cc1(argc, argv);
  return 0;
}
```

`cc1()` is the existing compile pipeline lifted into a function:

```c
static void cc1(void) {
  // Tokenize and parse.
  Token *tok = tokenize_file(input_path);
  Obj *prog = parse(tok);

  // Traverse the AST to emit assembly.
  FILE *out = open_file(opt_o);
  fprintf(out, ".file 1 \"%s\"\n", input_path);
  codegen(prog, out);
}
```

— literally the body of the old `main`, with no behavioral change. `run_cc1` is the new driver function that re-invokes the binary with `-cc1` prepended:

```c
static void run_cc1(int argc, char **argv) {
  char **args = calloc(argc + 10, sizeof(char *));
  memcpy(args, argv, argc * sizeof(char *));
  args[argc++] = "-cc1";
  run_subprocess(args);
}
```

`run_subprocess` is the new fork/exec/wait helper:

```c
static void run_subprocess(char **argv) {
  // If -### is given, dump the subprocess's command line.
  if (opt_hash_hash_hash) {
    fprintf(stderr, "%s", argv[0]);
    for (int i = 1; argv[i]; i++)
      fprintf(stderr, " %s", argv[i]);
    fprintf(stderr, "\n");
  }

  if (fork() == 0) {
    // Child process. Run a new command.
    execvp(argv[0], argv);
    fprintf(stderr, "exec failed: %s: %s\n", argv[0], strerror(errno));
    _exit(1);
  }

  // Wait for the child process to finish.
  int status;
  while (wait(&status) > 0);
  if (status != 0)
    exit(1);
}
```

This is shell scripting written in C. `fork()` splits the process. The child calls `execvp(argv[0], argv)`, replacing its image with whatever `argv[0]` names — for the cc1 invocation, `argv[0]` is the same chibicc binary the parent is running, so the child becomes a fresh chibicc process that sees `-cc1` in its argument list. The parent waits for the child to finish, checks the exit status, propagates failure. The `wait` loop (`while (wait(&status) > 0)`) catches all children, not just the one we just spawned — it's defensive, in case some signal-handler or async fork has left a zombie around.

`fork`, `execvp`, `wait`, `_exit`, `errno`, `strerror` — these come from POSIX, declared in the new headers chibicc includes:

```diff
 #include <strings.h>
+#include <sys/types.h>
+#include <sys/wait.h>
+#include <unistd.h>
```

The `-###` flag (which dumps the subprocess command line instead of running it — borrowed from GCC's identical flag) helps debug the driver: `chibicc -### foo.c` shows what `cc1` would have been invoked as, without actually running it. This debugging surface is going to grow as the driver learns to invoke `as` and `ld`.

Two new globals track the driver-vs-cc1 state:

```c
static bool opt_cc1;
static bool opt_hash_hash_hash;
```

— and the argument parser learns to recognize them:

```c
if (!strcmp(argv[i], "-###")) {
  opt_hash_hash_hash = true;
  continue;
}

if (!strcmp(argv[i], "-cc1")) {
  opt_cc1 = true;
  continue;
}
```

The `-cc1` flag is parsed but not advertised in `--help`. It's an internal flag the driver passes to itself; users typing `chibicc input.c` never see it.

This is the *split*. The driver and cc1 share a binary, share argument parsing, share most of the same object files. The branch on `opt_cc1` in `main` is the entire dispatcher: with the flag, do the compile work; without it, fork-exec ourselves with the flag to do the compile work in a child process. The reason it's structured this way — rather than as a separate `cc1` binary — is bootstrap simplicity. Building two binaries from the same source tree with different `main` functions is a Makefile complication chibicc doesn't pay for. Sharing the binary and dispatching on a flag is a single-binary build with a tiny runtime branch.

The choice has consequences. Each compilation forks a new process — the driver doesn't compile in-process, even though it could (the cc1 entry point is just a function call away). The fork-exec round-trip is slower than an in-process function call by milliseconds, which matters when compiling many small files. Real GCC's driver makes the same choice for the same reasons: process isolation makes the compiler's memory state easier to reason about, and the cost is small in the context of a multi-second compile. Chibicc inherits the choice without commentary.

The driver doesn't yet handle multiple input files (that's §16.5), doesn't yet run `as` (§16.4), doesn't yet run `ld` (§16.6). Those will accumulate. After this commit, `chibicc input.c -o output.s` is the same end-to-end behavior as before — just implemented through a fork-exec round-trip instead of a single function call.

`self.py` gains a few new function declarations to match — `basename`, `strrchr`, `unlink`, `mkstemp`, `close`, `fork`, `execvp`, `_exit`, `wait`, `atexit`. None of these are used in the cc1 path yet, but the next commits will need them, and adding them all at once keeps the stage-2 build green:

```diff
+char *basename(char *path);
+char *strrchr(char *s, int c);
+int unlink(char *pathname);
+int mkstemp(char *template);
+int close(int fd);
+int fork(void);
+int execvp(char *file, char **argv);
+void _exit(int code);
+int wait(int *wstatus);
+int atexit(void (*)(void));
```

The `atexit` declaration takes a function pointer parameter — `void (*)(void)`. This is the first appearance of a function-pointer type in chibicc's `self.py` shim, and it works because §16.2 (which landed in the same chapter, two commits earlier in `main` order) made function pointers a first-class declarator.

### Where we are

Chibicc is two roles in one binary. The driver role handles arguments, decides what subprocesses to run, fork-execs them, waits for them. The cc1 role does the actual compile. The two are distinguished by the `-cc1` flag. The fork-exec helper `run_subprocess` will be the shared mechanism for invoking `cc1`, `as`, `ld`, and any other tool the driver delegates to. The `-###` debug flag echoes those commands without running them. A real driver shape is in place, ready to be filled in.

---

## 16.4 — Running the assembler

> `git checkout 140b43358c33fb5e9f86789541dbca306bb64fcc` — *Run "as" command unless -S is given*

The next step is for the driver to invoke the system assembler on cc1's output. After this commit, `chibicc input.c` produces `input.o` — not `input.s` — by running cc1 to produce a temporary `.s` file and then running `as -c` to assemble it into the final `.o`.

Two new flags arrive, plus the `-S` flag whose semantics the commit defines: `-S` says *stop after assembly text generation; don't run the assembler*. Without `-S`, the driver runs the assembler. With `-S`, it doesn't, and the assembly text is the final output.

The commit adds three substantive pieces. The first is a `StringArray` data structure — a small dynamic array of strings — declared in `chibicc.h`:

```c
typedef struct {
  char **data;
  int capacity;
  int len;
} StringArray;

void strarray_push(StringArray *arr, char *s);
```

— and implemented in `strings.c`:

```c
void strarray_push(StringArray *arr, char *s) {
  if (!arr->data) {
    arr->data = calloc(8, sizeof(char *));
    arr->capacity = 8;
  }

  if (arr->capacity == arr->len) {
    arr->data = realloc(arr->data, sizeof(char *) * arr->capacity * 2);
    arr->capacity *= 2;
    for (int i = arr->len; i < arr->capacity; i++)
      arr->data[i] = NULL;
  }

  arr->data[arr->len++] = s;
}
```

Doubling realloc, NULL-fills the new slack so a NULL terminator is always at `data[len]`. This `StringArray` is going to be used everywhere the driver constructs argument lists — it's the C analog of a Python list of strings, and the driver's code reads more cleanly when argument lists can be built up incrementally.

The second piece is temp-file management. The driver needs to create a `.s` file as the cc1 output, then feed it to `as`, then delete it. The four helpers handle the lifecycle:

```c
static char *create_tmpfile(void) {
  char *path = strdup("/tmp/chibicc-XXXXXX");
  int fd = mkstemp(path);
  if (fd == -1)
    error("mkstemp failed: %s", strerror(errno));
  close(fd);

  strarray_push(&tmpfiles, path);
  return path;
}

static void cleanup(void) {
  for (int i = 0; i < tmpfiles.len; i++)
    unlink(tmpfiles.data[i]);
}
```

`mkstemp` creates a file with a unique name (the `XXXXXX` suffix is replaced by random characters) and opens it for writing. The driver immediately closes the file descriptor — it doesn't write through `fd`, it just wants the *name* — and remembers the path in a global `tmpfiles` array. At program exit, `cleanup` walks the array and unlinks each one.

The `cleanup` function is registered with `atexit`:

```c
int main(int argc, char **argv) {
  atexit(cleanup);
  parse_args(argc, argv);
  // ...
}
```

`atexit(cleanup)` says "when this process exits, regardless of how, run `cleanup`." This catches the normal-exit path and also the `error()` exit path (since `error` calls `exit`, which runs registered atexit handlers). What it doesn't catch is a hard kill (`SIGKILL`, segfault, `_exit`, etc.) — but those are unusual, and the `/tmp` filesystem will eventually clean itself.

The third piece is `replace_extn`, a helper for swapping a file's extension:

```c
static char *replace_extn(char *tmpl, char *extn) {
  char *filename = basename(strdup(tmpl));
  char *dot = strrchr(filename, '.');
  if (dot)
    *dot = '\0';
  return format("%s%s", filename, extn);
}
```

`basename` strips the directory part, `strrchr` finds the last `.`, the truncation cuts it off, and `format` (chibicc's existing printf-wrapper) reassembles `name + extn`. Calling `replace_extn("foo/bar.c", ".o")` returns `"bar.o"`. Note the basename strip — the result is *just the filename*, no directory prefix. This means `chibicc foo/bar.c` produces `./bar.o` in the current directory, not `foo/bar.o`. That's GCC's behavior too.

With these three pieces in place, the dispatch in `main` extends:

```c
char *output;
if (opt_o)
  output = opt_o;
else if (opt_S)
  output = replace_extn(input_path, ".s");
else
  output = replace_extn(input_path, ".o");

// If -S is given, assembly text is the final output.
if (opt_S) {
  run_cc1(argc, argv, input_path, output);
  return 0;
}

// Otherwise, run the assembler to assemble our output.
char *tmpfile = create_tmpfile();
run_cc1(argc, argv, input_path, tmpfile);
assemble(tmpfile, output);
return 0;
```

The output filename comes from `-o` if given, otherwise from the input filename with the extension swapped. With `-S`, cc1 writes the `.s` directly to the output. Without `-S`, cc1 writes the `.s` to a tempfile, then the new `assemble` function runs `as`:

```c
static void assemble(char *input, char *output) {
  char *cmd[] = {"as", "-c", input, "-o", output, NULL};
  run_subprocess(cmd);
}
```

`as -c input -o output` is the GNU assembler invocation: assemble *input* into *output*, producing an ELF `.o` object file. The `-c` flag (curiously named the same as the GCC compile-only flag, but distinct in semantics) tells `as` to skip a startup-banner-style warning. `run_subprocess` is the same fork/exec/wait helper from §16.3.

`run_cc1` now takes input and output paths as parameters:

```c
static void run_cc1(int argc, char **argv, char *input, char *output) {
  char **args = calloc(argc + 10, sizeof(char *));
  memcpy(args, argv, argc * sizeof(char *));
  args[argc++] = "-cc1";

  if (input)
    args[argc++] = input;

  if (output) {
    args[argc++] = "-o";
    args[argc++] = output;
  }

  run_subprocess(args);
}
```

— so the driver can hand cc1 a different input/output than what the user typed at the command line. When cc1 writes to a tempfile, it doesn't see `-o tmpfile` from the user's argv; the driver constructs that argument list at `run_cc1` time.

The Makefile updates to match the new default output extension:

```diff
 test/%.exe: chibicc test/%.c
-	$(CC) -o- -E -P -C test/$*.c | ./chibicc -o test/$*.s -
-	$(CC) -o $@ test/$*.s -xc test/common
+	$(CC) -o- -E -P -C test/$*.c | ./chibicc -o test/$*.o -
+	$(CC) -o $@ test/$*.o -xc test/common
```

The test pipeline used to be: host preprocessor → chibicc (produces `.s`) → host cc (assembles + links). After this commit it's: host preprocessor → chibicc (produces `.o`, by internally running cc1 then `as`) → host cc (links). Chibicc has taken over the assembler role.

The `test/driver.sh` shell test gains coverage:

```bash
# -S
echo 'int main() {}' | $chibicc -S -o - - | grep -q 'main:'
check -S

# Default output file
rm -f $tmp/out.o $tmp/out.s
echo 'int main() {}' > $tmp/out.c
(cd $tmp; $OLDPWD/$chibicc out.c)
[ -f $tmp/out.o ]
check 'default output file'

(cd $tmp; $OLDPWD/$chibicc -S out.c)
[ -f $tmp/out.s ]
check 'default output file'
```

Three checks: `-S` produces assembly text on stdout; the default output for `.c` is `.o`; the default output with `-S` is `.s`.

### Where we are

The driver runs `as` automatically. `chibicc input.c` produces `input.o`; `chibicc -S input.c` produces `input.s`; `chibicc -o name input.c` produces `name`. Tempfiles are created by `mkstemp`, registered for cleanup at `atexit`, and unlinked when the process exits. The `StringArray` type is the driver's growing-list-of-strings data structure. The driver is now a real C compiler driver in miniature — it just doesn't yet handle multiple input files or linking.

---

## 16.5 — Multiple input files

> `git checkout b833cd0f297ba7979c23cff1b88c27beb4f2f737` — *Accept multiple input files*

Real compilers accept many input files at once: `cc foo.c bar.c baz.c`. After this commit, chibicc does too. The driver's argument parser collects all the non-flag arguments into an array and the main loop runs the cc1+as pipeline once per input.

The change is concentrated in `main.c`. The single `input_path` global is replaced with a `StringArray`:

```diff
-static char *input_path;
+static char *base_file;
+static char *output_file;
+
+static StringArray input_paths;
 static StringArray tmpfiles;
```

`input_paths` is the driver's collected list. `base_file` and `output_file` are the cc1-side replacements for `input_path` and `opt_o` — when running as cc1, the driver passes `-cc1-input <file>` and `-cc1-output <file>` to disambiguate which file is the cc1 input and where its assembly should go (since `argv` may also contain unrelated input files for *other* cc1 invocations).

The argument parser splits `-o` handling into two phases. A pre-pass validates that any flag taking an argument (`-o`) actually has one:

```c
static bool take_arg(char *arg) {
  return !strcmp(arg, "-o");
}

static void parse_args(int argc, char **argv) {
  // Make sure that all command line options that take an argument
  // have an argument.
  for (int i = 1; i < argc; i++)
    if (take_arg(argv[i]))
      if (!argv[++i])
        usage(1);
  // ... main loop ...
}
```

This pre-pass is purely defensive: `chibicc -o` (with no following argument) used to crash inside the main loop; now it errors cleanly via `usage(1)`. The main loop drops the redundant `if (!argv[++i])` and just consumes the argument:

```diff
     if (!strcmp(argv[i], "-o")) {
-      if (!argv[++i])
-        usage(1);
-      opt_o = argv[i];
+      opt_o = argv[++i];
       continue;
     }
```

The cc1-only flags `-cc1-input` and `-cc1-output` are introduced:

```c
if (!strcmp(argv[i], "-cc1-input")) {
  base_file = argv[++i];
  continue;
}

if (!strcmp(argv[i], "-cc1-output")) {
  output_file = argv[++i];
  continue;
}
```

These are the driver-to-cc1 communication channel for "compile *this* input to *this* output." The user-facing `-o` continues to mean what it has always meant. `-cc1-input` and `-cc1-output` are constructed by the driver inside `run_cc1`:

```c
if (input) {
  args[argc++] = "-cc1-input";
  args[argc++] = input;
}

if (output) {
  args[argc++] = "-cc1-output";
  args[argc++] = output;
}
```

The cc1 path reads from `base_file` (instead of the now-removed `input_path`) and writes to `output_file` (instead of `opt_o`):

```c
static void cc1(void) {
  // Tokenize and parse.
  Token *tok = tokenize_file(base_file);
  Obj *prog = parse(tok);

  // Traverse the AST to emit assembly.
  FILE *out = open_file(output_file);
  fprintf(out, ".file 1 \"%s\"\n", base_file);
  codegen(prog, out);
}
```

The driver's main loop wraps the previous single-input pipeline in a `for` loop:

```c
if (input_paths.len > 1 && opt_o)
  error("cannot specify '-o' with multiple files");

for (int i = 0; i < input_paths.len; i++) {
  char *input = input_paths.data[i];

  char *output;
  if (opt_o)
    output = opt_o;
  else if (opt_S)
    output = replace_extn(input, ".s");
  else
    output = replace_extn(input, ".o");

  // If -S is given, assembly text is the final output.
  if (opt_S) {
    run_cc1(argc, argv, input, output);
    continue;
  }

  // Otherwise, run the assembler to assemble our output.
  char *tmpfile = create_tmpfile();
  run_cc1(argc, argv, input, tmpfile);
  assemble(tmpfile, output);
}
```

The `-o` rule extends: `-o` is valid with one input or with the (yet-to-arrive) link path; it's not valid with multiple inputs in `-c` or `-S` modes (because each input produces its own output and they can't all share a name). The error message is the GCC-style diagnostic.

Each iteration creates its own tempfile (via `create_tmpfile`, which appends to the global `tmpfiles` registry), runs cc1 to write into it, runs `as` to assemble it. The tempfiles all get cleaned up at exit by the same `cleanup` handler.

The `test/driver.sh` test extends:

```bash
# Multiple input files
rm -f $tmp/foo.o $tmp/bar.o
echo 'int x;' > $tmp/foo.c
echo 'int y;' > $tmp/bar.c
(cd $tmp; $OLDPWD/$chibicc $tmp/foo.c $tmp/bar.c)
[ -f $tmp/foo.o ] && [ -f $tmp/bar.o ]
check 'multiple input files'

rm -f $tmp/foo.s $tmp/bar.s
echo 'int x;' > $tmp/foo.c
echo 'int y;' > $tmp/bar.c
(cd $tmp; $OLDPWD/$chibicc -S $tmp/foo.c $tmp/bar.c)
[ -f $tmp/foo.s ] && [ -f $tmp/bar.s ]
check 'multiple input files'
```

Two checks: with `-c`-style default behavior, two `.c` inputs produce two `.o` outputs; with `-S`, two `.c` inputs produce two `.s` outputs.

There's a subtle invariant to notice. The driver's main loop runs `cc1` and `as` in sequence per input, *not* in a parallelized fan-out. `chibicc foo.c bar.c` compiles foo, then bar, sequentially. A real driver might parallelize this (GCC's `-j` lives in `make`, not in the driver itself), but chibicc doesn't bother. The test pipeline gets the same result either way, just slightly slower.

### Where we are

The driver accepts multiple input files. Each one runs through its own cc1-and-as pipeline, with its own tempfile and its own output path. The `-o` flag is restricted to single-input-with-`-S`-or-`-c` use, because multiple outputs can't share a single name. The cc1-driver protocol uses `-cc1-input` and `-cc1-output` to communicate the per-invocation file paths, separate from the user-facing `-o` argument.

---

## 16.6 — Running the linker

> `git checkout 8b726b54893e11427533fcceb7206b97c25f50a6` — *Run "ld" unless -c is given*

The chapter's final commit teaches the driver to invoke `ld`. With this in place, `chibicc input.c` produces an executable named `a.out` (or whatever `-o` says); `chibicc -c input.c` stops at `.o` (the previous default); `chibicc input.c -S` stops at `.s`. The conventional GCC flag set — `-c` for compile-and-assemble-only, `-S` for compile-only, plain invocation for the full pipeline — is implemented.

The change is in three corners. A new `-c` flag, file-extension dispatch in the main loop, and the `run_linker` function with its system-path probing.

The flag:

```c
if (!strcmp(argv[i], "-c")) {
  opt_c = true;
  continue;
}
```

Boolean global, parses on `-c`, controls the main-loop's behavior at the bottom of the per-input pipeline.

The file-extension dispatch is the more interesting change. Before this commit, every input file was treated as a `.c` file. After, the driver looks at each input's extension and routes accordingly:

```c
// Handle .o
if (endswith(input, ".o")) {
  strarray_push(&ld_args, input);
  continue;
}

// Handle .s
if (endswith(input, ".s")) {
  if (!opt_S)
    assemble(input, output);
  continue;
}

// Handle .c
if (!endswith(input, ".c") && strcmp(input, "-"))
  error("unknown file extension: %s", input);
```

A `.o` file is added to the linker argument list directly (it's already an object). A `.s` file is run through `as` (unless `-S` is given, in which case `-S` is meaningless on a `.s` input and the driver does nothing for it). A `.c` file (or `-`, meaning stdin) takes the cc1+as pipeline. Anything else errors. The `endswith` helper is straightforward:

```c
static bool endswith(char *p, char *q) {
  int len1 = strlen(p);
  int len2 = strlen(q);
  return (len1 >= len2) && !strcmp(p + len1 - len2, q);
}
```

The full per-input dispatch now looks like:

```c
for (int i = 0; i < input_paths.len; i++) {
  char *input = input_paths.data[i];
  char *output = /* ... */;

  if (endswith(input, ".o")) { strarray_push(&ld_args, input); continue; }
  if (endswith(input, ".s")) { if (!opt_S) assemble(input, output); continue; }
  if (!endswith(input, ".c") && strcmp(input, "-"))
    error("unknown file extension: %s", input);

  // Just compile (-S)
  if (opt_S) {
    run_cc1(argc, argv, input, output);
    continue;
  }

  // Compile and assemble (-c)
  if (opt_c) {
    char *tmp = create_tmpfile();
    run_cc1(argc, argv, input, tmp);
    assemble(tmp, output);
    continue;
  }

  // Compile, assemble and link
  char *tmp1 = create_tmpfile();
  char *tmp2 = create_tmpfile();
  run_cc1(argc, argv, input, tmp1);
  assemble(tmp1, tmp2);
  strarray_push(&ld_args, tmp2);
}

if (ld_args.len > 0)
  run_linker(&ld_args, opt_o ? opt_o : "a.out");
```

Three modes per input, plus the link mode at the end. With `-S`, the per-input loop stops at the assembly file. With `-c`, it stops at the `.o` file. Without either, the per-input loop produces a `.o` *and* appends it to `ld_args`; after the loop, if `ld_args` has anything in it, `run_linker` runs once over the collected list.

`run_linker` is the chapter's longest function, and most of it is paths:

```c
static void run_linker(StringArray *inputs, char *output) {
  StringArray arr = {};

  strarray_push(&arr, "ld");
  strarray_push(&arr, "-o");
  strarray_push(&arr, output);
  strarray_push(&arr, "-m");
  strarray_push(&arr, "elf_x86_64");
  strarray_push(&arr, "-dynamic-linker");
  strarray_push(&arr, "/lib64/ld-linux-x86-64.so.2");

  char *libpath = find_libpath();
  char *gcc_libpath = find_gcc_libpath();

  strarray_push(&arr, format("%s/crt1.o", libpath));
  strarray_push(&arr, format("%s/crti.o", libpath));
  strarray_push(&arr, format("%s/crtbegin.o", gcc_libpath));
  strarray_push(&arr, format("-L%s", gcc_libpath));
  strarray_push(&arr, format("-L%s", libpath));
  strarray_push(&arr, format("-L%s/..", libpath));
  strarray_push(&arr, "-L/usr/lib64");
  strarray_push(&arr, "-L/lib64");
  strarray_push(&arr, "-L/usr/lib/x86_64-linux-gnu");
  strarray_push(&arr, "-L/usr/lib/x86_64-pc-linux-gnu");
  strarray_push(&arr, "-L/usr/lib/x86_64-redhat-linux");
  strarray_push(&arr, "-L/usr/lib");
  strarray_push(&arr, "-L/lib");

  for (int i = 0; i < inputs->len; i++)
    strarray_push(&arr, inputs->data[i]);

  strarray_push(&arr, "-lc");
  strarray_push(&arr, "-lgcc");
  strarray_push(&arr, "--as-needed");
  strarray_push(&arr, "-lgcc_s");
  strarray_push(&arr, "--no-as-needed");
  strarray_push(&arr, format("%s/crtend.o", gcc_libpath));
  strarray_push(&arr, format("%s/crtn.o", libpath));
  strarray_push(&arr, NULL);

  run_subprocess(arr.data);
}
```

This is what GCC's driver does, distilled. Read it as a piece of incantation rather than a piece of design — the link command for a Linux x86-64 dynamically-linked program is intrinsically long, and chibicc is constructing it from scratch.

The pieces, briefly. `-m elf_x86_64` tells `ld` to produce an ELF binary for x86-64. `-dynamic-linker /lib64/ld-linux-x86-64.so.2` embeds the path to the runtime loader in the executable's `PT_INTERP` segment — when the kernel runs the binary, it actually runs the loader, which then loads the binary and any shared libraries it depends on. `crt1.o`, `crti.o`, `crtbegin.o`, `crtend.o`, `crtn.o` are the C runtime startup files that supply `_start` (the actual entry point that calls `main`), the `.init`/`.fini` section glue, and various pre-`main` and post-`main` registration routines (constructors of file-scope objects in C++, for one). `-lc -lgcc` link against libc and libgcc. `--as-needed -lgcc_s --no-as-needed` is the GCC-traditional dance that links against `libgcc_s` only if any unresolved symbols actually need it.

The user's input `.o` files (and the tempfiles produced by the cc1+as pipeline for `.c` inputs) go *between* the startup files and the libraries. ELF link order matters: undefined symbols in earlier objects are resolved by definitions in later objects; `crt1.o`'s `_start` calls `main`, which the user's object files define; the user's object files reference `printf`, which `-lc` provides.

The `find_libpath` and `find_gcc_libpath` functions probe the system to find where the startup files actually live. Different distributions install them in different places, so the driver tries the conventional locations:

```c
static char *find_libpath(void) {
  if (file_exists("/usr/lib/x86_64-linux-gnu/crti.o"))
    return "/usr/lib/x86_64-linux-gnu";
  if (file_exists("/usr/lib64/crti.o"))
    return "/usr/lib64";
  error("library path is not found");
}

static char *find_gcc_libpath(void) {
  char *paths[] = {
    "/usr/lib/gcc/x86_64-linux-gnu/*/crtbegin.o",
    "/usr/lib/gcc/x86_64-pc-linux-gnu/*/crtbegin.o", // For Gentoo
    "/usr/lib/gcc/x86_64-redhat-linux/*/crtbegin.o", // For Fedora
  };

  for (int i = 0; i < sizeof(paths) / sizeof(*paths); i++) {
    char *path = find_file(paths[i]);
    if (path)
      return dirname(path);
  }

  error("gcc library path is not found");
}
```

`find_libpath` looks for `crti.o` in two conventional places. The first is the Debian/Ubuntu multiarch path (`x86_64-linux-gnu` is the architecture triplet); the second is the older Red Hat-ish `lib64` location. `find_gcc_libpath` is messier: GCC's libgcc lives in a versioned subdirectory of `/usr/lib/gcc/<triplet>/<version>/`, where `<version>` is a number that changes whenever GCC is upgraded. The function uses `glob(3)` to expand the wildcard:

```c
static char *find_file(char *pattern) {
  char *path = NULL;
  glob_t buf = {};
  glob(pattern, 0, NULL, &buf);
  if (buf.gl_pathc > 0)
    path = strdup(buf.gl_pathv[buf.gl_pathc - 1]);
  globfree(&buf);
  return path;
}
```

`glob` returns matching paths sorted; `[buf.gl_pathc - 1]` picks the last one, which under sort order is usually the highest-numbered version. `dirname` strips the `crtbegin.o` suffix to leave just the directory.

This is all *Linux-specific and brittle*. The path list reflects the distros chibicc was tested against (Debian/Ubuntu, Gentoo, Fedora). On a distro Rui hasn't seen, or on a non-Linux system, `find_gcc_libpath` errors out and the driver fails. There's no fallback, no autodetect-by-querying-`gcc -print-search-dirs`, no PATH-walking. The brittleness is on display, and Rui leaves it that way: the link-command-construction problem is, in practice, complicated enough that a small compiler accepts the trade-off. A real compiler driver delegates this to `gcc`'s own `-print-search-dirs` output or (on GCC's side) a configure-time-baked path list.

The Makefile updates to use `chibicc -c` for the test build (since the default invocation now links, and the test pipeline doesn't want a final link from chibicc):

```diff
 test/%.exe: chibicc test/%.c
-	$(CC) -o- -E -P -C test/$*.c | ./chibicc -o test/$*.o -
+	$(CC) -o- -E -P -C test/$*.c | ./chibicc -c -o test/$*.o -
 	$(CC) -o $@ test/$*.o -xc test/common
```

— and likewise for the stage-2 targets. The `-c` flag tells chibicc to stop at `.o`; the host `cc` continues to do the final link. The reason the test pipeline still uses host `cc` for the final link is that the test `.exe` files are linked together with `test/common` (a small testing helper that doesn't fit cleanly into chibicc's link path). Once the test harness is restructured (which won't happen in this chapter), chibicc's `ld` invocation could take over; for now, both paths coexist.

The driver tests grow:

```bash
# Run linker
rm -f $tmp/foo
echo 'int main() { return 0; }' | $chibicc -o $tmp/foo -
$tmp/foo
check linker

rm -f $tmp/foo
echo 'int bar(); int main() { return bar(); }' > $tmp/foo.c
echo 'int bar() { return 42; }' > $tmp/bar.c
$chibicc -o $tmp/foo $tmp/foo.c $tmp/bar.c
$tmp/foo
[ "$?" = 42 ]
check linker

# a.out
rm -f $tmp/a.out
echo 'int main() {}' > $tmp/foo.c
(cd $tmp; $OLDPWD/$chibicc foo.c)
[ -f $tmp/a.out ]
check a.out
```

Three checks. The first is a single-input link: chibicc compiles, assembles, and links `int main() { return 0; }`. The second is a multi-input link: `foo.c` and `bar.c` separately compiled, then linked together; `main` calls `bar`, which is defined in the other translation unit; the linker resolves the cross-translation-unit reference and the resulting binary returns 42. The third checks the default output name: with no `-o`, the default is `a.out` in the current directory.

### Where we are

`chibicc input.c` produces an executable. The driver dispatches on file extension: `.c` files take the full cc1+as pipeline; `.s` files take only the `as` pipeline; `.o` files skip directly to the linker. With `-S`, the pipeline stops at `.s`; with `-c`, it stops at `.o`; without either, it runs `ld` on all the produced and given object files together. The link command is hardcoded for Linux x86-64 with glibc and GCC's runtime files, with directory probing across the major distributions. It's brittle and Linux-specific — the cost of writing a compiler driver from scratch — and Rui accepts the brittleness as an artifact of the C runtime's own messiness.

---

## Where the chapter leaves us

Eight commits, six sections. The driver came together from a Makefile change, three commits on function pointers, the cc1-vs-driver split, and three commits that progressively taught the driver to run the assembler, dispatch on multiple inputs, and run the linker.

| Commit | Topic |
|---|---|
| `5d15431` | Stage-2 build. New Makefile targets `stage2/chibicc`, `test-stage2`, `test-all`. The `self.py` script does the C preprocessor's job by hand for the stage-2 source — strip `#include`, substitute `bool`/`true`/`false`/`NULL`, expand `va_start`. Stage-2 is the canary for self-hosting; self-hosting itself arrives in Chapter 17. |
| `d06a8ac` | Function pointers. `gen_addr` learns the GOT/PIC distinction for `is_definition`-vs-not functions. `load` learns `TY_FUNC` (skip-the-load, like `TY_ARRAY`). `funcall` is rewritten to take any expression as callee. `Node->funcname` field deleted. `call *%rax` replaces `call name`. |
| `c5953ba` | Function-decay in parameter context. Six-line `else if` in `func_params` mirroring the array-decay arm. `int x()` parameter silently rewrites to `int (*x)()`. |
| `53e8103` | Function-pointer arithmetic conversion. Five-line addition in `get_common_type`. Conditional-expression arms of function type get decayed to function pointer. Test: `(1 ? ret10 : (void *)0)()`. |
| `f3d9613` | Split cc1 from driver. Same binary, dispatched by `-cc1` flag. New `run_subprocess` helper (fork/execvp/wait). New `-###` debug flag dumps subprocess command lines. POSIX headers added: `sys/types.h`, `sys/wait.h`, `unistd.h`. |
| `140b433` | Run `as`. New `StringArray` type. New `create_tmpfile`/`cleanup` (atexit-registered). New `replace_extn`. Driver runs `as -c tmp.s -o output.o` after cc1. With `-S`, the assembler step is skipped. |
| `b833cd0` | Multiple input files. `input_path` becomes `input_paths` (StringArray). Pre-pass validates argument-taking flags. New `-cc1-input`/`-cc1-output` flags for driver→cc1 communication. Per-input main loop. `-o` restricted to single-input use. |
| `8b726b5` | Run `ld`. New `-c` flag. File-extension dispatch (`.c`, `.s`, `.o`, `-`). New `run_linker`, `find_libpath`, `find_gcc_libpath` (glob-based). The link command embeds Linux/glibc-specific paths probed across Debian, Gentoo, Fedora layouts. Default output `a.out`. |

Three structural moves carry forward.

The first is the *driver-vs-cc1 split*. Chibicc has two roles in one binary: the driver (which dispatches), and the cc1 (which compiles). They're distinguished by the `-cc1` flag, communicate via `-cc1-input` and `-cc1-output`, and run as separate processes (the driver fork-execs cc1 for each input). The shape mirrors GCC's. The cost — fork-exec round-trip per input — is small in absolute terms and pays for clean process isolation. As Chapter 17 lands the preprocessor, it could plausibly become a third role (`-cpp`?), or it could fold into cc1's existing pipeline; the chapter's split machinery leaves the choice open.

The second is the *function-pointer / function-decay arc*. It mirrors Chapter 6's array-decay arc structurally: a type kind exists, a parameter-context decay rule fires, and the `get_common_type` rule unifies decayed pointers with other pointer expressions. The mirror is exact in two of the three pieces (the parameter-context arms and the codegen `load` skip) and approximate in the third (`get_common_type` for arrays does the decay through a different code path, but the unification logic is parallel). One tracking note: the `gen_addr` GOT-vs-RIP-relative branch is the chapter's psABI conformance correction. Function calls into shared-library symbols (everything that wasn't `is_definition`) now go through the GOT, where before they would have used a relative `lea` that the linker couldn't resolve to a non-local address. This is a quiet bugfix; the existing tests didn't exercise it because the test pipeline kept everything in one translation unit linked by the host `cc`.

The third is the *full pipeline finally being end-to-end chibicc*. After §16.6, `chibicc input.c` produces `./a.out`, and that `a.out` runs. Through Chapters 1–15, the build pipeline was *chibicc plus host-cc-as-glue*: chibicc emitted assembly text, the host cc filled in the gaps. After Chapter 16, chibicc owns the full `.c → executable` story (modulo the missing preprocessor, which Chapter 17 supplies). The host `cc` continues to be invoked for the test harness's final link with `test/common`, but the user-facing `chibicc input.c` no longer needs `cc` at all.

The chapter doesn't add to the *canonicalization-at-parse-time* count (still eight). It doesn't add to the *pre-factor-before-feature* count (still seven; the cc1-vs-driver split is in spirit a refactor, but the split *is* the feature). It doesn't add a fifth namespace. It doesn't extend the cast table (function pointers don't need cast-table cells). It doesn't grow `VarAttr`. The `Initializer` tree is unchanged. The `Relocation` mechanism gets implicit new uses through `gen_addr`'s GOT path, but no new code in `Relocation` itself.

A few standing notes.

- The `is_definition` flag on `Obj` (Chapter 13 §13.1) finally has a second reader: `gen_addr`'s function-pointer path uses it to choose between `lea name(%rip)` and `mov name@GOTPCREL(%rip)`. Before Chapter 16, only the `extern` path read the flag.
- `Node->funcname` is deleted. Function calls now identify the callee by the `lhs` expression. Chapter 17's preprocessor doesn't touch function-pointer machinery, so this is the final shape.
- The `call *%rax` indirect-call sequence is uniform for all calls — direct named calls, indirect calls through pointers, calls through conditional expressions, calls through `&function`. The previous direct `call name` shape is gone. Small pessimization, large simplification.
- The `StringArray` type lives in `chibicc.h`. It's the driver's growing-list-of-strings carrier and is used four places after this chapter: `tmpfiles`, `input_paths`, `ld_args`, and inside `run_linker` for the link argument list.
- The `atexit(cleanup)` registration is the driver's tempfile-disposal mechanism. The `cleanup` function unlinks each tempfile in `tmpfiles`. This catches normal exits and `error()` exits; it doesn't catch hard kills.
- The `find_libpath`/`find_gcc_libpath` probing is Linux/glibc-specific. The hardcoded distro list (Debian, Gentoo, Fedora) is what the chibicc CI was tested against. On other distros, the driver errors out at link time. Errata candidate, lower priority — the alternative (querying `gcc -print-search-dirs` at runtime) would defeat the purpose of writing a driver from scratch.
- The `--help` text is unchanged from earlier chapters. It still says `chibicc [ -o <path> ] <file>`. The new flags (`-S`, `-c`, `-###`, `-cc1`, `-cc1-input`, `-cc1-output`) aren't documented. The driver's user interface is shaped like GCC's but its self-documentation isn't.
- The `-cc1-input` and `-cc1-output` protocol is *internal only*: a user typing those flags directly would shadow the driver's intent. The driver doesn't error on them — they're just ordinary flags from the parser's perspective — but they're not advertised.
- The cc1 binary takes its `.c` input via `-cc1-input` after this chapter. The `tokenize_file` path (which `cc1()` calls) accepts a single filename and reads it. It also accepts `-` for stdin, since `tokenize_file` was extended at some point to recognize `"-"`. Verify in source.
- The Makefile's `test/%.exe` target uses host `cc -E -P -C` to run the C preprocessor on test files. After Chapter 16, that's still the case — chibicc still has no preprocessor of its own. The `-c` switch is added in this chapter so chibicc stops at `.o` (because the host `cc` does the final link with `test/common`), but the `cc -E` preprocessor invocation is unchanged.
- The `-### foo.c` debug invocation prints the cc1 command line. After §16.4, it also prints the `as` command line; after §16.6, it also prints the `ld` command line. Useful for debugging the driver-shape changes; the chapter doesn't lean on it but it's there.
- The stage-2 Makefile target is the canary for self-hosting. After Chapter 16, stage-2 still uses `self.py`. After Chapter 17 §17.6 (per the chapter mapping forecast), it should produce a stage-2 chibicc that doesn't need `self.py` at all.
- The psABI conformance thread *does* tick up by one in §16.2: `gen_addr` learns to use `@GOTPCREL` for non-locally-defined functions, which is the psABI's required addressing mode for shared-library function references. The chapter's other commits don't touch the psABI thread. New count: nine.

Forward references for Chapter 17 (a preprocessor from scratch, commits 158–197):

- The C preprocessor is the single largest absent feature. Chapter 17 is the longest single arc in chibicc's history — about 40 commits — covering tokenization integration, `#include`, `#define` (object-like and function-like macros), `#if`/`#ifdef`/`#elif`/`#else`/`#endif` conditional compilation, `#error` and `#warning`, `#pragma` (mostly ignored), `#line`, predefined macros (`__FILE__`, `__LINE__`, `__DATE__`, `__TIME__`, `__STDC__`), `__has_include`, the `#` and `##` operators, and the macro-expansion algorithm with all its subtleties (the no-rescan-set, function-like macro recursion, variadic macros).
- Once the preprocessor lands, `self.py` retires. The stage-2 Makefile target stops using it; the test pipeline stops using `cc -E`; `chibicc input.c` (without external help) goes from C source to executable through chibicc's own pipeline, with chibicc's own preprocessor doing the `#include` expansion and `#define` substitution.
- The driver-vs-cc1 split shape will likely accommodate the preprocessor without a second binary — the preprocessor is a phase of cc1 (running before tokenization, or integrated with it), not a separate process. The cc1 path stays single-process for everything before assembly.
- The link path doesn't change in Chapter 17. The `find_libpath`/`find_gcc_libpath` probing is preprocessor-independent.

Eight commits, the chapter maps the structural shape of a real C compiler driver onto chibicc. The next chapter lands the largest remaining absence — the C preprocessor — which is the reason `self.py` exists, the reason the test pipeline still pipes through `cc -E`, and the reason chibicc still can't be honestly called a self-hosting compiler. Chapter 17 closes that gap.
