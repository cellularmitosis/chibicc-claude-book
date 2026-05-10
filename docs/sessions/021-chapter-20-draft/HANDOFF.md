# Handoff: Ch 20 done → proceed to Ch 21

**For:** the next claude session.
**From:** session 021.
**Status:** Ch 20 drafted (~9,994 words, twenty-two commits, the GCC-extensions arc — multibyte error-column display, `#line` and the GNU line marker, four predefined macros, `__VA_OPT__` and the `,##__VA_ARGS__` swallow, `#pragma`, GCC-style variadic, `typeof`, `__builtin_types_compatible_p`, `_Generic`, `sizeof(<function-type>)`, the GNU `?:`-with-omitted-middle, basic `asm`, `inline`-as-static, dead static-inline elimination, `__attribute__((format))`, `-idirafter`, `offsetof`, tentative definitions, `-fcommon`/`-fno-common`). Continue autonomously to Ch 21 (Thread-local, alloca, VLAs, commits 267–283 — seventeen commits covering thread-local variables, `-include` and `-x` driver options, `alloca`, the VLA arc, `-l`/`-s`, ELF size/type emission, `.a`/`.so` recognition, `long double`, GNU case ranges, GNU array range designators, labels-as-values). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/021-chapter-20-draft/README.md`](README.md)** — what session 021 did, including the six-section structure (per-commit subsections in §20.1, §20.2, §20.3, §20.4, §20.5, §20.6; no concept interlude despite handoff conditional), the new errata candidate (`is_compatible` array arm bug), the canonicalization-at-parse-time count tick from nine to ten, the unchanged counts (pre-factor at nine, psABI conformance at sixteen), the unresolved §20.5.4 mystery about `__attribute__` self-host parsing.
2. **[`docs/sessions/020-chapter-19-draft/HANDOFF.md`](../020-chapter-19-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply with Ch 20 updates folded in (see §21 README for the running list).
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`20-gcc-extensions-worth-supporting.md`](../../../chapters/20-gcc-extensions-worth-supporting.md)** — the twenty chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 21 = commits 267–283 (17 commits, scoped to "Thread-local, alloca, VLAs"). Note the trailing "(and compile-time constant)" phrase in the mapping is ambiguous; commit position numbers are authoritative — Ch 21 ends at `4f165ec` (labels-as-values), Ch 22 starts at `f0c98e0` (labels-as-values as compile-time constant).
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes. Ch 21's commits are mostly feature additions; the alloca and VLA commits may have design notes worth scanning.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; thread-local and VLAs aren't commonly featured topics in compiler tutorials. Probably no concept-interlude candidates.

## Chapter 21 scope

**Title (working):** *Thread-local, alloca, VLAs*.
**Commits:** 267–283 in chronological order on `main`. **Seventeen commits** — significantly fewer than Ch 20 (22 commits) and Ch 19 (24 commits).
**Concept interlude:** Possibly one. The VLA arc (commits 272–275) is the most invasive single stretch — VLAs require run-time-computed sizes which means the codegen has to evaluate size expressions at function-prologue time and store them in stack-allocated cells. This is a genuinely new mechanism, parallel in shape to (but distinct from) `alloca`, which is also added in this chapter. A short interlude on *how chibicc handles dynamically-sized stack allocations* could land in §21.4 or §21.5. Default conditional — judge while reading the commits.

| # | Hash | Subject |
|---|---|---|
| 267 | `b377284` | Add thread-local variable |
| 268 | `8f5ff07` | Add -include option |
| 269 | `ee0a951` | Add -x option |
| 270 | `4064871` | Make -E to imply -xc |
| 271 | `77275c5` | Add alloca() |
| 272 | `e8667af` | Add sizeof() for VLA |
| 273 | `07f9010` | Add pointer arithmetic for VLA |
| 274 | `2fa8f48` | Support sizeof(typename) where typename is a VLA |
| 275 | `b0109a3` | Do not define __STDC_NO_VLA__ |
| 276 | `bc25279` | Add -l option |
| 277 | `c32f0e2` | Add -s option |
| 278 | `8d130ab` | Emit size and type for symbols |
| 279 | `d56dd2f` | Recognize .a and .so files |
| 280 | `e0bf168` | Add long double |
| 281 | `d90c73b` | [GNU] Support case ranges |
| 282 | `3d5550e` | [GNU] Support array range designator |
| 283 | `4f165ec` | [GNU] Support labels-as-values |

Seventeen commits. The natural section grouping:

- **§21.1 — Thread-local variables** (commit 267). One commit. The `_Thread_local` keyword is added; storage class becomes a `VarAttr` flag; codegen emits `.tdata`/`.tbss` and uses `%fs:`-relative addressing for accesses. Likely the longest single-commit section in the chapter. Walk carefully — the AMD64 TLS model has subtleties around initial-exec vs general-dynamic.
- **§21.2 — Driver: `-include`, `-x`, `-E` implies `-xc`** (commits 268–270). Three commits. `-include FILE` prepends a `#include` of `FILE` before the source. `-x LANG` overrides language detection (`c`, `assembler`, `none`). `-E` implies `-xc` so preprocessing-only mode still works on stdin without a `.c` extension.
- **§21.3 — `alloca`** (commit 271). One commit. Run-time stack allocation. Likely a builtin that emits a stack-pointer adjustment plus alignment. Walk how the result interacts with the function epilogue.
- **§21.4 — VLAs** (commits 272–275). Four commits. The arc: VLAs as types, then `sizeof(VLA)` evaluating at run time, then pointer arithmetic on VLAs, then `sizeof(typename)` where typename contains a VLA, then dropping `__STDC_NO_VLA__`. The sizes are stored in compiler-generated locals; VLAs are allocated like `alloca`.
- **§21.5 — Linker-driver: `-l`, `-s`, ELF size/type, `.a`/`.so`** (commits 276–279). Four commits. `-l NAME` adds `libNAME.so` or `libNAME.a` to the link. `-s` strips the binary. The ELF symbol-table emission gets size and type (`.size`/`.type`) directives. `.a` and `.so` files are recognized by `run_linker` as input.
- **§21.6 — `long double`, case ranges, array range designators, labels-as-values** (commits 280–283). Four commits. `long double` is finally implemented as actual extended-precision rather than aliased to `double` (this closes one of the standing errata candidates from earlier chapters!). GNU case ranges (`case 1 ... 5:`) generate range checks in switch lowering. Array range designators (`[3 ... 7] = x`) finally honor the range in elaboration (closing the §19.7 errata candidate). Labels-as-values is the GNU `&&label` and `goto *expr` pair, used for computed gotos in interpreters.

