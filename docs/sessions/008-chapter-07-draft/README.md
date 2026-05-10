# Session 008 — Chapter 7 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that produced sessions 002–007).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 007 just delivered Ch 6. Same conversation, healthy context budget. User confirmed continued autonomous progress. Ch 7 covers commits 32–43 (twelve commits) — by far the longest commit range so far. The chapter spans global variables, the `char` type, string literals plus three escape-sequence commits, statement expressions (a GNU extension), the move from `argv[1]` to file/stdin input, a `printf`-to-`println` refactor, the `-o`/`--help` flags, and source comments.

## What was done

### Drafting decisions

- **Length:** ~12,500 words. Larger than every prior chapter (Ch 6 was ~9,000, Ch 5 ~8,650). Twelve commits is twice the previous high, and four of them are substantive feature commits in their own right (globals, char, string literals, statement expressions). The driver-maturation trio (`d9ea597`, `7b8528f`, `a0388ba`) bundled into one §7.6 keeps the section count manageable. Comments got their own short §7.7 because they're a separate concern from the driver and fit better as an outro.
- **Section structure:** seven sections plus the integer-representation interlude. The handoff suggested possibly bundling 40/41/42 + 43 into one driver-maturation section; chose to keep comments separate because they're a tokenizer/language concern, not a driver concern, and the natural narrative arc lands with comments as the chapter's quiet closer.
- **Concept interlude on integer representation placed between §7.1 (globals) and §7.2 (char),** per the HANDOFF plan. Came in at ~1,400 words — slightly shorter than Ch 5's calling-convention interlude (~2,000) and Ch 6's array-decay one (~1,200). Calibrated for the audience: explicit statement that the book's readers know two's complement, then focused on the *codegen-relevant* facts — `movsx` family, the asymmetry between widening loads and narrow stores, chibicc's specific choice of 8-byte int / 1-byte char with no unsigned. Forward-pointed to Ch 10 for the integer-rules expansion and Ch 14 for unsigned.
- **Anonymous globals framed carefully in §7.3.** The HANDOFF flagged this as conceptually significant and the prose treats it as the moment chibicc first emits something with a synthesized name. The three-layer ladder (`new_unique_name` → `new_anon_gvar` → `new_string_literal`) is presented as forward-compatible scaffolding for Ch 12 (compound literals) and Ch 17 (preprocessor `__COUNTER__`), which is a real forward-reference both grounded in `chapter-mapping.md`.
- **Statement expressions framed as the first GNU extension.** Per HANDOFF acceptance criterion. The §7.5 prose explicitly calls out that GCC and chibicc accept the form but other C compilers reject it, and forward-points to Ch 20 for the larger collection of GNU extensions chibicc eventually picks up (typeof, asm, etc.).
- **Canonicalization-at-parse-time picked up its fifth named instance** (statement expressions in §7.5). The framing is slightly different from the four Ch 6 instances — it's *delegation* rather than *desugaring* (the parser borrows `compound_stmt`'s logic but wraps in a different node kind, rather than rewriting one form to another) — and the prose flags this as a sub-variant of the pattern. The recap calls it "a slightly different mode of canonicalization."
- **Pre-factor-before-feature got its second clear named instance** (the `printf`-to-`println` refactor in `7b8528f` immediately preceding `-o`/`--help` in `a0388ba`). The §7.6 prose explicitly back-references the Ch 6 §6.5 naming and notes that two examples are enough for the pattern to feel earned.
- **The "Trusting Trust" comment in `read_escaped_char` got full attention in §7.4.** Rui's comment quotes "Reflections on Trusting Trust" and chibicc has nothing else like this in its source. The chapter quotes the comment in full and uses it as an opportunity to talk about the tautological style of escape-sequence handling and what self-hosting (Ch 17) means for the same code. This is one of the chapter's interpretive moments — Rui doesn't actually say "this is a security issue we're not fixing," but the comment's placement in the source invites the framing.
- **The lookahead pattern in `is_function` (§7.1) got the comment-block-from-Chapter-2 payoff.** The `parse.c` opening comment about "very easy to lookahead arbitrary number of tokens" was forward-looking when introduced; this is the first commit that actually uses the ability, and the chapter quotes the original comment to make the connection explicit.
- **The "no Rui-quote citations" call from Ch 4–6 was reversed for `read_escaped_char` only.** Rui's source comment in that function is itself a quotable Rui statement on the security implications of self-hosting compilers, and it's apt for this chapter's introduction of the bootstrap-relevant escape-sequence logic. `quotes-rui.md` doesn't contain it (it's a source comment, not external commentary), but the chapter quotes it directly from the source. Worth flagging for the next session — if the source has more quotable comments like this, lean on them.
- **Commit reordering acknowledged in the chapter intro.** Several Ch 7 commits are dated months apart (the comments commit is 2019, the global-variables commit is 2020) — the chapter's order follows the canonical-history order, not the wall-clock dates. The intro explicitly addresses this so a reader who runs `git log --date=short` and notices the dates skipping around isn't confused.
- **Diff format** consistent with prior chapters: `diff` blocks for line-level changes, full quoted code for new functions. The §7.6 driver-maturation section quotes `read_file`, `parse_args`, and `verror_at` in full because each is a new self-contained piece worth showing complete.
- **Forward references** kept short and grounded. Ch 8 mentioned for: block scope, the precomputed line-number array, and the test-rewrite-in-C use of statement expressions and file I/O. Ch 10 for full integer-system expansion. Ch 12 for compound-literal use of `new_anon_gvar`. Ch 13 for `static`-vs-`.globl` distinction. Ch 14 for unsigned types. Ch 16 for multi-file compilation building on `read_file`. Ch 17 for preprocessor `__COUNTER__` use of `new_unique_name` and self-hosting making the Trusting-Trust point concrete. Ch 20 for more GNU extensions. Ch 22 for hashmap-based symbol lookup. All cross-checked against `chapter-mapping.md`.

### Three small interpretive calls

1. **The `is_function` lookahead is framed as the first time chibicc cashes in the parser-design comment from Chapter 2.** The connection is real (the comment was forward-looking; this is the first use), but spelling it out is a stylistic choice the prose makes to give the §7.1 commit a hook beyond "the parser learns to dispatch."
2. **The Trusting-Trust comment gets full-quote treatment and a paragraph of interpretation about what self-hosting in Ch 17 will mean.** This is a load-bearing interpretive moment — the chapter takes a position on what Rui's comment is doing in the source, framing it as a flag for the reader rather than a fix-this-please TODO. The position seems clearly right but it is a position.
3. **The `printf`-to-`println` refactor is framed as the second instance of the pre-factor pattern.** This is partly an editorial choice — the commit message ("Refactor -- no functionality change") doesn't *say* it's a pre-factor for the next commit, but the structure makes it obvious in retrospect. Naming the pattern is a service to the reader.

### Two careful avoidances

- **The hex-escape overflow (`\x00ff` truncating to one byte without a warning) was acknowledged but not "fixed" in the prose.** The §7.4 walkthrough notes that real compilers warn and chibicc doesn't, but doesn't dwell on it as a defect — it's an errata-appendix candidate and not the chapter's job to belabor. This is consistent with the Ch 1 errata-list standing notes.
- **The lack of integer promotion in `char + char` was mentioned in the integer-representation interlude but not exercised in the prose for §7.2.** The interlude says "chibicc doesn't model integer promotion until Ch 10" and the §7.2 prose lets that statement carry the weight rather than re-explaining it inline. Avoids redundant explanation.

### Voice / structure inherited from Ch 1–6

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Section opens with `git checkout <full-hash>`. Bundled sections (§7.3, §7.4, §7.6) include a separate `git checkout` line for each commit at the natural sub-section breakpoint.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with a feature table — twelve rows, one per commit, organized in commit order. The handoff suggested considering grouping by section but per-commit was clearer.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **Canonicalization-at-parse-time is now a five-instance pattern with one sub-variant.** The four Ch 6 instances are all *desugaring* (rewriting surface form A as surface form B). The Ch 7 statement-expression instance is *delegation* (parsing surface form A using parser logic for form B, then wrapping in a different node kind). Both deserve to be called canonicalization but the difference is real — future instances should be classified into one mode or the other. Likely Ch 11 candidate: `+=` family (almost certainly desugaring); Ch 12 candidate: compound initializers (ambiguous, depends on implementation).
- **Pre-factor-before-feature is now a two-instance named pattern.** Ch 6 §6.5 (`Function`/`Obj` merge → globals) and Ch 7 §7.6 (`printf`-to-`println` → `-o`/`--help`). The pattern is "structural change in commit N, feature in commit N+1, both small." Future instances likely at the front of Ch 10 (the `int`-becomes-32-bit refactor) and possibly mid-Ch 17 (preprocessor refactors before the harder features).
- **The `format` helper landed in §7.3.** Used heavily in later chapters. When it next appears, the prose can say "the format helper from Ch 7 §7.3" without re-explanation.
- **The `is_typename` helper grows steadily through the book.** Chapter 7 has it at two keywords. Chapter 10 grows it substantially. Each future chapter that adds a type keyword will touch this function; the §7.2 framing as "centralized for future expansion" sets up that recurring touch.
- **The trailing-newline guarantee in `read_file`** (§7.6) protects the line-comment loop in §7.7 and several places in error reporting. When `read_file` gets revisited (Ch 16 multi-file?), the guarantee should be preserved or its consumers updated.
- **The lookahead pattern from §7.1** (run a parser function with a copied cursor, inspect the result, throw away the AST) is reused in Ch 10 for nested-declarator parsing decisions and (probably) elsewhere. When that lands, the prose can say "another instance of the lookahead-by-probe pattern from Ch 7 §7.1."
- **The Trusting-Trust framing for `read_escaped_char`** sets up Ch 17 (self-hosting). When chibicc actually compiles itself, the prose should explicitly close the loop: "the tautological escape-handling from Ch 7 §7.4 is now a fixed point; if the bootstrap was clean, the output is clean." This is a specific cross-reference to plant.
- **The `.text`/`.data` directive pair is fully landed.** Ch 6 §6.5 introduced `.text`; Ch 7 §7.1 introduced `.data`. No more carrying this note.
- **The `argreg` 64-bit-only assumption is broken** (Ch 7 §7.2 split into `argreg8`/`argreg64`). When 32-bit `int` arrives in Ch 10, this will need a `argreg32` and possibly `argreg16`. Similarly `load`/`store` will gain more size cases. The size-based dispatch (`ty->size == 1`) is the shape that will generalize.
- **The `add_type` `ND_ADDR` simplification from Ch 6** is unchanged. Still a Ch 10 fix-target.
- **TY_FUNC still has no consumer.** Chapter 10 still marks the moment.
- **Ch 1 errata list** unchanged from session 007's notes.
- **`mov $0, %rax` (variadic `%al`-zeroing)** noted in Ch 5 §5.1. Pending footnote with SysV ABI section reference.
- **Six-argument cap** still implicit in `argreg` arrays. Unchanged.
- **The `add_type` rule for `ND_STMT_EXPR`** rejects void-returning bodies. Real C extension supports them; chibicc doesn't. Errata-appendix candidate.
- **The hex-escape silent truncation** (`\x00ff` → one byte without warning) is a Ch 7 §7.4 wart noted in prose but not flagged. Errata candidate.

## Exit state

- `chapters/07-globals-characters-strings.md` drafted, ~12,500 words.
- Session 008 dir populated.
- HANDOFF.md primes session 009 (Chapter 8 — Scopes and source locations, commits 44–48).
- CLAUDE.md status note will need updating (chapter count goes from "Ch 6 drafted" to "Ch 7 drafted").
