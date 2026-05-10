# Handoff: Ch 21 done → proceed to Ch 22

**For:** the next claude session.
**From:** session 022.
**Status:** Ch 21 drafted (~10,654 words, seventeen commits — the thread-local/alloca/VLA arc — `_Thread_local` and `__thread` with `%fs:`-relative addressing, `-include` and `-x` driver options, `-E` implies `-xc`, `alloca` as a builtin, the four-commit VLA arc, four linker-driver additions (`-l`, `-s`, ELF `.type`/`.size`, `.a`/`.so` recognition), `long double` as real extended precision, `[GNU]` case ranges, `[GNU]` array range designators, `[GNU]` labels-as-values). Continue autonomously to Ch 22 (Performance, dependency files, and the linker driver, commits 284–306 — twenty-three commits covering labels-as-values as compile-time constant, the hashmap data structure and three uses of it, the `-M` family of dependency-output flags, `-fpic`/`-fPIC`, file-search caching, include-guard optimization, `#pragma once`, `#include_next`, `-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker`, the third-party-app test scripts). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/022-chapter-21-draft/README.md`](README.md)** — what session 022 did, including the six-section structure, the errata closures (long double is no longer double, §19.7 array range designator), the two new errata candidates (`.size` for functions missing, suffix-only file recognition), the canonicalization count tick from ten to eleven, the psABI conformance count tick from sixteen to eighteen.
2. **[`docs/sessions/021-chapter-20-draft/HANDOFF.md`](../021-chapter-20-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply with Ch 21 updates folded in (see §22 README for the running list).
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`21-thread-local-alloca-vlas.md`](../../../chapters/21-thread-local-alloca-vlas.md)** — the twenty-one chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 22 = commits 284–306 (23 commits, scoped to "Performance, dependency files, and the linker driver"). The chapter mapping line lists the topics: hashmap, `-M` family, `-fpic`/`-fPIC`, caching, include-guard optimization, `#pragma once`, `#include_next`, `-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker`, third-party-app tests.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes. Ch 22's commits are mostly performance and convenience features; the hashmap commits may have design notes worth scanning.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; performance and dependency files aren't commonly featured topics in compiler tutorials. The hashmap is the most reusable data-structure addition of the entire compiler; it might be a concept-interlude candidate.

## Chapter 22 scope

**Title (working):** *Performance, dependency files, and the linker driver*.
**Commits:** 284–306 in chronological order on `main`. **Twenty-three commits** — the longest-running chapter since Ch 19.
**Concept interlude:** Possibly one. The hashmap (commits 285–288: introduce, then macro lookup, scope lookup, keyword lookup) is a genuine reusable data-structure addition. Its three call sites profile different access patterns: macro names (insert-heavy, occasional lookup), block-scope names (lookup-heavy, brief lifetime), keywords (insert-once, lookup-many). A short interlude on *open-addressing tradeoffs and the three call-site profiles* could land between §22.1 (the data structure) and §22.2 (the three uses). Default conditional — judge while reading the commits.

| # | Hash | Subject |
|---|---|---|
| 284 | `f0c98e0` | [GNU] Treat labels-as-values as compile-time constant |
| 285 | `0aad326` | Add string hashmap |
| 286 | `30520e5` | Use hashmap for macro name lookup |
| 287 | `655954e` | Use hashmap for block-scope lookup |
| 288 | `f694413` | Use hashmap for keyword lookup |
| 289 | `d0c4667` | Add -M option |
| 290 | `95d5a46` | Add -MF option |
| 291 | `57c1d4e` | Add -MP option |
| 292 | `db850f3` | Add -MT option |
| 293 | `fb5cfe5` | Add -MD option |
| 294 | `7aa72e4` | Add -MQ option |
| 295 | `c3edffb` | Add -MMD option |
| 296 | `86785fc` | Add -fpic and -fPIC options |
| 297 | `c0f0614` | Cache file search results |
| 298 | `d48d9e5` | Add include guard optimization |
| 299 | `a6c6622` | [GNU] Add "#pragma once" |
| 300 | `f10bceb` | [GNU] Add #include_next |
| 301 | `1e9b6dd` | Add -static option |
| 302 | `4e5de36` | Add -shared option |
| 303 | `c8df787` | Add -L option |
| 304 | `d1bc9a4` | Add -Wl, option |
| 305 | `469f159` | Add -Xlinker option |
| 306 | `fb49370` | Add scripts to test third-party apps |

