# /analyse - Analyse a session's source material

Single entry point for all source analysis. Asks for source type upfront, then runs the appropriate pipeline.

## Inputs

$ARGUMENTS - Path to a file in `Research/inbox/`, a session file, a source file, or a URL (http/https). If omitted, auto-scan the inbox.

---

## Interaction pattern

Wherever this skill asks the user to choose between options, use `AskUserQuestion` with option lists. This enables arrow-key navigation. Never present choices as a written list with "type 1/2/3" or similar.

**Manual mode checkpoint:** After each major step in manual mode, ask:
> Use `AskUserQuestion`: "Continue to next step?" with options:
> - **Continue** (default, first option)
> - **Stop here**

If the user picks Stop, summarize what was done and exit.

---

## Step 0: Auto mode check

Use `AskUserQuestion`: "How should I run this?"
- **Manual** - I ask before each stage (default)
- **Yolo** - I make all decisions, no confirmations

Set `{auto_mode}` to `false` (manual) or `true` (yolo).

---

## Step 1: Source type

Use `AskUserQuestion`: "What type of source is this?"
- **Transcript** - verbatim interview or usability session recording
- **Observer / interviewer notes** - third-person notes taken during or after a session
- **Report or article** - third-party research, PDF, or online article

Set `{source_type}` to one of: `transcript`, `notes`, `report`.

**If `{auto_mode}` is true,** infer from filename and content instead of asking:
- Filename contains "transcript" → `transcript`
- Filename contains "notes", "observer", "interviewer" → `notes`
- Input is a URL, PDF, or report-like structure → `report`
- Otherwise → ask

---

## Pipeline A: Session (transcript or observer notes)

*Follow this pipeline when `{source_type}` is `transcript` or `notes`.*

### A1. Find the source material

**If $ARGUMENTS is empty (no path given):**
- Glob `Research/inbox/` for all files (exclude `processed/` subfolder).
- If no files found: tell the user the inbox is empty and exit.
- If one file found: use it automatically, tell the user.
- If multiple files found: use `AskUserQuestion` to let the user pick which file(s) to process. Allow selecting all or a subset.

**If $ARGUMENTS points to a file in `Research/inbox/`:**
- Use that file. This is a new, unprocessed session source.

After identifying the source file(s):
- Glob `Research/Studies/*/study.md` to get available studies.
- Use `AskUserQuestion`: "Which study does this session belong to?"
  - List each study by name (one option per study)
  - Add **"+ New study"** as the last option
- If the user picks **"+ New study"**: run the `/new-study` flow to create the study, then return here and continue with the newly created study.
- Determine the next session number by checking existing `session-*.md` files in that study's `sessions/`.
- Create:
  - `sessions/session-{nn}.md` from the session template
  - `sessions/session-{nn}-sources/`
- Copy the inbox file(s) to `sessions/session-{nn}-sources/`. Name by type: `transcript.md` or `notes.md`. If multiple notes files, name them `notes-1.md`, `notes-2.md`, etc.
- Link the session to the study in the frontmatter.
- Move inbox files to `processed/` at the end (step A7).

**If $ARGUMENTS points to a session file or source file:**
- Use existing session structure. See CLAUDE.md "File Discovery".
- Check what source files already exist in `session-{nn}-sources/`. Copy in any new files. If re-running extraction, use all source files present.

*Manual mode checkpoint.*

### A2. Auto-fill participant info

Read the source content. Infer what you can:
- **role** (merchant, consumer, partner, agency, developer, etc.)
- **country** (location mentions, currency, language cues)
- **age-range** and **gender** (if mentioned)

If `{auto_mode}` is false: present inferences and use `AskUserQuestion` "Does this participant info look right?" with options:
- **Looks good - continue**
- **Let me correct something** (if chosen, ask freeform for corrections)

Update session frontmatter with confirmed or corrected values.
If `{auto_mode}` is true: write inferences directly to frontmatter.

*Manual mode checkpoint.*

### A3. PII scrub

Run `/scrub-pii` on each source file. Pass `{auto_mode}`.

Each source file gets its own scrubbed copy: `{filename}-scrubbed.md` in the same folder.

If `{auto_mode}` is false: after showing proposed redactions, use `AskUserQuestion`: "Scrub and save?"
- **Yes - scrub and save**
- **No - skip scrubbing**

*Manual mode checkpoint.*

### A4. Load taxonomy and study context

Follow CLAUDE.md "Taxonomy Loading" pattern. Read research questions from study.md - these become `{focus_areas}` for extraction.

If study.md frontmatter contains a `pico` block, append the `outcome` field to `{focus_areas}`.

If the study has no taxonomy set and `{auto_mode}` is false:
- Glob `taxonomy/*.yaml` in the Attic project root
- Use `AskUserQuestion`: "Which taxonomy should I use?"
  - **vipps-mobilepay** - domain-specific tags for Vipps MobilePay (show as default/first)
  - **core** - universal tags only
  - (any other .yaml files found)

*Manual mode checkpoint.*

### A5. Extract per source

For each scrubbed source file, load the matching prompt:
- `{source_type}` is `transcript` → read `prompts/extract-observations-transcript.md`
- `{source_type}` is `notes` → read `prompts/extract-observations-notes.md`

**If a session has both a transcript and notes,** run extraction on each separately using its own matching prompt. Each produces its own set of observations. Do NOT merge sources into a single extraction pass.

