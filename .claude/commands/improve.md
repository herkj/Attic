# /improve - Improve extraction quality from review history

Analyzes accumulated review decisions to propose improvements to extraction prompts, taxonomy, and preferences. Tracks improvement generations to measure whether changes are working.

## Inputs

$ARGUMENTS - optional: "history" to jump straight to history view, "run" to jump straight to improvement cycle.

## Steps

### 1. Entry menu

If $ARGUMENTS is "history", go to step 2. If "run", go to step 3.

Otherwise, use `AskUserQuestion`: "What would you like to do?"
- **Run improvement cycle** - analyze learning data and propose changes
- **See history** - show past improvement runs and current learning stats

---

## Step 2: See history

### 2a. Load data

Find the Research root (see `docs/skill-patterns.md` - File Discovery). Read:
- `Research/config/learning/journal.yaml` - all review entries
- `Research/config/learning/improve-history.yaml` - past improvement runs
- `Research/config/learning/examples.yaml` - current example bank
- `Research/config/learning/preferences.yaml` - current preferences

If `journal.yaml` does not exist, show "No review data yet. Run `/review-observations` on at least one session first." and stop.

### 2b. Compute metrics

For each journal entry, compute four rates from its `stats`:

- **Miss rate** = `added_by_user / (approved + added_by_user)` - what the human wanted that the AI didn't surface
- **Noise rate** = `rejected / extracted` - what the AI surfaced that the human didn't want
- **Edit rate** = `edited_by_user / approved` - how often first-pass observations needed correction
- **Tag accuracy** = `1 - (count of tag_edits / approved)` - whether the AI assigns the right tags

### 2c. Show generation comparison

Group journal entries by their `generation` field. For each generation, compute the average of each metric across all sessions in that generation.

Display as a table:

```
Generation 0 (5 sessions, no learning data):
  miss rate: 12%  |  noise rate: 28%  |  edit rate: 15%  |  tag accuracy: 82%

Generation 1 (4 sessions, after first /improve):
  miss rate: 8%   |  noise rate: 24%  |  edit rate: 11%  |  tag accuracy: 88%
```

Highlight miss rate as the primary metric. If miss rate is trending down across generations, note this as a positive signal. If it's flat or rising, flag it.

### 2d. Show additional stats

- **Top rejection reasons** (all-time frequency, and last generation only)
- **Example bank** - number of gold examples and anti-examples currently stored
- **Preference notes** - list all current preference notes
- **Improvement runs** - date of each past `/improve` run and summary of what changed

Stop here. Do not proceed to the improvement cycle unless the user asks.

---

## Step 3: Run improvement cycle

### 3a. Load all data

Read:
- `Research/config/learning/journal.yaml`
- `Research/config/learning/examples.yaml` (may not exist)
- `Research/config/learning/preferences.yaml` (may not exist)
- `Research/config/learning/improve-history.yaml` (may not exist)
- All taxonomy files (follow `docs/skill-patterns.md` - Taxonomy Loading, using the most commonly used taxonomy across journal entries)

If `journal.yaml` does not exist or has fewer than 3 entries, show "Not enough review data yet. Run `/review-observations` on at least 3 sessions before running /improve." and stop.

### 3b. Analyze patterns

Compute the metrics from step 2b across all journal entries. Then analyze:

1. **Rejection patterns:** Which observation types get rejected most? Which rejection reasons are most common? Are there specific tag combinations that correlate with rejection?

2. **Miss patterns:** What types of observations do users add most often? Are there common tags on user-added observations that suggest taxonomy gaps?

3. **Edit patterns:** Which tags get edited most? Are there recurring tag renames (suggesting missing aliases)? Do users frequently change observation types (suggesting misclassification)?

4. **Taxonomy usage:** Which taxonomy tags appear across approved observations? Which tags are never used? Which tags do users add manually during review that don't exist in the taxonomy?

### 3c. Propose changes

Present proposals in groups. For each proposal, use `AskUserQuestion` with options: **Accept**, **Reject**, **Edit** (modify and accept).

**Group 1: Example bank updates**

Propose new examples by reading the actual session files referenced in recent journal entries:

- **Gold examples (up to 3):** Find approved observations from recent sessions that best represent what the team values. Prioritize: behavioral evidence, workaround type, problem type with business impact, observations with high tag accuracy (no tag edits). Show each proposed example with its text, type, tags, and a brief note on why it's a good example.

