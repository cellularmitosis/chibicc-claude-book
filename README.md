# A walk through chibicc

_Disclaimer: this book was written by AI and Rui Ueyama has no involvement._

A book about [rui314/chibicc](https://github.com/rui314/chibicc) — Rui Ueyama's small C compiler — that walks the repository commit by commit, in the order Rui wrote them, with prose connecting each commit to the next.

Rui's compiler is unusual: it was built as a teaching artifact, one feature per commit, each commit small enough to read in isolation and ordered so that the language grows from a four-function calculator into something that can compile itself and a handful of real C programs. If the 316 commits on `main` were the curriculum, this book could be the lecture notes.

## Authorship

This book is being written **entirely by Claude Code** (Opus 4.7), with the project owner directing scope, approving structure, and steering revisions. Rui Ueyama is not a co-author and has no involvement in the book. The book follows Rui's pedagogical structure — his README, his commit ordering, the table of contents of his existing Japanese compiler book at <https://www.sigbus.info/compilerbook> — but the prose is original. Where the book explains *why* a design choice was made, it either cites Rui's own words or labels the explanation as interpretation.

## Status

**First draft complete.** All 23 chapters are drafted; ~180,000 words; roughly 600–700 printed pages. The draft has not yet been through a revision pass, errata appendix, or copyedit. Voice and structure should be consistent across chapters but have not been audited end-to-end.

## Who this is for

Experienced programmers who want to understand how a real C compiler is structured. No prior compiler background is assumed; the book builds the vocabulary as it goes. If you have written C and are comfortable reading C source, you have the prerequisites. The book is comparable in audience to Nora Sandler's *Writing a C Compiler* and Robert Nystrom's *Crafting Interpreters*, though it differs from both in following someone else's existing codebase rather than building one from scratch.

## How to read it

The book is designed to be read alongside a clone of chibicc. Each section opens with a `git checkout <hash>` command pinning the exact commit being discussed; the prose then walks the diff and the surrounding context.

```
git clone https://github.com/rui314/chibicc.git
cd chibicc
# then, at the start of each section in the book:
git checkout <hash>
```

You do not need to type the code. Reading the diff with the prose as a guide is enough. Running `make test` after each checkout is rewarding but optional.

## Table of contents

The chapters are in [chapters/](chapters/). Each chapter covers a contiguous run of commits.

1. [A calculator](chapters/01-a-calculator.md)
2. [From program to programs](chapters/02-from-program-to-programs.md)
3. [Statements and local variables](chapters/03-statements-and-local-variables.md)
4. [Pointers](chapters/04-pointers.md)
5. [Functions](chapters/05-functions.md)
6. [Arrays](chapters/06-arrays.md)
7. [Globals, characters, strings](chapters/07-globals-characters-strings.md)
8. [Scopes and source locations](chapters/08-scopes-and-source-locations.md)
9. [Structs and unions](chapters/09-structs-and-unions.md)
10. [Filling out the type system](chapters/10-filling-out-the-type-system.md)
11. [All the operators](chapters/11-all-the-operators.md)
12. [Initializers](chapters/12-initializers.md)
13. [Linkage](chapters/13-linkage.md)
14. [Variadics, signedness, qualifiers](chapters/14-variadics-signedness-qualifiers.md)
15. [Floating point](chapters/15-floating-point.md)
16. [The compiler driver](chapters/16-the-compiler-driver.md)
17. [A preprocessor from scratch](chapters/17-a-preprocessor-from-scratch.md)
18. [The full ABI](chapters/18-the-full-abi.md)
19. [Unicode and designated initializers](chapters/19-unicode-and-designated-initializers.md)
20. [GCC extensions worth supporting](chapters/20-gcc-extensions-worth-supporting.md)
21. [Thread-local, alloca, VLAs](chapters/21-thread-local-alloca-vlas.md)
22. [Performance, deps, and the linker driver](chapters/22-performance-deps-and-the-linker-driver.md)
23. [Atomics and the final polish](chapters/23-atomics-and-the-final-polish.md)

## Voice

The book uses **we** for the reader's shared journey ("we've added a tokenizer, so let's…"), **Rui** for design intent, and third person for everything else. Past tense describes what a commit *did*; present tense describes current behavior. The book is explicit when it's interpreting rather than reporting — phrases like "Rui doesn't say why he chose X here, but a likely reason is Y" are common.

There are no callout boxes, no admonitions, no emoji. If something matters, it's in the prose.

## Source material

- chibicc itself: <https://github.com/rui314/chibicc>
- Rui's existing Japanese compiler book (TOC used as a structural template for the early chapters): <https://www.sigbus.info/compilerbook>
- Verbatim Rui quotes that constrain authorial choices: [research/notes/quotes-rui.md](research/notes/quotes-rui.md)
- Full bibliography: [research/notes/sources.md](research/notes/sources.md)
- The chapter→commit mapping: [research/commits/chapter-mapping.md](research/commits/chapter-mapping.md)

## Project files

For the maintainer/contributor view of this working directory — workflow, session logs, project conventions — see [CLAUDE-README.md](CLAUDE-README.md) and [CLAUDE.md](CLAUDE.md).
