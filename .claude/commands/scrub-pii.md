# /scrub-pii - Scrub PII from a source file

Scans a file for personally identifiable information and saves a redacted copy. Can be run standalone or is called by `/analyse`.

## Inputs

$ARGUMENTS - Path to a source file to scrub

## Steps

### 1. Load the "do not redact" list from taxonomy

Follow `docs/skill-patterns.md` - Taxonomy Loading to load the merged taxonomy. If the taxonomy defines `product_areas` and/or `external_entities` categories, extract all their `name` and `aliases` values.

These are named products, brands, and entities that must never be redacted, even if they look like proper nouns. A core-only taxonomy has no such categories - in that case this list is simply empty and you rely on the general redaction rules below.

### 2. Resolve or assign participant pseudonyms

Check whether the session file for this source already has a participant assignment in its frontmatter (the session file follows the `P{nn}-{Pseudonym}-{age}.md` convention - see `docs/skill-patterns.md` - Naming Rules). If a pseudonym is already assigned, reuse it.

If no assignment exists yet:
1. Ask: "How many participants are in this file, and do you know their country/region and gender?"
   - Accept answers like "2 participants, both women from Brazil" or "1 participant, region unknown"
   - Country/region and gender are optional context that make the generated pseudonym feel natural; they are never looked up in a file
   - If called with `{auto_mode} = true`, infer from context in the file and proceed
2. For each participant, generate a pseudonym (see `docs/skill-patterns.md` - Participant Naming Convention):
   - Assign the next available P-number (P01, P02, ...) sequentially across the study
   - Generate a plausible, clearly-fictional given name. If the participant's country/region is known, pick a common given name from that culture; if gender is known, match it; otherwise choose a neutral name
   - Ensure the name is unique within the study (skip any pseudonym already used)
   - Result: `P01 - {Pseudonym}`, `P02 - {Pseudonym}`, etc.
3. Write the assignment to the session frontmatter's `participant` block (one participant per session; for a group session, list each under a `participants:` array instead):
   ```yaml
   participant:
     id: P01
     pseudonym: {Pseudonym}
     gender: {gender or blank}
     country: {country/region or blank}
   ```

### 3. Scan for PII

Read the source file. Identify all instances of:

**Redact these:**
- The participant's real name (first name, last name, or full name) - replace with their pseudonym (e.g. "P01 - {Pseudonym}" on first mention)
- Interviewer's real name - replace with [Interviewer]
- Any other real person names - replace with [Person]
- Email addresses - replace with [Email]
- Phone numbers - replace with [Phone]
- Physical addresses - replace with [Address]
- Personal ID numbers (national ID, passport, etc.) - replace with [ID]
- Slack @mentions with real names - replace with [Person]

**Never redact these:**
- Everything in the taxonomy "do not redact" list from step 1
- City names used as market/location references (e.g. "Berlin", "São Paulo")
- Job titles and role descriptions
- Nationalities and languages
- Company or brand names not already covered by the taxonomy
- All formatting, structure, and non-personal content

### 4. Present findings

List all proposed redactions with the original text and proposed replacement. Ask: "Scrub all and save?"

If called from another skill with `{auto_mode} = true`, skip this step and apply all redactions automatically.

### 5. Save scrubbed file

Write the redacted content to `{original-filename}-scrubbed.md` in the same folder as the source file. Do not overwrite the original.

### 6. Summary

Show:
- Pseudonym assignments (e.g. "P01 - {Pseudonym} (region, gender)")
- Number of items redacted by type (names, emails, phones, etc.)
- Or "No PII found" if nothing was redacted
