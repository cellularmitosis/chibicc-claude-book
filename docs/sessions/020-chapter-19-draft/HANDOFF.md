# Handoff: Ch 19 done → proceed to Ch 20

**For:** the next claude session.
**From:** session 020.
**Status:** Ch 19 drafted (~12,128 words, twenty-four commits, the full Unicode arc plus designated initializers plus the trailing date/time/counter macros). Continue autonomously to Ch 20 (GCC extensions worth supporting, commits 245–266 — twenty-two commits covering improved multibyte error messages, `#line` and the GNU line marker directive, `__TIMESTAMP__` and `__BASE_FILE__`, `__VA_OPT__` and the GNU `,##__VA_ARGS__` swallow, `#pragma`, GCC-style variadic macros, `typeof`, `__builtin_types_compatible_p`, `_Generic`, `sizeof` of a function type, the GNU `?:` with omitted middle operand, basic `asm`, `inline` as static, `__attribute__((format))`, `-idirafter`, `offsetof`, tentative definitions, and `-fcommon`/`-fno-common`). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/020-chapter-19-draft/README.md`](README.md)** — what session 020 did, including the seven-section structure (per-commit subsections in §19.4, §19.5, §19.6, §19.7; integrated subsections in §19.1; no subsections in §19.2 and §19.3), the closure of the Ch 17 `L''` ≡ `''` errata in §19.4, three new errata candidates surfaced (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block, half-implemented range designator), the unchanged counts (canonicalization-at-parse-time at nine, pre-factor at nine, psABI conformance at sixteen).
2. **[`docs/sessions/019-chapter-18-draft/HANDOFF.md`](../019-chapter-18-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply with Ch 19 updates folded in (see §20 README for the running list).
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`19-unicode-and-designated-initializers.md`](../../../chapters/19-unicode-and-designated-initializers.md)** — the nineteen chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 20 = commits 245–266 (22 commits, scoped to "GCC extensions worth supporting").
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes. Ch 20's commits are mostly feature additions; less commit-message material than the early chapters but worth scanning. The `_Generic` and `typeof` and `__VA_OPT__` commits may have design notes.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; `_Generic` and `typeof` could be candidates for a concept interlude.

## Chapter 20 scope

**Title (working):** *GCC extensions worth supporting*.
**Commits:** 245–266 in chronological order on `main`. **Twenty-two commits** — back below the size of Chapter 19 (24 commits) and Chapter 18 (23 commits).
**Concept interlude:** Possibly one. `_Generic` is type-driven dispatch at compile time and the implementation walks each association arm; `typeof` is a parser-side feature that has interesting interactions with the type-name vs expression decision. A short interlude on *how chibicc routes type-vs-expression contextual decisions* (the `is_typename` predicate, the cast-vs-compound-literal disambiguator from Ch 13, and now `typeof`/`_Generic`) could land in §20.5 or §20.6. Default conditional — judge while reading the commits.

| # | Hash | Subject |
|---|---|---|
| 245 | `37998be` | Improve error message for multibyte characters |
| 246 | `c61c0d0` | Add #line |
| 247 | `aaf20fb` | [GNU] Add line marker directive |
| 248 | `922604a` | [GNU] Add __TIMESTAMP__ macro |
| 249 | `3a10c8a` | [GNU] Add __BASE_FILE__ macro |
| 250 | `3381448` | Add __VA_OPT__ |
| 251 | `083c275` | [GNU] Handle ,##__VA_ARG__ |
| 252 | `74ec9f6` | Ignore #pragma |
| 253 | `007e526` | [GNU] Support GCC-style variadic macro |
| 254 | `7d80a51` | Add typeof |
| 255 | `1433b40` | [GNU] Add __builtin_types_compatible_p |
| 256 | `1faab48` | Add _Generic |
| 257 | `aee7891` | [GNU] Allow sizeof(<function type>) |
| 258 | `e28a612` | [GNU] Add ?: operator with omitted operand |
| 259 | `a253516` | Add basic "asm" statement |
| 260 | `31087f8` | Handle inline functions as static functions |
| 261 | `e5f4ca9` | Do not emit static inline functions if referenced by no one |
| 262 | `6a2dc5a` | Use __attribute__((format(print, ...))) to find programming errors |
| 263 | `11fc259` | Add -idirafter option |
| 264 | `1b99bad` | Add offsetof |
| 265 | `85e46b1` | Add tentative definition |
| 266 | `6d344ed` | Add -fcommon and -fno-common flags |

Twenty-two commits. The natural section grouping:

- **§20.1 — Multibyte error message + `#line` and line markers** (commits 245–247). Three commits. The error-message tweak for multibyte characters is the smallest possible polish for §19. `#line` is a directive that overrides the line/file the preprocessor reports in subsequent tokens; the GNU line-marker directive (`# 123 "file"`) is gcc's preprocessor-output format that cc1 also has to read for the `-E` round-trip.
- **§20.2 — Macro extensions: `__TIMESTAMP__`, `__BASE_FILE__`, `__VA_OPT__`, `,##__VA_ARGS__`, `#pragma`, GCC-style variadic** (commits 248–253). Six commits. `__TIMESTAMP__` is "MM DD HH:MM:SS YYYY" of the source file's mtime. `__BASE_FILE__` is the top-level source filename. `__VA_OPT__` is C2X's conditional-expansion feature. The `,##__VA_ARGS__` is GCC's swallow-the-comma trick for empty variadics. `#pragma` is silently ignored. GCC-style variadic macros are `#define foo(args...)` (vs C's `#define foo(...)`).
- **§20.3 — Type-side extensions: `typeof`, `__builtin_types_compatible_p`, `_Generic`** (commits 254–256). Three commits. These are the chapter's most parser-invasive changes. `typeof(expr)` and `typeof(type)` produce a type from an expression or a typename. `__builtin_types_compatible_p` is a compile-time predicate over two type-names. `_Generic` is the C11 type-driven dispatch.
- **§20.4 — Sizeof-of-function and the GNU ternary middle** (commits 257–258). Two commits. `sizeof(<function-type>)` returns 1 (an extension; standard C says it's a constraint violation). `a ?: b` is a GNU extension where the middle operand is omitted and `a` serves as both the condition and (when truthy) the value.
- **§20.5 — `asm`, `inline` (× 2), `__attribute__((format))`** (commits 259–262). Four commits. Basic `asm` statement support. `inline` is treated as `static` (chibicc doesn't do real inlining). The "do not emit static inline functions if referenced by no one" commit is a small linker-friendliness optimization. The `__attribute__((format(print, ...)))` annotation is added to chibicc's own `error*` and `warn` functions to catch printf-format mismatches at compile time.
- **§20.6 — `-idirafter`, `offsetof`, tentative definitions, `-fcommon`** (commits 263–266). Four commits. `-idirafter` is the include-path family's "after the standard paths" entry. `offsetof` is the standard library macro that chibicc can now define using `__builtin_offsetof` (added in this commit). Tentative definitions are the C feature that `int x;` at file scope is a tentative definition that can be overridden by a later real definition. `-fcommon` and `-fno-common` toggle the tentative-definition placement (`.bss` vs `.comm`).

That's six sections from twenty-two commits. **Target chapter length: ~11,000–14,000 words.** Likely closer to 12K — the GCC-extension commits are mostly small (`#pragma` is one line; `sizeof(<function>)` is two lines), and the larger ones (`_Generic`, `typeof`, `asm`, line markers, tentative definitions) compress well.

## Steps

1. `cd research/sources/chibicc && for h in 37998be c61c0d0 aaf20fb 922604a 3a10c8a 3381448 083c275 74ec9f6 007e526 7d80a51 1433b40 1faab48 aee7891 e28a612 a253516 31087f8 e5f4ca9 6a2dc5a 11fc259 1b99bad 85e46b1 6d344ed; do echo "===== $h ====="; git show --stat $h | head -10; done` to scan all 22 diffs.
2. Read each commit. Pay particular attention to:
   - **§20.1's `#line` and line markers** — both have to coexist with chibicc's per-token line-number tracking from Ch 8 §8.3. Read carefully. The GNU line-marker form is `# 123 "file"` or `# 123 "file" 1` etc.; the trailing flag is a pushed/popped/system-header marker.
   - **§20.2's `__VA_OPT__`** — C2X feature. Walk the implementation; this is the most interesting macro-side change in the chapter.
   - **§20.2's `,##__VA_ARGS__`** — the swallow-the-comma trick. Walk how the preprocessor recognizes the pattern. Probably a special case in the substitution code.
   - **§20.2's GCC variadic** — `args...` instead of `...`. Probably a one-line addition to `read_macro_params`.
   - **§20.3's `typeof`** — parser-side change. Read the new type-parser arm. Probably reuses `is_typename` but extends it for `typeof` token.
   - **§20.3's `_Generic`** — the most invasive single change. Parse the association list, compare types, return the matching expression. Walk the implementation step by step. The "default" arm and the "no match found" error case are both interesting.
   - **§20.5's `asm`** — basic only (no clobbers, no inputs/outputs). Read the implementation. Probably a one-statement parser arm that emits raw `.s` output.
   - **§20.5's inline-as-static** — chibicc doesn't actually inline; `inline` is just a synonym for `static`. The "don't emit if no references" commit follows up with a deferred-emit pass. Walk both.
   - **§20.6's tentative definitions** — Ch 13's linkage section had this on the to-do list. Now it's done. Walk how `new_gvar` and `parse` cooperate. The `-fcommon` flag toggles placement.
3. Read the destination state at `6d344ed` for `parse.c`, `tokenize.c`, `preprocess.c`, `codegen.c`, `chibicc.h`, `main.c`. The `_Generic` and `typeof` parser changes are likely the most invasive; tentative definitions touch `parse.c` and `codegen.c`.
4. Draft `chapters/20-gcc-extensions-worth-supporting.md`. Likely 11,000–14,000 words. Six sections.
5. Write `docs/sessions/021-chapter-20-draft/README.md`.
6. Write `HANDOFF.md` for session 022 (Chapter 21 — Thread-local, alloca, VLAs, commits 267–283).

## Voice / structure rules

Same as Ch 1–19:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, list the checkouts at the section opener.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present tense for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — twenty-two rows, single table is fine.
- Diff format: lean toward inline diff fragments and quoted file snippets. The §20.3 `_Generic` and `typeof` will want larger code blocks; the §20.6 tentative-definition codegen too.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **§20.1's `#line` is a preprocessor directive**, not a tokenizer feature — it lands in `preprocess.c`. The GNU line-marker is parsed when the preprocessor *reads* a file produced by `-E` (cc1 must round-trip its own preprocessor output).
- **§20.2's `__TIMESTAMP__` is the file's mtime**, not the compilation time. Different from `__DATE__`/`__TIME__` (which are compilation-time). Walk how the implementation gets the mtime — probably `stat` or `fstat`.
- **§20.2's `__VA_OPT__` is non-trivial to implement.** It's a token-list operation: in `__VA_OPT__(X)`, the tokens between the parens are emitted iff `__VA_ARGS__` is non-empty. Walk carefully.
- **§20.3's `typeof`** must be careful around side effects — `typeof` doesn't evaluate its argument. The chibicc implementation probably suspends evaluation while parsing.
- **§20.3's `_Generic`** is type-equality-based dispatch. The C standard's compatibility rule for "same type" is subtle (e.g., `int` and `int` are compatible; `int` and `int *` are not; `int *` and `void *` are not under `_Generic`'s equality). Walk what chibicc does.
- **§20.4's `sizeof(<function-type>)`** returning 1 is a GNU extension; standard C makes this a constraint violation. The motivation is that gcc uses it for various sneaky tricks (`offsetof`-style member-pointer arithmetic, etc.). Note this in the prose.
- **§20.5's `asm`** is very minimal in chibicc — just emits the string as raw assembly. No clobbers, no inputs, no outputs, no goto. Real codebases that use `asm` will hit this limit.
- **§20.5's inline-as-static** — chibicc treats `inline` as `static`. This is *almost* what the C standard says, but not quite (the standard's `inline` rules are more complex around external linkage). Note the simplification.
- **§20.6's tentative definition** is a real C feature, not a GNU extension. Two file-scope `int x;` declarations were a parse error pre-this-commit; post-commit, they collapse into one definition. The `-fcommon` flag (default off in modern gcc, default on historically) controls whether tentative definitions go to `.bss` or `.comm`.
- **§20.6's `-fcommon` is default-off** — gcc 10+ defaults to `-fno-common` (which puts tentative defs in `.bss`). chibicc's default is unclear; check the diff.
- **The `__attribute__((format))` commit** is just adding annotations to chibicc's own source; don't read it as adding parser support for `__attribute__` (that's a different, smaller change probably also in the chapter — verify while drafting).

## Standing notes worth tracking across sessions

- **The hideset on Token** — unchanged through Ch 19. Ch 20's `__VA_OPT__` and `,##__VA_ARGS__` will likely interact with the existing hideset/expansion machinery. Verify while drafting.
- **The Token->origin chain** — likely picks up new uses for the line-marker round-trip.
- **The eval-quartet duplication** — unchanged through Ch 19.
- **The cc1-vs-driver split** — unchanged.
- **The `Initializer` tree** — Ch 19 added `Member *mem` for unions. No expected changes in Ch 20.
- **The local-vs-global split** — stable. Tentative definitions affect the global path.
- **The `Relocation` mechanism** — likely unchanged in Ch 20.
- **The anonymous-global pattern** — likely unchanged in Ch 20.
- **The `is_static` default in `new_gvar`** — likely changes for `inline`-as-static (Ch 20 §20.5).
- **The `is_definition` flag on `Obj`** — likely changes for tentative definitions (Ch 20 §20.6).
- **The `is_unsigned` flag on `Type`** — unchanged.
- **The `__va_area__` magic name** — unchanged.
- **The register-save-area layout** — unchanged.
- **The argreg integer/FP split** — unchanged.
- **The `Member->idx` field and bitfield siblings** — unchanged.
- **The `is_flexible` flag** — unchanged. The dead-code duplicate from §19.7's `835cd24` is still in the source; if `array_initializer1` is touched in Ch 20 the prose should note whether the duplicate is finally removed.
- **`copy_struct_type`** — unchanged.
- **`MIN`/`MAX` macros** — unchanged.
- **`is_numeric` predicate** — unchanged.
- **Canonicalization-at-parse-time count is at nine.** Ch 20 might add one in §20.4 (the `?:`-with-omitted-middle commit may rewrite to a temporary-binding form) — verify while drafting.
- **Pre-factor-before-feature count is at nine.** Ch 20 unlikely to add new entries.
- **psABI conformance count is at sixteen.** Ch 20 unlikely to touch it.
- **The fourth namespace (labels)** is unchanged.
- **The `is_typename` predicate** likely changes in §20.3 (`typeof` is a typename-introducer).
- **The VarAttr channel** has four fields. Ch 20 might grow it — `inline` and `is_tentative` are candidate fields.
- **The `ND_NULL_EXPR` seed-pattern** — unchanged.
- **The `rep stosb` pattern** — unchanged.
- **The `unreachable()` macro** — likely picks up new callers in `_Generic`'s default-not-found case and elsewhere.
- **Per-token line numbers** — preserved through preprocessing as of Ch 17. Will be touched by `#line` in §20.1.
- **GDB-debuggable output** — unchanged.
- **Tests are in C.** New test file likely for `_Generic`. Driver tests for `-idirafter`, `-fcommon`. `asm` tests in shell driver.
- **The `Obj->tok` field** added in Ch 14 §14.11 still has no readers.
- **The `Type->name_pos` field** — no new uses since Ch 14.
- **The `>>` codegen quirk** — partially repaired.
- **The `add_type` rule for `ND_STMT_EXPR`** — errata candidate.
- **The hex-escape silent truncation** — errata candidate.
- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels, struct-members — five errata candidates.
- **`f()` and `f(void)` are accepted as identical** — errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension.
- **`.bss` is the third assembly section.** `-fcommon` may add `.comm` as a fourth path.
- **`.align`** — unchanged.
- **The `mov $0, %rax`** for variadic FP-count — errata candidate.
- **The `fp_offset = fp * 8 + 48` non-conforming stride** — errata candidate.
- **`long double` is `double`** — errata candidate.
- **The default-argument-promotion gap for chars and shorts** — errata candidate.
- **Float literals are inlined as integer-immediate-bit-cast.**
- **Ch 1 errata list** unchanged.
- **The cast/compound-literal disambiguator** — same lookahead pattern.
- **The cast table is 10×10.** Possibly grows in §20.3 if `_Generic` introduces new cast cells; verify while drafting.
- **Driver brittleness** — unchanged.
- **The link command's hardcoded distro list** — errata candidate.
- **`Node->funcname` is gone.**
- **`mov %rax, %r10; call *%r10` is uniform across all calls.**
- **The `StringArray` type** — used by `include_paths`. `-idirafter` will likely add to it.
- **`atexit(cleanup)` for tempfile disposal** — unchanged.
- **The `run_subprocess` helper** — unchanged.
- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `L''` ≡ `''` (closed by Ch 19's `a57c661`); `__va_arg_mem` divides by zero (closed by Ch 18); `opt_S | opt_E` typo; default include paths Linux/glibc-specific. Two closed; three remaining.
- **Errata candidates added in Ch 18:** None high-priority. The bitfield zero-width test exposed the missing struct-member-name redeclaration check.
- **Errata candidates added in Ch 19:**
  - UTF-16 character-literal silent truncation of code points above U+FFFF (in §19.4, commit `454618c`).
  - Dead-code duplicate `is_flexible` block in `array_initializer1` (in §19.7, commit `835cd24`).
  - Range designators `[3 ... 7]` syntactically accepted but not honored in elaboration (in §19.7, commit `835cd24`).
- **`self.py` is gone.**
- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean.
- **Chibicc compiles itself** as of commit 197 (Ch 17 §17.6).
- **The `has_flonum` family** unchanged.
- **Bitfield support is feature-complete.**
- **Anonymous struct/union members** flatten via recursive `get_struct_member`.
- **The pre-tokenize pass count is four** (Ch 19 §19.6): BOM, newline, backslash-newline, UCN. Order matters.
- **The four char-literal prefixes** are functional. Types: `'X'` → `int` then `(char)`; `L'X'` → `int`; `u'X'` → `unsigned short` masked to 16; `U'X'` → `unsigned int`.
- **The four string-literal prefixes** are functional. Element types: ordinary and `u8` → `char`; `u` → `unsigned short`; `U` → `unsigned int`; `L` → `int`. The cross-prefix concat re-tokenizes ordinary literals to match wide neighbors.
- **`__STDC_UTF_16__` and `__STDC_UTF_32__`** are defined.
- **UTF-8 in identifiers** uses C11 Annex D ranges. `is_ident1`/`is_ident2` live in `unicode.c`.
- **The GNU `$` extension** in identifiers.
- **`__DATE__`, `__TIME__`, `__COUNTER__`** are predefined.
- **Designated initializers** work for arrays, structs, unions, anonymous-struct, plus the GNU `=`-omission. The `Initializer` tree's `Member *mem` field handles unions.

## Acceptance criteria for Ch 20

- [ ] `chapters/20-gcc-extensions-worth-supporting.md` exists, end-to-end readable.
- [ ] All twenty-two commits covered, grouped into ~6 sections.
- [ ] §20.1 explains `#line` and line markers and how they coexist with the existing per-token line-number tracking from Ch 8 §8.3.
- [ ] §20.2 walks `__VA_OPT__` and `,##__VA_ARGS__` step by step, with at least one test case for each.
- [ ] §20.3 walks `typeof` as a parser-side typename-introducer, walks `_Generic` as a type-equality-based dispatcher, and walks `__builtin_types_compatible_p` as a compile-time predicate.
- [ ] §20.5 names that chibicc treats `inline` as `static` (a simplification of the standard's actual `inline` rules).
- [ ] §20.6 walks tentative definitions and the `-fcommon`/`-fno-common` toggle.
- [ ] Voice matches Ch 1–19.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] psABI conformance thread count noted as still at sixteen unless Ch 20 commits add to it (they probably don't).
- [ ] `docs/sessions/021-chapter-20-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 022 (Chapter 21 — Thread-local, alloca, VLAs, commits 267–283).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/020-chapter-19-draft/HANDOFF.md  (this handoff)
2. docs/sessions/020-chapter-19-draft/README.md   (what session 020 did)
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
21. chapters/19-unicode-and-designated-initializers.md (most recent chapter)
22. research/commits/chapter-mapping.md            (confirms Ch 20 scope)
23. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 20 (GCC extensions worth supporting, commits
245–266) per the steps in the handoff. Twenty-two commits, six
sections proposed in the handoff. The type-side extensions arc (§20.3,
three commits — typeof, __builtin_types_compatible_p, _Generic) is the
chapter's most parser-invasive stretch and is where a possible concept
interlude on type-vs-expression context lands. End-of-session: write
your session dir under docs/sessions/021-chapter-20-draft/ with a
README and a HANDOFF for session 022 (Chapter 21 — Thread-local,
alloca, VLAs, commits 267–283).
```
