# Session 009 — Chapter 8 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that produced sessions 002–008).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 008 just delivered Ch 7 (Globals, characters, and strings, twelve commits, ~12,500 words). User direction is still autonomous — no chapter-by-chapter review. Ch 8 covers commits 44–48: block scope, the shell-to-C test rewrite, precomputed token line numbers, `.file`/`.loc` debug directives, and the comma operator. Five commits, mid-range size.

## What was done

### Drafting decisions

- **Length:** ~7,400 words. Smaller than every chapter since Ch 4 (which was the last single-language-feature-cluster chapter). Five commits is the lowest count since Ch 4's six, and only two are substantive (block scope and `.loc`). The proportions inside the chapter reflect that: §8.1 is ~1,900 words, §8.4 is ~1,200 words, the others are 800–1,200 each.
- **No concept interlude.** The chapter mapping doesn't call for one and the §8.4 prose handles DWARF in a single paragraph in-text. A stand-alone DWARF interlude would have padded the chapter without adding what a reader needs.
- **Section structure:** five sections, one per commit, in commit order. No bundling. Each commit has enough self-contained material to deserve its own section.
- **§8.1 (block scope) framed as the §7.5 callback the previous chapter promised.** Chapter 7's statement-expression section explicitly noted that locals inside `({...})` lived for the whole function until block scope arrived. §8.1 closes the loop in two places: in the "Back-reference" sub-section, and in the recap.
- **Two-scope-per-function structure flagged explicitly.** `function` does its own `enter_scope` *before* `compound_stmt` does another, giving every function an outer scope for parameters and an inner one for the first level of body locals. The handoff flagged this as "subtle" and the prose calls it out as the C-correct shape (parameter and body-local of the same name are formally a redeclaration error in the same scope, but chibicc isn't checking that yet).
- **§8.2 (tests in C) kept short, per handoff.** ~1,200 words. Walked three representative tests (one trivial from `arith.c`, one statement-expression-using from `pointer.c`, one block-scope-testing from `variable.c`), explained the host-`cc`-as-preprocessor pipeline, covered the `ASSERT(x,y)` macro and `#y` stringification. Did not walk every test that changed.
- **§8.2's host-cc-preprocessor framing forward-points to Ch 17.** When chibicc gets its own preprocessor, the host-cc-as-preprocessor pipeline collapses. The §8.2 prose plants this cross-reference.
- **§8.3 (precompute) framed as the third instance of the pre-factor pattern.** Chapter 6 §6.5 introduced it (Function/Obj merge before globals); Chapter 7 §7.6 was the second (printf→println before -o/--help); §8.3 is the third (precompute line numbers before .loc directives). Three instances now make this firmly part of chibicc's commit-style vocabulary, and the recap says so.
- **§8.4 (DWARF debug info) deliberately kept shallow.** The prose explains DWARF's `.debug_line` section and how `.file`/`.loc` feed it, but doesn't go into the line-number state-machine encoding or the difference between `.debug_line` and `.debug_info`. The "Trade-offs left on the table" subsection notes that chibicc doesn't emit `.debug_info` and explains why (cost vs payoff). This is one of the chapter's interpretive moments: the prose takes a position on what chibicc is and isn't trying to be as a debug-info producer.
- **§8.4's worked example shows the assembly directly.** A small `int main()` program with three statements is shown both as C source and as the resulting assembly skeleton with `.file 1` and `.loc 1 N` directives interleaved with the instructions. Then a GDB session demonstrates the user-visible payoff (breakpoints, stepping, listing). Both fragments are illustrative and not byte-exact transcripts of chibicc output — the prose says the actual output has more directives because every recursive `gen_expr` emits one.
- **§8.5 (comma operator) frames the GCC "generalized lvalue" extension carefully.** Rui's commit message explicitly calls it a "deprecated GCC language extension" and says he's implementing it because it's useful for other features. The §8.5 prose quotes the message and identifies the likely "other feature" as compound assignment (Ch 11), with the lowering `a += b` → `(tmp = &a, *tmp = *tmp + b)` named as the use case. This is an interpretive forward-reference but it's the natural one and grounded in standard compiler-engineering practice.
- **§8.5 places comma at the correct grammar level** (per acceptance criterion). The prose explains that comma's grammar position (`expr` rather than `assign`) is below assignment in precedence, and that function arguments / declarator lists / initializer lists already call `assign` so they don't get confused. Notes that this is the first time `expr` has been more than a thin wrapper around `assign`.
- **The grammar comment in `expr` is `expr = assign ("," expr)?`** (right-recursive optional, not iterative `("," assign)*` as the handoff predicted). The prose reflects what Rui actually wrote, not what the handoff predicted. Notes that the right-recursive form parses `1, 2, 3` as `1, (2, 3)` and that this is observably equivalent to left-associative parsing because of single-register codegen.
- **Date-vs-position note in the intro,** matching Ch 7's treatment. The five Ch 8 commits are dated August 2019 (the comma operator), April 2020 (precompute and `.loc`), and September 2020 (block scope and tests-in-C). The chapter follows commit-list order.
- **Diff format** matches Ch 7: inline diff fragments where the change is a small edit, full quoted snippets where a function is new or substantially rewritten. `find_var`'s before/after is a full diff because the contrast between the locals/globals split and the unified scope-walking is the point.
- **Forward references kept short and grounded:** Ch 9 (next chapter, scoped tag names that the §8.1 mechanism enables); Ch 11 (compound assignment as the likely consumer of generalized-lvalue comma); Ch 17 (preprocessor, when the host-cc-preprocessor pipeline collapses); Ch 22 (hash-table symbol lookup, when linear scope walking finally becomes a problem). All cross-checked against `chapter-mapping.md`.

### Three small interpretive calls

1. **The two-scope-per-function structure framed as deliberately C-correct rather than incidental.** Rui's commit just calls `enter_scope` in `function` before `create_param_lvars` and lets `compound_stmt` do its own enter inside. The prose names this as the right shape for parameter-vs-body-local distinction in the C standard, even though chibicc doesn't check the redeclaration error yet. This is a small interpretive layer on top of "look, two enter_scopes."
2. **The "generalized lvalue" forward-reference to Ch 11 compound assignment.** Rui's commit message says "useful for other features" without naming them. The `+=` lowering through `(tmp = &a, *tmp = *tmp + b)` is the standard compiler-engineering trick and is plausibly what Rui meant, but the prose flags this as an interpretation. Worth checking when Ch 11 actually lands — if Rui uses a different lowering, the §8.5 prose should be revised.
3. **The `add_line_numbers` precompute commit framed as the third instance of pre-factor.** This is editorial — Rui's commit message ("No functionality change") signals it but doesn't name a pattern. The chapter names the pattern and counts instances.

### Two careful avoidances

- **Did not write a DWARF interlude.** §8.4 has one paragraph of DWARF context (what `.debug_line` is, what the assembler does with the directives, how GDB consumes the result). The handoff explicitly said "default to no interlude unless the prose develops a clear need for one" and it didn't.
- **Did not walk every test in §8.2.** The handoff warned against this and the prose stuck to three representative cases plus the macro-and-Makefile mechanics.

### Voice / structure inherited from Ch 1–7

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with a feature table — five rows, one per commit, in commit order.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **Pre-factor-before-feature is now a three-instance named pattern.** Ch 6 §6.5 (Function/Obj merge → globals), Ch 7 §7.6 (printf→println → -o/--help), Ch 8 §8.3 (precompute line numbers → .loc directives). All three are explicitly called "no functionality change" or implied as such in the commit message. The pattern is firmly established. Likely Ch 10 instance: the int-becomes-32-bit refactor right before the new types arrive.
- **Canonicalization-at-parse-time** is unchanged from Ch 7's notes (five instances, one sub-variant). No new instance in Ch 8 — the new commits don't introduce surface-form-A-becomes-surface-form-B rewrites. Ch 9 is unlikely to add one either (struct/union work is mostly type-system and codegen), but Ch 11 (`+=` family) almost certainly will.
- **The lookahead-by-probe pattern** from Ch 7 §7.1 has no Ch 8 instance. Likely Ch 10 use for nested-declarator parsing decisions.
- **The Trusting-Trust framing** from Ch 7 §7.4 has no Ch 8 instance. Still pointing at Ch 17 self-hosting as the moment to close the loop.
- **Block scope is now established.** When future chapters introduce new symbol categories (struct tags in Ch 9, typedef names in Ch 10, label names in Ch 11), the prose can lean on the existing scope mechanism rather than re-explaining. Ch 9 will likely add a `tags` field to `Scope` alongside `vars`.
- **The precomputed `Token.line_no` field** is now load-bearing for `.loc` directives and could become load-bearing for any future "what line is this on" lookup. When the preprocessor lands in Ch 17, line numbers will need to track preprocessor synthesis (`#line` directives, macro expansion locations) — the existing precompute pass will need to handle that.
- **Tests are now in C.** New language features in subsequent chapters get tests in the per-feature `.c` files (`test/control.c`, `test/variable.c`, etc.) rather than in `test.sh`. When a Ch 9 commit adds `test/struct.c`, the prose can say "a new test file in the `test/` directory" without re-explaining the test harness.
- **The host-cc-as-preprocessor pipeline** is what makes the in-C tests work. When Ch 17 lands chibicc's preprocessor, this pipeline collapses and the Makefile can be simplified. Cross-reference to plant in Ch 17.
- **GDB-debuggable output** is now a property of chibicc-compiled programs. Future debugging-related chapters (or sections) can take this for granted.
- **The comma operator's generalized-lvalue extension** is unused in Ch 8. When Ch 11 lowers `+=` through the comma trick, the §8.5 forward-reference becomes concrete and the prose there should explicitly close the loop: "the generalized-lvalue path in `gen_addr` from Ch 8 §8.5 is what makes this lowering work."
- **The two-scope-per-function structure** (params in outer, body in inner) is a subtle invariant. When Ch 16 adds function-pointer support and Ch 17 adds preprocessor macros, the parameter-scope mechanism shouldn't change.
- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()`.
- **`mov $0, %rax`** noted in Ch 5 §5.1. Pending footnote.
- **The `add_type` rule for `ND_STMT_EXPR`** still rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** still a Ch 7 §7.4 wart. Errata candidate.
- **The redeclaration-in-same-scope check** is *not* implemented in Ch 8. Declaring `int x; int x;` in one block produces two `VarScope` entries and the lookup finds the second. Errata candidate. (Also: parameter-and-body-local with same name is the same issue.)
- **Chapter 7's mention of commit hash `46c75e7`** for the precompute commit is wrong. The actual hash is `6647ad9`. Worth noting for the revision pass — Ch 7 §7.6 has a typo.

## Exit state

- `chapters/08-scopes-and-source-locations.md` drafted, ~7,400 words.
- Session 009 dir populated.
- HANDOFF.md primes session 010 (Chapter 9 — Structs and unions, commits 49–55).
- CLAUDE.md status note will need updating (chapter count goes from "Ch 7 drafted" to "Ch 8 drafted").
