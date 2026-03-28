---
name: ci:import
description: Import an existing skill file (extractor, synthesizer, or command) into the competitive analysis pipeline. Reads the file or pasted content, detects what it is, validates it against the pipeline contract, adapts what doesn't conform, places it in the right directory, and registers it in the right index.
---

# /ci:import — Import an Existing Skill

## Role

Take a skill the user already has — written for this pipeline, adapted from another, or built elsewhere — and make it usable in `/ci:run`. Detect its type, validate it against the contract for that type, repair what's missing or misaligned, place it in the right location, and register it. The output is a file ready to be activated with a single status change.

This command does not run or test the skill. It makes it structurally correct and registered. The user activates it.

---

## Inputs

The user invokes this one of three ways:

```
/ci:import path/to/skill.md
/ci:import <url>
/ci:import [paste content inline]
```

**If a file path is given:** read the file directly.

**If a URL is given:**
- If it is a GitHub `blob` URL (`github.com/.../blob/...`), convert to raw before fetching:
  `https://raw.githubusercontent.com/[owner]/[repo]/[branch]/[path]`
- Prefer `gh api repos/[owner]/[repo]/contents/[path]` (gh CLI) if available — it handles auth and works on private repos.
- Fall back to WebFetch with the raw URL.
- If fetch fails (private repo, auth wall, rate limit): abort and ask the user to paste the content directly.

**If content is pasted:** treat it as the raw skill content.

Either way, proceed with the same process from Step 1.

---

## Step 1 — Read and parse

**If fetching from a URL via WebFetch**, use this exact prompt to avoid summarization:
> *"Return the complete raw markdown text of this file verbatim, including all frontmatter, headings, tables, code blocks, and every line of content. Do not summarize, paraphrase, or omit anything. Start your response with the very first character of the file."*

Read the content. Extract:
- Frontmatter (if present): `name`, `description`, and any typed fields (`profile_section`, `sources`, `tier_scope`, `stage`, `owns`, `frameworks`)
- Section headings and their content
- Any existing contracts, inputs tables, output schemas, fallback sections

Summarise what you found in one line before continuing:
> "Found: [what it appears to be — e.g. 'an extractor for job postings with partial frontmatter and a research steps section']"

---

## Step 2 — Detect type

Determine whether this is an extractor, market synthesizer, or command.

**Signals to look for:**

| Signal | Likely type |
|--------|-------------|
| `profile_section` in frontmatter starting with `## Analysis —`, or applies a named framework per-competitor, writes to profile | Profile analyzer |
| `profile_section` in frontmatter (other), or `## Research Steps`, or references to fetching URLs | Extractor |
| `stage: 2`, `frameworks:`, `## Analysis Process`, writes to `{workspace_root}/syntheses/` | Market synthesizer |
| References to reading profiles/syntheses and returning output to the user, no fetching | Command |
| `## Steps`, user-facing instructions, invoked as `/something` | Command |

If the type is ambiguous, ask:
> "This looks like it could be an [X] or a [Y]. Which is it?"

**Profile analyzer vs. profile builder:**

If the imported skill reads a single competitor's snapshot and writes a named `## Analysis — [Framework]` section to the profile, it is a **profile analyzer** — import it as one.

If it appears to be replacing or modifying the profile builder (writing `## Current State` or `## Direction`), stop and tell the user:
> "The profile builder is a fixed system component — it can't be replaced. If you want to add a new analytical framework per-competitor, import this as a profile analyzer instead."

**Foreign skill formats:** Skills written for other systems may not use pipeline conventions. Common patterns to recognise:
- `$ARGUMENTS` variable → written for Claude Projects skill format, not this pipeline
- `## Instructions` with a single monolithic block → likely a general-purpose prompt, not a structured pipeline component
- No frontmatter at all → may be a raw prompt or documentation

When a foreign format is detected, note it in the Step 1 summary line and describe what structural work will be needed to adapt it.

---

## Step 3 — Validate against contract

Run the appropriate contract check.

### Extractor contract

