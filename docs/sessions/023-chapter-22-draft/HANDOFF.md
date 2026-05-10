# Handoff: Ch 22 done → proceed to Ch 23

**For:** the next claude session.
**From:** session 023.
**Status:** Ch 22 drafted (~9,320 words, twenty-three commits — labels-as-values as compile-time constant, the string hashmap and three uses of it, the seven-commit `-M` family, `-fpic`/`-fPIC` (real codegen change with `@GOTPCREL` and `__tls_get_addr`), file-search caching, include-guard optimization, `#pragma once`, `#include_next`, five linker pass-throughs (`-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker`), and the third-party-app shell-script harness). Continue autonomously to Ch 23 (Atomics and the final polish, commits 307–316 — ten commits covering `atomic_compare_exchange`, `atomic_exchange`, `_Atomic` and the atomic op-assigns, `stdatomic.h`, the cpython smoke test, `__attribute__((packed))`, `__attribute__((aligned))`, member access through `=` and `?:`, plus the very last commit). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/023-chapter-22-draft/README.md`](README.md)** — what session 023 did, including the seven-section structure, the two new errata candidates (one-sided `quote_makefile`, stale `include_next_idx` after cache hit), the psABI conformance count tick from eighteen to nineteen.
2. **[`docs/sessions/022-chapter-21-draft/HANDOFF.md`](../022-chapter-21-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply with Ch 22 updates folded in (see §23 README for the running list).
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`22-performance-deps-and-the-linker-driver.md`](../../../chapters/22-performance-deps-and-the-linker-driver.md)** — the twenty-two chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 23 = commits 307–316 (10 commits, "Atomics and the final polish — the very last commit"). The chapter mapping line lists the topics: `atomic_compare_exchange`, `atomic_exchange`, `_Atomic`, atomic op-assigns, `stdatomic.h`, cpython smoke test, `__attribute__((packed))`, `__attribute__((aligned))`, member access through `=` and `?:`.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes. Ch 23 closes the project; the chapter recap may want a Rui quote about the project's scope or what's been left out.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; atomics aren't commonly featured in compiler tutorials. They might warrant explanation as their own concept-interlude candidate.

## Chapter 23 scope

**Title (working):** *Atomics and the final polish*.
**Commits:** 307–316 in chronological order on `main`. **Ten commits** — the final chapter, closing the project.
**Concept interlude:** Possibly one. Atomics are unusual enough as a language feature that a short interlude on memory-order semantics, the C11 atomic model, and what `_Atomic` actually means at the codegen level might be warranted. Default conditional — judge while reading the commits.

| # | Hash | Subject |
|---|---|---|
| 307 | `ca27455` | Add atomic_compare_exchange |
| 308 | `80ea9d4` | Add atomic_exchange |
| 309 | `d69a11d` | Add _Atomic and atomic ++, -- and op= operators |
| 310 | `0a5d08c` | Complete stdatomic.h |
| 311 | `2ed3fda` | Add test/thirdparty/cpython.sh |
| 312 | `395308c` | redefinition |
| 313 | `44bea4c` | Add __attribute__((packed)) |
| 314 | `b35d148` | Add __attribute__((aligned(N)) for struct declaration |
| 315 | `982041f` | Update README |
| 316 | `90d1f7f` | Make struct member access to work with `=` and `?:` |

Ten commits. The natural section grouping:

- **§23.1 — Atomic builtins: compare-exchange and exchange** (commits 307–308). Two commits. The `__sync_*` (or `__atomic_*`) builtin family that backs C11 `atomic_compare_exchange` and `atomic_exchange`. Walk how chibicc lowers these to `lock cmpxchg` and `lock xchg`. Likely ~1,000 words.
- **§23.2 — `_Atomic` qualifier and atomic op-assigns** (commit 309). One commit. The `_Atomic` type qualifier and atomic compound-assignment lowering (`*=`, `+=`, etc. on atomic-qualified types lower to a CAS-loop). Walk the type system extension (a new `is_atomic` flag on `Type`, or a new wrapper type kind), the CAS-loop codegen pattern, and how the increment/decrement (`++`/`--`) cases differ from full op-assigns. Likely ~1,500 words.
- **§23.3 — `stdatomic.h`** (commit 310). One commit. The library-side header that wraps the builtins behind C11 names. Walk what's in it; it's likely a thin shim. ~600 words.
- **§23.4 — The cpython test** (commit 311). One commit. A test script that compiles cpython with chibicc. Walk what it builds and what it verifies. ~400 words.
- **§23.5 — Two redefinition cleanups** (commit 312). One commit titled simply "redefinition" — likely fixes a redefinition diagnostic or relaxes one. Read the diff carefully to decide section placement; may fold into §23.6 or §23.7 if small. ~400 words.
- **§23.6 — `__attribute__((packed))` and `__attribute__((aligned))`** (commits 313–314). Two commits. Both extend struct layout with attribute-driven overrides. Walk the parser side (attribute parsing extension) and the layout side (`offsetof` and `sizeof` honor the override). ~1,200 words.
- **§23.7 — README update** (commit 315). One commit. The README gets updated to reflect the project's final state. Likely a short walk of what changed. ~300 words.
- **§23.8 — Member access through `=` and `?:`** (commit 316). One commit. The very last commit on `main`. Allows `(a ? b : c).x = 5` and similar patterns. Walk the parser and lvalue-conversion piece. ~600 words.

That's eight sections from ten commits. **Target chapter length: ~6,500–8,000 words.** Likely closer to 7,000 — most commits are small. The two atomics-related sections will dominate.

This is the project's last chapter. The chapter recap should be a chapter-and-project recap: not just what Ch 23 added, but a brief survey of where the compiler stands. Don't write a "Phase 3" plan or post-mortem — that belongs in a separate session if the user wants it.

## Steps

1. `cd research/sources/chibicc && for h in ca27455 80ea9d4 d69a11d 0a5d08c 2ed3fda 395308c 44bea4c b35d148 982041f 90d1f7f; do echo "===== $h ====="; git show --stat $h | head -10; done` to scan all 10 diffs.
2. Read each commit. Pay particular attention to:
   - **§23.1's `ca27455`/`80ea9d4`** — the compare-exchange and exchange operations. Walk the `lock cmpxchg` and `lock xchg` codegen.
   - **§23.2's `d69a11d`** — the `_Atomic` qualifier extension. Walk the type-system change and the CAS-loop codegen pattern. The most complex commit in the chapter.
   - **§23.3's `0a5d08c`** — the `stdatomic.h` header. Walk what it defines.
   - **§23.4's `2ed3fda`** — the cpython test script. Walk what it builds. May be very small.
   - **§23.5's `395308c`** — titled simply "redefinition." Read the diff carefully to decide what it does.
   - **§23.6's `44bea4c`/`b35d148`** — `packed` and `aligned`. Walk the attribute parser and the struct layout calculation.
   - **§23.7's `982041f`** — README update. Walk what changed if substantive.
   - **§23.8's `90d1f7f`** — the very last commit. Walk member access through `=` and `?:`.
3. Read the destination state at `90d1f7f` for `parse.c`, `tokenize.c`, `codegen.c`, `chibicc.h`, `main.c`, `preprocess.c`, plus the new atomic-related includes. The atomics changes may touch `Type`, `Node`, and `gen_expr`/`gen_addr` substantially.
4. Draft `chapters/23-atomics-and-the-final-polish.md`. Likely 6,500–8,000 words. Eight sections.
5. Write `docs/sessions/024-chapter-23-draft/README.md`.
6. **No further chapter handoff.** Ch 23 is the final chapter. The session 024 README should note that the bulk-drafting phase is complete and the next phase (full-pass review/revision) is the user's call. Optionally write a `HANDOFF.md` aimed at a "Phase 3 setup" session that the user may or may not run.

## Voice / structure rules

Same as Ch 1–22:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, list the checkouts at the section opener.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present tense for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — ten rows, single table is fine.
- Diff format: lean toward inline diff fragments and quoted file snippets. The `_Atomic` and CAS-loop section will want larger code blocks.
- **Chapter close should be a project close.** A short paragraph or two surveying what chibicc handles after this final commit. Don't write a phase-3 plan.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.
- Don't invent features chibicc doesn't have.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **§23.1/2's atomics are subtle.** C11 atomics have a memory-order parameter (`memory_order_seq_cst`, `memory_order_acquire`, etc.). Chibicc's implementation likely emits the strongest order (sequentially consistent) regardless of the parameter, since `lock`-prefixed instructions on x86 are seq-cst by default. Walk this honestly.
- **§23.2's CAS loop** is the canonical lock-free update pattern: load, compute new value, `cmpxchg`, retry on failure. Walk the loop structure and how `++`/`--`/`+=` etc. lower to it.
- **§23.6's `packed`/`aligned` change struct layout calculation.** The layout function (`new_struct_or_union_type` or wherever) needs to honor the attribute. Walk how the attribute is parsed and threaded through.
- **§23.8's "member access through `=` and `?:`"** is a parser-side change to `postfix`. The gcc extension allows `(a = b).x` and `(a ? b : c).x` to work as both rvalue and lvalue, with the assignment/conditional synthesizing a struct value. Walk how chibicc represents this.
- **The chapter is the project's last.** The recap should briefly survey the whole compiler — what it handles, what it doesn't (no optimization, no register allocation, single back end, x86-64 Linux only, etc.). Honest closure.

## Standing notes worth tracking across sessions

- **The hideset on Token** — unchanged through Ch 22.
- **The Token->origin chain** — unchanged.
- **The `Token` line-marker fields** — `display_name`, `filename`, `line_delta` added in Ch 20 §20.1. Stable.
- **The eval-quartet duplication** — has a fifth member (`is_const_expr`) since Ch 21. Ch 22 §22.1 extended `eval2`/`eval_rval` to use `char ***` for label addresses. Ch 23's atomics shouldn't touch the quartet.
- **The cc1-vs-driver split** — unchanged.
- **The `Initializer` tree** — Ch 19 added `Member *mem`; Ch 21 §21.6 made array range designators honored. Stable.
- **The local-vs-global split** — Ch 21 added `is_tls`; Ch 22 didn't change it.
- **The `Relocation` mechanism** — `label` field is `char **` since Ch 22 §22.1.
- **The anonymous-global pattern** — unchanged.
- **The `is_static` default in `new_gvar`** — gained `is_tls` companion in Ch 21. Stable.
- **The `is_definition` flag on `Obj`** — stable since Ch 20.
- **The `is_unsigned` flag on `Type`** — unchanged.
- **The `__va_area__` magic name** — unchanged.
- **The register-save-area layout** — unchanged.
- **The argreg integer/FP split** — long double on-stack, SSE for FP, GP for integer. Stable.
- **The `Member->idx` field and bitfield siblings** — Ch 23 §23.6 may add an attribute-influenced sibling for `packed`/`aligned`.
- **The `is_flexible` flag** — unchanged. Dead-code duplicate from §19.7's `835cd24` is still in the source.
- **`copy_struct_type`** — unchanged.
- **`MIN`/`MAX` macros** — unchanged.
- **`is_numeric` predicate** — gained TY_LDOUBLE in Ch 21. Stable.
- **`is_flonum` and `has_flonum` diverged in Ch 21.**
- **Canonicalization-at-parse-time count is at eleven.** Ch 22 didn't change it.
- **Pre-factor-before-feature count is at nine.** Ch 22 didn't change it.
- **psABI conformance count is at nineteen.** Ch 22 §22.5 added `-fpic`/`-fPIC`. Ch 23's atomics may grow it (the `lock`-prefixed instructions are the standard psABI atomic forms).
- **The fourth namespace (labels)** is unchanged.
- **The `is_typename` predicate** — unchanged through Ch 22; uses a hashmap as of Ch 22.
- **The `VarAttr` channel** has six fields after Ch 21. Stable through Ch 22.
- **The `ND_NULL_EXPR` seed-pattern** — unchanged.
- **The `rep stosb` pattern** — unchanged.
- **The `unreachable()` macro** — unchanged.
- **Per-token line numbers** — unchanged.
- **GDB-debuggable output** — unchanged.
- **Tests are in C.** Ch 22 added `test/pragma-once.c`; Ch 23 will likely add atomics tests and possibly attribute tests.
- **The `Obj->tok` field** added in Ch 14 §14.11 still has no readers.
- **The `Type->name_pos` field** — no new uses.
- **The `Type->origin` field** added in Ch 20 §20.3 for compatibility tracking. Stable.
- **The `Obj` struct gained two fields in Ch 21** (`is_tls`, `alloca_bottom`). Stable through Ch 22.
- **`Type` gained `vla_len`/`vla_size`** in Ch 21. Stable through Ch 22.
- **The `Token`/`Node` `fval`** widened to `long double` in Ch 21. Stable.
- **The `>>` codegen quirk** — partially repaired.
- **The `add_type` rule for `ND_STMT_EXPR`** — errata candidate.
- **The hex-escape silent truncation** — errata candidate.
- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels, struct-members — five errata candidates.
- **`f()` and `f(void)` are accepted as identical** — errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension.
- **`.bss` is the third assembly section.** Five sections (`.text`, `.data`, `.bss`, `.tdata`, `.tbss`) plus `.comm`. Stable through Ch 22.
- **`.align`** — unchanged.
- **The `mov $0, %rax`** for variadic FP-count — errata candidate.
- **The `fp_offset = fp * 8 + 48` non-conforming stride** — errata candidate.
- **`long double` is real 80-bit** — closed in Ch 21.
- **The default-argument-promotion gap for chars and shorts** — errata candidate.
- **Float literals are inlined as integer-immediate-bit-cast.**
- **Long double literals are split across two halves through the redzone.**
- **Ch 1 errata list** unchanged.
- **The cast/compound-literal disambiguator** — same lookahead pattern.
- **The cast table is 11×11.** Stable through Ch 22.
- **Driver brittleness** — addressed by Ch 21's `-include`, `-x`, `-l`, `-s` and Ch 22's `-M` family, `-fpic`/`-fPIC`, `-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker`. The driver vocabulary is now near-comprehensive.
- **The link command's hardcoded distro list** — partially addressed in Ch 22 §22.7.1's `-static` path-cleanup. Errata candidate remaining.
- **`Node->funcname` is gone.**
- **`mov %rax, %r10; call *%r10` is uniform across all calls.**
- **The `StringArray` type** — picks up `std_include_paths` in Ch 22 §22.4.7.
- **`atexit(cleanup)` for tempfile disposal** — unchanged.
- **The `run_subprocess` helper** — unchanged.
- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `opt_S | opt_E` typo; default include paths Linux/glibc-specific. Three remaining.
- **Errata candidates added in Ch 18:** None high-priority.
- **Errata candidates added in Ch 19:** UTF-16 char-literal silent truncation; dead-code duplicate `is_flexible` block. Two remaining.
- **Errata candidates added in Ch 20:** `is_compatible` array arm bug. One remaining.
- **Errata candidates added in Ch 21:** `.size` missing for function symbols; suffix-only `.a`/`.so` recognition. Two remaining.
- **Errata candidates added in Ch 22:** one-sided `quote_makefile` (target-only escaping, dependencies unescaped); `include_next_idx` not updated on cache hit. Two remaining.
- **Errata candidates closed in Ch 21:** "long double is double"; range designators not honored.
- **Errata candidates closed in Ch 22:** none.
- **`self.py` is gone.**
- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean.
- **Chibicc compiles itself** as of commit 197 (Ch 17 §17.6).
- **Bitfield support is feature-complete.**
- **Anonymous struct/union members** flatten via recursive `get_struct_member`.
- **The pre-tokenize pass count is four.**
- **The four char-literal prefixes** are functional.
- **The four string-literal prefixes** are functional.
- **`__STDC_UTF_16__` and `__STDC_UTF_32__`** are defined.
- **`__STDC_NO_VLA__`** — no longer defined as of Ch 21.
- **`__STDC_NO_THREADS__`** — no longer defined as of Ch 21.
- **UTF-8 in identifiers** uses C11 Annex D ranges.
- **The GNU `$` extension** in identifiers.
- **`__DATE__`, `__TIME__`, `__COUNTER__`, `__TIMESTAMP__`, `__BASE_FILE__`** are predefined.
- **Designated initializers** work for arrays, structs, unions, anonymous-struct, plus the GNU `=`-omission, plus array range designators.
- **`__VA_OPT__` and `,##__VA_ARGS__` are functional.**
- **GCC-style variadic macros (`name...`)** are functional.
- **`#pragma` is silently consumed** for everything except `#pragma once` (Ch 22 §22.6.2).
- **`typeof`, `_Generic`, `__builtin_types_compatible_p`** are functional.
- **`sizeof(<function-type>)` returns 1.**
- **The GNU `?:`-omitted-middle** is functional.
- **`asm`** is verbatim-string-only.
- **`inline` is treated as `static`**, with dead-static-inline elimination.
- **`__attribute__` is macro-stubbed when `__GNUC__` is undefined.** Ch 23 §23.6 adds `packed` and `aligned` as real recognized attributes.
- **`-idirafter`, `-fcommon`/`-fno-common`** are functional.
- **`offsetof` is in `<stddef.h>`.**
- **Tentative definitions are functional.**
- **`_Thread_local`/`__thread`** are functional.
- **`alloca` is a builtin.**
- **VLAs are functional.**
- **`-include`, `-x`, `-E xc`, `-l`, `-s`, `.a`/`.so`** are in the driver vocabulary.
- **`.type`/`.size`** directives are emitted.
- **`long double` is real 80-bit extended precision.**
- **GNU case ranges** are functional.
- **GNU array range designators** are honored in elaboration.
- **GNU labels-as-values** are functional inside function bodies and as compile-time constants in static initializers (Ch 22 §22.1).
- **Hashmap is the workhorse data structure.** Eight `HashMap` instances across the compiler.
- **The `-M` family is complete.** Seven flags.
- **`-fpic`/`-fPIC`** generate real PIC codegen with `@GOTPCREL` and `__tls_get_addr`.
- **Include-guard optimization, `#pragma once`, `#include_next`** are functional.
- **`-static`, `-shared`, `-L`, `-Wl,`, `-Xlinker`** are in the driver vocabulary.
- **Third-party harness exists** for git, libpng, sqlite, tinycc.

