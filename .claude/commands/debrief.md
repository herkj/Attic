# /debrief - Post-interview technique feedback

Coaching tool that analyzes a scrubbed transcript and provides constructive feedback on interview technique.

## Inputs

$ARGUMENTS - Path to session file or scrubbed transcript

## Steps

### 1. Load transcript and study context
Find the scrubbed transcript (see `docs/skill-patterns.md` - File Discovery). Read study.md for research questions - these frame what "good coverage" means.

### 2. Analyse across six dimensions

**A. Question Quality** - Open-ended? Good use of probes ("why", "tell me more")? Leading questions avoided?

**B. Follow-up Depth** - Were interesting threads pursued or left hanging? Was the "why behind the why" explored? Was "why today and not yesterday?" used when relevant?

**C. Coverage** - Were research questions explored? Topics mentioned but not pursued? Time well-distributed?

**D. Bias and Leading** - Confirmation bias? Assumptions projected? Space given to disagree?

**E. Pivotal Moments** - Identify 2-3 most analytically rich moments. Was each explored fully?

**F. Forces of Progress** (only if switching/adoption topics arose)
- Push: dissatisfaction with current solution explored?
- Pull: attraction to new solution explored?
- Anxiety: fears about switching explored?
- Habit: anchoring to current solution explored?

### 3. Format output

```
## Interview Debrief - Session {nn}

### Strengths
### Areas for Improvement
### Missed Opportunities
### Pivotal Moments
### Forces of Progress (if applicable)
### Coverage Assessment
### Overall Assessment
```

Be specific - reference actual moments in the transcript. Be encouraging - this is coaching, not criticism. Norwegian transcripts analysed as-is.

### 4. Present and optionally save
Display to user. Offer to save as `{session_name}-sources/debrief.md` (e.g. `P01-Astrid-34-sources/debrief.md`).
