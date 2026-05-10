# Session 011 — Chapter 10 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that has produced sessions 002–010).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 010 delivered Ch 9 (Structs and unions, seven commits, ~9,300 words). User direction is still autonomous — no chapter-by-chapter review. Ch 10 covers commits 56–75: the type system fill-out — `int` becomes 32 bits, `long`/`short`/`void`/`_Bool`, nested declarators, function declarations, complex declspec, `long long`, `typedef`, `sizeof(typename)`, 32-bit register usage, casts, the usual arithmetic conversion, function-call type conversion, character literals, `enum`, file-scope `static`. Twenty commits — the largest commit count of any chapter in the book.

## What was done

### Drafting decisions

- **Length:** ~14,800 words. Largest chapter so far. Target was 14,000–16,000 per the handoff, hit the middle of the band. Compares to Ch 7 (~13,800 — previous record), Ch 9 (~9,300), Ch 8 (~7,400). Per-section budget skewed toward the structural-shift sections: §10.5 (declspec rewrite), §10.6 (typedef), §10.10 (usual arithmetic conversion), §10.11 (function-call conversions), §10.14 (enum) each ran 800–1,400 words; the smaller "add a type" sections (§10.13 character literals, §10.15 static functions) ran 200–400.
- **Section structure:** 15 sections plus one concept interlude. Followed the handoff's bundling proposal closely with a few adjustments:
  - §10.2 bundled commits 57, 58, 61 (`long`, `short`, `void`) — three same-shape additions, walked one (`long`) in detail and noted what differs for `short` and `void`. Per handoff.
  - §10.5 bundled commits 62, 63 (complex declspec + `long long` alias). Per handoff. The bit-field-counter mechanism is the centerpiece; the `long long` extension is two switch arms.
  - §10.11 bundled commits 69, 70, 71 (undeclared-function error, return-type conversion, argument-type conversion). Per handoff. The three commits form one logical arc — function-call type-checking — and reading them as a unit lets the prose explain why `9e211cb` adds the symbol-table lookup that `818352a` and `fdc80bc` use.
  - Did **not** bundle §10.13 (character literals) into §10.6 (typedef) or §10.7 (sizeof typename). The handoff floated folding the small commits if the chapter became too long; it didn't.
- **Concept interlude placed between §10.2 and §10.3** — directly before the nested-declarator commit (§10.3 = commit 59). The interlude introduces "how to read a C type declaration" with the spiral rule and a worked-out table of nine progressively gnarlier declarations. Runs ~600 words on its own; the centerpiece is the `int (*x)[10]` vs. `int *x[10]` vs. `int (*x)(int, int)` triple, walked through identifier-outward.
- **§10.1 is framed as a long-running pre-factor.** The handoff's framing said the codegen catch-up "doesn't happen until commit 66"; that's slightly inaccurate. Commit 56 already adds `argreg32`, `movsxd`, `mov %eax`, and `store_gp` — the load/store/parameter-pass machinery for a 4-byte type. Commit 66 is a *separate* arithmetic-codegen change (using `%eax`/`%edi` instead of `%rax`/`%rdi` for non-`long`, non-pointer operands). The chapter calls §10.1 the fourth instance of pre-factor-before-feature, with the *arithmetic* catch-up arriving in §10.8 ten commits later.
- **§10.10 (usual arithmetic conversion) is the chapter's most concept-heavy section.** Ran ~1,400 words and pulls together five threads: `get_common_type` and the rules; `usual_arith_conv` insertion at six binary-operator types; the `ND_NUM` value-dependent type rule; `new_long` for pointer-arithmetic constants; the `load` widening change from "extend to quad" to "extend to long" (and the documenting comment). Includes a small interpretive note that chibicc's rules are simpler than C99's full integer-promotion rules — chibicc has no unsigned types yet (Ch 14).
- **§10.14 (enum) corrects a Ch 9 errata.** The Ch 9 prose flagged "struct and union tags share a namespace" as an errata candidate — actually this is C-correct (C99 §6.2.3: struct, union, and enum tags share a single namespace per scope). The Ch 10 §10.14 prose includes one sentence quietly correcting course without re-litigating the Ch 9 framing: "(The Ch 9 prose treated struct/union tag-sharing as an errata candidate; that was a misreading on our part — sharing the tag namespace across struct/union/enum is exactly what the standard requires. We'll quietly skip restating it as a wart in this section.)" The recap also names this as a quiet correction.
- **Date-vs-position note in the intro.** Roughly seven of the twenty commits are dated August 2019; the rest fall between March and September 2020. Notably commit 56 (`5831eda`, dated 2020-09) sits in `main` order *before* the August-2019 commits, which means in chronological terms the August-2019 commits were originally written when `int` was still 8 bytes — and were rewritten or re-tested when Rui changed the size. Same intro pattern as Chs 7, 8, 9.
- **Diff format** matches Chs 7–9: inline diff fragments where the change is small, full quoted snippets where a function is new or substantially rewritten. The §10.5 declspec rewrite is shown as the full new function (it's the chapter's centerpiece). The §10.6 typedef parser-side changes are a series of inline diff fragments and full snippets of `parse_typedef` and the `compound_stmt` integration. The §10.9 cast table is shown in full because its 4×4 structure is the explanation. The §10.14 `enum_specifier` is shown in full because the two-case structure (with-body, without-body) is the explanation.
- **Two tables in the recap** — the handoff predicted twenty rows would be too many for one table, and that proved right. Split into two ten-row tables, themed around "types and parsing" vs. "casts, conversions, and codegen." Reading either as a vertical scan is manageable.

