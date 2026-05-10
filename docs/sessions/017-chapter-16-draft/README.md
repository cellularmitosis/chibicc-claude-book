# Session 017 — Chapter 16 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that has produced sessions 002–016).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 016 delivered Ch 15 (Floating point, eleven commits, ~11,995 words). User direction is still autonomous — no chapter-by-chapter review. Ch 16 covers commits 150–157: the stage-2 Makefile target, the function-pointer trio (type, decay, arithmetic conversion), the cc1-vs-driver split, the `as` invocation, multiple input files, and the `ld` invocation. Eight commits — driver-shape changes plus three function-pointer codegen/parser/type-system commits.

## What was done

### Drafting decisions

- **Length:** ~9,688 words. Slightly above the handoff's 7,000–9,000 forecast upper edge. The chapter ran a bit longer than predicted because §16.2 (the function-pointer trio) wanted full bundle treatment and §16.6's link-command walk warranted extended prose — the Linux/glibc/distro path probing isn't a one-paragraph topic. Neither §16.2 nor §16.6 could be compressed without losing the walk.
- **Section structure:** 6 sections from 8 commits, with the function-pointer trio bundled into §16.2 exactly as the handoff proposed. No other bundling. The other six commits each get their own section.
- **No concept interlude.** The handoff defaulted to no, conditional on whether the cc1-vs-driver split surfaced a need. The split's substance lived comfortably in §16.3 inline — pulling it out into a separate "GCC driver shape" interlude would have left §16.3 with no body. The IEEE-754-style move (an interlude on the GOT/PIC story) was tempting in §16.2 but the same logic applied: the GOT/PIC machinery only matters in the function-pointer addressing context, so it stayed inline. Rui's source comment block in `gen_addr` is the longest prose chunk in `codegen.c`; the chapter quotes a reduced version of it and explains the rest.
- **§16.1 names the stage-2 build as a stepping stone, not self-hosting** — handoff acceptance criterion. The §16.1 prose calls `self.py` "a preprocessor implemented as a Python regex pipeline — pragmatic, brittle, and exactly enough for chibicc's source code in its present shape." It also calls out that the stage-2 Makefile target "is the canary for self-hosting" and forward-references Ch 17 §17.6 for when `self.py` retires.
- **§16.2 covers the function-pointer trio together** — handoff acceptance criterion. The bundling decision is named in the section's framing: "the same three-step shape Chapter 6 walked through for arrays." The Ch 6 array-decay arc structural mirror is named explicitly. The first commit gets the most attention (it's the largest); the second and third get short walks because they're 6 and 5 lines respectively.
- **§16.3 walks the cc1-vs-driver split as the chapter's structural anchor** — handoff acceptance criterion. The single-binary-two-roles trick is named, the fork-exec round-trip cost is named, the choice's parallel with GCC is named. The `run_subprocess` helper's full body is quoted.
- **§16.4 walks the `as` invocation including temp-file handling** — handoff acceptance criterion. The `mkstemp`/`atexit(cleanup)` mechanism is walked. The `replace_extn` helper's basename-strip side-effect is named (chibicc's default-output behavior puts everything in the cwd, which is GCC's behavior).
- **§16.5 walks multiple-input-file handling and `-o` semantics** — handoff acceptance criterion. The `-cc1-input`/`-cc1-output` driver→cc1 protocol is named. The `-o`-with-multiple-inputs-error rule is named. Sequential-not-parallel is briefly noted.
- **§16.6 walks the `ld` invocation including the brittle Linux/glibc path hardcoding** — handoff acceptance criterion. The full `run_linker` body is quoted. The path-probing functions are quoted. The brittleness is named explicitly: "Linux-specific and brittle." The errata-candidate framing applies.
- **One-table recap.** Eight rows. Resisted splitting into multiple tables.

### Interpretive calls

