# Handoff: Ch 15 done → proceed to Ch 16

**For:** the next claude session.
**From:** session 016.
**Status:** Ch 15 drafted (~11,995 words, eleven commits, the floating-point chapter — literals, types, casts, comparison, arithmetic, control flow, function calls and definitions through the System V two-register-file convention, default argument promotion, variadic floats closing the §14.2 spill area's FP half, parallel `eval_double` for constant expressions, and `long double` collapsed to `double`). Continue autonomously to Ch 16 (The compiler driver). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/016-chapter-15-draft/README.md`](README.md)** — what session 016 did, including the eleven-section structure (no bundling), the no-interlude decision (the IEEE 754 walk *is* §15.1's literal mechanism, the SSE/XMM walk lives split across §15.6/§15.7), the §15.2 cast-table extension to 10×10 with the `u64f64` unsigned-to-double dance, the §15.3 NaN-correct equality dance with the parity flag, the §15.6 recursive `push_args` pattern (counted as the chapter's one pre-factor, taking the count to seven), the §15.9 `fp * 8 + 48` formula closing the §14.2 loop, the anonymous-global-forecast-was-wrong owning (§15.1 inlines float literals as integer immediates bit-cast into XMM via `%rax`), the psABI conformance thread continuation (now eight corrections after Ch 13 §13.8/§13.9 + Ch 14 §14.1/§14.2/§14.8 + Ch 15 §15.6/§15.7/§15.8/§15.9), and the chapter's ~11,995 word length (right at upper edge of target).
2. **[`docs/sessions/015-chapter-14-draft/HANDOFF.md`](../015-chapter-14-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply. The Initializer tree, anonymous-global pattern (still used for strings and compound literals; not used for float literals), and the `__va_area__` magic-name trick are unchanged after Ch 15.
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`15-floating-point.md`](../../../chapters/15-floating-point.md)** — the fifteen chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 16 = commits 150–157.
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; Ch 16 (compiler driver) probably doesn't have an obvious interlude candidate, but the cc1-as-separate-binary split or the GCC-driver-shape comparison might be candidates if the prose surfaces a need.

## Chapter 16 scope

**Title (working):** *The compiler driver*.
**Commits:** 150–157 in chronological order on `main`. **Eight commits — small, mostly driver-shape changes with two function-pointer detours.**
**Concept interlude:** Probably not. If §16.X's driver split surfaces a need to explain the GCC driver-vs-cc1 separation, run with it as a brief interlude. Default-no on a separate interlude.

| # | Hash | Subject |
|---|---|---|
| 150 | `5d15431` | Add stage2 build |
| 151 | `d06a8ac` | Add function pointer |
| 152 | `c5953ba` | Decay a function to a pointer in the func param context |
| 153 | `53e8103` | Add usual arithmetic conversion for function pointer |
| 154 | `f3d9613` | Split cc1 from compiler driver |
| 155 | `140b433` | Run "as" command unless -S is given |
| 156 | `b833cd0` | Accept multiple input files |
| 157 | `8b726b5` | Run "ld" unless -c is given |

Eight commits. **Bundling is feasible.** Rough proposal:

- **§16.1 — Stage-2 build** (commit 150). The Makefile gains a stage-2 target that compiles chibicc with itself (well, with the host compiler, then re-compiles with the just-built chibicc). This isn't self-hosting yet (that's Ch 17 commit 197) — chibicc can't compile its own preprocessor inputs because it has no preprocessor. The stage-2 build runs the host preprocessor first, then chibicc on the preprocessed output. Read carefully — this is a Makefile change with no real chibicc-source change, but the *structure* it sets up is what makes self-hosting possible later.
- **§16.2 — Function pointers** (commits 151, 152, 153). Three commits on function pointers. Probably bundle. The first introduces the type and basic syntax (`int (*fp)(int);`); the second is the function-decay rule (a function used in expression context decays to a pointer-to-function); the third is the usual-arithmetic-conversion rule for function pointers (which mostly means "function-pointer types are compatible the way pointer types are, but with stricter type matching").
- **§16.3 — Splitting cc1 from the driver** (commit 154). The compiler proper (the existing `chibicc` binary) gets renamed to `cc1`; a new `chibicc` driver wraps it. The driver invokes `cc1` via fork/exec. This is the structural split that Ch 16 is named for.
- **§16.4 — Running `as`** (commit 155). The driver learns to run the system assembler on `cc1`'s output unless `-S` is passed. With this, `chibicc input.c` produces `input.o` instead of `input.s`. The driver also handles `-o` for output naming.
- **§16.5 — Multiple input files** (commit 156). The driver accepts a list of input files and runs the cc1+as pipeline for each. Probably also extends `-o` to "output to that file if exactly one input, error otherwise."
- **§16.6 — Running `ld`** (commit 157). The driver learns to invoke the linker unless `-c` is passed. With this, `chibicc input.c` produces an executable. The driver constructs the link command line — including paths to crt1.o, libc, etc.