### Interpretive calls

1. **Counting pre-factor-before-feature instances.** The handoff suggested §10.1 was the fourth instance. Ran with that. Did *not* count §10.6's `VarAttr` introduction or §10.6's `parse_typedef` factoring as a fifth pre-factor for §10.15's `static`, because those changes ship in the same commit as their first user (`typedef`). The pattern's strict version is "pre-factor in commit N, feature using it in commit M > N"; mid-commit refactor-then-extend doesn't qualify.
2. **Counting canonicalization-at-parse-time instances.** The handoff suggested Ch 10 might add none. Ran with that. The cast insertions in §10.10 (usual arithmetic conversion), §10.11 (return/argument conversion), and §10.10's `ND_ASSIGN` rule are *type-level* rewrites — they insert `ND_CAST` nodes but don't desugar one AST shape into a more-primitive shape. The recap explicitly notes this distinction. Count remains at six.
3. **The `is_typename` framing.** Called the §10.6 typedef-handling change "the chapter's third interesting symbol-table extension" (after typedef and after struct tags), and "the standard C lexer-versus-parser hack — sometimes known as 'the lexer hack'." The recap quotes the standard name because readers familiar with C compiler folklore will recognize it; explained in-line for those who don't.
4. **The cast table's elision of char-or-short → 64.** Worked out the 4×4 cast table's logic from first principles in the prose (rather than just quoting it), because the elision of char/short → long is non-obvious but follows from the load-with-sign-extension-to-int convention. Future reference for the §10.10 prose.
5. **The Ch 9 errata correction.** Done as one sentence, in §10.14, without re-litigating. Did not retroactively edit Ch 9's text because the user has said to continue forward. Will add to the carry-forward notes in this README that Ch 9 has an errata candidate of its own (the errata-candidate flag was the wrong call).

### Voice / structure inherited from Ch 1–9

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote (multiple openers for bundled sections).
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature tables (two tables this chapter — twenty rows split by theme).

### Two careful avoidances

