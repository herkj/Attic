# Attic Backlog

This backlog captures known sharp edges that are safe and useful for a new owner to pick up.

Each item includes: what is wrong, why it matters, suggested fix, and rough effort.

## 1) Participant frontmatter shape mismatch

**What is wrong**
- `templates/session.md` defines a single `participant:` object.
- `/scrub-pii` describes writing a `participants:` list.

**Why it matters**
- Tools and humans can read/write different shapes.
- Parsing scripts and manual edits become inconsistent.

**Suggested fix**
- Decide one canonical shape for session frontmatter.
- Update `templates/session.md`, `.claude/commands/scrub-pii.md`, and any references in docs.
- Add a migration note for old files.

**Effort**
- Small (1-2 hours).

## 2) Participant name pool exhaustion behavior

**What is wrong**
- `participant-names.yaml` has finite lists.
- No explicit fallback strategy if a study exceeds available names.

**Why it matters**
- Long-running programs can hit undefined behavior.
- Collisions or repeated pseudonyms can break readability.

**Suggested fix**
- Add deterministic fallback rules in `/scrub-pii`:
  - append numeric suffix after list exhaustion,
  - or move to combined country + neutral pool,
  - and always enforce per-study uniqueness.
- Document fallback in `docs/skill-patterns.md`.

**Effort**
- Small to medium (2-4 hours).

## 3) No transcript chunking for long sessions

**What is wrong**
- Large transcripts can approach or exceed practical context limits once taxonomy + learning section are included.

**Why it matters**
- Increases extraction failures and quality drift.
- Hard to reason about partial failures.

**Suggested fix**
- Implement chunking proposal in `docs/proposals/transcript-chunking.md`.

**Effort**
- Medium (1-2 days).

## 4) `pico.outcome` substitution is not explicitly standardized

**What is wrong**
- `pico.outcome` is appended into `{{focus_areas}}` in `.claude/commands/analyse.md`.
- It follows skill logic, but this path is not clearly separated from prompt placeholder substitution rules.

**Why it matters**
- Future maintainers may confuse skill variable assembly and prompt placeholder substitution.

**Suggested fix**
- Add a short note in `docs/skill-patterns.md` under Prompt Substitution Rule:
  - skill-internal composition happens first,
  - prompt placeholder substitution check happens last.
- Add a tiny example for `{{focus_areas}}`.

**Effort**
- Small (30-60 minutes).

## 5) Session role vocabulary mismatch

**What is wrong**
- `templates/session.md` currently implies a limited role set.
- `/analyse` inference examples include broader roles (partner, agency, developer).

**Why it matters**
- Frontmatter can suggest constraints that the workflow does not actually enforce.

**Suggested fix**
- Make role field free-text in template guidance, with optional suggested values.
- Keep examples in `/analyse` but avoid implying a hard enum.

**Effort**
- Small (30-60 minutes).

## 6) `Research/Repository/` is scaffolded but mostly unused

**What is wrong**
- `/setup-attic` creates `Research/Repository/`.
- Current skills do not meaningfully populate it.

**Why it matters**
- New users expect functionality that is not there yet.

**Suggested fix**
- Either:
  - remove folder creation for now, or
  - explicitly assign it a role (for cross-study artifacts and exports).
- Align this with `docs/proposals/cross-study-synthesis.md` and `docs/proposals/improved-output-format.md`.

**Effort**
- Small if remove, medium if activate.

## 7) In-repo `config/` is not an active path

**What is wrong**
- Current workflow writes dynamic taxonomy/learning data to `Research/config/...` in the vault.
- In-repo `config/` is not part of normal runtime behavior.

**Why it matters**
- Creates ambiguity for maintainers about which config path is authoritative.

**Suggested fix**
- Decide one strategy:
  - keep `config/` only for local dev notes and document that,
  - or remove/empty it to avoid confusion.

**Effort**
- Small (30-60 minutes).

## 8) Auto-mode A6 code block reads like executable code

**What is wrong**
- `.claude/commands/analyse.md` step A6 includes a Python `re.sub` snippet.
- In practice, this is behavioral intent for the agent, not a script that is run as-is.

**Why it matters**
- New maintainers can misinterpret this as a concrete runtime component.

**Suggested fix**
- Rewrite that section as explicit behavior:
  - "Replace all observation status markers `[?]` with `[x]` in the session file."
- Keep code snippet only if a real script path exists.

**Effort**
- Small (30 minutes).

## 9) Output discoverability is weak at scale

**What is wrong**
- Core output is one markdown file per session/study.
- Cross-study navigation and search remain mostly manual.

**Why it matters**
- This is the main risk to long-term adoption.

**Suggested fix**
- Prioritize `docs/proposals/improved-output-format.md`.
- Choose a near-term output improvement and implement one increment first.

**Effort**
- Medium to large, depends on option.

## Prioritization suggestion

### Do first (low effort, high clarity)
1. Participant shape mismatch
2. Role vocabulary mismatch
3. Auto-mode A6 wording
4. `pico.outcome` substitution clarity note

### Do next (enables scale)
5. Transcript chunking
6. Output format improvements
7. Cross-study synthesis
