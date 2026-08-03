# Proposal: Eval Harness for Extraction Quality

**Status:** Proposed, not implemented.
**Owner:** Open - first picker-upper claims it.
**Last updated:** 2026-04-27

---

## TL;DR

Add an offline regression test for the observation-extraction prompts. A small set of curated fixture transcripts + "gold" expected outputs lets us check whether prompt or learning-data changes improve or regress extraction quality - in seconds, instead of waiting for 5+ real reviewed sessions to give a signal.

---

## Why this matters

Today, the only feedback signal for extraction quality is the **learning journal** at `Research/config/learning/journal.yaml`, populated by `/review-observations` and analysed by `/improve`.

That loop has two structural weaknesses:

1. **Slow.** A meaningful trend in miss-rate / noise-rate / edit-rate / tag-accuracy needs at least 3-5 reviewed sessions per generation. Until then, every change is essentially a guess.
2. **Confounded.** The metrics conflate three different sources of variance:
   - Did the prompt get better?
   - Did the learning data (examples + preferences) get better?
   - Or was this batch of interviews simply easier or harder than the last?

   Real interviews vary in length, language, complexity, and topic. You cannot tell a prompt regression apart from a hard interview just by looking at one generation's metrics.

3. **Unreviewable.** Every change to `prompts/extract-observations-*.md` and to `Research/config/learning/examples.yaml` is currently a leap of faith. Reviewers (humans or other AI agents) have nothing to point at to decide whether a proposed change is good.

An eval harness fixes all three: same fixtures every run, controlled signal, fast iteration, reviewable diffs.

---

## What success looks like

An eval is "successful" when:

1. **A picker-upper can run `/eval` in under 60 seconds** and get a one-screen report.
2. The report tells them, for each fixture: how many "must-find" observations were surfaced, how many "must-not-extract" items leaked through, whether the observation count fell in the expected range, and what changed since the last run.
3. **A `/improve` cycle can run the eval before and after** accepting changes, so any regression is caught before it reaches real research data.
4. The harness is **maintainable by one person** - adding a new fixture takes <30 minutes, gold files are human-readable, and the comparison logic is easy to reason about.

---

## Design

### File layout

```
tests/
  fixtures/
    transcript-onboarding-study-01.md          # an anonymised real transcript
    transcript-onboarding-study-01.gold.yaml   # expected output spec
    notes-search-redesign-02.md                 # observer notes fixture
    notes-search-redesign-02.gold.yaml
    report-industry-survey.md              # report fixture
    report-industry-survey.gold.yaml
  runs/
    2026-04-27-gen0.yaml                          # logged eval results, one per run
    2026-05-04-gen1.yaml
```

The `tests/fixtures/` folder is committed. `tests/runs/` should be either committed (run history is interesting) or gitignored (noise). Decision deferred - probably commit it; a year of runs is small.

### Gold file format

Loose specification rather than exact-match. LLM output varies in wording even at temperature 0; we want to check **which insights got surfaced**, not whether the wording is byte-identical.

```yaml
# tests/fixtures/transcript-onboarding-study-01.gold.yaml
fixture: transcript-onboarding-study-01
source_type: transcript
taxonomy: core

# Observations the extraction MUST surface. Match on title keywords + type.
# Required tags is a subset - the output may have additional tags as long as
# every required tag appears.
must_find:
  - title_keywords: ["password", "reset"]
    type: problem
    required_tags: ["Login"]
    why: "Most-cited friction point, mentioned twice in transcript"

  - title_keywords: ["spreadsheet", "track"]
    type: workaround
    required_tags: ["Workaround"]
    why: "Concrete behavioural workaround with clear user impact"

  - title_keywords: ["export"]
    type: observation
    required_tags: []
    why: "External service named - tag it if the taxonomy defines External Entities"

# Observations the extraction MUST NOT surface (over-fragmentation, vague,
# leading-question artefacts, etc.). Match on title keywords.
must_not_extract:
  - title_keywords: ["nice interface"]
    reason: "Generic positive statement without behavioural evidence"

  - title_keywords: ["good experience"]
    reason: "Quote summary without distinct insight - covered by other obs"

# Acceptable observation count range. Failing this means under- or over-extraction.
expected_observation_count_range: [10, 20]

# Optional: tag-coverage assertions. Useful for catching taxonomy drift.
expected_tags_present:
  - "Settlement"
  - "BankID"
  - "Login"
expected_tags_absent:
  - "Onboarding"  # this transcript doesn't actually cover onboarding despite the topic
```

### Matching logic

For each `must_find` entry, an observation in the output matches if:

1. **All title keywords** appear in the observation title (case-insensitive substring match).
2. The observation `type` equals the entry's `type`.
3. **Every** required tag appears in the observation's tags (subset check, not equality).

For each `must_not_extract` entry, the observation matches if all title keywords appear in the title (regardless of type or tags). Any match is a failure.

### Metrics reported

For each fixture:

| Metric | Definition |
|--------|------------|
| `match_rate` | `must_find_matched / must_find_total` (target: ≥ 0.8) |
| `false_positive_rate` | `must_not_extract_matched / must_not_extract_total` (target: 0) |
| `observation_count_in_range` | boolean |
| `tag_coverage` | `expected_tags_present_found / expected_tags_present_total` |
| `tag_violations` | count of `expected_tags_absent` that appeared anyway |

Aggregate across fixtures: simple unweighted mean.

### Pass/fail criteria for v1

A run "passes" if **all** of the following hold across all fixtures:

- `match_rate ≥ 0.8` per fixture
- `false_positive_rate == 0` per fixture
- `observation_count_in_range == true` per fixture