Apply the prompt with:
- `{taxonomy_section}`: merged taxonomy from step A4
- `{max_observations}`: 30 (across all sources combined)
- `{source_label}`: the source type (e.g. "transcript", "observer notes", "interviewer notes")
- `{content}`: scrubbed content of that source file
- `{focus_areas}`: research questions from study.md, or "general exploration"

### A6. Format and write

Format each observation per CLAUDE.md "Observation Format" (all marked `[?]`). Write all observations into the session file under `## Observations`.

**If `{auto_mode}` is true:** Auto-approve all observations by bulk-replacing `[?]` with `[x]` in the session file:

```python
import re
with open(session_path, 'r') as f:
    content = f.read()
content = re.sub(r'(### )\[\?\]', r'\1[x]', content)
with open(session_path, 'w') as f:
    f.write(content)
```

Then immediately run synthesis: read `prompts/synthesize-session.md` and apply it to all approved observations. Format each insight per CLAUDE.md "Session Insight Format". Write insights under `## Session Insights` in the session file.

**If `{auto_mode}` is false:** Leave all observations as `[?]`. Leave `## Session Insights` empty. Insights are generated separately by `/synthesize` after review.

### A7. Move to processed

If the source came from `Research/inbox/`, move it to `Research/inbox/processed/`. Create `processed/` if it doesn't exist.

### A8. Summary

Show: participant info confirmed, PII result, observation count by type and source, top tags.

If `{auto_mode}` is true: also show insight count.

If `{auto_mode}` is false, use `AskUserQuestion`: "What would you like to do next?"
- **Run /review-observations now**
- **Done for now**

If the user picks review-observations, immediately run that skill on the session file.

---

## Pipeline B: Report or article

*Follow this pipeline when `{source_type}` is `report`.*

### B1. Choose study context

Glob `Research/Studies/*/study.md` to list available studies.

Use `AskUserQuestion`: "What study context should I use for extraction?"
- List each study by name
- **Custom focus** - I'll describe what I'm looking for
- **General** - no study context, extract broadly

Store as `{study_context}`. If "Custom focus" is chosen, ask freeform for the focus description.

*Manual mode checkpoint.*

### B2. Gather report metadata

**If $ARGUMENTS is a URL:** Use the `defuddle` skill to fetch and clean the article.

**If $ARGUMENTS is a file path:** Read the file (first 20 pages for PDFs, use `pages` parameter for large PDFs).

Infer:
- **Report name** (a descriptive label)
- **Source organization** (who produced it)
- **Source date** (when published)
- **Methodology** (e.g. "survey of 500 users", "usability study with 12 participants")
- **Scope** (product area, market, segment)
- **Source URL** (if input was a URL)
- **Related studies** (from step B1, if any)

If `{auto_mode}` is false: present inferred metadata and use `AskUserQuestion`: "Does this metadata look right?"
- **Looks good - continue**
- **Let me correct something**

If `{auto_mode}` is true: write inferences directly.

*Manual mode checkpoint.*

### B3. Load taxonomy

Follow CLAUDE.md "Taxonomy Loading" pattern.

If `{auto_mode}` is false:
- Glob `taxonomy/*.yaml` in the Attic project root
- Use `AskUserQuestion`: "Which taxonomy should I use?"
  - **vipps-mobilepay** - domain-specific tags (default/first)
  - **core** - universal tags only
  - (any other .yaml files found)

### B4. Create report structure

Find the vault Research path (see CLAUDE.md "File Discovery"). Create:
```
Research/Reports/{Report-Name}/
  report.md              # from templates/report.md
  report-sources/
    original.{ext}       # copy of source file (file input only)
    content.md           # markdown content
```

Fill in the template frontmatter and About section with metadata from step B2.

### B5. Get content as markdown

- **URL:** Use the `defuddle` skill to fetch and clean. Save output as `content.md` in `report-sources/`. No `original.{ext}` needed.
- **PDF:** Read with the Read tool. Convert to clean markdown - preserve headings, lists, tables. Remove headers/footers and page numbers. Save as `content.md`.
- **Markdown:** Copy as-is to `content.md`.
- **Text:** Copy with minimal markdown formatting to `content.md`.

### B6. Extract observations

Read `prompts/extract-observations-report.md`. Apply with:
- `{taxonomy_section}`: from step B3
- `{max_observations}`: 30
- `{content}`: converted markdown from step B5
- `{source_org}`, `{source_date}`, `{methodology}`: from step B2
- `{focus_areas}`: study RQs if study context chosen; user's text if custom; "general exploration" if general

When study context is provided, note which RQ or hypothesis each observation speaks to where relevant.

### B7. Format and write

Format each observation per CLAUDE.md "Observation Format" (all marked `[?]`). Write into `report.md` under `## Observations`.

**Do NOT run review or synthesis.** Remind to run `/review-observations`.

### B8. Move to processed

If the source came from `Research/inbox/`, move it to `Research/inbox/processed/`. Create `processed/` if it doesn't exist.

Skip if the source was a URL or a file outside the inbox.

### B9. Summary

Show: report metadata confirmed, observation count by type, top tags.

If `{auto_mode}` is false, use `AskUserQuestion`: "What would you like to do next?"
- **Run /review-observations now**
- **Done for now**
