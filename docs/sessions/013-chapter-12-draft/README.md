# Session 013 — Chapter 12 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that has produced sessions 002–012).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 012 delivered Ch 11 (All the operators, twenty-one commits, ~12,260 words). User direction is still autonomous — no chapter-by-chapter review. Ch 12 covers commits 97–115: the full C initializer grammar, plus `void` parameter lists and global alignment. Nineteen commits — the densest single arc in the chibicc commit log.

## What was done

### Drafting decisions

- **Length:** ~10,900 words. Below the handoff target of 13,000–15,000. The chapter ran shorter than predicted because half of the nineteen commits are very small (the three ergonomics commits in §12.8, the `.bss` commit, the two-line `void` commit, the one-line `.align` commit) and per-commit prose for one-line patches has a floor. Compares to: Ch 11 (~12,260), Ch 10 (~14,800), Ch 9 (~9,300), Ch 8 (~7,400). Decided not to pad. The §12.1 anchor walk is full-length; the small commits get appropriately tight treatment.
- **Section structure:** 11 sections, no concept interlude. Followed the handoff's bundling proposal exactly:
  - §12.2 bundled commits 98, 99 (excess-zero-fill and excess-skip). Per handoff.
  - §12.5 bundled commits 102, 103, 104 (struct local + struct-from-struct + union local). Per handoff.
  - §12.7 bundled commits 106, 107 (global struct + global union/pointer). Per handoff. The pointer/relocation walk is the section's centerpiece.
  - §12.8 bundled commits 108, 109, 110 (omit-braces, extra-braces, trailing-comma). Per handoff.
  - §12.10 bundled commits 112, 113 (flexible array decl + init). Per handoff.
  - §12.11 bundled commits 114, 115 (`void` params + `.align`). Per handoff.
- **No concept interlude.** The handoff defaulted to no interlude; held to that. The local-vs-global split is named in §12.6 prose without a separate section.
- **§12.1's `Initializer` walk is the chapter's anchor** — handoff acceptance criterion. The struct definition is shown in full; `new_initializer`, `initializer2`, `init_desg_expr`, `create_lvar_init`, and `lvar_initializer` are each walked. The `ND_NULL_EXPR` seed-pattern is named.
- **§12.4 names the closure of the §11.9 sentinel** — handoff acceptance criterion. Prose: "This commit closes the §11.9 prediction." The `is_flexible` flag is the channel.
- **§12.5 covers struct-from-struct explicitly** — handoff acceptance criterion. The lookahead-for-`{` dispatch is named.
- **§12.6 names the local-vs-global split** — handoff acceptance criterion. Prose: "This commit names the chapter's central tension." The split is one of the chapter-closer's three structural shifts.
- **§12.10 explains the relationship to §11.9 incomplete arrays** — handoff acceptance criterion. The §11.9 sentinel's three consumers are named in the section and in the closer.
- **§12.11 distinguishes `f()` from `f(void)`** — handoff acceptance criterion. The parser-only-not-semantic distinction is called out: chibicc accepts the syntax but doesn't enforce the standard's K&R distinction at call sites.
- **Two tables in the recap** — split at the local/global boundary. The first table covers commits 97–104 (local-side); the second covers 105–115 (global-side and the late polish). Same theme-split logic as Ch 10 and Ch 11.

### Interpretive calls

