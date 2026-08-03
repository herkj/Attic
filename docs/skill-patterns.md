# Skill Patterns

Shared patterns used by every skill in `.claude/commands/`. Skill files reference sections here by name (e.g. *"see skill-patterns.md - File Discovery"*) instead of repeating the logic.

If you change a pattern here, every skill that references it picks up the change on the next run.

---

## Vault Structure

Research data lives in the user's Obsidian vault (not this repo):

```
Research/                             # vault root - default name; may be user-chosen (see Vault Root Resolution)
  inbox/                              # Drop raw notes/transcripts here
    processed/                        # Originals moved here after /analyse
  Studies/
    {Study Name}/
      study.md                        # Study metadata, taxonomy ref, research questions, insights
      sessions/
        P01-Maya-34.md              # Session file (P{nn}-{Pseudonym}-{age})
        P01-Maya-34-sources/
          transcript.md               # Raw transcript
          transcript-scrubbed.md      # PII-scrubbed
          notes.md                    # Observer / interviewer notes
          notes-scrubbed.md           # PII-scrubbed notes
          debrief.md                  # Interview technique feedback (optional)
        P02-Sam-28.md
        P02-Sam-28-sources/
          ...
  Reports/
    {Report Name}/
      report.md                       # Report metadata, observations, insights
      report-sources/
        original.pdf                  # Original report file (URL inputs skip this)
        content.md                    # Markdown conversion of the report
  Repository/                         # Cross-study artefacts (manual)
```

---

## Naming Rules

- **Study folders:** Use the study name as-is, spaces are fine (Obsidian handles them).
- **Session files:** `P{nn}-{Pseudonym}-{age}.md` (e.g. `P01-Maya-34.md`). If age is unknown: `P{nn}-{Pseudonym}.md`. Created with temp name `P{nn}-pending.md` and renamed after `/scrub-pii` assigns a pseudonym (see `/analyse` step A3).
- **Source folders:** `P{nn}-{Pseudonym}-{age}-sources/` matching the session file name.
- **Scrubbed files:** Append `-scrubbed` before the extension (e.g. `transcript-scrubbed.md`).
- **Fixed transcript files:** Append `-fixed` before the extension (e.g. `transcript-fixed.md`, written by `/fix-transcript`).
- **Participant IDs:** `P01`, `P02`, etc. Sequential per study.
- **Report folders:** Use the report name as-is, spaces are fine.
- **Report sources:** `report-sources/` contains `original.`* and `content.md`.

---

## Vault Root Resolution

The research vault is created by `/setup-attic`. Its default name is `Research`, but the user can choose any name/location, so **never hardcode the name `Research`**. Resolve the vault root once, then derive everything from it:

1. Glob for `**/Studies/*/study.md` (or `**/Studies/` if no studies exist yet). The directory that contains the `Studies/` folder is the **vault root** (`{vaultRoot}`).
2. If multiple candidates match, prefer the one that also contains a `config/` and/or `inbox/` folder; if still ambiguous, ask the user to pick.
3. From `{vaultRoot}`, derive: `{vaultRoot}/Studies/`, `{vaultRoot}/Reports/`, `{vaultRoot}/inbox/`, `{vaultRoot}/config/taxonomy/`, `{vaultRoot}/config/learning/`.

When Attic lives inside `{vault}/Attic/`, the vault data is a sibling of the Attic repo. The `**/Studies/` glob works regardless of where the vault sits or what it is named.

## File Discovery

When a skill receives a path or reference:

1. If it's a full file path, use it directly.
2. If it looks like `Study Name/P01-Maya-34`, resolve to `{vaultRoot}/Studies/{Study Name}/sessions/P01-Maya-34.md` (see Vault Root Resolution).
3. If ambiguous, use Glob to find matching files and ask the user to pick.

---

## Taxonomy Field

The `taxonomy` field in `study.md` frontmatter tells skills which domain taxonomy to load:

