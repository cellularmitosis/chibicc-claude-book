# Handoff: Ch 10 done → proceed to Ch 11

**For:** the next claude session.
**From:** session 011.
**Status:** Ch 10 drafted (~14,800 words, twenty commits, the largest chapter so far). Continue autonomously to Ch 11 (All the operators). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/011-chapter-10-draft/README.md`](README.md)** — what session 011 did, including the seven-then-revised structural shifts in §10.5/§10.6/§10.10/§10.11/§10.14, the framing of §10.1 as a long-running pre-factor, and the quiet correction of the Ch 9 errata about struct/union tag namespaces.
2. **[`docs/sessions/010-chapter-09-draft/HANDOFF.md`](../010-chapter-09-draft/HANDOFF.md)** — the previous handoff. Many standing notes still apply, but two updates: (a) the struct-and-union-tag-namespace "wart" is **not** a wart — it's C-correct (struct/union/enum tags share one namespace per scope per C99 §6.2.3); (b) `is_typename` has now become a context-sensitive predicate that consults the symbol table.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`10-filling-out-the-type-system.md`](../../../chapters/10-filling-out-the-type-system.md)** — the ten chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 11 = commits 76–96.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.

## Chapter 11 scope

**Title (working):** *All the operators*.
**Commits:** 76–96 in chronological order on `main`. **Twenty-one commits — second-largest chapter by commit count, just behind Ch 10.**
**Concept interlude:** None called out by the chapter mapping. The chapter is operator-by-operator and doesn't have a single concept that needs one. *Possibly* an interlude on `goto` and structured programming if the prose calls for it; default to no interlude.

| # | Hash | Subject |
|---|---|---|
| 76 | `a4fea2b` | Allow for-loops to define local variables |
| 77 | `01a94c0` | Add `+=`, `-=`, `*=` and `/=` |
| 78 | `47f1937` | Add pre `++` and `--` |
| 79 | `e8ca48c` | Add post `++` and `--` |
| 80 | `7df934d` | Add hexadecimal, octal and binary number literals |
| 81 | `6b88bcb` | Add `!` operator |
| 82 | `46a96d6` | Add `~` operator |
| 83 | `daa7398` | Add `%` and `%=` |
| 84 | `8644006` | Add `&`, `|`, `^`, `&=`, `|=` and `^=` |
| 85 | `f30f781` | Add `&&` and `||` |
| 86 | `29ed294` | Add a notion of an incomplete array type |
| 87 | `7963221` | Decay an array to a pointer in the func param context |
| 88 | `61a1055` | Add a notion of an incomplete struct type |
| 89 | `6116cae` | Add `goto` and labeled statement |
| 90 | `a4be55b` | Resolve conflict between labels and typedefs |
| 91 | `b3047f2` | Add `break` statement |
| 92 | `3c83dfd` | Add `continue` statement |
| 93 | `044d9ae` | Add `switch-case` |
| 94 | `d0c0cb7` | Add `<<`, `>>`, `<<=` and `>>=` |
| 95 | `447ee09` | Add `?:` operator |
| 96 | `79f5de2` | Add constant expression |

This is the chapter where chibicc gets the rest of the C operator surface. Twenty-one commits is too many for one section per commit. **Bundle aggressively, again.** Rough proposal:

- **§11.1 — For-loop locals** (commit 76). Small. The `for` parser changes to accept either an `expr-stmt` or a `declaration` in the init slot. Probably one section.
- **§11.2 — Compound assignment** (commit 77). The `+=`, `-=`, `*=`, `/=` family. The single-evaluation lowering is the centerpiece — `a += b` should evaluate `&a` once. **Likely the chapter's first canonicalization-at-parse-time addition.** Watch for whether chibicc uses the `tmp = &a, *tmp = *tmp + b` lowering with the §8.5 generalized-lvalue comma extension. Substantive section.
- **§11.3 — Pre/post increment and decrement** (commits 78, 79). Bundle. The pre-form is short — desugar `++x` to `x += 1`. The post-form is fiddlier — needs to return the old value. May be another canonicalization-at-parse-time instance, or may use a small `+ 1, -1` rewrite directly. Substantive.
- **§11.4 — Number-literal bases** (commit 80). Hex (`0x`), octal (`0`), binary (`0b`). Tokenizer change. Small.
- **§11.5 — `!` and `~`** (commits 81, 82). Bundle. `!` is "compare-zero, set-equal" (similar to `_Bool` cast); `~` is `not %rax`. Both small.
- **§11.6 — `%` and `%=`** (commit 83). Modulo and modulo-assign. The `idiv` instruction already produces remainder in `%rdx`/`%edx`; just need to read it. Small to medium.
- **§11.7 — Bitwise `&`, `|`, `^` and their compound-assign forms** (commit 84). Six operators in one commit, but they share machinery. Substantive.
- **§11.8 — `&&` and `||`** (commit 85). Short-circuit evaluation — the result of `a && b` is `a` if `a` is zero, otherwise `b`, and `b` is *not* evaluated if `a` is zero. Codegen with labels. Substantive.
- **§11.9 — Incomplete types (array, struct)** (commits 86, 87, 88). Bundle? Three commits about incompleteness — incomplete arrays in declarations, array-decay in function parameters, incomplete structs (forward declarations). The function-param-array-decay (commit 87) is the most subtle: `int f(int x[])` and `int f(int *x)` are equivalent. Substantive.
- **§11.10 — `goto` and labels** (commits 89, 90). Bundle. The labels-typedef conflict (commit 90) is a real C grammar issue: `typedef int x; x:` could be a label or a declaration starter. Resolution requires lookahead. Substantive.
- **§11.11 — `break` and `continue`** (commits 91, 92). Bundle. Both walk the enclosing-loop label stack. Small to medium.
- **§11.12 — `switch`/`case`** (commit 93). Substantive section. `case` values must be constant expressions; `default`; the falling-through behavior; codegen via comparison-and-jump or jump table. Probably comparison-and-jump in chibicc.
- **§11.13 — Shift operators `<<`, `>>` and compound-assigns** (commit 94). Four operators, share machinery. Medium.
- **§11.14 — `?:`** (commit 95). The conditional operator. Three operands, codegen with labels, `usual_arith_conv` between the second and third operands. Medium.
- **§11.15 — Constant expressions** (commit 96). The chapter's closer. A simple constant-folding evaluator that walks the AST and produces an integer result, used for `case` values, enum initializers, array sizes, etc. The `eval` function. Substantive.

That's 15 sections from 21 commits. **Target chapter length: ~15,000–17,000 words.** Slightly larger than Ch 10 because the operators have more codegen detail per commit (each operator has its own assembly idiom).

## Steps

1. `cd research/sources/chibicc && for h in a4fea2b 01a94c0 47f1937 e8ca48c 7df934d 6b88bcb 46a96d6 daa7398 8644006 f30f781 29ed294 7963221 61a1055 6116cae a4be55b b3047f2 3c83dfd 044d9ae d0c0cb7 447ee09 79f5de2; do echo "===== $h ====="; git show --stat $h | head -8; done` to scan all twenty-one diffs.
2. Read each commit. Pay particular attention to:
   - **`01a94c0`** (commit 77): the compound-assignment lowering. The §8.5 generalized-lvalue comma extension was planted explicitly for this. Watch whether chibicc uses `(tmp = &a, *tmp = *tmp + b)` or builds a new node kind. The handoff prediction is the comma lowering.
   - **`47f1937`, `e8ca48c`** (78, 79): pre and post increment. The pre-form likely shares with compound-assignment. The post-form needs to return the *old* value — usually `a = ({ tmp = a; a += 1; tmp; })` style.
   - **`f30f781`** (commit 85): `&&` and `||`. Short-circuit codegen with labels — the standard pattern is `cmp; jz/jnz; gen_rhs; setne; movzx`.
   - **`29ed294`, `7963221`, `61a1055`** (commits 86–88): incomplete types. The function-param array decay (commit 87) is the C-standard rule that `int f(int x[10])` is the same as `int f(int *x)`. Watch for the parser-side rewrite.
   - **`6116cae`, `a4be55b`** (commits 89, 90): `goto` and the typedef-conflict resolution. The 4th namespace (labels) joins. The conflict resolution is a parser-side lookahead.
   - **`044d9ae`** (commit 93): `switch-case`. Codegen with comparison-and-jump (probably) and a `default`-fallback label.
   - **`447ee09`** (commit 95): `?:`. Three operands; uses `usual_arith_conv` between then and else expressions.
   - **`79f5de2`** (commit 96): constant expression. The `eval` function. Used by `case` values.
3. Read the destination state at `79f5de2` (or shortly after) for `chibicc.h`, `parse.c`, `codegen.c`, `tokenize.c`, all relevant test files. The Ch 11 codegen surface is broader than Ch 10's; expect to spend longer on this step.
4. Draft `chapters/11-all-the-operators.md`. Likely 15,000–17,000 words. **No concept interlude unless the prose surfaces a clear need** — possible candidate is "structured programming and the case for/against `goto`" if §11.10 calls for it, but default to no interlude.
5. Write `docs/sessions/012-chapter-11-draft/README.md`.
6. Write `HANDOFF.md` for session 013 (Chapter 12 — Initializers, commits 97–115; nineteen commits, the densest arc in the compiler per the chapter mapping).

## Voice / structure rules

Same as Ch 1–10:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — twenty-one rows; consider splitting into two tables by theme (operators vs. control-flow) as Ch 10 did.
- Diff format: lean toward inline diff fragments and quoted file snippets. The cast-table-style 4×4 dispatcher in §10.9 is a good model when codegen has table structure (e.g., the bitwise operators all share a small dispatcher).

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **Bundle aggressively.** Twenty-one commits won't fit one-per-section at any reasonable chapter length. Group by theme; a section can cover three commits if they share a pattern (the pattern in this chapter is "operator-family with same lowering shape" — `+=` family, bitwise family, shift family, increment family).
- **§11.2 (compound assignment) closes the §8.5 comma extension's loop.** The §8.5 prose said the comma's generalized-lvalue extension was unused as of Ch 9 and predicted it would land here. Confirm or disconfirm in the §11.2 prose. If chibicc uses a different lowering, name the discrepancy.
- **§11.10 (`goto` + labels) introduces a fourth namespace.** Labels are function-scoped, not block-scoped. The Ch 9 §9.4 prose listed two namespaces (variables, tags); Ch 10 §10.6 added a third logical namespace inside `vars` (typedef names share with variables). Labels are a fourth, *separate*, function-scoped namespace.
- **§11.10's typedef-vs-label conflict is real C.** `typedef int x; x:;` could be a label-statement (`x:` is a label) or a declaration (`x` is a typedef-name and `:` would be a parse error). The C standard says label, because labels-then-statement is the only form that starts with `IDENT :`. Watch chibicc's resolution — probably one-token lookahead at `expr_stmt` or `stmt`.
- **§11.12 (switch) interacts with the constant-expression machinery.** `case` labels need integer constants. If chibicc writes the constant evaluator first (as a §11.15 commit), the `switch` commit (§11.12, commit 93) precedes it — meaning `switch`'s `case` parsing will need *something* to evaluate constants and the something probably isn't the full `eval`. Watch for any temporary hack vs. a real call to `eval`. The chronological order — commit 93 is before commit 96 — suggests a temporary `get_number` is used until commit 96 generalizes it.
- **§11.15 (constant expression) is small but load-bearing.** The `eval` function will get more callers in Ch 12 (initializers) and Ch 13 (compound literals). Closing the chapter with it sets up Ch 12.
- **The compound-assignment family (§11.2) is the canonicalization-at-parse-time hot spot.** Likely instance count goes from six to seven, eight, or nine depending on whether `+=`, `++`, and `--` are each counted as their own desugaring or as one mechanism. Treat them as one mechanism in the count (compound-assign-via-comma) — but name the variants individually in prose.
- **The `is_typename` symbol-table lookup from §10.6** gets stressed in §11.10. The label-typedef conflict means `is_typename` and the new `is_label` (or whatever chibicc calls it) need to coexist. Watch for whether `is_typename` gains any extension or whether the label-form is handled entirely in `stmt`.

## Standing notes worth tracking across sessions

- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.
- **The `mov $0, %rax` (variadic `%al`-zeroing)** noted in Ch 5 §5.1. Pending footnote for revision pass.
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4. Errata appendix candidate. §10.11 does *not* fix this — it adds parameter-count-aware casting but skips the cast for extra arguments rather than erroring.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables (Ch 8 §8.1), tags (Ch 9 §9.4), and typedef names (Ch 10 §10.6). Three errata candidates.
- **Block scope is established** as of Ch 8 §8.1. Ch 9 §9.4 added tags; Ch 10 §10.6 added typedef names (sharing `vars`); Ch 10 §10.14 added enum constants (also sharing `vars`); Ch 11 will likely add labels (separate, function-scoped namespace).
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted. Should keep working through all of Ch 11; the new operator codegen continues to emit `.loc` directives via `gen_expr`'s opening line.
- **Tests are in C** as of Ch 8 §8.2. New language features get tests in `test/<feature>.c`.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17.
- **The comma operator's generalized-lvalue extension** (Ch 8 §8.5) is the load-bearing mechanism for compound assignment. Ch 11 §11.2 closes the loop predicted in Ch 8.
- **Canonicalization-at-parse-time** is a six-instance pattern after Ch 10. Ch 11 will add at least one (compound-assign), possibly more (pre/post-increment if implemented as a separate desugaring, ?: if implemented via a desugaring rather than a new node kind). Treat related variants as one mechanism in the count.
- **Pre-factor-before-feature** is a four-instance named pattern after Ch 10 §10.1. Ch 11 likely adds none — most operator additions are self-contained — but watch for any scaffolding-in-advance commits.
- **The argreg 8/16/32/64 split** (Ch 10 §10.1, §10.2) is fully in place. Ch 11 codegen will use the right width per operand type via the same machinery.
- **The `is_typename` helper is a context-sensitive predicate** as of Ch 10 §10.6. Ch 11 §11.10 stresses this when labels join the namespace landscape.
- **The cast machinery** (`new_cast` + 4×4 `cast_table`) is the type system's load-bearing core. Ch 11's `?:` (§11.14) and shifts (§11.13) use `usual_arith_conv` for operand promotion. The bitwise operators (§11.7) likely also use it.
- **The `format` helper landed in Ch 7 §7.3.** Workhorse going forward.
- **The trailing-newline guarantee in `read_file`** (Ch 7 §7.6) protects line-comment skipping. When `read_file` is revisited (Ch 16), preserve.
- **The lookahead-by-probe pattern** named in Ch 7 §7.1. Ch 10 §10.3 (nested declarators) and §10.7 (abstract declarators) used the double-walk-with-dummy-type idiom — same family. Ch 11 §11.10 (label-vs-typedef) is the next likely consumer.
- **The Trusting-Trust framing for `read_escaped_char`** (Ch 7 §7.4) sets up Ch 17 (self-hosting). §10.13 (character literals) leaned on this.
- **The everything-fits-in-rax codegen invariant** is now a three-regime rule (scalar < 8 in `%eax`, scalar = 8 in `%rax`, struct/union at address pointed to by `%rax`). Cast machinery moves between regimes. Ch 11 doesn't change this.
- **Member lookup is linear** (Ch 9 §9.1). Fine.
- **The struct/union/enum tag namespace** is one shared namespace per scope, per C99. Earlier Ch 9 framing as a wart was wrong; Ch 10 §10.14 quietly corrected.
- **The `unreachable()` macro** (Ch 10 §10.1) lives in `chibicc.h`. Used by `store_gp` and the `declspec` arm-counter. Will get more callers — Ch 11's `switch` codegen may use it for invalid `case` arms.
- **The `VarAttr` channel** (Ch 10 §10.6, extended in §10.15) is the carrier for storage-class specifiers. Currently `is_typedef` and `is_static`. Ch 13's `extern` adds a third flag through this channel.

## Acceptance criteria for Ch 11

- [ ] `chapters/11-all-the-operators.md` exists, end-to-end readable.
- [ ] All twenty-one commits covered, grouped into ~15 sections (or fewer with bundling).
- [ ] §11.2 (compound assignment) closes the §8.5 generalized-lvalue comma loop and identifies the canonicalization-at-parse-time count's update.
- [ ] §11.5 (`!`) explains the parallel with the §10.12 `_Bool` cast (both use `cmp; setne; movzx`).
- [ ] §11.8 (`&&`/`||`) covers short-circuit codegen with labels.
- [ ] §11.9 (incomplete types) walks the function-param array-decay rule (commit 87) explicitly.
- [ ] §11.10 (`goto` + labels) names labels as the fourth namespace and walks the typedef-vs-label conflict.
- [ ] §11.12 (switch) covers `case` falling through and notes the constant-expression dependency.
- [ ] §11.15 (constant expression) introduces `eval` and previews its Ch 12 / Ch 13 callers.
- [ ] Each commit has a `git checkout <full-hash>` opener.
- [ ] Voice matches Ch 1–10.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/012-chapter-11-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 013 (Chapter 12 — Initializers, commits 97–115).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/011-chapter-10-draft/HANDOFF.md  (this handoff)
2. docs/sessions/011-chapter-10-draft/README.md   (what session 011 did)
3. chapters/01-a-calculator.md                     (template, voice)
4. chapters/02-from-program-to-programs.md
5. chapters/03-statements-and-local-variables.md
6. chapters/04-pointers.md
7. chapters/05-functions.md
8. chapters/06-arrays.md
9. chapters/07-globals-characters-strings.md
10. chapters/08-scopes-and-source-locations.md
11. chapters/09-structs-and-unions.md
12. chapters/10-filling-out-the-type-system.md     (most recent chapter)
13. research/commits/chapter-mapping.md            (confirms Ch 11 scope)
14. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 11 (All the operators, commits 76–96) per the steps
in the handoff. Twenty-one commits — bundle aggressively. No concept
interlude unless the prose surfaces a clear need. End-of-session: write
your session dir under docs/sessions/012-chapter-11-draft/ with a README
and a HANDOFF for session 013 (Chapter 12 — Initializers, commits 97–115).
```
