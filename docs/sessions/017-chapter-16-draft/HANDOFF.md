# Handoff: Ch 16 done → proceed to Ch 17

**For:** the next claude session.
**From:** session 017.
**Status:** Ch 16 drafted (~9,688 words, eight commits, the compiler-driver chapter — stage-2 Makefile target, function-pointer trio bundled into one section, cc1-vs-driver split, `as` invocation, multiple input files, `ld` invocation). Continue autonomously to Ch 17 (A preprocessor from scratch, the longest single arc in the book — 40 commits ending in self-hosting). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/017-chapter-16-draft/README.md`](README.md)** — what session 017 did, including the six-section structure (function-pointer trio bundled into §16.2 as predicted, the other six commits getting their own sections), the no-interlude decision (the cc1-vs-driver split's substance is §16.3's body, the GOT/PIC story is §16.2's body — pulling either out leaves the host section without prose), the "psABI conformance ticks to nine" call-out (the `gen_addr` GOT path is the chapter's quiet conformance correction), the "function-pointer/array-decay parallel is exact in two of three pieces" framing, the `Node->funcname` deletion noted, the `is_definition` finally has a second reader noted, the `StringArray` type's four post-Ch 16 uses noted, the chapter's ~9,688 word length (slightly above forecast upper edge).
2. **[`docs/sessions/016-chapter-15-draft/HANDOFF.md`](../016-chapter-15-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply with Ch 16 updates folded in (see §17 README for the running list).
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`16-the-compiler-driver.md`](../../../chapters/16-the-compiler-driver.md)** — the sixteen chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 17 = commits 158–197 (40 commits, sub-sectioned by topic). The chapter ends with self-hosting.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes. The preprocessor commits may have especially apt commit-message material.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; Ch 17 (preprocessor) likely doesn't have a clean interlude candidate, but the *macro expansion algorithm* (the no-rescan-set / hideset machinery) might be a candidate if the prose surfaces a need.

## Chapter 17 scope

**Title (working):** *A preprocessor from scratch*.
**Commits:** 158–197 in chronological order on `main`. **Forty commits — the longest single arc in the book. Ends with self-hosting at commit 197.**
**Concept interlude:** Likely. The macro-expansion algorithm (the "don't expand a token more than once" rule, implemented via a hideset on each token) is a substantial enough concept that an interlude on *how compilers stop macro recursion without reaching deep nesting* might earn its place. Default-yes, conditional on whether §17.4's prose surfaces the need. Optional second interlude on what *self-hosting* means as a concept (it's a chapter-closer; could be its own discussion).

| # | Hash | Subject |
|---|---|---|
| 158 | `1e1ea39` | Add a do-nothing preprocessor |
| 159 | `146c7b3` | Add the null directive |
| 160 | `d367510` | Add `#include "..."` |
| 161 | `ec149f6` | Skip extra tokens after `#include "..."` |
| 162 | `d138864` | Add `-E` option |
| 163 | `bf6ff92` | Add `#if` and `#endif` |
| 164 | `aa570f3` | Skip nested `#if` in a skipped `#if`-clause |
| 165 | `c6e81d2` | Add `#else` |
| 166 | `e7a1857` | Add `#elif` |
| 167 | `97d33ad` | Add objlike `#define` |
| 168 | `9ad60e4` | Add `#undef` |
| 169 | `2651448` | Expand macros in the `#if` and `#elif` argument context |
| 170 | `acce002` | Do not expand a token more than once for the same objlike macro |
| 171 | `1f80f58` | Add `#ifdef` and `#ifndef` |
| 172 | `dec3b3f` | Add zero-arity funclike `#define` |
| 173 | `b9ad3e4` | Add multi-arity funclike `#define` |
| 174 | `dd4306c` | Allow empty macro arguments |
| 175 | `c7d7ce0` | Allow parenthesized expressions as macro arguments |
| 176 | `1313fc6` | Do not expand a token more than once for the same funclike macro |
| 177 | `8f6f792` | Add macro stringizing operator (`#`) |
| 178 | `8f561ae` | Add macro token-pasting operator (`##`) |
| 179 | `769b5a0` | Use chibicc's preprocessor for all tests |
| 180 | `5cb2f89` | Add `defined()` macro operator |
| 181 | `a8d76ad` | Replace remaining identifiers with 0 in macro constexpr |
| 182 | `8075582` | Preserve newline and space during macro expansion |
| 183 | `b33fe0e` | Support line continuation |
| 184 | `d85fc4f` | Add `#include <...>` |
| 185 | `a1dd621` | Add `-I<dir>` option |
| 186 | `a939a7a` | Add default include paths |
| 187 | `e7fdc2e` | Add `#error` |
| 188 | `5f5a850` | Add predefine macros such as `__STDC__` |
| 189 | `6f17071` | Add `__FILE__` and `__LINE__` |
| 190 | `dc01f94` | Add `__VA_ARGS__` |
| 191 | `ba6b4b6` | Add `__func__` |
| 192 | `82ba010` | [GNU] Add `__FUNCTION__` |
| 193 | `ab4f1e1` | Concatenate adjacent string literals |
| 194 | `7746e4e` | Recognize wide character literal |
| 195 | `7cbfd11` | Add stdarg.h, stdbool.h, stddef.h, stdalign.h and float.h |
| 196 | `5322ea8` | Add `va_arg()` |
| 197 | `12a9e75` | Self-host: including preprocessor, chibicc can compile itself |

