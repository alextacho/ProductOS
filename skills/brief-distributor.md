---
name: brief-distributor
description: Reads context/distribution.yaml and dispatches the completed brief to configured targets (Slack, email, Notion). When no targets are connected, returns a user-facing message prompting setup. Called by the orchestrator after the brief-composer completes.
layer: system
runs: once, after brief-composer
---

# Brief Distributor

## Role

Read the distribution config and dispatch the brief to configured targets. This skill is a routing layer — it does not format the brief, it does not decide what to send. It reads what's configured, checks what's available, fires or skips accordingly, and always tells the user what happened.

Distribution is best-effort. A failed or unavailable target never blocks the brief from being returned to the user.

---

## Inputs

| Input | Type | Required | Notes |
|-------|------|----------|-------|
| `brief_path` | string | yes | Path to the brief file: `workspace/briefs/[run_date].md` |
| `run_date` | string (YYYY-MM-DD) | yes | Used in subject lines and page titles |
| `brief_summary` | string | yes | The TL;DR from the brief (~150 words). Used for targets with `format: summary`. |
| `brief_full` | string | yes | The full brief content. Used for targets with `format: full`. |
| `channel_filter` | list of strings | no | If provided, dispatch only to channels in this list (e.g. `["notion", "slack"]`). If omitted, dispatch to all enabled targets as usual. Used by `/ci:publish-brief` for selective publishing. |

---

## Process

### Step 1 — Read config

Read `context/distribution.yaml`.

If `channel_filter` is provided, restrict the active target set to only the channels named in the list. Targets not in `channel_filter` are treated as if `enabled: false` for this run — do not dispatch to them and do not mention them in the result note.

Determine the state (after applying any `channel_filter`):
- **None enabled:** all targets have `enabled: false` (or were filtered out)
- **Enabled, needs setup:** at least one target is enabled but required MCP/credential is not available
- **Ready:** at least one target is enabled and its MCP/credential is available

### Step 2 — Determine content per target

For each enabled target, select content based on its `format` field:
- `summary`: use `brief_summary`
- `full`: use `brief_full`

### Step 3 — Dispatch to each enabled target

Process each enabled target independently. A failure on one target does not affect others.

---

#### Slack

**Check availability (in order):**
1. `mcp__slack__post_message` available → use it
2. `webhook_url` set in config → POST via bash curl
3. Neither → `skipped`

**If MCP available:**
```
mcp__slack__post_message:
  channel: config.slack.channel
  text: [content per format]
```

**If webhook_url set (fallback):**
```
POST config.slack.webhook_url
  { "text": "[content per format]" }
```

Record: `{ target: "slack", status: "sent" | "failed" | "skipped", detail: ... }`

---

#### Email

**Check availability (in order):**
1. `mcp__email__send_email` available → use it
2. `SENDGRID_API_KEY` env var set → POST to SendGrid API
3. Neither → `skipped`

**If MCP available:**
```
mcp__email__send_email:
  to: config.email.to
  subject: "[config.email.subject_prefix] [run_date]"
  body: [content per format]
```

**If SENDGRID_API_KEY set (fallback):**
```
POST https://api.sendgrid.com/v3/mail/send
  Authorization: Bearer [SENDGRID_API_KEY]
  { from, to, subject, content }
```

Record: `{ target: "email", status: "sent" | "failed" | "skipped", detail: ... }`

---

#### Notion

**Check availability:**
1. `mcp__notion__create_page` (or equivalent write tool) available → use it
2. Not available → `skipped`

**If MCP available:**
```
mcp__notion__create_page:
  parent: { page_id: config.notion.parent_page_id }
  properties:
    title: "Competitive Brief — [run_date]"
  children: [content per format as Notion blocks]
```

Record: `{ target: "notion", status: "sent" | "failed" | "skipped", detail: ... }`

---

### Step 4 — Build the distribution note

Based on the results, return the appropriate message for the orchestrator to append to the brief.

**Case A — Nothing enabled (all disabled):**

```
📬 **Brief ready to send.** Set up delivery targets in `context/distribution.yaml` to publish to Slack, email, or Notion.
```

**Case B — Enabled but not connected (all skipped, none sent):**

```
📬 **Brief ready to send.** Connections not active for: [list of skipped targets].
   Connect the required MCP tools or credentials — see `context/distribution.yaml`.
```

List the tool or credential required per skipped target:
- Slack → `mcp__slack` or `webhook_url` in config
- Email → `mcp__email` or `SENDGRID_API_KEY` env var
- Notion → `mcp__notion`

**Case C — At least one sent, some skipped:**

```
_Distributed to: [sent targets]. Not sent: [skipped targets] — connections not active._
```

**Case D — All sent:**

```
_Distributed to: [comma-separated list of targets]._
```

**Case E — A target failed (MCP call returned error):**

```
_[Target] distribution failed: [brief error]. Brief saved to workspace/briefs/[run_date].md._
```

---

### Step 5 — Return result

Return a structured result to the orchestrator:

```
{
  results: [
    { target: "slack",  status: "sent" | "failed" | "skipped", detail: "..." },
    { target: "email",  status: "sent" | "failed" | "skipped", detail: "..." },
    { target: "notion", status: "sent" | "failed" | "skipped", detail: "..." }
  ],
  note: "[the distribution note string from Step 4 — orchestrator appends this to the brief]"
}
```

---

## Quality Rules

- Always return a note — even if nothing was configured. The user should know their brief is ready and how to distribute it.
- Distribution failure never silences the brief. The brief is returned regardless.
- Do not retry failed dispatches. Log and move on.
- Case A and Case B messages use 📬 to be visible without being alarming. Cases C/D use plain italic text — distribution succeeded, no noise needed.

---

## Out of Scope

- **Formatting the brief** — brief-composer owns that; this skill receives formatted content
- **Deciding what's significant enough to distribute** — all completed briefs are dispatched
- **Managing distribution config** — the user owns `context/distribution.yaml`; this skill reads it
