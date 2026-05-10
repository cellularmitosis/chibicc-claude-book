# Session 020 — Chapter 19 draft

**Date:** 2026-05-10 (continuation of the autonomous-drafting run that has produced sessions 002–019).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 019 delivered Ch 18 (The full ABI, twenty-three commits, ~14,357 words). User direction is still autonomous — no chapter-by-chapter review. Ch 19 covers commits 221–244: twenty-four commits, the Unicode arc plus designated initializers plus the trailing date/time/counter macros that Ch 17 deferred.

## What was done

### Drafting decisions

- **Length:** ~12,128 words. At the lower end of the 12,000–15,000-word handoff forecast. The Unicode arc compresses well — most commits are small enough that a per-commit subsection of 100–200 words covers them adequately, and the four string-literal-prefix commits in §19.5 share enough structure that they can be walked in parallel without redundancy. The designated-initializer arc in §19.7 is the chapter's longest single stretch (~2,800 words across the six commits).
- **Section structure:** 7 sections from 24 commits, exactly as the handoff proposed. §19.1 (2 commits, two integrated subsections in prose). §19.2 (1 commit, no subsections). §19.3 (1 commit, no subsections). §19.4 (3 commits, three named subsections). §19.5 (7 commits, seven named subsections — kept each commit explicit even though the four prefix-form commits are structurally similar). §19.6 (4 commits, four named subsections). §19.7 (6 commits, six named subsections, no skipping).
- **No concept interlude.** The handoff defaulted to "possibly one" with the conditional escape on §19.7 if the prose felt overstuffed. Reading the §19.7 prose, the cursor-jump rule and the bail-and-restart pattern in `array_initializer2` / `struct_initializer2` fit inline without straining. The short summary in the §19.7 opening (explaining what the cursor-jump rule does) is enough scaffolding.
- **§19.4 closes the Ch 17 `L''` ≡ `''` errata** — explicitly named in the §19.4 `a57c661` subsection and in the chapter recap.
- **§19.7 surfaces two new errata candidates** — the dead-code duplicate `is_flexible` block in `array_initializer1` (introduced by `835cd24` and never deleted), and the half-implemented range designator `[3 ... 7]` syntax (recognized in `count_array_init_elements` for length computation, not honored in elaboration). Both are named in the §19.7 prose and in the chapter recap.
- **§19.5 walks each of the four prefixes (L, u, U, u8) and their type assignments** — and the two initializer commits (232–233) get explicit treatment as separate subsections. The `__STDC_UTF_*` macros commit (234) gets its own short subsection.
- **§19.6 names UTF-8 BOM stripping as a tokenizer-pre-pass concern** — the chapter recap also names the pre-tokenize pass count moving from 1 (just `remove_backslash_newline`) to 4 (BOM, newline, backslash-newline, UCN).
- **One-table recap** at the chapter close, twenty-four rows. Resisted multi-table-by-section.

### Interpretive calls

