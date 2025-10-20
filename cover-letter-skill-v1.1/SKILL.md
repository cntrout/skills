---
name: "cover-letter"
description: "Generate five distinct, copy-ready cover letters grounded in the job-research output and the candidate’s verified career file; applies the natural-tone skill to avoid AI tells."
version: "1.1"
---

# Cover Letter — v1.1 (natural, verified, five-variant)

_Last updated: 2025-10-20_

## Purpose
Produce **five fundamentally different** cover letters for a specific role and company using:
- The **job-research** skill output (company, role, culture, links).
- The candidate’s **career experience** file (text or structured YAML).
- The **natural-tone** skill for clarity and human cadence (no emojis, no em dashes, no ellipses, no clichés).

## Guardrails
- **Verified-only sourcing for candidate facts.** Use only what appears in the provided career file or explicitly supplied `user_profile_*` fields. If details are missing, omit rather than invent.
- **Attribution awareness.** If job-research lacks a fact, do not assume it—leave it out or state it as an open question in an internal note.
- **Copy/paste friendly.** Output uses plain punctuation and simple paragraphing for Google Docs.
- **No private data pulls.** Do not bypass logins/bot protections; rely on supplied job-research artifact.

---

## Inputs Schema (YAML)
```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "cover-letter input"
type: object
additionalProperties: false
required: ["job_research","career"]
properties:
  job_research:
    type: object
    description: "Direct output object from job-research skill."
  career:
    type: object
    description: "Candidate career data."
    required: ["source_mode"]
    properties:
      source_mode:
        type: string
        enum: ["user_supplied_only"]
        default: "user_supplied_only"
      user_profile_struct:
        type: ["object","null"]
        description: "Structured profile data (name, headline, positions[], achievements[], skills[])."
      user_profile_markdown:
        type: ["string","null"]
        description: "Raw text/markdown to parse deterministically."
  natural_tone_knobs:
    type: object
    default: { brevity: 6, specificity: 8, warmth: 5, idiom_density: 2, formality: 5, jitter: 4 }
    properties:
      brevity: { type: integer, minimum: 0, maximum: 10 }
      specificity: { type: integer, minimum: 0, maximum: 10 }
      warmth: { type: integer, minimum: 0, maximum: 10 }
      idiom_density: { type: integer, minimum: 0, maximum: 10 }
      formality: { type: integer, minimum: 0, maximum: 10 }
      jitter: { type: integer, minimum: 0, maximum: 10 }
  letter_length:
    type: string
    enum: ["short","standard","long"]
    default: "standard"
  include_header_block:
    type: boolean
    default: true
  include_selection_guidance:
    type: boolean
    default: true
```

---

## Output Schema (YAML)
```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "cover-letter output"
type: object
additionalProperties: false
required: ["generated_at","provenance","letters"]
properties:
  generated_at: { type: string, format: "date-time" }
  provenance:
    type: object
    required: ["candidate_facts","job_research_refs"]
    properties:
      candidate_facts: { type: string, enum: ["user_supplied"] }
      job_research_refs: { type: array, items: { type: string } }
      notes: { type: array, items: { type: string } }
  letters:
    type: array
    minItems: 5
    maxItems: 5
    items:
      type: object
      required: ["approach","word_count","body","facts_used","company_specifics"]
      properties:
        approach: { type: string }
        word_count: { type: integer }
        body: { type: string }
        facts_used: { type: array, items: { type: string } }
        company_specifics: { type: array, items: { type: string } }
  guidance:
    type: ["object","null"]
    properties:
      selection_tips: { type: array, items: { type: string } }
      edit_suggestions: { type: array, items: { type: string } }
```

---

## Execution Flow
1) **Load guardrails** from natural-tone: avoid emojis, em dashes, ellipses, and AI clichés. Prefer concrete nouns, short verbs, specific metrics.
2) **Parse job_research**: role title, team/domain, must-have skills, success metrics, product names, and any links.
3) **Parse candidate profile** from `user_profile_struct` or `user_profile_markdown`. Treat as the **only** source of biographical facts.
4) **Map connections**: select 3–6 proof points that align with the role’s needs (metrics, launches, leadership, GTM, experimentation).
5) **Generate five distinct approaches** (not paraphrases):
   - Problem-Solver
   - Mission-Driven
   - Industry Expert
   - Growth Story
   - Direct Case
6) **Format for Google Docs**: optional header block; 200–450 words per letter based on `letter_length`.
7) **Return guidance**: which letter suits which culture/context; quick edits to tailor further.

---

## Style Constraints
- Plain punctuation only; no emojis, em dashes, or ellipses.
- Avoid boilerplate like “I am writing to express my interest.”
- Prefer short, declarative sentences. Vary cadence to reduce AI detectability.
- Cite concrete numbers where provided; never invent missing ones.

---

## Template (rendered per letter)
```
[Name]
[Email] | [Phone] | [LinkedIn]
[Date]

[Hiring Manager or Team]
[Company]

Dear [Name/Team],

[Body — 3–5 short paragraphs, 200–450 words]

Sincerely,
[Name]
```