Twenty-three commits. The natural section grouping:

- **§22.1 — Labels-as-values as compile-time constant** (commit 284). One commit. The §21.6 labels-as-values gave us `&&label` as an expression; this commit makes it usable in static initializers (e.g. global `void *jump_table[] = {&&l1, &&l2, &&l3};`). Touches the `eval` machinery and the global-initializer path. Walk how it slots into the existing `eval2`/`eval_rval` structure.
- **§22.2 — The string hashmap** (commit 285). One commit. A from-scratch open-addressing hashmap keyed by C strings. The most reusable data-structure addition of the entire compiler. Walk the API (`hashmap_get`, `hashmap_put`, `hashmap_delete`), the hash function, the load-factor and rehash trigger, the deletion-tombstone scheme.
- **§22.3 — Three hashmap users: macros, scopes, keywords** (commits 286–288). Three commits. Each replaces a linear-search structure with a hashmap lookup. Walk the access-pattern differences. Likely a concept interlude candidate before this section, or fold the comparison into the section opener.
- **§22.4 — The `-M` family** (commits 289–295). Seven commits. `-M`, `-MF`, `-MP`, `-MT`, `-MD`, `-MQ`, `-MMD` — the dependency-file generation that lets `make` know which headers a `.c` file pulls in. Each option does a small piece (output target name, escape characters, suppress-system-headers, write-to-file). Walk all seven; they're individually small but cumulatively define the output format. Likely the longest section of the chapter at ~2,500 words.
- **§22.5 — `-fpic`/`-fPIC` and file-search caching** (commits 296–297). Two commits. `-fpic`/`-fPIC` enable position-independent code generation; chibicc's implementation likely flips a flag without changing codegen substantially (it's a marker for the linker). File-search caching speeds up include resolution by remembering successful and failed lookups. Walk both. The cache is the more interesting half — it's where the `c0f0614` commit threads the hashmap from §22.2 into the include-search path.
- **§22.6 — Include-guard optimization, `#pragma once`, `#include_next`** (commits 298–300). Three commits. Include-guard optimization detects the classic `#ifndef X #define X ... #endif` pattern and skips the file body on subsequent inclusions. `#pragma once` is the GNU/MSVC extension that asks for the same effect explicitly. `#include_next` is the GNU mechanism for a header to re-include "the next file with the same name in the search path" (used for chained system headers). Walk all three; they form a small triplet of include-handling improvements.
- **§22.7 — Linker-driver pass-throughs and the third-party-app harness** (commits 301–306). Six commits. `-static`, `-shared`, `-L`, `-Wl,`, `-Xlinker` — five linker-related driver options that pass through to `ld`. Each is a small driver-side addition. The final commit (`fb49370`) adds shell scripts that build real-world C programs (libpng, sqlite, etc.) with chibicc. Walk all; the harness commit is a fitting place to close the chapter.

That's seven sections from twenty-three commits. **Target chapter length: ~12,000–14,000 words.** Likely closer to 13K — the `-M` family alone is seven commits with format details to walk, and the hashmap warrants substantial time.

## Steps

