# Sources Consulted

Research session: 2026-05-07.

## Primary sources (Rui Ueyama)

### The chibicc repository
- **URL:** https://github.com/rui314/chibicc
- **Local clone:** `research/sources/chibicc/`
- **Branches:**
  - `main` — 316 commits, dates 2019-08-03 → 2020-12-07. The carefully-curated "book" branch. **This is the canonical source.**
  - `historical/old` — 115 commits. Pre-rewrite chibicc (~2019, based on 9cc lineage). Likely the implementation that the existing Japanese book documents.
  - `new` — 304 commits. Older snapshot; tip has experimental `_Atomic` + cpython tests.
  - `reference` — 93 commits. Earliest reference snapshot.
- **README** — extremely informative; explicitly states book intent, design principles, and pedagogical philosophy. Quoted in `quotes-rui.md`.

### Rui's existing Japanese compiler book
- **URL:** https://www.sigbus.info/compilerbook
- **Title:** 低レイヤを知りたい人のためのCコンパイラ作成入門
  ("Introduction to C Compiler Creation for Low-Level Knowledge Seekers")
- **Status:** Half-written. Covers ~steps 1–28 (≈ first 50 chibicc commits); "Step 29 and beyond" explicitly marked `[要加筆]` (to be written).
- **Significance:** This is the *predecessor* book. Its TOC reveals Rui's preferred chapter structure for the early material. Likely tracks the `historical/old` branch, not the rewritten `main`.
- **Translated TOC:** see `notes/japanese-book-toc.md`.

### Rui's homepage
- **URL:** https://www.sigbus.info/
- Hosts the compiler book and other writing.

### HN comments by rui314 (2020-09 thread)
- **URL:** https://news.ycombinator.com/item?id=24676851
- Confirms half-Japanese book exists; "plans to translate chapters to English, though nothing is set in stone yet."

### HN comments by rui314 (2022-11 thread)
- **URL:** https://news.ycombinator.com/item?id=33581704
- Direct quote: *"the chibicc book is not available yet. I'm busy working on the other project (the mold linker) and don't have time to work on it."*
- This is the most recent confirmation that the English book remains unwritten.

### Reddit announcement (2020)
- **URL:** https://www.reddit.com/r/Compilers/comments/j2py9x/im_writing_a_c_compiler_called_chibicc_and_a_book/
- Could not be fetched directly (Claude Code blocked from reddit and web.archive.org).
- Content is largely captured by README and HN threads; gap is acceptable.

### GitHub Issue #78 ("The book?")
- **URL:** https://github.com/rui314/chibicc/issues/78
- Reader asking about book status (Dec 2021); no reply from Rui in the visible thread.

## Secondary sources (community)

### Wikis & summaries
- **DeepWiki chibicc page:** https://deepwiki.com/rui314/chibicc
- **Bill Mill notes:** https://notes.billmill.org/programming/compilers/Chibicc_-_C_compiler_built_for_learning.html

### Community reimplementations of the existing Japanese book
These follow `sigbus.info/compilerbook` and are useful for sanity-checking the early-chapter mapping:
- https://github.com/pokotsun/chibicc — most-cited
- https://github.com/r1ru/clang-compiler — direct reference to sigbus book in description
- https://github.com/yorisilo/compiler-book-rui-ueyama
- https://github.com/jethrodaniel/z (Crystal port)

### Community discussion / context
- Brian Callahan, "OpenBSD has two new C compilers: chibicc and kefir": https://briancallahan.net/blog/20220629.html

## Sources cited by Rui in the chibicc README

These are the works Rui himself called out as influences. Worth referencing in our book.

1. **An Incremental Approach to Compiler Construction** — Abdulaziz Ghuloum (Scheme 2006)
   - PDF: http://scheme2006.cs.uchicago.edu/11-ghuloum.pdf
   - Locally: `~/prog/_compilers/An Incremental Approach to Compiler Construction.pdf`
   - **The structural inspiration for chibicc's curriculum.** Ghuloum's thesis: a compiler course should produce a working compiler at every step, expanding the language one feature at a time. Rui adopted this wholesale.
2. **lcc** by Fraser & Hanson — https://github.com/drh/lcc
   - Companion book *A Retargetable C Compiler: Design and Implementation* — locally at `~/prog/_compilers/A Retargetable C Compiler.pdf`. Rui calls this "a good resource to see how a compiler is implemented."
3. **tcc** by Fabrice Bellard — https://bellard.org/tcc/
   - Rui notes design differs (tcc is one-pass; chibicc is multi-pass).
4. **Rob Pike's 5 Rules of Programming** — https://users.ece.utexas.edu/~adnan/pike.html
   - Captures chibicc's "no premature optimization, prefer dumb code" ethos.
5. **DMD memory-allocation policy article** (Dr. Dobb's, Walter Bright) — Rui cites this to justify chibicc's "calloc-and-never-free" approach.

## Other reference works available locally

In `~/prog/_compilers/`:
- **Writing a C Compiler** (Nora Sandler, 2024) — closest peer to what we're writing. Pedagogical comparison only.
- **Essentials of Compilation: An Incremental Approach** (Siek) — modern incremental approach, Racket→x86. Structural reference.
- **A Nanopass Framework for Compiler Education** (Sarkar et al.) — alternative pedagogical paradigm.
- **A Retargetable C Compiler** (Fraser & Hanson) — see above.
- **Engineering a Compiler** (Cooper & Torczon, 2nd) — heavyweight reference.
- **Modern Compiler Implementation in C/Java/ML** (Appel) — heavyweight reference.
- **System V ABI AMD64 Processor Supplement** — *essential* reference for x86-64 codegen chapters.
- **The Linux Programming Interface** (Kerrisk) — for ELF/process/syscall background chapters.
