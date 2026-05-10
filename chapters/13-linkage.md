# Chapter 13 — Linkage

> Commits covered: `006a45c`, `2764745`, `9df5178`, `310a87e`, `319772b`, `127056d`, `30b3e21`, `eb85527`, `ee252e6`, `6a0ed71`, `dcd4579`. Eleven commits — a return to a moderate-size chapter after Ch 11's twenty-one and Ch 12's nineteen — adding `extern`, `static` (both flavors), `_Alignof`/`_Alignas`, compound literals, `do … while`, bare `return;`, and two ABI-tightening codegen fixes.

Chapter 12 finished the initializer story: every shape of `=` after a declarator now works, for both locals and globals. What it didn't touch was *which symbols other translation units can see, or that this one expects to find elsewhere*. Globals were `.globl` unconditionally; there was no way to declare a name without defining it; no way to keep a definition private to the file. Static local variables — globals-in-disguise scoped to a function — didn't exist either. This chapter closes that gap, plus a cluster of nearby small features that share parts of the new machinery.

The chapter title — *Linkage* — reaches a little. Only four of the eleven commits are linkage proper (`extern` × 2, `static` global, static local). The rest are along for the ride: `_Alignof`/`_Alignas` extend the `VarAttr` channel that `extern` arrives on; compound literals reuse the synthesized-anonymous-global trick that static locals introduce; `do … while` is the fourth loop construct; bare `return;` is the single-line patch the test suite asks for; the two final commits tighten codegen against the x86-64 psABI. Read the chapter as a moderate-length cleanup pass with linkage as the through-line.

The nine sections:

- **§13.1** — `extern` at file and block scope (commits 116, 117).
- **§13.2** — `_Alignof` and `_Alignas` (commits 118, 119).
- **§13.3** — Static local variables (commit 120).
- **§13.4** — Compound literals (commit 121).
- **§13.5** — Bare `return;` (commit 122).
- **§13.6** — Static global variables (commit 123).
- **§13.7** — `do … while` (commit 124).
- **§13.8** — 16-byte stack alignment (commit 125).
- **§13.9** — Truncating small return values (commit 126).

The chapter has no concept interlude. The chapter mapping flagged a possible one on *static vs dynamic linking*, drawn from Rui's Japanese book. The §13.1 prose turned out not to need it — chibicc never participates in dynamic linking (it always emits position-independent-by-default x86-64 with relocations the system linker resolves at link time), so the static-vs-dynamic distinction has no representation in the codegen, and walking it would be a digression. The C model of *external* and *internal* linkage carries the section on its own.

The chapter follows `main` order. As with every chapter from 7 onward, the calendar dates scatter — `127056d` (compound literals) is dated August 2019, `006a45c`/`eb85527` (`extern` and static globals) are September 2020, `dcd4579` (small returns) is mid-September 2020. The chapter does not try to untangle them. Order is the order Rui pinned.

---

## 13.1 — `extern` at file scope and inside a block

> `git checkout 006a45ccd475296ee19ec87891523d89ce3f2f24` — *Add extern*
>
> `git checkout 27647455e4cb7db1545a7b69c3a324aa025a471a` — *Handle extern declarations in a block*

Two commits land `extern`, one for file scope and one for block scope. The mechanism is the same in both places — a third flag on the `VarAttr` struct, threaded through `declspec` like `static` and `typedef` already are — but the section's centerpiece isn't the implementation. It's *what `extern` means*.

