# /synthesize - Session-level insight synthesis

Groups approved observations into higher-level findings for a single session.

## Inputs

$ARGUMENTS - Path to session file

## Steps

### 1. Load approved observations
Find session file (see `docs/skill-patterns.md` - File Discovery). Collect only `[x]` observations. Need at least 2. Warn if `[?]` pending observations exist.

### 2. Check for existing insights
If ## Session Insights already has content, ask user: replace or skip?

### 3. Format for prompt
Assign each observation an ID (obs-1, obs-2, ...) and format as:
```
[obs-1] (type) [source] [evidenceType] Title | Text | Evidence: "quote" | Tags: tag1, tag2
```
Include `evidenceType` (behavioral/attitudinal) from the observation's **Evidence type** field. If the field is absent on older observations, infer: transcript quotes = behavioral, observer/interviewer notes = attitudinal.

### 4. Run synthesis
Read `prompts/synthesize-session.md`. Send formatted observations. Parse the response.

### 5. Format and write insights
For each insight, format per `docs/skill-patterns.md` - Session Insight Format:
- Tags = union of supporting observations' tags (deduplicated)
- Based on = wikilinks to observation headings: `[[#Title]]`

Write under ## Session Insights in the session file.

### 6. Summary
Show insights with observation counts. If multiple sessions in study, suggest `/study-synthesize`.
