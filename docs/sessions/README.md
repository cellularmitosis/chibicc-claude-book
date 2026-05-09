# Sessions

One directory per work session, globally numbered. Each session dir contains:

- `README.md` — narrative of the session. First paragraph = state on arrival. Last paragraph = state on exit. Everything in between is what happened, in roughly chronological order, with enough detail that a future session can reconstruct the thinking.
- `HANDOFF.md` — primer for the next session, written when there's specific unfinished work or open questions to pass forward. Skip when the session ended at a clean stopping point.
- `findings.md` — "things we learned this session that will matter later" (optional; many sessions fold this into README.md).
- `commits.md` — commits landed this session, one-liner each (optional; only useful for sessions that produced many commits).

## Naming

`NNN-<short-slug>/`

`NNN` is a global, monotonically-increasing session number. The slug should hint at substantive content so sessions are skimmable from `ls`:

- `001-research-and-plan/`
- `002-chapter-01-draft/`
- `003-chapter-01-revisions/` *(hypothetical)*
- `004-chapter-02-and-03/` *(hypothetical, if a small chapter pairs with the next)*

The default cadence is **one chapter per session**, but reality varies. Plan/research/revision sessions are normal. Some chapters (e.g. the ~40-commit preprocessor chapter) may need multiple sessions; small chapters (e.g. Ch 2 = 1 commit) may pair with the next.

## Start-of-session checklist

1. **Read the most recent session's `README.md`** for context — what just happened, what state things are in.
2. **Read `HANDOFF.md` if one exists** in the most recent session dir. It supersedes anything ambiguous in the README.
3. **Glance at [book-plan.md](../../book-plan.md)** for the master plan, especially "open questions" if any are flagged.
4. **Glance at [research/commits/chapter-mapping.md](../../research/commits/chapter-mapping.md)** if drafting a new chapter — confirms which commits you're covering.
5. **Create your session dir** before doing real work: `mkdir docs/sessions/NNN-<slug>` and write a one-line "starting state" stub in `README.md`. Lets you capture inline as you go.

## End-of-session ritual

1. **Finalize the work itself** — chapter file, plan updates, research notes, etc. saved.
2. **Write this session's `README.md`** with arrival → work → exit. Include:
   - What you did and why.
   - Decisions made (especially ones not in the user's instructions — i.e. judgment calls).
   - Dead ends or rolled-back experiments worth remembering.
   - Open questions surfaced for the user.
3. **Write `HANDOFF.md`** if work continues into the next session and there's nontrivial context to pass. Include the literal prompt-block to paste at the top of the next session.
4. **Update [book-plan.md](../../book-plan.md)** if the plan shifted (chapter scope, structure, voice, etc.). The plan is living.
5. **Surface the session briefly to the user** before closing — what's where, what to read, what to confirm or push back on.

## Session vs. chapter numbering

Don't conflate them. Session number is the conversation count; chapter number is the book chapter. A chapter draft might span sessions 005, 008, 009 if it gets revised twice. The session README's slug should always include the chapter number when relevant (`007-chapter-04-pointers`) so the relationship is grep-able.
