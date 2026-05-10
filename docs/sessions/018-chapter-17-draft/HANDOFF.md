# Handoff: Ch 17 done → proceed to Ch 18

**For:** the next claude session.
**From:** session 018.
**Status:** Ch 17 drafted (~17,134 words, forty commits, the C preprocessor from a do-nothing seam to self-hosting). Continue autonomously to Ch 18 (The full ABI, commits 198–220 — twenty-three commits covering stack-passed args/parameters, struct-by-value, variadic-with-more-than-6-fixed-params, `va_copy`, function-deref no-op, pp-numbers, `-D`/`-U`, bitfields, buffered output, ignored flags, `-Wall`-clean self-build, 16-byte array alignment, implicit `return 0` in `main`, anonymous struct/union). Don't pause for review.

## Read these first, in order

1. **[`docs/sessions/018-chapter-17-draft/README.md`](README.md)** — what session 018 did, including the six-section structure (per-commit subsections in §17.1 through §17.4, four sub-bundles in §17.5, single-section §17.6), the concept interlude on the no-rescan rule (lands in §17.3.4, ~400 words, named "A concept interlude — why the paint must terminate", names Prosser's algorithm and the blue-paint metaphor), the "chibicc compiles chibicc" framing for §17.6, the `cpp-as-separate-binary forecast was wrong` note resolved, the five new errata candidates added to the catalog, the pre-factor count incremented to eight, the canonicalization-at-parse-time count unchanged at eight, the psABI conformance count unchanged at nine.
2. **[`docs/sessions/017-chapter-16-draft/HANDOFF.md`](../017-chapter-16-draft/HANDOFF.md)** — the previous handoff. Standing notes still apply with Ch 17 updates folded in (see §18 README for the running list).
3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`17-a-preprocessor-from-scratch.md`](../../../chapters/17-a-preprocessor-from-scratch.md)** — the seventeen chapters drafted. Match the register.
4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** — confirms Ch 18 = commits 198–220 (23 commits, scoped to "the full ABI" plus a few unrelated polish commits).
5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** — quotable Rui quotes. Ch 18's commits are mostly bugfixes-and-completions; less likely to have especially apt commit-message material.
6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** — JP TOC; the SysV ABI's struct-passing rules might be a candidate for a concept interlude in §18.2 (struct-by-value), but default-no — the algorithm is intricate but its *why* is "the SysV ABI says so," which doesn't repay an interlude.

## Chapter 18 scope

**Title (working):** *The full ABI*.
**Commits:** 198–220 in chronological order on `main`. **Twenty-three commits** — back to a manageable size after Ch 17's forty-commit arc.
**Concept interlude:** Likely no. The chapter's substance is "fill in the gaps the ABI work has been hiding under." A concept interlude on *the SysV AMD64 ABI's eightbyte classification algorithm* is tempting — it's a clever piece of design — but the algorithm only lands in chibicc as the function-by-function classification implementation, which §18.2 prose can carry inline. Default no, conditional on whether §18.2's prose feels overstuffed.

| # | Hash | Subject |
|---|---|---|
| 198 | `b29f052` | Support passed-on-stack arguments |
| 199 | `9021f7f` | Support passed-on-stack parameters |
| 200 | `5e0f8c4` | Allow struct parameter |
| 201 | `d63b1f4` | Allow struct argument |
| 202 | `c72df1c` | Allow to call a fucntion returning a struct |
| 203 | `d7bad96` | Allow to define a function returning a struct |
| 204 | `b6d3cd0` | Allow variadic function to take more than 6 parameters |
| 205 | `603de50` | Add va_copy() |
| 206 | `e0b5da3` | Dereferencing a function shouldn't do anything |
| 207 | `3f2c2d5` | Tokenize numeric tokens as pp-numbers |
| 208 | `fc69f5c` | Add -D option |
| 209 | `be8b6f6` | Add -U option |
| 210 | `cc852fe` | Add bitfield |
| 211 | `441a89b` | Support global struct bitfield initializer |
| 212 | `54c2b3b` | Handle op=-style assignments to bitfields |
| 213 | `17ea802` | Handle zero-width bitfield member |
| 214 | `c302a96` | Do not allow to obtain an address of a bitfield |
| 215 | `2bdc6b8` | Write to an in-memory buffer before writing to an actual output file |
| 216 | `b1fdddf` | Ignore -O, -W and -g and other flags |
| 217 | `2c91da5` | Turn on -Wall compiler flag and fix compiler warnings |
| 218 | `5257ee0` | Make an array of at least 16 bytes long to have alignment of at least 16 bytes |
| 219 | `9c36dd7` | Make "main" to implicitly return 0 |
| 220 | `c3075b3` | Add anonymous struct and union |

