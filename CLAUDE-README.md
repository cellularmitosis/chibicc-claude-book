# chibicc-book (working dir)

A book about [rui314/chibicc](https://github.com/rui314/chibicc), authored entirely by Claude. This file is the maintainer/contributor view of the working directory; for the reader-facing entry point see [README.md](README.md).

## Where to start

- **Reading the book:** [chapters/](chapters/).
- **Reviewing the plan:** [book-plan.md](book-plan.md).
- **Picking up the project:** [CLAUDE.md](CLAUDE.md) → [docs/sessions/README.md](docs/sessions/README.md) → most recent session under `docs/sessions/`.

## Layout

```
.
├── README.md             — reader-facing entry point
├── CLAUDE-README.md      — this file (maintainer view)
├── CLAUDE.md             — project conventions, voice, authorship
├── book-plan.md          — master plan: TOC, voice, phasing, style guide
├── chapters/             — the book itself, 23 chapters
├── research/             — durable research artifacts
│   ├── sources/chibicc/  — cloned upstream repo (all branches)
│   ├── notes/            — sources, Rui quotes, JP book TOC
│   └── commits/          — full commit list + chapter mapping
└── docs/
    └── sessions/         — one dir per session, globally numbered
        ├── README.md     — session checklist
        └── NNN-<slug>/
            ├── README.md     — arrival → work → exit
            └── HANDOFF.md    — primer for next session (when work continues)
```

## Status

- **Phase 0 (research + plan):** complete. Session 001.
- **Phase 1 (Ch 1 pilot):** drafted. Session 002.
- **Phase 2 (bulk drafting):** complete. All 23 chapters / 316 commits walked. Sessions 003–024.
- **Phase 3 (review/revision/end-matter):** not started; user's call.
- **Workflow:** sessions live under `docs/sessions/NNN-<slug>/`. See [docs/sessions/README.md](docs/sessions/README.md) for the checklist.
