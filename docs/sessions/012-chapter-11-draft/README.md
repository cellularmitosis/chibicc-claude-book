# Session 012 — Chapter 11 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that has produced sessions 002–011).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 011 delivered Ch 10 (Filling out the type system, twenty commits, ~14,800 words — the largest chapter so far by word count). User direction is still autonomous — no chapter-by-chapter review. Ch 11 covers commits 76–96: for-loop locals, compound assignment, increment/decrement, number-literal bases, `!`/`~`, `%`/`%=`, bitwise + their compound-assigns, `&&`/`||`, incomplete arrays/forward struct decls/parameter array decay, `goto`/labels, `break`/`continue`, `switch`/`case`/`default`, shifts + their compound-assigns, `?:`, constant expressions. Twenty-one commits — the second-largest chapter by commit count.

## What was done

### Drafting decisions

- **Length:** ~12,260 words. Below the handoff target of 15,000–17,000 but covering all twenty-one commits. The chapter ran shorter than predicted because per-commit prose for operator codegen ends up tighter than per-commit prose for type-system commits — most operators are one or three lines of x86-64, and there's no equivalent of §10.5's declspec rewrite or §10.10's usual-arithmetic-conversion arc to extend prose. Compares to: Ch 10 (~14,800), Ch 7 (~13,800), Ch 9 (~9,300), Ch 8 (~7,400).
- **Section structure:** 15 sections, no concept interlude. Followed the handoff's bundling proposal closely:
  - §11.3 bundled commits 78, 79 (pre and post `++`/`--`). Per handoff. The post-form's `(typeof A)((A += 1) - 1)` lowering is the section's centerpiece.
  - §11.5 bundled commits 81, 82 (`!`, `~`). Per handoff. Both are minimal one-or-three-line codegen commits.
  - §11.9 bundled commits 86, 87, 88 (incomplete arrays, parameter array decay, forward struct decls). Per handoff. The unifying mechanism is the `size = -1` sentinel.
  - §11.10 bundled commits 89, 90 (`goto`/labels and the typedef conflict resolution). Per handoff.
  - §11.11 bundled commits 91, 92 (`break`/`continue`). Per handoff. Both reuse `ND_GOTO`.
- **No concept interlude.** The handoff floated `goto` and structured programming as a possible interlude topic; the §11.10 prose didn't surface a need. Default-no held.
- **§11.2 closes the §8.5 generalized-lvalue comma loop.** Per handoff acceptance criterion. The prose explicitly names §8.5's prediction and confirms the loop closes here. The canonicalization-at-parse-time count update is named: six → eight (compound-assign + pre/post-increment counted as two mechanisms).
- **§11.5 (`!`) explicitly parallels the §10.12 `_Bool` cast.** Per handoff. The prose names them as mirror-image one-letter variants (`sete` vs. `setne`).
- **§11.8 (`&&`/`||`) covers short-circuit codegen with labels.** Per handoff. The prose walks both `ND_LOGAND` and `ND_LOGOR` codegen as mirror-images and notes the C requirement that the result be exactly `0` or `1`, not the operand value.
- **§11.9 walks function-param array-decay explicitly.** Per handoff. The prose names the C-standard rule, shows the `if (ty2->kind == TY_ARRAY) { ty2 = pointer_to(ty2->base); }` swap, and notes the test that pins down `int x[]` ≡ `int *x` at the calling-convention level.
- **§11.10 names labels as the fourth namespace.** Per handoff. The prose: "Labels are a *fourth namespace* in C. Variables and typedef names share one. Struct/union/enum tags share another. Members within each struct/union live in a per-struct namespace. Labels are the fourth — they belong to a function, not a block." The label-vs-typedef parser conflict is walked with the one-token lookahead in `compound_stmt` named explicitly.
- **§11.12 covers fallthrough and notes the const-expression dependency.** Per handoff. The prose names the temporary `get_number` for case values and forward-references §11.15 as the replacement.
- **§11.15 introduces `eval` and previews Ch 12/Ch 13 callers.** Per handoff. The prose names `eval`'s structure (one switch, thirty arms) and its forward callers (Ch 12 initializers).
- **Two tables in the recap.** Per the handoff's prediction; same split logic as Ch 10 (operators on one side, control flow + types on the other).

