---
name: ci:new
description: Scaffold and register a new extractor, synthesizer, or command for the competitive analysis pipeline. Guides you through what it needs, generates the file from the right template, and registers it so the orchestrator picks it up automatically.
---

# /ci:new — Create a New Pipeline Component

## Role

Guide the user through creating a new extractor, synthesizer, or command. Scaffold the file, fill in what's known, leave clear TODOs for what the user must complete, and register it in the right index. When done, the new component is either immediately active or one field change away from being active.

---

## Step 1 — Ask what type to create

> What would you like to create?
> [1] Extractor — fetches a new type of signal from competitor websites (pricing, reviews, job postings, etc.)
> [2] Profile analyzer — applies a framework to each competitor's snapshot and adds an ## Analysis — [Name] section to their profile (e.g., JTBD, Porter's per-competitor, custom)
> [3] Market synthesizer — applies a cross-competitor framework across all profiles on demand (e.g., positioning map, differentiation analysis)
> [4] Command — a new `/ci:` command that runs a custom analysis

Show a one-line description of each to help the user choose. Wait for their answer before continuing.

**Note:** The profile builder (which writes Current State and Direction) is a fixed system component — it is not user-extensible. Profile analyzers are the extension point for per-competitor analysis frameworks.

---

## Step 2 — Collect parameters

### If extractor:

