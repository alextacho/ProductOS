---
name: ci:publish-brief
description: Publish a saved brief to configured distribution channels (Slack, email, Notion). Defaults to the latest brief. Use --brief <date> for a specific brief or --all to publish all unpublished briefs. Use --yes to skip all prompts (for automated/ci:run use). Prompts for channel selection and confirmation before sending.
type: command
---

# /ci:publish-brief — Publish Brief

## Role

Take a saved brief from `{workspace_root}/briefs/` and dispatch it to one or more distribution channels. This command lets you publish on demand, independently of `/ci:run`. It does not re-run analysis — it only sends what's already written.

---

## Arguments

| Argument | Values | Default | Notes |
|----------|--------|---------|-------|
| `--brief <date>` | `YYYY-MM-DD` | most recent brief | Publish a specific brief by date |
| `--all` | flag | off | Publish all briefs in `{workspace_root}/briefs/` |
| `--yes` | flag | off | Skip channel-select and confirmation prompts; publish to all ready channels automatically |

**Parsing rules:**
- No arguments → publish the most recent brief
- `--brief 2026-03-19` → publish `{workspace_root}/briefs/2026-03-19.md`
- `--all` → publish every `.md` file in `{workspace_root}/briefs/`, oldest first
- `--all` and `--brief` together → `--all` takes precedence
- `--yes` → non-interactive mode; skip Steps 4 and 5, publish to all `ready` channels

---

## Steps

### Step 1 — Resolve target brief(s)

`{workspace_root}` and `{product_name}` are available from context (set in `AGENTS.md` by `/ci:setup`).

List all `.md` files in `{workspace_root}/briefs/`. Sort by filename (date) descending.

**If no briefs exist:** abort with:
> No briefs found in `{workspace_root}/briefs/`. Run `/ci:run` first to generate one.

**If `--brief <date>`:** find `{workspace_root}/briefs/<date>.md`. If not found, abort with:
> Brief `{workspace_root}/briefs/<date>.md` not found. Available briefs:
> [list filenames, most recent first]

**If `--all`:** collect all brief files, sorted oldest first (so they land in channels in chronological order).

**Default (no args):** use the file with the most recent date in its filename.

---

### Step 2 — Read distribution config

Read `{workspace_root}/config.yaml` and extract the `distribution:` block.

For each target, determine its state:

| State | Condition |
|-------|-----------|
| `ready` | `enabled: true` AND required MCP/credential is available |
| `enabled-not-connected` | `enabled: true` AND MCP/credential not available |
| `disabled` | `enabled: false` |

**Availability checks:**
- Slack → `mcp__slack__post_message` available, OR `webhook_url` is non-empty in config
- Email → `mcp__email__send_email` available, OR `SENDGRID_API_KEY` env var set
- Notion → `mcp__notion__create_page` (or equivalent write tool) available

If no targets are `ready`: abort with:
> No connected distribution channels found.
>
> Configured but not connected: [list targets with their missing requirement]
> Disabled: [list disabled targets]
>
> Set up a channel in `{workspace_root}/config.yaml` (under `distribution:`) or connect the required MCP tool, then re-run `/ci:publish-brief`.

---

### Step 3 — Show what will be published

Print a confirmation preview before asking anything:

```
Publishing:
──────────────────────────────────────────────────────────
  Brief(s):   {workspace_root}/briefs/2026-03-19.md
  Format:     full

Available channels:
  ✓ Notion    ready      (parent page: 3296f6ca...)
  ~ Slack     not connected (no MCP or webhook)
  — Email     disabled
```

For `--all`, list each brief filename in the "Brief(s)" row.

If `--yes` is set, append `(non-interactive — publishing to all ready channels)` to the preview and skip Steps 4 and 5.

---

### Step 4 — Ask: which channels?

**Skip this step if `--yes` is set** — use all `ready` channels.

Present only the `ready` channels as selectable options. If only one channel is ready, skip this prompt and proceed with that channel (note it in the confirmation).

> Publish to which channels?
> [multi-select from ready channels]

---

### Step 5 — Confirm

**Skip this step if `--yes` is set** — proceed directly to Step 6.

Show a single confirmation line and ask yes/no:

```
Publish [N brief(s)] to [channel list]?
```

For `--all` with multiple briefs, show the count. If the user says no, abort with "Cancelled — no briefs were sent."

---

### Step 6 — Dispatch

For each brief (in order if `--all`):

1. Read the brief file content
2. Extract the TL;DR / summary section (first `## TL;DR` or `## Summary` block, up to ~150 words) as `brief_summary`
3. Use full file content as `brief_full`
4. Invoke `brief-distributor` with:
   - `workspace_root`: the resolved `{workspace_root}` value
   - `brief_path`: path to the brief file
   - `run_date`: date from the filename
   - `brief_summary`: extracted summary
   - `brief_full`: full content
   - `channel_filter`: the channels selected in Step 4 (pass as a list — distributor should skip channels not in this list)

If dispatching multiple briefs, send them sequentially. Report each result before moving to the next.

---

### Step 7 — Report results

After all dispatches complete, print a summary:

```
Published:
──────────────────────────────────────────────────────────
  2026-03-19.md    → Notion ✓
  2026-03-18.md    → Notion ✓
```

If any dispatch failed:
```
  2026-03-17.md    → Notion ✗  (error: page not found)
```

Failed dispatches do not block others. Always report what succeeded and what didn't.

---

## When to use

- After `/ci:run` ran automatically (via cron) and you want to push the brief now
- To re-send a brief to a new channel after adding it to distribution config
- To backfill older briefs into Notion when setting up a new workspace
- When distribution failed during `/ci:run` and you want to retry manually