1. **The cc1-vs-driver split is in spirit a refactor *and* a feature.** The handoff hedged on whether it counts toward the pre-factor-before-feature count. Decision: it doesn't. The split *is* the feature; the existing `main` shape couldn't have been extended to multi-stage compilation without it, and the same commit that does the split adds the driver-mode dispatch — they're a unit. Pre-factor count stays at seven.
2. **The GOT/PIC story is `gen_addr`'s, not the chapter's.** The handoff didn't predict the GOT/PIC story would land in the chapter; reading the diff: §16.2's commit 151 includes a fifteen-line source comment in `gen_addr` walking through address randomization and dynamic linking. Chapter prose paraphrases it briefly rather than quoting in full. The §16.2 prose owns this as part of the function-pointer story even though it's about *all* function-and-global addressing post-this-commit.
3. **psABI conformance thread ticks to nine.** The handoff predicted Ch 16 (driver-shape) probably wouldn't add. Reading the diff: the `gen_addr` GOT path is a conformance correction (calls into shared-library symbols now go through GOT, the psABI-required addressing mode for non-local function references). Counted explicitly in the chapter closer. Thread now stands at nine.
4. **Function-pointer codegen note: every call is now `call *%rax`, no fast path for direct named calls.** Named in §16.2 as "a small pessimization, large simplification." Errata-candidate-style framing wasn't used because Rui clearly chose this; it's a design choice, not an oversight.
5. **The "default output is in cwd, not the input directory" semantics is named explicitly.** §16.4's `replace_extn` strips the basename. `chibicc foo/bar.c` produces `./bar.o`, not `foo/bar.o`. That's GCC's behavior too; the prose names it without flagging.
6. **The link command's brittleness is named, the errata-candidate label is *lower priority*.** The fix would be querying `gcc -print-search-dirs` at runtime, which (as the chapter says) "would defeat the purpose of writing a driver from scratch." Errata-candidate framing applies but it's not on the same priority axis as e.g. integer-promotion gaps.
7. **Function-decay parallel with array-decay is exact in two of three pieces.** §16.2's prose names this: parameter-context arms parallel exactly, codegen `load` skip parallels exactly, `get_common_type` for arrays goes through a different code path but the unification logic is parallel. Naming the "exact in two of three" is more precise than calling it a perfect mirror.
8. **The chapter doesn't add a fifth namespace, doesn't grow `VarAttr`, doesn't extend the cast table, doesn't add a canonicalization-at-parse-time arm.** All of those forecasts from the handoff held.

### Voice / structure inherited from Ch 1–15

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote; §16.2 has three openers (the bundling).
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- Closing recap with one feature table (eight rows).

### Three careful avoidances

