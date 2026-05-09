# Session 003 — Chapter 2 draft

**Date:** 2026-05-09.
**Phase:** 2 (bulk drafting).

## Arrival state

User trusts judgment on the six Ch 1 review questions and wants to proceed without per-chapter review. Ch 1 stands as drafted; user will evaluate the whole book at the end. Workflow scaffolding (`CLAUDE.md`, `docs/sessions/`) was settled in session 002. Ch 2 is a single commit — `725badf` — splitting `main.c` into modules.

## What was done

### Drafting decisions

- **Did not pair Ch 2 with Ch 3 in this session.** The handoff suggested it as an option. Declined for one reason: Ch 1 was the prose template; Ch 2 should establish that the template transfers cleanly to a one-commit chapter before bundling. If something is off about Ch 2's voice or pacing, it'll be cheaper to fix in isolation than tangled with a 10-commit Ch 3 draft.
- **Length:** ~2,650 words / ~9 pages. Ch 1 was ~8,600 / ~30 pages over 7 commits, so ~1,200 words/commit there. Ch 2 is one commit of pure refactoring — so by raw commit count it should be ~1,200 words, but a refactor commit needs more *structural* explanation (what each file owns, why split here, the Makefile changes) than a feature commit. ~2,650 felt natural without padding.
- **No concept interludes.** Nothing called for one. The next interlude candidate is "how the System V AMD64 ABI lays out a stack frame" in Ch 3.
- **Mentioned the date oddity once.** Commit `725badf` is dated 2020-10-07 but the next commit on `main` is dated 2020-09-26. Rui rewrote history to put the modular split first in the curriculum even though it was typed later. Worth flagging once — it teaches the reader that the curriculum order Rui chose ≠ wall-clock order, and that the `git log` dates aren't load-bearing. Did not dwell.
- **Forward references** point at Ch 3's actual commits (9–17 are features, 18 is a LICENSE/README admin commit not listed in the forward-reference list). Cross-checked against `chapter-mapping.md` and the actual commit list.

### One factual fix mid-draft

Initially wrote a paragraph claiming this commit upgraded `skip()` from `error()` to `error_tok()` as a side-effect of the move. Verified against `git show <hash>:main.c` for each Ch 1 commit and found `skip` was actually upgraded back in commit `cc5a6d9` (Ch 1 §1.4). Removed the paragraph. Note: Ch 1's prose at §1.3 shows the original `error()` version of `skip` and never explicitly mentions §1.4 changing it — possibly worth a one-sentence addition to Ch 1 in a future revision pass, but not urgent.

### Voice / structure conventions inherited from Ch 1

- Section opens with `git checkout <full-hash>` and the commit subject.
- "we" for reader-journey, "Rui" for design intent, third-person for everything else.
- Past tense for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- "Where we are" at the end.
- Cited Rui's README once (the bug-free-history quote, supporting the date-reordering observation).
- Diff-format choice: a stat line at the top, then quoted file fragments inline rather than full unified diffs. Felt right for a refactor commit where the *boundaries* are the story, not the per-line changes. (Ch 1 used inline unified diffs for small commits and full code blocks for larger ones; this is a third pattern. Probably fine — acceptance criterion is "voice matches Ch 1", which I read as register-and-discipline matching, not formatting being identical.)

## Open questions surfaced for user

None — user explicitly asked for autonomous progress.

## Exit state

- `chapters/02-from-program-to-programs.md` drafted, ~2,650 words.
- Session 003 dir populated.
- HANDOFF.md primes session 004 (Chapter 3).
- Next session: Chapter 3 — Statements and local variables, commits 9–17 (10 features) + commit 18 (administrative). Stack frames, prologue/epilogue, the first concept interlude.
