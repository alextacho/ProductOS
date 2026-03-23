---
name: ci:market
description: Run market synthesis across all competitor profiles — positioning map, JTBD gaps, whitespace, differentiation opportunities, and strategic implications. Run monthly or before a strategy review.
type: command
---

# /ci:market — Market Synthesis

## Role

Read all competitor profiles and run the market-synthesizer to produce a single strategic synthesis document. This is the on-demand command for cross-competitor analysis.

---

## Steps

### Step 1 — Check profiles

Read all `.md` files in `workspace/profiles/`. Verify each has a populated `## SWOT` and `## Our Read` section. If missing, include the profile anyway but flag reduced confidence.

If fewer than 2 profiles exist: abort with:
> "Need at least 2 competitor profiles to run market synthesis. Run /ci:run first."

### Step 2 — Read context

Read in parallel:
- `workspace/ourproduct.md` — our product and positioning context
- `workspace/positioning.md` — current positioning (if exists)
- Most recent `workspace/syntheses/market-*.md` (if any exists) — passed as `previous_synthesis` to surface what changed

### Step 3 — Run market-synthesizer

Follow the full analysis process in `agents/synthesizers/market-synthesizer.md`, passing:
- `all_profiles`: full content of all profiles in `workspace/profiles/`
- `our_product_context`: content of `workspace/ourproduct.md` (or empty string)
- `our_positioning`: content of `workspace/positioning.md` (or empty string)
- `previous_synthesis`: content of most recent market synthesis (or empty string)
- `run_date`: today's date

### Step 4 — Confirm output

After the synthesizer writes its file, print the path:
> `Market synthesis written → workspace/syntheses/market-[date].md`

### Step 5 — Summary to user

Print a 4–6 sentence headline read:
- The most revealing insight from the positioning map
- The top 1–2 underserved jobs or whitespace gaps
- The single most important strategic implication

The full analysis is in the file — this is just the headline.

---

## When to run

- Monthly (after at least one full `/ci:run` cycle has completed)
- After a major competitor move — new product line, pricing change, funding
- Before a strategy review, positioning exercise, or pitch
- When `/ci:run` brief notes market synthesis is stale (threshold: 30 days)
- Automatically after a full pipeline run via `/ci:run --with-market`
