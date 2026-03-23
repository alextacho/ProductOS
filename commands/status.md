---
name: ci:status
description: Show the current state of the competitive analysis pipeline — extractors, synthesizers, schedule, distribution, and workspace health. A quick dashboard before running or after setup.
---

# /ci:status — Pipeline Status

## Role

Read the pipeline's index files and config, then render a single-screen status dashboard. No fetching, no analysis — just a clear picture of what's configured, what's active, what's pending, and what needs attention.

---

## Steps

### Step 1 — Read everything in parallel

Read all of the following:
- `agents/extractors/0_EXTRACTORS.md`
- `agents/synthesizers/0_SYNTHESIZERS.md` — market synthesizer index
- `workspace/schedule.yaml` (if exists)
- `context/distribution.yaml`
- `workspace/competitors.yaml` (if exists)
- `workspace/briefs/` — find the most recent file by date
- `workspace/syntheses/` — find the most recent `market-*.md` and `blue-ocean-*.md`

### Step 2 — Render the dashboard

Print the following sections in order.

---

#### Extractors

```
Extractors
──────────────────────────────────────────────────────────
  positioning-extractor       direct, adjacent    active
  pricing-extractor           direct, adjacent    active
  changelog-extractor         direct              active
  review-extractor            direct              planned
  jobs-extractor              direct              planned
  news-extractor              direct, adjacent    planned
  [custom-extractor]          [tiers]             draft    ← TODO: complete before activating
```

Show every row from 0_EXTRACTORS.md. Flag `draft` rows with `← TODO: complete before activating`.

---

#### Synthesizers

```
Synthesizers
──────────────────────────────────────────────────────────
  System (runs automatically with /ci:run — not user-extensible)
    profile-synthesizer       Direction, SWOT, Our Read    system

  Market synthesis (on-demand, via command — user-extensible)
    competitive-differentiation-synthesizer    /ci:differentiation    active
    market-blue-ocean-synthesizer              /ci:blue-ocean         active
    market-synthesizer                         /ci:market             active
    [custom-synthesizer]                       /ci:[name]             draft    ← TODO: complete before activating
```

---

#### Schedule

If `workspace/schedule.yaml` exists:
```
Schedule
──────────────────────────────────────────────────────────
  /ci:run          Every Monday at 9:00am        on
  /ci:market       1st of each month at 9:00am   on
  /ci:blue-ocean   Quarterly (Jan/Apr/Jul/Oct)    off
```

If `workspace/schedule.yaml` does not exist:
```
Schedule
──────────────────────────────────────────────────────────
  Not configured — run /ci:schedule to set up automated runs
```

---

#### Distribution

Read `context/distribution.yaml`. For each target:
```
Distribution
──────────────────────────────────────────────────────────
  Slack     #competitive-intelligence   enabled    not connected (no MCP or webhook)
  Email     you@yourcompany.com         disabled   —
  Notion    —                           disabled   —
```

"Connected" means the required MCP tool or credential would be available in the current session. Check for: `mcp__slack__post_message`, `mcp__email__send_email`, `mcp__notion__create_page`, `SENDGRID_API_KEY`, and `webhook_url` in config.

---

#### Workspace

```
Workspace
──────────────────────────────────────────────────────────
  Competitors        3 tracked   (2 direct, 1 adjacent)
  Last brief         2026-03-19  (today)
  Market synthesis   2026-03-19  (fresh)
  Blue ocean         2026-03-19  (fresh)
```

Staleness thresholds: read the `Stale after` column from `agents/synthesizers/0_SYNTHESIZERS.md` for each active synthesizer. Use each synthesizer's own threshold — do not hardcode. If a file doesn't exist yet: show `never run`.

If `workspace/competitors.yaml` does not exist: show `No competitors configured — run /ci:setup`.

---

#### Attention (only shown if there's something to flag)

Collect any issues found above and surface them once at the bottom:

```
Needs attention
──────────────────────────────────────────────────────────
  ! jobs-extractor is draft — complete TODOs before activating
  ! Slack distribution enabled but not connected
  ! Blue ocean analysis is stale (last run: 2025-12-19)
```

Only show this section if there's at least one item. If everything is healthy, omit it entirely.

---

## Quality Rules

- Read-only — never write, register, or change anything
- If a file is missing, show a graceful fallback for that section rather than erroring
- Stale thresholds must match the orchestrator exactly (30 days market, 90 days blue ocean)
- "Connected" check is best-effort — if the session doesn't expose whether an MCP is active, note `(unknown — check MCP connection)`
