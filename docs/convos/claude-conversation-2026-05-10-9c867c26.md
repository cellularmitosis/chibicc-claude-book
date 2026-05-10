# Claude Conversation Log

Session ID: 9c867c26-ee84-4c88-8202-dcdc35a73a64
Date: 2026-05-10 14:21:32

---

## 👤 User

please read docs/sessions/021-chapter-20-draft/HANDOFF.md and proceed

---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/021-chapter-20-draft/HANDOFF.md"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/021-chapter-20-draft/README.md"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "mkdir -p /Users/cell/claude/chibicc-book/docs/sessions/022-chapter-21-draft && cd /Users/cell/claude/chibicc-book/research/sources/chibicc && for h in b377284 8f5ff07 ee0a951 4064871 77275c5 e8667af 07f9010 2fa8f48 b0109a3 bc25279 c32f0e2 8d130ab d56dd2f e0bf168 d90c73b 3d5550e 4f165ec; do echo \"===== $h =====\"; git show --stat $h | head -15; done",
  "description": "Scan stat for all 17 Ch 21 commits"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/20-gcc-extensions-worth-supporting.md",
  "offset": 1,
  "limit": 100
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/20-gcc-extensions-worth-supporting.md",
  "offset": 1500,
  "limit": 300
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/20-gcc-extensions-worth-supporting.md",
  "offset": 1300,
  "limit": 71
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show b377284",
  "description": "Show thread-local commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 8f5ff07",
  "description": "Show -include commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show ee0a951 4064871",
  "description": "Show -x and -E imply -xc commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 77275c5",
  "description": "Show alloca commit"
}


---

## 🤖 Claude


