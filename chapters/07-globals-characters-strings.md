# Chapter 7 — Globals, characters, and strings

> Commits covered: `a4d3223`, `be38d63`, `4cedda2`, `35a0bcd`, `ad7749f`, `699d2b7`, `c2cc1d3`, `9dae234`, `d9ea597`, `7b8528f`, `a0388ba`, `6c0a429`. Twelve commits — the longest range so far. They cover global variables, the `char` type, string literals with their full escape vocabulary, statement expressions (a GNU extension), the move from `argv[1]` to file-and-stdin input, a `printf`-to-`println` refactor, the `-o` and `--help` driver flags, and source comments.

Chapter 6 ended with `Function` retired into `Obj` and the `.text` directive emitted but unused. The compiler had arrays, `sizeof`, the `[]` operator, and an internal data structure ready for a second kind of named object. What it didn't have was anything to put in that second slot, anything smaller than 8 bytes, any way to say "the letter `H`" without writing `72`, or any sense of itself as a *program* — it still consumed source code from `argv[1]` and wrote assembly to stdout, with no other entry point.

Chapter 7 turns those remaining gaps into features. It is the longest commit range so far — twelve commits, where Chapter 1 had only seven actual feature commits and the rest of the chapters have run between four and six. The commits are mostly small, but they touch every part of the compiler: the parser learns to look ahead and dispatch on whether a top-level identifier introduces a function or a global; the type system gains its second integer kind; the codegen splits its single argument-register table into one for 8-bit and one for 64-bit operands; the tokenizer learns four new lexical rules (string literals, three flavors of escape sequences, and comments); and the driver finally grows up — reading from a file, accepting an output flag, and printing a usage message when invoked with `--help`.

The chapter has one concept interlude, on **integer representation** — bytes and bits, signed versus unsigned, sign extension when widening, and what chibicc has chosen for its own integer model. It lands between §7.1 (where global variables make use of the size machinery from Chapter 6) and §7.2 (where `char` becomes the first chibicc type whose size is not 8).

The seven sections:

- **§7.1** — Global variables (commit 32).
- **§7.2** — The `char` type (commit 33).
- **§7.3** — String literals (commits 34, 35).
- **§7.4** — Escape sequences (commits 36, 37, 38).
- **§7.5** — Statement expressions (commit 39).
- **§7.6** — The driver grows up (commits 40, 41, 42).
- **§7.7** — Comments (commit 43).

Two of these — §7.3 and §7.6 — bundle multiple commits that go together. Each commit still gets its own `git checkout` opener inside its section, but the prose treats them as a unit.

The order Rui committed these things is not the order this chapter presents them. As with Chapter 6's `Function`/`Obj` merge, the chapter follows the *commit-list order in `chapter-mapping.md`*, which sorts the commits by their position on `main` after Rui's "rewrite history to keep every commit bug-free" passes. Some of these commits were originally written months apart — the comments commit (`6c0a429`) is dated August 2019, while the global-variables commit (`a4d3223`) is dated September 2020. The ordering here is the ordering Rui chose for the canonical history, which is what the book follows.

---

## 7.1 — Global variables

> `git checkout a4d3223a7215712b86076fad8aaf179d8f768b14` — *Add global variables*

The Chapter 6 finale set this up. `Function` had been folded into `Obj`, the parser kept a `globals` list alongside `locals`, `new_gvar` was waiting unused, and codegen filtered the `Obj` list with `if (!fn->is_function) continue;` — a filter that never skipped anything because the parser only produced functions. This commit fills in the other side of the filter. After it, programs like

```c
int x;
int main() { x = 3; return x; }
```

compile and produce 3. A global integer is declared at file scope, written from inside `main`, and read back through the function return. The compiler emits `x` into the `.data` section of the assembly output, references it via RIP-relative addressing, and `find_var` resolves the name by searching globals after locals.

Three things have to happen for this to work: the parser has to *decide* whether a top-level declaration introduces a function or a global, codegen has to learn a second kind of variable address, and the data section has to actually get emitted.

### Lookahead, finally

C's grammar at file scope is awkward. Both `int x;` and `int main() { ... }` start with `int` followed by an identifier, and the parser can't tell them apart from the prefix alone. It has to look at the token *after* the identifier — `(` means function, anything else means variable. Up to this commit, chibicc has avoided lookahead of this kind; declarations inside compound statements were unambiguous because `int` always introduced a `declaration` and never anything else. At file scope, that's no longer true.

Rui's approach is to run the declarator parser as a *probe*:

```c
// Lookahead tokens and returns true if a given token is a start
// of a function definition or declaration.
static bool is_function(Token *tok) {
  if (equal(tok, ";"))
    return false;

  Type dummy = {};
  Type *ty = declarator(&tok, tok, &dummy);
  return ty->kind == TY_FUNC;
}
```

`declarator` is called with a dummy base type and a *local copy* of the token cursor. C passes function arguments by value, so even though `declarator` advances `tok` by walking through `*`s, an identifier, and a type-suffix, the caller's `tok` is unchanged. The parser builds a throwaway `Type` chain, inspects the kind, and discards everything. If the result is `TY_FUNC`, the next thing in the source is a function definition; otherwise it's a global variable.

There's a comment near the top of `parse.c` from Chapter 2 that gets a quiet payoff here:

```c
// Input tokens are represented by a linked list. Unlike many recursive
// descent parsers, we don't have the notion of the "input token stream".
// Most parsing functions don't change the global state of the parser.
// So it is very easy to lookahead arbitrary number of tokens in this
// parser.
```

That comment was forward-looking when it was written. This commit is the first place chibicc cashes it in. Because every parser function takes its input cursor by value (or as a `Token **rest` out-parameter that the caller controls), arbitrary lookahead is literally a matter of "call a parsing function with a copy of the cursor and look at what it returned." No backtracking infrastructure, no token-stream snapshots, no memoization — the immutability of the linked list of tokens does all the work.

The dispatch in `parse` is then routine:

```diff
 Obj *parse(Token *tok) {
   globals = NULL;

   while (tok->kind != TK_EOF) {
     Type *basety = declspec(&tok, tok);
-    tok = function(tok, basety);
+
+    // Function
+    if (is_function(tok)) {
+      tok = function(tok, basety);
+      continue;
+    }
+
+    // Global variable
+    tok = global_variable(tok, basety);
+
   }
   return globals;
 }
```

`declspec` is consumed first (it has to be, to know whether we're parsing `int` or — in two commits — `char`), then `is_function` peeks ahead at the rest, and the dispatch picks one of two parsers. Both parsers take the consumed `basety` and return the updated cursor. Chapter 6's "function returns Token *, registers via side effect" shape pays off here: `global_variable` is the same shape, just registering with `new_gvar` instead of building a function body.

### `global_variable` and comma-separated declarators

```c
static Token *global_variable(Token *tok, Type *basety) {
  bool first = true;

  while (!consume(&tok, tok, ";")) {
    if (!first)
      tok = skip(tok, ",");
    first = false;

    Type *ty = declarator(&tok, tok, basety);
    new_gvar(get_ident(ty->name), ty);
  }
  return tok;
}
```

Twelve lines. The loop handles `int x, y, z;` by walking each comma-separated declarator and registering each one as a global with the shared base type. The `consume` helper (introduced back in Chapter 4) tries to match `;` and advances the cursor on success, returning `true` so the loop terminates cleanly on the trailing semicolon. Between declarators, `skip(tok, ",")` insists on the comma and errors if it isn't there.

The grammar in this commit allows arrays at file scope (`int x[4];`) because `declarator`'s call to `type_suffix` already handles `[N]`. The tests confirm:

```sh
assert 0 'int x[4]; int main() { x[0]=0; x[1]=1; x[2]=2; x[3]=3; return x[0]; }'
```

Arrays of 4 ints reserve 32 bytes of `.data` space, and the same array-decay rules from Chapter 6 work just as well for a global as for a local. There's nothing new to learn at the language level — only at the storage level.

### `find_var` finds globals too

```diff
 static Obj *find_var(Token *tok) {
   for (Obj *var = locals; var; var = var->next)
     if (strlen(var->name) == tok->len && !strncmp(tok->loc, var->name, tok->len))
       return var;
+
+  for (Obj *var = globals; var; var = var->next)
+    if (strlen(var->name) == tok->len && !strncmp(tok->loc, var->name, tok->len))
+      return var;
+
   return NULL;
 }
```

The fallback is significant. Locals shadow globals, by C's scoping rules: if a function declares `int x` locally, references to `x` inside the function bind to the local, not the global. Chibicc gets this for free by searching `locals` first. The two lists are independent, and the local list is rebuilt for each function (`locals = NULL;` at the start of `function`), so there's no chance of cross-function pollution.

The current implementation is linear in the number of variables — both lists are walked from head to end — and the comparison is byte-for-byte. For a program with one function and a handful of globals, this is fast enough. For a real C program with thousands of names, it would be a problem; Chapter 22 will eventually swap the linear search for a hash table. But chibicc's "slow algorithms are fine if `n` isn't too big" stance, established back in Chapter 1, applies here as elsewhere.

### Codegen: a second kind of address

```diff
 static void gen_addr(Node *node) {
   switch (node->kind) {
   case ND_VAR:
-    printf("  lea %d(%%rbp), %%rax\n", node->var->offset);
+    if (node->var->is_local) {
+      // Local variable
+      printf("  lea %d(%%rbp), %%rax\n", node->var->offset);
+    } else {
+      // Global variable
+      printf("  lea %s(%%rip), %%rax\n", node->var->name);
+    }
     return;
```

Every `ND_VAR` access now branches on `is_local` (the flag the Chapter 6 merge added to `Obj`). Locals are still RBP-relative — `lea -8(%%rbp), %%rax` puts the address of the slot into `%rax`. Globals are *RIP-relative*: `lea x(%%rip), %%rax` asks the assembler to encode an instruction whose displacement is computed at link time as the distance from the next instruction (the value of `%rip` after the `lea`) to the symbol `x`.

RIP-relative addressing is the modern x86-64 default for accessing globals. The reason is position-independent code: an executable that uses RIP-relative addressing for its data references can be loaded at any virtual address and still find its globals, because the offset from `%rip` to the global is fixed at link time and unaffected by where the loader puts the segment. The instruction is one byte longer than absolute addressing on average, and Apple's macOS toolchain effectively requires PIC for executables. Linux is more permissive but uses RIP-relative by convention. Chibicc matches the convention without comment.

What chibicc *doesn't* do is generate the `@GOTPCREL` form that real PIC code uses for globals defined in other translation units. A reference of the form `lea x(%rip), %rax` resolves to a direct displacement only if `x` is in the same compilation unit; cross-unit references in true PIC mode have to go through the global offset table. Chibicc compiles each input file as its own translation unit and links statically against `tmp2.o` (the test's helper), and the test programs declare every global they reference, so the simple form works. When Chapter 16 introduces multi-file compilation, this won't change — chibicc never grows GOT support — and the limitation will quietly persist, papered over by linking everything statically.

