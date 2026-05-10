# Handoff: Ch 13 done → proceed to Ch 14

**For:** the next claude session.
**From:** session 014.
**Status:** Ch 13 drafted (~8,600 words, eleven commits, the linkage chapter — `extern`, alignment keywords, static locals/globals, compound literals, do-while, bare return, two psABI fixes). Continue autonomously to Ch 14 (Variadics, signedness, qualifiers). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/014-chapter-13-draft/README.md`](README.md)** — what session 014 did, including the nine-section structure, the no-interlude decision, the VarAttr-channel-now-at-four-fields tracking note, the anonymous-global pattern as the chapter's most-reused machinery (three sections hinge on it), the `is_static` default in `new_gvar` as the structural choice that makes anonymous globals work for free, the local-vs-global blur named in §13.3, the psABI conformance thread opening with §13.8/§13.9, and the chapter's ~8,600 word length (under handoff target — same call as session 013).
2. **[`docs/sessions/013-chapter-12-draft/HANDOFF.md`](../013-chapter-12-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply. The Initializer tree from Ch 12 is unchanged in Ch 13 and likely unchanged in Ch 14.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`13-linkage.md`](../../../chapters/13-linkage.md)** — the thirteen chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 14 = commits 127–138.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; the chapter mapping doesn't flag a specific interlude for Ch 14, but the variadic-call ABI (the System V x86-64 register-save-area dance) might be a candidate if the prose surfaces a need.

## Chapter 14 scope

**Title (working):** *Variadics, signedness, qualifiers*.
**Commits:** 127–138 in chronological order on `main`. **Twelve commits — slightly more than Ch 13, well below Ch 11/12.**
**Concept interlude:** Probably not. If §14.1's `va_start` walk surfaces a need to explain the System V psABI's register-save-area design, run with it as a brief interlude. Otherwise default-no.

| # | Hash | Subject |
|---|---|---|
| 127 | `58fc861` | Allow to call a variadic function |
| 128 | `754a24f` | Add `va_start` to support variadic functions |
| 129 | `197689a` | Check the number of function arguments |
| 130 | `3f59ce7` | Add `signed` keyword |
| 131 | `34ab83b` | Add unsigned integral types |
| 132 | `aaf1045` | Add `U`, `L` and `LL` suffixes |
| 133 | `8b8f3de` | Use long or ulong instead of int for some expressions |
| 134 | `6880a39` | When comparing two pointers, treat them as unsigned |
| 135 | `7ba6fe8` | Handle unsigned types in the constant expression |
| 136 | `b773554` | Ignore `const`, `volatile`, `auto`, `register`, `restrict` or `_Noreturn` |
| 137 | `93d1277` | Ignore `static` and `const` in array-dimensions |
| 138 | `1fad259` | Allow to omit parameter name in function declaration |

Twelve commits. **Bundling is moderate.** Rough proposal:

- **§14.1 — Calling variadic functions** (commit 127). The mechanism: variadic call sites need the `%al` register zeroed (since the start of chibicc — the placeholder noted in Ch 5 §5.1 was forecasting this), but the chibicc side now actually emits it correctly. Likely small.
- **§14.2 — Defining variadic functions: `va_start`** (commit 128). Substantive. The System V x86-64 register-save-area: the function prologue saves all six argument registers to a fixed offset in the stack frame, and `va_start` initializes the `va_list` struct to point at the right slot. This is the chapter's anchor; ~1,500–2,000 words.
- **§14.3 — Argument-count checking** (commit 129). One commit. Straightforward — `funcall` checks `nargs == fn->params count` (or `>= fn->params count` for variadics).
- **§14.4 — `signed` keyword** (commit 130). One commit. Probably bundles with §14.5 if the diffs interact heavily.
- **§14.5 — Unsigned integral types** (commit 131). The big signedness commit. New TY_USHORT, TY_UINT, TY_ULONG (or maybe a flag on existing types — read the diff). Affects the codegen's integer-arithmetic instructions (signed vs unsigned division, comparison, shift).
- **§14.6 — Integer suffixes `U`/`L`/`LL`** (commit 132). The lexer picks up suffix parsing; the resulting numeric token carries width and signedness. Substantive but small.
- **§14.7 — `long`/`ulong` for some expressions** (commit 133). Promotion rules — sizeof returns `ulong` not `int`, etc. Read carefully.
- **§14.8 — Pointer comparisons as unsigned** (commit 134). One-line codegen change probably (`jb` instead of `jl` for `<`, etc.).
- **§14.9 — Unsigned in const-expr** (commit 135). The §11.15 `eval` evaluator picks up unsigned arithmetic. Possibly an arm-by-arm rewrite.
- **§14.10 — Qualifier soup** (commits 136, 137). Bundle. `const`/`volatile`/`auto`/`register`/`restrict`/`_Noreturn` parsed-and-discarded; `static`/`const` in array dimensions parsed-and-discarded. The "we accept these but enforce nothing" theme.
- **§14.11 — Empty parameter names** (commit 138). One-line patch. `int f(int)` (no name) becomes legal in declarations.

That's eleven sections from twelve commits. **Target chapter length: ~10,000–12,000 words.** Possibly shorter. The variadic anchor (§14.2) carries weight; signedness (§14.5–14.9) is a long arc with many small pieces; the qualifier soup is small.

## Steps

1. `cd research/sources/chibicc && for h in 58fc861 754a24f 197689a 3f59ce7 34ab83b aaf1045 8b8f3de 6880a39 7ba6fe8 b773554 93d1277 1fad259; do echo "===== $h ====="; git show --stat $h | head -8; done` to scan all twelve diffs.
2. Read each commit. Pay particular attention to:
   - **`58fc861`** (commit 127): variadic call. The `mov $0, %al` placeholder from Ch 5 §5.1 — does this commit make the placeholder real, or remove it as no-longer-needed because the right thing is now emitted?
   - **`754a24f`** (commit 128): `va_start`. The register-save-area mechanism. Read the prologue change carefully — it likely emits a fixed save of all six argreg64 registers to a fixed stack offset (something like `-176(%rbp)` for the gp-save area), and `va_start` initializes the `va_list` struct's `gp_offset`/`reg_save_area`/`overflow_arg_area` fields. This is the Sys V psABI's variadic-call-ABI in code.
   - **`197689a`** (commit 129): arg-count check. Probably an `error_tok` if `nargs != fn->params count`. For variadics, `>=` instead of `==`. Read the variadic predicate.
   - **`3f59ce7`** (commit 130): `signed` keyword. Probably parses-and-discards — `signed int` is the same as `int`.
   - **`34ab83b`** (commit 131): unsigned integral types. The big one. New types? Or a `is_unsigned` flag on `Type`? Read carefully. Likely affects: `add_type`'s implicit-conversion rules, the codegen's `cqo`/`cqto` (signed division setup), the comparison codegen (`jl`/`jg` vs `jb`/`ja`), the cast-to-narrower-type behavior.
   - **`aaf1045`** (commit 132): integer suffixes. Tokenizer change. The numeric value's type is determined by suffix (`123U` → `unsigned int`, `123L` → `long`, `123LL` → `long long`). Read the tokenizer.
   - **`8b8f3de`** (commit 133): `long`/`ulong` for some expressions. Probably `sizeof` returns `ulong`, pointer subtraction returns `long`, etc. Read for promotion rules.
   - **`6880a39`** (commit 134): pointer comparison as unsigned. The codegen for `p < q` where `p`, `q` are pointers should use `jb` (unsigned below) not `jl` (signed less than).
   - **`7ba6fe8`** (commit 135): unsigned in const-expr. The `eval` function picks up arms for unsigned arithmetic.
   - **`b773554`** (commit 136): qualifier ignore. `const`/`volatile`/`auto`/`register`/`restrict`/`_Noreturn` keywords added to `is_typename` and `declspec`'s loop, with no effect.
   - **`93d1277`** (commit 137): `static`/`const` in array dimensions. C99 allows `void f(int x[static 10])` and `void f(int x[const 10])` — chibicc parses-and-discards.
   - **`1fad259`** (commit 138): empty parameter name. `int f(int)` becomes legal. Probably a `declarator` change — when the name token is missing, synthesize an empty name.
3. Read the destination state at `1fad259` for `chibicc.h`, `parse.c`, `codegen.c`, `tokenize.c`, all relevant test files.
4. Draft `chapters/14-variadics-signedness-qualifiers.md`. Likely 10,000–12,000 words.
5. Write `docs/sessions/015-chapter-14-draft/README.md`.
6. Write `HANDOFF.md` for session 016 (Chapter 15 — Floating point, commits 139–149; eleven commits, mostly self-contained).

## Voice / structure rules

Same as Ch 1–13:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — twelve rows; consider whether splitting into two tables makes sense (signedness vs everything else?).
- Diff format: lean toward inline diff fragments and quoted file snippets. The `va_start` section (§14.2) and the unsigned-types section (§14.5) may want larger code blocks.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **§14.2's `va_start` walk is the chapter's intuition gap.** The System V psABI's register-save-area design is non-obvious — variadic functions can't know ahead of time how many arguments to expect, so the prologue *unconditionally* spills all six argument registers to known stack slots, and `va_start` sets up a pointer that walks them. Walk the design carefully. May want to inline a small assembly diff showing the prologue change.
- **§14.5 (unsigned types) interacts with §11.15's `eval` and the codegen's arithmetic.** The change is broad. Walk the cast machinery's response to signed-vs-unsigned conversions (truncation, extension, wrap). The §10.x `_Bool` cast precedent applies.
- **§14.7 (long/ulong promotion) interacts with `sizeof`.** Until this commit, `sizeof(int)` returned an `int`. Now it returns `ulong`. The change ripples: any expression that uses `sizeof` in arithmetic now has a `ulong` somewhere in its type tree.
- **§14.8 (pointer comparison as unsigned) is small but ABI-correct.** Pointers in C are abstract addresses; comparison should treat them as unsigned because addresses span the full 64-bit space (and signed comparison would split addresses around 0x8000_0000_0000_0000). One-line codegen change with a clear correctness story.
- **§14.10 (qualifier soup) is the "we accept-and-discard" theme.** Worth a paragraph on the trade-off: chibicc parses these so source code compiles, but the semantics they imply (immutability for `const`, prevention of optimization for `volatile`, allocation hints for `register`) aren't enforced. Most of them don't matter for chibicc's output anyway because chibicc doesn't optimize.
- **§14.11 (empty parameter name) is small but it's the closer.** `int f(int)` is the K&R-style declaration shape; chibicc accepting it is one more piece of the "real-C compatibility" arc.
- **The `mov $0, %al` placeholder from Chapter 5** finally has a real explanation in §14.1 or §14.2. Close the loop: the chapter prose should name the Ch 5 forward-reference and note that it's now real.
- **The VarAttr channel** (now at four fields after Ch 13) likely grows by 5+ in Ch 14 with `is_signed`/`is_unsigned` plus the qualifier set. Track the channel's growth carefully — Ch 13 §13.1 named it as "the channel was the prediction; the third occupant is the close" for `is_extern`; Ch 14's qualifiers should close more predictions.

## Standing notes worth tracking across sessions

- **The `Initializer` tree** is the load-bearing data structure of Ch 12. Unchanged in Ch 13. Likely unchanged in Ch 14.
- **The local-vs-global split** named in §12.6 survived the static-locals blur in Ch 13 §13.3. Ch 14 doesn't add new storage classes; the split should stay stable.
- **The `eval`/`eval2`/`eval_rval` trio** (Ch 12 §12.7) is the constant-expression evaluator. Ch 14 §14.9 will likely add unsigned arms to `eval`.
- **The `Relocation` mechanism** (Ch 12 §12.7) is unchanged. Ch 14 doesn't touch it.
- **The anonymous-global pattern** (`new_anon_gvar`) was Ch 13's most-reused machinery. Ch 14 may use it for floating-point literals (Ch 15) but probably not for integer literals (which are always inline).
- **The `is_static` default in `new_gvar`** (Ch 13 §13.6) is load-bearing for anonymous globals. Watch for any Ch 14 commit that adds a new `new_gvar` caller.
- **The `is_definition` flag on `Obj`** (Ch 13 §13.1) is used today only by `extern`. Ch 14 doesn't extend it.
- **The `Member->idx` field** (Ch 12 §12.5) — no new uses since.
- **The `is_flexible` flag** (Ch 12 §12.4 on `Initializer`, §12.10 on `Type`) is unchanged.
- **`copy_struct_type`** (Ch 12 §12.10) — no new uses since.
- **`MIN`/`MAX` macros** (Ch 12 §12.3) — no new uses since.
- **Canonicalization-at-parse-time count is at eight.** Ch 13 added zero. Ch 14 unlikely to add — variadics, signedness, and qualifier-discarding aren't AST rewrites.
- **Pre-factor-before-feature count is at six.** Ch 13 added zero. Ch 14 unlikely — same reason.
- **The fourth namespace (labels)** is unchanged. Ch 14 doesn't add a fifth.
- **The `is_typename` predicate** is unchanged in shape. Ch 14 will add many keywords to the array (`signed`, `unsigned`, `const`, `volatile`, `auto`, `register`, `restrict`, `_Noreturn`); shape stays the same.
- **The VarAttr channel** has four fields after Ch 13 (`is_typedef`, `is_static`, `is_extern`, `align`). Ch 14 likely adds 5+ (signedness + qualifier flags).
- **The `ND_NULL_EXPR` seed-pattern** (Ch 12 §12.1) — no new uses in Ch 13.
- **The `rep stosb` pattern** (Ch 12 §12.2) — no new uses in Ch 13.
- **The `unreachable()` macro** lives in `chibicc.h`. Used by `store_gp`, `declspec`, `write_buf`. Ch 14 likely adds new callers in the unsigned-arithmetic codegen.
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Tests are in C** as of Ch 8 §8.2.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17.
- **The argreg 8/16/32/64 split** is fully in place. Ch 14 may add a `xmm` register set for floating-point args (but that's Ch 15).
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels. Four errata candidates.
- **`f()` and `f(void)` are accepted as identical** by chibicc. Errata or intentional simplification.
- **Empty brace initializer (`int x[3] = {};`)** is a chibicc-specific extension matching GCC.
- **`.bss` is the third assembly section.** Section list: `.text`, `.data`, `.bss`.
- **`.align`** is emitted for every global (uniform; slightly wasteful for char globals).
- **The `>>` codegen quirk** (Ch 11 §11.13). Errata candidate. Ch 14 §14.5's unsigned types may interact — `>>` for unsigned should be `shr` (logical), for signed `sar` (arithmetic). Read carefully.
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4. Errata candidate.
- **The `mov $0, %al`** (variadic `%al`-zeroing) noted in Ch 5 §5.1. **Becomes real in Ch 14** — close this loop in the chapter prose.
- **Ch 1 errata list** unchanged.
- **The cast/compound-literal disambiguator** (Ch 13 §13.4) — same lookahead pattern as `sizeof typename` vs `sizeof unary`. Worth remembering as a parser idiom.
- **The anonymous-global pattern** (Ch 13) — three callers in Ch 13. May get more in Ch 15 (FP literals).
- **The `is_static` default in `new_gvar`** (Ch 13 §13.6) — load-bearing for anonymous globals.
- **Test file proliferation:** by Ch 13 the suite spans `test/arith.c`, `test/control.c`, `test/function.c`, `test/pointer.c`, `test/decl.c`, `test/string.c`, `test/struct.c`, `test/union.c`, `test/sizeof.c`, `test/initializer.c`, `test/extern.c`, `test/alignof.c`, `test/complit.c`, etc. Ch 14 may add `test/varargs.c`, `test/usualconv.c`, etc.

## Acceptance criteria for Ch 14

- [ ] `chapters/14-variadics-signedness-qualifiers.md` exists, end-to-end readable.
- [ ] All twelve commits covered, grouped into ~10–11 sections.
- [ ] §14.2 walks the System V x86-64 register-save-area design and the `va_list` struct's three pointer/offset fields.
- [ ] §14.1 or §14.2 closes the Ch 5 §5.1 forward-reference about `mov $0, %al`.
- [ ] §14.5 walks the unsigned-types impact on the codegen (signed vs unsigned division, comparison, shift) and on the type system.
- [ ] §14.6 explains how integer suffixes flow from tokenizer to type assignment.
- [ ] §14.7 names the `sizeof`-now-returns-`ulong` ripple effect.
- [ ] §14.8 names *why* pointer comparisons should be unsigned (address space wraps).
- [ ] §14.10 names the parse-and-discard pattern as a deliberate trade-off.
- [ ] Each commit has a `git checkout <full-hash>` opener.
- [ ] Voice matches Ch 1–13.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/015-chapter-14-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 016 (Chapter 15 — Floating point, commits 139–149).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/014-chapter-13-draft/HANDOFF.md  (this handoff)
2. docs/sessions/014-chapter-13-draft/README.md   (what session 014 did)
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
15. chapters/13-linkage.md                         (most recent chapter)
16. research/commits/chapter-mapping.md            (confirms Ch 14 scope)
17. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 14 (Variadics, signedness, qualifiers, commits 127–138) per the
steps in the handoff. Twelve commits — moderate bundling. End-of-session:
write your session dir under docs/sessions/015-chapter-14-draft/ with a
README and a HANDOFF for session 016 (Chapter 15 — Floating point,
commits 139–149).
```
