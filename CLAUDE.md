# CLAUDE.md - Attic

## What This Is

Attic is a CLI-based UX research toolkit built as Claude Code skills. It processes interview transcripts and research notes into structured, tagged observations and insights - with human approval at every step.

Markdown files are the database. Claude is the AI engine. Obsidian is the UI.

## Language Rules

- **Never use em dashes.** Use a regular hyphen (-) or rewrite the sentence.
- Code and documentation in English. Research content may be in Norwegian - preserve it as-is, never translate.

## Key Principles

- AI suggests, humans decide
- Every insight traces back to evidence (via wikilinks)
- Taxonomy guides extraction toward canonical values
- Skills are lean orchestration - shared patterns live here in CLAUDE.md

## Skills

| Skill | Purpose |
|-------|---------|
| `/setup-attic` | One-time setup: creates vault structure and generates domain taxonomy |
| `/new-study` | Interactive study setup with research questions and method |
| `/analyse` | Single entry point for all analysis: asks source type, runs transcript, notes, or report pipeline |
| `/review-observations` | Interactive human approval of observations |
| `/synthesize` | Session-level insight synthesis |
| `/study-synthesize` | Cross-session study synthesis with prevalence |
| `/debrief` | Post-interview technique feedback |
| `/scrub-pii` | Scrub PII from a source file, assigns participant pseudonyms |
| `/fix-transcript` | Clean ASR/automated transcript errors before analysis |
| `/improve` | Analyze review history, propose extraction improvements, track quality over generations |

---

## Shared Patterns (used by all skills)

### Vault Structure

Research data lives in the user's Obsidian vault (not this repo):

```
Research/
  inbox/                              # Drop raw notes/transcripts here
    processed/                        # Originals moved here after /analyse
  Studies/
    {Study Name}/
      study.md                        # Study metadata, taxonomy ref, research questions
      sessions/
      session-01.md                   # Observations + insights
      session-01-sources/
        transcript.md                 # Raw transcript
        transcript-scrubbed.md        # PII-scrubbed
        notes.md                      # Observer notes
        notes-scrubbed.md             # PII-scrubbed notes
        debrief.md                    # Interview technique feedback (optional)
```

### File Discovery

When a skill receives a path or reference:
1. If it's a full file path, use it directly
2. If it looks like "Study Name/session-01", resolve to `Research/Studies/{Study Name}/sessions/session-01.md`
3. If ambiguous, use Glob to find matching files and ask the user to pick
4. To find the vault root, Glob for `**/Research/Studies/`

When Attic lives inside `{vault}/Attic/`, Research data is at `../Research/` relative to the Attic root. The `**/Research/Studies/` glob pattern still works regardless of location.

### Taxonomy Loading

Skills that need taxonomy (analyse) follow this pattern:
1. Read the study.md frontmatter `taxonomy` field (e.g., "vipps-mobilepay")
2. Find the Attic project root via Glob for `**/taxonomy/core.yaml`
3. Always load `taxonomy/core.yaml`
4. If taxonomy is not "core", also load `taxonomy/{value}.yaml`
5. Find the Research root (see File Discovery). Check for any `.yaml` files in `Research/config/taxonomy/`. If any exist, merge them as a third layer on top of the previous two. Multiple files are all merged - this allows teams to maintain separate taxonomy files per domain or topic.
6. Merge and format as a flat list per category:
   ```
   ## Product Area
   - Payment API (aliases: payment-api, betalings-api)
   - Checkout (aliases: checkout, kasse)
   ## Emotion
   - Frustration (aliases: frustrasjon)
   ```

### Observation Format (in session files)

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

### Session Insight Format

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

### Study Insight Format

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

### Participant Naming Convention

Each participant gets a code + a memorable pseudonym: `P01 - Astrid`, `P02 - Robin`.

The pseudonym is picked from `participant-names.yaml` in the Attic root:
- If participant country is known, pick from that country's list
- If participant gender is known, pick from the matching gender list within that country
- If gender is unknown, pick from the shared `gender_neutral` list (one list, all countries)
- Assign names in order - P01 gets the first available name, P02 the second, etc.
- Within a study, names must be unique

This makes sessions easier to remember and discuss without using real names.

### PII Scrubbing Rules

**Redact:** Replace real participant names with their assigned pseudonym (e.g. "Astrid" or "P01 - Astrid"). Replace interviewer names with [Interviewer]. Replace other person names with [Person]. Replace emails -> [Email], phones -> [Phone], addresses -> [Address], ID numbers -> [ID], Slack @mentions with real names.

**Keep unchanged:** All names and aliases found in the taxonomy (product_areas and external_entities), city names as market references, job titles, nationalities, all formatting and structure