### `emit_data` and the second pass

The codegen loop splits in two:

```c
static void emit_data(Obj *prog) {
  for (Obj *var = prog; var; var = var->next) {
    if (var->is_function)
      continue;

    printf("  .data\n");
    printf("  .globl %s\n", var->name);
    printf("%s:\n", var->name);
    printf("  .zero %d\n", var->ty->size);
  }
}

static void emit_text(Obj *prog) {
  for (Obj *fn = prog; fn; fn = fn->next) {
    if (!fn->is_function)
      continue;
    ...
  }
}

void codegen(Obj *prog) {
  assign_lvar_offsets(prog);
  emit_data(prog);
  emit_text(prog);
}
```

`emit_data` walks the same `Obj` list, skips functions, and emits one `.data`/`.globl`/label/`.zero` four-line stanza per global. `.zero N` is the GAS directive for "reserve N bytes initialized to zero" — exactly the right semantics for a C global without an initializer, which the C standard requires to be zero-initialized.

The `.text` directive that landed in Chapter 6 §6.5 finally has a counterpart. Without explicit section directives, the assembler would default to whichever was last emitted; mixing globals and functions in source order would scramble the output into random-looking section transitions. Each `emit_data` iteration restates `.data`, each `emit_text` iteration restates `.text`. The `.globl` directive marks each symbol as visible to the linker — without it, the symbol would be local to the object file and other translation units couldn't reference it. Globals with internal linkage (`static int x;`) won't get `.globl`; that distinction arrives in Chapter 13.

Calling `assign_lvar_offsets` first, then `emit_data`, then `emit_text` is a small but real architectural choice. The frame allocator runs once for all functions before any output is emitted; data globals come out as a block; functions come out as a second block. In a more sophisticated compiler, you might emit globals interleaved with the functions that use them, or sort them by alignment for cache reasons. Chibicc doesn't bother. The two-pass shape is clean and reads top-down.

### Tests

```sh
assert 0 'int x; int main() { return x; }'
assert 3 'int x; int main() { x=3; return x; }'
assert 7 'int x; int y; int main() { x=3; y=4; return x+y; }'
assert 7 'int x, y; int main() { x=3; y=4; return x+y; }'
assert 0 'int x[4]; int main() { x[0]=0; x[1]=1; x[2]=2; x[3]=3; return x[0]; }'
...
assert 8 'int x; int main() { return sizeof(x); }'
assert 32 'int x[4]; int main() { return sizeof(x); }'
```

Ten new tests. The first checks default zero-initialization (the test passes because `.zero 8` emits eight zero bytes for `int x;`, and the function returns whatever those bytes contain). The next three exercise read-write and the comma-separated form. Four more confirm that array globals work — written from inside `main` via `x[i]=v`, read back via `x[i]`. The last two are `sizeof` checks confirming the size machinery from Chapter 6 transfers cleanly to globals.

### Where we are

File-scope variables exist. The parser dispatches between functions and globals by lookahead; codegen branches between RBP-relative and RIP-relative addressing on the `is_local` flag; the data section emerges as `.zero` blocks. Comma-separated declarations work. The shadowing rule falls out of the order `find_var` checks the two lists. Everything Chapter 6's `Obj` merge prepared now has a use.

What chibicc still doesn't have at this point is more than one integer type, more than 8 bytes per integer, or any way to write a non-numeric character literal. Before we add `char`, we should pause and look at what an integer actually is at the bit level — because the next commit is the first place chibicc deals with types of more than one width, and the codegen tricks for that are short but assume a working knowledge of how integers are represented.

---

## Concept interlude — Integer representation

Up to this commit, every integer in chibicc is 8 bytes. The `int` type's `size` field is 8; pointers are 8; `array_of(int, 3)` is 24. The type system has been *aware* of size since Chapter 6, but the only number it ever stored was 8. Chibicc was using a 64-bit `int` not by choice but by accident — there was no other size to use.

The next commit introduces `char`, whose size is 1. That single change forces three things: the codegen has to know how to load a 1-byte value into a 64-bit register (a question the hardware answers in three different ways depending on signedness and zero-extension), the parameter-passing path has to know which width of register to use, and the type system has to start tracking which arithmetic operations promote and how. Chibicc handles the first two in `be38d63` and largely punts on the third until Chapter 10. To follow the codegen, we need a few facts about how integers live in memory and registers.

### Bits, bytes, and widths

A byte is 8 bits. An x86-64 general-purpose register is 64 bits. The instruction set lets you address each register at four widths: 8 bits (`%al`), 16 bits (`%ax`), 32 bits (`%eax`), and 64 bits (`%rax`). The four names refer to the same physical hardware location, with the smaller names referring to the low-order bits of the larger ones. A write to `%al` modifies only the low 8 bits of `%rax`; a write to `%eax` writes the low 32 bits and (by an x86-64 convention) zeroes the upper 32. Writes to `%al` and `%ax` leave the higher bits untouched.

Memory works the same way at the byte level: every memory address points at one byte. Loading multi-byte values is a question of *how many adjacent bytes* the load instruction reads. `mov (%rax), %rax` reads 8 bytes; `movb (%rax), %al` reads 1 byte; `movw (%rax), %ax` reads 2 bytes. The byte-order question (which byte goes where in the register) is answered by the architecture: x86-64 is little-endian, so the lowest address holds the least significant byte.

When a value is *narrower* than the register holding it, the question of what to put in the unused upper bits has two answers: zero them, or replicate the value's high bit. The choice depends on whether the value is signed.

### Two's complement, sign extension, and the `movsx` family

