# chibicc-book

A book about [rui314/chibicc](https://github.com/rui314/chibicc) — Rui Ueyama's small C compiler. The book walks the repository commit by commit, in the order Rui wrote them, with prose connecting each commit to the next.

Plan: [book-plan.md](book-plan.md). Chapter→commit mapping: [research/commits/chapter-mapping.md](research/commits/chapter-mapping.md).

## Authorship

The book is being written **entirely by Claude** (Anthropic), with the project owner directing scope, approving each chapter, and steering revisions. Rui Ueyama is not a co-author and has no involvement. The foreword will state this explicitly.

We follow Rui's pedagogical structure (his README, his commit ordering, the TOC of his existing Japanese book at https://www.sigbus.info/compilerbook) but the prose is original. Where we explain *why* a design choice was made, we either cite Rui's own words ([research/notes/quotes-rui.md](research/notes/quotes-rui.md)) or label it as our interpretation.

## Repo layout

```
.
├── README.md             — project entry, "what is this"
├── CLAUDE.md             — this file
├── book-plan.md          — master plan: audience, voice, phasing, style guide
├── chapters/             — the book itself, one .md per chapter
│   └── 01-a-calculator.md
├── research/             — durable research artifacts
│   ├── sources/chibicc/  — local clone of upstream repo (all branches)
│   ├── notes/            — sources, verbatim Rui quotes, JP book TOC, etc.
│   └── commits/          — full commit list + chapter mapping
└── docs/
    └── sessions/         — one dir per session, globally numbered
        ├── README.md     — session checklist
        └── NNN-<slug>/
            ├── README.md     — arrival → work → exit
            ├── HANDOFF.md    — primer for next session (when work continues)
            └── findings.md   — durable lessons (optional)
```

## Workflow

The project runs in **session units**. One Claude conversation produces one session, recorded under `docs/sessions/NNN-<slug>/`. The default cadence is **one chapter per session**, plus dedicated sessions for research, replanning, and revision passes when needed.

The full session lifecycle is in [docs/sessions/README.md](docs/sessions/README.md). A session ends with a written `README.md` summarizing arrival → work → exit, and (when there's specific unfinished work to pass) a `HANDOFF.md` priming the next session.

## Document everything

Default to capturing liberally in the active session directory. Plans, decisions, dead ends, voice/structure experiments, "I almost wrote X but did Y because Z," scope shifts, things-I'd-do-differently — all worth writing down. The point is that a future Claude session can pick up the thread without the user having to re-explain. **Err toward more capture, not less.**

## Voice and style — first cut

(See [book-plan.md § "Style guide — first cut"](book-plan.md) for the full version.)

- **Audience:** competent programmers, no compiler background assumed. Comparable to Sandler's *Writing a C Compiler*.
- **Person:** "we" for the reader's journey ("we've added a tokenizer, so let's…"). "Rui" (first name, his preferred form) for design intent. Third-person for everything else. Avoid first-person singular — there is no narrator-figure.
- **Tense:** past tense for what a commit *did*, present tense for current behavior.
- **Honesty:** explicit when interpreting. "Rui doesn't say why he chose X here, but a likely reason is Y."
- **No filler.** No emoji, no callout boxes, no "tip" / "warning" / "info" admonitions. If something matters, it goes in the prose.
- **Don't fix Rui's code in the prose.** If we find something surprising, flag it for the (TBD) errata appendix — don't silently re-explain it as if it's cleaner than it is.

## Working with the chibicc repo

- Clone is at `research/sources/chibicc/`. All branches present (`main`, `historical/old`, `new`, `reference`).
- `main` is canonical (316 commits, 2019-08-03 → 2020-12-07).
- We **trust upstream commit hashes** — Rui's last commit was 2020-12, force-push risk is judged low after 5+ years of stability. We pin chapter sections to short hashes (e.g. `0522e2d`) and don't mirror.
- The reader is expected to `git checkout <hash>` as they read along; each chapter section opens with the checkout command.

## Research base

- [research/notes/sources.md](research/notes/sources.md) — bibliography
- [research/notes/quotes-rui.md](research/notes/quotes-rui.md) — verbatim Rui quotes that constrain authorial choices
- [research/notes/japanese-book-toc.md](research/notes/japanese-book-toc.md) — TOC of Rui's existing JP book (structural template for early chapters)
- [research/commits/main-commits.txt](research/commits/main-commits.txt) — all 316 commits
- [research/commits/chapter-mapping.md](research/commits/chapter-mapping.md) — proposed chapter→commit mapping

## Status

Phase 0 (research + plan): complete (session 001).
Phase 1 (Chapter 1 pilot): drafted (session 002). User opted not to review chapter-by-chapter; will evaluate the whole book at the end.
Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017). Continuing autonomously per user direction.
