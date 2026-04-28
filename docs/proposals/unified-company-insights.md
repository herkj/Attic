# Proposal: Unified Company Insights Integration (Scoping)

**Status:** Scoping only, not designed.
**Owner:** Open - future platform/research collaboration.
**Last updated:** 2026-04-28

---

## TL;DR

Attic currently stores qualitative research insights in markdown. Company data systems store adjacent signals (support tickets, analytics, NPS, product telemetry). Long-term value likely comes from a unified insight layer that connects these worlds. This document scopes the opportunity and decision points.

---

## Why this matters

Research findings are strongest when triangulated with operational data. Today, Attic insights are mostly isolated from:

- customer support trends,
- product analytics,
- transactional behavior,
- satisfaction metrics.

Unifying these sources can improve prioritization, confidence, and cross-team trust in insights.

---

## What success looks like

1. A person can view a qualitative insight and quickly see related quantitative signals.
2. Product teams can query one place for both research and operational evidence.
3. Data sharing is privacy-safe and does not weaken PII controls.
4. Integration adds value without forcing a rewrite of Attic's markdown workflow.

---

## Scope for discovery phase

This proposal does **not** define a final architecture. It defines what must be learned first.

### Discovery track A: Source mapping

Identify candidate company data systems and owners:
- support systems (for example Zendesk),
- analytics systems (for example Mixpanel, Amplitude, internal BI),
- satisfaction systems (NPS/CSAT),
- warehouse tables with customer behavior signals.

For each source capture:
- owner team,
- access model,
- update cadence,
- data quality confidence.

### Discovery track B: Joining model

Define realistic join points between Attic insights and company data:
- tag/theme joins (lowest complexity),
- product-area joins,
- market/segment joins,
- time-window joins.

Avoid person-level joins in early phases.

### Discovery track C: Output expectations

Clarify what users need in a unified view:
- insight cards with related metrics?
- weekly trend digest?
- searchable index by theme and confidence?

---

## Candidate integration patterns

### Pattern 1: Attach metadata into Attic output

Enrich Attic insights with links/metrics from external systems during synthesis/export.

Pros:
- low infrastructure change,
- keeps markdown workflow.

Cons:
- dependency on data fetch reliability,
- risk of stale values in static files.

### Pattern 2: Publish Attic insights to a shared insights service

Export normalized insights to an internal API/database consumed by other tools.

Pros:
- reusable by many teams,
- better long-term platform potential.

Cons:
- significantly more engineering overhead,
- requires ownership and governance.

### Pattern 3: Federated read layer only

Keep Attic and company systems separate, but build a query layer that can present both.

Pros:
- minimal write-path changes.

Cons:
- discovery complexity and access control can be high.

---

## Recommended first move

Run a scoped 2-3 week discovery spike:

1. Source inventory and ownership map.
2. One pilot integration with a single data source.
3. Demo of one combined output artifact.
4. Decision memo: continue, pause, or narrow.

Do not commit to a platform build before this spike.

---

## Open questions

1. Which team would own a unified insights product long term?
2. What privacy and legal constraints apply to cross-source joining?
3. Which source provides the strongest first pilot signal?
4. Should Attic remain markdown-first if integration succeeds?

---

## Out of scope

- Full data warehouse redesign.
- Real-time customer-level personalization.
- Replacing existing BI stack.