- **Did not invent a GCC-driver-vs-cc1 history interlude.** The cc1 model predates GCC by a long way (it's a Bell Labs UNIX-era artifact); there's a real story to be told about it, but it's not chibicc's story, and the chapter doesn't try to teach it. §16.3's prose names "Real GCC works the same way" without going into history.
- **Did not invent an IEEE-754-equivalent interlude on linker-and-loader.** The §16.6 link command is brittle and worth walking, but the *why* of dynamic linking and PIC and the runtime loader is a Linkers and Loaders chapter, not a chibicc chapter. The §16.6 prose stays in the lane of "what chibicc constructs and why each piece is there."
- **Did not invent forward references that don't exist.** The Ch 17 forward references are all grounded in the chapter mapping or in the chapter's own forecast about what stage-2 needs to retire `self.py`.

### Date-vs-position note

Calendar dates scatter widely. Commit 150 (stage-2) is dated August 2019 — predating Chapters 12–15 entirely. Commits 151–157 cluster across August, September, and October 2020. The chapter follows `main` order without comment, the same as Ch 7–15. Commit 150's calendar position predating the type-system work is the most extreme date-vs-position case in the book so far, and the chapter opening notes it explicitly: "Reading the diff explains why: it's a Makefile change, untouched by the type-system work that came later."

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The cc1-vs-driver split** is the chapter's structural anchor. Same binary, dispatched by `-cc1` flag. The driver fork-execs cc1 for each input. Real GCC's shape.
- **`Node->funcname` is deleted.** Function calls identify the callee by `lhs`. The `call *%rax` indirect-call sequence is uniform across all calls.
- **`is_definition` flag finally has a second reader.** `gen_addr`'s function-pointer path uses it to choose between `lea name(%rip)` and `mov name@GOTPCREL(%rip)`. Before Ch 16, only the `extern` path read it.
- **The `StringArray` type lives in `chibicc.h`.** Used four places after Ch 16: `tmpfiles`, `input_paths`, `ld_args`, and inside `run_linker`. Likely to be reused in Ch 17 for `#include` search paths and macro-table-style storage.
- **The `atexit(cleanup)` registration** is the driver's tempfile-disposal mechanism. Catches normal exits and `error()` exits; doesn't catch hard kills. Same shape as GCC's.
- **The `find_libpath`/`find_gcc_libpath` probing** is Linux/glibc-specific. Hardcoded distro list (Debian, Gentoo, Fedora). On other distros, errors at link time. Errata candidate, lower priority.
- **The `run_subprocess` helper** is the shared fork/execvp/wait mechanism for cc1, as, and ld. Future tools (e.g., a hypothetical separate `cpp` binary, though Ch 17 likely won't take that route) would use the same helper.
- **The `-###` debug flag** echoes subprocess command lines without running them. GCC-borrowed.
- **The `-cc1-input`/`-cc1-output` flags** are internal-only. The user-facing `-o` continues to mean what it has always meant; the `-cc1-*` flags are constructed by the driver inside `run_cc1`.
- **The full `chibicc input.c` pipeline** now goes from C source to executable end-to-end through chibicc's tools, except for the preprocessor (which Ch 17 supplies). The host `cc` is still invoked for the test harness's final link with `test/common`, but the user-facing `chibicc input.c` no longer needs `cc` at all.
- **psABI conformance thread stands at nine** after Ch 16 (Ch 13 §13.8/§13.9 + Ch 14 §14.1/§14.2/§14.8 + Ch 15 §15.6/§15.7/§15.8/§15.9 + Ch 16 §16.2 GOT path = nine). Ch 17 (preprocessor) probably won't add to this thread.
- **Canonicalization-at-parse-time count is unchanged at eight.** Function-decay in parameter context lands in `func_params`, which is parse-time; it could plausibly count, but it's grouped with the array-decay arm from Chapter 6 which is also parse-time and was *not* counted. Decision: don't double-count by counting parameter-context decays separately from the array-decay precedent. Stays at eight.
- **Pre-factor-before-feature count is unchanged at seven.** The cc1-vs-driver split is the feature, not a refactor.
- **The fourth namespace (labels)** is unchanged. Ch 16 doesn't add a fifth.
- **The `is_typename` predicate** is unchanged. Ch 16 doesn't extend it.
- **The VarAttr channel** stays at four fields.
- **The cast table** is unchanged at 10×10. Function pointers don't need cast-table cells.
- **The `Initializer` tree** is unchanged.
- **The `Relocation` mechanism** has implicit new uses through the GOT path in `gen_addr`, but no new code in `Relocation` itself. Function-pointer initializers (e.g., `int (*fn)(int) = add2;`) work via `gen_expr` on the initializer's RHS, which goes through `gen_addr` for the function name; they don't need new `Relocation` machinery.
- **The `Obj->tok` field** added in Ch 14 §14.11 still has no readers after Ch 16.
- **The `Type->name_pos` field** (Ch 14 §14.11) — no new uses since.
- **Tests are in C** as of Ch 8 §8.2. Driver tests are still in shell (`test/driver.sh`).
- **The host-cc-as-preprocessor pipeline** (Ch 8 §8.2) still in place. Collapses in Ch 17.
- **GDB-debuggable output** (Ch 8 §8.4) — already taken for granted.
- **Per-token line numbers** (Ch 8 §8.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.
- **Ch 1 errata list** unchanged.
- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 §7.5) — errata candidate.
- **The hex-escape silent truncation** (Ch 7 §7.4) — errata candidate.
- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels — four errata candidates.
- **`f()` and `f(void)` are accepted as identical** by chibicc, with §14.3 adding a third equivalence (`f()` becomes variadic). Errata candidate.
- **Empty brace initializer (`int x[3] = {};`)** — chibicc-specific extension matching GCC.
- **`.bss` is the third assembly section.** Section list: `.text`, `.data`, `.bss`.
- **The `>>` codegen quirk** (Ch 11 §11.13) — partially repaired by Ch 14 §14.5 and Ch 15 §15.4.
- **"More than 6 integer args silently miscompiles"** in Ch 5 §5.4 — errata candidate. Sibling: "more than 8 FP args silently miscompiles" from Ch 15 §15.6.
- **The `mov $0, %rax`** (Ch 5 §5.1) — closed loop in Ch 14 §14.1. Still pessimistic. Plus the variadic-FP-call wrongness from Ch 15 §15.6.
- **The `fp_offset = fp * 8 + 48` non-conforming stride** (Ch 15 §15.9) — errata candidate.
- **`long double` is `double`** (Ch 15 §15.11) — errata candidate.
- **The default-argument-promotion gap for chars and shorts** (Ch 15 §15.8) — errata candidate.
- **Float literals are inlined as integer-immediate-bit-cast** (Ch 15 §15.1), not anonymous-global.
- **The cast/compound-literal disambiguator** (Ch 13 §13.4) — same lookahead pattern. No new uses in Ch 16.
- **Driver brittleness** — the `find_libpath`/`find_gcc_libpath` probing is hardcoded to Linux x86-64 with glibc and three specific distros. Errata candidate, lower priority.

## Exit state

- `chapters/16-the-compiler-driver.md` drafted, ~9,688 words.
- Session 017 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 018 (Chapter 17 — A preprocessor from scratch, commits 158–197).
- CLAUDE.md status note will be updated to "Ch 16 drafted".
