# Session 014 — Chapter 13 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that has produced sessions 002–013).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 013 delivered Ch 12 (Initializers, nineteen commits, ~10,900 words). User direction is still autonomous — no chapter-by-chapter review. Ch 13 covers commits 116–126: `extern` (file + block scope), `_Alignof`/`_Alignas`, static locals, compound literals, bare `return;`, static globals, `do … while`, 16-byte stack alignment, small-return truncation. Eleven commits — about half the size of the last two chapters by commit count.

## What was done

### Drafting decisions

- **Length:** ~8,600 words. Below the handoff target of 10,000–12,000. The chapter ran shorter than predicted because more than half of the eleven commits are very small (bare `return;` is a one-line patch on each side; the two psABI fixes are a few lines each; static globals is six lines; static locals is nine lines), and per-commit prose for these has a floor. The substantive sections (§13.1 extern, §13.2 alignment, §13.4 compound literals) got full-length treatment. Decided not to pad. Same call as session 013, which also went under target without padding.
- **Section structure:** 9 sections, no concept interlude. Followed the handoff's bundling proposal exactly:
  - §13.1 bundled commits 116, 117 (`extern` at file + block scope). Per handoff. The C linkage model walk is the section's centerpiece, ~1,500 words.
  - §13.2 bundled commits 118, 119 (`_Alignof`/`_Alignas` + GNU variable-operand extension). Per handoff. The §12.11 `.align` interaction is named explicitly.
  - All other sections single-commit (per handoff).
- **No concept interlude.** The handoff defaulted to no interlude on static-vs-dynamic linking; held to that. The §13.1 prose carries the C linkage model on its own. The static-vs-dynamic distinction is noted briefly in the chapter intro as deliberately omitted (chibicc never participates in dynamic linking).
- **§13.1 walks the C linkage model** — handoff acceptance criterion. External (default file scope), internal (`static`), none (block-scope locals). The `is_extern` flag's role and the `is_definition` flag are both walked. The first-time-real-multi-translation-unit data linking in the test suite is named.
- **§13.2 names the §12.11 interaction** — handoff acceptance criterion. The `.align` directive's source migrates from `var->ty->align` to `var->align`. Per-member alignment overrides via `_Alignas` are walked.
- **§13.3 names the local-vs-global blur** — handoff acceptance criterion. Section prose: "This is the local-versus-global split *blurring*." Four pieces of inherited machinery are named (`new_anon_gvar`, `push_scope`, `gvar_initializer`, the absence of `lvar_initializer`).
- **§13.4 reuses the §12.1 Initializer tree without re-derivation** — handoff acceptance criterion. The reuse ratio is called out. The `&(Tree){...}` test is walked through the §12.7 relocation channel.
- **§13.5 walks the `ND_RETURN`-with-null-`lhs` reuse pattern** — handoff acceptance criterion. The §11.11 `ND_GOTO`-with-partial-fields precedent is named.
- **§13.8 explains *why* 16-byte alignment matters** — handoff acceptance criterion. The psABI invariant is stated, the SSE-trap-in-glibc consequence is named, the no-test-because-invisible-without-glibc-call observation is made. The two-place alignment (frame + per-call) distinction is walked.
- **§13.9 names the §10.12 `_Bool` cast parallel** — handoff acceptance criterion. The asymmetry (callee canonicalizes, caller only zero-extends) is called out as deliberate.
- **One-table recap.** Eleven rows. Resisted splitting into two tables (the handoff suggested probably-not, and held to that). The chapter's commits don't divide cleanly into two themes the way Ch 12's local-vs-global did.

### Interpretive calls

