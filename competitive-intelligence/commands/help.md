---
name: ci:help
description: Show all available /ci: commands with their options, arguments, and usage examples. Covers built-in and any custom commands in the commands/ directory.
type: command
---

# /ci:help — Command Reference

## Role

Read all command files in `commands/` and render a complete reference guide. This is a read-only command — it discovers commands dynamically so custom commands added via `/ci:new` appear automatically without needing to update this file.

---

## Steps

### Step 1 — Discover all commands

Read every `.md` file in `commands/`. For each file, extract:
- `name` from frontmatter
- `description` from frontmatter
- Arguments table (if present)
- "When to use" or "When to run" section (if present)

Also read:
- `agents/extractors/0_EXTRACTORS.md` — for extractor names and tiers (used in `/ci:run` reference)
- `agents/synthesizers/0_SYNTHESIZERS.md` — for synthesizer names (used to explain what `/ci:run` triggers)

### Step 2 — Render the reference

Print the full reference in the order below. Within each group, sort commands alphabetically.

---

## Output Format

```
/ci: Command Reference
══════════════════════════════════════════════════════════

PIPELINE
  /ci:run             Run the competitive analysis pipeline
  /ci:profile         Re-synthesize one competitor's profile from latest snapshot
  /ci:market          Cross-competitor: positioning map, Porter's 5, whitespace
  /ci:differentiation Cross-competitor: JTBD gaps, underserved segments, positioning rec
  /ci:blue-ocean      Cross-competitor: Blue Ocean value curve

WORKSPACE
  /ci:setup       First-run setup — create workspace files and seed competitors
  /ci:status      Show pipeline health dashboard
  /ci:publish-brief  Publish a saved brief to distribution channels

CONFIGURATION
  /ci:schedule    Configure automated run cadences

EXTENSIBILITY
  /ci:new         Scaffold a new extractor, market synthesizer, or command
  /ci:import      Import an existing skill and transforms it into extractor or synthesizer

HELP
  /ci:help        Show this reference

[CUSTOM]          (shown only if custom commands exist in commands/)
  /ci:[name]      [description from frontmatter]
```

Then render each command as a full entry:

---

```
/ci:run
──────────────────────────────────────────────────────────
Run the full competitive analysis pipeline — extract signals
from due competitors, detect deltas, synthesize, and deliver
a brief.

Usage:
  /ci:run
  /ci:run <competitor-name>
  /ci:run --name <extractor-name>
  /ci:run force_run=true
  /ci:run deep-dive <competitor-name>

Options:
  <competitor-name>          Run only this competitor (bypasses cadence check)
  --name <extractor-name>    Run only this extractor for due competitors
  force_run=true             Run all competitors regardless of cadence
  deep-dive                  Return full analysis instead of brief summary
  alert-check                Run all direct-tier competitors immediately

Extractors (active):
  positioning-extractor      direct, adjacent
  pricing-extractor          direct, adjacent
  changelog-extractor        direct

Extractors (planned — not yet active):
  review-extractor, jobs-extractor, news-extractor

Cadences (when no filter is set):
  direct competitors         every 7 days
  adjacent competitors       every 30 days
  aspirational competitors   every 90 days

Examples:
  /ci:run                         → check all due competitors
  /ci:run Aha.io                  → run Aha.io only
  /ci:run force_run=true          → refresh everything now
  /ci:run deep-dive Productboard  → full analysis for Productboard
  /ci:run --name changelog-extractor Aha.io  → just changelog for Aha.io
```

---

```
/ci:publish-brief
──────────────────────────────────────────────────────────
Publish a saved brief to configured distribution channels
(Slack, email, Notion). Prompts for channel selection and
confirmation before sending.

Usage:
  /ci:publish-brief
  /ci:publish-brief --brief <date>
  /ci:publish-brief --all

Options:
  (no args)          Publish the most recent brief
  --brief <date>     Publish a specific brief by date (YYYY-MM-DD)
  --all              Publish all briefs in workspace/briefs/, oldest first

Examples:
  /ci:publish-brief                      → publish latest brief
  /ci:publish-brief --brief 2026-03-19   → publish a specific date
  /ci:publish-brief --all                → backfill all briefs to Notion
```

---

```
/ci:market
──────────────────────────────────────────────────────────
Run Stage 2 market synthesis across all competitor profiles.
Produces a positioning map, Porter's 5 Forces, whitespace
gaps, and strategic implications.

Usage:
  /ci:market

No options. Reads all profiles in workspace/profiles/.
Requires at least 2 profiles with populated SWOT sections.
Writes to workspace/syntheses/market-[date].md.

When to run:
  Monthly, after a major competitor move, or before a
  strategy or positioning review.
```

---

```
/ci:blue-ocean
──────────────────────────────────────────────────────────
Run Blue Ocean value curve analysis across all competitor
profiles. Identifies where the market converges (red ocean)
and where uncontested strategic space exists.

Usage:
  /ci:blue-ocean

No options. Reads all profiles in workspace/profiles/.
Writes to workspace/syntheses/blue-ocean-[date].md.

When to run:
  Quarterly or when doing strategy work.
```

---

```
/ci:setup
──────────────────────────────────────────────────────────
First-run setup. Creates workspace/ourproduct.md,
workspace/positioning.md, and optionally seeds
workspace/competitors.yaml.

Usage:
  /ci:setup

Run this once before your first /ci:run.
```

---

```
/ci:status
──────────────────────────────────────────────────────────
Show a pipeline health dashboard — extractors, synthesizers,
schedule, distribution, and workspace state.

Usage:
  /ci:status

Read-only. Shows what's active, what's planned, and anything
needing attention.
```

---

```
/ci:schedule
──────────────────────────────────────────────────────────
Configure OS-level cron jobs for automated pipeline runs.

Usage:
  /ci:schedule

Interactive. Shows current schedule, then lets you set cadences
for /ci:run, /ci:market, and /ci:blue-ocean. Writes to
workspace/schedule.yaml and registers entries in crontab.

Cron jobs fire even when no Claude Code session is open.
```

---

```
/ci:import
──────────────────────────────────────────────────────────
Import an existing extractor, synthesizer, or command into
the pipeline. Validates, adapts if needed, and registers it.

Usage:
  /ci:import

Interactive. Paste or provide the file path. Detects type,
validates against the pipeline contract, places in the correct
directory, and adds to the relevant index.
```

---

```
/ci:new
──────────────────────────────────────────────────────────
Scaffold a new extractor, synthesizer, or command from the
right template. Guides you through what it needs and registers
it so the orchestrator picks it up automatically.

Usage:
  /ci:new

Interactive. Choose the type, provide a name and description,
and the command generates the file and registers it.
```

---

If any custom commands were found in `commands/` (files not in the built-in list above), append a **Custom Commands** section with the same entry format.

---

## Quality Rules

- Read-only — never write, register, or modify anything
- If a command file can't be parsed, skip it and note: `[ci:<name> — could not parse, check commands/<name>.md]`
- Arguments and options shown in the reference must match the actual command files — derive them from the files, don't invent them
- Custom commands appear under their own section so they're clearly distinguishable from built-ins