### Interpretive calls

1. **Counting canonicalization-at-parse-time instances.** The handoff said "treat related variants as one mechanism." Ran with that. Compound-assign (the `to_assign` mechanism that handles `+=`/`-=`/`*=`/`/=`/`%=`/`&=`/`|=`/`^=`/`<<=`/`>>=`) is *one* mechanism. Pre/post-increment (which adds the cast-back step on top) is a second. So count goes 6 → 8. Did *not* count `?:` as a canonicalization, because it ships as a new node kind with branching codegen rather than desugaring into existing AST shapes. Did *not* count `&&`/`||` for the same reason.
2. **Counting pre-factor-before-feature instances.** §11.1 (for-loop locals) is a small enabler for tests that follow but the chapter doesn't formally count it as a pre-factor. The pre-factor pattern is "code in commit N supports a feature in commit M > N"; §11.1 enables a *style of test*, not a future commit. Count remains at four.
3. **The `ND_GOTO` reuse for `break`/`continue`.** Called this out as a "small cute trick" in the recap. The §11.11 prose explicitly notes the reuse: rather than `ND_BREAK` and `ND_CONTINUE`, both are `ND_GOTO` with a pre-set `unique_label`.
4. **The temporary-scaffolding pattern.** §11.12 uses `get_number` for case values, knowing it'll be replaced in §11.15. The recap names this as the chapter's only "ship a scaffold, swap it later" instance.
5. **The label-vs-typedef-name two-namespace point.** The §10.6 prose framed typedef-vs-variable name sharing as the same namespace. §11.10 needs to say labels are *separate* — and explicitly distinct from `vars`/`tags`/`members`. Wrote it in the §11.10 prose with a one-paragraph C99-style namespace recap.
6. **`?:` does not canonicalize.** The handoff was open to either a canonicalization or a new node kind for `?:`. Rui ships it as a new node kind (`ND_COND` with branching codegen). The §11.14 prose notes this and explains why: there's no clean way to express short-circuiting via existing AST shapes.
7. **The `>>` codegen quirk.** Both arms of `if (node->ty->size == 8)` in `ND_SHR` emit the same instruction (`sar`). Probably an unfinished differentiation in Rui's commit. Noted in the §11.13 prose as "perhaps Rui intended `sar` for 8-byte and a different mnemonic for 4-byte and didn't finish the differentiation," without re-litigating; this is a candidate for the errata appendix.

### Voice / structure inherited from Ch 1–10

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote (multiple openers for bundled sections).
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature tables (two tables, by theme — operators vs. control-flow + types).

### Three careful avoidances

- **Did not introduce a label-namespace concept-interlude.** The handoff defaulted to no interlude; held to that.
- **Did not over-explain the `to_assign` mechanism in three sections.** §11.2 walks it in detail. §11.3 (pre/post-increment), §11.6 (`%=`), §11.7 (`&=`/etc.), §11.13 (`<<=`/`>>=`) reference it without re-deriving.
- **Did not re-explain `usual_arith_conv` in §11.14.** The §10.10 prose introduced it; §11.14 names the mechanism and lets it stand.

### Date-vs-position note