Forty commits. **Bundling is essential** — at one section per commit this would be a 30,000-word chapter, and the chapter mapping forecast already groups by topic. The proposed structure:

- **§17.1 — The skeleton** (commits 158–162). Five commits: do-nothing preprocessor, null directive, `#include "..."`, skip-extra-tokens, `-E` option. The shape is laid out: a `preprocess()` pass between tokenize and parse, a directive recognizer, the basic file-inclusion machinery, the user-visible `-E` flag for "preprocess-only output."
- **§17.2 — Conditionals** (commits 163–166). Four commits: `#if` and `#endif`, nested-skip, `#else`, `#elif`. The `#if` constexpr evaluator is the meat — chibicc has to evaluate constant expressions at preprocess time, before parse time, with no type-system context. Worth bundling because the four commits are progressively-bigger conditional shapes.
- **§17.3 — Object-like macros** (commits 167–171, with 169 as a sub-step). Five commits: `#define`, `#undef`, expansion in `#if`/`#elif` context, the *no-rescan-of-already-expanded-token* rule, `#ifdef`/`#ifndef`. The no-rescan rule is the most subtle thing in the chapter and probably wants the concept interlude.
- **§17.4 — Function-like macros** (commits 172–178). Seven commits: zero-arity, multi-arity, empty args, parenthesized args, the second no-rescan rule (function-like-macro variant), stringizing `#`, pasting `##`. The function-like-macro arc is the most code-heavy stretch in the chapter.
- **§17.5 — Polish and full preprocessor surface** (commits 179–196). Eighteen commits: switch tests to chibicc's preprocessor, `defined()`, identifier-to-zero in constexpr, whitespace preservation, line continuation, `<...>` includes, `-I`, default include paths, `#error`, predefined macros (`__STDC__`, `__FILE__`, `__LINE__`, `__VA_ARGS__`, `__func__`, `__FUNCTION__`), adjacent-string concatenation, wide character literal, stdarg-and-friends header bundle, `va_arg()`. Lots of small commits. Probably needs sub-grouping inside §17.5: paths/include (commits 184–186), error-and-line (commits 181–183, 187), predefined macros (188–192), tail-of-language (193–196).
- **§17.6 — Self-hosting** (commit 197). The chapter's punchline. `self.py` retires; chibicc compiles itself. Worth its own short section even though it's one commit, because it's the moment chibicc *finishes*.

That's six sections from forty commits. **Target chapter length: ~15,000–20,000 words.** This is going to be the longest chapter in the book; resist trying to make it shorter than the material wants. The Japanese book's preprocessor coverage runs similarly long; the chibicc commits naturally bundle into the six topics above.

## Steps

