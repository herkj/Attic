# Proposal: Cross-Study Synthesis

**Status:** Proposed, not implemented.
**Owner:** Open - first picker-upper claims it.
**Last updated:** 2026-04-28

---

## TL;DR

Attic should synthesize patterns across studies, not only inside each study. This proposal introduces a new cross-study synthesis flow that identifies recurring themes, contradictions, and confidence across all active studies.

---

## Why this matters

Current architecture can produce:
- observations per session,
- insights per session,
- insights per study.

What is missing is a global layer that answers:
- Which signals are repeating across teams?
- Which friction points are stable over time?
- Which findings are contradictory by segment or market?

Without this layer, Attic cannot deliver its intended cross-team value.

---

## What success looks like

1. A new command can synthesize across multiple studies.
2. Output clearly separates repeated patterns and isolated signals.
3. Contradictions are surfaced explicitly, not buried.
4. Confidence is transparent and tied to evidence breadth.

---

## Proposed command

`/synthesize-cross`

### Inputs
- Optional study filter (all studies by default)
- Optional date range
- Optional tag or theme focus

### Output
- `Research/Repository/cross-study-insights.md`
- Optional per-theme files in `Research/Repository/themes/`

---

## Suggested output format

```markdown
# Cross-Study Insights

## Pattern: Onboarding friction repeats
**Prevalence:** 6 of 8 studies
**Confidence:** high
**Tags:** Onboarding, Frustration
**Based on:** [[Study A]], [[Study B]], [[Study C]]

Description...

## Contradiction: Preference differs by segment
**Prevalence:** 3 studies vs 2 studies
**Confidence:** medium
**Tags:** Preference
**Based on:** [[Study D]], [[Study E]], [[Study F]]

Description...
```

---

## Confidence model (draft)

- **High**
  - signal appears in 3+ studies
  - at least one supporting study insight is behavioral
  - studies are reasonably recent
- **Medium**
  - appears in 2 studies, or
  - appears in many studies but all attitudinal
- **Low**
  - appears once, or
  - evidence is old or weak

---

## Implementation plan

1. Create `.claude/commands/synthesize-cross.md`.
2. Add prompt `prompts/synthesize-cross-study.md`.
3. Gather all study insights from `Research/Studies/*/study.md`.
4. Convert to normalized input list (study, insight text, tags, evidence strength, date).
5. Run synthesis prompt to cluster and summarize.
6. Write outputs into `Research/Repository/`.
7. Add usage and caveats to `docs/faq.md`.

---

## Open questions

1. Should inactive/archived studies be excluded by default?
2. Should recency be weighted automatically in confidence?
3. Should cross-study synthesis include report-derived observations?
4. Is one global file enough, or should per-theme files be default?

---

## Out of scope

- Auto-prioritization by user impact score.
- Integration with external analytics in v1.
- Real-time updating as each new session lands.
