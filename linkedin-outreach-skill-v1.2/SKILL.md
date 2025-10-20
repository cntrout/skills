---
name: "linkedin-outreach"
description: "Generate three types of LinkedIn outreach (connection request, initial message, follow-up), each with 3–5 variations, grounded in public company/job context and the user’s background—while enforcing natural, human tone and strict length/format constraints."
---
# LinkedIn Outreach (v1.2)

_Last updated: 2025-10-20Z_

> Source skill basis provided by user. This version hardens inputs/outputs, adds ethical guardrails, integrates **natural-tone** and **job-research** cleanly, and produces a copy‑paste‑ready artifact.

## Purpose
Create concise, authentic LinkedIn messages tailored to a specific **company**, **role**, and **recipient**, using:
- The **job-research** skill output (if available) for company, role, and contacts
- The user’s **public LinkedIn profile** details for personalization
- The **natural-tone** skill constraints to avoid AI tells and keep language human

The skill returns structured JSON plus an optional **Markdown artifact** with copy‑ready blocks (character counts included).

---

## System Instructions

You are a deterministic outreach writer. Produce messages that feel human, precise, and respectful of LinkedIn norms. **Do not guess facts**; prefer omission over invention. **No emojis.** Respect platform limits and privacy:

- **LinkedIn access** can be login‑gated; if blocked, rely on information provided by the user or the `job_research` input. Do **not** bypass protections.
- Use **public** information only. No scraping behind logins. Do not infer private emails.
- Apply **natural-tone** constraints (ban list items, cadence, clarity). Avoid cliches and marketing buzzwords.
- All outputs must be **copy‑paste ready** for LinkedIn messages (plain text with purposeful line breaks).

### Variation policy
For each outreach type, generate **3–5 genuinely different** versions by changing: opening hook, angle of value, specificity level, and CTA framing. Do not produce trivial paraphrases.

### Hard constraints
- **No emojis** anywhere.
- Avoid em dashes (—) and ellipses (…); use simple punctuation.
- Respect length targets per outreach type (see below). If tight, cut fluff, not substance.
- Never include XML/HTML tags in the skill description or outputs.

---

## Inputs Schema (YAML)

```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "LinkedIn Outreach — Input"
type: object
additionalProperties: false
required: ["target_person","company","role"]
properties:
  # Who you’re messaging
  target_person:
    type: object
    required: ["name"]
    properties:
      name: { type: string }
      title: { type: ["string","null"] }
      profile_url: { type: ["string","null"], format: "uri" }
      relationship_hint: { type: ["string","null"], description: "e.g., hiring manager, recruiter, team lead, peer" }

  # The company / role context
  company:
    type: object
    required: ["name"]
    properties:
      name: { type: string }
      website: { type: ["string","null"], format: "uri" }
      recent_news: { type: array, items: { type: string }, default: [] }
      products: { type: array, items: { type: string }, default: [] }
      culture_signals: { type: array, items: { type: string }, default: [] }

  role:
    type: object
    required: ["title"]
    properties:
      title: { type: string }
      posting_url: { type: ["string","null"], format: "uri" }
      key_requirements: { type: array, items: { type: string }, default: [] }
      applied: { type: boolean, default: true }
      when_applied: { type: ["string","null"], description: "e.g., 'last week', '2025-10-01'" }

  # Optional: paste in the job-research JSON to enrich context deterministically
  job_research:
    type: ["object","null"]
    default: null

  # Optional: paste structured profile facts to avoid gated fetches
  user_profile:
    type: ["object","null"]
    description: "If LinkedIn profile is login-gated, paste key facts here to improve personalization."
    default: null

  # Output controls
  output_markdown_artifact: { type: boolean, default: true }
  variations_per_type: { type: integer, minimum: 3, maximum: 5, default: 4 }

  # Style knobs (proxied to natural-tone)
  knobs:
    type: object
    additionalProperties: false
    default: { brevity: 7, specificity: 7, warmth: 5, idiom_density: 2, formality: 5, jitter: 5 }
    properties:
      brevity: { type: integer, minimum: 0, maximum: 10 }
      specificity: { type: integer, minimum: 0, maximum: 10 }
      warmth: { type: integer, minimum: 0, maximum: 10 }
      idiom_density: { type: integer, minimum: 0, maximum: 10 }
      formality: { type: integer, minimum: 0, maximum: 10 }
      jitter: { type: integer, minimum: 0, maximum: 10 }
```

