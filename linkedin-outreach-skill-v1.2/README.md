# linkedin-outreach — v1.2

Personalized, copy‑ready LinkedIn messages for a target person at a target company, informed by job context and your background. Produces 3–5 variations for each of three outreach types and an optional Markdown artifact.

## Quick Start
```yaml
target_person:
  name: "Jane Smith"
  title: "Head of Product"
  profile_url: "https://www.linkedin.com/in/jane-smith/"
  relationship_hint: "hiring manager"
company:
  name: "Acme Corp"
role:
  title: "Senior Product Manager"
  posting_url: "https://jobs.acme.com/roles/spm-123"
variations_per_type: 4
output_markdown_artifact: true
```

If LinkedIn is gated, paste key facts into `user_profile` and/or the `job_research` JSON from your research skill run.

## Outputs
- `messages.connection_request` — 3–5 variants (200–300 chars)
- `messages.initial_message` — 3–5 variants (aim 500–800 chars)
- `messages.follow_up` — 3–5 variants (200–500 chars)
- Optional `artifact` with all blocks in one file
- `attempts_log` and `limits` for transparency

## Notes
- No emojis; avoid em dashes and ellipses.
- Respect privacy and platform rules (no login bypass).
- Use clear line breaks; keep each paragraph purposeful.
