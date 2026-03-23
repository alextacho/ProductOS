---
name: setup
description: Interactive first-run setup for the competitive analysis plugin. Creates workspace/ourproduct.md, workspace/positioning.md, and optionally seeds workspace/competitors.yaml. Run this before your first /ci:run run.
type: command
---

# Competitive Analysis — Setup

## Role

Walk the user through creating the required context files for the competitive analysis plugin. Check what already exists, only ask for what's missing, and write well-structured files that the pipeline can use immediately.

---

## Steps

### Step 0 — Ensure commands are registered

Copy all files from `commands/` into `.claude/commands/ca/`, creating the directory if it doesn't exist. This ensures any commands not yet present (e.g. `help.md`, `publish-brief.md`) are discoverable by Claude Code.

Do this silently — no output unless a copy fails.

---

### Step 1 — Check what exists

Check for each required and optional file:

| File | Required |
|------|----------|
| `workspace/ourproduct.md` | yes |
| `workspace/positioning.md` | yes |
| `workspace/competitors.yaml` | optional |

For each file that already exists, skip its section and note it's already set up.

If all required files exist: say "You're already set up. Run /ci:run to start your first competitive analysis run." and stop.

---

### Step 2 — Create workspace/ourproduct.md (if missing)

Tell the user:

> "Let's describe your product so the synthesizer can frame competitor analysis in terms of your actual context. I'll ask a few questions."

Ask these questions **one at a time**, waiting for a response before continuing:

1. **What does your product do?** (1–3 sentences — what it is and what problem it solves)
2. **Who is it for?** (Role, company stage, or context — be specific)
3. **How do you win?** (2–4 specific differentiators — what makes you different or better for your target user)
4. **What are you not?** (1–2 things you explicitly don't do — helps frame threat assessments correctly)
5. **What's your one-line positioning?** (The way you'd describe it in a pitch or header — optional, skip if they don't have one yet)

Once you have responses, write `workspace/ourproduct.md`:

```markdown
# Our Product Context

_Read by competitor-synthesizer to calibrate SWOT and threat level assessments._

---

## What We're Building

[Answer to question 1]

## Who It's For

[Answer to question 2]

## How We Win

[Answer to question 3 — formatted as bullet points if multiple differentiators]

## What We're Not

[Answer to question 4 — formatted as bullet points]

## Positioning

[Answer to question 5, or omit this section if skipped]
```

Confirm the file was written.

---

### Step 3 — Create workspace/positioning.md (if missing)

Tell the user:

> "Now let's capture your current positioning. This gets used to track how your positioning compares to competitors over time."

Ask:

1. **What's your core message?** (The headline you'd use — what you do and for whom, in one sentence)
2. **Who specifically do you target?** (More specific than question 2 above — the job title, workflow, or moment you're targeting)
3. **What differentiators do you lead with?** (The 2–4 things you emphasize most in sales and marketing — use your actual language)
4. **What's your tone?** (How do you sound? 2–3 words — e.g., "direct and opinionated", "friendly and approachable", "technical and precise")

Write `workspace/positioning.md`:

```markdown
# Our Positioning

_Used to compare our messaging against competitor positioning over time._

---

**Core message:** [Answer to question 1]
**Target audience:** [Answer to question 2]
**Tone:** [Answer to question 4]

**Differentiators we lead with:**
- [Differentiator 1 — their exact language where possible]
- [Differentiator 2]
- [Add as many as stated]
```

Confirm the file was written.

---

### Step 4 — Seed workspace/competitors.yaml (if missing, optional)

Ask:

> "Do you want to add your first competitors now? You can always add more later. (yes / skip)"

If yes:

Tell the user:

> "Tell me your competitors — for each one, give me the name and website. I'll ask about tier after."

For each competitor the user names, ask:

> "Is [name] a **direct** competitor (same buyer, same job), **adjacent** (overlapping use cases but different focus), or **aspirational** (a market leader you're watching)?"

Once you have the list, write `workspace/competitors.yaml`:

```yaml
competitors:
  - name: [Name]
    website: [https://...]
    tier: direct | adjacent | aspirational
    last_updated: null  # will be set after first run
    changelog_url: null  # set once discovered — extractor reads this first, skips discovery if present
```

Add one entry per competitor. Confirm the file was written.

If skip: note that they can add competitors later by editing `workspace/competitors.yaml` directly.

---

### Step 5 — Configure distribution (optional)

Tell the user:

> "Briefs can be automatically sent to Slack, email, or Notion after each run. Would you like to configure a delivery target now? (yes / skip)"

If yes:

Ask which target they want to set up:
> "[1] Slack  [2] Email  [3] Notion  [4] Skip for now"

For each chosen target, ask for the one required field:
- **Slack:** channel name (e.g. `#competitive-intelligence`) — and whether they have a Slack MCP connected or prefer a webhook URL
- **Email:** the address(es) to send to
- **Notion:** the parent page ID (paste from the page URL)

Once collected, update `context/distribution.yaml` — set `enabled: true` for the chosen target and fill in the field. Do not overwrite targets the user didn't configure.

Confirm what was set:
> "Slack distribution enabled → #competitive-intelligence. Briefs will be sent there after each run (requires Slack MCP or webhook to be active)."

If skip: tell the user they can configure it any time by editing `context/distribution.yaml`.

---

### Step 6 — Summary

Show a completion summary:

```
Setup complete.

  ✓ workspace/ourproduct.md
  ✓ workspace/positioning.md
  ✓ workspace/competitors.yaml       (or: ✗ skipped — add competitors before your first run)
  ✓ context/distribution.yaml        (or: — skipped)

Next steps:
  Run /ci:run to start your first competitive analysis.
  Run /ci:schedule to set up automated weekly/monthly runs.
```

If `workspace/competitors.yaml` was skipped, add:

> "You'll need to create `workspace/competitors.yaml` before running the pipeline. See the format in the summary above."

---

## Rules

- Ask questions one at a time — don't dump all questions at once
- Use their words, don't paraphrase when writing the files
- Don't overwrite a file that already exists unless the user explicitly asks you to redo that section
- Don't generate placeholder content — if the user skips a question or gives a vague answer, write what they gave and note `(draft)` inline
