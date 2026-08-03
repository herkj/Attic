# Taxonomy

The taxonomy is split into two layers:

- **`core.yaml`** - Universal tags that apply to any UX research project (emotions, feedback types, journey stages, user segments, etc.)
- **`<domain>.yaml`** - Domain-specific tags for a particular business or product area

This repo ships only `core.yaml`. Domain taxonomy is user-specific and lives in your vault, not here.

## Adding a domain taxonomy

Run `/setup-attic` and choose to generate a domain taxonomy from your product/company websites. It is saved to your vault at `Research/config/taxonomy/{name}.yaml` (not committed to this repo). Set `taxonomy: {name}` in a study's frontmatter to load it alongside `core.yaml`. You can also hand-write additional `.yaml` files in `Research/config/taxonomy/` - all of them are merged.

## How skills use taxonomy

Skills load `core.yaml` + the relevant domain file, merge them, and include the full tag list in the AI prompt. The AI is instructed to only use tags from the provided taxonomy.