That's six sections from eight commits (commits 151/152/153 bundled into §16.2). **Target chapter length: ~7,000–9,000 words.** Could run shorter — driver-shape changes are mostly straightforward shell-out logic; the function-pointer commits are the only ones with substantive parser/codegen content.

## Steps

1. `cd research/sources/chibicc && for h in 5d15431 d06a8ac c5953ba 53e8103 f3d9613 140b433 b833cd0 8b726b5; do echo "===== $h ====="; git show --stat $h | head -8; done` to scan all eight diffs.
2. Read each commit. Pay particular attention to:
   - **`5d15431`** (commit 150): Stage-2 build. Likely just a Makefile change (`stage2`/`stage3` targets that re-compile chibicc using the previously-built chibicc binary). Read the Makefile diff for the exact pipeline. The chibicc-source itself probably doesn't change.
   - **`d06a8ac`** (commit 151): Function pointers. Function-pointer types as `int (*fp)(int)`. The parser's declarator already handles parenthesized declarators (since Chapter 4 or so); the change is in how the declared name resolves to a function-typed identifier vs a function-pointer-typed one. Probably new arms in `funcall` to handle calling through a pointer (`call *%rax` instead of `call name`).
   - **`c5953ba`** (commit 152): Function-decay. Functions used in expression context decay to pointers. Like array decay (Ch 6), but for `TY_FUNC`. Probably a small `add_type` arm.
   - **`53e8103`** (commit 153): Usual arithmetic conversion for function pointers. Probably enables comparing two function pointers, and conditional expressions where one arm is a function and the other is a function pointer.
   - **`f3d9613`** (commit 154): Split cc1 from driver. The biggest commit in the chapter. The existing `main.c` gets renamed or split — `chibicc.c`/`main.c` becomes `cc1.c` (the compiler proper), and a new `main.c` (or similar) becomes the driver. The driver does argument parsing, then `fork`/`exec`s `cc1` for each translation unit. Read the new `main.c`'s structure carefully — it's the skeleton that grows for the rest of the chapter.
   - **`140b433`** (commit 155): Run `as`. The driver appends another fork/exec to invoke `as` on `cc1`'s `.s` output. Probably uses temp files (`/tmp/chibicc-XXXXXX.s` and `/tmp/chibicc-XXXXXX.o`).
   - **`b833cd0`** (commit 156): Multiple input files. The driver loops over input files. Argument parsing extends.
   - **`8b726b5`** (commit 157): Run `ld`. The driver constructs a `ld` command line with crt1.o, crti.o, libc.so, crtn.o, etc. The exact set of paths is Linux/glibc-specific. Read the path-construction logic carefully — it's brittle but it's also the moment chibicc actually links real binaries.
3. Read the destination state at `8b726b5` for `chibicc.h`, `parse.c`, `codegen.c`, `tokenize.c`, `type.c`, all relevant test files, `main.c`/`cc1.c`, and the Makefile.
4. Draft `chapters/16-the-compiler-driver.md`. Likely 7,000–9,000 words.
5. Write `docs/sessions/017-chapter-16-draft/README.md`.
6. Write `HANDOFF.md` for session 018 (Chapter 17 — A preprocessor from scratch, commits 158–197; ~40 commits, the longest single arc in the book).

## Voice / structure rules

