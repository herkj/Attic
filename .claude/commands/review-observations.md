# /review-observations - Review extracted observations

Interactive approval of pending observations. The human-in-the-loop quality gate before synthesis.

## Inputs

$ARGUMENTS - Path to session file

## Steps

### 1. Load observations
Find the session file (see `docs/skill-patterns.md` - File Discovery). Parse ## Observations. Count by status: `[?]` pending, `[x]` approved, `[-]` rejected.

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

**When the user picks Reject,** immediately follow up with a rejection reason using `AskUserQuestion`: "Why are you rejecting this? (helps improve future extraction)"
- **Too vague** - not specific enough
- **Duplicate** - overlaps another observation
- **Not relevant** - not relevant to this study
- **Wrong type** - wrong type or classification
- **Obvious** - not insightful
- **No reason** - skip

Track the reason alongside each rejection for the learning journal (step 6).

**When the user picks Edit,** track which fields were changed (especially tag changes) for the learning journal.

### 3a. Add a new observation (on demand)

When the user wants to add an observation they noticed themselves:

1. Ask: "Describe what you observed." (free text)
2. Ask: "Do you have a quote from the source?" (optional - paste it or skip)
3. Load taxonomy (follow `docs/skill-patterns.md` - Taxonomy Loading). Suggest 2-5 relevant tags based on the description and quote. Show them with a brief reason for each suggestion.
4. Ask the user to confirm, remove, or add tags.
5. Ask: "What type is this?" - offer the standard types as options: problem, observation, preference, workaround, quote_summary, hypothesis.
6. Write the new observation to the session file in the standard format, marked `[x]` (already approved - the user wrote it).
7. Resume the review loop.

### 4. Save after each batch
Update the session file after each decision batch to avoid losing progress.

### 5. Final summary
Show: "{n} approved, {n} rejected, {n} still pending, {n} added by you"

If 2+ approved, suggest `/synthesize`. If pending remain, note they can re-run this command.

### 6. Log to learning journal

After the final summary, append a learning entry to `Research/config/learning/journal.yaml`. Create the directory and file if they don't exist.

1. Read the session file frontmatter to get the `learning_generation` value (default `0` if not set).
2. Determine the study name and session identifier from the file path.
3. Append an entry:

```yaml
- session: "Study Name / Session NN"
  date: YYYY-MM-DD
  source_type: transcript
  generation: 0
  stats:
    extracted: 24
    approved: 18
    rejected: 6
    added_by_user: 2
    edited_by_user: 3
  rejections:
    - title: "observation title"
      type: observation
      tags: [Tag1, Tag2]
      reason: "too vague"
  additions:
    - title: "user-added observation title"
      type: workaround
      tags: [Tag1]
  tag_edits:
    - observation: "observation title"
      original: [Login]
      changed_to: [Authentication, Login]
```

Only include `rejections`, `additions`, and `tag_edits` sections if there are entries for them. The `generation` comes from the session file's `learning_generation` frontmatter field.

### 7. Improvement nudge

Check for `Research/config/learning/improve-history.yaml`. Count how many journal entries exist since the last `/improve` run (by comparing dates, or counting all entries if no improve run has ever happened).

If 5 or more sessions have been reviewed since the last `/improve` run (or ever, if no run exists), show:

> "You've reviewed {n} sessions since the last /improve run ({date}). Consider running `/improve` to refine extraction quality."

If fewer than 5, or if no journal exists yet, skip this silently.