1. `cd research/sources/chibicc && for h in f0c98e0 0aad326 30520e5 655954e f694413 d0c4667 95d5a46 57c1d4e db850f3 fb5cfe5 7aa72e4 c3edffb 86785fc c0f0614 d48d9e5 a6c6622 f10bceb 1e9b6dd 4e5de36 c8df787 d1bc9a4 469f159 fb49370; do echo "===== $h ====="; git show --stat $h | head -10; done` to scan all 23 diffs.
2. Read each commit. Pay particular attention to:
   - **§22.1's `f0c98e0`** — the `eval` machinery has to learn that `&&label` is a compile-time-constant address. Walk how `eval2`/`eval_rval` extend; likely adds an `ND_LABEL_VAL` arm that returns a relocation against the label's symbol.
   - **§22.2's `0aad326`** — the hashmap is the most reusable data structure in the compiler. Walk its API, the hash function (likely FNV or DJBX33A), the resize/rehash logic, and the deletion-tombstone scheme.
   - **§22.3's three users** — macros (insert-heavy, lookup-occasional), scopes (lookup-heavy, lifetime-brief), keywords (insert-once, lookup-many). Walk each conversion — what was the linear search before, what's the hashmap call now, and what's the asymptotic improvement.
   - **§22.4's `-M` family** — these are subtle. `-M` writes the dependency rule to stdout; `-MF FILE` redirects to a file; `-MP` adds phony rules so deleted headers don't break the build; `-MT TARGET` overrides the target name; `-MD` enables dependency-file generation alongside compilation; `-MQ` is `-MT` with quoting; `-MMD` is `-MD` minus system headers. Walk how each option modifies the dependency-output state.
   - **§22.5's `-fpic`/`-fPIC`** — chibicc's codegen probably already emits PIC-friendly code (rip-relative for globals, no absolute addresses), so the option may be a flag-flip without semantic changes. Confirm.
   - **§22.5's `c0f0614` file-search cache** — uses the hashmap from §22.2. Walk the cache key (probably the include path entry) and the cache value (resolved path or "not found" sentinel).
   - **§22.6's include-guard optimization** — detects `#ifndef X #define X ... #endif`. Walk how the preprocessor recognizes the pattern, what it caches, and how subsequent `#include` calls short-circuit. Likely uses the hashmap.
   - **§22.6's `#pragma once`** — the explicit form of the same optimization. Walk how it integrates with the include-guard machinery.
   - **§22.6's `#include_next`** — the GNU extension. Walk how the search-path is iterated past the current file's directory.
   - **§22.7's linker pass-throughs** — small. `-Wl,foo,bar` becomes `foo bar` in the linker invocation. `-Xlinker arg` becomes `arg`. `-static`/`-shared`/`-L` are pass-throughs. Walk all five briefly.
   - **§22.7's `fb49370` third-party-app scripts** — verifies chibicc can build libpng, sqlite, etc. Walk the script structure and what it checks.
3. Read the destination state at `fb49370` for `parse.c`, `tokenize.c`, `codegen.c`, `chibicc.h`, `main.c`, `preprocess.c`, plus the new `hashmap.c`. The hashmap, the `-M` machinery, and the include-guard optimization are the three most invasive subsystems.
4. Draft `chapters/22-performance-deps-and-the-linker-driver.md`. Likely 12,000–14,000 words. Seven sections.
5. Write `docs/sessions/023-chapter-22-draft/README.md`.
6. Write `HANDOFF.md` for session 024 (Chapter 23 — Atomics and the final polish, commits 307–316).

## Voice / structure rules

Same as Ch 1–21:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, list the checkouts at the section opener.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present tense for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — twenty-three rows, single table is fine.
- Diff format: lean toward inline diff fragments and quoted file snippets. The §22.2 hashmap, §22.4 `-M` family, and §22.6 include-guard optimization will want larger code blocks.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **§22.2's hashmap is generic but the call sites have specific types.** The hashmap is keyed on C strings (`char *`) and stores `void *` values. Each call site casts the value at the boundary. Walk this; don't hide the casts.
- **§22.3's keyword-lookup conversion is the most interesting** because the keyword set is fixed. Pre-hashmap, keyword lookup was a linear scan of an array of ~30 strings. Post-hashmap, it's an `O(1)` lookup. Walk whether the hashmap is built once at startup or rebuilt per call.
- **§22.4's `-M` family generates Makefile syntax.** The dependency rule looks like `target: dep1 dep2 dep3 \\\n  dep4 dep5`. Walk how chibicc handles escape characters (spaces in paths become `\ `, dollar signs become `$$`). The format is documented in gcc's manual; chibicc reproduces it.
- **§22.5's `-fpic`/`-fPIC`** — these may not actually change codegen. Chibicc already emits rip-relative addressing for globals. The flag may just gate `.PIE`-related linker invocation. Confirm before writing.
- **§22.6's include-guard optimization is subtle.** The pattern is `#ifndef X / #define X / ... / #endif` with `X` as the same identifier in both lines. The preprocessor must verify the entire file body is wrapped in the guard (no leading non-whitespace content). Walk what counts as "leading."
- **§22.6's `#include_next`** is order-sensitive. The search starts *after* the current file's location in the search path. If the current file is `/usr/include/foo.h`, `#include_next` finds the next `foo.h` in `-I` paths after `/usr/include`. Walk how the resume-point is tracked.
- **§22.7's linker driver** — five small commits. Don't over-explain each one individually; group them in a single subsection with a small example.
- **§22.7's third-party-app harness** — the commit adds shell scripts. Walk what they build (libpng, sqlite, others?) and what they verify (compilation, linking, basic test execution). Don't list every script.

