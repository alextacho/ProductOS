# Market Synthesizer Index

Market synthesizers read **all** competitor profiles and produce market-level analysis documents. They are not run automatically by `/ci:run` — use `/ci:market` to run on demand, or `/ci:run --with-market` to run after a full pipeline cycle.

To add a new synthesizer: run `/ci:new` and choose "Market synthesizer."

| Synthesizer | Owns | Frameworks | Command | Stale after | Status |
|-------------|------|------------|---------|-------------|--------|
| [market-synthesizer.md](./market-synthesizer.md) | `{workspace_root}/syntheses/market-[date].md` | competitive positioning map, jobs-to-be-done, value curve, differentiation mapping | `/ci:market` | 30 days | active |

---

## What market synthesizers do vs. don't do

**Do:**
- Read across all competitor profiles
- Produce interpretation, implications, strategic reads at the market level
- Apply frameworks to structure the analysis
- Flag uncertainty and conflicting signals

**Don't:**
- Fetch anything from the web — that's extraction
- Write to individual competitor profiles — that's the profile synthesizer
- Reproduce raw facts already in the profiles — summarize and interpret only

---

## Adding a new market synthesizer

Run `/ci:new` and choose "Market synthesizer." All user-created synthesizers are Stage 2 — cross-competitor, on-demand, each gets its own `/ci:` command.