Twenty-three commits. The natural section grouping:

- **§18.1 — Stack-passed args and parameters** (commits 198–199). Two commits. Closes the "more than 6 integer args silently miscompiles" errata from Ch 5 §5.4 and the "more than 8 FP args silently miscompiles" errata from Ch 15 §15.6. Codegen learns to push extra args onto the stack and pop them after the call; parameter handling learns to read them from the caller's stack frame.
- **§18.2 — Struct-by-value** (commits 200–203). Four commits. Struct as parameter, struct as argument, struct as return type (caller side and callee side). This is the SysV AMD64 ABI's *eightbyte classification* — a struct ≤16 bytes splits into one or two 8-byte chunks, each classified as INTEGER or SSE based on its members; chunks pass via registers (`%rdi`/`%rsi`/`%xmm0`/`%xmm1`/etc.) or memory based on classification. Larger structs pass by hidden pointer. Returns work the same way (with a hidden first parameter for memory-class returns).
- **§18.3 — Variadic with stack-resident fixed parameters; va_copy** (commits 204–205). Two commits. Closes the "variadic functions whose fixed parameters spill onto the stack don't work yet" gap from Ch 14. `va_copy(dst, src)` lets the user duplicate a `va_list`, which is a one-line copy because `va_list` is `__va_elem[1]` (an array, so assignment is illegal but pointer-style copy via `*dst = *src` works).
- **§18.4 — Small completion commits** (commits 206–209). Four commits. *Dereferencing a function* (`(*fp)(x)` should be a no-op when `fp` is `int(*)(int)` — it stays a function-pointer). *Pp-numbers* (preprocessor numbers, the C-spec lexical category that lets `0x1.5p-3` lex as one token). *`-D` option* (command-line `#define`). *`-U` option* (command-line `#undef`).
- **§18.5 — Bitfields** (commits 210–214). Five commits. The bitfield arc: `int x:3;` syntax in struct declarations; storage-allocation rules (consecutive bitfields share storage; alignment forces a new unit when needed); the read/write codegen (mask-and-shift); global initializers; op-assign (`x.bf += 1` requires loading, modifying, storing back); zero-width bitfield (forces alignment-to-next-storage-unit); address-of restriction (a bitfield doesn't have an address — `&s.bf` is illegal).
- **§18.6 — Polish and tail** (commits 215–220). Six commits. Buffered output (write the assembly into an `open_memstream` buffer first, then dump). Ignoring `-O`/`-W`/`-g`/`-MD`/`-pipe` and other GCC flags (with a TODO to actually honor them). `-Wall` self-build (chibicc compiles cleanly under `gcc -Wall`, which catches uninitialized variables and other host-cc-detectable bugs). 16-byte alignment for arrays >= 16 bytes (an x86-64 ABI nicety that helps SSE intrinsics). `main` implicitly returns 0 (per the C standard's special case). Anonymous struct/union (`struct { struct { int a, b; }; int c; }` lets `s.a` reach the inner anonymous member).

That's six sections from twenty-three commits. **Target chapter length: ~10,000–13,000 words.** Likely closer to 12K — the bitfield arc is detailed and §18.2's eightbyte classification is involved.

## Steps

1. `cd research/sources/chibicc && for h in b29f052 9021f7f 5e0f8c4 d63b1f4 c72df1c d7bad96 b6d3cd0 603de50 e0b5da3 3f2c2d5 fc69f5c be8b6f6 cc852fe 441a89b 54c2b3b 17ea802 c302a96 2bdc6b8 b1fdddf 2c91da5 5257ee0 9c36dd7 c3075b3; do echo "===== $h ====="; git show --stat $h | head -10; done` to scan all 23 diffs.
2. Read each commit. Pay particular attention to:
   - **§18.1's stack-passed-args codegen (198–199):** The push-extras-then-call sequence is x86-64's standard: fixed-args go in registers, overflow-args go on the stack in left-to-right order (so caller and callee agree which is which). The callee reads them from positive offsets relative to `%rbp`. Note the alignment: x86-64 requires `%rsp` 16-byte-aligned at the call instruction.
   - **§18.2's eightbyte classification (200–203):** Read carefully. The SysV ABI's algorithm is: walk the struct's members; for each 8-byte chunk, take the *worst* class among the members in that chunk (memory > integer > SSE); if any chunk is memory, the whole struct is memory. The implementation lives in a small helper function (probably `classify_param` or similar). The 16-byte limit is fundamental — at 17 bytes, the whole thing goes to memory.
   - **§18.3's `va_copy` (205):** `va_copy(dst, src)` is `*(dst) = *(src)` — but in chibicc's `<stdarg.h>`, `va_list` is `__va_elem[1]`, an array, so `dst` and `src` are pointers. A real C compiler implements `va_copy` as a builtin; chibicc gets away with a one-line macro because of the array-as-pointer-decay equivalence.
   - **§18.4's pp-numbers (207):** The C spec defines a lexical category called *preprocessor numbers* that's broader than the ordinary integer/float syntax. A pp-number can be `0x1.5p-3`, `1e+10`, `1.5e-3.foo` — anything starting with a digit (or `.` followed by a digit) and continuing with digits, letters, periods, and `+`/`-` after `e`/`E`/`p`/`P`. The preprocessor passes pp-numbers through; the parser later attempts to interpret them as actual numbers. This matters for `#define X 1e10` and `0xff.5` and similar.
   - **§18.5's bitfields (210–214):** The most code-heavy section after §18.2. Read `Member`'s new fields (`is_bitfield`, `bit_offset`, `bit_width`). The codegen for read needs `shr`/`and`-mask. The codegen for write needs read-modify-write (load the unit, clear the bits, OR in the new value, store back). Op-assign (`s.bf += 1`) is read-modify-write twice. Zero-width bitfield is a parser-side rule — `int :0;` forces alignment to the next storage unit, no member is created. Address-of restriction: the parser rejects `&s.bf` for bitfield members.
   - **§18.6's `-Wall`-clean build (217):** This commit is interesting because it shows what host-cc warnings catch in chibicc's source: probably unused variables, possibly some uninitialized-on-some-paths cases, possibly some signed/unsigned compares. Worth a paragraph on what kinds of issues `-Wall` finds.
3. Read the destination state at `c3075b3` for `parse.c`, `codegen.c`, `type.c`, `chibicc.h`, `main.c`. The struct-by-value codegen will be the most invasive change.
4. Draft `chapters/18-the-full-abi.md`. Likely 10,000–13,000 words. Six sections.
5. Write `docs/sessions/019-chapter-18-draft/README.md`.
6. Write `HANDOFF.md` for session 020 (Chapter 19 — Unicode and designated initializers, commits 221–244).

## Voice / structure rules

Same as Ch 1–17:
- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For bundled sub-sections, each commit gets its own opener.
- "we" for reader, "Rui" for design intent.
- Past tense for what the commit did, present tense for current behavior.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with feature table — twenty-three rows, single table is fine.
- Diff format: lean toward inline diff fragments and quoted file snippets. The §18.2 struct-by-value codegen and the §18.5 bitfield codegen will want larger code blocks.

## Pitfalls to avoid

(Carried forward and updated.)

- Don't switch voice mid-chapter.
- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.
- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.
- Don't ventriloquize Rui — quote `quotes-rui.md` only when there's a genuinely apt passage.
- **§18.1's stack-passed-args closes errata items.** The "more than 6 integer args silently miscompiles" (Ch 5 §5.4) and "more than 8 FP args silently miscompiles" (Ch 15 §15.6) errata candidates are *closed by* this section. Name the closure explicitly.
- **§18.2's eightbyte classification is the ABI's central trick.** It's tempting to say "the ABI says structs ≤16 bytes go in registers" — that's almost right but oversimplifies. The classification is per-eightbyte-chunk, and a struct can have one chunk in a register and another in memory; or a chunk that's INTEGER in `%rdi` and a chunk that's SSE in `%xmm0`. Walk it carefully.
- **§18.3's `va_copy` is one line of macro.** Don't oversell. The interesting story is *why* it's one line (the array-decay equivalence), not the implementation.
- **§18.4's pp-numbers lexer change is invasive.** It moves number tokenization from the integer/float branches into a unified pp-number branch, then the parser interprets the pp-number's text. The pre-Ch 18 number lexer is a *parser*; the post-Ch 18 number lexer is a *recognizer*. The semantic change is small but it touches a lot of code paths.
- **§18.5's bitfield arc is the most subtle in Ch 18.** Read each commit; don't blur the five into one walk. The op-assign commit (212) is the hardest — the read-modify-write sequence has to be ordered correctly and shouldn't double-evaluate the lhs. The zero-width commit (213) has a small alignment-rule subtlety. The address-of commit (214) is a one-line restriction in the parser.
- **§18.6's `-Wall`-clean build is interesting but not deep.** A sentence per kind of warning fixed is plenty.
- **The buffered-output commit (215)** changes how `codegen` writes to its output file: it goes through an `open_memstream` buffer, then dumps. The reason is to avoid partial-output situations on errors. Don't mistake it for performance work.
- **The implicit-`return 0`-in-`main` commit (219)** lands in `parse.c`'s `function`. `main` (and only `main`) gets a synthetic `return 0;` appended if the function falls off the end. This is the C standard's special case for `main`; chibicc implements it minimally.
- **The anonymous-struct/union commit (220)** is the §18 chapter's only struct-system extension. `Member` gains a way to inline an unnamed struct's members into the parent's member-list, so member lookup walks transparent layers.

## Standing notes worth tracking across sessions

- **The hideset on Token** is unchanged. Ch 18 doesn't touch the preprocessor.
- **The Token->origin chain** is unchanged. Ch 18 doesn't touch the preprocessor.
- **The eval-quartet duplication** is unchanged.
- **The cc1-vs-driver split** is unchanged. Ch 18 doesn't touch the driver.
- **The `Initializer` tree** likely changes in §18.5 (bitfield initializers).
- **The local-vs-global split** is stable.
- **The `Relocation` mechanism** likely changes in §18.5 (global bitfield initializers).
- **The anonymous-global pattern** (no new uses) — Ch 18 unlikely to use.
- **The `is_static` default in `new_gvar`** — unchanged.
- **The `is_definition` flag on `Obj`** — unchanged.
- **The `is_unsigned` flag on `Type`** — unchanged.
- **The `__va_area__` magic name** — unchanged in chibicc's source.
- **The register-save-area layout** — likely changes in §18.3 to handle stack-resident fixed parameters.
- **The argreg integer/FP split** — unchanged.
- **The `Member->idx` field** — likely gets new bitfield-related siblings in §18.5.
- **The `is_flexible` flag** — unchanged.
- **`copy_struct_type`** — unchanged.
- **`MIN`/`MAX` macros** — unchanged.
- **`is_numeric` predicate** — unchanged.
- **Canonicalization-at-parse-time count is at eight.** Ch 17 added zero. Ch 18 *might* add one in §18.4 (pp-numbers parsed from text at parse time) — verify while drafting.
- **Pre-factor-before-feature count is at eight.** Ch 17 added one (the do-nothing preprocessor). Ch 18 might add: §18.5's bitfield work likely splits a `Member` extension before the bitfield feature lands.
- **The fourth namespace (labels)** is unchanged.
- **The `is_typename` predicate** is unchanged.
- **The VarAttr channel** has four fields. Ch 18 unlikely to grow it.
- **The `ND_NULL_EXPR` seed-pattern** — no new uses since Ch 12.
- **The `rep stosb` pattern** — no new uses since Ch 12.
- **The `unreachable()` macro** — Ch 18 likely to add callers in struct-by-value codegen.
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserved through preprocessing as of Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Tests are in C** (Ch 8 §8.2). Driver tests in shell (`test/driver.sh`). The host-cc-as-preprocessor pipeline is *gone* (Ch 17 §17.5.1).
- **The `Obj->tok` field** added in Ch 14 §14.11 still has no readers.
- **The `Type->name_pos` field** (Ch 14 §14.11) — no new uses since.
- **The `>>` codegen quirk** (Ch 11 §11.13) — partially repaired.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) — errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4) — errata candidate.
- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels — four errata candidates.
- **`f()` and `f(void)` are accepted as identical** — errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension.
- **`.bss` is the third assembly section.**
- **`.align`** is emitted for every global. §18.6's commit 218 adds 16-byte alignment for arrays >=16 bytes; check if the rule extends to other types.
- **The "more than 6 integer args silently miscompiles"** — *will be closed* by §18.1.
- **The "more than 8 FP args silently miscompiles"** (Ch 15 §15.6) — *will be closed* by §18.1.
- **The `mov $0, %rax`** — closed loop; still pessimistic; plus the variadic-FP-call wrongness (Ch 15 §15.6) — *will be closed* by §18.3.
- **psABI conformance thread:** stands at nine after Ch 17. Ch 18 will land *several* psABI corrections — stack-passed args/parameters (§18.1, two more), struct-by-value (§18.2, four more), variadic-with-stack-fixed-parameters (§18.3, one more), bitfields (§18.5, one more for layout). New count should reach approximately fifteen by Ch 18 close. Verify exact tally while drafting.
- **The `fp_offset = fp * 8 + 48` non-conforming stride** (Ch 15 §15.9) — errata candidate, *might be closed* by §18.3 — verify.
- **`long double` is `double`** (Ch 15 §15.11) — errata candidate.
- **The default-argument-promotion gap for chars and shorts** (Ch 15 §15.8) — errata candidate.
- **Float literals are inlined as integer-immediate-bit-cast** (Ch 15 §15.1).
- **Ch 1 errata list** unchanged.
- **The cast/compound-literal disambiguator** (Ch 13 §13.4) — same lookahead pattern.
- **The cast table is 10×10.** Possibly grows in §18.2 if struct-by-value introduces a new cast cell.
- **Driver brittleness** — unchanged.
- **The link command's hardcoded distro list** (Ch 16 §16.6) — errata candidate, lower priority.
- **`Node->funcname` is gone** (Ch 16 §16.2). Function calls identify the callee by `lhs`.
- **`call *%rax` is uniform across all calls** (Ch 16 §16.2). No fast path for direct named calls. Ch 18 §18.1's stack-args work doesn't change this.
- **The `StringArray` type** (Ch 16 §16.4) — used by `include_paths` (Ch 17 §17.5.2). Ch 18 might add new users for `-D`/`-U` macro lists.
- **`atexit(cleanup)` for tempfile disposal** — unchanged.
- **The `run_subprocess` helper** — unchanged.
- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `L''` ≡ `''`; `__va_arg_mem` divides by zero; `opt_S | opt_E` bitwise-`|` typo; default include paths Linux/glibc-specific.
- **`self.py` is gone** (Ch 17 §17.6).
- **Stage-2 build** is end-to-end chibicc as of Ch 17 §17.6.
- **Chibicc compiles itself** as of commit 197.