| Check | Pass condition | If missing |
|-------|---------------|------------|
| `name` ends in `-extractor` | frontmatter or derivable from filename | Derive from content; ask if ambiguous |
| `description` present | one sentence describing signal domain | Draft from Role section or ask |
| `profile_section` defined | exact `## Heading` it writes to | Ask the user — cannot be inferred reliably |
| `sources` defined | comma-separated list of source types | Derive from Research Steps if present |
| `tier_scope` defined | one or more of: direct, adjacent, aspirational | Default to `direct` and tell the user |
| `## Research Steps` present | numbered, specific, executable steps | Flag as TODO — cannot be auto-generated |
| `## Output Schema` present | markdown block matching the profile section | Draft from existing section if profile_section matches an existing extractor |
| `## Fallback` present | what to do when sources are unavailable | Draft a standard fallback sequence |
| `## Confidence Flags` or inline flags | uses `unknown`, `(unconfirmed)`, `[unavailable]` | Add the conventions section |
| Returns data only, no interpretation | no "this suggests", "this means" in output | Flag any interpretive language found; ask user to review |

### Profile analyzer contract

| Check | Pass condition | If missing |
|-------|---------------|------------|
| `name` ends in `-analyzer` | frontmatter or derivable | Derive from content |
| `profile_section` starts with `## Analysis —` | exact section heading | Ask the user — must match the framework name |
| `inputs` defined | snapshot + profile section at minimum | Default to standard analyzer inputs |
| `## Analysis Process` or `## Steps` present | structured per-competitor analysis logic | Flag as TODO |
| `## Output Schema` present | exact markdown for `## Analysis — [Name]` | Draft from content if framework is clear |
| Merge rules defined | what rewrites vs. carries forward | Add standard merge rules scaffolded for the framework |
| Source annotations used | `_(src: ...)_` on findings | Add conventions section |
| Missing data hints present | inline hints per section, not top-level errors | Add note about the convention |

---

### Market synthesizer contract

| Check | Pass condition | If missing |
|-------|---------------|------------|
| `name` ends in `-synthesizer` | frontmatter or derivable | Derive |
| `stage` is `2` | frontmatter | Ask — if Stage 1, block the import (see Step 2) |
| `owns` defined | what file/section it writes (must be `{workspace_root}/syntheses/`) | Ask |
| `frameworks` defined | named framework(s) applied | Derive from Analysis Process if present |
| `stale_after` defined | how many days before the brief nudges the user to re-run | Ask: "How often should this run? (e.g. 30, 60, 90 days)" — suggest 90 days for strategic frameworks, 30 days for tactical/positioning ones |
| `## Inputs` table present | lists all inputs with types | Draft from content |
| `## Analysis Process` or `## Steps` present | structured analysis logic | Flag as TODO if absent |
| `## Output Schema` present | exact markdown structure of output | Draft if stage and owns are known |
| Reads profiles/snapshots, writes to owned section only | does not fetch web data | Flag if web fetching found |

### Command contract

| Check | Pass condition | If missing |
|-------|---------------|------------|
| Frontmatter `name` matches `ci:[name]` pattern | present | Derive from filename or ask |
| `description` present | one sentence | Draft |
| Steps are numbered and sequential | `### Step N` structure | Restructure if content is there but unformatted |
| Clear output — what it produces and where | file path or user-facing output defined | Ask |

---

## Step 4 — Adapt

For each failed contract check, either fix it automatically or flag it for the user.

**Fix automatically (no confirmation needed):**
- Add missing `name`, `description`, `sources`, `tier_scope` if clearly derivable from content
- Add `source_url` to frontmatter if the input was a URL — preserves provenance so the origin can be checked if the upstream skill is updated
- Add standard `## Confidence Flags` section if missing (use the standard text from an existing extractor)
- Add standard `## Fallback` section scaffolded for the source types present
- Normalise frontmatter format to match existing files
- Rename section headings to match pipeline conventions (e.g., `## Output` → `## Output Schema`)

**Flag as TODO (user must complete):**
- `profile_section` for extractors — wrong section breaks the pipeline
- `## Research Steps` if absent — the extractor won't know what to fetch
- `stage` for synthesizers — cannot be inferred reliably
- Any interpretive language found in extractor output schema — mark with `# TODO: review — extractors should return data, not interpretation`