1. `cd research/sources/chibicc && for h in 1e1ea39 146c7b3 d367510 ec149f6 d138864 bf6ff92 aa570f3 c6e81d2 e7a1857 97d33ad 9ad60e4 2651448 acce002 1f80f58 dec3b3f b9ad3e4 dd4306c c7d7ce0 1313fc6 8f6f792 8f561ae 769b5a0 5cb2f89 a8d76ad 8075582 b33fe0e d85fc4f a1dd621 a939a7a e7fdc2e 5f5a850 6f17071 dc01f94 ba6b4b6 82ba010 ab4f1e1 7746e4e 7cbfd11 5322ea8 12a9e75; do echo "===== $h ====="; git show --stat $h | head -8; done` to scan all forty diffs.
2. Read each commit. Pay particular attention to:
   - **§17.1's skeleton (158–162):** The `preprocess()` function's structure — how tokens flow through the new pass, how directives are recognized, how `#include` handles file lookup and recursive expansion. Also: the `-E` flag's interaction with the §16 driver (does `-E` go through cc1, or is preprocessing a separate phase?). Read `cc1.c`/`main.c` for how `-E` is handled.
   - **§17.2's `#if` constexpr evaluator (163–166):** Chibicc's preprocessor has to evaluate constant expressions *before* the regular parser runs. There's likely a small evaluator that runs on tokens directly, separate from `eval`/`eval2`/`eval_double` (which run on AST nodes). Note the duplication if it's there — it's interesting precedent for what *can* be unified vs. what stays separate.
   - **§17.3's no-rescan rule (170):** This is the most subtle thing in the C preprocessor. The rule prevents `#define X X` from infinitely expanding. Chibicc probably implements it via a "hideset" on each token — a set of macro names that have already been expanded into this token's neighborhood. Read carefully; the data structure is the chapter's most interesting piece of design.
   - **§17.4's function-like macro arc (172–178):** Argument tokenization, parameter substitution, the second no-rescan rule (which is more subtle than the first because function-like macros can be nested), stringizing (`#x` → `"x"`), pasting (`a ## b` → `ab`). The token-pasting operator's interaction with whitespace and macro arguments is the trickiest single piece.
   - **§17.5's predefined macros (188–192):** `__FILE__` and `__LINE__` in particular need to track where the preprocessor *thinks* it is, which differs from the original tokenizer's position when macros expand. Note how chibicc tracks this — probably a `Token->file` and `Token->line` pair that survive expansion.
   - **§17.6's self-hosting (197):** The Makefile change that makes `self.py` retire. Likely a one-line Makefile change plus possibly some chibicc.h / source adjustments to fit the now-real preprocessor's expectations. Read for *what no longer needs `self.py`*.
3. Read the destination state at `12a9e75` for `chibicc.h`, `preprocess.c` (new file likely), `tokenize.c`, `main.c`, and the Makefile.
4. Draft `chapters/17-a-preprocessor-from-scratch.md`. Likely 15,000–20,000 words. Six sections.
5. Write `docs/sessions/018-chapter-17-draft/README.md`.
6. Write `HANDOFF.md` for session 019 (Chapter 18 — The full ABI, commits 198–220; ~23 commits covering stack-passed args, struct-by-value, `va_copy`, bitfields, etc.).

## Voice / structure rules

