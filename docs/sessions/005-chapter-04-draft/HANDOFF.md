# Handoff: Ch 4 done → proceed to Ch 5

**For:** the next claude session.
**From:** session 005.
**Status:** Ch 4 drafted. Continue autonomously to Ch 5 (Functions). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/005-chapter-04-draft/README.md`](README.md)** — what session 005 did, including the three interpretive calls and the type-system-evolution notes.
2. **[`docs/sessions/004-chapter-03-draft/HANDOFF.md`](../004-chapter-03-draft/HANDOFF.md)** — the previous handoff. The autonomous-mode rules and standing notes still apply.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)**, **[`02-from-program-to-programs.md`](../../../chapters/02-from-program-to-programs.md)**, **[`03-statements-and-local-variables.md`](../../../chapters/03-statements-and-local-variables.md)**, **[`04-pointers.md`](../../../chapters/04-pointers.md)** — the four chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 5 = commits 23–26.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.

## Chapter 5 scope

**Title (working):** *Functions*.
**Commits:** 23–26 in chronological order on `main`.

| # | Hash | Subject |
|---|---|---|
| 23 | `30a3992` | Support zero-arity function calls |
| 24 | `964b1d2` | Support function call with up to 6 arguments |
| 25 | `6cb4220` | Support zero-arity function definition |
| 26 | `aacc0cf` | Support function definition up to 6 parameters |

**Concept interlude (per chapter mapping):** the SysV AMD64 *calling convention* — argument-passing registers (`%rdi, %rsi, %rdx, %rcx, %r8, %r9`), caller/callee responsibilities, the 16-byte alignment rule for the call instruction. The Ch 3 stack-frame interlude already mentioned 16-byte alignment as foreshadowing; here is where it pays off, because chibicc starts emitting `call` instructions for the first time. Place the interlude *between* §5.1 (zero-arity calls) and §5.2 (multi-arg calls) — the first `call` instruction lands in §5.1 and the alignment question becomes immediate, then §5.2 needs the argument-register convention explicitly.

The interlude should cover at minimum:
- The six argument registers in order, what GCC and Clang both follow.
- Caller-saved vs. callee-saved registers (chibicc treats almost everything as caller-saved by ignoring the question, but the reader should know what the actual rule is).
- The 16-byte stack alignment rule at the moment of `call` — and where chibicc enforces it (or doesn't).
- A brief note on what "up to 6 arguments" means and why chibicc caps it there (the SysV ABI's 7th-and-beyond-go-on-the-stack rule is something chibicc punts on).

## Steps

1. `cd research/sources/chibicc && for h in 30a3992 964b1d2 6cb4220 aacc0cf; do git show --stat $h; done` to scan diffs.
2. Read each commit in full (`git show <hash>`). The codegen.c changes here are the most interesting — pointer arithmetic was parser-only, but function calls are *fundamentally* a codegen feature.
3. Read the destination state at `aacc0cf` (chibicc.h, parse.c, codegen.c). Particularly: how does `Function` change to support multiple functions in one program? How does codegen emit a per-function prologue/epilogue?
4. Draft `chapters/05-functions.md`. Likely 6,000–9,000 words. The interlude on the calling convention can be substantial because there's real content to cover.
5. Write `docs/sessions/006-chapter-05-draft/README.md`.
6. Write `HANDOFF.md` for session 007 (Chapter 6 — Arrays, commits 27–31).

## Voice / structure rules

Same as Ch 1–4:
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
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; create it the first time something genuinely worth flagging shows up.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits; cross-check `chapter-mapping.md`.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- The Ch 5 interlude has the most prior-art on the web of any chapter so far (every C-compiler tutorial covers calling conventions); resist the urge to copy structure from external sources. Match the voice and frame from Ch 3's stack-frame interlude.

## Standing notes worth tracking across sessions

- **Ch 1 errata list** unchanged: test-count off-by-one (claims 28 at end, actual 27) and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`. Address in a revision pass.
- **The `Obj` type is still locals-only.** Chapter 5 will likely make `Function` and `Obj` start to converge (function definitions need names, types, and bodies — and `Obj` already has the first two). Worth watching for the moment they merge.
- **Canonicalization-at-parse-time** now has a fourth instance from Ch 4 (initializers desugar to declaration + assignment). Three instances may be enough to name-and-call-out the pattern in Chapter 5 or 6 — but only if the right moment presents itself. Don't force it.
- **The hardcoded `8` in `new_add`/`new_sub`** is destined to become `lhs->ty->base->size` in Chapter 6. The Ch 4 prose flagged this in passing. When Ch 6 makes the change, point back at the Ch 4 mention as foreshadowing earned.

## Acceptance criteria for Ch 5

- [ ] `chapters/05-functions.md` exists, end-to-end readable.
- [ ] Concept interlude on the calling convention lands between §5.1 and §5.2.
- [ ] Each section opens with `git checkout <full-hash>` and the commit's subject.
- [ ] Voice matches Ch 1–4.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/006-chapter-05-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 007.

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/005-chapter-04-draft/HANDOFF.md  (this handoff)
2. docs/sessions/005-chapter-04-draft/README.md   (what session 005 did)
3. chapters/01-a-calculator.md                     (template, voice)
4. chapters/02-from-program-to-programs.md
5. chapters/03-statements-and-local-variables.md
6. chapters/04-pointers.md                         (most recent chapter)
7. research/commits/chapter-mapping.md             (confirms Ch 5 scope)
8. CLAUDE.md and book-plan.md                      (conventions)

Then draft Chapter 5 (Functions, commits 23–26) per the steps in the
handoff. End-of-session: write your session dir under
docs/sessions/006-chapter-05-draft/ with a README and a HANDOFF for
session 007 (Chapter 6 — Arrays).
```
