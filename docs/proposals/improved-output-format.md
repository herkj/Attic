# Proposal: Improved Output Format for Discovery and Reuse

**Status:** Proposed, not implemented.
**Owner:** Open - first picker-upper claims it.
**Last updated:** 2026-04-28

---

## TL;DR

Attic currently writes core outputs into long markdown files per session and study. This is practical for generation but weak for search and reuse across teams. This proposal compares four output patterns and recommends an incremental path that improves discoverability without forcing a full app rebuild.

---

## Why this matters

Attic's strategic value is cross-study learning, not just per-session summaries. Long files are manageable early, but as volume grows:

- insights are hard to find by theme,
- the same signals are rediscovered repeatedly,
- stakeholders outside research struggle to consume outputs.

Without better output structure, adoption will likely stall.

---

## What success looks like

A successful output format improvement should make it easy to:

1. Find all insights by tag/theme across studies.
2. Trace each insight back to evidence quickly.
3. Create stakeholder summaries without manually reading many files.
4. Keep authoring simple for researchers and agents.
5. Avoid forcing a heavy frontend before value is proven.

---

## Option analysis

### Option 1: Atomic insight files (recommended first step)

**Concept**
- Split high-value outputs into one-file-per-insight records.
- Keep session/study files as orchestration summaries and indexes.

**Example path**
`Research/Repository/insights/{slug}.md`

**Pros**
- Excellent discoverability in Obsidian search and graph.
- Easy backlinks between studies and shared themes.
- Simple to implement with current markdown workflow.

**Cons**
- Increases file count.
- Requires clear naming/ID conventions.

### Option 2: Obsidian-native dashboards (recommended second step)

**Concept**
- Build dynamic views with Obsidian Bases or Dataview over structured markdown properties.

**Pros**
- No new infrastructure.
- Fast way to provide "all checkout friction insights" style views.

**Cons**
- Depends on Obsidian plugin usage and team habits.
- Less portable to non-Obsidian consumers.

### Option 3: Standalone web viewer

**Concept**
- Build a read-only web layer that indexes markdown outputs.
- Keep authoring in markdown; render/search in a web UI.

**Pros**
- Best stakeholder experience.
- Easier access for non-research functions.

**Cons**
- Highest implementation and maintenance cost.
- Risks re-introducing complexity that Attic intentionally removed.

### Option 4: Structured export (JSON/SQLite)

**Concept**
- Export normalized artifacts from markdown into machine-friendly formats.

**Pros**
- Integrates with analytics and internal data tools.
- Strong base for long-term automation.

**Cons**
- Requires schema governance.
- Adds synchronization concerns with markdown source-of-truth.

---

## Recommended path

### Phase 1 (now): Option 1

Create atomic insight files while preserving existing study/session docs.

### Phase 2 (soon): Option 2

Add Obsidian dashboards on top of atomic files for immediate discoverability gains.

### Phase 3 (later): Option 3 or 4 based on demand

Only build a web viewer or structured export when there is a clear consumer and owner.

---

## Design sketch for Phase 1

### Proposed file model

```
Research/
  Repository/
    insights/
      checkout-friction-self-service-onboarding.md
      settlement-manual-reconciliation-workaround.md
```

### Suggested frontmatter

```yaml
type: research/insight
status: open
created: 2026-04-28
updated: 2026-04-28
insight-id: INS-0001
title: Checkout friction in self-service onboarding
insight-type: friction
evidence-strength: behavioral
tags: [Checkout, Onboarding, Frustration]
studies:
  - Merchant Onboarding
sessions:
  - P01-Astrid-34
source-links:
  - "[[Merchant Onboarding/sessions/P01-Astrid-34]]"
```

### Migration rule

- Keep `study.md` as the canonical study summary.
- Add `Based on` links from study insights to atomic insight files.
- Do not remove current study/session outputs in the first migration.

---

## Implementation plan

1. Define naming and ID conventions for atomic files.
2. Add a new skill or extend `/study-synthesize` to emit atomic insight files.
3. Backfill one pilot study manually to validate readability.
4. Add a search-oriented index file in `Research/Repository/` linking key themes.
5. If this works, standardize and apply to future studies.

---

## Open questions

1. Should `study.md` remain the source of truth, or should atomic files become primary?
2. Should IDs be deterministic (slug hash) or sequential (`INS-0001`)?
3. How should conflicting insights across studies be represented?
4. What is the minimum metadata needed for strong search?
5. Who owns taxonomy hygiene as output granularity increases?

---

## Out of scope for this proposal

- Full frontend authoring app.
- Real-time multi-user editing.
- Permissions and access control model.
- Replacing markdown with a database.
