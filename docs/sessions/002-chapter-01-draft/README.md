# Session 002 — Chapter 1 draft + workflow setup

**Date:** 2026-05-07 (same Claude conversation as session 001).
**Phase:** 1 (Ch 1 pilot).

## Arrival state

Phase 0 complete from session 001. Plan approved by user with four decisions resolved (markdown output, trust upstream hashes, inline diffs, content-driven length). Ready to draft Ch 1 ("A calculator," covering commits 1–7).

## What was done

### Chapter 1 draft

Examined all seven commits in detail with `git show <hash>` (full diffs captured in chat). Drafted [`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md) — ~8,600 words / ~30 printed pages.

Structure landed on:

- Brief chapter opener (framing: "most compiler books begin with theory; chibicc starts with `42`").
- §1.1–§1.4: commits 1–4 (single integer → `+ -` → tokenizer → caret errors).
- Concept interlude: BNF + recursive descent, ~3 pages, placed exactly where the JP book places it (between commits 4 and 5, before the parser is introduced).
- §1.5–§1.7: commits 5–7 (parser + AST + codegen → unary → comparisons).
- "Where we are" recap closing the chapter.
- LOC-growth recap table at the end.

### Decisions made during drafting

1. **Machine-language background folded into §1.1**, not made a standalone interlude. Rationale: a competent-programmer audience doesn't need a 30-page "what is assembly" chapter; the four-line program from commit 1 is enough surface area to explain AT&T syntax, function-call ABI, and "why emit text not bytes" inline. If later chapters need a more formal interlude we can always extract one.
2. **Inline diffs for small commits, full code blocks for the bigger commits (3 and 5).** Strict diffs would make the "AST + parser + codegen all introduced together" section impossible to read.
3. **No emoji, no callouts, no admonitions.** Per style guide.
4. **Cited Rui's README four times verbatim** where his words explain a choice better than our prose could.
5. **Ended with a recap table.** Considered cutting it as filler; kept it because LOC growth is a tangible "we built something" hit that pays off the chapter's framing.

### Workflow setup (this part of the session)

User asked for a sessions workflow analogous to `~/claude/llvm-7-darwin-ppc/`. Set up:

- [`CLAUDE.md`](../../../CLAUDE.md) — project conventions, voice, repo layout, sessions pointer.
- [`docs/sessions/README.md`](../README.md) — checklist + start/end-of-session ritual.
- This dir + the session 001 backfill.
- Forthcoming: [`HANDOFF.md`](HANDOFF.md) for the next session.

Workflow design notes:

- Mirrored llvm structure pretty closely: numbered session dirs, README + optional HANDOFF/findings/commits, project-level CLAUDE.md.
- Did NOT copy the "Release workflow" section — books don't ship like compilers. Kept "Document everything" verbatim in spirit.
- Did NOT add "Unsupervised mode" with full risk-tolerance discussion — book authoring is collaborative; the user reviews each chapter. If unsupervised work happens later, we can add the section then.
- `research/` stays at top level rather than moving under `docs/`. It's a stable knowledge base, not a session artifact, and the top-level position makes it easy to reach.

## Open questions surfaced for user (Ch 1 review)

Six items, listed at the end of the chat reply summarizing the chapter:

1. Length / density — ~30 pages OK or compress?
2. Voice — Sandler-like register OK?
3. Interlude length — ~3 pages enough?
4. Diff format — mix of inline and full code OK?
5. The opener stance — "compiler books begin with theory" framing land OK?
6. The recap table — useful or filler?

## Exit state

- `chapters/01-a-calculator.md` drafted, ~8,600 words.
- Sessions workflow established under `docs/sessions/`.
- `CLAUDE.md` written.
- Six review questions surfaced for the user.
- Next session waits on user feedback: either ch1 revisions or proceed to ch2 (commit 8 only — `Split main.c into multiple small files`).

`HANDOFF.md` in this dir primes the next session.