Roughly a third of the commits are dated August 2019; about half are October 2020 (Rui's late-year cleanup); a few are scattered between (April 2020, August 2020, September 2020). Per the same pattern as Chs 7–10, position on `main` doesn't match chronology — commit 78 (pre-increment) is dated 2020-10-07, commit 79 (post-increment) is dated 2020-04-13, but `main` order is 78 → 79. Walked the case briefly in §11.3 prose.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The §8.5 generalized-lvalue comma loop is closed as of §11.2.** The §8.5 prose's prediction is now confirmed in print. The mechanism handles ten compound-assign operators plus four increment-form variants — twelve C operators canonicalize through `to_assign` and so through the comma extension. Load-bearing.
- **The fourth namespace (labels) is in.** As of §11.10, chibicc's namespace landscape has four flavors: variables/typedefs/enum-constants (the `vars` chain), tags (the `tags` chain), per-struct members (linear search inside each struct), and labels (function-scoped, separate from any `Scope`). Watch for further interactions when Ch 12 introduces initializer scopes (it shouldn't add a fifth namespace; struct-member-name lookup will be reused) and Ch 13's `extern` adds a fifth `VarAttr` flag through the same channel as `is_typedef`/`is_static`.
- **The `eval` function (§11.15) is small but load-bearing.** It will get more callers in Ch 12 (initializers — file-scope `int x = 1+2;`) and Ch 13 (`_Alignof` / `_Alignas`). Currently lives in `parse.c`, has no external callers.
- **Pre-factor-before-feature count remains at four.** §11 added zero formal pre-factors. (§11.1's for-loop locals are a pre-factor for *tests*, not for any future commit.)
- **Canonicalization-at-parse-time count is now eight.** Compound-assign (counted as one mechanism via `to_assign`, regardless of which `op=` it is) plus pre/post-increment (counted as one mechanism, with the postfix cast-back as a variant). The count's pace will slow from here — Ch 12 doesn't add any obvious canonicalizations.
- **The `ND_GOTO` reuse pattern.** `break` and `continue` reuse `ND_GOTO` rather than introducing distinct node kinds. Trick worth remembering when Ch 13 adds `return;` (bare return) — it might similarly reuse an existing node.
- **The temporary `get_number` → `const_expr` swap pattern.** §11.12 used a placeholder for case values; §11.15 replaced it. The pattern is unusual for chibicc, but if it recurs in Ch 12 (initializer-list-element-count or similar), watch for it.
- **The `>>` arithmetic-shift-only codegen.** When Ch 14 adds unsigned types, the §11.13 codegen will need to branch on signedness — `sar` for signed, `shr` for unsigned. Currently both arms of the size-8 dispatch emit `sar`; the dispatch itself is a no-op. Noted in the chapter prose; carry-forward errata candidate.
- **The `>>` codegen has a bug-shaped artifact.** Both arms of `if (node->ty->size == 8)` emit `sar %cl, %s` — the `if` is structurally unused. Errata candidate; not fixed in the prose.
- **The `_Bool` cast and `!` are mirror-images.** §10.12 emits `setne`; §11.5 emits `sete`. One-letter difference. The pairing is a small piece of mental shorthand worth keeping.
- **The structural difference between `ND_COND` and `&&`/`||` codegen.** All three short-circuit; all three need labels. `?:` chooses between two arms; `&&`/`||` produce `0` or `1`. Different shapes; noted in §11.14 prose.
- **Labels are unique by `new_unique_name`.** The `.L..NNNN` naming convention used by chibicc since §3 generates labels that won't collide across functions. Chapter 12 doesn't touch this; preserve.
- **The `is_typename`+`tok->next == ":"` lookahead in `compound_stmt`** is the load-bearing part of the label-vs-typedef resolution. The lookahead is local to one site; no broader mechanism is introduced. Watch for whether Ch 12's initializer-list parsing has similar conflicts.
- **The struct forward-decl-mutation pattern (§11.9).** `*sc->ty = *ty;` overwrites the previously-incomplete tag's type with the now-complete one. This is the first time chibicc's parser mutates an already-registered tag. Watch for when Ch 12 adds flexible array members (which extend struct types in-place) and Ch 13 adds global initializers.
- **The `for`-loop-local scope (§11.1)** opens at the `for` keyword, not at `{`. Same shape as the §8.1 block-scope mechanism but anchored at a different parser point.
- **The `array_of(ty, -1)` sentinel** for incomplete arrays will get more uses. Ch 12's `int x[] = {1, 2, 3};` will deduce the size from the initializer count and patch the type's `array_len` field after parsing. Watch for `array_len < 0` checks.
- **The Ch 9 errata-correction (§10.14)** is a stable closed loop. Chapter 11 doesn't revisit.
- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.
- **The `mov $0, %rax`** (variadic `%al`-zeroing) noted in Ch 5 §5.1. Pending footnote for revision pass.
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4. Errata candidate.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and now labels. Four errata candidates. (`goto a; a:; a:;` is silently accepted in chibicc; the second `a:` overwrites the first label entry.)

## Exit state

- `chapters/11-all-the-operators.md` drafted, ~12,260 words.
- Session 012 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 013 (Chapter 12 — Initializers, commits 97–115; nineteen commits, the densest arc in the compiler per the chapter mapping).
- CLAUDE.md status note will need updating (chapter count goes from "Ch 10 drafted" to "Ch 11 drafted").
