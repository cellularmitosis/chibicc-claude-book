# Chapter 21 — Thread-local, alloca, VLAs

> Commits covered: `b377284`, `8f5ff07`, `ee0a951`, `4064871`, `77275c5`, `e8667af`, `07f9010`, `2fa8f48`, `b0109a3`, `bc25279`, `c32f0e2`, `8d130ab`, `d56dd2f`, `e0bf168`, `d90c73b`, `3d5550e`, `4f165ec`. Seventeen commits — thread-local storage and the `%fs:`-relative addressing it brings in, three driver options that finish off the cc1-vs-driver split (`-include`, `-x`, the `-E` implies `-xc` rule), `alloca` as a real builtin, the four-commit VLA arc, four small linker-driver additions (`-l`, `-s`, ELF size/type emission, `.a`/`.so` recognition), and finally `long double` plus three GNU bracket-range features (`case 1 ... 5:`, `[3 ... 7] = x`, `&&label` and `goto *expr`).

Through Chapter 20 chibicc handles the gcc-extension surface that real-world C reaches for most often. What it doesn't yet handle, and what real-world code sometimes still wants, is thread-local storage, runtime-sized stack allocation (`alloca`), and variable-length arrays. None of these is a polish-level addition — each lands a new mechanism in the codegen. Thread-local storage requires `%fs:`-relative addressing and two new ELF sections. `alloca` requires the function epilogue to forget that it ever knew the post-prologue stack size. VLAs require the parser to invent hidden locals to remember runtime sizes, and `sizeof` has to know to read those locals instead of returning a compile-time constant.

The chapter also closes one long-standing errata candidate (`long double` finally becomes extended-precision rather than aliased to `double`) and another that has been outstanding since Chapter 19 (array range designators are finally honored in elaboration). And it adds the GNU bracket-range pattern in two more places: `case 1 ... 5:` and `&&label` for labels-as-values.

Six sections from seventeen commits.

- **§21.1** — Thread-local variables (commit 267).
- **§21.2** — Driver: `-include`, `-x`, `-E` implies `-xc` (commits 268–270).
- **§21.3** — `alloca` (commit 271).
- **§21.4** — VLAs: `sizeof(VLA)`, pointer arithmetic, `sizeof(typename)`, dropping `__STDC_NO_VLA__` (commits 272–275).
- **§21.5** — Linker driver: `-l`, `-s`, ELF size/type, `.a`/`.so` (commits 276–279).
- **§21.6** — `long double`, case ranges, array range designators, labels-as-values (commits 280–283).

The chapter follows `main` order. As before, calendar order and `main` order differ — the four VLA commits land at positions 272–275 but were written in three separate days across August 25 (`2fa8f48`, `b0109a3`) and September 3–4 (`07f9010`, `e8667af`), and the chapter does not remark on this except to note that the order in `main` is the order in which we walk the work.

---

## 21.1 — Thread-local variables

> `git checkout b3772845bd07fb695ca6b6e67ad7640776ae0f6c` — *Add thread-local variable*

C11 added the `_Thread_local` storage-class specifier (and gcc had long supported `__thread` as an extension): a variable so qualified gets a separate instance for each thread of execution. Reads and writes from one thread don't see the value the other thread has stored. The mechanism on x86-64 Linux is the `%fs` segment register, which the loader and pthread runtime point at the current thread's TLS block.

Rui's commit adds both keywords (`_Thread_local` and `__thread`), threads them through `VarAttr` and `Obj`, and emits the right addressing pattern in codegen.

### The keyword pair and the `VarAttr` slot

`is_keyword` and `is_typename` both grow two entries:

```c
"typeof", "asm", "_Thread_local", "__thread",
```

`VarAttr` gains a fifth boolean:

```c
typedef struct {
  bool is_typedef;
  bool is_static;
  bool is_extern;
  bool is_inline;
  bool is_tls;
  int align;
} VarAttr;
```

The `declspec` storage-class branch already handled `typedef`, `static`, `extern`, and `inline` as a four-way exclusion — the existing code rejects combinations that aren't allowed. Thread-local enters the same channel, but with the relaxed rule that `_Thread_local` may *combine* with `static` or `extern` (a TLS variable can have either internal or external linkage). The diagnostic check picks up TLS as another mutual-exclusion partner with `typedef`:

```c
if (attr->is_typedef &&
    attr->is_static + attr->is_extern + attr->is_inline + attr->is_tls > 1)
  error_tok(tok, "typedef may not be used together with static,"
            " extern, inline, __thread or _Thread_local");
```

That's enough to admit `static _Thread_local int x;` (which is what real-world code most often writes). The standard's full set of rules for storage-class combinations isn't reproduced; chibicc accepts what gcc accepts in practice and rejects the worst combinations.

### `Obj->is_tls` and the global-variable wiring

`Obj` gains an `is_tls` flag, sitting next to `is_tentative`:

```c
struct Obj {
  ...
  // Global variable
  bool is_tentative;
  bool is_tls;
  char *init_data;
  Relocation *rel;
  ...
};
```

The flag is set by `global_variable` from the `attr` channel, and there's a one-line behavioral change: a TLS variable is *not* tentative even without an initializer. Tentative definitions are an artifact of the `.bss`/`.comm` machinery; thread-local globals don't share that path.

```c
var->is_definition = !attr->is_extern;
var->is_static = attr->is_static;
var->is_tls = attr->is_tls;
...
if (equal(tok, "="))
  gvar_initializer(&tok, tok->next, var);
else if (!attr->is_extern && !attr->is_tls)
  var->is_tentative = true;
```

A bare `_Thread_local int x;` therefore goes straight to `.tbss` (the zero-initialized TLS section) rather than to `.comm`.

### `%fs:0` and `@tpoff`

The address-generation half is the chapter's first new asm pattern. `gen_addr` learns to recognize a TLS variable and emit a two-instruction sequence:

```c
// Thread-local variable
if (node->var->is_tls) {
  println("  mov %%fs:0, %%rax");
  println("  add $%s@tpoff, %%rax", node->var->name);
  return;
}
```

`mov %fs:0, %rax` reads the per-thread base address from the segment register. The TLS implementation Linux uses places a small bookkeeping structure at `%fs:0` whose first slot is a self-pointer back to the TLS block. The result is a thread-specific pointer which `add $name@tpoff` then offsets into.

`@tpoff` is the assembler relocation for "thread-pointer offset." The linker computes the symbol's offset from the thread-pointer base at link time; the runtime adds that constant to whatever the segment register provides. There are several TLS access models (initial-exec, local-exec, general-dynamic, local-dynamic) with different cost/flexibility tradeoffs; the pattern chibicc emits is the *initial-exec* model, which is what gcc emits by default for TLS variables compiled into the executable. It's the cheapest model — two instructions, no library call — and works as long as the TLS block is statically allocated at program startup. Code in dynamically loaded shared libraries needs a more elaborate dance (`__tls_get_addr` and the global-dynamic model). Chibicc emits only initial-exec, which is sufficient for the things chibicc compiles.

### `.tdata` and `.tbss`

`emit_data` learns two new section directives:

```c
// .data or .tdata
if (var->init_data) {
  if (var->is_tls)
    println("  .section .tdata,\"awT\",@progbits");
  else
    println("  .data");
  ...
}

// .bss or .tbss
if (var->is_tls)
  println("  .section .tbss,\"awT\",@nobits");
else
  println("  .bss");
```

The section flags `awT` mark the section as allocatable, writable, and TLS (the `T` is the TLS bit). `@progbits` versus `@nobits` is the same distinction `.data` and `.bss` make: `.tdata` carries initialized template data; `.tbss` is zero-fill. The loader copies both into per-thread storage at thread creation; `_Thread_local int x = 5;` becomes a `.tdata` entry that every thread initializes to 5 on creation.

This is also where the count of assembly sections chibicc can emit grows. Through Chapter 20 the count was three (`.text`, `.data`, `.bss`) plus the `.comm` directive for tentative commons. After this commit it's five — `.text`, `.data`, `.bss`, `.tdata`, `.tbss` — plus `.comm`.

### `__STDC_NO_THREADS__` and the test plumbing

Chibicc previously defined `__STDC_NO_THREADS__` to declare itself non-thread-aware. The flag's removal here is honest: chibicc doesn't implement `<threads.h>` (the C11 thread library), but the underlying compiler now emits TLS, which is the language piece. `<threads.h>` is a libc concern, not a compiler concern, and pthreads (which the test uses) is a separate POSIX interface that chibicc doesn't need to know about at the language level.

```c
-  define_macro("__STDC_NO_THREADS__", "1");
```

The `Makefile` change adds `-pthread` to the link command for tests, so the new `test/tls.c` (which uses `pthread_create` to verify thread-local separation) links against libpthread.

### The test: thread separation

`test/tls.c` is the smallest test that proves the feature works:

```c
_Thread_local int v1;
_Thread_local int v2 = 5;
int v3 = 7;

int thread_main(void *unused) {
  ASSERT(0, v1);   // child sees fresh zero
  ASSERT(5, v2);   // child sees fresh initialized value
  ASSERT(7, v3);   // shared global
  v1 = 1; v2 = 2; v3 = 3;
  ...
}

int main() {
  ...
  ASSERT(0, pthread_create(&thr, NULL, thread_main, NULL));
  ASSERT(0, pthread_join(thr, NULL));
  ASSERT(0, v1);   // main's v1 unchanged
  ASSERT(5, v2);   // main's v2 unchanged
  ASSERT(3, v3);   // shared global was modified
}
```

