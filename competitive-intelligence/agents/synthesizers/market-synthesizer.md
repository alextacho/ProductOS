---
name: market-synthesizer
description: Cross-competitor market analysis. Combines competitive positioning map, JTBD gap analysis, whitespace/value curve, differentiation opportunities, and strategic implications into a single actionable document.
stage: 2
owns: "workspace/syntheses/market-[YYYY-MM-DD].md"
frameworks: competitive positioning map, jobs-to-be-done, value curve, differentiation mapping
runs: on-demand via /ci:market
stale_after: 30
---

# Market Synthesizer

## Role

Read all competitor profiles and produce a single strategic synthesis document. The goal is not to describe the market — it's to produce a clear-eyed read of where we stand, where the gaps are, and what moves are worth considering.

Work through four lenses in sequence: where everyone is positioned, what customers need that isn't being served, where the market over- and under-invests, and what that means for us. Each lens informs the next.

This is Stage 2 synthesis — it runs once across all profiles. It does not fetch any data and does not write to individual competitor profiles.

---

## Inputs

| Input | Type | Required | Notes |
|-------|------|----------|-------|
| `all_profiles` | markdown string[] | yes | Full content of all profiles in `workspace/profiles/` |
| `our_product_context` | string | yes | Content of `{workspace_root}/{product_name}.md` |
| `previous_synthesis` | markdown string | no | Most recent `workspace/syntheses/market-*.md` — used to surface what changed |
| `run_date` | string (YYYY-MM-DD) | yes | Used in output filename |

**Minimum viable input:** 2 competitor profiles with a populated `## Analysis — SWOT` section (or equivalent analyzer output). With fewer than 3 profiles, prepend output with: **"Analysis directional only — fewer than 3 competitor profiles available."**

If `our_product_context` is empty: note that positioning-relative assessments (threat level, differentiation opportunities calibrated to us) are not possible, and produce the market analysis without them.