## Standing notes worth tracking across sessions

- **The hideset on Token** — unchanged through Ch 21. Ch 22's macro-lookup hashmap doesn't touch hideset semantics.
- **The Token->origin chain** — unchanged in Ch 21.
- **The `Token` line-marker fields** — `display_name`, `filename`, `line_delta` added in Ch 20 §20.1. Stable.
- **The eval-quartet duplication** — has a fifth member (`is_const_expr`) since Ch 21 §21.4. Likely picks up a sixth member or extends an existing one for §22.1's labels-as-values-as-compile-time-constant.
- **The cc1-vs-driver split** — unchanged.
- **The `Initializer` tree** — Ch 19 added `Member *mem`; Ch 21 §21.6 made array range designators honored in elaboration. Stable.
- **The local-vs-global split** — Ch 21 §21.1 added a third storage class for thread-local. `Obj` carries `is_tls`. Stable.
- **The `Relocation` mechanism** — likely changes in §22.1 for label-address compile-time relocations.
- **The anonymous-global pattern** — likely unchanged in Ch 22.
- **The `is_static` default in `new_gvar`** — gained `is_tls` companion in Ch 21. Stable.
- **The `is_definition` flag on `Obj`** — stable since Ch 20.
- **The `is_unsigned` flag on `Type`** — unchanged.
- **The `__va_area__` magic name** — unchanged.
- **The register-save-area layout** — unchanged.
- **The argreg integer/FP split** — long double is on-stack; SSE for FP, GP for integer. Stable.
- **The `Member->idx` field and bitfield siblings** — unchanged.
- **The `is_flexible` flag** — unchanged. The dead-code duplicate from §19.7's `835cd24` is still in the source.
- **`copy_struct_type`** — unchanged.
- **`MIN`/`MAX` macros** — unchanged.
- **`is_numeric` predicate** — gained TY_LDOUBLE in Ch 21 §21.6.
- **`is_flonum` and `has_flonum` diverged in Ch 21 §21.6.** `is_flonum` says yes for FLOAT, DOUBLE, LDOUBLE; `has_flonum` says yes only for FLOAT, DOUBLE.
- **Canonicalization-at-parse-time count is at eleven.** Up from ten with Ch 21 §21.4's VLA declaration rewrite. Ch 22 may add one in §22.1 (label-as-value rewriting to a relocation, or none).
- **Pre-factor-before-feature count is at nine.** Ch 22 unlikely to add new entries.
- **psABI conformance count is at eighteen.** Ch 22's `-fpic`/`-fPIC` may add one for PIC-specific psABI rules.
- **The fourth namespace (labels)** is unchanged. Labels-as-values still uses the same label table.
- **The `is_typename` predicate** stable since Ch 21 (added `_Thread_local`, `__thread`).
- **The `VarAttr` channel** has six fields after Ch 21 (typedef, static, extern, inline, tls, align). Stable.
- **The `ND_NULL_EXPR` seed-pattern** — used in Ch 21 §21.4's `compute_vla_size`. Stable.
- **The `rep stosb` pattern** — unchanged. `alloca` and VLA regions are not zero-initialized.
- **The `unreachable()` macro** — likely unchanged.
- **Per-token line numbers** — unchanged through Ch 21.
- **GDB-debuggable output** — unchanged.
- **Tests are in C.** Likely new test files for hashmap users (indirect — through existing tests with bigger inputs), `-M` family (driver tests), include-guard optimization (preprocessor tests), `#include_next` (specific test).
- **The `Obj->tok` field** added in Ch 14 §14.11 still has no readers.
- **The `Type->name_pos` field** — no new uses since Ch 14.
- **The `Type->origin` field** added in Ch 20 §20.3 for compatibility tracking. Stable.
- **The `Obj` struct gained two fields in Ch 21** (`is_tls`, `alloca_bottom`).
- **`Type` gained `vla_len`/`vla_size`** in Ch 21 §21.4.
- **The `Token`/`Node` `fval`** widened to `long double` in Ch 21 §21.6.
- **The `>>` codegen quirk** — partially repaired.
- **The `add_type` rule for `ND_STMT_EXPR`** — errata candidate.
- **The hex-escape silent truncation** — errata candidate.
- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels, struct-members — five errata candidates.
- **`f()` and `f(void)` are accepted as identical** — errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension.
- **`.bss` is the third assembly section.** `.tdata`/`.tbss` are added in Ch 21 §21.1, plus `.comm` for tentative commons. Total: five sections (`.text`, `.data`, `.bss`, `.tdata`, `.tbss`) plus `.comm` directive.
- **`.align`** — unchanged.
- **The `mov $0, %rax`** for variadic FP-count — errata candidate.
- **The `fp_offset = fp * 8 + 48` non-conforming stride** — errata candidate.
- **`long double` is `long double`** — *closed in Ch 21 §21.6*. No longer aliased to `double`.
- **The default-argument-promotion gap for chars and shorts** — errata candidate.
- **Float literals are inlined as integer-immediate-bit-cast.**
- **Long double literals are split across two halves through the redzone.**
- **Ch 1 errata list** unchanged.
- **The cast/compound-literal disambiguator** — same lookahead pattern.
- **The cast table is 11×11.** Grew in Ch 21 §21.6 with the F80 row/column. Stable.
- **Driver brittleness** — partially addressed by Ch 21's `-include`, `-x`, `-l`, `-s` additions. Ch 22's `-M` family, `-fpic`/`-fPIC`, `-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker` will round it out.
- **The link command's hardcoded distro list** — errata candidate.
- **`Node->funcname` is gone.**
- **`mov %rax, %r10; call *%r10` is uniform across all calls.**
- **The `StringArray` type** — picks up new uses for `-M` flags. Likely stable; may be supplemented by hashmap.
- **`atexit(cleanup)` for tempfile disposal** — unchanged.
- **The `run_subprocess` helper** — unchanged.
- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `opt_S | opt_E` typo; default include paths Linux/glibc-specific. Three remaining.
- **Errata candidates added in Ch 18:** None high-priority.
- **Errata candidates added in Ch 19:**
  - UTF-16 character-literal silent truncation of code points above U+FFFF.
  - Dead-code duplicate `is_flexible` block in `array_initializer1`.
  - Range designators not honored — *closed in Ch 21 §21.6 commit `3d5550e`*.
