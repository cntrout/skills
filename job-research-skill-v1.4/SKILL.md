---
name: "job-research"
description: "Deep-dive job application research: parse a job post, map the company, and identify the most relevant public points of contact for outreach—ethically and reproducibly."
---
# Job Application Research (v1.4)

_Last updated: 2025-10-20Z_

## Purpose
Given a job posting URL (or pasted posting content), produce a **structured research report** that:
1) understands the **role** and **requirements**,  
2) maps the **company** (mission, products, culture, recent news), and  
3) identifies **public** points of contact most relevant to the posting, with rationale and ranked outreach order,  
4) **captures the original job posting** as an artifact (URL, platform, posted date, and full body when available).

> This skill uses only **publicly available** sources. It respects robots.txt, avoids login-gated pages, and will not guess private emails.

---

## System Instructions (for the model)

You are a precise research analyst. Follow the protocol deterministically and fill the output schema exactly. When data is missing, record nulls and add an entry to `limits` and `attempts_log`. Use **public** information only. Do not bypass bot protections or paywalls.

### Priority of Sources
1. The **job posting** itself (URL or pasted content).
2. The company’s public website: About, Leadership/Team, Careers, Products/Solutions, Culture/Life at, Newsroom/Press, Investor Relations, Blog.
3. Public company profiles (e.g., LinkedIn company page URL, Crunchbase company page URL) when accessible without login.
4. Reputable media coverage and the company’s own press/news posts.

### Contact Discovery Targets (ranked relevance)
- **Hiring Manager (likely supervisor)**
- **Recruiter / Talent Acquisition tied to this role/team**
- **Department Head / Skip-level leader** (e.g., Dir/VP of the function)
- **Adjacent ICs** (senior peers in the team for authentic networking)
- **People Ops leadership** (only if role or posting indicates their involvement)

For each target, try multiple search patterns. Confirm affiliation by cross-checking titles across at least two public sources when possible. If you cannot access LinkedIn, try press pages, leadership/team pages, speaker bios, blog bylines, or engineering/design handbooks. Log failures.

### Ethics & Privacy
- Do not collect non-public personal data.
- If a public press kit documents an email pattern (e.g., "firstname.lastname@company.com"), you may record the pattern only (not specific private addresses).
- Never fabricate contacts. Prefer “Not found (reason)” over speculation.

---

## Inputs Schema (YAML)

```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "Job Research — Input"
type: object
additionalProperties: false
required: ["job_posting_url"]
properties:
  job_posting_url: { type: string, format: uri }
  job_posting_html:
    type: ["string","null"]
    description: "Optional: paste full HTML if the job board blocks automated fetch."
    default: null
  role_hint:
    type: ["string","null"]
    description: "Optional role/level disambiguation (e.g., 'Senior Product Manager, Growth')."
    default: null
  company_hint:
    type: ["string","null"]
    description: "Optional company name if the posting obfuscates it."
    default: null
  location_hint:
    type: ["string","null"]
    description: "Optional: city/region to disambiguate multi-site companies."
    default: null
  respect_robots: { type: boolean, default: true }
  render_js: { type: boolean, default: false }
  timeout_seconds: { type: integer, minimum: 5, maximum: 120, default: 45 }
  headers:
    type: object
    additionalProperties: { type: string }
    default:
      User-Agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
      Accept-Language: "en-US,en;q=0.9"
  include_artifacts:
    type: object
    additionalProperties: false
    default:
      outreach_plan: true
      contacts_csv: true
      company_snapshot: true
      original_job_posting: true
    properties:
      outreach_plan: { type: boolean }
      contacts_csv: { type: boolean }
      company_snapshot: { type: boolean }
      original_job_posting: { type: boolean }
```

---

## Output Schema (YAML)

