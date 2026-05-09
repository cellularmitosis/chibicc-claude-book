# Handoff: Ch 2 done → proceed to Ch 3

**For:** the next claude session.
**From:** session 003.
**Status:** Ch 2 drafted. User has asked for autonomous progress until the book is done — do not stop for per-chapter review. Continue with Ch 3.

## Read these first, in order

1. **[`docs/sessions/003-chapter-02-draft/README.md`](README.md)** — what session 003 did and the conventions established.
2. **[`docs/sessions/002-chapter-01-draft/HANDOFF.md`](../002-chapter-01-draft/HANDOFF.md)** — the previous handoff. Pitfalls and acceptance criteria still apply.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** and **[`chapters/02-from-program-to-programs.md`](../../../chapters/02-from-program-to-programs.md)** — the two chapters drafted so far. Match their register before writing.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 3 = commits 9–18.
5. **[`book-plan.md`](../../../book-plan.md)** — master plan; style guide.
6. **[`CLAUDE.md`](../../../CLAUDE.md)** — project conventions.

## What user wants

Quoted from the chat preceding session 003: *"I'm going to just trust your judgement and evaluate the final result when we are [done] with the book. I have plenty of token budget for it but am short on time I can give for attention."*

**Implications:**

- Proceed without surfacing review questions. Don't pause between chapters waiting for approval.
- Open six Ch 1 review questions (length, voice, interlude length, diff format, opener stance, recap table) are dismissed-by-default; if at any point one of them creates a tension worth flagging, note it in the session README but don't block on it.
- Eventually Ch 1 will need a revision pass to harmonize with the rest of the book. Defer that until a few more chapters are drafted and patterns are clearer.

## Chapter 3 scope

**Title (working):** *Statements and local variables*.
**Commits:** 9–18 in chronological order on `main`.

| # | Hash | Subject |
|---|---|---|
| 9 | `76cae0a` | Accept multiple statements separated by semicolons |
| 10 | `1f9f3ad` | Support single-letter local variables |
| 11 | `482c26b` | Support multi-letter local variables |
| 12 | `6cc1c1f` | Add "return" statement |
| 13 | `18ac283` | Add `{ ... }` |
| 14 | `ff8912c` | Add null statement |
| 15 | `72b8415` | Add "if" statement |
| 16 | `f5d480f` | Add "for" statement |
| 17 | `1f3eb34` | Add "while" statement |
| 18 | `5b142b1` | Add LICENSE and README.md |

Commit 18 is administrative — handle it briefly (or skip it from the "feature" prose and just note it in the recap table). Don't pad.

**Concept interlude:** the chapter mapping flags one for Ch 3 — *"how the System V AMD64 ABI lays out a stack frame."* Place it where chibicc first computes stack-frame size, which is the first locals commit (`1f9f3ad`) — that's where the prologue/epilogue dance becomes load-bearing. The interlude needs to cover: `%rbp`/`%rsp`, push/pop conventions, why every function reserves space at entry and releases it at exit, why locals are addressed as negative offsets from `%rbp`, and the 16-byte alignment rule (relevant to a much later commit but worth foreshadowing in a sentence).

## Steps

1. `cd research/sources/chibicc && for h in 76cae0a 1f9f3ad 482c26b 6cc1c1f 18ac283 ff8912c 72b8415 f5d480f 1f3eb34 5b142b1; do git show --stat $h; done` to scan the diffs.
2. Read the three core files (`tokenize.c`, `parse.c`, `codegen.c`) at `1f3eb34` (end of chapter) to know the destination state.
3. Draft `chapters/03-statements-and-local-variables.md`. Likely 6,000–10,000 words depending on how the interlude lands.
4. Write `docs/sessions/004-chapter-03-draft/README.md` with arrival → work → exit.
5. Write `HANDOFF.md` for session 005 (Chapter 4 — Pointers).

## Voice / structure rules to keep matching

- Section opens with `git checkout <full-hash>` and the commit subject as a blockquote.
- "we" for reader-journey, "Rui" for design intent, past for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- Each section ends with a "where we are" closer (Ch 1 used these per-section; Ch 2 used one at the chapter level since it was one commit. For Ch 3, use per-section).
- Diff format: lean toward inline unified diffs for short changes, full file blocks for the bigger ones. Don't be religious about it.
- Concept interlude: third-person, no `git checkout`, ~2–4 pages. Place between commits, not within.

## Pitfalls to avoid

(Carried forward from session 002's HANDOFF.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. Flag surprises for a (still-not-yet-created) errata appendix; create it the first time you actually need it.
- Don't invent features chibicc doesn't have. Forward references must point at actual upcoming commits.
- Don't pad. If something doesn't need explanation, don't explain it.
- Authorship transparency. Don't ventriloquize Rui.

## Acceptance criteria for Ch 3

- [ ] `chapters/03-statements-and-local-variables.md` exists, end-to-end readable.
- [ ] Each section opens with `git checkout <full-hash>` and the commit's subject.
- [ ] Voice matches Ch 1 and Ch 2.
- [ ] Stack-frame interlude lands somewhere it makes sense (probably between §3.1 and §3.2 — i.e. between the semicolons commit and the first locals commit).
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/004-chapter-03-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 005 (Chapter 4 — Pointers, commits 19–22).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/003-chapter-02-draft/HANDOFF.md  (this handoff)
2. docs/sessions/003-chapter-02-draft/README.md   (what session 003 did)
3. chapters/01-a-calculator.md                     (template, voice)
4. chapters/02-from-program-to-programs.md         (most recent chapter)
5. research/commits/chapter-mapping.md             (confirms Ch 3 scope)
6. CLAUDE.md and book-plan.md                      (conventions)

Then draft Chapter 3 (Statements and local variables, commits 9–18) per
the steps in the handoff. End-of-session: write your session dir under
docs/sessions/004-chapter-03-draft/ with a README and a HANDOFF for the
next session (Chapter 4 — Pointers).
```
