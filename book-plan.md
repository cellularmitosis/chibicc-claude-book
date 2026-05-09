# Plan: writing the chibicc book

**Working title:** *chibicc: A Small C Compiler, Commit by Commit*
**Authorship:** Authored entirely by Claude (Anthropic). Reviewed and directed by the project owner. **Not written by Rui Ueyama**, and not a translation of his Japanese book — though it follows his repository, his stated intent, and the structural template of his predecessor book.
**Audience:** Competent programmers (Sandler-equivalent target). Comfortable with C; do not assume any compiler background.
**Length target:** ~600 pages, 23 implementation chapters + foreword/afterword/appendices. Estimated based on Sandler-comparable density (~25 pages/chapter).
**Reference compiler:** `rui314/chibicc` `main` branch, 316 commits, 2019-08-03 → 2020-12-07.

---

## What this book is

A guided walkthrough of every commit in chibicc — Rui Ueyama's small C compiler — in the order he wrote them. The book treats the commit history as a curriculum: each section corresponds to one commit, and chapters group commits that share a theme (a feature being built up, an arc through the type system, the preprocessor's incremental construction). Concept chapters punctuate the implementation chapters, providing background on machine code, ABI, integer representation, linking, and the C type system at the moment readers need each topic.

The destination is **self-hosting**: by Chapter 17 the compiler builds itself, and the remaining chapters extend it to compile real third-party programs (sqlite, libpng, git, cpython).

## What this book is not

- **Not a textbook on compiler theory.** No DFA construction, no LR parsing, no SSA, no register allocation. We follow what chibicc does, and chibicc is a hand-written recursive-descent compiler with naive code generation.
- **Not an opinionated essay on language design.** We're documenting a real C compiler that exists, not arguing about how C should be implemented.
- **Not a translation of Rui's Japanese book.** That book covers ~28 steps of an *older* chibicc (the `historical/old` branch). This book covers all 316 commits of the `main` branch, including the preprocessor and self-hosting that the Japanese book never reached.
- **Not co-authored or endorsed by Rui.** We are inferring intent from the repo, README, and his public comments. The foreword will be explicit about this.

## Why we think we can write it

- **Rui designed the curriculum.** The repo is engineered for exactly this kind of book — every commit is a teachable unit, and Rui has stated this explicitly. We are filling in prose around an existing skeleton, not inventing a curriculum.
- **The early chapters have a working template.** Rui's Japanese book covers commits 1–~50 in roughly the chapter structure we need. We can mirror its chapter boundaries and concept-chapter placements for that range.
- **The compiler is small, readable, and bug-free by construction.** Rui rewrites history to keep every commit bug-free; we don't have to apologize for past mistakes or explain corrected detours.

---

## Structural decisions (from the research phase)

| Decision | Choice | Why |
|---|---|---|
| **Section unit** | One commit = one section | Rui's explicit intent. Confirmed by README and the JP book. |
| **Chapter unit** | A coherent group of commits sharing a theme | What Rui did in the JP book. ~10–25 sections per chapter. |
| **Concept chapters** | Yes, interleaved just-in-time | What Rui did in the JP book — JP has ~6 concept chapters spread through ~28 step chapters. |
| **Voice** | Third-person, "we" inclusive of reader | Standard for compiler books. Avoids ventriloquizing Rui. |
| **Code presentation** | Show diffs (unified or before/after side-by-side) for each commit | Mirrors how the reader would explore the repo themselves. |
| **Companion repo** | Reader checks out commits as they read | Built into the book's flow: each section opens with `git checkout <hash>`. |
| **Book's git policy** | Pin a specific chibicc commit hash per section. Rui has warned he may force-push, so we cite immutable hashes, not branch names. | Rui's contributor note says "I will occasionally force-push." |

---

## Top-of-book scaffolding (need to author up front)

These are *not* chapter content but they govern the whole book. They should be drafted before Chapter 1, even if briefly:

1. **The companion repo plan.** Mirror chibicc to a stable fork we control? Or rely on commit hashes alone? My recommendation: a stable mirror at e.g. `claude/chibicc-book-companion` with a tag per section. Worth discussing.
2. **The "how to read this book" preface.** Tells readers to keep `chibicc` checked out alongside, and walks them through the first `git checkout` of commit 1.
3. **The diff-presentation convention.** What format do diffs appear in? Inline unified diff with line-level highlighting? Side-by-side? My recommendation: unified diff for short changes (<20 lines), full file blocks for longer ones, with annotations in the margin or as numbered call-outs.
4. **Cross-reference convention.** Forward-references to later chapters happen often (e.g., "we'll come back to this when we add typedefs in Chapter 10"). Decide format now.
5. **The errata appendix policy.** As we write, we will likely find places where chibicc's behavior surprised us or where a commit doesn't quite do what its message says. Errata appendix collects these. Don't silently fix Rui's code in the prose — flag it.

---

## Phasing the work

### Phase 0 (complete) — Research and plan
- Research the repository, README, Japanese book, community discussion, and Rui's stated intent.
- Map all 316 commits to a proposed chapter structure (`research/commits/chapter-mapping.md`).
- Output: this document and the `research/` folder.

