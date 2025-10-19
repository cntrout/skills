---
name: "design-system-extractor"
description: "Extracts design tokens, components, effects, and brand assets from HTML/CSS and user-provided files. v2.5 adds Brand Site Discovery—automatic search for brand/design‑system hubs (e.g., brand.example.com, /brand, /brand-guidelines) and merges their tokens, CSS, and assets with provenance."
---
# Design System & Brand Guidelines Extractor (v2.5)

_Last updated: 2025-10-19Z_

> **What’s new in v2.5**  
> - **Brand Site Discovery**: Automatically probes common brand/design‑system locations and link trails to find official guidelines (e.g., `https://brand.<root>/`, `/<brand|press|media|design-system|styleguide>`).  
> - **Priority merge**: Treats discovered brand hubs as **authoritative** sources; merges tokens/assets with provenance and conflict reporting.  
> - Works with v2.4 features (assets ZIP + Kitchen Sink Brand Assets).

## System Prompt

```
You are a deterministic design-system extractor and reporter.

Sources (priority order):
1) Discovered **brand/design-system hubs** (v2.5) and their assets.
2) Tool-fetched site HTML/CSS when accessible.
3) Public-CDN assets.
4) User-provided overrides (HAR/ZIP/CSS/HTML/URL list).
5) Optional screenshots (for palette sampling only, origin=image_evidence).

Never speculate; set unknown fields to null; record reasons in `limits` and a chronological `attempts_log`. Sort deterministically.

### Brand Site Discovery (v2.5)
Goal: find the site's official brand/design-system guidelines and leverage their CSS/tokens/assets.

**Heuristics (run before standard crawl):**
- Build candidate URLs from the root domain:
  - Subdomains: `brand.<root>`, `design.<root>`, `style.<root>`, `identity.<root>`, `press.<root>`, `media.<root>`, `assets.<root>`
  - Paths on main host: `/brand`, `/brand/`, `/brand-guidelines`, `/brand-guidelines/`, `/design`, `/design-system`, `/style`, `/styleguide`, `/identity`, `/press`, `/press-kit`, `/media`, `/media-kit`
- From the homepage HTML, follow internal <a> that contain tokens: `brand`, `guidelines`, `press`, `media`, `design system`, `style guide`, `logo`, `download`, `assets`.
- Try `/<robots.txt>` to locate `Sitemap:` lines; fetch sitemap and look for above keywords.
- If a Web App Manifest is present, check for `icons` and related branding.

**Constraints:**
- Respect `respect_robots`. Do not bypass access controls.
- Only visit hosts within the root or those listed in `public_asset_domains`.
- Limit discovery to `brand_discovery.max_pages` total and stop after first definitive hub (HTTP 200 + presence of brand keywords or logo downloads).

**If a brand hub is found:**
- Crawl it (bounded) and extract CSS, variables, logo/icon downloads, and any “press kit / media kit” assets.
- Mark tokens/assets from this source with `source: "brand_hub"` and keep full provenance (URL).
- On conflicts between main-site CSS and brand-hub CSS, **prefer brand_hub** values but record an `inconsistencies` item listing both values and sources.

**Merging & Provenance:**
- For each token type, keep `{value, sources[]}`. Preserve all unique sources; first element is the chosen canonical (brand_hub if present).
- For assets, keep all distinct SVGs and highest-resolution rasters. Prefer brand_hub logos as canonical variants.

### Existing responsibilities
- Colors, typography, spacing, layout, effects, components.
- WCAG contrast where pairs are explicit.
- Assets ZIP packaging and Brand Assets section in the Kitchen Sink.
- Deterministic JSON output (and optional artifacts).

Finally, emit EXACTLY the output schema. Do not invent token values.
```

## Inputs Schema (YAML)

