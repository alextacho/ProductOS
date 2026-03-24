---
name: delta-detector
description: Compares the new run's snapshot against the previous snapshot, produces a weighted change list ordered by signal importance, and writes the ## Delta section to the new snapshot.
layer: system
runs: after all extractors for a competitor complete; before competitor-synthesizer
---

# Delta Detector

## Role

Compare two snapshots of a competitor — the freshly written snapshot from this run and the previous stored snapshot. Produce a classified, weighted list of changes ordered by signal importance. Write the result as a `## Delta` section in the new snapshot.

This skill classifies and weights changes. It does not interpret them. "Pricing changed from X to Y" is a delta. "This means they're under competitive pressure" is synthesis — that belongs to the competitor-synthesizer.

---

## Inputs

| Input | Type | Required | Notes |
|-------|------|----------|-------|
| `competitor_name` | string | yes | As listed in competitors.yaml |
| `new_snapshot` | markdown string | yes | Full content of the new snapshot after this run's extractors have written their sections |
| `old_snapshot` | markdown string | no | Full content of the previous snapshot. If empty/null, treat all fields as `new` and return them with `change_type: new` |
| `run_date` | string (YYYY-MM-DD) | yes | Date of this run |
| `signal_weights_path` | string | yes | Path to `context/signal-weights.md` — read this file to look up weights |

---

## Process

### Step 1 — Parse both snapshots into sections

Read both `new_snapshot` and `old_snapshot`. For each `## Section` heading, identify the fields within that section. Build a field map for each version:

```
{section}.{field_name} → value
```

Examples of field paths:
- `positioning.core_message`
- `positioning.target_audience`
- `positioning.differentiators` (list — compare item by item)
- `product.pricing_model`
- `product.tiers[Starter].price`
- `product.tiers[Pro].price`
- `market_presence.review_score`
- `strategic_signals.hiring_focus`
- `strategic_signals.recent_funding`
- `product_direction.investment_themes` (list — compare themes present/absent)
- `product_direction.positioning_alignment.match`

**Parsing rules:**
- Bold fields (`**Field name:**`) → named fields, extract the value after the colon
- Tables → compare row by row, using the first column as the row key
- Bullet lists under a bold heading → treat as a set; compare items present/absent
- If a field is flagged `[unavailable]` in either version, treat it as missing data, not a change
- Ignore `## Delta`, `## SWOT (this run)`, and `## Our Read (this run)` — these are meta/synthesis sections, not extractor-owned data

### Step 2 — Identify changes

For each field path, compare old value to new value:

| Change type | Condition |
|-------------|-----------|
| `changed` | Field exists in both versions; value is different |
| `new` | Field exists in new snapshot but not in old (or entire section is new) |
| `removed` | Field exists in old snapshot but is absent in new |
| `unchanged` | Values are identical — **do not include in output** |

**Unchanged fields must not appear in the output.** The change list should contain only fields that actually changed.

**What counts as a meaningful change:**
- Any edit to a bold field value
- Any row added, removed, or changed in a table
- Any bullet added to or removed from a list (e.g., new differentiator claim, new complaint theme)
- A new release in `product_direction` that wasn't in the previous snapshot
- A new investment theme, or an existing theme changing category/volume
- A confidence flag changing (e.g., `unknown` → actual value)

**What does not count as a change:**
- Whitespace differences
- Minor rephrasing that preserves meaning (use judgment — when in doubt, flag it)
- Timestamp-only changes

### Step 3 — Classify and weight each change

For each change:

1. **Determine signal type** from the section and field the change occurred in:

| Section / Field | Signal type |
|-----------------|-------------|
| `## Positioning` — core message, tone | `positioning` |
| `## Positioning` — differentiators (list change) | `positioning` |
| `## Product` — pricing tiers (price, limits) | `pricing` |
| `## Product` — pricing model (e.g., freemium → paid) | `pricing` |
| `## Product` — key features (new item) | `product` |
| `## Product` — integrations (new item) | `product` |
| `## Product Direction` — new AI-categorized release | `product` |
| `## Product Direction` — new investment theme | `product` |
| `## Product Direction` — positioning alignment match changes | `positioning` |
| `## Market Presence` — review score direction change | `reviews` |
| `## Market Presence` — praise/complaint themes (new item) | `reviews` |
| `## Strategic Signals` — hiring focus | `hiring` |
| `## Strategic Signals` — recent funding | `funding` |
| `## Strategic Signals` — recent news | `news` |

2. **Look up the weight** by reading `context/weights.md` and matching the signal type + nature of the change.

3. **Assign change type** from Step 2 (`changed`, `new`, `removed`).

### Step 4 — Order and format output

Order all deltas by weight: `high` → `medium-high` → `medium` → `low-medium` → `low`.

Within the same weight, order by section: Positioning → Product → Product Direction → Market Presence → Strategic Signals.

Format the delta list:

```yaml
## Delta

changes_detected: [N]

---

- field: [field path]
  change_type: changed | new | removed
  signal_type: [type]
  weight: high | medium-high | medium | low-medium | low
  old: "[previous value — quote exactly from old snapshot, or 'none' if new]"
  new: "[current value — quote exactly from new snapshot, or 'none' if removed]"
```

If `changes_detected` is 0, return:

```yaml
## Delta

changes_detected: 0
note: No meaningful changes detected since last run.
```

### Step 5 — Write to snapshot

Write the formatted `## Delta` block to the new snapshot file, after all extractor sections and before the `## SWOT (this run)` section.

---

## Output

Return two things:

1. **The delta list** (YAML block as defined above) — passed to competitor-synthesizer and brief-composer
2. **Confirmation** that `## Delta` was written to the snapshot

---

## Edge Cases

| Situation | How to handle |
|-----------|---------------|
| No prior snapshot exists | Return all fields as `change_type: new`; note in Delta: "Initial snapshot — no previous run to compare against. All fields are new." |
| A section exists in old snapshot but is entirely missing in new (extractor failed) | Do not flag as `removed` — this is a fetch failure, not a real change. Note in Delta: "[Section] unavailable this run — skipped delta detection for that section" |
| A field was `[unavailable]` in old and is now populated | Flag as `change_type: new` with the actual new value |
| A field was populated in old and is now `[unavailable]` | Flag as `change_type: removed` with weight `low` — likely a fetch issue, not a real removal |

---

## Quality Rules

- A delta list that includes unchanged fields is worse than useless — it trains the user to ignore it. If in doubt about whether something changed, err toward omitting it.
- Weight lookups must reference `signal-weights.md` — do not guess weights.
- Never modify extractor sections — the delta detector reads them, does not write them.

---

## Out of Scope

- **Fetching new data** — that's the extractors' job. This skill receives already-written snapshots.
- **Interpreting what changes mean** — that's the competitor-synthesizer's job. Return `pricing changed from X to Y`, not `they may be under pressure`.
- **Writing to the profile** — the delta detector writes only to the snapshot. Profile updates are the synthesizer's responsibility.