1. **The cross-prefix re-tokenization in `2382777` is named as a token-level rewrite within the preprocessor**, not a parse-time canonicalization. The canonicalization-at-parse-time count stays at nine. The mechanism is similar in spirit (rewriting tokens before the parser sees them) but lives in `preprocess.c` rather than `parse.c`, and the prose treats it as a different category.
2. **The pre-factor count stays at nine.** The closest candidate was `read_char_literal`'s `Type *ty` parameter in `454618c`, but that change and its first use are in the same commit. `read_string_literal`'s `quote` parameter in `57b21fe` is similarly same-commit. Neither qualifies as pre-factor-before-feature.
3. **The psABI conformance count stays at sixteen.** Ch 19's commits don't touch the ABI surface.
4. **The Initializer tree is named as gaining one new field (`Member *mem` for the union case).** No other shape changes — the children array is still position-indexed, designators just jump the cursor without restructuring the tree.
5. **The `init->expr = NULL` line in `designation`'s struct arm (commit `67f5834`) is given its own short walk.** It clears prior scalar bindings on re-designation. The test case `T y[]={x, [0].b=3}` exercises the line — without it, both the struct-copy and the b-field designation would emit stores, and the order would determine the final value.
6. **The anonymous-struct designator's `*rest = start` trick (commit `95eb5b0`) gets one paragraph.** The trick lets the same `.field` token be re-parsed at each anonymous layer; the prose walks the test case `{1,2,3,.b=4,5}` against a struct-of-anonymous-struct-of-anonymous-struct to show how `b=4` lands and `5` continues to `c`.
7. **Ch 17 errata closures: two now closed, three remaining.** `L''` ≡ `''` closed by `a57c661`. `__va_arg_mem` divides by zero closed by Ch 18. The remaining three: `#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific.
8. **The UTF-16 character literal silent truncation is named as errata.** Commit `454618c`'s `cur->val &= 0xffff` truncates code points above U+FFFF to their low 16 bits. The C standard says this should be a constraint violation; chibicc is permissive. Test case in §19.4 exercises it (`U+1F363` becomes `0xF363`).
9. **The §19.5 prose handles `read_escaped_char` consistently across the prefixes.** The note that `u"\xff"` produces a single 16-bit unit (`0x00FF`) rather than the UTF-16 encoding of U+00FF is named in the §19.5 UTF-16 subsection. gcc and clang behave the same way, so it's not a chibicc-specific quirk.
10. **The §19.6 cross-prefix concatenation walks the `("\343\201\202" L"")` test case.** The test case shows that an octal-escape ordinary string and a UTF-8 source string are byte-identical at parse but *differ* under cross-prefix promotion (escape sequences are kept as bytes; UTF-8 source is decoded to code points). This is the most subtle distinction in the chapter; the prose names it explicitly.

### Voice / structure inherited from Ch 1–18

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, all hashes listed at the top.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- One-table recap at the chapter close.
- No concept interludes.

### Three careful avoidances

- **Did not invent a "history of UTF-8" interlude.** UTF-8 has a famous design history (Pike/Thompson, 1992); walking it would have been a detour. The chapter cites the encoding rule and walks chibicc's encode/decode helpers; it doesn't try to reconstruct the design history.
- **Did not over-explain the surrogate-pair encoding.** UTF-16's surrogate pairs are a curious historical artifact; the prose walks the encoding step-by-step for one test case (`🍣`, U+1F363 → 0xD83C, 0xDF63) and leaves the abstract rule to the encoded comment. Walking the abstract rule plus several test cases would have inflated §19.5 without adding clarity.
- **Did not invent a "C99 designated initializer history" detour.** Designated initializers have a long history in C compilers (gcc had them as an extension for a decade before C99); walking that history would be a detour. The chapter sticks to chibicc's specific implementation choices.

### Date-vs-position note

The twenty-four commits scatter wildly across calendar time: April 2020 (no commits in Ch 19), May 2020 (`a57c661` wide-char, `0e5d250` UTF-8 in identifier, `e27417f` `__DATE__`), June 2020 (`0e77f3d` `__COUNTER__`, `74bcec5` newline canonicalization), July 2020 (`adb8b98` `$`, `cae061a` wide string, `36230e0` UTF-16 init, `6adba75` UTF-32 init, `e4491b8` `__STDC_UTF_*`, `691c4fa` `=`-omission, `95eb5b0` anonymous-struct designator), August 2020 (`c31886a` UCN, `31dc1df` union designator), September 2020 (`454618c` UTF-16 char, `2dac3af` UTF-32 char, `57b21fe` UTF-8 string, `9cabe1f` UTF-16 string, `c467ee6` UTF-32 string, `c618c3b` array designator, `835cd24` incomplete-array sizing), October 2020 (`67f5834` struct designator, `2382777` cross-prefix concat, `2b2fa25` BOM). The chapter follows `main` order without remark — `a57c661` (May 6, 2020) lands at position 225 even though several later-dated commits appear before it on `main`. This is consistent with prior chapters' policy.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The Unicode baseline:** chibicc's source-file convention is now formally UTF-8. `decode_utf8` and `encode_utf8` are the only two byte-level Unicode helpers needed. `unicode.c` is the new file; `tokenize.c` calls into it; `chibicc.h` declares its public surface.
- **The pre-tokenize pass count** is now 4: BOM, newline, backslash-newline, UCN. All four are destructive in-place rewrites of the source buffer. Order matters: BOM first, newline second, backslash-newline third, UCN fourth.
- **The four character-literal prefixes** are functional with type assignments matching gcc on Linux: `'X'` is `int` then narrowed to `char`, `L'X'` is `int`, `u'X'` is `unsigned short` truncated to 16 bits, `U'X'` is `unsigned int`.
- **The four string-literal prefixes** are functional with these element types: ordinary and `u8` are `char`, `u` is `unsigned short`, `U` is `unsigned int`, `L` is `int`. The `Initializer` tree's `string_initializer` dispatches on `init->ty->base->size`, with cases for 1, 2, and 4.
- **`__STDC_UTF_16__` and `__STDC_UTF_32__`** are defined to 1.
- **The cross-prefix concatenation handler** does two passes: detect-and-promote, then concatenate. The promotion rewrites ordinary literals to match a wide neighbor's encoding using `tokenize_string_literal` (the new public entry point in `tokenize.c`). The `StringKind` enum and `getStringKind` helper classify literals by prefix without re-parsing.
- **UTF-8 in identifiers** uses the C11 Annex D ranges. The range tables are in `unicode.c` as static `uint32_t[]` arrays terminated by `-1`. `is_ident1` and `is_ident2` move from `tokenize.c` (where they were `char`-based) to `unicode.c` (where they are `uint32_t`-based). The new `read_ident` in `tokenize.c` is the canonical identifier recognizer; the do-while-on-bytes loop is gone.
- **The GNU `$` extension** adds two range-table entries to `is_ident1` and `is_ident2`. Tests live in `test/unicode.c`.
- **The UTF-8 BOM skip** is a three-byte `EF BB BF` test at the start of `tokenize_file`, before all other pre-tokenize passes.
- **The designator parsing surface** is three new helpers (`array_designator`, `struct_designator`, `designation`) plus the cursor-jump-aware variants of three existing helpers (`array_initializer2`, `struct_initializer2`, `count_array_init_elements`).
- **The `Initializer` struct gains `Member *mem`** for the union case. Set by `union_initializer` or `designation`'s union arm. Read by `create_lvar_init` and `write_gvar_data`'s union branches. The fallback `init->mem ? init->mem : ty->members` in `create_lvar_init` handles implicit-first-member cases (`union x = {}`).
- **The cursor-bail pattern** in `array_initializer2` and `struct_initializer2`: when the loop sees a designator (`[` or `.`), it restores `*rest` to the bail position and returns. The bail propagates up to whichever outer initializer handler (`array_initializer1`, `struct_initializer1`, or `designation`'s nested call) parses the designator.
- **The `init->expr = NULL` line in `designation`'s struct arm** clears prior scalar bindings on re-designation. Necessary for cases where a scalar struct value is followed by a designation that overrides it.
- **The anonymous-struct designator integration** reuses `get_struct_member` from Chapter 18 §18.6. The `*rest = start` trick lets the same `.field` token be re-parsed at each anonymous layer.
- **Two new errata candidates from §19.7:**
  - The dead-code duplicate `if (init->is_flexible)` block in `array_initializer1` (introduced by `835cd24`).
  - Range designators `[3 ... 7]` syntactically accepted in `count_array_init_elements` but not honored in elaboration.
- **One new errata candidate from §19.4:** UTF-16 character-literal silent truncation of code points above U+FFFF.
- **One Ch 17 errata closed:** `L''` ≡ `''` (closed by `a57c661`).
- **Pre-factor-before-feature count** stays at nine.
- **Canonicalization-at-parse-time count** stays at nine.
- **psABI conformance count** stays at sixteen.
- **`unreachable()` callers** gain two new sites: `string_initializer`'s switch default, `getStringKind`'s switch default.
- **`MAX` macro** picks up a new use in `count_array_init_elements`.
- **The `is_unsigned` flag on `Type`** has new readers via the UTF-16 (`ty_ushort`) and UTF-32 (`ty_uint`) string-literal element types.
- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean — unchanged.
- **Chibicc compiles itself** — unchanged.

## Exit state

- `chapters/19-unicode-and-designated-initializers.md` drafted, ~12,128 words.
- Session 020 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 021 (Chapter 20 — GCC extensions worth supporting, commits 245–266, ~22 commits).
- CLAUDE.md status note will be updated to "Ch 19 drafted".
