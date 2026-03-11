# /scrub-pii - Scrub PII from a source file

Scans a file for personally identifiable information and saves a redacted copy. Can be run standalone or is called by `/analyse`.

## Inputs

$ARGUMENTS - Path to a source file to scrub

## Steps

### 1. Load the "do not redact" list from taxonomy

Follow CLAUDE.md "Taxonomy Loading" pattern to load the merged taxonomy. Extract all `name` and `aliases` values from:
- `product_areas`
- `external_entities`

These are named products, brands, and entities that must never be redacted, even if they look like proper nouns.

### 2. Scan for PII

Read the source file. Identify all instances of:

**Redact these:**
- Personal names (first name, last name, or full name of real people) → [Participant] for the person being interviewed, [Interviewer] for the interviewer, [Person] for anyone else mentioned
- Email addresses → [Email]
- Phone numbers → [Phone]
- Physical addresses → [Address]
- Personal ID numbers (national ID, passport, etc.) → [ID]
- Slack @mentions with real names → [Person]

**Never redact these:**
- Everything in the taxonomy "do not redact" list from step 1
- City names used as market references (e.g. "Oslo", "Copenhagen")
- Job titles and role descriptions
- Nationalities and languages
- Company or brand names not already covered by the taxonomy
- All formatting, structure, and non-personal content

### 3. Present findings

List all proposed redactions with the original text and proposed replacement. Ask: "Scrub all and save?"

If called from another skill with `{auto_mode} = true`, skip this step and apply all redactions automatically.

### 4. Save scrubbed file

Write the redacted content to `{original-filename}-scrubbed.md` in the same folder as the source file. Do not overwrite the original.

### 5. Summary

Show: number of items redacted by type (names, emails, phones, etc.), or "No PII found" if nothing was redacted.