### Common Anti-Patterns (apply to all extraction and synthesis)

- Do not create an observation for every utterance - focus on distinct insights
- Do not merge unrelated observations just to reduce count
- Do not invent tags outside the taxonomy
- Do not frame insights as recommendations - they are factual findings
- Do not translate Norwegian content
- Do not discard outliers - they may be the most interesting signals
- When stated preference contradicts behavior, create separate observations for each
- Always err toward surfacing more observations rather than fewer - the human review step exists to filter, but missed observations are unrecoverable

### Learning System

Attic improves over time by tracking review decisions and feeding them back into extraction. Learning data lives in the Research folder (team-specific, not in the Attic git repo), following the same pattern as custom taxonomy.

**Principle:** Optimize for calibration - understanding what the human values - not for approval rate alone. The system should always err toward surfacing too much rather than too little.

#### Learning Data Location

```
Research/
  config/
    learning/
      journal.yaml          # Accumulated review decisions (one entry per /review-observations run)
      examples.yaml          # Gold and anti-examples for extraction prompts
      preferences.yaml       # Team preference notes and type approval rates
      improve-history.yaml   # When /improve ran, what changed, metrics snapshot
    taxonomy/
      learned.yaml           # Taxonomy additions from /improve (tags, aliases)
```

#### Generation Tracking

Each `/improve` run increments a generation counter. When `/analyse` runs extraction, it reads the current generation from `improve-history.yaml` (count of entries, or 0 if no file) and writes `learning_generation: {n}` to the session file frontmatter. When `/review-observations` logs to the journal, it copies this generation number into the entry. This links each review to the version of learning data that produced its observations, enabling before/after comparison.

#### Four Metrics

Computed from journal entries by `/improve`:

| Metric | Formula | Measures | Better = |
|--------|---------|----------|----------|
| Miss rate | `added_by_user / (approved + added_by_user)` | Things the human wanted that the AI didn't surface | Lower |
| Noise rate | `rejected / extracted` | Things the AI surfaced that the human didn't want | Lower |
| Edit rate | `edited_by_user / approved` | How often the first pass needed correction | Lower |
| Tag accuracy | `1 - (tag_edits / approved)` | Whether the AI assigns the right tags | Lower miss, higher accuracy |

Miss rate is the most important. A system that never misses what the human would catch is doing its job, even if it produces some noise. Noise is cheap to filter. Misses are invisible and unrecoverable.

#### How the Loop Works

1. `/analyse` loads learning data (examples + preferences) alongside taxonomy, records the current generation in session frontmatter, and runs extraction with the learning section injected into prompts.
2. `/review-observations` collects approve/reject/edit/add decisions with rejection reasons, logs everything to `journal.yaml` with the generation tag, and nudges the user to run `/improve` after 5+ sessions.
3. `/improve` reads the journal, computes metrics by generation, proposes example bank updates, preference notes, taxonomy additions, and extraction rules. Accepted changes are written to `Research/config/learning/`. The generation counter increments.
4. The next `/analyse` run loads the updated learning data - the loop repeats.

#### Learning Data Loading (used by /analyse)

When `/analyse` reaches the taxonomy loading step, it also checks for learning data:

1. Check for `Research/config/learning/examples.yaml` and `Research/config/learning/preferences.yaml`.
2. If either exists, build `{learning_section}` with up to 3 gold examples, up to 3 anti-examples, and all preference notes. Format as a block that gets injected into extraction prompts after the taxonomy section.
3. If neither exists (first use, no learning data yet), set `{learning_section}` to empty string. The placeholder in the prompt stays blank.

---

## Project Structure

```
{VaultRoot}/
  Attic/                        <- git repo, cloned here
    .claude/commands/           # Skill definitions (slash commands)
    prompts/                    # AI prompt templates (referenced by skills)
    taxonomy/                   # core.yaml + example domain files (version controlled)
    templates/                  # Markdown templates for studies, sessions, reports
    docs/                       # Architecture docs and reference materials
    config/                     # gitignored - user-specific, survives git pull
      taxonomy/
        custom.yaml             # Generated by /setup-attic from user's websites
  Research/                     # Created by /setup-attic
    inbox/
    Studies/
    Reports/
    Repository/
    config/
      learning/
        journal.yaml            # Review decisions log (written by /review-observations)
        examples.yaml           # Gold + anti examples (written by /improve)
        preferences.yaml        # Team preferences (written by /improve)
        improve-history.yaml    # Improvement run log (written by /improve)
      taxonomy/
        learned.yaml            # Taxonomy additions from /improve
```
