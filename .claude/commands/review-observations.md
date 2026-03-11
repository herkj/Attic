# /review-observations - Review extracted observations

Interactive approval of pending observations. The human-in-the-loop quality gate before synthesis.

## Inputs

$ARGUMENTS - Path to session file

## Steps

### 1. Load observations
Find the session file (see CLAUDE.md "File Discovery"). Parse ## Observations. Count by status: `[?]` pending, `[x]` approved, `[-]` rejected.

If no pending observations, say so and stop.

### 2. Present summary
Show: "{n} pending, {n} approved, {n} rejected"

### 3. Walk through each pending observation
For each `[?]` observation, show it with type, tags, evidence, and interpretation. Ask the user:
- **Approve** - mark `[x]`
- **Reject** - mark `[-]`
- **Edit** - modify text/type/tags, then mark `[x]`
- **Skip** - leave as `[?]`
- **Add new** - pause the review loop and go to step 3a, then resume

Present in batches of 3-4, or all at once if the user prefers. Use AskUserQuestion for decisions.

Also offer "Add new observation" as a standing option at any point during the review - not just between batches.

### 3a. Add a new observation (on demand)

When the user wants to add an observation they noticed themselves:

1. Ask: "Describe what you observed." (free text)
2. Ask: "Do you have a quote from the source?" (optional - paste it or skip)
3. Load taxonomy (follow CLAUDE.md "Taxonomy Loading" pattern). Suggest 2-5 relevant tags based on the description and quote. Show them with a brief reason for each suggestion.
4. Ask the user to confirm, remove, or add tags.
5. Ask: "What type is this?" - offer the standard types as options: problem, observation, preference, workaround, quote_summary, hypothesis.
6. Write the new observation to the session file in the standard format, marked `[x]` (already approved - the user wrote it).
7. Resume the review loop.

### 4. Save after each batch
Update the session file after each decision batch to avoid losing progress.

### 5. Final summary
Show: "{n} approved, {n} rejected, {n} still pending, {n} added by you"

If 2+ approved, suggest `/synthesize`. If pending remain, note they can re-run this command.
