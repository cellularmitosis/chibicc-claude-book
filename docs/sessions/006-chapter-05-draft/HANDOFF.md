# Handoff: Ch 5 done → proceed to Ch 6

**For:** the next claude session.
**From:** session 006.
**Status:** Ch 5 drafted. Continue autonomously to Ch 6 (Arrays). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/006-chapter-05-draft/README.md`](README.md)** — what session 006 did, including the three interpretive calls, the deliberate non-anticipation of the `Function`/`Obj` merge, and the calling-convention interlude shape.
2. **[`docs/sessions/005-chapter-04-draft/HANDOFF.md`](../005-chapter-04-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)**, **[`02-from-program-to-programs.md`](../../../chapters/02-from-program-to-programs.md)**, **[`03-statements-and-local-variables.md`](../../../chapters/03-statements-and-local-variables.md)**, **[`04-pointers.md`](../../../chapters/04-pointers.md)**, **[`05-functions.md`](../../../chapters/05-functions.md)** — the five chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 6 = commits 27–31.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.

## Chapter 6 scope

**Title (working):** *Arrays*.
**Commits:** 27–31 in chronological order on `main`.

| # | Hash | Subject |
|---|---|---|
| 27 | `8b6395d` | Add one dimensional arrays |
| 28 | `3ce1b2d` | Add arrays of arrays |
| 29 | `648646b` | Add `[]` operator |
| 30 | `3e55caf` | Add `sizeof` |
| 31 | `0b76634` | Merge `Function` with `Var` |

Five commits — the longest chapter so far by commit count. The natural pacing is one section per commit. The chapter mapping notes the chapter as covering "1-D arrays; arrays-of-arrays; subscript operator; `sizeof`; merging `Function` with `Var`. Array-to-pointer decay."

**Concept interlude:** none mandated by the chapter mapping. *Optional* candidates (decide based on flow as you draft):

- A short interlude on **array-to-pointer decay** — what it is, why C does it, why chibicc handles it the way it does. Could land between §6.1 (1-D arrays) and §6.2 (arrays-of-arrays), once the reader has seen one array and is ready to think about what `int x[5]` *really* is. Probably worth doing. Keep it short — 600–1,000 words. The Ch 4 type interlude is the model for "small interlude doing real work."
- A short note on **how `sizeof` is implemented** as a parse-time computation (it returns an integer literal because all sizes are known at parse time). Probably belongs inside §6.4 rather than as a separate interlude.

If the chapter naturally wants two interludes, that's fine. The Ch 5 calling-convention interlude was substantial (~2,000 words) but Ch 5 also had only four commits; Ch 6 has five and is going to be tighter on per-section budget.

## Steps

1. `cd research/sources/chibicc && for h in 8b6395d 3ce1b2d 648646b 3e55caf 0b76634; do git show --stat $h; done` to scan diffs.
2. Read each commit in full (`git show <hash>`). The codegen impact is concentrated in `8b6395d` (loads/stores have to know array size) and `3e55caf` (`sizeof` returns a parse-time constant); the parser does most of the work in the others.
3. Read the destination state at `0b76634` (chibicc.h, parse.c, codegen.c, type.c). Particularly: how does the merged `Function`/`Obj` look? How does `Type` carry `size`?
4. Draft `chapters/06-arrays.md`. Likely 7,000–10,000 words depending on whether the array-to-pointer decay interlude lands and how big it gets.
5. Write `docs/sessions/007-chapter-06-draft/README.md`.
6. Write `HANDOFF.md` for session 008 (Chapter 7 — Globals, characters, and strings, commits 32–43, an unusually large commit range for one chapter).

## Voice / structure rules

Same as Ch 1–5:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with a feature table.
- Diff format: lean toward inline diff fragments and quoted file snippets; break larger diffs into thematic chunks.

## Pitfalls to avoid

(Carried forward.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits; cross-check `chapter-mapping.md`.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- Array-to-pointer decay has substantial prior-art on the web; resist external-source structure. Match the voice from the Ch 4 type interlude and the Ch 3 stack-frame interlude.
- The `0b76634` commit is a refactor with no new feature — make sure the section still has shape and opinion, not just "Rui renamed things." The merge has a *reason* (functions and variables really are the same shape: name, type, optionally a body or a value), and that reason is the section's spine.

## Standing notes worth tracking across sessions

- **Ch 1 errata list** unchanged: test-count off-by-one (claims 28 at end, actual 27) and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`. Address in a revision pass.
- **The `mov $0, %rax` (variadic `%al`-zeroing)** is now established in Ch 5 §5.1. A footnote with the SysV ABI spec section number (3.2.3) would be a nice revision-pass addition but isn't blocking.
- **The "more than 6 args silently miscompiles"** call-out is established in Ch 5 §5.4 and the calling-convention interlude. Worth tagging for the errata appendix when one exists.
- **The hardcoded `8` in `new_add`/`new_sub`** is destined to become `lhs->ty->base->size` in Ch 6 §6.4 (commit `3e55caf` — `Add sizeof`). Both Ch 4 and Ch 5 flagged it; when Ch 6 makes the change, the natural move is to *show* the change rather than re-explain the foreshadowing — three setups is plenty.
- **`Function` and `Obj` merge in Ch 6 §6.5 (commit `0b76634`).** Ch 5 §5.4 noted that `fn->params` is a prefix of `fn->locals` and parameters are "also" locals — that sets up the merge as intuitive rather than surprising. The Ch 6 prose can briefly back-reference this Ch 5 framing.
- **TY_FUNC has no user yet.** Chapter 10 (Filling out the type system) is where it gets pulled into nested-declarator parsing. Mention in the Ch 10 session, not before.
- **Canonicalization-at-parse-time** still has just three named instances (`>` rewritten as `<` swap, `while` as a degenerate `for`, pointer arithmetic scaled in `new_add`/`new_sub`). Ch 6's `[]` operator is the *fourth* instance — Rui implements `a[b]` as `*(a+b)` in the parser, and codegen never sees `[]`. This is the moment to formally name the pattern. The HANDOFF from session 005 flagged this; session 006's note is that we still didn't name it because Ch 5 didn't add a clear new instance, but Ch 6 has one.
- **Multi-byte sizes finally arrive in Ch 6.** Since Ch 1 chibicc has assumed every value is 8 bytes. Arrays with `int x[5]` means 5 × 8 = 40 bytes. Then Ch 7's `char` breaks the assumption that scaling is always by 8. The Ch 6 prose should set up the moment of "sizes vary" without going so far as to rehearse Chapter 7. The natural framing: Ch 6 adds `sizeof`, which is the *first* feature where size-of-thing matters; Ch 7 will add a non-`int` type that exercises it for real.