Otherwise "fail" - useful for `/improve` to gate proposed changes.

### Run output format

```yaml
# tests/runs/2026-04-27-gen0.yaml
date: 2026-04-27
generation: 0
prompt_version: "8104be5"  # git SHA at time of run
overall: pass
fixtures:
  - fixture: transcript-onboarding-study-01
    extracted_count: 14
    match_rate: 0.86          # 6 of 7 must_find
    false_positive_rate: 0.0
    observation_count_in_range: true
    tag_coverage: 1.0
    tag_violations: 0
    matched_must_find:
      - "password reset / problem"
      - "spreadsheet tracking / workaround"
      ...
    missed_must_find:
      - "export retry / observation - nothing matched"
    leaked_must_not_extract: []

diff_vs_previous:
  - fixture: transcript-onboarding-study-01
    match_rate_delta: +0.14   # was 0.72 in gen-1
    false_positive_rate_delta: 0
```

---

## Implementation plan

Estimated effort: 1-2 days for v1, including writing the first 2 fixtures.

### Step 1: Pick and anonymise 2-3 fixture transcripts

The hardest part is finding fixtures. Pick from sessions you've already reviewed thoroughly (gold answers are basically the approved observations from `/review-observations`). Aim for variety:

- One interview transcript (ideally in a non-English language, to exercise language preservation)
- One observer-notes set
- One report or article (optional in v1)

Anonymise more aggressively than `/scrub-pii` does - no city names, no employer hints, no industry-specific jargon that uniquely identifies a participant. These fixtures will live in git forever.

### Step 2: Write gold YAML for each fixture

For each fixture:
- Look at the real approved observations from the original `/review-observations` run.
- Pick 5-10 of them as `must_find` - the ones any reasonable extraction must surface.
- Pick 3-5 things that were rejected or never extracted as `must_not_extract`.
- Set `expected_observation_count_range` based on the original extraction (give ±20% slack).

### Step 3: Write the `/eval` skill

New file: `.claude/commands/eval.md`. Sketch:

```
# /eval - Run extraction against fixtures and compare to gold

## Inputs
$ARGUMENTS - optional fixture name to run a single fixture; otherwise run all.

## Steps
1. Glob `tests/fixtures/*.gold.yaml` to find all fixtures (or just the named one).
2. For each fixture:
   a. Read the fixture content and gold YAML.
   b. Run extraction by loading the matching prompt from `prompts/`,
      filling placeholders, and sending to the model.
   c. Parse the JSON output into a list of observations.
   d. Compute metrics: match_rate, false_positive_rate,
      observation_count_in_range, tag_coverage, tag_violations.
3. Read the most recent `tests/runs/*.yaml` (if any) to compute deltas.
4. Write the new run as `tests/runs/{date}-gen{N}.yaml` where N is the
   current learning generation (or 0).
5. Print a one-screen summary table to the user.
```

### Step 4: Wire `/eval` into `/improve` (optional, do after v1)

In `/improve` step 3a, before proposing changes, suggest running `/eval` first. After accepting changes, suggest running `/eval` again and surface any metric regression. Make the regression check a soft warning in v1, a hard gate later.

### Step 5: Document in CLAUDE.md

Add `/eval` to the Skills table in `CLAUDE.md`. Add a short note in `docs/skill-patterns.md` (probably under Learning System) about how the eval harness complements the journal-based feedback loop.

---

## Open questions

These should be decided before or during implementation:

1. **Where do fixtures live for a multi-tenant Attic?** The plan above puts them in `tests/fixtures/` inside the Attic repo. That's fine for a single-team tool. If Attic ever supports multiple teams with different domains, fixtures may need to live in `Research/config/eval-fixtures/` (vault-side) instead. **Recommendation for v1:** repo-side, since Attic is currently used by one team.

2. **How strict is the `must_find` keyword match?** Substring match is forgiving but may produce false positives (an unrelated observation that happens to contain the keywords). Alternatives: exact title match (too strict), embedding similarity (more code, more dependencies), LLM-as-judge (slow, expensive). **Recommendation for v1:** plain substring + type + required_tags. Upgrade later if needed.

3. **Should the eval be deterministic?** LLM output varies even at temperature 0 due to sampling and model versioning. Two consecutive `/eval` runs may differ. Options: (a) accept the noise and run multiple times for trend, (b) cache extraction outputs by fixture+prompt-hash and only re-run when the prompt changes. **Recommendation for v1:** accept the noise, document it, run 3 times when a metric is borderline.

4. **Run history: commit or gitignore?** Committing means the team sees the long-term quality trend. Gitignoring keeps the repo clean. **Recommendation:** commit, but rotate (keep last 30 runs).

5. **Where does "must_find" reasoning live?** Right now in YAML `why:` fields. As fixtures grow, this could move to a markdown sidecar (`fixture.gold.md`) for richer prose. **Recommendation for v1:** keep it in YAML.

---

## Out of scope for v1

To avoid scope creep:

- ✗ Don't try to evaluate `/synthesize` or `/study-synthesize` outputs. Synthesis is harder to gold-spec because it groups observations, and grouping is more subjective. Add later.
- ✗ Don't build a UI. CLI / markdown report is fine.
- ✗ Don't try to A/B test prompt variants automatically. That's a fun future feature but not v1.
- ✗ Don't replace the learning journal. The journal works on real data; the eval works on fixtures. They complement each other - both should run.

---

## Success indicator

You'll know the eval harness is working when someone proposes a change to `prompts/extract-observations-transcript.md`, runs `/eval`, sees `match_rate` drop from 0.86 to 0.71 on one fixture, and chooses **not** to merge the change without first understanding why. That moment is the entire point.