- **Errata candidates added in Ch 20:**
  - `is_compatible` array arm bug.
- **Errata candidates added in Ch 21:**
  - Missing `.size` directive for function symbols (in §21.5.3, commit `8d130ab`).
  - Suffix-only `.a`/`.so` recognition (in §21.5.4, commit `d56dd2f`).
- **Errata candidates closed in Ch 21:**
  - "long double is double" (closed by `e0bf168` in §21.6).
  - Range designators not honored (closed by `3d5550e` in §21.6).
- **`self.py` is gone.**
- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean.
- **Chibicc compiles itself** as of commit 197 (Ch 17 §17.6).
- **The `has_flonum` family** updated for long double in Ch 21 §21.6 (it now distinguishes from `is_flonum`).
- **Bitfield support is feature-complete.**
- **Anonymous struct/union members** flatten via recursive `get_struct_member`.
- **The pre-tokenize pass count is four** (Ch 19 §19.6): BOM, newline, backslash-newline, UCN. Unchanged.
- **The four char-literal prefixes** are functional.
- **The four string-literal prefixes** are functional.
- **`__STDC_UTF_16__` and `__STDC_UTF_32__`** are defined.
- **`__STDC_NO_VLA__`** — *no longer defined as of Ch 21 §21.4 commit `b0109a3`*.
- **`__STDC_NO_THREADS__`** — *no longer defined as of Ch 21 §21.1 commit `b377284`*.
- **UTF-8 in identifiers** uses C11 Annex D ranges.
- **The GNU `$` extension** in identifiers.
- **`__DATE__`, `__TIME__`, `__COUNTER__`, `__TIMESTAMP__`, `__BASE_FILE__`** are predefined.
- **Designated initializers** work for arrays, structs, unions, anonymous-struct, plus the GNU `=`-omission, plus array range designators.
- **`__VA_OPT__` and `,##__VA_ARGS__` are functional.**
- **GCC-style variadic macros (`name...`)** are functional.
- **`#pragma` is silently consumed** (will be partially superseded by `#pragma once` in Ch 22 §22.6).
- **`typeof`, `_Generic`, `__builtin_types_compatible_p`** are functional.
- **`sizeof(<function-type>)` returns 1.**
- **The GNU `?:`-omitted-middle** is functional.
- **`asm`** is verbatim-string-only.
- **`inline` is treated as `static`**, with dead-static-inline elimination.
- **`__attribute__` is macro-stubbed when `__GNUC__` is undefined.**
- **`-idirafter`, `-fcommon`/`-fno-common`** are functional.
- **`offsetof` is in `<stddef.h>`.**
- **Tentative definitions are functional.** `.comm` (under `-fcommon`) or `.bss` (under `-fno-common`).
- **`_Thread_local`/`__thread` (`%fs:`-relative addressing, `.tdata`/`.tbss`) are functional.**
- **`alloca` is a builtin** that synthesizes inline stack manipulation.
- **VLAs are functional**, allocated via `alloca` with sizes stored in hidden locals.
- **`-include`, `-x`, `-E xc`, `-l`, `-s`, `.a`/`.so`** are in the driver vocabulary.
- **`.type`/`.size`** directives are emitted (with `.size` missing for functions).
- **`long double` is real 80-bit extended precision.**
- **GNU case ranges (`case 1 ... 5:`)** are functional.
- **GNU array range designators (`[3 ... 7] = x`)** are honored in elaboration.
- **GNU labels-as-values (`&&label`, `goto *expr`)** are functional inside function bodies (compile-time constant variant in Ch 22).

