---
name: "brand-identity-extractor"
description: "Extracts a complete brand identity (voice, tone, personality, pillars, taglines, glossary, dos/donts, examples) from web pages and user-provided context. Designed to work when sites are bot-protected by accepting HTML/CSS/HAR/ZIP overrides, and prioritizes any official brand or media hubs when found."
---
# Brand Identity Extractor (v1.6)

_Last updated: 2025-10-19Z_

> **What this skill does**
> - Crawls or ingests provided material to assemble a consistent **brand identity**:
>   - voice & tone ranges, personality traits, messaging pillars
>   - value props, taglines, elevator statements
>   - writing patterns: sentence length, verbs, specificity, banned clichés
>   - audience glossary, preferred/avoid lists, word list by category
>   - structure & formatting rules (headings, bullets, CTA patterns)
>   - examples: before/after rewrites, “golden paragraph” samples
> - Works with **blocked sites** by using pasted page HTML, CSS, HAR files, ZIP bundles, or URL lists.
> - Searches for any **official brand/press/style pages** and treats them as authoritative.

## System Prompt

```
You are a deterministic brand-identity analyst and reporter.

Sources (priority order):
1) Discovered brand/design/press hubs (e.g., brand subdomain or /brand, /press-kit paths).
2) Tool-fetched site pages when accessible.
3) User-provided overrides (HAR/ZIP/HTML/CSS/URL list).
4) Public-CDN assets (for logos/tagline images if embedded).

Rules:
- Never invent claims. If unknown, set fields to null and add a short reason to `limits`.
- Keep provenance: each extracted item includes one or more source URLs or labels (e.g., "override", "har", "zip").
- Prefer primary/official content over third-party reviewers.
- Normalize language; deduplicate across pages; sort deterministically.
- When multiple styles exist, provide a recommended canonical style and list alternatives with sources.
- Respect robots and do not attempt to bypass bot protections.
```

## Inputs Schema (YAML)

```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "Brand Identity Extractor — Input"
type: object
additionalProperties: false
required: ["url"]
properties:
  url: { type: string, format: uri }
  seed_urls:
    type: array
    default: []
    items: { type: string, format: uri }
  discover_brand_sites: { type: boolean, default: true }
  brand_discovery:
    type: object
    additionalProperties: false
    default:
      max_pages: 10
      candidate_subdomains: ["brand","press","media","style","identity","design","assets"]
      candidate_paths: ["/brand","/brand-guidelines","/press","/press-kit","/media","/media-kit","/styleguide","/voice"]
      anchor_keywords: ["brand","guidelines","press","media","voice","tone","style","logo","download","assets"]
      sitemap_enabled: true
    properties:
      max_pages: { type: integer, minimum: 1, maximum: 40 }
      candidate_subdomains: { type: array, items: { type: string } }
      candidate_paths: { type: array, items: { type: string } }
      anchor_keywords: { type: array, items: { type: string } }
      sitemap_enabled: { type: boolean }
  max_pages: { type: integer, minimum: 1, maximum: 100, default: 8 }
  include_paths: { type: array, items: { type: string }, default: ["^/$","^/about$","^/careers$","^/pricing$"] }
  exclude_paths: { type: array, items: { type: string }, default: ["\\.(png|jpg|svg|ico|gif|webp)$"] }
  respect_robots: { type: boolean, default: true }
  render_js: { type: boolean, default: false }
  timeout_seconds: { type: integer, minimum: 5, maximum: 120, default: 45 }
  headers:
    type: object
    additionalProperties: { type: string }
    default:
      User-Agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
      Accept: "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"
      Accept-Language: "en-US,en;q=0.9"
  page_html: { type: ["string","null"], default: null }
  css_overrides:
    type: array
    default: []
    items:
      anyOf:
        - { type: string, format: uri }
        - { type: string }
  ingest_notes:
    type: ["string","null"]
    description: Free-text notes about brand you want to enforce (owner-provided truths)
    default: null
  output_format: { type: string, enum: ["json","markdown","both"], default: "json" }
  include_voicebook: { type: boolean, default: true }
  voicebook_options:
    type: object
    additionalProperties: false
    properties:
      filename: { type: string, default: "brand-voicebook.html" }
      include_examples: { type: boolean, default: true }
      example_inputs:
        type: array
        default: ["Landing page hero", "Product update email", "Tweet length social post"]
        items: { type: string }
  include_assets_zip: { type: boolean, default: true }
  assets_filters:
    type: object
    additionalProperties: false
    default:
      include_exts: ["svg","png","ico","webp","pdf"]
      include_keywords: ["logo","brandmark","wordmark","icon","favicon","press-kit","media-kit"]
      max_asset_bytes: 2097152
      max_total_bytes: 16777216
      max_files: 120
    properties:
      include_exts: { type: array, items: { type: string } }
      include_keywords: { type: array, items: { type: string } }
      max_asset_bytes: { type: integer, minimum: 10240, maximum: 10485760 }
      max_total_bytes: { type: integer, minimum: 1048576, maximum: 52428800 }
      max_files: { type: integer, minimum: 1, maximum: 500 }
```

