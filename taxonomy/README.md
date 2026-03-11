# Taxonomy

The taxonomy is split into two layers:

- **`core.yaml`** - Universal tags that apply to any UX research project (emotions, feedback types, journey stages, user segments, etc.)
- **`<domain>.yaml`** - Domain-specific tags for a particular business or product area

## Current domains

- `vipps-mobilepay.yaml` - Vipps MobilePay products, competitors, and payment-specific tags

## Adding a new domain

1. Copy the structure from an existing domain file
2. Add product areas, domain topics, user segments, and external entities relevant to your context
3. Reference it in your study's config

## How skills use taxonomy

Skills load `core.yaml` + the relevant domain file, merge them, and include the full tag list in the AI prompt. The AI is instructed to only use tags from the provided taxonomy.