```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "Job Research — Output"
type: object
additionalProperties: false
required: ["job","company","contacts","generated_at"]
properties:
  job:
    type: object
    required: ["url"]
    properties:
      url: { type: string, format: uri }
      title: { type: ["string","null"] }
      department: { type: ["string","null"] }
      location: { type: ["string","null"] }
      description_excerpt: { type: ["string","null"] }
      key_requirements: { type: array, items: { type: string }, default: [] }
      reporting_line_hint: { type: ["string","null"], description: "If the posting mentions who it reports to." }
      sources: { type: array, items: { type: string }, default: [] }
  company:
    type: object
    properties:
      name: { type: ["string","null"] }
      website: { type: ["string","null"], format: "uri" }
      industry: { type: ["string","null"] }
      size_hint: { type: ["string","null"] }
      hq_hint: { type: ["string","null"] }
      products: { type: array, items: { type: string }, default: [] }
      mission_values: { type: array, items: { type: string }, default: [] }
      culture_signals: { type: array, items: { type: string }, default: [] }
      recent_news: { type: array, items: { type: string }, default: [] }
      key_pages:
        type: object
        additionalProperties: false
        properties:
          about: { type: ["string","null"] }
          leadership: { type: ["string","null"] }
          careers: { type: ["string","null"] }
          culture: { type: ["string","null"] }
          newsroom: { type: ["string","null"] }
          investor: { type: ["string","null"] }
          products: { type: ["string","null"] }
          blog: { type: ["string","null"] }
      public_profiles:
        type: object
        additionalProperties: false
        properties:
          linkedin_company: { type: ["string","null"] }
          crunchbase: { type: ["string","null"] }
      sources: { type: array, items: { type: string }, default: [] }
  contacts:
    type: array
    default: []
    items:
      type: object
      required: ["name","title"]
      properties:
        name: { type: string }
        title: { type: ["string","null"] }
        department: { type: ["string","null"] }
        relation: { type: string, enum: ["hiring_manager","recruiter","ta","dept_head","peer","people_ops","other"] }
        linkedin_url: { type: ["string","null"] }
        company_page_url: { type: ["string","null"] }
        notes: { type: array, items: { type: string }, default: [] }
        priority: { type: integer, minimum: 1, description: "1 = highest priority" }
        sources: { type: array, items: { type: string }, default: [] }
  outreach_order_rationale:
    type: array
    items: { type: string }
    default: []
  email_pattern_hint:
    type: ["string","null"]
    description: "Record only if a public press/media kit explicitly lists a pattern (e.g., 'firstname.lastname@'). No private emails."
  attempts_log:
    type: array
    items:
      type: object
      properties:
        step: { type: string }
        target: { type: string }
        status: { type: ["integer","null"] }
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
  artifacts:
    type: object
    additionalProperties: false
    properties:
      outreach_plan:
        type: object
        required: ["filename","markdown"]
        properties:
          filename: { type: string }
          markdown: { type: string }
      contacts_csv:
        type: object
        required: ["filename","bytes_b64"]
        properties:
          filename: { type: string }
          bytes_b64: { type: string }
      company_snapshot:
        type: object
        required: ["filename","markdown","sources"]
        properties:
          filename: { type: string }
          markdown: { type: string, description: "Include inline reference links for each section." }
          sources:
            type: object
            additionalProperties: false
            properties:
              mission_values: { type: array, items: { type: string }, default: [] }
              products: { type: array, items: { type: string }, default: [] }
              culture_signals: { type: array, items: { type: string }, default: [] }
              recent_news: { type: array, items: { type: string }, default: [] }
              key_pages: { type: array, items: { type: string }, default: [] }
              public_profiles: { type: array, items: { type: string }, default: [] }
      original_job_posting:
        type: object
        required: ["filename","markdown"]
        properties:
          filename: { type: string }
          markdown: { type: string, description: "Start with URL, Platform, Posted Date; then include full job body if available. Leave blank fields if unknown." }
  generated_at:
    type: string
    format: date-time
```

---

## Tools (YAML)

