# /analyse - Analyse a session's source material

Full pipeline: pick up raw notes, create session structure, identify participant, scrub PII, extract observations.

## Inputs

$ARGUMENTS - Path to a file in the global `Research/inbox/` folder, a session file, or a source file

## Steps

### 0. Auto mode check

Before doing anything else, ask the user:

> "Run fully automatic? (no questions asked - I'll make all decisions for you)"

- **Yes - fully automatic:** Set `{auto_mode} = true`. Skip all confirmation prompts and "ask the user" steps throughout. Make reasonable inferences for participant info, PII, and study assignment without asking. Proceed through all steps end-to-end.
- **No - step by step:** Set `{auto_mode} = false`. Proceed with the normal interactive flow below.

### 1. Find the source material

**If $ARGUMENTS points to a file in `Research/inbox/`:**
- This is a new, unprocessed interview.
- **If `{auto_mode}` is false:** Ask the user which study this file belongs to. List existing studies found by globbing `Research/Studies/*/study.md`. Let them pick one or create a new study.
- **If `{auto_mode}` is true:** Infer the most likely study from the file name and content. If only one study exists, assign it automatically.
- Determine the next session number by checking existing `session-*.md` files in that study's `sessions/`.
- Create:
  - `sessions/session-{nn}.md` from the session template
  - `sessions/session-{nn}-sources/`
- Copy the inbox file to `sessions/session-{nn}-sources/notes.md` (or `transcript.md` if it looks like a transcript based on filename or content structure)
- Link the session to the study in the frontmatter
- **If `{auto_mode}` is false:** Ask: "Do you have other source files for this session? (e.g. observer notes, a transcript, a second interviewer's notes) You can paste additional file paths now, or drop more files in inbox and run `/analyse` on the session file later to add them." If the user provides more paths, copy each into the same `session-{nn}-sources/` folder, naming them by type: `transcript.md`, `notes.md`, `observer-notes.md`, etc. Move all provided inbox files to `processed/` at the end.

**If $ARGUMENTS points to a session file or source file:**
- Use existing session structure. See CLAUDE.md "File Discovery".
- Check what source files already exist in `session-{nn}-sources/`. If new files are being added, copy them in and name them appropriately. If re-running extraction, use all source files present.

### 2. Auto-fill participant info
Read the source content. Infer what you can about the participant:
- **role** (merchant, consumer, partner, agency, developer, etc.)
- **country** (location mentions, currency, language cues)
- **age-range** and **gender** (if mentioned)

If `{auto_mode}` is false: Present inferences and ask the user to confirm or correct. Update the session file frontmatter.
If `{auto_mode}` is true: Write inferences directly to frontmatter without asking.

### 3. PII scan
Scan source content using the PII rules in CLAUDE.md.

- **If no PII found:** Tell the user, create identical scrubbed copy, go to step 5.
- **If PII found:** If `{auto_mode}` is true, scrub all automatically and continue. If `{auto_mode}` is false, list all items with proposed replacements and ask "Scrub all and proceed?"

Save scrubbed file as `{filename}-scrubbed.md` in the same folder.

If multiple source files exist (notes.md + transcript.md), scrub each one.

### 4. Combine sources for extraction
Merge all scrubbed content with clear labels:
```
=== OBSERVER NOTES ===
{content}
=== TRANSCRIPT ===
{content}
```

### 5. Load taxonomy and study context
Follow CLAUDE.md "Taxonomy Loading" pattern. Also read research questions from study.md - these become focus areas.

### 6. Extract observations
Read `prompts/extract-observations.md` from the Attic project root. Apply it with:
- {taxonomy_section} from step 5
- {max_observations}: 30
- {content}: combined scrubbed sources from step 4
- {focus_areas}: research questions, or "general exploration"

### 7. Format and write
Format each observation per CLAUDE.md "Observation Format" (all marked `[?]`). Write into the session file under ## Observations.

**Do NOT generate Session Insights.** Leave the `## Session Insights` section empty. Insights are generated separately by `/synthesize` after the user has reviewed and approved observations with `/review-observations`.

### 8. Move original to processed
If the source came from `Research/inbox/`, move it to `Research/inbox/processed/`.

### 9. Summary
Show: participant info confirmed, PII result, observation count by type, top tags. Remind to run `/review-observations`.
