# Proposal: Transcript Chunking for Long Sessions

**Status:** Proposed, not implemented.
**Owner:** Open - first picker-upper claims it.
**Last updated:** 2026-04-28

---

## TL;DR

Long transcripts should be chunked before extraction to reduce context overflow risk and improve consistency. Chunking should preserve speaker boundaries, run extraction per chunk, then merge and deduplicate observations at session level.

---

## Why this matters

As sessions get longer, prompt context includes:
- full taxonomy section,
- learning section,
- extraction instructions,
- transcript content.

This can exceed practical context windows or degrade model quality even before hard limits are hit.

---

## What success looks like

1. `/analyse` handles 90+ minute transcripts reliably.
2. Observation quality remains stable versus short-session baseline.
3. Evidence snippets still point to useful source text.
4. Duplicate observations introduced by chunk boundaries are minimized.

---

## Proposed design

### Chunking strategy

- Split transcript by speaker-turn boundaries.
- Target chunk size by token budget, not line count.
- Use overlap windows to preserve context at boundaries.

Suggested defaults (first version):
- `target_tokens`: 5,000
- `overlap_tokens`: 500
- `max_chunk_count_warning`: 12

### Processing flow

1. Preprocess transcript into ordered chunks.
2. For each chunk:
   - run standard extraction prompt,
   - tag each observation with `chunk_id`.
3. Merge all observations.
4. Deduplicate near-identical observations.
5. Write merged output into session file as normal.

### Deduplication rules (v1)

Two observations are merge candidates when:
- same `Type`,
- high lexical similarity in title/body,
- overlapping key tags.

If merged:
- keep strongest evidence snippet,
- keep union of tags,
- append a note like "Seen in chunks 2 and 3".

---

## Data and traceability

Add chunk trace metadata during extraction:

```yaml
chunk:
  id: chunk-03
  start_turn: 120
  end_turn: 176
```

This metadata can stay internal and be stripped from final markdown if too noisy.

---

## Implementation plan

1. Add chunking section to `.claude/commands/analyse.md` for transcript path.
2. Define chunking pseudocode in the skill doc.
3. Add deduplication heuristic in the same flow.
4. Test on one short and one long transcript fixture.
5. Record known limitations in `docs/faq.md`.

---

## Open questions

1. Should chunking be always-on for transcripts or only above a threshold?
2. Should overlap be token-based or turn-based?
3. How should evidence references be stored if observations merge across chunks?
4. Should chunk metadata be visible in final session markdown?

---

## Out of scope

- Real-time streaming chunk extraction.
- Rewriting old study files for backfill.
- Complex semantic dedupe models in v1.
