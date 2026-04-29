# Attic

A human-in-the-loop AI assistant for UX research. Drop in interview notes, get structured observations and insights - all traceable back to evidence.

New here? Start with `docs/handover.md`.

## Get started

1. Clone this repo anywhere
2. Open Claude Code in the Attic folder
3. Run `/setup-attic`

## How it works

Each interview is a **session**. A session can have multiple source files - a transcript, your own notes, observer notes from a colleague. Attic reads all of them together before extracting observations, so evidence from multiple sources gets cross-referenced into stronger, better-supported insights.

**The standard workflow:**

1. Drop source files in `Research/inbox/`
2. Run `/analyse` - it asks which study the session belongs to, then asks if you have additional source files (transcript + notes + observer notes all go in together)
3. Run `/review-observations` - approve, reject, edit, or add your own observations
4. Run `/synthesize` - turns approved observations into session insights

Once you have multiple sessions, run `/study-synthesize` to find patterns across all of them.

**Adding sources later:** If you get a transcript after you've already run `/analyse` on your notes, just run `/analyse` again on the session file - it will pick up any new files and re-extract with the full set of sources.

**Skip the review step:** `/analyse` asks "Manual or Yolo?" upfront. Pick **Yolo** to run the full pipeline (extract → auto-approve → synthesize) without confirmations - you only choose which study to assign.