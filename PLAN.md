# Plugin Plan

## Design

End-to-end competitive intelligence pipeline that tracks a defined set of competitors, detects meaningful changes over time, and delivers a weekly brief.

**Key components:**
- `agents/orchestrator.md` — entry point; reads competitor registry, runs extractors in parallel, triggers delta detection and synthesis, delivers brief
- `skills/delta-detector.md` — compares new snapshot against previous, produces a weighted change list ordered by signal importance
- `skills/brief-composer.md` — formats delta list + synthesis into a weekly brief saved to `workspace/briefs/[YYYY-MM-DD].md`
- `skills/create-skill.md` — lets users define new competitive analysis skills that plug into the pipeline automatically

**Infrastructure:**
- Web search MCP for live data extraction
- Scheduled runs (cron) for automated weekly cadence
- Snapshot store in `workspace/snapshots/` for delta comparison
- Competitor profiles in `workspace/profiles/`

**Open questions:**
- Competitor registry format — YAML file, Notion DB, or inline config?
- Snapshot schema — per-competitor markdown or structured JSON?
- Scoring model for delta weighting (recency, category, magnitude)
- Output distribution — file only, or also email/Slack?

## Tasks
- [ ] Define competitor registry format
- [ ] Write `agents/orchestrator.md`
- [ ] Write `skills/delta-detector.md`
- [ ] Write `skills/brief-composer.md`
- [ ] Write `skills/create-skill.md`
- [ ] Define snapshot schema
- [ ] Add web search MCP configuration
- [ ] Set up scheduled run hooks
- [ ] Write at least one extractor skill (e.g. `skills/extractors/website.md`)
- [ ] Test end-to-end with one competitor

## Done
