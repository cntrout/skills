# cover-letter skill — v1.1

This skill creates **five distinct** cover letters grounded in:
- The **job-research** skill output
- The **candidate career file** you supply
- The **natural-tone** style (human cadence; bans emojis, em dashes, and ellipses)

No fabrication: biographical claims come only from your provided profile.

## Quick Start
1. Run **job-research** for your target role.
2. Prepare your career file (structured YAML or pasted text).
3. Invoke this skill with:
   - `job_research`: the full object from job-research
   - `career.source_mode: user_supplied_only`
   - `career.user_profile_struct` **or** `career.user_profile_markdown`
   - optional `natural_tone_knobs`

### Minimal Example
```yaml
job_research: { ...full object from job-research... }
career:
  source_mode: "user_supplied_only"
  user_profile_markdown: |
    # Name
    Headline: ...
    Positions:
      - Company: ...
        Title: ...
        Dates: ...
        Bullets:
          - ...
natural_tone_knobs: { brevity: 6, specificity: 8, warmth: 5, idiom_density: 2, formality: 5, jitter: 4 }
letter_length: "standard"
include_header_block: true
include_selection_guidance: true
```

## Outputs
- `letters[5]`: `approach`, `word_count`, `body`, `facts_used`, `company_specifics`
- `provenance`: `candidate_facts: user_supplied`; array of links from job-research
- Optional `guidance` with selection tips and edit suggestions

## Tips
- Keep candidate facts concise and quantified.
- If the job-research lacks specifics (e.g., pricing model), do not speculate; keep language conditional.
- After pasting into Google Docs, apply your template styles.

## Version
- v1.1 — 2025-10-20: hardened sourcing rules, deterministic schemas, and natural-tone bans.
