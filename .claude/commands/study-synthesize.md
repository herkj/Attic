# /study-synthesize - Cross-session study synthesis

Aggregates session insights across all sessions to produce study-level findings with prevalence. Optionally answers research questions.

## Inputs

$ARGUMENTS - Study name or path to study.md

## Steps

### 1. Load study
Find study.md (see `docs/skill-patterns.md` - File Discovery). Read research questions and session count.

### 2. Collect session insights
Glob for `session-*.md` in the sessions/ directory. Read each, parse ## Session Insights. Skip sessions without insights (but warn the user). Assign IDs: `si-{session}-{n}`.

### 3. Run synthesis
Read `prompts/synthesize-study.md`. Fill placeholders (`{{question}}`, `{{total}}`, `{{with_insights}}`, `{{count}}`, `{{json_array_of_insights}}`). Before sending: verify no `{{...}}` placeholders remain in the assembled prompt (see `docs/skill-patterns.md` - Prompt Substitution Rule). Parse the response.

### 4. Validate
Every session insight ID must appear in at least one study insight. Flag any orphans.

### 5. Format and write
Format per `docs/skill-patterns.md` - Study Insight Format. Write under ## Study Insights and ## Research Question Answers in study.md. Include contradictions and limitations sections if present.

Confidence criteria are in `docs/skill-patterns.md` and `prompts/synthesize-study.md`.

### 6. Summary
Show insights, prevalence, confidence, contradictions, and limitations.