The two TLS variables retain their per-thread initial values in `main` even after the child has overwritten its copies; the non-TLS `v3` shows the shared-state baseline. That's the entire semantic surface.

**Where we are.** `_Thread_local` and `__thread` are accepted as storage-class specifiers. TLS variables are placed in `.tdata` (initialized) or `.tbss` (zero-fill); accesses use `%fs:0 + name@tpoff`, which is the initial-exec model. Dynamic-library TLS access (which would call `__tls_get_addr`) is not implemented. `__STDC_NO_THREADS__` is no longer defined — chibicc no longer claims to lack threads at the language level, even though it doesn't implement `<threads.h>`. The psABI conformance count ticks up by one: TLS access patterns are part of the psABI's thread-local model.

---

## 21.2 — Driver: `-include`, `-x`, and `-E` implies `-xc`

> `git checkout 8f5ff07dc08d258209adf60ed8e796efa7b7a476` — *Add -include option*
>
> `git checkout ee0a951b30646023ccc9a144afb4b380bf8d09b1` — *Add -x option*
>
> `git checkout 4064871212049d82af3632941d15e6a0757ebc3c` — *Make -E to imply -xc*

Three driver-side commits. They deal with two adjacent ergonomic problems: getting headers into the input without rewriting the source (`-include`), and naming the input language explicitly when extension-based detection won't do (`-x`).

### 21.2.1 — `-include`: prepended #includes

`-include FILE` tells the compiler to behave as if `#include "FILE"` appeared at the top of the input. gcc uses it for project-wide preludes and for selectively defining feature macros. The implementation is straightforward — collect `-include` paths in a `StringArray`, then prepend tokenized copies of each file before the main input.

The driver gains `opt_include`:

```c
static StringArray opt_include;
```

Argument parsing accepts `-include FILE` (the option needs a separate argument; `take_arg` learns the new name):

```c
static bool take_arg(char *arg) {
  char *x[] = {"-o", "-I", "-idirafter", "-include"};
  ...
}
```

```c
if (!strcmp(argv[i], "-include")) {
  strarray_push(&opt_include, argv[++i]);
  continue;
}
```

The interesting half is in `cc1`. Previously it did `tokenize_file(base_file)` and hand-waved the rest. Now it processes `-include` first and chains the resulting token streams:

```c
static void cc1(void) {
  Token *tok = NULL;

  // Process -include option
  for (int i = 0; i < opt_include.len; i++) {
    char *incl = opt_include.data[i];

    char *path;
    if (file_exists(incl)) {
      path = incl;
    } else {
      path = search_include_paths(incl);
      if (!path)
        error("-include: %s: %s", incl, strerror(errno));
    }

    Token *tok2 = must_tokenize_file(path);
    tok = append_tokens(tok, tok2);
  }

  Token *tok2 = must_tokenize_file(base_file);
  tok = append_tokens(tok, tok2);
  tok = preprocess(tok);
  ...
}
```

Two helpers fall out: `must_tokenize_file` (tokenize-or-die) and `append_tokens` (concatenate two streams, dropping the terminating `TK_EOF` of the first). Note the path resolution: a `-include` argument that looks like a file path is taken verbatim if it exists; otherwise it's resolved through `search_include_paths`, which is now exposed in `chibicc.h` because `cc1` lives in `main.c` and the search function lives in `preprocess.c`.

