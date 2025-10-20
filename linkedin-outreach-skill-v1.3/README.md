# linkedin-outreach — v1.3 (verified‑profile)

This version **never invents** details about the sender. It uses only verified profile fields (public LinkedIn page if accessible, or user‑supplied profile content). If insufficient, it writes neutral messages without resume claims and logs the limitation.

## Quick Start (gated LinkedIn — paste your profile)
```yaml
target_person:
  name: "Jane Smith"
  title: "Head of Product"
company:
  name: "Acme Corp"
role:
  title: "Senior Product Manager"
profile_source_mode: "user_supplied_only"
user_profile_struct:
  name: "Your Name"
  headline: "Product leader focused on X"
  positions:
    - company: "Contoso"
      title: "Senior PM"
      dates: "2022–present"
      bullets: ["Shipped A", "Improved B"]
variations_per_type: 4
output_markdown_artifact: true
```

## If your LinkedIn is publicly viewable
```yaml
target_person: { name: "Jane Smith", title: "Head of Product" }
company: { name: "Acme Corp" }
role: { title: "Senior Product Manager" }
profile_source_mode: "auto"
# include target_person.profile_url if public; the assistant will parse only if accessible
```

## Outputs
- `profile_provenance`: { status: parsed_public_profile | user_supplied | insufficient }
- `messages.*[]`: text + char_count + approach + facts_used
- Optional artifact `linkedin-outreach.md`
