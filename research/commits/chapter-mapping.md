# Commit → Chapter mapping

316 commits on `main`, dates 2019-08-03 → 2020-12-07.

Numbers below are commit indices in `research/commits/main-commits.txt` (1-based, in chronological order). Commit hashes are in `main-commits-detailed.txt`.

This is a *first-pass* mapping. Granularity will likely be revised after writing Chapter 1 in full and seeing what feels right.

## Proposed structure: 22 chapters, 4 "parts"

Pattern: most chapters are *implementation* chapters (each section = one commit, in order). Concept chapters (machine language, ABI, executable image, type syntax, etc.) are interleaved just-in-time. This follows Rui's Japanese-book pattern and his explicit pedagogical philosophy.

### Front matter
- **Foreword** — The book wasn't written by Rui. Why we wrote it. How to read along with the repo.
- **Chapter 0: Setup** — toolchain (gcc, gdb, make), git, x86-64 Linux/macOS assumptions, how to clone chibicc and check out commits as you read.

---

### PART I — A working compiler in 30 commits

> Goal: get to a place where the compiler handles arithmetic, control flow, pointers, functions, and arrays. By the end of Part I we have a recognizably C-like language.

- **Chapter 1: A calculator** *(commits 1–7)*
  Single integer; `+ -`; tokenizer; error messages; `* / ()`; unary `+ -`; comparisons.
  *(Concept interludes within: machine language and assembly; recursive-descent parsing & BNF.)*

- **Chapter 2: From program to programs** *(commit 8)*
  The first refactor: splitting `main.c` into tokenizer/parser/codegen modules. Why this is the right time to do it.

- **Chapter 3: Statements and local variables** *(commits 9–18)*
  Single-letter locals → multi-char locals → return → blocks → null statement → `if` → `for` → `while`. Stack frames, prologue/epilogue.
  *(Concept interlude: how the System V AMD64 ABI lays out a stack frame.)*

- **Chapter 4: Pointers** *(commits 19–22)*
  Better error nodes; unary `&` and `*`; pointer arithmetic; the `int` keyword and mandatory variable declarations. Where types start to matter.
  *(Concept interlude: what a type *is*, and why the parser now needs to track them.)*

- **Chapter 5: Functions** *(commits 23–26)*
  Zero-arity calls; up-to-six-argument calls; function definitions. Caller/callee responsibilities, argument registers.

- **Chapter 6: Arrays** *(commits 27–31)*
  1-D arrays; arrays-of-arrays; subscript operator; `sizeof`; merging `Function` with `Var`. Array-to-pointer decay.

---

### PART II — Real C: globals, types, and initializers

> Goal: cover the C type system (structs, unions, typedef, enum), all the operators, all the statement forms, and the full initializer machinery. By the end we can compile programs you'd actually want to write.

