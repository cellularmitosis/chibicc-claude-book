# Handoff: Ch 11 done → proceed to Ch 12

**For:** the next claude session.
**From:** session 012.
**Status:** Ch 11 drafted (~12,260 words, twenty-one commits, the second-largest by commit count). Continue autonomously to Ch 12 (Initializers). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/012-chapter-11-draft/README.md`](README.md)** — what session 012 did, including the closure of the §8.5 generalized-lvalue comma loop in §11.2, the framing of labels as the fourth namespace in §11.10, the `ND_GOTO` reuse pattern for `break`/`continue` in §11.11, the temporary-`get_number`-then-`const_expr`-swap pattern in §11.12 and §11.15, and the canonicalization-at-parse-time count update from six to eight.
2. **[`docs/sessions/011-chapter-10-draft/HANDOFF.md`](../011-chapter-10-draft/HANDOFF.md)** — the previous handoff. Many standing notes still apply; `is_typename` is now coupled to the symbol table (since §10.6) and to a one-token lookahead (since §11.10).
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`11-all-the-operators.md`](../../../chapters/11-all-the-operators.md)** — the eleven chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 12 = commits 97–115.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.

## Chapter 12 scope

**Title (working):** *Initializers*.
**Commits:** 97–115 in chronological order on `main`. **Nineteen commits — the densest arc in the compiler per the chapter mapping.**
**Concept interlude:** None obvious. Possibly an interlude on "what an initializer-list grammar means" or "the local-vs-global initializer split" if the prose calls for it; default to no interlude.

| # | Hash | Subject |
|---|---|---|
| 97 | `22dd560` | Support local variable initializers |
| 98 | `ae0a37d` | Initialize excess array elements with zero |
| 99 | `a754732` | Skip excess initializer elements |
| 100 | `0d71737` | Add string literal initializer |
| 101 | `5b95533` | Allow to omit array length if an initializer is given |
| 102 | `e9d2c46` | Handle struct initializers for local variables |
| 103 | `aca19dd` | Allow to initialize a struct with other struct |
| 104 | `483b194` | Handle union initializers for local variables |
| 105 | `bbfe3f4` | Add global initializer for scalar and string |
| 106 | `eeb62b6` | Add struct initializer for global variable |
| 107 | `1eae5ae` | Handle union initializers for global variable |
| 108 | `efa0f33` | Allow parentheses in initializers to be omitted |
| 109 | `a58958c` | Allow extraneous braces for scalar initializer |
| 110 | `fde464c` | Allow extraneous comma at the end of enum or initializer list |
| 111 | `3d216e3` | Emit uninitialized global data to .bss instead of .data |
| 112 | `824543b` | Add flexible array member |
| 113 | `cd688a8` | Allow to initialize struct flexible array member |
| 114 | `7a1f816` | Accept `void` as a parameter list |
| 115 | `157356c` | Align global variables |

This is the chapter where chibicc's initializers go from "scalar at definition" to "the full C grammar of initializers." Nineteen commits is too many for one section per commit. **Bundle aggressively, again.** Rough proposal:

- **§12.1 — Local scalar and array initializers** (commit 97). The base case — `int x = 5;` and `int x[3] = {1, 2, 3};`. Sets up the `Initializer` data structure, the `initializer` and `lvar_initializer` parser, and the lowering to a series of assignments. Probably the chapter's largest section because the data structure is built here.
- **§12.2 — Initializer edge cases for arrays** (commits 98, 99). Bundle. Excess elements get zero (for `int x[5] = {1, 2}`); excess initializers are silently dropped (for `int x[2] = {1, 2, 3, 4}`). Both small.
- **§12.3 — String literal initializers** (commits 100). `char x[5] = "abc";` — one of C's stranger quirks where a string literal can initialize a `char[]`. Substantive.
- **§12.4 — Length-omitted arrays from initializers** (commit 101). `int x[] = {1, 2, 3};` deduces size 3. Closes the §11.9 `array_of(ty, -1)` sentinel — the parser can now patch the size after seeing the initializer. Substantive.
- **§12.5 — Struct and union local initializers** (commits 102, 103, 104). Bundle. Local-scope `struct point p = {1, 2};`. Commit 103 adds the *struct-from-struct* case (`struct point p = q;`). Commit 104 adds union initializers. Substantive.
- **§12.6 — Global initializers — scalar and string** (commit 105). The split: globals can't use the assignment-lowering trick, so a separate code path emits `.data` directives. Substantive.
- **§12.7 — Global struct and union initializers** (commits 106, 107). Bundle. Same shape as the local commits but routed through the `.data` emitter. Medium.
- **§12.8 — Initializer-list ergonomics** (commits 108, 109, 110). Bundle. Three small commits about syntax: omit braces (`int x[2] = 1;` is the same as `{1, 0}`); allow extraneous braces (`int x = {5};`); allow trailing comma in enum or initializer list. Each small; bundled.
- **§12.9 — `.bss` for uninitialized globals** (commit 111). One small commit: zero-initialized global data goes in `.bss` instead of `.data` (saves space in the object file). Small.
- **§12.10 — Flexible array members** (commits 112, 113). Bundle. `struct s { int n; int data[]; };` — the trailing array with no size. Initialization (commit 113) is the unusual case. Substantive.
- **§12.11 — `void` as a parameter list and global alignment** (commits 114, 115). Bundle? Two unrelated commits at the chapter's end. `f(void)` means "no parameters" (vs `f()` which is "unspecified parameters"). Global variables now get `.align` directives. Both small; bundled because they don't fit elsewhere.

That's ~11 sections from 19 commits. **Target chapter length: ~13,000–15,000 words.** Roughly Ch 11's size — slightly larger because the initializer data structure is the chapter's most complex single thing and needs walked-through detail.

## Steps

1. `cd research/sources/chibicc && for h in 22dd560 ae0a37d a754732 0d71737 5b95533 e9d2c46 aca19dd 483b194 bbfe3f4 eeb62b6 1eae5ae efa0f33 a58958c fde464c 3d216e3 824543b cd688a8 7a1f816 157356c; do echo "===== $h ====="; git show --stat $h | head -8; done` to scan all nineteen diffs.
2. Read each commit. Pay particular attention to:
   - **`22dd560`** (commit 97): the local initializer base case. The `Initializer` struct (parallel to `Type`?), the `initializer` recursive parser, and the `create_lvar_init` lowering. The chapter's load-bearing data structure lives here.
   - **`ae0a37d`, `a754732`** (commits 98, 99): excess-array-element handling. Likely small `if` checks in the array-initializer arm.
   - **`0d71737`** (commit 100): string literal initializers. The `char x[] = "abc"` case is special — the string isn't a single `ND_STR` node but a per-byte sequence. Watch for whether chibicc treats it as `{a, b, c, 0}` syntactically.
   - **`5b95533`** (commit 101): array-length-from-initializer. The `array_of(ty, -1)` sentinel from §11.9 gets its first user.
   - **`e9d2c46`, `aca19dd`, `483b194`** (commits 102, 103, 104): struct/union initializers. The struct-from-struct case (commit 103) is interesting because it's not a brace-list — it's `struct point p = q;` with an existing struct on the right.
   - **`bbfe3f4`, `eeb62b6`, `1eae5ae`** (commits 105, 106, 107): global initializers. Different code path — global initializers emit `.data` directives at compile time, not runtime assignments.
   - **`824543b`, `cd688a8`** (commits 112, 113): flexible array members. The trailing-array-with-no-size in struct definitions. Inititaliziation extends the struct's size.
   - **`7a1f816`** (commit 114): `f(void)` parameter list. C's `f()` and `f(void)` distinction is a parser quirk; the implementation is a one-line check.
3. Read the destination state at `157356c` (or shortly after) for `chibicc.h`, `parse.c`, `codegen.c`, all relevant test files. The Ch 12 surface is *narrow* (most changes live in `parse.c`'s initializer handlers and `codegen.c`'s `.data`/`.bss` emission) but *deep* (the initializer data structure is the chapter's centerpiece).
4. Draft `chapters/12-initializers.md`. Likely 13,000–15,000 words.
5. Write `docs/sessions/013-chapter-12-draft/README.md`.
6. Write `HANDOFF.md` for session 014 (Chapter 13 — Linkage, commits 116–126; eleven commits including `extern`, `_Alignof`/`_Alignas`, static locals, compound literals, do-while, return-without-value, 16-byte stack alignment).

## Voice / structure rules

Same as Ch 1–11:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — nineteen rows; consider splitting into two tables by theme (local-scope vs. global-scope) as Ch 10 and Ch 11 did.
- Diff format: lean toward inline diff fragments and quoted file snippets. The `Initializer` struct (probably) is a candidate for full-snippet display the first time it appears.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **Bundle aggressively.** Nineteen commits won't fit one-per-section at any reasonable chapter length.
- **§12.1's data structure walk is the chapter's anchor.** The `Initializer` struct (whatever it ends up being called) is referenced by every later section. Walk it carefully the first time.
- **The local-vs-global split is the chapter's central tension.** Local initializers lower to assignments at runtime; global initializers emit data directives at compile time. The split shows up as two parallel code paths. Name the split explicitly.
- **String literals as initializers are weird.** `char x[5] = "abc"` is *syntactically* a string literal but *semantically* equivalent to `char x[5] = {'a', 'b', 'c', 0, 0}`. Walk the expansion.
- **Flexible array members (§12.10) extend the §11.9 incomplete-struct mechanism.** A flexible array is essentially "the last member is an incomplete array of some element type." Watch for whether chibicc reuses the `array_of(ty, -1)` sentinel or introduces a new flag.
- **`void` as a parameter list (§12.11) is a one-line check** but it's *structurally distinct* from "no parameters." `f()` accepts any arguments (legacy K&R); `f(void)` accepts none. Pin down the test that distinguishes them.
- **Global alignment (§12.11) is small but interacts with §12.6's `.data` emission.** The `.align N` directive precedes each global. Walk the emitted assembly.
- **The `.bss` shift (§12.9) is a small, isolated commit** but worth a section because it's the first time chibicc emits `.bss` (vs. `.data`).
- **Watch for the `Initializer` data structure's relationship to `Type`.** If chibicc parallels them (one-to-one, one struct per `Type` shape), name the parallel. If it's a more general AST-of-initializer, walk that.

## Standing notes worth tracking across sessions

- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.
- **The `mov $0, %rax`** (variadic `%al`-zeroing) noted in Ch 5 §5.1. Pending footnote for revision pass.
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4. Errata candidate.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels. Four errata candidates.
- **Block scope is established** as of Ch 8 §8.1. Tag scope from Ch 9 §9.4. Typedef-name scope (sharing `vars`) from Ch 10 §10.6. Enum-constant scope (also `vars`) from Ch 10 §10.14. Label namespace (function-scoped, separate) from Ch 11 §11.10. Watch for a fifth Ch 12 addition (probably none — initializer scopes likely reuse block scope).
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Tests are in C** as of Ch 8 §8.2. New language features get tests in `test/<feature>.c`.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17.
- **The comma operator's generalized-lvalue extension** (Ch 8 §8.5) is the load-bearing mechanism for compound assignment, closed in Ch 11 §11.2. By Ch 12 the loop is closed.
- **Canonicalization-at-parse-time count is now eight** (compound-assign + pre/post-increment, plus the six from before Ch 11). Ch 12's initializers are a *form* of canonicalization (initializer → assignments) but the chapter conventionally calls them "initializer lowering," not "canonicalization." Decide whether to count them as a ninth instance — if so, name the call-out in §12.1.
- **Pre-factor-before-feature is a four-instance named pattern.** Ch 12's `array_of(ty, -1)` reuse from Ch 11 §11.9 is an instance — the sentinel was set up in Ch 11 and is consumed in Ch 12 §12.4 and §12.10. Possibly count this as a fifth.
- **The argreg 8/16/32/64 split** is fully in place. Ch 12 doesn't touch it.
- **The `is_typename` helper is a context-sensitive predicate** as of Ch 10 §10.6 + Ch 11 §11.10. Ch 12's parser additions probably don't extend it; initializer lists don't introduce a typename ambiguity.
- **The cast machinery** is Ch 10's. Ch 12's initializers likely use `new_cast` for type-promoting initializers (`int x = (long)5;`).
- **The `format` helper landed in Ch 7 §7.3.** Workhorse going forward.
- **The trailing-newline guarantee in `read_file`** (Ch 7 §7.6) protects line-comment skipping.
- **The lookahead-by-probe pattern** is now a four-instance family: §7.1, §10.3, §10.7, §10.6, §11.10.
- **The everything-fits-in-rax codegen invariant** continues. Ch 12 doesn't touch.
- **The `array_of(ty, -1)` sentinel** for incomplete arrays (Ch 11 §11.9) gets its first real consumer in Ch 12 §12.4 (length-from-initializer).
- **The `eval` function** (Ch 11 §11.15) gets more callers in Ch 12 (compile-time-constant initializers — `int x = 1+2;` at file scope).
- **The struct forward-decl mutation pattern** (`*sc->ty = *ty;` in Ch 11 §11.9) repeats in Ch 12 §12.4 — array length is patched in place after the initializer is parsed.
- **The `unreachable()` macro** (Ch 10 §10.1) lives in `chibicc.h`. Used by `store_gp`, `declspec`, and possibly more in Ch 12.
- **The `VarAttr` channel** (Ch 10 §10.6, extended in §10.15) currently carries `is_typedef` and `is_static`. Ch 13's `extern` adds a third flag.
- **The `ND_GOTO` reuse for `break`/`continue`** (Ch 11 §11.11) is a chapter-internal trick worth remembering. Ch 13's bare `return;` may use a similar reuse.
- **Labels are the fourth namespace** as of Ch 11 §11.10. Watch for a fifth in Ch 12 (probably none).

## Acceptance criteria for Ch 12

- [ ] `chapters/12-initializers.md` exists, end-to-end readable.
- [ ] All nineteen commits covered, grouped into ~11 sections (or fewer with bundling).
- [ ] §12.1 walks the `Initializer` data structure carefully — this is the chapter's anchor.
- [ ] §12.4 names the closure of the §11.9 `array_of(ty, -1)` sentinel.
- [ ] §12.5 covers the struct-from-struct initializer case explicitly.
- [ ] §12.6 names the local-vs-global initializer split.
- [ ] §12.10 (flexible array) explains the relationship to §11.9 incomplete arrays.
- [ ] §12.11 distinguishes `f()` from `f(void)`.
- [ ] Each commit has a `git checkout <full-hash>` opener.
- [ ] Voice matches Ch 1–11.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/013-chapter-12-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 014 (Chapter 13 — Linkage, commits 116–126).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/012-chapter-11-draft/HANDOFF.md  (this handoff)
2. docs/sessions/012-chapter-11-draft/README.md   (what session 012 did)
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
13. chapters/11-all-the-operators.md               (most recent chapter)
14. research/commits/chapter-mapping.md            (confirms Ch 12 scope)
15. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 12 (Initializers, commits 97–115) per the steps in
the handoff. Nineteen commits — bundle aggressively. End-of-session:
write your session dir under docs/sessions/013-chapter-12-draft/ with a
README and a HANDOFF for session 014 (Chapter 13 — Linkage, commits
116–126).
```
