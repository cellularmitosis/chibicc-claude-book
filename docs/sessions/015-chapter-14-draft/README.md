# Session 015 — Chapter 14 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that has produced sessions 002–014).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 014 delivered Ch 13 (Linkage, eleven commits, ~8,600 words). User direction is still autonomous — no chapter-by-chapter review. Ch 14 covers commits 127–138: variadic call sites and `va_start`, `signed` and `unsigned` keywords, integer literal suffixes, the wider-`sizeof`/`ptrdiff` ripple, pointer comparison as unsigned, unsigned arithmetic in the constant evaluator, the qualifier soup (`const`/`volatile`/`auto`/`register`/`restrict`/`_Noreturn`), array-dimension qualifiers, and unnamed parameters in declarations. Twelve commits — slightly more than Ch 13.

## What was done

### Drafting decisions

- **Length:** ~12,150 words. Inside the handoff target of 10,000–12,000 (just over the upper edge). The chapter ran at the predicted size because §14.2 (the `va_start` register-save area) and §14.5 (unsigned types throughout the codegen) both got full-length treatments — neither could be compressed without losing the design walk that justifies the section. The smaller commits (§14.3, §14.4, §14.8, §14.11) got proportionally smaller sections. Three sections (§14.6, §14.7, §14.10) were medium, around 700–1,000 words each.
- **Section structure:** 11 sections, no concept interlude. Followed the handoff's bundling proposal exactly:
  - §14.10 bundled commits 136, 137 (qualifier soup + array-dimension qualifiers). Both are parse-and-discard in the same spirit; bundling let the section name the trade-off once rather than splitting it across two near-duplicate sections.
  - All other sections single-commit (per handoff).
- **No concept interlude.** The handoff defaulted to no interlude on the System V psABI register-save-area design; the §14.2 prose carried it inline. The design *is* the section, and excerpting it would have left the section without a body. Same call as session 013/014 to default-no on interludes when the prose carries the explanation.
- **§14.1 closes the Ch 5 §5.1 forward-reference** — handoff acceptance criterion. The `mov $0, %rax` is named in the §14.1 prose: "the variadic-safety zero finally has a customer." Closing line in the section: "The codegen doesn't change."
- **§14.2 walks the System V x86-64 register-save-area design** — handoff acceptance criterion. The four `va_list` header fields (`gp_offset`, `fp_offset`, `overflow_arg_area`, `reg_save_area`) are walked with their layout rules. The 136-byte size is decomposed (24 + 48 + 64). The chibicc trick of exposing `__va_area__` as a magic local-variable name (rather than implementing `va_start` as a builtin) is named explicitly.
- **§14.5 walks the unsigned-types impact on codegen** — handoff acceptance criterion. The four codegen pieces (cast table, `_Bool` cast / load extension, division, comparison, shift) are each given their own subsection. The `get_common_type` rule is walked. The flag-on-existing-kind versus separate-kind design call is named.
- **§14.6 explains how integer suffixes flow from tokenizer to type assignment** — handoff acceptance criterion. The `Token->ty` field's second use (after string literals) is named. The `add_type` for `ND_NUM` simplification (now overwritten by `primary` after a `tok->ty` copy) is called out as a small regression that the `primary` overwrite covers.
- **§14.7 names the `sizeof`-now-returns-`ulong` ripple effect** — handoff acceptance criterion. The §14.5 usual-arithmetic-conversion interaction is named.
- **§14.8 names *why* pointer comparisons should be unsigned** — handoff acceptance criterion. The address-space-wraps walk is in the prose: 0x7FFF…FFFF vs 0x8000…0000 example.
- **§14.10 names the parse-and-discard pattern as a deliberate trade-off** — handoff acceptance criterion. Last paragraph of §14.10: chibicc's codegen doesn't optimize, so most of the qualifier semantics aren't relevant; `const` enforcement would be a real diagnostic improvement but isn't on the roadmap.
- **One-table recap.** Twelve rows. Resisted splitting into two tables (signedness vs everything else), per the handoff's "consider whether splitting" note. The chapter's commits don't divide cleanly into two themes — the threads are *variadics* (§14.1, §14.2, §14.3), *signedness* (§14.4, §14.5, §14.6, §14.7, §14.9), *psABI* (§14.8 plus the variadics ones), and *qualifiers* (§14.10, §14.11). Three or four threads is too many to split.

