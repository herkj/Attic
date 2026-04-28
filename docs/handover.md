# Attic Handover

## Purpose

This document is the primary handover entry point for Attic. It explains why Attic exists, what state it is in today, and where a new owner should start.

## State of the world

- Nobody is using Attic in production yet.
- It has only been tested quickly with a small number of colleagues.
- There is no designated successor at the time of writing.
- If Attic is not touched for a few months, there is no immediate business risk.
- The learning loop (`/improve`) has not been fully demoed across a long sequence of sessions yet.

## Why Attic exists

Attic exists to make user research insights reusable across studies and teams, without buying a heavyweight external platform.

Current research practice often leaves findings trapped inside workshop boards and thick artifacts that are hard to search across teams. Commercial tools can solve parts of this problem, but they are often expensive, complex, and not tailored to Vipps' internal way of working.

Attic was started as an internal experiment to test whether Claude Code skills, markdown, and strict research conventions could deliver a practical alternative.

## Why these design choices

### From app-first to skills-first

Attic started as a full application with frontend navigation and interaction design work. That approach worked technically, but it introduced complexity that was not directly related to research quality.

The switch to skills-based CLI reduced the problem to its core:
- Put source material in a folder.
- Run slash commands with clear rules.
- Save structured outputs in another folder.

This removed login, user management, frontend state, and UI maintenance from the critical path.

### Why Atomic Research

Attic follows Atomic Research because it is concrete, practical, and easy to enforce at scale. It provides clear rules for moving from evidence to observations to insights, and those rules map well to deterministic skill flows.

### Henrik's notes

> "The point was to build something that could collect all the insights across all the studies we do across all teams."
>
> "Dovetail is extremely complicated and very expensive and not really tailored to what we need."
>
> "When skills came out, it felt like a huge redundant app. A folder plus slash commands did most of the job."

## What is in this repo

- `.claude/commands/` - operational skills (`/analyse`, `/review-observations`, `/synthesize`, etc.)
- `prompts/` - extraction and synthesis prompt templates
- `taxonomy/` - core and domain taxonomy files
- `templates/` - session, study, and report templates
- `docs/skill-patterns.md` - single source of truth for shared conventions
- `docs/proposals/` - future-work proposals (includes eval harness)

## Known direction and next priorities

The next owner should focus on two themes first:

1. **Quality loop maturity**
   - Build and run the eval harness proposed in `docs/proposals/eval-harness.md`.
2. **Output structure**
   - Improve how outputs are stored and discovered.
   - Current state is mostly one markdown file per study/session.
   - Better discoverability is likely needed for long-term cross-team value.

See `docs/proposals/improved-output-format.md` for options and recommendations.

## Repository ownership note

At time of writing, the repository has been hosted in a personal GitHub account (`herkj/Attic`) and is expected to be copied/moved into a Vipps organization repository as part of handover.

Treat links in old commit messages as potentially stale after transfer.

## No designated owner yet

There is no assigned long-term maintainer yet. The team should explicitly decide:

- who owns day-to-day triage,
- who approves changes to skills/prompts,
- who owns taxonomy quality,
- who drives adoption across teams.

Without explicit ownership, Attic is likely to stall even if the codebase is healthy.

## Where to start

1. Read this file.
2. Run through `docs/walkthrough.md`.
3. Read `docs/backlog.md` for sharp edges and low-risk fixes.
4. Read `docs/contributing-skills.md` before adding or modifying skills.
5. Use `docs/faq.md` for common operational issues.
6. Prioritize proposals in `docs/proposals/` based on owner capacity.
