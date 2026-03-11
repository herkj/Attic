# Attic Architecture

This document describes the current architecture of the Attic application, a UX research analysis platform.

## Overview

Attic uses a **4-tier hierarchical model** to organize research data, from raw sources through to synthesized insights:

```
Study
  └── Session
        ├── SourceArtifact
        │     ├── Highlight (evidence snippets)
        │     └── Facet (structured metadata)
        └── Observation
              └── SessionInsight
                    └── StudyInsight
```

## Core Concepts

### Study
A container for a coherent research effort. Studies have research questions and aggregate insights from multiple sessions.

### Session
One real-world interaction (interview, usability test, workshop). Each session is linked to a participant and contains source artifacts.

### SourceArtifact
Raw content attached to a session (transcripts, notes, surveys). Artifacts go through a processing pipeline:
1. **PII Scrubbing** - Removes personally identifiable information
2. **Facet Extraction** - Extracts structured metadata (product, topic, emotion, etc.)
3. **Observation Extraction** - Generates interpreted observations with supporting highlights

### Observation
An interpreted meaning extracted from source content. Observations capture:
- Problems users face
- Preferences they express
- Workarounds they employ
- Behavioral patterns

Observations are linked to Highlights (evidence snippets from the source text).

### SessionInsight
Session-level synthesis that aggregates observations from a single session into patterns, behaviors, or frictions.

### StudyInsight
Study-level aggregation that identifies patterns across multiple session insights, providing cross-session findings.

## Processing Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                      PROCESSING PIPELINE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Source Upload                                                 │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────────┐                                               │
│   │ PII Scrub   │ ──► Removes names, emails, phone numbers      │
│   └─────────────┘                                               │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────────┐                                               │
│   │ Extract     │ ──► Facets (metadata) + Observations          │
│   │ Facets/Obs  │     with Highlights (evidence)                │
│   └─────────────┘                                               │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────────┐                                               │
│   │ Session     │ ──► Aggregates observations into              │
│   │ Synthesis   │     session-level insights                    │
│   └─────────────┘                                               │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────────┐                                               │
│   │ Study       │ ──► Identifies patterns across                │
│   │ Synthesis   │     session insights                          │
│   └─────────────┘                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Key Files

### Pages
- `app/studies/` - Study management pages
- `app/sessions/` - Session workflow pages
- `app/repository/` - Knowledge graph visualization
- `app/settings/` - Application settings

### Server Actions
- `app/actions/study.ts` - Study CRUD operations
- `app/actions/session.ts` - Session CRUD and source processing
- `app/actions/observation.ts` - Observation management
- `app/actions/synthesis.ts` - Session and study synthesis
- `app/actions/insights-canvas.ts` - Study insights canvas operations
- `app/actions/jobs.ts` - Background job processing
- `app/actions/tags.ts` - Tag management

### AI Pipeline
- `lib/ai/scrubber.ts` - PII scrubbing
- `lib/ai/facet-extractor.ts` - Structured metadata extraction
- `lib/ai/observation-extractor.ts` - Observation extraction with highlights
- `lib/ai/session-synthesizer.ts` - Session-level synthesis
- `lib/ai/study-synthesizer.ts` - Study-level synthesis
- `lib/ai/config.ts` - AI model configuration

### Core Utilities
- `lib/status.ts` - Status enums and helpers
- `lib/prisma.ts` - Database client
- `lib/utils.ts` - General utilities

## Status Enums

All status values are defined in `lib/status.ts`:

### SourceStatus
- `pending` - Awaiting processing
- `scrubbed` - PII removed
- `facets_extracted` - Metadata extracted
- `extracted` - Observations extracted
- `processed` - Fully processed

### ObservationStatus
- `suggested` - AI-generated, pending review
- `approved` - Verified by researcher
- `rejected` - Excluded from synthesis

### SessionInsightStatus
- `draft` - Initial synthesis output
- `verified` - Confirmed by researcher
- `published` - Ready for study synthesis

### StudyInsightStatus
- `open` - Active insight
- `validated` - Confirmed pattern
- `stale` - Needs re-synthesis
- `deprecated` - No longer relevant

### StalenessState
- `fresh` - Up to date with source data
- `stale` - Underlying data has changed

## Staleness Propagation

When observations are edited, staleness propagates up the chain:
1. Observation marked stale
2. Linked SessionInsights marked stale
3. Linked StudyInsights marked stale

This enables the UI to show which insights need re-synthesis.

## Background Processing

Long-running AI tasks (scrubbing, extraction, synthesis) run as background jobs:

- Jobs are created via `app/actions/jobs.ts`
- Status is tracked in the `ProcessingJob` model
- The `ProcessingPill` component polls for updates
- Jobs cannot call `revalidatePath()` directly (runs outside request context)

## Data Model

See `prisma/schema.prisma` for the complete database schema. Key models:

| Model | Purpose |
|-------|---------|
| Study | Research project container |
| Session | Single research interaction |
| Participant | Research subject |
| SourceArtifact | Raw content (transcript, notes) |
| Highlight | Evidence snippet from source |
| Facet | Structured metadata |
| Observation | Interpreted finding |
| SessionInsight | Session-level synthesis |
| StudyInsight | Study-level synthesis |
| Tag | Taxonomy labels |
| ProcessingJob | Background task tracking |

## Tags and Facets

### Tags
User-managed taxonomy labels that can be applied to sessions, observations, and insights. Tags support:
- Aliases for flexible matching
- Localized names
- Categories for grouping

### Facets
AI-extracted structured metadata describing:
- Product/feature mentioned
- Topic/theme
- User role
- Journey stage
- Emotion/sentiment
- Device context
