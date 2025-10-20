---
name: "apply-to-job-posting"
description: "One-click pipeline: job URL + pasted posting + verified profile -> run job-research, then generate LinkedIn outreach and five cover-letter variants with natural-tone; return a consolidated package."
metadata:
  version: "1.0"
  updated: "2025-10-20"
  dependencies:
    - "job-research"
    - "linkedin-outreach"
    - "cover-letter"
    - "natural-tone"
  notes:
    - "If dependent skills are not installed, degrade gracefully with instructions for manual runs."
---

# Apply to Job Posting — v1.0 (Orchestrator)

_Last updated: 2025-10-20_

## Purpose
Given a **job posting URL**, the **pasted job description**, and the **candidate’s verified profile file**, this skill orchestrates:
1) **job-research** -> structured brief
2) **linkedin-outreach** -> 3 stages x 3-5 variants
3) **cover-letter** -> five distinct letters
4) Consolidated digest + optional raw JSON

## Inputs Schema (YAML)
```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "apply-to-job-posting input"
type: object
additionalProperties: false
required: ["job_post","candidate"]
properties:
  job_post:
    type: object
    required: ["url","text"]
    properties:
      url: { type: "string", format: "uri" }
      text: { type: "string" }
      platform: { type: ["string","null"] }
      posted_at: { type: ["string","null"] }
  candidate:
    type: object
    required: ["source_mode"]
    properties:
      source_mode:
        type: "string"
        enum: ["user_supplied_only"]
        default: "user_supplied_only"
      user_profile_struct:
        type: ["object","null"]
      user_profile_markdown:
        type: ["string","null"]
  knobs:
    type: object
    default: { brevity: 6, specificity: 8, warmth: 5, idiom_density: 2, formality: 5, jitter: 4 }
    properties:
      brevity: { type: integer, minimum: 0, maximum: 10 }
      specificity: { type: integer, minimum: 0, maximum: 10 }
      warmth: { type: integer, minimum: 0, maximum: 10 }
      idiom_density: { type: integer, minimum: 0, maximum: 10 }
      formality: { type: integer, minimum: 0, maximum: 10 }
      jitter: { type: integer, minimum: 0, maximum: 10 }
  cover_letter_length:
    type: "string"
    enum: ["short","standard","long"]
    default: "standard"
  artifacts:
    type: object
    default: { digest_markdown: true, include_raw_json: true }
    properties:
      digest_markdown: { type: boolean }
      include_raw_json: { type: boolean }
```

## Output Schema (YAML)
```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "apply-to-job-posting output"
type: object
additionalProperties: false
required: ["generated_at","provenance","results"]
properties:
  generated_at: { type: "string", format: "date-time" }
  provenance:
    type: object
    required: ["candidate_facts","job_posting_source","pipeline"]
    properties:
      candidate_facts: { type: "string", enum: ["user_supplied"] }
      job_posting_source: { type: "string", enum: ["user_supplied_text"] }
      pipeline:
        type: array
        items:
          type: object
          required: ["step","status"]
          properties:
            step: { type: "string", enum: ["job-research","linkedin-outreach","cover-letter"] }
            status: { type: "string", enum: ["executed","skipped"] }
            note: { type: ["string","null"] }
  results:
    type: object
    required: ["job_research","linkedin_outreach","cover_letters"]
    properties:
      job_research: { type: "object" }
      linkedin_outreach: { type: "object" }
      cover_letters: { type: "object" }
  artifacts:
    type: ["object","null"]
    properties:
      digest_markdown: { type: ["string","null"] }
      files:
        type: array
        items:
          type: object
          properties:
            filename: { type: "string" }
            kind: { type: "string", enum: ["markdown","json"] }
            size_bytes: { type: "integer" }
```

## Execution Flow
1) Validate inputs and enforce verified-only profile facts.
2) Run **job-research** using `job_post.url` and `job_post.text`; embed the original posting.
3) Run **linkedin-outreach** using job-research output + candidate profile + `knobs`.
4) Run **cover-letter** using the same inputs + `cover_letter_length`.
5) Build a digest Markdown with summary, outreach, and letters.
6) Return raw per-skill outputs when requested.

## Natural-Tone
No emojis, no em dashes, no ellipses; avoid boilerplate; prefer concrete nouns and short verbs; vary cadence.

## Degraded Mode
If a dependent skill is missing, skip it and note the limitation in `provenance.pipeline`.

## Example Invocation
```yaml
job_post:
  url: "https://example.com/jobs/123"
  text: |
    Senior Product Manager
    Remote - Full-time
    About the role...
  platform: "LinkedIn"
  posted_at: null

candidate:
  source_mode: "user_supplied_only"
  user_profile_struct:
    name: "Your Name"
    headline: "Product leader focused on growth and experimentation"
    positions:
      - company: "iVisa"
        title: "Director of Product Management"
        dates: "2025–Present"
        bullets:
          - "Delivered $32M incremental revenue in one quarter via funnel optimization"
          - "Ran 300+ experiments over three years"

knobs: { brevity: 6, specificity: 8, warmth: 5, idiom_density: 2, formality: 5, jitter: 4 }
cover_letter_length: "standard"
artifacts: { digest_markdown: true, include_raw_json: true }
```
