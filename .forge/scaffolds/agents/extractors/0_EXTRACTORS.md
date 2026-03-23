# Extractor Index

Each extractor owns one section of the run snapshot. The orchestrator spawns all applicable extractors in parallel for each competitor, and each extractor writes its section directly to the new snapshot file.

To add a new extractor: copy `TEMPLATE.md`, fill it in, add a row here, wire into the orchestrator.

| Extractor | Snapshot Section | Sources | Tier Scope | Status |
|-----------|----------------|---------|------------|--------|
| [positioning-extractor.md](./positioning-extractor.md) | `## Positioning` | Homepage, About | direct, adjacent | active |
| [changelog-extractor.md](./changelog-extractor.md) | `## Product Direction` | Changelog, release blog, GitHub releases | direct | active |

_The orchestrator only spawns extractors with `status: active`. Set to `active` once an extractor has been tested end-to-end against a real competitor._

## Extractor contract (invariants all extractors must follow)

1. **Return data, never interpretation.** "They emphasize collaboration" is data. "This suggests they're targeting teams" is interpretation — that's synthesis's job.
2. **Return the snapshot section verbatim.** Output maps exactly to the markdown section the extractor owns. The orchestrator writes it directly into the new snapshot file — not the profile.
3. **Never omit a field.** If unknown, write `unknown`. If uncertain, append `(unconfirmed)`.
4. **Flag instead of guess.** When sources conflict or are unavailable, flag it inline. Don't pick one and stay silent.
5. **Stay in your lane.** If you encounter data that belongs to another extractor's section, ignore it.
