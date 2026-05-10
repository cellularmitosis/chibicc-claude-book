# Handoff: Ch 18 done → proceed to Ch 19

**For:** the next claude session.
**From:** session 019.
**Status:** Ch 18 drafted (~14,357 words, twenty-three commits, the full SysV AMD64 calling convention plus bitfields plus a handful of polish commits). Continue autonomously to Ch 19 (Unicode and designated initializers, commits 221–244 — twenty-four commits covering `__DATE__`/`__TIME__`/`__COUNTER__`, newline canonicalization, `\u`/`\U`, multibyte and UTF-{16,32} character and string literals, identifier UTF-8 support, GNU `$` in identifiers, cross-prefix string concatenation, UTF-8 BOM, array and struct and union designated initializers, and the GNU "omit `=`" extension). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/019-chapter-18-draft/README.md`](README.md)** — what session 019 did, including the six-section structure (per-commit subsections in §18.2, §18.5, §18.6 and integrated subsections-within-prose in §18.1, §18.3; per-commit subsections in §18.4), no concept interludes (the SysV ABI eightbyte classification was tempting but wasn't broken out), the closure of two errata items in §18.1 ("more than 6 integer args silently miscompiles" and "more than 8 FP args silently miscompiles") plus one Ch 17 errata (`__va_arg_mem` divide-by-zero), the *non*-closure of the `fp_offset = fp * 8 + 48` non-conforming stride, the canonicalization-at-parse-time count incremented to nine, the pre-factor count incremented to nine, the psABI conformance count up to sixteen.
2. **[`docs/sessions/018-chapter-17-draft/HANDOFF.md`](../018-chapter-17-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply with Ch 18 updates folded in (see §19 README for the running list).
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`18-the-full-abi.md`](../../../chapters/18-the-full-abi.md)** — the eighteen chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 19 = commits 221–244 (24 commits, scoped to "Unicode and designated initializers").
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes. Ch 19's commits are mostly feature additions; less commit-message material than the early chapters but worth scanning for design notes on Unicode / multibyte handling.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; designated initializers are a candidate for a concept interlude (the parser-side "skip-or-replace existing initializer entries" rule is intricate enough to repay one).

## Chapter 19 scope

**Title (working):** *Unicode and designated initializers*.
**Commits:** 221–244 in chronological order on `main`. **Twenty-four commits** — back to the size of an ordinary chapter after Ch 18's twenty-three.
**Concept interlude:** Possibly one. Designated initializers introduce a non-trivial parser-side rule for sparse and out-of-order initialization (`int x[10] = {[3] = 1, [7] = 2}`); the `Initializer` tree gains the `.member` and `[index]` parsing with a recovery rule for sparse positions and a precedence rule for nested designators (`x[3].a`, `x.foo[2]`). A short interlude on *how a sparse initializer commits to the AST* could sit at the start of §19.6 (the designated-initializer subsection). Default conditional — judge while reading the commits.

| # | Hash | Subject |
|---|---|---|
| 221 | `e27417f` | Add __DATE__ and __TIME__ macros |
| 222 | `0e77f3d` | [GNU] Add __COUNTER__ macro |
| 223 | `74bcec5` | Canonicalize newline character |
| 224 | `c31886a` | Add \u and \U escape sequences |
| 225 | `a57c661` | Accept multibyte character as wide character literal |
| 226 | `454618c` | Add UTF-16 character literal |
| 227 | `2dac3af` | Add UTF-32 character literal |
| 228 | `57b21fe` | Add UTF-8 string literal |
| 229 | `9cabe1f` | Add UTF-16 string literal |
| 230 | `c467ee6` | Add UTF-32 string literal |
| 231 | `cae061a` | Add wide string literal |
| 232 | `36230e0` | Add UTF-16 string literal initializer |
| 233 | `6adba75` | Add UTF-32 string literal initializer |
| 234 | `e4491b8` | Define __STDC_UTF_{16,32}__ macros |
| 235 | `0e5d250` | Allow multibyte UTF-8 character in identifier |
| 236 | `adb8b98` | [GNU] Accept $ as an identifier character |
| 237 | `2382777` | Allow to concatenate regular string literals with L/u/U string literals |
| 238 | `2b2fa25` | Skip UTF-8 BOM markers |
| 239 | `c618c3b` | Add array designated initializer |
| 240 | `835cd24` | Allow array designators to initialize incomplete arrays |
| 241 | `691c4fa` | [GNU] Allow to omit "=" in designated initializers |
| 242 | `67f5834` | Add struct designated initializer |
| 243 | `31dc1df` | Add union designated initializer |
| 244 | `95eb5b0` | Handle struct designator for anonymous struct member |

Twenty-four commits. The natural section grouping:

- **§19.1 — Date/time/counter macros** (commits 221–222). Two commits. `__DATE__` (and `__TIME__` in the same commit) format the cc1 startup time as ``"Mmm DD YYYY"`` / ``"HH:MM:SS"``. `__COUNTER__` is a stateful macro that yields 0, 1, 2, ... on each expansion. All three use the `Macro->handler` field added in Ch 17 §17.5.3.
- **§19.2 — Newline canonicalization** (commit 223). One commit. The tokenizer converts `\r\n` and `\r` to `\n` at the start, before any other tokenizing. This makes Windows-style line endings transparent.
- **§19.3 — Universal character names: `\u` and `\U`** (commit 224). One commit. UCNs in character and string literals. `\uXXXX` and `\UXXXXXXXX` produce the corresponding Unicode code point; the encoding (UTF-8, UTF-16, UTF-32) depends on the literal's prefix, which §19.4 starts handling.
- **§19.4 — Multibyte and wide character literals** (commits 225–227). Three commits. `L'<multibyte>'` decodes a UTF-8 source sequence to a `wchar_t` (32-bit). `u'X'` and `U'X'` are UTF-16 and UTF-32 character literals.
- **§19.5 — UTF-8/16/32 and wide string literals plus initializers** (commits 228–234). Seven commits. The string-literal-prefix family. `u8"..."` for UTF-8, `u"..."` for UTF-16, `U"..."` for UTF-32, `L"..."` for wide. The initializer side handles `wchar_t s[] = L"..."`, `char16_t s[] = u"..."`, `char32_t s[] = U"..."`. The `__STDC_UTF_16__` and `__STDC_UTF_32__` predefined macros announce that the implementation uses UTF-16 and UTF-32.
- **§19.6 — Identifier-side Unicode and BOM** (commits 235–238). Four commits. UTF-8 multibyte characters are accepted as identifier characters (so an identifier like `日本語` is legal). GNU's `$` in identifiers is allowed. Cross-prefix string-literal concatenation handles `"foo" L"bar"` (becomes a wide string). UTF-8 BOM markers at the start of a file are skipped.
- **§19.7 — Designated initializers** (commits 239–244). Six commits. Array form (`{[3] = 1}`), the "incomplete array gets size from the largest designator" rule, the GNU `=`-omission extension (`{[3] 1}`), struct form (`{.x = 1}`), union form, and the anonymous-struct designator interaction. This is the chapter's longest single arc and is where the concept interlude (if used) lands.

That's seven sections from twenty-four commits. **Target chapter length: ~12,000–15,000 words.** Likely closer to 13K — Unicode handling is detailed but each commit is small; designated initializers are intricate but compress well into per-commit subsections.

## Steps

1. `cd research/sources/chibicc && for h in e27417f 0e77f3d 74bcec5 c31886a a57c661 454618c 2dac3af 57b21fe 9cabe1f c467ee6 cae061a 36230e0 6adba75 e4491b8 0e5d250 adb8b98 2382777 2b2fa25 c618c3b 835cd24 691c4fa 67f5834 31dc1df 95eb5b0; do echo "===== $h ====="; git show --stat $h | head -10; done` to scan all 24 diffs.
2. Read each commit. Pay particular attention to:
   - **§19.1's `__DATE__`/`__TIME__`/`__COUNTER__`** — the cc1 startup-time-formatting code is small but worth walking. Check whether `__DATE__` uses `localtime` or `gmtime` (tracking the locale-vs-UTC question for "what date does this build show?").
   - **§19.3's `\u` and `\U`** — UCNs must be decoded to UTF-8 in source-file representation. Check the encoding helper functions (probably `encode_utf8` or similar).
   - **§19.4's multibyte → wchar_t conversion** — UTF-8 decoding is non-trivial. Check the multibyte-decoding helper.
   - **§19.5's string-literal-prefix family** — these are spread over seven commits; the commits are mostly parallel (`u8` parallels `u` parallels `U` parallels `L` minus one bit of encoding logic). Don't blur them — each gets its own short subsection. The initializer commits (232–233) are interesting because they show how `wchar_t s[] = L"abc"` is parsed.
   - **§19.6's identifier UTF-8** — check how the tokenizer's identifier-recognizer handles multibyte characters. Probably a `is_ident1`/`is_ident2` predicate gets a UTF-8 path.
   - **§19.6's UTF-8 BOM skip** — three-byte sequence at the very start of the file (`EF BB BF`). Probably stripped in `tokenize_file` or at the start of `tokenize`.
   - **§19.7's designated initializers** — the most code-heavy section. Read each commit carefully. The array case introduces a rule for "if `[3]` is designated, fill in indices 0-2 with zero values"; the incomplete-array case extends that to "the array's size is `max designator + 1`"; the struct case threads the same logic through struct-member lookup; the anonymous-struct designator case (commit 244) reuses §18.6's anonymous-struct lookup machinery.
   - **§19.7's `=`-omission GNU extension** — `{[3] 1}` instead of `{[3] = 1}`. Probably a one-line parser-side change.
3. Read the destination state at `95eb5b0` for `parse.c`, `tokenize.c`, `preprocess.c`, `chibicc.h`. The designated-initializer codepath in `parse.c` will be the most invasive change.
4. Draft `chapters/19-unicode-and-designated-initializers.md`. Likely 12,000–15,000 words. Seven sections.
5. Write `docs/sessions/020-chapter-19-draft/README.md`.
6. Write `HANDOFF.md` for session 021 (Chapter 20 — GCC extensions worth supporting, commits 245–266).

## Voice / structure rules

Same as Ch 1–18:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, list the checkouts at the section opener.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present tense for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — twenty-four rows, single table is fine.
- Diff format: lean toward inline diff fragments and quoted file snippets. The §19.7 designated-initializer codegen will want larger code blocks; the Unicode encoders too.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **§19.1's three macros are stateless except for `__COUNTER__`.** Don't over-tell the stateful-vs-stateless story; just note that `__COUNTER__` increments per expansion via `Macro->handler` while `__DATE__`/`__TIME__` produce a fixed value computed once at cc1 startup.
- **§19.3's UCN handling is a tokenizer-level concern.** The decoded UTF-8 lands in the `Token`'s `str` field. Don't conflate with `\u` *escape sequences* in identifier contexts (which are a different feature, not implemented).
- **§19.4's `wchar_t` is 32 bits.** Chibicc uses Linux convention. The L'X' literal's value is a 32-bit Unicode code point.
- **§19.5's L/u/U/u8 prefixes are parallel but not identical.** UTF-8 string literals (`u8"..."`) have type `char[]`; UTF-16 (`u"..."`) has type `unsigned short[]` (with `__STDC_UTF_16__` defined); UTF-32 (`U"..."`) has type `unsigned int[]` (with `__STDC_UTF_32__` defined); wide (`L"..."`) has type `int[]` (the `wchar_t` typedef). The exact types matter for the array-of-what?-rule and for sizeof.
- **§19.6's UTF-8 identifier rule.** The C11 standard allows specific Unicode ranges in identifiers (Annex D); chibicc's implementation is *more permissive* — it accepts any non-ASCII UTF-8-valid byte sequence as identifier characters. Note this as a small chibicc-specific simplification.
- **§19.7's designated-initializer parsing is the chapter's most invasive change.** The `Initializer` tree's child indexing changes — instead of "initializer N goes to child N," initializers can now jump to arbitrary positions via `[N]` or `.member`. The position tracker in `array_initializer` and `struct_initializer` advances normally for un-designated entries but jumps for designated ones; subsequent un-designated entries continue from the jumped-to position.
- **The anonymous-struct designator (commit 244)** reuses §18.6's anonymous-struct member lookup. `{.a = 1}` works on a struct with an anonymous inner struct containing `a` because `get_struct_member` recursively descends; the designator has to thread that descent through the initializer tree.
- **The "incomplete array gets size from designators" rule (commit 240)** changes the `is_flexible` machinery slightly. Pre-commit, an incomplete array's size was determined by the count of consecutive non-designated initializers. Post-commit, the size is `max(designator_index, count)` where designator indexes are gathered as the initializer is parsed.

## Standing notes worth tracking across sessions

- **The hideset on Token** is unchanged. Ch 19 doesn't touch the preprocessor's expansion algorithm.
- **The Token->origin chain** is unchanged.
- **The eval-quartet duplication** is unchanged.
- **The cc1-vs-driver split** is unchanged.
- **The `Initializer` tree** changes in §19.7. New positional/designator-aware fields likely added.
- **The local-vs-global split** is stable.
- **The `Relocation` mechanism** likely gains string-literal-related new uses for UTF-16/UTF-32 string literals.
- **The anonymous-global pattern** likely picks up new uses for UTF-16/UTF-32 string literals (each is a global-with-no-name carrying `unsigned short[]` or `unsigned int[]` data).
- **The `is_static` default in `new_gvar`** — unchanged.
- **The `is_definition` flag on `Obj`** — unchanged.
- **The `is_unsigned` flag on `Type`** — used for UTF-16/UTF-32 string literal types (which are `unsigned short[]` and `unsigned int[]`).
- **The `__va_area__` magic name** — unchanged.
- **The register-save-area layout** — unchanged.
- **The argreg integer/FP split** — unchanged.
- **The `Member->idx` field** — unchanged. The bitfield-related siblings (`is_bitfield`/`bit_offset`/`bit_width`) — unchanged.
- **The `is_flexible` flag** — likely changes in §19.7.
- **`copy_struct_type`** — unchanged.
- **`MIN`/`MAX` macros** — likely picks up a new use in §19.7's incomplete-array sizing.
- **`is_numeric` predicate** — unchanged.
- **Canonicalization-at-parse-time count is at nine.** Ch 19 might add one in §19.7 (designated-initializer-to-positional-initializer rewrite) — verify while drafting.
- **Pre-factor-before-feature count is at nine.** Ch 19 unlikely to add new entries — most §19's commits are direct feature adds.
- **psABI conformance count is at sixteen.** Ch 19 unlikely to touch it.
- **The fourth namespace (labels)** is unchanged.
- **The `is_typename` predicate** is unchanged.
- **The VarAttr channel** has four fields. Ch 19 unlikely to grow it.
- **The `ND_NULL_EXPR` seed-pattern** — likely picks up new uses in §19.7's designated-initializer "fill-in-zero-for-skipped-positions" rule.
- **The `rep stosb` pattern** — no new uses since Ch 12; unlikely to in Ch 19.
- **The `unreachable()` macro** — Ch 19 likely to add callers in the new Unicode-encoding helpers.
- **Per-token line numbers** (Ch 8 §8.3) — preserved through preprocessing as of Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — unchanged.
- **Tests are in C** (Ch 8 §8.2). Driver tests in shell. New test file likely for Unicode tests; designated initializers go into existing `test/initializer.c`.
- **The `Obj->tok` field** added in Ch 14 §14.11 still has no readers.
- **The `Type->name_pos` field** (Ch 14 §14.11) — no new uses since.
- **The `>>` codegen quirk** (Ch 11 §11.13) — partially repaired.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) — errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4) — errata candidate.
- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels, struct-members — five errata candidates.
- **`f()` and `f(void)` are accepted as identical** — errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension.
- **`.bss` is the third assembly section.**
- **`.align`** is emitted for every global. Ch 18's 16-byte rule applies to arrays >= 16 bytes; struct alignment unchanged.
- **The `mov $0, %rax`** for variadic FP-count is still pessimistic; the variadic-FP-call wrongness is *closed* by Ch 18's `b6d3cd0` (the fall-through-to-memory path is correct now). The pessimistic-zero-when-non-variadic remains.
- **The `fp_offset = fp * 8 + 48` non-conforming stride** — errata candidate, *not* closed by Ch 18.
- **`long double` is `double`** — errata candidate.
- **The default-argument-promotion gap for chars and shorts** — errata candidate.
- **Float literals are inlined as integer-immediate-bit-cast.**
- **Ch 1 errata list** unchanged.
- **The cast/compound-literal disambiguator** (Ch 13 §13.4) — same lookahead pattern.
- **The cast table is 10×10.** Possibly grows in §19.5 if UTF-16/UTF-32 string literals introduce new cast cells; verify while drafting.
- **Driver brittleness** — unchanged.
- **The link command's hardcoded distro list** (Ch 16 §16.6) — errata candidate, lower priority.
- **`Node->funcname` is gone** (Ch 16 §16.2).
- **`call *%r10` is uniform across all calls** (Ch 18 §18.1, replacing the earlier `call *%rax`). No fast path for direct named calls.
- **The `StringArray` type** (Ch 16 §16.4) — used by `include_paths` (Ch 17 §17.5.2). Ch 19 unlikely to add new users.
- **`atexit(cleanup)` for tempfile disposal** — unchanged.
- **The `run_subprocess` helper** — unchanged.
- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `L''` ≡ `''` (this *was* the wide-character-literal punt; closed by Ch 19's `a57c661`); `__va_arg_mem` divides by zero (closed by Ch 18); `opt_S | opt_E` typo; default include paths Linux/glibc-specific.
- **Errata candidates added in Ch 18:** None high-priority. The bitfield zero-width test exposes the missing struct-member-name-redeclaration check (one of the five redeclaration-check errata).
- **`self.py` is gone** (Ch 17 §17.6).
- **Stage-2 build** is end-to-end chibicc, plus `-Wall`-clean as of Ch 18 §18.6.
- **Chibicc compiles itself** as of commit 197 (Ch 17 §17.6); plus `-Wall`-clean as of Ch 18 §18.6.
- **The `has_flonum` family** (Ch 18 §18.2) is the SysV ABI eightbyte classifier. Used in five places. The asymmetry between caller-side `has_flonum2` (offset 0) and callee-side direct call (offset 8) is a quiet inconsistency, not a bug.
- **Bitfield support is feature-complete.** `Member` has `is_bitfield`/`bit_offset`/`bit_width`; `struct_decl` lays them out per the SysV-ABI-derived rule; codegen reads via shift-extract and writes via read-modify-write; op-assign goes through a parser-side rewrite using two ND_MEMBER nodes; address-of is rejected; zero-width is handled.
- **Anonymous struct/union members** (Ch 18 §18.6) flatten via recursive `get_struct_member` and a `struct_ref` outer loop. Designator handling for these (Ch 19 §19.7's commit 244) builds on this machinery.

## Acceptance criteria for Ch 19

- [ ] `chapters/19-unicode-and-designated-initializers.md` exists, end-to-end readable.
- [ ] All twenty-four commits covered, grouped into ~7 sections.
- [ ] §19.1 names `__COUNTER__`'s use of `Macro->handler` (the dynamic-builtin-macro hook from Ch 17 §17.5.3).
- [ ] §19.4 walks UTF-8-source-byte-to-wchar_t decoding.
- [ ] §19.5 walks each of the four prefixes (`L`, `u`, `U`, `u8`) and their type assignments. The two initializer commits (232–233) get explicit treatment.
- [ ] §19.6 names UTF-8 BOM stripping as a tokenizer-pre-pass concern (parallel to newline canonicalization in §19.2).
- [ ] §19.7 walks each of the six designated-initializer commits with no skipping. The array form, incomplete-array size rule, GNU `=`-omission, struct form, union form, anonymous-struct designator each get their own subsection.
- [ ] §19.7 closes the `L''` ≡ `''` errata from Ch 17 §17.5.3 (closed by `a57c661`, which is in §19.4 not §19.7 — so this acceptance item lands in §19.4).
- [ ] Voice matches Ch 1–18.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] psABI conformance thread count noted as still at sixteen unless Ch 19 commits add to it (they probably don't).
- [ ] `docs/sessions/020-chapter-19-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 021 (Chapter 20 — GCC extensions worth supporting, commits 245–266).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/019-chapter-18-draft/HANDOFF.md  (this handoff)
2. docs/sessions/019-chapter-18-draft/README.md   (what session 019 did)
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
20. chapters/18-the-full-abi.md                    (most recent chapter)
21. research/commits/chapter-mapping.md            (confirms Ch 19 scope)
22. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 19 (Unicode and designated initializers, commits
221–244) per the steps in the handoff. Twenty-four commits, seven
sections proposed in the handoff. The designated-initializer arc
(§19.7, six commits) is the chapter's longest single stretch and is
where a possible concept interlude lands; the Unicode arc (§19.3
through §19.6, twelve commits) is the deepest. End-of-session: write
your session dir under docs/sessions/020-chapter-19-draft/ with a
README and a HANDOFF for session 021 (Chapter 20 — GCC extensions
worth supporting, commits 245–266).
```
