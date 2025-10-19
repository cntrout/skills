# brand-identity-extractor — v1.6

**Purpose**  
Extract a complete brand identity from a company website and related brand/press hubs, even when the main site is bot-protected (by accepting user-provided HTML/CSS/HAR/ZIP). Produces a JSON report, an optional **Voicebook HTML**, and an optional **Assets ZIP** of logos/press-kit files.

## Quick Start
```yaml
url: "https://example.com"
discover_brand_sites: true
include_voicebook: true
include_assets_zip: true
output_format: "json"
```

If the site is protected, paste **page source** into `page_html`, provide CSS via `css_overrides`, or attach a **HAR/ZIP** when prompted by the skill.

## Outputs
- `identity` (voice, tone, traits, pillars with proof, taglines, elevator, preferred/avoid, glossary, rules)
- `examples` with before/after rewrites
- `brand_assets` manifest (logos/icons/presskit docs)
- `artifacts.voicebook` (HTML), `artifacts.assets_zip` (ZIP)
- `attempts_log` and `limits` for transparency

## Examples
- `examples/quick.yaml` — defaults on
- `examples/overrides.yaml` — paste page HTML + CSS
- `examples/brand-first.yaml` — try brand hub first
