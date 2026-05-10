# Handoff: Ch 9 done → proceed to Ch 10

**For:** the next claude session.
**From:** session 010.
**Status:** Ch 9 drafted. Continue autonomously to Ch 10 (Filling out the type system). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/010-chapter-09-draft/README.md`](README.md)** — what session 010 did, including the six-instance canonicalization count, the second-namespace framing for tags, and the everything-fits-in-rax invariant that breaks in §9.7.
2. **[`docs/sessions/009-chapter-08-draft/HANDOFF.md`](../009-chapter-08-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`09-structs-and-unions.md`](../../../chapters/09-structs-and-unions.md)** — the nine chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 10 = commits 56–75.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.

## Chapter 10 scope

**Title (working):** *Filling out the type system*.
**Commits:** 56–75 in chronological order on `main`. **Twenty commits — the largest chapter in the book.**
**Concept interlude:** Yes — *how to read a C type declaration*, sourced from the JP book. Place it around the nested-declarators commit (`a817b23`, commit 59) or the complex-type-declarations commit (`287906a`, commit 62).

| # | Hash | Subject |
|---|---|---|
| 56 | `5831eda` | Change size of int from 8 to 4 |
| 57 | `43c2f08` | Add long type |
| 58 | `9d48eef` | Add short type |
| 59 | `a817b23` | Add nested type declarators |
| 60 | `74e3acc` | Add function declaration |
| 61 | `8c3503b` | Add void type |
| 62 | `287906a` | Handle complex type declarations correctly |
| 63 | `f46370e` | Add `long long` as an alias for `long` |
| 64 | `a6b82da` | Add typedef |
| 65 | `67543ea` | Make sizeof to accept not only an expression but also a typename |
| 66 | `cb81a37` | Use 32 bit registers for char, short and int |
| 67 | `cfc4fa9` | Add type cast |
| 68 | `8b430a6` | Implement usual arithmetic conversion |
| 69 | `9e211cb` | Report an error on undefined/undeclared functions |
| 70 | `818352a` | Handle return type conversion |
| 71 | `fdc80bc` | Handle function argument type conversion |
| 72 | `44bba96` | Add _Bool type |
| 73 | `aa0accc` | Add character literal |
| 74 | `48ba265` | Add enum |
| 75 | `736232f` | Support file-scope functions |

This is the chapter where chibicc gets serious about types. Twenty commits is too many for one section per commit at this length — the chapter would explode. **Bundle aggressively.** Rough proposal:

- **§10.1 — `int` becomes 32 bits** (commit 56). The pre-factor for everything that follows. One section, short — the change is mechanical (`ty_int = {TY_INT, 4, 4}` and 32-bit register usage *isn't* yet — that comes in commit 66) but the implications run through the chapter.
- **§10.2 — Adding `long`, `short`, `void`** (commits 57, 58, 61). Three commits but they share a pattern: each adds a `TY_X` enum entry, a `ty_x` static, a `declspec` branch, an `is_typename` line, a tokenizer keyword, and `void` adds a parser-rejection rule. Bundle into one section; walk one in detail (probably `long` since it's first), then quickly cover what `short` and `void` do differently.
- **§10.3 — Nested declarators and complex type declarations** (commits 59, 62). These are the "how to read a C declaration" pair. The concept interlude lives between them or just before §10.3. Walk through `int (*x)(int, int)` and `int *x[10]` vs `int (*x)[10]`. Reference `cdecl`. The JP book has a treatment to draw from.
- **§10.4 — Function declarations** (commit 60). Forward declarations of functions. Probably its own short section because it changes the parser's top-level loop and introduces the distinction between definition and declaration.
- **§10.5 — `long long` as alias for `long`** (commit 63). Small. Could fold into §10.2 as a footnote-paragraph.
- **§10.6 — `typedef`** (commit 64). The C99 ordinary-identifier-namespace question (typedef names live with vars, not in their own namespace). Watch for whether chibicc shares `vars` with a discriminator, adds a flag to `VarScope`, or does something else.
- **§10.7 — `sizeof(typename)`** (commit 65). One section, small. The grammar gets `sizeof "(" typename ")"` as a parser-level alternative to `sizeof unary`.
- **§10.8 — 32-bit register usage** (commit 66). The big codegen commit of the chapter. Adds `argreg32`, the `mov`-vs-`movzbl` story for narrowing/widening, and the `eax`/`ax`/`al` register names. The `argreg` story from Ch 7 §7.2 generalizes here. Substantive section.
- **§10.9 — Casts** (commit 67). The `(type) expr` syntax, `ND_CAST` node, codegen for narrowing/widening between `char`/`short`/`int`/`long`.
- **§10.10 — Usual arithmetic conversion** (commit 68). The C standard's rules for promoting operands of binary operators. Substantive section. Probably reference the standard explicitly here.
- **§10.11 — Return-type and argument-type conversion** (commits 70, 71). Bundle. Both apply the conversion machinery from §10.10 across function boundaries. §10.9 (undeclared-function error) is small enough to fit at the front of this bundle or as its own short subsection.
- **§10.12 — `_Bool`** (commit 72). Special integer type with two values; conversion has `0/1` rather than `0/N` semantics. Short.
- **§10.13 — Character literals** (commit 73). `'a'` becomes a tokenizer change plus a parser path. Short.
- **§10.14 — `enum`** (commit 74). Enum types, enum tags (third entry in the tag namespace, alongside struct and union — *if* chibicc keeps them all sharing). Substantive section. Watch for whether `enum` constants live in the `vars` or `tags` namespace (standard says `vars`; the values themselves are integers, the type is a separate thing).
- **§10.15 — File-scope functions** (commit 75). The `static` keyword on functions. Probably small. Closes the chapter.

That's 15 sections from 20 commits. If even that's too many, fold §10.5/§10.7/§10.13 into adjacent sections. **Target chapter length: ~14,000–16,000 words**, the largest chapter so far. The concept interlude on type declarations probably runs 600–1,000 words on its own.

## Steps

1. `cd research/sources/chibicc && for h in 5831eda 43c2f08 9d48eef a817b23 74e3acc 8c3503b 287906a f46370e a6b82da 67543ea cb81a37 cfc4fa9 8b430a6 9e211cb 818352a fdc80bc 44bba96 aa0accc 48ba265 736232f; do echo "===== $h ====="; git show --stat $h | head -8; done` to scan all twenty diffs.
2. Read each commit. Pay particular attention to:
   - **`5831eda`**: probably trivial — `ty_int = &(Type){TY_INT, 4, 4}` and updates to tests that were checking `sizeof(int) == 8`. Watch for any consequential changes in codegen (the 32-bit register usage doesn't arrive until commit 66, so this commit may produce bugs that the test suite catches and the codegen commit eventually fixes).
   - **`43c2f08`, `9d48eef`, `8c3503b`, `f46370e`**: the four "add a type" commits. Look for the pattern.
   - **`a817b23`**: nested declarators — function pointers, pointers to arrays, etc. The grammar changes are substantial.
   - **`287906a`**: complex declarations — this is where chibicc's parser actually starts handling C declarators correctly. The `declarator` function probably gets significant rework.
   - **`a6b82da`**: typedef. The interaction with the parser's `is_typename` is tricky — `is_typename` has been a static keyword check, but typedef means it now has to look up the symbol table. This may force an `is_typename` rewrite.
   - **`cb81a37`**: 32-bit register usage. The big codegen commit. Adds `argreg32`, may add `argreg16`, changes `load`/`store` and the integer-arithmetic codegen to use the right register width.
   - **`cfc4fa9`**: type cast. New `ND_CAST` node, parser grammar `(type) unary`, codegen for narrowing/widening.
   - **`8b430a6`**: usual arithmetic conversion. The `add_type` rule for binary operators starts inserting `ND_CAST` nodes to promote operands.
   - **`818352a`, `fdc80bc`**: return-type and argument conversion. Apply the conversion to function boundaries.
   - **`48ba265`**: enum. Watch the namespace question for enum constants and enum tags.
   - **`736232f`**: file-scope functions (static). Adds `is_static` flag on `Obj`.
3. Read the destination state at `736232f` (or shortly after) for `chibicc.h`, `parse.c`, `codegen.c`, `type.c`, all relevant test files.
4. Draft `chapters/10-filling-out-the-type-system.md`. Likely 14,000–16,000 words. Include the concept interlude on reading C type declarations.
5. Write `docs/sessions/011-chapter-10-draft/README.md`.
6. Write `HANDOFF.md` for session 012 (Chapter 11 — All the operators, commits 76–96).

## Voice / structure rules

Same as Ch 1–9:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with a feature table — twenty rows, one per commit (this'll be the longest table in the book; consider if it's worth splitting into two tables or grouping).
- Diff format: lean toward inline diff fragments and quoted file snippets.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage. The 32-bit-register commit and the usual-arithmetic-conversion commit may have apt commit messages worth quoting verbatim.
- **Bundle aggressively.** Twenty commits is too many for one-section-per-commit at any reasonable chapter length. Group by theme; a section can cover three commits if they share a pattern.
- **§10.1 (`int` becomes 32-bit) is the fourth pre-factor instance.** Commit 56 is the change-only-the-types commit; the codegen catch-up doesn't happen until commit 66. Worth naming this as a particularly long-running pre-factor, where the pre-factor and the feature are separated by ten commits.
- **§10.6 (typedef) interacts with `is_typename`.** Watch whether `is_typename` becomes a symbol-table lookup. If so, this is a structural shift in how the parser knows whether a token starts a type — significant enough to call out.
- **§10.8 (32-bit registers) is a big codegen change.** Don't gloss the eax/ax/al/movzbl story. The `argreg32`/`argreg64`/`argreg8` triple was foreshadowed in Ch 7 §7.2; close the loop here.
- **§10.10 (usual arithmetic conversion) needs care.** This is one of the parts of C that programmers get wrong. The prose should explain the rules clearly (probably with a small table of which type wins in each case) and tie them to the `ND_CAST` insertion in `add_type`.
- **§10.14 (enum) namespace question.** If chibicc puts enum constants in the variable namespace, the prose should say so explicitly — that's the C-correct behavior. If it puts them in their own namespace, that's a wart for the errata appendix.
- The **concept interlude on reading C type declarations** is the chapter's pedagogical centerpiece. Draw on the JP book's treatment if there's a relevant section in the TOC notes. Reference `cdecl`. Walk through at least three examples: a function pointer, a pointer to a function returning a pointer, and an array of pointers to functions. The "spiral rule" is the standard mnemonic.
- Watch the date weirdness. Less obviously messy than Ch 7–9 because Ch 10 commits are mostly clustered in late 2020, but spot-check a few.

## Standing notes worth tracking across sessions

- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.
- **The `mov $0, %rax` (variadic `%al`-zeroing)** is established in Ch 5 §5.1. Footnote with SysV ABI section reference (3.2.3) is a possible revision-pass addition.
- **The "more than 6 args silently miscompiles"** call-out is established in Ch 5 §5.4. Errata appendix candidate.
- **The `add_type` `ND_ADDR` simplification** (Ch 6) is still a Ch 10 fix-target — actually probably not anymore, since the type system rework in Ch 10 may revisit `add_type` thoroughly. Watch for it.
- **TY_FUNC still has no consumer** as of Ch 9. Ch 10 commit 60 (function declaration) may finally use it for forward declarations, and commit 69 (undefined-function error) almost certainly will.
- **Canonicalization-at-parse-time** is a six-instance pattern after Ch 9 §9.5. Five strict desugarings (Ch 3 §3.4, Ch 3 §3.9, Ch 4 §4.3, Ch 6 §6.1, Ch 9 §9.5) and one delegation (Ch 7 §7.5). Ch 10 may add an instance — function-pointer parsing is a candidate, if `int (*f)(int)` desugars in any way. More likely: nothing new in Ch 10, and Ch 11's `+=` family adds several at once.
- **Pre-factor before feature** is a three-instance named pattern after Ch 8 §8.3. Ch 10's commit 56 (`int` → 32-bit) is the fourth instance, and an unusually long-running one — the codegen catch-up is in commit 66, ten commits later.
- **The argreg 8/64 split** (Ch 7 §7.2) gets generalized in Ch 10 §10.8 (commit 66) with the addition of `argreg32` and possibly `argreg16`. Close the loop in §10.8.
- **The `is_typename` helper** has been growing one keyword at a time; in Ch 10 it may rewrite to a symbol-table lookup if typedef forces it. Worth tracking as a structural-change moment.
- **The `format` helper landed in Ch 7 §7.3.** Workhorse going forward. May get used in Ch 10 for assembly with type-dependent formatting.
- **The trailing-newline guarantee in `read_file`** (Ch 7 §7.6) protects line-comment skipping. When `read_file` is revisited (Ch 16), preserve.
- **The lookahead-by-probe pattern** named in Ch 7 §7.1. Ch 10's nested-declarators commit (59) and complex-declarations commit (62) are likely consumers — distinguishing `int (*x)(int, int)` (function pointer) from `int (*x)` (parenthesized pointer) requires lookahead.
- **The Trusting-Trust framing for `read_escaped_char`** (Ch 7 §7.4) sets up Ch 17 (self-hosting).
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **Block scope is established** as of Ch 8 §8.1. Ch 9 added tags as a parallel namespace; Ch 10 will likely add typedef as a discriminated entry in the same `vars` namespace (C-correct), and enum constants either as `vars` entries (C-correct) or in their own namespace (wart).
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Tests are in C** as of Ch 8 §8.2. New language features get tests in `test/<feature>.c`.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17.
- **The comma operator's generalized-lvalue extension** (Ch 8 §8.5) is unused as of Ch 9. Ch 11's `+=` family is the likely consumer; close the loop in Ch 11 prose.
- **The redeclaration-in-same-scope check** is missing for both variables (Ch 8 §8.1) and tags (Ch 9 §9.4). Errata candidates.
- **Struct and union tags share a namespace** (Ch 9 §9.4, §9.6). Errata candidate. Ch 10's enum tags may join this list — if enum tags share with struct/union tags, that's another wart.
- **The everything-fits-in-rax codegen invariant** broke in Ch 9 §9.7 for struct/union (gen_expr leaves the address in `%rax`). Ch 10's 32-bit register usage in commit 66 will need to navigate this — small types fit in `%eax`, struct/union don't fit at all.
- **Member lookup is linear** (Ch 9 §9.1). Fine for chibicc.
- **Chapter 7's mention of commit hash `46c75e7`** for the precompute commit is wrong (actual is `6647ad9`). Errata for the revision pass.

## Acceptance criteria for Ch 10

- [ ] `chapters/10-filling-out-the-type-system.md` exists, end-to-end readable.
- [ ] All twenty commits covered, grouped into ~14 sections (or fewer with bundling).
- [ ] Concept interlude on reading C type declarations included.
- [ ] §10.1 (`int` → 32-bit) framed as a pre-factor whose payoff is in §10.8.
- [ ] §10.3 (declarators) walks at least three complex declaration examples (function pointer, pointer to function returning a pointer, array of pointers to functions).
- [ ] §10.6 (typedef) explains the symbol-table lookup change (if any) in `is_typename`.
- [ ] §10.8 (32-bit registers) covers eax/ax/al, movzbl/movzwl widening, and closes the argreg-split loop from Ch 7 §7.2.
- [ ] §10.10 (usual arithmetic conversion) has a clear explanation of the rules.
- [ ] §10.14 (enum) addresses the namespace question (constants in `vars`, tags in `tags` if chibicc gets it right; flag any deviation as errata).
- [ ] Each commit has a `git checkout <full-hash>` opener.
- [ ] Voice matches Ch 1–9.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/011-chapter-10-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 012 (Chapter 11 — All the operators, commits 76–96).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/010-chapter-09-draft/HANDOFF.md  (this handoff)
2. docs/sessions/010-chapter-09-draft/README.md   (what session 010 did)
3. chapters/01-a-calculator.md                     (template, voice)
4. chapters/02-from-program-to-programs.md
5. chapters/03-statements-and-local-variables.md
6. chapters/04-pointers.md
7. chapters/05-functions.md
8. chapters/06-arrays.md
9. chapters/07-globals-characters-strings.md
10. chapters/08-scopes-and-source-locations.md
11. chapters/09-structs-and-unions.md              (most recent chapter)
12. research/commits/chapter-mapping.md            (confirms Ch 10 scope)
13. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 10 (Filling out the type system, commits 56–75) per
the steps in the handoff. Twenty commits — bundle aggressively. Include
the concept interlude on reading C type declarations. End-of-session:
write your session dir under docs/sessions/011-chapter-10-draft/ with a
README and a HANDOFF for session 012 (Chapter 11 — All the operators,
commits 76–96).
```
