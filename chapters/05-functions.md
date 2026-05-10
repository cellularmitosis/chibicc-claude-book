# Chapter 5 — Functions

> Commits covered: `30a3992`, `964b1d2`, `6cb4220`, `aacc0cf`. Four commits, the last two separated from the first two by more than a year of upstream history but logically continuous: chibicc gains the ability to *call* functions, then to *define* them.

Up to the end of Chapter 4 every chibicc program has lived inside a single implicit function. The `main` that wraps every test was synthesized by codegen — three lines of prologue, the user's compound statement, three lines of epilogue, all coming out of one `codegen()` call that knew there was exactly one function to emit. There were no calls anywhere; there was no notion of an argument; the word "function" appeared in the source code only as the name of a struct (`Function`) that was really just "a body and its locals."

Chapter 5 ends that. By the last commit of the chapter, chibicc compiles programs like

```c
int main() { return fib(9); }
int fib(int x) { if (x<=1) return 1; return fib(x-1) + fib(x-2); }
```

and produces a working recursive Fibonacci. The compiler emits `call` instructions; the parser handles parameter lists; the codegen produces a separate `.globl` for every function in the input; and the test suite — which has lived inside `'{ ... }'` shells since Chapter 3 — gets rewritten so every test starts with `int main()`.

The four commits split cleanly into two pairs.

1. `30a3992` adds zero-arity calls. `f()` works, but only with no arguments.
2. `964b1d2` extends calls to up to six arguments by introducing the SysV argument-register convention.

Then a year-long gap in upstream history (Rui returns in September 2020), and:

3. `6cb4220` adds zero-arity function *definitions*. A program can now contain more than one function, and the parser stops auto-creating `main`.
4. `aacc0cf` adds parameters to definitions. The compiler is now self-sufficient for any single-file C program whose every function has at most six `int`-or-pointer parameters.

The chapter has one concept interlude, on the System V AMD64 calling convention, between §5.1 and §5.2. The Chapter 3 stack-frame interlude foreshadowed the 16-byte alignment rule and the existence of a calling convention; this one cashes in. Argument registers, caller- vs. callee-saved registers, and what "up to six arguments" really means all live there.

---

## 5.1 — Zero-arity function calls

> `git checkout 30a39926272a8341c52018654ca18d2c86ba662b` — *Support zero-arity function calls*

This is the smallest commit in the chapter. It's also the first one in chibicc's history that emits a `call` instruction. The diff is twenty-nine lines added across five files, and after it the test program

```c
{ return ret3(); }
```

compiles and runs, exiting with 3.

The catch is that chibicc isn't generating `ret3` itself. There is still no notion of a function definition; `ret3` is going to be linked in from somewhere else.

### A separately-compiled object file as the callee

The first chunk of the diff is in the test harness:

```diff
 #!/bin/bash
+cat <<EOF | gcc -xc -c -o tmp2.o -
+int ret3() { return 3; }
+int ret5() { return 5; }
+EOF
+
 assert() {
   expected="$1"
   input="$2"

   ./chibicc "$input" > tmp.s || exit
-  gcc -static -o tmp tmp.s
+  gcc -static -o tmp tmp.s tmp2.o
   ./tmp
   actual="$?"
```

Before the assertions run, `test.sh` builds `tmp2.o` by piping a snippet of C source into a real GCC. The snippet contains two trivial functions, `ret3` and `ret5`. Then every test compiled by chibicc is *linked* against `tmp2.o`. So when chibicc-emitted code writes `call ret3`, the linker finds `ret3` in the GCC-compiled object file and stitches the two together.

This is the moment chibicc starts to interoperate with the Unix toolchain at the call-and-return level. The assembler resolves register names and instruction encodings; the linker resolves symbol names. Chibicc only has to emit a `call <symbol>` instruction with the right symbol name, and the rest of the toolchain handles getting the bits there.

What it gives Rui is an *enormous* shortcut. Without it, supporting function calls would require also supporting function definitions, plus a way to put more than one function into a program, plus prologues and epilogues for the callees. By outsourcing the callee side to GCC, the commit can focus purely on the caller side: "what does an x86-64 `call` look like?"

### Two pieces of AST

The parser learns to recognize a function-call syntax:

```diff
-// primary = "(" expr ")" | ident | num
+// primary = "(" expr ")" | ident args? | num
+// args = "(" ")"
 static Node *primary(Token **rest, Token *tok) {
   ...
   if (tok->kind == TK_IDENT) {
+    // Function call
+    if (equal(tok->next, "(")) {
+      Node *node = new_node(ND_FUNCALL, tok);
+      node->funcname = strndup(tok->loc, tok->len);
+      *rest = skip(tok->next->next, ")");
+      return node;
+    }
+
+    // Variable
     Obj *var = find_var(tok);
     if (!var)
       error_tok(tok, "undefined variable");
```

The grammar for `primary` grows a single optional `args`. When the parser sees an identifier followed by `(`, it builds an `ND_FUNCALL` instead of an `ND_VAR`. Two tokens of lookahead is all this takes — `tok->next` is the next token in the linked list, and the parser is allowed to read arbitrarily far ahead because the token stream is just a list. (The "no input stream, no global state" comment block that landed in §4.2 is doing real work here.)

The new node kind:

```diff
   ND_BLOCK,     // { ... }
+  ND_FUNCALL,   // Function call
   ND_EXPR_STMT, // Expression statement
```

and one new field on `Node`:

```diff
   // Block
   Node *body;

+  // Function call
+  char *funcname;
+
   Obj *var;      // Used if kind == ND_VAR
```

`funcname` is just a `char *` — the identifier text, copied out of the token. There's no `Obj`, no symbol-table lookup, no function pointer, no signature. Recall how every variable used to be auto-created on first use; functions are still on that schedule. The parser doesn't ask whether `ret3` exists. It just records the name and lets the linker complain if the symbol is missing.

This permissiveness mirrors the variable-handling style chibicc used through Chapter 3, before declarations were mandatory. The reasoning is the same: until the language has a way to *declare* the thing, the compiler can't reasonably check for it. C, in real life, lets you call a function that hasn't been declared (and emits a warning); chibicc, at this commit, does the same thing minus the warning.

### Codegen: the world's smallest call sequence