- **Did not over-explain the `(*x)[3]` vs. `*x[3]` reading.** The concept interlude does the work in one place; §10.3 trusts the reader and shows the parser code without re-deriving the spiral rule.
- **Did not re-explain `add_type` from scratch in §10.10.** The §6.4 prose introduced `add_type`; §10.10 names the new arms and walks the new logic, but doesn't re-walk the recursive descent through children. The reader has it from Ch 6.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The `is_typename` symbol-table coupling** is the chapter's biggest structural shift. After §10.6, the parser's distinction between declaration and statement consults the symbol table at every point where it asks "does this start a type." The standard C lexer-versus-parser hack lives here. Watch for further interactions when Ch 11 adds `goto`/labels (which have their own fourth namespace) and Ch 13 adds `extern` (which adds another `VarAttr` flag through the same channel).
- **The bit-field-counter `declspec`** (§10.5) is the framework everything that adds a type-specifier or storage-class keyword slots into. `_Bool` (§10.12), `static` (§10.15), and Chapter 11+ extensions (`extern`, `signed`, `unsigned`, `const`, `volatile`) all walk through this.
- **The cast machinery** (§10.9 + §10.10) is now the type system's load-bearing core. Casts get inserted everywhere — usual arithmetic conversion, return values, arguments, assignments. The 4×4 `cast_table` plus the `_Bool` special case is the entire codegen surface. Ch 14's signedness and Ch 15's floats both extend the table.
- **Pre-factor-before-feature count is now four.** §6.5, §7.6, §8.3, §10.1. The §10.1 instance is unusually long-running — ten commits between pre-factor and codegen catch-up.
- **Canonicalization-at-parse-time count remains at six.** Ch 11's `+=` family is the next likely instance — probably several at once, as `a += b` lowers via the §8.5 generalized-lvalue comma extension.
- **The everything-fits-in-rax invariant** is now a three-regime rule: scalar values < 8 bytes live in `%eax` with low bits valid and upper bits undefined; scalar values of size 8 live in `%rax`; struct/union "values" live at addresses pointed to by `%rax`. Cast machinery moves values between regimes.
- **The argreg split is complete.** §7.2 introduced `argreg8`/`argreg64`; §10.1 added `argreg32`; §10.2 added `argreg16`. All four widths exist; `store_gp` dispatches.
- **The `is_typename` helper is now a context-sensitive predicate.** Returns true for keyword type-specifiers (lookup in `kw[]`) or for typedef names (lookup in symbol table). The dual structure persists.
- **The `VarAttr` channel** is the carrier for storage-class specifiers. Currently `is_typedef` and `is_static`. Ch 13's `extern` adds a third flag.
- **The `unreachable()` macro** lives in `chibicc.h` and aborts the compile with file-and-line. Used by `store_gp` (§10.1) and the `declspec` arm-counter `unreachable()` arm (§10.5). Will get more callers.
- **The `error_at`/`error_tok` exit-from-wrapper** change in §10.11 (commit 69) is a small architectural shift: `verror_at` no longer aborts. Nothing yet calls `verror_at` for warnings, but the structural separation is there.
- **The `Node.func_ty` field** (§10.11, commit 71) is laid for later codegen work. Ch 14 (variadics) and Ch 15 (floating-point arguments) will use it.
- **The `current_fn` static in `parse.c`** (§10.11, commit 70) is separate from the same-named `current_fn` in `codegen.c` — different translation units. Whether to merge is a code-cleanliness question we don't engage with.
- **The Ch 9 errata-candidate "struct and union tags share a namespace"** is not actually errata. C99 says struct, union, and enum tags share one namespace per scope (§6.2.3), and chibicc gets this right. §10.14 quietly corrects course; the carry-forward errata list should drop this entry.
- **The struct-and-union-tag wart** in the standing notes from Sessions 008–010 is also not a wart — see above.
- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.
- **The `mov $0, %rax`** (variadic `%al`-zeroing) noted in Ch 5 §5.1. Pending footnote for revision pass.
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4. Errata candidate. Note that §10.11's argument-conversion code does *not* fix this — when a call has more arguments than the declared parameter list, the loop's `if (param_ty)` guard skips the cast for the extras, but the codegen still pushes them and the callee still reads them. The check in §10.11 catches *implicit declaration of a function* and *not a function*, but doesn't catch *too many arguments*.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **The redeclaration-in-same-scope check** is still missing for variables (Ch 8 §8.1) and tags (Ch 9 §9.4). Both errata candidates. §10.6 typedef adds another opportunity for this — `typedef int x; typedef int x;` is silently accepted in chibicc; not tested.
- **Chapter 7's mention of commit hash `46c75e7`** for the precompute commit is wrong (actual is `6647ad9`). Errata for the revision pass.
- **The `add_type` rule for binary operators changed twice** — first in §10.2 (long becomes the catch-all type), then in §10.10 (split into per-operator arms with `usual_arith_conv`). The §10.2 catch-all is short-lived; the §10.10 split is the durable structure.
- **`enum_type()` returns a fresh type each call.** No caching. Two `enum { A };` declarations in different scopes get different `Type *` instances, even though they're structurally identical. Unlike `ty_int` etc. which are shared globals.

## Exit state

- `chapters/10-filling-out-the-type-system.md` drafted, ~14,800 words.
- Session 011 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 012 (Chapter 11 — All the operators, commits 76–96; twenty-one commits, the second-largest chapter by commit count).
- CLAUDE.md status note will need updating (chapter count goes from "Ch 9 drafted" to "Ch 10 drafted").
