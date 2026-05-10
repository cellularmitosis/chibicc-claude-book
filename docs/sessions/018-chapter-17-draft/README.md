# Session 018 — Chapter 17 draft

**Date:** 2026-05-10 (continuation of the autonomous-drafting run that has produced sessions 002–017).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 017 delivered Ch 16 (The compiler driver, eight commits, ~9,688 words). User direction is still autonomous — no chapter-by-chapter review. Ch 17 covers commits 158–197: forty commits, the longest single arc in chibicc's history, building the C preprocessor from a do-nothing seam to a full-feature preprocessor and ending in self-hosting at commit 197.

## What was done

### Drafting decisions

- **Length:** ~17,134 words. Within the 15,000–20,000-word handoff forecast, near the middle. Forty commits can't be compressed below ~15K without dropping substance; the chapter ran to its natural length without padding. The §17.4 function-like-macro arc and the §17.3 hideset interlude are the two single biggest stretches; both wanted full treatment.
- **Section structure:** 6 sections from 40 commits, exactly as the handoff proposed. §17.1 (5 commits, the skeleton) gets per-commit subsections (17.1.1 through 17.1.5) since each of those five commits introduces something distinct enough to warrant its own header. §17.2 (4 commits, conditionals) has per-commit subsections likewise. §17.3 (5 commits, object-like macros) has per-commit subsections plus a *concept interlude* on the no-rescan rule between §17.3.4 and §17.3.5. §17.4 (7 commits, function-like macros) has per-commit subsections. §17.5 (18 commits, polish) groups into four sub-bundles by topic (17.5.1 tests/defined/whitespace, 17.5.2 line-continuation/include-paths, 17.5.3 #error/predefined/__FILE__/__VA_ARGS__/__func__, 17.5.4 string-concat/wide-char/headers/va_arg) with each sub-bundle covering 4–6 commits. §17.6 (1 commit, self-host) gets its own focused section as the chapter's punchline.
- **Concept interlude on the no-rescan rule** lands inside §17.3.4. The handoff defaulted to yes, conditional on whether §17.4's prose surfaces the need; reading the §17.3.4 implementation, the *why* of the per-token paint mechanism — versus a recursion-depth limit, or a global exclusion list — wanted explicit framing. The interlude is short (~400 words) and titled "A concept interlude — why the paint must terminate." It walks the per-token-vs-global question, the set-vs-single-name question, and names *Prosser's algorithm* / *blue paint algorithm* explicitly. The handoff defaulted to "no" on naming Prosser by name, but the §17.3.4 code itself updates its comment block to credit "Dave Prossor" [sic — Rui's typo] in commit 1313fc6, which makes the citation natural.
- **No second concept interlude** on what self-hosting means. §17.6 prose handles it inline. Pulling self-hosting out into its own interlude would have duplicated content with §17.6's ~700 words.
- **§17.1 names the do-nothing-skeleton-first approach as deliberate** — handoff acceptance criterion. §17.1.1's prose calls it "the chapter's pre-factor entry" and explicitly increments the count from seven to eight.
- **§17.2 names the token-level evaluator as separate from the AST-level eval quartet** — handoff acceptance criterion. §17.2.1's prose explicitly calls out the duplication: "chibicc has, post this commit, *two* constant-expression evaluation paths." The note is repeated in the section closer.
- **§17.3 walks the no-rescan rule and the hideset** — handoff acceptance criterion. §17.3.4 walks the rule with two worked examples (`#define M2 M2 + 3` and the mutually-recursive `M4`/`M5` case). The concept interlude reinforces the *why*.
- **§17.4 walks function-like macros, the second no-rescan rule, stringizing, pasting** — handoff acceptance criterion. §17.4.5 walks the hideset *intersection* rule with a worked example (`dbl`/`M10` mutual recursion). §17.4.6 walks stringizing's `quote_string` + tokenize-the-result mechanism. §17.4.7 walks pasting's `paste` helper and the standard's "macros not expanded across `##` operands" rule.
- **§17.5 covers all eighteen polish commits** — handoff acceptance criterion. Sub-grouped exactly as the handoff proposed (four sub-bundles, 4–6 commits each).
- **§17.6 names self-hosting as the chapter's punchline** — handoff acceptance criterion. The §17.6 prose calls it "the chapter's, and arguably the book's, punchline" and walks the Makefile change. References §16.1's stage-2 forecast explicitly.
- **Commit 179 (`use chibicc's preprocessor for all tests`) is named as the moment the host `cc -E` pipeline retires** — handoff acceptance criterion. Lands in §17.5.1.
- **Commit 197 (self-host) is named as the moment chibicc compiles itself end-to-end without `self.py`** — handoff acceptance criterion. §17.6 carries this.
- **One-table recap** at the chapter close, forty rows. Resisted multi-table-by-section, since the table is intended as a single index of features for readers to find a commit.

### Interpretive calls

1. **The do-nothing-preprocessor counts as a pre-factor.** The handoff said "Ch 17 likely to add: the `preprocess()` skeleton (§17.1) is the pre-factor for everything that follows." Decision: yes. Pre-factor count goes from seven to eight.
2. **The eval-quartet duplication is named twice** — once in §17.2.1's prose (where the duplication first appears) and once in the §17.2 closer (where it's flagged as "principled" with the small-duplications-are-tolerated framing from `quotes-rui.md`).
3. **Prosser's algorithm gets named.** The handoff defaulted to no, but commit 1313fc6's diff itself updates the source comment to credit "Dave Prossor" [sic]. Naming it in the chapter is faithful to Rui's own source comments. The naming is restrained (the algorithm is named once in the chapter overview, once in §17.3.4, and once in the closer; the metaphor "blue paint" is mentioned twice).
4. **The hideset section is the longest single subsection in the chapter.** §17.3.4 plus the concept interlude run to ~1,800 words. This is consistent with the handoff's prediction that the no-rescan rule "is the most subtle thing in the chapter and probably wants the concept interlude."
5. **The §17.4 worked examples are explicit.** Both the object-like recursion (`M2 = M2 + 3`) and the function-like recursion (`dbl`/`M10`) get step-by-step walks with hideset states tracked at each step. This is unusual depth for chibicc-book; the chapter mapping flagged the no-rescan rule as the chapter's most subtle, and the prose follows.
6. **The `__func__` / `__FUNCTION__` commits land in `parse.c`, not `preprocess.c`** — and the prose calls this out explicitly in §17.5.3, naming `__func__` as a *predefined identifier* not a macro. This matters because a reader expecting `__func__` to be a preprocessor feature would be looking in the wrong file; the chapter's prose corrects the expectation.
7. **The `va_arg(ap, type)` macro from commit 196 gets the "magic name retired user-side" framing** — exactly as the handoff predicted. §17.5.4 names it as "the long-awaited finishing move on Chapter 14's variadic-function magic." The `__builtin_reg_class` intrinsic gets walked because it's the bridge between the macro body and the codegen.
8. **The cpp-as-separate-binary forecast gets the explicit "wrong-prediction" treatment.** §17.0 (the chapter overview) names it: "There's no separate `cpp` binary, no second fork-exec round-trip beyond Chapter 16's driver-cc1 split." This is the handoff's forecast about the §16.1 prose being wrong; the chapter-17 prose handles it.
9. **Errata candidates added in §17.5 are catalogued in the chapter closer.** Five small items: `#error` doesn't print the message text, `L''` ≡ `''`, `__va_arg_mem` divides by zero, the `opt_S | opt_E` bitwise-`|` typo, and the Linux/glibc-specific default include paths.
10. **The chapter doesn't add a fifth namespace, doesn't grow `VarAttr`, doesn't extend the cast table, doesn't add a canonicalization-at-parse-time arm, doesn't add psABI conformance corrections.** All of those forecasts from the handoff held.

### Voice / structure inherited from Ch 1–16

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For sub-section openers (within §17.1, §17.2, §17.3, §17.4), each commit's subsection has its own opener.
- §17.5's sub-bundle openers list the commits in the bundle with their subjects compact-style (one line).
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers (and per-subsection "Where we are" closers in §17.1).
- One-table recap at the chapter close.
- One concept interlude (in §17.3.4) titled and offset.

### Three careful avoidances

- **Did not invent a "history of macro expansion algorithms" interlude.** The hideset machinery has a real history (Prosser's 1986 document, the C committee's adoption of it, alternative approaches in pre-standard preprocessors), but that history is a detour. The chapter cites Prosser's name and the wiki link Rui himself includes; it doesn't try to compete with the standards-history literature.
- **Did not invent a Linkers-and-Loaders-style chapter on dynamic linking.** §17.5.2's `-I`/default-paths discussion stays in chibicc's lane: what chibicc does, why it's brittle, what the alternative would cost.
- **Did not over-explain the difference between `__func__` and `__FUNCTION__`.** The two-line GNU-extension commit gets ~50 words. The C standard's intent (predefined identifier rather than macro, named-after-the-current-function rather than computed-by-substitution) is summarized once and not belabored.

### Date-vs-position note

The forty commits scatter across calendar time: March 2020 (commits 163, 164, 165, 166, 167, 168, 170, 171, 172, 173, 174, 175, 187, 191, 192, 193, 195, 197) interleaved with August 2020 (158, 161, 162, 169, 178, 188, 189) interleaved with September 2020 (160, 184, 185) interleaved with April 2020 (177, 186, 190, 194, 196). The chapter follows `main` order without remark, the same as Chapters 7–16. The §17.0 overview prose names the date scatter as "even more scattered by calendar date than Chapter 16" — Rui clearly worked on the preprocessor in non-linear bursts, and `main` order isn't `chronological` order.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The hideset on Token** is the chapter's signature data structure. Per-token, paint-on-expansion, gate-on-expansion, intersect at function-like-macro close-paren. Chapter 18 doesn't touch the preprocessor; the hideset machinery is stable post-Ch 17.
- **The Token->origin chain** is the chapter's other major Token-struct addition. Every expanded token knows its source. `__FILE__`/`__LINE__` walk it. Error reporting *could* walk it (chibicc doesn't yet) — future-error-message work is a candidate for using `origin`.
- **The eval-quartet duplication** stays. AST-level constexpr (`eval`/`eval2`/`eval_rval`/`eval_double`) for parser uses; token-level (`eval_const_expr` → `const_expr`) for `#if`/`#elif`. Both share `eval` itself.
- **The host-cc-as-preprocessor pipeline retired at commit 179.** The test Makefile no longer pipes through `cc -E`. The host `cc` is still invoked for the test harness's final link with `test/common`, but the preprocessor side is fully chibicc.
- **`self.py` is deleted.** 127 lines of Python, gone with commit 197. The compiler's input pipeline is now entirely chibicc.
- **`stage2` is bug-equivalent to `stage1`.** The test suite passes against stage 2, which means stage 2 *is* a correct chibicc.
- **Default include paths** — Linux/glibc-specific, hardcoded to three system paths plus `${argv0}/include`. Errata candidate, lower priority.
- **`#error` doesn't print the message text** — three-line implementation in commit 187. Errata candidate.
- **`L''` is `''`** — wide-char punted in commit 194. Errata candidate; full wide-char arrives in Chapter 19.
- **The standard headers** in `include/` are minimal-but-functional: `stdarg.h` with `va_start`/`va_end`/`va_arg` plus the `__va_elem`/`va_list` typedefs; `stdbool.h` with `bool`/`true`/`false`/`__bool_true_false_are_defined`; `stddef.h` with `NULL`/`size_t`/`ptrdiff_t`/`wchar_t`/`max_align_t`; `stdalign.h` with `alignas`/`alignof` aliases; `float.h` with the `FLT_*`/`DBL_*`/`LDBL_*` constants; `stdnoreturn.h` with `noreturn`. Together: ~95 lines.
- **`__builtin_reg_class`** is the new chibicc-specific intrinsic from commit 196. Used only by `<stdarg.h>`'s `va_arg(ap, type)`. Not documented.
- **The `Macro->handler` field** is the dynamic-builtin-macro hook. Used by `__FILE__` and `__LINE__`. Could be reused by future builtins (`__DATE__`, `__TIME__`, `__COUNTER__` arrive in Chapter 19).
- **Pre-factor-before-feature count is now eight.** §17.1.1's do-nothing preprocessor is the latest entry.
- **Canonicalization-at-parse-time count is unchanged at eight.** The preprocessor runs before parse.
- **psABI conformance thread is unchanged at nine.** Preprocessor doesn't touch ABI.
- **The cc1-vs-driver split** is unchanged. The preprocessor lives inside cc1.
- **The `Initializer` tree, `Relocation` mechanism, `is_static` default, `is_definition` flag, `is_unsigned` flag, register-save-area layout, argreg integer/FP split, fourth namespace (labels), `is_typename` predicate, VarAttr channel, cast table, anonymous-global pattern (no new uses), `Member->idx` field (no new uses), `is_flexible` flag, `copy_struct_type` (no new uses), `MIN`/`MAX` macros (no new uses), `is_numeric` predicate, `Obj->tok` field (still no readers), `Type->name_pos` field (no new uses), `>>` codegen quirk, the `add_type` rule for `ND_STMT_EXPR` (errata candidate), the hex-escape silent truncation (errata candidate), the redeclaration-in-same-scope check missing for variables/tags/typedef-names/labels (four errata), `f()` and `f(void)` accepted as identical (errata), empty brace initializer (chibicc-specific extension), `.bss` as third assembly section, `.align` for every global, "more than 6 integer args silently miscompiles" / "more than 8 FP args silently miscompiles" (errata, both), the variadic-FP-call wrongness (errata), the `fp_offset = fp * 8 + 48` non-conforming stride (errata), `long double` ≡ `double` (errata), default-argument-promotion gap for chars/shorts (errata), float literals inlined as integer-immediate-bit-cast, the cast/compound-literal disambiguator, the cast table at 10×10, driver brittleness in `find_libpath`/`find_gcc_libpath` (errata), the link command's hardcoded distro list (errata), `Node->funcname` being gone, `call *%rax` being uniform across all calls, the `StringArray` type, `atexit(cleanup)` for tempfile disposal, the `run_subprocess` helper** — all unchanged from Ch 16.

## Exit state

- `chapters/17-a-preprocessor-from-scratch.md` drafted, ~17,134 words.
- Session 018 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 019 (Chapter 18 — The full ABI, commits 198–220, ~23 commits).
- CLAUDE.md status note will be updated to "Ch 17 drafted".
