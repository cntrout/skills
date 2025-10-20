# apply-to-job-posting — v1.0

End-to-end application pipeline:
- Ingest a job posting (URL + pasted text)
- Call job-research
- Call linkedin-outreach
- Call cover-letter
- Return a consolidated digest and optional raw JSON

## Install
Make sure these skills exist in your workspace (or update names to match yours):
- job-research
- linkedin-outreach (verified-profile mode)
- cover-letter (v1.1+)
- natural-tone

## Quick Start
Use the example in `SKILL.md` or `examples/example-input.yaml`. Supply `candidate.user_profile_struct` or `.user_profile_markdown`. Do not fetch gated pages.

## Outputs
- `results.job_research`, `results.linkedin_outreach`, `results.cover_letters`
- `artifacts.digest_markdown` for quick review
- `provenance.pipeline` with step statuses