1. **Initializer lowering is not counted as canonicalization.** The §12.1 prose calls this out explicitly: initializer lowering desugars `int x[3] = {1,2,3};` into a comma chain of three assignments, which would qualify as canonicalization-at-parse-time, but the mechanism goes through a separate intermediate (the Initializer tree) before producing the lowered AST. Convention is to call this "initializer lowering" rather than "canonicalization." Count stays at eight.
2. **Pre-factor-before-feature count goes 4 → 6.** §12.4 (length-from-initializer consumes §11.9's `array_of(ty, -1)` sentinel) and §12.10 (flexible array members consume the same sentinel) each count as a new instance. The §11.9 sentinel now has three consumers in chibicc — both Ch 12 cases plus the Ch 11 §11.9 forward-declared-struct case it was originally written for. Did *not* count both Ch 12 commits as two pre-factors of §11.9 (the handoff said "five" was the proposal; ran with six because they're independent features sharing one mechanism, which is the canonical pre-factor-before-feature pattern).
3. **The `eval` split is named in §12.7, not §11.15.** The Ch 11 §11.15 prose set up `eval`; the Ch 12 §12.7 prose walks the `eval`/`eval2`/`eval_rval` trio that the global-pointer-initializer feature requires. Threading a `label` out-parameter through `eval2` is the load-bearing change.
4. **The local-vs-global split is the chapter's central structural idea.** Named in §12.6 prose as "the chapter's central tension." Returned to in the closer as "shared front end with diverging back ends — a pattern Chapter 13 will return to with linkage." Set up as a forward-reference for Ch 13.
5. **`.bss` gets a section, not a one-line mention.** The handoff suggested it deserves its own section; ran with that. §12.9 is short (~400 words) but covers the rationale for why uninitialized globals shouldn't occupy bytes in the object file.
6. **The `copy_struct_type` deep-copy in §12.10 is contrasted explicitly with the §11.9 mutation-in-place pattern.** Section prose: "The §11.9 commit's mutation-in-place pattern (`*sc->ty = *ty;`) does *not* repeat for flexible arrays. §12.10 uses `copy_struct_type` instead — because two variables with the same flexible-struct type can have different trailing-array sizes, the patched type belongs to the variable, not to the type." This is a structural distinction worth naming because it might otherwise look like inconsistent style.
7. **The `MIN`/`MAX` macros are noted as introduction-points in §12.3.** First appearance in the codebase. Carried as a small ongoing thread.

### Voice / structure inherited from Ch 1–11

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote (multiple openers for bundled sections).
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature tables (two tables, by local-vs-global theme).

### Three careful avoidances

- **Did not invent a fifth namespace.** The handoff predicted Ch 12 would not add one; held to that. Initializer scopes reuse block scope; flexible-array members live in the existing struct member namespace.
- **Did not over-explain the codegen of `rep stosb`.** §12.2 walks it once; later mentions of zero-fill don't re-derive.
- **Did not re-explain `eval` from §11.15.** §12.6 references it; §12.7 walks the *split* into `eval`/`eval2`/`eval_rval`, which is the new material.

### Date-vs-position note

