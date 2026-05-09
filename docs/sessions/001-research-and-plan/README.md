# Session 001 — Research and plan

**Date:** 2026-05-07.
**Phase:** 0 (research + plan).
**Note:** This README is a retroactive write-up — sessions 001 and 002 happened in the same Claude conversation, before the sessions workflow was introduced. The split into 001 (research/plan) and 002 (chapter 1 draft + workflow setup) reflects the natural phase boundary, not separate Claude conversations.

## Arrival state

Empty working directory at `/Users/cell/claude/chibicc-book/`. User context: rui314 wrote chibicc as a teaching compiler with the explicit intent to write a book using the commit history as a curriculum, but never finished the book (he's been busy with mold). The user wants Claude to attempt that book.

User direction at session start:

- Research in depth, document findings and sources.
- Draft a plan, then iterate per feedback.
- Use this dir as scratch.
- Foreword will be transparent: 100% Claude-authored, not Rui's voice, not the user's.
- Audience: competent programmer (Sandler-equivalent).
- Language: English.
- Reference materials available locally: `~/prog/_compilers/` has dozens of compiler textbooks; `~/junk/compiler/2025/ch1/` has the user's prior Sandler ch1 attempt.

## What was done

### Research

Established sources (full bibliography in [`research/notes/sources.md`](../../../research/notes/sources.md)):

- **chibicc repo** cloned to `research/sources/chibicc/`. Confirmed: 316 commits on `main` (2019-08-03 → 2020-12-07), with three other branches (`historical/old` 115 commits, `new` 304 commits, `reference` 93 commits).
- **The chibicc README** turned out to be the single most informative source — Rui states "Each commit of this project corresponds to a section of the book," credits Ghuloum's incremental paper, and articulates design principles (calloc-and-never-free; "be dumb on purpose"; small duplications fine; no premature optimization).
- **Rui's existing Japanese book** at https://www.sigbus.info/compilerbook — discovered indirectly via pokotsun's reimplementation. Title: *低レイヤを知りたい人のためのCコンパイラ作成入門*. Half-written; covers ~28 steps of an older chibicc, then explicitly stops at "Step 29 and beyond `[要加筆]` (to be written)." Maps to roughly the first 50 commits of the new chibicc.
- **HN threads** (2020-09 #24676851, 2022-11 #33581704) — Rui confirms the English book is on hold because he's working on mold.
- **Reddit announcement** — Claude can't fetch reddit/web.archive.org. Acceptable gap; README + HN cover the substantive content.
- **GitHub issue #78** ("The book?") — reader asking; Rui never replied in the visible thread.

### Synthesized findings

Captured in three notes files:

- [`research/notes/sources.md`](../../../research/notes/sources.md) — full bibliography.
- [`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md) — verbatim Rui statements that constrain authorial choices. Key ones: "each commit corresponds to a section," "I wrote this code not for me but for first-time readers," "small duplications instead of clever abstractions," "calloc and never free."
- [`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md) — translated TOC of the JP book. Critical structural insight: Rui interleaves *concept* chapters (machine language, integer representation, executable image, linking, type syntax) between *step* (implementation) chapters. Ratio in JP book is roughly 1 concept chapter per 2 implementation chapters.

### Commit mapping

Walked all 316 commits, grouped them by feature arc, produced [`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md): a 23-chapter, 4-part proposal. Climax: self-hosting at commit 197 (end of the preprocessor chapter). Notable arc lengths: 40-commit preprocessor (Ch 17), 17-commit floating-point (Ch 15), 24-commit unicode+designated-init (Ch 19).

### Master plan

Wrote [`book-plan.md`](../../../book-plan.md) covering:

- What the book is and isn't.
- Structural decisions (sections=commits, chapters=groups, concept interludes interleaved, third-person voice, inline diffs, ~25 pages/chapter target).
- Phasing: pilot Ch 1 → review → bulk drafting in batches.
- Risks (force-push of upstream, voice drift, preprocessor-chapter difficulty).
- Style guide first cut.
- Open questions for the user.

## Decisions made (noting them so they're not forgotten)

1. **Section unit = commit.** Backed by Rui's explicit README statement.
2. **Chapter unit = themed group of commits.** Mirrors Rui's JP book grouping.
3. **Concept chapters are first-class**, interleaved just-in-time. Mirrors JP book pattern. 23 implementation chapters + ~6 concept-interlude slots is the working count.
4. **Voice**: third-person, "we" for reader-journey, "Rui" for design intent. Authorial transparency about being Claude-written.
5. **Audience**: Sandler-equivalent (competent programmers, no compiler background).
6. **Self-hosting is the structural climax**, falling at end of Part III.
7. **The Japanese book is the structural template for ~the first 50 commits** but not a translation source. Everything past JP step 28 is uncharted.

## Open questions surfaced for user

1. Output format (Markdown vs. richer markup)
2. Companion repo strategy (mirror vs. trust upstream hashes)
3. Diff format (inline vs. side-by-side)
4. Length target (~25 pp/chapter, ~600 pp total)
5. Anything missing from plan

## Exit state

Research artifacts captured in `research/`. Master plan in `book-plan.md`. Working dir layout established. Awaiting user response on the five open questions before starting Phase 1 (Ch 1 pilot).

## Resolutions received from user (between session 001 and 002)

User responded:

- **Format:** plain Markdown.
- **Companion repo:** trust upstream hashes (last commit was 6 years ago); revisit if needed.
- **Diff format:** inline unified.
- **Length:** content-driven; chapters can vary widely.
- **Approach approved.** Proceed to Ch 1 pilot.
