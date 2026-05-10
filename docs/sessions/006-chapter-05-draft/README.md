# Session 006 — Chapter 5 draft

**Date:** 2026-05-09 (continuation of the autonomous-drafting run that produced sessions 002–005).
**Phase:** 2 (bulk drafting).

## Arrival state

Session 005 just delivered Ch 4. Same conversation, healthy context budget. User wants autonomous progress. Ch 5 covers commits 23–26: zero-arity calls, multi-arg calls, zero-arity definitions, parametered definitions. The big architectural shift in this chapter is that `Function` becomes a *list* of functions rather than a single record — chibicc programs grow from "one body" to "a list of bodies" mid-chapter.

## What was done

### Drafting decisions

- **Length:** ~8,650 words. Slightly above Ch 4 (~8,130) and on par with Ch 1 (~8,600). The handoff predicted "6,000–9,000" so this lands at the upper end. The interlude on the calling convention is the main source of bulk: it earned about 2,000 words because there's real content (caller/callee-saved, alignment, the "why six" question) that the rest of the book will keep referring back to.
- **Concept interlude on the SysV AMD64 calling convention placed between §5.1 and §5.2,** per the HANDOFF plan. It opens by explicitly back-pointing at the Ch 3 stack-frame interlude, framing this one as "the part Ch 3 foreshadowed and didn't pay off." Covers the six argument registers in order, return-value rule, caller- vs. callee-saved (with chibicc's "treats everything as caller-saved by ignoring the question" framing the HANDOFF asked for), and the 16-byte alignment rule. Ends with three rules summarized for the rest of the chapter to lean on.
- **Interpretive call: the `mov $0, %rax` mystery.** Chibicc emits `mov $0, %rax` before every `call` with no comment in the source. The book identifies it as the variadic-function `%al` zeroing rule and explains why chibicc does it for *every* call (it doesn't yet know which callees are variadic). This is the single highest-leverage interpretive moment in the chapter — without an explanation, the reader sees a curious instruction with no context; with one, they have a piece of ABI knowledge they can use elsewhere.
- **Section structure** mirrors Ch 1, 3, and 4: each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote, ends with a "Where we are." Recap table at the end with one row per commit.
- **No Rui-quote citations this chapter.** The README has no apt passage on calling conventions or the function-definition pivot, and `quotes-rui.md` doesn't have a function-related quote that would feel natural to invoke. Following the Ch 4 precedent — when there's nothing canonical, don't fake one.
- **Forward references** kept short and grounded. Ch 6 mentioned for: arrays piggyback on `base`, hardcoded `8` becomes a per-type lookup, `sizeof`. Ch 7 mentioned for `char`. Ch 9 for structs. Ch 10 for full declarators (the parenthesized-name trick) and the function-type pivot. Ch 13 for proper stack alignment under arbitrary nesting. All cross-checked against `chapter-mapping.md`.
- **Diff format** consistent with prior chapters: `diff` blocks for targeted line-level changes, full quoted code for new functions (`funcall`, `function`, `create_param_lvars`), one ASCII frame diagram in the interlude (the SysV stack-frame view from Ch 3 is referenced rather than redrawn).
- **Test-suite rewrite called out explicitly** at §5.3 — the wholesale rewrite of every test from `'{ ... }'` to `'int main() { ... }'` is the single biggest change in that commit, and it shapes how to read it. Decided not to reproduce the whole diff (it's pure busywork) but to show the pattern with one example.

### Three small interpretive calls

1. **The `mov $0, %rax` is identified as variadic-`%al` housekeeping.** This is research-rather-than-source-comment territory. The chibicc source has no comment; the SysV ABI doc does. The book takes the position that this is worth surfacing once, in §5.1, because the instruction will recur in front of every call from this point on and the reader will keep wondering what it's for. (Could become an errata-appendix entry — "Rui's source comment would help here" — but the omission isn't a bug.)
2. **`create_param_lvars`'s recursion-then-act trick gets a full paragraph.** It's small but locally elegant — recurse first, then `new_lvar`, so the prepending-`new_lvar` ends up putting parameters in declaration order in `locals`. The book walks through *why* this works and lists three other ways it could have been done. The rationale: this is the kind of code an attentive reader will pause on, and a missed explanation is a cliff.
3. **The "more than 6 arguments crashes" admission lands explicitly.** Both the parser (in `type_suffix`) and the codegen (the parameter-spill loop) index `argreg[i]` without bounds-checking. The book calls this out as a real punt that chibicc will not address in this chapter, per the HANDOFF's "don't fix Rui's code in the prose" rule. Tagged for potential errata appendix when one exists.

### One careful avoidance

The `Function` / `Obj` convergence flagged in the HANDOFF (parameters are both Obj-shaped *and* Function-shaped — both have names and types) is acknowledged structurally (the prose notes that `fn->params` is a prefix of `fn->locals`) but not framed as "this is going to merge soon." Ch 6 does the merge (commit `0b76634` — *Merge Function with Var*), and it'll be more satisfying to call out the convergence at the moment it actually happens than to anticipate it now. Decided to let the merge speak for itself.

### The canonicalization pattern still not formally named

Ch 5 doesn't really add a new canonicalization-at-parse-time instance — the parameter-handling is the closest, but it's a different kind of move (use the ambient-`locals` global as a buffer, snapshot it). Per the HANDOFF's note that we should wait for Ch 6 (`[]`) and Ch 7 (`+=`) to give clearer cases, no naming pass happened. The standing note carries forward unchanged.

### Voice / structure inherited from Ch 1–4

- "we" for reader-journey, "Rui" for design intent.
- Past tense for what the commit did, present for current behavior.
- Section opens with `git checkout <full-hash>`.
- No emoji, no callouts, no admonitions.
- Per-section "Where we are" closers.
- The chapter has a closing recap with a feature table.

## Open questions surfaced for user

None — autonomous mode.

## Notes worth carrying forward

- **The hardcoded `8` in `new_add`/`new_sub` survives Chapter 5 unchanged.** It's still going to become `lhs->ty->base->size` in Chapter 6's `sizeof` commit (`3e55caf`). The Ch 4 prose flagged it once; the Ch 5 prose flagged it again in the closing of §5.4. Three mentions across two chapters is enough setup; the Ch 6 mention should *show the change*, not re-explain the foreshadowing.
- **`Function` and `Obj` are about to merge in Ch 6 (`0b76634`).** Per the HANDOFF's call-out. The Ch 5 prose noted that `fn->params` is a prefix of `fn->locals` and that parameters are "also" locals — preparing the reader to find the merge intuitive rather than surprising. Ch 6 should land the merge with a brief "as we noted in Ch 5" reference.
- **The `mov $0, %rax` interpretation may need a footnote treatment in revision.** It's correct (we double-checked: the SysV ABI section 3.2.3 specifies `%al` as the SSE register count for variadic calls), but readers who Google it will see various other explanations; a one-line footnote with the spec section number would help. Defer to revision pass — the chapter still reads cleanly without one.
- **The Ch 1 errata list is unchanged from session 005's notes.** No new items found while drafting Ch 5. Test-count off-by-one and the `skip → error_tok` upgrade are still the only known issues. The `mov $0, %rax` and the "6+ args silently miscompiles" notes from Ch 5 are tagged for possible errata-appendix items but aren't Ch 1 issues.
- **Six argument cap is now a structural constraint of the language chibicc compiles.** Worth keeping in mind: any future test the book writes that needs to demonstrate something with many arguments has to stay at six or fewer. Easy to forget once we're deeper in.
- **The TY_FUNC type kind exists but has no user yet.** `func_type` is built during parsing and immediately discarded after `name` and `params` are extracted. Chapter 10 (Filling out the type system) is where TY_FUNC starts to do real work, when nested declarators force the compiler to distinguish "function returning int" from "pointer to function returning int." Worth noting in the Ch 10 session that this is the payoff for Ch 5's quiet groundwork.

## Exit state

- `chapters/05-functions.md` drafted, ~8,650 words.
- Session 006 dir populated.
- HANDOFF.md primes session 007 (Chapter 6 — Arrays, commits 27–31).
- CLAUDE.md status note still reflects "autonomous progress" mode; no edit needed beyond updating the chapter count.
