# Contributing Skills to Attic

This guide explains how to add or modify a skill in Attic without breaking shared conventions.

## Where skills live

- Skill files: `.claude/commands/*.md`
- Prompt templates: `prompts/*.md`
- Shared conventions: `docs/skill-patterns.md`

## Skill file naming

- Use kebab-case file names, matching slash command name:
  - `.claude/commands/review-observations.md` -> `/review-observations`
- Keep one command per file.

## Required skill structure

Every skill should follow this baseline shape:

```markdown
# /skill-name - One-line purpose

## Inputs
$ARGUMENTS - what the command accepts

## Steps
### 1. ...
### 2. ...
```

Recommended additions:
- explicit stop conditions,
- explicit output summary section,
- links to `docs/skill-patterns.md` sections when shared behavior is used.

## AskUserQuestion vs free-form prompt

Use `AskUserQuestion` when:
- user must choose among known options,
- there is a branch in flow (continue/stop, yes/no, mode selection),
- consistency matters across runs.

Use free-form prompts when:
- collecting open text (study background, custom focus, hypothesis text),
- collecting corrections to inferred metadata.

Rule of thumb:
- If options are known up front, use `AskUserQuestion`.
- If content is open-ended, use free-form.

## Prompt authoring conventions

When a skill uses an AI prompt template:

1. Put template in `prompts/`.
2. Use `{{placeholder}}` for substitution variables.
3. In the skill, document what each placeholder receives.
4. Before sending the assembled prompt, verify no `{{...}}` placeholders remain.

See `docs/skill-patterns.md` -> **Prompt Substitution Rule**.

## Reuse shared patterns, do not duplicate

Do not copy-paste large convention blocks into each skill.

Reference these sections from `docs/skill-patterns.md`:
- File Discovery
- Taxonomy Loading
- Observation / Insight formats
- PII Scrubbing Rules
- Learning System

This keeps one source of truth.

## Adding a new skill checklist

1. Create `.claude/commands/<skill>.md`.
2. Add prompt file in `prompts/` if needed.
3. Reference shared patterns where applicable.
4. Ensure run flow has clear stop/continue points.
5. Add the new skill to the Skills table in `CLAUDE.md`.
6. Add or update docs if behavior affects users (`docs/faq.md`, `docs/walkthrough.md`, or proposals).

## Editing an existing skill safely

Before edits:
- read the full skill file,
- read any prompt files it uses,
- read linked sections in `docs/skill-patterns.md`.

After edits:
- scan for stale references,
- ensure placeholders are consistent,
- ensure branching logic still has a valid path for both manual and auto/yolo modes.

## Worked example: hypothetical `/summarize-session`

### Goal

Create a lightweight command that summarizes one session file without generating new observations.

### Skill sketch

```markdown
# /summarize-session - Quick summary from existing approved observations

## Inputs
$ARGUMENTS - path to a session file

## Steps
### 1. Locate file
Find the session file (see `docs/skill-patterns.md` - File Discovery).

### 2. Read approved observations
Parse `## Observations`, collect only `[x]` entries.
If none exist, stop and tell user to run `/review-observations` first.

### 3. Build summary prompt
Read `prompts/summarize-session.md`.
Fill placeholders:
- `{{approved_observations}}`
- `{{session_name}}`
Before sending, verify no `{{...}}` placeholders remain.

### 4. Write summary
Append under `## Session Summary` in the session file.

### 5. Report output
Show number of approved observations summarized.
```

### Prompt sketch

`prompts/summarize-session.md`

```markdown
Summarize this session in 5 bullets.

Session: {{session_name}}

Approved observations:
{{approved_observations}}
```

## Common mistakes to avoid

- Duplicating taxonomy loading logic inside multiple skills instead of referencing shared pattern.
- Mixing prompt placeholders (`{{var}}`) and skill-internal vars (`{var}`) without clarity.
- Adding hidden branch conditions without user confirmation in manual mode.
- Forgetting to update `CLAUDE.md` skills table after adding a command.
- Leaving legacy references to removed docs or commands.