## Acceptance criteria for Ch 18

- [ ] `chapters/18-the-full-abi.md` exists, end-to-end readable.
- [ ] All twenty-three commits covered, grouped into ~6 sections.
- [ ] §18.1 names the closure of "more than 6 integer args silently miscompiles" and "more than 8 FP args silently miscompiles" errata items.
- [ ] §18.2 walks the SysV AMD64 eightbyte classification — the per-chunk classification, the worst-class merge rule, the 16-byte size cutoff to memory.
- [ ] §18.3 walks `va_copy` and the closure of the variadic-stack-fixed-parameters gap.
- [ ] §18.4 walks pp-numbers as a lexical category and the lexer-vs-parser split.
- [ ] §18.5 walks each of the five bitfield commits with no skipping. The op-assign and zero-width commits each get their own subsection.
- [ ] §18.6 walks the buffered-output commit, the ignored-flags commit, the `-Wall`-clean self-build, the 16-byte array alignment, the implicit-`return 0`-in-`main`, and the anonymous struct/union commit.
- [ ] Voice matches Ch 1–17.
- [ ] No emoji, no callouts, no admonitions.
- [ ] Forward-references checked against `chapter-mapping.md`.
- [ ] psABI conformance thread count updated to its new value (approximately fifteen).
- [ ] `docs/sessions/019-chapter-18-draft/README.md` written.
- [ ] `HANDOFF.md` written for session 020 (Chapter 19 — Unicode and designated initializers, commits 221–244).

## Prompt block to paste into a fresh session

```
Continue the chibicc book project. The user has asked for autonomous
progress — do not stop between chapters for review.

Read in order:
1. docs/sessions/018-chapter-17-draft/HANDOFF.md  (this handoff)
2. docs/sessions/018-chapter-17-draft/README.md   (what session 018 did)
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
17. chapters/15-floating-point.md
18. chapters/16-the-compiler-driver.md
19. chapters/17-a-preprocessor-from-scratch.md     (most recent chapter)
20. research/commits/chapter-mapping.md            (confirms Ch 18 scope)
21. CLAUDE.md and book-plan.md                     (conventions)

Then draft Chapter 18 (The full ABI, commits 198–220) per the steps in
the handoff. Twenty-three commits, six sections proposed in the handoff.
The bitfield arc (§18.5, five commits) is the chapter's longest single
stretch; the struct-by-value SysV ABI work (§18.2, four commits) is the
deepest. Likely no concept interlude. End-of-session: write your session
dir under docs/sessions/019-chapter-18-draft/ with a README and a
HANDOFF for session 020 (Chapter 19 — Unicode and designated
initializers, commits 221–244).
```
