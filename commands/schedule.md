---
name: ci:schedule
description: Configure scheduled runs for any /ci: command. Add, remove, or list scheduled jobs. Uses Claude's built-in session scheduler — no OS setup required. Run without arguments to add a new schedule interactively.
type: command
---

# /ci:schedule — Configure Scheduled Runs

## Role

Add, remove, or list scheduled runs for any `/ci:` command using Claude's built-in cron scheduler. Jobs run automatically while Claude is open — no OS configuration, no API keys, no PATH issues.

**Important:** Jobs are session-bound — they disappear when Claude exits, and auto-expire after 7 days. Re-run `/ci:schedule` to restore them in a new session. The schedule config in `workspace/schedule.yaml` serves as the source of truth to restore from.

---

## Arguments

```
/ci:schedule                          Interactive — show current jobs, then prompt to add/remove
/ci:schedule list                     Show active jobs only
/ci:schedule add /ci:run daily 8:30am Schedule a command
/ci:schedule remove /ci:run           Remove the scheduled job for a command
/ci:schedule clear                    Remove all scheduled jobs
/ci:schedule restore                  Re-register all jobs from workspace/schedule.yaml (use after starting a new session)
```

---

## Steps

### Step 1 — Read current state

Run `CronList` to get all active jobs in this session. Match each job back to a `/ci:` command by inspecting the prompt field.

Read `workspace/schedule.yaml` if it exists — this is the persisted config, used to show what's configured even if jobs aren't currently registered (e.g. after a session restart).

---

### Step 2 — Show current schedule

Always show this first:

```
Scheduled runs
──────────────────────────────────────────────────────────────────────
  Command               Cadence                  Status    Expires
  ────────────────────────────────────────────────────────────────────
  /ci:run               Every day at 8:30am      active    2026-03-27
  /ci:differentiation   Quarterly                inactive  (not registered this session)

Jobs are session-bound — use /ci:schedule restore after starting a new Claude session.
```

**Status:**
- `active` — job is registered and will fire in this session
- `inactive` — in `schedule.yaml` but not registered this session (Claude was restarted)

If nothing is scheduled and no `schedule.yaml` exists:
```
No scheduled runs configured.
```

---

### Step 3 — Act on arguments

**If `list`:** stop after Step 2.

**If `restore`:** read `workspace/schedule.yaml`. For each enabled entry not currently active in CronList, call `CronCreate` to re-register it. Report what was restored. Stop.

**If `add /ci:[name] [cadence]`:** skip to Step 5 with command and cadence pre-filled.

**If `remove /ci:[name]`:** find the matching job ID from CronList, call `CronDelete`, update `workspace/schedule.yaml` to set `enabled: false`. Confirm. Stop.

**If `clear`:** call `CronDelete` for all managed jobs, clear `workspace/schedule.yaml`. Confirm. Stop.

**If no arguments (interactive):** show the schedule (Step 2), then ask:

```
What would you like to do?
  [1] Add or update a scheduled run
  [2] Remove a scheduled run
  [3] Restore all from saved config
  [4] Clear all
  [5] Keep as-is
```

Wait for answer. Route accordingly.

---

### Step 4 — Collect command and cadence (interactive)

Ask which command to schedule:

```
Which /ci: command would you like to schedule?

  Examples:
    /ci:run              Main pipeline — extractors, delta detection, brief (daily or weekly)
    /ci:market           Market synthesis — positioning map, Porter's forces (monthly)
    /ci:blue-ocean       Blue ocean analysis — uncontested space map (quarterly)
    /ci:differentiation  Differentiation strategy — JTBD gaps and positioning rec (quarterly)

  Any /ci: command can be scheduled. Type the command name (with or without /ci:):
```

Then ask when:

```
When should it run? Some examples:

  "every day at 8:30am"          → daily
  "every Monday at 9am"          → weekly
  "every Tuesday and Friday"     → twice a week
  "1st of every month at 9am"    → monthly
  "quarterly"                    → Jan/Apr/Jul/Oct 1st
  "weekdays at 7am"              → Mon–Fri only
  "in 5 minutes"                 → one-time test (fires once, then deletes itself)
```

---

### Step 5 — Convert cadence to cron expression

Convert the natural language cadence to a 5-field cron expression (local time — no timezone conversion needed).

**Avoid :00 and :30 minute marks** unless the user names them exactly — nudge a few minutes to avoid thundering-herd on the API:
- "every day at 8:30am" → `30 8 * * *` (user said exactly 8:30 — use it)
- "every morning around 9" → `57 8 * * *` or `3 9 * * *`
- "daily" with no time given → pick an off-minute like `27 8 * * *`

**Special case — "in N minutes":** calculate the exact fire time from `date`, pin day and month (`MM HH DD Mon *`), set `recurring: false`. Tell the user:
> "One-time test — fires at [time], then deletes itself. Run `/ci:schedule restore` won't include this; it's not saved to `schedule.yaml`."

**Standard cadences:**

| Input | Cron |
|-------|------|
| every day at 8:30am | `30 8 * * *` |
| every Monday at 9am | `0 9 * * 1` |
| every Tuesday and Friday at 8am | `0 8 * * 2,5` |
| weekdays at 7am | `0 7 * * 1-5` |
| 1st of every month at 9am | `0 9 1 * *` |
| quarterly | `0 9 1 1,4,7,10 *` |

If it doesn't map cleanly, show the closest approximation and confirm:
> "Every 2 weeks isn't exact in cron — I'll schedule it on the 1st and 15th of each month. OK?"

---

### Step 6 — Register with CronCreate

Call `CronCreate` with:
- `cron`: the expression from Step 5
- `prompt`: `/ci:[name]`
- `recurring`: `true` (default) — or `false` for one-time test runs

Note the returned job ID.

Remind the user about the 7-day auto-expiry:
> "This job will auto-expire in 7 days. Re-run `/ci:schedule restore` or `/ci:schedule add` before then to keep it running."

---

### Step 7 — Write schedule config

Write or update `workspace/schedule.yaml`. One entry per scheduled command. Skip one-time test entries — those don't belong in the persistent config.

```yaml
# Competitive analysis schedule config
# Source of truth for restoring jobs after a session restart.
# Run /ci:schedule restore to re-register all enabled entries.
# Any /ci: command can be scheduled.

schedules:
  run:
    enabled: true
    cron: "30 8 * * *"
    human_readable: "Every day at 8:30am"

  market:
    enabled: true
    cron: "0 9 1 * *"
    human_readable: "1st of each month at 9:00am"

  differentiation:
    enabled: true
    cron: "0 9 1 1,4,7,10 *"
    human_readable: "Quarterly — Jan/Apr/Jul/Oct 1st at 9:00am"
```

---

### Step 8 — Confirm

Show the updated schedule table (Step 2 format) with status and expiry dates.

---

## Notes

- **Session-bound:** jobs only fire while Claude is open and idle. They disappear on exit.
- **7-day expiry:** recurring jobs auto-delete after 7 days. Re-register with `/ci:schedule restore`.
- **Restore after restart:** run `/ci:schedule restore` at the start of a new session to re-register all saved schedules.
- **`workspace/schedule.yaml` is gitignored** — schedule config is machine-specific.
- **One-time tests:** use `"in 5 minutes"` as the cadence. These are not saved to `schedule.yaml`.
- **To run immediately:** invoke the command directly (`/ci:run`, `/ci:market`, etc.).
