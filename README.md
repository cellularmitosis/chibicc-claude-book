# chibicc-book (working dir)

A book about [rui314/chibicc](https://github.com/rui314/chibicc), authored entirely by Claude.

This is the project working directory. Nothing here is the book yet — it's research, planning, drafts, and the session log.

## Where to start

- **Reading the book:** [chapters/](chapters/) (currently `01-a-calculator.md`).
- **Reviewing the plan:** [book-plan.md](book-plan.md).
- **Picking up the project:** [CLAUDE.md](CLAUDE.md) → [docs/sessions/README.md](docs/sessions/README.md) → most recent session under `docs/sessions/`.

## Layout

```
.
├── README.md             — this file
├── CLAUDE.md             — project conventions, voice, authorship
├── book-plan.md          — master plan: TOC, voice, phasing, style guide
├── chapters/             — the book itself
│   └── 01-a-calculator.md
├── research/             — durable research artifacts
│   ├── sources/chibicc/  — cloned upstream repo (all branches)
│   ├── notes/            — sources, Rui quotes, JP book TOC
│   └── commits/          — full commit list + chapter mapping
└── docs/
    └── sessions/         — one dir per session, globally numbered
        ├── README.md     — session checklist
        ├── 001-research-and-plan/
        └── 002-chapter-01-draft/
            ├── README.md
            └── HANDOFF.md  — primer for next session
```

## Status

- **Phase 0 (research + plan):** complete. Session 001.
- **Phase 1 (Ch 1 pilot):** drafted, awaiting review. Session 002.
- **Workflow:** sessions live under `docs/sessions/NNN-<slug>/`. See [docs/sessions/README.md](docs/sessions/README.md) for the checklist.
