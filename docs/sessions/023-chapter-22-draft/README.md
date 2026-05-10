# Session 023 — Chapter 22 draft

**Date:** 2026-05-10 (continuation of the autonomous-drafting run that has produced sessions 002–022).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 022 delivered Ch 21 (Thread-local, alloca, VLAs, seventeen commits, ~10,654 words). User direction is still autonomous — no chapter-by-chapter review. Ch 22 covers commits 284–306: twenty-three commits — labels-as-values as compile-time constant, the string hashmap and three uses of it, the seven-commit `-M` family, `-fpic`/`-fPIC` (which actually changes codegen), file-search caching, three include-handling improvements (include-guard optimization, `#pragma once`, `#include_next`), five linker-driver pass-throughs (`-static`, `-shared`, `-L`, `-Wl,`, `-Xlinker`), and the third-party-app shell-script harness.

## What was done

### Drafting decisions

- **Length:** ~9,320 words. Below the handoff's 12,000–14,000 target. Honest reading: most commits in this chapter are small driver-side or one-line preprocessor changes whose interesting content is one or two paragraphs. The §22.4 `-M` family (seven commits) and §22.2 hashmap (one commit) carry most of the weight at ~2,500 and ~1,800 words respectively. Padding the chapter to 13,000 would require either repeating the hashmap walk or inventing concept interludes; neither was warranted.
- **Section structure:** 7 sections from 23 commits, exactly as the handoff proposed. §22.1 single-commit, walks the `char ***` indirection. §22.2 single-commit, walks the hashmap as a self-contained data-structure addition. §22.3 three commits with three named subsections. §22.4 seven commits with seven named subsections (longest section, ~2,500 words). §22.5 two commits (the `-fpic` half is structurally larger than the file-search half). §22.6 three commits, three subsections. §22.7 six commits, six subsections.
- **No concept interlude.** The handoff said "possibly one" around the hashmap. Reading the §22.2 prose, the hashmap walk is self-contained enough at ~1,800 words; pulling out a separate "open-addressing tradeoffs" interlude would have introduced repetition with the §22.3 three-call-site walk. The §22.2 → §22.3 progression is enough scaffolding.
- **§22.1 explains the `char ***` (`Relocation->label` becomes `char **`, plus the parser-side `char ***`) by naming the lazy-resolution mechanism.** The reason for the indirection is that `Node->unique_label` is filled in by codegen, after `eval2` runs over the global initializer; storing a pointer to the slot lets `emit_data` read the eventually-resolved value.
- **§22.2 walks the hashmap as a complete data structure.** Open-addressing, FNV-1a (called out as FNV-1a not FNV-1, with the multiply-before-XOR ordering noted), 70/50 watermarks, tombstone-and-NULL probe rules, the match-first ordering on insertion. Two API conventions named: the `(pointer, length)` key shape that fits tokens, and the `2`-suffixed variants vs the `strlen`-shimmed non-`2` variants.
- **§22.3's three subsections each name the access pattern.** Macros (lookup-heavy, lifetime full translation unit), block scopes (lookup-medium, lifetime per scope), keywords (insert-once, lookup-many). The keyword walk explains the lazy-init trick (`if (map.capacity == 0)` plus `static HashMap`) that builds the map on first call.
- **§22.4 walks all seven `-M` flags individually.** Each subsection names the specific behavior: `-M` writes to stdout; `-MF` redirects; `-MP` adds phony rules (with `i = 1` to skip the source file itself); `-MT` overrides the target with appending behavior; `-MD` enables alongside compilation; `-MQ` is `-MT` with `quote_makefile` escaping (covering `$`, `#`, whitespace); `-MMD` is `-MD` minus system headers (with the `std_include_paths` separation).
- **§22.5's `-fpic` walk corrects the handoff's prediction.** The handoff said `-fpic` "may not actually change codegen" since chibicc already emits rip-relative addressing. Wrong. `-fpic` adds a real PIC-mode branch in `gen_addr` that uses `mov name@GOTPCREL(%rip), %rax` for globals and the four-instruction `__tls_get_addr` sequence (with `data16` and `0x6666` padding for linker-rewritable general-dynamic TLS) for thread-local variables. The chapter's biggest surprise.
- **§22.5's file-search cache is named as a real performance win** (an order of magnitude on header-heavy translation units). The cache stores positive results only; negative results would require a separate sentinel, which Rui doesn't bother with.
- **§22.6's include-guard optimization is walked subsection-by-subsection** with attention to the three checks (first directive is `#ifndef IDENT`; second is `#define IDENT` with the same identifier; the closing `#endif` is the last token). Nested conditionals are walked via `skip_cond_incl` so they don't disqualify the file.
- **§22.6's `#pragma once` is shown as a thin reuse of the include-file cache pattern** with a separate `pragma_once` hashmap.
- **§22.6's `#include_next` walk names the `include_next_idx` global,** and the subtle gap that it's only updated on fresh (non-cached) searches. Flagged as an errata candidate in the closing recap.
- **§22.7 covers five linker-driver pass-throughs and the third-party harness.** `-static` walks the library-grouping change (`--start-group`/`--end-group` for libgcc/libgcc_eh/libc circular dependency) and the dynamic-linker omission. `-shared` walks the `crtbeginS.o`/`crtendS.o` substitution. `-L` walks the spaced-vs-joined form acceptance. `-Wl,` walks the `strtok` comma-split and the `input_paths` routing for ordering. `-Xlinker` walks the literal-pass-through to `ld_extra_args`. The third-party harness names the four pinned repos (git, libpng, sqlite, tinycc), the shared `common` script, and the `libtool` `sed` patch (the `wl=-Wl,` and `pic_flag=-fPIC` substitutions).

