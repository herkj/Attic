# First Day on Attic

This walkthrough is for a colleague who has never used Attic before.

Time needed: 45-60 minutes.

## Before starting

- Install and open Claude Code.
- Have access to an Obsidian vault where Attic can create `Research/`.
- Prepare one transcript or notes file that can be used as test input.

## Step 1 - Clone and open

1. Clone the Attic repository into the vault root:
   - expected location: `{VaultRoot}/Attic/`
2. Open Claude Code in the `Attic` folder.
3. Confirm the repo contains `.claude/commands/` and `prompts/`.

## Step 2 - Run setup

1. Run `/setup-attic`.
2. Choose the default vault location unless there is a specific reason to use custom.
3. When asked about domain taxonomy:
   - choose **Yes - generate from websites** if setting up fresh,
   - or skip and use core taxonomy for a first test.

Expected result:
- `Research/` is created beside `Attic/`.
- inbox/studies/reports folders exist.

## Step 3 - Add one source file

1. Put a single transcript or notes file in `Research/inbox/`.
2. Keep the first test simple (one file only, no batch).

## Step 4 - Run `/analyse` in Manual mode first

1. Run `/analyse`.
2. Choose **Manual** at the run-mode question.
3. Choose source type.
4. Select or create a study when prompted.
5. Continue through each checkpoint.

What to pay attention to:
- How participant info is inferred and confirmed.
- How `/scrub-pii` assigns pseudonym and creates `-scrubbed` source files.
- How observations are written as `[?]` pending under `## Observations`.

## Step 5 - Inspect output in Obsidian

Open the generated session file under `Research/Studies/{Study}/sessions/`.

Verify:
- Frontmatter has participant info and `learning_generation`.
- Observations include Type, Evidence type, Source, Tags, and Evidence.
- Source folder exists as `{session-name}-sources/`.

## Step 6 - Run `/review-observations`

1. Run `/review-observations` on the session file.
2. Approve at least one observation.
3. Reject at least one observation and choose a rejection reason.
4. Optionally edit one observation to see how edits are captured.

Expected result:
- statuses move from `[?]` to `[x]` or `[-]`,
- learning journal entry is appended in `Research/config/learning/journal.yaml`.

## Step 7 - Run `/synthesize`

1. Run `/synthesize` on the same session file.
2. Review `## Session Insights`.
3. Check that `Based on` links point to existing observation headings.

## Step 8 - Run `/analyse` in Yolo mode

1. Put a second file in `Research/inbox/`.
2. Run `/analyse`.
3. Choose **Yolo**.

Expected behavior:
- extraction runs without per-step confirmations,
- observations are auto-approved (`[x]`),
- session insights are generated in the same run.

## Step 9 - Try the learning loop

After enough reviewed sessions (target: 5+), run `/improve`:
- inspect history view,
- run the improvement cycle,
- review proposed examples/preferences/taxonomy suggestions.

The first day goal is not perfect metrics. The goal is to understand the loop.

## Step 10 - Where to go next

- Read `docs/handover.md` for context and ownership gaps.
- Read `docs/backlog.md` for known sharp edges.
- Read `docs/proposals/eval-harness.md` for the next major implementation.
- Read `docs/proposals/improved-output-format.md` for output architecture options.

## If something breaks

Go to `docs/faq.md` first. Most first-day issues are covered there.
