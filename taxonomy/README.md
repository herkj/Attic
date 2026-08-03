# Taxonomy

The taxonomy is split into two layers:

- **`core.yaml`** - Universal tags that apply to any UX research project (emotions, feedback types, journey stages, user segments, etc.)
- **`<domain>.yaml`** - Domain-specific tags for a particular business or product area

This repo ships only `core.yaml`. Domain taxonomy is user-specific and lives in your vault, not here.

Anything tied to a specific business, product, or industry - product areas, competitors and other external entities, pricing/commercial concepts - belongs in a domain taxonomy, **not** in `core.yaml`. Keeping core universal is what lets Attic work unchanged across any domain. Domain files can define categories such as `product_areas` and `external_entities`, which some skills (PII scrubbing, transcript fixing) and prompts use when present.

## Adding a domain taxonomy

Run `/setup-attic` and choose to generate a domain taxonomy from your product/company websites. It is saved to your vault at `Research/config/taxonomy/{name}.yaml` (not committed to this repo). Set `taxonomy: {name}` in a study's frontmatter to load it alongside `core.yaml`. You can also hand-write additional `.yaml` files in `Research/config/taxonomy/` - all of them are merged.

## How skills use taxonomy

Skills load `core.yaml` + the relevant domain file, merge them, and include the full tag list in the AI prompt. The AI is instructed to only use tags from the provided taxonomy.