Same as Ch 1–15:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present tense for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — eight rows; one table.
- Diff format: lean toward inline diff fragments and quoted file snippets. The driver-split commit (§16.3) and the link-command-construction commit (§16.6) will want larger code blocks.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **§16.1's stage-2 build is a stepping stone, not self-hosting.** Don't claim self-hosting until commit 197 (Ch 17). The §16.1 prose should name the gap clearly: chibicc has no preprocessor, so stage-2 uses the host preprocessor first.
- **§16.2's three commits are tightly related.** Bundle them. The function-pointer story is *type → decay → arithmetic-conversion*, a three-step shape that mirrors the array story (Ch 6). The bundling decision should be named in the section's framing.
- **§16.3's cc1-vs-driver split is the chapter's structural anchor.** This is the moment chibicc starts looking like a real compiler. The new driver is small and shell-script-shaped (fork/exec/wait); the cc1 binary is the existing compiler with its main function trimmed down. Walk the split carefully.
- **§16.6's link command is brittle and Linux/glibc-specific.** The exact paths (`/usr/lib/x86_64-linux-gnu/crt1.o` etc.) depend on the distro. Rui hardcodes paths that work on the Ubuntu CI image. Note this; it's a real-world constraint not a chibicc design flaw.
- **The driver doesn't replace the test harness.** The Chapter 8 §8.2 test harness (host-cc-as-preprocessor + chibicc + host-cc-as-linker) continues to work. The new driver is a separate path — `chibicc input.c -o output` — that didn't exist before. The test harness collapses into the driver in Ch 17 once the preprocessor lands.
- **Function-pointer codegen is `call *%rax`, not `call name`.** §16.2 should name this distinction. The dispatch in `funcall` checks whether the callee is a named function or a computed function pointer.
- **The stage-2 Makefile target is the canary for self-hosting.** Watch how §16.1's stage-2 evolves through Ch 17 — by Ch 17 §17.6 it should produce a chibicc that compiles its own preprocessor.

## Standing notes worth tracking across sessions

