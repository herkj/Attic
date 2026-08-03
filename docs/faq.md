# Attic FAQ

## 1) The AI extracted weak or noisy observations. What should be done?

Run `/review-observations` and reject or edit weak entries. Add missing observations manually. After enough reviewed sessions, run `/improve` so the system can learn from rejections and edits.

## 2) PII scrub missed a name. Is the session unusable?

No. Edit the `*-scrubbed.md` file manually, then continue with `/analyse` from that scrubbed source. Do not overwrite the raw source file.

## 3) Can Attic handle English transcripts?

Yes. Prompts are written in English and can process English and Norwegian content. Keep source language as-is. Do not translate source content before analysis.

## 4) How can a study use a specific domain taxonomy?

Set `taxonomy:` in the study frontmatter (for example `core` or another domain file name). `/analyse` loads core plus the selected domain file, then merges any `Research/config/taxonomy/*.yaml` overrides.

## 5) `/analyse` stopped mid-run. Is data corrupted?

Usually no. Attic writes plain markdown files. Re-run `/analyse` on the same session or source file. Check whether expected source files exist in the `-sources/` folder and whether observations were partially written.

## 6) Why are observations marked `[?]` by default?

`[?]` means AI-suggested and pending human review. This is intentional. It preserves a human approval gate before synthesis.

## 7) Why did Yolo mode auto-approve everything?

That is expected behavior. In Yolo mode, `/analyse` runs extract -> auto-approve -> synthesize without manual checkpoints.

## 8) How many reviewed sessions are needed before `/improve` is useful?

The practical minimum is around 3 sessions. Better signal is usually around 5 or more reviewed sessions since the last improvement run.

## 9) Can multiple people use the same Research vault?

Yes, but with process discipline:
- agree on taxonomy ownership,
- avoid editing the same session file concurrently,
- review git conflicts carefully in markdown sections.

## 10) Manual edits were made in Obsidian. Will that break Attic?

Manual edits are supported as long as core structure is preserved:
- keep heading blocks (`## Observations`, `## Session Insights`),
- keep status markers (`[?]`, `[x]`, `[-]`) in observation headings,
- keep required metadata lines (`Type`, `Evidence type`, `Source`, `Tags`, `Evidence`).

## 11) Why is there a `Research/Repository/` folder if little is written there?

It is scaffolded for cross-study artifacts and future exports, but current core skills mainly write to `Studies/` and `Reports/`. See proposals for future use.

## 12) Where should future contributors start?

Start in this order:
1. `docs/handover.md`
2. `docs/walkthrough.md`
3. `docs/skill-patterns.md`
4. `docs/contributing-skills.md`
5. `docs/backlog.md`
