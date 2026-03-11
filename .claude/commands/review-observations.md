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

Present in batches of 3-4, or all at once if the user prefers. Use AskUserQuestion for decisions.

### 4. Save after each batch
Update the session file after each decision batch to avoid losing progress.

### 5. Final summary
Show: "{n} approved, {n} rejected, {n} still pending"

If 2+ approved, suggest `/synthesize`. If pending remain, note they can re-run this command.