### Interpretive calls

1. **§22.1's `Relocation->label` indirection is named as a lazy-resolution mechanism.** The reason for `char **` (rather than `char *`) is that `Node->unique_label` is generated during codegen, after `eval2` runs. Storing a pointer-to-pointer captures the slot's address, which gets dereferenced at emit time when the label name is filled in. The chapter names this; without the explanation the diff is mysterious.
2. **§22.2's hash function is named as FNV-1a, not FNV-1.** The constants `0xcbf29ce484222325` (offset basis) and `0x100000001b3` (prime) are the canonical 64-bit FNV constants. The multiply-before-XOR ordering is the FNV-1a variant. Both work; FNV-1a is the more recommended.
3. **§22.2's `(void *)-1` tombstone is named as conventional.** Any sentinel distinct from NULL and from a valid heap pointer would work; `(void *)-1` is the C tradition.
4. **§22.2 names the missing iteration primitive.** The hashmap has no API for walking all entries. A future include-guard optimization needs iteration over `include_paths`, but that's a `StringArray`, not a `HashMap`. Rui doesn't add iteration to the hashmap.
5. **§22.3 names the per-call-site asymptotic improvements.** Macros: a few hundred entries → ~30 keyword entries, both lookup-heavy. Scopes: up to 200 string compares per identifier reference (10 nested scopes × 20 locals) → 10 hashmap_get calls. Keywords: 30-element linear scan → O(1) lookup.
6. **§22.3.3 names that the keyword hashmap is built once per program** via the `if (map.capacity == 0)` lazy-init guard plus `static HashMap`. Two separate static hashmaps for `is_keyword` (tokenize.c) and `is_typename` (parse.c); Rui doesn't try to share them.
7. **§22.4.6's `quote_makefile` is named as one-sided.** The escaping is applied to the rule's *target* (and via `-MP` to phony-rule names), but not to the dependency list. A header path containing `$` or `#` would produce a malformed `.d` file. Errata candidate.
8. **§22.4.7's `-MMD` filter is named as path-prefix-based.** The `in_std_include_path` predicate compares the dependency path's prefix against entries in `std_include_paths` (the snapshot of system include paths taken before user `-I` flags arrive). Simple and correct for the common case.
9. **§22.5.1's `-fpic` walk names the four-instruction TLS sequence as a linker-rewritable padding pattern.** The `data16` prefix and `0x6666` value are deliberate padding bytes that the linker can rewrite to convert general-dynamic TLS into local-dynamic or initial-exec, if it determines the variable is reachable in the local module. The 16-byte total occupancy is what the linker needs.
10. **§22.5.1 names that `-fpic` and `-fPIC` are both treated identically by chibicc** (in real toolchains, `-fpic` allows a smaller GOT). Chibicc uses the large-model code, which is a strict superset of what `-fpic` requires.
11. **§22.5.1 grows the psABI conformance count by one** for the GOT-and-`__tls_get_addr` sequences. New count: nineteen.
12. **§22.5.2 names that negative results aren't cached.** A `#include "missing.h"` will repeat the full search every time. In practice this is rare (preprocessing hits a missing header once and errors).
13. **§22.6.1 names the optimization's gap relative to gcc.** Chibicc's optimization caches the guard macro name; on subsequent includes, if the macro is defined, skip. If the macro is `#undef`-ed, chibicc retokenizes and runs through the conditional. Gcc has more elaborate mechanisms; chibicc's is the simple version.
14. **§22.6.3 names the `include_next_idx` cache-miss gap.** The index is only updated on a fresh (non-cached) `search_include_paths` call. A cached lookup leaves the index at whatever value the most recent fresh search produced. In the common case (a wrapper header that does `#include_next` once) this works; in elaborate scenarios it could surprise. Errata candidate.
15. **§22.7.1's `-static` walk names the library-circular-dependency reason for `--start-group`/`--end-group`.** Static builds need libgcc → libc → libgcc retries; dynamic builds use `--as-needed` plus `libgcc_s` (the shared variant) instead.
16. **§22.7.4 names the routing distinction between `-Wl,` and `-Xlinker`.** `-Wl,` goes through `input_paths` (preserves command-line ordering), `-Xlinker` goes through `ld_extra_args` (no ordering). Justified by the typical use cases.
17. **§22.7.6 names the `libtool sed` patch as load-bearing.** Configure-generated `libtool` doesn't recognize chibicc and falls back to defaults that don't match. The `wl=-Wl,` and `pic_flag=-fPIC` substitutions patch in chibicc-compatible values after the fact.
18. **§22.7.6 names that the harness exists outside `make test`.** The third-party scripts require network access and many minutes; they're invoked manually. But their existence is the chapter's milestone — chibicc can build production C codebases.