- **Anti-examples (up to 3):** Find rejected observations from recent sessions that have clear rejection reasons. Show each with its text, type, and the rejection reason. Skip rejections with reason "no reason."

- **Stale example removal:** If the example bank has examples from more than 3 generations ago, propose removing them to keep examples fresh and relevant.

**Group 2: Preference updates**

- Compute approval rates by observation type across all sessions. Present as a table.
- If a pattern emerges (e.g., quote_summary rejected >50% of the time), propose a preference note explaining when that type is valuable.
- If users consistently add a certain type that the AI underproduces, propose a note about it.

**Group 3: Taxonomy suggestions**

- **Unused tags:** Tags that appear zero times across all approved observations. Suggest adding aliases or improving descriptions (written to `Research/config/taxonomy/learned.yaml`).
- **Missing tags:** Tags that users manually added during review (from `additions` and `tag_edits` in the journal) that don't exist in the current taxonomy. Suggest adding them.
- **Alias suggestions:** Tag renames that appear in `tag_edits` more than once. Suggest adding the "from" value as an alias for the "to" value.

Write any accepted taxonomy changes to `Research/config/taxonomy/learned.yaml`, not to the Attic-managed taxonomy files. Create the file if it doesn't exist.

**Group 4: Extraction rule suggestions**

If the analysis reveals a pattern of rejections that could be prevented by a new rule:
- Propose adding it as a preference note (which gets loaded into prompts via the learning section).
- Frame it as: "When [condition], [what to do differently]."
- Example: "When a participant makes a generic positive statement without describing specific behavior, do not extract it as a standalone observation."

### 3d. Apply changes

Write all accepted changes to the appropriate files:
- Gold and anti-examples to `Research/config/learning/examples.yaml`
- Preference notes to `Research/config/learning/preferences.yaml`
- Taxonomy additions to `Research/config/taxonomy/learned.yaml`

Create the `Research/config/learning/` directory and files if they don't exist.

**File format for `examples.yaml`:**
```yaml
gold_examples:
  - text: "Merchant relies on screenshot-based reconciliation because the settlement report lacks transaction-level detail"
    type: workaround
    evidence_type: behavioral
    tags: [Settlement, Workaround]
    source_session: "Merchant Onboarding / Session 03"
    generation_added: 1
    why_good: "Clear workaround with behavioral evidence and business impact"

anti_examples:
  - text: "User prefers simple flow"
    type: observation
    tags: [Checkout]
    source_session: "Merchant Onboarding / Session 03"
    generation_added: 1
    reason_rejected: "Too vague - no specific behavior or evidence"
```

**File format for `preferences.yaml`:**
```yaml
last_updated: 2026-03-12
sessions_analyzed: 12
type_approval_rates:
  problem: 0.85
  workaround: 0.92
  observation: 0.68
  preference: 0.71
  quote_summary: 0.55
  hypothesis: 0.40
notes:
  - "Values workaround observations highly - almost always approved"
  - "Tends to reject quote_summary unless the quote is emotionally charged or captures a key contradiction"
  - "When a participant makes a generic positive statement without describing specific behavior, do not extract it as a standalone observation"
```

### 3e. Log improvement run

Compute the current generation's metrics (same as step 2b-2c, but only for sessions in the current generation). Increment the generation counter and append to `Research/config/learning/improve-history.yaml`:

```yaml
- generation: 1
  date: 2026-03-12
  sessions_in_generation: 5
  metrics_at_run:
    miss_rate: 0.12
    noise_rate: 0.28
    edit_rate: 0.15
    tag_accuracy: 0.82
  changes:
    gold_examples_added: 2
    anti_examples_added: 3
    examples_removed: 0
    preferences_updated: true
    preference_notes_added: 2
    taxonomy_tags_added: 1
    taxonomy_aliases_added: 2
```

### 3f. Summary

Show what was changed and the current generation number. Remind that the next `/analyse` run will use the updated learning data (generation N+1).

If the generation comparison shows improvement (miss rate or noise rate trending down), note this. If metrics are flat or worse, note that too - it may mean the proposals need a different approach, or that the sample size is still too small to draw conclusions.