```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
title: "Design System Extractor — Input"
type: object
additionalProperties: false
required: ["url"]
properties:
  url: { type: string, format: uri }
  seed_urls: { type: array, default: [], items: { type: string, format: uri } }
  public_asset_domains: { type: array, default: [], items: { type: string } }
  max_pages: { type: integer, minimum: 1, maximum: 100, default: 6 }
  follow_subdomains: { type: boolean, default: false }
  include_paths: { type: array, items: { type: string }, default: ["^/$", "^/pricing$", "^/about$"] }
  exclude_paths: { type: array, items: { type: string }, default: ["\\.(png|jpg|svg|ico|gif|webp)$"] }
  respect_robots: { type: boolean, default: true }
  render_js: { type: boolean, default: false }
  timeout_seconds: { type: integer, minimum: 5, maximum: 120, default: 45 }
  infer_frameworks: { type: boolean, default: true }
  headers:
    type: object
    additionalProperties: { type: string }
    default:
      User-Agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
      Accept: "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"
      Accept-Language: "en-US,en;q=0.9"
  cookies: { type: object, additionalProperties: { type: string }, default: {} }
  page_html: { type: ["string","null"], default: null }
  css_overrides:
    type: array
    default: []
    items:
      anyOf:
        - { type: string, format: uri }
        - { type: string }
  screenshot_path: { type: ["string","null"], default: null }
  output_format: { type: string, enum: ["json","markdown","both"], default: "json" }
  include_kitchen_sink: { type: boolean, default: false }
  kitchen_sink_options:
    type: object
    additionalProperties: false
    properties:
      filename: { type: string, default: "design-system-kitchen-sink.html" }
      brand_name: { type: ["string","null"] }
      show_components: { type: boolean, default: true }
      show_accessibility: { type: boolean, default: true }
      max_colors: { type: integer, minimum: 1, maximum: 200, default: 64 }
      max_type_steps: { type: integer, minimum: 1, maximum: 40, default: 12 }
      max_assets_preview: { type: integer, minimum: 1, maximum: 120, default: 40 }

  # NEW: Brand Site Discovery controls
  discover_brand_sites: { type: boolean, default: true }
  brand_discovery:
    type: object
    additionalProperties: false
    default:
      max_pages: 10
      candidate_subdomains: ["brand","design","style","identity","press","media","assets"]
      candidate_paths: ["/brand","/brand/","/brand-guidelines","/brand-guidelines/","/design","/design-system","/style","/styleguide","/identity","/press","/press-kit","/media","/media-kit"]
      anchor_keywords: ["brand","guidelines","press","media","design system","style guide","logo","download","assets"]
      sitemap_enabled: true
    properties:
      max_pages: { type: integer, minimum: 1, maximum: 40 }
      candidate_subdomains: { type: array, items: { type: string } }
      candidate_paths: { type: array, items: { type: string } }
      anchor_keywords: { type: array, items: { type: string } }
      sitemap_enabled: { type: boolean }

  include_assets_zip: { type: boolean, default: true }
  assets_filters:
    type: object
    additionalProperties: false
    default:
      include_exts: ["svg","png","ico","webp"]
      include_keywords: ["logo","brandmark","wordmark","icon","favicon","apple-touch-icon","mask-icon","sprite"]
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
title: "Design System Extractor — Output"
type: object
additionalProperties: false
required: ["site", "generated_at"]
properties:
  site:
    type: object
    additionalProperties: false
    required: ["url"]
    properties:
      name: { type: ["string","null"] }
      url: { type: string, format: uri }
  frameworks:
    type: array
    default: []
    items:
      type: object
      required: ["name","evidence"]
      properties:
        name: { type: string }
        evidence: { type: array, items: { type: string } }
  colors:
    type: array
    default: []
    items:
      type: object
      required: ["name","value","sources"]
      properties:
        name: { type: string }
        value: { type: ["string","null"], pattern: "^#([A-Fa-f0-9]{6})$" }
        origin: { type: ["string","null"], enum: ["css","html","image_evidence", null] }
        formats:
          type: object
          additionalProperties: false
          properties:
            hex: { type: string }
            rgb: { type: ["string","null"] }
            hsl: { type: ["string","null"] }
        usage_examples:
          type: array
          default: []
          items:
            type: object
            required: ["selector","property"]
            properties:
              selector: { type: string }
              property: { type: string }
        contrast:
          type: array
          default: []
          items:
            type: object
            required: ["against","ratio","wcag"]
            properties:
              against: { type: string, pattern: "^#([A-Fa-f0-9]{6})$" }
              ratio: { type: number }
              wcag: { type: string, enum: ["AA","AAA","fail"] }
        sources: { type: array, items: { type: string } }
  typography:
    type: object
    additionalProperties: false
    properties:
      families:
        type: array
        default: []
        items:
          type: object
          required: ["stack","sources"]
          properties:
            stack: { type: ["string","null"] }
            sources: { type: array, items: { type: string } }
      scale:
        type: array
        default: []
        items:
          type: object
          required: ["token","sources"]
          properties:
            token: { type: string }
            size_px: { type: ["number","null"] }
            weight: { type: ["integer","null"] }
            line_height: { type: ["number","null"] }
            sources: { type: array, items: { type: string } }
  spacing:
    type: object
    required: ["evidence"]
    properties:
      base_px: { type: ["number","null"] }
      scale_px: { type: array, items: { type: number } }
      evidence: { type: array, items: { type: string } }
  layout:
    type: object
    properties:
      container_widths: { type: array, items: { type: number } }
      breakpoints_px: { type: array, items: { type: number } }
      sources: { type: array, items: { type: string } }
  components:
    type: array
    default: []
    items:
      type: object
      required: ["type","selectors","sources"]
      properties:
        type: { type: string }
        variant: { type: ["string","null"] }
        props: { type: object }
        selectors: { type: array, items: { type: string } }
        sources: { type: array, items: { type: string } }
  effects:
    type: object
    properties:
      radius: { type: array, items: { type: number } }
      shadows: { type: array, items: { type: string } }
      transitions: { type: array, items: { type: string } }
      sources: { type: array, items: { type: string } }
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
            kind: { type: string, enum: ["logo","icon","favicon","mask","appicon","sprite","unknown"] }
            format: { type: string, enum: ["svg","png","ico","webp","other"] }
            normalized_filename: { type: ["string","null"] }
            origin_url: { type: ["string","null"] }
            source: { type: string, enum: ["brand_hub","html","css","manifest","har","zip","override"] }
            width: { type: ["integer","null"] }
            height:{ type: ["integer","null"] }
            density: { type: ["string","null"], enum: ["1x","1.5x","2x","3x","4x", null] }
            media: { type: ["string","null"] }
            bytes: { type: ["integer","null"] }
            sha256: { type: ["string","null"] }
      notes: { type: array, items: { type: string }, default: [] }
  inferences:
    type: object
    properties:
      aesthetic: { type: array, items: { type: string } }
      principles: { type: array, items: { type: string } }
      confidence: { type: number, minimum: 0, maximum: 1 }
  inconsistencies:
    type: array
    default: []
    items:
      type: object
      required: ["issue"]
      properties:
        issue: { type: string }
        values: { type: array, items: { type: string } }
        locations: { type: array, items: { type: string } }
  limits:
    type: array
    default: []
    items:
      type: object
      required: ["type"]
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
  errors:
    type: array
    default: []
    items:
      type: object
      properties:
        path: { type: ["string","null"] }
        reason: { type: string }
  artifacts:
    type: object
    properties:
      kitchen_sink:
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
  description: Fetches HTML and metadata with optional headers/cookies.
  input_schema:
    type: object
    required: ["url"]
    properties:
      url: { type: string, format: uri }
      max_pages: { type: integer, minimum: 1, maximum: 100 }
      follow_subdomains: { type: boolean }
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
        text_preview: { type: ["string","null"] }
        headers: { type: object }
        http_status: { type: ["integer","null"] }
- name: get_asset
  description: Fetches raw text or binary assets (CSS/JS/images) by URL or path with optional headers/cookies.
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
  description: Accepts user-provided files or raw strings when crawling is blocked (HAR/ZIP/CSS/HTML/Image/URL list).
  input_schema:
    type: object
    required: ["kind","payload"]
    properties:
      kind: { type: string, enum: ["har","zip","css_text","html_text","image","url_list"] }
      payload: { type: string, description: "Base64-encoded file content, raw text, or newline-delimited URLs" }
      filename: { type: ["string","null"] }
  output_schema:
    type: object
    required: ["status"]
    properties:
      status: { type: string, enum: ["ok","error"] }
      note: { type: ["string","null"] }
```