## Output Schema (YAML)

```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "Brand Identity Extractor — Output"
type: object
additionalProperties: false
required: ["site","generated_at"]
properties:
  site:
    type: object
    required: ["url"]
    properties:
      name: { type: ["string","null"] }
      url: { type: string, format: uri }
  identity:
    type: object
    properties:
      voice_core: { type: ["string","null"] }
      tone_range: { type: array, items: { type: string }, default: [] }
      personality_traits: { type: array, items: { type: string }, default: [] }
      messaging_pillars:
        type: array
        default: []
        items:
          type: object
          required: ["name"]
          properties:
            name: { type: string }
            proof_points: { type: array, items: { type: string }, default: [] }
            sources: { type: array, items: { type: string }, default: [] }
      taglines: { type: array, items: { type: string }, default: [] }
      elevator: { type: ["string","null"] }
      preferred: { type: array, items: { type: string }, default: [] }
      avoid: { type: array, items: { type: string }, default: [] }
      glossary:
        type: array
        default: []
        items:
          type: object
          required: ["term"]
          properties:
            term: { type: string }
            definition: { type: ["string","null"] }
            sources: { type: array, items: { type: string }, default: [] }
      structure_rules: { type: array, items: { type: string }, default: [] }
      formatting_rules: { type: array, items: { type: string }, default: [] }
      sentence_stats:
        type: object
        properties:
          avg_len: { type: ["number","null"] }
          stdev_len: { type: ["number","null"] }
          opener_diversity: { type: ["number","null"] }
          notes: { type: array, items: { type: string }, default: [] }
  examples:
    type: array
    default: []
    items:
      type: object
      properties:
        scenario: { type: string }
        before: { type: ["string","null"] }
        after: { type: ["string","null"] }
        notes: { type: array, items: { type: string }, default: [] }
        sources: { type: array, items: { type: string }, default: [] }
  brand_assets:
    type: object
    properties:
      assets_manifest:
        type: array
        default: []
        items:
          type: object
          required: ["kind","format","source"]
          properties:
            kind: { type: string, enum: ["logo","icon","favicon","presskit","other"] }
            format: { type: string, enum: ["svg","png","ico","webp","pdf","other"] }
            normalized_filename: { type: ["string","null"] }
            origin_url: { type: ["string","null"] }
            source: { type: string, enum: ["brand_hub","html","css","manifest","har","zip","override"] }
            bytes: { type: ["integer","null"] }
            sha256: { type: ["string","null"] }
  inferences:
    type: object
    properties:
      confidence: { type: number, minimum: 0, maximum: 1 }
      notes: { type: array, items: { type: string }, default: [] }
  inconsistencies:
    type: array
    default: []
    items:
      type: object
      properties:
        issue: { type: string }
        values: { type: array, items: { type: string }, default: [] }
        locations: { type: array, items: { type: string }, default: [] }
  limits:
    type: array
    default: []
    items:
      type: object
      properties:
        type: { type: string }
        detail: { type: string }
  attempts_log:
    type: array
    default: []
    items:
      type: object
      properties:
        step: { type: string }
        target: { type: string }
        status: { type: ["integer","null"] }
        note: { type: ["string","null"] }
  artifacts:
    type: object
    additionalProperties: false
    properties:
      voicebook:
        type: object
        required: ["filename","html"]
        properties:
          filename: { type: string }
          html: { type: string }
          summary: { type: ["string","null"] }
      assets_zip:
        type: object
        required: ["filename","bytes_b64","file_count","total_bytes"]
        properties:
          filename: { type: string }
          bytes_b64: { type: string }
          file_count: { type: integer }
          total_bytes: { type: integer }
  generated_at:
    type: string
    format: date-time
```