Chibicc's `int` and `char` are signed integers represented in two's complement. The C standard didn't formally require two's complement until C23, but every compiler that has ever mattered uses it, and chibicc inherits the choice from its host (since the literal `-1` has whatever bits the host's `-1` has, and that's two's complement on every platform anyone runs chibicc on).

The relevant property of two's complement, for codegen, is that a signed value's high bit is its *sign bit*: 0 for non-negative, 1 for negative. To widen a signed value from 8 bits to 64 bits without changing what it represents, you have to copy the sign bit into all the new high bits. A signed byte `0xFF` is `-1` and has to widen to the 64-bit value `0xFFFFFFFFFFFFFFFF` (which is also `-1`, eight bytes wide). A signed byte `0x7F` is `127` and widens to `0x000000000000007F`.

x86-64 has a one-instruction encoding of this rule, called `movsx` ("move with sign extension"). `movsbq (%rax), %rax` reads one byte from the address in `%rax`, sign-extends it to 64 bits, and stores the result back in `%rax`. The mnemonic decomposes as `mov` + `s` (sign-extend) + `b` (source is byte) + `q` (destination is quadword, 8 bytes). The next commit's `load` function for 1-byte types is exactly this one instruction.

For *unsigned* values, the rule is different — the high bits are zeroed instead of replicated. The instruction is `movzx` (zero-extend), and chibicc doesn't use it in this chapter because chibicc has no unsigned types yet. Unsigned arrives in Chapter 14. Until then, every integer chibicc widens is signed, and `movsx` is the only widening instruction the codegen knows.

There's a related question of how to *store* a narrower value to memory. The store side has no widening question: `mov %al, (%rdi)` writes one byte from the low end of `%rax` to the address in `%rdi`, and the upper bytes of `%rax` are simply ignored. Whatever sign-extension state `%rax` holds doesn't matter for the store; only the low byte makes it to memory. The next commit's `store` function for 1-byte types is `mov %al, (%rdi)` for the same reason.

### Chibicc's choices

Chibicc has made some choices that distinguish it from C as you'd write it for a desktop x86-64 compiler. Until Chapter 10, `int` is **8 bytes**. Real C on x86-64 uses 4 bytes for `int` and 8 bytes for `long`. Rui starts with 8-byte `int` because it lets the codegen treat every variable as a full register, simplifying the early chapters; the change to 32-bit `int` in Chapter 10 is what forces the introduction of `long`, the four-width register tables, and the proper integer-conversion rules. Holding off lets each conversion-rule complication land in the chapter where there's a concrete reason for it.

After this chapter, `char` is 1 byte. That part matches C. There is no `short`, no `long`, no `long long`, and no unsigned variants. There are no integer suffixes like `123L`. Numeric literals are always typed as `int`. There is no integer promotion in arithmetic — `char + char` doesn't promote to `int` the way C requires; it stays at whatever the operands' widths suggest. (In practice, the codegen always works in 64-bit registers regardless of operand type, so the absence of explicit promotion rules just means the codegen is always-promoted-by-default and the type system doesn't bother modeling it.)

The full integer system — 32-bit `int`, the other widths, signedness, the conversion ranks, the promotion rules — arrives in Chapter 10 (commits 56–75). The single addition this chapter brings is `char`, which is the smallest possible step from "one integer width" to "two." It exposes exactly the codegen issues a multi-width type system has to solve, in a setting small enough to read in one sitting.

### What this interlude is and isn't

The point of this interlude isn't to teach two's complement from first principles. The book's audience is competent programmers who have probably seen `0xFF == -1`, who know that `int main()` returns a 32-bit value in real C, and who have read about little-endian byte order. The point is to fix vocabulary before §7.2 starts emitting `movsbq` and `mov %al, (%rdi)` without comment.

If those instructions read as opaque, the chapter's prose explanation will help; if they read as immediately obvious, the explanation will feel redundant. Either is fine. The §7.2 commit is the first place chibicc has to think about *width*, and width is one of the corners of C where the compiler's job is most directly tied to the hardware it generates code for.

---

## 7.2 — The `char` type

> `git checkout be38d63d1b9cd236ef3ec884eedad8112bb6e6f9` — *Add char type*

After this commit, `char` is a keyword, `char x;` reserves one byte in the frame, and `char` parameters are passed in 8-bit registers. Twenty-eight lines of changes across five files, plus a six-test addition. Most of the work happens in codegen, where loading and storing 1-byte values turns out to need three instruction-selection branches: one in `load`, one in `store`, and one in the function prologue.

### The type itself

```diff
 typedef enum {
+  TY_CHAR,
   TY_INT,
   TY_PTR,
   TY_FUNC,
   TY_ARRAY,
 } TypeKind;
```

```diff
+extern Type *ty_char;
 extern Type *ty_int;
```

```diff
+Type *ty_char = &(Type){TY_CHAR, 1};
 Type *ty_int = &(Type){TY_INT, 8};
```

```diff
 bool is_integer(Type *ty) {
-  return ty->kind == TY_INT;
+  return ty->kind == TY_CHAR || ty->kind == TY_INT;
 }
```

The pattern matches Chapter 6's introduction of `TY_ARRAY`: a new `TypeKind`, a new singleton constructed by compound literal in `type.c`, an `extern` declaration in the header. The size field — added back in Chapter 6 §6.1 in anticipation of exactly this moment — is `1` for `char`, the only difference from `ty_int` being the kind tag and the byte count. `is_integer` widens its check to admit the new kind, which matters in `add_type` for any context where pointer arithmetic, comparisons, or the `is_integer` guard around `ND_DEREF`'s base check has to accept either width.

### `declspec` and `is_typename`

```diff
-// declspec = "int"
+// declspec = "char" | "int"
 static Type *declspec(Token **rest, Token *tok) {
+  if (equal(tok, "char")) {
+    *rest = tok->next;
+    return ty_char;
+  }
+
   *rest = skip(tok, "int");
   return ty_int;
 }
```

The grammar comment grows one alternative. `declspec` is the function `parse` calls before deciding function-or-global, and `function` calls before parsing parameter types, and `declaration` calls before parsing local declarations. All four of those callers now transparently support both keywords because the dispatching is centralized.

A small new helper appears alongside:

```c
// Returns true if a given token represents a type.
static bool is_typename(Token *tok) {
  return equal(tok, "char") || equal(tok, "int");
}
```

```diff
   while (!equal(tok, "}")) {
-    if (equal(tok, "int"))
+    if (is_typename(tok))
       cur = cur->next = declaration(&tok, tok);
     else
       cur = cur->next = stmt(&tok, tok);
```

The `compound_stmt` parser used to peek at the literal token `"int"` to decide whether the next thing was a declaration. With two type keywords, that peek has to expand. `is_typename` factors the question out into one place — every future "is this token starting a declaration?" check will go through this function. By Chapter 14, `is_typename` will have a long body listing every type keyword and `typedef`-named alias. Putting the helper in this commit, with two keywords in the body, is foresight: the check has been generalized as soon as it has more than one possibility, instead of being inlined again and refactored later.

The keyword also has to be added to the tokenizer's `is_keyword` list:

```diff
   static char *kw[] = {
-    "return", "if", "else", "for", "while", "int", "sizeof",
+    "return", "if", "else", "for", "while", "int", "sizeof", "char",
   };
```

— the same one-entry change every keyword commit makes. After this, `convert_keywords` reclassifies any identifier matching `"char"` as `TK_KEYWORD`, and `equal(tok, "char")` works in the parser.

### Codegen splits its register tables

```diff
-static char *argreg[] = {"%rdi", "%rsi", "%rdx", "%rcx", "%r8", "%r9"};
+static char *argreg8[] = {"%dil", "%sil", "%dl", "%cl", "%r8b", "%r9b"};
+static char *argreg64[] = {"%rdi", "%rsi", "%rdx", "%rcx", "%r8", "%r9"};
```

Two tables. Chapter 5's standing assumption — "the argument-register array uses 64-bit names" — breaks here. `argreg64` is the old `argreg` renamed. `argreg8` lists the 8-bit alias of each register: `%rdi`'s low 8 bits are `%dil` ("D-I-L"), `%rsi`'s are `%sil`, and so on. The `%r8` family uses the `b` suffix for "byte" because the names `r8`-`r15` were added by AMD64 and don't have the historical low-byte mnemonics that `%rax`/`%rbx`/`%rcx`/`%rdx` do.

This is the second place in the codebase where chibicc has to choose between names for the same physical register based on operand width. The first will be `load` and `store`, immediately below.

### `load` learns sign extension

```diff
 static void load(Type *ty) {
   if (ty->kind == TY_ARRAY) {
     ...
     return;
   }

-  printf("  mov (%%rax), %%rax\n");
+  if (ty->size == 1)
+    printf("  movsbq (%%rax), %%rax\n");
+  else
+    printf("  mov (%%rax), %%rax\n");
 }
```

Two paths. For an 8-byte value, the existing `mov` is correct: read 8 bytes from `(%rax)` into `%rax`. For a 1-byte value, `movsbq` reads one byte and sign-extends it to 64 bits in the destination register. After the load, `%rax` holds the value in its low byte and the upper 56 bits are sign-extension of bit 7. From this point on, the value behaves as a normal 64-bit integer — chibicc's arithmetic, comparisons, and stores all work on 64-bit registers, and the sign-extension is the conversion that lets a `char` participate in 64-bit operations.

The choice to sign-extend rather than zero-extend is the choice to treat `char` as signed. C leaves the signedness of plain `char` implementation-defined; chibicc picks signed because `is_integer` returns true for both `TY_CHAR` and `TY_INT` and the rest of the codegen treats integers as signed (e.g., `idiv` for division, the `setl`/`setle` opcodes for comparison). A char value of `0xFF` will be loaded as `-1`. A char value of `0x7F` will be loaded as `127`. Real C on x86-64 also defines `char` as signed, by convention; chibicc agrees by accident-of-choice.

The branch is keyed on `ty->size == 1` rather than on `ty->kind == TY_CHAR`. This is forward-looking: when Chapter 10 adds `short` (2 bytes) and the redefinition of `int` as 4 bytes, the codegen will need branches for sizes 2 and 4 too, and the size-based dispatch will generalize cleanly. `kind`-based dispatch wouldn't.

### `store` learns to write a single byte

```diff
-static void store(void) {
+static void store(Type *ty) {
   pop("%rdi");
-  printf("  mov %%rax, (%%rdi)\n");
+
+  if (ty->size == 1)
+    printf("  mov %%al, (%%rdi)\n");
+  else
+    printf("  mov %%rax, (%%rdi)\n");
 }
```

`store` now takes a `Type *` parameter so it can match the size of the value being stored. For 8 bytes, the existing `mov %rax, (%rdi)` writes all 8 bytes of `%rax` to memory. For 1 byte, `mov %al, (%rdi)` writes only the low byte. The upper bits of `%rax` are ignored — this is the asymmetry from the interlude: writing a narrow value doesn't care about the high bits, but reading a narrow value has to decide what to do with them.

The single caller — `case ND_ASSIGN:` in `gen_expr` — passes `node->ty`, which was set by `add_type`. For a `char x = e;` declaration, the assignment's type is `char`, so size is 1, so we get `mov %al, (%rdi)`. For `int x = e;`, it's `mov %rax, (%rdi)`. The codegen reads the type the parser inferred and selects the right instruction.

### The function prologue learns the same trick

```diff
     // Save passed-by-register arguments to the stack
     int i = 0;
-    for (Obj *var = fn->params; var; var = var->next)
-      printf("  mov %s, %d(%%rbp)\n", argreg[i++], var->offset);
+    for (Obj *var = fn->params; var; var = var->next) {
+      if (var->ty->size == 1)
+        printf("  mov %s, %d(%%rbp)\n", argreg8[i++], var->offset);
+      else
+        printf("  mov %s, %d(%%rbp)\n", argreg64[i++], var->offset);
+    }
```

Same shape, third copy. When a function with `char` parameters is called, the SysV ABI says the byte values live in the low end of the argument registers — the value of the first `char` parameter is in `%dil`, not `%rdi`. The prologue has to use the right name to copy that byte to the parameter's stack slot. The `argreg8`/`argreg64` tables exist for exactly this dispatch.

The call-site path, by contrast, doesn't change:

```diff
     for (int i = nargs - 1; i >= 0; i--)
-      pop(argreg[i]);
+      pop(argreg64[i]);
```

When chibicc *makes* a call, it always pops 8-byte values into the 64-bit argument registers. The reason this works: chibicc evaluates argument expressions into 64-bit `%rax` (with sign extension applied at load time, if the operand was a `char`) and pushes them onto the stack as 8-byte slots. By the time the values are popped into argument registers, they're already 64-bit-wide signed integers, and the receiving function's prologue uses the 8-bit names to copy them back to byte-sized slots. Real C on x86-64 promotes arguments more carefully — narrow types get promoted to `int` at the call site, with explicit sign- or zero-extension instructions — but chibicc's path of "everything is 64 bits in transit" arrives at the same place for signed types.

### Tests

```sh
assert 1 'int main() { char x=1; return x; }'
assert 1 'int main() { char x=1; char y=2; return x; }'
assert 2 'int main() { char x=1; char y=2; return y; }'

assert 1 'int main() { char x; return sizeof(x); }'
assert 10 'int main() { char x[10]; return sizeof(x); }'
assert 1 'int main() { return sub_char(7, 3, 3); } int sub_char(char a, char b, char c) { return a-b-c; }'
```

Six tests. The first three check that two adjacent `char` declarations don't step on each other — `x` and `y` get distinct frame slots, and the `assign_lvar_offsets` machinery from Chapter 6 puts them at offsets one byte apart (well, one byte for `x`, one more for `y`, so `y` ends up at an offset four less than `x` would have if both were ints). The next two check `sizeof`: a single `char` is 1, an array of 10 is 10. The last is the parameter-passing test: a function with three `char` parameters returns their alternating-sign sum, which exercises the 8-bit prologue copy and the call-site setup.

There's a subtle detail in the `sub_char` test: the call `sub_char(7, 3, 3)` passes integer literals (which are typed `int` in chibicc) to `char` parameters. Chibicc doesn't check or convert; the values 7, 3, 3 are pushed as 8-byte slots, popped into `%rdi`/`%rsi`/`%rdx`, and the prologue then copies the low bytes (`%dil`/`%sil`/`%dl`) to byte-sized stack slots. The high bytes of the registers at the time of the prologue copy are whatever they were after the pops — non-zero in general, but ignored. The byte that gets stored is the low byte, which holds the original literal value. The test passes because chibicc's lack of conversion happens to match what a C programmer would expect for small positive values that fit in a byte.

### Where we are

`char` exists. The compiler has two integer types whose sizes differ; the codegen selects 8-bit vs 64-bit register names and `movsbq` vs `mov` based on operand size; the parameter-passing convention works for both widths. `is_typename` is centralized for future expansion. The size-based dispatch in `load`, `store`, and the prologue is the shape that will generalize when `short` and 4-byte `int` arrive in Chapter 10.

The next commit puts the new `char` type to work in a way that exposes another piece of compiler machinery — the data section gets its first non-zero contents, in the form of string literals.

---

## 7.3 — String literals

> `git checkout 4cedda2dbeca6bd81d2bd00032f7cff46e0a985e` — *Add string literal*

After this commit, `"abc"` is a valid expression of type `char[4]` (three characters plus a NUL terminator), and the bytes of the string get emitted into `.data` under a synthesized name. Forty-five lines added across the four core files.

The implementation is the first time chibicc emits something with a name it invented itself, rather than a name the source code provided. This is conceptually significant — the data section has, until now, contained only globals declared by the user, and codegen has only ever named symbols using identifiers from the AST. String literals introduce *anonymous globals*: storage that the source program needs but didn't name, allocated at file scope and addressed via a compiler-chosen label.

### The lexer's new string-literal token

```diff
 typedef enum {
   TK_IDENT,   // Identifiers
   TK_PUNCT,   // Punctuators
   TK_KEYWORD, // Keywords
+  TK_STR,     // String literals
   TK_NUM,     // Numeric literals
   TK_EOF,     // End-of-file markers
 } TokenKind;
```

```diff
 struct Token {
   ...
   int val;        // If kind is TK_NUM, its value
   char *loc;      // Token location
   int len;        // Token length
+  Type *ty;       // Used if TK_STR
+  char *str;      // String literal contents including terminating '\0'
 };
```

`TK_STR` is a fifth token kind, alongside the four that have existed since Chapter 1. The token carries two pieces of payload that no other token kind needs: the array type (`char[N]`, where N is the length including the NUL) and the string's bytes (a heap-allocated buffer with a trailing NUL that the assembler will emit as `.byte`s). Storing the type in the token feels slightly out of place — types are usually built by the parser, not the tokenizer — but for string literals the type is determined by the literal's length, and computing the length is part of lexing the token. Pushing the work to the parser would mean either re-walking the literal or threading the length through some other channel; embedding the type in `Token` is the most direct option.

The tokenizer's main loop gains one branch:

```c
// String literal
if (*p == '"') {
  cur = cur->next = read_string_literal(p);
  p += cur->len;
  continue;
}
```

— and the supporting reader is small:

```c
static Token *read_string_literal(char *start) {
  char *p = start + 1;
  for (; *p != '"'; p++)
    if (*p == '\n' || *p == '\0')
      error_at(start, "unclosed string literal");

  Token *tok = new_token(TK_STR, start, p + 1);
  tok->ty = array_of(ty_char, p - start);
  tok->str = strndup(start + 1, p - start - 1);
  return tok;
}
```

Walk forward from the opening quote until you hit a closing quote. If the line ends or the input ends before that, raise an error pointing at the opening quote. Once you find the closing quote, the token spans `[start, p+1)` — including both quote characters. The type is `array_of(ty_char, p - start)`: `p - start` counts the closing quote (which won't be in the data) but excludes the implicit NUL terminator (which will be in the data); the two cancel and the array length comes out right at "number of characters between the quotes, plus one for NUL." `strndup` copies `[start+1, p)` — just the contents, not the quotes — and the trailing NUL is added because `strndup` always NUL-terminates its output.

This early version has no escape sequences. A backslash in the source is just a literal backslash character in the string. The next three commits will fix that.

### The parser's primary case

```diff
-// primary = "(" expr ")" | "sizeof" unary | ident func-args? | num
+// primary = "(" expr ")" | "sizeof" unary | ident func-args? | str | num
 static Node *primary(Token **rest, Token *tok) {
   ...
+  if (tok->kind == TK_STR) {
+    Obj *var = new_string_literal(tok->str, tok->ty);
+    *rest = tok->next;
+    return new_var_node(var, tok);
+  }
```

A string literal in expression context creates an anonymous global, registers it in the `globals` list, and returns an `ND_VAR` node referencing it. From this point on, the string literal *is* a variable as far as codegen is concerned. The same `ND_VAR` codegen path that emits `lea x(%rip), %rax` for a user-declared global emits `lea .L..0(%rip), %rax` for a string literal — the only difference is the name.

The desugaring is small but consequential. By turning `"abc"` into `ND_VAR(<anonymous global>)`, the parser plugs string literals into every existing path that handles arrays. `"abc"[0]` works because the postfix subscript operator from Chapter 6 doesn't care that the array came from a literal. `sizeof("abc")` is 4 because the `Type` constructed in the tokenizer says so. The string literal acquires all the array-decay behavior from Chapter 6 for free.

### Anonymous globals

```c
static char *new_unique_name(void) {
  static int id = 0;
  char *buf = calloc(1, 20);
  sprintf(buf, ".L..%d", id++);
  return buf;
}

static Obj *new_anon_gvar(Type *ty) {
  return new_gvar(new_unique_name(), ty);
}

static Obj *new_string_literal(char *p, Type *ty) {
  Obj *var = new_anon_gvar(ty);
  var->init_data = p;
  return var;
}
```

Three new helpers, layered. `new_unique_name` returns a fresh symbol name like `.L..0`, `.L..1`, etc. The `.L` prefix is GAS-specific: any symbol whose name starts with `.L` is treated as *local* to the assembly file and not emitted into the symbol table of the resulting object file. This is the convention compilers use for symbols that exist only to make assembly self-referential — branch labels, jump targets, anonymous string literals — without leaking into linker namespace pollution. The `..` between `.L` and the digit is just chibicc's house style; GCC uses `.LC` for the same purpose.

`new_anon_gvar` wraps `new_gvar` with a generated name. `new_string_literal` wraps `new_anon_gvar` with the string contents stored in `init_data`. The distinct functions read like over-decomposition for a single caller, but the layered shape is forward-compatible: Chapter 12's compound-literal handling will use `new_anon_gvar` for non-string anonymous data, and Chapter 17's preprocessor will use `new_unique_name` for synthesized identifiers in macros that reference `__COUNTER__`. The three-layer ladder means each later use case can grab the layer it needs without re-implementing the others.

### `init_data` and the data emitter

```diff
 struct Obj {
   ...
   bool is_function;

+  // Global variable
+  char *init_data;
+
   // Function
   ...
 };
```

```diff
     printf("  .data\n");
     printf("  .globl %s\n", var->name);
     printf("%s:\n", var->name);
-    printf("  .zero %d\n", var->ty->size);
+
+    if (var->init_data) {
+      for (int i = 0; i < var->ty->size; i++)
+        printf("  .byte %d\n", var->init_data[i]);
+    } else {
+      printf("  .zero %d\n", var->ty->size);
+    }
   }
```

A `char *init_data` field on `Obj`, NULL if the variable should be zero-initialized, a heap buffer if it should be initialized with specific bytes. The data emitter branches on it: emit `.byte N` per source byte, or fall through to `.zero` if no init data was given. Each `.byte` directive emits one byte of data with the integer value given; for `"abc"` (four bytes: `'a'`, `'b'`, `'c'`, `'\0'`), the assembler sees four `.byte` lines emitting 97, 98, 99, 0.

Emitting one `.byte` per source byte is the simplest possible representation. `.ascii "abc"` would have been one line; `.string "abc"` would have implicitly added the NUL. Chibicc avoids both for a reason that will become clear in the next commit — once strings can contain arbitrary bytes (including embedded zeros from `\0` escapes), `.ascii` and `.string` get awkward, while the `.byte` loop continues to work uniformly. Doing it the bulky way from the start avoids a retrofit.

### Tests

```sh
assert 0 'int main() { return ""[0]; }'
assert 1 'int main() { return sizeof(""); }'

assert 97 'int main() { return "abc"[0]; }'
assert 98 'int main() { return "abc"[1]; }'
assert 99 'int main() { return "abc"[2]; }'
assert 0 'int main() { return "abc"[3]; }'
assert 4 'int main() { return sizeof("abc"); }'
```

Seven tests. The first two check the empty string: `""` has length 1 (just the NUL) and `""[0]` is 0. The next four check that `"abc"`'s four bytes are `'a'`, `'b'`, `'c'`, `'\0'` — values 97, 98, 99, 0 respectively. The last confirms `sizeof` measures the array including the terminator.

The subscript syntax `"abc"[0]` is doing real work here. It's the postfix operator from Chapter 6 §6.3 applied to a string literal — under the hood, `"abc"[0]` parses as `*("abc" + 0)`, the literal decays to a pointer (because `load(TY_ARRAY)` skips the load), the addition by 0 is a no-op pointer-wise, the dereference reads one byte from the address of the literal's first character, and the value comes back sign-extended in `%rax`. Every Chapter 6 mechanism participates in evaluating this six-character expression.

### Where we are

The compiler emits string literals to `.data` under generated `.L..N` labels and consumes them as anonymous global arrays of `char`. All of Chapter 6's array machinery transfers without modification. The data section now holds non-zero contents, which the assembler turns into actual bytes in the resulting object file's data segment. The pieces are in place for non-trivial programs that print to stdout — although chibicc still doesn't have anything resembling a `printf` declaration or a way to call into the C library beyond the existing `tmp2.o` test helper.

What strings can't yet contain is any character that doesn't appear in the source: tabs (you'd have to literally type a tab inside the quotes, which works but is fragile), newlines (which the tokenizer rejects as "unclosed string literal"), the NUL byte, or non-ASCII characters. The C language has a vocabulary of *escape sequences* for exactly these cases. The next three commits add them.

But before the escape sequences land, one tiny refactor commit that's worth mentioning.

> `git checkout 35a0bcd366163168bf3337975130f62fc1c30235` — *Refactoring: Add a utility function*

Twenty-line commit, not even a new feature. A new file `strings.c` is created, containing one function:

```c
// Takes a printf-style format string and returns a formatted string.
char *format(char *fmt, ...) {
  char *buf;
  size_t buflen;
  FILE *out = open_memstream(&buf, &buflen);

  va_list ap;
  va_start(ap, fmt);
  vfprintf(out, fmt, ap);
  va_end(ap);
  fclose(out);
  return buf;
}
```

`format` is `sprintf` that returns the buffer it allocated. Its only caller in this commit is `new_unique_name`, which collapses to one line:

```diff
 static char *new_unique_name(void) {
   static int id = 0;
-  char *buf = calloc(1, 20);
-  sprintf(buf, ".L..%d", id++);
-  return buf;
+  return format(".L..%d", id++);
 }
```

The arithmetic of "20 bytes is enough for `.L..%d` for any reasonable counter value" was always slightly suspect; `format` removes the question by computing the right size automatically (`open_memstream` grows the buffer as `vfprintf` writes into it). The commit is small enough to almost not deserve a section, but it deserves one line of acknowledgment: `format` becomes a workhorse helper used dozens of times in later chapters, especially in the preprocessor and the codegen for label generation. Putting it in its own file (`strings.c`) is forward-looking — by Chapter 8, `strings.c` will pick up more string-manipulation helpers and start to feel like a small utility library.

This refactor is the smallest commit in the chapter and arguably in the book. It's worth showing because it'll be referenced ("`format` was introduced in Chapter 7") more often than its size suggests.

---

## 7.4 — Escape sequences

> `git checkout ad7749f2fad87a4b1df644d4e1c345b3f87d386d` — *Add `\a`, `\b`, `\t`, `\n`, `\v`, `\f`, `\r`, and `\e`*

Three commits in sequence add the three flavors of C escape sequences: the named ones (`\n`, `\t`, etc.), the octal ones (`\101` = `'A'`), and the hex ones (`\x41` = `'A'`). Each commit is small individually; together they bring chibicc to feature parity with C's full escape vocabulary. The implementation lives entirely in the tokenizer — the parser and codegen don't know or care that escapes exist; they just see a byte buffer of whatever length the tokenizer produced.

This section walks all three commits in order, then closes with a thought on how the implementation reflects an interesting fact about bootstrapping.

### Named escapes

```diff
+static int read_escaped_char(char *p) {
+  switch (*p) {
+  case 'a': return '\a';
+  case 'b': return '\b';
+  case 't': return '\t';
+  case 'n': return '\n';
+  case 'v': return '\v';
+  case 'f': return '\f';
+  case 'r': return '\r';
+  // [GNU] \e for the ASCII escape character is a GNU C extension.
+  case 'e': return 27;
+  default: return *p;
+  }
+}
```

A flat switch over the character following the backslash. Eight cases for the standard named escapes (the source comment lists them in the commit message), one case for `\e` (a GNU extension, the ASCII escape character, octal `033`, decimal `27`), and a default that returns the literal character — so `\j` becomes `'j'`, `\\` becomes `'\\'`, and so on.

The tokenizer's string reader rewrites to call this:

```c
static Token *read_string_literal(char *start) {
  char *end = string_literal_end(start + 1);
  char *buf = calloc(1, end - start);
  int len = 0;

  for (char *p = start + 1; p < end;) {
    if (*p == '\\') {
      buf[len++] = read_escaped_char(p + 1);
      p += 2;
    } else {
      buf[len++] = *p++;
    }
  }

  Token *tok = new_token(TK_STR, start, end + 1);
  tok->ty = array_of(ty_char, len + 1);
  tok->str = buf;
  return tok;
}
```

Two-pass shape: first find the closing quote, then walk again building the unescaped buffer. The closing-quote search has to know about backslashes too, because a `\"` inside the string shouldn't terminate it. A small `string_literal_end` helper handles that:

```c
// Find a closing double-quote.
static char *string_literal_end(char *p) {
  char *start = p;
  for (; *p != '"'; p++) {
    if (*p == '\n' || *p == '\0')
      error_at(start, "unclosed string literal");
    if (*p == '\\')
      p++;
  }
  return p;
}
```

When we see a backslash, advance one extra character so we skip whatever follows. This handles `\"` correctly (the `"` is consumed as the escape character) and incidentally handles `\\` (the second `\` is consumed as the escape character). It does not validate that the escape sequence is well-formed; the second pass through `read_escaped_char` does that.

The `default: return *p;` path is what handles `\\` in the second pass — when we see `\\`, the next character is `\`, and the default case returns `*p` which is `\`. So `"\\"` produces a one-character string containing a single backslash. The same default handles `\?` (returns `?`) and `\"` (returns `"`).

The C standard says specifying an escape sequence not in the standard list is undefined behavior, with a footnote that compilers are encouraged to issue a diagnostic. Chibicc takes the most permissive interpretation: silently accept anything and pass through the literal character. The tests confirm this:

```sh
assert 106 'int main() { return "\j"[0]; }'
assert 107 'int main() { return "\k"[0]; }'
assert 108 'int main() { return "\l"[0]; }'
```

`'j'` is 106, `'k'` is 107, `'l'` is 108. No diagnostic, no error — chibicc trusts the programmer to know what they wrote.

The interesting comment block in `read_escaped_char` is worth quoting in full:

> Escape sequences are defined using themselves here. E.g. `'\n'` is implemented using `'\n'`. This tautological definition works because the compiler that compiles our compiler knows what `'\n'` actually is. In other words, we "inherit" the ASCII code of `'\n'` from the compiler that compiles our compiler, so we don't have to teach the actual code here.
>
> This fact has huge implications not only for the correctness of the compiler but also for the security of the generated code. For more info, read "Reflections on Trusting Trust" by Ken Thompson.

This is the only place in chibicc where Rui pauses mid-source to recommend external reading, and the recommendation is pointed: Ken Thompson's 1984 Turing Award lecture, which explores how a compiler that recognizes its own bootstrap source can plant a backdoor that survives source-level inspection. The argument applies directly here. Chibicc doesn't say anywhere "the byte value of `\n` is `10`." It says "`\n`" (in C) and trusts that whatever compiler is compiling chibicc will turn that into the byte `10`. If the host compiler had been modified to emit `11` for `'\n'`, every chibicc binary built with that host would also emit `11` for `'\n'` — without any change to chibicc's source.

The tautological style isn't just convenient; it's also a place where a compromised compiler can hide. Rui doesn't fix this — chibicc isn't trying to be a trust-rooting compiler — but the comment is there as a flag. Self-hosting (Chapter 17) will eventually mean chibicc compiles itself and produces this same code, at which point the tautology becomes a fixed point: the `'\n'` chibicc emits matches whatever `'\n'` the host that bootstrapped it produced. If the bootstrap was clean, the output is clean.

### Octal escapes

> `git checkout 699d2b7e3f4ea4ba6ec2d5080f87e243989a5835` — *Add `\<octal-sequence>`*

`\nnn` for a 1- to 3-digit octal value. The implementation prepends a check at the top of `read_escaped_char`, before the switch:

```c
static int read_escaped_char(char **new_pos, char *p) {
  if ('0' <= *p && *p <= '7') {
    // Read an octal number.
    int c = *p++ - '0';
    if ('0' <= *p && *p <= '7') {
      c = (c << 3) + (*p++ - '0');
      if ('0' <= *p && *p <= '7')
        c = (c << 3) + (*p++ - '0');
    }
    *new_pos = p;
    return c;
  }

  *new_pos = p + 1;
  // ... switch as before ...
}
```

The function signature changes: it now takes `char **new_pos` so it can return the position where the escape ended. Octal escapes can be 1, 2, or 3 digits long, and the named escapes are always 1 character — but the caller used to assume "1 character" by hardcoding `p += 2` after the call. With the new signature, the caller asks the function "where did you stop?" and uses that as the next position:

```diff
   for (char *p = start + 1; p < end;) {
-    if (*p == '\\') {
-      buf[len++] = read_escaped_char(p + 1);
-      p += 2;
-    } else {
+    if (*p == '\\')
+      buf[len++] = read_escaped_char(&p, p + 1);
+    else
       buf[len++] = *p++;
-    }
   }
```

The octal-reading loop accumulates into `c` by left-shifting 3 bits per digit (because each octal digit represents 3 bits of value) and ORing in the new digit. Three iterations max — `\377` is `(3 << 3 << 3) | (7 << 3) | 7 = 0xFF = 255`, the largest 3-digit octal value, which fits in a byte. Longer sequences of octal-digit characters are not consumed: `"\1500"` is `\150` followed by `0`, producing the two-byte sequence `'h'` `'0'` (104, 48), which the tests confirm:

```sh
assert 104 'int main() { return "\1500"[0]; }'
```

The first byte is 104 (`'\150'` = decimal 104 = `'h'`); the second byte (which the test doesn't exercise but is there in the data) is 48 (`'0'`). The escape "stops" after three digits, and the next character is consumed normally.

The handling of `\0` (a single octal digit, valid escape, value zero) is a special and important case: `"\0"` is a one-character string containing a NUL byte, plus the implicit NUL terminator. `sizeof("\0")` is 2, and `"\0"[0]` is 0. The 1-digit-octal case in the loop handles this correctly.

### Hex escapes

> `git checkout c2cc1d3c4500caa34da5e68eb62b7474caf96fe2` — *Add `\x<hexadecimal-sequence>`*

`\x` followed by any number of hex digits. The implementation adds another prepended block to `read_escaped_char`:

```c
if (*p == 'x') {
  // Read a hexadecimal number.
  p++;
  if (!isxdigit(*p))
    error_at(p, "invalid hex escape sequence");

  int c = 0;
  for (; isxdigit(*p); p++)
    c = (c << 4) + from_hex(*p);
  *new_pos = p;
  return c;
}
```

with a small helper:

```c
static int from_hex(char c) {
  if ('0' <= c && c <= '9')
    return c - '0';
  if ('a' <= c && c <= 'f')
    return c - 'a' + 10;
  return c - 'A' + 10;
}
```

The structure mirrors octal: read a digit, shift the accumulator left by 4 bits (because each hex digit is 4 bits), OR in the new digit, repeat. The differences are that hex digits include letters (handled by `from_hex`'s three-way split) and that hex escapes have *no length limit* — the loop reads as many hex digits as it can find. This is what C says, and it's why the tests include:

```sh
assert 255 'int main() { return "\x00ff"[0]; }'
```

`\x00ff` is a four-digit hex escape, value `0x00FF = 255`. Stored in a `char` (1 byte), it overflows — but `c` is an `int`, the assignment to `buf[len++]` truncates to one byte, and the result is `0xFF = 255`. There's an underlying wart here: hex escapes can produce values larger than a byte and chibicc silently truncates. The C standard says the behavior is undefined if the escape's value exceeds the target character type. Real compilers warn; chibicc doesn't.

The error case is when `\x` is followed by no hex digit at all — `"\xz"` is invalid, and chibicc reports `"invalid hex escape sequence"` pointing at the `z`. This is one of the few places in the tokenizer that issues a *positive* error (not "unclosed string literal" or "invalid character", but a specific complaint about a particular escape form).

### Where we are

Strings now contain whatever the programmer types between the quotes, with the standard C escape vocabulary. Tabs, newlines, embedded NULs, and arbitrary byte values (via octal or hex) all work. `read_escaped_char` is a three-tier dispatch: octal first (because digits 0-7 overlap with everything else), hex second (because `x` is unambiguous), named last (with default fallthrough for anything else). The signature evolved from `(char *)` to `(char **, char *)` mid-arc to support variable-length escapes — a small reminder that the simplest interface isn't always the right one and that some refactors are best done as soon as the second use case appears.

What's still missing is a way to use string literals interactively. Chibicc can put `"hello\n"` in `.data`, but there's no way to call `puts` or `printf` to actually print it. The driver still doesn't have a way to compile multiple files or link to a stub C library. Both of those land later; for now, a string literal is just data the program can subscript.

---

## 7.5 — Statement expressions

> `git checkout 9dae23461eb6250865f4ee727a0e727a6a4e03ba` — *[GNU] Add statement expression*

After this commit, `({ stmts; expr; })` is a valid expression that evaluates `stmts` for their side effects and yields `expr`'s value. This is a *GNU C extension* — not part of standard C — and the commit's message says explicitly "will be useful for writing tests." Chibicc's tests will start using this syntax in upcoming chapters because it lets a test inline multiple statements into a single expression context, which is exactly what `assert <expected> '...'` wants.

The implementation is small: a new node kind, a new parser case, four lines of codegen, and a special-purpose `add_type` rule.

### The grammar grows a primary alternative

```diff
-// primary = "(" expr ")" | "sizeof" unary | ident func-args? | str | num
+// primary = "(" "{" stmt+ "}" ")"
+//         | "(" expr ")"
+//         | "sizeof" unary
+//         | ident func-args?
+//         | str
+//         | num
 static Node *primary(Token **rest, Token *tok) {
+  if (equal(tok, "(") && equal(tok->next, "{")) {
+    // This is a GNU statement expresssion.
+    Node *node = new_node(ND_STMT_EXPR, tok);
+    node->body = compound_stmt(&tok, tok->next->next)->body;
+    *rest = skip(tok, ")");
+    return node;
+  }
```

Two-token lookahead: `(` followed by `{`. If both are present, we have a statement expression. The body is parsed by the existing `compound_stmt` helper from Chapter 3, starting from the token after `{`. `compound_stmt` returns an `ND_BLOCK` node whose `body` field is a linked list of statements; the statement-expression parser steals that list directly into its own `body` field, throwing away the `ND_BLOCK` wrapper.

The closing `)` is skipped after `compound_stmt` returns. `compound_stmt` ends positioned just after the `}`, so the next token is the `)` we need to consume.

Reusing `compound_stmt` is the canonicalization-at-parse-time pattern from Chapter 6, but in a slightly different mode. Chapter 6 named four instances of the pattern where surface syntax desugars to a different surface form (`>` to `<`, `[]` to `*+`, etc.). Statement expressions are a fifth instance, but the desugaring is more like *delegation*: rather than rewriting `({...})` into a different AST shape, the parser borrows the *parsing logic* of the form it most closely resembles (a compound statement) and wraps the result in a different node kind (`ND_STMT_EXPR` instead of `ND_BLOCK`). The codegen then handles `ND_STMT_EXPR` similarly but distinctly.

### The codegen — almost identical to a block

```c
static void gen_expr(Node *node) {
  switch (node->kind) {
  ...
  case ND_STMT_EXPR:
    for (Node *n = node->body; n; n = n->next)
      gen_stmt(n);
    return;
```

Walk the body, emitting each statement. That's the entire codegen for `ND_STMT_EXPR`. Compare with `ND_BLOCK`:

```c
case ND_BLOCK:
  for (Node *n = node->body; n; n = n->next)
    gen_stmt(n);
  return;
```

— identical, except `ND_BLOCK` lives in `gen_stmt` (a statement-context handler) and `ND_STMT_EXPR` lives in `gen_expr` (an expression-context handler). The two are placed in the right context for what they parse from, even though their bodies are the same. This separation is what makes the rest of the compiler treat them differently: `gen_expr` is called when an expression value is expected (and the value will end up in `%rax`), while `gen_stmt` is called when no value is expected.

The fact that `ND_STMT_EXPR`'s codegen is structurally identical to `ND_BLOCK`'s is what makes the value end up in the right place. The last statement of a statement expression must be an `ND_EXPR_STMT` — an expression-statement whose value is computed but normally discarded. When `gen_stmt` runs on the last `ND_EXPR_STMT`, it computes the expression's value into `%rax` and returns, leaving the value in `%rax`. The caller of `gen_expr(ND_STMT_EXPR)` finds `%rax` holding the right value, exactly as if a single expression had been evaluated.

This is a clever piece of repurposing. `gen_stmt` doesn't *intend* to leave a value in `%rax` for an `ND_EXPR_STMT` — the discard is implicit, because nobody reads `%rax` after a statement. But nothing actively zeroes `%rax` between statements either, so the residual value is still there at the end. Statement expressions exploit this implementation accident as a feature.

### `add_type` enforces the "must end in expression" rule

```c
case ND_STMT_EXPR:
  if (node->body) {
    Node *stmt = node->body;
    while (stmt->next)
      stmt = stmt->next;
    if (stmt->kind == ND_EXPR_STMT) {
      node->ty = stmt->lhs->ty;
      return;
    }
  }
  error_tok(node->tok, "statement expression returning void is not supported");
  return;
```

Walk to the last statement of the body. If it's an `ND_EXPR_STMT`, the statement-expression's type is the contained expression's type. Otherwise — empty body, or last statement is something other than an expression-statement — error out with "statement expression returning void is not supported."

The error message is more specific than the check: chibicc rejects any statement expression whose last statement isn't an expression, but the error wording assumes the user wrote `return;` or some other void-typed thing. The real C extension does support void-typed statement expressions; chibicc doesn't. The simplification is fine for the tests-as-source use case the commit message names.

The walk to the last statement is O(n) in the body length, which would be a quadratic blowup if `add_type` were called repeatedly on the same node. It isn't — `add_type` walks the AST once — and statement-expression bodies are typically short (a handful of statements), so the walk is cheap.

### Tests

```sh
assert 0 'int main() { return ({ 0; }); }'
assert 2 'int main() { return ({ 0; 1; 2; }); }'
assert 1 'int main() { ({ 0; return 1; 2; }); return 3; }'
assert 6 'int main() { return ({ 1; }) + ({ 2; }) + ({ 3; }); }'
assert 3 'int main() { return ({ int x=3; x; }); }'
```

Five tests. The first checks the simplest single-expression case. The second checks the "value of the last statement" rule — the `0;` and `1;` are evaluated and discarded, the `2;` provides the value. The third is interesting: the statement-expression contains a `return 1;`, which short-circuits the entire enclosing function and the `return 3;` after it never runs. This works because `gen_stmt(ND_RETURN)` jumps to the function epilogue regardless of where it's called from — the codegen for `return` doesn't know or care that it's inside an expression.

The fourth uses three statement expressions as operands to `+`, demonstrating that they compose like any other expression. The fifth declares a local inside the body — which works because `compound_stmt` already handles declarations, and `int x` lands in the enclosing function's `locals` list. The `x` is in scope for the entire function body, not just the statement expression — chibicc has no notion of block scope yet, so the `x` outlives the `({ })`. Block scope arrives in Chapter 8 §8.1; until then, every local inside any compound statement (including a statement-expression body) is a frame-level local.

### Where we are

Statement expressions work as a GNU extension. The parser recognizes `({...})` by two-token lookahead, reuses `compound_stmt` for the body, and wraps the result in a distinct node kind. The codegen emits the body's statements in order, leveraging the implementation detail that `gen_stmt(ND_EXPR_STMT)` leaves its value in `%rax`. The type system enforces a "must end in an expression" rule.

This is the first GNU extension chibicc supports. The book should call this out: GNU extensions are useful but not portable, and a programmer using `({...})` writes code that GCC and chibicc accept but that other C compilers (Clang, ICC, MSVC) will reject. Real compilers gate GNU extensions behind a `-std=gnu99`-vs-`-std=c99` distinction; chibicc has no `-std` flag and accepts the extension unconditionally. By Chapter 20 ("GCC extensions worth supporting"), chibicc will pick up several more of these — `typeof`, the `,##__VA_ARGS__` macro form, `asm` blocks. This commit is the first one.

The next few commits stop being about the language and start being about the compiler-as-program.

---

## 7.6 — The driver grows up

> `git checkout d9ea59757e2710e34f105e98230f30f578e0e662` — *Read code from a file instead of argv[1]*

Three commits in sequence transform chibicc from a glorified one-liner into something that resembles a real Unix compiler driver. After these three, chibicc reads its source from a named file (or stdin via `-`), accepts an `-o output` flag, and prints a usage message in response to `--help`. Each commit is small; together they're the moment the program looks like `cc`, not `cc1`.

### Reading from a file

The first of the three commits replaces the assumption that source code arrives in `argv[1]`. The new flow: take a filename on the command line, open it (or use stdin for `-`), read the whole file into a buffer, and tokenize that buffer. Error messages now show the file name and line number where the error occurred.

The header gets one new include and a renamed entry point:

```diff
 #define _POSIX_C_SOURCE 200809L
 #include <assert.h>
 #include <ctype.h>
+#include <errno.h>
 #include <stdarg.h>
 ...
-Token *tokenize(char *input);
+Token *tokenize_file(char *filename);
```

`errno.h` is needed for `strerror` in the "cannot open file" error message. `tokenize` becomes a static (file-private) helper in `tokenize.c`; the public entry point is `tokenize_file`, which takes a filename rather than a string.

The new helpers in `tokenize.c`:

```c
// Returns the contents of a given file.
static char *read_file(char *path) {
  FILE *fp;

  if (strcmp(path, "-") == 0) {
    // By convention, read from stdin if a given filename is "-".
    fp = stdin;
  } else {
    fp = fopen(path, "r");
    if (!fp)
      error("cannot open %s: %s", path, strerror(errno));
  }

  char *buf;
  size_t buflen;
  FILE *out = open_memstream(&buf, &buflen);

  // Read the entire file.
  for (;;) {
    char buf2[4096];
    int n = fread(buf2, 1, sizeof(buf2), fp);
    if (n == 0)
      break;
    fwrite(buf2, 1, n, out);
  }

  if (fp != stdin)
    fclose(fp);

  // Make sure that the last line is properly terminated with '\n'.
  fflush(out);
  if (buflen == 0 || buf[buflen - 1] != '\n')
    fputc('\n', out);
  fputc('\0', out);
  fclose(out);
  return buf;
}

Token *tokenize_file(char *path) {
  return tokenize(path, read_file(path));
}
```

`read_file` is straightforward: open the file (or use stdin for `"-"`), read into a buffer (4 KB at a time, growing the output via `open_memstream`), close the file, append a newline if the file didn't end with one, NUL-terminate, return.

The trailing-newline insertion is small but consequential. The tokenizer's lookahead for line-comment termination (which arrives in §7.7) and several places in the error reporter assume that any character pointer into the input can find a `\n` ahead of it before hitting the NUL. A file that ends `int main(){return 0;}` without a trailing newline would otherwise trip those assumptions. Inserting one if needed is a defensive fix that costs nothing and prevents a class of off-the-end bugs.

The `"-"` convention is older than POSIX but POSIX-blessed: many Unix tools (`cat`, `tar`, `gzip`, `gcc` itself) treat a single hyphen as "read from stdin" when an input filename is expected. Chibicc inherits the convention. The test driver from `test.sh` uses it:

```diff
-  ./chibicc "$input" > tmp.s || exit
+  echo "$input" | ./chibicc - > tmp.s || exit
```

Each test now pipes the source to chibicc instead of passing it as a literal command-line argument. This is partly cosmetic but partly substantive — a long source program would overflow the command-line length limit, but stdin has no such limit. The test infrastructure is being prepared for tests that contain newlines, multi-line declarations, and (eventually, in §7.7) source comments.

### Errors get file names and line numbers

The biggest piece of the commit by line count is the error reporter:

```c
// Reports an error message in the following format and exit.
//
// foo.c:10: x = y + 1;
//               ^ <error message here>
static void verror_at(char *loc, char *fmt, va_list ap) {
  // Find a line containing `loc`.
  char *line = loc;
  while (current_input < line && line[-1] != '\n')
    line--;

  char *end = loc;
  while (*end != '\n')
    end++;

  // Get a line number.
  int line_no = 1;
  for (char *p = current_input; p < line; p++)
    if (*p == '\n')
      line_no++;

  // Print out the line.
  int indent = fprintf(stderr, "%s:%d: ", current_filename, line_no);
  fprintf(stderr, "%.*s\n", (int)(end - line), line);

  // Show the error message.
  int pos = loc - line + indent;

  fprintf(stderr, "%*s", pos, ""); // print pos spaces.
  fprintf(stderr, "^ ");
  vfprintf(stderr, fmt, ap);
  fputc('\n', stderr);
}
```

Three new pieces of work compared to the previous version. First, find the start of the line containing the error location by walking backward until either the start of input or a newline. Second, find the end of the line by walking forward to the next newline (which is guaranteed to exist because of the trailing-newline insertion). Third, count newlines from the start of input to determine the line number.

The output format is the standard Unix compiler error shape: `<file>:<line>: <source line>`, then on the next line, spaces aligned with the file:line prefix and the error location, a caret pointing at the column, and the message. This is exactly what GCC and Clang emit for syntax errors, and it's the format Emacs and Vim's `make` integrations parse to jump to the offending line.

The line-number computation is O(n) per error — walk the entire input from the start each time. For a single-error-and-exit pattern, that's fine. Real compilers cache line offsets to make this O(log n), but chibicc doesn't bother because it errors out on the first problem and doesn't try to recover. A precomputed line-number array arrives in Chapter 8 (commit `46c75e7`, paired with `.file`/`.loc` debug directives), where the table earns its keep.

`current_filename` is a new file-static variable, set by `tokenize`:

```diff
-Token *tokenize(char *p) {
+static Token *tokenize(char *filename, char *p) {
+  current_filename = filename;
   current_input = p;
```

The two pieces of state — file name and input buffer — are conventionally globals here. A real compiler driver might have multiple files in flight at once, but chibicc compiles one file per invocation, so file-static state is fine. Chapter 16 will revisit this when multi-file compilation arrives.

### A `println` refactor

> `git checkout 7b8528f71c78a01e8ff41a76a83a320d1ef80e93` — *Refactor -- no functionality change*

The shortest commit in this trio. Every `printf` call in `codegen.c` becomes a `println` call:

```c
static void println(char *fmt, ...) {
  va_list ap;
  va_start(ap, fmt);
  vprintf(fmt, ap);
  va_end(ap);
  printf("\n");
}
```

`println` is `printf` plus an automatic trailing newline. Every call site loses its `\n` (and almost every call site in `codegen.c` had one). The diff is mechanical — about thirty lines change from `printf("...\n", args);` to `println("...", args);` — and the resulting output is byte-identical to before.

This is another instance of the **pre-factor before feature** pattern named in Chapter 6 §6.5. The next commit (the one that adds `-o`/`--help`) needs to redirect output from stdout to a named file. With the old code, that would mean rewriting every `printf` call site to use `fprintf` and pass the output file. With `println` in place, only `println`'s body needs to change — the call sites are unaffected. The pre-factor commit moves the structural change first, and the feature commit becomes small and focused.

The pattern is now the second clearly named instance: Chapter 6 §6.5 introduced it (the `Function`/`Obj` merge before global variables), and this is the second time we've seen it work the same way. Both commits' messages are explicit: the Chapter 6 one was titled "Merge Function with Var" but had "no functional change" implied, and this one says "no functionality change" outright. The pattern is now part of chibicc's commit-style vocabulary.

### `-o` and `--help`

> `git checkout a0388bada4016bc0c3be6154c159faf80ce18d01` — *Add `-o` and `--help` options*

The actual driver-flag commit. `main.c` gets a real argument parser:

```c
static char *opt_o;
static char *input_path;

static void usage(int status) {
  fprintf(stderr, "chibicc [ -o <path> ] <file>\n");
  exit(status);
}

static void parse_args(int argc, char **argv) {
  for (int i = 1; i < argc; i++) {
    if (!strcmp(argv[i], "--help"))
      usage(0);

    if (!strcmp(argv[i], "-o")) {
      if (!argv[++i])
        usage(1);
      opt_o = argv[i];
      continue;
    }

    if (!strncmp(argv[i], "-o", 2)) {
      opt_o = argv[i] + 2;
      continue;
    }

    if (argv[i][0] == '-' && argv[i][1] != '\0')
      error("unknown argument: %s", argv[i]);

    input_path = argv[i];
  }

  if (!input_path)
    error("no input files");
}
```

The parser handles three forms: `--help` exits with the usage message, `-o path` (with a separate argument) sets the output path, and `-opath` (with the path glued to the flag) does the same. The `-opath` form is a minor C-tradition convenience — most Unix command-line tools support both `-o foo` and `-ofoo` interchangeably for short flags — and the `!strncmp(argv[i], "-o", 2)` check at the bottom handles it as a fall-through. Anything starting with `-` that wasn't recognized is reported as "unknown argument."

The "no input files" check at the end is GCC's exact wording. Chibicc consistently mimics GCC's error and usage messages where possible; this is a small but visible part of feeling like a real compiler.

The output side is the open-and-emit pair:

```c
static FILE *open_file(char *path) {
  if (!path || strcmp(path, "-") == 0)
    return stdout;

  FILE *out = fopen(path, "w");
  if (!out)
    error("cannot open output file: %s: %s", path, strerror(errno));
  return out;
}

int main(int argc, char **argv) {
  parse_args(argc, argv);

  // Tokenize and parse.
  Token *tok = tokenize_file(input_path);
  Obj *prog = parse(tok);

  // Traverse the AST to emit assembly.
  FILE *out = open_file(opt_o);
  codegen(prog, out);
  return 0;
}
```

`open_file` mirrors `read_file`'s convention: no path or `-` means stdout, anything else is opened with `fopen`. The output `FILE *` is threaded into `codegen`, which used to take only the parsed program:

```diff
-void codegen(Obj *prog) {
+void codegen(Obj *prog, FILE *out) {
+  output_file = out;
+
   assign_lvar_offsets(prog);
   emit_data(prog);
   emit_text(prog);
 }
```

`codegen` stashes the file in a static, and `println` (introduced in the previous commit) uses it:

```diff
 static void println(char *fmt, ...) {
   va_list ap;
   va_start(ap, fmt);
-  vprintf(fmt, ap);
+  vfprintf(output_file, fmt, ap);
   va_end(ap);
-  printf("\n");
+  fprintf(output_file, "\n");
 }
```

This is the payoff for the pre-factor. Two lines change in `println`, no other call site in `codegen.c` is touched, and the entire codegen now writes to whatever file `main` opened. Without the `printf`-to-`println` refactor, this commit would have been thirty-plus mechanical changes scattered across the file; with the refactor, it's a focused two-line edit.

The new `test-driver.sh` script tests both flags:

```bash
# -o
rm -f $tmp/out
./chibicc -o $tmp/out $tmp/empty.c
[ -f $tmp/out ]
check -o

# --help
./chibicc --help 2>&1 | grep -q chibicc
check --help
```

Two checks: `-o` produces a file, `--help` prints something containing "chibicc". The `Makefile`'s `test` target now invokes both `test.sh` (which exercises the language) and `test-driver.sh` (which exercises the driver).

`test.sh` itself updates to use the new flag:

```diff
-  echo "$input" | ./chibicc - > tmp.s || exit
+  echo "$input" | ./chibicc -o tmp.s - || exit
```

Same end result — chibicc reads from stdin and writes to `tmp.s` — but routed through the new flag rather than through shell redirection. This is incidental but representative: as the driver gains real options, the test infrastructure switches to using them, which exercises the option parser and ensures the driver behavior stays working.

### Where we are

Chibicc is now invoked the way `cc` is invoked: `chibicc -o output.s input.c`. It reads from stdin if asked, writes to stdout if not given an output path, prints a usage message for `--help`, and produces standard Unix-style error messages with file names and line numbers. All of the *language* parts of the compiler — the parser, the type system, the codegen — are unchanged from before this trio of commits; the changes are entirely in the driver and the I/O layer.

The trio has also given us a second instance of the pre-factor pattern, named in Chapter 6 §6.5 and now visible as a recurring discipline. The `printf`-to-`println` commit is the structural change; the `-o`/`--help` commit is the feature that depends on it. Splitting them keeps each diff small and the review surface narrow.

One last commit closes out the chapter, and it's the smallest one yet.

---

## 7.7 — Comments

> `git checkout 6c0a42926a10ea5abc781c9db89b105e007512b1` — *Add line and block comments*

Twenty-line commit. The tokenizer learns to skip `//`-introduced line comments and `/* ... */` block comments. This is the kind of commit a reader could write themselves in five minutes, and it's the chapter's closer because there's nothing else to add at the language level until the next chapter.

```diff
   while (*p) {
+    // Skip line comments.
+    if (startswith(p, "//")) {
+      p += 2;
+      while (*p != '\n')
+        p++;
+      continue;
+    }
+
+    // Skip block comments.
+    if (startswith(p, "/*")) {
+      char *q = strstr(p + 2, "*/");
+      if (!q)
+        error_at(p, "unclosed block comment");
+      p = q + 2;
+      continue;
+    }
+
     // Skip whitespace characters.
```

Two new branches at the top of the tokenizer's main loop, both before the whitespace-skipping case. Line comments: advance past the `//`, then walk forward until `\n`, then `continue` (the outer loop will pick up the newline as whitespace and skip it). Block comments: advance past the `/*`, search forward for `*/` with `strstr`, error if not found, otherwise jump past it.

The line-comment loop relies on the trailing-newline guarantee from §7.6 (`read_file` ensures the input always ends with `\n`). Without it, a file ending in a `//` comment with no final newline would walk off the end of the buffer into undefined memory.

Block comments use `strstr`, which is a linear scan from `p+2` to the first occurrence of `*/`. This means any `*/` inside a string literal between the comment markers would terminate the comment early, but since the tokenizer hasn't seen the string yet (it's inside a comment, so it's not being lexed), the issue is academic — the bytes `"foo"` inside `/* ... "foo" ... */` are just bytes, not a string literal, and `strstr` doesn't care.

The check for unclosed block comments is the second positive error introduced by the tokenizer (the first being "invalid hex escape sequence" from §7.4). A `/*` with no matching `*/` reports the location of the `/*`, which is what the user wants — they need to know where the runaway comment began, not where the end-of-file is.

Tests:

```sh
assert 2 'int main() { /* return 1; */ return 2; }'
assert 2 'int main() { // return 1;
return 2; }'
```

Two tests. The first inlines a block comment containing fake source code; the second uses a multi-line shell string with an embedded newline to exercise the line-comment form. The expected return value is 2 in both cases, which would only happen if the commented-out `return 1;` wasn't being parsed.

This commit is the smallest in the chapter and it doesn't deserve more than this. Comments are a tokenizer-only feature, the implementation is twenty lines, and the tests exercise both forms once each. The reason it's worth writing about at all is that it closes the chapter neatly: chibicc can now read source code that *looks* like real C source, with comments and whitespace and a usage-pattern that matches `cc`. The next chapter will start using this — its very first commit lands block comments inside the test harness so that tests can be rewritten in C instead of in shell heredocs.

### Where we are

Comments work. The tokenizer is now a four-case dispatch at the top of its main loop: line comment, block comment, whitespace, then the per-token-kind branches that have been there all along. The two new cases `continue` to restart the loop, ensuring that any whitespace immediately following a comment is also skipped.

That's the chapter.

---

## Recap

| Commit | What it added |
|---|---|
| `a4d3223` | Global variables; lookahead-based dispatch in `parse` (`is_function`); `global_variable` parser; `find_var` falls back to globals; `gen_addr` branches on `is_local`; RIP-relative addressing; `emit_data`/`emit_text` split; `.data` directive |
| `be38d63` | `TY_CHAR`; `ty_char` singleton; `is_typename` helper; `argreg8`/`argreg64` split; `movsbq` for sign-extending byte loads; `mov %al, ...` for byte stores; size-based dispatch in `load`/`store`/prologue |
| `4cedda2` | `TK_STR` token kind; string-literal lexing (no escapes yet); `Type *ty` and `char *str` on `Token`; anonymous globals via `new_unique_name`/`new_anon_gvar`/`new_string_literal`; `init_data` field on `Obj`; `.byte` emission in `emit_data` |
| `35a0bcd` | New file `strings.c`; `format()` helper; `new_unique_name` rewritten to use it |
| `ad7749f` | Named escape sequences (`\a`, `\b`, `\t`, `\n`, `\v`, `\f`, `\r`, `\e`); `read_escaped_char`; two-pass string-literal reader; the "Trusting Trust" comment |
| `699d2b7` | Octal escape sequences (`\nnn`); `read_escaped_char` signature changes to `(char **, char *)` |
| `c2cc1d3` | Hex escape sequences (`\xHH...`); `from_hex` helper; first "invalid hex escape sequence" diagnostic |
| `9dae234` | Statement expressions (`({...})`); `ND_STMT_EXPR` node kind; two-token lookahead in `primary`; `add_type` rule enforcing "ends in expression"; first GNU extension |
| `d9ea597` | `read_file` and `tokenize_file`; stdin via `-`; trailing-newline guarantee; error reporter shows `<file>:<line>: <source>` with caret |
| `7b8528f` | `printf` → `println` refactor in `codegen.c`; pre-factor for `-o` |
| `a0388ba` | `-o` and `--help` flags; `parse_args`; `usage`; `open_file`; `codegen` takes `FILE *`; `output_file` static in `codegen.c`; `test-driver.sh` |
| `6c0a429` | Line comments (`//`) and block comments (`/* ... */`) in tokenizer |

Twelve commits, four genuine "new feature" arcs (globals, `char`, string literals + escapes, statement expressions), three driver-maturation commits (file I/O, `println` pre-factor, `-o`/`--help`), one tiny refactor (`format`), and one tiny tokenizer addition (comments). The chapter's center of gravity is split between the first commit (`a4d3223`, which lands globals and the lookahead pattern) and the string-literal-plus-escape-sequence arc (`4cedda2` through `c2cc1d3`, four commits that build out the data side of the `.data` section).

Two patterns continued to crystallize. **Canonicalization-at-parse-time** picked up its fifth named instance with statement expressions — the `({...})` form delegates to `compound_stmt` and wraps in a different node kind, which is a slightly different mode of canonicalization (delegating the parser, not the desugaring) but still in the family. **Pre-factor before feature** got its second clear named instance with the `printf`-to-`println` refactor right before `-o`/`--help`. Both patterns now have at least two examples each, which is enough to call them part of chibicc's commit-style vocabulary.

Three smaller things landed that the next few chapters will lean on. The `format` helper (§7.3) becomes a workhorse later. The `read_file`/`tokenize_file` split (§7.6) is the foundation for multi-file compilation in Chapter 16. The `is_typename` helper (§7.2) starts as a two-keyword check and grows steadily through Chapter 14, eventually listing every C type keyword and `typedef`-named alias.

Chapter 8 turns to scopes and source locations. Block scope finally arrives — the function-level locals from Chapters 3 through 7 get nested into per-block symbol tables — and the source-location infrastructure from this chapter's error reporter gets reused for `.file` and `.loc` directives in the assembly output, so that GDB can step through chibicc-compiled programs at the source level. The chapter also rewrites the test harness in C, taking advantage of statement expressions from §7.5 and reading-from-file from §7.6 to make tests look like C functions instead of shell heredocs. Five commits, much shorter than this chapter.