🔧 Using tool: ToolSearch
Input: {
  "query": "select:TodoWrite",
  "max_results": 1
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 17 Ch 21 commits",
      "activeForm": "Reading all 17 Ch 21 commits",
      "status": "in_progress"
    },
    {
      "content": "Read destination state of key files",
      "activeForm": "Reading destination state of key files",
      "status": "pending"
    },
    {
      "content": "Draft chapter 21",
      "activeForm": "Drafting chapter 21",
      "status": "pending"
    },
    {
      "content": "Write session 022 README",
      "activeForm": "Writing session 022 README",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF for session 023",
      "activeForm": "Writing HANDOFF for session 023",
      "status": "pending"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show e8667af 2>&1 | head -200",
  "description": "Show sizeof(VLA) commit head"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show e8667af 2>&1 | tail -120",
  "description": "Show sizeof(VLA) commit tail"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 07f9010 2fa8f48 b0109a3",
  "description": "Show VLA pointer arith, sizeof typename, drop NO_VLA"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show bc25279 c32f0e2 8d130ab d56dd2f",
  "description": "Show -l, -s, ELF size/type, .a/.so commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show e0bf168 2>&1 | head -300",
  "description": "Show long double commit head"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show e0bf168 2>&1 | wc -l && git show e0bf168 2>&1 | tail -200",
  "description": "Show long double commit tail"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show e0bf168 2>&1 | sed -n '290,460p'",
  "description": "Show long double commit middle (binary ops)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show d90c73b 3d5550e 4f165ec",
  "description": "Show case ranges, array range designators, labels-as-values"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout 4f165ec 2>&1 | tail -3 && grep -n '\"\\\\.\\\\.\\\\.\"' tokenize.c | head -5; grep -n \"\\\"&&\\\"\" tokenize.c | head -5",
  "description": "Check ... and && in tokenizer"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -n '\\\\.\\\\.\\\\.' tokenize.c | head -5",
  "description": "find dots in tokenize.c"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -n 'punct\\|\\\\.\\\\.\\\\.' tokenize.c | head -10",
  "description": "find punctuator code in tokenize"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/research/sources/chibicc/tokenize.c",
  "offset": 143,
  "limit": 15
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout main 2>&1 | tail -3 && cd /Users/cell/claude/chibicc-book && ls chapters/",
  "description": "Restore main and list chapters"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 17 Ch 21 commits",
      "activeForm": "Reading all 17 Ch 21 commits",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files",
      "activeForm": "Reading destination state of key files",
      "status": "completed"
    },
    {
      "content": "Draft chapter 21",
      "activeForm": "Drafting chapter 21",
      "status": "in_progress"
    },
    {
      "content": "Write session 022 README",
      "activeForm": "Writing session 022 README",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF for session 023",
      "activeForm": "Writing HANDOFF for session 023",
      "status": "pending"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude

Now I have all the material. Let me draft the chapter.

---

## 🤖 Claude

Now drafting the chapter.

---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/21-thread-local-alloca-vlas.md",
  "content": "# Chapter 21 \u2014 Thread-local, alloca, VLAs\n\n> Commits covered: `b377284`, `8f5ff07`, `ee0a951`, `4064871`, `77275c5`, `e8667af`, `07f9010`, `2fa8f48`, `b0109a3`, `bc25279`, `c32f0e2`, `8d130ab`, `d56dd2f`, `e0bf168`, `d90c73b`, `3d5550e`, `4f165ec`. Seventeen commits \u2014 thread-local storage and the `%fs:`-relative addressing it brings in, three driver options that finish off the cc1-vs-driver split (`-include`, `-x`, the `-E` implies `-xc` rule), `alloca` as a real builtin, the four-commit VLA arc, four small linker-driver additions (`-l`, `-s`, ELF size/type emission, `.a`/`.so` recognition), and finally `long double` plus three GNU bracket-range features (`case 1 ... 5:`, `[3 ... 7] = x`, `&&label` and `goto *expr`).\n\nThrough Chapter 20 chibicc handles the gcc-extension surface that real-world C reaches for most often. What it doesn't yet handle, and what real-world code sometimes still wants, is thread-local storage, runtime-sized stack allocation (`alloca`), and variable-length arrays. None of these is a polish-level addition \u2014 each lands a new mechanism in the codegen. Thread-local storage requires `%fs:`-relative addressing and two new ELF sections. `alloca` requires the function epilogue to forget that it ever knew the post-prologue stack size. VLAs require the parser to invent hidden locals to remember runtime sizes, and `sizeof` has to know to read those locals instead of returning a compile-time constant.\n\nThe chapter also closes one long-standing errata candidate (`long double` finally becomes extended-precision rather than aliased to `double`) and another that has been outstanding since Chapter 19 (array range designators are finally honored in elaboration). And it adds the GNU bracket-range pattern in two more places: `case 1 ... 5:` and `&&label` for labels-as-values.\n\nSix sections from seventeen commits.\n\n- **\u00a721.1** \u2014 Thread-local variables (commit 267).\n- **\u00a721.2** \u2014 Driver: `-include`, `-x`, `-E` implies `-xc` (commits 268\u2013270).\n- **\u00a721.3** \u2014 `alloca` (commit 271).\n- **\u00a721.4** \u2014 VLAs: `sizeof(VLA)`, pointer arithmetic, `sizeof(typename)`, dropping `__STDC_NO_VLA__` (commits 272\u2013275).\n- **\u00a721.5** \u2014 Linker driver: `-l`, `-s`, ELF size/type, `.a`/`.so` (commits 276\u2013279).\n- **\u00a721.6** \u2014 `long double`, case ranges, array range designators, labels-as-values (commits 280\u2013283).\n\nThe chapter follows `main` order. As before, calendar order and `main` order differ \u2014 the four VLA commits land at positions 272\u2013275 but were written in three separate days across August 25 (`2fa8f48`, `b0109a3`) and September 3\u20134 (`07f9010`, `e8667af`), and the chapter does not remark on this except to note that the order in `main` is the order in which we walk the work.\n\n---\n\n## 21.1 \u2014 Thread-local variables\n\n> `git checkout b3772845bd07fb695ca6b6e67ad7640776ae0f6c` \u2014 *Add thread-local variable*\n\nC11 added the `_Thread_local` storage-class specifier (and gcc had long supported `__thread` as an extension): a variable so qualified gets a separate instance for each thread of execution. Reads and writes from one thread don't see the value the other thread has stored. The mechanism on x86-64 Linux is the `%fs` segment register, which the loader and pthread runtime point at the current thread's TLS block.\n\nRui's commit adds both keywords (`_Thread_local` and `__thread`), threads them through `VarAttr` and `Obj`, and emits the right addressing pattern in codegen.\n\n### The keyword pair and the `VarAttr` slot\n\n`is_keyword` and `is_typename` both grow two entries:\n\n```c\n\"typeof\", \"asm\", \"_Thread_local\", \"__thread\",\n```\n\n`VarAttr` gains a fifth boolean:\n\n```c\ntypedef struct {\n  bool is_typedef;\n  bool is_static;\n  bool is_extern;\n  bool is_inline;\n  bool is_tls;\n  int align;\n} VarAttr;\n```\n\nThe `declspec` storage-class branch already handled `typedef`, `static`, `extern`, and `inline` as a four-way exclusion \u2014 the existing code rejects combinations that aren't allowed. Thread-local enters the same channel, but with the relaxed rule that `_Thread_local` may *combine* with `static` or `extern` (a TLS variable can have either internal or external linkage). The diagnostic check picks up TLS as another mutual-exclusion partner with `typedef`:\n\n```c\nif (attr->is_typedef &&\n    attr->is_static + attr->is_extern + attr->is_inline + attr->is_tls > 1)\n  error_tok(tok, \"typedef may not be used together with static,\"\n            \" extern, inline, __thread or _Thread_local\");\n```\n\nThat's enough to admit `static _Thread_local int x;` (which is what real-world code most often writes). The standard's full set of rules for storage-class combinations isn't reproduced; chibicc accepts what gcc accepts in practice and rejects the worst combinations.\n\n### `Obj->is_tls` and the global-variable wiring\n\n`Obj` gains an `is_tls` flag, sitting next to `is_tentative`:\n\n```c\nstruct Obj {\n  ...\n  // Global variable\n  bool is_tentative;\n  bool is_tls;\n  char *init_data;\n  Relocation *rel;\n  ...\n};\n```\n\nThe flag is set by `global_variable` from the `attr` channel, and there's a one-line behavioral change: a TLS variable is *not* tentative even without an initializer. Tentative definitions are an artifact of the `.bss`/`.comm` machinery; thread-local globals don't share that path.\n\n```c\nvar->is_definition = !attr->is_extern;\nvar->is_static = attr->is_static;\nvar->is_tls = attr->is_tls;\n...\nif (equal(tok, \"=\"))\n  gvar_initializer(&tok, tok->next, var);\nelse if (!attr->is_extern && !attr->is_tls)\n  var->is_tentative = true;\n```\n\nA bare `_Thread_local int x;` therefore goes straight to `.tbss` (the zero-initialized TLS section) rather than to `.comm`.\n\n### `%fs:0` and `@tpoff`\n\nThe address-generation half is the chapter's first new asm pattern. `gen_addr` learns to recognize a TLS variable and emit a two-instruction sequence:\n\n```c\n// Thread-local variable\nif (node->var->is_tls) {\n  println(\"  mov %%fs:0, %%rax\");\n  println(\"  add $%s@tpoff, %%rax\", node->var->name);\n  return;\n}\n```\n\n`mov %fs:0, %rax` reads the per-thread base address from the segment register. The TLS implementation Linux uses places a small bookkeeping structure at `%fs:0` whose first slot is a self-pointer back to the TLS block. The result is a thread-specific pointer which `add $name@tpoff` then offsets into.\n\n`@tpoff` is the assembler relocation for \"thread-pointer offset.\" The linker computes the symbol's offset from the thread-pointer base at link time; the runtime adds that constant to whatever the segment register provides. There are several TLS access models (initial-exec, local-exec, general-dynamic, local-dynamic) with different cost/flexibility tradeoffs; the pattern chibicc emits is the *initial-exec* model, which is what gcc emits by default for TLS variables compiled into the executable. It's the cheapest model \u2014 two instructions, no library call \u2014 and works as long as the TLS block is statically allocated at program startup. Code in dynamically loaded shared libraries needs a more elaborate dance (`__tls_get_addr` and the global-dynamic model). Chibicc emits only initial-exec, which is sufficient for the things chibicc compiles.\n\n### `.tdata` and `.tbss`\n\n`emit_data` learns two new section directives:\n\n```c\n// .data or .tdata\nif (var->init_data) {\n  if (var->is_tls)\n    println(\"  .section .tdata,\\\"awT\\\",@progbits\");\n  else\n    println(\"  .data\");\n  ...\n}\n\n// .bss or .tbss\nif (var->is_tls)\n  println(\"  .section .tbss,\\\"awT\\\",@nobits\");\nelse\n  println(\"  .bss\");\n```\n\nThe section flags `awT` mark the section as allocatable, writable, and TLS (the `T` is the TLS bit). `@progbits` versus `@nobits` is the same distinction `.data` and `.bss` make: `.tdata` carries initialized template data; `.tbss` is zero-fill. The loader copies both into per-thread storage at thread creation; `_Thread_local int x = 5;` becomes a `.tdata` entry that every thread initializes to 5 on creation.\n\nThis is also where the count of assembly sections chibicc can emit grows. Through Chapter 20 the count was three (`.text`, `.data`, `.bss`) plus the `.comm` directive for tentative commons. After this commit it's five \u2014 `.text`, `.data`, `.bss`, `.tdata`, `.tbss` \u2014 plus `.comm`.\n\n### `__STDC_NO_THREADS__` and the test plumbing\n\nChibicc previously defined `__STDC_NO_THREADS__` to declare itself non-thread-aware. The flag's removal here is honest: chibicc doesn't implement `<threads.h>` (the C11 thread library), but the underlying compiler now emits TLS, which is the language piece. `<threads.h>` is a libc concern, not a compiler concern, and pthreads (which the test uses) is a separate POSIX interface that chibicc doesn't need to know about at the language level.\n\n```c\n-  define_macro(\"__STDC_NO_THREADS__\", \"1\");\n```\n\nThe `Makefile` change adds `-pthread` to the link command for tests, so the new `test/tls.c` (which uses `pthread_create` to verify thread-local separation) links against libpthread.\n\n### The test: thread separation\n\n`test/tls.c` is the smallest test that proves the feature works:\n\n```c\n_Thread_local int v1;\n_Thread_local int v2 = 5;\nint v3 = 7;\n\nint thread_main(void *unused) {\n  ASSERT(0, v1);   // child sees fresh zero\n  ASSERT(5, v2);   // child sees fresh initialized value\n  ASSERT(7, v3);   // shared global\n  v1 = 1; v2 = 2; v3 = 3;\n  ...\n}\n\nint main() {\n  ...\n  ASSERT(0, pthread_create(&thr, NULL, thread_main, NULL));\n  ASSERT(0, pthread_join(thr, NULL));\n  ASSERT(0, v1);   // main's v1 unchanged\n  ASSERT(5, v2);   // main's v2 unchanged\n  ASSERT(3, v3);   // shared global was modified\n}\n```\n\nThe two TLS variables retain their per-thread initial values in `main` even after the child has overwritten its copies; the non-TLS `v3` shows the shared-state baseline. That's the entire semantic surface.\n\n**Where we are.** `_Thread_local` and `__thread` are accepted as storage-class specifiers. TLS variables are placed in `.tdata` (initialized) or `.tbss` (zero-fill); accesses use `%fs:0 + name@tpoff`, which is the initial-exec model. Dynamic-library TLS access (which would call `__tls_get_addr`) is not implemented. `__STDC_NO_THREADS__` is no longer defined \u2014 chibicc no longer claims to lack threads at the language level, even though it doesn't implement `<threads.h>`. The psABI conformance count ticks up by one: TLS access patterns are part of the psABI's thread-local model.\n\n---\n\n## 21.2 \u2014 Driver: `-include`, `-x`, and `-E` implies `-xc`\n\n> `git checkout 8f5ff07dc08d258209adf60ed8e796efa7b7a476` \u2014 *Add -include option*\n>\n> `git checkout ee0a951b30646023ccc9a144afb4b380bf8d09b1` \u2014 *Add -x option*\n>\n> `git checkout 4064871212049d82af3632941d15e6a0757ebc3c` \u2014 *Make -E to imply -xc*\n\nThree driver-side commits. They deal with two adjacent ergonomic problems: getting headers into the input without rewriting the source (`-include`), and naming the input language explicitly when extension-based detection won't do (`-x`).\n\n### 21.2.1 \u2014 `-include`: prepended #includes\n\n`-include FILE` tells the compiler to behave as if `#include \"FILE\"` appeared at the top of the input. gcc uses it for project-wide preludes and for selectively defining feature macros. The implementation is straightforward \u2014 collect `-include` paths in a `StringArray`, then prepend tokenized copies of each file before the main input.\n\nThe driver gains `opt_include`:\n\n```c\nstatic StringArray opt_include;\n```\n\nArgument parsing accepts `-include FILE` (the option needs a separate argument; `take_arg` learns the new name):\n\n```c\nstatic bool take_arg(char *arg) {\n  char *x[] = {\"-o\", \"-I\", \"-idirafter\", \"-include\"};\n  ...\n}\n```\n\n```c\nif (!strcmp(argv[i], \"-include\")) {\n  strarray_push(&opt_include, argv[++i]);\n  continue;\n}\n```\n\nThe interesting half is in `cc1`. Previously it did `tokenize_file(base_file)` and hand-waved the rest. Now it processes `-include` first and chains the resulting token streams:\n\n```c\nstatic void cc1(void) {\n  Token *tok = NULL;\n\n  // Process -include option\n  for (int i = 0; i < opt_include.len; i++) {\n    char *incl = opt_include.data[i];\n\n    char *path;\n    if (file_exists(incl)) {\n      path = incl;\n    } else {\n      path = search_include_paths(incl);\n      if (!path)\n        error(\"-include: %s: %s\", incl, strerror(errno));\n    }\n\n    Token *tok2 = must_tokenize_file(path);\n    tok = append_tokens(tok, tok2);\n  }\n\n  Token *tok2 = must_tokenize_file(base_file);\n  tok = append_tokens(tok, tok2);\n  tok = preprocess(tok);\n  ...\n}\n```\n\nTwo helpers fall out: `must_tokenize_file` (tokenize-or-die) and `append_tokens` (concatenate two streams, dropping the terminating `TK_EOF` of the first). Note the path resolution: a `-include` argument that looks like a file path is taken verbatim if it exists; otherwise it's resolved through `search_include_paths`, which is now exposed in `chibicc.h` because `cc1` lives in `main.c` and the search function lives in `preprocess.c`.\n\nToken streams concatenated this way are preprocessed as a single unit. A `#define` in a `-include` file is visible to the main translation unit, and so are any tokenizer-state effects (though there shouldn't be any \u2014 the tokenizer is stateless apart from line numbering, which it tracks per-file).\n\n### 21.2.2 \u2014 `-x`: language override\n\n`-x LANG` overrides the file-extension-based language detection that `cc1` previously did with `endswith(input, \".c\")` and `endswith(input, \".s\")`. The motivation is twofold: stdin (`-`) has no extension; and you sometimes want to compile a file as if it had a different extension (`-x assembler` to treat a `.S` input as raw assembly without the C preprocessor, for example).\n\nThe valid arguments are `c`, `assembler`, and `none`. `none` resets the override so that subsequent inputs go back to extension-based detection, which means a single command line can mix languages:\n\n```\n$chibicc -c -x assembler $tmp/foo.s -x none $tmp/bar.c\n```\n\nThe implementation introduces a `FileType` enum:\n\n```c\ntypedef enum { FILE_NONE, FILE_C, FILE_ASM, FILE_OBJ } FileType;\n```\n\nand an `opt_x` global. Argument parsing handles both the spaced (`-x c`) and the joined (`-xc`) forms:\n\n```c\nif (!strcmp(argv[i], \"-x\")) {\n  opt_x = parse_opt_x(argv[++i]);\n  continue;\n}\n\nif (!strncmp(argv[i], \"-x\", 2)) {\n  opt_x = parse_opt_x(argv[i] + 2);\n  continue;\n}\n```\n\nThe joined form is what gcc accepts and what real Makefiles tend to use. The spaced form has to be matched first because `-x` is on the `take_arg` list.\n\n`get_file_type` is now the centralized lookup:\n\n```c\nstatic FileType get_file_type(char *filename) {\n  if (endswith(filename, \".o\"))\n    return FILE_OBJ;\n  if (opt_x != FILE_NONE)\n    return opt_x;\n  if (endswith(filename, \".c\"))\n    return FILE_C;\n  if (endswith(filename, \".s\"))\n    return FILE_ASM;\n  error(\"<command line>: unknown file extension: %s\", filename);\n}\n```\n\nThe `.o` case takes precedence over `opt_x` \u2014 a compiled object file is an object file regardless of any `-x` setting.\n\n`main`'s per-input loop then dispatches on the result:\n\n```c\nFileType type = get_file_type(input);\nif (type == FILE_OBJ) { ... }\nif (type == FILE_ASM) { ... }\nassert(type == FILE_C);\n```\n\nNote that the previous \"no extension; assume C\" hack \u2014 the explicit `if (!endswith(input, \".c\") && strcmp(input, \"-\"))` \u2014 disappears. Stdin no longer falls through magic; it's explicitly handled with `-x` (or by the `-E implies -xc` rule below).\n\nThe test driver picks up an awkward consequence: every test that pipes into chibicc now needs `-xc -` so the driver knows the input is C. That's the bulk of the `test/driver.sh` diff \u2014 twenty-odd lines change from `... | $chibicc -E - | ...` to `... | $chibicc -E -xc - | ...`. It's a churn cost paid in test infrastructure for a clean cc1 abstraction.\n\n### 21.2.3 \u2014 `-E` implies `-xc`\n\nThree commits later (about a week of calendar time), Rui revisits the test churn from `-x` and adds a small ergonomic rule: if `-E` is given, the input is assumed to be C. That makes sense \u2014 `-E` means \"preprocess only\" and the C macro language is what the preprocessor handles.\n\n```c\n// -E implies that the input is the C macro language.\nif (opt_E)\n  opt_x = FILE_C;\n```\n\nThe change reverses the test-driver churn for the `-E` path: `echo foo | $chibicc -E -` works again.\n\n**Where we are.** Three driver options. `-include FILE` prepends a tokenized include file before the main input; multiple `-include` arguments stack in command-line order. `-x LANG` overrides extension-based detection for subsequent inputs (`c`, `assembler`, `none`). `-E` implies `-xc` so that preprocessing-only mode keeps working on stdin. The pre-existing `cc1` hack of \"if there's no extension and the input isn't `-`, error out\" is gone \u2014 language detection is centralized in `get_file_type`.\n\n---\n\n## 21.3 \u2014 `alloca`\n\n> `git checkout 77275c546a5340f94ad011cd759ef162bc714ba6` \u2014 *Add alloca()*\n\n`alloca(n)` allocates `n` bytes on the stack and returns a pointer to the allocation. It's a long-standing extension (GCC, BSD libc, glibc), not a standard library function \u2014 the C standard takes no position on it because no portable way to implement it within standard C exists. Its appeal: very fast allocation (a stack-pointer adjustment) with automatic deallocation at function return. Its risks: stack exhaustion if `n` is unbounded, and undefined behavior if the result outlives the caller.\n\nChibicc's implementation is a builtin. The compiler recognizes calls to a function named `alloca` and emits an inline asm sequence rather than a real function call.\n\n### The builtin declaration\n\nBefore parsing starts, the compiler synthesizes a declaration for `alloca`:\n\n```c\nstatic void declare_builtin_functions(void) {\n  Type *ty = func_type(pointer_to(ty_void));\n  ty->params = copy_type(ty_int);\n  Obj *builtin = new_gvar(\"alloca\", ty);\n  builtin->is_definition = false;\n}\n```\n\nA `void *alloca(int)` is added to globals as a non-definition. Programs that declare `void *alloca(size_t);` themselves shadow this gracefully \u2014 the typechecking comes from the program's declaration. The builtin is just there so an unprefixed `alloca(n)` doesn't fail to resolve.\n\nThe reason the builtin is registered as a global rather than recognized by name in codegen alone is that `alloca`'s call site needs to typecheck like any other function call \u2014 `alloca(16)` returns a `void *`, the argument is an integer expression, and the parser needs `func_ty` and `return_ty` plumbed correctly. Codegen then specializes when it recognizes the call.\n\n### The codegen specialization\n\n`gen_expr`'s `ND_FUNCALL` case learns to recognize the builtin:\n\n```c\ncase ND_FUNCALL: {\n  if (node->lhs->kind == ND_VAR && !strcmp(node->lhs->var->name, \"alloca\")) {\n    gen_expr(node->args);\n    println(\"  mov %%rax, %%rdi\");\n    builtin_alloca();\n    return;\n  }\n  ...\n}\n```\n\nThe argument is evaluated into `%rax`, copied into `%rdi`, and the helper emits the allocation sequence inline.\n\n### `builtin_alloca` and the `alloca_bottom` cell\n\nThe hard part is making the result interact correctly with the rest of the function. chibicc's stack frame currently holds locals at fixed offsets from `%rbp`, so `%rsp` between the prologue and a return is normally fixed. `alloca` has to grow the frame downward at runtime, which means the *temporary* area chibicc uses for evaluation pushes (between `%rsp` and the locals) has to move to make room \u2014 without disturbing any pushed values that have meaningful contents.\n\nRui's approach: for each function, reserve a hidden local that records the current bottom of the temporary area. Call this `alloca_bottom`. When we want to allocate `n` bytes, slide the contents of the temporary area down by `n` aligned bytes and update `alloca_bottom` to remember the new boundary. Future allocations slide again.\n\nThe hidden local is added in `function`:\n\n```c\nfn->alloca_bottom = new_lvar(\"__alloca_size__\", pointer_to(ty_char));\n```\n\n`Obj` gets a new field:\n\n```c\nObj *alloca_bottom;\n```\n\nand the prologue emits a store of the post-prologue `%rsp` into the local:\n\n```c\nprintln(\"  push %%rbp\");\nprintln(\"  mov %%rsp, %%rbp\");\nprintln(\"  sub $%d, %%rsp\", fn->stack_size);\nprintln(\"  mov %%rsp, %d(%%rbp)\", fn->alloca_bottom->offset);\n```\n\nAfter the prologue, the cell holds the current bottom of \"memory below `%rsp` is yours to allocate.\" Initially that's the same as `%rsp` because nothing has been pushed for evaluation yet.\n\n`builtin_alloca` is then a small in-line assembler that does three things: align the allocation size to 16 bytes, copy the existing temporary area down by that many bytes, and update `alloca_bottom`:\n\n```c\nstatic void builtin_alloca(void) {\n  // Align size to 16 bytes.\n  println(\"  add $15, %%rdi\");\n  println(\"  and $0xfffffff0, %%edi\");\n\n  // Shift the temporary area by %rdi.\n  println(\"  mov %d(%%rbp), %%rcx\", current_fn->alloca_bottom->offset);\n  println(\"  sub %%rsp, %%rcx\");\n  println(\"  mov %%rsp, %%rax\");\n  println(\"  sub %%rdi, %%rsp\");\n  println(\"  mov %%rsp, %%rdx\");\n  println(\"1:\");\n  println(\"  cmp $0, %%rcx\");\n  println(\"  je 2f\");\n  println(\"  mov (%%rax), %%r8b\");\n  println(\"  mov %%r8b, (%%rdx)\");\n  println(\"  inc %%rdx\");\n  println(\"  inc %%rax\");\n  println(\"  dec %%rcx\");\n  println(\"  jmp 1b\");\n  println(\"2:\");\n\n  // Move alloca_bottom pointer.\n  println(\"  mov %d(%%rbp), %%rax\", current_fn->alloca_bottom->offset);\n  println(\"  sub %%rdi, %%rax\");\n  println(\"  mov %%rax, %d(%%rbp)\", current_fn->alloca_bottom->offset);\n}\n```\n\nThe byte-by-byte copy is the bookkeeping: any value chibicc has previously pushed for evaluation (between `%rsp` and `alloca_bottom`) needs to live at the *new* bottom after the slide, not the old one. The new allocation goes into the gap that opens up between the new `%rsp` and where the old `%rsp` was. The returned pointer is left in `%rax` \u2014 `alloca_bottom - aligned_n`, which is the start of the freshly-allocated region.\n\nThe function epilogue is the saving grace. chibicc's epilogue is uniformly `mov %rbp, %rsp; pop %rbp; ret` (or an explicit `mov` to `%rsp` from the saved `%rbp`), which means the entire `alloca`-allocated region \u2014 and the slid temporary area \u2014 is released in one stroke when the function returns. This is exactly why `alloca` works in chibicc's compilation model: the epilogue tears down the frame in one step, regardless of how it grew during the body.\n\nThe 16-byte alignment is overkill for most uses but matches what gcc does, and it's free (one masked `add`).\n\n### `alloca_bottom` as a tracked field\n\nThe local is hidden \u2014 its name is the empty string \u2014 but its `Obj` is on the function's locals list and thus gets a stack offset assigned by `assign_lvar_offsets` like any other local. The codegen then references it by `offset(%rbp)` in the prologue and in `builtin_alloca`.\n\nThe presence of an `__alloca_size__` local in *every* function (regardless of whether `alloca` is actually called) is the small cost of always-on instrumentation. It's eight bytes per function. Worth nothing.\n\n### Test cases\n\n`test/alloca.c` exercises four allocations interleaved with arithmetic and a function call:\n\n```c\nint main() {\n  int i = 0;\n\n  char *p1 = alloca(16);\n  char *p2 = alloca(16);\n  char *p3 = 1 + (char *)alloca(3) + 1;\n  p3 -= 2;\n  char *p4 = fn(1, alloca(16), 3);\n\n  ASSERT(16, p1 - p2);\n  ASSERT(16, p2 - p3);\n  ASSERT(16, p3 - p4);\n  ...\n}\n```\n\nThe pointer-arithmetic test (`1 + (char *)alloca(3) + 1`) checks that the allocation outlives the surrounding arithmetic, and the `fn(1, alloca(16), 3)` call checks that an `alloca` result computed as a function argument doesn't get clobbered by other evaluation pushes.\n\n**Where we are.** `alloca(n)` is a builtin. The compiler synthesizes a `void *alloca(int)` declaration at parse start, recognizes `ND_FUNCALL` to a variable named `alloca` in codegen, and emits an inline shift-the-temp-area sequence that reserves `n` aligned bytes and returns a pointer. The implementation depends on a per-function hidden local `__alloca_size__` (the field is `alloca_bottom` on `Obj`). The allocation lives until the function epilogue tears down the frame. Stack-exhaustion is not detected; that's the standard `alloca` caveat. The eight-byte cost is paid by every function whether it calls `alloca` or not.\n\n---\n\n## 21.4 \u2014 Variable-length arrays\n\n> `git checkout e8667afd08ecbf7c9b05beb4ff399959d9722ff9` \u2014 *Add sizeof() for VLA*\n>\n> `git checkout 07f901057f5c6aa77c0f15f7a22dc0b88923c227` \u2014 *Add pointer arithmetic for VLA*\n>\n> `git checkout 2fa8f489f3a852bd5bb17e023fdc5ea3a606100d` \u2014 *Support sizeof(typename) where typename is a VLA*\n>\n> `git checkout b0109a30c9fa24fedcb4d79bb17788e7ed228636` \u2014 *Do not define __STDC_NO_VLA__*\n\nVLAs were added to C in C99 and made optional in C11 (a hosted implementation may define `__STDC_NO_VLA__` to opt out). They allow array dimensions to be ordinary integer expressions evaluated at run time:\n\n```c\nvoid f(int n) {\n  int x[n];      // VLA \u2014 size is n at the time f is entered\n  int y[n+2][m]; // multi-dimensional VLA\n}\n```\n\nThe size has to be remembered between the declaration and any later use of `sizeof(x)`. chibicc's approach: rewrite the VLA declaration into a sequence \u2014 compute the size into a hidden local, allocate via `alloca`, and remember the local for `sizeof` to read later.\n\nThe arc lands in four commits, in main order. The first three actually add the feature; the last just drops the `__STDC_NO_VLA__` predefined macro because chibicc no longer lacks the feature. Commit order doesn't match calendar order \u2014 `2fa8f48` and `b0109a3` are dated August 25, `07f9010` and `e8667af` come in early September. Rui's branches landed in the order of dependent functionality rather than the order they were written.\n\n### 21.4.1 \u2014 `TY_VLA` and `vla_of`\n\n`TypeKind` gains a new entry, and `Type` gains two new fields:\n\n```c\ntypedef enum {\n  ...\n  TY_ARRAY,\n  TY_VLA, // variable-length array\n  ...\n} TypeKind;\n\nstruct Type {\n  ...\n  // Variable-length array\n  Node *vla_len;  // # of elements\n  Obj *vla_size;  // sizeof() value\n  ...\n};\n```\n\n`vla_len` is the parsed expression for the dimension (the `n+2` in `int x[n+2]`). `vla_size` is the hidden local where the *byte count* (length \u00d7 element size) gets stored at run time.\n\nA constructor sits next to `array_of`:\n\n```c\nType *vla_of(Type *base, Node *len) {\n  Type *ty = new_type(TY_VLA, 8, 8);\n  ty->base = base;\n  ty->vla_len = len;\n  return ty;\n}\n```\n\nNote the size is 8 (a pointer's worth) and the alignment is 8. `TY_VLA` represents the *array as a value*, which from the codegen's perspective is a stack-allocated pointer \u2014 the underlying storage lives wherever `alloca` put it.\n\n### 21.4.2 \u2014 `array_dimensions` becomes the VLA decision point\n\nThe dimension-parser previously folded the bracket expression to a constant:\n\n```c\nint sz = const_expr(&tok, tok);\nreturn array_of(ty, sz);\n```\n\nNow it parses the bracket as an arbitrary expression and decides afterward whether it folds to a constant:\n\n```c\nNode *expr = conditional(&tok, tok);\ntok = skip(tok, \"]\");\nty = type_suffix(rest, tok, ty);\n\nif (ty->kind == TY_VLA || !is_const_expr(expr))\n  return vla_of(ty, expr);\nreturn array_of(ty, eval(expr));\n```\n\nThe check is \"is the result type already a VLA, or is the dimension non-constant?\" \u2014 if either, the result is a VLA. The \"already a VLA\" test handles arrays-of-VLAs (the inner type is fixed, the outer dimension carries the variability). `is_const_expr` is the new compile-time-constancy predicate.\n\n### 21.4.3 \u2014 `is_const_expr`: a structural recursion\n\n`eval` evaluates a node *as a constant expression*; if the node isn't constant, `eval` errors. Previously the only way to know if a node was constant was to try `eval` and catch the error. The dimension parser needs to *test* without throwing, so `is_const_expr` is added as a structural recursion that returns false on anything not statically computable:\n\n```c\nstatic bool is_const_expr(Node *node) {\n  add_type(node);\n\n  switch (node->kind) {\n  case ND_ADD: case ND_SUB: case ND_MUL: case ND_DIV:\n  case ND_BITAND: case ND_BITOR: case ND_BITXOR:\n  case ND_SHL: case ND_SHR:\n  case ND_EQ: case ND_NE: case ND_LT: case ND_LE:\n  case ND_LOGAND: case ND_LOGOR:\n    return is_const_expr(node->lhs) && is_const_expr(node->rhs);\n  case ND_COND:\n    if (!is_const_expr(node->cond))\n      return false;\n    return is_const_expr(eval(node->cond) ? node->then : node->els);\n  case ND_COMMA:\n    return is_const_expr(node->rhs);\n  case ND_NEG: case ND_NOT: case ND_BITNOT: case ND_CAST:\n    return is_const_expr(node->lhs);\n  case ND_NUM:\n    return true;\n  }\n  return false;\n}\n```\n\nThe `ND_COND` arm is interesting: if the condition is constant, only the chosen arm has to be constant. That matches C's \"discard the unselected arm\" rule for constant expressions. `ND_COMMA` follows the same rule (the value is the right-hand side; the left-hand side just runs).\n\nThe omitted node kinds (variables, function calls, dereferences, struct accesses, address-of, pre/post increments, anything not numerically reducible) all fall through to `return false`. So `int x[n]` \u2014 where `n` is a variable \u2014 yields `is_const_expr(n) == false`, which tips `array_dimensions` into the VLA path.\n\nThe eval-quartet picture from earlier chapters now has a fifth member: `eval`, `eval2`, `eval_rval`, `eval_double`, and `is_const_expr`. The duplication that would let one walk subsume the others remains.\n\n### 21.4.4 \u2014 `compute_vla_size`: chained ND_COMMA evaluations\n\nWhen a VLA declaration is processed, we have to:\n\n1. Evaluate every `vla_len` expression in declaration order.\n2. Multiply each by the element size of its base.\n3. Store the products into the per-dimension `vla_size` locals.\n\nMulti-dimensional VLAs cascade \u2014 `int x[m][n]` needs the inner dimension's `vla_size` to be the inner row size in bytes (`n * sizeof(int)`), and the outer dimension's `vla_size` to be the full array size (`m * inner_vla_size`).\n\n`compute_vla_size` walks the type and produces a comma-expression that does all the work:\n\n```c\nstatic Node *compute_vla_size(Type *ty, Token *tok) {\n  Node *node = new_node(ND_NULL_EXPR, tok);\n  if (ty->base)\n    node = new_binary(ND_COMMA, node, compute_vla_size(ty->base, tok), tok);\n\n  if (ty->kind != TY_VLA)\n    return node;\n\n  Node *base_sz;\n  if (ty->base->kind == TY_VLA)\n    base_sz = new_var_node(ty->base->vla_size, tok);\n  else\n    base_sz = new_num(ty->base->size, tok);\n\n  ty->vla_size = new_lvar(\"\", ty_ulong);\n  Node *expr = new_binary(ND_ASSIGN, new_var_node(ty->vla_size, tok),\n                          new_binary(ND_MUL, ty->vla_len, base_sz, tok),\n                          tok);\n  return new_binary(ND_COMMA, node, expr, tok);\n}\n```\n\nThe recursion descends `ty->base` first so inner VLAs have their `vla_size` locals populated before the outer multiplication uses them. The result is a chain of `ND_COMMA`s starting with a `ND_NULL_EXPR` (a do-nothing seed), followed by one `vla_size = vla_len * base_sz` assignment per VLA dimension. The `vla_size` itself is an unsigned-long local, allocated by `new_lvar(\"\", ty_ulong)` with an empty name (it can't collide with anything user-typed).\n\nThe tree shape, for `int x[m][n]`, is roughly:\n\n```\n((NULL_EXPR, (NULL_EXPR, NULL_EXPR)),\n (inner_size = n * sizeof(int))),\n (outer_size = m * inner_size)\n```\n\nwhich evaluates left-to-right, so `n` is read before `m * inner_size` is computed.\n\n### 21.4.5 \u2014 The declaration rewrite\n\nIn `declaration`, after parsing a declarator, we synthesize the size-computation as an expression statement, and (if the type is a VLA) follow it with an `alloca`-and-assign:\n\n```c\n// Generate code for computing a VLA size. We need to do this\n// even if ty is not VLA because ty may be a pointer to VLA\n// (e.g. int (*foo)[n][m] where n and m are variables.)\ncur = cur->next = new_unary(ND_EXPR_STMT, compute_vla_size(ty, tok), tok);\n\nif (ty->kind == TY_VLA) {\n  if (equal(tok, \"=\"))\n    error_tok(tok, \"variable-sized object may not be initialized\");\n\n  // Variable length arrays (VLAs) are translated to alloca() calls.\n  // For example, `int x[n+2]` is translated to `tmp = n + 2,\n  // x = alloca(tmp)`.\n  Obj *var = new_lvar(get_ident(ty->name), ty);\n  Token *tok = ty->name;\n  Node *expr = new_binary(ND_ASSIGN, new_var_node(var, tok),\n                          new_alloca(new_var_node(ty->vla_size, tok)),\n                          tok);\n\n  cur = cur->next = new_unary(ND_EXPR_STMT, expr, tok);\n  continue;\n}\n```\n\nThe comment is the spec of the rewrite: `int x[n+2]` becomes (conceptually) `tmp = (n+2) * sizeof(int); x = alloca(tmp);`. The actual lowering is a pair of expression statements \u2014 the size computation, then the assignment of the `alloca` result to the array variable.\n\n`new_alloca` is the helper that builds an `ND_FUNCALL` to the synthesized `alloca` builtin:\n\n```c\nstatic Node *new_alloca(Node *sz) {\n  Node *node = new_unary(ND_FUNCALL, new_var_node(builtin_alloca, sz->tok), sz->tok);\n  node->func_ty = builtin_alloca->ty;\n  node->ty = builtin_alloca->ty->return_ty;\n  node->args = sz;\n  add_type(sz);\n  return node;\n}\n```\n\n`builtin_alloca` is now also captured into a file-static so this code can refer to it without re-looking up. The call goes through the normal `ND_FUNCALL` path, which means codegen sees the same special-case it added in \u00a721.3 \u2014 a call to a variable named `alloca` becomes the inline allocation sequence.\n\nA VLA initializer is rejected with an explicit error: `\"variable-sized object may not be initialized\"`. That's what the standard requires; chibicc just reproduces the rule.\n\nNote also the comment at the top about pointer-to-VLA: even if the declared type isn't itself a VLA, its base may be (`int (*foo)[n][m]`), and we still need to evaluate `n` and `m` at the declaration site to fix `sizeof(*foo)` later. The unconditional `compute_vla_size` call handles that.\n\n### 21.4.6 \u2014 `ND_VLA_PTR` and the VLA-typed local\n\nInitially the rewrite uses `new_var_node(var, tok)` to build the assignment target. After the next commit (`07f9010`, \"Add pointer arithmetic for VLA\") it switches to a new node kind:\n\n```c\nNode *expr = new_binary(ND_ASSIGN, new_vla_ptr(var, tok),\n                        new_alloca(new_var_node(ty->vla_size, tok)),\n                        tok);\n```\n\nWhy? A `TY_VLA` lvalue normally decays to a pointer when read (just like `TY_ARRAY`), but the *target* of the assignment is the eight-byte slot that holds the pointer. `gen_addr` for an `ND_VAR` of `TY_VLA` already emits `mov offset(%rbp), %rax` (loading the pointer). For the assignment target we want `lea offset(%rbp), %rax` (the address of the slot). `ND_VLA_PTR` is the new node kind for \"the slot itself, not the pointer in it\":\n\n```c\ncase ND_VLA_PTR:\n  println(\"  lea %d(%%rbp), %%rax\", node->var->offset);\n  return;\n```\n\n`add_type` reuses `ND_VAR`'s type rule:\n\n```c\ncase ND_VAR:\ncase ND_VLA_PTR:\n  node->ty = node->var->ty;\n  return;\n```\n\nThe split between \"read the pointer at the slot\" and \"address of the slot\" is the same kind of split that a pointer-to-array forces in `&arr` versus `arr`. VLAs need the same machinery because a VLA-typed local is, at the codegen level, exactly a pointer to a fresh `alloca` block.\n\n### 21.4.7 \u2014 `gen_addr` for VLA-typed locals\n\n```c\ncase ND_VAR:\n  // Variable-length array, which is always local.\n  if (node->var->ty->kind == TY_VLA) {\n    println(\"  mov %d(%%rbp), %%rax\", node->var->offset);\n    return;\n  }\n  ...\n```\n\nReading an `ND_VAR` of VLA type loads the slot's contents (the pointer to the alloca'd buffer). Decay-to-pointer is implicit: any later subscripting indexes that pointer with the right element size. The note in the comment (\"which is always local\") is the chibicc-specific narrowing \u2014 global VLAs aren't a thing the language allows anyway, but the comment names that codegen relies on it.\n\n### 21.4.8 \u2014 Pointer arithmetic on VLA-typed bases\n\n`new_add` and `new_sub` previously multiplied integer offsets by `lhs->ty->base->size`. For a VLA base, `size` is meaningless (it's whatever `new_type` happened to set; in practice 8). The replacement reads `vla_size` instead:\n\n```c\n// VLA + num\nif (lhs->ty->base->kind == TY_VLA) {\n  rhs = new_binary(ND_MUL, rhs, new_var_node(lhs->ty->base->vla_size, tok), tok);\n  return new_binary(ND_ADD, lhs, rhs, tok);\n}\n```\n\nThe runtime-loaded `vla_size` becomes the multiplier. `new_sub` does the same. So `&x[i]` for a VLA `x` correctly scales `i` by the row size that was computed at declaration time.\n\nThe `load` function also has to know not to load a value from a VLA-typed lvalue \u2014 `TY_VLA` joins `TY_ARRAY`, `TY_STRUCT`, `TY_UNION`, `TY_FUNC` in the \"don't try to fit this in a register\" club:\n\n```c\ncase TY_VLA:\n  // If it is an array, do not attempt to load a value to the\n  // register because in general we can't load an entire array to a\n  // register. ...\n```\n\nThe `gen_addr` change above has already made `&x` (or any direct read of `x`) produce the pointer; `load` just doesn't second-guess.\n\n### 21.4.9 \u2014 `sizeof(VLA)` reads the hidden local\n\n`primary`'s `sizeof` arms learn to check for `TY_VLA`:\n\n```c\nif (equal(tok, \"sizeof\") && equal(tok->next, \"(\") && is_typename(tok->next->next)) {\n  Type *ty = typename(&tok, tok->next->next);\n  *rest = skip(tok, \")\");\n  if (ty->kind == TY_VLA)\n    return new_var_node(ty->vla_size, tok);\n  return new_ulong(ty->size, start);\n}\n\nif (equal(tok, \"sizeof\")) {\n  Node *node = unary(rest, tok->next);\n  add_type(node);\n  if (node->ty->kind == TY_VLA)\n    return new_var_node(node->ty->vla_size, tok);\n  return new_ulong(node->ty->size, tok);\n}\n```\n\nFor a fixed array the result is a compile-time `ulong` constant. For a VLA it's a runtime read of the `vla_size` local. That local has already been written by the declaration's `compute_vla_size` lowering, so the read returns the right value at the right time.\n\nThe `2fa8f48` commit then extends `sizeof(typename)` to handle the case where the `typename` itself is a VLA. The previous form assumed the VLA was already a declared variable (so `vla_size` was already populated); for `sizeof(int [n][n+2])` we have to compute the size on the spot:\n\n```c\nif (ty->kind == TY_VLA) {\n  if (ty->vla_size)\n    return new_var_node(ty->vla_size, tok);\n\n  Node *lhs = compute_vla_size(ty, tok);\n  Node *rhs = new_var_node(ty->vla_size, tok);\n  return new_binary(ND_COMMA, lhs, rhs, tok);\n}\n```\n\nIf the type already has a populated `vla_size` (because it came from a declared variable), we reuse it. Otherwise we generate an inline `compute_vla_size` chain followed by a read of the freshly-populated local. The result is a `(compute, read)` comma-expression that evaluates the dimensions and returns the byte count.\n\n### 21.4.10 \u2014 `__STDC_NO_VLA__` removed\n\nThe fourth commit is one line:\n\n```c\n-  define_macro(\"__STDC_NO_VLA__\", \"1\");\n```\n\nThe macro signaled \"this implementation deliberately doesn't support VLAs\"; chibicc no longer needs to claim that.\n\n### Test cases\n\n`test/vla.c` ranges from the simplest declaration (`int n=5; int x[n]; sizeof(x)`) through nested pointer-to-VLA (`int (*x)[n][n+2]`) through fill-and-read (using the row-size-aware pointer arithmetic) through bare `sizeof(char[2][n])`:\n\n```c\nASSERT(20, ({ int n=5; int x[n]; sizeof(x); }));\nASSERT((5+1)*(8*2)*4, ({ int m=5, n=8; int x[m+1][n*2]; sizeof(x); }));\nASSERT(8, ({ char n=10; int (*x)[n][n+2]; sizeof(x); }));\nASSERT(480, ({ char n=10; int (*x)[n][n+2]; sizeof(*x); }));\nASSERT(48, ({ char n=10; int (*x)[n][n+2]; sizeof(**x); }));\nASSERT(4, ({ char n=10; int (*x)[n][n+2]; sizeof(***x); }));\nASSERT(60, ({ char n=3; int x[5][n]; sizeof(x); }));\n...\nASSERT(0, ({ int n=10; int x[n+1][n+6]; int *p=x;\n             for (int i = 0; i<sizeof(x)/4; i++) p[i]=i;\n             x[0][0]; }));\nASSERT(10, ({ int n=5; sizeof(char[2][n]); }));\n```\n\nThe fill-and-read tests are the proof that pointer arithmetic uses `vla_size` and not a stale compile-time size: `for (int i = 0; i < sizeof(x)/4; i++) p[i]=i` writes through a `int *`-decayed VLA, and the subsequent `x[5][2]` reads back the value at the right scaled offset.\n\n**Where we are.** VLAs are functional. `TY_VLA` is the new type kind; `Type` carries `vla_len` (the parsed expression) and `vla_size` (a hidden local for the byte count). Declarations rewrite to `compute_vla_size` followed by an `alloca` whose result is stored in a `TY_VLA`-typed local; the local is read as a pointer at every use. `sizeof(VLA)` reads the hidden size local. `sizeof(typename)` of a not-yet-declared VLA computes inline and reads. Pointer arithmetic uses the runtime size. `is_const_expr` is the new structural-recursion predicate that decides whether a bracketed expression makes the type a VLA. `__STDC_NO_VLA__` is no longer defined. The eval-quartet from earlier chapters now has a fifth co-located walker. The canonicalization-at-parse-time count ticks from ten to eleven \u2014 VLA declarations rewrite to a compute-then-alloca pair.\n\n---\n\n## 21.5 \u2014 Linker driver: `-l`, `-s`, ELF size/type, `.a`/`.so`\n\n> `git checkout bc2527944a83c1bc951a429530f39e93dc5235b2` \u2014 *Add -l option*\n>\n> `git checkout c32f0e21e71f43e64a7b98c9d96d4c513d42ba37` \u2014 *Add -s option*\n>\n> `git checkout 8d130ab93f65f7ef79839aba87459e4f9507ba39` \u2014 *Emit size and type for symbols*\n>\n> `git checkout d56dd2f46e4049f017eae0dc99b2d16e78b88bee` \u2014 *Recognize .a and .so files*\n\nFour small commits. They round out the linker-driver story: library linking with `-l`, stripped output with `-s`, ELF symbol-table size and type emission, and recognition of `.a`/`.so` files as linker inputs.\n\n### 21.5.1 \u2014 `-l NAME`: library linking\n\n`-l NAME` tells the linker to find `libNAME.so` (or `libNAME.a` if no shared library exists) in its search path and link against it. chibicc's job here is just to pass the option through:\n\n```c\nif (!strncmp(argv[i], \"-l\", 2)) {\n  strarray_push(&input_paths, argv[i]);\n  continue;\n}\n```\n\nThe `-l` argument is collected on `input_paths` rather than on `ld_args` directly, which means it's processed in the same loop as filename inputs. The compile-vs-link disposition then identifies it by prefix and pushes it to the linker arguments unchanged:\n\n```c\nif (!strncmp(input, \"-l\", 2)) {\n  strarray_push(&ld_args, input);\n  continue;\n}\n```\n\nThe reason for routing through `input_paths` rather than directly to `ld_args` is order: the linker resolves symbols left-to-right, and a `-l` argument has to be at the right position relative to object files for unresolved symbols to find the library. By putting `-l foo` and `bar.o` on the same list, chibicc preserves the user's command-line ordering through the loop and into the final link command.\n\nSearch-path resolution is the linker's job, not chibicc's. chibicc already passes `-L/usr/lib` and `-L/lib` to the linker (added in earlier chapters), and that's enough for `-lfoo` to resolve to `/usr/lib/libfoo.so` (or wherever).\n\n### 21.5.2 \u2014 `-s`: strip the binary\n\n`-s` is a linker flag that tells the linker to produce an output without symbol-table or debug information. chibicc adds a pass-through:\n\n```c\nif (!strcmp(argv[i], \"-s\")) {\n  strarray_push(&ld_extra_args, \"-s\");\n  continue;\n}\n```\n\nThe new `ld_extra_args` `StringArray` is a separate channel for \"linker-only options that aren't input paths.\" The link command is built by `run_linker`, which now appends `ld_extra_args` between the `-L` paths and the input list:\n\n```c\nstrarray_push(&arr, \"-L/usr/lib\");\nstrarray_push(&arr, \"-L/lib\");\n\nfor (int i = 0; i < ld_extra_args.len; i++)\n  strarray_push(&arr, ld_extra_args.data[i]);\n\nfor (int i = 0; i < inputs->len; i++)\n  strarray_push(&arr, inputs->data[i]);\n```\n\nSplitting \"input paths and `-l` flags\" from \"other linker options\" keeps the order-sensitive part separate from the order-insensitive part. `-s` doesn't care where it sits; library inputs do.\n\n### 21.5.3 \u2014 `.type` and `.size` directives\n\nThrough Chapter 20, chibicc emitted symbol *labels* but not the metadata that the ELF format and downstream tools want for stripping, debugging, and dynamic linking \u2014 specifically the symbol's *type* (function vs object) and *size* (number of bytes occupied). The fix is a handful of `.type` and `.size` directives in `emit_data` and `emit_text`:\n\n```c\n// emit_data, in the .data/.tdata branch:\nprintln(\"  .type %s, @object\", var->name);\nprintln(\"  .size %s, %d\", var->name, var->ty->size);\nprintln(\"  .align %d\", align);\n\n// emit_text:\nprintln(\"  .text\");\nprintln(\"  .type %s, @function\", fn->name);\nprintln(\"%s:\", fn->name);\n```\n\n`.type name, @object` marks `name` as a data symbol; `@function` marks it as code. `.size` records the byte count. The emission also moves the `.align` directive after the type/size declarations (the previous order had `.align` first, which is fine but inconsistent with the standard ELF assembly layout).\n\nThe `.type` for the function gets emitted in `emit_text`. The `.size` for the function does *not* \u2014 chibicc doesn't track function size as a parse artifact, and computing it would mean deferring emission until after the function body or running an assembler-level fixup pass. Tools that need function size compute it from labels (the `name`-to-next-label distance) instead. That works in practice but means `.size` is missing for code symbols. Real toolchains emit `.size name, .-name` (the dot is the current location counter, so the difference is the function's byte count); chibicc could do the same if needed but doesn't here.\n\nThe motivation for adding `.type` and `.size` at all is dynamic linking: when chibicc-compiled object files are linked into shared libraries (`.so`), the dynamic linker needs the symbol type to handle relocations correctly. Static linking is mostly forgiving; dynamic linking is not.\n\n### 21.5.4 \u2014 `.a` and `.so` recognition\n\n`get_file_type` learns two new extensions:\n\n```c\ntypedef enum {\n  FILE_NONE, FILE_C, FILE_ASM, FILE_OBJ, FILE_AR, FILE_DSO,\n} FileType;\n\nstatic FileType get_file_type(char *filename) {\n  if (opt_x != FILE_NONE)\n    return opt_x;\n\n  if (endswith(filename, \".a\"))\n    return FILE_AR;\n  if (endswith(filename, \".so\"))\n    return FILE_DSO;\n  if (endswith(filename, \".o\"))\n    return FILE_OBJ;\n  ...\n}\n```\n\n`.a` is a static archive (the unix `ar` format \u2014 a bundle of `.o` files); `.so` is a shared object. Both are passed to the linker unchanged:\n\n```c\nif (type == FILE_OBJ || type == FILE_AR || type == FILE_DSO) {\n  strarray_push(&ld_args, input);\n  continue;\n}\n```\n\nThe recognition is purely by filename suffix \u2014 chibicc doesn't sniff file magic. That works because `ar` archives are conventionally `.a` and shared libraries are conventionally `.so`. Real-world tools sometimes accept `.so.1.2.3` (versioned shared libraries) and `.a.bundle` (rare but seen); chibicc's strict suffix check will miss those. They're rare enough that the simple approach is fine.\n\nThere's also a small refactor in this commit: the `.o` check used to short-circuit `opt_x` (an `.o` file is an object regardless of `-x`), but the order is now `opt_x` first, then suffix. That matches gcc \u2014 `-x assembler foo.o` would treat `foo.o` as an assembler input. In practice nobody does that, and the existing `.o` case still works because most `.o` filenames don't co-exist with `-x` flags.\n\nThe test driver picks up two shape examples for archives and shared objects:\n\n```bash\necho 'void foo() {}' | $chibicc -c -xc -o $tmp/foo.o -\necho 'void bar() {}' | $chibicc -c -xc -o $tmp/bar.o -\nar rcs $tmp/foo.a $tmp/foo.o $tmp/bar.o\necho 'void foo(); void bar(); int main() { foo(); bar(); }' > $tmp/main.c\n$chibicc -o $tmp/foo $tmp/main.c $tmp/foo.a\n```\n\nThe `.so` test does the same with system `cc -shared -fPIC` (chibicc itself doesn't emit position-independent code, so the shared object has to be built with the system compiler, but chibicc can *link* against the resulting `.so`).\n\n**Where we are.** Four linker-driver additions. `-l NAME` flows through `input_paths` into the linker command, in command-line order. `-s` flows through a new `ld_extra_args` channel. `.type @object` / `@function` and `.size` directives are emitted for data and function symbols (with `.size` missing on functions). `.a` and `.so` are recognized as linker inputs by suffix. The driver's vocabulary now covers everything most projects need at the chibicc-as-cc1 level \u2014 the remaining gaps are debugging information and position-independent code.\n\n---\n\n## 21.6 \u2014 `long double`, case ranges, array range designators, labels-as-values\n\n> `git checkout e0bf168041ef60687b5d4454a93fc78c4f3acc48` \u2014 *Add long double*\n>\n> `git checkout d90c73b6058af4b22a4edd610713f75b2478e356` \u2014 *[GNU] Support case ranges*\n>\n> `git checkout 3d5550e29a92708613c3a351c0857aea90e147a5` \u2014 *[GNU] Support array range designator*\n>\n> `git checkout 4f165ec60baa74f244d0a7c9b64c4bb3cbb76173` \u2014 *[GNU] Support labels-as-values*\n\nThe chapter's last four commits are a mixed bag: `long double` finally becomes its own type rather than an alias for `double`, and three GNU bracket-range features land \u2014 `case 1 ... 5:` in switches, `[3 ... 7] = x` in initializers (which closes the \u00a719.7 errata candidate), and `&&label`/`goto *expr` for computed gotos.\n\n### 21.6.1 \u2014 `long double`: real extended precision\n\nThrough Chapter 20, `long double` parsed as a synonym for `double`:\n\n```c\ncase DOUBLE:\ncase LONG + DOUBLE:\n  ty = ty_double;\n  break;\n```\n\nThat was an open errata candidate flagged in earlier chapters. This commit closes it:\n\n```c\ncase DOUBLE:\n  ty = ty_double;\n  break;\ncase LONG + DOUBLE:\n  ty = ty_ldouble;\n  break;\n```\n\nA new `TY_LDOUBLE` enters `TypeKind`. A `ty_ldouble` global type is added with size 16 and alignment 16 (the SysV AMD64 layout \u2014 80-bit x87 extended precision padded to 16 bytes for alignment):\n\n```c\nType *ty_ldouble = &(Type){TY_LDOUBLE, 16, 16};\n```\n\n`is_flonum` is widened:\n\n```c\nbool is_flonum(Type *ty) {\n  return ty->kind == TY_FLOAT || ty->kind == TY_DOUBLE ||\n         ty->kind == TY_LDOUBLE;\n}\n```\n\n`is_compatible` accepts cross-floating-point compatibility (`is_compatible(double, long double)` returns true at the matching arm \u2014 though by the time we reach this arm both types must have the same `kind`, so this just pattern-matches).\n\n`get_common_type` learns the new ladder rule:\n\n```c\nif (ty1->kind == TY_LDOUBLE || ty2->kind == TY_LDOUBLE)\n  return ty_ldouble;\nif (ty1->kind == TY_DOUBLE || ty2->kind == TY_DOUBLE)\n  return ty_double;\nif (ty1->kind == TY_FLOAT || ty2->kind == TY_FLOAT)\n  return ty_float;\n```\n\nThe standard's promotion order: any operand promoted to long double makes the result long double, etc.\n\nThe token side promotes the literal carrier to `long double`:\n\n```c\nstruct Token {\n  ...\n  long double fval;\n  ...\n};\nstruct Node {\n  ...\n  long double fval;\n  ...\n};\n```\n\nand the literal-conversion code reads `strtold` instead of `strtod`, classifying an `L`-suffixed literal as `ty_ldouble`:\n\n```c\nlong double val = strtold(tok->loc, &end);\n\nType *ty;\nif (*end == 'f' || *end == 'F') { ... }\nelse if (*end == 'l' || *end == 'L') {\n  ty = ty_ldouble;\n  end++;\n}\n```\n\nThat's the type system. The codegen side is much larger.\n\n### 21.6.2 \u2014 x87 codegen and the F80 cast row\n\nx86-64 SysV passes `long double` differently from `double`. `double` lives in xmm registers and uses SSE instructions; `long double` (x87 80-bit extended) lives on the *x87 floating-point stack* and uses the older x87 instructions (`fld`, `fst`, `faddp`, `fsubrp`, `fmulp`, `fdivrp`, `fcomip`, etc.). The two coprocessors are separate; chibicc has to emit different instruction families depending on type.\n\nLoads and stores:\n\n```c\ncase TY_LDOUBLE:\n  println(\"  fldt (%%rax)\");   // load a long double from memory onto the x87 stack\n  return;\n...\ncase TY_LDOUBLE:\n  println(\"  fstpt (%%rdi)\");  // store and pop the top of the x87 stack to memory\n  return;\n```\n\nZero comparison (for boolean coercion):\n\n```c\ncase TY_LDOUBLE:\n  println(\"  fldz\");\n  println(\"  fucomip\");\n  println(\"  fstp %%st(0)\");\n  return;\n```\n\n`fldz` pushes zero onto the x87 stack; `fucomip` compares and pops one operand; `fstp` discards the leftover.\n\nNegation:\n\n```c\ncase TY_LDOUBLE:\n  println(\"  fchs\");   // flip sign on top of x87 stack\n  return;\n```\n\n### 21.6.3 \u2014 Casts: the F80 row of the cast table\n\nThe cast table grows from 10\u00d710 to 11\u00d711. There's a new `F80` slot, and every existing row picks up an extra column (cast-to-F80) while a brand-new F80 row provides the cast-from-F80 column entries. The new conversions use the x87 instructions `fildl`/`fildll` (load integer onto x87 stack) and `fistp` (store integer with control-word adjustment for truncation):\n\n```c\nstatic char i32f80[] = \"mov %eax, -4(%rsp); fildl -4(%rsp)\";\nstatic char u32f80[] = \"mov %eax, %eax; mov %rax, -8(%rsp); fildll -8(%rsp)\";\nstatic char i64f80[] = \"movq %rax, -8(%rsp); fildll -8(%rsp)\";\nstatic char u64f80[] =\n  \"mov %rax, -8(%rsp); fildq -8(%rsp); test %rax, %rax; jns 1f;\"\n  \"mov $1602224128, %eax; mov %eax, -4(%rsp); fadds -4(%rsp); 1:\";\n```\n\nThe unsigned-64-to-F80 case is the trickiest: there's no `fild` for unsigned 64-bit integers, so the code does a signed `fildq` and then conditionally adds 2^64 (encoded as the float `1.8446744e19`, which has the bit-pattern `0x5F800000`, decimal `1602224128`) if the high bit was set. That's the same trick gcc uses for the same conversion.\n\nThe F80-to-integer conversions use the FPU control-word twiddle for truncation:\n\n```c\n#define FROM_F80_1                                           \\\n  \"fnstcw -10(%rsp); movzwl -10(%rsp), %eax; or $12, %ah; \" \\\n  \"mov %ax, -12(%rsp); fldcw -12(%rsp); \"\n\n#define FROM_F80_2 \" -24(%rsp); fldcw -10(%rsp); \"\n\nstatic char f80i32[] = FROM_F80_1 \"fistpl\" FROM_F80_2 \"mov -24(%rsp), %eax\";\n```\n\n`fnstcw` saves the FPU control word; `or $12, %ah` sets bits that select round-toward-zero (truncation); `fldcw` loads the new control word; `fistpl` stores-and-pops the integer with the truncation rounding; the second `fldcw` restores the original control word. The two-macro approach is just a way to make the cast-string literals fit on a line each.\n\nCross-floating-point F80 conversions:\n\n```c\nstatic char f32f80[] = \"movss %xmm0, -4(%rsp); flds -4(%rsp)\";\nstatic char f64f80[] = \"movsd %xmm0, -8(%rsp); fldl -8(%rsp)\";\nstatic char f80f32[] = \"fstps -8(%rsp); movss -8(%rsp), %xmm0\";\nstatic char f80f64[] = \"fstpl -8(%rsp); movsd -8(%rsp), %xmm0\";\n```\n\nEach crosses the SSE/x87 boundary by spilling through memory.\n\n### 21.6.4 \u2014 Binary ops on `long double`\n\nThe `gen_expr` binary-op section gains a third branch (after the existing `is_flonum` branch is split into TY_FLOAT/TY_DOUBLE):\n\n```c\nswitch (node->lhs->ty->kind) {\ncase TY_FLOAT:\ncase TY_DOUBLE: {\n  // existing SSE codegen\n  ...\n}\ncase TY_LDOUBLE: {\n  gen_expr(node->lhs);\n  gen_expr(node->rhs);\n\n  switch (node->kind) {\n  case ND_ADD: println(\"  faddp\"); return;\n  case ND_SUB: println(\"  fsubrp\"); return;\n  case ND_MUL: println(\"  fmulp\"); return;\n  case ND_DIV: println(\"  fdivrp\"); return;\n  case ND_EQ: case ND_NE: case ND_LT: case ND_LE:\n    println(\"  fcomip\");\n    println(\"  fstp %%st(0)\");\n    if (node->kind == ND_EQ) println(\"  sete %%al\");\n    else if (node->kind == ND_NE) println(\"  setne %%al\");\n    else if (node->kind == ND_LT) println(\"  seta %%al\");\n    else println(\"  setae %%al\");\n    println(\"  movzb %%al, %%rax\");\n    return;\n  }\n  ...\n}\n}\n```\n\nEach of `gen_expr(lhs)` and `gen_expr(rhs)` pushes a long double onto the x87 stack; the `faddp` family operates on the top two and pops one. Note the operand-order quirk in subtraction and division: `fsubrp` and `fdivrp` reverse the operands relative to what the obvious code would do, because the x87 stack has the second-pushed operand on top and the first-pushed below it. The instruction names with `r` swap the order; the `p` pops one register after.\n\nThe comparison sequence uses `fcomip` (compare-pop, set flags) followed by an `fstp %st(0)` to discard the second operand (the comparison only consumes one). The flags-test instructions `sete`/`setne`/`seta`/`setae` are the same as the SSE comparison path, except `seta`/`setae` (above/above-or-equal) are used instead of `setb`/`setbe`. That matches the x87 flag semantics \u2014 `fcomip` produces the C flags in a way that maps \"less than\" to \"above\" in the unsigned sense.\n\n### 21.6.5 \u2014 `long double` as a function argument\n\nThe SysV AMD64 ABI passes `long double` *on the stack* \u2014 it does not assign x87 registers to it the way it assigns xmm registers to `double`. `push_args` recognizes this:\n\n```c\ncase TY_LDOUBLE:\n  arg->pass_by_stack = true;\n  stack += 2;\n  break;\n```\n\n`stack += 2` because each long double consumes 16 bytes (two 8-byte slots).\n\n`push_args2` emits the actual stack push:\n\n```c\ncase TY_LDOUBLE:\n  println(\"  sub $16, %%rsp\");\n  println(\"  fstpt (%%rsp)\");\n  depth += 2;\n  break;\n```\n\nThe argument is popped from the top of the x87 stack and stored to the new stack space. The `depth += 2` updates chibicc's running stack-depth tracker so subsequent operations stay 16-byte-aligned for SSE pushes.\n\n`assign_lvar_offsets` adds an empty case for `TY_LDOUBLE` in the same switch where the integer and double branches handle register passing \u2014 long doubles aren't register-passed, so the case is a no-op (but the switch must explicitly mention `TY_LDOUBLE` to avoid falling into the `gp++` arm):\n\n```c\ncase TY_LDOUBLE:\n  break;\n```\n\n`has_flonum` (for struct-passing classification) gets a small narrowing \u2014 it now treats long double as not-quite-flonum for the struct-arm-classification purpose, because long double doesn't go through the same eight-byte SSE-classification machinery:\n\n```c\nreturn offset < lo || hi <= offset || ty->kind == TY_FLOAT || ty->kind == TY_DOUBLE;\n```\n\nNote the explicit kind check rather than `is_flonum(ty)`. The two predicates have diverged; `is_flonum` says \"yes for any of the three FP types,\" `has_flonum` says \"yes only for FLOAT and DOUBLE because LDOUBLE isn't passed in xmm.\" That's the kind of subtle ABI-vs-language-type split the SysV ABI forces on every compiler that supports long double.\n\nThe psABI conformance count grows by one for the long-double calling convention.\n\n### 21.6.6 \u2014 `long double` literal emission\n\nFloat and double literals encode as bit-cast immediates in earlier chapters. Long double extends the pattern with two halves \u2014 the low 8 bytes and the high 8 bytes \u2014 written through the stack:\n\n```c\ncase TY_LDOUBLE: {\n  union { long double f80; uint64_t u64[2]; } u;\n  memset(&u, 0, sizeof(u));\n  u.f80 = node->fval;\n  println(\"  mov $%lu, %%rax  # long double %Lf\", u.u64[0], node->fval);\n  println(\"  mov %%rax, -16(%%rsp)\");\n  println(\"  mov $%lu, %%rax\", u.u64[1]);\n  println(\"  mov %%rax, -8(%%rsp)\");\n  println(\"  fldt -16(%%rsp)\");\n  return;\n}\n```\n\nThe `memset` zeroes any padding bits in the union (the 80-bit value occupies only 10 of the 16 bytes; the high 6 bytes need to be deterministic). The two halves are stored to the redzone area (`-16(%rsp)` to `-1(%rsp)`, which is below the current stack pointer but reserved for leaf-function temporaries by the SysV ABI), and then `fldt` pulls the 80-bit value back onto the x87 stack.\n\nThis pattern repeats in cast-table entries and in argument-passing \u2014 the redzone is the universal scratchpad for x87 \u2194 memory \u2194 SSE conversions.\n\n### 21.6.7 \u2014 Tests and the closing of an errata\n\n`test/sizeof.c` flips a long-standing assertion:\n\n```diff\n-  ASSERT(8, sizeof(long double));\n+  ASSERT(16, sizeof(long double));\n```\n\n`test/literal.c` matches:\n\n```diff\n-  ASSERT(8, sizeof(5.l));\n-  ASSERT(8, sizeof(2.0L));\n+  ASSERT(16, sizeof(5.l));\n+  ASSERT(16, sizeof(2.0L));\n```\n\n`test/function.c` and `test/arith.c` add long-double-specific function-call and arithmetic tests. The most colorful one verifies `long double`'s precision against `printf`:\n\n```c\nASSERT(0, ({ char buf[100]; sprintf(buf, \"%Lf\", (long double)12.3);\n             strncmp(buf, \"12.3\", 4); }));\n```\n\n(The libc-level `%Lf` format depends on chibicc-compiled code passing `long double` correctly through `vsprintf`; the test passes, which proves the calling convention works end-to-end.)\n\nThe \"long double is double\" errata is now closed.\n\n### 21.6.8 \u2014 Case ranges: `case begin ... end:`\n\nGNU C's case ranges generalize a switch arm to match any value in `[begin, end]`. The implementation has to remember both bounds and emit a range check rather than a single equality.\n\n`Node` gains two fields:\n\n```c\n// Switch\nNode *case_next;\nNode *default_case;\n\n// Case\nlong begin;\nlong end;\n```\n\nThe case parser reads the optional `... end` clause:\n\n```c\nint begin = const_expr(&tok, tok->next);\nint end;\n\nif (equal(tok, \"...\")) {\n  // [GNU] Case ranges, e.g. \"case 1 ... 5:\"\n  end = const_expr(&tok, tok->next);\n  if (end < begin)\n    error_tok(tok, \"empty case range specified\");\n} else {\n  end = begin;\n}\n```\n\nA single-value case is the degenerate range `[begin, begin]`. The codegen in `gen_stmt` then dispatches:\n\n```c\nchar *ax = (node->cond->ty->size == 8) ? \"%rax\" : \"%eax\";\nchar *di = (node->cond->ty->size == 8) ? \"%rdi\" : \"%edi\";\n\nif (n->begin == n->end) {\n  println(\"  cmp $%ld, %s\", n->begin, ax);\n  println(\"  je %s\", n->label);\n  continue;\n}\n\n// [GNU] Case ranges\nprintln(\"  mov %s, %s\", ax, di);\nprintln(\"  sub $%ld, %s\", n->begin, di);\nprintln(\"  cmp $%ld, %s\", n->end - n->begin, di);\nprintln(\"  jbe %s\", n->label);\n```\n\nThe single-value path is unchanged. The range path uses the unsigned-comparison trick: subtract `begin` from the switch value, compare against `end - begin`, jump if below-or-equal (unsigned). That handles negative ranges and ranges that span zero correctly because the wrap-around makes any out-of-range value compare *above* `end - begin` in unsigned terms.\n\nNotably, this generates code: a `case 1 ... 1000000` would emit a range check, not a million jump-table entries. Chibicc never builds jump tables anyway \u2014 every case is a sequential `cmp/je` chain \u2014 so the range form doesn't blow up codegen the way it might for a real compiler trying to balance jump-table density.\n\n### 21.6.9 \u2014 Array range designators: `[3 ... 7] = x`\n\nThis commit closes the \u00a719.7 errata candidate. Through Chapter 19, the parser recognized `[3 ... 7]` syntactically (it accepted the tokens) but only honored `[3]` semantically \u2014 the elaboration loop walked just the first index of the range. Now the elaboration honors the full range.\n\n`array_designator` is rewritten to take output parameters for both ends:\n\n```c\nstatic void array_designator(Token **rest, Token *tok, Type *ty, int *begin, int *end) {\n  *begin = const_expr(&tok, tok->next);\n  if (*begin >= ty->array_len)\n    error_tok(tok, \"array designator index exceeds array bounds\");\n\n  if (equal(tok, \"...\")) {\n    *end = const_expr(&tok, tok->next);\n    if (*end >= ty->array_len)\n      error_tok(tok, \"array designator index exceeds array bounds\");\n    if (*end < *begin)\n      error_tok(tok, \"array designator range [%d, %d] is empty\", *begin, *end);\n  } else {\n    *end = *begin;\n  }\n\n  *rest = skip(tok, \"]\");\n}\n```\n\nThe two callers \u2014 `designation` (for the right-hand side after the bracket) and `array_initializer1` (the main loop) \u2014 both now iterate from `begin` to `end`, applying the same designation expression to each child:\n\n```c\nint begin, end;\narray_designator(&tok, tok, init->ty, &begin, &end);\n\nToken *tok2;\nfor (int i = begin; i <= end; i++)\n  designation(&tok2, tok, init->children[i]);\narray_initializer2(rest, tok2, init, begin + 1);\n```\n\nEach iteration re-tokenizes the same source range \u2014 `tok` is captured before the loop and the loop reuses it \u2014 so the same expression seeds every index in the range. After the loop, `tok2` (the position after the last designation parse) is the resume point.\n\nTest:\n\n```c\nASSERT(16, ({ char x[]={[2 ... 10]='a', [7]='b', [15 ... 15]='c', [3 ... 5]='d'};\n              sizeof(x); }));\nASSERT(0, ({ char x[]={[2 ... 10]='a', [7]='b', [15 ... 15]='c', [3 ... 5]='d'};\n             memcmp(x, \"\\0\\0adddabaaa\\0\\0\\0\\0c\", 16); }));\n```\n\nThe expected layout (`\"\\0\\0adddabaaa\\0\\0\\0\\0c\"`) shows the overlapping ranges resolving last-write-wins: the `[3 ... 5]='d'` overwrites the `'a'`s set by `[2 ... 10]`, and the standalone `[7]='b'` overwrites that range too. Indices 11\u201314 are the untouched zeroes between the highest-set range and the top index 15.\n\nThe \u00a719.7 errata candidate is now closed.\n\n### 21.6.10 \u2014 Labels-as-values: `&&label` and `goto *expr`\n\nGNU C's labels-as-values feature lets you take the address of a label (`&&label`) and `goto` indirectly through an address (`goto *expr`). Used together they enable computed gotos \u2014 a common technique in interpreter inner loops, threaded code, and finite state machines.\n\nThe token side is already there: `&&` is in the punctuator list (it's the logical-AND operator). The parser distinguishes by context \u2014 in an expression after a unary operator slot, `&&label` means \"address of label.\"\n\n`Node` gains two new kinds:\n\n```c\nND_GOTO_EXPR, // \"goto\" labels-as-values\n...\nND_LABEL_VAL, // [GNU] Labels-as-values\n```\n\nThe parser's `unary` arm picks up the `&&` form:\n\n```c\n// [GNU] labels-as-values\nif (equal(tok, \"&&\")) {\n  Node *node = new_node(ND_LABEL_VAL, tok);\n  node->label = get_ident(tok->next);\n  node->goto_next = gotos;\n  gotos = node;\n  *rest = tok->next->next;\n  return node;\n}\n```\n\nThe label name is captured immediately, but the resolution to a unique generated assembly label has to wait until the function body has been parsed (because labels can be defined after their first use \u2014 a `&&forward_label` may reference a label defined later in the function). The `gotos` chain that already handled forward `goto` references is reused: `node->goto_next = gotos; gotos = node;` threads the new node onto the same list, and `resolve_goto_labels` (renamed in spirit only \u2014 the code is unchanged) walks both `ND_GOTO` and `ND_LABEL_VAL` nodes to fill in `unique_label`.\n\nThe `goto *expr` form lives in `stmt`:\n\n```c\nif (equal(tok, \"goto\")) {\n  if (equal(tok->next, \"*\")) {\n    // [GNU] `goto *ptr` jumps to the address specified by `ptr`.\n    Node *node = new_node(ND_GOTO_EXPR, tok);\n    node->lhs = expr(&tok, tok->next->next);\n    *rest = skip(tok, \";\");\n    return node;\n  }\n\n  Node *node = new_node(ND_GOTO, tok);\n  ...\n}\n```\n\nThe `*` after `goto` is a syntactic giveaway. The expression is a normal expression; codegen treats its result as a code pointer.\n\nCodegen for both:\n\n```c\ncase ND_LABEL_VAL:\n  println(\"  lea %s(%%rip), %%rax\", node->unique_label);\n  return;\n...\ncase ND_GOTO_EXPR:\n  gen_expr(node->lhs);\n  println(\"  jmp *%%rax\");\n  return;\n```\n\n`lea label(%rip), %rax` produces the address of the label using rip-relative addressing (the standard PIC-friendly form). `jmp *%rax` is the indirect jump.\n\n`add_type` gives the label-address node a void-pointer type:\n\n```c\ncase ND_LABEL_VAL:\n  node->ty = pointer_to(ty_void);\n  return;\n```\n\nThat matches the gcc convention: `&&label` has type `void *`.\n\nNote what this commit *doesn't* do: it doesn't make label addresses usable as compile-time constants for global initializers. A later commit (in Chapter 22) handles that. For now, `&&label` is an expression, evaluable inside a function body but not in a static initializer.\n\nThe label namespace is unchanged. Labels-as-values uses the same label table that ordinary `goto`s use; `&&label` and `goto label` resolve to the same internal label.\n\nTest:\n\n```c\nASSERT(3, ({ void *p = &&v11; int i=0; goto *p;\n             v11:i++; v12:i++; v13:i++; i; }));\n```\n\n`&&v11` produces a code pointer; `goto *p` jumps to it; the labeled statements increment `i`. The fall-through nature of statement-level labels means the jump to `v11` runs all three increments; jumping to `v33` runs just one.\n\n**Where we are.** `long double` is real extended-precision \u2014 16-byte size, 16-byte alignment, x87-stack codegen for arithmetic, on-stack passing for function arguments, the cast table grows from 10\u00d710 to 11\u00d711 with a full new F80 row and column. The \"long double is double\" errata is closed. Case ranges (`case 1 ... 5:`) generate inline range checks using the unsigned-subtract-and-compare trick; chibicc's no-jump-table approach scales naturally to large ranges. Array range designators (`[3 ... 7] = x`) are now honored in elaboration, not just parsed; the \u00a719.7 errata is closed. Labels-as-values gives `&&label` (a `void *` value) and `goto *expr` (indirect jump). Resolution piggybacks on the existing forward-`goto` resolution pass.\n\nThe psABI conformance count ticks up by one for the long-double calling convention. The cast table grows from 10\u00d710 to 11\u00d711.\n\n---\n\n## Recap\n\nSeventeen commits. The chapter adds three substantial pieces of machinery (thread-local storage, `alloca`, VLAs) plus four small linker-driver additions and four bracket-range-and-precision additions:\n\n- `Obj` gains `is_tls` (for thread-local globals) and `alloca_bottom` (a hidden local for the per-function `alloca` slide).\n- `VarAttr` gains `is_tls`.\n- `Type` gains `vla_len` (the parsed dimension expression) and `vla_size` (a hidden local for the runtime byte count). New kind `TY_VLA`.\n- `Node` gains `ND_VLA_PTR` (slot-address for a VLA-typed local), `ND_GOTO_EXPR` and `ND_LABEL_VAL` (labels-as-values), and the `Node->begin`/`end` fields (for case ranges).\n- A new `TY_LDOUBLE` kind, a `ty_ldouble` global, a fifth eval-quartet member `is_const_expr`, a synthesized `alloca` builtin in `globals`, two new TLS sections (`.tdata`/`.tbss`), and a 11\u00d711 cast table replacing the 10\u00d710.\n- `parse()` does no new top-level passes (it already runs `mark_live` and `scan_globals`); the new work is per-declaration (`compute_vla_size`) and per-call (the `alloca`-recognition).\n- The driver gains `-include`, `-x`, `-l`, `-s`, `.a`/`.so` recognition, and the `-E` implies `-xc` rule.\n- `emit_data` and `emit_text` emit `.type` and `.size` directives.\n- The keyword list gains `_Thread_local` and `__thread`.\n\nThe chapter's seventeen-row summary, in `main` order:\n\n| # | Hash | Subject | Section |\n|---|---|---|---|\n| 267 | `b377284` | Add thread-local variable | \u00a721.1 |\n| 268 | `8f5ff07` | Add `-include` option | \u00a721.2 |\n| 269 | `ee0a951` | Add `-x` option | \u00a721.2 |\n| 270 | `4064871` | Make `-E` imply `-xc` | \u00a721.2 |\n| 271 | `77275c5` | Add `alloca()` | \u00a721.3 |\n| 272 | `e8667af` | Add `sizeof()` for VLA | \u00a721.4 |\n| 273 | `07f9010` | Add pointer arithmetic for VLA | \u00a721.4 |\n| 274 | `2fa8f48` | Support `sizeof(typename)` where typename is a VLA | \u00a721.4 |\n| 275 | `b0109a3` | Do not define `__STDC_NO_VLA__` | \u00a721.4 |\n| 276 | `bc25279` | Add `-l` option | \u00a721.5 |\n| 277 | `c32f0e2` | Add `-s` option | \u00a721.5 |\n| 278 | `8d130ab` | Emit size and type for symbols | \u00a721.5 |\n| 279 | `d56dd2f` | Recognize `.a` and `.so` files | \u00a721.5 |\n| 280 | `e0bf168` | Add `long double` | \u00a721.6 |\n| 281 | `d90c73b` | `[GNU]` Support case ranges | \u00a721.6 |\n| 282 | `3d5550e` | `[GNU]` Support array range designator | \u00a721.6 |\n| 283 | `4f165ec` | `[GNU]` Support labels-as-values | \u00a721.6 |\n\nErrata candidates closed this chapter: two of the long-standing ones. `long double` is no longer aliased to `double` (the `e0bf168` commit makes it real 80-bit extended precision in the \u00a721.6 walk). Array range designators are honored in elaboration (`3d5550e` in \u00a721.6 closes the \u00a719.7 errata). The remaining errata candidates are unchanged: Ch 17's three (`#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific), Ch 19's two remaining (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block), and Ch 20's one (`is_compatible` array arm bug).\n\nErrata candidates added this chapter:\n\n- The `.size` directive is missing for function symbols (in \u00a721.5.3, commit `8d130ab`). Tools that read function size from the symbol table will see zero for chibicc-emitted functions; tools that compute size from labels will be fine.\n- File-magic-vs-suffix recognition for `.a` and `.so` is suffix-only (in \u00a721.5.4, commit `d56dd2f`). Versioned shared libraries (`libfoo.so.1.2.3`) won't be recognized.\n\nThe canonicalization-at-parse-time count ticks from ten to eleven with the VLA declaration rewrite (`int x[n]` becomes a `compute_vla_size; x = alloca(...)` pair). The pre-factor-before-feature count is unchanged at nine. The psABI conformance count grows by two: one for thread-local TLS access patterns, one for the long-double calling convention. New count: eighteen.\n\nThe standing notes for the next session: `Obj` is now even more substantial; `Type` has gained `vla_len`/`vla_size` and `TY_VLA`; the cast table is 11\u00d711; the keyword list is around thirty-two entries. Two new node kinds for labels-as-values; one new for VLA pointers. The eval-quartet has a fifth member.\n\nThrough Chapter 21 chibicc handles thread-local storage, `alloca`, VLAs, `long double`, GNU case ranges and array range designators and labels-as-values, plus four more driver options. What remains: dependency-file emission for build systems, performance work (the parser and tokenizer have grown to the point where some operations are visibly slow), the linker-driver edge cases that real distributions exercise, and a long tail of compile-time-constant features. Those are the next twenty-three commits, in Chapter 22.\n"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "wc -w /Users/cell/claude/chibicc-book/chapters/21-thread-local-alloca-vlas.md",
  "description": "Count words in chapter"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 17 Ch 21 commits",
      "activeForm": "Reading all 17 Ch 21 commits",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files",
      "activeForm": "Reading destination state of key files",
      "status": "completed"
    },
    {
      "content": "Draft chapter 21",
      "activeForm": "Drafting chapter 21",
      "status": "completed"
    },
    {
      "content": "Write session 022 README",
      "activeForm": "Writing session 022 README",
      "status": "in_progress"
    },
    {
      "content": "Write HANDOFF for session 023",
      "activeForm": "Writing HANDOFF for session 023",
      "status": "pending"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cat /Users/cell/claude/chibicc-book/research/commits/chapter-mapping.md | grep -A 30 \"Chapter 22\"",
  "description": "find Ch 22 mapping"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/research/sources/chibicc && git log --oneline main 2>/dev/null | tac | nl | sed -n '284,306p'",
  "description": "Get commits 284-306 for Ch 22"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/022-chapter-21-draft/README.md",
  "content": "# Session 022 \u2014 Chapter 21 draft\n\n**Date:** 2026-05-10 (continuation of the autonomous-drafting run that has produced sessions 002\u2013021).\n**Phase:** 2 (bulk drafting).\n\n## Arrival state\n\nSession 021 delivered Ch 20 (GCC extensions worth supporting, twenty-two commits, ~9,994 words). User direction is still autonomous \u2014 no chapter-by-chapter review. Ch 21 covers commits 267\u2013283: seventeen commits \u2014 thread-local variables, three driver options (`-include`, `-x`, `-E` implies `-xc`), `alloca`, the four-commit VLA arc, four linker-driver additions (`-l`, `-s`, ELF size/type, `.a`/`.so`), and four \u00a721.6 mixed features (`long double`, case ranges, array range designators, labels-as-values).\n\n## What was done\n\n### Drafting decisions\n\n- **Length:** ~10,654 words. Right in the handoff's 10,000\u201312,000-word target range. The chapter's commits are mixed in scale: thread-local (~1,400 words), driver triple (~1,200), `alloca` (~1,500), VLAs (~3,000 \u2014 the largest section), linker-driver quartet (~1,400), and \u00a721.6 quartet (~3,200 \u2014 long double dominates with ~1,800 of those). No padding; everything lands.\n- **Section structure:** 6 sections from 17 commits, exactly as the handoff proposed. \u00a721.1 single-commit, no subsections needed (just a flowing walk through the AMD64 TLS pattern). \u00a721.2 three commits with three named subsections. \u00a721.3 single-commit, no subsections. \u00a721.4 four commits with ten named subsections (the largest section by far \u2014 VLAs are genuinely complex). \u00a721.5 four commits with four named subsections. \u00a721.6 four commits with ten named subsections (long double dominates).\n- **No concept interlude.** The handoff said \"possibly one\" around the VLA arc as a \"dynamically-sized stack allocations\" interlude. Reading the \u00a721.4 prose, the through-line from `compute_vla_size` to `new_alloca` to `ND_VLA_PTR` to the `gen_addr` arms hangs together as a single arc; pulling out a separate interlude would have introduced repetition. The \u00a721.4.1\u2013\u00a721.4.10 subsection chain is enough scaffolding.\n- **\u00a721.4 makes the VLA-vs-alloca lineage explicit.** \u00a721.3's `alloca` builtin is what \u00a721.4 reuses for VLA allocation \u2014 the `new_alloca` helper builds an `ND_FUNCALL` to the same synthesized `Obj`. This is one of the few places where reading-order and writing-order pay off: walking `alloca` first makes the VLA implementation feel inevitable.\n- **\u00a721.6 closes two errata candidates** \u2014 `long double` as `double`, and the \u00a719.7 array range designator. Both are explicitly named in the \u00a721.6 closer and the chapter recap.\n- **\u00a721.6.2\u2013\u00a721.6.3 walks the cast table growth from 10\u00d710 to 11\u00d711 explicitly,** including the F80 row's `FROM_F80_1`/`FROM_F80_2` macros and the unsigned-64-to-F80 sign-test trick. Long double codegen is mostly x87 instruction soup and the section names what every instruction does without overspecifying the FPU stack model.\n- **The psABI conformance count grows by two** (TLS access patterns + long double calling convention). New count: eighteen, up from sixteen.\n- **Canonicalization-at-parse-time count ticks from ten to eleven** with the VLA declaration rewrite (`int x[n]` \u2192 `compute_vla_size; x = alloca(...)`).\n- **Two new errata candidates added in \u00a721.5:** missing `.size` directive for function symbols (8d130ab), and suffix-only `.a`/`.so` recognition that won't catch versioned shared libraries (d56dd2f). Both noted in \u00a721.5 prose and the chapter recap.\n- **One-table recap** at the chapter close, seventeen rows. Same one-table-with-section-column shape as Ch 20.\n\n### Interpretive calls\n\n1. **TLS access model named as initial-exec.** \u00a721.1 calls out that chibicc emits the cheapest TLS model (`%fs:0` + `name@tpoff`), which works for variables in the executable but not for dynamically-loaded shared-library TLS (which would need `__tls_get_addr` and the global-dynamic model). Rui's choice; the prose names it.\n2. **`Makefile`'s `-pthread` link flag** is named in \u00a721.1 as the test-plumbing piece that lets the new `test/tls.c` link against libpthread. This is a test-infrastructure consequence, not a language feature.\n3. **The `-pthread` removal of `__STDC_NO_THREADS__`** is given a paragraph that distinguishes language-level threads (TLS, the underlying compiler feature) from library-level threads (`<threads.h>`, which chibicc still doesn't implement). Honest framing.\n4. **The `-include`/`-x`/`-E xc` rule is treated as one logical group** even though commit dates span ten days and `-E xc` is partly a reaction to test churn introduced by `-x`. The \u00a721.2.3 prose names this \u2014 \"three commits later (about a week of calendar time), Rui revisits the test churn.\"\n5. **\u00a721.3's `alloca` description is candid about safety.** Stack-exhaustion isn't detected; the eight-byte cost per function is named; the function-epilogue dependency that makes the implementation work is named.\n6. **\u00a721.4's VLA arc is walked subsection-by-subsection** with an explicit ordering: type kind first, then dimension parser, then `is_const_expr`, then `compute_vla_size`, then declaration rewrite, then `ND_VLA_PTR`, then `gen_addr`, then pointer arithmetic, then `sizeof(VLA)` and `sizeof(typename)`, then `__STDC_NO_VLA__`. The reader gets the pieces in dependency order rather than commit order (the four commits are interleaved across the subsections).\n7. **\u00a721.4.4 explains the comma-expression chain** built by `compute_vla_size` with a worked example for `int x[m][n]`. The `ND_NULL_EXPR` seed is named.\n8. **\u00a721.4.6 explains why `ND_VLA_PTR` exists** (the assignment target is the slot's address, not the pointer-stored-in-the-slot \u2014 analogous to the `&arr` vs `arr` distinction).\n9. **\u00a721.5.3's `.size` for functions is noted as missing.** Real toolchains emit `.size name, .-name`; chibicc doesn't. The prose names this and routes it to errata.\n10. **\u00a721.5.4's suffix-only file-magic recognition** is named as a real-world limitation. Versioned shared libraries (`libfoo.so.1.2.3`) won't be recognized. Errata candidate.\n11. **\u00a721.6.1 makes \"long double is double\" closure explicit** with a side-by-side diff of the type-kind switch.\n12. **\u00a721.6.4's operand-order quirk in `fsubrp`/`fdivrp`** is named \u2014 the x87 stack puts the second-pushed operand on top, so the `r` (reverse) instructions are needed.\n13. **\u00a721.6.5's `has_flonum` divergence from `is_flonum`** is given a paragraph. The two predicates were unified through Ch 20; long double splits them: `is_flonum` says \"yes for any FP type\"; `has_flonum` says \"yes only for FLOAT and DOUBLE because LDOUBLE doesn't go through the SSE eight-byte classification.\"\n14. **\u00a721.6.6's redzone use** is named as the universal scratchpad for x87\u2194memory\u2194SSE transitions.\n15. **\u00a721.6.8 names the unsigned-comparison trick for case ranges** \u2014 subtract `begin`, compare against `end - begin` unsigned, jump if below-or-equal. The wraparound makes out-of-range values compare above. Standard compiler trick, worth naming.\n16. **\u00a721.6.10 names the `&&label` resolution path** as reusing the existing forward-`goto` machinery \u2014 `gotos` chain, `resolve_goto_labels` walk. No new pass.\n\n### Voice / structure inherited from Ch 1\u201320\n\n- \"we\" for reader-journey, \"Rui\" for design intent.\n- Past tense for what the commit did, present for current behavior.\n- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, all hashes listed at the top.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- One-table recap at the chapter close (with \u00a7-section column).\n- No concept interludes.\n\n### Three careful avoidances\n\n- **Did not invent a \"history of TLS models\" interlude.** The Linux TLS implementation has multiple models (initial-exec, local-exec, general-dynamic, local-dynamic) with different cost/flexibility tradeoffs. The chapter names initial-exec as what chibicc does and points at the existence of the others without walking through them. A history walk would be a detour \u2014 chibicc's choice is the simple one.\n- **Did not invent a \"history of VLAs\" interlude.** VLAs were added in C99 and made optional in C11; gcc had them earlier in proprietary forms. The chapter cites both standardization milestones in passing but doesn't walk the full history. Standard chibicc-book convention.\n- **Did not over-explain the x87 FPU stack.** The \u00a721.6 long-double codegen names the stack-based operand model, the operand-order quirks of `fsubrp`/`fdivrp`, and the FPU control-word manipulation for truncation casts. It doesn't walk the full eight-element x87 stack model or the deprecated 287/387 history. Acceptable \u2014 chibicc emits what it emits, and the prose explains what each instruction does.\n\n### Date-vs-position note\n\nThe seventeen commits scatter across late-August and September 2020. The four VLA commits in particular span August 25 (`2fa8f48`, `b0109a3`) and September 3\u20134 (`07f9010`, `e8667af`), which means in main order the *earlier-written* commits land *later* (positions 274 and 275). Rui's branches landed in the order of dependent functionality rather than in chronological order. The chapter follows main order and notes this in \u00a721.4's opening paragraph but doesn't dwell on it.\n\n## Open questions surfaced for user\n\nNone \u2014 autonomous mode.\n\n## Notes worth carrying forward\n\n- **The TLS access model is initial-exec only.** `mov %fs:0, %rax; add $name@tpoff, %rax`. Dynamic-library TLS (which would call `__tls_get_addr`) is not implemented. Chibicc-compiled object files used in shared libraries with TLS variables will not work correctly with thread-local storage in those libraries. Named in \u00a721.1.\n- **`Obj->is_tls`** is set in `global_variable` from `attr->is_tls`. Tentative-definition handling is suppressed for TLS \u2014 a bare `_Thread_local int x;` goes to `.tbss`, not `.comm`.\n- **`Obj->alloca_bottom`** is a hidden per-function local that records the current bottom of the temporary-evaluation area. Set in the prologue to `%rsp`; updated by `builtin_alloca`. Costs eight bytes per function whether `alloca` is called or not.\n- **`alloca` is a builtin.** A `void *alloca(int)` is synthesized at parse start; codegen recognizes `ND_FUNCALL` to a variable named `alloca` and emits the inline shift sequence.\n- **`builtin_alloca` (in parse.c) holds a file-static reference to the synthesized `Obj`** so the VLA code can build `ND_FUNCALL` nodes without re-looking up. The original `alloca` declaration is registered as a global; `builtin_alloca` is the captured pointer.\n- **`TY_VLA`** is the new type kind. `Type` carries `vla_len` (parsed expression for the dimension) and `vla_size` (a hidden local for the runtime byte count). VLA-typed locals are 8 bytes on the stack \u2014 pointers to alloca'd buffers.\n- **`compute_vla_size`** builds an `ND_COMMA` chain that evaluates dimensions in declaration order and stores byte counts into `vla_size` locals. Multi-dimensional VLAs cascade \u2014 inner row size goes into the inner `vla_size`, then outer total goes into the outer `vla_size`.\n- **`is_const_expr`** is the fifth eval-quartet member. Structural recursion that returns false on anything not statically computable. Used by `array_dimensions` to decide between `array_of` and `vla_of`.\n- **`ND_VLA_PTR`** is the new node kind for \"address of a VLA-typed local's slot\" \u2014 distinct from `ND_VAR` (which loads the pointer stored in the slot). `gen_addr` emits `lea offset(%rbp)`; `add_type` reuses `ND_VAR`'s rule.\n- **`-l NAME`** flows through `input_paths` (not `ld_args`) to preserve command-line ordering relative to filename inputs. The disposition loop identifies it by prefix and pushes to `ld_args` in order.\n- **`-s`** flows through a new `ld_extra_args` channel, distinct from `ld_args` because order doesn't matter for `-s`.\n- **`.type @object`/`@function`** and **`.size`** are now emitted for data and function symbols. `.size` is missing for functions (chibicc doesn't track function size as a parse artifact). Errata candidate.\n- **`.a`, `.so`, `.o`** are recognized by suffix only. Versioned shared libraries (`libfoo.so.1.2.3`) won't be recognized. Errata candidate.\n- **`get_file_type`'s order** changed in `d56dd2f`: `opt_x` is now checked before `.o` (so `-x assembler foo.o` would treat `foo.o` as assembler). Real-world usage doesn't trigger this; flagged for completeness.\n- **`TY_LDOUBLE`** is the new floating-point kind. Size 16, alignment 16. Codegen uses x87 stack (`fld`, `fst`, `faddp`, etc.). Function arguments go on the stack (not in xmm regs). `is_compatible` accepts cross-FP compatibility; `get_common_type` promotes to `long double` if either operand is `long double`.\n- **The cast table is now 11\u00d711** \u2014 F80 row and column added. F80 conversions to integer use the FPU control-word truncation trick (`fnstcw`/`fldcw`); F80 conversions across SSE-FP go through the redzone.\n- **The `Token`/`Node` `fval` widened from `double` to `long double`.** `convert_pp_number` reads `strtold`. The literal-emission code writes the bits as two halves through the redzone.\n- **`is_flonum` and `has_flonum` diverge.** `is_flonum` returns true for FLOAT, DOUBLE, LDOUBLE; `has_flonum` (struct classification) returns true only for FLOAT, DOUBLE.\n- **The keyword list is now around thirty-two entries** with `_Thread_local` and `__thread` added.\n- **Case ranges** generate inline range checks via the unsigned-subtract-and-compare trick. `node->begin` and `node->end` carry the bounds. Single-value cases use `begin == end` as the degenerate case.\n- **Array range designators are honored in elaboration.** `array_designator` returns `(begin, end)`; the two callers (`designation` and `array_initializer1`) loop. \u00a719.7 errata closed.\n- **Labels-as-values** uses `gotos` chain for resolution; `&&label` is `ND_LABEL_VAL` (rip-relative `lea`); `goto *expr` is `ND_GOTO_EXPR` (indirect `jmp`). Compile-time-constant variant (`f0c98e0`) is in Ch 22, not here.\n- **psABI conformance count is at eighteen** (up from sixteen). New: TLS access pattern, long-double calling convention.\n- **Canonicalization-at-parse-time count is at eleven** (up from ten). New: VLA declaration rewrite to `compute_vla_size; x = alloca(...)`.\n- **Pre-factor-before-feature count is at nine** (unchanged).\n- **Errata candidates closed in Ch 21:** \"long double is double\" (closed by `e0bf168`); array range designators not honored (closed by `3d5550e`).\n- **Errata candidates added in Ch 21:** missing `.size` for function symbols (`8d130ab`); suffix-only `.a`/`.so` recognition (`d56dd2f`).\n- **Errata candidates remaining:** Ch 17's three (#error doesn't print message text, opt_S | opt_E typo, default include paths Linux/glibc-specific), Ch 19's two (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block), Ch 20's one (`is_compatible` array arm bug), Ch 21's two new.\n- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean \u2014 unchanged.\n- **Chibicc compiles itself** \u2014 unchanged.\n\n## Exit state\n\n- `chapters/21-thread-local-alloca-vlas.md` drafted, ~10,654 words.\n- Session 022 dir populated with this README and HANDOFF.md.\n- HANDOFF.md primes session 023 (Chapter 22 \u2014 Performance, dependency files, and the linker driver, commits 284\u2013306, 23 commits).\n- CLAUDE.md status note updated to \"Ch 21 drafted\".\n"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 17 Ch 21 commits",
      "activeForm": "Reading all 17 Ch 21 commits",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files",
      "activeForm": "Reading destination state of key files",
      "status": "completed"
    },
    {
      "content": "Draft chapter 21",
      "activeForm": "Drafting chapter 21",
      "status": "completed"
    },
    {
      "content": "Write session 022 README",
      "activeForm": "Writing session 022 README",
      "status": "completed"
    },
    {
      "content": "Write HANDOFF for session 023",
      "activeForm": "Writing HANDOFF for session 023",
      "status": "in_progress"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/022-chapter-21-draft/HANDOFF.md",
  "content": "# Handoff: Ch 21 done \u2192 proceed to Ch 22\n\n**For:** the next claude session.\n**From:** session 022.\n**Status:** Ch 21 drafted (~10,654 words, seventeen commits \u2014 the thread-local/alloca/VLA arc \u2014 `_Thread_local` and `__thread` with `%fs:`-relative addressing, `-include` and `-x` driver options, `-E` implies `-xc`, `alloca` as a builtin, the four-commit VLA arc, four linker-driver additions (`-l`, `-s`, ELF `.type`/`.size`, `.a`/`.so` recognition), `long double` as real extended precision, `[GNU]` case ranges, `[GNU]` array range designators, `[GNU]` labels-as-values). Continue autonomously to Ch 22 (Performance, dependency files, and the linker driver, commits 284\u2013306 \u2014 twenty-three commits covering labels-as-values as compile-time constant, the hashmap data structure and three uses of it, the `-M` family of dependency-output flags, `-fpic`/`-fPIC`, file-search caching, include-guard optimization, `#pragma once`, `#include_next`, `-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker`, the third-party-app test scripts). Don't pause for review.\n\n## Read these first, in order\n\n1. **[`docs/sessions/022-chapter-21-draft/README.md`](README.md)** \u2014 what session 022 did, including the six-section structure, the errata closures (long double is no longer double, \u00a719.7 array range designator), the two new errata candidates (`.size` for functions missing, suffix-only file recognition), the canonicalization count tick from ten to eleven, the psABI conformance count tick from sixteen to eighteen.\n2. **[`docs/sessions/021-chapter-20-draft/HANDOFF.md`](../021-chapter-20-draft/HANDOFF.md)** \u2014 the previous handoff. Standing notes still apply with Ch 21 updates folded in (see \u00a722 README for the running list).\n3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`21-thread-local-alloca-vlas.md`](../../../chapters/21-thread-local-alloca-vlas.md)** \u2014 the twenty-one chapters drafted. Match the register.\n4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** \u2014 confirms Ch 22 = commits 284\u2013306 (23 commits, scoped to \"Performance, dependency files, and the linker driver\"). The chapter mapping line lists the topics: hashmap, `-M` family, `-fpic`/`-fPIC`, caching, include-guard optimization, `#pragma once`, `#include_next`, `-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker`, third-party-app tests.\n5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** \u2014 quotable Rui quotes. Ch 22's commits are mostly performance and convenience features; the hashmap commits may have design notes worth scanning.\n6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** \u2014 JP TOC; performance and dependency files aren't commonly featured topics in compiler tutorials. The hashmap is the most reusable data-structure addition of the entire compiler; it might be a concept-interlude candidate.\n\n## Chapter 22 scope\n\n**Title (working):** *Performance, dependency files, and the linker driver*.\n**Commits:** 284\u2013306 in chronological order on `main`. **Twenty-three commits** \u2014 the longest-running chapter since Ch 19.\n**Concept interlude:** Possibly one. The hashmap (commits 285\u2013288: introduce, then macro lookup, scope lookup, keyword lookup) is a genuine reusable data-structure addition. Its three call sites profile different access patterns: macro names (insert-heavy, occasional lookup), block-scope names (lookup-heavy, brief lifetime), keywords (insert-once, lookup-many). A short interlude on *open-addressing tradeoffs and the three call-site profiles* could land between \u00a722.1 (the data structure) and \u00a722.2 (the three uses). Default conditional \u2014 judge while reading the commits.\n\n| # | Hash | Subject |\n|---|---|---|\n| 284 | `f0c98e0` | [GNU] Treat labels-as-values as compile-time constant |\n| 285 | `0aad326` | Add string hashmap |\n| 286 | `30520e5` | Use hashmap for macro name lookup |\n| 287 | `655954e` | Use hashmap for block-scope lookup |\n| 288 | `f694413` | Use hashmap for keyword lookup |\n| 289 | `d0c4667` | Add -M option |\n| 290 | `95d5a46` | Add -MF option |\n| 291 | `57c1d4e` | Add -MP option |\n| 292 | `db850f3` | Add -MT option |\n| 293 | `fb5cfe5` | Add -MD option |\n| 294 | `7aa72e4` | Add -MQ option |\n| 295 | `c3edffb` | Add -MMD option |\n| 296 | `86785fc` | Add -fpic and -fPIC options |\n| 297 | `c0f0614` | Cache file search results |\n| 298 | `d48d9e5` | Add include guard optimization |\n| 299 | `a6c6622` | [GNU] Add \"#pragma once\" |\n| 300 | `f10bceb` | [GNU] Add #include_next |\n| 301 | `1e9b6dd` | Add -static option |\n| 302 | `4e5de36` | Add -shared option |\n| 303 | `c8df787` | Add -L option |\n| 304 | `d1bc9a4` | Add -Wl, option |\n| 305 | `469f159` | Add -Xlinker option |\n| 306 | `fb49370` | Add scripts to test third-party apps |\n\nTwenty-three commits. The natural section grouping:\n\n- **\u00a722.1 \u2014 Labels-as-values as compile-time constant** (commit 284). One commit. The \u00a721.6 labels-as-values gave us `&&label` as an expression; this commit makes it usable in static initializers (e.g. global `void *jump_table[] = {&&l1, &&l2, &&l3};`). Touches the `eval` machinery and the global-initializer path. Walk how it slots into the existing `eval2`/`eval_rval` structure.\n- **\u00a722.2 \u2014 The string hashmap** (commit 285). One commit. A from-scratch open-addressing hashmap keyed by C strings. The most reusable data-structure addition of the entire compiler. Walk the API (`hashmap_get`, `hashmap_put`, `hashmap_delete`), the hash function, the load-factor and rehash trigger, the deletion-tombstone scheme.\n- **\u00a722.3 \u2014 Three hashmap users: macros, scopes, keywords** (commits 286\u2013288). Three commits. Each replaces a linear-search structure with a hashmap lookup. Walk the access-pattern differences. Likely a concept interlude candidate before this section, or fold the comparison into the section opener.\n- **\u00a722.4 \u2014 The `-M` family** (commits 289\u2013295). Seven commits. `-M`, `-MF`, `-MP`, `-MT`, `-MD`, `-MQ`, `-MMD` \u2014 the dependency-file generation that lets `make` know which headers a `.c` file pulls in. Each option does a small piece (output target name, escape characters, suppress-system-headers, write-to-file). Walk all seven; they're individually small but cumulatively define the output format. Likely the longest section of the chapter at ~2,500 words.\n- **\u00a722.5 \u2014 `-fpic`/`-fPIC` and file-search caching** (commits 296\u2013297). Two commits. `-fpic`/`-fPIC` enable position-independent code generation; chibicc's implementation likely flips a flag without changing codegen substantially (it's a marker for the linker). File-search caching speeds up include resolution by remembering successful and failed lookups. Walk both. The cache is the more interesting half \u2014 it's where the `c0f0614` commit threads the hashmap from \u00a722.2 into the include-search path.\n- **\u00a722.6 \u2014 Include-guard optimization, `#pragma once`, `#include_next`** (commits 298\u2013300). Three commits. Include-guard optimization detects the classic `#ifndef X #define X ... #endif` pattern and skips the file body on subsequent inclusions. `#pragma once` is the GNU/MSVC extension that asks for the same effect explicitly. `#include_next` is the GNU mechanism for a header to re-include \"the next file with the same name in the search path\" (used for chained system headers). Walk all three; they form a small triplet of include-handling improvements.\n- **\u00a722.7 \u2014 Linker-driver pass-throughs and the third-party-app harness** (commits 301\u2013306). Six commits. `-static`, `-shared`, `-L`, `-Wl,`, `-Xlinker` \u2014 five linker-related driver options that pass through to `ld`. Each is a small driver-side addition. The final commit (`fb49370`) adds shell scripts that build real-world C programs (libpng, sqlite, etc.) with chibicc. Walk all; the harness commit is a fitting place to close the chapter.\n\nThat's seven sections from twenty-three commits. **Target chapter length: ~12,000\u201314,000 words.** Likely closer to 13K \u2014 the `-M` family alone is seven commits with format details to walk, and the hashmap warrants substantial time.\n\n## Steps\n\n1. `cd research/sources/chibicc && for h in f0c98e0 0aad326 30520e5 655954e f694413 d0c4667 95d5a46 57c1d4e db850f3 fb5cfe5 7aa72e4 c3edffb 86785fc c0f0614 d48d9e5 a6c6622 f10bceb 1e9b6dd 4e5de36 c8df787 d1bc9a4 469f159 fb49370; do echo \"===== $h =====\"; git show --stat $h | head -10; done` to scan all 23 diffs.\n2. Read each commit. Pay particular attention to:\n   - **\u00a722.1's `f0c98e0`** \u2014 the `eval` machinery has to learn that `&&label` is a compile-time-constant address. Walk how `eval2`/`eval_rval` extend; likely adds an `ND_LABEL_VAL` arm that returns a relocation against the label's symbol.\n   - **\u00a722.2's `0aad326`** \u2014 the hashmap is the most reusable data structure in the compiler. Walk its API, the hash function (likely FNV or DJBX33A), the resize/rehash logic, and the deletion-tombstone scheme.\n   - **\u00a722.3's three users** \u2014 macros (insert-heavy, lookup-occasional), scopes (lookup-heavy, lifetime-brief), keywords (insert-once, lookup-many). Walk each conversion \u2014 what was the linear search before, what's the hashmap call now, and what's the asymptotic improvement.\n   - **\u00a722.4's `-M` family** \u2014 these are subtle. `-M` writes the dependency rule to stdout; `-MF FILE` redirects to a file; `-MP` adds phony rules so deleted headers don't break the build; `-MT TARGET` overrides the target name; `-MD` enables dependency-file generation alongside compilation; `-MQ` is `-MT` with quoting; `-MMD` is `-MD` minus system headers. Walk how each option modifies the dependency-output state.\n   - **\u00a722.5's `-fpic`/`-fPIC`** \u2014 chibicc's codegen probably already emits PIC-friendly code (rip-relative for globals, no absolute addresses), so the option may be a flag-flip without semantic changes. Confirm.\n   - **\u00a722.5's `c0f0614` file-search cache** \u2014 uses the hashmap from \u00a722.2. Walk the cache key (probably the include path entry) and the cache value (resolved path or \"not found\" sentinel).\n   - **\u00a722.6's include-guard optimization** \u2014 detects `#ifndef X #define X ... #endif`. Walk how the preprocessor recognizes the pattern, what it caches, and how subsequent `#include` calls short-circuit. Likely uses the hashmap.\n   - **\u00a722.6's `#pragma once`** \u2014 the explicit form of the same optimization. Walk how it integrates with the include-guard machinery.\n   - **\u00a722.6's `#include_next`** \u2014 the GNU extension. Walk how the search-path is iterated past the current file's directory.\n   - **\u00a722.7's linker pass-throughs** \u2014 small. `-Wl,foo,bar` becomes `foo bar` in the linker invocation. `-Xlinker arg` becomes `arg`. `-static`/`-shared`/`-L` are pass-throughs. Walk all five briefly.\n   - **\u00a722.7's `fb49370` third-party-app scripts** \u2014 verifies chibicc can build libpng, sqlite, etc. Walk the script structure and what it checks.\n3. Read the destination state at `fb49370` for `parse.c`, `tokenize.c`, `codegen.c`, `chibicc.h`, `main.c`, `preprocess.c`, plus the new `hashmap.c`. The hashmap, the `-M` machinery, and the include-guard optimization are the three most invasive subsystems.\n4. Draft `chapters/22-performance-deps-and-the-linker-driver.md`. Likely 12,000\u201314,000 words. Seven sections.\n5. Write `docs/sessions/023-chapter-22-draft/README.md`.\n6. Write `HANDOFF.md` for session 024 (Chapter 23 \u2014 Atomics and the final polish, commits 307\u2013316).\n\n## Voice / structure rules\n\nSame as Ch 1\u201321:\n- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, list the checkouts at the section opener.\n- \"we\" for reader, \"Rui\" for design intent.\n- Past tense for what the commit did, present tense for current behavior.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- Closing recap with feature table \u2014 twenty-three rows, single table is fine.\n- Diff format: lean toward inline diff fragments and quoted file snippets. The \u00a722.2 hashmap, \u00a722.4 `-M` family, and \u00a722.6 include-guard optimization will want larger code blocks.\n\n## Pitfalls to avoid\n\n(Carried forward and updated.)\n\n- Don't switch voice mid-chapter.\n- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.\n- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.\n- Don't ventriloquize Rui \u2014 quote `quotes-rui.md` only when there's a genuinely apt passage.\n- **\u00a722.2's hashmap is generic but the call sites have specific types.** The hashmap is keyed on C strings (`char *`) and stores `void *` values. Each call site casts the value at the boundary. Walk this; don't hide the casts.\n- **\u00a722.3's keyword-lookup conversion is the most interesting** because the keyword set is fixed. Pre-hashmap, keyword lookup was a linear scan of an array of ~30 strings. Post-hashmap, it's an `O(1)` lookup. Walk whether the hashmap is built once at startup or rebuilt per call.\n- **\u00a722.4's `-M` family generates Makefile syntax.** The dependency rule looks like `target: dep1 dep2 dep3 \\\\\\n  dep4 dep5`. Walk how chibicc handles escape characters (spaces in paths become `\\ `, dollar signs become `$$`). The format is documented in gcc's manual; chibicc reproduces it.\n- **\u00a722.5's `-fpic`/`-fPIC`** \u2014 these may not actually change codegen. Chibicc already emits rip-relative addressing for globals. The flag may just gate `.PIE`-related linker invocation. Confirm before writing.\n- **\u00a722.6's include-guard optimization is subtle.** The pattern is `#ifndef X / #define X / ... / #endif` with `X` as the same identifier in both lines. The preprocessor must verify the entire file body is wrapped in the guard (no leading non-whitespace content). Walk what counts as \"leading.\"\n- **\u00a722.6's `#include_next`** is order-sensitive. The search starts *after* the current file's location in the search path. If the current file is `/usr/include/foo.h`, `#include_next` finds the next `foo.h` in `-I` paths after `/usr/include`. Walk how the resume-point is tracked.\n- **\u00a722.7's linker driver** \u2014 five small commits. Don't over-explain each one individually; group them in a single subsection with a small example.\n- **\u00a722.7's third-party-app harness** \u2014 the commit adds shell scripts. Walk what they build (libpng, sqlite, others?) and what they verify (compilation, linking, basic test execution). Don't list every script.\n\n## Standing notes worth tracking across sessions\n\n- **The hideset on Token** \u2014 unchanged through Ch 21. Ch 22's macro-lookup hashmap doesn't touch hideset semantics.\n- **The Token->origin chain** \u2014 unchanged in Ch 21.\n- **The `Token` line-marker fields** \u2014 `display_name`, `filename`, `line_delta` added in Ch 20 \u00a720.1. Stable.\n- **The eval-quartet duplication** \u2014 has a fifth member (`is_const_expr`) since Ch 21 \u00a721.4. Likely picks up a sixth member or extends an existing one for \u00a722.1's labels-as-values-as-compile-time-constant.\n- **The cc1-vs-driver split** \u2014 unchanged.\n- **The `Initializer` tree** \u2014 Ch 19 added `Member *mem`; Ch 21 \u00a721.6 made array range designators honored in elaboration. Stable.\n- **The local-vs-global split** \u2014 Ch 21 \u00a721.1 added a third storage class for thread-local. `Obj` carries `is_tls`. Stable.\n- **The `Relocation` mechanism** \u2014 likely changes in \u00a722.1 for label-address compile-time relocations.\n- **The anonymous-global pattern** \u2014 likely unchanged in Ch 22.\n- **The `is_static` default in `new_gvar`** \u2014 gained `is_tls` companion in Ch 21. Stable.\n- **The `is_definition` flag on `Obj`** \u2014 stable since Ch 20.\n- **The `is_unsigned` flag on `Type`** \u2014 unchanged.\n- **The `__va_area__` magic name** \u2014 unchanged.\n- **The register-save-area layout** \u2014 unchanged.\n- **The argreg integer/FP split** \u2014 long double is on-stack; SSE for FP, GP for integer. Stable.\n- **The `Member->idx` field and bitfield siblings** \u2014 unchanged.\n- **The `is_flexible` flag** \u2014 unchanged. The dead-code duplicate from \u00a719.7's `835cd24` is still in the source.\n- **`copy_struct_type`** \u2014 unchanged.\n- **`MIN`/`MAX` macros** \u2014 unchanged.\n- **`is_numeric` predicate** \u2014 gained TY_LDOUBLE in Ch 21 \u00a721.6.\n- **`is_flonum` and `has_flonum` diverged in Ch 21 \u00a721.6.** `is_flonum` says yes for FLOAT, DOUBLE, LDOUBLE; `has_flonum` says yes only for FLOAT, DOUBLE.\n- **Canonicalization-at-parse-time count is at eleven.** Up from ten with Ch 21 \u00a721.4's VLA declaration rewrite. Ch 22 may add one in \u00a722.1 (label-as-value rewriting to a relocation, or none).\n- **Pre-factor-before-feature count is at nine.** Ch 22 unlikely to add new entries.\n- **psABI conformance count is at eighteen.** Ch 22's `-fpic`/`-fPIC` may add one for PIC-specific psABI rules.\n- **The fourth namespace (labels)** is unchanged. Labels-as-values still uses the same label table.\n- **The `is_typename` predicate** stable since Ch 21 (added `_Thread_local`, `__thread`).\n- **The `VarAttr` channel** has six fields after Ch 21 (typedef, static, extern, inline, tls, align). Stable.\n- **The `ND_NULL_EXPR` seed-pattern** \u2014 used in Ch 21 \u00a721.4's `compute_vla_size`. Stable.\n- **The `rep stosb` pattern** \u2014 unchanged. `alloca` and VLA regions are not zero-initialized.\n- **The `unreachable()` macro** \u2014 likely unchanged.\n- **Per-token line numbers** \u2014 unchanged through Ch 21.\n- **GDB-debuggable output** \u2014 unchanged.\n- **Tests are in C.** Likely new test files for hashmap users (indirect \u2014 through existing tests with bigger inputs), `-M` family (driver tests), include-guard optimization (preprocessor tests), `#include_next` (specific test).\n- **The `Obj->tok` field** added in Ch 14 \u00a714.11 still has no readers.\n- **The `Type->name_pos` field** \u2014 no new uses since Ch 14.\n- **The `Type->origin` field** added in Ch 20 \u00a720.3 for compatibility tracking. Stable.\n- **The `Obj` struct gained two fields in Ch 21** (`is_tls`, `alloca_bottom`).\n- **`Type` gained `vla_len`/`vla_size`** in Ch 21 \u00a721.4.\n- **The `Token`/`Node` `fval`** widened to `long double` in Ch 21 \u00a721.6.\n- **The `>>` codegen quirk** \u2014 partially repaired.\n- **The `add_type` rule for `ND_STMT_EXPR`** \u2014 errata candidate.\n- **The hex-escape silent truncation** \u2014 errata candidate.\n- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels, struct-members \u2014 five errata candidates.\n- **`f()` and `f(void)` are accepted as identical** \u2014 errata candidate.\n- **Empty brace initializer (`int x[3] = {};`)** \u2014 chibicc-specific extension.\n- **`.bss` is the third assembly section.** `.tdata`/`.tbss` are added in Ch 21 \u00a721.1, plus `.comm` for tentative commons. Total: five sections (`.text`, `.data`, `.bss`, `.tdata`, `.tbss`) plus `.comm` directive.\n- **`.align`** \u2014 unchanged.\n- **The `mov $0, %rax`** for variadic FP-count \u2014 errata candidate.\n- **The `fp_offset = fp * 8 + 48` non-conforming stride** \u2014 errata candidate.\n- **`long double` is `long double`** \u2014 *closed in Ch 21 \u00a721.6*. No longer aliased to `double`.\n- **The default-argument-promotion gap for chars and shorts** \u2014 errata candidate.\n- **Float literals are inlined as integer-immediate-bit-cast.**\n- **Long double literals are split across two halves through the redzone.**\n- **Ch 1 errata list** unchanged.\n- **The cast/compound-literal disambiguator** \u2014 same lookahead pattern.\n- **The cast table is 11\u00d711.** Grew in Ch 21 \u00a721.6 with the F80 row/column. Stable.\n- **Driver brittleness** \u2014 partially addressed by Ch 21's `-include`, `-x`, `-l`, `-s` additions. Ch 22's `-M` family, `-fpic`/`-fPIC`, `-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker` will round it out.\n- **The link command's hardcoded distro list** \u2014 errata candidate.\n- **`Node->funcname` is gone.**\n- **`mov %rax, %r10; call *%r10` is uniform across all calls.**\n- **The `StringArray` type** \u2014 picks up new uses for `-M` flags. Likely stable; may be supplemented by hashmap.\n- **`atexit(cleanup)` for tempfile disposal** \u2014 unchanged.\n- **The `run_subprocess` helper** \u2014 unchanged.\n- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `opt_S | opt_E` typo; default include paths Linux/glibc-specific. Three remaining.\n- **Errata candidates added in Ch 18:** None high-priority.\n- **Errata candidates added in Ch 19:**\n  - UTF-16 character-literal silent truncation of code points above U+FFFF.\n  - Dead-code duplicate `is_flexible` block in `array_initializer1`.\n  - Range designators not honored \u2014 *closed in Ch 21 \u00a721.6 commit `3d5550e`*.\n- **Errata candidates added in Ch 20:**\n  - `is_compatible` array arm bug.\n- **Errata candidates added in Ch 21:**\n  - Missing `.size` directive for function symbols (in \u00a721.5.3, commit `8d130ab`).\n  - Suffix-only `.a`/`.so` recognition (in \u00a721.5.4, commit `d56dd2f`).\n- **Errata candidates closed in Ch 21:**\n  - \"long double is double\" (closed by `e0bf168` in \u00a721.6).\n  - Range designators not honored (closed by `3d5550e` in \u00a721.6).\n- **`self.py` is gone.**\n- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean.\n- **Chibicc compiles itself** as of commit 197 (Ch 17 \u00a717.6).\n- **The `has_flonum` family** updated for long double in Ch 21 \u00a721.6 (it now distinguishes from `is_flonum`).\n- **Bitfield support is feature-complete.**\n- **Anonymous struct/union members** flatten via recursive `get_struct_member`.\n- **The pre-tokenize pass count is four** (Ch 19 \u00a719.6): BOM, newline, backslash-newline, UCN. Unchanged.\n- **The four char-literal prefixes** are functional.\n- **The four string-literal prefixes** are functional.\n- **`__STDC_UTF_16__` and `__STDC_UTF_32__`** are defined.\n- **`__STDC_NO_VLA__`** \u2014 *no longer defined as of Ch 21 \u00a721.4 commit `b0109a3`*.\n- **`__STDC_NO_THREADS__`** \u2014 *no longer defined as of Ch 21 \u00a721.1 commit `b377284`*.\n- **UTF-8 in identifiers** uses C11 Annex D ranges.\n- **The GNU `$` extension** in identifiers.\n- **`__DATE__`, `__TIME__`, `__COUNTER__`, `__TIMESTAMP__`, `__BASE_FILE__`** are predefined.\n- **Designated initializers** work for arrays, structs, unions, anonymous-struct, plus the GNU `=`-omission, plus array range designators.\n- **`__VA_OPT__` and `,##__VA_ARGS__` are functional.**\n- **GCC-style variadic macros (`name...`)** are functional.\n- **`#pragma` is silently consumed** (will be partially superseded by `#pragma once` in Ch 22 \u00a722.6).\n- **`typeof`, `_Generic`, `__builtin_types_compatible_p`** are functional.\n- **`sizeof(<function-type>)` returns 1.**\n- **The GNU `?:`-omitted-middle** is functional.\n- **`asm`** is verbatim-string-only.\n- **`inline` is treated as `static`**, with dead-static-inline elimination.\n- **`__attribute__` is macro-stubbed when `__GNUC__` is undefined.**\n- **`-idirafter`, `-fcommon`/`-fno-common`** are functional.\n- **`offsetof` is in `<stddef.h>`.**\n- **Tentative definitions are functional.** `.comm` (under `-fcommon`) or `.bss` (under `-fno-common`).\n- **`_Thread_local`/`__thread` (`%fs:`-relative addressing, `.tdata`/`.tbss`) are functional.**\n- **`alloca` is a builtin** that synthesizes inline stack manipulation.\n- **VLAs are functional**, allocated via `alloca` with sizes stored in hidden locals.\n- **`-include`, `-x`, `-E xc`, `-l`, `-s`, `.a`/`.so`** are in the driver vocabulary.\n- **`.type`/`.size`** directives are emitted (with `.size` missing for functions).\n- **`long double` is real 80-bit extended precision.**\n- **GNU case ranges (`case 1 ... 5:`)** are functional.\n- **GNU array range designators (`[3 ... 7] = x`)** are honored in elaboration.\n- **GNU labels-as-values (`&&label`, `goto *expr`)** are functional inside function bodies (compile-time constant variant in Ch 22).\n\n## Acceptance criteria for Ch 22\n\n- [ ] `chapters/22-performance-deps-and-the-linker-driver.md` exists, end-to-end readable.\n- [ ] All twenty-three commits covered, grouped into ~7 sections.\n- [ ] \u00a722.1 walks how labels-as-values gain compile-time-constant status.\n- [ ] \u00a722.2 walks the hashmap data structure (API, hash function, resize, deletion).\n- [ ] \u00a722.3 walks the three call sites and their access-pattern differences.\n- [ ] \u00a722.4 walks all seven `-M` options with attention to the dependency-file output format.\n- [ ] \u00a722.5 walks `-fpic`/`-fPIC` and the file-search cache, naming whether `-fpic` actually changes codegen.\n- [ ] \u00a722.6 walks include-guard optimization, `#pragma once`, and `#include_next`.\n- [ ] \u00a722.7 walks the five linker pass-throughs and the third-party-app harness.\n- [ ] Voice matches Ch 1\u201321.\n- [ ] No emoji, no callouts, no admonitions.\n- [ ] Forward-references checked against `chapter-mapping.md` (Ch 23 = 307\u2013316).\n- [ ] psABI conformance count noted (`-fpic`/`-fPIC` may grow it by one).\n- [ ] `docs/sessions/023-chapter-22-draft/README.md` written.\n- [ ] `HANDOFF.md` written for session 024 (Chapter 23 \u2014 Atomics and the final polish, commits 307\u2013316).\n\n## Prompt block to paste into a fresh session\n\n```\nContinue the chibicc book project. The user has asked for autonomous\nprogress \u2014 do not stop between chapters for review.\n\nRead in order:\n1. docs/sessions/022-chapter-21-draft/HANDOFF.md  (this handoff)\n2. docs/sessions/022-chapter-21-draft/README.md   (what session 022 did)\n3. chapters/01-a-calculator.md                     (template, voice)\n4. chapters/02-from-program-to-programs.md\n5. chapters/03-statements-and-local-variables.md\n6. chapters/04-pointers.md\n7. chapters/05-functions.md\n8. chapters/06-arrays.md\n9. chapters/07-globals-characters-strings.md\n10. chapters/08-scopes-and-source-locations.md\n11. chapters/09-structs-and-unions.md\n12. chapters/10-filling-out-the-type-system.md\n13. chapters/11-all-the-operators.md\n14. chapters/12-initializers.md\n15. chapters/13-linkage.md\n16. chapters/14-variadics-signedness-qualifiers.md\n17. chapters/15-floating-point.md\n18. chapters/16-the-compiler-driver.md\n19. chapters/17-a-preprocessor-from-scratch.md\n20. chapters/18-the-full-abi.md\n21. chapters/19-unicode-and-designated-initializers.md\n22. chapters/20-gcc-extensions-worth-supporting.md\n23. chapters/21-thread-local-alloca-vlas.md (most recent chapter)\n24. research/commits/chapter-mapping.md            (confirms Ch 22 scope)\n25. CLAUDE.md and book-plan.md                     (conventions)\n\nThen draft Chapter 22 (Performance, dependency files, and the linker\ndriver, commits 284\u2013306) per the steps in the handoff. Twenty-three\ncommits, seven sections proposed in the handoff. The \u00a722.2 hashmap is\nthe most reusable data-structure addition of the entire compiler; the\n\u00a722.3 three-call-site walk profiles macros, scopes, and keywords; the\n\u00a722.4 -M family is seven small commits whose cumulative format defines\nchibicc's dependency-file output. End-of-session: write your session\ndir under docs/sessions/023-chapter-22-draft/ with a README and a\nHANDOFF for session 024 (Chapter 23 \u2014 Atomics and the final polish,\ncommits 307\u2013316).\n```\n"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 17 Ch 21 commits",
      "activeForm": "Reading all 17 Ch 21 commits",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files",
      "activeForm": "Reading destination state of key files",
      "status": "completed"
    },
    {
      "content": "Draft chapter 21",
      "activeForm": "Drafting chapter 21",
      "status": "completed"
    },
    {
      "content": "Write session 022 README",
      "activeForm": "Writing session 022 README",
      "status": "completed"
    },
    {
      "content": "Write HANDOFF for session 023",
      "activeForm": "Writing HANDOFF for session 023",
      "status": "completed"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "in_progress"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017), Ch 17 drafted (session 018), Ch 18 drafted (session 019), Ch 19 drafted (session 020), Ch 20 drafted (session 021). Continuing autonomously per user direction.",
  "new_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017), Ch 17 drafted (session 018), Ch 18 drafted (session 019), Ch 19 drafted (session 020), Ch 20 drafted (session 021), Ch 21 drafted (session 022). Continuing autonomously per user direction."
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "offset": 60,
  "limit": 20
}


