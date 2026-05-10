# Session 022 — Chapter 21 draft

**Date:** 2026-05-10 (continuation of the autonomous-drafting run that has produced sessions 002–021).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 021 delivered Ch 20 (GCC extensions worth supporting, twenty-two commits, ~9,994 words). User direction is still autonomous — no chapter-by-chapter review. Ch 21 covers commits 267–283: seventeen commits — thread-local variables, three driver options (`-include`, `-x`, `-E` implies `-xc`), `alloca`, the four-commit VLA arc, four linker-driver additions (`-l`, `-s`, ELF size/type, `.a`/`.so`), and four §21.6 mixed features (`long double`, case ranges, array range designators, labels-as-values).

## What was done

### Drafting decisions

- **Length:** ~10,654 words. Right in the handoff's 10,000–12,000-word target range. The chapter's commits are mixed in scale: thread-local (~1,400 words), driver triple (~1,200), `alloca` (~1,500), VLAs (~3,000 — the largest section), linker-driver quartet (~1,400), and §21.6 quartet (~3,200 — long double dominates with ~1,800 of those). No padding; everything lands.
- **Section structure:** 6 sections from 17 commits, exactly as the handoff proposed. §21.1 single-commit, no subsections needed (just a flowing walk through the AMD64 TLS pattern). §21.2 three commits with three named subsections. §21.3 single-commit, no subsections. §21.4 four commits with ten named subsections (the largest section by far — VLAs are genuinely complex). §21.5 four commits with four named subsections. §21.6 four commits with ten named subsections (long double dominates).
- **No concept interlude.** The handoff said "possibly one" around the VLA arc as a "dynamically-sized stack allocations" interlude. Reading the §21.4 prose, the through-line from `compute_vla_size` to `new_alloca` to `ND_VLA_PTR` to the `gen_addr` arms hangs together as a single arc; pulling out a separate interlude would have introduced repetition. The §21.4.1–§21.4.10 subsection chain is enough scaffolding.
- **§21.4 makes the VLA-vs-alloca lineage explicit.** §21.3's `alloca` builtin is what §21.4 reuses for VLA allocation — the `new_alloca` helper builds an `ND_FUNCALL` to the same synthesized `Obj`. This is one of the few places where reading-order and writing-order pay off: walking `alloca` first makes the VLA implementation feel inevitable.
- **§21.6 closes two errata candidates** — `long double` as `double`, and the §19.7 array range designator. Both are explicitly named in the §21.6 closer and the chapter recap.
- **§21.6.2–§21.6.3 walks the cast table growth from 10×10 to 11×11 explicitly,** including the F80 row's `FROM_F80_1`/`FROM_F80_2` macros and the unsigned-64-to-F80 sign-test trick. Long double codegen is mostly x87 instruction soup and the section names what every instruction does without overspecifying the FPU stack model.
- **The psABI conformance count grows by two** (TLS access patterns + long double calling convention). New count: eighteen, up from sixteen.
- **Canonicalization-at-parse-time count ticks from ten to eleven** with the VLA declaration rewrite (`int x[n]` → `compute_vla_size; x = alloca(...)`).
- **Two new errata candidates added in §21.5:** missing `.size` directive for function symbols (8d130ab), and suffix-only `.a`/`.so` recognition that won't catch versioned shared libraries (d56dd2f). Both noted in §21.5 prose and the chapter recap.
- **One-table recap** at the chapter close, seventeen rows. Same one-table-with-section-column shape as Ch 20.

### Interpretive calls

