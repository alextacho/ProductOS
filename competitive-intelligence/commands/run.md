---
name: ci:run
description: Run the competitive analysis pipeline — extracts signals from due competitors, detects deltas, runs synthesis, and delivers a brief. The main weekly driver of the pipeline.
type: command
---

# /ci:run — Run the Pipeline

## Role

Entry point for the competitive analysis pipeline. Parse any arguments, then hand off to the `agents/orchestrator.md` orchestrator with the correct inputs.

---

## Arguments

Arguments can be passed inline, e.g. `/ci:run force_run=true` or `/ci:run aha.io` or `/ci:run deep-dive productboard`.

| Argument | Values | Default | Notes |
|----------|--------|---------|-------|
| `force_run` | `true` / `false` | `false` | Run all competitors regardless of schedule |
| `competitor_filter` | competitor name | none | Run a single named competitor only |
| `run_mode` | `brief` / `alert-check` / `deep-dive` | `brief` | Controls output depth |
| `--extractor <name>` | extractor name | none | Run a single extractor only — skips brief, auto re-synthesizes profile |
| `--with-market` | flag | off | After the full pipeline completes, run all active market synthesizers via `/ci:market` |

**Parsing rules:**
- A bare name that matches a competitor in `workspace/competitors.yaml` → set as `competitor_filter`
- `force_run=true` or `force_run` alone → set `force_run: true`
- `deep-dive`, `alert-check`, `brief` → set `run_mode`
- `--extractor <name>` → set `extractor_filter` to that name; activates single-extractor mode (see below)
- `--with-market` → set `run_with_market: true`
- Arguments can be combined in any order: `/ci:run force_run=true aha.io deep-dive`

---

## Steps

### Step 1 — Parse arguments and confirm scope

Parse the arguments above. Then show a one-line confirmation before running:

- Default: `Running pipeline — checking all due competitors.`
- With competitor_filter: `Running pipeline — [name] only.`
- With force_run: `Running pipeline — all competitors (force_run).`
- With run_mode != brief: `Running pipeline in [mode] mode.`
- With --extractor: `Running [extractor-name] extractor for [competitor or "all due competitors"] — will re-synthesize profile after.`
- With --with-market: append `Will run market synthesis after.`

### Step 2 — Route based on mode

**If `--extractor <name>` is set → Single-extractor mode**

Run the single-extractor flow (Step 2a) instead of the full pipeline.

**Otherwise → Full pipeline mode**

Follow the full execution process in `agents/orchestrator.md` (Step 2b).

---

### Step 2a — Single-extractor mode

Use this flow when `--extractor <name>` is set.

**Purpose:** run one extractor for one or all due competitors, update that section in the snapshot, then auto re-synthesize the profile. No brief is generated — this is for targeted signal refresh, not a full run.

1. **Resolve scope:** if `competitor_filter` is set, run only that competitor. Otherwise, determine due competitors using the same cadence logic as the orchestrator (Step 2 of `agents/orchestrator.md`). If `force_run` is set, include all competitors.

2. **Validate extractor:** check `agents/extractors/0_EXTRACTORS.md` for a row matching `<name>` with `status: active`. If not found or not active:
   > "Extractor '[name]' not found or not active. Run /ci:status to see available extractors."

3. **For each in-scope competitor — in parallel:**
   - Resolve the most recent snapshot file in `workspace/snapshots/[competitor]/`
   - If no snapshot exists, create a new one (using the same header format as the orchestrator)
   - Run the specified extractor for this competitor, writing its section to the snapshot
   - After the extractor completes: run the delta detector against the previous snapshot (or skip if no previous snapshot)
   - Run the profile synthesizer with the updated snapshot — this re-synthesizes SWOT, Our Read, Direction

4. **Update registry:** set `last_updated` in `workspace/competitors.yaml` only for competitors where the extractor succeeded.

5. **Report results:**
   ```
   Extractor run: [name]
   ──────────────────────────────────────────────────────────
     [Competitor A]   → snapshot updated, profile re-synthesized
     [Competitor B]   → snapshot updated, profile re-synthesized
   ```
   Print a 1–2 sentence summary of the most significant finding per competitor.

**Do not generate a brief.** If the user wants a brief after a targeted run, they can run `/ci:run` (full pipeline) or the brief will be generated on the next scheduled full run.

---

### Step 2b — Full pipeline mode

Follow the full execution process in `agents/orchestrator.md`, passing:
- `run_mode` — from parsed arguments
- `competitor_filter` — from parsed arguments
- `force_run` — from parsed arguments

The orchestrator handles everything: determining what's due, running extractors, delta detection, synthesis, brief composition, and distribution.

---

### Step 3 — Market synthesis (if --with-market)

After the full pipeline (Step 2b) or single-extractor run (Step 2a) completes, if `--with-market` was set:

Follow `/ci:market` — runs all active market synthesizers in sequence. Append market synthesis results to the output.

---

## When to run

- **Automatically** — via cron job configured by `/ci:schedule` (fires `claude -p "/ci:run"`)
- **Manually** — any time you want a fresh brief
- **Force run** — `/ci:run force_run=true` to refresh all competitors regardless of cadence
- **Single competitor** — `/ci:run [name]` to check one competitor without waiting for their schedule
- **Deep dive** — `/ci:run deep-dive [name]` for full analysis output on a specific competitor
- **Single extractor** — `/ci:run --extractor changelog Aha.io` to refresh just one data source and re-synthesize
- **With market** — `/ci:run --with-market` to run the full pipeline and immediately follow with market synthesis