## Acceptance criteria for Ch 22

- [ ] `chapters/22-performance-deps-and-the-linker-driver.md` exists, end-to-end readable.
- [ ] All twenty-three commits covered, grouped into ~7 sections.
- [ ] §22.1 walks how labels-as-values gain compile-time-constant status.
- [ ] §22.2 walks the hashmap data structure (API, hash function, resize, deletion).
- [ ] §22.3 walks the three call sites and their access-pattern differences.
- [ ] §22.4 walks all seven `-M` options with attention to the dependency-file output format.
- [ ] §22.5 walks `-fpic`/`-fPIC` and the file-search cache, naming whether `-fpic` actually changes codegen.
- [ ] §22.6 walks include-guard optimization, `#pragma once`, and `#include_next`.
- [ ] §22.7 walks the five linker pass-throughs and the third-party-app harness.
- [ ] Voice matches Ch 1–21.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md` (Ch 23 = 307–316).
- [ ] psABI conformance count noted (`-fpic`/`-fPIC` may grow it by one).
- [ ] `docs/sessions/023-chapter-22-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 024 (Chapter 23 — Atomics and the final polish, commits 307–316).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/022-chapter-21-draft/HANDOFF.md  (this handoff)
2. docs/sessions/022-chapter-21-draft/README.md   (what session 022 did)
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
23. chapters/21-thread-local-alloca-vlas.md (most recent chapter)
24. research/commits/chapter-mapping.md            (confirms Ch 22 scope)
25. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 22 (Performance, dependency files, and the linker
driver, commits 284–306) per the steps in the handoff. Twenty-three
commits, seven sections proposed in the handoff. The §22.2 hashmap is
the most reusable data-structure addition of the entire compiler; the
§22.3 three-call-site walk profiles macros, scopes, and keywords; the
§22.4 -M family is seven small commits whose cumulative format defines
chibicc's dependency-file output. End-of-session: write your session
dir under docs/sessions/023-chapter-22-draft/ with a README and a
HANDOFF for session 024 (Chapter 23 — Atomics and the final polish,
commits 307–316).
```
