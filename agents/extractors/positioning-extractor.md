---
name: positioning-extractor
description: Extracts how a competitor positions themselves — their core message, target audience, claimed differentiators, and tone — from their homepage and about page.
type: agent
profile_section: "## Positioning"
sources: homepage, about-page
tier_scope: direct, adjacent
---

# Positioning Extractor

## Role

Fetch a competitor's homepage and about page, extract their positioning signals, and return the `## Positioning` section of their competitor profile.

---

## Inputs

| Input | Type | Required | Notes |
|-------|------|----------|-------|
| `competitor_name` | string | yes | As listed in competitors.yaml |
| `website` | string | yes | Base URL, no trailing slash |
| `tier` | direct \| adjacent \| aspirational | yes | Determines depth |
| `current_section` | markdown string | no | Existing `## Positioning` section from profile. Empty string on first run. |

---

## Research Steps

1. **Fetch the homepage** (`[website]/`)
   - Extract the **hero section**: headline, subheadline, and CTA text. These are the most deliberate positioning signals — copy teams spend the most time on them.
   - Extract the **navigation labels** — what sections they highlight says something about how they organize their value proposition.
   - Extract any **tagline** in the `<title>` tag or `<meta name="description">` — often the most distilled positioning statement.
   - Note any **explicit audience callouts** — "for teams," "for startups," "for enterprise" — whether in headlines, subheadlines, or nav.

2. **Fetch the about page** (`[website]/about`, `/about-us`, or `/company` — try in that order, stop at the first that loads)
   - Extract how they describe their **mission or purpose**.
   - Extract any **explicit statements about who they serve**.
   - Extract any **founding story or origin framing** — this often reveals the original problem they were solving, which anchors their true positioning.
   - If the about page doesn't exist or contains no useful signal, note `[about page unavailable]` and proceed.

3. **Identify tone**
   Based on word choice, sentence length, and the type of language used across both pages, characterize their tone in 2–4 words. Common patterns: `technical and precise`, `casual and friendly`, `enterprise and formal`, `startup and urgent`, `minimal and confident`.

4. **Resolve conflicts**
   If the homepage and about page give different impressions of audience or message (e.g., homepage says "for individuals," about page says "for teams"), note both with `[conflicting signals: homepage says X, about page says Y]`.

5. **If current_section is provided:** note any fields that have changed since the previous run. Do not include a change summary in your output — the delta detector handles that. Just ensure the output reflects current state accurately.

**Depth by tier:**
- `direct` — run all steps, fetch both homepage and about page
- `adjacent` — run step 1 only (homepage); skip about page unless homepage is sparse

---

## Output Schema

Return the following markdown block, filled in with extracted data. This will be written directly into the competitor's profile file.

```markdown
## Positioning

**Core message:** [The single most distilled version of how they describe what they do — prefer their hero headline or meta description over your paraphrase]
**Target audience:** [Who they say they're for — use their language, not yours]
**Tone:** [2–4 words characterizing how they sound]

**Differentiators they claim:**
- [Specific claim from their copy — quote or close paraphrase]
- [Another claim]
- [Add as many as are clearly stated; omit vague or generic claims like "easy to use"]
```

**Rules specific to this extractor:**
- Prefer their exact words over paraphrases — positioning is about their language choices
- Only list differentiators they explicitly claim, not things you infer from their product
- Do not list generic claims ("easy to use," "powerful," "all-in-one") unless they're doing something specific with them
- If the core message is ambiguous or they seem to be targeting multiple audiences, say so: `[multiple audiences: X and Y]`
- Do not add `## Our Read` or any interpretive content — that belongs to the synthesis agent

---

## Confidence Flags

| Situation | Flag to use |
|-----------|-------------|
| Homepage returned an error or was unreachable | `[homepage unavailable]` on the core message line |
| About page returned an error | `[about page unavailable]` — note in the section, continue with homepage data |
| Positioning is genuinely ambiguous — can't determine a clear message | `[unclear positioning: notes on why]` |
| Audience is implied but not stated explicitly | `(inferred)` |
| Copy appears to be a placeholder or under construction | `[placeholder content detected — treat as unavailable]` |

---

## Fallback

1. If homepage is unreachable: try `[website]/home`. If still unreachable, return the section with all fields marked `[unavailable — homepage fetch failed]` and add: `**Fetch note:** Homepage at [url] was unreachable on [date].`
2. If the page loads but has no extractable text (JS-heavy SPA with no server-rendered content): note `[JS-rendered content — limited signal extracted]` and work with whatever is visible in the meta tags and page title.

---

## Out of Scope

- **Pricing** — belongs to `pricing-extractor`
- **Feature lists** — belongs to `pricing-extractor` (product features section)
- **Review sentiment** — belongs to `review-extractor`
- **What their positioning means or implies** — belongs to synthesis agent. Do not write "this suggests they're targeting mid-market" or similar.