## Tools (YAML)

```yaml
- name: web_fetch
  description: Fetches HTML with optional headers/cookies.
  input_schema:
    type: object
    required: ["url"]
    properties:
      url: { type: string, format: uri }
      max_pages: { type: integer, minimum: 1, maximum: 100 }
      include_paths: { type: array, items: { type: string } }
      exclude_paths: { type: array, items: { type: string } }
      respect_robots: { type: boolean }
      render_js: { type: boolean }
      timeout_seconds: { type: integer, minimum: 5, maximum: 120 }
      headers: { type: object, additionalProperties: { type: string } }
      cookies: { type: object, additionalProperties: { type: string } }
  output_schema:
    type: array
    items:
      type: object
      required: ["path","html"]
      properties:
        path: { type: string }
        url: { type: string }
        html: { type: string }
        http_status: { type: ["integer","null"] }
- name: get_asset
  description: Fetches raw assets (CSS/JS/images/docs) by URL or path.
  input_schema:
    type: object
    required: ["path_or_url"]
    properties:
      path_or_url: { type: string }
      headers: { type: object, additionalProperties: { type: string } }
      cookies: { type: object, additionalProperties: { type: string } }
  output_schema:
    type: object
    required: ["content"]
    properties:
      path: { type: string }
      content: { type: string, description: "Binary content must be base64-encoded" }
      content_type: { type: string }
      http_status: { type: ["integer","null"] }
- name: ingest_asset
  description: Accepts user-provided files or raw strings when crawling is blocked (HAR/ZIP/HTML/CSS/Image/URL list). Images/docs are captured for the Assets ZIP.
  input_schema:
    type: object
    required: ["kind","payload"]
    properties:
      kind: { type: string, enum: ["har","zip","html_text","css_text","image","url_list","doc"] }
      payload: { type: string }
      filename: { type: ["string","null"] }
  output_schema:
    type: object
    required: ["status"]
    properties:
      status: { type: string, enum: ["ok","error"] }
      note: { type: ["string","null"] }
```

## Execution Flow

1. Brand Discovery (if enabled): probe candidate subdomains/paths and homepage anchors for authoritative brand/press hubs; respect robots.
2. Crawl or ingest supplied materials (bounded by max_pages). Use `page_html` and `css_overrides` for blocked sites.
3. Extract identity (voice, tone, traits, pillars with proof, taglines, elevator, preferred/avoid, glossary, rules, sentence stats).
4. Collect and package assets (logos/icons/press-kit docs) into an optional ZIP; record provenance.
5. Create examples (before/after) aligned to discovered voice without altering factual claims.
6. Assemble JSON with `attempts_log` and `limits`.
7. Emit artifacts when enabled:
   - `artifacts.voicebook` — self-contained HTML voicebook (render even if fields are empty, with an “empty state” and notes).
   - `artifacts.assets_zip` — valid ZIP; emit empty ZIP if nothing qualifies.