## Acceptance criteria for Ch 23

- [ ] `chapters/23-atomics-and-the-final-polish.md` exists, end-to-end readable.
- [ ] All ten commits covered, grouped into ~8 sections.
- [ ] §23.1 walks `atomic_compare_exchange` and `atomic_exchange` codegen (`lock cmpxchg`, `lock xchg`).
- [ ] §23.2 walks `_Atomic` and the CAS-loop pattern for atomic compound-assignments.
- [ ] §23.3 walks `stdatomic.h`.
- [ ] §23.4 walks the cpython smoke test.
- [ ] §23.5 walks the "redefinition" cleanup commit (or folds it into another section).
- [ ] §23.6 walks `__attribute__((packed))` and `__attribute__((aligned))`.
- [ ] §23.7 walks the README update (or notes it briefly).
- [ ] §23.8 walks "member access through `=` and `?:`".
- [ ] Voice matches Ch 1–22.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references not needed (this is the last chapter).
- [ ] Chapter recap is also a project recap — short, honest, no phase-3 plan.
- [ ] psABI conformance count noted (atomics may grow it).
- [ ] `docs/sessions/024-chapter-23-draft/README.md` written.
- [ ] No further `HANDOFF.md` needed unless the user wants a "Phase 3 setup" session primed.

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/023-chapter-22-draft/HANDOFF.md  (this handoff)
2. docs/sessions/023-chapter-22-draft/README.md   (what session 023 did)
3. chapters/01-a-calculator.md                     (template, voice)
4. chapters/02-from-program-to-programs.md
5. chapters/03-statements-and-local-variables.md
6. chapters/04-pointers.md
7. chapters/05-functions.md
8. chapters/06-arrays.md
9. chapters/07-globals-characters-strings.md
10. chapters/08-scopes-and-source-locations.md
11. chapters/09-structs-and-unions.md
12. chapters/10-filling-out-the-type-system.md
13. chapters/11-all-the-operators.md
14. chapters/12-initializers.md
15. chapters/13-linkage.md
16. chapters/14-variadics-signedness-qualifiers.md
17. chapters/15-floating-point.md
18. chapters/16-the-compiler-driver.md
19. chapters/17-a-preprocessor-from-scratch.md
20. chapters/18-the-full-abi.md
21. chapters/19-unicode-and-designated-initializers.md
22. chapters/20-gcc-extensions-worth-supporting.md
23. chapters/21-thread-local-alloca-vlas.md
24. chapters/22-performance-deps-and-the-linker-driver.md (most recent)
25. research/commits/chapter-mapping.md            (confirms Ch 23 scope)
26. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 23 (Atomics and the final polish, commits 307–316)
per the steps in the handoff. Ten commits, eight sections proposed.
The §23.2 _Atomic walk is the most complex commit in the chapter
(CAS-loop codegen, type system extension). The chapter is the
project's last; the recap should briefly survey the whole compiler.
End-of-session: write your session dir under
docs/sessions/024-chapter-23-draft/ with a README. No further
HANDOFF.md needed unless the user wants a "Phase 3 setup" session
primed.
```