1. **TLS access model named as initial-exec.** §21.1 calls out that chibicc emits the cheapest TLS model (`%fs:0` + `name@tpoff`), which works for variables in the executable but not for dynamically-loaded shared-library TLS (which would need `__tls_get_addr` and the global-dynamic model). Rui's choice; the prose names it.
2. **`Makefile`'s `-pthread` link flag** is named in §21.1 as the test-plumbing piece that lets the new `test/tls.c` link against libpthread. This is a test-infrastructure consequence, not a language feature.
3. **The `-pthread` removal of `__STDC_NO_THREADS__`** is given a paragraph that distinguishes language-level threads (TLS, the underlying compiler feature) from library-level threads (`<threads.h>`, which chibicc still doesn't implement). Honest framing.
4. **The `-include`/`-x`/`-E xc` rule is treated as one logical group** even though commit dates span ten days and `-E xc` is partly a reaction to test churn introduced by `-x`. The §21.2.3 prose names this — "three commits later (about a week of calendar time), Rui revisits the test churn."
5. **§21.3's `alloca` description is candid about safety.** Stack-exhaustion isn't detected; the eight-byte cost per function is named; the function-epilogue dependency that makes the implementation work is named.
6. **§21.4's VLA arc is walked subsection-by-subsection** with an explicit ordering: type kind first, then dimension parser, then `is_const_expr`, then `compute_vla_size`, then declaration rewrite, then `ND_VLA_PTR`, then `gen_addr`, then pointer arithmetic, then `sizeof(VLA)` and `sizeof(typename)`, then `__STDC_NO_VLA__`. The reader gets the pieces in dependency order rather than commit order (the four commits are interleaved across the subsections).
7. **§21.4.4 explains the comma-expression chain** built by `compute_vla_size` with a worked example for `int x[m][n]`. The `ND_NULL_EXPR` seed is named.
8. **§21.4.6 explains why `ND_VLA_PTR` exists** (the assignment target is the slot's address, not the pointer-stored-in-the-slot — analogous to the `&arr` vs `arr` distinction).
9. **§21.5.3's `.size` for functions is noted as missing.** Real toolchains emit `.size name, .-name`; chibicc doesn't. The prose names this and routes it to errata.
10. **§21.5.4's suffix-only file-magic recognition** is named as a real-world limitation. Versioned shared libraries (`libfoo.so.1.2.3`) won't be recognized. Errata candidate.
11. **§21.6.1 makes "long double is double" closure explicit** with a side-by-side diff of the type-kind switch.
12. **§21.6.4's operand-order quirk in `fsubrp`/`fdivrp`** is named — the x87 stack puts the second-pushed operand on top, so the `r` (reverse) instructions are needed.
13. **§21.6.5's `has_flonum` divergence from `is_flonum`** is given a paragraph. The two predicates were unified through Ch 20; long double splits them: `is_flonum` says "yes for any FP type"; `has_flonum` says "yes only for FLOAT and DOUBLE because LDOUBLE doesn't go through the SSE eight-byte classification."
14. **§21.6.6's redzone use** is named as the universal scratchpad for x87↔memory↔SSE transitions.
15. **§21.6.8 names the unsigned-comparison trick for case ranges** — subtract `begin`, compare against `end - begin` unsigned, jump if below-or-equal. The wraparound makes out-of-range values compare above. Standard compiler trick, worth naming.
16. **§21.6.10 names the `&&label` resolution path** as reusing the existing forward-`goto` machinery — `gotos` chain, `resolve_goto_labels` walk. No new pass.

### Voice / structure inherited from Ch 1–20

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, all hashes listed at the top.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- One-table recap at the chapter close (with §-section column).
- No concept interludes.

### Three careful avoidances

- **Did not invent a "history of TLS models" interlude.** The Linux TLS implementation has multiple models (initial-exec, local-exec, general-dynamic, local-dynamic) with different cost/flexibility tradeoffs. The chapter names initial-exec as what chibicc does and points at the existence of the others without walking through them. A history walk would be a detour — chibicc's choice is the simple one.
- **Did not invent a "history of VLAs" interlude.** VLAs were added in C99 and made optional in C11; gcc had them earlier in proprietary forms. The chapter cites both standardization milestones in passing but doesn't walk the full history. Standard chibicc-book convention.
- **Did not over-explain the x87 FPU stack.** The §21.6 long-double codegen names the stack-based operand model, the operand-order quirks of `fsubrp`/`fdivrp`, and the FPU control-word manipulation for truncation casts. It doesn't walk the full eight-element x87 stack model or the deprecated 287/387 history. Acceptable — chibicc emits what it emits, and the prose explains what each instruction does.

### Date-vs-position note

The seventeen commits scatter across late-August and September 2020. The four VLA commits in particular span August 25 (`2fa8f48`, `b0109a3`) and September 3–4 (`07f9010`, `e8667af`), which means in main order the *earlier-written* commits land *later* (positions 274 and 275). Rui's branches landed in the order of dependent functionality rather than in chronological order. The chapter follows main order and notes this in §21.4's opening paragraph but doesn't dwell on it.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The TLS access model is initial-exec only.** `mov %fs:0, %rax; add $name@tpoff, %rax`. Dynamic-library TLS (which would call `__tls_get_addr`) is not implemented. Chibicc-compiled object files used in shared libraries with TLS variables will not work correctly with thread-local storage in those libraries. Named in §21.1.
- **`Obj->is_tls`** is set in `global_variable` from `attr->is_tls`. Tentative-definition handling is suppressed for TLS — a bare `_Thread_local int x;` goes to `.tbss`, not `.comm`.
- **`Obj->alloca_bottom`** is a hidden per-function local that records the current bottom of the temporary-evaluation area. Set in the prologue to `%rsp`; updated by `builtin_alloca`. Costs eight bytes per function whether `alloca` is called or not.
- **`alloca` is a builtin.** A `void *alloca(int)` is synthesized at parse start; codegen recognizes `ND_FUNCALL` to a variable named `alloca` and emits the inline shift sequence.
- **`builtin_alloca` (in parse.c) holds a file-static reference to the synthesized `Obj`** so the VLA code can build `ND_FUNCALL` nodes without re-looking up. The original `alloca` declaration is registered as a global; `builtin_alloca` is the captured pointer.
- **`TY_VLA`** is the new type kind. `Type` carries `vla_len` (parsed expression for the dimension) and `vla_size` (a hidden local for the runtime byte count). VLA-typed locals are 8 bytes on the stack — pointers to alloca'd buffers.
- **`compute_vla_size`** builds an `ND_COMMA` chain that evaluates dimensions in declaration order and stores byte counts into `vla_size` locals. Multi-dimensional VLAs cascade — inner row size goes into the inner `vla_size`, then outer total goes into the outer `vla_size`.
- **`is_const_expr`** is the fifth eval-quartet member. Structural recursion that returns false on anything not statically computable. Used by `array_dimensions` to decide between `array_of` and `vla_of`.
- **`ND_VLA_PTR`** is the new node kind for "address of a VLA-typed local's slot" — distinct from `ND_VAR` (which loads the pointer stored in the slot). `gen_addr` emits `lea offset(%rbp)`; `add_type` reuses `ND_VAR`'s rule.
- **`-l NAME`** flows through `input_paths` (not `ld_args`) to preserve command-line ordering relative to filename inputs. The disposition loop identifies it by prefix and pushes to `ld_args` in order.
- **`-s`** flows through a new `ld_extra_args` channel, distinct from `ld_args` because order doesn't matter for `-s`.
- **`.type @object`/`@function`** and **`.size`** are now emitted for data and function symbols. `.size` is missing for functions (chibicc doesn't track function size as a parse artifact). Errata candidate.
- **`.a`, `.so`, `.o`** are recognized by suffix only. Versioned shared libraries (`libfoo.so.1.2.3`) won't be recognized. Errata candidate.
- **`get_file_type`'s order** changed in `d56dd2f`: `opt_x` is now checked before `.o` (so `-x assembler foo.o` would treat `foo.o` as assembler). Real-world usage doesn't trigger this; flagged for completeness.
- **`TY_LDOUBLE`** is the new floating-point kind. Size 16, alignment 16. Codegen uses x87 stack (`fld`, `fst`, `faddp`, etc.). Function arguments go on the stack (not in xmm regs). `is_compatible` accepts cross-FP compatibility; `get_common_type` promotes to `long double` if either operand is `long double`.
- **The cast table is now 11×11** — F80 row and column added. F80 conversions to integer use the FPU control-word truncation trick (`fnstcw`/`fldcw`); F80 conversions across SSE-FP go through the redzone.
- **The `Token`/`Node` `fval` widened from `double` to `long double`.** `convert_pp_number` reads `strtold`. The literal-emission code writes the bits as two halves through the redzone.
- **`is_flonum` and `has_flonum` diverge.** `is_flonum` returns true for FLOAT, DOUBLE, LDOUBLE; `has_flonum` (struct classification) returns true only for FLOAT, DOUBLE.
- **The keyword list is now around thirty-two entries** with `_Thread_local` and `__thread` added.
- **Case ranges** generate inline range checks via the unsigned-subtract-and-compare trick. `node->begin` and `node->end` carry the bounds. Single-value cases use `begin == end` as the degenerate case.
- **Array range designators are honored in elaboration.** `array_designator` returns `(begin, end)`; the two callers (`designation` and `array_initializer1`) loop. §19.7 errata closed.
- **Labels-as-values** uses `gotos` chain for resolution; `&&label` is `ND_LABEL_VAL` (rip-relative `lea`); `goto *expr` is `ND_GOTO_EXPR` (indirect `jmp`). Compile-time-constant variant (`f0c98e0`) is in Ch 22, not here.
- **psABI conformance count is at eighteen** (up from sixteen). New: TLS access pattern, long-double calling convention.
- **Canonicalization-at-parse-time count is at eleven** (up from ten). New: VLA declaration rewrite to `compute_vla_size; x = alloca(...)`.
- **Pre-factor-before-feature count is at nine** (unchanged).
- **Errata candidates closed in Ch 21:** "long double is double" (closed by `e0bf168`); array range designators not honored (closed by `3d5550e`).
- **Errata candidates added in Ch 21:** missing `.size` for function symbols (`8d130ab`); suffix-only `.a`/`.so` recognition (`d56dd2f`).
- **Errata candidates remaining:** Ch 17's three (#error doesn't print message text, opt_S | opt_E typo, default include paths Linux/glibc-specific), Ch 19's two (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block), Ch 20's one (`is_compatible` array arm bug), Ch 21's two new.
- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean — unchanged.
- **Chibicc compiles itself** — unchanged.

## Exit state

- `chapters/21-thread-local-alloca-vlas.md` drafted, ~10,654 words.
- Session 022 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 023 (Chapter 22 — Performance, dependency files, and the linker driver, commits 284–306, 23 commits).
- CLAUDE.md status note updated to "Ch 21 drafted".