- **The `Initializer` tree** is unchanged. Likely unchanged in Ch 16.
- **The local-vs-global split** is stable.
- **The `eval`/`eval2`/`eval_rval`/`eval_double` quartet** is the constant-expression evaluator. Ch 16 might add function-pointer arms (probably to `eval2`'s pointer-or-label path, since function-pointers compute addresses).
- **The `Relocation` mechanism** — no new uses since Ch 13. Ch 16 might use it for function-pointer initializers.
- **The anonymous-global pattern** (`new_anon_gvar`) — no new uses in Ch 14 or Ch 15 (Ch 15 chose inline-immediate for floats). Ch 16 unlikely to use.
- **The `is_static` default in `new_gvar`** — unchanged.
- **The `is_definition` flag on `Obj`** — used today only by `extern`. No new uses since Ch 13.
- **The `is_unsigned` flag on `Type`** — irrelevant to function pointers (the kind axis, not the signedness axis).
- **The `__va_area__` magic name** — unchanged.
- **The register-save-area layout** — unchanged. The 8-byte-vs-ABI-16-byte stride for FP slots is still non-conforming (errata candidate).
- **The argreg 8/16/32/64 split for integers and `%xmm0`–`%xmm7` for floats** — unchanged.
- **The `Member->idx` field** (Ch 12 §12.5) — no new uses since.
- **The `is_flexible` flag** — unchanged.
- **`copy_struct_type`** — no new uses since Ch 12.
- **`MIN`/`MAX` macros** — no new uses since Ch 12.
- **`is_numeric` predicate** (Ch 15 §15.4) — used by `new_add`/`new_sub`. Ch 16 might add callers if function-pointer arithmetic surfaces.
- **Canonicalization-at-parse-time count is at eight.** Ch 15 added zero. Ch 16 might add one if function-decay is implemented as parse-time AST rewriting (Ch 6 array-decay was a `add_type` rule, not parse-time).
- **Pre-factor-before-feature count is at seven.** Ch 15's recursive `push_args` was the new entry. Ch 16 might add — possibly the cc1-vs-driver split (§16.3) is in spirit a pre-factor (the existing `main` shape couldn't have been extended to multi-stage compilation), but it's also the feature itself.
- **The fourth namespace (labels)** is unchanged. Ch 16 doesn't add a fifth.
- **The `is_typename` predicate** is unchanged. Ch 16 unlikely to add (function-pointer types are constructed from existing keywords).
- **The VarAttr channel** has four fields. Ch 15 didn't grow it. Ch 16 unlikely to grow it.
- **The `ND_NULL_EXPR` seed-pattern** (Ch 12 §12.1) — no new uses since Ch 12.
- **The `rep stosb` pattern** (Ch 12 §12.2) — no new uses since Ch 12.
- **The `unreachable()` macro** lives in `chibicc.h`. Used by `store_gp`, `declspec`, `write_buf`, and (Ch 15) `store_fp`. Ch 16 unlikely to add.
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Tests are in C** as of Ch 8 §8.2.
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) collapses in Ch 17.
- **The `Obj->tok` field** added in Ch 14 §14.11 still has no readers. Ch 16 might add.
- **The `Type->name_pos` field** (Ch 14 §14.11) — no new uses since.
- **The `>>` codegen quirk** (Ch 11 §11.13) — partially repaired. Ch 15 §15.4 closed one more case (bitwise-op width). Ch 16 unlikely to touch.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) — errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4) — errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels — four errata candidates.
- **`f()` and `f(void)` are accepted as identical** — errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension.
- **`.bss` is the third assembly section.**
- **`.align`** is emitted for every global.
- **The "more than 6 integer args silently miscompiles"** — errata candidate.
- **The "more than 8 FP args silently miscompiles"** (Ch 15 §15.6) — errata candidate, sibling.
- **The `mov $0, %rax`** — closed loop; still pessimistic; plus the variadic-FP-call wrongness (Ch 15 §15.6) — errata candidate.
- **psABI conformance thread:** Ch 13 §13.8/§13.9 + Ch 14 §14.1/§14.2/§14.8 + Ch 15 §15.6/§15.7/§15.8/§15.9 = eight corrections. Ch 16 (driver-shape changes) probably doesn't add to this thread.
- **The `fp_offset = fp * 8 + 48` non-conforming stride** (Ch 15 §15.9) — errata candidate.
- **`long double` is `double`** (Ch 15 §15.11) — errata candidate. Real `long double` would need new kind, x87 stack, separate calling convention.
- **The default-argument-promotion gap for chars and shorts** (Ch 15 §15.8) — errata candidate.
- **Float literals are inlined as integer-immediate-bit-cast** (Ch 15 §15.1), not anonymous-global. Worth noting if Ch 16 has a literal-related decision.
- **Ch 1 errata list** unchanged.
- **The cast/compound-literal disambiguator** (Ch 13 §13.4) — same lookahead pattern. No new uses.
- **The cast table is 10×10.** Ch 16 unlikely to extend (function pointers don't need cast-table cells; pointer-to-pointer casts are NULL/identity).

## Acceptance criteria for Ch 16

- [ ] `chapters/16-the-compiler-driver.md` exists, end-to-end readable.
- [ ] All eight commits covered, grouped into ~5–6 sections (with the function-pointer trio bundled as §16.2).
- [ ] §16.1 names the stage-2 build as a stepping stone, not self-hosting; names the preprocessor gap.
- [ ] §16.2 covers the function-pointer trio together — type, decay, arithmetic conversion. Mirrors Chapter 6's array-decay arc structurally.
- [ ] §16.3 walks the cc1-vs-driver split as the chapter's structural anchor.
- [ ] §16.4 walks the `as` invocation including temp-file handling.
- [ ] §16.5 walks multiple-input-file handling and `-o` semantics.
- [ ] §16.6 walks the `ld` invocation including the brittle Linux/glibc path hardcoding. Names this as a real-world constraint, not a chibicc flaw.
- [ ] Each commit has a `git checkout <full-hash>` opener. The bundled §16.2 has multiple openers.
- [ ] Voice matches Ch 1–15.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] `docs/sessions/017-chapter-16-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 018 (Chapter 17 — A preprocessor from scratch, commits 158–197; the longest arc, ~40 commits with sub-section grouping).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/016-chapter-15-draft/HANDOFF.md  (this handoff)
2. docs/sessions/016-chapter-15-draft/README.md   (what session 016 did)
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
14. chapters/12-initializers.md
15. chapters/13-linkage.md
16. chapters/14-variadics-signedness-qualifiers.md
17. chapters/15-floating-point.md                  (most recent chapter)
18. research/commits/chapter-mapping.md            (confirms Ch 16 scope)
19. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 16 (The compiler driver, commits 150–157) per the
steps in the handoff. Eight commits — function-pointer trio bundled,
the rest mostly driver-shape changes. End-of-session: write your
session dir under docs/sessions/017-chapter-16-draft/ with a README
and a HANDOFF for session 018 (Chapter 17 — A preprocessor from
scratch, commits 158–197; the book's longest single arc, ~40 commits).
```
