# Knowledge Synthesis Skill (Anthropic)

> Source: github.com/anthropics/knowledge-work-plugins/enterprise-search/skills/knowledge-synthesis/SKILL.md
> Saved as reference material for Attic skill development.

The last mile of enterprise search. Takes raw results from multiple sources and produces a coherent, trustworthy answer.

## Deduplication

### Cross-Source Deduplication

The same information often appears in multiple places. Identify and merge duplicates:

**Signals that results are about the same thing:**
- Same or very similar text content
- Same author/sender
- Timestamps within a short window (same day or adjacent days)
- References to the same entity (project name, document, decision)
- One source references another ("as discussed in chat", "per the email", "see the doc")

**How to merge:**
- Combine into a single narrative item
- Cite all sources where it appeared
- Use the most complete version as the primary text
- Add unique details from each source

### Deduplication Priority

When the same information exists in multiple sources, prefer:
1. The most complete version (fullest context)
2. The most authoritative source (official doc > chat)
3. The most recent version (latest update wins for evolving info)

### What NOT to Deduplicate

Keep as separate items when:
- The same topic is discussed but with different conclusions
- Different people express different viewpoints
- The information evolved meaningfully between sources (v1 vs v2 of a decision)
- Different time periods are represented

## Confidence Levels

Not all results are equally trustworthy. Assess confidence based on:

### Freshness

| Recency | Confidence impact |
|---------|------------------|
| Today / yesterday | High confidence for current state |
| This week | Good confidence |
| This month | Moderate - things may have changed |
| Older than a month | Lower confidence - flag as potentially outdated |

### Authority

| Source type | Authority level |
|-------------|----------------|
| Official wiki / knowledge base | Highest - curated, maintained |
| Shared documents (final versions) | High - intentionally published |
| Email announcements | High - formal communication |
| Meeting notes | Moderate-high - may be incomplete |
| Chat messages (thread conclusions) | Moderate - informal but real-time |
| Draft documents | Low - not finalized |

### Expressing Confidence

When confidence is high (multiple fresh, authoritative sources agree):
- Direct statement

When confidence is moderate (single source or somewhat dated):
- Qualify with "Based on..." and flag potential staleness

When confidence is low (old data, informal source, or conflicting signals):
- Flag explicitly, suggest verification

### Conflicting Information

When sources disagree, always surface conflicts rather than silently picking one version.

## Summarization Strategies

### For Small Result Sets (1-5 results)
Present each result with context. No summarization needed - give the user everything.

### For Medium Result Sets (5-15 results)
Group by theme and summarize each group.

### For Large Result Sets (15+ results)
Provide a high-level synthesis with the option to drill down.

### Summarization Rules
- Lead with the answer, not the search process
- Do not list raw results - synthesize them into narrative
- Group related items from different sources together
- Preserve important nuance and caveats
- Include enough detail that the user can decide whether to dig deeper

## Synthesis Workflow

1. Deduplicate - merge same info from different sources
2. Cluster - group related results by theme/topic
3. Rank - order clusters and items by relevance to query
4. Assess confidence - freshness x authority x agreement
5. Synthesize - produce narrative answer with attribution
6. Format - choose appropriate detail level for result count

## Anti-Patterns

**Do not:**
- List results source by source
- Include irrelevant results just because they matched a keyword
- Bury the answer under methodology explanation
- Present conflicting info without flagging the conflict
- Omit source attribution
- Present uncertain information with the same confidence as well-supported facts
- Summarize so aggressively that useful detail is lost

**Do:**
- Lead with the answer
- Group by topic, not by source
- Flag confidence levels when appropriate
- Surface conflicts explicitly
- Attribute all claims to sources
- Offer to go deeper when result sets are large