### Voice / structure inherited from Ch 1–21

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. Multi-commit sections list all hashes at the top.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- One-table recap at the chapter close (with §-section column).
- No concept interludes.

### Three careful avoidances

- **Did not invent a "history of make and dependency files" interlude.** Make has a long history (1976), and the `-M` family in gcc has its own evolution. The chapter focuses on what the seven flags do in chibicc and what their output looks like, without walking the standardization history.
- **Did not invent a "hashmap design tradeoffs" interlude.** Open addressing vs separate chaining, FNV vs SipHash, robin-hood probing, etc. — all valid topics, but chibicc's hashmap is the simple version and over-explaining the alternatives would crowd out the actual walk.
- **Did not over-explain the GOT and TLS access models.** The §22.5 `-fpic` walk names the GOT, the GOTPCREL relocation, the `__tls_get_addr` runtime call, and the linker-rewritable padding. It doesn't walk the full Linux TLS spec or the `R_X86_64_REX_GOTPCRELX` ABI document.

### Date-vs-position note

The twenty-three commits scatter widely across August, September, and October 2020. The seven `-M` commits in particular were drafted across a month — `-MF`, `-MP`, `-MT`, `-MD` on August 18; `-MQ` on September 3; `-M` on September 3 (later that day); `-MMD` on September 19. The chapter follows `main` order (which corresponds to logical-dependency order: `-M` first, then variants, then the `-MMD` polish) without commenting on the dates.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **Hashmap is now the workhorse data structure.** Five distinct call sites: `macros` (preprocess.c), `Scope->vars` and `Scope->tags` (parse.c), `is_keyword`'s static map (tokenize.c), `is_typename`'s static map (parse.c), `search_include_paths`'s `cache` (preprocess.c), `include_guards` and `pragma_once` (preprocess.c). Eight `HashMap` instances total. Future commits adding to the compiler should reach for the hashmap, not a linear scan.
- **`Macro` lost `next` and `deleted` fields.** Now stored only in the hashmap.
- **`VarScope` lost `next` and `name` fields.** Now stored only as hashmap key+value.
- **`TagScope` is gone entirely** — the hashmap stores `name → Type *` directly.
- **`Relocation->label` is `char **`.** The dereference happens in `emit_data`. The double-pointer accommodates lazy-resolved label names (`Node->unique_label`) and same-channel global names (`Obj->name`).
- **`-fpic`/`-fPIC` is real codegen, not a flag-flip.** Two new asm patterns: `mov name@GOTPCREL(%rip), %rax` for globals and the four-instruction `data16 lea ... __tls_get_addr@PLT` sequence for TLS. The non-PIC paths from Ch 21 still fire when the flag is absent. Flag selects between them at codegen time.
- **`std_include_paths`** is a snapshot of `include_paths` taken at the end of `add_default_include_paths` (i.e., before user `-I` flags). Used by `-MMD` to filter out system headers from dependency lists.
- **`print_dependencies`** is the new function in main.c that emits Makefile-shaped rules. Reads `get_input_files()` to enumerate every File the tokenizer touched.
- **`quote_makefile`** escapes `$` (→ `$$`), `#` (→ `\#`), and whitespace (→ `\ ` with backslash doubling for prior backslashes). Applied to the rule's target only; *not* to the dependency list (errata candidate).
- **`include_next_idx`** is a file-scope int in preprocess.c. Updated only on fresh (non-cached) `search_include_paths` calls. `#include_next` after a cache hit may use stale index (errata candidate).
- **`detect_include_guard`** runs once per first-time include. Three checks: `#ifndef IDENT` first, `#define IDENT` (same identifier) second, `#endif` last token. Walks past nested conditionals via `skip_cond_incl`.
- **`pragma_once`** hashmap stores file paths that asked for the optimization. Separate from `include_guards` (which stores file → guard-macro-name).
- **`opt_static` and `opt_shared`** are mutually-exclusive driver booleans. Each restructures `run_linker` for its case. `-static` adds `--start-group`/`--end-group` brackets and omits `-dynamic-linker`. `-shared` swaps `crt1.o`/`crtbegin.o`/`crtend.o` for `crti.o`/`crtbeginS.o`/`crtendS.o` (the `S` suffix marks PIC-friendly startup files).
- **`-Wl,arg1,arg2`** routes through `input_paths` so command-line ordering is preserved relative to other inputs. The main loop splits on commas via `strtok` and pushes each piece to `ld_args`.
- **`-Xlinker arg`** routes through `ld_extra_args` (no ordering). Each `-Xlinker` takes one literal argument; comma-handling is not required.
- **psABI conformance count is at nineteen** (up from eighteen). `-fpic`/`-fPIC` adds the GOT-and-`__tls_get_addr` sequences as conformant PIC forms.
- **Canonicalization-at-parse-time count is unchanged at eleven.**
- **Pre-factor-before-feature count is unchanged at nine.** The hashmap could be argued as a pre-factor for its three users, but the walk treats hashmap-and-three-users as a single arc rather than counting the hashmap as a separate refactor.
- **Errata candidates added in Ch 22:**
  - `quote_makefile` is one-sided (target only, not dependencies). Filenames containing `$` or `#` produce malformed `.d` files.
  - `include_next_idx` is only updated on fresh `search_include_paths` calls. Cached lookups leave it stale.
- **Errata candidates closed in Ch 22:** none.
- **Errata candidates remaining:** Ch 17's three (`#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific), Ch 19's two (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block), Ch 20's one (`is_compatible` array arm bug), Ch 21's two (`.size` missing for functions, suffix-only `.a`/`.so` recognition), Ch 22's two new — total: ten.
- **Stage-2 build is end-to-end chibicc, `-Wall`-clean** — unchanged.
- **Chibicc compiles itself** — unchanged.
- **Third-party harness exists** for git, libpng, sqlite, tinycc. Manual invocation; not part of `make test`.

## Exit state

- `chapters/22-performance-deps-and-the-linker-driver.md` drafted, ~9,320 words.
- Session 023 dir populated with this README and HANDOFF.md.
- HANDOFF.md primes session 024 (Chapter 23 — Atomics and the final polish, commits 307–316, 10 commits).
- CLAUDE.md status note updated to "Ch 22 drafted".
