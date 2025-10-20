# job-research — v1.3

Deep-dive job application research. Parse a job post, map the company, and identify the most relevant **public** points of contact with rationale and a suggested outreach order. Optional artifacts: **Outreach Plan** (Markdown), **Contacts CSV**, and a **Company Snapshot** (one-pager Markdown).

## Quick Start
```yaml
job_posting_url: "https://jobs.example.com/openings/senior-product-manager-1234"
include_artifacts:
  outreach_plan: true
  contacts_csv: true
  company_snapshot: true
```

### Blocked board recovery
If the job board blocks automated fetch, paste the full **job HTML**:
```yaml
job_posting_html: "<!doctype html>...PASTE FULL JOB HTML..."
```

## Outputs
- `job`: title, department, location, key requirements, reporting line hint
- `company`: website, industry/size, mission/values, products, culture signals, recent news, key pages, public profiles
- `contacts`: ranked list with relation (hiring_manager, recruiter/ta, dept_head, peer, people_ops), URLs, notes, priority
- `outreach_order_rationale`, `email_pattern_hint` (pattern only, if publicly listed)
- `attempts_log` and `limits` for transparency
- **Artifacts**: outreach plan (md), contacts CSV (base64), company snapshot (md)

## Examples
- `examples/basic.yaml` — crawl public pages
- `examples/blocked.yaml` — paste job HTML
- `examples/hints.yaml` — provide role/company/location hints
