# /auto-insight - Fully automatic session pipeline

Takes a raw file from inbox and runs the full analysis pipeline end-to-end with no interruptions. Only asks one question: which study to assign.

## Inputs

$ARGUMENTS - Path to a file in `Research/inbox/`

## Steps

### 1. Assign study

List all existing studies by globbing `Research/Studies/*/study.md`. Ask the user:

> "Which study does this session belong to?"

This is the only question asked. Wait for the answer before proceeding.

Read the chosen study's `study.md` to extract:
- Research questions (used as focus areas for extraction)
- Taxonomy field (used to confirm domain taxonomy)

### 2. Create session structure

- Determine the next session number by counting existing `session-*.md` files in the study's `sessions/` folder.
- Create `sessions/session-{nn}.md` from `templates/session.md`, filling in `{{date}}`, `{{study-name}}`, and `{{number}}`.
- Create `sessions/session-{nn}-sources/` folder.
- Copy the inbox file into that folder as `notes.md` (or `transcript.md` if it looks like a transcript based on the filename or content structure).

### 3. Infer participant info

Read the source file. Infer without asking:
- **role** (consumer, merchant, partner, etc.) - default to "consumer" if unclear
- **country** - infer from language, currency, place mentions; default to "Norway" if unclear
- **age-range** - only if clearly stated
- **gender** - only if clearly stated

Write inferred values directly to the session file frontmatter. Set participant id to `P{{number}}` matching session number.

### 4. PII scrub

Scan the source using PII rules from CLAUDE.md. Apply all redactions automatically without asking:
- Personal names → [Participant] / [Interviewer] / [Person]
- Emails → [Email]
- Phone numbers → [Phone]
- Addresses → [Address]
- ID numbers → [ID]
- Slack @mentions with real names → [Person]

Save scrubbed file as `{filename}-scrubbed.md` in the same sources folder. If both `notes.md` and `transcript.md` exist, scrub each one.

### 5. Load taxonomy

1. Find Attic project root via Glob for `**/taxonomy/core.yaml`
2. Always load `taxonomy/core.yaml`
3. Always also load `taxonomy/vipps-mobilepay.yaml`
4. Merge and format as flat list per category for use in extraction

### 6. Extract observations

Read `prompts/extract-observations.md` from the Attic project root. Apply it with:
- `{taxonomy_section}`: merged taxonomy from step 5
- `{max_observations}`: 30
- `{content}`: scrubbed source content from step 4
- `{focus_areas}`: research questions from the chosen study's `study.md`

Format each observation per CLAUDE.md "Observation Format", all marked `[?]`.

Write all observations into the session file under `## Observations`.

### 7. Move original to processed

Move the original inbox file to `Research/inbox/processed/`. Create `processed/` if it doesn't exist.

### 8. Auto-approve all observations

Use Python to bulk-replace all `[?]` status markers with `[x]` in the session file:

```python
import re
with open(session_path, 'r') as f:
    content = f.read()
content = re.sub(r'(### )\[\?\]', r'\1[x]', content)
with open(session_path, 'w') as f:
    f.write(content)
```

### 9. Synthesize

Read `prompts/synthesize-session.md` from the Attic project root. Run synthesis on all approved observations. Format each insight per CLAUDE.md "Session Insight Format". Write insights under `## Session Insights` in the session file.

### 10. Summary

Show:
- Session file created
- Participant info inferred
- PII: items scrubbed (or "none found")
- Observation count by type
- Insight count
- Top tags
