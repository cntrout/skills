---
name: "first-interview-prep"
description: "Prepare for a first-round interview: ingest a job post and verified career profile, research company and target team, and generate two PDFs with sources. No fabrication; unknowns remain blank and are labeled as unavailable."
metadata:
  version: "1.1"
  updated: "2025-10-22"
---

# First Interview Prep - v1.1

This skill prepares you for a first-call interview by turning a job post and your verified background into two polished PDFs: a Company & Team Overview and a Role Analysis & Interview Prep. It performs web research for public info and clearly labels anything it could not verify.

## Principles
- Verified-only: do not invent facts. Leave unverified fields blank with a note.
- Source-first: add source links for all non-trivial claims.
- Natural tone: no emojis, no em dashes, no ellipses.
- Respect gates: no bypassing logins, paywalls, or bot protections.

## Inputs (YAML schema)
```yaml
title: "first-interview-prep input"
type: object
additionalProperties: false
required: ["job_post","career","focus_team"]
properties:
  job_post:
    type: object
    properties:
      url: {{ type: ["string","null"] }}
      text: {{ type: ["string","null"] }}
      platform: {{ type: ["string","null"] }}
      posted_at: {{ type: ["string","null"] }}
    anyOf:
      - required: ["url"]
      - required: ["text"]

  career:
    type: object
    required: ["source_mode"]
    properties:
      source_mode:
        type: "string"
        enum: ["user_supplied_only"]
        default: "user_supplied_only"
      user_profile_struct: {{ type: ["object","null"] }}
      user_profile_markdown: {{ type: ["string","null"] }}

  focus_team:
    type: object
    required: ["name"]
    properties:
      name: {{ type: "string" }}
      include_titles:
        type: array
        default: ["Product Manager","Senior Product Manager","Principal Product Manager","Director of Product","VP Product","Chief Product Officer"]
        items: {{ type: "string" }}

  knobs:
    type: object
    default: {{ brevity: 6, specificity: 8, warmth: 5, idiom_density: 2, formality: 5, jitter: 4 }}
    properties:
      brevity: {{ type: integer, minimum: 0, maximum: 10 }}
      specificity: {{ type: integer, minimum: 0, maximum: 10 }}
      warmth: {{ type: integer, minimum: 0, maximum: 10 }}
      idiom_density: {{ type: integer, minimum: 0, maximum: 10 }}
      formality: {{ type: integer, minimum: 0, maximum: 10 }}
      jitter: {{ type: integer, minimum: 0, maximum: 10 }}

  artifacts:
    type: object
    default: {{ emit_pdfs: true, emit_markdown_digest: true, include_raw_json: true }}
    properties:
      emit_pdfs: {{ type: boolean }}
      emit_markdown_digest: {{ type: boolean }}
      include_raw_json: {{ type: boolean }}
```

## Outputs (YAML schema)
```yaml
title: "first-interview-prep output"
type: object
additionalProperties: false
required: ["generated_at","provenance","results"]
properties:
  generated_at: {{ type: "string" }}

  provenance:
    type: object
    required: ["candidate_facts","job_posting_source","search_log"]
    properties:
      candidate_facts: {{ type: "string", enum: ["user_supplied"] }}
      job_posting_source: {{ type: "string", enum: ["user_supplied_text","user_supplied_url","both"] }}
      search_log:
        type: array
        items:
          type: object
          properties:
            query: {{ type: "string" }}
            url: {{ type: "string" }}
            status: {{ type: "string" }}
            note: {{ type: ["string","null"] }}

  results:
    type: object
    required: ["company_team_pdf","role_prep_pdf","digest_markdown"]
    properties:
      company_team_pdf: {{ type: ["string","null"] }}
      role_prep_pdf: {{ type: ["string","null"] }}
      digest_markdown: {{ type: ["string","null"] }}
      raw: {{ type: ["object","null"] }}
```

## Research and Generation
1) Parse inputs. Persist original job text in Role PDF.
2) Use only public sources; include links. If not found, leave blank and label.
3) Map team members for the focus area via public profiles; include URLs.
4) Analyze role: compensation (confirmed/estimated/blank with reason), ranked qualifications, day-to-day estimate with caveats, experience-to-requirement matrix, talking points, and 10 interview questions.
5) Produce two PDFs with clean headings, tables where helpful, clickable links, and a Sources appendix.
6) If PDFs are disabled, return Markdown digest only.

## Example Invocation
```yaml
job_post:
  url: "https://www.linkedin.com/jobs/view/1234567890"
  text: |
    Senior Product Manager
    Remote - Full-time
    About the company...
    About the role...
  platform: "LinkedIn"
  posted_at: null

career:
  source_mode: "user_supplied_only"
  user_profile_markdown: |
    # Candidate Name
    Headline: Product leader focused on experimentation and CRO
    Positions:
      - Company: iVisa
        Title: Director of Product Management
        Dates: 2025- Present
        Bullets:
          - Delivered $32M incremental revenue in one quarter via funnel optimization
          - Ran 300+ experiments

focus_team:
  name: "Product Management"

knobs: {{ brevity: 6, specificity: 8, warmth: 5, idiom_density: 2, formality: 5, jitter: 4 }}
artifacts: {{ emit_pdfs: true, emit_markdown_digest: true, include_raw_json: true }}
```

## Degraded Mode
Continue with user-supplied text if web research is limited. Never fabricate details. Unknowns remain blank and labeled.
