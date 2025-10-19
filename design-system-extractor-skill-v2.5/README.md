# design-system-extractor — v2.5 (Brand Site Discovery)

**What’s new in v2.5**
- Auto-discovers brand/design-system hubs (e.g., `brand.<root>`, `/brand`, `/brand-guidelines`) and merges those tokens/assets with precedence.
- Keeps v2.4 features: Assets ZIP + Kitchen Sink Brand Assets.

## Quick Start
```yaml
url: "https://yourdomain.com"
discover_brand_sites: true
include_kitchen_sink: true
include_assets_zip: true
output_format: "json"
```
If the site is protected, add **HAR/ZIP/CSS overrides**; discovery will still evaluate the provided HTML/CSS and any brand paths you include.

## Tuning Brand Discovery
```yaml
brand_discovery:
  max_pages: 10
  candidate_subdomains: ["brand","design","style","identity","press","media","assets"]
  candidate_paths: ["/brand","/brand-guidelines","/design-system","/styleguide","/press","/media-kit"]
  anchor_keywords: ["brand","guidelines","press","media","design system","style guide","logo","download","assets"]
  sitemap_enabled: true
```

## Merge & Provenance
- Tokens from a discovered brand hub are marked `source: "brand_hub"` and chosen as canonical when conflicts exist.
- All values retain `sources[]` and the Kitchen Sink lists the origin for transparency.

## Examples
- `examples/brand-discovery.yaml` — attempts brand hub first
- `examples/assets.yaml` — enable Assets ZIP + Kitchen Sink
- `examples/overrides.yaml` — paste HTML/CSS when blocked

## Notes
- Always respect `robots.txt`. The skill never bypasses bot protections.
- For best results on protected sites, supply a HAR or ZIP and/or paste real CSS into `css_overrides`.