```diff
   case ND_ASSIGN:
     ...
     return;
+  case ND_FUNCALL:
+    printf("  mov $0, %%rax\n");
+    printf("  call %s\n", node->funcname);
+    return;
   }
```

Two instructions. `call <name>` does the actual work — it pushes the return address onto the stack and jumps to the symbol — and the result conventionally arrives back in `%rax`, which is exactly where chibicc wants its expression results. So the call's output flows naturally into the surrounding expression context.

The `mov $0, %rax` is the curiosity. The function we're calling has no arguments and its return type is `int`; clearing `%rax` looks like dead code. It isn't, quite. The SysV AMD64 ABI says that when calling a *variadic* function (one declared with `...`), `%al` (the low byte of `%rax`) must hold the count of vector registers used to pass arguments. Functions like `printf` actually read `%al` on entry. Chibicc doesn't yet know which functions are variadic and which aren't, so it plays safe and zeroes `%rax` before *every* call. For non-variadic callees, the write is harmless; for variadic ones, it correctly says "zero vector arguments."

A reader hunting for an explanation of this `mov $0, %rax` won't find one in the chibicc source — there's no comment, and Rui doesn't address it in the README. We're calling it out because it'll keep appearing in front of every `call` for the rest of the book, and "this is here for variadics that aren't here yet" is the smallest explanation that makes sense of it.

### One-line type rule

`ND_FUNCALL` joins the list of node kinds whose result is `int`:

```diff
   case ND_LE:
   case ND_NUM:
+  case ND_FUNCALL:
     node->ty = ty_int;
     return;
```

There is no return-type tracking yet. Every function chibicc calls is implicitly `int`-returning. The C standard's "implicit int" rules from K&R C agree with this — undeclared functions in pre-C99 K&R were assumed to return `int` — and chibicc happens to be living in that world by accident, because it doesn't yet know how to declare functions at all.

### Test-suite addition

```diff
 assert 8 '{ int x=3, y=5; return x+y; }'

+assert 3 '{ return ret3(); }'
+assert 5 '{ return ret5(); }'
+
 echo OK
```

Two tests. They invoke each of the GCC-compiled helpers and check the result. From this commit on, every chibicc-compiled test will be linked against the GCC-compiled `tmp2.o`, even when the test doesn't call into it. That's fine — the linker just doesn't pull anything in if nothing references it.

### Where we are

Chibicc emits its first `call` instruction. The language has gained one new node kind, the parser learns one new piece of grammar, and the codegen for the call is two lines. But before we add arguments, we need vocabulary: what does the calling convention say about who puts what where, and why does it say six?

---

## Concept interlude — The System V AMD64 calling convention

The Chapter 3 stack-frame interlude introduced the call stack and the standard prologue and epilogue, then mentioned in passing that "the System V AMD64 ABI requires `%rsp` to be a multiple of 16 at the moment of a `call` instruction." That sentence didn't have to do any work in Chapter 3, because Chapter 3's chibicc never emitted a `call`. With §5.1 it does, and the rest of the chapter is going to lean on the rules we sketched.

A *calling convention* is the contract between the caller of a function and the callee. The contract specifies which registers carry arguments, which register carries the return value, who is allowed to clobber what, and what state the stack must be in at the moment of the call. The convention exists so that code from different sources — different compilers, hand-written assembly, the C library, the kernel — can call each other without each side having to know how the other side was compiled.