About half the commits are dated August 2019 (Rui's initial implementation push); a handful from April–July 2020; the bulk from September–October 2020 (the same late-2020 polish window that produced most of Ch 11). `main` order interleaves them — commit 97 (`22dd560`, August 2019) is followed by commit 98 (`ae0a37d`, September 2020) is followed by commit 99 (`a754732`, April 2020). Did not call this out in chapter prose; the chapter follows `main` order without comment, the same as Ch 7–11.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The `Initializer` tree shape is final as of Ch 12.** Chapter 13 doesn't extend it. New initializer features (compound literals, `_Alignas`) reuse the existing infrastructure.
- **The local-vs-global split is a stable separation.** Chapter 13 adds `extern`, which routes through `VarAttr` like Chapter 10's `static`/`typedef`, and `_Alignof`/`_Alignas`, which interact with the global path's alignment emission.
- **The `eval`/`eval2`/`eval_rval` trio is the constant-expression evaluator's stable shape.** Chapter 13's `_Alignof` adds an arm; Chapter 14's bit-field-width handling adds more callers.
- **The `Relocation` mechanism** is the only place chibicc emits link-time-relocatable directives. Chapter 13's `extern` and static locals interact through the symbol table, not the relocation list.
- **The `Member->idx` field** (added in §12.5) bridges the linked-list-members representation and the indexed-children Initializer representation. Chapter 13 reuses it for compound literals.
- **The `is_flexible` flag** lives on both `Initializer` (§12.4) and `Type` (§12.10). Two different fields, both spelled the same way; both mean "this slot is waiting for its size to be discovered."
- **`copy_struct_type` (§12.10) is the deep-copy pattern.** Used when a type needs to be patched per-variable rather than once globally. Watch for further uses; Chapter 13's `_Alignas` may reuse the same shape.
- **`rep stosb` for ND_MEMZERO codegen** (§12.2) is the first SIMD-adjacent instruction chibicc emits. Three setup instructions plus one repeated store. Codegen pattern worth remembering.
- **The `ND_NULL_EXPR` seed-pattern** (§12.1) lets a comma-chain accumulator avoid a special case for the first element. Cute trick worth remembering.
- **The `MIN`/`MAX` macros** (introduced in §12.3) are tiny but they're the codebase's only general-purpose macros besides `unreachable()`. New uses likely.
- **Canonicalization-at-parse-time count is unchanged at eight.** Initializer lowering is not counted (it goes through the Initializer tree intermediate, not direct AST rewriting).
- **Pre-factor-before-feature count is now six.** §12.4 and §12.10 each consume the §11.9 incomplete-array sentinel (`array_of(ty, -1)`).
- **The struct-mutation-in-place pattern (§11.9 `*sc->ty = *ty;`) is *not* used for flexible arrays.** §12.10 uses `copy_struct_type` instead because per-variable patching is required. Worth tracking as a divergence from the §11.9 precedent.
- **The fourth namespace (labels, §11.10) is unchanged.** No fifth namespace from Ch 12.
- **The VarAttr channel** (`is_typedef`, `is_static`) is unchanged. Chapter 13 will add `is_extern`.
- **The `is_typename` predicate** is unchanged in shape since §11.10's lookahead-by-probe addition.
- **The split parser pattern (§12.8)** — `array_initializer1`/`array_initializer2`, `struct_initializer1`/`struct_initializer2` — is a new local idiom for "with-or-without enclosing braces." Watch for further splits.
- **Test-line counts in `test/initializer.c`** balloon throughout the chapter. By the end of §12.10 the file is the largest single test in the suite. Worth flagging if any future commit touches initializer tests.
- **The tag namespace** from Chapter 9 §9.4 + Chapter 10 §10.14 is unchanged.
- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.
- **The `mov $0, %rax`** (variadic `%al`-zeroing) noted in Ch 5 §5.1. Pending footnote for revision pass.
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4. Errata candidate.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels. Four errata candidates.
- **Empty brace initializer (`int x[3] = {};`)** is a chibicc-specific extension. Standard C99 requires at least one element. Noted in §12.2 prose; not flagged as errata because it matches GCC behavior.
- **Chibicc accepts `f()` and `f(void)` as identical declarations.** §12.11 calls this out. Real C distinguishes them at call sites for K&R-era unspecified-parameter functions. Errata candidate (or "intentional simplification" candidate).
- **`.bss` is the third assembly section chibicc emits.** Until §12.9 it was just `.text` and `.data`. New section list: `.text`, `.data`, `.bss`.
- **Pointer arithmetic via `eval2` propagates a relocation through left operand only.** §12.7 prose walks this. The asymmetry is correct (`a + b` is a relocation if `a` is, but `a * b` isn't relocatable) but worth remembering when extending the evaluator.
- **The `.align` directive** (§12.11) is emitted for *every* global, not just ones with strict alignment. Slightly wasteful but uniform. No code-size impact since `.align` for already-aligned data is a no-op.
- **The everything-fits-in-rax codegen invariant** continues. The new `ND_MEMZERO` and `ND_NULL_EXPR` don't return a value at all (they're statement-position expressions through `ND_EXPR_STMT`).
- **The argreg 8/16/32/64 split** is fully in place. Ch 12 doesn't touch it.

## Exit state

- `chapters/12-initializers.md` drafted, ~10,900 words.
- Session 013 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 014 (Chapter 13 — Linkage, commits 116–126; eleven commits including `extern`, `_Alignof`/`_Alignas`, static locals, compound literals, do-while, return-without-value, 16-byte stack alignment).
- CLAUDE.md status note updated (chapter count goes from "Ch 11 drafted" to "Ch 12 drafted").
