# Handoff: Ch 14 done → proceed to Ch 15

**For:** the next claude session.
**From:** session 015.
**Status:** Ch 14 drafted (~12,150 words, twelve commits, the variadics-signedness-qualifiers chapter — variadic call sites and `va_start`, `signed`/`unsigned`, integer literal suffixes, the wider-`sizeof`/`ptrdiff` ripple, pointer comparison as unsigned, unsigned in const-expr, the qualifier soup, unnamed parameters). Continue autonomously to Ch 15 (Floating point). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/015-chapter-14-draft/README.md`](README.md)** — what session 015 did, including the eleven-section structure (only §14.10 bundled), the no-interlude decision (the §14.2 register-save-area design *is* the section), the §14.2 chapter-anchor at ~1,800 words, the §14.5 codegen subsections (cast table, load extension, division, comparison, shift), the VarAttr-forecast-was-wrong owning, the `is_unsigned`-flag-on-existing-kind design call named twice, the §14.10 bundling rationale (parse-and-discard shared theme), the psABI conformance thread continuation (Ch 13 §13.8/§13.9 plus Ch 14 §14.1/§14.2/§14.8 = five corrections), the closed Ch 5 §5.1 `mov $0, %rax` loop, and the chapter's ~12,150 word length (right at upper edge of target).
2. **[`docs/sessions/014-chapter-13-draft/HANDOFF.md`](../014-chapter-13-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply. The Initializer tree and anonymous-global pattern are unchanged after Ch 14.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`14-variadics-signedness-qualifiers.md`](../../../chapters/14-variadics-signedness-qualifiers.md)** — the fourteen chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 15 = commits 139–149.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; Ch 15 floating-point doesn't have an obvious interlude candidate, but the SSE / XMM register convention or the IEEE 754 `double` representation might be candidates if the prose surfaces a need.

## Chapter 15 scope

**Title (working):** *Floating point*.
**Commits:** 139–149 in chronological order on `main`. **Eleven commits — moderate, mostly self-contained.**
**Concept interlude:** Probably not. If §15.1's flonum-literal walk surfaces a need to explain the IEEE 754 `double` bit pattern, run with it as a brief interlude. If §15.6's variadic-floats walk surfaces a need to explain the XMM half of the register-save area (which §14.2 set up but didn't consume), that could go inline. Default-no on a separate interlude.

| # | Hash | Subject |
|---|---|---|
| 139 | `1e57f72` | Add floating-point constant |
| 140 | `29de46a` | Add "float" and "double" local variables and casts |
| 141 | `cf9ceec` | Add flonum ==, !=, < and <= |
| 142 | `83f76eb` | Add flonum +, -, * and / |
| 143 | `0ce1093` | Handle flonum for if, while, do, !, ?:, \|\| and && |
| 144 | `8ec1ebf` | Allow to call a function that takes/returns flonums |
| 145 | `c6b3056` | Allow to define a function that takes/returns flonums |
| 146 | `8b14859` | Implement default argument promotion for float |
| 147 | `e452cf7` | Support variadic function with floating-point parameters |
| 148 | `ffea421` | Add flonum constant expression |
| 149 | `9bf9612` | Add "long double" as an alias for "double" |

Eleven commits. **Bundling is light.** Rough proposal:

- **§15.1 — Floating-point literals** (commit 139). Tokenizer: parse `1.0`, `1e10`, `0x1p0`, suffix `f`/`F`/`l`/`L`. New `TY_FLOAT`/`TY_DOUBLE` kinds. Anonymous-global pattern (Ch 13 §13.4) likely used to back literals — read carefully.
- **§15.2 — `float`/`double` locals and casts** (commit 140). The cast table extended with float arms. New `cvtsi2ss`/`cvtss2si` etc. instructions. Possibly substantive.
- **§15.3 — Float comparison** (commit 141). `ucomiss`/`ucomisd`; `setp`/`setnp` for NaN-correct equality. Read the comparison sequence carefully.
- **§15.4 — Float arithmetic** (commit 142). `addss`/`subss`/`mulss`/`divss` (and `sd` variants). The codegen mirrors the integer arithmetic but with XMM registers.
- **§15.5 — Float in control flow** (commit 143). Truth value of floats; `cmp_zero` extension to handle floats; `if`/`while`/`do`/`!`/`?:`/`||`/`&&` all need updates.
- **§15.6 — Float function args/returns** (commit 144). Caller side. XMM args use `%xmm0`–`%xmm7`; returns use `%xmm0`. The argreg machinery (Ch 5 §5.x) gets a parallel xmm reg track.
- **§15.7 — Float function definitions** (commit 145). Callee side. Prologue spills float params from `%xmm0`–`%xmm7` to stack slots.
- **§15.8 — Default argument promotion** (commit 146). C says `float` arguments to variadic or unprototyped functions are promoted to `double`. The `promote` rule lands here.
- **§15.9 — Variadic floats** (commit 147). The XMM half of the §14.2 spill area finally has a real reader. `va_arg(ap, double)` walks the fp_offset.
- **§15.10 — Float in const-expr** (commit 148). `eval_double` parallel to `eval`; folds `1.5 + 2.5` at parse time.
- **§15.11 — `long double` as `double` alias** (commit 149). Three-line patch: parse `long double` as `double`. Same trick as `signed int = int`.

That's eleven sections from eleven commits. **Target chapter length: ~10,000–12,000 words.** Could run shorter — many of the commits are codegen extensions of existing patterns (XMM versions of integer arithmetic), and the prose for those has a floor but a lower ceiling than §14.2's psABI walk.

## Steps

1. `cd research/sources/chibicc && for h in 1e57f72 29de46a cf9ceec 83f76eb 0ce1093 8ec1ebf c6b3056 8b14859 e452cf7 ffea421 9bf9612; do echo "===== $h ====="; git show --stat $h | head -8; done` to scan all eleven diffs.
2. Read each commit. Pay particular attention to:
   - **`1e57f72`** (commit 139): floating-point literal tokenizer. Does it use `strtof`/`strtod` for the value parse? How are float literals represented in the AST? `Node` likely gains an `fval` field (alongside the existing `int64_t val`). The `TY_FLOAT`/`TY_DOUBLE` kinds are introduced here. Check whether the literal is held inline (in `Node->fval`) or as an anonymous global (Ch 13 §13.4 pattern) — chibicc may use either.
   - **`29de46a`** (commit 140): cast table extension. The 8×8 integer cast table (§14.5) is probably extended — is it still one matrix, or a separate float-only matrix? `cvtsi2ss` (int to float), `cvtss2si` (float to int), `cvtss2sd` (float to double), `cvtsd2ss` (double to float). The cast machinery from §10.x gets a fourth dimension.
   - **`cf9ceec`** (commit 141): float comparison. `ucomiss` (unordered compare scalar single) leaves results in EFLAGS. NaN handling: the *parity flag* (`PF`) is set on NaN; correct equality has to check `PF` separately to distinguish NaN==NaN (false) from equal (true). Read the assembly sequence.
   - **`83f76eb`** (commit 142): float arithmetic. `addss` (add scalar single), `addsd` (add scalar double), and so on. The `gen_expr` `ND_ADD` arm grows a float branch. Check how the codegen routes — by `node->ty->kind` likely.
   - **`0ce1093`** (commit 143): float in control flow. `cmp_zero` (the helper that emits "compare to zero" for use in `if`/`while`/`!`/etc.) gets float arms. Float zero is `0.0`, which has the same all-zeros bit pattern as integer zero, but the *comparison instruction* (`ucomiss` vs. `cmp`) is different.
   - **`8ec1ebf`** (commit 144): float function calls (caller). The argreg machinery gets `xmm` versions. `argreg_xmm` array probably (parallel to `argreg64`/etc.). The dispatch decides which register based on argument type.
   - **`c6b3056`** (commit 145): float function definitions (callee). The prologue's "save args to stack" loop gets a float arm. Read the stack-slot assignment logic — XMM args probably get 8-byte slots regardless of `float` (4 bytes) vs `double` (8 bytes).
   - **`8b14859`** (commit 146): default argument promotion. C's rule: `float` passed to a variadic function is promoted to `double`. Probably implemented in `funcall` as a cast-to-double for float args beyond the prototype's fixed parameters. Read carefully — it might also apply to args matching `...` even when the prototype has matching fixed parameters.
   - **`e452cf7`** (commit 147): variadic floats. The `__va_area__` fp half (Ch 14 §14.2) finally has consumers. The user-side `va_arg(ap, double)` macro pattern is what changes — possibly nothing in the compiler proper if the user is writing all the macro plumbing themselves.
   - **`ffea421`** (commit 148): float const-expr. Probably a parallel `eval_double` function that returns `double` instead of `int64_t`. The dispatch from `eval2` checks the type and routes.
   - **`9bf9612`** (commit 149): `long double` as `double`. Probably a one-line addition to `declspec`'s switch — `LONG + DOUBLE` resolves to `ty_double`. Smallest commit in the chapter.
3. Read the destination state at `9bf9612` for `chibicc.h`, `parse.c`, `codegen.c`, `tokenize.c`, `type.c`, all relevant test files.
4. Draft `chapters/15-floating-point.md`. Likely 10,000–12,000 words.
5. Write `docs/sessions/016-chapter-15-draft/README.md`.
6. Write `HANDOFF.md` for session 017 (Chapter 16 — The compiler driver, commits 150–157; eight commits, mostly driver-shape changes).

## Voice / structure rules

Same as Ch 1–14:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present tense for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — eleven rows; one table.
- Diff format: lean toward inline diff fragments and quoted file snippets. Several sections will want larger code blocks (cast table extension, comparison sequence, prologue float-spill).

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **§15.1's flonum-literal handling is the chapter's first mechanism question.** Are floats inlined in the `Node` (via a new `fval` field) or backed by anonymous globals (the Ch 13 §13.4 pattern)? The answer determines how §15.4's arithmetic loads operands. Read the diff before drafting the section's framing.
- **§15.3's NaN-correct equality is non-obvious.** `ucomiss`/`ucomisd` set the parity flag (`PF`) on unordered (NaN) operands; correct equality requires checking `PF` in addition to `ZF`. Walk the assembly sequence carefully — it's a small block but the flag interaction is the section's substance.
- **§15.6 and §15.7 are call vs. definition pair**, parallel to Ch 14's §14.1/§14.2 split. Don't bundle — caller-side and callee-side codegen are different mechanics, and bundling would muddy both.
- **§15.8 (default argument promotion) is the small but important rule.** A `float` argument to `printf` is promoted to `double`. The codegen has to insert the cast at the call site for variadic args. Read carefully — the rule applies to *unprototyped* parameters too (the `f()` shape that §14.3 collapses to variadic), so it kicks in for more cases than just `...`.
- **§15.9 (variadic floats) is the loop-closing for §14.2.** The XMM half of the spill area was set up empty; this commit gives it a reader. The `__va_area__` magic name from Ch 14 §14.2 is reused — no new magic needed.
- **§15.11 (`long double` as `double`) is small but it's the closer.** Three-line patch. The chapter's last line should note that `long double` is *not* really `long double` in chibicc — it's truly aliased to `double`, with no extended-precision arithmetic. Real C distinguishes them; chibicc collapses them. Errata candidate.
- **The `is_unsigned` flag on `Type`** doesn't apply to floats. The flag-on-existing-kind pattern from Ch 14 §14.5 is fine for orthogonal axes (signedness is orthogonal to width); float kinds (`TY_FLOAT`, `TY_DOUBLE`) are *not* a flag on `TY_INT` etc., they're new kinds. Worth noting if the section frames the design choice.
- **The constant evaluator** likely gets a parallel `eval_double` function (returning `double`) alongside `eval`/`eval2`/`eval_rval` (returning `int64_t`). The shape is parallel but the data type is different. §15.10 is where this lives.

## Standing notes worth tracking across sessions

- **The `Initializer` tree** is the load-bearing data structure of Ch 12. Unchanged in Ch 13 and Ch 14. Likely unchanged in Ch 15 unless float literals interact with it (probably not).
- **The local-vs-global split** named in §12.6 survived Ch 13's static-locals blur and Ch 14's variadics. Should stay stable through Ch 15.
- **The `eval`/`eval2`/`eval_rval` trio** (Ch 12 §12.7) is the constant-expression evaluator. Ch 14 §14.9 added unsigned arms. Ch 15 §15.10 likely adds a parallel `eval_double` for float folding.
- **The `Relocation` mechanism** (Ch 12 §12.7) — no new uses since Ch 13. Ch 15 might use it if float literals are backed by anonymous globals.
- **The anonymous-global pattern** (`new_anon_gvar`) was Ch 13's most-reused machinery. Ch 14 didn't use it. Ch 15 §15.1 probably does (for float literals that can't be encoded as immediate operands).
- **The `is_static` default in `new_gvar`** (Ch 13 §13.6) is load-bearing for anonymous globals. Watch for any Ch 15 commit that adds a new `new_gvar` caller.
- **The `is_definition` flag on `Obj`** (Ch 13 §13.1) is used today only by `extern`. No new uses in Ch 14. Ch 15 unlikely to add.
- **The `is_unsigned` flag on `Type`** (Ch 14 §14.5) is the doubling-on-existing-kind pattern. Ch 15 introduces *new* kinds (`TY_FLOAT`/`TY_DOUBLE`); the flag is irrelevant to them.
- **The `__va_area__` magic name** (Ch 14 §14.2) is the chibicc-specific user-side hook for `va_start`. Ch 15 §15.9 reuses it for fp `va_arg`.
- **The register-save-area layout** (`__va_area__`, 136 bytes, 24-byte header + 48 gp + 64 fp) was set up in Ch 14 §14.2. Ch 15 §15.9 finally consumes the fp half.
- **The argreg 8/16/32/64 split** is fully in place for integers. Ch 15 §15.6 adds an XMM-register parallel track. The dispatch by argument type is new.
- **The `Member->idx` field** (Ch 12 §12.5) — no new uses since.
- **The `is_flexible` flag** (Ch 12 §12.4 on `Initializer`, §12.10 on `Type`) is unchanged.
- **`copy_struct_type`** (Ch 12 §12.10) — no new uses since.
- **`MIN`/`MAX` macros** (Ch 12 §12.3) — no new uses since.
- **Canonicalization-at-parse-time count is at eight.** Ch 14 added zero. Ch 15 probably adds zero — float operations are codegen, not AST rewriting.
- **Pre-factor-before-feature count is at six.** Ch 14 added zero. Ch 15 might add — possibly the cast-table refactor when float arms are added, but more likely it just extends in place.
- **The fourth namespace (labels)** is unchanged. Ch 15 doesn't add a fifth.
- **The `is_typename` predicate** is unchanged in shape. Ch 15 will add `float` and `double` to the array; shape stays the same.
- **The VarAttr channel** has four fields (`is_typedef`, `is_static`, `is_extern`, `align`). Ch 14 didn't grow it (the forecast was wrong). Ch 15 unlikely to grow it either.
- **The `ND_NULL_EXPR` seed-pattern** (Ch 12 §12.1) — no new uses since Ch 12.
- **The `rep stosb` pattern** (Ch 12 §12.2) — no new uses since Ch 12.
- **The `unreachable()` macro** lives in `chibicc.h`. Used by `store_gp`, `declspec`, `write_buf`. Ch 14 didn't add new callers. Ch 15 likely adds new callers in the float-codegen `getTypeId`-equivalent.
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Tests are in C** as of Ch 8 §8.2.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17.
- **The `Obj->tok` field** added in Ch 14 §14.11 isn't used by §14.11. Watch for Ch 15 callers.
- **The `Type->name_pos` field** (Ch 14 §14.11) is the always-set position pointer for use in error messages.
- **The `>>` codegen quirk** (Ch 11 §11.13) was partially repaired in Ch 14 §14.5's `sar`/`shr` split. Ch 15 doesn't touch this.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels. Four errata candidates.
- **`f()` and `f(void)` are accepted as identical** by chibicc, with §14.3 adding a third equivalence (`f()` becomes variadic). Errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** is a chibicc-specific extension matching GCC.
- **`.bss` is the third assembly section.** Section list: `.text`, `.data`, `.bss`.
- **`.align`** is emitted for every global.
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4. Errata candidate. Ch 14 §14.2's variadic spill addresses the *register* side but not the stack-overflow side.
- **The `mov $0, %rax`** (Ch 5 §5.1) — closed loop in Ch 14 §14.1. Still pessimistic.
- **psABI conformance thread:** Ch 13 §13.8/§13.9 + Ch 14 §14.1/§14.2/§14.8 = five corrections so far. Ch 15 will continue with the XMM register convention (Sys V psABI's float-passing rules) and possibly the default-argument-promotion rule.
- **Ch 1 errata list** unchanged.
- **The cast/compound-literal disambiguator** (Ch 13 §13.4) — same lookahead pattern as `sizeof typename` vs `sizeof unary`. No new uses.
- **The cast table is now 8×8** (Ch 14 §14.5). Ch 15 will likely add float dimensions — possibly extending to 10×10 (adding F32, F64) or splitting into a parallel float-only matrix.

## Acceptance criteria for Ch 15

- [ ] `chapters/15-floating-point.md` exists, end-to-end readable.
- [ ] All eleven commits covered, grouped into ~10–11 sections.
- [ ] §15.1 names the floating-point literal mechanism (inline `fval` field vs. anonymous global) and walks the new `TY_FLOAT`/`TY_DOUBLE` kinds.
- [ ] §15.2 walks the cast table extension and the new `cvtss2sd`/`cvtsd2ss`/`cvtsi2ss`/`cvtss2si` instructions.
- [ ] §15.3 walks the NaN-correct equality sequence with `ucomiss`/`ucomisd` and the parity-flag check.
- [ ] §15.6/§15.7 cover caller and callee sides separately; the XMM argreg machinery is named.
- [ ] §15.8 names default-argument-promotion as a C standard rule, not a chibicc-specific quirk.
- [ ] §15.9 closes the loop on the §14.2 spill area's fp half.
- [ ] §15.11 names `long double` as a chibicc-specific aliasing (errata candidate).
- [ ] Each commit has a `git checkout <full-hash>` opener.
- [ ] Voice matches Ch 1–14.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/016-chapter-15-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 017 (Chapter 16 — The compiler driver, commits 150–157).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/015-chapter-14-draft/HANDOFF.md  (this handoff)
2. docs/sessions/015-chapter-14-draft/README.md   (what session 015 did)
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
16. chapters/14-variadics-signedness-qualifiers.md (most recent chapter)
17. research/commits/chapter-mapping.md            (confirms Ch 15 scope)
18. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 15 (Floating point, commits 139–149) per the
steps in the handoff. Eleven commits — light bundling. End-of-session:
write your session dir under docs/sessions/016-chapter-15-draft/ with a
README and a HANDOFF for session 017 (Chapter 16 — The compiler driver,
commits 150–157).
```
