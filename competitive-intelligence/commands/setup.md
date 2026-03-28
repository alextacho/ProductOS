---
name: ci:setup
description: Interactive first-run setup for the competitive analysis plugin. Creates a product context file, seeds {workspace_root}/competitors.yaml, and configures the pipeline. Run this before your first /ci:run.
type: command
---

# /ci:setup — Setup the Project Structure

## Role

Walk the user through creating the required context files for the competitive analysis plugin. Check what already exists, only ask for what's missing, and write well-structured files that the pipeline can use immediately.

---

## Steps

---

### Step 1 — Check what exists

Read the `<!-- ci:config:start -->` block in `AGENTS.md` to resolve `{workspace_root}` and `{product_name}`. If the values are empty or the block is not yet populated, use defaults (`workspace/competitor-analysis` and `""` respectively) — Step 2 will confirm or change these.

Check for each required and optional file under `{workspace_root}`:

| File | Required |
|------|----------|
| `{workspace_root}/config.yaml` | yes |
| `{workspace_root}/{product_name}.md` | yes (if product_name is set) |
| `{workspace_root}/competitors.yaml` | optional |

For each file that already exists, skip its section and note it's already set up.

If all required files exist: say "You're already set up. Run /ci:run to start your first competitive analysis run." and stop.

---

### Step 2 — Configure workspace

Ask two questions, one at a time:

**2a — Product name:**
> "What's the name of your product?"

Use the answer as `{product_name}`. This names your product context file: `{product_name}.md`.

**2b — Workspace location:**
> "Where should competitive analysis data live?
>
> Default: `workspace/competitor-analysis`
>
> Type "approve", or type a path relative to the project root."

Strip any leading `./` or trailing `/` from the path.

Write the resolved values into the `<!-- ci:config:start -->` block in `AGENTS.md`:

```
workspace_root: [chosen path]
product_name: [product name]
```

This is the single source of truth for `{workspace_root}` and `{product_name}`. All commands and agents read these values from context at session start — no per-command config reading required.

Also copy `templates/config.TEMPLATE.yaml` to `{workspace_root}/config.yaml` and set `workspace_root` and `product_name` there too (used for distribution config and pipeline settings — not for path resolution).

Create `{workspace_root}/` if it doesn't already exist.

Copy `context/signal-weights.md` → `{workspace_root}/context/signal-weights.md`.

Do not overwrite any destination files that already exist.

If the `AGENTS.md` config block already has `workspace_root` and `product_name` set: skip this step and use the existing values.

---

### Step 2b — Choose setup mode (if any required files are missing)

Ask the user:

> "To create your product and positioning context, I can either:
>
> **[1] Analyse your product** — give me your product URL (or a few URLs) and I'll research your messaging, positioning, and differentiators, then draft the files for you to review and edit.
>
> **[2] Manual entry** — I'll ask you a few questions and write the files from your answers.
>
> Which would you prefer?"

---

#### Mode 1 — Analysis run

Ask:

> "What URL(s) should I start from? (e.g. your homepage, about page, or pricing page — paste one or more)"

For each URL the user provides, fetch the page and extract:
- What the product does
- Who it's for
- Key differentiators and positioning claims
- Tone and language patterns
- Any explicit "not for" or scope statements

Then draft `{workspace_root}/{product_name}.md` (see format in Step 3 below) using the extracted content. Where information wasn't found, mark the field `(not found — please fill in)`.

Show the user a preview of what you'll write and ask:

> "Here's what I found. Does this look right, or would you like to adjust anything before I write the files?"

Incorporate any corrections, then write the files and skip to **Step 5**.

---

#### Mode 2 — Manual entry

Continue to Step 3.

---

### Step 3 — Create {workspace_root}/{product_name}.md (if missing, manual mode)

Tell the user:

> "I'll ask you a few questions to set up your product context."

Ask these questions **one at a time**, waiting for a response before continuing:

1. **What does your product do?** (1–3 sentences — what it is and what problem it solves)
2. **Who is it for?** (Be specific — role, context, or the moment they reach for your product)
3. **What are your key differentiators?** (2–4 things — use your own language, the way you'd say it in a pitch or marketing copy)
4. **What are you not?** (1–2 explicit scope boundaries — helps frame threat assessments correctly)
5. **What's your core message — your one-liner?** (Optional — skip if you don't have one yet)

Once you have responses, write `{workspace_root}/{product_name}.md`:

```markdown
# {product_name}

_Read by the pipeline to calibrate competitor analysis and positioning comparisons._

---

## What We're Building

[Answer to question 1]

## Who It's For

[Answer to question 2]

## Differentiators

[Answer to question 3 — formatted as bullet points, their exact language]

## What We're Not

[Answer to question 4 — formatted as bullet points]

## Core Message

[Answer to question 5, or omit this section if skipped]
```

Confirm the file was written.

---

### Step 5 — Seed {workspace_root}/competitors.yaml (if missing, optional)

Ask:

> "Do you want to add your first competitors now? You can always add more later. (yes / skip)"

If yes:

Tell the user:

> "Tell me your competitors — for each one, give me the name and website. I'll ask about tier after."

For each competitor the user names, ask:

> "Is [name] a **direct** competitor (same buyer, same job), **adjacent** (overlapping use cases but different focus), or **aspirational** (a market leader you're watching)?"

Once you have the list, write `{workspace_root}/competitors.yaml`:

```yaml
competitors:
  - name: [Name]
    website: [https://...]
    tier: direct | adjacent | aspirational
    last_updated: null  # will be set after first run
    changelog_url: null  # set once discovered — extractor reads this first, skips discovery if present
```

Add one entry per competitor. Confirm the file was written.

If skip: note that they can add competitors later by editing `{workspace_root}/competitors.yaml` directly.

---

### Step 6 — Configure distribution (optional)

Tell the user:

> "Briefs can be automatically sent to Slack, email, or Notion after each run. Would you like to configure a delivery target now? (yes / skip)"

If yes:

Ask which target they want to set up:
> "[1] Slack  [2] Email  [3] Notion  [4] Skip for now"

For each chosen target, ask for the one required field:
- **Slack:** channel name (e.g. `#competitive-intelligence`) — and whether they have a Slack MCP connected or prefer a webhook URL
- **Email:** the address(es) to send to
- **Notion:** the parent page ID (paste from the page URL)

Once collected, update `{workspace_root}/config.yaml` under `distribution:` — set `enabled: true` for the chosen target and fill in the required field. Do not overwrite targets the user didn't configure.

Confirm what was set:
> "Slack distribution enabled → #competitive-intelligence. Briefs will be sent there after each run (requires Slack MCP or webhook to be active)."

If skip: tell the user they can configure it any time by editing `{workspace_root}/config.yaml`.

---

### Step 7 — Summary

Show a completion summary:

```
Setup complete.

  Workspace: {workspace_root}/

  ✓ {workspace_root}/{product_name}.md
  ✓ {workspace_root}/competitors.yaml   (or: ✗ skipped — add competitors before your first run)
  ✓ distribution configured             (or: — skipped)

Next steps:
  Run /ci:run to start your first competitive analysis.
  Run /ci:schedule to set up automated weekly/monthly runs.
```

If competitors were skipped, add:

> "You'll need to create `{workspace_root}/competitors.yaml` before running the pipeline. See the format in the summary above."

---

## Rules

- Ask questions one at a time — don't dump all questions at once
- Use their words, don't paraphrase when writing the files
- Don't overwrite a file that already exists unless the user explicitly asks you to redo that section
- Don't generate placeholder content — if the user skips a question or gives a vague answer, write what they gave and note `(draft)` inline