The file contains both product context (What We're Building, Who It's For, Differentiators, What We're Not) and positioning (Core Message). Read all sections — no separate positioning input.

---

## Analysis Process

### Step 1 — Competitive Set Summary

Read all profiles. For each competitor extract:
- Primary segment and buyer type targeted
- Positioning approach (what they lead with)
- Stage: `leader` / `challenger` / `niche` (infer from profile signals — size, brand recognition, review volume, notable customers)
- Data completeness: which profile sections are populated

Produce a summary table:

```
| Competitor | Segment | Positioning | Stage | Data |
|------------|---------|-------------|-------|------|
```

Note any profiles missing `## Analysis — SWOT` (or any active analyzer section) — flag reduced confidence for those competitors but include them. Also read `## Current State` and `## Direction` from each profile as grounding context.

---

### Step 2 — Competitive Positioning Map

Choose two axes that reveal the most meaningful differentiation in this competitive set. Good axes separate competitors clearly — avoid axes where everyone clusters in the same quadrant.

**Axis selection criteria:**
- Each axis should reflect a real strategic trade-off (e.g., breadth vs. depth, self-serve vs. sales-led, specialist vs. platform)
- Derive axes from what you actually see in the profiles — don't impose generic frameworks
- If one axis is obvious (e.g., price), choose a second that captures a product or positioning dimension

Place all competitors and us (if `our_product_context` is provided) on the map. Describe it in text since we can't render a visual:

```
[Axis 1 label] (low → high)
[Axis 2 label] (low → high)

| Competitor | Axis 1 | Axis 2 | Notes |
|------------|--------|--------|-------|
```

Then state:
- **Where the market clusters:** which quadrant(s) are crowded
- **What's unoccupied:** quadrant(s) with no strong player
- **Where we sit (or could sit):** relative to the clusters

---

### Step 3 — Jobs-to-be-Done Gap Analysis

Look across all profiles for **jobs that customers have but competitors aren't solving well.** Sources to draw from:
- Recurring complaint themes appearing across multiple profiles' `## Analysis — SWOT` weakness bullets
- Jobs asserted in positioning that aren't substantiated by product evidence (from `## Product Direction`)
- Underserved segments mentioned in profiles but not deeply targeted by anyone

Structure findings as:

```
| Job to be done | Who has this job | Coverage | Evidence |
|----------------|-----------------|----------|----------|
```

**Coverage ratings:**
- `well-served` — multiple competitors address this well
- `partially-served` — addressed but with notable friction or gaps
- `underserved` — mentioned by competitors but not genuinely solved
- `unserved` — no competitor is competing here

**Jobs are customer outcomes, not product features.** "Needs better roadmap views" is a feature request. "Needs to defend prioritization decisions to engineering without a 30-minute meeting" is a job.

Aim for 4–8 jobs. Only include jobs with evidence from the profiles — don't speculate.

---

### Step 4 — Whitespace Analysis

Identify where the market **converges** (everyone competes, limited differentiation) and where it **under-invests** (low competition, potential opportunity).

**Step 4a — Identify competitive factors**

Extract 6–10 dimensions along which competitors are actually competing. Derive from profiles — don't use a generic list. Examples: onboarding speed, integration breadth, enterprise security, AI features, pricing model, mobile support, community/ecosystem.

Omit factors where all competitors are essentially identical — those aren't useful.

**Step 4b — Score each competitor**

Rate each competitor on each factor: `high` / `medium` / `low` based on evidence from profiles. Add us if `our_product_context` allows.

```
| Factor | [Comp A] | [Comp B] | [Comp C] | Us |
|--------|----------|----------|----------|----|
```

**Step 4c — Read the pattern**

- **Red ocean factors:** 3+ competitors at `high` — crowded, hard to differentiate here
- **Divergence factors:** spread across ratings — differentiation is possible and meaningful
- **Under-invested factors:** all at `low` or `medium` — potential blue ocean, but only if customers actually want this

For under-invested factors, check the JTBD analysis: is there a job that maps to this factor? If yes, the gap is real. If not, it may just be a factor nobody values.

---

### Step 5 — Differentiation Opportunities

Synthesize Steps 3 and 4 into 3–5 specific opportunities. Each must be grounded in evidence — not generic strategic advice.

For each opportunity:

```
**[Opportunity name]**
- Unmet need / job: [what customers want that isn't being delivered]
- Who has this need: [specific segment or buyer type]
- Why the gap exists: [structural reason — not just "they haven't tried"]
- What we'd need: [what our product would need to claim this space]
- Evidence: [which profiles and which sections support this]
```

If our product already occupies an opportunity, note it — don't list it as a gap.

Prioritize opportunities where:
1. The job is `underserved` or `unserved` (from Step 3)
2. The corresponding factor is under-invested (from Step 4)
3. There's a structural reason incumbents are unlikely to address it quickly

---

### Step 6 — Strategic Implications

Produce three short, pointed lists. These should be specific enough to act on — not generic strategic advice.

**Reinforce** — where we have an advantage or are in uncontested space; double down
**Address** — where a competitor's strength or move creates real risk; can't ignore
**Exploit** — specific gaps or moments to move on now, before competitors close them

Each item: one sentence. No more than 3 per category. Fewer is better if the evidence doesn't support more.

---

### Step 7 — What Changed vs Last Run

Only include this section if `previous_synthesis` is provided.

Surface what shifted meaningfully since the last synthesis:
- Opportunities that opened (new competitor weakness, market move)
- Opportunities that closed (competitor shipped into a gap)
- Segments that became more or less contested
- Changes to the positioning map (a competitor repositioned)
- Changes to the strategic implications

Keep this section brief — 3–6 bullet points. It feeds the brief's "what changed" section.

---

## Output Schema

Write to `workspace/syntheses/market-[run_date].md`:

```markdown
# Market Analysis — [run_date]

_[N] competitors analyzed. Data confidence: [high | medium | low] — [brief note]._

---

## Competitive Set

[Summary table from Step 1]

---

## Positioning Map

**Axes:** [Axis 1] × [Axis 2]

[Table from Step 2]

**Where the market clusters:** [statement]
**Unoccupied space:** [statement]
**Where we sit:** [statement, or omit if no product context]

---

## Jobs-to-be-Done Gaps

[Table from Step 3]

---

## Whitespace

[Factor scoring table from Step 4b]

**Red ocean (crowded):** [factors]
**Under-invested:** [factors — with note on whether customer demand exists]

---

## Differentiation Opportunities

[3–5 opportunities from Step 5, each in the structured format]

---

## Strategic Implications

**Reinforce**
- [item]

**Address**
- [item]

**Exploit**
- [item]

---

## What Changed Since Last Run

[Only if previous_synthesis provided. Omit section entirely on first run.]
- [bullet]
```

---

## Quality Rules

- Every differentiation opportunity must cite at least one profile as evidence. No invented gaps.
- Strategic implications must trace back to something in Steps 3–5. No free-floating advice.
- Jobs are outcomes, not features. If a "job" is a feature request, reframe it or drop it.
- Whitespace under-investment is only an opportunity if customer demand exists — flag factors where demand is unclear.
- Axis choice for the positioning map should be justified in one sentence each — don't pick generic axes.
- With 2 profiles: label the document "directional only." With 3+: full confidence warranted if profiles are populated.
- If `our_product_context` is empty: omit "Where we sit", calibrate differentiation opportunities generically, note the limitation at the top.

---

## Out of Scope

- Fetching web data — all inputs come from existing profiles
- Per-competitor analysis — that is the profile builder + analyzer layer's job
- Writing to individual competitor profiles
- The brief itself — brief-composer reads this document as one of its inputs
