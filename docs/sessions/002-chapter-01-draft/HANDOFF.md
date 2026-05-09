# Handoff: Ch 1 reviewed → revise or proceed to Ch 2

**For:** the next claude session.
**From:** session 002.
**Status:** Ch 1 drafted, awaiting user review. Workflow scaffolding in place. Two-path handoff depending on user feedback.

## Read these first, in order

1. **[`docs/sessions/002-chapter-01-draft/README.md`](README.md)** — what session 002 actually did, decisions made, open review questions.
2. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** — the chapter under review.
3. **The user's most recent message** in the current chat — likely review feedback on Ch 1, or "go" for Ch 2.
4. **[`book-plan.md`](../../../book-plan.md)** — master plan; check the "Style guide" section for voice rules.
5. **[`CLAUDE.md`](../../../CLAUDE.md)** — project conventions; especially the authorship section ("100% Claude-authored, not Rui's voice, not the user's").

## Two paths

### Path A — User asks for Ch 1 revisions

Six explicit review questions were surfaced. The user may answer some/all/none:

1. Length / density — currently ~8,600 words / ~30 pages. Compress to closer to Sandler density?
2. Voice — Sandler-like register OK?
3. Interlude length — BNF / recursive-descent interlude is ~3 pages. More? Less?
4. Diff format — mix of inline diffs and full code blocks OK, or strict-inline?
5. Opener stance — "compiler books begin with theory; chibicc starts with `42`" framing.
6. End-of-chapter recap table — useful or filler?

If the user gives directional feedback (e.g. "voice is too hand-holdy"), revise the chapter and re-surface for review. **Don't proceed to Ch 2 until Ch 1 is approved** — Ch 1 sets the template for everything that follows; downstream chapters will be expensive to retro-fit.

After revisions: write a fresh session dir for the revision pass (e.g. `003-chapter-01-revisions/`). Update `book-plan.md`'s style guide section if any voice / structure decisions change.

### Path B — User approves Ch 1 and says "do Ch 2"

Ch 2 is **one commit** — `725badf Split main.c into multiple small files`. It's the smallest chapter in the book by far. Mapping is in [`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md) under "Chapter 2."

Steps:

1. `cd research/sources/chibicc && git show 725badf` to see the diff.
2. Write `chapters/02-from-program-to-programs.md` (current working title; can adjust). Possibly very short (5–10 pages) — but don't artificially pad. A chapter about "we split one file into several" should feel proportionate.
3. Cover *why* this is a good moment to refactor: the codebase has just developed three internal phases (tokenize, parse, codegen) that map cleanly onto three files. Touching this just before adding the first language feature (variables) saves churn later.
4. Note the new files: `main.c`, `tokenize.c`, `parse.c`, `codegen.c`, `chibicc.h`. Discuss the header file's `extern` declarations and what the "module" boundary becomes.
5. Update the Makefile section.

Open structural question for Ch 2: it's small enough that **pairing it with Ch 3 in a single session might make sense** — Ch 3 is "Statements and local variables," 10 commits. Could be one combined session producing two chapter files. Use judgment based on session-time constraints; if pairing, slug the session dir `003-chapters-02-and-03/`.

### Path C — User wants something else entirely

Possibilities: revising the plan, expanding research, drafting the foreword, writing a sample marketing/landing page. Follow user direction; document the session as usual.

## State of the world

### Repo

- `main` branch tip: `90d1f7f` (2020-12-07). Stable for 5+ years; trust upstream hashes.
- Ch 1 covers commits `0522e2d` through `25b4b85`.
- Ch 2 covers commit `725badf` (commit #8 chronologically).

### Files in this repo

- `chapters/` — only `01-a-calculator.md` exists.
- `research/` — populated, complete for early chapters.
- `book-plan.md` — current; "open questions" section addressed.
- `docs/sessions/` — three things: `README.md` (workflow), `001-research-and-plan/`, `002-chapter-01-draft/` (this dir).

### What does NOT exist yet, in case relevant

- **No foreword.** Will need to be written eventually; not urgent. Should establish: this is Claude-authored, not Rui's, not the user's; how to read alongside the repo; how diffs work; what's covered and what isn't.
- **No companion repo.** Decided to trust upstream hashes (per book-plan).
- **No errata appendix.** Add when first errata-worthy thing is found during writing.
- **No overall TOC file.** The chapter mapping in `research/` is a planning doc; a reader-facing TOC will appear once we have more chapters.

## Pitfalls to avoid

- **Don't switch voice mid-chapter.** Ch 1 establishes "we" for reader-journey, "Rui" for design intent, third-person for everything else. New chapters should match.
- **Don't fix Rui's code in the prose.** If you find something surprising, flag it for an errata appendix (not yet created — start one when needed). Don't silently re-explain it as if it's cleaner than it is.
- **Don't invent features chibicc doesn't have.** Forward-references like "we'll come back to this when we add typedefs" should point to actual upcoming commits. Cross-check with [`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md) before promising the reader anything.
- **Don't pad short chapters.** Ch 2 is one commit. If the prose comes out at 8 pages, that's the right length. The book varies wildly per chapter and that's by design.
- **Authorship transparency.** Don't ventriloquize Rui. Don't speak as if you (Claude) are him. When explaining intent, cite his words from `research/notes/quotes-rui.md` or label it as our interpretation.

## Acceptance criteria for a chapter session

- [ ] Chapter file under `chapters/NN-<slug>.md` exists, end-to-end readable.
- [ ] Each section opens with `git checkout <short-hash>` and the commit's subject line.
- [ ] Voice matches Ch 1's register.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against the chapter mapping.
- [ ] `docs/sessions/NNN-<slug>/README.md` written with arrival → work → exit.
- [ ] Open questions for user surfaced in chat reply.
- [ ] If chapter scope or structure shifted, [`book-plan.md`](../../../book-plan.md) updated.

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. Read in order:

1. docs/sessions/002-chapter-01-draft/HANDOFF.md  (this handoff;
   has full context and two paths depending on user feedback)
2. docs/sessions/002-chapter-01-draft/README.md   (what session 002
   did and the open review questions)
3. chapters/01-a-calculator.md                     (the draft under review)
4. CLAUDE.md and docs/sessions/README.md           (project conventions)
5. book-plan.md                                    (master plan)

Then act on the user's most recent message:
- If they want Ch 1 revisions, take Path A.
- If they approve and want Ch 2, take Path B.
- Otherwise follow user direction.

End-of-session: write your session dir under docs/sessions/NNN-<slug>/
with arrival → work → exit, and a HANDOFF.md if work continues.
```
