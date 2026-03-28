---
name: ci:schedule
description: Configure scheduled runs for any /ci: command. Defaults to Desktop scheduled tasks (persistent, no expiry). Falls back to session-bound CronCreate if Desktop is unavailable. Run without arguments to set up interactively.
type: command
---

# /ci:schedule — Configure Scheduled Runs

## Role

Create, list, or remove scheduled runs for any `/ci:` command. Desktop scheduled tasks are the default — they persist across restarts, have no expiry, and run automatically on your machine even when no Claude session is open. Session-bound scheduling (CronCreate) is available as a fallback for quick in-session use.

---

## Scheduling options

| | Desktop task | Session-bound (`/loop`) |
|---|---|---|
| Persists across restarts | ✓ yes | ✗ no |
| Requires open session | no | yes |
| Expiry | none | 3 days |
| Access to local files | yes | yes |
| Min interval | 1 minute | 1 minute |

**Use Desktop tasks** for weekly `/ci:run` and monthly `/ci:market` automation — the primary use case.

**Use session-bound** only for quick in-session polling (e.g. "check back on this in 30 minutes").

---

## Arguments

```
/ci:schedule                          Interactive — show current schedule, prompt to add/remove
/ci:schedule list                     Show active Desktop tasks and session-bound jobs
/ci:schedule add /ci:run weekly Monday 9am    Schedule a Desktop task
/ci:schedule remove /ci:run           Remove Desktop task (or session-bound job) for a command
/ci:schedule clear                    Remove all managed tasks and jobs
/ci:schedule session /ci:run daily 8:30am     Session-bound only — explicitly use CronCreate
```

---

## Steps

### Step 1 — Read current state

`{workspace_root}` and `{product_name}` are available from context (set in `AGENTS.md` by `/ci:setup`).

Ask Claude to list all Desktop scheduled tasks by saying: "List my scheduled tasks."

Also run `CronList` to get any active session-bound jobs.

Read `{workspace_root}/schedule.yaml` if it exists — used to track what this plugin has registered.

---

### Step 2 — Show current schedule

Always show this first:

```
Scheduled runs
──────────────────────────────────────────────────────────────────────
  Command               Cadence                  Type       Status
  ────────────────────────────────────────────────────────────────────
  /ci:run               Every Monday at 9:00am   Desktop    active
  /ci:market            1st of each month 9am    Desktop    active
  /ci:run               Every day at 8:30am      Session    active (expires in 2 days)
```

**Type column:**
- `Desktop` — persistent task managed by the Desktop app. No expiry.
- `Session` — session-bound CronCreate job. Expires after 3 days. Disappears on restart.

If nothing is scheduled:
```
No scheduled runs configured.
Run /ci:schedule to set one up.
```

---

### Step 3 — Act on arguments

**If `list`:** stop after Step 2.

**If `add /ci:[name] [cadence]`:** skip to Step 5 with command and cadence pre-filled.

**If `remove /ci:[name]`:**
- Find matching Desktop task and ask Claude to delete it: "Delete the [name] scheduled task."
- Also find matching session-bound job from CronList and call `CronDelete` if present.
- Update `{workspace_root}/schedule.yaml` to mark the entry removed.
- Confirm. Stop.

**If `clear`:**
- Ask Claude to "Delete all competitive intelligence scheduled tasks" (Desktop).
- Call `CronDelete` for all session-bound CI jobs.
- Clear `{workspace_root}/schedule.yaml`.
- Confirm. Stop.

**If `session /ci:[name] [cadence]`:** skip to Step 6 (session-bound path, skipping Desktop).

**If no arguments (interactive):** show the schedule (Step 2), then ask:

```
What would you like to do?
  [1] Add a scheduled run (Desktop — persistent, recommended)
  [2] Add a quick session run (session-bound, expires in 3 days)
  [3] Remove a scheduled run
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
    /ci:run              Main pipeline — extractors, delta detection, brief (weekly)
    /ci:market           Market synthesis — positioning map, JTBD gaps (monthly)

  Any /ci: command can be scheduled. Type the command name (with or without /ci:):
```