Same as Ch 1–16:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For bundled sections (most of Ch 17's sections), each bundled commit gets its own opener.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present tense for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — forty rows is too many; consider grouping into the six section topics with a sub-table per section, or doing the closer table at the section-summary level rather than the commit level. Decide while drafting.
- Diff format: lean toward inline diff fragments and quoted file snippets. The §17.3 no-rescan rule and §17.4 function-like-macro logic will want larger code blocks. The §17.6 self-hosting commit may be one of the *smallest* code blocks in the chapter (a Makefile change), but the *prose* around it should reflect the milestone weight.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage. Ch 17 may have especially good commit-message material; check `quotes-rui.md` before drafting.
- **§17.1's skeleton is a do-nothing pass at first.** Don't oversell it. The first commit literally does nothing — `preprocess()` returns its input unchanged. The shape is what matters; the function will fill in over the next 39 commits.
- **§17.2's `#if` evaluator runs on tokens, not AST nodes.** Don't conflate it with `eval`/`eval2`/`eval_double`. The preprocessor evaluator is a separate code path; it has to be, because the regular evaluator runs after parsing and the preprocessor runs before parsing.
- **§17.3's no-rescan rule is *not* the same thing as a recursion-depth limit.** The rule is about *which token can be expanded by which macro*, tracked per-token via a hideset. Don't describe it as "stop after N levels" — that would be wrong and would miss the elegance.
- **§17.4's function-like-macro expansion has its own no-rescan rule, which is subtly different from §17.3's.** Walk both carefully. The C standard says they're *technically the same rule applied to different scopes*, but most implementations (and chibicc) treat them as two related-but-separate mechanisms.
- **§17.4's `##` operator's interaction with whitespace is fiddly.** The pasting operator joins tokens at the lexical level, not at the textual level. `a ## b` produces the token `ab`, but `a ## b()` produces the token `ab` followed by `()`, not `a()` followed by `b()`. Walk carefully.
- **§17.5's predefined macros need `Token->file` and `Token->line` to be preserved through expansion.** This is the chapter's most invasive change to the existing `Token` struct. Note when the field is added (probably in §17.1 or §17.5; check the commit).
- **§17.5's `va_arg()` (commit 196) is *not* the same as Ch 14's `__va_area__` / `va_arg(ap, type)`.** Ch 14 added the magic-name plumbing; Ch 17's `va_arg()` makes it a real macro that expands to the magic-name access pattern. The user-side hook that's been "magic" since Ch 14 finally gets a normal `#define` body. Walk this — it's the moment the magic name stops being magic.
- **§17.6's self-hosting is the chapter's punchline. Don't bury it.** The §17.6 prose should name what it means: chibicc compiles itself, end-to-end, with no `self.py`, with chibicc's own preprocessor doing the `#include` and `#define` work. The stage-2 build's Makefile target finally gets the hands-off behavior the §16.1 prose forecast.
- **The macro expansion algorithm has a name** — *Prosser's algorithm* (sometimes called the "blue paint" algorithm in pedagogical writing, after a Stanford technical report). The algorithm is what underlies the C preprocessor as defined by the C standard. Worth naming if the §17.3 prose surfaces the need; default to no, since "the algorithm" is fine and naming it might be over-citing.
- **The cpp-as-separate-binary forecast was wrong.** The handoff (§16.1's forecast) suggested the preprocessor might land as a third binary alongside cc1 and chibicc. Reading the destination state: it doesn't. The preprocessor is a phase of cc1, integrated with tokenization, run before parsing. Note this in the chapter's framing.

## Standing notes worth tracking across sessions

- **The cc1-vs-driver split** is unchanged. Ch 17's preprocessor is a phase of cc1, not a separate binary.
- **The `Initializer` tree** is unchanged. Likely unchanged in Ch 17.
- **The local-vs-global split** is stable.
- **The `eval`/`eval2`/`eval_rval`/`eval_double` quartet** is the AST-level constant-expression evaluator. Ch 17's `#if` evaluator is a *separate* token-level evaluator. The two share node-kind-or-token-kind handling but live in different files (`parse.c` vs `preprocess.c`).
- **The `Relocation` mechanism** — no new uses since the implicit-via-GOT case in Ch 16 §16.2.
- **The anonymous-global pattern** (`new_anon_gvar`) — no new uses since Ch 13. Ch 17 unlikely to use.
- **The `is_static` default in `new_gvar`** — unchanged.
- **The `is_definition` flag on `Obj`** — used by `extern` and (Ch 16 §16.2) the `gen_addr` GOT path.
- **The `is_unsigned` flag on `Type`** — unchanged.
- **The `__va_area__` magic name** — Ch 17 §17.5 commit 196 (`va_arg`) makes this user-side hook accessible via a real macro for the first time. The magic name itself stays magic in chibicc's source, but the user-facing surface becomes a normal `#define`.
- **The register-save-area layout** — unchanged.
- **The argreg 8/16/32/64 split for integers and `%xmm0`–`%xmm7` for floats** — unchanged.
- **The `Member->idx` field** (Ch 12 §12.5) — no new uses since.
- **The `is_flexible` flag** — unchanged.
- **`copy_struct_type`** — no new uses since Ch 12.
- **`MIN`/`MAX` macros** — no new uses since Ch 12.
- **`is_numeric` predicate** (Ch 15 §15.4) — unchanged.
- **Canonicalization-at-parse-time count is at eight.** Ch 16 added zero. Ch 17 unlikely to add (preprocessor work is upstream of parsing).
- **Pre-factor-before-feature count is at seven.** Ch 16 added zero. Ch 17 likely to add: the `preprocess()` skeleton (§17.1) is the pre-factor for everything that follows; the do-nothing first commit makes this explicit. Count to eight after §17.1.
- **The fourth namespace (labels)** is unchanged.
- **The `is_typename` predicate** is unchanged.
- **The VarAttr channel** has four fields. Ch 17 unlikely to grow it.
- **The `ND_NULL_EXPR` seed-pattern** — no new uses since Ch 12.
- **The `rep stosb` pattern** — no new uses since Ch 12.
- **The `unreachable()` macro** — Ch 17 likely to add callers in `preprocess.c`.
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Ch 17 §17.5's `__FILE__`/`__LINE__` make these user-visible. Important to preserve through macro expansion.
- **GDB-debuggable output** (Ch 8 §8.4) — the preprocessor's line-tracking matters for `.loc` correctness. After Ch 17, `.loc` will track the *original source* line, not the expanded line.
- **Tests are in C** as of Ch 8 §8.2. After Ch 17 §17.5 (commit 179, "Use chibicc's preprocessor for all tests"), the test pipeline stops piping through `cc -E` and uses chibicc's own preprocessor.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in §17.5 commit 179. After that, chibicc's preprocessor is the test-pipeline preprocessor.
- **The `Obj->tok` field** added in Ch 14 §14.11 still has no readers. Ch 17 might add — the preprocessor's `#error` directive may use it, or the predefined-macro substitution.
- **The `Type->name_pos` field** (Ch 14 §14.11) — no new uses since.
- **The `>>` codegen quirk** (Ch 11 §11.13) — partially repaired.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) — errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4) — errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels — four errata candidates.
- **`f()` and `f(void)` are accepted as identical** — errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension.
- **`.bss` is the third assembly section.**
- **`.align`** is emitted for every global.
- **The "more than 6 integer args silently miscompiles"** — errata candidate.
- **The "more than 8 FP args silently miscompiles"** (Ch 15 §15.6) — errata candidate, sibling.
- **The `mov $0, %rax`** — closed loop; still pessimistic; plus the variadic-FP-call wrongness (Ch 15 §15.6) — errata candidate.
- **psABI conformance thread:** Ch 13 §13.8/§13.9 + Ch 14 §14.1/§14.2/§14.8 + Ch 15 §15.6/§15.7/§15.8/§15.9 + Ch 16 §16.2 (GOT path) = nine corrections. Ch 17 (preprocessor) probably won't add to this thread.
- **The `fp_offset = fp * 8 + 48` non-conforming stride** (Ch 15 §15.9) — errata candidate.
- **`long double` is `double`** (Ch 15 §15.11) — errata candidate.
- **The default-argument-promotion gap for chars and shorts** (Ch 15 §15.8) — errata candidate.
- **Float literals are inlined as integer-immediate-bit-cast** (Ch 15 §15.1).
- **Ch 1 errata list** unchanged.
- **The cast/compound-literal disambiguator** (Ch 13 §13.4) — same lookahead pattern.
- **The cast table is 10×10.**
- **Driver brittleness** — `find_libpath`/`find_gcc_libpath` Linux/glibc-specific. Errata candidate, lower priority.
- **The link command's hardcoded distro list** (Ch 16 §16.6) — errata candidate, lower priority.
- **`Node->funcname` is gone** (Ch 16 §16.2). Function calls identify the callee by `lhs`.
- **`call *%rax` is uniform across all calls** (Ch 16 §16.2). No fast path for direct named calls.
- **The `StringArray` type** (Ch 16 §16.4). Ch 17 likely to add new users for `#include` search paths.
- **`atexit(cleanup)` for tempfile disposal** (Ch 16 §16.4). Continues unchanged.
- **The `run_subprocess` helper** (Ch 16 §16.3). Continues unchanged. Used by the driver for cc1, as, ld; not used by the preprocessor (which is in-process).