The C linkage model is one of those features that looks obvious until someone asks for a definition. A function-scope `int x;` has *no linkage*: no other translation unit can refer to it, and the name disappears at the end of the function. A file-scope `int x;` has *external linkage* by default: every translation unit that declares the same name refers to the same object, and the linker is expected to find a single definition somewhere in the program. A file-scope `static int x;` has *internal linkage*: the name is visible only inside this translation unit, and no other unit can refer to it even by re-declaring it (`static` at file scope is §13.6's commit). And `extern int x;`, at either file or block scope, is a *declaration without a definition* — the name has external linkage, and the linker is expected to find the storage somewhere else.

The distinction that matters for the compiler is *definition versus declaration*. A definition allocates storage and may have an initializer; a declaration introduces a name and asserts that storage is allocated elsewhere. Until this commit, every file-scope variable in chibicc was a definition: `new_gvar` allocated an `Obj`, the codegen's `emit_data` pass walked the `Obj` list and emitted `.globl` plus `.data` or `.bss` for each. With `extern`, that pass needs a way to skip names that aren't ours to define.

The mechanism is the `is_definition` flag on `Obj`. The first commit threads it through:

```c
typedef struct {
  bool is_typedef;
  bool is_static;
  bool is_extern;
} VarAttr;
```

A new flag in the storage-class triple. `declspec` recognizes the keyword and sets it:

```c
if (equal(tok, "typedef") || equal(tok, "static") || equal(tok, "extern")) {
  if (!attr)
    error_tok(tok, "storage class specifier is not allowed in this context");

  if (equal(tok, "typedef"))
    attr->is_typedef = true;
  else if (equal(tok, "static"))
    attr->is_static = true;
  else
    attr->is_extern = true;

  if (attr->is_typedef && attr->is_static + attr->is_extern > 1)
    error_tok(tok, "typedef may not be used together with static or extern");
  tok = tok->next;
  continue;
}
```

The mutual-exclusion check tightens slightly: `typedef` still can't combine with anything, but the prior `is_typedef + is_static > 1` rule (which prevented two specifiers in any combination) becomes `typedef may not be used with static or extern`. The standard does allow `static` and `extern` to coexist at the same site only in degenerate ways that chibicc doesn't bother to support; the looser check is good enough for what the test suite exercises.

`new_gvar` flips `is_definition` on by default for a freshly-created global, and `global_variable` clears it back to false when `extern` is set:

```c
static Obj *new_gvar(char *name, Type *ty) {
  Obj *var = new_var(name, ty);
  var->next = globals;
  var->is_definition = true;
  globals = var;
  return var;
}
```

```c
Obj *var = new_gvar(get_ident(ty->name), ty);
var->is_definition = !attr->is_extern;
```

Default-true and override-on-extern: a small structural choice. `new_gvar` is also called for anonymous globals — string literals (since §7.x) and the synthesized backings for static locals (§13.3) and global compound literals (§13.4). All of those *are* definitions, so the default-true makes them work without each call site re-asserting the flag.

The codegen change is one half of one line:

```diff
-    if (var->is_function)
+    if (var->is_function || !var->is_definition)
       continue;
```

`emit_data` walks the global `Obj` list and emits `.data`/`.bss` for variables; it now skips both functions (already the case) and any global without a definition. An `extern int ext1;` produces an `Obj` in the symbol table — so other places in the parser can resolve `ext1` as a known name — but produces no assembler output. The system linker resolves the reference at link time against whichever object file actually defines the symbol.

The test file pins it:

```c
extern int ext1;
extern int *ext2;

int main() {
  ASSERT(5, ext1);
  ASSERT(5, *ext2);
  ...
}
```

with `int ext1 = 5;` and `int *ext2 = &ext1;` in `test/common`, a separate translation unit linked against `test/extern.c`. The first time chibicc's test harness exercises *real* multi-file linking with name resolution. Until this commit, every cross-file reference in the suite was a function call (functions have always been resolvable by name through the symbol table without needing `extern`); now data symbols join the same channel.

The second commit (`2764745`) extends `extern` to block scope. The C standard allows `extern int x;` inside a function — the name is introduced into the block scope, but the storage lives elsewhere with external linkage. The implementation is small. `compound_stmt` already had branches for typedef and (since §10.2) for nested function definitions. The new branch routes `extern` declarations at block scope through `global_variable`:

```c
if (is_function(tok)) {
  tok = function(tok, basety, &attr);
  continue;
}

if (attr.is_extern) {
  tok = global_variable(tok, basety, &attr);
  continue;
}

cur = cur->next = declaration(&tok, tok, basety);
```

`global_variable` is reused as-is — it adds an `Obj` to the global list, sets `is_definition = false` (because `attr.is_extern` is true), and returns. The block-scope `extern int ext3;` and the file-scope one produce identical `Obj` records; the only difference is that the block-scope name doesn't enter the function's local scope chain (because `global_variable` doesn't push it as a local). The lookup later, when `ext3` is referenced, walks up the scope chain to global scope and finds it there.

The two test additions in `test/extern.c` cover both: a block-scope variable declaration and a block-scope function declaration (with and without the `extern` keyword — function declarations are implicitly extern):

```c
extern int ext3;
ASSERT(7, ext3);

int ext_fn1(int x);
ASSERT(5, ext_fn1(5));

extern int ext_fn2(int x);
ASSERT(8, ext_fn2(8));
```

Function declarations without `extern` are picked up by the `is_function(tok)` branch — chibicc treats every block-scope function declaration as adding the name to the symbol table, which works for the tests. There's no enforcement of "function declarations at block scope must not have a body" (you can't write a function body anywhere except file scope in chibicc anyway, so the standard's restriction is moot).

The forward-prediction made in Chapter 11 and Chapter 12 closes here: `is_extern` is the third flag on `VarAttr`, alongside `is_typedef` and `is_static`. The channel was the prediction; the third occupant is the close.

### Where we are

Chibicc supports `extern` at file scope and at block scope. The mechanism is a third `VarAttr` flag (`is_extern`) plus an `is_definition` flag on `Obj` that the codegen consults when deciding whether to emit a definition or skip the symbol. The C linkage model — external (the default for file-scope names), internal (`static`, §13.6), none (block-scope locals) — is now representable in chibicc's symbol table. Cross-file linking of data names works for the first time; the test suite exercises it in `test/extern.c`.

---

## 13.2 — `_Alignof` and `_Alignas`

> `git checkout 9df51789e7fd36fc1580bcd80676f9bcc4e24be1` — *Add _Alignof and _Alignas*
>
> `git checkout 310a87e15e98bb5abfd86ea7bb2a1cca1f5243c7` — *[GNU] Allow a variable as an operand of _Alignof*

C11 added two alignment keywords. `_Alignof(T)` is a constant expression evaluating to the alignment requirement of type `T`, in bytes — a counterpart to `sizeof`. `_Alignas(N)` is a declaration specifier (sitting alongside `static`, `typedef`, and the type names themselves) that overrides the natural alignment of the variable being declared, forcing it to a multiple of `N` bytes. Both land in one commit; a small follow-up extends `_Alignof` to accept a variable expression as a GNU extension.

The work is more than the syntax. Until this commit, alignment in chibicc had been *the type's alignment*. A `Type` carries an `align` field (initialized when the type is constructed: 1 for `char`, 4 for `int`, 8 for `long`, max-of-members for struct), and the codegen used `var->ty->align` directly when computing stack offsets and emitting `.align` directives. With `_Alignas`, alignment becomes *the variable's alignment* — two variables of the same type can have different alignment requirements. The `align` field has to migrate from `Type` to `Obj`.

The `chibicc.h` change introduces the new field:

```c
struct Obj {
  Obj *next;
  char *name;
  Type *ty;
  bool is_local;
  int align;     // alignment

  // Local variable
  int offset;
  ...
};
```

And on `Member`, for the per-member `_Alignas` case (which the test suite exercises):

```c
struct Member {
  Member *next;
  Type *ty;
  Token *tok;
  Token *name;
  int idx;
  int align;
  int offset;
};
```

Every place that previously read `var->ty->align` now reads `var->align` (and similarly for members). The default initialization preserves the old behavior: when `new_var` constructs an `Obj`, it copies the type's alignment in:

```c
static Obj *new_var(char *name, Type *ty) {
  Obj *var = calloc(1, sizeof(Obj));
  var->name = name;
  var->ty = ty;
  var->align = ty->align;
  push_scope(name)->var = var;
  return var;
}
```

A variable declared without `_Alignas` keeps its type's natural alignment. A variable declared with `_Alignas(32)` overrides the field after construction. The codegen, which only looks at `var->align`, doesn't need to know which path the value came from.

`_Alignas` parses through `declspec`, the same routing as `static`/`typedef`/`extern`. It needs to carry an integer back to the variable, so `VarAttr` grows a fourth field:

```c
typedef struct {
  bool is_typedef;
  bool is_static;
  bool is_extern;
  int align;
} VarAttr;
```

`align` is zero by default (no `_Alignas`); a non-zero value means the declaration carried an `_Alignas` specifier with that value. The parser:

```c
if (equal(tok, "_Alignas")) {
  if (!attr)
    error_tok(tok, "_Alignas is not allowed in this context");
  tok = skip(tok->next, "(");

  if (is_typename(tok))
    attr->align = typename(&tok, tok)->align;
  else
    attr->align = const_expr(&tok, tok);
  tok = skip(tok, ")");
  continue;
}
```

`_Alignas(int)` and `_Alignas(4)` and `_Alignas(2+2)` all reach the same place. The first form looks up the type's alignment via the new `typename` helper (a thin wrapper around `declspec` + `abstract_declarator`); the second form invokes `const_expr` from §11.15, the same constant-expression evaluator used everywhere a compile-time integer is needed. The branching on `is_typename(tok)` is the same trick `sizeof` uses to discriminate between a parenthesized type and a parenthesized expression.

Two callers consume `attr->align`. Local variables in `declaration`:

```c
Obj *var = new_lvar(get_ident(ty->name), ty);
if (attr && attr->align)
  var->align = attr->align;
```

And global variables in `global_variable`:

```c
Obj *var = new_gvar(get_ident(ty->name), ty);
var->is_definition = !attr->is_extern;
if (attr->align)
  var->align = attr->align;
```

The struct-member case is parallel. `struct_members` was previously calling `declspec(&tok, tok, NULL)` (passing null for the attr, because struct members can't carry storage classes); now it passes a real `VarAttr` to capture `_Alignas`:

```c
VarAttr attr = {};
Type *basety = declspec(&tok, tok, &attr);
```

and uses it when constructing the member:

```c
mem->align = attr.align ? attr.align : mem->ty->align;
```

A struct member keeps its type's alignment unless `_Alignas` overrode it. The struct-layout pass — which assigns offsets and computes the struct's overall alignment — switches from `mem->ty->align` to `mem->align` everywhere:

```c
for (Member *mem = ty->members; mem; mem = mem->next) {
  offset = align_to(offset, mem->align);
  mem->offset = offset;
  offset += mem->ty->size;

  if (ty->align < mem->align)
    ty->align = mem->align;
}
```

A struct with one `_Alignas(16) char x;` member has alignment 16, even though the natural alignment of `char` is 1. The struct's `align` propagates up to anything containing the struct, the same way it always did.

The codegen sites — two of them, in `assign_lvar_offsets` and `emit_data` — switch from the type field to the obj field:

```diff
-      offset = align_to(offset, var->ty->align);
+      offset = align_to(offset, var->align);
```

```diff
-    println("  .align %d", var->ty->align);
+    println("  .align %d", var->align);
```

This is the §12.11 interaction the handoff flagged. §12.11 added `.align` emission for every global, sourced from the type. §13.2 keeps the directive but switches the source to the per-variable field. A global declared `int _Alignas(512) g1;` now emits `.align 512` instead of `.align 4`. The test pins it with `(long)(char *)&g1 % 512 == 0` — at runtime the address must be a multiple of 512.

`_Alignof` is a primary expression. It compiles to a constant — the alignment of the operand type — which `eval` (§11.15) then folds:

```c
if (equal(tok, "_Alignof")) {
  tok = skip(tok->next, "(");
  Type *ty = typename(&tok, tok);
  *rest = skip(tok, ")");
  return new_num(ty->align, tok);
}
```

The same `typename` helper that `_Alignas(int)` used. `_Alignof(int)` becomes the integer literal `4`; `_Alignof(struct {char a; long b;}[2])` reaches into struct layout and returns `8` (the long forces 8-byte alignment, the array preserves it).

The follow-up commit (`310a87e`) handles the GNU `_Alignof(x)` form where `x` is a variable, not a type. The lookahead is identical to the one §6.x's `sizeof` and §13.4's compound-literal share — peek past the `(` to see whether what follows is a type:

```c
if (equal(tok, "_Alignof") && equal(tok->next, "(") && is_typename(tok->next->next)) {
  Type *ty = typename(&tok, tok->next->next);
  *rest = skip(tok, ")");
  return new_num(ty->align, tok);
}

if (equal(tok, "_Alignof")) {
  Node *node = unary(rest, tok->next);
  add_type(node);
  return new_num(node->ty->align, tok);
}
```

Two arms: the `_Alignof(typename)` arm runs first and consumes the parenthesized type if it sees one; otherwise the `_Alignof unary` arm parses an expression, calls `add_type` to attach a type to it (since the expression is otherwise unevaluated — `_Alignof x` doesn't read `x`), and returns the type's alignment. The expression isn't actually compiled; `add_type` is run only for its side effect of populating `.ty` so the alignment can be read. The pattern is the same one `sizeof unary` used since §6.5.

The test file `test/alignof.c` covers each shape:

```c
ASSERT(4, _Alignof(int));
ASSERT(8, _Alignof(struct {char a; long b;}[2]));
ASSERT(32, ({ _Alignas(32) char x, y; &y-&x; }));
ASSERT(16, ({ struct { _Alignas(16) char x, y; } a; &a.y-&a.x; }));
ASSERT(0, (long)(char *)&g1 % 512);
ASSERT(1, ({ char x; _Alignof(x); }));
```

The fourth line is the per-member `_Alignas` case: two adjacent `char x, y;` members with `_Alignas(16)` end up at offsets 0 and 16 (instead of the natural 0 and 1), so `&a.y - &a.x` returns 16 / sizeof(char) = 16. The fifth line pins the global-alignment case from `g1`. The sixth is the GNU extension.

### Where we are

Alignment is now a per-variable property, not a per-type property. `_Alignof(T)` and `_Alignof(x)` both fold to constants at parse time. `_Alignas(N)` rides on `VarAttr` like the other declaration specifiers. The §12.11 `.align` emission gets a user-settable source. Struct member alignment is also overridable, with the struct's overall alignment propagating up from members the same way it did before. The constant-expression evaluator from §11.15 picks up its second non-test caller (after `gvar_initializer` in §12.6).

---

## 13.3 — Static local variables

> `git checkout 319772b42ebc2311a56ef54e1e9a60c5583971b1` — *Add static local variables*

A nine-line patch to `parse.c`. Conceptually it's the chapter's most interesting commit, because *it blurs the local-versus-global split named in §12.6*. A `static` inside a function has local scope (the name is visible only inside the function body) but global storage (the variable lives in `.data` or `.bss`, not on the stack, and persists across calls). Initialization happens once, at program start, not on each call.

The fix is small because every piece of machinery is already in place:

```c
if (attr && attr->is_static) {
  // static local variable
  Obj *var = new_anon_gvar(ty);
  push_scope(get_ident(ty->name))->var = var;
  if (equal(tok, "="))
    gvar_initializer(&tok, tok->next, var);
  continue;
}

Obj *var = new_lvar(get_ident(ty->name), ty);
```

Four ideas land in one branch.

The first is `new_anon_gvar`. It's existed since Chapter 7's string literal commit — a helper that allocates a global with an internal name (`new_unique_name` returns `.L..0`, `.L..1`, …):

```c
static char *new_unique_name(void) {
  static int id = 0;
  return format(".L..%d", id++);
}

static Obj *new_anon_gvar(Type *ty) {
  return new_gvar(new_unique_name(), ty);
}
```

The synthesized name is what goes in the assembler output; the C-level name (`i`, in the test's `static int i;`) never appears in the compiled object. The internal name is `.L..N`, where the `.L.` prefix tells the GNU assembler that the symbol is local to the file (a convention that avoids cluttering the object's symbol table). Two functions with `static int i;` get different `.L..N` symbols and don't collide.

The second is `push_scope`. The `Obj` for the static local is global (it's on `globals`, the file-scope chain), but its *name* needs to be visible only inside the enclosing function. `push_scope(get_ident(ty->name))->var = var` puts the C-level name into the current scope, mapping it to the global `Obj`. When `i++` is parsed inside the function, the scope walk finds the entry, and the resulting `ND_VAR` node points to the global. After the function ends, `leave_scope` discards the scope frame; the name is no longer visible, but the `Obj` lives on in `globals`.

The third is `gvar_initializer` from §12.6. A static local with `static int j = 1+1;` needs its initializer evaluated at compile time and written to `.data` (so the variable starts at the right value when the program loads), not at function-entry time. `gvar_initializer` does exactly that: it parses the initializer into an `Initializer` tree, walks it with `write_gvar_data`, and stashes a byte buffer in `var->init_data`. The `2` is computed by `eval` and written into the four bytes of the global.

The fourth is the absence of an `lvar_initializer` call. A normal local with `int i = 5;` lowers to a runtime `i = 5;` assignment; a static local with `static int i = 5;` does not. The initialization happens once at program load, not on each call. The test confirms it:

```c
int counter() {
  static int i;
  static int j = 1+1;
  return i++ + j++;
}
...
ASSERT(2, counter());
ASSERT(4, counter());
ASSERT(6, counter());
```

`i` starts at 0 (uninitialized, so `.bss`) and `j` starts at 2 (initialized, so `.data`). On the first call, `counter()` returns `0 + 2 = 2`, leaving `i = 1` and `j = 3`. On the second call, `1 + 3 = 4`, leaving `i = 2, j = 4`. On the third, `2 + 4 = 6`. The values persist across calls because the storage is global, even though the name is function-scoped.

This is the local-versus-global split *blurring*. §12.6 named the split as the chapter's central tension: locals lower to assignments at parse time and live on the stack; globals serialize to byte buffers and live in `.data`/`.bss`. A static local is on the global side of the split for everything that matters — storage, initialization, codegen — but on the local side for one thing only: scope. The split survives the blur because the *back end* doesn't know about scope. The codegen walks `globals`; it sees the `Obj` for the static local right alongside the file-scope globals. The `.L..N` name keeps the symbol private to the translation unit. The function is none the wiser.

The `continue` at the end of the branch matters too. The fall-through path (`new_lvar(...)` and the rest of `declaration`) handles the *runtime* declaration — it appends an `ND_EXPR_STMT` for the initializer (when present) to the function body. Static locals skip all of that, because there's no runtime work to emit. The function's compiled body has *zero* instructions corresponding to `static int i;`. The variable just exists in `.bss` from the moment the program loads.

A consequence the chapter doesn't dwell on: chibicc doesn't implement the C standard's "first-call initialization" semantics for static locals with non-constant initializers. The standard says `static int x = some_function_call();` is rejected (initializers for static locals must be constant expressions); chibicc rejects the same way (via `eval`'s `not a compile-time constant` error in `gvar_initializer`). The test suite doesn't exercise the rejection, but the path is the same one §12.6 set up. There's no special "run this initializer once on first entry" machinery.

### Where we are

Static local variables work. The mechanism is `new_anon_gvar` (synthesized internal name), `push_scope` (function-scoped lookup), and `gvar_initializer` (compile-time initialization in `.data` or `.bss`). The local-versus-global split from §12.6 *blurs* here: a static local is on the global side for storage and initialization, on the local side for scope. The four pieces of machinery were all in place before this commit; the work was choosing the right four.

---

## 13.4 — Compound literals

> `git checkout 127056dc1de6ddad280f6cf09cb15538dca22f43` — *Add compound literals*

A compound literal is an initializer-list at expression position. `(int[]){1, 2, 3}` is an array of three ints, allocated and initialized inline; `(struct point){1, 2}` is a struct value with `x = 1` and `y = 2`. The C99 feature reuses the `Initializer` tree from §12.1, with one new ingredient: the compound literal needs *somewhere to live*. The parser synthesizes an anonymous variable to back it.

The commit's parse.c hunk lives in `postfix`:

```c
// postfix = "(" type-name ")" "{" initializer-list "}"
//         | primary ("[" expr "]" | "." ident | "->" ident | "++" | "--")*
static Node *postfix(Token **rest, Token *tok) {
  if (equal(tok, "(") && is_typename(tok->next)) {
    // Compound literal
    Token *start = tok;
    Type *ty = typename(&tok, tok->next);
    tok = skip(tok, ")");

    if (scope->next == NULL) {
      Obj *var = new_anon_gvar(ty);
      gvar_initializer(rest, tok, var);
      return new_var_node(var, start);
    }

    Obj *var = new_lvar("", ty);
    Node *lhs = lvar_initializer(rest, tok, var);
    Node *rhs = new_var_node(var, tok);
    return new_binary(ND_COMMA, lhs, rhs, start);
  }

  Node *node = primary(&tok, tok);
  ...
}
```

The recognizer is `(` followed by a typename. That conflicts with the cast syntax `(T) expr` — both start `( typename )`. The disambiguator is in `cast`, which used to commit unconditionally to a cast after seeing `(typename)` and now peeks at what comes next:

```c
if (equal(tok, "(") && is_typename(tok->next)) {
  Token *start = tok;
  Type *ty = typename(&tok, tok->next);
  tok = skip(tok, ")");

  // compound literal
  if (equal(tok, "{"))
    return unary(rest, start);

  // type cast
  Node *node = new_cast(cast(rest, tok), ty);
  ...
}
```

If the next token after `(typename)` is `{`, the parsed source is a compound literal — `cast` rewinds (`return unary(rest, start)` re-parses from the original `(`) and the postfix arm picks it up. Otherwise it's a regular cast and the existing path proceeds. The two-token lookahead for `{` after `)` is small but it's the dispatch keystone.

Once the compound-literal arm fires, the question is *where the storage lives*. C answers: it depends on where the compound literal appears. At file scope, the literal has static storage duration (lives until program exit, addressable as a global). At block scope, it has automatic storage duration (lives until the enclosing block ends, lives on the stack). The two arms split on `scope->next == NULL`, the predicate that means "we're at the outermost (file) scope."

The file-scope arm uses §13.3's `new_anon_gvar` plus §12.6's `gvar_initializer`. Both pieces were already in place; the compound-literal commit just calls them. The result is a synthesized global with a `.L..N` name, whose initial bytes are written to `.data` (or `.bss`), and whose name is never visible at the C level — only the resulting `ND_VAR` node carries it forward into the AST.

The block-scope arm uses `new_lvar("", ty)` (a local with an empty name — the lookup chain never finds it because no caller searches for `""`) plus `lvar_initializer` from §12.1. The result is an unnamed stack slot, initialized via the comma chain `lvar_initializer` builds, with the synthesized variable's address as the chain's value. The compound-literal expression evaluates to `(initializer-chain, the-variable)` — a comma expression whose left operand performs the initialization for side effect and whose right operand returns the variable. Compound assignment (`+=`) uses the same `ND_COMMA` shape, since §11.2.

The test file pins both shapes:

```c
ASSERT(1, (int){1});
ASSERT(2, ((int[]){0,1,2})[2]);
ASSERT('a', ((struct {char a; int b;}){'a', 3}).a);
ASSERT(3, ({ int x=3; (int){x}; }));
(int){3} = 5;

Tree *tree = &(Tree){
  1,
  &(Tree){
    2,
    &(Tree){ 3, 0, 0 },
    &(Tree){ 4, 0, 0 }
  },
  0
};

ASSERT(1, tree->val);
ASSERT(2, tree->lhs->val);
ASSERT(3, tree->lhs->lhs->val);
ASSERT(4, tree->lhs->rhs->val);
```

The first five lines exercise block-scope literals. The interesting test is `(int){3} = 5;` — a compound literal is an *lvalue* in C, so assigning to it is well-formed (and useless, since the literal is then discarded). Chibicc accepts it because the synthesized `Obj` is a regular variable; assigning to it goes through the same path as any other assignment.

The `tree` initializer is the file-scope case, and it's the more interesting half. Each `&(Tree){...}` is a global compound literal — a synthesized global of type `Tree`, initialized in place. The `&` takes its address. `tree` is a regular global initialized with a pointer-to-global, which §12.7's `eval2`/`eval_rval` machinery handles: each address-of-anonymous-global becomes a relocation in `tree`'s relocation list, written as `.quad .L..N+0` in the assembler output. The relocation channel that §12.7 added for `int *p = &x;` carries through unchanged.

Three pieces of inherited machinery do the heavy lifting:

1. **The Initializer tree** (§12.1) holds the parsed brace-list, identical in shape to a normal declaration's initializer.
2. **`gvar_initializer` and `lvar_initializer`** (§12.1, §12.6) consume the tree, producing either a byte buffer (with relocations) or a comma chain of assignments.
3. **The Relocation mechanism** (§12.7) connects pointer-to-anonymous-global initializers to the `.data` section.

The compound-literal commit adds none of this. It adds only the ten-line postfix arm, the four-line cast disambiguator, and a test file. The reuse ratio is the chapter's highest.

A note on the parse-time canonicalization count. Compound literals could be argued either way. They desugar a brace-list-at-expression-position into an anonymous-variable-plus-initializer-plus-reference, which is a parse-time AST rewrite — a candidate for the canonicalization family that started in §6.5 (`x[y]` to `*(x+y)`). The chapter doesn't add it to the count, on the same grounds §12.1 didn't add initializer lowering: the mechanism goes through the Initializer tree intermediate, not through direct AST rewriting. The count stays at eight.

### Where we are

Compound literals work at both file and block scope. The trick is to synthesize an anonymous variable to back the literal: a `.L..N` global at file scope, an empty-named stack slot at block scope. The Initializer tree from §12.1 and the lowering paths from §12.1 and §12.6 carry the actual initialization. The cast/literal disambiguator in `cast` (peek for `{` after `(typename)`) is the parser's only new dispatch.

---

## 13.5 — Bare `return;`

> `git checkout 30b3e216cd4eca3b8a13cb0a0613f053ac1d4925` — *Add return that doesn't take any value*

A one-line patch on each side.

The parser:

```c
if (equal(tok, "return")) {
  Node *node = new_node(ND_RETURN, tok);
  if (consume(rest, tok->next, ";"))
    return node;

  Node *exp = expr(&tok, tok->next);
  *rest = skip(tok, ";");
  ...
}
```

If the token after `return` is `;`, build an `ND_RETURN` with no `lhs` and stop. Otherwise the existing expression-parsing code runs.

The codegen:

```c
case ND_RETURN:
  if (node->lhs)
    gen_expr(node->lhs);
  println("  jmp .L.return.%s", current_fn->name);
  return;
```

If there's an expression, emit it (it lands in `%rax`, which is the return-value register). Either way, jump to the function's epilogue.

The handoff predicted Rui would either reuse `ND_RETURN` with a null `lhs` or introduce a new node kind. He reused. The reuse is in the same shape as the §11.11 `ND_GOTO`-with-extra-fields pattern — a node kind whose fields can be partially populated, with the codegen branching on which fields are present. The grammar comment updates from `"return" expr ";"` to `"return" expr? ";"`, and that's the entire parser-side documentation change.

Bare `return;` is C's way to exit a `void`-returning function early. It's a small feature in isolation, but it's the kind of cleanup that compounds: until this commit, every `void` function in chibicc had to fall off the end (or return a junk integer). The test confirms the obvious case:

```c
void ret_none() {
  return;
}
...
ret_none();
```

Followed by `printf("OK\n")` — the test exists to confirm the function returns at all and doesn't trap on garbage in `%rax`.

### Where we are

`return;` works for void functions. The mechanism is one nullable field on `ND_RETURN` and a one-line guard in the codegen. The §11.11 `ND_GOTO`-with-partial-fields pattern repeats; the node kind taxonomy stays where it was.

---

## 13.6 — Static global variables

> `git checkout eb85527656f77b9532f3a78cefde7a2eb739189e` — *Add static global variables*

The flip side of §13.3. `static` at file scope means *internal linkage* — the name is visible only inside this translation unit, and other units can't refer to it even with `extern`. The codegen marker for internal linkage in GNU assembler syntax is `.local` instead of `.globl`. The commit is two parser lines plus four codegen lines.

The parser:

```c
Obj *var = new_gvar(get_ident(ty->name), ty);
var->is_definition = !attr->is_extern;
var->is_static = attr->is_static;
```

Reading `attr->is_static` off the `VarAttr` and stashing it on the `Obj`. The flag also defaults to `true` in `new_gvar`:

```c
static Obj *new_gvar(char *name, Type *ty) {
  Obj *var = new_var(name, ty);
  var->next = globals;
  var->is_static = true;
  var->is_definition = true;
  globals = var;
  return var;
}
```

The default-static is the subtle move. Every global created through `new_gvar` starts as static; `global_variable` overwrites it back to `attr->is_static` (which is `true` only when the user wrote `static`, false otherwise). What's the point of the round trip?

The point is the *anonymous globals*. `new_anon_gvar` (used by string literals since §7.x, by static locals in §13.3, and by file-scope compound literals in §13.4) calls `new_gvar` directly and never goes through `global_variable`. With default-static, every anonymous global gets `.local` automatically — which is what you want, because their `.L..N` names exist only inside the translation unit anyway. Without the default, the codegen would emit `.globl .L..0` for every string literal, exposing internal symbols at link time. The default is a cheap way to get the right behavior for the unnamed cases without writing a separate code path.

The codegen:

```c
if (var->is_static)
  println("  .local %s", var->name);
else
  println("  .globl %s", var->name);
```

`.local` and `.globl` are assembler directives, not assembly instructions. `.globl` exports a symbol from this object file; `.local` keeps it internal. The two are mutually exclusive — every defined global gets exactly one of them, and the linker uses the choice to decide whether the symbol participates in cross-object resolution.

Bundling §13.3 (static locals) and §13.6 (static globals) into one section was tempting and rejected. They share a keyword and an `attr` flag, and the storage characteristics overlap (static locals are file-internal because their names are `.L..N` and `.L..N` is a local symbol by default). But the two commits answer different questions. §13.3 asks "how do we make a function-scoped name back to a piece of global storage?" and the answer is `new_anon_gvar` plus `push_scope`. §13.6 asks "how do we make a file-scoped global invisible to other translation units?" and the answer is `.local` instead of `.globl`. The mechanisms don't overlap. Treating them as one section would obscure the local-vs-global split that the chapter is otherwise tracking carefully.

The test addition is one global at file scope:

```c
static int g3 = 3;
...
ASSERT(3, g3);
```

The assertion is local to the same translation unit. There's no cross-file test of static-globals' invisibility; that would require a deliberate link failure, which the test runner isn't set up to expect. The current test confirms only that `static int g3` *works inside its own file*. The cross-file invisibility is a property of the assembler output that has to be inspected by hand (or trusted by reading the diff).

### Where we are

Static globals work. The parser sets `is_static` on the `Obj`; the codegen emits `.local` for static and `.globl` for external. The default-static-in-`new_gvar` is the structural choice that makes anonymous globals (string literals, static-local backings, file-scope compound literals) all get `.local` for free. The C linkage model — external (default for explicit globals), internal (`static`), none (block-scope locals) — is now fully representable on the global path of the codegen.

---

## 13.7 — `do … while`

> `git checkout ee252e6ce79d752526504cf034fd41f070191824` — *Add do ... while*

The fourth loop construct. `for` and `while` were the first two (§3.x and §3.x); §11.11 extended both with `break` and `continue`. `do … while` is the post-test loop: the body always runs at least once, and the condition is evaluated after the body, with the loop repeating only while the condition is non-zero.

A new node kind, because `do … while` doesn't fit the `for`/`while` shape. `ND_FOR` has init/cond/inc/then; `ND_DO` needs only cond/then plus the break/continue labels:

```c
typedef enum {
  ...
  ND_FOR,       // "for" or "while"
  ND_DO,        // "do"
  ...
} NodeKind;
```

The parser:

```c
if (equal(tok, "do")) {
  Node *node = new_node(ND_DO, tok);

  char *brk = brk_label;
  char *cont = cont_label;
  brk_label = node->brk_label = new_unique_name();
  cont_label = node->cont_label = new_unique_name();

  node->then = stmt(&tok, tok->next);

  brk_label = brk;
  cont_label = cont;

  tok = skip(tok, "while");
  tok = skip(tok, "(");
  node->cond = expr(&tok, tok);
  tok = skip(tok, ")");
  *rest = skip(tok, ";");
  return node;
}
```

The save-and-restore of `brk_label`/`cont_label` is the same shape §11.11 used for the other loops. `break` jumps to `brk_label` (which is set to the label after the loop); `continue` jumps to `cont_label` (which is set to the start of the per-iteration condition check). Each loop construct nests by saving the outer labels on entry and restoring on exit.

The codegen:

```c
case ND_DO: {
  int c = count();
  println(".L.begin.%d:", c);
  gen_stmt(node->then);
  println("%s:", node->cont_label);
  gen_expr(node->cond);
  println("  cmp $0, %%rax");
  println("  jne .L.begin.%d", c);
  println("%s:", node->brk_label);
  return;
}
```

Six instructions and three labels. `.L.begin.N` marks the body. The body runs unconditionally (no entry condition check). `cont_label` marks the per-iteration condition site, so `continue` skips the rest of the body and jumps to the condition. The condition lands in `%rax`; `cmp $0, %rax; jne .L.begin.N` jumps back to the body if the condition is non-zero. `brk_label` is after the conditional jump, so `break` exits the loop entirely and `cmp/jne` falling through reaches the same place.

The test pins both the basic shape and the `break`/`continue` interaction:

```c
ASSERT(7, ({ int i=0; int j=0; do { j++; } while (i++ < 6); j; }));
ASSERT(4, ({ int i=0; int j=0; int k=0; do { if (++j > 3) break; continue; k++; } while (1); j; }));
```

The first counts 7 iterations: `i` runs 0, 1, 2, 3, 4, 5, 6, and the loop exits when the post-increment pushes `i` past 6. The second tests that `break` bails out at the right value (4) and that `continue` skips the unreachable `k++` (which is dead code anyway after the `continue`, but the test is checking that the flow-control labels work).

### Where we are

Four loop constructs: `for`, `while`, `do … while`, and the desugared-from-`while` case. Each has its own `gen_stmt` arm, all with the same break/continue label discipline. The chapter doesn't introduce a new control-flow primitive — `do … while` is built from the pieces §11.11 already had.

---

## 13.8 — 16-byte stack alignment

> `git checkout 6a0ed71107670b404af04bc20a2461165483f390` — *Align stack frame to 16 byte boundaries*

A nine-line codegen patch with an outsized impact. Until this commit, chibicc-emitted code happened to work on most x86-64 inputs but had a latent bug at every function call: `%rsp` was not guaranteed to be 16-byte aligned at the call site, which the x86-64 psABI requires.

The x86-64 psABI says: at the moment a `call` instruction executes, `%rsp` must be a multiple of 16. The `call` instruction itself pushes the 8-byte return address, leaving `%rsp` *eight bytes off* a 16-byte boundary at the start of the called function. The called function's prologue (`push %rbp`, etc.) brings it back to alignment. The `printf` family in glibc relies on this — internal SSE moves require 16-byte-aligned stack slots, and a misaligned call eventually traps inside the variadic-argument unpacking.

Chibicc's codegen pushes 8-byte values at every operand-stack push (`push %rax`), so `depth` (a static variable in the codegen tracking the operand stack's height in 8-byte slots) tells you the parity. If `depth` is even, `%rsp` is 16-byte aligned. If `depth` is odd, `%rsp` is 8 bytes off.

The fix:

```c
println("  mov $0, %%rax");

if (depth % 2 == 0) {
  println("  call %s", node->funcname);
} else {
  println("  sub $8, %%rsp");
  println("  call %s", node->funcname);
  println("  add $8, %%rsp");
}
```

When depth is even (already aligned), call directly. When depth is odd, push 8 bytes of padding, call, then pop the padding. The pad-call-unpad is a three-instruction sequence; a smarter compiler might align the function's stack frame as a whole, but chibicc patches per-call. The cost is two extra instructions in the odd-depth case; the benefit is correctness against the ABI.

The function prologue, separately, was already aligning the stack frame to 16 in `assign_lvar_offsets` (`fn->stack_size = align_to(offset, 16);`, since §x). The two alignments are different: frame alignment ensures the function's local-variable region is 16-byte aligned for cases where SSE locals exist; the per-call alignment ensures that *callees* see an aligned stack pointer. Both are needed.

There is no test. The alignment bug is invisible until you call into something that requires it (a variadic glibc function, an SSE-using library), and the test suite up to this point hasn't exercised any such path. The commit is a *correctness preemption* — Rui clearly noticed the issue while reading the psABI rather than while debugging a failing test.

This is also the chapter's first instance of chibicc-as-real-compiler concerns. Until §12.x, the calling convention was treated as an interface-with-glibc detail that could be approximated. The 16-byte alignment is the first place chibicc cares about ABI invariants that aren't visible from C semantics. Variadics in Chapter 14 will go further down the same road.

### Where we are

Calls into other code obey the x86-64 psABI's 16-byte stack alignment. The mechanism is a parity check on the operand-stack depth at each call site, with a pad-call-unpad sequence when the depth is odd. The fix has no semantic effect on chibicc-only programs (which mostly didn't trigger the bug); it matters at the boundary with glibc's printf, which the test suite calls heavily.

---

## 13.9 — Truncating small return values

> `git checkout dcd45792264795a32f19581a904dda8bf6d3ad06` — *Handle a function returning bool, char or short*

Another psABI-correctness patch. Functions in x86-64 return integer values in `%rax`. The psABI says that for return types narrower than 64 bits, only the low N bits of `%rax` are guaranteed to hold the return value; the high bits may contain garbage. The caller is responsible for sign-extending (or zero-extending, for unsigned types) the result if it needs the full 64 bits.

Chibicc, before this commit, was treating `%rax` as if it always contained the right value. A function declared `bool f()` that did `return 512;` could leave `%rax = 512`, which the caller would compare against zero and conclude `true` — but the standard says the caller should be looking at only the low byte, where `512` is `0` (since 512 = 0x200, and the low byte is `0`).

The fix is in the funcall codegen, after the `call` returns:

```c
// It looks like the most significant 48 or 56 bits in RAX may
// contain garbage if a function return type is short or bool/char,
// respectively. We clear the upper bits here.
switch (node->ty->kind) {
case TY_BOOL:
  println("  movzx %%al, %%eax");
  return;
case TY_CHAR:
  println("  movsbl %%al, %%eax");
  return;
case TY_SHORT:
  println("  movswl %%ax, %%eax");
  return;
}
```

Three instructions, one per type:

- `movzx %al, %eax` — zero-extend `%al` (the low byte of `%rax`) into `%eax`. `_Bool` is unsigned by definition, so zero-extension is correct.
- `movsbl %al, %eax` — sign-extend `%al` into `%eax`. `char` in chibicc is signed (since §10.x), so sign-extension matches.
- `movswl %ax, %eax` — sign-extend `%ax` (the low 16 bits) into `%eax`. `short` is signed.

A 32-bit move into `%eax` automatically zero-extends to `%rax` (an x86-64 quirk that distinguishes 32-bit register writes from 8/16/64-bit ones). So after any of the three, `%rax` contains the canonical truncated/extended return value.

The test setup is fiddly:

```c
int false_fn() { return 512; }
int true_fn() { return 513; }
int char_fn() { return (2<<8)+3; }
int short_fn() { return (2<<16)+5; }
```

Four C functions in `test/common`, each returning a value with garbage in the high bits. From `test/function.c`:

```c
_Bool true_fn();
_Bool false_fn();
char char_fn();
short short_fn();
...
ASSERT(1, true_fn());
ASSERT(0, false_fn());
ASSERT(3, char_fn());
ASSERT(5, short_fn());
```

The trick is that the *callee* returns an `int` (`512` or `513`) but the *caller* declares the return type as `_Bool` (or `char`, or `short`). The caller's compiled code uses the small-return truncation arm; the callee's compiled code returns the full integer in `%rax`. Without the truncation, `_Bool true_fn()` would compare `513` to `0` and return `1`; `_Bool false_fn()` would compare `512` to `0` and *also* return `1` — which is wrong. With the truncation, `false_fn()`'s low byte is `0x00`, so the result is `0`. With the same logic, `char_fn()` returns `(2<<8)+3 = 515`, low byte `0x03`, result `3`. `short_fn()` returns `(2<<16)+5 = 131077`, low 16 bits `0x0005`, result `5`.

The parallel to §10.12 is real but loose. §10.12 added the `_Bool` cast inside `gen_expr` — when an expression of integer type is cast to `_Bool`, the cast generates a `cmp $0, %rax; setne %al; movzx %al, %rax` sequence to canonicalize to 0 or 1. §13.9 does *not* do that for the return path; it does a simple zero-extend. The asymmetry is correct: a `_Bool`-returning function is supposed to have already canonicalized its return value to 0 or 1 inside the callee (via the `_Bool` cast on the `return expr;`), and the caller's job is only to clear garbage from the high bits. The test passes because `false_fn` happens to return 512 (low byte 0) — chibicc's truncation is doing the right thing not because of full canonicalization but because of the psABI's narrow-return semantics.

There's also a stray parser change in the same commit:

```diff
-// func-params = ("void" | param ("," param)*)? ")"
+// func-params = ("void" | param ("," param)*?)? ")"
```

A grammar comment fix — adding `?` after the trailing `*` to indicate the whole `param-list` is optional. No functional change; a documentation tightening. Worth noting only because the commit's title doesn't mention it.

### Where we are

Functions returning `bool`, `char`, or `short` have their return values correctly truncated and extended at the call site. The mechanism is a three-arm switch in the funcall codegen, parallel in shape to (but smaller than) §10.12's `_Bool` cast. Combined with §13.8's stack alignment, chibicc-emitted code now obeys the two main x86-64 psABI invariants that show up in glibc-call boundaries.

---

## Where the chapter leaves us

Eleven commits, nine sections, and a stack of new pieces. Half of them are about linkage; the other half are nearby cleanups that share parts of the same machinery.

| Commit | Topic |
|---|---|
| `006a45c` | `extern` at file scope. Third `VarAttr` flag (`is_extern`); `is_definition` flag on `Obj`; codegen skips non-definitions. First multi-translation-unit data linking in the test suite. |
| `2764745` | `extern` at block scope. Routes through `global_variable` from `compound_stmt`. |
| `9df5178` | `_Alignof` and `_Alignas`. `align` field on `Obj` and `Member`; alignment migrates from per-type to per-variable; fourth `VarAttr` field; `.align` directive in §12.11 now sourced from per-variable field. |
| `310a87e` | `_Alignof(x)` GNU extension. Lookahead-on-`(` arm; second arm parses a unary expression and reads its type's align. |
| `319772b` | Static local variables. `new_anon_gvar` + `push_scope` + `gvar_initializer`. The local-vs-global split blurs. |
| `127056d` | Compound literals. Anonymous variable backs the literal: `.L..N` global at file scope, empty-named lvar at block scope. Cast/literal disambiguator in `cast`. |
| `30b3e21` | Bare `return;`. Nullable `lhs` on `ND_RETURN`. |
| `eb85527` | Static global variables. `is_static` flag on `Obj`; `.local` instead of `.globl` for static. Default-static in `new_gvar` makes anonymous globals work for free. |
| `ee252e6` | `do … while`. New `ND_DO` kind with cond/then/brk/cont fields. |
| `6a0ed71` | 16-byte stack alignment at call sites. Parity check on operand-stack depth; pad-call-unpad when odd. |
| `dcd4579` | Truncate `bool`/`char`/`short` return values at call site. Three-arm switch with `movzx`/`movsbl`/`movswl`. |

Three structural moves deserve calling out.

The first is the *VarAttr channel filling out*. Chapter 11 had `is_typedef` and `is_static`. Chapter 12 left it at two. Chapter 13 adds `is_extern` (§13.1) and `align` (§13.2), bringing the record to four fields. The channel was forecast as the right place for these in the previous two handoffs; both predictions land here. The channel is also the place where the upcoming `signed`/`unsigned`/`const`/`volatile`/`restrict` qualifiers from Chapter 14 will route — the fifth, sixth, seventh, eighth, and ninth fields are all queued up. The channel's stable shape — a small struct passed by pointer through `declspec`, populated as keywords are recognized, consumed by callers — has been right since §10.x.

The second is the *anonymous-global pattern*. Three sections in this chapter (§13.3 static locals, §13.4 compound literals, §13.6 static globals) all hinge on `new_anon_gvar`: a global with a synthesized `.L..N` name that the C-level scope chain doesn't see. The pattern existed since Chapter 7's string literals, but the chapter is the first time it's load-bearing for user-facing features. The default-static-in-`new_gvar` choice in §13.6 is the structural detail that makes all three uses work uniformly: anonymous globals get `.local` for free, definitions of explicit static globals get `.local` because the parser sets the flag, and explicit non-static globals get `.globl` because `global_variable` overwrites the default.

The third is the *psABI conformance pass*. §13.8 (16-byte stack alignment) and §13.9 (small-return truncation) are corrections, not features — chibicc's compiled output was technically out of spec before these landed. Neither is visible from the test suite without crossing into glibc-using code; both are the kind of correctness work that compilers accumulate as they grow. Chapter 14 will continue the psABI thread with variadic argument handling.

The chapter doesn't add a fifth namespace. (The four namespaces — labels and struct-tags and ordinary identifiers and the typedef/enum identifier population — are all the same as Chapter 12.) The chapter doesn't add to the canonicalization-at-parse-time count (still eight; compound literals were considered and not counted, on the same grounds as initializer lowering). The chapter adds zero pre-factor-before-feature instances (§11.9's incomplete-array sentinel was the last one; the count stays at six).

A few standing notes carried forward to Chapter 14:

- The *VarAttr channel* now carries four fields (`is_typedef`, `is_static`, `is_extern`, `align`). Chapter 14 will add `signed`/`unsigned` (probably as one mutually-exclusive channel) and the qualifier set.
- The *anonymous-global pattern* (`new_anon_gvar` + optional `push_scope`) is the chapter's most-reused piece of machinery. Chapter 14 may not need it directly; later chapters with floating-point literals likely will.
- The *Initializer tree* is unchanged. Compound literals reused it without extending it.
- The *constant-expression evaluator* (`eval`/`eval2`/`eval_rval`) gets a third caller in `_Alignof` and a fourth in `_Alignas(N)`'s `const_expr`. The shape stays the same.
- The *local-versus-global split* survived the static-locals blur. The codegen's `globals` list still drives `emit_data`; the C-level scope chain is what tells you the name's visibility, and the two are now decoupled enough that a name can be on the global list with only function-scope visibility.
- The *psABI conformance thread* opens with §13.8 and §13.9. Chapter 14 will continue with variadic argument handling (`%al` zeroing, the `va_start` / `va_arg` mechanism), which has been a TODO since Chapter 5.
- The *`Relocation` mechanism* from §12.7 picks up a new use case in §13.4: address-of-anonymous-global initializers (the `&(Tree){...}` test). The mechanism doesn't extend; only the source of the labels broadens.
- The *`is_definition` flag* on `Obj` is a new persistent piece of state. Used today only by `extern`; will be used by Chapter 17's preprocessor when handling `#include`-introduced declarations.
- The *`is_static` default in `new_gvar`* is a load-bearing default. Anonymous globals (string literals, static-local backings, file-scope compound-literal backings) all rely on it. Watch for any future commit that adds a new caller of `new_gvar` and forgets to override the flag.
- The *`.L..N` symbol naming convention* is the GNU assembler's hint that the symbol is local to the file. Chibicc relies on this for both correctness (anonymous globals shouldn't be visible to the linker) and aesthetics (the symbol table stays clean).
- The *fourth loop construct* (`do … while`) brings the loop family to four. Each has its own codegen arm; the brk/cont label discipline is uniform across all four.
- The *cast/compound-literal disambiguator* in `cast` (peek for `{` after `(typename)`) is a small but novel parser pattern. The lookahead-on-the-following-token shape is the same one §10.x used to disambiguate `sizeof typename` from `sizeof unary`.

Forward references for Chapter 14:

- Variadics. The `mov $0, %al` placeholder noted in Chapter 5 will become real in Chapter 14, when chibicc grows `va_start`/`va_arg`/`va_end` and starts emitting the register-save area for variadic functions.
- `signed` and `unsigned` keywords. The integer-promotion rules and the codegen's `cqo`-vs-zero-extend dichotomy will flip per type.
- Integer literal suffixes (`L`, `LL`, `U`, `LU`, …). The constant-expression evaluator from §11.15 will need to track signedness and width.
- The qualifier soup: `const`, `volatile`, `restrict`. Chibicc parses-and-discards each — the compiler doesn't enforce any of them.
- Type qualifiers on pointers. `const int *` and `int *const` will both parse; chibicc treats both as `int *`.

Twelve commits in Chapter 14, with `va_start` as the most substantive.