Then ask when:

```
When should it run?

  "every Monday at 9am"          → weekly
  "every day at 8:30am"          → daily
  "1st of every month at 9am"    → monthly
  "quarterly"                    → Jan/Apr/Jul/Oct 1st
  "weekdays at 7am"              → Mon–Fri only
```

---

### Step 5 — Create Desktop scheduled task (default path)

Ask Claude to create a Desktop scheduled task using natural language:

> "Create a scheduled task called 'ci-[command-name]' with the prompt `/ci:[name]` to run [cadence]. Use the current project folder."

For example:
> "Create a scheduled task called 'ci-run' with the prompt `/ci:run` to run every Monday at 9am. Use the current project folder."

If Claude confirms the task was created:
- Write or update `{workspace_root}/schedule.yaml` (see Step 7 format).
- Go to Step 8.

**If Desktop task creation fails** (e.g. running in CLI without Desktop app, or app not available):

Tell the user:
> "Desktop task creation requires the Claude Desktop app. Falling back to session-bound scheduling — this job will expire in 3 days and requires Claude to be open."

Then follow Step 6 (session-bound path).

---

### Step 6 — Session-bound fallback (CronCreate)

Convert the cadence to a 5-field cron expression.

**Avoid :00 and :30 minute marks** unless the user named them exactly — nudge a few minutes off to avoid thundering-herd on the API:
- "every Monday at 9am" → `0 9 * * 1` (user said exactly 9am — use it)
- "daily" with no time given → pick an off-minute like `27 8 * * *`

**Standard cadences:**

| Input | Cron |
|-------|------|
| every Monday at 9am | `0 9 * * 1` |
| every day at 8:30am | `30 8 * * *` |
| weekdays at 7am | `0 7 * * 1-5` |
| 1st of every month at 9am | `0 9 1 * *` |
| quarterly | `0 9 1 1,4,7,10 *` |

Call `CronCreate` with:
- `cron`: the expression
- `prompt`: `/ci:[name]`
- `recurring`: `true`

Note the returned job ID.

Warn the user:
> "Session-bound job created. **Expires in 3 days** and requires Claude Code to be open. To set up a persistent schedule that runs automatically, open the Claude Desktop app and run `/ci:schedule add /ci:[name] [cadence]`."

Write or update `{workspace_root}/schedule.yaml` marking type as `session`.

---

### Step 7 — Write schedule config

Write or update `{workspace_root}/schedule.yaml`:

```yaml
# Competitive analysis schedule config
# Managed by /ci:schedule. Edit via /ci:schedule commands, not directly.

schedules:
  run:
    type: desktop          # desktop | session
    enabled: true
    cadence: "Every Monday at 9:00am"

  market:
    type: desktop
    enabled: true
    cadence: "1st of each month at 9:00am"
```

For session-bound entries, include the `cron` expression (useful for re-creating later):

```yaml
  run:
    type: session
    enabled: true
    cadence: "Every day at 8:30am"
    cron: "30 8 * * *"
    expires: "3 days from creation"
```

---

### Step 8 — Confirm and show permission note

Show the updated schedule table (Step 2 format).

For **Desktop tasks**, add this note the first time:

> "Desktop task created. To avoid permission prompts during automated runs:
> 1. Open the Claude Desktop app → Schedule page
> 2. Find the task and click **Run now** to do a test run
> 3. When prompted for permissions, select **Always allow**
>
> Future runs will be fully automated."

---

## Notes

- **Desktop tasks have no expiry** — they persist until you delete them from the Desktop app or via `/ci:schedule remove`.
- **Session-bound jobs expire in 3 days** — use these only for temporary in-session polling.
- **`{workspace_root}/schedule.yaml` is gitignored** — schedule config is machine-specific.
- **Desktop task prompts live at** `~/.claude/scheduled-tasks/<task-name>/SKILL.md` — you can edit the prompt directly there; schedule and folder are managed via the Desktop app.
- **To run immediately:** invoke the command directly (`/ci:run`, `/ci:market`, etc.).
