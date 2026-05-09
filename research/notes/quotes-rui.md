# Verbatim quotes from Rui Ueyama

These are statements by Rui that the book should be faithful to. Source for each is given.

## On the book's pedagogical structure
*(chibicc README)*

> chibicc is developed as the reference implementation for a book I'm currently writing about the C compiler and the low-level programming. The book covers the vast topic with an incremental approach; in the first chapter, readers will implement a "compiler" that accepts just a single number as a "language", which will then gain one feature at a time in each section of the book until the language that the compiler accepts matches what the C11 spec specifies. I took this incremental approach from the paper by Abdulaziz Ghuloum.

> Each commit of this project corresponds to a section of the book. For this purpose, not only the final state of the project but each commit was carefully written with readability in mind. Readers should be able to learn how a C language feature can be implemented just by reading one or a few commits of this project.

## On the bug-free-history policy
*(chibicc README)*

> When I find a bug in this compiler, I go back to the original commit that introduced the bug and rewrite the commit history as if there were no such bug from the beginning. This is an unusual way of fixing bugs, but as a part of a book, it is important to keep every commit bug-free.

## On the design principles
*(chibicc README)*

> chibicc's core value is its simplicity and the reability of its source code. To achieve this goal, I was careful not to be too clever when writing code.

> Oftentimes, as you get used to the code base, you are tempted to *improve* the code using more abstractions and clever tricks. But that kind of *improvements* don't always improve readability for first-time readers and can actually hurts it. I tried to avoid the pitfall as much as possible. I wrote this code not for me but for first-time readers.

> The recursive descendent parser contains many similar-looking functions for similar-looking generative grammar rules. You might be tempted to *improve* it to reduce the duplication using higher-order functions or macros, but I thought that that's too complicated. It's better to allow small duplications instead.

> chibicc allocates memory using `calloc` but never calls `free`. Allocated heap memory is not freed until the process exits. I'm sure that this memory management policy (or lack thereof) looks very odd, but it makes sense for short-lived programs such as compilers.

> Slow algorithms are fine if we know that n isn't too big.

## On the existing Japanese book and the English book
*(HN, 2020-09)*

> [A] half-written Japanese version is freely available online, with plans to translate chapters to English, though nothing is set in stone yet.

## On the book's status
*(HN, 2022-11)*

> the chibicc book is not available yet. I'm busy working on the other project (the mold linker) and don't have time to work on it.

But of his code:

> chibicc's each commit was carefully written so that you can read one commit at a time.

## On the audience and goal of the Japanese book
*(sigbus.info/compilerbook, paraphrased / translated)*

The Japanese book targets "ordinary C programmers" who understand pointers and arrays but don't need to be language-spec experts. The stated goal is to **build a self-hosting compiler through incremental steps** rather than to teach all of compiler theory. The book "explicitly rejects traditional compiler textbooks that separate parsing, intermediate passes, and code generation into isolated chapters."

---

## Implications for the book we're writing

1. **Sections = commits.** This is Rui's explicit intent. Don't over-aggregate.
2. **Chapters = coherent groups of commits.** The Japanese TOC confirms this is what Rui himself does (e.g., "Functions and Local Variables" groups steps 9–15).
3. **Mix in concept chapters** between implementation chapters when readers need background (machine language, integer representation, executable structure, type syntax — see the JP TOC).
4. **Follow Rui's "be dumb on purpose" ethos.** If we're tempted to abstract or DRY-up the code we're explaining, *don't*. Explain why the duplication is intentional.
5. **Self-hosting is the destination.** This is the explicit endpoint. Frame the whole book around it.
6. **Don't shy away from low-level systems chapters.** The Japanese book has chapters on CPUs/memory, integer representation, executable image, and linking — Rui considers these in scope.
