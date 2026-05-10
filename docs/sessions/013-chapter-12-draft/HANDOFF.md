# Handoff: Ch 12 done → proceed to Ch 13

**For:** the next claude session.
**From:** session 013.
**Status:** Ch 12 drafted (~10,900 words, nineteen commits, the densest commit arc in the compiler). Continue autonomously to Ch 13 (Linkage). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/013-chapter-12-draft/README.md`](README.md)** — what session 013 did, including the eleven-section structure, the no-interlude decision, the local-vs-global split as the chapter's central tension named in §12.6, the closure of §11.9's `array_of(ty, -1)` sentinel in §12.4 and §12.10, the `eval`/`eval2`/`eval_rval` split walked in §12.7, and the pre-factor-before-feature count update from four to six.
2. **[`docs/sessions/012-chapter-11-draft/HANDOFF.md`](../012-chapter-11-draft/HANDOFF.md)** — the previous handoff. Many standing notes still apply; the `Initializer` tree and the global-initializer machinery are now in place since §12.1 and §12.6/§12.7 respectively.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`12-initializers.md`](../../../chapters/12-initializers.md)** — the twelve chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 13 = commits 116–126.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; the chapter mapping flags a possible interlude on *static vs dynamic linking* drawn from this source.

## Chapter 13 scope

**Title (working):** *Linkage*.
**Commits:** 116–126 in chronological order on `main`. **Eleven commits — a return to a moderate-size chapter after Ch 11's twenty-one and Ch 12's nineteen.**
**Concept interlude:** Chapter mapping flags one — *static vs dynamic linking, from the JP book*. Decide whether the §13.1/§13.2 `extern` prose surfaces a need; if it does, run with it. If not, default-no.

| # | Hash | Subject |
|---|---|---|
| 116 | `006a45c` | Add extern |
| 117 | `2764745` | Handle extern declarations in a block |
| 118 | `9df5178` | Add `_Alignof` and `_Alignas` |
| 119 | `310a87e` | [GNU] Allow a variable as an operand of `_Alignof` |
| 120 | `319772b` | Add static local variables |
| 121 | `127056d` | Add compound literals |
| 122 | `30b3e21` | Add return that doesn't take any value |
| 123 | `eb85527` | Add static global variables |
| 124 | `ee252e6` | Add `do ... while` |
| 125 | `6a0ed71` | Align stack frame to 16 byte boundaries |
| 126 | `dcd4579` | Handle a function returning bool, char or short |

Eleven commits — about half the size of Ch 11 or Ch 12 by commit count. **Bundling is less aggressive than the last two chapters.** Rough proposal:

- **§13.1 — `extern` declarations** (commits 116, 117). Bundle. The first adds `extern` at file scope; the second extends it to block scope. The mechanism is one new `VarAttr` flag (`is_extern`) routed through `declspec`; the implementation is small but the *concept* — declaration-without-definition, link-time resolution, the difference from `static` — is the section's prose centerpiece. Substantive.
- **§13.2 — `_Alignof` and `_Alignas`** (commits 118, 119). Bundle. The first adds the operator and storage-class specifier; the second is a small GNU extension allowing `_Alignof(x)` (variable, not just type). Substantive — the alignment machinery interacts with §12.11's global `.align` emission.
- **§13.3 — Static local variables** (commit 120). The single-commit but conceptually-rich case: a `static` inside a function. Storage is global (`.data`/`.bss`) but scope is the function. The local-vs-global split from §12.6 *blurs* here — a static local goes through `gvar_initializer` and gets `.data` emission, but its visibility is function-scoped. Substantive.
- **§13.4 — Compound literals** (commit 121). `(struct s){1, 2}` — an initializer-list at expression position, not at declarator position. Reuses the `Initializer` tree from §12.1. Substantive.
- **§13.5 — Bare `return;`** (commit 122). One-line commit. `return` without an expression. The §11.11 `ND_GOTO`-reuse pattern hint from the previous handoff: does Rui reuse an existing node, or add a new one? Read the diff.
- **§13.6 — Static globals** (commit 123). The flip side of §13.3: `static` at file scope means *internal linkage* (no `.globl` directive). Small. Bundle with §13.3? Probably not — they're about different scopes and different mechanisms.
- **§13.7 — `do … while`** (commit 124). The fourth loop construct. Small.
- **§13.8 — 16-byte stack alignment** (commit 125). x86-64 ABI requirement: `%rsp` must be 16-byte aligned at every `call` instruction. Codegen change. Small but worth a section because it's the first time chibicc cares about ABI alignment.
- **§13.9 — Small return types** (commit 126). Functions returning `bool`/`char`/`short` need to truncate the return value to the declared type before storing into `%rax`. Codegen-side. Small.

That's nine sections from eleven commits. **Target chapter length: ~10,000–12,000 words.** Slightly smaller than Ch 12 — the commit count is lower, but `extern`/`_Alignof`/`_Alignas`/static-locals/compound-literals each get full-section treatment.

## Steps

1. `cd research/sources/chibicc && for h in 006a45c 2764745 9df5178 310a87e 319772b 127056d 30b3e21 eb85527 ee252e6 6a0ed71 dcd4579; do echo "===== $h ====="; git show --stat $h | head -8; done` to scan all eleven diffs.
2. Read each commit. Pay particular attention to:
   - **`006a45c`** (commit 116): `extern` at file scope. The `is_extern` `VarAttr` flag. Routed through `declspec`. Crucially, the variable is *declared but not defined* — `gvar_initializer` shouldn't run, no `.data`/`.bss` should be emitted, only the symbol is referenced.
   - **`2764745`** (commit 117): `extern` at block scope. `extern int x;` inside a function. The variable resolves to a global lookup; no local stack slot.
   - **`9df5178`** (commit 118): `_Alignof` and `_Alignas`. `_Alignof(int)` is a constant expression returning the type's alignment. `_Alignas(N)` is a declaration specifier — like `static`/`typedef`/`extern`, it routes through `VarAttr`. Interacts with the `.align N` emission from §12.11 (`var->ty->align` is now user-settable).
   - **`310a87e`** (commit 119): `_Alignof(x)` where `x` is a variable, not a type. GNU extension. One-line patch in the parser.
   - **`319772b`** (commit 120): static local variables. Storage allocated in `.data`/`.bss`; symbol synthesized via `new_unique_name` so it doesn't clash with other functions' statics. The function-scoped name lookup, file-scoped storage, and once-only initialization (chibicc may or may not handle this — read the commit).
   - **`127056d`** (commit 121): compound literals. `(struct s){1, 2}` at expression position. Likely a new helper in `unary` or `postfix` that calls `lvar_initializer`-shaped logic but synthesizes a fresh anonymous variable.
   - **`30b3e21`** (commit 122): bare `return;`. The handoff guess is that `return` without an operand reuses some existing node-kind shape. Verify.
   - **`eb85527`** (commit 123): static global variables. The `.globl` directive is suppressed; the symbol is local to the translation unit.
   - **`ee252e6`** (commit 124): `do … while`. New `ND_DO` or reuse of `ND_FOR`?
   - **`6a0ed71`** (commit 125): 16-byte stack alignment. `align_to(stack_size, 16)` somewhere in the prologue. Watch for the round-up math.
   - **`dcd4579`** (commit 126): small return-type truncation. `bool`/`char`/`short` returns. Likely a `cast` call inserted before the `mov %rax, ret`.
3. Read the destination state at `dcd4579` for `chibicc.h`, `parse.c`, `codegen.c`, all relevant test files.
4. Draft `chapters/13-linkage.md`. Likely 10,000–12,000 words.
5. Write `docs/sessions/014-chapter-13-draft/README.md`.
6. Write `HANDOFF.md` for session 015 (Chapter 14 — Variadics, signedness, qualifiers, commits 127–138; twelve commits including `va_start`, `signed`/`unsigned`, integer suffixes, and the qualifier soup).

## Voice / structure rules

Same as Ch 1–12:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — eleven rows; consider whether splitting into two tables makes sense (probably not — eleven rows is fine in one table).
- Diff format: lean toward inline diff fragments and quoted file snippets. The `_Alignof`/`_Alignas` and compound-literal sections may want larger code blocks.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **§13.1's `extern` mechanism is the chapter's intuition gap.** The C linkage model (external/internal/none) is not obvious to programmers who learned C from a "what compiles" angle rather than a linker angle. The §13.1 prose may want to walk it carefully — possibly as a concept interlude.
- **§13.3 (static locals) blurs the local-vs-global split from §12.6.** Static locals have local scope but global storage and initialization semantics. Name the blur explicitly.
- **§13.4 (compound literals) reuses the §12.1 Initializer tree.** Don't re-derive the tree machinery; reference §12.1 and walk the new piece (the synthesized anonymous variable).
- **§13.2 (`_Alignof`/`_Alignas`) interacts with §12.11.** The `.align N` emission now has a user-settable input. Walk the interaction.
- **§13.8 (16-byte alignment) is small but ABI-load-bearing.** Without it, calls into glibc's printf etc. trap on SSE-aligned argument loads. Worth a paragraph on *why* the alignment matters, not just *what* the codegen does.
- **§13.9 (small returns) interacts with §10.x's cast machinery.** A function declared `bool f()` should return `0` or `1` at the call boundary, not whatever happened to be in `%rax`. The fix is a cast-back at the return point, parallel to the §10.12 `_Bool` cast.
- **The `is_extern` flag on `VarAttr` is a third entry alongside `is_typedef` and `is_static`.** This was forecast in the Ch 11 and Ch 12 handoffs; closing the loop on the prediction is worth noting.

## Standing notes worth tracking across sessions

- **The `Initializer` tree** is the load-bearing data structure of Ch 12. Ch 13's compound literals (§13.4) reuse it.
- **The local-vs-global split** named in §12.6 is stable. Ch 13's static locals (§13.3) blur it; static globals (§13.6) are clearly on the global side; `extern` (§13.1) is global without storage.
- **The `eval`/`eval2`/`eval_rval` trio** (Ch 12 §12.7) is the constant-expression evaluator's stable shape. Ch 13's `_Alignof` (§13.2) probably adds an arm to `eval`.
- **The `Relocation` mechanism** (Ch 12 §12.7) is unchanged in Ch 13 — no new pointer-initializer features.
- **The `Member->idx` field** (Ch 12 §12.5) is reused for compound literals (Ch 13 §13.4).
- **The `is_flexible` flag** (Ch 12 §12.4 on `Initializer`, §12.10 on `Type`) is unchanged.
- **`copy_struct_type`** (Ch 12 §12.10) — watch for further uses.
- **`MIN`/`MAX` macros** (introduced Ch 12 §12.3) — watch for new callers.
- **Canonicalization-at-parse-time count is at eight.** Ch 13 may add a ninth (compound literals could be argued either way; static locals lower to globals at parse time).
- **Pre-factor-before-feature count is at six.** Ch 11 §11.9's incomplete-array sentinel has three consumers: Ch 12 §12.4, Ch 12 §12.10, and the §11.9 forward-decl case.
- **The fourth namespace (labels)** is unchanged. Ch 13 doesn't add a fifth.
- **The `is_typename` predicate** is unchanged in shape.
- **The `ND_GOTO` reuse pattern** (Ch 11 §11.11) is the precedent for *bare `return;`* in Ch 13 §13.5 — does Rui reuse `ND_RETURN` with a null `lhs`, or introduce a new node? Read the diff.
- **The struct-mutation-in-place pattern (§11.9)** does not extend further; Ch 12 §12.10 used `copy_struct_type` instead. Ch 13 doesn't add new struct mutations.
- **The VarAttr channel** (`is_typedef`, `is_static`) gets `is_extern` in Ch 13 §13.1 (and possibly an `align` field for `_Alignas` in §13.2).
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Tests are in C** as of Ch 8 §8.2.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17.
- **The argreg 8/16/32/64 split** is fully in place. Ch 13 doesn't touch it.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) rejects void-returning bodies. Errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4). Errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels. Four errata candidates.
- **`f()` and `f(void)` are accepted as identical** by chibicc. Errata or intentional simplification.
- **Empty brace initializer (`int x[3] = {};`)** is a chibicc-specific extension matching GCC.
- **`.bss` is the third assembly section.** Sections list: `.text`, `.data`, `.bss`.
- **`.align`** is emitted for every global (uniform; slightly wasteful for char globals).
- **The `>>` codegen quirk** (Ch 11 §11.13). Errata candidate.
- **The "more than 6 args silently miscompiles"** in Ch 5 §5.4. Errata candidate.
- **The `mov $0, %rax`** (variadic `%al`-zeroing) noted in Ch 5 §5.1. Pending footnote for revision pass.
- **Ch 1 errata list** unchanged.
- **The `ND_NULL_EXPR` seed-pattern** (Ch 12 §12.1) — small structural choice for accumulator loops.
- **The `rep stosb` pattern** (Ch 12 §12.2 for `ND_MEMZERO`) — first SIMD-adjacent x86 instruction. Noted.
- **The `unreachable()` macro** (Ch 10 §10.1) lives in `chibicc.h`. Used by `store_gp`, `declspec`, `write_buf`, and likely more in Ch 13.
- **Test file proliferation:** by Ch 12 the suite spans `test/arith.c`, `test/control.c`, `test/function.c`, `test/pointer.c`, `test/decl.c`, `test/string.c`, `test/struct.c`, `test/union.c`, `test/sizeof.c`, `test/initializer.c`, etc. Ch 13 may add `test/extern.c`, `test/literal.c`, etc.

## Acceptance criteria for Ch 13

- [ ] `chapters/13-linkage.md` exists, end-to-end readable.
- [ ] All eleven commits covered, grouped into ~9 sections.
- [ ] §13.1 walks the C linkage model (external/internal/none) and the `is_extern` `VarAttr` channel.
- [ ] §13.2 names the interaction with §12.11's `.align` emission.
- [ ] §13.3 names the local-vs-global split blur from §12.6.
- [ ] §13.4 reuses the §12.1 Initializer tree without re-derivation.
- [ ] §13.5 walks the `return;` mechanism (likely reuse of `ND_RETURN` with null `lhs`).
- [ ] §13.8 explains *why* 16-byte alignment matters (calling-convention requirement, SSE alignment).
- [ ] §13.9 names the parallel to §10.12 `_Bool` cast.
- [ ] Each commit has a `git checkout <full-hash>` opener.
- [ ] Voice matches Ch 1–12.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/014-chapter-13-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 015 (Chapter 14 — Variadics, signedness, qualifiers, commits 127–138).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/013-chapter-12-draft/HANDOFF.md  (this handoff)
2. docs/sessions/013-chapter-12-draft/README.md   (what session 013 did)
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
14. chapters/12-initializers.md                    (most recent chapter)
15. research/commits/chapter-mapping.md            (confirms Ch 13 scope)
16. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 13 (Linkage, commits 116–126) per the steps in
the handoff. Eleven commits — moderate bundling. End-of-session:
write your session dir under docs/sessions/014-chapter-13-draft/ with a
README and a HANDOFF for session 015 (Chapter 14 — Variadics,
signedness, qualifiers, commits 127–138).
```