That's six sections from seventeen commits. **Target chapter length: ~10,000–12,000 words.** Likely closer to 10K — most commits are small except for the thread-local, VLA, and long-double additions, which may each warrant ~1,500 words.

## Steps

1. `cd research/sources/chibicc && for h in b377284 8f5ff07 ee0a951 4064871 77275c5 e8667af 07f9010 2fa8f48 b0109a3 bc25279 c32f0e2 8d130ab d56dd2f e0bf168 d90c73b 3d5550e 4f165ec; do echo "===== $h ====="; git show --stat $h | head -10; done` to scan all 17 diffs.
2. Read each commit. Pay particular attention to:
   - **§21.1's thread-local** — read carefully for the AMD64 TLS access pattern. The `%fs:` segment register is used for thread-local addressing on Linux. The `.tdata` and `.tbss` sections hold the per-thread initial values; the loader copies them into per-thread storage at thread creation. Walk the codegen for both the variable definition (the `.tdata`/`.tbss` placement) and the variable read (the `%fs:` indirection).
   - **§21.3's `alloca`** — chibicc's `alloca` is a builtin (`__builtin_alloca`?) that emits inline stack adjustment. Walk how alignment is enforced. The result interacts with the function epilogue in a way that requires the saved `%rbp` to be the cleanup anchor (pop `%rbp` resets `%rsp`). Confirm by reading the prologue/epilogue.
   - **§21.4's VLA arc** — the most invasive single stretch. VLAs as types likely add a `vla_size` field to `Type` (a `Node *` for the size expression). `sizeof(VLA)` becomes runtime evaluation. The size is computed at the declaration site and stored in a hidden local; later `sizeof` reads from that local. Walk the four commits in sequence.
   - **§21.5's `.a`/`.so` recognition** — `run_linker` already accepts `.o` files; `.a` (archives) and `.so` (shared objects) are recognized by file-magic or by extension and passed to the linker with appropriate flags.
   - **§21.6's `long double`** — finally extended-precision (probably uses x87 stack via `fld`, `fst`, etc., or possibly maps to `__float128`). Walk carefully — the calling convention is its own beast (long double passed in x87 stack regs). Closes the long-standing errata candidate from earlier chapters.
   - **§21.6's case ranges** — `case 1 ... 5:` generates a range check in the switch dispatch. Walk how `gen_stmt`'s switch arm handles the range; probably emits an `if (val >= 1 && val <= 5) goto label;` style sequence.
   - **§21.6's array range designator** — `[3 ... 7] = x` finally honors the range. Closes the §19.7 errata candidate. Walk how the elaboration loop iterates over the range.
   - **§21.6's labels-as-values** — `&&label` produces an address; `goto *expr` jumps to a runtime-computed address. Used for computed-goto interpreters. Two commits: the basic feature (`4f165ec`) and the compile-time-constant version (`f0c98e0`, which is in Ch 22 not Ch 21).
