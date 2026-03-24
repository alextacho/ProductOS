# Signal Weights

Used by the delta-detector to assign importance to each type of change detected in a competitor profile.

The delta detector reads this file to look up the weight for each change it detects. **Do not assign weights from memory — always reference this file.**

---

## Weight Scale

| Weight | Meaning | Action |
|--------|---------|--------|
| `high` | Significant strategic change — surfaces immediately in brief and alerts | Always surface; leads the change list |
| `medium-high` | Strong signal worth tracking — high confidence it reflects a real strategic shift | Surface in brief; explain briefly |
| `medium` | Notable change — worth knowing but not urgent | Surface in brief; no elaboration needed |
| `low-medium` | Possible signal — could be meaningful in context | Include in brief only if part of a pattern |
| `low` | Weak signal — often noise | Omit from brief unless asked; track in changelog |

---

## Weights by Signal Type

### Pricing signals

| Change | Weight | Notes |
|--------|--------|-------|
| Free tier removed | `high` | Monetization strategy shift — direct competitive pressure on freemium tools |
| Pricing increased | `high` | Signals confidence or competitive repositioning |
| Pricing decreased | `high` | Signals competitive pressure or market share push |
| New tier added (fills a gap) | `medium-high` | New segment being targeted |
| Tier renamed | `low` | Usually cosmetic; check if limits changed |
| Pricing model changed (e.g., per-seat → usage-based) | `high` | Fundamental go-to-market shift |
| Trial period changed | `medium` | Conversion funnel change |
| Limits changed within existing tier | `medium` | Feature gating shift; watch direction |

### Positioning signals

| Change | Weight | Notes |
|--------|--------|-------|
| Core message / hero headline changed | `medium` | Messaging pivots take months to confirm; track direction over time |
| Target audience changed or expanded | `medium` | New segment pursuit |
| New differentiator claimed | `medium` | Check if backed by product; often aspirational |
| Differentiator removed | `medium` | May signal they've abandoned a bet |
| Tone shift (e.g., startup → enterprise) | `medium` | Often lags product/GTM shift by months |

### Product signals

| Change | Weight | Notes |
|--------|--------|-------|
| Major new feature announced or launched | `high` | Capability gap closing or opening |
| New product line or separate product | `high` | Category expansion |
| Feature deprecated or removed | `medium-high` | Strategic retreat; potential opening |
| New integration added | `medium` | Ecosystem expansion |
| Platform added (e.g., mobile, desktop) | `medium` | Reach expansion |
| Minor product update / changelog entry | `low` | Normal product iteration |

### Hiring signals

| Change | Weight | Notes |
|--------|--------|-------|
| Hiring in a new function (no prior openings) | `medium-high` | Investment signal — lags ~6 months to impact |
| Hiring accelerating in an existing function | `medium` | Scaling a known bet |
| Hiring in a specific technical area (e.g., ML, security) | `medium-high` | Signals capability build |
| Hiring slowdown or layoffs (if detectable) | `medium-high` | Runway or strategic contraction signal |
| Executive hire (C-suite, VP) | `medium` | Leadership/strategy change; watch for 90-day moves |

### Funding signals

| Change | Weight | Notes |
|--------|--------|-------|
| New funding round | `medium-high` | Signals runway, aggression, and growth bets |
| Acquisition (acquired or acqui-hired) | `high` | Major strategic change |
| IPO announced or filed | `high` | Significant behavioral change likely |

### Review / customer signals

| Change | Weight | Notes |
|--------|--------|-------|
| Review score direction change (improving trend) | `medium` | Customer satisfaction improving — weakness closing |
| Review score direction change (declining trend) | `medium` | Customer satisfaction declining — potential opening |
| New recurring complaint theme emerges | `medium` | Structural weakness surfacing |
| Previously recurring complaint disappears | `medium` | They fixed something; exploit window closing |
| Notable customer lost (if detectable) | `medium-high` | Churn signal |
| New enterprise logo added | `low-medium` | GTM signal |

### News / strategic signals

| Change | Weight | Notes |
|--------|--------|-------|
| Partnership announced | `medium` | Ecosystem or distribution bet |
| New market expansion announced | `medium-high` | Competitive territory shift |
| Exec departure (C-suite, VP) | `medium` | Instability or strategy change |
| Press coverage of a new direction | `low-medium` | Often forward-looking; wait for product evidence |
| Blog post or content published | `low` | Usually marketing; occasionally signals focus |
| Award or analyst recognition | `low` | Vanity metric; ignore unless context is relevant |

---

## Compound Signals

When multiple changes in the same direction appear in one run, treat them as a compound signal and consider raising the weight by one level.

Examples:
- Pricing increase + free tier removed in same run → `high` (not just `high` + `high` — they're one signal)
- New ML hiring + AI feature announced in same run → compound signal; synthesizer should connect them
- Review score declining + new complaint theme in same run → treat as pattern, not two separate `medium` signals

The delta detector notes compound signals in the changelog entry. The synthesizer interprets them.
