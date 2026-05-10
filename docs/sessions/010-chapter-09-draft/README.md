# Session 010 — Chapter 9 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that produced sessions 002–009).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 009 delivered Ch 8 (Scopes and source locations, five commits, ~7,400 words). User direction is still autonomous — no chapter-by-chapter review. Ch 9 covers commits 49–55: `struct`, member alignment, local-variable alignment, struct tags, `->`, `union`, struct assignment. Seven commits, the largest count yet for a chapter in Phase 2.

## What was done

### Drafting decisions

- **Length:** ~9,300 words. Larger than Ch 8 (~7,400) but smaller than Ch 7 (~12,500). Seven commits, five substantive (`struct` introduction, member alignment, struct tags, `union`, struct assignment) and two small (local-align, `->`). Word budget per section: §9.1 ~2,000, §9.2 ~1,400, §9.3 ~600, §9.4 ~1,500, §9.5 ~900, §9.6 ~1,400, §9.7 ~1,400. Shorter sections (§9.3 and §9.5) per the handoff.
- **No concept interlude.** Per handoff: chibicc's alignment story is mechanical enough that the in-prose paragraph in §9.2 covers it. Intro flags this explicitly.
- **Section structure:** seven sections, one per commit, in commit order. No bundling. The handoff floated bundling §9.2 + §9.3 (both alignment) but the prose worked better as separate sections — they cover different mechanisms (struct member layout vs stack slot layout) and §9.3 is explicitly small enough to stand alone.
- **§9.5 names the canonicalization-at-parse-time count as six.** The previous five are enumerated with chapter/section pointers (the four named in Ch 6 §6.1 plus the Ch 7 §7.5 delegation variant). The `->` instance is classified as a strict desugaring (no new node kind, AST shape coincides with `(*p).y`).
- **§9.4 (struct tags) framed as the second namespace in chibicc.** The §8.1 `Scope` chain is reused by adding a parallel `tags` field; the prose calls out that block-scope behavior for tags is automatic because the existing `enter_scope`/`leave_scope` machinery is shared. The C99 wart (struct and union tags share a namespace in chibicc but should be separate) is mentioned as an errata candidate, picked up again in §9.6 when union joins.
- **§9.7 (struct assignment) walks the codegen flow explicitly.** The interlocking changes — `load` short-circuiting on `TY_STRUCT`/`TY_UNION` and `store` byte-looping — are presented together with the `ND_ASSIGN` codegen as a four-step trace. The byte-by-byte loop with `%r8b` is explained, and the alternative (`rep movsb`, vector loads) is noted as the path Rui doesn't take.
- **Date-vs-position note in the intro.** The seven commits are dated 2019–2020 in mixed order; commit 51 (`dfec115`, dated 2019-08-09) appears in `main` order *after* commit 50 (`9443e4b`, dated 2020-08-30). Same intro pattern as Chs 7 and 8.
- **Diff format** matches Chs 7–8: inline diff fragments where the change is a small edit, full quoted snippets where a function is new or substantially rewritten. `struct_union_decl` factor in §9.6 is shown as the rewritten functions side by side; `gen_addr`/`gen_expr` for `ND_MEMBER` is shown as small additions; the `load`/`store` changes in §9.7 are shown as inline diff fragments because the surrounding code is the point.
- **Forward references kept short and grounded:** Ch 10 (next chapter, `int` becoming 32-bit, the type-system fill-out, `enum` in the struct/union neighborhood), Ch 11 (`+=` family as the likely consumer of the §8.5 generalized-lvalue comma extension and the next likely canonicalization-at-parse-time instance), Ch 22 (hash-table symbol lookup — not actually mentioned in this chapter, struct member lookup is small enough that linear scan is fine and the prose doesn't dwell). All cross-checked against `chapter-mapping.md`.

### Three small interpretive calls

1. **Counting the canonicalization instances.** The handoff said "five desugarings (four in Ch 6, one in Ch 9)" plus the Ch 7 delegation. The prose in §9.5 lists them as five prior instances (with the four-from-Ch-6 framing as "named together in Chapter 6 §6.1") plus `->` as the sixth. The framing is consistent with how Ch 6 §6.1 and Ch 7 §7.5 named them.
2. **Trailing padding in §9.2 framed as "matters for arrays of the struct."** The `align_to(offset, ty->align)` at the end of `struct_decl`'s offset loop is explained as producing trailing padding that lets array-of-struct elements all be properly aligned. The §9.2 prose explicitly walks the `struct {int a; char b;}` example through size 16 (8 + 1 + 7 padding) to make the rule concrete.
3. **The `struct_union_decl` factor in §9.6 framed as a refactor-and-feature in one commit.** The handoff named pre-factor-before-feature as a three-instance pattern after Ch 8. §9.6's `struct_union_decl` is a refactor-plus-feature combined commit, not a separate pre-factor — but the impulse is the same. The prose mentions this in the recap without claiming it as a fourth instance of the pattern.

### Two careful avoidances

- **Did not over-explain §9.3 (local alignment).** Per handoff: four-line diff, two paragraphs. The §9.3 prose is one paragraph of explanation, one paragraph walking the layout arithmetic, and one short paragraph on the test cases. The "why this matters now" framing (struct types finally have nontrivial alignment) opens the section in two sentences.
- **Did not re-explain the parser machinery in §9.6.** Per handoff: the union parser shares almost all of struct's logic. The prose covers what changes (kind enum, keyword, offset/size logic, `struct_ref` widening, tag-namespace sharing) and skips re-explaining the declarator/declspec wiring, which §9.1 already covered.

### Voice / structure inherited from Ch 1–8

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with a feature table — seven rows, one per commit, in commit order.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **Canonicalization-at-parse-time** is now a six-instance pattern: Ch 3 §3.4 (`>` swap), Ch 3 §3.9 (`while` → `for`), Ch 4 §4.3 (pointer-arithmetic scaling), Ch 6 §6.1 (`x[y]` → `*(x+y)`), Ch 7 §7.5 (`({...})` delegation), Ch 9 §9.5 (`->` desugaring). Five strict desugarings, one delegation. The `+x` reduction from Ch 6 §6.2 isn't part of the named count (it's noted there as an identity reduction, not a desugaring). The initializer-split (`int x = 3` → `int x; x = 3`) was named as a "near-miss" in Ch 6 §6.1 and isn't counted in the official enumeration. Ch 11's `+=` family will likely add several desugaring instances — `a += b` → `(tmp = &a, *tmp = *tmp + b)` is the standard lowering and uses the §8.5 generalized-lvalue comma.
- **Pre-factor-before-feature** count is unchanged at three instances (Ch 6 §6.5, Ch 7 §7.6, Ch 8 §8.3). §9.6's `struct_union_decl` is a refactor-and-feature in one commit, not a clean pre-factor. The next likely clean instance is in Ch 10 — the int-becomes-32-bit refactor that probably precedes the new types.
- **The two-namespaces-per-scope structure** is now established in Ch 9 §9.4. Ch 10 is likely to add `typedef` names, which the C standard says go in the *ordinary identifier* namespace alongside variables and function names — meaning `typedef int foo;` followed by `int foo;` is a redeclaration error. Watch for whether chibicc's typedef implementation adds a third field on `Scope` or shares with `vars` (the latter is correct).
- **The struct-and-union-tags-share-a-namespace wart** is the Ch 9 errata-list entry. Both §9.4 and §9.6 mention it; the recap also mentions it. This is the second errata candidate after Ch 8's redeclaration-in-same-scope wart.
- **The redeclaration-in-same-scope check** is still missing (now also for tags). Ch 9 §9.4 explicitly notes this for tags; the wart compounds the Ch 8 §8.1 wart for variables. Both are errata candidates.
- **Block scope is reused for tags without modification.** The Ch 8 §8.1 mechanism extends to tags by adding one extra field to `Scope` and one extra inner loop in the lookup function. When Ch 10 adds typedef names, the same mechanism should extend further — but typedef names live in the same namespace as ordinary identifiers, so the extension is to share `vars` with a discriminator, not to add a third field.
- **The everything-fits-in-rax codegen invariant is broken in Ch 9 §9.7.** Until this commit, every value chibicc handled fit in `%rax`. After this commit, struct and union values "live at an address" — `gen_expr` for a struct-typed `ND_VAR` or `ND_MEMBER` leaves the *address* in `%rax`, and consumers (currently only `store`) have to know to read from there. This is a significant change in the codegen contract. Future codegen work will need to remember that `gen_expr` returns "the value, except for arrays/structs/unions where it returns the address." When Ch 10 adds short/long, the value-vs-address invariant will need to be revisited (`%rax` still fits short and long; the array/struct case stays distinct).
- **`align_to` is now used in two places** (codegen's `assign_lvar_offsets` and stack-prologue rounding; parser's `struct_decl` offset loop and `union_decl` size rounding). When Ch 13 adds `_Alignas` and explicit alignment overrides, the same helper will get a third consumer.
- **Member lookup is linear.** `get_struct_member` walks the `members` list. Fine for the program sizes chibicc cares about; would be worth revisiting if real C codebases (with structs of dozens of members) became targets. Doesn't pair naturally with the Ch 22 hash-table comment for variable lookup — struct members are per-type, not per-program.
- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()`.
- **`mov $0, %rax`** noted in Ch 5 §5.1. Pending footnote for revision pass.
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4. Errata appendix candidate.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **The redeclaration-in-same-scope check** is still missing (Ch 8 §8.1, Ch 9 §9.4). Errata candidate.
- **Struct and union tags share a namespace** (Ch 9 §9.4, §9.6). Errata candidate.
- **Chapter 7's mention of commit hash `46c75e7`** for the precompute commit is wrong (actual is `6647ad9`). Already noted in Ch 8 README; carrying forward.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17. Forward-reference plant.
- **Tests are in C** as of Ch 8 §8.2. Ch 9 added `test/struct.c` and `test/union.c` per the established pattern; no harness changes needed.
- **GDB-debuggable output** (Ch 8 §8.4) now applies to programs using structs and unions too — the byte-by-byte assignment loop in §9.7 produces a torrent of `mov` instructions, all annotated with the same `.loc` directive (the line of the assignment), which means stepping over a struct assignment in GDB stops at the next source line as expected, not partway through the byte loop.

## Exit state

- `chapters/09-structs-and-unions.md` drafted, ~9,300 words.
- Session 010 dir populated.
- HANDOFF.md primes session 011 (Chapter 10 — Filling out the type system, commits 56–75; the largest commit count of any chapter and the chapter where `int` becomes 32-bit).
- CLAUDE.md status note will need updating (chapter count goes from "Ch 8 drafted" to "Ch 9 drafted").
