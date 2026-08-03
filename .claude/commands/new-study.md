# /new-study - Create and set up a new research study

Interactive walkthrough to create a study with properly defined research questions, hypotheses, and method.

## Inputs

$ARGUMENTS - Study name (e.g. "Search Redesign", "First-Run Onboarding")

## Steps

### 1. Create the study folder

- Resolve the vault root (see `docs/skill-patterns.md` - Vault Root Resolution)
- Create `{vaultRoot}/Studies/{Study Name}/` folder
- Create `sessions/` subfolder inside it
- Copy the study template from `templates/study.md`, filling in `{{study-name}}` and `{{date}}` (fill `{{language}}` in step 2b, or leave blank)

### 2. Choose taxonomy

Glob for domain files: `taxonomy/*.yaml` in the Attic project root and `{vaultRoot}/config/taxonomy/*.yaml` in the vault.

Use `AskUserQuestion`: "Which taxonomy should I use?"
- (any domain `.yaml` files found - show these first if present)
- **core** - universal tags only (default if no domain files exist)

Set the `taxonomy` frontmatter field.

### 2b. Primary language (optional)

Ask: "What language is the research mostly in? (optional - press enter to auto-detect)"

Attic auto-detects and preserves the source language of every source file regardless of this setting, and never translates. This field is only a hint that gets surfaced to extraction. Replace the `{{language}}` placeholder in study.md: set it to the user's answer (e.g. `en`, `es`, `ja`, or a plain language name), or to an empty value if they skip. Never leave the literal `{{language}}` placeholder in the file.

### 3. Background

Ask: "In 1-2 sentences, what triggered this study? What do you want to learn?"

Write the response under ## Background.

### 4. Research questions

Ask the user to list their research questions. Coach briefly:
- Good RQs start with "How", "What", "Why", or "When"
- They should be answerable through the chosen method
- 3-5 questions is typical; fewer is fine

Write under ## Research Questions. Also populate the `research-questions` frontmatter array.

### 4b. PICO structure (optional)

Ask: "Do you want to define a structured scope for this study? This helps keep extraction focused. (optional - press enter to skip)"

If the user says yes or provides any input, prompt for each PICO component:
- **Population:** Who are we studying? (e.g. "new users, first 30 days after signup")
- **Phenomenon:** What behavior, experience, or product area are we exploring? (e.g. "first-run onboarding")
- **Context:** Under what conditions or constraints? (e.g. "no prior product experience, self-service signup")
- **Outcome:** What do we want to learn or be able to say? (e.g. "Where participants get stuck and why")

Write the PICO block under ## Scope in study.md:
```markdown
## Scope
- **Population:** {value}
- **Phenomenon:** {value}
- **Context:** {value}
- **Outcome:** {value}
```

Also store as structured frontmatter:
```yaml
pico:
  population: "..."
  phenomenon: "..."
  context: "..."
  outcome: "..."
```

In `/analyse` step 4, when `pico` is present in study.md frontmatter, pass the Outcome field as an additional `{{focus_areas}}` instruction to the extraction prompt: "Focus extraction on: {pico.outcome}". This keeps observations anchored to what the study is trying to answer.

### 5. Hypotheses

Use `AskUserQuestion`: "Do you have any hypotheses going in?"
- **Skip - no hypotheses yet**
- **Yes - I have some**

If "Yes", ask freeform. Write under ## Hypotheses. If skipped, remove the placeholder dash.

### 6. Method

Ask about:
- **Type:** interview, usability test, survey, diary study, or other
- **Participants:** who and how many (e.g. "8-10 new users")

Write under ## Method.

### 7. Summary

Show the completed study.md contents. Confirm with user. Remind them to drop raw files in `{vaultRoot}/inbox/` and run `/analyse` to start processing sessions.