```yaml
- name: web_fetch
  description: Fetches public web pages with basic headers; obeys robots.txt if requested.
  input_schema:
    type: object
    required: ["url"]
    properties:
      url: { type: string, format: uri }
      respect_robots: { type: boolean }
      render_js: { type: boolean }
      timeout_seconds: { type: integer, minimum: 5, maximum: 120 }
      headers: { type: object, additionalProperties: { type: string } }
  output_schema:
    type: object
    required: ["url","http_status","html"]
    properties:
      url: { type: string }
      http_status: { type: ["integer","null"] }
      html: { type: ["string","null"] }
      text_preview: { type: ["string","null"] }
- name: ingest_asset
  description: Accepts pasted job posting HTML when job boards block automated fetch.
  input_schema:
    type: object
    required: ["kind","payload"]
    properties:
      kind: { type: string, enum: ["job_posting_html"] }
      payload: { type: string, description: "Raw HTML of the job posting." }
  output_schema:
    type: object
    required: ["status"]
    properties:
      status: { type: string, enum: ["ok","error"] }
      note: { type: ["string","null"] }
```

---

## Execution Flow

1) **Ingest Job Posting**
   - If `job_posting_html` provided, parse it first. Otherwise `web_fetch(job_posting_url)`.
   - Extract: title, department/team, location, requirements, any “reports to” hints, application process details, links.
   - Capture **Original Job Posting** artifact with:
     - URL (from `job_posting_url` or discovered canonical URL)
     - Platform (e.g., company site, Greenhouse, Lever, Workday, LinkedIn Jobs). Leave blank if unknown.
     - Posted date (from page metadata or visible text). Leave blank if unknown.
     - Full body text (verbatim where accessible; otherwise leave blank and log the reason in `limits`).

2) **Establish Company Identity**
   - From posting, derive likely official company name.
   - Find the **official website** (verify name/logo/industry match).
   - Build key page list: About, Leadership/Team, Careers, Culture/Life at, Products/Solutions, Newsroom/Press, Investor, Blog.
   - For each page found, `web_fetch` and extract concise bullets (mission/values, product capsules, culture signals, recent announcements). **Embed source links inline in the `company_snapshot` Markdown and populate `company_snapshot.sources` for each section.**

3) **Public Profiles**
   - Locate LinkedIn company page URL and Crunchbase company page URL if discoverable without login.
   - Do not rely on gated content. If blocked, write a brief note in `limits`.

4) **Contact Discovery (Ranked Targets)**
   - Hiring Manager (likely supervisor): search for function + team + company; cross-check with team pages, press/bylines.
   - Recruiter / TA for the function: look for role-specific recruiting posts; check Careers/Team pages.
   - Department Head / Skip-level: VP/Director of the function.
   - Adjacent ICs: senior peers on the product/discipline (useful for warm intros).
   - People Ops leadership only if posting indicates their involvement.
   - For each candidate contact, record name, title, URLs, relation, priority, notes, sources. Deduplicate and rank.

5) **Email Pattern Hint (Optional)**
   - If a public press/media kit lists an org-wide pattern, store the pattern (e.g., "first.last@company.com") in `email_pattern_hint`. Do not list private addresses.

6) **Assemble Outputs**
   - Populate `job`, `company`, `contacts` (ranked), and `outreach_order_rationale`.
   - Add `attempts_log` entries for fetches, and `limits` entries for any blocked/missing areas.

7) **Artifacts (if enabled)**
   - `outreach_plan`: Markdown with a 1–3 contact sequence, personalization hooks per person, and two short outreach examples (email + LinkedIn note).
   - `contacts_csv`: name, title, relation, URLs, priority, notes.
   - `company_snapshot`: one-pager Markdown with **inline source links in each section** and the `sources` map populated.
   - `original_job_posting`: Markdown with **URL**, **Platform**, **Posted Date**, and the **full posting body** if accessible; blank fields if unknown.
