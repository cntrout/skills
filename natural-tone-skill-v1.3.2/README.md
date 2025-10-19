# natural-tone — Upload Bundle (v1.3)

**Updated:** 2025-10-19Z

This bundle gives you a ready-to-use writing style skill for Claude that makes text sound natural and human—without changing meaning. It also includes step‑by‑step usage and copy‑paste examples for common scenarios.

---

## A) One‑time setup (2 minutes)
1. **Download** this folder as a zip (you likely already did).  
2. In Claude (web): go to **Skills → Create Skill → Upload**, choose `SKILL.md`.  
3. Confirm the preview shows:  
   - `name: "natural-tone"`  
   - description as expected  
4. Click **Create** (or **Save**). The skill is now available in any chat via **Use skill**.

> Tip: You can update the skill later by re‑uploading `SKILL.md` with changes.

---

## B) Run the skill in a chat (step‑by‑step)
1. Open a new chat in Claude.  
2. Paste your **draft text** (or say what you want written).  
3. Click **Use skill** → choose **natural‑tone**.  
4. In the **Inputs** panel, you can **accept defaults** or set options:

**Common inputs (safe defaults in bold):**
- `reading_level`: one of `6th`, `8th`, **`10th`**, `12th`, `college`
- `formality`: `casual`, **`neutral`**, `formal`
- `directness`: `soft`, **`balanced`**, `blunt`
- `max_sentence_len`: integer (8–40), default **24**
- `use_contractions`: **true** / false
- `allow_fragments`: **true** / false
- `variance_level`: `low`, **`medium`**, `high`
- `burstiness` (advanced):  
  ```yaml
  burstiness:
    avg_len_target: 22   # typical average sentence length
    stdev_min: 6         # minimum variation
    min_short: 8         # include short sentences
    min_long: 32         # include some longer ones
  ```
- `use_questions`: **true** / false
- `question_rate`: **1**  # about 1 question per ~6–8 sentences
- `opener_diversity_min`: **0.6**
- `paragraph_len_stdev_min`: **1.0**
- `idiom_level`: **"light"** or `"none"`
- `allow_first_person`: **true** / false
- `banlist_extra`: `[ "add any extra phrase to ban" ]`
- `must_include`: `[ "phrases that must appear verbatim" ]`

5. Click **Run**. The output will be rewritten with the rules + inputs.  
6. If it feels too “clean,” bump **variance**: increase `stdev_min` to 7 or 8, and set `avg_len_target` to 24–26.  
7. If it feels too punchy, reduce `question_rate` to 0 and set `avg_len_target` 18–20.

---

## C) Change defaults (optional)
You can keep the UI inputs blank and set your own “house defaults” by editing `SKILL.md`:
- Search for **Inputs Schema** and change the `default` values.  
- Re-upload the updated `SKILL.md` to Claude (Skills → Create/Update → Upload).

---

## D) Guardrails the skill enforces
- Never uses the **em dash (—)**; replaces with period or hyphen.  
- Removes AI giveaways (clichés, heavy transitions, template openers).  
- Keeps meaning intact; no new claims added.  
- Adds controlled variety (burstiness), opener diversity, and optional questions.

---

## E) Troubleshooting
- If output seems robotic: raise `stdev_min`, add `use_questions: true`, and allow **light** idioms.  
- If output seems too casual: set `formality: "formal"` and `directness: "soft"`.  
- If specific terms must appear: add them to `must_include`.  
- If something must **not** appear: add it to `banlist_extra` (e.g., "robust").

---

## F) Copy‑paste examples (set via **Use skill → Inputs**)

You can also use the example YAMLs in `/examples/prompts.yaml` included with this bundle.

### 1) General web copy (landing pages, product pages)
```yaml
formality: "neutral"
directness: "balanced"
reading_level: "10th"
variance_level: "medium"
burstiness: { avg_len_target: 22, stdev_min: 6, min_short: 8, min_long: 32 }
use_questions: true
question_rate: 1
opener_diversity_min: 0.6
paragraph_len_stdev_min: 1.0
idiom_level: "light"
allow_first_person: true
```

**Prompt starter you can paste with your draft:**
> Rewrite this section for clarity and flow. Keep the meaning. Keep key terms. Improve rhythm and vary openings.

### 2) Blog / editorial copy
```yaml
formality: "neutral"
directness: "balanced"
reading_level: "12th"
variance_level: "high"
burstiness: { avg_len_target: 25, stdev_min: 7, min_short: 8, min_long: 36 }
use_questions: true
question_rate: 2
opener_diversity_min: 0.7
paragraph_len_stdev_min: 1.4
idiom_level: "light"
allow_first_person: true
```
**Prompt:**  
> Edit for a natural magazine voice. Vary sentence rhythm. Keep quotes and dates exactly as written.

### 3) Product specs / PRDs
```yaml
formality: "formal"
directness: "balanced"
reading_level: "12th"
variance_level: "low"
burstiness: { avg_len_target: 20, stdev_min: 5, min_short: 8, min_long: 30 }
use_questions: false
question_rate: 0
opener_diversity_min: 0.5
paragraph_len_stdev_min: 0.8
idiom_level: "none"
allow_first_person: false
```
**Prompt:**  
> Tighten this PRD section. Keep strict meaning. Use precise verbs. Avoid hype and vague terms.

### 4) Marketing / ad copy
```yaml
formality: "neutral"
directness: "blunt"
reading_level: "10th"
variance_level: "medium"
burstiness: { avg_len_target: 21, stdev_min: 7, min_short: 6, min_long: 28 }
use_questions: true
question_rate: 2
opener_diversity_min: 0.65
paragraph_len_stdev_min: 1.2
idiom_level: "light"
allow_first_person: true
```
**Prompt:**  
> Make this copy punchy and specific. Keep claims accurate. Shorten long lines. Keep verbs active.

---

## G) Example Inputs file
See `/examples/prompts.yaml` for all presets above, ready to copy into the Inputs panel.
