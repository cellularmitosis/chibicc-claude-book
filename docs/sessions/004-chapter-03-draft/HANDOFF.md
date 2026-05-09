# Handoff: Ch 3 done → proceed to Ch 4

**For:** the next claude session.
**From:** session 004.
**Status:** Ch 3 drafted. Continue autonomously to Ch 4 (Pointers). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/004-chapter-03-draft/README.md`](README.md)** — what session 004 did, including drafting decisions and a note about Ch 1 errata to address later.
2. **[`docs/sessions/003-chapter-02-draft/HANDOFF.md`](../003-chapter-02-draft/HANDOFF.md)** — the previous handoff. The "what user wants" section there still applies: autonomous, no per-chapter review.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)**, **[`02-from-program-to-programs.md`](../../../chapters/02-from-program-to-programs.md)**, and **[`03-statements-and-local-variables.md`](../../../chapters/03-statements-and-local-variables.md)** — the three chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 4 = commits 19–22.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes; pull from here when citing design intent.

## Chapter 4 scope

**Title (working):** *Pointers*.
**Commits:** 19–22 in chronological order on `main`.

| # | Hash | Subject |
|---|---|---|
| 19 | `3d86277` | Add a representative node to each Node to improve error messages |
| 20 | `863e2b8` | Add unary `&` and `*` |
| 21 | `a6bc4ab` | Make pointer arithmetic work |
| 22 | `b4e82cf` | Add keyword "int" and make variable definition mandatory |

**Concept interlude (per chapter mapping):** *"what a type is, and why the parser now needs to track them."* This is the moment chibicc has to start tracking types in the AST — pointer arithmetic semantics differ from integer arithmetic semantics (`p + 1` advances `p` by `sizeof(*p)`, not by 1), and the parser has no way to know that without a type system. Place the interlude *before* §4.3 (the pointer-arithmetic commit), because §4.3 is where chibicc first peeks at a node's type to decide what to emit.

The interlude should cover at minimum:
- What "type" means in a compiler context: a static label attached to expressions/values that tells the parser/codegen what operations are legal and what their effects should be.
- Why C's type system is more than a labeling convention: pointer arithmetic, sizeof, function-call argument promotion all depend on it.
- Foreshadow the *very* limited type system chibicc has at this point: only `int` and `pointer to int`, then `pointer to pointer to int`, etc. No `char`, no struct, no function types yet — those arrive much later.

## Steps

1. `cd research/sources/chibicc && for h in 3d86277 863e2b8 a6bc4ab b4e82cf; do git show --stat $h; done` to scan diffs.
2. Read the destination state (chibicc.h, parse.c, codegen.c, tokenize.c at `b4e82cf`) to know where the chapter lands. Particularly: how does Type get represented? Where does add_type() live? How does the parser attach a Type to each Node?
3. Draft `chapters/04-pointers.md`. Likely 5,000–8,000 words.
4. Write `docs/sessions/005-chapter-04-draft/README.md`.
5. Write `HANDOFF.md` for session 006 (Chapter 5 — Functions, commits 23–26).

## Voice / structure rules

Same as Ch 1–3:
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
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; create `chapters/appendix-d-errata.md` (or similar) the first time something genuinely worth flagging shows up.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits; cross-check `chapter-mapping.md`.
- Don't pad. Ch 4 is shorter than Ch 3 (4 commits vs 9); the prose should reflect that. The concept interlude can absorb some length, but only if it's earning its keep.
- Authorship transparency: don't ventriloquize Rui.

## Standing notes worth tracking across sessions

- **Ch 1 has at least one off-by-one in test counts** (claims 28 at end, actual 27) and a missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`. Add to a revision-pass list. Don't fix mid-stream.
- **The `Obj` type was introduced for locals only** but is named generically. When future commits make it cover globals/functions/string-literals, circle back.
- **Canonicalization-at-parse-time** is now a recurring pattern (`>` → swapped `<`; `while` → degenerate `for`). Future chapters will have more (e.g., `do…while`, compound assignments, `[]` indexing → `*(p+i)`). Consider whether to call it out as a named pattern when it next appears, or keep treating each instance as local color.

## Acceptance criteria for Ch 4

- [ ] `chapters/04-pointers.md` exists, end-to-end readable.
- [ ] Concept interlude on types lands somewhere it makes sense (probably between §4.2 and §4.3).
- [ ] Each section opens with `git checkout <full-hash>` and the commit's subject.
- [ ] Voice matches Ch 1–3.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/005-chapter-04-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 006.

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/004-chapter-03-draft/HANDOFF.md  (this handoff)
2. docs/sessions/004-chapter-03-draft/README.md   (what session 004 did)
3. chapters/01-a-calculator.md                     (template, voice)
4. chapters/02-from-program-to-programs.md
5. chapters/03-statements-and-local-variables.md   (most recent chapter)
6. research/commits/chapter-mapping.md             (confirms Ch 4 scope)
7. CLAUDE.md and book-plan.md                      (conventions)

Then draft Chapter 4 (Pointers, commits 19–22) per the steps in the
handoff. End-of-session: write your session dir under
docs/sessions/005-chapter-04-draft/ with a README and a HANDOFF for
session 006 (Chapter 5 — Functions).
```