Ask in sequence (one or two questions at a time — don't dump a form):

1. **What signal domain does it cover?**
   Example answers: "job postings", "social media activity", "tech stack from job listings", "G2 reviews", "press releases"
   → Derive the extractor name: `[domain]-extractor`

2. **Where does it get its data?**
   What URLs or sources will it check? (e.g., careers page, G2 profile, LinkedIn, web search, Twitter/X)

3. **What profile section does it write?**
   Does it contribute to an existing section (e.g., `## Strategic Signals`, `## Market Presence`) or does it need a new one?
   Read `agents/extractors/0_EXTRACTORS.md` and show the list of existing sections so the user can pick or name a new one.

   **Section ownership constraint:** Each snapshot section can only be owned by one extractor — the orchestrator writes each extractor's output directly to its section, so two extractors claiming the same section will overwrite each other. If the user picks a section already owned by another extractor, flag the conflict and suggest a more specific section name (e.g., if `## Positioning` is taken, suggest `## Advertising` or `## [Domain] Signals`). Confirm with the user before proceeding.

4. **Which competitor tiers should it run for?**
   `direct` only | `direct` and `adjacent` | all three
   Remind the user: direct = weekly, adjacent = monthly, aspirational = quarterly

5. **Any per-competitor config it needs?**
   e.g., "a custom G2 URL per competitor" — if yes, this becomes a field in `{workspace_root}/competitors.yaml`

### If profile analyzer:

All user-created profile analyzers run per-competitor after the profile builder. Each writes exactly one `## Analysis — [Name]` section to the profile.

1. **What framework or lens does it apply?**
   e.g., "JTBD", "Porter's per-competitor", "sentiment analysis", "GTM motion inference"
   → Derive the analyzer name: `[framework]-analyzer`

2. **What question does it answer for each competitor?**
   e.g., "What jobs does this competitor get hired for?" or "What sales motion are they running?"
   This becomes the Role section.

3. **What snapshot sections does it need?**
   Which extractor outputs does this framework rely on? (e.g., JTBD needs reviews + positioning; GTM motion needs positioning + strategic-signals)
   If required sections don't have active extractors, note it — the user should activate or build those extractors first.

4. **What does the output section look like?**
   Draft the structure of `## Analysis — [Name]`. Keep it parallel to how SWOT structures its output — named subsections, source annotations, merge rules.

5. **Should it be added to `{workspace_root}/config.yaml` immediately?**
   If yes, add it after scaffolding. If no, leave it out and tell the user to add it when ready.

---

### If market synthesizer:

All user-created synthesizers are Stage 2 — they run on-demand across all competitor profiles, not per-competitor. The per-competitor profile synthesis is handled by the system profile-synthesizer.

1. **What framework or lens does it apply?**
   e.g., "Jobs-to-be-done", "Ansoff matrix", "sales motion inference", "ICP derivation from positioning"

2. **What section does it write?**
   It writes to `{workspace_root}/syntheses/[name]-[date].md` — what's the filename prefix?

3. **What inputs does it need?**
   Which profile sections or files does it read? (e.g., `## SWOT`, `## Our Read`, `{workspace_root}/{product_name}.md`)

4. **How often should it run?** (used to set the staleness reminder in briefs)
   Suggest based on the framework: strategic frameworks (Blue Ocean, differentiation) → 90 days; tactical/positioning frameworks → 30 days. Confirm with the user or accept their answer as-is.

5. **Should it have its own `/ci:` command?**
   Market synthesizers are always invoked via a command. If yes, also scaffold the command (go to If command below). If no, the synthesizer is not runnable until a command exists.

### If command:

1. **What does it do in one sentence?**
   e.g., "Runs GTM synthesis across all profiles and writes to {workspace_root}/syntheses/"

2. **What does it read?**
   Profile sections? Synthesis files? Workspace files? Web data?

3. **What does it produce?**
   A synthesis file in `{workspace_root}/syntheses/`? Updates to profiles? Just returns output to the user?

4. **Should it run on a schedule?**
   If yes, suggest running `/ci:schedule` after creation to register it. Note the suggested cadence.

5. **Does it need a new synthesizer, or does it call an existing one?**
   If it needs a new synthesizer, create that first (go to synthesizer flow above), then come back to scaffold the command.

---

## Step 3 — Scaffold the file

### Extractor

Create `agents/extractors/[name]-extractor.md` using an existing extractor (e.g., `agents/extractors/pricing-extractor.md`) as a structural reference.

Fill in everything derivable from the user's answers:
- `name`, `description`, `profile_section`, `sources`, `tier_scope` in the frontmatter
- The Role section (one sentence from their answer to Q1)
- Research Steps: draft specific steps based on the sources they named. Be concrete — what URL to fetch, what to look for on the page. Mark anything you can't determine as `TODO:`
- Output Schema: draft the field list based on the signal domain. If writing to an existing section, copy the schema from that section in an existing extractor. Mark uncertain fields as `TODO:`
- Fallback: draft a standard fallback sequence for the source types they named

Leave a `TODO:` comment wherever the user must complete something before the extractor is usable.

### Profile Analyzer

Create `agents/analyzers/[name]-analyzer.md`.

Use `agents/analyzers/swot-analyzer.md` as the reference implementation.

Fill in:
- Frontmatter: `name`, `description`, `profile_section` (exact `## Analysis — [Name]`), `inputs`, `status: draft`
- Role section
- Inputs table
- Analysis Process: draft the steps based on the framework. Be specific about what each snapshot section contributes to each framework output. Mark anything needing user input as `TODO:`
- Output Schema: the exact markdown structure for `## Analysis — [Name]`, including subsections and source annotation format
- Merge rules: which parts rewrite on active sources, which carry forward on inactive sources
- Missing data hints: which extractor types would improve which parts of the analysis

---

### Market Synthesizer

Create `agents/synthesizers/[name]-synthesizer.md`.

Use the structure from `agents/synthesizers/market-synthesizer.md` as a reference.

Fill in:
- Frontmatter: `name`, `description`, `stage: 2`, `owns`, `frameworks`, `inputs`
- Role section
- Inputs table (from their Q4 answer)
- Analysis Process: draft the steps based on the framework they named. Use the framework's standard methodology as a starting scaffold. Mark anything needing user input as `TODO:`
- Output Schema: draft the markdown structure for what it writes

### Command

Create `commands/[name].md`.

Use the structure from an existing command as a reference (read `commands/market.md` or `commands/blue-ocean.md`).

Fill in:
- Frontmatter: `name` (as `ci:[name]`), `description`
- Role section
- Steps: draft the pipeline — what to read, what synthesizer to call, where to write output, what to return to the user
- When to run section

---

## Step 3b — Profile-synthesizer section check (extractors only)

After scaffolding an extractor, check whether the profile-synthesizer will leverage the new section.

Read `skills/profile-builder.md` Step 1. Look for the new extractor's `profile_section` heading in the section table (e.g., `## Pricing`).

**If the section is already listed:** confirm to the user:
> "Profile-builder already references `[section]` — no changes needed there."

**If the section is NOT listed:** tell the user:
> "`skills/profile-builder.md` Step 1 does not reference `[section]`. The builder will read it as raw text but won't know what to infer from it. Add a row to the Step 1 table describing what this section contributes to Current State."

Then open `skills/profile-builder.md` and add a row to the Step 1 table:

```markdown
| `## [Section]` | [What this section tells you — 1 sentence] |
```

Use the extractor's `description` and `profile_section` to write the row. If you can't determine what to infer, write a `TODO:` placeholder and tell the user:
> "Added a TODO row for `## [Section]` in profile-builder Step 1 — fill in what to infer from this data before activating the extractor."

---

## Step 4 — Register

### Profile Analyzer → 0_ANALYZERS.md + config.yaml

Add a row to `agents/analyzers/0_ANALYZERS.md`:

```
| [[name]-analyzer.md](./[name]-analyzer.md) | [profile section] | [inputs] | draft |
```

Set `status: draft`.

If the user said yes to adding it to config (Step 2 Q5): add the name to `profile_analyzers` in `{workspace_root}/config.yaml`.

Tell the user:
> Your analyzer is registered with `status: draft`. Once you've reviewed and completed the TODOs in `agents/analyzers/[name]-analyzer.md`, change the status to `active` in `0_ANALYZERS.md` and ensure the name is in `{workspace_root}/config.yaml`. Then run `/ci:profile [competitor]` to test it against an existing snapshot.

---

### Extractor → 0_EXTRACTORS.md

Add a row to `agents/extractors/0_EXTRACTORS.md`:

```
| [[name]-extractor.md](./ [name]-extractor.md) | [section] | [sources] | [tier scope] | draft |
```

Set `status: draft` — not `active`. The user must review the scaffolded file and set it to `active` themselves when they're satisfied it's correct.

Tell the user:
> Your extractor is registered with `status: draft`. Once you've reviewed and completed the TODOs in `agents/extractors/[name]-extractor.md`, change the status to `active` — the orchestrator will pick it up on the next `/ci:run`.

### Market Synthesizer → 0_SYNTHESIZERS.md

Add a row to `agents/synthesizers/0_SYNTHESIZERS.md` under the **Market Synthesizers** section:

```
| [[name]-synthesizer.md](./[name]-synthesizer.md) | [what it owns] | [framework] | `/ci:[command-name]` | [N] days | draft |
```

Use the cadence confirmed in Q4 for the `Stale after` value. Set `status: draft`.

Tell the user:
> Invoke this synthesizer via `/ci:[command-name]`. It won't run automatically with `/ci:run` — market synthesizers are on-demand only.

### Command

No registration needed — Claude Code discovers `commands/*.md` automatically. The command is available as `/ci:[name]` immediately.

Tell the user:
> Your command `/ci:[name]` is ready to use. Commands are auto-discovered — no registration needed.

If the user asked to schedule it: remind them to run `/ci:schedule` and add it there.

---

## Step 5 — Summary

Show what was created:

```
Created:
  [file path]             [what it is]
  [file path if command]  [what it is]

Registered in:
  [index file]            [row added, status: draft]

Next steps:
  1. Review [file] — complete the TODO: sections
  2. Change status from `draft` to `active` in [index]  (extractors and synthesizers only)
  3. Test it: run `/ci:run force_run=true` with one competitor to see it fire
  4. [Any other specific next step based on what was created]
```

---

## Quality Rules

- **Scaffold, don't stub.** The generated file should be 70–80% complete — specific enough to run against a real competitor with minimal editing. A file full of `[fill this in]` placeholders is not a scaffold, it's a form.
- **Draft, not active.** New components always start as `draft` in the index. The user activates them after review. Never set `status: active` automatically.
- **One component at a time.** If the user needs both a synthesizer and a command, create the synthesizer first, then the command. Show progress between steps.
- **Use what's there.** Read existing extractors/synthesizers before scaffolding — the patterns, field names, and section headings should be consistent with what already exists.
