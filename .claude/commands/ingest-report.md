# /ingest-report - Ingest a third-party research report

Takes a PDF, markdown, text report, or a URL to an online article and extracts taxonomy-tagged observations into the Attic research repository.

## Inputs

$ARGUMENTS - Path to the report file (PDF, .md, or .txt), or a URL (http/https) to an online article

## Steps

### 1. Choose study context

Before fetching or reading anything, ask the user which study context to use for extraction. Glob `Research/Studies/*/study.md` to list available studies.

Present the options:
- **A specific study** - list each available study by name. If chosen, read that study's `study.md` to extract research questions and hypotheses. These will be used to focus and frame observations.
- **Custom focus** - user types a free-text description of what to focus on (e.g. "merchant onboarding friction" or "younger users and Klarna")
- **General** - no study context, extract broadly relevant observations

Store the chosen context as `{study_context}` for use in step 5.

### 2. Gather report metadata

**If $ARGUMENTS is a URL:** Use the `defuddle` skill to fetch and clean the article. Extract title, author, publication, and date from the fetched content.

**If $ARGUMENTS is a file path:** Read the file (first 20 pages for PDFs) to understand what it covers.

Then infer:
- **Report name** (a descriptive label)
- **Source organization** (who produced it)
- **Source date** (when published)
- **Methodology** (e.g. "survey of 500 users", "usability study with 12 participants")
- **Scope** (product area, market, segment)
- **Source URL** (if input was a URL, include it; otherwise omit)
- **Related studies** - set to the study chosen in step 1 (if any), otherwise leave empty

Present all inferred metadata to the user and **always ask for confirmation before proceeding**. Let them correct, add, or remove any field. Skip asking for fields the user has already provided explicitly.

### 3. Choose taxonomy

List available taxonomy files by globbing `taxonomy/*.yaml` in the Attic project root. Ask which domain taxonomy to use, or "core" only.

### 4. Create report structure

Find the vault Research path (see CLAUDE.md "File Discovery"). Create:
```
Research/Reports/{Report-Name}/
  report.md              # from templates/report.md
  report-sources/
    original.{ext}       # copy of source file (file input only)
    content.md           # markdown content
```

Fill in the template frontmatter and About section with metadata from step 2. If input was a URL, populate the `source-url` frontmatter field. If a study was chosen in step 1, populate `related-studies`.

### 5. Get content as markdown

- **URL:** Use the `defuddle` skill to fetch and clean the article. Save the output as `content.md` in `report-sources/`. No `original.{ext}` file is needed.
- **PDF:** Read with the Read tool (use `pages` parameter for large PDFs, 20 pages per chunk). Convert to clean markdown - preserve headings, lists, tables. Remove headers/footers and page numbers. Save as `content.md`.
- **Markdown:** Copy as-is to `content.md`.
- **Text:** Copy with minimal markdown formatting to `content.md`.

### 6. Load taxonomy and extract observations

Follow CLAUDE.md "Taxonomy Loading" pattern. Read `prompts/extract-report-observations.md` from the Attic project root. Apply it with:
- {taxonomy_section} from taxonomy loading
- {max_observations}: 30
- {content}: the converted markdown
- {source_org}, {source_date}, {methodology}: from step 2
- {focus_areas}: if study context was chosen, summarize the study's RQs and hypotheses as focus areas; if custom, use the user's text; if general, use "general exploration"

When study context is provided, frame each observation's interpretation in relation to the study's RQs and hypotheses - not just as generic market data. Note which RQ or hypothesis each observation speaks to where relevant.

### 7. Format and write

Format each observation per CLAUDE.md "Observation Format" (all marked `[?]`). Write into report.md under ## Observations.

**Do NOT run review or synthesis.** Remind the user to run `/review-observations` followed by `/synthesize`.

### 8. Move original to processed

If the source came from `Research/inbox/`, move it to `Research/inbox/processed/`. Create the `processed/` folder if it doesn't exist.

Skip this step if the source was a URL or a file outside the inbox.

### 9. Summary

Show: report metadata, observation count by type, top tags. Note that report observations will be included in `/cross-study` analysis once that skill is built.
