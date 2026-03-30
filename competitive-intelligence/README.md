# Competitive Intelligence

A Claude Code plugin that runs a competitive analysis pipeline — extracts signals from competitor websites, detects changes over time, synthesizes profiles, and delivers briefs. Designed to be run weekly with minimal effort once set up.

---

## Quick start

```
/ci:setup    ← run once to configure your workspace
/ci:run      ← run the pipeline
/ci:market   ← cross-competitor analysis
```

---

## How it works

Each pipeline run:
1. Checks which competitors are **due** based on their tier cadence
2. Runs **extractors** in parallel (positioning, pricing, changelog) to fetch fresh signals
3. Runs **delta detection** — compares the new snapshot against the previous one
4. **Synthesizes** a profile for each competitor (Current State, Direction, SWOT)
5. Composes a **brief** and delivers it to configured channels (Notion, Slack, email)

Data is stored in `workspace/competitor-analysis/`:
- `snapshots/[name]/[date].md` — raw extractor output per run
- `profiles/[name].md` — living profile updated each run
- `briefs/[date].md` — output brief per run
- `syntheses/market-[date].md` — market-level analysis (on-demand)

---

## Commands

### `/ci:setup`
**First-run setup.** Creates your product context file, seeds `competitors.yaml`, and configures the pipeline. Run once before your first `/ci:run`. Interactive — asks questions one at a time.

Can auto-research your product from a URL or take manual answers. Optionally seeds an initial competitor list and configures distribution targets (Slack, email, Notion).

---

### `/ci:run`
**Main pipeline command.** Extracts signals from due competitors, detects deltas, synthesizes profiles, and delivers a brief.

```
/ci:run                          Check all due competitors
/ci:run <name>                   Run one competitor only (e.g. /ci:run Todoist)
/ci:run force_run=true           Run all competitors regardless of schedule
/ci:run deep-dive <name>         Full analysis output instead of brief
/ci:run alert-check              Run all direct-tier competitors immediately
/ci:run --extractor <name>       Run one extractor only, re-synthesize profile
/ci:run --with-market            Full pipeline + market synthesis after
```

---

### `/ci:market`
**Cross-competitor market synthesis.** Reads all competitor profiles and runs the market synthesizer — produces a positioning map, JTBD gaps, whitespace analysis, and strategic implications.

Writes to `workspace/competitor-analysis/syntheses/market-[date].md`. Stale after 30 days (the pipeline will nudge you).

```
/ci:market
```

Run monthly, after a major competitor move, or before a strategy review. Requires at least 2 populated competitor profiles.

---

### `/ci:profile`
**Re-synthesize one competitor's profile** from their most recent snapshot, without re-running extraction. Use after manually editing a snapshot, after adding a new analyzer, or when you want a fresh read without a full pipeline run.

```
/ci:profile <name>    e.g. /ci:profile Todoist
```

---

### `/ci:publish-brief`
**Publish a saved brief** to configured distribution channels (Notion, Slack). Briefs are auto-published at the end of `/ci:run`, but use this to re-send, publish to a new channel, or backfill older briefs.

```
/ci:publish-brief                     Publish the latest brief
/ci:publish-brief --brief 2026-03-19  Publish a specific date
/ci:publish-brief --all               Publish all briefs (oldest first)
```

---

### `/ci:schedule`
**Configure automated runs.** Defaults to [Desktop scheduled tasks](https://code.claude.com/docs/en/scheduled-tasks) — persistent, no expiry, run automatically even when no session is open. Falls back to session-bound jobs if the Desktop app isn't available.

```
/ci:schedule                               Interactive — show current schedule, add/remove
/ci:schedule list                          Show active tasks and jobs
/ci:schedule add /ci:run weekly Monday 9am    Add a Desktop task (persistent, no expiry)
/ci:schedule remove /ci:run                Remove a task or job
/ci:schedule clear                         Remove all
/ci:schedule session /ci:run daily 8:30am  Session-bound only (expires in 3 days)
```

| Type | Requires | Expiry | Best for |
|------|----------|--------|----------|
| **Desktop task** (default) | Claude Desktop app | None | Weekly `/ci:run`, monthly `/ci:market` |
| **Session-bound** | Open Claude session | 3 days | Quick in-session polling |

See the [Claude Code scheduling docs](https://code.claude.com/docs/en/scheduled-tasks) for the full comparison of scheduling options.

---

### `/ci:status`
**Pipeline health dashboard.** Shows what extractors and synthesizers are active, which competitors are tracked, when things last ran, distribution status, and anything needing attention. Read-only.

```
/ci:status
```

---

### `/ci:new`
**Scaffold a new pipeline component.** Generates a file from the right template and registers it in the appropriate index. Starts as `draft` — you activate it after reviewing.

```
/ci:new
```

Choose from:
- **Extractor** — fetches a new signal type per competitor (reviews, job postings, news, etc.)
- **Profile analyzer** — applies a framework per competitor and adds an `## Analysis — [Name]` section to the profile (e.g. JTBD, Porter's per-competitor)
- **Market synthesizer** — cross-competitor analysis on demand (e.g. Blue Ocean, differentiation map)
- **Command** — a new `/ci:` command that runs custom analysis

---

### `/ci:import`
**Import an existing skill** into the pipeline. Accepts a file path, GitHub URL, or pasted content. Detects the type, validates it against the pipeline contract, adapts what doesn't conform, and registers it as `draft`.

```
/ci:import path/to/skill.md
/ci:import https://github.com/.../SKILL.md
/ci:import [paste content inline]
```

---

### `/ci:help`
**Command reference.** Discovers all commands in `commands/` and renders a complete reference — including any custom commands you've added via `/ci:new`.

```
/ci:help
```

---

## Configuration

**`workspace/competitor-analysis/config.yaml`** — pipeline config: workspace path, product name, active analyzers, distribution targets.

**`workspace/competitor-analysis/competitors.yaml`** — competitor registry: name, website, tier, last_updated, changelog_url, pricing_url.

**`workspace/competitor-analysis/{product_name}.md`** — your product context: what you're building, who it's for, differentiators, what you're not. Used to calibrate SWOT and threat assessments.

**`context/signal-weights.md`** — weights used by the delta detector to prioritize changes (high / medium-high / medium / low-medium / low).

---

## Extending the pipeline

All components are discovered from index files — add a row and the orchestrator picks it up automatically.

| To add... | Use | Registered in |
|-----------|-----|---------------|
| A new data source | `/ci:new` → Extractor | `agents/extractors/0_EXTRACTORS.md` |
| A new per-competitor analysis framework | `/ci:new` → Profile analyzer | `agents/analyzers/0_ANALYZERS.md` + `config.yaml` |
| A new cross-competitor analysis | `/ci:new` → Market synthesizer | `agents/synthesizers/0_SYNTHESIZERS.md` |
| A new command | `/ci:new` → Command | auto-discovered from `commands/` |

New components start as `status: draft`. Review the generated file, complete any `TODO:` items, then change status to `active`.