## Acceptance criteria for Ch 6

- [ ] `chapters/06-arrays.md` exists, end-to-end readable.
- [ ] Each section opens with `git checkout <full-hash>` and the commit's subject.
- [ ] Voice matches Ch 1–5.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] The `[]`-as-`*(a+b)` desugaring is explicitly framed (and probably named-and-called-out as the canonicalization-at-parse-time pattern).
- [ ] The `Function`/`Obj` merge is given real treatment in §6.5 — not skipped as "just a refactor."
- [ ] `docs/sessions/007-chapter-06-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 008.

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/006-chapter-05-draft/HANDOFF.md  (this handoff)
2. docs/sessions/006-chapter-05-draft/README.md   (what session 006 did)
3. chapters/01-a-calculator.md                     (template, voice)
4. chapters/02-from-program-to-programs.md
5. chapters/03-statements-and-local-variables.md
6. chapters/04-pointers.md
7. chapters/05-functions.md                        (most recent chapter)
8. research/commits/chapter-mapping.md             (confirms Ch 6 scope)
9. CLAUDE.md and book-plan.md                      (conventions)

Then draft Chapter 6 (Arrays, commits 27–31) per the steps in the
handoff. End-of-session: write your session dir under
docs/sessions/007-chapter-06-draft/ with a README and a HANDOFF for
session 008 (Chapter 7 — Globals, characters, and strings).
```