The convention chibicc targets is the one used by GCC, Clang, and the whole Linux/BSD/macOS userland on x86-64: System V AMD64. (The Windows-on-x86-64 convention is different, and chibicc doesn't support it.) The full ABI document is over a hundred pages; what follows is the slice that's relevant to the rest of this chapter and most of the book.

### The argument-passing registers

The first six integer-or-pointer arguments to a function are passed in registers, in this order:

| Position | Register |
|---|---|
| 1st | `%rdi` |
| 2nd | `%rsi` |
| 3rd | `%rdx` |
| 4th | `%rcx` |
| 5th | `%r8` |
| 6th | `%r9` |

These are the same six registers chibicc will declare in §5.2:

```c
static char *argreg[] = {"%rdi", "%rsi", "%rdx", "%rcx", "%r8", "%r9"};
```

The order is not alphabetical, not register-number order, and not obviously memorable. It's a historical accident, frozen into the ABI in 2003. Some mnemonics exist ("Diane's silk dress costs $89" — `D`, `S`, `D`, `C`, `8`, `9`), but the easier path is just to check the array.

Beyond six arguments, the convention is to push the rest onto the stack, with the seventh argument at the lowest address (closest to `%rsp`) at the moment of the call, and arguments going up in memory from there. *Chibicc does not implement this fallback.* A chibicc program with seven arguments is a program chibicc rejects — well, currently, mis-compiles — and the cap of six is baked into the language by the size of `argreg[]`. We'll see in §5.2 that chibicc loops over `argreg[i]` directly, so the seventh argument would silently scribble past the array; in §5.4 the same shape repeats on the callee side. The book follows chibicc's punt.

Floating-point arguments use a different set of registers (`%xmm0`–`%xmm7`), and structures big enough to need passing by reference get their own rules. None of that matters until Chapter 9 (structs) and Chapter 15 (floats) — for now, integers and pointers are the only types, and `argreg[0..5]` is the whole story.

### The return value

The integer return value lives in `%rax`. We've known this since Chapter 1 — `mov $42, %rax; ret` was how the very first chibicc program exited with status 42 — but it's worth re-stating now that we're emitting `call` instructions. When a function returns, its caller looks in `%rax` for the result. Chibicc's codegen has been writing every expression result into `%rax` since the beginning, which means any `call` instruction's "result" automatically lands in the slot the rest of the codegen expects.

Pointers return the same way (a pointer fits in `%rax`). Larger return values, structs returned by value, and floating-point returns each have their own rules. Chibicc again punts on all of those for many chapters.

### Caller-saved vs. callee-saved

The other half of the contract is about which registers can be safely clobbered across a call.

- **Caller-saved** (also called *volatile*): registers the caller doesn't expect to survive a function call. If the caller has a value in one of these and wants it preserved, it must save the value (typically by pushing it) before the call and restore it after. The callee is free to overwrite caller-saved registers without restoring them. The full caller-saved set in SysV AMD64 is `%rax`, `%rcx`, `%rdx`, `%rsi`, `%rdi`, `%r8`–`%r11`. That includes all six argument registers.
- **Callee-saved** (also called *non-volatile*): registers the caller expects to find unchanged after a call. If the callee wants to use one of these, it must save the original value (push) and restore it (pop) before returning. The callee-saved set is `%rbx`, `%rbp`, `%r12`–`%r15`, plus `%rsp` (which is conceptually saved by the prologue/epilogue dance).

Chibicc's relationship with this distinction is a study in benign neglect. It uses `%rbp` as the frame pointer and saves/restores it correctly in every prologue/epilogue, so it gets the callee-saved rule right for the one register it actually cares about. It uses `%rax` and `%rdi` heavily, and a few others occasionally, but only as scratch — never for values that need to live across a `call`. So the question of "save before, restore after" never arises. Effectively, chibicc treats every register as caller-saved by treating none of them as needing to live across a call, which is a defensible choice for a compiler whose idea of an expression is "evaluate it and put the result in `%rax` immediately." It's also a choice that real compilers can't get away with as soon as register allocation enters the picture — but register allocation isn't a chibicc feature.

The reader doesn't need to track which-register-is-which to follow the book. The relevant fact is: chibicc's argument registers (`%rdi`–`%r9`) are caller-saved, so when a function call returns, *any* value chibicc had stashed in them is gone. Chibicc's solution is to never stash anything there in the first place — argument values get pushed onto the stack, then popped into the argument registers immediately before the call.

### The 16-byte alignment rule

The single rule that's most likely to bite a from-scratch compiler is the alignment requirement. SysV AMD64 mandates that at the moment a `call` instruction executes, `%rsp` must be a multiple of 16. The reason is that some library functions (any that use SSE registers, which includes most of the math library and parts of the runtime) require 16-byte-aligned stack-allocated locals, and the simplest way to guarantee that is to have the alignment baked into the call itself.

A `call` instruction pushes an 8-byte return address as part of its work. So immediately *after* `call`, when control reaches the callee, `%rsp` is 8 bytes below a multiple of 16 — i.e. `%rsp ≡ 8 (mod 16)`. The callee's prologue then pushes `%rbp` (8 bytes), bringing `%rsp` back to a multiple of 16, and the callee can call other functions with `%rsp` still 16-aligned at the moment of those subsequent calls.

Chibicc enforces this rule in exactly one place: `assign_lvar_offsets` rounds the per-function `stack_size` up to a multiple of 16 (since Chapter 3, §3.3). That keeps the callee's locals area a 16-byte multiple, which means `sub $stack_size, %rsp` doesn't disturb the alignment. As long as we entered the function with `%rsp` aligned (which the caller's `call` guarantees), and we don't push or pop an odd number of 8-byte values between the entry and the next `call`, we're fine. Chibicc's `push`/`pop` pairs are always balanced (we push to evaluate, pop to consume), so this is true by construction.

There's one place where the balance could be wrong: pushing arguments before a call. If a function pushes seven 8-byte values onto the stack to set up a call, the stack pointer at the moment of the call is `8 × 7 = 56` below where it started — and 56 isn't a multiple of 16. This is precisely the "more than six arguments goes on the stack" case the ABI handles by rounding up the on-stack argument area. Chibicc never has more than six arguments on the stack waiting to go into registers — pushes happen during evaluation and pops happen immediately after — so the pushed-but-not-yet-popped count is always at most six. Six 8-byte values is 48 bytes, a multiple of 16. The alignment survives.

(A pedantic reader will notice that "at most six pushed" goes wrong in nested calls — `f(g(1,2,3,4,5,6))` could in principle have eleven 8-byte values stacked at some moment. In practice the inner `g` call pops its six before any of the outer `f`'s arguments get pushed — see §5.2 for the exact ordering — so this concern is theoretical. The deeper question of stack alignment under arbitrarily nested calls is something chibicc punts on; the right fix is to align before each call as needed, and chibicc will eventually do this in Chapter 13.)

### Where this leaves us

Three rules will govern the rest of the chapter:

1. **The first six args go in `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9`.** Chibicc supports up to six and rejects (or, more accurately, doesn't implement) more.
2. **The result comes back in `%rax`.** Every `ND_FUNCALL` will leave its result there, exactly where the surrounding expression codegen expects it.
3. **`%rsp` is 16-aligned at every `call`.** Chibicc enforces this by aligning each function's locals area, and by keeping argument-evaluation pushes and pops balanced.

Now we can put arguments into the picture.

---

## 5.2 — Function calls with up to six arguments

> `git checkout 964b1d2a0e3e46882743f16703cb12b51e724179` — *Support function call with up to 6 arguments*

This commit lifts the zero-arity restriction. After it, programs like

```c
{ return add6(1, 2, 3, 4, 5, 6); }
```

work, returning 21. So do recursive nests like `add6(1, 2, add6(3, 4, 5, 6, 7, 8), 9, 10, 11)` — each inner call evaluates its arguments, places them in argument registers, calls, and yields a value the outer call then treats as another argument.

The diff is fifty lines, mostly in the parser's new `funcall` helper.

### A node kind reused, with one more field

`ND_FUNCALL` already exists; this commit gives it an argument list:

```diff
   // Function call
   char *funcname;
+  Node *args;
```

`args` is a linked list of expression nodes — chibicc's standard "next pointer in the node itself" idiom, the same one `ND_BLOCK` uses for statement bodies. Each list element is one argument expression.

### The parser splits `primary`'s funcall branch into its own function

```diff
+// funcall = ident "(" (assign ("," assign)*)? ")"
+static Node *funcall(Token **rest, Token *tok) {
+  Token *start = tok;
+  tok = tok->next->next;
+
+  Node head = {};
+  Node *cur = &head;
+
+  while (!equal(tok, ")")) {
+    if (cur != &head)
+      tok = skip(tok, ",");
+    cur = cur->next = assign(&tok, tok);
+  }
+
+  *rest = skip(tok, ")");
+
+  Node *node = new_node(ND_FUNCALL, start);
+  node->funcname = strndup(start->loc, start->len);
+  node->args = head.next;
+  return node;
+}
```

The shape is the same head-and-cursor idiom we've now seen many times: a sentinel `head` keeps the list-construction code uniform, and the real list starts at `head.next`. The loop reads `assign`-level expressions separated by commas. `assign` is the right level — argument expressions can include almost anything (`add(x=3, y)` should work and does), but the comma between arguments is *not* the C comma operator, so we deliberately stop at `assign` rather than recursing into `expr` (which would consume the comma as an operator).

The grammar comment is worth pausing on. `funcall = ident "(" (assign ("," assign)*)? ")"` is the standard EBNF dance for "zero-or-more comma-separated items." The outer `()?` makes the whole argument list optional (zero arguments is allowed); inside it, `assign` followed by `("," assign)*` says "one expression, then zero or more comma-then-expression repeats." This shape will recur throughout chibicc's parser, including in §5.4 (function parameters), Chapter 9 (struct member initializers), and Chapter 12 (initializer lists).

`primary`'s call site shrinks to one call:

```diff
-    if (equal(tok->next, "(")) {
-      Node *node = new_node(ND_FUNCALL, tok);
-      node->funcname = strndup(tok->loc, tok->len);
-      *rest = skip(tok->next->next, ")");
-      return node;
-    }
+    if (equal(tok->next, "("))
+      return funcall(rest, tok);
```

The two-token lookahead — "is the next token after the identifier a `(`?" — stays in `primary`. Once that's confirmed, all of the actual parsing happens in `funcall`.

### Codegen: push, then pop

```diff
 static int depth;
+static char *argreg[] = {"%rdi", "%rsi", "%rdx", "%rcx", "%r8", "%r9"};

 static void gen_expr(Node *node);
 ...
-  case ND_FUNCALL:
+  case ND_FUNCALL: {
+    int nargs = 0;
+    for (Node *arg = node->args; arg; arg = arg->next) {
+      gen_expr(arg);
+      push();
+      nargs++;
+    }
+
+    for (int i = nargs - 1; i >= 0; i--)
+      pop(argreg[i]);
+
     printf("  mov $0, %%rax\n");
     printf("  call %s\n", node->funcname);
     return;
   }
+  }
```

The strategy is "evaluate everything, stash on the stack, then move into registers." Concretely:

1. Walk the argument list left-to-right. For each argument, generate its code (which leaves the result in `%rax`) and push `%rax` onto the stack. Count the arguments as we go.
2. After all arguments are evaluated, pop them off the stack in reverse order into the argument registers. Reverse order matters: the last argument was pushed last, so it's on top of the stack — popping goes top-to-bottom, which is right-to-left in argument order.
3. Zero `%rax` (the variadic-safety zero) and emit the `call`.

The choice to evaluate everything first and only then move into argument registers is dictated by the calling convention. The argument registers are caller-saved, but they're also where the *next* `call` will read its arguments from. If chibicc evaluated argument 1 directly into `%rdi` and *then* evaluated argument 2 (which involves another call, in the recursive case), argument 2's evaluation would clobber `%rdi`. By holding all argument values on the stack until the last possible moment, chibicc guarantees the registers are written in a single uninterrupted block right before the `call` — no other code runs between the moves and the call, so nothing can clobber.

This is also why the alignment math works out, as discussed in the interlude. At the moment of the last `pop`, the stack is restored to whatever depth it had before the argument evaluation began. The `call` instruction sees `%rsp` at the same alignment it was at the start of the funcall codegen. As long as the surrounding context had `%rsp` 16-aligned, the call does too.

### A subtle thing about evaluation order

Argument evaluation in C is *unspecified*. The standard says a compiler can evaluate `f(a(), b())` left-to-right, right-to-left, or in any order it likes — the only constraint is that each argument is fully evaluated before the call happens. Chibicc's choice to walk the linked list in order means it evaluates left-to-right, but a reader shouldn't take this as portable. Different compilers — and even different versions of the same compiler — have made different choices over the years. The book sticks to "chibicc evaluates left-to-right" as a fact about chibicc, not a fact about C.

### Test additions

```diff
 cat <<EOF | gcc -xc -c -o tmp2.o -
 int ret3() { return 3; }
 int ret5() { return 5; }
+int add(int x, int y) { return x+y; }
+int sub(int x, int y) { return x-y; }
+
+int add6(int a, int b, int c, int d, int e, int f) {
+  return a+b+c+d+e+f;
+}
 EOF
```

Three new GCC-compiled helpers join `ret3` and `ret5`. The chibicc-compiled tests:

```sh
assert 8 '{ return add(3, 5); }'
assert 2 '{ return sub(5, 3); }'
assert 21 '{ return add6(1,2,3,4,5,6); }'
assert 66 '{ return add6(1,2,add6(3,4,5,6,7,8),9,10,11); }'
assert 136 '{ return add6(1,2,add6(3,add6(4,5,6,7,8,9),10,11,12,13),14,15,16); }'
```

The two nested-`add6` tests are doing real work. They exercise the "evaluate and stash before moving into registers" discipline — the inner `add6` call needs to leave the outer call's already-evaluated arguments untouched. If chibicc had naively done "evaluate arg, move to register, evaluate next arg" the inner call's register usage would have stomped on the outer call's argument 1 before it ever got to `%rdi`. The fact that all three nested tests pass is the test suite confirming the stack-then-register strategy is implemented correctly.

### Where we are

Calling functions works. The compiler can express any expression that involves up to six-argument calls, including recursive nests of them. But every callee still has to be defined elsewhere — the test harness's `tmp2.o` is doing all of the actual function-definition work. The next two commits bring definitions in-house, starting with the simplest case: a function with no arguments.

---

## 5.3 — Zero-arity function definitions

> `git checkout 6cb4220f339e7d2a894e44b61c90c576a482914b` — *Support zero-arity function definition*

A year of upstream history passes between this commit and the previous one (August 2019 → September 2020 — Rui was working on his other compiler, [9cc](https://github.com/rui314/9cc), in the interim and a personal book on C compilers). When chibicc resumes, the immediate target is functions: a chibicc program can now contain more than one function, each with its own prologue, body, and epilogue.

The diff is the largest of the chapter — about 160 lines, with the test suite getting heavily rewritten on top — and three pieces are doing the work: the `Function` struct grows into a list, the parser learns to recognize function definitions instead of bare blocks, and codegen emits one prologue/body/epilogue per function in a top-level loop.

### `Function` becomes a list of functions

```diff
 // Function
 typedef struct Function Function;
 struct Function {
+  Function *next;
+  char *name;
   Node *body;
   Obj *locals;
   int stack_size;
 };
```

Two new fields. `next` makes `Function` a singly-linked list, the same idiom used by `Token`, `Node`, `Obj`, and `Type`. `name` is the function's identifier text. The other three fields — `body`, `locals`, `stack_size` — keep their meanings, now per-function instead of program-wide.

The `Type` struct also grows a `TY_FUNC` kind:

```diff
 typedef enum {
   TY_INT,
   TY_PTR,
+  TY_FUNC,
 } TypeKind;

 struct Type {
   TypeKind kind;
   ...
   Token *name;
+
+  // Function type
+  Type *return_ty;
 };
```

A function type has a return type. (Parameter types arrive in §5.4.) The `func_type` constructor is small:

```c
Type *func_type(Type *return_ty) {
  Type *ty = calloc(1, sizeof(Type));
  ty->kind = TY_FUNC;
  ty->return_ty = return_ty;
  return ty;
}
```

This is the third member of the `Type` family, joining `ty_int` and `pointer_to`. It's only used inside the parser, as a transient representation of a declarator; the rest of the compiler doesn't yet do anything with `TY_FUNC`. (Chapter 10 will, when nested declarators force the parser to distinguish "function returning int" from "pointer to function returning int.")

### `parse` returns a list, not a single function

```diff
-// program = stmt*
-Function *parse(Token *tok) {
+static Function *function(Token **rest, Token *tok) {
+  Type *ty = declspec(&tok, tok);
+  ty = declarator(&tok, tok, ty);
+
+  locals = NULL;
+
+  Function *fn = calloc(1, sizeof(Function));
+  fn->name = get_ident(ty->name);
+
   tok = skip(tok, "{");
+  fn->body = compound_stmt(rest, tok);
+  fn->locals = locals;
+  return fn;
+}
+
+// program = function-definition*
+Function *parse(Token *tok) {
+  Function head = {};
+  Function *cur = &head;

-  Function *prog = calloc(1, sizeof(Function));
-  prog->body = compound_stmt(&tok, tok);
-  prog->locals = locals;
-  return prog;
+  while (tok->kind != TK_EOF)
+    cur = cur->next = function(&tok, tok);
+  return head.next;
 }
```

The top-level grammar changes from `stmt*` to `function-definition*`. `parse` becomes a head-and-cursor loop: while there are more tokens, call `function` to consume one definition. The list-building pattern is the same one used everywhere else in the parser.

`function` itself is small. It calls `declspec` to consume the return-type keyword (`int`), then `declarator` to consume the function's name (and its `()`, via `type_suffix`). The `Type` it gets back is a `TY_FUNC` whose `name` field points at the identifier token. From that we extract the function name with `get_ident` (the helper from §4.4).

Then comes the body. `locals = NULL` resets the global locals list — recall from Chapter 3 that `locals` is a parser-global mutable list that `new_lvar` prepends to. With multiple functions in one compilation unit, each function needs its own list, and the simplest approach is to reset the list at the start of each function and snapshot it at the end. That's what these two lines do:

```c
locals = NULL;
...
fn->locals = locals;
```

The function definition consumes `{` (with `skip`), then parses a compound statement, then assigns the accumulated `locals` to the function's record. After the function returns, `parse`'s outer loop will re-enter `function` for the next definition, which will reset `locals` again.

There's a small wrinkle. The parser's global `locals` is a true global (declared `Obj *locals;` in `parse.c`), not a stack-allocated thing the parser passes around. Resetting it isn't symmetric with how the parser handles, say, the token cursor (which is threaded through every function). The asymmetry reflects an architectural choice: locals are a property of "the function being parsed right now," and rather than thread that context through every parsing function as a parameter, the parser uses a global as ambient state. This is the same trade-off you'd see in many recursive-descent parsers — purity vs. plumbing — and chibicc errs on the side of less plumbing.

### `type_suffix`: a placeholder for parameters

```diff
-// declarator = "*"* ident
+// type-suffix = ("(" func-params)?
+static Type *type_suffix(Token **rest, Token *tok, Type *ty) {
+  if (equal(tok, "(")) {
+    *rest = skip(tok->next, ")");
+    return func_type(ty);
+  }
+  *rest = tok;
+  return ty;
+}
+
+// declarator = "*"* ident type-suffix
 static Type *declarator(Token **rest, Token *tok, Type *ty) {
   while (consume(&tok, tok, "*"))
     ty = pointer_to(ty);

   if (tok->kind != TK_IDENT)
     error_tok(tok, "expected a variable name");
-
+  ty = type_suffix(rest, tok->next, ty);
   ty->name = tok;
-  *rest = tok->next;
   return ty;
 }
```

`declarator` no longer ends at the identifier. It calls `type_suffix`, which optionally consumes a `(` and `)` and wraps the type in `func_type`. So the same `declarator` that parses `int x` in a variable declaration now also parses `int main()` in a function definition — the `()` after the name turns `int` into "function returning int." The grammar comment `("(" func-params)?` reserves space for parameters, but the body skips straight to `)` for now.

There's an ordering subtlety. The pre-commit code wrote `ty->name = tok; *rest = tok->next;` — store name on the type, advance the cursor. The new code reverses these: it calls `type_suffix` first (which may consume `()` and re-set `*rest`), then writes `ty->name = tok`. The reason is that `type_suffix` may *replace* `ty` with `func_type(ty)` — when that happens, we want the *new* `ty` (the function type) to carry the name, not the inner return type. So the name assignment moves to after the wrap. The wrap-then-name order is what the C declarator syntax wants: `int main()` declares `main` as a function-returning-int, with `main` being the name of the function, not the name of the int.

Chibicc isn't yet at the point where this distinction stresses the parser — there are no nested declarators, no `int (*f)()` (a pointer to a function) versus `int *f()` (a function returning a pointer) ambiguity. Those arrive in Chapter 10. For now, every function definition has exactly one level of declarator, the `()` is right after the name, and the name-then-suffix parsing just works.

### `declaration` learns to forward-declare two helpers

```diff
+static Type *declspec(Token **rest, Token *tok);
+static Type *declarator(Token **rest, Token *tok, Type *ty);
 static Node *declaration(Token **rest, Token *tok);
```

Forward declarations at the top of `parse.c`. They're needed because `function` (defined near the bottom) calls `declspec` and `declarator` (which were `static`-and-defined later). The same housekeeping we did in §4.2 when `gen_expr` and `gen_addr` became mutually recursive.

### Codegen: prologue/body/epilogue per function

The most-rewritten file is `codegen.c`. The single-function `codegen` becomes a loop:

```diff
-void codegen(Function *prog) {
+void codegen(Function *prog) {
   assign_lvar_offsets(prog);
-
-  printf("  .globl main\n");
-  printf("main:\n");
-
-  // Prologue
-  printf("  push %%rbp\n");
-  printf("  mov %%rsp, %%rbp\n");
-  printf("  sub $%d, %%rsp\n", prog->stack_size);
-
-  gen_stmt(prog->body);
-  assert(depth == 0);
-
-  printf(".L.return:\n");
-  printf("  mov %%rbp, %%rsp\n");
-  printf("  pop %%rbp\n");
-  printf("  ret\n");
+
+  for (Function *fn = prog; fn; fn = fn->next) {
+    printf("  .globl %s\n", fn->name);
+    printf("%s:\n", fn->name);
+    current_fn = fn;
+
+    // Prologue
+    printf("  push %%rbp\n");
+    printf("  mov %%rsp, %%rbp\n");
+    printf("  sub $%d, %%rsp\n", fn->stack_size);
+
+    // Emit code
+    gen_stmt(fn->body);
+    assert(depth == 0);
+
+    // Epilogue
+    printf(".L.return.%s:\n", fn->name);
+    printf("  mov %%rbp, %%rsp\n");
+    printf("  pop %%rbp\n");
+    printf("  ret\n");
+  }
 }
```

The hard-coded `main` is gone. Each function emits its own `.globl` directive and label, runs the prologue with its own `stack_size`, walks its own body, and tears down with its own epilogue.

`current_fn` is new:

```diff
 static int depth;
 static char *argreg[] = {"%rdi", "%rsi", "%rdx", "%rcx", "%r8", "%r9"};
+static Function *current_fn;
```

It's a pointer to whichever function the codegen is currently emitting. The reason it has to exist is the `return` statement:

```diff
   case ND_RETURN:
     gen_expr(node->lhs);
-    printf("  jmp .L.return\n");
+    printf("  jmp .L.return.%s\n", current_fn->name);
     return;
```

Chapter 3's epilogue label was `.L.return` — fine when there's exactly one function in the program, terrible when there are several, because the assembler will reject multiple definitions of the same label. The fix is to make the label per-function: `.L.return.main`, `.L.return.fib`, etc. The `return` codegen needs to know which function it's inside, hence `current_fn`. The variable is set at the top of each iteration of the codegen loop, and `gen_stmt`'s `ND_RETURN` arm reads it.

This is the second instance of "ambient global state during codegen" we've seen, the first being `depth` (the stack-balance assertion counter). Both are solving the same shape of problem: information that's lexically scoped to one function but logically inherited by every recursive descent into the AST. Threading it through every function call would be tedious; a global suffices.

### Per-function locals offsets

`assign_lvar_offsets` becomes a loop:

```diff
 static void assign_lvar_offsets(Function *prog) {
-  int offset = 0;
-  for (Obj *var = prog->locals; var; var = var->next) {
-    offset += 8;
-    var->offset = -offset;
-  }
-  prog->stack_size = align_to(offset, 16);
+  for (Function *fn = prog; fn; fn = fn->next) {
+    int offset = 0;
+    for (Obj *var = fn->locals; var; var = var->next) {
+      offset += 8;
+      var->offset = -offset;
+    }
+    fn->stack_size = align_to(offset, 16);
+  }
 }
```

Each function's locals get their own offset assignment, and each function's `stack_size` is rounded to 16 independently. Locals from different functions share the same physical stack region in different time slices (each call uses the same offsets relative to its own `%rbp`), but conceptually each function has its own private numbering. The 16-byte alignment continues to hold per-function.

### The test suite gets `int main()`

This commit's test diff is enormous — every test that was previously written as `'{ ... }'` (for chibicc to wrap in an implicit `main`) is rewritten as `'int main() { ... }'` (now an explicit definition that chibicc has to parse). It's pure busywork; we won't reproduce the whole diff. The shape:

```diff
-assert 0 '{ return 0; }'
-assert 42 '{ return 42; }'
+assert 0 'int main() { return 0; }'
+assert 42 'int main() { return 42; }'
```

repeated for every test in the file. This is the moment chibicc's test programs become real C. They have a return type, a function name, parameter parens, and a body. They look like programs you might compile with GCC (and in fact, `int main() { return 0; }` will compile under both).

One new test comes in alongside the rewrites:

```sh
assert 32 'int main() { return ret32(); } int ret32() { return 32; }'
```

This is the first test where chibicc compiles *two* functions in the same input. `main` calls `ret32`, and `ret32` is defined right after. The fact that it works — that chibicc's `parse` loop handles both definitions, that `codegen`'s outer loop emits both, and that the linker finds the cross-reference — is the whole point of the commit.

### Where we are

A chibicc program is no longer "one function's worth of code we wrap in `main`." It's a list of function definitions, each compiled to its own labeled chunk of assembly, with cross-references resolved at link time. The compiler doesn't yet handle parameters, so all the user-defined functions are zero-arity, which is a sharp limitation — it means user-defined functions can talk to each other only via globals (which don't exist yet) or by calling externs (which puts you back in the GCC-helper-`tmp2.o` regime). The next commit fixes that.

---

## 5.4 — Function definitions with parameters

> `git checkout aacc0cfec24e0aef1e884ac8b657e182a33a7b1c` — *Support function definition up to 6 parameters*

Last commit of the chapter. After it:

```c
int main() { return fib(9); }
int fib(int x) {
  if (x<=1) return 1;
  return fib(x-1) + fib(x-2);
}
```

compiles, runs, and exits with 55. The diff is fifty lines, almost entirely in the parser, and mostly inside `type_suffix`.

### `type_suffix` grows a parameter list

```diff
-// type-suffix = ("(" func-params)?
+// type-suffix = ("(" func-params? ")")?
+// func-params = param ("," param)*
+// param       = declspec declarator
 static Type *type_suffix(Token **rest, Token *tok, Type *ty) {
   if (equal(tok, "(")) {
-    *rest = skip(tok->next, ")");
-    return func_type(ty);
+    tok = tok->next;
+
+    Type head = {};
+    Type *cur = &head;
+
+    while (!equal(tok, ")")) {
+      if (cur != &head)
+        tok = skip(tok, ",");
+      Type *basety = declspec(&tok, tok);
+      Type *ty = declarator(&tok, tok, basety);
+      cur = cur->next = copy_type(ty);
+    }
+
+    ty = func_type(ty);
+    ty->params = head.next;
+    *rest = tok->next;
+    return ty;
   }
+
   *rest = tok;
   return ty;
 }
```

The skeleton is the same one chibicc has used for every comma-separated list — sentinel head, cursor, loop until the closing token, expect commas after the first item. Each parameter is parsed by *reusing the same machinery as a top-level variable declaration*: `declspec` for `int`, then `declarator` for `*`s and a name. Parameters are full declarators, which means `int *p` and `int **q` work as parameter types just as they do as local types.

Two storage details:

```diff
 struct Type {
   ...
   // Function type
   Type *return_ty;
+  Type *params;
+  Type *next;
 };
```

`Type` gains a `params` field (the list head) and a `next` field (so types can be linked into a list). The list-via-`next` idiom now exists in three structs — `Node`, `Obj`, `Type` — for the same reason every time: cheap composition without allocating a separate cons cell.

The `copy_type` helper is also new:

```c
Type *copy_type(Type *ty) {
  Type *ret = calloc(1, sizeof(Type));
  *ret = *ty;
  return ret;
}
```

It's a one-liner that allocates a fresh `Type` and copies the bytes. The reason it's needed is that `declarator` returns a `Type *` whose lifetime and storage are determined by `pointer_to` and `func_type`'s allocations — *but* the type has a `next` field, and chaining types into the parameter list would corrupt the `next` of types that might be shared. Allocating a copy gives the parameter list its own private `next` chain, leaving the original types alone. (In practice, declarator-built types aren't shared — each `pointer_to` call makes a new one — but `copy_type` is conservative, and Chapter 10 will introduce situations where it actually matters.)

### Parameter `Obj`s for the function record

```diff
 struct Function {
   Function *next;
   char *name;
+  Obj *params;
+
   Node *body;
   Obj *locals;
   int stack_size;
 };
```

A function's parameters are kept in their own list, separate from the locals. The relationship is subtle: every parameter is *also* a local (it lives in the function's stack frame, gets a `%rbp`-relative offset, and is read like any other variable), but the function needs to know which subset of locals are parameters so codegen can move the argument-register values into the right slots on entry.

The `function` parser populates both lists from the same declarator-walk:

```diff
 static Function *function(Token **rest, Token *tok) {
   Type *ty = declspec(&tok, tok);
   ty = declarator(&tok, tok, ty);

   locals = NULL;

   Function *fn = calloc(1, sizeof(Function));
   fn->name = get_ident(ty->name);
+  create_param_lvars(ty->params);
+  fn->params = locals;

   tok = skip(tok, "{");
   fn->body = compound_stmt(rest, tok);
   fn->locals = locals;
   return fn;
 }
```

`locals` is reset to NULL, then `create_param_lvars` is called to register each parameter type as a new local. After it returns, `locals` holds exactly the parameters (in the right order — see below), and we snapshot it into `fn->params`. Then the body is parsed; `compound_stmt` may add more locals via `declaration`s; and at the end, `fn->locals` captures the full set, which now includes both the parameters (added first) and the body's declared variables (added later).

So `fn->params` is a *prefix* of `fn->locals`. The two lists share their tail. After this point, codegen iterates `fn->locals` to assign offsets (`assign_lvar_offsets`) and `fn->params` to emit register-to-stack moves. Because the parameter list is the prefix added first, and `assign_lvar_offsets` walks `locals` in order, the parameters end up at the lowest-address (most-negative) offsets in the frame, which is the natural place for them.

### `create_param_lvars`: the recursion-then-act trick

```c
static void create_param_lvars(Type *param) {
  if (param) {
    create_param_lvars(param->next);
    new_lvar(get_ident(param->name), param);
  }
}
```

This is a four-line function and worth pausing on. The recursion is *before* the action: `create_param_lvars` calls itself on the rest of the list, *then* registers the current parameter. The effect is to call `new_lvar` on the *last* parameter first, second-to-last second, and so on, with the *first* parameter being registered last.

Why? Because `new_lvar` prepends to the global `locals` list. If we'd called it left-to-right — first parameter first, then second, etc. — `locals` would end up in *reverse* order, with the last parameter at the head. By reversing the iteration via recursion, the head of `locals` after the recursion ends up being the *first* parameter, and traversing `fn->params` in order gives back the original parameter order.

There are other ways to do this. `create_param_lvars` could iterate left-to-right and append (which would require carrying a tail pointer), or it could iterate left-to-right and prepend then reverse the list at the end, or `new_lvar` could be parameterized with a "where to insert." Recursion-then-act is the shortest expression of "register these in the right order given that the registration primitive prepends." It's the kind of small, locally-elegant solution that's easy to miss on a first read.

### Codegen: register-to-stack on entry

```diff
     printf("  push %%rbp\n");
     printf("  mov %%rsp, %%rbp\n");
     printf("  sub $%d, %%rsp\n", fn->stack_size);

+    // Save passed-by-register arguments to the stack
+    int i = 0;
+    for (Obj *var = fn->params; var; var = var->next)
+      printf("  mov %s, %d(%%rbp)\n", argreg[i++], var->offset);
+
     // Emit code
     gen_stmt(fn->body);
```

Five lines. Right after the prologue, before the body runs, walk `fn->params` and emit one `mov` instruction per parameter, copying from `argreg[i]` into the parameter's stack slot. After this, the parameters look like ordinary locals — they have a stack slot, they have an offset stored in the `Obj`, and any `ND_VAR` reference to them generates the same `lea offset(%rbp), %rax; mov (%rax), %rax` as any other variable read.

This is the simplest possible parameter handling. A more sophisticated compiler might leave parameters in their argument registers and only spill the ones it can't keep — which is what register allocation buys you. Chibicc's choice is to spill them all immediately. The cost is six redundant memory writes for a six-argument function; the benefit is that the rest of the codegen doesn't need to know parameters exist. Once the moves are emitted, every variable lives in memory, and the same `gen_addr` and `gen_expr` logic that handled locals in Chapter 3 handles parameters now.

There's also a quiet bug in this codegen, by the strictest reading. The loop assumes that `fn->params` has at most six entries, because it indexes into `argreg[i]` without bounds-checking. If a chibicc user wrote `int f(int a, int b, int c, int d, int e, int f, int g) { ... }`, the seventh iteration would read past the end of the six-element array. The `type_suffix` parser doesn't check for the cap either — it'll happily collect ten parameter types into the list. The book's position on this matches the handoff's: chibicc punts on the "more than six arguments" case, and we don't refactor in the prose. A reader who tries it will get a compiler crash or silent miscompilation, and that's the state of the art for now.

### Tests: parameters and recursion

```diff
 assert 32 'int main() { return ret32(); } int ret32() { return 32; }'
+assert 7 'int main() { return add2(3,4); } int add2(int x, int y) { return x+y; }'
+assert 1 'int main() { return sub2(4,3); } int sub2(int x, int y) { return x-y; }'
+assert 55 'int main() { return fib(9); } int fib(int x) { if (x<=1) return 1; return fib(x-1) + fib(x-2); }'
```

The third test is the highlight. `fib(9)` is recursive — `fib` calls itself twice per non-base case — and the test confirms that the parameter and return-value plumbing all works under recursion. That implies several things: the argument register is correctly read and stored on entry; the body's reference to `x` correctly reads the stored value; the recursive `fib(x-1)` call correctly evaluates `x-1` (which involves loading `x` from the frame, subtracting one, and pushing onto the stack), then loads it into `%rdi` via the pop-loop, and calls. When the recursive call returns, `%rax` has the inner result, which the outer expression uses. The whole machinery from §5.2 (calls) and §5.4 (definitions and parameters) is exercised in one nine-character test case.

It also exercises the per-function `.L.return.fib` label, since `fib` has two `return` statements. Both jump to the same label (the label is per-function, not per-return); both reach the same epilogue; both pop `%rbp` and `ret`. The discriminator is `current_fn->name` at codegen time, and `fib` got its own unique label because it had its own iteration of the codegen loop.

### `add_type` learns about `args`

A small house-keeping change in `type.c`:

```diff
   for (Node *n = node->body; n; n = n->next)
     add_type(n);
+  for (Node *n = node->args; n; n = n->next)
+    add_type(n);
```

`add_type` now also walks the argument list of a function-call node, so each argument's `ty` gets filled in. None of the current uses care — codegen doesn't consult arg types, and the existing pointer-arithmetic logic in `new_add`/`new_sub` only walks operand types of `+` and `-` directly — but it makes the whole tree consistently typed, which keeps future commits from running into "this subtree has NULL types" bugs.

### Where we are

A chibicc program is now a list of full function definitions, each with up to six `int`-or-pointer parameters, a body of arbitrary statement complexity, and the ability to call any other function in the program (or any externally-linked function with a compatible signature). The recursive-`fib` test is a good summary of what the chapter built: every part of it — the call, the parameter passing, the return value, the recursion — uses machinery that didn't exist at the start of the chapter.

What's still missing is bigger than what's there. There are no global variables. There's no `char`, no string, no array. The type system has three kinds (`int`, `pointer`, `function`) and only one of them carries a size — implicitly, by hardcoded `8`. Function types exist but only inside the parser; they never make it out to be useful to codegen or the type-checker. The toolchain interoperability is real but one-directional — chibicc can call into GCC code, but a chibicc-compiled object file isn't yet something you'd link from somewhere else (it would be, but there's no `-c` flag yet, no `-o` flag — chibicc still emits to stdout).

The compiler is structurally further along than its language is. The next chapter — arrays — is going to lean hard on this fact. Arrays are not a function-shaped feature; they don't add new top-level forms or reshape the codegen loop. They poke at the type system, they poke at the parser's declarator (`int x[5]` is a new declarator suffix), and they introduce the first real `sizeof` requirement. Chapter 6 finishes off the type-system limitations the book has been signposting since §4.3 — the hardcoded `8` finally becomes `lhs->ty->base->size`, paid back as foreshadowing earned.

---

## Recap

| Commit | What it added |
|---|---|
| `30a3992` | `ND_FUNCALL` (zero-arity); `mov $0, %rax; call <name>`; GCC-compiled `tmp2.o` linked into tests |
| `964b1d2` | `args` list on `ND_FUNCALL`; `argreg[]`; push-then-pop strategy for arg-evaluation |
| `6cb4220` | `Function` becomes a list; per-function prologue/body/epilogue; `current_fn`; `.L.return.<name>`; tests rewritten to `int main()` |
| `aacc0cf` | Parameter parsing in `type_suffix`; `create_param_lvars`; spill-args-to-stack on entry; recursive `fib` works |

Four commits, three of which touch the parser substantially and two of which touch codegen. The center of gravity is the third — the function-definition commit, which is where chibicc stops being "one function's worth of code in disguise" and becomes a real C compiler in shape if not yet in coverage. The fourth commit is a smaller delta in lines but a much bigger one in expressive power, because it's the one that closes the loop: chibicc-defined functions can finally take arguments, and chibicc-compiled programs no longer need a GCC-built helper file to do anything interesting.

The SysV calling convention has now done two distinct jobs: it shaped the prologue/epilogue back in §3.2, and now it shapes the call sequence and the parameter-spill on entry. The rules from the interlude — six argument registers, return value in `%rax`, 16-byte alignment at call time — will keep mattering for the rest of the book, and most of the time they'll be invisible in the prose because chibicc's discipline of spilling early and keeping pushes balanced makes them automatic.

Chapter 6 turns to arrays. Five commits: 1-D arrays, multi-dimensional arrays, the `[]` operator, the `sizeof` operator, and a small refactor that finally merges `Function`'s name/locals/body trio with `Obj`. Arrays will share `Type`'s `base` field with pointers, which means the "is this a thing that points at something?" test from §4.3 keeps working. The hardcoded `8` in `new_add`/`new_sub` finally becomes a per-type lookup. And `sizeof` makes the "every value is 8 bytes" assumption visible to the source language for the first time, foreshadowing the later arrival of `char`.
