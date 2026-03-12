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

### 2. Load participant name lists

Find the Attic project root via Glob for `**/participant-names.yaml`. Load it.

### 3. Resolve or assign participant pseudonyms

Check whether the session file for this source already has a `participants` block in its frontmatter (in the corresponding `session-XX.md`). If it does, use the existing pseudonym assignments.

If no assignments exist yet:
1. Ask: "How many participants are in this file, and do you know their country and gender?"
   - Accept answers like "2 participants, both Norwegian women" or "1 participant, Finnish, gender unknown"
   - If called with `{auto_mode} = true`, make a best guess from context in the file and proceed
2. For each participant:
   - Assign the next available P-number (P01, P02, ...)
   - Pick a name from `participant-names.yaml`:
     - If country known: use that country's list
     - If gender known: use the matching gender list
     - If gender unknown: use `gender_neutral` list
     - Pick in order from the top of the list, skipping any already used in this study
   - Result: `P01 - Astrid`, `P02 - Robin`, etc.
3. Write the assignments to the session frontmatter:
   ```yaml
   participants:
     - id: P01
       pseudonym: Astrid
       country: norway
       gender: female
   ```

### 4. Scan for PII

Read the source file. Identify all instances of:

**Redact these:**
- The participant's real name (first name, last name, or full name) - replace with their pseudonym (e.g. "Astrid" or "P01 - Astrid" on first mention)
- Interviewer's real name - replace with [Interviewer]
- Any other real person names - replace with [Person]
- Email addresses - replace with [Email]
- Phone numbers - replace with [Phone]
- Physical addresses - replace with [Address]
- Personal ID numbers (national ID, passport, etc.) - replace with [ID]
- Slack @mentions with real names - replace with [Person]

**Never redact these:**
- Everything in the taxonomy "do not redact" list from step 1
- City names used as market references (e.g. "Oslo", "Copenhagen")
- Job titles and role descriptions
- Nationalities and languages
- Company or brand names not already covered by the taxonomy
- All formatting, structure, and non-personal content

### 5. Present findings

List all proposed redactions with the original text and proposed replacement. Ask: "Scrub all and save?"

If called from another skill with `{auto_mode} = true`, skip this step and apply all redactions automatically.

### 6. Save scrubbed file

Write the redacted content to `{original-filename}-scrubbed.md` in the same folder as the source file. Do not overwrite the original.

### 7. Summary

Show:
- Pseudonym assignments (e.g. "P01 - Astrid (Norway, female)")
- Number of items redacted by type (names, emails, phones, etc.)
- Or "No PII found" if nothing was redacted