## Execution Flow

### Artifacts Emission Rules (v2.5.2)
- If `include_kitchen_sink` is **true**, always emit `artifacts.kitchen_sink` with:
  - `filename`: from `kitchen_sink_options.filename` (default "design-system-kitchen-sink.html")
  - `html`: a self-contained HTML string, even when no tokens are discovered (render an empty state with headings).
  - `summary`: short text summary (optional).
- If `include_assets_zip` is **true`, always emit `artifacts.assets_zip`:
  - When no qualifying assets are found, emit a ZIP with **0 files** (valid empty archive) and set `file_count: 0`, `total_bytes` accordingly.
  - When assets exceed limits, include the top-priority subset and log truncation in `limits`.


1. **Brand Site Discovery** (if `discover_brand_sites`): probe candidate subdomains/paths, homepage anchors, optional sitemap; stop on first definitive brand hub. Log findings.
2. Fetch HTML via `web_fetch` for discovered brand hub (bounded by `brand_discovery.max_pages`); extract CSS/assets first.
3. Standard crawl of main site (bounded).
4. Fetch CSS via `get_asset`; resolve `@import`.
5. Discover & package **brand assets** (logos/icons) from all sources; dedupe; build Assets ZIP if enabled.
6. Apply user overrides via `ingest_asset`, `css_overrides`, `page_html`.
7. Parse & normalize tokens (colors, typography, spacing, layout, effects, components).
8. Merge with **brand_hub** precedence; record conflicts in `inconsistencies`.
9. Compute WCAG contrast where pairs are explicit.
10. Assemble JSON; log steps in `attempts_log`; fill `limits` if truncated/blocked.
11. **Kitchen Sink**: if requested, render tokens + **Brand Assets** previews.
12. Emit artifacts (Kitchen Sink HTML and Assets ZIP) per inputs.