When flagging, be specific:
> "Added `TODO: define profile_section` on line 4 — this must be an exact ## heading from the competitor profile."

Show a diff-style summary of changes made before writing the file.

---

## Step 5 — Place the file

Determine the destination based on type:

| Type | Destination |
|------|-------------|
| Profile analyzer | `agents/analyzers/[name]-analyzer.md` |
| Extractor | `agents/extractors/[name]-extractor.md` |
| Stage 2 synthesizer | `agents/synthesizers/[name]-synthesizer.md` |
| Command | `commands/[name].md` |

If a file already exists at that path:
> "A file already exists at `[path]`. Overwrite it, save as `[name]-imported.md`, or cancel?"

Write the adapted file.

---

## Step 5b — Profile-synthesizer section check (extractors only)

After placing an extractor file, verify the profile-synthesizer will leverage the new snapshot section.

Read `skills/profile-builder.md` Step 1. Look for the extractor's `profile_section` heading in the section table.

**If the section is already listed:** note it in the Step 7 summary as confirmed.

**If NOT listed:** add a row to the Step 1 table in `skills/profile-builder.md`:

```markdown
| `## [Section]` | [What this section tells you — 1 sentence, derived from the extractor's description] |
```

If it's unclear what the synthesizer should infer, write a `TODO:` placeholder and add it to the Step 7 summary under "Needs your attention":
> `! profile-synthesizer Step 1 — TODO row added for ## [Section]: fill in what to infer before activating`

---

## Step 6 — Register

Add a row to the right index with `status: draft`.

- **Profile analyzer** → row in `agents/analyzers/0_ANALYZERS.md`. Ask: "Should I also add this to `{workspace_root}/config.yaml` so it runs on the next pipeline run? (It will stay as `draft` in the index until you activate it.)"
- **Extractor** → row in `agents/extractors/0_EXTRACTORS.md`
- **Market synthesizer** → row in `agents/synthesizers/0_SYNTHESIZERS.md`; include the `Stale after` value confirmed during the contract check

For commands: no registration needed. Confirm it's discoverable:
> "Command `/ci:[name]` is available immediately — Claude Code auto-discovers files in `commands/`."

### Stage 2 synthesizers: scaffold companion command

If the imported skill is a Stage 2 synthesizer, it needs a `/ci:` command to be invokable. After registering, ask:

> "Stage 2 synthesizers are invoked via a `/ci:` command. Should I scaffold `commands/[name].md` now? (The synthesizer is registered but won't be runnable without it.)"

If yes: scaffold the command using the same process as `/ci:new` → If command, using `commands/market.md` as a reference. Fill in:
- Frontmatter: `name` as `ci:[synthesizer-name-without-synthesizer-suffix]`, `description`
- Role: one sentence derived from the synthesizer's Role section
- Steps: read profiles → read context → run synthesizer → write output → summary to user
- When to run: derive from the synthesizer's cadence signals

Place at `commands/[name].md`. No index registration needed — auto-discovered.

Tell the user:
> "Command `/ci:[name]` scaffolded at `commands/[name].md`. Commands are auto-discovered — no registration needed."

---

## Step 7 — Summary

```
Imported: [original source]
Placed at: [file path]
Type: [extractor | synthesizer (stage N) | command]

Changes made:
  ✓ Added missing frontmatter fields: [list]
  ✓ Added standard Fallback section
  ✓ Normalised section headings

Needs your attention (TODOs in file):
  ! profile_section — must be defined before activating
  ! Research Steps — drafted but incomplete
  [list any other TODOs]

Registered in: [index file] — status: draft

To activate:
  1. Review [file path] and complete the TODO: items
  2. Set status from `draft` to `active` in [index]
  3. Test: /ci:run force_run=true [competitor]
```

---

## Quality Rules

- **Preserve intent.** When adapting the file, change structure and format — not meaning. If the logic is wrong for the pipeline, flag it; don't silently rewrite it.
- **Specific TODOs.** Every `TODO:` comment must say exactly what's needed and why. "TODO: fill this in" is not acceptable.
- **Draft, not active.** Always `status: draft` on registration. Never activate automatically.
- **Show changes before writing.** Before overwriting or placing a file, summarise what was changed. The user should know what the import did to their skill.