## Acceptance criteria for Ch 17

- [ ] `chapters/17-a-preprocessor-from-scratch.md` exists, end-to-end readable.
- [ ] All forty commits covered, grouped into ~6 sections.
- [ ] §17.1 names the do-nothing-skeleton-first approach as deliberate. Counts as the chapter's pre-factor entry.
- [ ] §17.2 walks the `#if` token-level evaluator and names it as separate from the AST-level `eval` quartet.
- [ ] §17.3 walks the no-rescan rule and the hideset data structure. Likely with concept interlude.
- [ ] §17.4 walks function-like macros, the second no-rescan rule, stringizing, pasting.
- [ ] §17.5 covers all eighteen polish commits; sub-grouping is allowed.
- [ ] §17.6 names self-hosting as the chapter's punchline. Walks the Makefile change. References §16.1's stage-2 forecast.
- [ ] Commit 179 (use chibicc's preprocessor for all tests) is named as the moment the host `cc -E` pipeline retires.
- [ ] Commit 197 (self-host) is named as the moment chibicc compiles itself end-to-end without `self.py`.
- [ ] Voice matches Ch 1–16.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/018-chapter-17-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 019 (Chapter 18 — The full ABI, commits 198–220; ~23 commits; stack-passed args, struct-by-value, va_copy, bitfields, more).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/017-chapter-16-draft/HANDOFF.md  (this handoff)
2. docs/sessions/017-chapter-16-draft/README.md   (what session 017 did)
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
18. chapters/16-the-compiler-driver.md             (most recent chapter)
19. research/commits/chapter-mapping.md            (confirms Ch 17 scope)
20. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 17 (A preprocessor from scratch, commits 158–197) per
the steps in the handoff. Forty commits — the longest single arc in the
book, ending in self-hosting at commit 197. Six sections proposed in the
handoff. Likely needs a concept interlude on the macro-expansion no-
rescan rule. End-of-session: write your session dir under
docs/sessions/018-chapter-17-draft/ with a README and a HANDOFF for
session 019 (Chapter 18 — The full ABI, commits 198–220).
```