1. **Compound literals are not counted as canonicalization.** §13.4 prose explicitly addresses this: compound literals desugar a brace-list-at-expression-position into anonymous-variable-plus-initializer-plus-reference, which is parse-time AST rewriting and could qualify. But the mechanism goes through the Initializer tree intermediate (same as initializer lowering in §12.1), and the convention is to count direct AST rewrites only. Count stays at eight.
2. **§13.6's default-`is_static`-in-`new_gvar` is a load-bearing structural choice.** The chapter calls this out twice — once in §13.6 ("the default is a cheap way to get the right behavior for the unnamed cases") and once in the closer. The default makes anonymous globals (string literals, static-local backings, file-scope compound-literal backings) all get `.local` for free. Without it, the linker would see `.L..N` symbols as global. Worth tracking as a structural choice that future commits could break by introducing a new `new_gvar` caller.
3. **§13.5 and §13.6 not bundled.** Bare `return;` (a one-line parser+codegen patch) and static globals (a two-line parser + four-line codegen patch) are both small; they could be one section. Held to one section each because they're conceptually unrelated.
4. **§13.3 and §13.6 not bundled.** Static locals and static globals share a keyword and the `is_static` `VarAttr` flag, but they answer different questions (function-scoped name backed by global storage vs. file-scoped name with internal linkage). Bundling would have obscured the local-vs-global split that the chapter is otherwise tracking carefully. Section prose explains the choice explicitly.
5. **The chapter title — *Linkage* — is acknowledged as reaching.** Only four of eleven commits are linkage proper; the rest are along for the ride. The chapter intro names this honestly. Same approach as the Ch 11 intro (which acknowledged that twenty-one commits don't all fit "operators" cleanly).
6. **psABI conformance thread named in the closer.** §13.8 and §13.9 are the chapter's two ABI-fixes; the chapter intro doesn't predict them as a theme, but the closer names them as a thread that Chapter 14 will continue with variadics. This is a forward-reference setup: when Chapter 14's `va_start` lands, the prose can point back to §13.8/§13.9 as the start of the chibicc-as-real-compiler arc.
7. **The `mov $0, %al` placeholder note from Chapter 5 is queued for Chapter 14.** Forward-reference in the closer. The pending footnote becomes a real explanation when variadic functions land.
8. **VarAttr channel forecast for Chapter 14.** Closer prose: "The fifth, sixth, seventh, eighth, and ninth fields are all queued up." The qualifier soup (`signed`/`unsigned` + `const`/`volatile`/`restrict` etc.) routes through the same channel. This sets up the prediction that Chapter 14's prose can close.

### Voice / structure inherited from Ch 1–12

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote (multiple openers for bundled sections).
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with one feature table (eleven rows; not split this time).

### Three careful avoidances

- **Did not invent the static-vs-dynamic interlude.** The handoff defaulted to no, and the §13.1 prose didn't surface a need.
- **Did not over-derive the Initializer tree in §13.4.** Compound literals reuse §12.1's machinery; the section walks the *new piece* (anonymous variable + cast/literal disambiguator) without re-explaining `Initializer`, `lvar_initializer`, or the lowering walk.
- **Did not present §13.8 as a normal feature commit.** It's a correctness preemption — Rui clearly read the psABI and noticed the bug, since no test in the suite was failing. Section prose names this character: "no test... a *correctness preemption*."

### Date-vs-position note

About half the commits are dated September 2020 (`extern`, `_Alignof`/`_Alignas`, static locals, static globals, `30b3e21` bare-return, `6a0ed71`/`dcd4579` psABI fixes — all within a few days). `127056d` (compound literals) is August 2019; `ee252e6` (`do … while`) is also August 2019; `310a87e` is late September 2020. `main` order interleaves them. Did not call this out in chapter prose; chapter follows `main` order without comment, the same as Ch 7–12.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The VarAttr channel is at four fields** (`is_typedef`, `is_static`, `is_extern`, `align`). Chapter 14 will likely add `is_signed`/`is_unsigned` (probably mutually-exclusive enums) and the qualifier set (`const`, `volatile`, `restrict`, etc.). Watch the channel grow.
- **The anonymous-global pattern** (`new_anon_gvar` + optional `push_scope`) is the chapter's most-reused machinery. Three sections (§13.3, §13.4, §13.6) hinge on it. The pattern existed since Ch 7 string literals; Ch 13 is the first chapter where it's load-bearing for user-facing features.
- **The `is_static` default in `new_gvar`** is a structural choice with non-obvious consequences. Anonymous globals get `.local` for free because of it. If a future commit adds a new `new_gvar` caller and the variable should be `.globl`, the default has to be overridden explicitly. Worth flagging for any future commit that touches `new_gvar`.
- **The `is_definition` flag on `Obj`** is a new persistent piece of state. Used today only by `extern` and indirectly by the `eval`-rejects-non-constant guard in `gvar_initializer`. May be reused by the preprocessor in Chapter 17 when handling header-introduced declarations.
- **The local-vs-global split survived the static-locals blur.** The codegen's `globals` list is what drives `emit_data`; the C-level scope chain is what drives name visibility. The two are now decoupled — a name can be on the global list with only function-scope visibility (static locals) or on the global list with file-scope-only visibility (static globals). The split is still real; it's just not "scope" anymore, it's "storage."
- **The psABI conformance thread opens in this chapter.** §13.8 (16-byte stack alignment) and §13.9 (small-return truncation) are corrections, not features. Chapter 14 will continue with variadic argument handling. Possibly more in later chapters as Rui starts targeting glibc-using code more aggressively.
- **The `Relocation` mechanism** picks up a use case in §13.4 (`&(Tree){...}` initializing a global pointer with the address of an anonymous-global compound literal). The mechanism doesn't extend; only the source of the relocation labels broadens.
- **The Initializer tree** is unchanged. Compound literals reused it without extending it. Confirms the "Initializer tree shape is final" prediction from session 013.
- **The constant-expression evaluator** (`eval`/`eval2`/`eval_rval`) gets two new callers: `_Alignof(typename)` (folds to `ty->align`) and `_Alignas(N)` (uses `const_expr` to evaluate the alignment value). The trio's shape is unchanged.
- **The `new_unique_name` / `.L..N` convention** is the GNU assembler's hint that a symbol is local to the file. Chibicc relies on it both for correctness (anonymous globals shouldn't be linker-visible) and for clean symbol tables.
- **Canonicalization-at-parse-time count is unchanged at eight.** Compound literals were considered and not counted, on the same grounds as initializer lowering (Initializer-tree intermediate, not direct AST rewriting).
- **Pre-factor-before-feature count is unchanged at six.** Chapter 13 didn't add new instances; the §11.9 incomplete-array sentinel still has its three consumers.
- **The fourth namespace (labels)** is unchanged. Chapter 13 doesn't introduce a fifth.
- **The `is_typename` predicate** is unchanged in shape. `_Alignas` and `_Alignof` keywords were added to the list (and to `tokenize.c`'s keyword list) but the structure is the same.
- **The cast/compound-literal disambiguator in `cast`** (peek for `{` after `(typename)`) is a small but novel parser pattern. The lookahead-on-the-following-token shape is the same one §10.x used to disambiguate `sizeof typename` from `sizeof unary`.
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Tests are in C** as of Ch 8 §8.2.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17.
- **The argreg 8/16/32/64 split** is fully in place. Ch 13 doesn't touch it.
- **The `unreachable()` macro** lives in `chibicc.h`. Used by `store_gp`, `declspec`, `write_buf`. Ch 13 didn't add new callers.
- **`copy_struct_type`** (Ch 12 §12.10) — no new uses in Ch 13. Watch for it later.
- **The `MIN`/`MAX` macros** (Ch 12 §12.3) — no new uses in Ch 13.
- **Test file additions:** `test/extern.c` (commit 116), `test/alignof.c` (commit 118), `test/complit.c` (commit 121). The test suite continues to grow per-feature.
- **Ch 1 errata list** unchanged.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels. Four errata candidates.
- **Empty brace initializer (`int x[3] = {};`)** is a chibicc-specific extension. Standard C99 requires at least one element. Matches GCC behavior; not flagged as errata.
- **Chibicc accepts `f()` and `f(void)` as identical declarations.** §12.11 calls this out. Real C distinguishes them at call sites for K&R-era unspecified-parameter functions. Errata candidate (or "intentional simplification" candidate).
- **`.bss` is the third assembly section chibicc emits.** Section list: `.text`, `.data`, `.bss`.
- **The `>>` codegen quirk** (Ch 11 §11.13). Errata candidate.
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4. Errata candidate.
- **The `mov $0, %rax`** (variadic `%al`-zeroing) noted in Ch 5 §5.1. Becomes real in Ch 14; pending footnote for revision pass.

## Exit state

- `chapters/13-linkage.md` drafted, ~8,600 words.
- Session 014 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 015 (Chapter 14 — Variadics, signedness, qualifiers, commits 127–138).
- CLAUDE.md status note updated (chapter count goes from "Ch 12 drafted" to "Ch 13 drafted").