3. Read the destination state at `4f165ec` for `parse.c`, `tokenize.c`, `codegen.c`, `chibicc.h`, `main.c`, `type.c`. The VLA changes are likely the most invasive; thread-local touches `parse.c` and `codegen.c`; long double touches `type.c`, `parse.c`, and `codegen.c` substantially.
4. Draft `chapters/21-thread-local-alloca-vlas.md`. Likely 10,000–12,000 words. Six sections.
5. Write `docs/sessions/022-chapter-21-draft/README.md`.
6. Write `HANDOFF.md` for session 023 (Chapter 22 — Performance, dependency files, and the linker driver, commits 284–306).

## Voice / structure rules

Same as Ch 1–20:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, list the checkouts at the section opener.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present tense for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — seventeen rows, single table is fine.
- Diff format: lean toward inline diff fragments and quoted file snippets. The §21.1 thread-local, §21.4 VLAs, and §21.6 long double will want larger code blocks.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **§21.1's thread-local AMD64 TLS** is its own sublanguage. The access pattern is `%fs:offset(%rip)` for initial-exec, which is what gcc emits for TLS variables in the executable. Real glibc has multiple TLS models; chibicc almost certainly does only initial-exec. Don't over-explain the model — name what chibicc does and cite GCC's TLS doc as further reading if needed.
- **§21.3's `alloca` is potentially unsafe** — the C standard doesn't endorse `alloca` (it's a POSIX/GNU extension). Many style guides forbid it. Note the safety issue (no error reporting on stack exhaustion) but don't moralize.
- **§21.4's VLA size evaluation order** is subtle. `int x[f()][g()]` calls `f()` and `g()` in declaration order, and the values are remembered for `sizeof(x)` later. Walk how chibicc threads this.
- **§21.4's `b0109a3` "Do not define __STDC_NO_VLA__"** is a one-line preprocessor change. Walk it briefly; it's not interesting in isolation but completes the VLA arc.
- **§21.5's `-l NAME` resolution** searches a path list. Walk the search order.
- **§21.6's `long double` calling convention** — on x86-64 SysV, long double is 16 bytes (80-bit extended on x87 with padding) and is passed differently from double. Calling conventions for long double are complex. Walk what chibicc does, name what it doesn't.
- **§21.6's case range** generates code; it doesn't generate jump-table entries. A `case 1 ... 1000000` would not blow up the codegen (a real compiler would either generate a range check or a million jump-table entries). Walk what chibicc does.
- **The "labels-as-values" feature** is GNU-only. The `&&` operator (the address-of-label) is a clear name conflict with `&&` (logical AND). The tokenizer must distinguish; chibicc likely treats `&&label` specially in unary-expression parsing. Walk it.
- **`f0c98e0` is in Chapter 22, not Chapter 21.** The mapping line "(and compile-time constant)" is misleading. Don't include it in Ch 21.

## Standing notes worth tracking across sessions

- **The hideset on Token** — unchanged through Ch 20. Ch 21's commits don't touch the macro-expansion machinery.
- **The Token->origin chain** — likely unchanged in Ch 21.
- **The `Token` line-marker fields** — `display_name`, `filename`, `line_delta` added in Ch 20 §20.1. Stable.
- **The eval-quartet duplication** — unchanged through Ch 20. May be touched by VLA size evaluation if chibicc's `eval` chain needs to handle runtime expressions.
- **The cc1-vs-driver split** — unchanged.
- **The `Initializer` tree** — Ch 19 added `Member *mem` for unions. Ch 20 unchanged. Ch 21's array range designator may add a new field or re-use existing range-walking code.
- **The local-vs-global split** — affected by Ch 21's thread-local (a new third storage class). Walk how `Obj` is extended.
- **The `Relocation` mechanism** — likely unchanged in Ch 21.
- **The anonymous-global pattern** — likely unchanged in Ch 21.
- **The `is_static` default in `new_gvar`** — probably gains an `is_tls` companion. Walk while drafting.
- **The `is_definition` flag on `Obj`** — stable since Ch 20.
- **The `is_unsigned` flag on `Type`** — unchanged.
- **The `__va_area__` magic name** — unchanged.
- **The register-save-area layout** — unchanged.
- **The argreg integer/FP split** — likely changes for `long double` (passed in x87 regs, distinct from FP argregs).
- **The `Member->idx` field and bitfield siblings** — unchanged.
- **The `is_flexible` flag** — unchanged. The dead-code duplicate from §19.7's `835cd24` is still in the source; if `array_initializer1` is touched in Ch 21 for range designators, the prose should note whether the duplicate is finally removed.
- **`copy_struct_type`** — unchanged.
- **`MIN`/`MAX` macros** — unchanged.
- **`is_numeric` predicate** — likely changes for `long double` (probably returns true for the new TY_LDOUBLE).
- **Canonicalization-at-parse-time count is at ten.** Ch 21 might add one in §21.6 (case ranges may rewrite to chains of compares; array range designators may rewrite to repeated single-element designations). Verify while drafting.
- **Pre-factor-before-feature count is at nine.** Ch 21 unlikely to add new entries.
- **psABI conformance count is at sixteen.** Ch 21 may add one or two for thread-local TLS access patterns and for `long double` calling convention.
- **The fourth namespace (labels)** is unchanged. Labels-as-values doesn't add a new namespace; it lets labels participate in the address-of operator.
- **The `is_typename` predicate** likely changes in §21.1 (for `_Thread_local`) and §21.6 (for `long double` — the keyword pair). Verify.
- **The `VarAttr` channel** has five fields after Ch 20 (typedef, static, extern, inline, align). Will grow in Ch 21 for thread-local. Walk while drafting.
- **The `ND_NULL_EXPR` seed-pattern** — unchanged.
- **The `rep stosb` pattern** — unchanged. `alloca`-allocated regions are not zero-initialized by chibicc.
- **The `unreachable()` macro** — likely picks up new callers in VLA-related code.
- **Per-token line numbers** — unchanged through Ch 20.
- **GDB-debuggable output** — unchanged.
- **Tests are in C.** New test files likely for thread-local, alloca, VLAs, long double, case ranges, label-as-values.
- **The `Obj->tok` field** added in Ch 14 §14.11 still has no readers.
- **The `Type->name_pos` field** — no new uses since Ch 14.
- **The `Type->origin` field** added in Ch 20 §20.3 for compatibility tracking. Stable.
- **The `Obj` struct grew five fields in Ch 20** (`is_inline`, `is_live`, `is_root`, `refs`, `is_tentative`). Likely grows again in Ch 21 for thread-local.
- **The `>>` codegen quirk** — partially repaired.
- **The `add_type` rule for `ND_STMT_EXPR`** — errata candidate.
- **The hex-escape silent truncation** — errata candidate.
- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels, struct-members — five errata candidates.
- **`f()` and `f(void)` are accepted as identical** — errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension.
- **`.bss` is the third assembly section.** `-fcommon`/`.comm` may add `.comm` as a fourth path. Ch 21's thread-local adds `.tdata` and `.tbss` as fifth and sixth. Walk while drafting.
- **`.align`** — unchanged.
- **The `mov $0, %rax`** for variadic FP-count — errata candidate.
- **The `fp_offset = fp * 8 + 48` non-conforming stride** — errata candidate.
- **`long double` is `double`** — *closed in Ch 21 §21.6*. Verify.
- **The default-argument-promotion gap for chars and shorts** — errata candidate.
- **Float literals are inlined as integer-immediate-bit-cast.**
- **Ch 1 errata list** unchanged.
- **The cast/compound-literal disambiguator** — same lookahead pattern.
- **The cast table is 10×10.** Likely grows in §21.6 if `long double` introduces a new TY_LDOUBLE row/column. Verify while drafting.
- **Driver brittleness** — partially addressed by Ch 21's `-include`, `-x`, `-l`, `-s` additions.
- **The link command's hardcoded distro list** — errata candidate.
- **`Node->funcname` is gone.**
- **`mov %rax, %r10; call *%r10` is uniform across all calls.**
- **The `StringArray` type** — unchanged.
- **`atexit(cleanup)` for tempfile disposal** — unchanged.
- **The `run_subprocess` helper** — unchanged.
- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `opt_S | opt_E` typo; default include paths Linux/glibc-specific. Three remaining.
- **Errata candidates added in Ch 18:** None high-priority.
- **Errata candidates added in Ch 19:**
  - UTF-16 character-literal silent truncation of code points above U+FFFF (in §19.4, commit `454618c`).
  - Dead-code duplicate `is_flexible` block in `array_initializer1` (in §19.7, commit `835cd24`).
  - Range designators `[3 ... 7]` syntactically accepted but not honored in elaboration (in §19.7, commit `835cd24`). **Likely closed in Ch 21 §21.6 commit `3d5550e`.** Verify while drafting.
- **Errata candidates added in Ch 20:**
  - `is_compatible` array arm bug — returns `true` only when both lengths are negative *and equal*; should be `||` (in §20.3.2, commit `1433b40`).
- **`self.py` is gone.**
- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean.
- **Chibicc compiles itself** as of commit 197 (Ch 17 §17.6).
- **The `has_flonum` family** unchanged — likely changes in Ch 21 §21.6 for long double.
- **Bitfield support is feature-complete.**
- **Anonymous struct/union members** flatten via recursive `get_struct_member`.
- **The pre-tokenize pass count is four** (Ch 19 §19.6): BOM, newline, backslash-newline, UCN. Unchanged in Ch 20.
- **The four char-literal prefixes** are functional.
- **The four string-literal prefixes** are functional.
- **`__STDC_UTF_16__` and `__STDC_UTF_32__`** are defined.
- **`__STDC_NO_VLA__`** is currently *defined* (chibicc has no VLAs as of Ch 20). **Will be undefined in Ch 21 §21.4 commit `b0109a3`.**
- **UTF-8 in identifiers** uses C11 Annex D ranges.
- **The GNU `$` extension** in identifiers.
- **`__DATE__`, `__TIME__`, `__COUNTER__`, `__TIMESTAMP__`, `__BASE_FILE__`** are predefined.
- **Designated initializers** work for arrays, structs, unions, anonymous-struct, plus the GNU `=`-omission.
- **`__VA_OPT__` and `,##__VA_ARGS__` are functional.**
- **GCC-style variadic macros (`name...`)** are functional.
- **`#pragma` is silently consumed.**
- **`typeof`, `_Generic`, `__builtin_types_compatible_p`** are functional.
- **`sizeof(<function-type>)` returns 1.**
- **The GNU `?:`-omitted-middle** is functional.
- **`asm`** is verbatim-string-only, no operand bindings.
- **`inline` is treated as `static`**, with dead-static-inline elimination.
- **`__attribute__` is macro-stubbed when `__GNUC__` is undefined.**
- **`-idirafter`, `-fcommon`/`-fno-common`** are functional.
- **`offsetof` is in `<stddef.h>`.**
- **Tentative definitions are functional.** `.comm` (under `-fcommon`) or `.bss` (under `-fno-common`).

## Acceptance criteria for Ch 21

- [ ] `chapters/21-thread-local-alloca-vlas.md` exists, end-to-end readable.
- [ ] All seventeen commits covered, grouped into ~6 sections.
- [ ] §21.1 walks the AMD64 TLS access pattern (`%fs:`-relative addressing) and names `.tdata`/`.tbss` as new assembly sections.
- [ ] §21.3 walks `alloca`'s stack-pointer manipulation and notes the safety concern.
- [ ] §21.4 walks the VLA arc, especially how size expressions are stored and re-read for `sizeof`.
- [ ] §21.6 walks `long double` and notes that this closes the standing "long double is double" errata.
- [ ] §21.6 walks array range designators and notes that this closes the §19.7 range-designator errata.
- [ ] §21.6 walks labels-as-values (the basic feature only — the compile-time-constant variant is in Ch 22).
- [ ] Voice matches Ch 1–20.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] psABI conformance thread count noted (likely grows by one or two for thread-local and long double calling convention).
- [ ] `docs/sessions/022-chapter-21-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 023 (Chapter 22 — Performance, dependency files, and the linker driver, commits 284–306).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/021-chapter-20-draft/HANDOFF.md  (this handoff)
2. docs/sessions/021-chapter-20-draft/README.md   (what session 021 did)
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
22. chapters/20-gcc-extensions-worth-supporting.md (most recent chapter)
23. research/commits/chapter-mapping.md            (confirms Ch 21 scope)
24. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 21 (Thread-local, alloca, VLAs, commits 267–283)
per the steps in the handoff. Seventeen commits, six sections proposed
in the handoff. The VLA arc (§21.4, four commits — sizeof(VLA),
pointer arithmetic, sizeof(typename), drop __STDC_NO_VLA__) is the
chapter's most invasive single stretch and is where a possible concept
interlude on dynamically-sized stack allocations lands. The §21.6
long-double commit closes the long-standing "long double is double"
errata; the §21.6 array-range-designator commit closes the §19.7
range-designator errata. End-of-session: write your session dir under
docs/sessions/022-chapter-21-draft/ with a README and a HANDOFF for
session 023 (Chapter 22 — Performance, dependency files, and the
linker driver, commits 284–306).
```