### Phase 1 — Pilot Chapter 1 (next step)
**Goal:** write Chapter 1 ("A calculator") in full as a quality reference. ~25 pages.
**Why first:** it's small, self-contained, and its commits closely mirror the Japanese book's first chapter — so we can validate that our voice, structure, and conventions feel right before committing to 22 more chapters.
**Includes:** all up-front decisions (diff format, code presentation, callout style, cross-reference style).
**Deliverable:** `chapters/01-a-calculator.md` (or whatever format we settle on).

### Phase 2 — Iterate based on Chapter 1
**Goal:** review Phase 1 output. Adjust:
- Voice (too academic? too casual?).
- Pacing (too dense? too thin?).
- Diff presentation (does it work in the chosen output format?).
- Concept-interlude placement (do they break flow or aid it?).

Update this plan and the chapter-mapping doc with revisions before Phase 3.

### Phase 3 — Bulk drafting
**Goal:** draft chapters 2–6 (Part I — "A working compiler in 30 commits").
**Why this batch:** completing Part I yields a self-contained, ~150-page mini-book that already demonstrates a working calculator-with-functions. This is a natural milestone for a second review pass.

### Phase 4 — Drafting Parts II, III, IV
Continue chapter by chapter. Periodically revisit the plan.

### Phase 5 — Polish, errata, technical review
- Compile each chapter against the cited commit (literally check that the diff in the prose matches the commit).
- Fill in cross-references that were stubbed.
- Errata appendix.
- Solicit a technical reviewer (someone who has read chibicc end-to-end).

### Phase 6 (optional) — Production format
PDF? Web? Markdown-as-source with multiple build targets? Defer.

---

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| **Rui force-pushes the chibicc repo** and our commit hashes go stale. | Mirror chibicc to a stable fork; cite hashes from there. The current `main` head `90d1f7f` from 2020-12-07 has been stable for 5+ years, so this is unlikely but not impossible. |
| **Some commits do something the message doesn't quite describe** (or something subtle that's hard to explain). | Flag in errata. Don't pretend the commit does something cleaner than it does. |
| **The preprocessor chapter (Ch 17) is genuinely hard.** ~40 commits, complex semantics, many "don't expand twice" subtleties. | Plan for it taking 2-3× the writing time of an average chapter. Consider whether to split into multiple chapters during Phase 4. |
| **Voice drift across chapters** as the project spans many sessions. | Establish a tight style guide after Chapter 1 lands. Re-read the previous chapter's last 3 pages before starting each new chapter. |
| **Writing in Claude's voice over months means inconsistent perspective.** | The foreword fixes the voice ("written by Claude, third-person, no fictional narrator"). Style guide enforces it. |
| **Reader fatigue with 316 sections.** | Each chapter ends with a "where we are now" recap. Each Part ends with a milestone (a working calculator → a recognizable C → self-hosting → real-world programs). |
| **Misrepresenting Rui's intent.** | The foreword is explicit: this is *our* reading of his code. Where we explain *why* a design choice was made, we either cite Rui's own words (`research/notes/quotes-rui.md`) or label our interpretation as such. |

---

## Style guide — first cut (revisit after Phase 1)

### Voice
- **First-person plural ("we") for the reader's journey.** "We've added a tokenizer, so let's…"
- **Third-person ("Rui", "the compiler") for everything else.** Avoid "I" — there is no narrator-figure.
- **Past tense for what the commit did, present tense for current behavior.** "Commit 5 added support for `*` and `/`. The parser now handles operator precedence by…"
- **Explicit when interpreting.** "Rui doesn't say why he chose X here, but a likely reason is Y."

### Code
- Each section opens with `git checkout <short-hash>` and the commit message verbatim.
- Diffs in unified format for changes under ~20 lines; new files shown in full.
- Highlight added/changed lines with marginal callouts for the interesting bits.
- Never abridge code without saying so.

### Cross-references
- Forward: "we'll address this when we get to typedefs (§10.4)."
- Backward: "as in §3.2."
- Always cite by section number, not page.

### Concepts
- A concept interlude is its own chapter, not a sidebar within an implementation chapter.
- Length target: 8–15 pages.
- Always answer: "why are we pausing here?" in the first paragraph.

### Acknowledgement of Rui throughout
- Use "Rui" (not "Ueyama" — he goes by his first name in his own writing) when explaining design intent.
- Be honest about what's interpretation vs. what's documented.

---

## Immediate next step (proposed)

Write **Chapter 1: A calculator** in full as a pilot. Treat it as a Phase 1 deliverable: about 25 pages of prose, covering commits 1–7 (single integer through comparison operators), with the two concept interludes (machine language; recursive-descent parsing) interleaved. Output as a single Markdown file.

After Chapter 1, we pause, review, refine the style guide and structure, then continue.

Open questions to settle before starting Chapter 1:
1. **Output format?** Plain Markdown, or something richer (Asciidoc, LaTeX, mdBook)? Suggest plain Markdown for drafting; convert later.
2. **Companion repo strategy?** Mirror or trust hashes? Suggest mirroring to a tagged fork.
3. **Diff format preference?** Unified (`diff -u`) inline, or side-by-side rendered? Suggest unified for drafting.
4. **Length target per chapter — sanity check?** ~25 pages avg ⇒ ~600 pages total. Acceptable, or aim shorter/longer?
5. **Where do code blocks come from at chapter-write time?** Suggested workflow: when writing section N, run `git show <hash>` and embed the output. Tooling can later validate that embedded diffs still match the upstream commit.
