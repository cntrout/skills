# job-research — v1.4

Deep-dive job application research. Parse a job post, map the company (with per-section source links), identify the most relevant **public** points of contact, and capture the **Original Job Posting** as an artifact (URL, platform, posted date, full body when available). Optional artifacts: **Outreach Plan** (Markdown) and **Contacts CSV**.

## Quick Start
```yaml
job_posting_url: "https://jobs.example.com/openings/senior-product-manager-1234"
include_artifacts:
  outreach_plan: true
  contacts_csv: true
  company_snapshot: true
  original_job_posting: true
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
- **Artifacts**:
  - `company_snapshot` (md): includes **inline source links per section** and a sources map
  - `original_job_posting` (md): **URL**, **Platform**, **Posted Date**, and **full body** if accessible
  - `outreach_plan` (md) and `contacts_csv` (base64)
