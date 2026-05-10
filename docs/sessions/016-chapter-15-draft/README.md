# Session 016 — Chapter 15 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that has produced sessions 002–015).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 015 delivered Ch 14 (Variadics, signedness, qualifiers, twelve commits, ~12,150 words). User direction is still autonomous — no chapter-by-chapter review. Ch 15 covers commits 139–149: floating-point literals, the new `TY_FLOAT`/`TY_DOUBLE` kinds, the cast table extending from 8×8 to 10×10, NaN-correct comparison, scalar arithmetic on the XMM register file, the truth-value extension to floats, the call/definition pair for floating-point parameters through the System V two-register-file convention, default argument promotion, the variadic-floats consumer of Ch 14 §14.2's spill area, the constant-evaluator parallel `eval_double`, and `long double` collapsed to `double`. Eleven commits — moderate, mostly self-contained codegen extensions of existing patterns.

## What was done

### Drafting decisions

- **Length:** ~11,995 words. Right at the upper edge of the 10,000–12,000 target — landed just under 12k. The chapter ran longer than the handoff predicted because §15.2's cast-table walk, §15.3's NaN-correct equality dance, and §15.6's call-site walk each got proper full-section treatment rather than the codegen-extension-only minimum the handoff anticipated. None of those could be compressed without losing the design walk.
- **Section structure:** 11 sections, no concept interlude. Followed the handoff's bundling proposal exactly — each commit gets its own section. No bundling decisions made, no merges. The eleven commits map one-to-one with eleven sections.
- **No concept interlude.** The handoff defaulted to no, conditional on whether §15.1 surfaced the IEEE 754 walk or §15.6 surfaced the SSE/XMM walk. Both ended up inline in their respective sections — the IEEE 754 bit-pattern walk lives inside §15.1 (where chibicc's literal mechanism *is* the IEEE 754 walk, since the literal is loaded as an integer immediate via the bit pattern), and the XMM-register convention lives split across §15.6 and §15.7 (call vs definition). Pulling either out into its own interlude would have left the surrounding section without a body.
- **§15.1 names the float-literal mechanism as bit-pattern-as-integer-immediate, not anonymous-global** — handoff acceptance criterion. The handoff predicted anonymous-global pattern (Ch 13 §13.4); the diff shows inline `union { float f32; double f64; uint32_t u32; uint64_t u64; }` with `mov $imm, %eax; movq %rax, %xmm0`. The chapter prose owns this: "floating-point literals are *not* backed by anonymous globals, despite the Chapter 13 §13.4 anonymous-global pattern being available."
- **§15.2 walks the cast-table extension to 10×10** — handoff acceptance criterion. Eighteen new conversion strings named, the `u64f64` unsigned-to-double dance walked through, the `cvtsi2ss`/`cvttss2si` naming convention named.
- **§15.3 walks the NaN-correct equality sequence** — handoff acceptance criterion. The parity-flag (PF) interaction is the section's substance. The full assembly sequence (`sete`/`setnp`/`and` for `==`; `setne`/`setp`/`or` for `!=`) is inline. The unsigned-setter reuse for `<`/`<=` (because of how `ucomi*` writes CF) is also named.
- **§15.6/§15.7 cover caller and callee separately** — handoff acceptance criterion. Each gets its own section. The XMM argreg machinery is named in §15.6 (no `argreg_xmm` table — chibicc constructs `%xmm{n}` strings on the fly via `popf(int)`); the prologue spill is named in §15.7 with the `store_fp` helper.
- **§15.8 names default-argument-promotion as a C standard rule** — handoff acceptance criterion. The integer-promotion gap (chars and shorts not promoted) is named as an errata candidate.
- **§15.9 closes the loop on §14.2's spill area** — handoff acceptance criterion. The `fp * 8 + 48` formula is walked. The 8-byte-vs-ABI-16-byte stride mismatch is named as a non-conformity that works for the test suite's single-FP-argument cases but would break with multiple FP variadic args. Errata candidate noted.
- **§15.11 names `long double` as a chibicc-specific aliasing** — handoff acceptance criterion. Errata candidate. The chapter prose calls out what would be needed for proper `long double` (TY_LDOUBLE kind, x87 instructions, separate calling convention) and names that as the cost-vs-benefit decision.
- **One-table recap.** Eleven rows. Resisted splitting into multiple tables. The chapter's commits cohere around the single floating-point thread, so the one-table choice is natural.

### Interpretive calls

1. **Anonymous-global forecast was wrong.** The handoff predicted the Ch 13 §13.4 anonymous-global pattern would back float literals. Reading the actual diff: float literals are inlined as integer immediates that bit-cast into XMM via `movq` through `%rax`. The chapter §15.1 prose owns this and notes the trade-off (instruction encoding is a few bytes larger, but no `Relocation`, no `.data` emission, no anonymous-global allocation — the right trade for a non-optimizing compiler).
2. **§15.6 push_args is a structural rework, counted as pre-factor.** The recursive `push_args` helper replaces the prior counter-and-index approach with a tree-walk approach so that two register files can be tracked in parallel. This is in-spirit a pre-factor (the old shape couldn't have been extended to two register files), but it landed in the same commit as the feature it enables. The chapter closer counts this as a pre-factor (now seven).
3. **NaN-aware control flow doesn't need PF.** §15.5's `cmp_zero` for floats uses `xorps + ucomiss` and tests ZF only. The desired NaN-as-truthy semantics happens to fall out of testing ZF alone (NaN sets PF but not ZF, so `je` doesn't jump for `if (NaN)`). The §15.5 prose walks this through. No PF check is needed there, even though §15.3's equality machinery does need it.
4. **The fp_offset bug in §14.2 was a placeholder, not a bug.** The Ch 14 §14.2 prose called `fp_offset = 0` "always 0 in chibicc, because chibicc has no floating-point parameters yet." Re-reading more carefully now: it would have been wrong if a `va_arg(ap, double)` had run against a chibicc-emitted `va_list`, regardless of whether the variadic call had FP args, because `fp_offset = 0` points into the GP region. The §15.9 prose calls this out: "the placeholder was harmless only because no test until this commit exercised `va_arg(ap, double)`." The Ch 14 prose was technically right (the placeholder was harmless) but the framing (`always 0` rather than `placeholder`) could have been clearer.
5. **Stride mismatch in `fp_offset = fp * 8 + 48` named explicitly.** Real ABI uses 16-byte XMM slots; chibicc uses 8-byte slots. §15.9 walks why this works for the test suite (one FP variadic argument is the maximum case) and names it as an errata candidate (multi-FP-variadic-call would break). Rui hasn't fixed this; the test suite never exercises the bug.
6. **The integer-promotion gap in §15.8 named as errata candidate.** C standard says `char` and `short` arguments to variadic calls also get promoted (to `int`). Chibicc only handles the `float → double` case. Tests don't depend on the integer side because the call-site ABI for `int` and `char` happens to land in the same register and `%d` reads tolerate it. Errata candidate.
7. **`mov $0, %rax` still emitted for variadic FP calls.** §15.6 doesn't fix the variadic-safety zero to actually count XMM registers used. Calls into variadic functions with FP arguments would technically be wrong (psABI requires `%al` to hold the XMM-register count for variadic calls), but glibc's variadic functions tolerate `%al = 0` when their format string doesn't ask for FP args, and the test suite's single variadic-FP-call (`fmt(buf, "%.1f", (float)3.5)`) reads from the spill area regardless of `%al`. Errata candidate, lower priority.
8. **psABI conformance thread continued.** §15.6 (XMM argreg track), §15.7 (XMM prologue spill), §15.8 (default argument promotion), §15.9 (`fp_offset` correction). Three corrections in this chapter; thread now stands at eight (Ch 13 §13.8/§13.9 + Ch 14 §14.1/§14.2/§14.8 + Ch 15 §15.6+§15.7 + §15.8 + §15.9).

### Voice / structure inherited from Ch 1–14

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with one feature table (eleven rows).

### Three careful avoidances

- **Did not invent a System V SSE register convention interlude.** The handoff defaulted to no, and the §15.6/§15.7 prose carried the design walk inline split across the call/definition pair. Pulling it out would have left both sections without bodies (each on their own would have been ~400 words; together with the design walk inline they're each ~800–900).
- **Did not invent an IEEE 754 bit-pattern interlude.** The §15.1 prose walks the union-based bit-pattern extraction and the `mov $imm, %rax; movq %rax, %xmm0` sequence inline. The IEEE 754 layout is referenced but not derived from scratch — readers either know IEEE 754 or can look it up; the chapter doesn't try to teach it.
- **Did not invent forward references that don't exist.** The §15.10 closer names `eval_double` arms that don't exist (no `ND_MOD`, no bitwise, no shifts) — those are walked as *correctly omitted*, not as forward references to fill in later. The chapter doesn't claim future commits will fill them; they shouldn't be filled, by C semantics.

### Date-vs-position note

This chapter's calendar dates scatter again. `8b14859` (default arg promotion) is dated April 30, 2020; `e452cf7` (variadic floats) is dated May 1, 2020 — both *earlier* than most of the chapter's commits, which cluster mid-September 2020. Rui's commit-rebasing-style historical reordering is again on display: the variadic-floats fix logically depends on the call/definition pair (§15.6/§15.7), but its calendar date is months earlier. The chapter follows `main` order without comment, the same as Ch 7–14.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The `is_unsigned` flag on `Type`** is irrelevant to floats (per Ch 14 forecast — held). New axes get new kinds.
- **The cast table is 10×10.** Future axis additions (`_Decimal32`, true `long double`, hypothetical SIMD types) would extend it again. The flag-or-kind decision follows the §14.5 precedent: orthogonal axes use a flag; new dimensions get new kinds.
- **The constant-expression evaluator is now a quartet** (`eval`/`eval2`/`eval_rval`/`eval_double`). `eval2` dispatches to `eval_double` by `is_flonum(node->ty)` at the top. `eval_double` falls through to `eval` for integer-typed nodes via `is_integer(node->ty)` check.
- **The argreg track is now two-track for non-variadic, plus a separate spill-everything path for variadic.** The non-variadic path uses `argreg64` for integer (six wide) and `%xmm0`–`%xmm7` for FP (eight wide). The variadic path spills all 14 registers regardless of how many parameters were declared.
- **The recursive `push_args` pattern** for argument evaluation supersedes the counter-and-index pattern. Future register-file additions (a hypothetical SIMD-vector track) would follow the same recursive pattern.
- **The `fp_offset = fp * 8 + 48` formula** is psABI-non-conforming for multi-FP-argument variadic calls (real ABI says 16-byte stride). Works for the test suite's single-FP-argument case. Errata candidate.
- **Float literals are inlined as integer-immediate-bit-cast, not anonymous-global.** The Ch 13 anonymous-global pattern was *not* reused in Ch 15 §15.1. String literals (Ch 7) and compound-literal-arrays (Ch 13) still use anonymous globals; floats don't.
- **The `__va_area__` magic local-variable name** continues to be the chibicc-specific user-side hook. Ch 15 §15.9 reads from the FP half of the area for the first time but doesn't change the magic-name mechanism.
- **`long double` is `double`.** Errata candidate. No `TY_LDOUBLE`, no x87 stack, no extended precision. This is a deliberate simplification.
- **The default-argument-promotion gap** (chars and shorts not promoted to int for variadic calls) is an errata candidate. Tests don't depend on it.
- **The `mov $0, %rax`** is still emitted for every call, including variadic-FP calls (where it should be the XMM count, not 0). Errata candidate.
- **Canonicalization-at-parse-time count is unchanged at eight.** Ch 15 doesn't add AST rewrites at parse time.
- **Pre-factor-before-feature count is now seven.** §15.6's recursive `push_args` is the new pre-factor. The §15.2 `load`/`store` switch reshape is a smaller refactor in the same commit as the float-load feature; doesn't count separately.
- **The fourth namespace (labels)** is unchanged. Ch 15 doesn't add a fifth.
- **The `is_typename` predicate** grows by two (`float`, `double`).
- **The VarAttr channel stays at four fields.** Ch 14 forecast was wrong (Ch 13 had predicted growth that didn't happen); Ch 15 forecast was right (no growth predicted, no growth happened).
- **`is_numeric(Type *ty)` joins `is_integer` and `is_flonum`** as a parser-side type discriminator. Used by `new_add` and `new_sub`'s pointer-arithmetic gate.
- **The `unreachable()` macro** lives in `chibicc.h`. Ch 15 adds one new caller: `store_fp`'s default arm.
- **The Initializer tree** (Ch 12 §12.1) — unchanged in Ch 13, Ch 14, and Ch 15. Float initializers reuse the existing tree shape via `write_gvar_data`'s new `TY_FLOAT`/`TY_DOUBLE` arms.
- **The Relocation mechanism** (Ch 12 §12.7) — no new uses in Ch 15. (Float literals don't need relocations because they're inlined.)
- **The `Obj->tok` field** added in Ch 14 §14.11 still has no readers. Reserved for later use; watch for callers in subsequent chapters.
- **The `Type->name_pos` field** (Ch 14 §14.11) — no new uses in Ch 15.
- **The `is_definition` flag on `Obj`** (Ch 13 §13.1) — used today only by `extern`. No new uses in Ch 14 or Ch 15.
- **The `is_static` default in `new_gvar`** (Ch 13 §13.6) — unchanged. No new `new_gvar` callers in Ch 15.
- **Tests are in C** as of Ch 8 §8.2.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **psABI conformance thread** stands at eight corrections after Ch 15. Ch 16 (the compiler driver) probably won't add to this thread — driver-shape changes are mostly orthogonal to ABI.
- **Ch 1 errata list** unchanged.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) — errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4) — errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels — four errata candidates.
- **`f()` and `f(void)` are accepted as identical** by chibicc, with §14.3 adding a third equivalence (`f()` becomes variadic). Errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension matching GCC.
- **`.bss` is the third assembly section.** Section list: `.text`, `.data`, `.bss`.
- **The `>>` codegen quirk** (Ch 11 §11.13) — partially repaired by §14.5's `sar`/`shr` split. The width selection now uses `ax`. Ch 15 §15.4 closes one more case (bitwise-op width fix slipped in alongside the float arithmetic).
- **The "more than 6 integer args silently miscompiles"** in Ch 5 §5.4 — errata candidate. Ch 15 adds a parallel "more than 8 FP args silently miscompiles" — errata candidate.
- **The `mov $0, %rax`** (Ch 5 §5.1) — closed loop in Ch 14 §14.1. Still pessimistic (every call). Plus the variadic-FP-call wrongness from Ch 15 §15.6.

## Exit state

- `chapters/15-floating-point.md` drafted, ~11,995 words.
- Session 016 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 017 (Chapter 16 — The compiler driver, commits 150–157).
- CLAUDE.md status note updated (chapter count goes from "Ch 14 drafted" to "Ch 15 drafted").