### Interpretive calls

1. **VarAttr forecast was wrong.** Session 013/014 forecast that signed/unsigned and the qualifier set would route through `VarAttr`, growing it to 9+ fields. Reading the actual diffs: signed/unsigned land in `declspec`'s bit-flag counter machinery (parallel to `VarAttr` but distinct), and the qualifier set goes nowhere — chibicc doesn't track them. The channel stays at four fields. The closer prose owns this: "The forecast wasn't quite right."
2. **`is_unsigned` flag-on-existing-kind versus separate-kinds.** The chapter calls this out twice (in §14.5 and in the closer) as a structural design choice. The flag-on-existing-kind doubles cell count for the cast table but adds only a single conditional to most other code paths; separate-kinds would multiply switch cardinality everywhere. Worth tracking as a precedent for future axis-doubling decisions.
3. **§14.4 and §14.5 not bundled.** The signed and unsigned commits are sibling features but very different in size — `signed` is parse-and-discard with no codegen change, while `unsigned` is a 90-line codegen change. Bundling would have buried the unsigned commit's substance under a "siblings" framing that doesn't quite fit. The two sections share the `declspec` enum walk; the second section refers back to the first rather than re-deriving it.
4. **§14.10 bundling decision.** Commits 136 and 137 are both *parse-and-discard*. Bundled because they share the trade-off: chibicc accepts the syntax, doesn't enforce the semantics. Splitting would have produced two near-duplicate sections. The §14.10 prose names both commits in its blockquote pair and walks them in order, but the trade-off paragraph at the end speaks to both.
5. **§14.2 is the chapter anchor.** ~1,800 words on a 30-line patch. The walk-through includes the full prologue spill instruction listing, the four header fields' layout, the `__va_area__` user-side trick, and the integration-with-glibc story (chibicc's prologue layout matching glibc's vsprintf expectations byte-for-byte). The handoff predicted this would carry the chapter; it does.
6. **The `__va_area__` trick is a chibicc-specific design call.** Real C compilers implement `va_start` as `__builtin_va_start`. Chibicc exposes the spill area by name and lets user code construct the `va_list` with a struct copy. The difference is where the work happens; the runtime cost is identical. Worth tracking as a chibicc-specific approach that simplifies the compiler at the cost of a quirky user-side macro.
7. **psABI conformance thread continued.** §14.1 closes the `mov $0, %rax` loop, §14.2 lays out a psABI-conformant register-save area, §14.8 fixes pointer comparison to unsigned. All three are chibicc-side corrections to match calling-convention guarantees. The closer prose names this as a continuation of the Ch 13 §13.8/§13.9 thread.
8. **Forecast for Ch 15 floating-point.** The §14.2 spill area populates the XMM register slots but no `va_arg` arm currently reads them. Ch 15 will introduce `float`/`double` and start populating/reading the fp half of the spill area. Forecast in the closer.

### Voice / structure inherited from Ch 1–13

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote (multiple openers for the bundled §14.10).
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with one feature table (twelve rows).

### Three careful avoidances

- **Did not invent a System V psABI interlude.** The handoff defaulted to no, and the §14.2 prose carried the design walk inline. Pulling it out would have left the section without a body.
- **Did not over-derive the `declspec` counter machinery in §14.4.** The bit-flag accumulator has been there since Chapter 9; §14.4 walks the new arms (`SIGNED`) without re-deriving the framework. §14.5 then refers back to §14.4 without re-walking the `UNSIGNED` mechanism.
- **Did not invent forward references for floating-point in the body of the chapter.** Floating-point is queued for Chapter 15. The §14.2 spill area's XMM half is mentioned (it's spilled but unused), but the forward reference is named only in the closer, not threaded through the chapter prose.

