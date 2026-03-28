---
name: profile-builder
description: Reads a competitor's completed snapshot and produces two framework-agnostic sections in the profile — Current State (normalized facts) and Direction (what changed this run). System component — runs automatically for every competitor in every pipeline run.
stage: system
layer: system
owns: "## Current State, ## Direction in profile"
runs: after delta-detector, before analyzers
---

# Profile Builder

## Role

Read a competitor's freshly completed snapshot. Extract and normalize key facts into `## Current State`. Write `## Direction` summarizing what changed this run. Both sections are framework-agnostic — they are inputs to analyzers and downstream consumers (brief composer, market synthesizer), not analysis outputs.

This is a **system component** — it runs automatically for every competitor in every pipeline run. It does not apply any analytical framework. Analysis is handled by the analyzer layer (`agents/analyzers/`).

---

## Inputs

| Input | Type | Required | Notes |
|-------|------|----------|-------|
| `competitor_name` | string | yes | |
| `snapshot` | markdown string | yes | Full snapshot after extractors and delta detector ran |
| `previous_snapshot` | markdown string | no | Previous snapshot. Used to write Direction. Empty string on first run. |
| `current_profile` | markdown string | no | Existing profile content. Used to preserve sections not owned by builder. Empty string on first run. |

---

## Process

### Step 1 — Extract facts from snapshot

Read all sections present in the snapshot. For each section present, extract the most important facts — compressed to their essential signal. Omit detail that belongs in the raw snapshot; this is the scannable summary layer.

**What to extract per section type** (only extract from sections that are actually present):

- **Positioning-type sections:** their headline claim, target audience, 2–3 stated differentiators
- **Product/pricing-type sections:** pricing model, entry price, notable tier structure
- **Product direction/changelog-type sections:** top 2–3 recent releases or investment themes (1 line each)
- **Market presence/reviews-type sections:** overall score or signal, dominant praise theme, dominant complaint theme
- **Signals-type sections:** most notable hiring signal, funding status if changed, key news item
- **Delta section:** highest-weight change from this run (1 line)

If a section is absent or `[unavailable]`, omit that field from Current State entirely — do not write "unknown" placeholders.

### Step 2 — Write Current State

Write `## Current State` to the profile. Only include fields where data is present.

```markdown
## Current State
_Last updated: [run_date]. Source: [snapshot path]._

[Field]: [Value — 1 line]
[Field]: [Value — 1 line]
...
```

Use plain field labels derived from what's present (e.g., "Positioning", "Pricing", "Recent releases", "Review signal", "Key signal"). Keep each value to one line. This section should be readable in under 30 seconds.

Always rewrite this section in full — it reflects current state, not accumulated history.

### Step 3 — Write Direction

Write `## Direction` to the profile. This is the first thing someone reads to understand what's happening with this competitor right now.

**If `previous_snapshot` is provided:**
- Lead with what changed this run — reference the highest-weight deltas from `## Delta`
- State current heading: where they're going based on the full snapshot picture
- Include Watch for: the leading indicator to monitor (carry from existing profile if strategic-signals didn't run this cycle)

**If no `previous_snapshot` (first run):**
- State current heading based on available snapshot data
- Note this is the initial run with no prior comparison

```markdown
## Direction
_Last updated: [run_date]._

- **What changed this run:** [1–2 sentences on highest-weight deltas, or "No significant changes detected"] _(omit on first run)_
- **Current heading:** [where they're going — 1 sentence]
- **Watch for:** [leading indicator — 1 sentence]
```

Always rewrite this section in full — it is inherently run-scoped.

---

## Output Schema

Update `{workspace_root}/profiles/[competitor_name].md`:

- **`## Current State`** — always rewrite in full
- **`## Direction`** — always rewrite in full
- All other sections (analyzer sections, Run History) — preserve exactly as-is

On first run: create the profile file with the standard header, Current State, and Direction. Leave space for analyzer sections — they will be added by the analyzer layer.

**Profile file structure (for reference — builder only writes its own sections):**

```markdown
# [Competitor Name]

---

## Current State
...

## Direction
...

## Analysis — SWOT
[written by swot-analyzer, preserved by builder]

## Analysis — [Other]
[written by other analyzers, preserved by builder]

## Run History
| Date | Snapshot | Summary |
|------|----------|---------|
[appended by orchestrator]
```

---

## Quality Rules

- Current State is facts only — no interpretation, no analysis
- Each field is one line — this is a summary layer, not a report
- Only write fields where data is actually present in the snapshot
- Never touch analyzer sections or Run History
- Direction must be scannable in 10 seconds
