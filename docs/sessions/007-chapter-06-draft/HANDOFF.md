# Handoff: Ch 6 done → proceed to Ch 7

**For:** the next claude session.
**From:** session 007.
**Status:** Ch 6 drafted. Continue autonomously to Ch 7 (Globals, characters, and strings). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/007-chapter-06-draft/README.md`](README.md)** — what session 007 did, including the canonicalization-at-parse-time naming, the pre-factor-before-feature pattern naming, and the array-decay interlude shape.
2. **[`docs/sessions/006-chapter-05-draft/HANDOFF.md`](../006-chapter-05-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`06-arrays.md`](../../../chapters/06-arrays.md)** — the six chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 7 = commits 32–43.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.

## Chapter 7 scope

**Title (working):** *Globals, characters, and strings*.
**Commits:** 32–43 in chronological order on `main`.

| # | Hash | Subject |
|---|---|---|
| 32 | `a4d3223` | Add global variables |
| 33 | `be38d63` | Add char type |
| 34 | `4cedda2` | Add string literal |
| 35 | `35a0bcd` | Refactoring: Add a utility function |
| 36 | `ad7749f` | Add `\a, \b, \t, \n \v, \f, \r` and `\e` |
| 37 | `699d2b7` | Add `\<octal-sequence>` |
| 38 | `c2cc1d3` | Add `\x<hexadecimal-sequence>` |
| 39 | `9dae234` | [GNU] Add statement expression |
| 40 | `d9ea597` | Read code from a file instead of argv[1] |
| 41 | `7b8528f` | Refactor -- no functionality change |
| 42 | `a0388ba` | Add `-o` and `--help` options |
| 43 | `6c0a429` | Add line and block comments |

**Twelve commits** is by far the longest commit range for one chapter so far (Chapter 1 had 18, but several were tiny Makefile/scaffolding tweaks; this is twelve real-feature commits). The natural sectioning probably *isn't* one section per commit — that would give a 12-section chapter that loses cohesion. A more reasonable shape:

- **§7.1 — Global variables** (commit 32)
- **§7.2 — The `char` type** (commit 33)
- **§7.3 — String literals** (commits 34, 35) — bundle the small refactor with the literal commit
- **§7.4 — Escape sequences** (commits 36, 37, 38) — bundle the three escape-sequence commits since they're all variations on the same parser logic
- **§7.5 — Statement expressions** (commit 39) — the GNU extension
- **§7.6 — Driver maturation** (commits 40, 41, 42, 43) — read-from-file, refactor, command-line flags, comments. Bundle these because they're all about the compiler-as-program rather than the language. Actually, this might split into two: a "driver maturation" §7.6 (40, 41, 42) and a "comments" §7.7 (43). Decide based on flow.

**Concept interlude:** the chapter mapping calls for a concept interlude on **integer representation — sizes, sign, two's complement** (cited as "from the JP book"). This is the natural moment because the `char` commit is when chibicc finally has more than one integer type, and the question "what's the difference between `char` and `int` other than size" wants an answer. Place it between §7.1 (globals, where the size-aware machinery from Ch 6 finally gets used at file scope) and §7.2 (char). Length: probably 1,500–2,500 words. Cover at minimum:

- Bits → bytes; what "size in bytes" actually means at the hardware level.
- Two's-complement representation of signed integers; why it's not "sign-and-magnitude."
- Sign extension when widening (the `movsx` family); zero extension for unsigned (which chibicc doesn't have yet).
- The C standard's "implementation-defined sizes" — chibicc's choice of 8-byte int and (after this chapter) 1-byte char.
- Brief note on what chibicc *doesn't* have: no unsigned types yet, no shorts/longs, no integer promotion rules. Those arrive in Ch 10.

The JP book's framing is more historical — it goes into how older machines used different representations. Chibicc's audience (per CLAUDE.md: "competent programmers") probably knows two's-complement already, so the interlude can be tighter than the JP book's chapter would be. Calibrate based on prose flow.

## Steps

1. `cd research/sources/chibicc && for h in a4d3223 be38d63 4cedda2 35a0bcd ad7749f 699d2b7 c2cc1d3 9dae234 d9ea597 7b8528f a0388ba 6c0a429; do git show --stat $h; done` to scan all twelve diffs at once.
2. Read each commit in full. Pay particular attention to:
   - **`a4d3223`**: how the parser dispatches between function and global var (the lookahead trick), the `.data` section directive, how globals are addressed in codegen (`%rip`-relative).
   - **`be38d63`**: the multi-width register tables in codegen (`argreg8`, `argreg64`?), the `movsx` instructions for sign extension, how `add_type` handles `char` (it should still classify as `is_integer`).
   - **`4cedda2`**: how string literals are anonymous globals with synthesized names (`L..1`, etc.), the `char[N]` typing.
   - **`9dae234`**: statement expressions are GNU `({ ... })` — almost certainly the parser desugars them into something chibicc already handles; expect another canonicalization-at-parse-time instance.
   - **`d9ea597`**: file I/O, plus likely the main.c refactor that goes with it.
3. Read the destination state at `6c0a429` (or the next major checkpoint) for `chibicc.h`, `parse.c`, `codegen.c`, `tokenize.c`. Particularly: how is the symbol-table search updated to look at globals as well as locals?
4. Draft `chapters/07-globals-characters-strings.md`. Likely 10,000–13,000 words (twelve commits is *a lot* of material). If it crosses 14,000 words, consider whether the chapter mapping should split it — but that's a question for a replanning session, not for the drafter.
5. Write `docs/sessions/008-chapter-07-draft/README.md`.
6. Write `HANDOFF.md` for session 009 (Chapter 8 — Scopes and source locations, commits 44–48).

## Voice / structure rules

Same as Ch 1–6:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote — but with a wrinkle for bundled sections like §7.4 (three commits about escape sequences). Each commit still gets its own checkout line; the section can have an opening paragraph that frames the trio, then walks each commit in turn. Look at how Chapter 1 handled bundled commits if needed (it had several).
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with a feature table — twelve rows is a lot; consider whether to group the table by §-section or list one row per commit. Per-commit feels more useful for someone using the recap as a reference.
- Diff format: lean toward inline diff fragments and quoted file snippets; the larger commits in this chapter (`a4d3223`, `be38d63`) deserve thematic chunking.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage. Chapter 7's globals or string-literal moments might be the first time something in `quotes-rui.md` lines up; check before passing.
- The integer-representation interlude has *enormous* prior art on the web. Resist external structure. The book's audience knows two's-complement; the interlude should focus on what's specifically relevant for *this compiler* (sign extension during widening, why `movsx` is one instruction, why chibicc skips unsigned for now), not on general bit-twiddling.
- Twelve commits in one chapter is a lot. Resist the temptation to give every single commit equal weight. The four "driver maturation" commits at the end are short and similar; bundling them and being terse is better than giving each its own subsection.
- The string-literal commit (`4cedda2`) introduces *anonymous globals* — the first time chibicc emits something with a synthesized name. This is conceptually significant and worth treating with care.
- Statement expressions (`9dae234`) are a GNU extension, not standard C. The book should call this out — it's the first chibicc feature that's not in the C standard.

## Standing notes worth tracking across sessions

- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`. Address in a revision pass.
- **The `mov $0, %rax` (variadic `%al`-zeroing)** is established in Ch 5 §5.1. Footnote with SysV ABI section reference (3.2.3) is a possible revision-pass addition.
- **The "more than 6 args silently miscompiles"** call-out is established in Ch 5 §5.4. Errata appendix candidate when one exists.
- **The hardcoded `8` in `new_add`/`new_sub`** is now removed (Ch 6 §6.1). Done — no more carrying this note.
- **`Function` and `Obj` merged in Ch 6 §6.5.** Done — no more carrying this note.
- **TY_FUNC still has no consumer.** Chapter 10 (Filling out the type system) is where it gets pulled into nested-declarator parsing. Mention in the Ch 10 session, not before.
- **Canonicalization-at-parse-time** is now a named pattern (four instances at end of Ch 6: `>` swap, `while` as `for`, pointer-arithmetic scaling, `[]` desugaring). Future instances can reference the Ch 6 §6.3 naming. Statement expressions in Ch 7 §7.5 are a likely fifth instance.
- **The "pre-factor before feature" pattern** is also named (Ch 6 §6.5). Future instances can reference. Most likely Ch 7 candidate: commit 41 (`Refactor -- no functionality change`) just before the `-o`/`--help` commit.
- **The `.text` directive** landed in Ch 6 §6.5 without a `.data` counterpart. Ch 7 §7.1 should land `.data` and back-reference.
- **The `add_type` `ND_ADDR` simplification** (collapsing `&array` to `pointer_to(base)`) is a Ch 6 limitation, fix-target in Ch 10.
- **The `argreg` array uses only 64-bit register names** at the end of Ch 6. Ch 7 §7.2 (`char`) breaks this assumption — register tables for multiple widths arrive there.