### Date-vs-position note

This chapter has the largest temporal scatter so far. `754a24f` (`va_start`) is dated August 2019; `aaf1045` (suffixes) is September 2019; `8b8f3de` (sizeof-as-ulong) and `7ba6fe8` (eval-unsigned) are March 2020; the rest cluster late August through October 2020. The chapter follows `main` order without comment, the same as Ch 7–13. The interleaving is more striking than usual but not commented on in chapter prose.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The `is_unsigned` flag on `Type`** is a per-kind orthogonal axis. Ch 15's floating-point types (`TY_FLOAT`, `TY_DOUBLE`) won't use it. The flag pattern's appeal is that it scales linearly when adding new orthogonal axes; future axes (atomic, thread-local, address-space) could use the same pattern.
- **The VarAttr channel stays at four fields** (`is_typedef`, `is_static`, `is_extern`, `align`). The forecast that signed/unsigned and qualifiers would route through `VarAttr` was wrong — those flags live on `Type` (signedness) or nowhere (qualifiers). The channel hasn't grown since Ch 13.
- **The `declspec` bit-flag counter** has nine flags now (after `SIGNED` and `UNSIGNED`): `VOID`/`BOOL`/`CHAR`/`SHORT`/`INT`/`LONG`/`OTHER`/`SIGNED`/`UNSIGNED`. The shape (counter increments for "doublable" specifiers like `LONG`, OR-equals for "single-shot" specifiers like `SIGNED`) is stable. Future additions would follow the same pattern.
- **The cast table is now 8×8.** Ch 15's float types will probably extend it again — though it might not be in the same matrix (float casts use entirely different instructions; could be a separate small table or a different switch).
- **The constant-expression evaluator** (`eval`/`eval2`/`eval_rval`) gained unsigned-arithmetic arms in §14.9. The trio's shape is unchanged. Ch 15 will likely add float arms, possibly with a parallel `eval_double` evaluator since `eval2` returns `int64_t`.
- **The register-save-area layout** (`__va_area__`, 136 bytes, 24-byte header + 48 gp + 64 fp) is psABI-conformant. The fp half is currently spilled but no `va_arg` reads it. Ch 15 will start populating the fp half with real values and introduce a `va_arg(ap, double)` path (probably also as user-side macros).
- **The `__va_area__` magic name** is a chibicc-specific user-side trick for `va_start`. Worth tracking as a quirk: real C compilers don't expose anything named `__va_area__`; users porting chibicc-compiled code to other compilers would find this surprising.
- **The qualifier accept-and-discard pattern** is now well-established. Future qualifier additions (`_Atomic`, `_Thread_local`, GNU `__attribute__((...))`) would route through the same `consume`/`continue` block in `declspec`.
- **The `pointers` helper** (§14.10) is a small refactor extracting the `*`-and-qualifiers loop out of `declarator`/`abstract_declarator`. Both call it; the extraction is uniform.
- **The unnamed-parameter relaxation** (§14.11) makes `int f(int);` legal. Combined with §14.3's empty-list-as-variadic, chibicc collapses three real-C distinctions: `f()` (unspecified parameters in K&R), `f(void)` (no parameters), and `f(int)` (one unnamed parameter). All three parse; only the third's behavior is fully spec-aligned.
- **The `mov $0, %rax`** (Chapter 5 §5.1) is now closed-loop. It's still emitted before *every* call (chibicc doesn't know which callees are variadic at codegen time without symbol-table lookup). The pessimism costs one instruction per call.
- **The `Obj->tok` field** added in §14.11 is a "representative token" but isn't used by any of the §14.11 changes. Reserved for later use; watch for callers in subsequent chapters.
- **The `Type->name_pos` field** (§14.11) is the always-set position pointer for use in error messages when `name` is null. Worth tracking — it's a small but real new piece of `Type` state.
- **Canonicalization-at-parse-time count is unchanged at eight.** Ch 14 doesn't add AST rewrites at parse time.
- **Pre-factor-before-feature count is unchanged at six.** The `pointers` helper in §14.10 is a refactor done as part of the same commit (not a separate pre-factor).
- **Test file additions:** `test/compat.c` (commit 136 — qualifier soup), `test/const.c` (commit 136 — const tests). Other commits added to existing test files.
- **The `is_typename` array** grew significantly: nine new keywords across §14.4, §14.5, and §14.10 (`signed`, `unsigned`, `const`, `volatile`, `auto`, `register`, `restrict`, `__restrict`, `__restrict__`, `_Noreturn`). The shape stays the same.
- **The `ND_MEMZERO` `mov $0, %al`** (`rep stosb` source byte, since Ch 12) is unrelated to variadics' `mov $0, %rax`. Both are tiny zero-loads but for different reasons. Worth distinguishing if they ever appear near each other in commentary.
- **psABI conformance thread:** Ch 13 §13.8/§13.9 + Ch 14 §14.1/§14.2/§14.8. Five corrections so far. Chapter 15 (floating-point) will probably continue the thread with the XMM-register passing convention.
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Tests are in C** as of Ch 8 §8.2.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17.
- **The argreg 8/16/32/64 split** is fully in place. Ch 14 didn't touch it. Ch 15 will add an XMM register set for floating-point args.
- **The `unreachable()` macro** lives in `chibicc.h`. Used by `store_gp`, `declspec`, `write_buf`. Ch 14 didn't add new callers.
- **`copy_struct_type`** (Ch 12 §12.10) — no new uses in Ch 14.
- **The `MIN`/`MAX` macros** (Ch 12 §12.3) — no new uses in Ch 14.
- **The Initializer tree** (Ch 12 §12.1) — unchanged in Ch 13 and Ch 14.
- **The anonymous-global pattern** (Ch 13 §13.4, §13.6) — no new uses in Ch 14, but Ch 15 will likely use it for floating-point literals.
- **The `is_static` default in `new_gvar`** (Ch 13 §13.6) — unchanged. No new `new_gvar` callers in Ch 14.
- **The `is_definition` flag on `Obj`** (Ch 13 §13.1) — used today only by `extern`. No new uses in Ch 14.
- **The Relocation mechanism** (Ch 12 §12.7) — unchanged. No new uses in Ch 14.
- **Ch 1 errata list** unchanged.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) — errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4) — errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels — four errata candidates.
- **`f()` and `f(void)` are accepted as identical** by chibicc, with §14.3 adding a third equivalence (`f()` becomes variadic). Real C distinguishes all three. Errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension matching GCC.
- **`.bss` is the third assembly section.** Section list: `.text`, `.data`, `.bss`.
- **The `>>` codegen quirk** (Ch 11 §11.13) — partially repaired by §14.5's `sar`/`shr` split. The width selection now uses `ax` (which picks `%rax` or `%eax` based on operand size).
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4 — errata candidate. Variadic functions in §14.2 spill all six integer args plus eight XMM args, but a variadic call with seven integer args still wouldn't work because the seventh would need to be on the stack and chibicc's call-site codegen doesn't push extra args.
- **The `mov $0, %rax`** (Ch 5 §5.1) — closed loop in Ch 14 §14.1. Still pessimistic (every call) for the reason noted above.

## Exit state

- `chapters/14-variadics-signedness-qualifiers.md` drafted, ~12,150 words.
- Session 015 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 016 (Chapter 15 — Floating point, commits 139–149).
- CLAUDE.md status note updated (chapter count goes from "Ch 13 drafted" to "Ch 14 drafted").
