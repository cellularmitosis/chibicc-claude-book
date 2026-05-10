# Session 019 — Chapter 18 draft

**Date:** 2026-05-10 (continuation of the autonomous-drafting run that has produced sessions 002–018).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 018 delivered Ch 17 (A preprocessor from scratch, forty commits, ~17,134 words). User direction is still autonomous — no chapter-by-chapter review. Ch 18 covers commits 198–220: twenty-three commits, the full SysV AMD64 calling convention plus bitfields plus a handful of polish commits, ending in anonymous struct/union members.

## What was done

### Drafting decisions

- **Length:** ~14,357 words. Above the 10,000–13,000-word handoff forecast by ~1.4K. The bitfield arc (§18.5) and struct-by-value arc (§18.2) both wanted full per-commit treatment, and §18.6 had six commits that — while individually small — added up. The chapter ran to its natural length without padding. If the chapter felt long anywhere, it would be in the §18.5 op-assign subsection (~700 words on a single commit), but that commit's parser-side rewrite is intricate enough that compressing it would have lost substance.
- **Section structure:** 6 sections from 23 commits, exactly as the handoff proposed. §18.1 (2 commits, stack args) gets two integrated subsections-within-prose without explicit numbered sub-headers, since the two commits are caller-side / callee-side mirror commits. §18.2 (4 commits, struct-by-value) gets four named subsections because each commit has its own substance. §18.3 (2 commits, variadic + va_copy) gets two subsections. §18.4 (4 commits, small completions) gets four named subsections. §18.5 (5 commits, bitfields) gets five subsections, one per commit, no skipping. §18.6 (6 commits, polish) gets six subsections.
- **No concept interlude.** The handoff defaulted to "likely no" with a conditional escape on §18.2 if the prose felt overstuffed; reading the §18.2 prose, the eightbyte classification fits inline without straining. The interlude on the SysV AMD64 ABI's classification rule that was tempting in the handoff would have been a detour — the *why* is "the SysV ABI says so" and the *how* is the `has_flonum` walk, which the prose carries.
- **§18.1 names the closure of two errata items.** The "more than 6 integer args silently miscompiles" (Ch 5 §5.4) and "more than 8 FP args silently miscompiles" (Ch 15 §15.6) are explicitly closed by the section, both in the section opening and the recap.
- **§18.2 walks the SysV AMD64 eightbyte classification.** The per-chunk classification, the worst-class merge rule (implemented as `has_flonum`'s recursive AND-fold), the 16-byte cutoff to memory, the hidden-pointer protocol for >16-byte returns. All four commits get full per-commit subsections.
- **§18.3 walks `va_copy` and the closure of the variadic-stack-fixed-parameters gap.** The prose names *why* `va_copy` can be a one-line macro (the array-decay equivalence). The variadic gap is named as closed.
- **§18.4 walks pp-numbers as a lexical category and the lexer-vs-parser split.** The prose explicitly names the change from "tokenizer is a parser of numbers" to "tokenizer is a recognizer of pp-numbers, parsing happens later in `convert_pp_number`."
- **§18.5 walks each of the five bitfield commits with no skipping.** The op-assign commit (`54c2b3b`) gets its own subsection, the zero-width commit (`17ea802`) gets its own subsection, the address-of commit (`c302a96`) gets its own subsection. The op-assign subsection is the most detailed in the chapter — the parser-side rewrite using two separate `ND_MEMBER` nodes for lvalue and rvalue is the chapter's most subtle code-level move, and the prose walks it.
- **§18.6 walks all six polish commits.** Buffered output, ignored flags, `-Wall`-clean self-build, 16-byte array alignment, implicit `return 0` in `main`, anonymous struct/union. Each gets one subsection.
- **One-table recap** at the chapter close, twenty-three rows. Resisted multi-table-by-section.

### Interpretive calls

1. **The pp-number change is named as a *kind* of canonicalization-at-parse-time but the count wasn't incremented for it.** The semantic shape of the move (canonicalize-late, in `convert_pp_tokens`) is in the canonicalization-at-parse-time family, but it lands in `convert_pp_tokens`, not `parse.c`. Increment was reserved for the bitfield op-assign rewrite, which lands in `to_assign` in `parse.c` — that *is* canonicalization-at-parse-time. New count: nine. Prose flags this carefully.
2. **The bitfield op-assign rewrite is named as canonicalization-at-parse-time** — it converts `A.x op= C` into `tmp = &A, (*tmp).x = (*tmp).x op C` *before* codegen sees it. Prose explicitly increments the count and notes that the rewrite is more elaborate than the ordinary `tmp = &A, *tmp = *tmp op C` form because it has to route through the bitfield-aware `ND_MEMBER` node twice.
3. **The pre-factor count goes up by one for §18.5.** The `Member` data-structure extension in `cc852fe` lays the new fields (`is_bitfield`, `bit_offset`, `bit_width`); the next four bitfield commits use those fields. New count: nine. The handoff predicted this; prose names it.
4. **The psABI conformance count goes up by seven** — two for §18.1 (stack-args caller and callee), four for §18.2 (struct-by-value × 4), one for §18.3 (variadic overflow), one for §18.5 (bitfield layout via the SysV rule), one for §18.6's 16-byte array alignment. Net: +8 from the handoff's expected +6, in fact, because the handoff didn't fully account for §18.6's array-alignment commit. Walking through the prose: 9 (Ch 17 close) + 4 (§18.2) + 1 (§18.3) + 1 (§18.5) + 1 (§18.6) = 16; §18.1's two commits fold into the same SysV ABI conformance corrections category. Prose lands on count of sixteen at chapter close.
5. **The `fp_offset = fp * 8 + 48` non-conforming stride is *not* closed by §18.3.** The handoff's forecast was conditional ("might be closed by §18.3 — verify"). Verification: the `b6d3cd0` diff still emits `movl $%d, %d(%%rbp)` with the eight-byte stride. Errata candidate remains open. Prose names the non-closure explicitly.
6. **The `__va_arg_mem` divide-by-zero IS closed by §18.3.** Ch 17's errata catalog flagged this; `b6d3cd0` replaces the placeholder with the real implementation. Prose names the closure explicitly.
7. **The `2bdc6b8` buffered-output commit is named as not-performance work.** Ch 18's handoff specifically warned against the "performance work" misreading. Prose calls it "to prevent partial `.s` files on compile errors" and quotes Rui's commit message.
8. **The `-Wall`-clean self-build commit gets a paragraph on what the warnings actually caught.** The handoff wanted "a sentence per kind of warning fixed." The actual diff only adds `noreturn` annotations to three `error*` declarations; the prose walks that and notes that chibicc's source was already mostly `-Wall`-clean before this commit, which makes the commit a configuration tightening rather than a bug-fixing.
9. **The anonymous-struct/union commit is named as the chapter's only struct-system extension.** Prose walks `get_struct_member`'s recursion, `struct_ref`'s outer loop, and the test cases.
10. **The `2c91da5` commit's `<stdnoreturn.h>` add to `chibicc.h` is mentioned as a Ch 17-supplied header now being used by chibicc itself.** The header existed in `include/` since Ch 17 §17.5.4 (the wide-character / standard headers commit); this is where it's first used.

### Voice / structure inherited from Ch 1–17

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For sub-sections within §18.5 and §18.6, each commit's subsection has its own informal opener (the section already showed the checkout list at the top, so the per-subsection openers don't repeat).
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- One-table recap at the chapter close.
- No concept interludes.

