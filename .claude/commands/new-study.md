# /new-study - Create and set up a new research study

Interactive walkthrough to create a study with properly defined research questions, hypotheses, and method.

## Inputs

$ARGUMENTS - Study name (e.g. "Checkout Redesign", "Merchant Onboarding")

## Steps

### 1. Create the study folder

- Find the vault Research/Studies/ directory (see `docs/skill-patterns.md` - File Discovery)
- Create `Research/Studies/{Study Name}/` folder
- Create `sessions/` subfolder inside it
- Copy the study template from `templates/study.md`, filling in `{{study-name}}` and `{{date}}`

### 2. Choose taxonomy

Glob `taxonomy/*.yaml` in the Attic project root.

Use `AskUserQuestion`: "Which taxonomy should I use?"
- **vipps-mobilepay** - domain-specific tags for Vipps MobilePay (show first as default if file exists)
- **core** - universal tags only
- (any other .yaml files found)

Set the `taxonomy` frontmatter field.

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
- **Population:** Who are we studying? (e.g. "Norwegian SME merchants, first 30 days after signup")
- **Phenomenon:** What behavior, experience, or product area are we exploring? (e.g. "Vipps PoS onboarding")
- **Context:** Under what conditions or constraints? (e.g. "no prior Vipps experience, self-service signup")
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
- **Participants:** who and how many (e.g. "8-10 SMB merchants in Norway")

Write under ## Method.

### 7. Summary

Show the completed study.md contents. Confirm with user. Remind them to drop raw files in `Research/inbox/` and run `/analyse` to start processing sessions.
