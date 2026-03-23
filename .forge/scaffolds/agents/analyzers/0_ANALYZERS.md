# Analyzer Index

Analyzers apply an analytical framework to a competitor's snapshot and write a named `## Analysis — [Framework]` section to the profile. Multiple analyzers can be active simultaneously — each owns its own section and runs independently.

Analyzers are configured in `workspace/config.yaml` under `profile_analyzers`. The orchestrator runs all active analyzers in parallel after the profile builder completes.

To add a new analyzer: run `/ci:new` and choose "Profile analyzer," or copy `TEMPLATE.md`, fill it in, add a row here, and add its name to `workspace/config.yaml`.

| Analyzer | Profile Section | Inputs | Status |
|----------|----------------|--------|--------|
| [swot-analyzer.md](./swot-analyzer.md) | `## Analysis — SWOT` | snapshot, existing profile section | active |

_The orchestrator only runs analyzers listed in `workspace/config.yaml` under `profile_analyzers` AND with `status: active` here. Both must be true._

---

## Analyzer contract (invariants all analyzers must follow)

1. **Own exactly one section.** Each analyzer writes to one `## Analysis — [Name]` section. Never write to another analyzer's section or to Current State / Direction.
2. **Read from the snapshot.** All inputs come from the snapshot. Do not fetch from the web.
3. **Manage your own merge rules.** An analyzer is responsible for deciding what to rewrite vs. carry forward within its section. Use source annotations (`_(src: ...)_`) to track which snapshot sections informed each finding.
4. **Be self-contained.** Another analyzer being absent or failing must not affect your output.
5. **Surface missing data inline.** If a finding would be stronger with data from an inactive extractor, note it within the relevant part of your output — not as a top-level error.
