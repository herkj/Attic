# Attic Backlog

Ideas that are interesting but not yet prioritized. Each entry includes what it is, how it differs from today, and where the idea came from.

---

## EARS Syntax for Extraction Prompts

**What it is:** Rewrite the observation extraction instructions in `/analyse` (and the `prompts/extract-observations.md` prompt) using EARS-style conditional rules instead of descriptive prose.

EARS (Easy Approach to Requirements Syntax) is a Rolls-Royce requirements engineering methodology. Rules take the form:
- "When [trigger], the system shall [action]"
- "While [condition], if [event], the system shall [action]"

**How it differs from today:** The current `extract-observations.md` prompt describes what good observations look like. EARS rewrites those descriptions as testable conditional rules. Example:
- Today: "Extract workarounds when participants describe doing something unexpected to achieve a goal"
- EARS: "When a participant describes a sequence of steps that substitutes for a direct feature, extract an observation with type: workaround and include the full quote as evidence"

The practical difference is debuggability - if Claude misses an observation or extracts the wrong type, you can point to which rule was violated.

**Source:** `prompt-optimizer` skill, daymade/claude-code-skills repo

---

## PICO Research Question Formalization in /new-study

**What it is:** Extend `/new-study` to prompt for structured research questions using the PICO framework (adapted from systematic literature review methodology):
- **P** - Population: who are we studying? (e.g. "Norwegian SME merchants, first 30 days")
- **I** - Phenomenon: what behavior or experience? (e.g. "Vipps PoS onboarding")
- **C** - Context: under what conditions? (e.g. "without prior Vipps experience")
- **O** - Outcome: what do we want to learn? (e.g. "where they get stuck and why")

Store the structured form in `study.md` alongside the free-text research questions.

**How it differs from today:** `/new-study` currently asks for research questions as free text. With PICO, the extraction prompt in `/analyse` would also receive the PICO structure as a focus lens - keeping observations anchored to what the study is actually trying to answer, not just "anything interesting."

**Practical consideration:** This adds a step to study setup. Worth it only if studies currently produce too many off-topic observations. Gate this behind an optional "add structured research questions?" prompt.

**Source:** `literature-review` skill, K-Dense-AI/claude-scientific-skills repo

---

## PRISMA-Style Coverage Tracking

**What it is:** Add a transparency layer to `study.md` (and optionally session files) that tracks research pipeline coverage:
- How many raw observations were extracted per session
- How many were approved vs. rejected in `/review-observations`
- Approval rate per observation type
- Which research questions have supporting observations vs. gaps

Example output block in `study.md`:
```
## Coverage
| Session | Extracted | Approved | Rejected | Approval rate |
|---------|-----------|----------|----------|---------------|
| 01      | 24        | 18       | 6        | 75%           |
| 02      | 31        | 22       | 9        | 71%           |
```

**How it differs from today:** No coverage tracking exists. The study summary at the end of `/study-synthesize` gives a count but nothing is persisted or comparable across sessions.

**Source:** `peer-review` skill structure, PRISMA flow diagram pattern, K-Dense-AI/claude-scientific-skills repo

---

## /trace-insight Skill

**What it is:** A reverse-lookup skill that walks backward from a study-level insight through the evidence chain to raw source quotes.

Usage: `/trace-insight "Participants expect real-time settlement"`

Output: study insight -> which sessions it appeared in -> which session insights -> which observations -> which exact quotes in which source file.

**How it differs from today:** The `[[wikilinks]]` chain already makes this theoretically possible in Obsidian - you can click from a study insight to session insights to observations. But `/trace-insight` would render the full chain in one view, useful when presenting findings to stakeholders who ask "but how do you know that?"

**Source:** `root-cause-tracing` pattern, obra/superpowers (via hesreallyhim/awesome-claude-code)

---

## Behavioral vs. Attitudinal Confidence Flag

**What it is:** Extend the confidence level scheme (currently high/medium/low) with an evidence type marker per insight:
- `behavioral` - based on direct observation of what someone did or quotes describing action
- `attitudinal` - based on stated preferences, opinions, or beliefs

The distinction matters because attitudinal evidence (what people say they do) is weaker than behavioral evidence (what they actually did or described doing). An insight based only on attitudinal evidence should be downgraded or flagged.

**How it differs from today:** Confidence is currently assigned at synthesis time based on session count. This adds a second axis: the quality of the evidence type, not just the quantity.

**Practical limitation:** Most of Attic's input is notes and transcripts, not behavioral logs. The distinction is not always clear from notes alone. A pragmatic version: if an insight is only supported by observer notes (no transcript quotes), auto-flag it as `attitudinal`. If it has direct quotes showing behavior, it can be `behavioral`.

**Source:** GRADE and Cochrane Risk of Bias frameworks, `scientific-critical-thinking` skill, K-Dense-AI/claude-scientific-skills repo

---

## Longitudinal Participant Tracking

**What it is:** Compare the same participant across multiple sessions or studies over time. Currently each session is independent and participants are anonymized with no cross-session identity.

This would require:
- A consistent participant ID scheme (beyond anonymization)
- A `/participant-arc` skill that loads all sessions for a participant and describes how their behavior or attitudes changed

**How it differs from today:** All synthesis is cross-participant-within-study. Longitudinal is cross-session-for-same-participant, which Attic currently has no support for.

**Practical consideration:** Requires rethinking PII scrubbing - you need a way to link "Participant A in Study 1" to "Participant A in Study 3" without storing identifying info. Probably a researcher-assigned stable pseudonym stored in session frontmatter.

**Source:** Cross-time comparison pattern, `meeting-insights-analyzer` skill, ComposioHQ/awesome-claude-skills repo

---

## open-notebook as Ingestion Backend

**What it is:** [open-notebook](https://github.com/lfnovo/open-notebook) is a self-hosted NotebookLM alternative with a REST API. It can ingest PDFs, audio files, video, and Office documents, then provide AI-powered note generation.

For Attic, this would mean being able to pass an audio recording directly to the pipeline instead of requiring a pre-transcribed text file.

**How it differs from today:** Attic only works with markdown and text files. Audio files must be transcribed externally first. An open-notebook integration would close that gap.

**Practical consideration:** Significant infrastructure dependency. Only worth it if Henrik regularly works with raw audio rather than pre-transcribed output from Granola/Zoom.

**Source:** `open-notebook` skill, K-Dense-AI/claude-scientific-skills repo