---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Ch 20 drafted (session 021). Continuing autonomously per user direction.",
  "new_string": "Ch 20 drafted (session 021), Ch 21 drafted (session 022). Continuing autonomously per user direction."
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 17 Ch 21 commits",
      "activeForm": "Reading all 17 Ch 21 commits",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files",
      "activeForm": "Reading destination state of key files",
      "status": "completed"
    },
    {
      "content": "Draft chapter 21",
      "activeForm": "Drafting chapter 21",
      "status": "completed"
    },
    {
      "content": "Write session 022 README",
      "activeForm": "Writing session 022 README",
      "status": "completed"
    },
    {
      "content": "Write HANDOFF for session 023",
      "activeForm": "Writing HANDOFF for session 023",
      "status": "completed"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "completed"
    }
  ]
}


---

## 🤖 Claude

Session 022 complete. Ch 21 drafted at ~10,654 words across six sections, covering all seventeen commits (267–283): thread-local variables (`%fs:`-relative addressing, `.tdata`/`.tbss`), the `-include`/`-x`/`-E xc` driver triple, `alloca` as a builtin (with the per-function `alloca_bottom` slide), the four-commit VLA arc (`TY_VLA`, `compute_vla_size`, `ND_VLA_PTR`, hidden-local-backed `sizeof`), four linker-driver additions (`-l`, `-s`, `.type`/`.size`, `.a`/`.so`), and the §21.6 quartet (`long double` as real x87 extended precision, GNU case ranges, GNU array range designators, GNU labels-as-values). Two errata closures (long-double-as-double, §19.7 array range designators) and two new errata (`.size` missing for functions, suffix-only file recognition). Cast table grew 10×10 → 11×11; psABI conformance count 16 → 18; canonicalization-at-parse-time count 10 → 11.

Handoff path: `docs/sessions/022-chapter-21-draft/HANDOFF.md`

---