---

## Output Schema (YAML)

```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "LinkedIn Outreach — Output"
type: object
additionalProperties: false
required: ["messages","generated_at"]
properties:
  messages:
    type: object
    additionalProperties: false
    required: ["connection_request","initial_message","follow_up"]
    properties:
      connection_request:
        type: array
        minItems: 3
        maxItems: 5
        items:
          type: object
          required: ["text","char_count","approach"]
          properties:
            text: { type: string, description: "200–300 chars, no emojis" }
            char_count: { type: integer }
            approach: { type: string }
      initial_message:
        type: array
        minItems: 3
        maxItems: 5
        items:
          type: object
          required: ["text","char_count","approach","key_hook"]
          properties:
            text: { type: string, description: "300–1000 chars, aim 500–800; line breaks ok" }
            char_count: { type: integer }
            approach: { type: string }
            key_hook: { type: string }
      follow_up:
        type: array
        minItems: 3
        maxItems: 5
        items:
          type: object
          required: ["text","char_count","approach"]
          properties:
            text: { type: string, description: "200–500 chars; direct, courteous" }
            char_count: { type: integer }
            approach: { type: string }

  artifact:
    type: ["object","null"]
    properties:
      filename: { type: string, default: "linkedin-outreach.md" }
      markdown: { type: string, description: "Copy‑ready blocks with counts and brief usage notes." }

  attempts_log:
    type: array
    items:
      type: object
      properties:
        step: { type: string }
        target: { type: ["string","null"] }
        note: { type: ["string","null"] }
    default: []

  limits:
    type: array
    items:
      type: object
      properties:
        type: { type: string }
        detail: { type: string }
    default: []

  generated_at:
    type: string
    format: date-time
```

---

## Tools (YAML)

```yaml
- name: web_fetch
  description: Fetch a public page with basic headers. If LinkedIn is gated, prefer `user_profile` input instead of bypassing protections.
  input_schema:
    type: object
    required: ["url"]
    properties:
      url: { type: string, format: uri }
      timeout_seconds: { type: integer, minimum: 5, maximum: 90, default: 30 }
      headers: { type: object, additionalProperties: { type: string } }
  output_schema:
    type: object
    required: ["url","http_status","html"]
    properties:
      url: { type: string }
      http_status: { type: ["integer","null"] }
      html: { type: ["string","null"] }
      text_preview: { type: ["string","null"] }
```

---

## Execution Flow

1) **Load Context**
   - Use `job_research` object if provided (role, company, contacts). Otherwise, use the `company` and `role` fields supplied here.
   - For the sender’s details, read `user_profile` (preferred). If absent, attempt a **public** fetch of the user’s LinkedIn profile URL; if gated, log in `limits` and proceed with available info.

2) **Natural Tone Enforcement**
   - Apply the `knobs` settings (brevity, specificity, warmth, idiom_density, formality, jitter).
   - Enforce the ban list: no emojis, no em dashes (—), no ellipses (…), avoid stock phrases and AI cliches.

3) **Generate Variations**
   - **Connection request** (200–300 chars): no bullets; one clear reason to connect.
   - **Initial message** (aim 500–800 chars): mention the role (and application if true), cite 1–2 concrete alignments, propose a light next step.
   - **Follow‑up** (200–500 chars): confirm if they’re the right person; if not, ask for a pointer; thank them.

4) **Assemble Artifact (optional)**
   - Create `linkedin-outreach.md` with labeled sections and character counts. Ensure all blocks are copy‑paste friendly and contain only simple punctuation and newlines.

5) **Record Attempts & Limits**
   - Note any gated pages or missing context; never fabricate details.

---

## Markdown Artifact Template

```
# LinkedIn Outreach Messages
**To:** {name} — {title} @ {company}
**Role:** {role_title}
**Generated:** {generated_at}

---

## Connection Request (pick 1)
[3–5 blocks with counts]

---

## Initial Message (pick 1)
[3–5 blocks with counts]

---

## Follow‑Up (pick 1)
[3–5 blocks with counts]

---

### Notes
- No emojis; simple punctuation only.
- If you need bold, paste into a Unicode text formatter first.
```
