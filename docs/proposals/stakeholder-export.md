# Proposal: Stakeholder-Friendly Export

**Status:** Proposed, not implemented.
**Owner:** Open - first picker-upper claims it.
**Last updated:** 2026-04-28

---

## TL;DR

Attic needs a clean export path for PMs, designers, and leadership who do not want to read raw study/session markdown files. This proposal adds a command that transforms Attic insights into concise, readable stakeholder reports.

---

## Why this matters

Research value depends on communication. Current outputs are rich but researcher-oriented. Non-research stakeholders need:

- shorter summaries,
- explicit business implications,
- confidence and prevalence in plain language,
- direct links to evidence when needed.

Without export, findings are harder to activate.

---

## What success looks like

1. One command generates a stakeholder-ready summary in under 2 minutes.
2. Output is understandable without Attic internals.
3. Evidence traceability remains available but optional.
4. Report can be copied into docs, slides, or chat updates with minimal editing.

---

## Proposed command

`/export-stakeholder`

### Inputs
- Study path (required)
- Audience profile (PM, design, leadership)
- Output length (short, standard, detailed)
- Optional focus tags

### Outputs
- `Research/Reports/Stakeholder/{study-name}-{date}.md`
- optional Slack-ready summary snippet in same file

---

## Suggested report structure

```markdown
# Study Brief: Merchant Onboarding

## Executive Summary
- 3-5 bullets in plain language

## Key Findings
### Finding 1
What happened, prevalence, confidence, why it matters.

## Evidence Snapshot
- Selected quotes or references (optional depth)

## Risks and Unknowns
- What is still uncertain

## Recommended Next Questions
- Follow-up research questions (not product prescriptions)
```

---

## Writing principles

- Plain language first.
- No internal skill jargon.
- Keep confidence explicit.
- Keep recommendations as research next steps, not product mandates.
- Preserve traceability links for anyone who wants to drill down.

---

## Implementation plan

1. Create `.claude/commands/export-stakeholder.md`.
2. Create `prompts/export-stakeholder.md` with audience-specific tone controls.
3. Pull inputs from study-level insights and supporting session insight references.
4. Render markdown report to output path.
5. Add FAQ entry with common usage patterns.

---

## Open questions

1. Should export include direct quotes by default or behind a toggle?
2. Is one template enough, or should each audience have its own template?
3. Should a PDF variant be generated in v1 or markdown only?
4. Should export include a one-paragraph business impact estimate?

---

## Out of scope

- Automated PowerPoint generation.
- Bi-directional sync with presentation tools.
- Executive dashboard UI.