## Acceptance criteria for Ch 7

- [ ] `chapters/07-globals-characters-strings.md` exists, end-to-end readable.
- [ ] Concept interlude on integer representation lands somewhere between §7.1 and §7.2.
- [ ] Each commit has a `git checkout <full-hash>` opener somewhere — even if multiple commits share a section, each commit still gets its own subsection-level checkout line.
- [ ] Voice matches Ch 1–6.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] String literals are framed as anonymous globals.
- [ ] Statement expressions are flagged as a GNU (non-standard-C) extension.
- [ ] The multi-width register tables in `codegen.c` for `char` are explained.
- [ ] `docs/sessions/008-chapter-07-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 009 (Chapter 8 — Scopes and source locations).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/007-chapter-06-draft/HANDOFF.md  (this handoff)
2. docs/sessions/007-chapter-06-draft/README.md   (what session 007 did)
3. chapters/01-a-calculator.md                     (template, voice)
4. chapters/02-from-program-to-programs.md
5. chapters/03-statements-and-local-variables.md
6. chapters/04-pointers.md
7. chapters/05-functions.md
8. chapters/06-arrays.md                           (most recent chapter)
9. research/commits/chapter-mapping.md             (confirms Ch 7 scope)
10. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 7 (Globals, characters, and strings, commits 32–43)
per the steps in the handoff. End-of-session: write your session dir
under docs/sessions/008-chapter-07-draft/ with a README and a HANDOFF
for session 009 (Chapter 8 — Scopes and source locations).
```
