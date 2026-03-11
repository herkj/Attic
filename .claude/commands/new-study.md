# /new-study - Create and set up a new research study

Interactive walkthrough to create a study with properly defined research questions, hypotheses, and method.

## Inputs

$ARGUMENTS - Study name (e.g. "Checkout Redesign", "Merchant Onboarding")

## Steps

### 1. Create the study folder

- Find the vault Research/Studies/ directory (see CLAUDE.md "File Discovery")
- Create `Research/Studies/{Study Name}/` folder
- Create `sessions/` subfolder inside it
- Copy the study template from `templates/study.md`, filling in `{{study-name}}` and `{{date}}`

### 2. Choose taxonomy

List available taxonomy files by globbing `taxonomy/*.yaml` in the Attic project root. Show them to the user:
- `core` (always loaded - universal tags)
- Any domain-specific files (e.g. `vipps-mobilepay`)

Ask which domain taxonomy to use, or "core" only. Set the `taxonomy` frontmatter field.

### 3. Background

Ask: "In 1-2 sentences, what triggered this study? What do you want to learn?"

Write the response under ## Background.

### 4. Research questions

Ask the user to list their research questions. Coach briefly:
- Good RQs start with "How", "What", "Why", or "When"
- They should be answerable through the chosen method
- 3-5 questions is typical; fewer is fine

Write under ## Research Questions. Also populate the `research-questions` frontmatter array.

### 5. Hypotheses

Ask: "Do you have any hypotheses going in? (optional but useful)"

If yes, write under ## Hypotheses. If none, remove the placeholder dash.

### 6. Method

Ask about:
- **Type:** interview, usability test, survey, diary study, or other
- **Participants:** who and how many (e.g. "8-10 SMB merchants in Norway")
- **Duration:** estimated per session (e.g. "45-60 min")

Write under ## Method.

### 7. Summary

Show the completed study.md contents. Confirm with user. Remind them to drop raw files in `Research/inbox/` and run `/analyse` to start processing sessions.