Token streams concatenated this way are preprocessed as a single unit. A `#define` in a `-include` file is visible to the main translation unit, and so are any tokenizer-state effects (though there shouldn't be any — the tokenizer is stateless apart from line numbering, which it tracks per-file).

### 21.2.2 — `-x`: language override

`-x LANG` overrides the file-extension-based language detection that `cc1` previously did with `endswith(input, ".c")` and `endswith(input, ".s")`. The motivation is twofold: stdin (`-`) has no extension; and you sometimes want to compile a file as if it had a different extension (`-x assembler` to treat a `.S` input as raw assembly without the C preprocessor, for example).

The valid arguments are `c`, `assembler`, and `none`. `none` resets the override so that subsequent inputs go back to extension-based detection, which means a single command line can mix languages:

```
$chibicc -c -x assembler $tmp/foo.s -x none $tmp/bar.c
```

The implementation introduces a `FileType` enum:

```c
typedef enum { FILE_NONE, FILE_C, FILE_ASM, FILE_OBJ } FileType;
```

and an `opt_x` global. Argument parsing handles both the spaced (`-x c`) and the joined (`-xc`) forms:

```c
if (!strcmp(argv[i], "-x")) {
  opt_x = parse_opt_x(argv[++i]);
  continue;
}

if (!strncmp(argv[i], "-x", 2)) {
  opt_x = parse_opt_x(argv[i] + 2);
  continue;
}
```

The joined form is what gcc accepts and what real Makefiles tend to use. The spaced form has to be matched first because `-x` is on the `take_arg` list.

`get_file_type` is now the centralized lookup:

```c
static FileType get_file_type(char *filename) {
  if (endswith(filename, ".o"))
    return FILE_OBJ;
  if (opt_x != FILE_NONE)
    return opt_x;
  if (endswith(filename, ".c"))
    return FILE_C;
  if (endswith(filename, ".s"))
    return FILE_ASM;
  error("<command line>: unknown file extension: %s", filename);
}
```

The `.o` case takes precedence over `opt_x` — a compiled object file is an object file regardless of any `-x` setting.

`main`'s per-input loop then dispatches on the result:

```c
FileType type = get_file_type(input);
if (type == FILE_OBJ) { ... }
if (type == FILE_ASM) { ... }
assert(type == FILE_C);
```

Note that the previous "no extension; assume C" hack — the explicit `if (!endswith(input, ".c") && strcmp(input, "-"))` — disappears. Stdin no longer falls through magic; it's explicitly handled with `-x` (or by the `-E implies -xc` rule below).

The test driver picks up an awkward consequence: every test that pipes into chibicc now needs `-xc -` so the driver knows the input is C. That's the bulk of the `test/driver.sh` diff — twenty-odd lines change from `... | $chibicc -E - | ...` to `... | $chibicc -E -xc - | ...`. It's a churn cost paid in test infrastructure for a clean cc1 abstraction.

### 21.2.3 — `-E` implies `-xc`

Three commits later (about a week of calendar time), Rui revisits the test churn from `-x` and adds a small ergonomic rule: if `-E` is given, the input is assumed to be C. That makes sense — `-E` means "preprocess only" and the C macro language is what the preprocessor handles.

```c
// -E implies that the input is the C macro language.
if (opt_E)
  opt_x = FILE_C;
```

The change reverses the test-driver churn for the `-E` path: `echo foo | $chibicc -E -` works again.

**Where we are.** Three driver options. `-include FILE` prepends a tokenized include file before the main input; multiple `-include` arguments stack in command-line order. `-x LANG` overrides extension-based detection for subsequent inputs (`c`, `assembler`, `none`). `-E` implies `-xc` so that preprocessing-only mode keeps working on stdin. The pre-existing `cc1` hack of "if there's no extension and the input isn't `-`, error out" is gone — language detection is centralized in `get_file_type`.

---

## 21.3 — `alloca`

> `git checkout 77275c546a5340f94ad011cd759ef162bc714ba6` — *Add alloca()*

`alloca(n)` allocates `n` bytes on the stack and returns a pointer to the allocation. It's a long-standing extension (GCC, BSD libc, glibc), not a standard library function — the C standard takes no position on it because no portable way to implement it within standard C exists. Its appeal: very fast allocation (a stack-pointer adjustment) with automatic deallocation at function return. Its risks: stack exhaustion if `n` is unbounded, and undefined behavior if the result outlives the caller.

Chibicc's implementation is a builtin. The compiler recognizes calls to a function named `alloca` and emits an inline asm sequence rather than a real function call.

### The builtin declaration

Before parsing starts, the compiler synthesizes a declaration for `alloca`:

```c
static void declare_builtin_functions(void) {
  Type *ty = func_type(pointer_to(ty_void));
  ty->params = copy_type(ty_int);
  Obj *builtin = new_gvar("alloca", ty);
  builtin->is_definition = false;
}
```

A `void *alloca(int)` is added to globals as a non-definition. Programs that declare `void *alloca(size_t);` themselves shadow this gracefully — the typechecking comes from the program's declaration. The builtin is just there so an unprefixed `alloca(n)` doesn't fail to resolve.

The reason the builtin is registered as a global rather than recognized by name in codegen alone is that `alloca`'s call site needs to typecheck like any other function call — `alloca(16)` returns a `void *`, the argument is an integer expression, and the parser needs `func_ty` and `return_ty` plumbed correctly. Codegen then specializes when it recognizes the call.

### The codegen specialization

`gen_expr`'s `ND_FUNCALL` case learns to recognize the builtin:

```c
case ND_FUNCALL: {
  if (node->lhs->kind == ND_VAR && !strcmp(node->lhs->var->name, "alloca")) {
    gen_expr(node->args);
    println("  mov %%rax, %%rdi");
    builtin_alloca();
    return;
  }
  ...
}
```

The argument is evaluated into `%rax`, copied into `%rdi`, and the helper emits the allocation sequence inline.

### `builtin_alloca` and the `alloca_bottom` cell

The hard part is making the result interact correctly with the rest of the function. chibicc's stack frame currently holds locals at fixed offsets from `%rbp`, so `%rsp` between the prologue and a return is normally fixed. `alloca` has to grow the frame downward at runtime, which means the *temporary* area chibicc uses for evaluation pushes (between `%rsp` and the locals) has to move to make room — without disturbing any pushed values that have meaningful contents.

Rui's approach: for each function, reserve a hidden local that records the current bottom of the temporary area. Call this `alloca_bottom`. When we want to allocate `n` bytes, slide the contents of the temporary area down by `n` aligned bytes and update `alloca_bottom` to remember the new boundary. Future allocations slide again.

The hidden local is added in `function`:

```c
fn->alloca_bottom = new_lvar("__alloca_size__", pointer_to(ty_char));
```

`Obj` gets a new field:

```c
Obj *alloca_bottom;
```

and the prologue emits a store of the post-prologue `%rsp` into the local:

```c
println("  push %%rbp");
println("  mov %%rsp, %%rbp");
println("  sub $%d, %%rsp", fn->stack_size);
println("  mov %%rsp, %d(%%rbp)", fn->alloca_bottom->offset);
```

After the prologue, the cell holds the current bottom of "memory below `%rsp` is yours to allocate." Initially that's the same as `%rsp` because nothing has been pushed for evaluation yet.

`builtin_alloca` is then a small in-line assembler that does three things: align the allocation size to 16 bytes, copy the existing temporary area down by that many bytes, and update `alloca_bottom`:

```c
static void builtin_alloca(void) {
  // Align size to 16 bytes.
  println("  add $15, %%rdi");
  println("  and $0xfffffff0, %%edi");

  // Shift the temporary area by %rdi.
  println("  mov %d(%%rbp), %%rcx", current_fn->alloca_bottom->offset);
  println("  sub %%rsp, %%rcx");
  println("  mov %%rsp, %%rax");
  println("  sub %%rdi, %%rsp");
  println("  mov %%rsp, %%rdx");
  println("1:");
  println("  cmp $0, %%rcx");
  println("  je 2f");
  println("  mov (%%rax), %%r8b");
  println("  mov %%r8b, (%%rdx)");
  println("  inc %%rdx");
  println("  inc %%rax");
  println("  dec %%rcx");
  println("  jmp 1b");
  println("2:");

  // Move alloca_bottom pointer.
  println("  mov %d(%%rbp), %%rax", current_fn->alloca_bottom->offset);
  println("  sub %%rdi, %%rax");
  println("  mov %%rax, %d(%%rbp)", current_fn->alloca_bottom->offset);
}
```

The byte-by-byte copy is the bookkeeping: any value chibicc has previously pushed for evaluation (between `%rsp` and `alloca_bottom`) needs to live at the *new* bottom after the slide, not the old one. The new allocation goes into the gap that opens up between the new `%rsp` and where the old `%rsp` was. The returned pointer is left in `%rax` — `alloca_bottom - aligned_n`, which is the start of the freshly-allocated region.

The function epilogue is the saving grace. chibicc's epilogue is uniformly `mov %rbp, %rsp; pop %rbp; ret` (or an explicit `mov` to `%rsp` from the saved `%rbp`), which means the entire `alloca`-allocated region — and the slid temporary area — is released in one stroke when the function returns. This is exactly why `alloca` works in chibicc's compilation model: the epilogue tears down the frame in one step, regardless of how it grew during the body.

The 16-byte alignment is overkill for most uses but matches what gcc does, and it's free (one masked `add`).

### `alloca_bottom` as a tracked field

The local is hidden — its name is the empty string — but its `Obj` is on the function's locals list and thus gets a stack offset assigned by `assign_lvar_offsets` like any other local. The codegen then references it by `offset(%rbp)` in the prologue and in `builtin_alloca`.

The presence of an `__alloca_size__` local in *every* function (regardless of whether `alloca` is actually called) is the small cost of always-on instrumentation. It's eight bytes per function. Worth nothing.

### Test cases

`test/alloca.c` exercises four allocations interleaved with arithmetic and a function call:

```c
int main() {
  int i = 0;

  char *p1 = alloca(16);
  char *p2 = alloca(16);
  char *p3 = 1 + (char *)alloca(3) + 1;
  p3 -= 2;
  char *p4 = fn(1, alloca(16), 3);

  ASSERT(16, p1 - p2);
  ASSERT(16, p2 - p3);
  ASSERT(16, p3 - p4);
  ...
}
```

The pointer-arithmetic test (`1 + (char *)alloca(3) + 1`) checks that the allocation outlives the surrounding arithmetic, and the `fn(1, alloca(16), 3)` call checks that an `alloca` result computed as a function argument doesn't get clobbered by other evaluation pushes.

**Where we are.** `alloca(n)` is a builtin. The compiler synthesizes a `void *alloca(int)` declaration at parse start, recognizes `ND_FUNCALL` to a variable named `alloca` in codegen, and emits an inline shift-the-temp-area sequence that reserves `n` aligned bytes and returns a pointer. The implementation depends on a per-function hidden local `__alloca_size__` (the field is `alloca_bottom` on `Obj`). The allocation lives until the function epilogue tears down the frame. Stack-exhaustion is not detected; that's the standard `alloca` caveat. The eight-byte cost is paid by every function whether it calls `alloca` or not.

---

## 21.4 — Variable-length arrays

> `git checkout e8667afd08ecbf7c9b05beb4ff399959d9722ff9` — *Add sizeof() for VLA*
>
> `git checkout 07f901057f5c6aa77c0f15f7a22dc0b88923c227` — *Add pointer arithmetic for VLA*
>
> `git checkout 2fa8f489f3a852bd5bb17e023fdc5ea3a606100d` — *Support sizeof(typename) where typename is a VLA*
>
> `git checkout b0109a30c9fa24fedcb4d79bb17788e7ed228636` — *Do not define __STDC_NO_VLA__*

VLAs were added to C in C99 and made optional in C11 (a hosted implementation may define `__STDC_NO_VLA__` to opt out). They allow array dimensions to be ordinary integer expressions evaluated at run time:

```c
void f(int n) {
  int x[n];      // VLA — size is n at the time f is entered
  int y[n+2][m]; // multi-dimensional VLA
}
```

The size has to be remembered between the declaration and any later use of `sizeof(x)`. chibicc's approach: rewrite the VLA declaration into a sequence — compute the size into a hidden local, allocate via `alloca`, and remember the local for `sizeof` to read later.

The arc lands in four commits, in main order. The first three actually add the feature; the last just drops the `__STDC_NO_VLA__` predefined macro because chibicc no longer lacks the feature. Commit order doesn't match calendar order — `2fa8f48` and `b0109a3` are dated August 25, `07f9010` and `e8667af` come in early September. Rui's branches landed in the order of dependent functionality rather than the order they were written.

### 21.4.1 — `TY_VLA` and `vla_of`

`TypeKind` gains a new entry, and `Type` gains two new fields:

```c
typedef enum {
  ...
  TY_ARRAY,
  TY_VLA, // variable-length array
  ...
} TypeKind;

struct Type {
  ...
  // Variable-length array
  Node *vla_len;  // # of elements
  Obj *vla_size;  // sizeof() value
  ...
};
```

`vla_len` is the parsed expression for the dimension (the `n+2` in `int x[n+2]`). `vla_size` is the hidden local where the *byte count* (length × element size) gets stored at run time.

A constructor sits next to `array_of`:

```c
Type *vla_of(Type *base, Node *len) {
  Type *ty = new_type(TY_VLA, 8, 8);
  ty->base = base;
  ty->vla_len = len;
  return ty;
}
```

Note the size is 8 (a pointer's worth) and the alignment is 8. `TY_VLA` represents the *array as a value*, which from the codegen's perspective is a stack-allocated pointer — the underlying storage lives wherever `alloca` put it.

### 21.4.2 — `array_dimensions` becomes the VLA decision point

The dimension-parser previously folded the bracket expression to a constant:

```c
int sz = const_expr(&tok, tok);
return array_of(ty, sz);
```

Now it parses the bracket as an arbitrary expression and decides afterward whether it folds to a constant:

```c
Node *expr = conditional(&tok, tok);
tok = skip(tok, "]");
ty = type_suffix(rest, tok, ty);

if (ty->kind == TY_VLA || !is_const_expr(expr))
  return vla_of(ty, expr);
return array_of(ty, eval(expr));
```

The check is "is the result type already a VLA, or is the dimension non-constant?" — if either, the result is a VLA. The "already a VLA" test handles arrays-of-VLAs (the inner type is fixed, the outer dimension carries the variability). `is_const_expr` is the new compile-time-constancy predicate.

### 21.4.3 — `is_const_expr`: a structural recursion

`eval` evaluates a node *as a constant expression*; if the node isn't constant, `eval` errors. Previously the only way to know if a node was constant was to try `eval` and catch the error. The dimension parser needs to *test* without throwing, so `is_const_expr` is added as a structural recursion that returns false on anything not statically computable:

```c
static bool is_const_expr(Node *node) {
  add_type(node);

  switch (node->kind) {
  case ND_ADD: case ND_SUB: case ND_MUL: case ND_DIV:
  case ND_BITAND: case ND_BITOR: case ND_BITXOR:
  case ND_SHL: case ND_SHR:
  case ND_EQ: case ND_NE: case ND_LT: case ND_LE:
  case ND_LOGAND: case ND_LOGOR:
    return is_const_expr(node->lhs) && is_const_expr(node->rhs);
  case ND_COND:
    if (!is_const_expr(node->cond))
      return false;
    return is_const_expr(eval(node->cond) ? node->then : node->els);
  case ND_COMMA:
    return is_const_expr(node->rhs);
  case ND_NEG: case ND_NOT: case ND_BITNOT: case ND_CAST:
    return is_const_expr(node->lhs);
  case ND_NUM:
    return true;
  }
  return false;
}
```

The `ND_COND` arm is interesting: if the condition is constant, only the chosen arm has to be constant. That matches C's "discard the unselected arm" rule for constant expressions. `ND_COMMA` follows the same rule (the value is the right-hand side; the left-hand side just runs).

The omitted node kinds (variables, function calls, dereferences, struct accesses, address-of, pre/post increments, anything not numerically reducible) all fall through to `return false`. So `int x[n]` — where `n` is a variable — yields `is_const_expr(n) == false`, which tips `array_dimensions` into the VLA path.

The eval-quartet picture from earlier chapters now has a fifth member: `eval`, `eval2`, `eval_rval`, `eval_double`, and `is_const_expr`. The duplication that would let one walk subsume the others remains.

### 21.4.4 — `compute_vla_size`: chained ND_COMMA evaluations

When a VLA declaration is processed, we have to:

1. Evaluate every `vla_len` expression in declaration order.
2. Multiply each by the element size of its base.
3. Store the products into the per-dimension `vla_size` locals.

Multi-dimensional VLAs cascade — `int x[m][n]` needs the inner dimension's `vla_size` to be the inner row size in bytes (`n * sizeof(int)`), and the outer dimension's `vla_size` to be the full array size (`m * inner_vla_size`).

`compute_vla_size` walks the type and produces a comma-expression that does all the work:

```c
static Node *compute_vla_size(Type *ty, Token *tok) {
  Node *node = new_node(ND_NULL_EXPR, tok);
  if (ty->base)
    node = new_binary(ND_COMMA, node, compute_vla_size(ty->base, tok), tok);

  if (ty->kind != TY_VLA)
    return node;

  Node *base_sz;
  if (ty->base->kind == TY_VLA)
    base_sz = new_var_node(ty->base->vla_size, tok);
  else
    base_sz = new_num(ty->base->size, tok);

  ty->vla_size = new_lvar("", ty_ulong);
  Node *expr = new_binary(ND_ASSIGN, new_var_node(ty->vla_size, tok),
                          new_binary(ND_MUL, ty->vla_len, base_sz, tok),
                          tok);
  return new_binary(ND_COMMA, node, expr, tok);
}
```

The recursion descends `ty->base` first so inner VLAs have their `vla_size` locals populated before the outer multiplication uses them. The result is a chain of `ND_COMMA`s starting with a `ND_NULL_EXPR` (a do-nothing seed), followed by one `vla_size = vla_len * base_sz` assignment per VLA dimension. The `vla_size` itself is an unsigned-long local, allocated by `new_lvar("", ty_ulong)` with an empty name (it can't collide with anything user-typed).

The tree shape, for `int x[m][n]`, is roughly:

```
((NULL_EXPR, (NULL_EXPR, NULL_EXPR)),
 (inner_size = n * sizeof(int))),
 (outer_size = m * inner_size)
```

which evaluates left-to-right, so `n` is read before `m * inner_size` is computed.

### 21.4.5 — The declaration rewrite

In `declaration`, after parsing a declarator, we synthesize the size-computation as an expression statement, and (if the type is a VLA) follow it with an `alloca`-and-assign:

```c
// Generate code for computing a VLA size. We need to do this
// even if ty is not VLA because ty may be a pointer to VLA
// (e.g. int (*foo)[n][m] where n and m are variables.)
cur = cur->next = new_unary(ND_EXPR_STMT, compute_vla_size(ty, tok), tok);

if (ty->kind == TY_VLA) {
  if (equal(tok, "="))
    error_tok(tok, "variable-sized object may not be initialized");

  // Variable length arrays (VLAs) are translated to alloca() calls.
  // For example, `int x[n+2]` is translated to `tmp = n + 2,
  // x = alloca(tmp)`.
  Obj *var = new_lvar(get_ident(ty->name), ty);
  Token *tok = ty->name;
  Node *expr = new_binary(ND_ASSIGN, new_var_node(var, tok),
                          new_alloca(new_var_node(ty->vla_size, tok)),
                          tok);

  cur = cur->next = new_unary(ND_EXPR_STMT, expr, tok);
  continue;
}
```

The comment is the spec of the rewrite: `int x[n+2]` becomes (conceptually) `tmp = (n+2) * sizeof(int); x = alloca(tmp);`. The actual lowering is a pair of expression statements — the size computation, then the assignment of the `alloca` result to the array variable.

`new_alloca` is the helper that builds an `ND_FUNCALL` to the synthesized `alloca` builtin:

```c
static Node *new_alloca(Node *sz) {
  Node *node = new_unary(ND_FUNCALL, new_var_node(builtin_alloca, sz->tok), sz->tok);
  node->func_ty = builtin_alloca->ty;
  node->ty = builtin_alloca->ty->return_ty;
  node->args = sz;
  add_type(sz);
  return node;
}
```

`builtin_alloca` is now also captured into a file-static so this code can refer to it without re-looking up. The call goes through the normal `ND_FUNCALL` path, which means codegen sees the same special-case it added in §21.3 — a call to a variable named `alloca` becomes the inline allocation sequence.

A VLA initializer is rejected with an explicit error: `"variable-sized object may not be initialized"`. That's what the standard requires; chibicc just reproduces the rule.

Note also the comment at the top about pointer-to-VLA: even if the declared type isn't itself a VLA, its base may be (`int (*foo)[n][m]`), and we still need to evaluate `n` and `m` at the declaration site to fix `sizeof(*foo)` later. The unconditional `compute_vla_size` call handles that.

### 21.4.6 — `ND_VLA_PTR` and the VLA-typed local

Initially the rewrite uses `new_var_node(var, tok)` to build the assignment target. After the next commit (`07f9010`, "Add pointer arithmetic for VLA") it switches to a new node kind:

```c
Node *expr = new_binary(ND_ASSIGN, new_vla_ptr(var, tok),
                        new_alloca(new_var_node(ty->vla_size, tok)),
                        tok);
```

Why? A `TY_VLA` lvalue normally decays to a pointer when read (just like `TY_ARRAY`), but the *target* of the assignment is the eight-byte slot that holds the pointer. `gen_addr` for an `ND_VAR` of `TY_VLA` already emits `mov offset(%rbp), %rax` (loading the pointer). For the assignment target we want `lea offset(%rbp), %rax` (the address of the slot). `ND_VLA_PTR` is the new node kind for "the slot itself, not the pointer in it":

```c
case ND_VLA_PTR:
  println("  lea %d(%%rbp), %%rax", node->var->offset);
  return;
```

`add_type` reuses `ND_VAR`'s type rule:

```c
case ND_VAR:
case ND_VLA_PTR:
  node->ty = node->var->ty;
  return;
```

The split between "read the pointer at the slot" and "address of the slot" is the same kind of split that a pointer-to-array forces in `&arr` versus `arr`. VLAs need the same machinery because a VLA-typed local is, at the codegen level, exactly a pointer to a fresh `alloca` block.

### 21.4.7 — `gen_addr` for VLA-typed locals

```c
case ND_VAR:
  // Variable-length array, which is always local.
  if (node->var->ty->kind == TY_VLA) {
    println("  mov %d(%%rbp), %%rax", node->var->offset);
    return;
  }
  ...
```

Reading an `ND_VAR` of VLA type loads the slot's contents (the pointer to the alloca'd buffer). Decay-to-pointer is implicit: any later subscripting indexes that pointer with the right element size. The note in the comment ("which is always local") is the chibicc-specific narrowing — global VLAs aren't a thing the language allows anyway, but the comment names that codegen relies on it.

### 21.4.8 — Pointer arithmetic on VLA-typed bases

`new_add` and `new_sub` previously multiplied integer offsets by `lhs->ty->base->size`. For a VLA base, `size` is meaningless (it's whatever `new_type` happened to set; in practice 8). The replacement reads `vla_size` instead:

```c
// VLA + num
if (lhs->ty->base->kind == TY_VLA) {
  rhs = new_binary(ND_MUL, rhs, new_var_node(lhs->ty->base->vla_size, tok), tok);
  return new_binary(ND_ADD, lhs, rhs, tok);
}
```

The runtime-loaded `vla_size` becomes the multiplier. `new_sub` does the same. So `&x[i]` for a VLA `x` correctly scales `i` by the row size that was computed at declaration time.

The `load` function also has to know not to load a value from a VLA-typed lvalue — `TY_VLA` joins `TY_ARRAY`, `TY_STRUCT`, `TY_UNION`, `TY_FUNC` in the "don't try to fit this in a register" club:

```c
case TY_VLA:
  // If it is an array, do not attempt to load a value to the
  // register because in general we can't load an entire array to a
  // register. ...
```

The `gen_addr` change above has already made `&x` (or any direct read of `x`) produce the pointer; `load` just doesn't second-guess.

### 21.4.9 — `sizeof(VLA)` reads the hidden local

`primary`'s `sizeof` arms learn to check for `TY_VLA`:

```c
if (equal(tok, "sizeof") && equal(tok->next, "(") && is_typename(tok->next->next)) {
  Type *ty = typename(&tok, tok->next->next);
  *rest = skip(tok, ")");
  if (ty->kind == TY_VLA)
    return new_var_node(ty->vla_size, tok);
  return new_ulong(ty->size, start);
}

if (equal(tok, "sizeof")) {
  Node *node = unary(rest, tok->next);
  add_type(node);
  if (node->ty->kind == TY_VLA)
    return new_var_node(node->ty->vla_size, tok);
  return new_ulong(node->ty->size, tok);
}
```

For a fixed array the result is a compile-time `ulong` constant. For a VLA it's a runtime read of the `vla_size` local. That local has already been written by the declaration's `compute_vla_size` lowering, so the read returns the right value at the right time.

The `2fa8f48` commit then extends `sizeof(typename)` to handle the case where the `typename` itself is a VLA. The previous form assumed the VLA was already a declared variable (so `vla_size` was already populated); for `sizeof(int [n][n+2])` we have to compute the size on the spot:

```c
if (ty->kind == TY_VLA) {
  if (ty->vla_size)
    return new_var_node(ty->vla_size, tok);

  Node *lhs = compute_vla_size(ty, tok);
  Node *rhs = new_var_node(ty->vla_size, tok);
  return new_binary(ND_COMMA, lhs, rhs, tok);
}
```

If the type already has a populated `vla_size` (because it came from a declared variable), we reuse it. Otherwise we generate an inline `compute_vla_size` chain followed by a read of the freshly-populated local. The result is a `(compute, read)` comma-expression that evaluates the dimensions and returns the byte count.

### 21.4.10 — `__STDC_NO_VLA__` removed

The fourth commit is one line:

```c
-  define_macro("__STDC_NO_VLA__", "1");
```

The macro signaled "this implementation deliberately doesn't support VLAs"; chibicc no longer needs to claim that.

### Test cases

`test/vla.c` ranges from the simplest declaration (`int n=5; int x[n]; sizeof(x)`) through nested pointer-to-VLA (`int (*x)[n][n+2]`) through fill-and-read (using the row-size-aware pointer arithmetic) through bare `sizeof(char[2][n])`:

```c
ASSERT(20, ({ int n=5; int x[n]; sizeof(x); }));
ASSERT((5+1)*(8*2)*4, ({ int m=5, n=8; int x[m+1][n*2]; sizeof(x); }));
ASSERT(8, ({ char n=10; int (*x)[n][n+2]; sizeof(x); }));
ASSERT(480, ({ char n=10; int (*x)[n][n+2]; sizeof(*x); }));
ASSERT(48, ({ char n=10; int (*x)[n][n+2]; sizeof(**x); }));
ASSERT(4, ({ char n=10; int (*x)[n][n+2]; sizeof(***x); }));
ASSERT(60, ({ char n=3; int x[5][n]; sizeof(x); }));
...
ASSERT(0, ({ int n=10; int x[n+1][n+6]; int *p=x;
             for (int i = 0; i<sizeof(x)/4; i++) p[i]=i;
             x[0][0]; }));
ASSERT(10, ({ int n=5; sizeof(char[2][n]); }));
```

The fill-and-read tests are the proof that pointer arithmetic uses `vla_size` and not a stale compile-time size: `for (int i = 0; i < sizeof(x)/4; i++) p[i]=i` writes through a `int *`-decayed VLA, and the subsequent `x[5][2]` reads back the value at the right scaled offset.

**Where we are.** VLAs are functional. `TY_VLA` is the new type kind; `Type` carries `vla_len` (the parsed expression) and `vla_size` (a hidden local for the byte count). Declarations rewrite to `compute_vla_size` followed by an `alloca` whose result is stored in a `TY_VLA`-typed local; the local is read as a pointer at every use. `sizeof(VLA)` reads the hidden size local. `sizeof(typename)` of a not-yet-declared VLA computes inline and reads. Pointer arithmetic uses the runtime size. `is_const_expr` is the new structural-recursion predicate that decides whether a bracketed expression makes the type a VLA. `__STDC_NO_VLA__` is no longer defined. The eval-quartet from earlier chapters now has a fifth co-located walker. The canonicalization-at-parse-time count ticks from ten to eleven — VLA declarations rewrite to a compute-then-alloca pair.

---

## 21.5 — Linker driver: `-l`, `-s`, ELF size/type, `.a`/`.so`

> `git checkout bc2527944a83c1bc951a429530f39e93dc5235b2` — *Add -l option*
>
> `git checkout c32f0e21e71f43e64a7b98c9d96d4c513d42ba37` — *Add -s option*
>
> `git checkout 8d130ab93f65f7ef79839aba87459e4f9507ba39` — *Emit size and type for symbols*
>
> `git checkout d56dd2f46e4049f017eae0dc99b2d16e78b88bee` — *Recognize .a and .so files*

Four small commits. They round out the linker-driver story: library linking with `-l`, stripped output with `-s`, ELF symbol-table size and type emission, and recognition of `.a`/`.so` files as linker inputs.

### 21.5.1 — `-l NAME`: library linking

`-l NAME` tells the linker to find `libNAME.so` (or `libNAME.a` if no shared library exists) in its search path and link against it. chibicc's job here is just to pass the option through:

```c
if (!strncmp(argv[i], "-l", 2)) {
  strarray_push(&input_paths, argv[i]);
  continue;
}
```

The `-l` argument is collected on `input_paths` rather than on `ld_args` directly, which means it's processed in the same loop as filename inputs. The compile-vs-link disposition then identifies it by prefix and pushes it to the linker arguments unchanged:

```c
if (!strncmp(input, "-l", 2)) {
  strarray_push(&ld_args, input);
  continue;
}
```

The reason for routing through `input_paths` rather than directly to `ld_args` is order: the linker resolves symbols left-to-right, and a `-l` argument has to be at the right position relative to object files for unresolved symbols to find the library. By putting `-l foo` and `bar.o` on the same list, chibicc preserves the user's command-line ordering through the loop and into the final link command.

Search-path resolution is the linker's job, not chibicc's. chibicc already passes `-L/usr/lib` and `-L/lib` to the linker (added in earlier chapters), and that's enough for `-lfoo` to resolve to `/usr/lib/libfoo.so` (or wherever).

### 21.5.2 — `-s`: strip the binary

`-s` is a linker flag that tells the linker to produce an output without symbol-table or debug information. chibicc adds a pass-through:

```c
if (!strcmp(argv[i], "-s")) {
  strarray_push(&ld_extra_args, "-s");
  continue;
}
```

The new `ld_extra_args` `StringArray` is a separate channel for "linker-only options that aren't input paths." The link command is built by `run_linker`, which now appends `ld_extra_args` between the `-L` paths and the input list:

```c
strarray_push(&arr, "-L/usr/lib");
strarray_push(&arr, "-L/lib");

for (int i = 0; i < ld_extra_args.len; i++)
  strarray_push(&arr, ld_extra_args.data[i]);

for (int i = 0; i < inputs->len; i++)
  strarray_push(&arr, inputs->data[i]);
```

Splitting "input paths and `-l` flags" from "other linker options" keeps the order-sensitive part separate from the order-insensitive part. `-s` doesn't care where it sits; library inputs do.

### 21.5.3 — `.type` and `.size` directives

Through Chapter 20, chibicc emitted symbol *labels* but not the metadata that the ELF format and downstream tools want for stripping, debugging, and dynamic linking — specifically the symbol's *type* (function vs object) and *size* (number of bytes occupied). The fix is a handful of `.type` and `.size` directives in `emit_data` and `emit_text`:

```c
// emit_data, in the .data/.tdata branch:
println("  .type %s, @object", var->name);
println("  .size %s, %d", var->name, var->ty->size);
println("  .align %d", align);

// emit_text:
println("  .text");
println("  .type %s, @function", fn->name);
println("%s:", fn->name);
```

`.type name, @object` marks `name` as a data symbol; `@function` marks it as code. `.size` records the byte count. The emission also moves the `.align` directive after the type/size declarations (the previous order had `.align` first, which is fine but inconsistent with the standard ELF assembly layout).

The `.type` for the function gets emitted in `emit_text`. The `.size` for the function does *not* — chibicc doesn't track function size as a parse artifact, and computing it would mean deferring emission until after the function body or running an assembler-level fixup pass. Tools that need function size compute it from labels (the `name`-to-next-label distance) instead. That works in practice but means `.size` is missing for code symbols. Real toolchains emit `.size name, .-name` (the dot is the current location counter, so the difference is the function's byte count); chibicc could do the same if needed but doesn't here.

The motivation for adding `.type` and `.size` at all is dynamic linking: when chibicc-compiled object files are linked into shared libraries (`.so`), the dynamic linker needs the symbol type to handle relocations correctly. Static linking is mostly forgiving; dynamic linking is not.

### 21.5.4 — `.a` and `.so` recognition

`get_file_type` learns two new extensions:

```c
typedef enum {
  FILE_NONE, FILE_C, FILE_ASM, FILE_OBJ, FILE_AR, FILE_DSO,
} FileType;

static FileType get_file_type(char *filename) {
  if (opt_x != FILE_NONE)
    return opt_x;

  if (endswith(filename, ".a"))
    return FILE_AR;
  if (endswith(filename, ".so"))
    return FILE_DSO;
  if (endswith(filename, ".o"))
    return FILE_OBJ;
  ...
}
```

`.a` is a static archive (the unix `ar` format — a bundle of `.o` files); `.so` is a shared object. Both are passed to the linker unchanged:

```c
if (type == FILE_OBJ || type == FILE_AR || type == FILE_DSO) {
  strarray_push(&ld_args, input);
  continue;
}
```

The recognition is purely by filename suffix — chibicc doesn't sniff file magic. That works because `ar` archives are conventionally `.a` and shared libraries are conventionally `.so`. Real-world tools sometimes accept `.so.1.2.3` (versioned shared libraries) and `.a.bundle` (rare but seen); chibicc's strict suffix check will miss those. They're rare enough that the simple approach is fine.

There's also a small refactor in this commit: the `.o` check used to short-circuit `opt_x` (an `.o` file is an object regardless of `-x`), but the order is now `opt_x` first, then suffix. That matches gcc — `-x assembler foo.o` would treat `foo.o` as an assembler input. In practice nobody does that, and the existing `.o` case still works because most `.o` filenames don't co-exist with `-x` flags.

The test driver picks up two shape examples for archives and shared objects:

```bash
echo 'void foo() {}' | $chibicc -c -xc -o $tmp/foo.o -
echo 'void bar() {}' | $chibicc -c -xc -o $tmp/bar.o -
ar rcs $tmp/foo.a $tmp/foo.o $tmp/bar.o
echo 'void foo(); void bar(); int main() { foo(); bar(); }' > $tmp/main.c
$chibicc -o $tmp/foo $tmp/main.c $tmp/foo.a
```

The `.so` test does the same with system `cc -shared -fPIC` (chibicc itself doesn't emit position-independent code, so the shared object has to be built with the system compiler, but chibicc can *link* against the resulting `.so`).

**Where we are.** Four linker-driver additions. `-l NAME` flows through `input_paths` into the linker command, in command-line order. `-s` flows through a new `ld_extra_args` channel. `.type @object` / `@function` and `.size` directives are emitted for data and function symbols (with `.size` missing on functions). `.a` and `.so` are recognized as linker inputs by suffix. The driver's vocabulary now covers everything most projects need at the chibicc-as-cc1 level — the remaining gaps are debugging information and position-independent code.

---

## 21.6 — `long double`, case ranges, array range designators, labels-as-values

> `git checkout e0bf168041ef60687b5d4454a93fc78c4f3acc48` — *Add long double*
>
> `git checkout d90c73b6058af4b22a4edd610713f75b2478e356` — *[GNU] Support case ranges*
>
> `git checkout 3d5550e29a92708613c3a351c0857aea90e147a5` — *[GNU] Support array range designator*
>
> `git checkout 4f165ec60baa74f244d0a7c9b64c4bb3cbb76173` — *[GNU] Support labels-as-values*

The chapter's last four commits are a mixed bag: `long double` finally becomes its own type rather than an alias for `double`, and three GNU bracket-range features land — `case 1 ... 5:` in switches, `[3 ... 7] = x` in initializers (which closes the §19.7 errata candidate), and `&&label`/`goto *expr` for computed gotos.

### 21.6.1 — `long double`: real extended precision

Through Chapter 20, `long double` parsed as a synonym for `double`:

```c
case DOUBLE:
case LONG + DOUBLE:
  ty = ty_double;
  break;
```

That was an open errata candidate flagged in earlier chapters. This commit closes it:

```c
case DOUBLE:
  ty = ty_double;
  break;
case LONG + DOUBLE:
  ty = ty_ldouble;
  break;
```

A new `TY_LDOUBLE` enters `TypeKind`. A `ty_ldouble` global type is added with size 16 and alignment 16 (the SysV AMD64 layout — 80-bit x87 extended precision padded to 16 bytes for alignment):

```c
Type *ty_ldouble = &(Type){TY_LDOUBLE, 16, 16};
```

`is_flonum` is widened:

```c
bool is_flonum(Type *ty) {
  return ty->kind == TY_FLOAT || ty->kind == TY_DOUBLE ||
         ty->kind == TY_LDOUBLE;
}
```

`is_compatible` accepts cross-floating-point compatibility (`is_compatible(double, long double)` returns true at the matching arm — though by the time we reach this arm both types must have the same `kind`, so this just pattern-matches).

`get_common_type` learns the new ladder rule:

```c
if (ty1->kind == TY_LDOUBLE || ty2->kind == TY_LDOUBLE)
  return ty_ldouble;
if (ty1->kind == TY_DOUBLE || ty2->kind == TY_DOUBLE)
  return ty_double;
if (ty1->kind == TY_FLOAT || ty2->kind == TY_FLOAT)
  return ty_float;
```

The standard's promotion order: any operand promoted to long double makes the result long double, etc.

The token side promotes the literal carrier to `long double`:

```c
struct Token {
  ...
  long double fval;
  ...
};
struct Node {
  ...
  long double fval;
  ...
};
```

and the literal-conversion code reads `strtold` instead of `strtod`, classifying an `L`-suffixed literal as `ty_ldouble`:

```c
long double val = strtold(tok->loc, &end);

Type *ty;
if (*end == 'f' || *end == 'F') { ... }
else if (*end == 'l' || *end == 'L') {
  ty = ty_ldouble;
  end++;
}
```

That's the type system. The codegen side is much larger.

### 21.6.2 — x87 codegen and the F80 cast row

x86-64 SysV passes `long double` differently from `double`. `double` lives in xmm registers and uses SSE instructions; `long double` (x87 80-bit extended) lives on the *x87 floating-point stack* and uses the older x87 instructions (`fld`, `fst`, `faddp`, `fsubrp`, `fmulp`, `fdivrp`, `fcomip`, etc.). The two coprocessors are separate; chibicc has to emit different instruction families depending on type.

Loads and stores:

```c
case TY_LDOUBLE:
  println("  fldt (%%rax)");   // load a long double from memory onto the x87 stack
  return;
...
case TY_LDOUBLE:
  println("  fstpt (%%rdi)");  // store and pop the top of the x87 stack to memory
  return;
```

Zero comparison (for boolean coercion):

```c
case TY_LDOUBLE:
  println("  fldz");
  println("  fucomip");
  println("  fstp %%st(0)");
  return;
```

`fldz` pushes zero onto the x87 stack; `fucomip` compares and pops one operand; `fstp` discards the leftover.

Negation:

```c
case TY_LDOUBLE:
  println("  fchs");   // flip sign on top of x87 stack
  return;
```

### 21.6.3 — Casts: the F80 row of the cast table

The cast table grows from 10×10 to 11×11. There's a new `F80` slot, and every existing row picks up an extra column (cast-to-F80) while a brand-new F80 row provides the cast-from-F80 column entries. The new conversions use the x87 instructions `fildl`/`fildll` (load integer onto x87 stack) and `fistp` (store integer with control-word adjustment for truncation):

```c
static char i32f80[] = "mov %eax, -4(%rsp); fildl -4(%rsp)";
static char u32f80[] = "mov %eax, %eax; mov %rax, -8(%rsp); fildll -8(%rsp)";
static char i64f80[] = "movq %rax, -8(%rsp); fildll -8(%rsp)";
static char u64f80[] =
  "mov %rax, -8(%rsp); fildq -8(%rsp); test %rax, %rax; jns 1f;"
  "mov $1602224128, %eax; mov %eax, -4(%rsp); fadds -4(%rsp); 1:";
```

The unsigned-64-to-F80 case is the trickiest: there's no `fild` for unsigned 64-bit integers, so the code does a signed `fildq` and then conditionally adds 2^64 (encoded as the float `1.8446744e19`, which has the bit-pattern `0x5F800000`, decimal `1602224128`) if the high bit was set. That's the same trick gcc uses for the same conversion.

The F80-to-integer conversions use the FPU control-word twiddle for truncation:

```c
#define FROM_F80_1                                           \
  "fnstcw -10(%rsp); movzwl -10(%rsp), %eax; or $12, %ah; " \
  "mov %ax, -12(%rsp); fldcw -12(%rsp); "

#define FROM_F80_2 " -24(%rsp); fldcw -10(%rsp); "

static char f80i32[] = FROM_F80_1 "fistpl" FROM_F80_2 "mov -24(%rsp), %eax";
```

`fnstcw` saves the FPU control word; `or $12, %ah` sets bits that select round-toward-zero (truncation); `fldcw` loads the new control word; `fistpl` stores-and-pops the integer with the truncation rounding; the second `fldcw` restores the original control word. The two-macro approach is just a way to make the cast-string literals fit on a line each.

Cross-floating-point F80 conversions:

```c
static char f32f80[] = "movss %xmm0, -4(%rsp); flds -4(%rsp)";
static char f64f80[] = "movsd %xmm0, -8(%rsp); fldl -8(%rsp)";
static char f80f32[] = "fstps -8(%rsp); movss -8(%rsp), %xmm0";
static char f80f64[] = "fstpl -8(%rsp); movsd -8(%rsp), %xmm0";
```

Each crosses the SSE/x87 boundary by spilling through memory.

### 21.6.4 — Binary ops on `long double`

The `gen_expr` binary-op section gains a third branch (after the existing `is_flonum` branch is split into TY_FLOAT/TY_DOUBLE):

```c
switch (node->lhs->ty->kind) {
case TY_FLOAT:
case TY_DOUBLE: {
  // existing SSE codegen
  ...
}
case TY_LDOUBLE: {
  gen_expr(node->lhs);
  gen_expr(node->rhs);

  switch (node->kind) {
  case ND_ADD: println("  faddp"); return;
  case ND_SUB: println("  fsubrp"); return;
  case ND_MUL: println("  fmulp"); return;
  case ND_DIV: println("  fdivrp"); return;
  case ND_EQ: case ND_NE: case ND_LT: case ND_LE:
    println("  fcomip");
    println("  fstp %%st(0)");
    if (node->kind == ND_EQ) println("  sete %%al");
    else if (node->kind == ND_NE) println("  setne %%al");
    else if (node->kind == ND_LT) println("  seta %%al");
    else println("  setae %%al");
    println("  movzb %%al, %%rax");
    return;
  }
  ...
}
}
```

Each of `gen_expr(lhs)` and `gen_expr(rhs)` pushes a long double onto the x87 stack; the `faddp` family operates on the top two and pops one. Note the operand-order quirk in subtraction and division: `fsubrp` and `fdivrp` reverse the operands relative to what the obvious code would do, because the x87 stack has the second-pushed operand on top and the first-pushed below it. The instruction names with `r` swap the order; the `p` pops one register after.

The comparison sequence uses `fcomip` (compare-pop, set flags) followed by an `fstp %st(0)` to discard the second operand (the comparison only consumes one). The flags-test instructions `sete`/`setne`/`seta`/`setae` are the same as the SSE comparison path, except `seta`/`setae` (above/above-or-equal) are used instead of `setb`/`setbe`. That matches the x87 flag semantics — `fcomip` produces the C flags in a way that maps "less than" to "above" in the unsigned sense.

### 21.6.5 — `long double` as a function argument

The SysV AMD64 ABI passes `long double` *on the stack* — it does not assign x87 registers to it the way it assigns xmm registers to `double`. `push_args` recognizes this:

```c
case TY_LDOUBLE:
  arg->pass_by_stack = true;
  stack += 2;
  break;
```

`stack += 2` because each long double consumes 16 bytes (two 8-byte slots).

`push_args2` emits the actual stack push:

```c
case TY_LDOUBLE:
  println("  sub $16, %%rsp");
  println("  fstpt (%%rsp)");
  depth += 2;
  break;
```

The argument is popped from the top of the x87 stack and stored to the new stack space. The `depth += 2` updates chibicc's running stack-depth tracker so subsequent operations stay 16-byte-aligned for SSE pushes.

`assign_lvar_offsets` adds an empty case for `TY_LDOUBLE` in the same switch where the integer and double branches handle register passing — long doubles aren't register-passed, so the case is a no-op (but the switch must explicitly mention `TY_LDOUBLE` to avoid falling into the `gp++` arm):

```c
case TY_LDOUBLE:
  break;
```

`has_flonum` (for struct-passing classification) gets a small narrowing — it now treats long double as not-quite-flonum for the struct-arm-classification purpose, because long double doesn't go through the same eight-byte SSE-classification machinery:

```c
return offset < lo || hi <= offset || ty->kind == TY_FLOAT || ty->kind == TY_DOUBLE;
```

Note the explicit kind check rather than `is_flonum(ty)`. The two predicates have diverged; `is_flonum` says "yes for any of the three FP types," `has_flonum` says "yes only for FLOAT and DOUBLE because LDOUBLE isn't passed in xmm." That's the kind of subtle ABI-vs-language-type split the SysV ABI forces on every compiler that supports long double.

The psABI conformance count grows by one for the long-double calling convention.

### 21.6.6 — `long double` literal emission

Float and double literals encode as bit-cast immediates in earlier chapters. Long double extends the pattern with two halves — the low 8 bytes and the high 8 bytes — written through the stack:

```c
case TY_LDOUBLE: {
  union { long double f80; uint64_t u64[2]; } u;
  memset(&u, 0, sizeof(u));
  u.f80 = node->fval;
  println("  mov $%lu, %%rax  # long double %Lf", u.u64[0], node->fval);
  println("  mov %%rax, -16(%%rsp)");
  println("  mov $%lu, %%rax", u.u64[1]);
  println("  mov %%rax, -8(%%rsp)");
  println("  fldt -16(%%rsp)");
  return;
}
```

The `memset` zeroes any padding bits in the union (the 80-bit value occupies only 10 of the 16 bytes; the high 6 bytes need to be deterministic). The two halves are stored to the redzone area (`-16(%rsp)` to `-1(%rsp)`, which is below the current stack pointer but reserved for leaf-function temporaries by the SysV ABI), and then `fldt` pulls the 80-bit value back onto the x87 stack.

This pattern repeats in cast-table entries and in argument-passing — the redzone is the universal scratchpad for x87 ↔ memory ↔ SSE conversions.

### 21.6.7 — Tests and the closing of an errata

`test/sizeof.c` flips a long-standing assertion:

```diff
-  ASSERT(8, sizeof(long double));
+  ASSERT(16, sizeof(long double));
```

`test/literal.c` matches:

```diff
-  ASSERT(8, sizeof(5.l));
-  ASSERT(8, sizeof(2.0L));
+  ASSERT(16, sizeof(5.l));
+  ASSERT(16, sizeof(2.0L));
```

`test/function.c` and `test/arith.c` add long-double-specific function-call and arithmetic tests. The most colorful one verifies `long double`'s precision against `printf`:

```c
ASSERT(0, ({ char buf[100]; sprintf(buf, "%Lf", (long double)12.3);
             strncmp(buf, "12.3", 4); }));
```

(The libc-level `%Lf` format depends on chibicc-compiled code passing `long double` correctly through `vsprintf`; the test passes, which proves the calling convention works end-to-end.)

The "long double is double" errata is now closed.

### 21.6.8 — Case ranges: `case begin ... end:`

GNU C's case ranges generalize a switch arm to match any value in `[begin, end]`. The implementation has to remember both bounds and emit a range check rather than a single equality.

`Node` gains two fields:

```c
// Switch
Node *case_next;
Node *default_case;

// Case
long begin;
long end;
```

The case parser reads the optional `... end` clause:

```c
int begin = const_expr(&tok, tok->next);
int end;

if (equal(tok, "...")) {
  // [GNU] Case ranges, e.g. "case 1 ... 5:"
  end = const_expr(&tok, tok->next);
  if (end < begin)
    error_tok(tok, "empty case range specified");
} else {
  end = begin;
}
```

A single-value case is the degenerate range `[begin, begin]`. The codegen in `gen_stmt` then dispatches:

```c
char *ax = (node->cond->ty->size == 8) ? "%rax" : "%eax";
char *di = (node->cond->ty->size == 8) ? "%rdi" : "%edi";

if (n->begin == n->end) {
  println("  cmp $%ld, %s", n->begin, ax);
  println("  je %s", n->label);
  continue;
}

// [GNU] Case ranges
println("  mov %s, %s", ax, di);
println("  sub $%ld, %s", n->begin, di);
println("  cmp $%ld, %s", n->end - n->begin, di);
println("  jbe %s", n->label);
```

The single-value path is unchanged. The range path uses the unsigned-comparison trick: subtract `begin` from the switch value, compare against `end - begin`, jump if below-or-equal (unsigned). That handles negative ranges and ranges that span zero correctly because the wrap-around makes any out-of-range value compare *above* `end - begin` in unsigned terms.

Notably, this generates code: a `case 1 ... 1000000` would emit a range check, not a million jump-table entries. Chibicc never builds jump tables anyway — every case is a sequential `cmp/je` chain — so the range form doesn't blow up codegen the way it might for a real compiler trying to balance jump-table density.

### 21.6.9 — Array range designators: `[3 ... 7] = x`

This commit closes the §19.7 errata candidate. Through Chapter 19, the parser recognized `[3 ... 7]` syntactically (it accepted the tokens) but only honored `[3]` semantically — the elaboration loop walked just the first index of the range. Now the elaboration honors the full range.

`array_designator` is rewritten to take output parameters for both ends:

```c
static void array_designator(Token **rest, Token *tok, Type *ty, int *begin, int *end) {
  *begin = const_expr(&tok, tok->next);
  if (*begin >= ty->array_len)
    error_tok(tok, "array designator index exceeds array bounds");

  if (equal(tok, "...")) {
    *end = const_expr(&tok, tok->next);
    if (*end >= ty->array_len)
      error_tok(tok, "array designator index exceeds array bounds");
    if (*end < *begin)
      error_tok(tok, "array designator range [%d, %d] is empty", *begin, *end);
  } else {
    *end = *begin;
  }

  *rest = skip(tok, "]");
}
```

The two callers — `designation` (for the right-hand side after the bracket) and `array_initializer1` (the main loop) — both now iterate from `begin` to `end`, applying the same designation expression to each child:

```c
int begin, end;
array_designator(&tok, tok, init->ty, &begin, &end);

Token *tok2;
for (int i = begin; i <= end; i++)
  designation(&tok2, tok, init->children[i]);
array_initializer2(rest, tok2, init, begin + 1);
```

Each iteration re-tokenizes the same source range — `tok` is captured before the loop and the loop reuses it — so the same expression seeds every index in the range. After the loop, `tok2` (the position after the last designation parse) is the resume point.

Test:

```c
ASSERT(16, ({ char x[]={[2 ... 10]='a', [7]='b', [15 ... 15]='c', [3 ... 5]='d'};
              sizeof(x); }));
ASSERT(0, ({ char x[]={[2 ... 10]='a', [7]='b', [15 ... 15]='c', [3 ... 5]='d'};
             memcmp(x, "\0\0adddabaaa\0\0\0\0c", 16); }));
```

The expected layout (`"\0\0adddabaaa\0\0\0\0c"`) shows the overlapping ranges resolving last-write-wins: the `[3 ... 5]='d'` overwrites the `'a'`s set by `[2 ... 10]`, and the standalone `[7]='b'` overwrites that range too. Indices 11–14 are the untouched zeroes between the highest-set range and the top index 15.

The §19.7 errata candidate is now closed.

### 21.6.10 — Labels-as-values: `&&label` and `goto *expr`

GNU C's labels-as-values feature lets you take the address of a label (`&&label`) and `goto` indirectly through an address (`goto *expr`). Used together they enable computed gotos — a common technique in interpreter inner loops, threaded code, and finite state machines.

The token side is already there: `&&` is in the punctuator list (it's the logical-AND operator). The parser distinguishes by context — in an expression after a unary operator slot, `&&label` means "address of label."

`Node` gains two new kinds:

```c
ND_GOTO_EXPR, // "goto" labels-as-values
...
ND_LABEL_VAL, // [GNU] Labels-as-values
```

The parser's `unary` arm picks up the `&&` form:

```c
// [GNU] labels-as-values
if (equal(tok, "&&")) {
  Node *node = new_node(ND_LABEL_VAL, tok);
  node->label = get_ident(tok->next);
  node->goto_next = gotos;
  gotos = node;
  *rest = tok->next->next;
  return node;
}
```

The label name is captured immediately, but the resolution to a unique generated assembly label has to wait until the function body has been parsed (because labels can be defined after their first use — a `&&forward_label` may reference a label defined later in the function). The `gotos` chain that already handled forward `goto` references is reused: `node->goto_next = gotos; gotos = node;` threads the new node onto the same list, and `resolve_goto_labels` (renamed in spirit only — the code is unchanged) walks both `ND_GOTO` and `ND_LABEL_VAL` nodes to fill in `unique_label`.

The `goto *expr` form lives in `stmt`:

```c
if (equal(tok, "goto")) {
  if (equal(tok->next, "*")) {
    // [GNU] `goto *ptr` jumps to the address specified by `ptr`.
    Node *node = new_node(ND_GOTO_EXPR, tok);
    node->lhs = expr(&tok, tok->next->next);
    *rest = skip(tok, ";");
    return node;
  }

  Node *node = new_node(ND_GOTO, tok);
  ...
}
```

The `*` after `goto` is a syntactic giveaway. The expression is a normal expression; codegen treats its result as a code pointer.

Codegen for both:

```c
case ND_LABEL_VAL:
  println("  lea %s(%%rip), %%rax", node->unique_label);
  return;
...
case ND_GOTO_EXPR:
  gen_expr(node->lhs);
  println("  jmp *%%rax");
  return;
```

`lea label(%rip), %rax` produces the address of the label using rip-relative addressing (the standard PIC-friendly form). `jmp *%rax` is the indirect jump.

`add_type` gives the label-address node a void-pointer type:

```c
case ND_LABEL_VAL:
  node->ty = pointer_to(ty_void);
  return;
```

That matches the gcc convention: `&&label` has type `void *`.

Note what this commit *doesn't* do: it doesn't make label addresses usable as compile-time constants for global initializers. A later commit (in Chapter 22) handles that. For now, `&&label` is an expression, evaluable inside a function body but not in a static initializer.

The label namespace is unchanged. Labels-as-values uses the same label table that ordinary `goto`s use; `&&label` and `goto label` resolve to the same internal label.

Test:

```c
ASSERT(3, ({ void *p = &&v11; int i=0; goto *p;
             v11:i++; v12:i++; v13:i++; i; }));
```

`&&v11` produces a code pointer; `goto *p` jumps to it; the labeled statements increment `i`. The fall-through nature of statement-level labels means the jump to `v11` runs all three increments; jumping to `v33` runs just one.

**Where we are.** `long double` is real extended-precision — 16-byte size, 16-byte alignment, x87-stack codegen for arithmetic, on-stack passing for function arguments, the cast table grows from 10×10 to 11×11 with a full new F80 row and column. The "long double is double" errata is closed. Case ranges (`case 1 ... 5:`) generate inline range checks using the unsigned-subtract-and-compare trick; chibicc's no-jump-table approach scales naturally to large ranges. Array range designators (`[3 ... 7] = x`) are now honored in elaboration, not just parsed; the §19.7 errata is closed. Labels-as-values gives `&&label` (a `void *` value) and `goto *expr` (indirect jump). Resolution piggybacks on the existing forward-`goto` resolution pass.

The psABI conformance count ticks up by one for the long-double calling convention. The cast table grows from 10×10 to 11×11.

---

## Recap

Seventeen commits. The chapter adds three substantial pieces of machinery (thread-local storage, `alloca`, VLAs) plus four small linker-driver additions and four bracket-range-and-precision additions:

- `Obj` gains `is_tls` (for thread-local globals) and `alloca_bottom` (a hidden local for the per-function `alloca` slide).
- `VarAttr` gains `is_tls`.
- `Type` gains `vla_len` (the parsed dimension expression) and `vla_size` (a hidden local for the runtime byte count). New kind `TY_VLA`.
- `Node` gains `ND_VLA_PTR` (slot-address for a VLA-typed local), `ND_GOTO_EXPR` and `ND_LABEL_VAL` (labels-as-values), and the `Node->begin`/`end` fields (for case ranges).
- A new `TY_LDOUBLE` kind, a `ty_ldouble` global, a fifth eval-quartet member `is_const_expr`, a synthesized `alloca` builtin in `globals`, two new TLS sections (`.tdata`/`.tbss`), and a 11×11 cast table replacing the 10×10.
- `parse()` does no new top-level passes (it already runs `mark_live` and `scan_globals`); the new work is per-declaration (`compute_vla_size`) and per-call (the `alloca`-recognition).
- The driver gains `-include`, `-x`, `-l`, `-s`, `.a`/`.so` recognition, and the `-E` implies `-xc` rule.
- `emit_data` and `emit_text` emit `.type` and `.size` directives.
- The keyword list gains `_Thread_local` and `__thread`.

The chapter's seventeen-row summary, in `main` order:

| # | Hash | Subject | Section |
|---|---|---|---|
| 267 | `b377284` | Add thread-local variable | §21.1 |
| 268 | `8f5ff07` | Add `-include` option | §21.2 |
| 269 | `ee0a951` | Add `-x` option | §21.2 |
| 270 | `4064871` | Make `-E` imply `-xc` | §21.2 |
| 271 | `77275c5` | Add `alloca()` | §21.3 |
| 272 | `e8667af` | Add `sizeof()` for VLA | §21.4 |
| 273 | `07f9010` | Add pointer arithmetic for VLA | §21.4 |
| 274 | `2fa8f48` | Support `sizeof(typename)` where typename is a VLA | §21.4 |
| 275 | `b0109a3` | Do not define `__STDC_NO_VLA__` | §21.4 |
| 276 | `bc25279` | Add `-l` option | §21.5 |
| 277 | `c32f0e2` | Add `-s` option | §21.5 |
| 278 | `8d130ab` | Emit size and type for symbols | §21.5 |
| 279 | `d56dd2f` | Recognize `.a` and `.so` files | §21.5 |
| 280 | `e0bf168` | Add `long double` | §21.6 |
| 281 | `d90c73b` | `[GNU]` Support case ranges | §21.6 |
| 282 | `3d5550e` | `[GNU]` Support array range designator | §21.6 |
| 283 | `4f165ec` | `[GNU]` Support labels-as-values | §21.6 |

Errata candidates closed this chapter: two of the long-standing ones. `long double` is no longer aliased to `double` (the `e0bf168` commit makes it real 80-bit extended precision in the §21.6 walk). Array range designators are honored in elaboration (`3d5550e` in §21.6 closes the §19.7 errata). The remaining errata candidates are unchanged: Ch 17's three (`#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific), Ch 19's two remaining (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block), and Ch 20's one (`is_compatible` array arm bug).

Errata candidates added this chapter:

- The `.size` directive is missing for function symbols (in §21.5.3, commit `8d130ab`). Tools that read function size from the symbol table will see zero for chibicc-emitted functions; tools that compute size from labels will be fine.
- File-magic-vs-suffix recognition for `.a` and `.so` is suffix-only (in §21.5.4, commit `d56dd2f`). Versioned shared libraries (`libfoo.so.1.2.3`) won't be recognized.

The canonicalization-at-parse-time count ticks from ten to eleven with the VLA declaration rewrite (`int x[n]` becomes a `compute_vla_size; x = alloca(...)` pair). The pre-factor-before-feature count is unchanged at nine. The psABI conformance count grows by two: one for thread-local TLS access patterns, one for the long-double calling convention. New count: eighteen.

The standing notes for the next session: `Obj` is now even more substantial; `Type` has gained `vla_len`/`vla_size` and `TY_VLA`; the cast table is 11×11; the keyword list is around thirty-two entries. Two new node kinds for labels-as-values; one new for VLA pointers. The eval-quartet has a fifth member.

Through Chapter 21 chibicc handles thread-local storage, `alloca`, VLAs, `long double`, GNU case ranges and array range designators and labels-as-values, plus four more driver options. What remains: dependency-file emission for build systems, performance work (the parser and tokenizer have grown to the point where some operations are visibly slow), the linker-driver edge cases that real distributions exercise, and a long tail of compile-time-constant features. Those are the next twenty-three commits, in Chapter 22.