- **Chapter 7: Globals, characters, and strings** *(commits 32–43)*
  Global variables; `char` type; string literals; escape sequences (`\a..\e`, octal, hex); statement expressions (GCC); reading from a file; `-o`/`--help`; line and block comments.
  *(Concept interlude: integer representation — sizes, sign, two's complement. From the JP book.)*

- **Chapter 8: Scopes and source locations** *(commits 44–48)*
  Block scope; rewriting tests in C; precomputed line numbers; `.file`/`.loc` directives; comma operator. Why the compiler now has enough infrastructure to give helpful diagnostics.

- **Chapter 9: Structs and unions** *(commits 49–55)*
  Struct definition and member access; alignment of struct and local-variable layouts; tags; `->`; unions; struct assignment.

- **Chapter 10: Filling out the type system** *(commits 56–75)*
  `int` becomes 32-bit; `long`, `short`, `void`, `_Bool`, `char` literal; nested declarators; function declarations and *complex* type declarations; `long long`; `typedef`; `sizeof(typename)`; 32-bit register usage; casts; usual arithmetic conversion; argument/return-type conversion; `enum`; file-scope functions.
  *(Concept interlude: how to read a C type declaration. From the JP book.)*

- **Chapter 11: All the operators** *(commits 76–96)*
  for-loop locals; `+=` family; pre/post `++ --`; hex/octal/binary literals; `!`, `~`, `%`, bitwise `&|^`, `&& ||`; incomplete arrays/structs; `goto` and labels (and the typedef/label conflict); `break`, `continue`; `switch`/`case`; shift operators; `?:`; constant expressions.

- **Chapter 12: Initializers** *(commits 97–115)*
  This is one of the densest arcs in the compiler — local then global; struct then union; flexible array members; `void` parameter list; alignment of globals. The whole grammar of C initializers, built up one form at a time.

- **Chapter 13: Linkage** *(commits 116–126)*
  `extern` (file and block scope); `_Alignof` and `_Alignas`; `static` locals and globals; compound literals; bare `return;`; `do…while`; 16-byte stack alignment; small-return-type handling.
  *(Concept interlude: static vs dynamic linking. From the JP book.)*

- **Chapter 14: Variadics, signedness, qualifiers** *(commits 127–138)*
  Calling and defining variadic functions with `va_start`; arg-count checking; `signed`/`unsigned`; integer suffixes; `const`/`volatile`/`auto`/`register`/`restrict`/`_Noreturn` (mostly ignored); empty parameter names.

- **Chapter 15: Floating point** *(commits 139–149)*
  A long arc, mostly self-contained: float/double literals; arithmetic; comparison; `if`/`while`/`!`/`?:`; floating-point function args/returns; default argument promotion; variadic floats; floating-point constexpr; `long double` as an alias for `double`.

---

### PART III — Becoming a real toolchain

> Goal: build the compiler driver, the preprocessor, and the rest of the ABI work needed to actually self-host. Self-hosting falls in the middle of this part as the climax.

- **Chapter 16: The compiler driver** *(commits 150–157)*
  stage2 build; function pointers (and arithmetic conversion thereof); split `cc1` from the driver; invoking `as` and `ld`; multiple input files. The compiler now looks like `gcc`, not just `cc1`.

- **Chapter 17: A preprocessor from scratch** *(commits 158–197)*
  This is the longest single arc — ~40 commits. Grouped into sub-sections:
  - 17.1 The skeleton: do-nothing preprocessor; null directive; `#include "..."`; `-E`.
  - 17.2 Conditionals: `#if`, `#endif`, nesting, `#else`, `#elif`.
  - 17.3 Macros: object-like; `#undef`; expansion in `#if`; the "don't expand twice" rule; `#ifdef`/`#ifndef`.
  - 17.4 Function-like macros: zero-arity; multi-arity; empty args; parenthesized args; the second "don't expand twice" rule; stringizing `#`; pasting `##`.
  - 17.5 Polish: `defined()`; identifier-to-zero in constexpr; whitespace preservation; line continuation; `<...>` includes; `-I`; default include paths; `#error`; predefined macros (`__STDC__`, `__FILE__`, `__LINE__`, `__VA_ARGS__`, `__func__`, `__FUNCTION__`); adjacent-string concatenation; wide character literal.
  - 17.6 **Self-hosting** *(commit 197)*: the moment chibicc compiles itself, including its own preprocessor.

- **Chapter 18: The full ABI** *(commits 198–220)*
  Stack-passed args/parameters; struct parameters/arguments/returns; >6-parameter variadics; `va_copy()`; function-deref no-op; pp-numbers; `-D`/`-U`; bitfields (and op=, zero-width, address-of restriction); buffered output; ignoring unknown flags; `-Wall`-clean self-build; 16-byte array alignment; implicit `return 0` in `main`; anonymous struct/union.

---

### PART IV — Standards conformance and extensions

> Goal: bring chibicc up to "compiles real third-party code" by filling in C11 features and the GCC extensions every codebase actually uses.

- **Chapter 19: Unicode and designated initializers** *(commits 221–244)*
  `__DATE__`/`__TIME__`/`__COUNTER__`; newline canonicalization; `\u`/`\U`; multibyte and UTF-{16,32} character and string literals (and initializers); identifier UTF-8 support; GNU `$` in identifiers; cross-prefix string concatenation; UTF-8 BOM. Then array/struct/union designated initializers and the GNU "omit `=`" extension.

- **Chapter 20: GCC extensions worth supporting** *(commits 245–266)*
  `#line` and line markers; `__TIMESTAMP__`/`__BASE_FILE__`; `__VA_OPT__`; the `,##__VA_ARGS__` GNU swallow; `#pragma`; GCC variadic macros; `typeof`; `__builtin_types_compatible_p`; `_Generic`; `sizeof` of function type; `?:` with omitted operand; basic `asm`; `inline` (as static); `__attribute__((format))`; `-idirafter`; `offsetof`; tentative definitions; `-fcommon`/`-fno-common`.

- **Chapter 21: Thread-local, alloca, VLAs** *(commits 267–283)*
  Thread-local variables; `-include`; `-x`; `-E` implies `-xc`; `alloca`; the VLA arc (`sizeof`, pointer arithmetic, `sizeof(typename)`, dropping `__STDC_NO_VLA__`); `-l`/`-s`; ELF size/type; `.a`/`.so` recognition; `long double`; case ranges; array range designators; labels-as-values (and compile-time constant).

- **Chapter 22: Performance, dependency files, and the linker driver** *(commits 284–306)*
  Hashmap and where to use it (macros, scopes, keywords); the `-M` family of dependency-output flags; `-fpic`/`-fPIC`; file-search caching; include-guard optimization; `#pragma once`; `#include_next`; `-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker`; the third-party-app test harness.

- **Chapter 23: Atomics and the final polish** *(commits 307–316)*
  `atomic_compare_exchange`; `atomic_exchange`; `_Atomic` and atomic op-assigns; `stdatomic.h`; the cpython smoke test; `__attribute__((packed))`; `__attribute__((aligned))`; member access through `=` and `?:`. The very last commit.

---

### Back matter
- **Afterword** — what we left out, what comes next (optimization, register allocation, alternative back ends), where chibicc fits in the lineage of small C compilers.
- **Appendix A: x86-64 instruction cheat sheet** *(from the JP book)*.
- **Appendix B: System V AMD64 ABI reference card** — calling convention, stack layout, type classification.
- **Appendix C: A guided tour of the final source tree** — what's in each `.c` file, in 2 pages.
- **Appendix D: Errata and corrections to chibicc** — Rui acknowledged he sometimes rewrites history to fix bugs in past commits. Anything we discover during writing belongs here.

---

## Sanity-check numbers

- Total commits: 316
- Chapter count: 23 implementation chapters + ~6 concept-interlude / appendix slots
- Average commits per chapter: ~14 (range: 1 [Ch 2] to ~40 [Ch 17, the preprocessor])
- The two outlier chapters (Ch 17 preprocessor, Ch 22 perf+linker-driver) are good outliers — they're naturally cohesive and the reader needs continuous prose to follow them.

## Open structural questions to decide while writing

1. **Should Chapter 1 cover commits 1–7 in one go, or split at "first tokenizer" (commit 3)?** The Japanese book groups them; I propose we do the same and revisit after writing.
2. **Should Chapter 17 be one chapter or split into 6 sub-chapters?** It's the natural boundary for a "Part" of its own. I lean toward one chapter with strong sub-headings, since the reader needs to feel the cumulative momentum that ends in self-hosting.
3. **How do we handle commits that are pure refactors (35, 41, 215, 285)?** Probably mention them in passing within the surrounding section — don't give them their own section.
4. **Where does the JP book's "1973 C compiler" historical interlude land?** Probably end of Chapter 5 (Functions), as it does in the Japanese book.
