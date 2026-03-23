---
name: ci:profile
description: Re-synthesize one competitor's profile from their most recent snapshot, without re-running extraction. Use after manually editing a snapshot, after adding a new Stage 1 synthesizer, or when you want a fresh profile read without a full pipeline run.
type: command
---

# /ci:profile — Re-synthesize Competitor Profile

## Role

Run the profile-synthesizer for a single competitor using their most recent snapshot as input. Does not fetch anything from the web — extraction is not re-run. Produces updated `## Direction`, `## SWOT`, and `## Our Read` sections in the competitor's profile.

---

## Arguments

| Argument | Values | Default | Notes |
|----------|--------|---------|-------|
| `competitor_name` | competitor name | required | Must match a name in `workspace/competitors.yaml` |

**Parsing rules:**
- A bare name is used as `competitor_name`: `/ci:profile Aha.io`
- If no name is given, ask: "Which competitor? (name or list with /ci:status)"

---

## Steps

### Step 1 — Resolve competitor

If no argument was given, ask the user to specify a competitor name.

Read `workspace/competitors.yaml` to validate the name exists. If not found:
> "No competitor named '[name]' found in workspace/competitors.yaml. Run /ci:status to see tracked competitors."

### Step 2 — Find most recent snapshot

Look in `workspace/snapshots/[competitor-name]/` for `.md` files. Sort by filename date descending — the most recent is the target snapshot.

If no snapshots exist:
> "No snapshots found for [name]. Run /ci:run [name] first to generate a snapshot."

Find the second-most-recent snapshot (if it exists) — this is `previous_snapshot`, used by the profile-synthesizer to write the Direction section.

Confirm what will be used:
> "Synthesizing profile for [name] using snapshot from [date]."

### Step 3 — Read inputs

Read in parallel:
- New snapshot: full content of `workspace/snapshots/[name]/[latest-date].md`
- Previous snapshot: full content of `workspace/snapshots/[name]/[second-latest].md` (or empty string if none)
- Current profile: full content of `workspace/profiles/[name].md` (or empty string if none)
- Our product context: `workspace/ourproduct.md` (or empty string if missing)

### Step 4 — Run profile builder

Follow the process in `skills/profile-builder.md`, passing:
- `competitor_name`: the name
- `snapshot`: content of the new snapshot
- `previous_snapshot`: content of the previous snapshot (or empty string)
- `current_profile`: content of the existing profile (or empty string)

The builder writes `## Current State` and `## Direction` to the profile.

### Step 5 — Run active analyzers

Read `workspace/config.yaml` to get `profile_analyzers`. For each name in the list, check `agents/analyzers/0_ANALYZERS.md` for `status: active`. Run all active analyzers in parallel.

**Inputs to each analyzer:**
- `competitor_name`: the name
- `snapshot`: content of the new snapshot
- `our_product_context`: content of `workspace/ourproduct.md` (or empty string)
- `current_analysis_section`: the existing `## Analysis — [Framework]` section from the current profile (or empty string)

Each analyzer writes its `## Analysis — [Framework]` section to the profile per its own merge rules.

### Step 6 — Append run history and return summary

Append a row to the `## Run History` table in the profile:

```
| [today's date] | [snapshot path] (profile re-synthesis only — no extraction) | [1-sentence summary of key finding or change] |
```

Print a summary:

```
Profile updated: workspace/profiles/[name].md
  Snapshot used: workspace/snapshots/[name]/[date].md
  Sections updated: Current State, Direction, [list of analyzer sections run]

[2–3 sentence read of the most important finding]
```

---

## When to use

- After manually editing a snapshot to correct or add data
- After adding a new analyzer — re-run to see what the new lens produces against existing snapshot data
- After adding a new extractor — re-run to see how new data lands in Current State and analysis sections
- When the profile feels stale but you don't need fresh extraction
