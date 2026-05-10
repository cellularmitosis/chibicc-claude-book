# Handoff: Ch 7 done → proceed to Ch 8

**For:** the next claude session.
**From:** session 008.
**Status:** Ch 7 drafted. Continue autonomously to Ch 8 (Scopes and source locations). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/008-chapter-07-draft/README.md`](README.md)** — what session 008 did, including the canonicalization-with-delegation sub-variant naming, the second pre-factor instance, the Trusting-Trust framing, and the lookahead-by-probe pattern naming.
2. **[`docs/sessions/007-chapter-06-draft/HANDOFF.md`](../007-chapter-06-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`07-globals-characters-strings.md`](../../../chapters/07-globals-characters-strings.md)** — the seven chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 8 = commits 44–48.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.

## Chapter 8 scope

**Title (working):** *Scopes and source locations*.
**Commits:** 44–48 in chronological order on `main`.

| # | Hash | Subject |
|---|---|---|
| 44 | `ca8b243` | Handle block scope |
| 45 | `cd832a3` | Rewrite tests in shell script in C |
| 46 | `6647ad9` | Precompute line number for each token |
| 47 | `1c91d19` | Emit `.file` and `.loc` assembler directives |
| 48 | `e6307ad` | Add comma operator |

**Five commits**, mid-range size. Two language features (block scope, comma operator), one test-infrastructure overhaul (the shell-script-to-C rewrite, which is large in line count but conceptually self-contained), and two debug-info commits (precomputed line numbers, `.file`/`.loc`). Natural sectioning probably matches the commit order:

- **§8.1 — Block scope** (commit 44). The first time chibicc grows nested symbol tables. `find_var` becomes scope-aware. Local variables declared inside `{}` blocks shadow outer locals and disappear at the closing brace. This is the commit Ch 7 §7.5 (statement expressions) flagged: until now, every "local" was function-level, even inside a `({...})` body. Now it isn't.
- **§8.2 — Tests in C** (commit 45). The shell-script-only test harness gets supplemented (or replaced?) by C-language tests in a new `test/` directory. This commit exercises everything Ch 7 §7.5 (statement expressions) and §7.6 (file I/O) prepared the ground for. Read the diff closely — the C-form tests almost certainly use `assert(cond, msg)` macros or statement expressions to compress what the shell-string form needed quoting tricks for. Length-wise this commit is large but the prose can be short: walk a few representative test transformations, link the `({...})` use to §7.5, and move on.
- **§8.3 — Precomputed line numbers** (commit 46). Builds on Ch 7 §7.6's `verror_at` line-number computation, which was O(n) per error. This commit precomputes line numbers for every token at lex time, storing them on the `Token` struct. The motivation isn't really speed (chibicc errors out on the first problem); it's that §8.4 needs per-token line numbers for `.loc` directives and the precomputed table is the right place to put them.
- **§8.4 — `.file` and `.loc` directives** (commit 47). Each statement now emits a `.loc <file-id> <line>` directive in the assembly output, which the assembler turns into DWARF line-number information. After this commit, GDB can step through chibicc-compiled programs at the source level. The `.file 1 "..."` directive at the top of the output associates a numeric ID with the source filename; subsequent `.loc 1 <line>` directives reference that ID. This is a real win — chibicc is now debuggable.
- **§8.5 — The comma operator** (commit 48). Small commit. `expr` (the top of the expression hierarchy) gains a comma-separated form: `e1, e2` evaluates `e1` for side effects and yields `e2`. Codegen evaluates left-to-right, discards the left value, returns the right. The `add_type` rule says the type of `e1, e2` is `e2`'s type. Worth flagging that this is the first time `expr` itself has gained a level — the hierarchy until now has been `expr → assign → equality → ...` and now becomes `expr → assign ("," assign)*`.

**Concept interlude:** The chapter mapping doesn't call for one. The natural moment if there were one would be on **DWARF debug info** (§8.4 is the obvious peg), but the chapter mapping is silent and the chapters that *do* have called-out interludes (Ch 4 type, Ch 5 calling convention, Ch 6 array decay, Ch 7 integer representation) all appear in `chapter-mapping.md` as "concept interlude: X". Chapter 8 isn't on that list. Default to no interlude unless the prose develops a clear need for one. If §8.4 ends up wanting to explain DWARF's design at length, consider a short ~600-word interlude. Otherwise, the chapter is five sections and an intro/recap.

## Steps

1. `cd research/sources/chibicc && for h in ca8b243 cd832a3 6647ad9 1c91d19 e6307ad; do git show --stat $h; done` to scan all five diffs at once.
2. Read each commit in full. Pay particular attention to:
   - **`ca8b243`**: how the symbol-table-stack is structured. Is it a linked list of scopes that `find_var` walks? Is there an `enter_scope`/`leave_scope` pair? How does `compound_stmt` use it? Does it interact with the `({...})` body parsing from Ch 7 §7.5?
   - **`cd832a3`**: how the tests are organized. New directory? New Makefile target? How does the C-form `assert` macro compare to the shell-form `assert` function? This commit is large in diff size but conceptually narrow — don't get bogged down.
   - **`6647ad9`**: the `Token.line_no` field, the precompute pass, where it's invoked from in the tokenize loop.
   - **`1c91d19`**: `.file`/`.loc` syntax, where in `gen_stmt`/`gen_expr` the `.loc` is emitted (probably at the start of every `gen_stmt` call), and how the file-ID assignment works (likely just `1` for the single-file case).
   - **`e6307ad`**: the grammar level change — `expr = assign ("," assign)*` — and the `ND_COMMA` node kind. Codegen for comma is interesting because the left operand's value has to be discarded; check whether chibicc uses `ND_EXPR_STMT` shape or something else.
3. Read the destination state at `e6307ad` (or the next major checkpoint after) for `chibicc.h`, `parse.c`, `codegen.c`, `tokenize.c`, and look at the new `test/` directory if it exists. Particularly: how does the symbol-table stack interact with the existing `locals` list?
4. Draft `chapters/08-scopes-and-source-locations.md`. Likely 7,000–9,000 words. Five commits, two of them substantive (§8.1 block scope and §8.4 `.loc` directives), two small (§8.3 precompute, §8.5 comma), and one large-but-mostly-mechanical (§8.2 test rewrite). The section budget should reflect this — §8.1 and §8.4 deserve full treatment, the others can be tighter.
5. Write `docs/sessions/009-chapter-08-draft/README.md`.
6. Write `HANDOFF.md` for session 010 (Chapter 9 — Structs and unions, commits 49–55).

## Voice / structure rules

Same as Ch 1–7:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with a feature table — five rows, one per commit.
- Diff format: lean toward inline diff fragments and quoted file snippets.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage. Ch 8's debug-info section might want to borrow Rui's "core value is simplicity" quote since `.file`/`.loc` is a notable instance of "do the simple right thing." Check before passing.
- §8.2 (test rewrite) is large in line count but should be short in prose. Resist the urge to walk every test that changed; pick 3–5 representative cases (one trivial, one statement-expression-using, one that exercises file I/O if any do). The point is the *infrastructure shift*, not the per-test detail.
- Don't write a DWARF interlude unless §8.4 actually demands one. The handoff calls for none and chibicc's `.loc` use is shallow enough that a one-paragraph in-prose explanation should suffice.
- The block-scope commit (`ca8b243`) likely interacts with the function-parameter handling in subtle ways. `function`'s prologue calls `create_param_lvars(ty->params)` which prepends to `locals`. The new scope mechanism has to handle the parameters being in the *function body's* scope, not in some outer scope. Check how this is wired.
- The comma operator's grammar position matters. C says `,` is the lowest-precedence binary operator (lower than `=`). Chibicc's grammar puts `,` at `expr` and `=` at `assign` immediately below — that's correct. But note that some contexts disallow comma (function arguments, initializers); check whether chibicc enforces this. Probably not in this commit.
- The tests-in-C commit changes the build/test infrastructure in a way that's worth understanding before writing about. Read the new Makefile carefully.

## Standing notes worth tracking across sessions

- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`. Address in a revision pass.
- **The `mov $0, %rax` (variadic `%al`-zeroing)** is established in Ch 5 §5.1. Footnote with SysV ABI section reference (3.2.3) is a possible revision-pass addition.
- **The "more than 6 args silently miscompiles"** call-out is established in Ch 5 §5.4. Errata appendix candidate.
- **The `add_type` `ND_ADDR` simplification** (Ch 6) is still a Ch 10 fix-target.
- **TY_FUNC still has no consumer.** Chapter 10 still marks the moment.
- **Canonicalization-at-parse-time** is now a five-instance pattern with one sub-variant (delegation in §7.5 vs desugaring in the four Ch 6 instances). Future instances should be classified into one mode or the other in the prose. Ch 11's `+=` family will likely be desugaring; Ch 12's compound initializers ambiguous.
- **Pre-factor before feature** is now a two-instance named pattern (Ch 6 §6.5 and Ch 7 §7.6). Likely Ch 10 instance at the front (`int`-becomes-32-bit refactor before the new types).
- **The `.text`/`.data` directive pair is fully landed** (Ch 6 §6.5 and Ch 7 §7.1). No more carrying.
- **The `argreg` 8/64 split is done** (Ch 7 §7.2). When `int` becomes 32-bit in Ch 10, will need `argreg32` and possibly `argreg16`. Size-based dispatch is the shape that generalizes.
- **The `format` helper landed in Ch 7 §7.3.** Workhorse going forward.
- **The `is_typename` helper landed in Ch 7 §7.2.** Grows steadily; touched by every new type keyword.
- **The trailing-newline guarantee in `read_file`** (Ch 7 §7.6) protects line-comment skipping and several error-reporting paths. When `read_file` is revisited (Ch 16), preserve.
- **The lookahead-by-probe pattern** named in Ch 7 §7.1. Likely Ch 10 instance for nested-declarator parsing decisions.
- **The Trusting-Trust framing for `read_escaped_char`** (Ch 7 §7.4) sets up Ch 17 (self-hosting). When chibicc actually compiles itself, the prose should explicitly close the loop: "the tautological escape-handling from Ch 7 §7.4 is now a fixed point."
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4) is a wart. Errata candidate.
- **Block scope arrives in Ch 8 §8.1** — once landed, the Ch 7 §7.5 note about statement-expression locals living at function scope will be obsolete (they'll live at the inner scope). When writing Ch 8 §8.1, mention this changes the behavior described in Ch 7 §7.5.

## Acceptance criteria for Ch 8

- [ ] `chapters/08-scopes-and-source-locations.md` exists, end-to-end readable.
- [ ] §8.1 explains the symbol-table-stack structure and how `find_var` becomes scope-aware.
- [ ] §8.1 mentions that this changes the Ch 7 §7.5 behavior of statement-expression locals.
- [ ] §8.4 explains `.file`/`.loc` and the resulting GDB-debuggability win without a full DWARF interlude.
- [ ] §8.5 places the comma operator at the correct grammar level (`expr = assign ("," assign)*`).
- [ ] Each commit has a `git checkout <full-hash>` opener.
- [ ] Voice matches Ch 1–7.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/009-chapter-08-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 010 (Chapter 9 — Structs and unions, commits 49–55).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/008-chapter-07-draft/HANDOFF.md  (this handoff)
2. docs/sessions/008-chapter-07-draft/README.md   (what session 008 did)
3. chapters/01-a-calculator.md                     (template, voice)
4. chapters/02-from-program-to-programs.md
5. chapters/03-statements-and-local-variables.md
6. chapters/04-pointers.md
7. chapters/05-functions.md
8. chapters/06-arrays.md
9. chapters/07-globals-characters-strings.md       (most recent chapter)
10. research/commits/chapter-mapping.md            (confirms Ch 8 scope)
11. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 8 (Scopes and source locations, commits 44–48)
per the steps in the handoff. End-of-session: write your session dir
under docs/sessions/009-chapter-08-draft/ with a README and a HANDOFF
for session 010 (Chapter 9 — Structs and unions).
```