- `core` -> loads only `core.yaml` (no domain-specific tags)
- `<domain>` -> loads `core.yaml` + the domain file `<domain>.yaml` (found in the Attic repo `taxonomy/` or the vault's `{vaultRoot}/config/taxonomy/`)

On top of those, any `.yaml` files in the vault's `{vaultRoot}/config/taxonomy/` are merged as a third layer (team-shared, not in this repo).

---

## Taxonomy Loading

Skills that need taxonomy follow this pattern:

1. Read the study.md frontmatter `taxonomy` field (e.g. "core" or a domain name).
2. Find the Attic project root via Glob for `**/taxonomy/core.yaml`.
3. Always load `taxonomy/core.yaml`.
4. If the field value is not "core", also load the matching domain file - check `taxonomy/{value}.yaml` in the Attic repo, then `{vaultRoot}/config/taxonomy/{value}.yaml` in the vault.
5. Resolve the vault root (see Vault Root Resolution). Check for any `.yaml` files in `{vaultRoot}/config/taxonomy/`. If any exist, merge them as a third layer on top of the previous two. Multiple files are all merged - this allows teams to maintain separate taxonomy files per domain or topic.
6. Merge and format as a flat list per category:
  ```
   ## Product Area
   - Dashboard (aliases: dashboard, home screen)
   - Settings (aliases: settings, preferences)
   ## Emotion
   - Frustration (aliases: frustrated, annoyed)
  ```

---

## Observation Format (in session files)

```markdown
### [?] Short observation title
**Type:** problem | observation | preference | workaround | quote_summary | hypothesis
**Evidence type:** behavioral | attitudinal
**Source:** transcript | observer notes | interviewer notes
**Tags:** Tag 1, Tag 2, Tag 3
**Evidence:**
> "Exact quote from source"

Interpreted statement about what was observed.
```

Status markers: `[?]` pending, `[x]` approved, `[-]` rejected

Evidence type:

- `behavioral` - quote or description shows what the participant actually did
- `attitudinal` - stated opinion, preference, or belief; or an observer's interpretation

---

## Session Insight Format

```markdown
### {Type}: Short insight title
**Type:** pattern | behavior | friction | preference | context
**Evidence strength:** behavioral | attitudinal
**Tags:** Tag 1, Tag 2, Tag 3
**Based on:** [[#Observation title 1]], [[#Observation title 2]]

Factual description of what was observed.

*Significance: One sentence on why this matters or what it tells us about the participant's experience.*
```

- Tags = union of all tags from supporting observations (deduplicated)
- Based on = wikilinks to observation headings (clickable in Obsidian)
- Evidence strength = "behavioral" if any supporting observation is behavioral; "attitudinal" if all are attitudinal only

---

## Study Insight Format

```markdown
### Title of cross-session finding
**Prevalence:** 4 of 6 sessions (67%)
**Based on:** Session 01, Session 03, Session 04, Session 06
**Confidence:** high | medium | low

Description of the pattern observed across sessions.
```

Confidence criteria:

- **High:** 3+ sessions, at least one session insight has behavioral evidence strength, recent data
- **Medium:** 2 sessions, OR 3+ sessions but all attitudinal evidence
- **Low:** single session, data older than 12 months, or primarily inferential

---

## Participant Naming Convention

Each participant gets a code + a memorable pseudonym: `P01 - {Pseudonym}`, `P02 - {Pseudonym}`.

The pseudonym is generated on demand (no name-list file to maintain):

- Assign P-numbers in order - P01 first, P02 second, etc. (sequential across the study).
- Generate a plausible, clearly-fictional given name. If the participant's country/region is known, choose a common given name from that culture so it reads naturally; if gender is known, match it; otherwise choose a neutral name.
- Within a study, pseudonyms must be unique - skip any already assigned.

This makes sessions easier to remember and discuss without using real names, and works for participants from any region without maintaining region-specific lists.

---

## PII Scrubbing Rules

**Redact:** Replace real participant names with their assigned pseudonym (e.g. "Maya" or "P01 - Maya"). Replace interviewer names with [Interviewer]. Replace other person names with [Person]. Replace emails -> [Email], phones -> [Phone], addresses -> [Address], ID numbers -> [ID], Slack @mentions with real names.

**Keep unchanged:** All names and aliases found in the taxonomy (product_areas and external_entities), city names as market references, job titles, nationalities, all formatting and structure.

---

## Prompt Substitution Rule

Prompt files in `prompts/` use `{{placeholder}}` (mustache-style double braces) for variables that the calling skill must fill in. Example placeholders: `{{taxonomy_section}}`, `{{learning_section}}`, `{{content}}`, `{{focus_areas}}`, `{{max_observations}}`.

Before sending an assembled prompt to the model, the skill MUST:

1. Substitute every `{{...}}` placeholder with the corresponding value.
2. Scan the assembled prompt for any remaining `{{...}}` patterns.
3. If any remain, **stop and surface the missing variable to the user**. Do not send the prompt to the model.

This is a safety check, not an optimisation. A leftover `{{learning_section}}` in a sent prompt produces silently wrong output that's hard to detect after the fact.

Note: this rule applies to **prompt files only** (the `prompts/` directory). Skill files in `.claude/commands/` use `{var}` (single braces) for internal workflow state - those are not sent to the model and don't need to be checked.

---

## Common Anti-Patterns (apply to all extraction and synthesis)

- Do not create an observation for every utterance - focus on distinct insights.
- Do not merge unrelated observations just to reduce count.
- Do not invent tags outside the taxonomy.
- Do not frame insights as recommendations - they are factual findings.
- Do not translate source content - preserve the original language of every quote and observation.
- Do not discard outliers - they may be the most interesting signals.
- When stated preference contradicts behavior, create separate observations for each.
- Always err toward surfacing more observations rather than fewer - the human review step exists to filter, but missed observations are unrecoverable.

---

## Learning System

Attic improves over time by tracking review decisions and feeding them back into extraction. Learning data lives in the vault under `{vaultRoot}/config/` (team-specific, not in the Attic git repo), following the same pattern as custom taxonomy. Resolve `{vaultRoot}` via Vault Root Resolution - do not assume the literal name `Research`.

**Principle:** Optimize for calibration - understanding what the human values - not for approval rate alone. The system should always err toward surfacing too much rather than too little.

### Learning Data Location

```
{vaultRoot}/               # default name "Research"; may be user-chosen
  config/
    learning/
      journal.yaml          # Accumulated review decisions (one entry per /review-observations run)
      examples.yaml         # Gold and anti-examples for extraction prompts
      preferences.yaml      # Team preference notes and type approval rates
      improve-history.yaml  # When /improve ran, what changed, metrics snapshot
    taxonomy/
      learned.yaml          # Taxonomy additions from /improve (tags, aliases)
```

### Generation Tracking

Each `/improve` run increments a generation counter. When `/analyse` runs extraction, it reads the current generation from `improve-history.yaml` (count of entries, or 0 if no file) and writes `learning_generation: {n}` to the session file frontmatter. When `/review-observations` logs to the journal, it copies this generation number into the entry. This links each review to the version of learning data that produced its observations, enabling before/after comparison.

### Four Metrics

Computed from journal entries by `/improve`:


| Metric       | Formula                                      | Measures                                           | Better = |
| ------------ | -------------------------------------------- | -------------------------------------------------- | -------- |
| Miss rate    | `added_by_user / (approved + added_by_user)` | Things the human wanted that the AI didn't surface | Lower    |
| Noise rate   | `rejected / extracted`                       | Things the AI surfaced that the human didn't want  | Lower    |
| Edit rate    | `edited_by_user / approved`                  | How often the first pass needed correction         | Lower    |
| Tag accuracy | `1 - (tag_edits / approved)`                 | Whether the AI assigns the right tags              | Higher   |


Miss rate is the most important. A system that never misses what the human would catch is doing its job, even if it produces some noise. Noise is cheap to filter. Misses are invisible and unrecoverable.

### How the Loop Works

1. `/analyse` loads learning data (examples + preferences) alongside taxonomy, records the current generation in session frontmatter, and runs extraction with the learning section injected into prompts.
2. `/review-observations` collects approve/reject/edit/add decisions with rejection reasons, logs everything to `journal.yaml` with the generation tag, and nudges the user to run `/improve` after 5+ sessions.
3. `/improve` reads the journal, computes metrics by generation, proposes example bank updates, preference notes, taxonomy additions, and extraction rules. Accepted changes are written to `{vaultRoot}/config/learning/`. The generation counter increments.
4. The next `/analyse` run loads the updated learning data - the loop repeats.

### Learning Data Loading (used by /analyse)

When `/analyse` reaches the taxonomy loading step, it also checks for learning data:

1. Check for `{vaultRoot}/config/learning/examples.yaml` and `{vaultRoot}/config/learning/preferences.yaml`.
2. If either exists, build `{{learning_section}}` with up to 3 gold examples, up to 3 anti-examples, and all preference notes. Format as a block that gets injected into extraction prompts after the taxonomy section.
3. If neither exists (first use, no learning data yet), set `{{learning_section}}` to empty string. The placeholder in the prompt stays blank.