### Three careful avoidances

- **Did not invent a "history of the SysV AMD64 ABI" interlude.** The eightbyte classification has a real history (the original SysV ABI for SPARC, the per-arch supplements, the AMD64 supplement adopted in 2003), but that history is a detour. The chapter cites the rule and walks the implementation; it doesn't try to reconstruct the design history.
- **Did not over-explain the bitfield layout's storage-unit-crossing rule.** The check `bits / (sz * 8) != (bits + mem->bit_width - 1) / (sz * 8)` is not the most readable line in chibicc, but the prose explains *what* it checks (whether a bitfield placed at the current `bits` would cross a storage-unit boundary) and leaves the cleverness to the reader. Walking the bitwise arithmetic step-by-step would have inflated the section without adding clarity.
- **Did not invent a "C bitfield history" detour.** Bitfields have a real history in the C standardization process (they predate ANSI C; the standard left layout largely implementation-defined; subsequent specs slowly tightened the rules). Walking that history would have been a detour from chibicc's specific layout choices. The chapter sticks to what chibicc does and how the C standard's rules constrain it.

### Date-vs-position note

The twenty-three commits scatter across calendar time: late August through early September 2020 for the ABI work (commits 198–205, 211, 213–214, 220), April 2020 for the function-deref commit (206), September 2020 for the pp-numbers commit (207), August 2020 for `-D`/`-U` and the bitfield base (208–210, 212), May 2020 for the polish commits (215–219, with `2c91da5` and `5257ee0` and `9c36dd7` clustered around May 10–11). The chapter follows `main` order without remark.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The ABI-conformance count stands at sixteen** after Chapter 18, up from nine.
- **Two errata closed:** "more than 6 integer args silently miscompiles" and "more than 8 FP args silently miscompiles." Plus one Ch 17 errata closed: `__va_arg_mem` divide-by-zero.
- **One forecasted closure didn't happen:** the `fp_offset = fp * 8 + 48` non-conforming stride is still on the errata list.
- **The `Member` struct** now has its bitfield fields permanently. `is_bitfield`, `bit_offset`, `bit_width`. Plus the existing `idx`, `align`, `offset`. Bitfield support is feature-complete.
- **The `ret_buffer` field on `Node`** is a single-purpose addition — only set for `ND_FUNCALL` whose return type is struct/union, only read by `gen_addr` (for `ND_FUNCALL` lvalue access) and `push_args` and the post-call `copy_ret_buffer` step. No other readers in chibicc.
- **The `pass_by_stack` field on `Node`** is similarly single-purpose — only set by `push_args`'s counting walk, only read by `push_args2`.
- **The `has_flonum` family** (`has_flonum`, `has_flonum1`, `has_flonum2`) is the chapter's central new helper. Used in five places: caller `push_args`, caller pop loop, callee `assign_lvar_offsets`, callee prologue parameter-store, and `copy_ret_buffer`/`copy_struct_reg` for both struct chunks. The asymmetry between the third argument being `0` (caller) vs `8` (callee, in the callee's offset-assign code only) is a quiet inconsistency that doesn't bug-affect correctness.
- **The `copy_ret_buffer`/`copy_struct_reg`/`copy_struct_mem`/`push_struct` quartet** is the chapter's struct-by-value codegen. Each is a small specialized helper. None has been generalized into a single struct-copy function, despite the obvious overlap; Rui keeps them separate because each has slightly different control flow (caller-side post-call vs callee-side return; small-struct vs large-struct; in-register vs in-memory).
- **The `TK_PP_NUM` token kind** is the chapter's tokenizer extension. Lives between tokenize and the post-preprocess `convert_pp_tokens`. Not visible to the parser — every `TK_PP_NUM` becomes `TK_NUM` before parsing starts.
- **`define_macro` and `undef_macro` are public API.** They were `static` in `preprocess.c`; the `-D`/`-U` commits exposed them through `chibicc.h`. The driver calls them from argument-parsing.
- **The variadic codegen** initializes `overflow_arg_area` to `%rbp + 16` and the `__va_arg_mem` helper walks it. The whole variadic-fall-through path is now functional.
- **The buffered output via `open_memstream`** is the driver's late-commit cc1 reshape. The buffer is `malloc`-allocated and never freed (the process exits next), which is fine.
- **The ignored-flags list** in `main.c` is a static list of `-O*`/`-W*`/`-g*`/`-std=*` prefix matches plus a list of exact-match `-f*`/`-m*`/`-w` flags. None changes chibicc's behavior. The list isn't extensible from the command line.
- **The `-Wall` Makefile change** does include `-Wno-switch` to suppress non-exhaustive `switch` warnings (chibicc's source has lots of those). This is a deliberate pragmatic choice — chibicc's switches are intentionally non-exhaustive in many places.
- **The 16-byte array alignment** is an SSE-friendliness rule in the SysV AMD64 ABI. Applied in two places: `assign_lvar_offsets` and `emit_data`. Doesn't apply to structs.
- **The implicit `return 0` for `main`** is a 7-line addition immediately before the function's epilogue. Only fires for `main` (`strcmp(fn->name, "main") == 0`).
- **Anonymous struct/union members** flatten into the outer struct's namespace via a recursive `get_struct_member` and a `struct_ref` outer loop. The result is a chain of `ND_MEMBER` nodes — one per anonymous layer plus the final named member.
- **The two-`ND_MEMBER`-nodes-for-bitfield-op-assign trick** is unique to this chapter. The op-assign commit's parser rewrite creates two separate `ND_MEMBER` nodes for the lvalue and rvalue sides of the assignment, each with its own `member` field set, so codegen can read each independently. This is the chapter's most subtle parser-level move.
- **The Initializer tree for bitfields** stores the bitfield's value as `init->children[mem->idx]->expr` (a Node). Compile-time evaluation merges into the storage unit. No relocations are emitted for global bitfield initializers (because they reduce to constant integer arithmetic).
- **Pre-factor-before-feature count is now nine.** §18.5's `Member` extension is the latest entry.
- **Canonicalization-at-parse-time count is now nine.** §18.5's bitfield op-assign rewrite is the latest entry.
- **psABI conformance count is now sixteen.** Up by seven from Ch 17's nine.
- **The cc1-vs-driver split** is unchanged.
- **The `Initializer` tree, `Relocation` mechanism, `is_static` default, `is_definition` flag, `is_unsigned` flag (with one new reader in the bitfield read codegen), register-save-area layout (with `overflow_arg_area` newly initialized), argreg integer/FP split, fourth namespace (labels), `is_typename` predicate, VarAttr channel, cast table (still 10×10), anonymous-global pattern (no new uses), `Member->idx` field (still has its existing role), `is_flexible` flag, `copy_struct_type` (no new uses), `MIN`/`MAX` macros (with new uses), `is_numeric` predicate, `Obj->tok` field (still no readers), `Type->name_pos` field (no new uses), `>>` codegen quirk, the `add_type` rule for `ND_STMT_EXPR` (errata candidate), the hex-escape silent truncation (errata candidate), the redeclaration-in-same-scope check missing for variables/tags/typedef-names/labels/struct-members (now five errata, since bitfield zero-width test exposed the missing struct-member-name check), `f()` and `f(void)` accepted as identical (errata), empty brace initializer (chibicc-specific extension), `.bss` as third assembly section, `.align` (with new 16-byte rule for arrays), the variadic-FP-call wrongness (errata, *not* closed), `long double` ≡ `double` (errata), default-argument-promotion gap for chars/shorts (errata), float literals inlined as integer-immediate-bit-cast, the cast/compound-literal disambiguator, driver brittleness in `find_libpath`/`find_gcc_libpath` (errata), the link command's hardcoded distro list (errata), `Node->funcname` being gone, `call *%rax` being uniform across all calls (now `mov %rax, %r10; call *%r10`), the `StringArray` type, `atexit(cleanup)` for tempfile disposal, the `run_subprocess` helper** — all unchanged in shape, with the noted small-additions/removals.
- **Errata candidates added in Ch 17:** `#error` doesn't print message text, `L''` ≡ `''`, `__va_arg_mem` divides by zero (closed by `b6d3cd0`), `opt_S | opt_E` typo, default include paths Linux/glibc-specific. One closed; four remain.
- **Errata candidates added in Ch 18:** None new of the high-priority kind; the bitfield op-assign and the struct-by-value classification are intricate but not buggy. The bitfield zero-width test exposes the existing missing-struct-member-name-redeclaration-check (which is the same flavor as the four already-flagged redeclaration-check errata).
- **Stage-2 build** is end-to-end chibicc, plus now `-Wall`-clean.
- **Chibicc compiles itself** as of commit 197 (Ch 17 §17.6); Chapter 18's commits don't change that, plus the `-Wall` clean build means chibicc's source is mostly host-cc-warning-clean.

## Exit state

- `chapters/18-the-full-abi.md` drafted, ~14,357 words.
- Session 019 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 020 (Chapter 19 — Unicode and designated initializers, commits 221–244, ~24 commits).
- CLAUDE.md status note will be updated to "Ch 18 drafted".
