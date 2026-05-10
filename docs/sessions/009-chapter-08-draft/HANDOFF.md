# Handoff: Ch 8 done → proceed to Ch 9

**For:** the next claude session.
**From:** session 009.
**Status:** Ch 8 drafted. Continue autonomously to Ch 9 (Structs and unions). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/009-chapter-08-draft/README.md`](README.md)** — what session 009 did, including the third pre-factor instance, the two-scope-per-function framing, the host-cc-as-preprocessor framing for Ch 17, and the generalized-lvalue forward-reference to Ch 11 compound assignment.
2. **[`docs/sessions/008-chapter-07-draft/HANDOFF.md`](../008-chapter-07-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`08-scopes-and-source-locations.md`](../../../chapters/08-scopes-and-source-locations.md)** — the eight chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 9 = commits 49–55.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.

## Chapter 9 scope

**Title (working):** *Structs and unions*.
**Commits:** 49–55 in chronological order on `main`.

| # | Hash | Subject |
|---|---|---|
| 49 | `f814033` | Add struct |
| 50 | `9443e4b` | Align struct members |
| 51 | `dfec115` | Align local variables |
| 52 | `e1e831e` | Support struct tags |
| 53 | `f0a018a` | Add `->` operator |
| 54 | `11e3841` | Add union |
| 55 | `bef0543` | Add struct assignment |

**Seven commits**, all of moderate size except `dfec115` (alignment of local variables, four-line diff). Mix of substantive (struct introduction, alignment, tags, union, struct assignment) and small (`->` is a syntactic-sugar commit, `dfec115` is a four-liner).

Natural sectioning probably matches commit order with one possible bundling (50 + 51 — both alignment commits — could share a section), though they're logically separate (struct member alignment vs local-variable alignment) and the diffs are independent. The default is one section per commit; consider bundling 50 + 51 only if the prose really wants it.

- **§9.1 — `struct` introduction** (commit 49, `f814033`). The big one. Adds `TY_STRUCT`, the `Member` struct, `struct-decl` parser, member parsing, member-access via `.`, member offset assignment in `struct_decl`, codegen for `ND_MEMBER`. About 150 lines of diff. The first compound type chibicc supports.
- **§9.2 — Aligning struct members** (commit 50, `9443e4b`). Each member gets aligned to its type's alignment, the struct itself gets aligned to its largest member's alignment, and the struct's size is rounded up to a multiple of its alignment. This is when chibicc grows an `align` field on `Type` (alongside `size`) and an `align_to` helper. Member offsets that were `0, 8, 16, ...` become properly aligned for the actual member sizes.
- **§9.3 — Aligning local variables** (commit 51, `dfec115`). Four-line diff. `assign_lvar_offsets` rounds each local's stack offset to the local's alignment, not just to its size. This commit is dated *before* the struct-alignment commit (Aug 9 2019 vs Aug 30 2020) but appears after it in the canonical history. Worth noting that the order makes pedagogical sense — once struct types exist with non-trivial alignment requirements, locals containing those struct types need aligned stack slots.
- **§9.4 — Struct tags** (commit 52, `e1e831e`). Adds named struct types: `struct Point { int x, y; }; struct Point p;`. The `Scope` struct gets a `tags` field alongside `vars`, and `find_tag`/`push_tag_scope` parallel the variable functions. Tags live in the same scope chain as variables but in a separate namespace — this is C's "struct tags are in a different namespace from ordinary identifiers" rule. The `struct Point p;` syntax requires the tag to be looked up at declaration time, and the resulting `Type` is shared between all uses of the tag in the same scope.
- **§9.5 — `->` operator** (commit 53, `f0a018a`). Smallest commit. `p->x` is parsed as `(*p).x`, which means the parser desugars `->` into a deref-then-`.` sequence. This is another **canonicalization-at-parse-time** instance — definitely *desugaring* (not delegation), since `->` is rewritten to a different node-shape (ND_DEREF wrapping ND_MEMBER). Worth flagging as the sixth canonicalization instance overall.
- **§9.6 — Union** (commit 54, `11e3841`). Adds `TY_UNION` and the `union` keyword. Most of the parsing logic is shared with struct (the same `struct_members` parser is used), but offsets are all 0 and the type's size is the maximum member size (not the sum). Tag scoping reuses the same `tags` chain from §9.4 — struct tags and union tags share the namespace, which is technically incorrect by the C standard (they're in *separate* namespaces in C99) but chibicc doesn't care. Errata candidate.
- **§9.7 — Struct assignment** (commit 55, `bef0543`). Until this commit, `s1 = s2` (where both are structs) wasn't supported — assignment was integer-and-pointer only. The codegen change is interesting: `gen_expr` for `ND_ASSIGN` checks if the lhs is a struct and, if so, emits a byte-by-byte copy loop instead of a single `mov`. The implementation walks `ty->size` bytes and emits `mov (%rsi), %al; mov %al, (%rdi)` pairs.

**Concept interlude:** The chapter mapping doesn't list one for Ch 9. The natural moment if there were one would be on **alignment and the System V ABI's struct layout rules** (around §9.2), but the chapter mapping is silent and chibicc's alignment story is mechanical enough that an in-prose paragraph in §9.2 should suffice. Default to no interlude.

## Steps

1. `cd research/sources/chibicc && for h in f814033 9443e4b dfec115 e1e831e f0a018a 11e3841 bef0543; do git show --stat $h; done` to scan all seven diffs.
2. Read each commit in full. Pay particular attention to:
   - **`f814033`**: how `Member` is structured (name, type, offset, next-pointer), how `struct_decl` parses the brace-enclosed list, how member offsets are assigned (sum-of-sizes initially, no alignment yet), how `ND_MEMBER` codegen works (compute the struct's address, add the offset, then load).
   - **`9443e4b`**: the new `align` field on `Type`, the `align_to(int n, int align)` helper, where alignment is computed for each Type kind (in `type.c`'s constructors), how `struct_decl` now rounds each member's offset and the struct's total size.
   - **`dfec115`**: just `assign_lvar_offsets` rounding offsets up to alignment. Tiny.
   - **`e1e831e`**: the `tags` field on `Scope`, `find_tag`, `push_tag_scope`, the parser's `struct-decl` learning to look for an optional tag name, the lookup-by-tag path in `struct-decl` when there are no `{`s.
   - **`f0a018a`**: the `->` punctuator added in tokenize.c (or the punctuator-reading machinery already supports it from earlier — check), and the parser's desugaring of `p->x` to `(*p).x`.
   - **`11e3841`**: how `TY_UNION` is added, how `struct_decl` is renamed/factored to handle both, where the size/offset logic differs.
   - **`bef0543`**: the codegen change in `gen_expr`'s `ND_ASSIGN` case, the byte-by-byte copy loop, and any `add_type` change for struct assignment's type rule.
3. Read the destination state at `bef0543` (or shortly after) for `chibicc.h`, `parse.c`, `codegen.c`, `type.c`, `test/struct.c`, `test/union.c`.
4. Draft `chapters/09-structs-and-unions.md`. Likely 8,000–10,000 words. Seven commits, four substantive (struct, member-align, tags, union, struct-assignment — that's five, depending how you count) and two small (`->`, local-align). The section budget should reflect that — §9.1, §9.2, §9.4, §9.6, §9.7 deserve full treatment; §9.3 and §9.5 can be tight.
5. Write `docs/sessions/010-chapter-09-draft/README.md`.
6. Write `HANDOFF.md` for session 011 (Chapter 10 — Filling out the type system, commits 56–75; the largest commit count of any chapter and the chapter where `int` becomes 32-bit).

## Voice / structure rules

Same as Ch 1–8:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with a feature table — seven rows, one per commit.
- Diff format: lean toward inline diff fragments and quoted file snippets.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage. Ch 9 may not have an obvious quote moment; check before passing.
- §9.3 (local-variable alignment) is a four-line diff. Resist the urge to over-explain. Two paragraphs, maybe three. The point is "now that struct types have non-trivial alignment, locals holding them need aligned stack slots, and here's the four-line change that does it."
- §9.5 (`->`) is desugaring at parse time, the sixth canonicalization instance. The prose should explicitly count it and classify it as desugaring (not delegation).
- §9.6 (union) shares a *lot* of parser code with struct. Don't re-explain the parser machinery; the prose should highlight only the differences (offset = 0 for all members, size = max).
- §9.4 (struct tags) introduces the *second* kind of namespace in chibicc. The §8.1 prose set up scopes for variables; tags live in a parallel chain. The wart that struct tags and union tags share a namespace (incorrect by C99) is an errata candidate. Mention in prose, don't dwell.
- §9.7 (struct assignment) is byte-by-byte copy in chibicc. Real compilers use `rep movsb` or vector loads. Worth noting that chibicc takes the simple path; don't dwell on the missing optimization.
- Watch the date weirdness. `dfec115` is dated 2019-08-09 but appears as commit 51, after `9443e4b` (2020-08-30). Same intro pattern as Chs 7 and 8 — mention the dates don't match commit-list position.
- The `->` commit's tokenizer change (or lack of it) is worth checking. The two-character punctuator `->` may already be lexable from earlier commits, in which case the commit only adds the parser side.

## Standing notes worth tracking across sessions

- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.
- **The `mov $0, %rax` (variadic `%al`-zeroing)** is established in Ch 5 §5.1. Footnote with SysV ABI section reference (3.2.3) is a possible revision-pass addition.
- **The "more than 6 args silently miscompiles"** call-out is established in Ch 5 §5.4. Errata appendix candidate.
- **The `add_type` `ND_ADDR` simplification** (Ch 6) is still a Ch 10 fix-target.
- **TY_FUNC still has no consumer.** Chapter 10 still marks the moment.
- **Canonicalization-at-parse-time** is a five-instance pattern with one sub-variant after Ch 7. Ch 9 §9.5 (`->` desugaring) makes it six instances, with the sub-variant breakdown remaining one delegation (Ch 7 §7.5) and five desugarings (four in Ch 6, one in Ch 9). Ch 11's `+=` family will likely add more desugaring instances; Ch 12's compound initializers ambiguous.
- **Pre-factor before feature** is now a three-instance named pattern after Ch 8 §8.3. Ch 9 doesn't obviously have one. Likely Ch 10 instance at the front (`int`-becomes-32-bit refactor before the new types).
- **The `.text`/`.data` directive pair is fully landed** (Ch 6 §6.5 and Ch 7 §7.1).
- **The `argreg` 8/64 split is done** (Ch 7 §7.2). When `int` becomes 32-bit in Ch 10, will need `argreg32` and possibly `argreg16`. Size-based dispatch is the shape that generalizes.
- **The `format` helper landed in Ch 7 §7.3.** Workhorse going forward.
- **The `is_typename` helper landed in Ch 7 §7.2.** Grows steadily; will gain `struct` and `union` keywords in Ch 9.
- **The trailing-newline guarantee in `read_file`** (Ch 7 §7.6) protects line-comment skipping and several error-reporting paths. When `read_file` is revisited (Ch 16), preserve.
- **The lookahead-by-probe pattern** named in Ch 7 §7.1. Likely Ch 10 instance for nested-declarator parsing decisions.
- **The Trusting-Trust framing for `read_escaped_char`** (Ch 7 §7.4) sets up Ch 17 (self-hosting).
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4) is a wart. Errata candidate.
- **Block scope arrived in Ch 8 §8.1.** The two-scope-per-function structure (params outer, body inner) is a subtle invariant. The §8.1 prose flagged the redeclaration-in-same-scope check as missing — Ch 9 §9.4 (tags) doesn't check it either, both are errata candidates.
- **Per-token line numbers landed in Ch 8 §8.3.** Used by `.loc` directives in Ch 8 §8.4 and by error-tok throughout. When the preprocessor lands in Ch 17, line numbers will need to track macro-expansion locations.
- **GDB-debuggable output landed in Ch 8 §8.4.** Future debugging-related sections can take this for granted.
- **Tests are in C** as of Ch 8 §8.2. New language features get tests in `test/<feature>.c`. `test/struct.c` and `test/union.c` arrive in Ch 9.
- **The host-cc-as-preprocessor pipeline** is Ch 8 §8.2 mechanism. Collapses in Ch 17 when chibicc gets its own preprocessor.
- **The comma operator's generalized-lvalue extension** (Ch 8 §8.5) is unused in Ch 8 itself. Likely Ch 11 consumer is `+=` lowering. When Ch 11 lands, the prose there should explicitly close the loop back to §8.5.
- **The redeclaration-in-same-scope check** is missing as of Ch 8 §8.1. Errata candidate.
- **Chapter 7's mention of commit hash `46c75e7`** for the precompute commit is wrong (actual is `6647ad9`). Errata for the revision pass.

## Acceptance criteria for Ch 9

- [ ] `chapters/09-structs-and-unions.md` exists, end-to-end readable.
- [ ] §9.1 explains the `Member` struct, member-list parsing, offset assignment, and `ND_MEMBER` codegen.
- [ ] §9.2 explains alignment: the `align` field on `Type`, the `align_to` helper, how struct members get aligned, and how struct size is rounded up.
- [ ] §9.4 explains the `tags` namespace as parallel to the `vars` namespace in `Scope`, and notes that struct/union tags incorrectly share a namespace in chibicc (errata candidate).
- [ ] §9.5 calls `->` desugaring at parse time the sixth canonicalization instance.
- [ ] §9.7 explains the byte-by-byte copy loop for struct assignment.
- [ ] Each commit has a `git checkout <full-hash>` opener.
- [ ] Voice matches Ch 1–8.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/010-chapter-09-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 011 (Chapter 10 — Filling out the type system, commits 56–75).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/009-chapter-08-draft/HANDOFF.md  (this handoff)
2. docs/sessions/009-chapter-08-draft/README.md   (what session 009 did)
3. chapters/01-a-calculator.md                     (template, voice)
4. chapters/02-from-program-to-programs.md
5. chapters/03-statements-and-local-variables.md
6. chapters/04-pointers.md
7. chapters/05-functions.md
8. chapters/06-arrays.md
9. chapters/07-globals-characters-strings.md
10. chapters/08-scopes-and-source-locations.md     (most recent chapter)
11. research/commits/chapter-mapping.md            (confirms Ch 9 scope)
12. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 9 (Structs and unions, commits 49–55) per the
steps in the handoff. End-of-session: write your session dir under
docs/sessions/010-chapter-09-draft/ with a README and a HANDOFF for
session 011 (Chapter 10 — Filling out the type system, commits 56–75,
the largest commit count of any chapter).
```
