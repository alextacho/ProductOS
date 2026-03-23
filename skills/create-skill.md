---
name: create-skill
description: Guided workflow for generating a new extractor or synthesizer that correctly slots into the competitive analysis pipeline. Produces a skill file, an index row, and orchestrator wiring instructions.
layer: system
---

# Create Skill

## Role

Guide the user through creating a new extractor or synthesizer for the competitive analysis pipeline. Ask the right questions, generate a correctly-formatted skill file, produce the index row, and show where it wires into the orchestrator.

The goal is a skill that works on the first try — no manual editing after generation.

---

## When to Use

Use this skill when:
- "I want to add a skill that extracts [signal type]"
- "I want a synthesizer that analyzes [pattern] across competitors"
- "How do I add [pricing/reviews/jobs/news] tracking to the pipeline?"

Do not use this skill for:
- Editing an existing skill (edit it directly)
- Changing the orchestrator or pipeline structure
- Creating a skill for a different domain (this skill is specific to this pipeline's interface contracts)

---

## Phase 1 — Classify the skill

Ask the user one question:

> **Is this skill extracting raw data from a source (extractor), or interpreting data that's already in profiles (synthesizer)?**

- If extracting from a website, review site, job board, or search results → **extractor**
- If reading profile sections and producing insight, analysis, or summaries → **synthesizer**
- If unclear → ask a follow-up: "Does this skill need to fetch anything from the internet?"

---

## Phase 2A — Extractor elicitation

Ask these questions in sequence. Do not ask them all at once.

**Q1: Profile section**
> Which section of the competitor profile will this extractor own and write to? (Check `DESIGN.md` for the section list — it must match an existing `## Section` heading, or you're adding a new section.)

Acceptable answers: `## Positioning`, `## Product`, `## Market Presence`, `## Strategic Signals`, or a new section (flag: adding a new section requires updating the profile template in `DESIGN.md`).

**Q2: Sources**
> Where does this extractor fetch its data? (e.g., pricing page, G2, LinkedIn jobs, web search for news)

**Q3: Tier scope**
> Which competitor tiers should this run for? (direct / adjacent / aspirational — or a subset)

**Q4: Output schema**
> Walk me through the fields you want to capture. What does a complete, well-filled version of this section look like?

Use the answer to draft the output schema. Show it to the user and ask: "Does this match what you had in mind, or should we change anything?"

**Q5: Hard cases**
> What should happen when a source is unavailable or returns no useful data? What should happen if sources conflict?

Use the answer to write the Confidence Flags and Fallback sections.

---

## Phase 2B — Synthesizer elicitation

Ask these questions in sequence.

**Q1: Stage**
> Is this per-competitor (Stage 1) or across all competitors at once (Stage 2 / market-level)?

- Stage 1: runs after extractors for one competitor, writes to that competitor's profile
- Stage 2: runs once after all Stage 1 synthesis, writes to a market-level document

**Q2: Inputs**
> Which profile sections does this synthesizer read? (It can only use data already in the profiles — it does not fetch.)

**Q3: Output**
> What sections does it write, and in what format? Show me a sketch of what a good output looks like.

**Q4: Framework (if applicable)**
> Does this synthesizer apply a specific analytical framework? (e.g., SWOT, Porter's 5 Forces, Blue Ocean value curve, Jobs-to-be-Done)

**Q5: Calibration**
> How should this synthesizer avoid being generic? What's the difference between a good output and a fluffy output for this type of analysis?

---

## Phase 3 — Validation

Before generating, run through this checklist internally:

**For extractors:**
- [ ] Output schema maps exactly to the named profile section — field names match
- [ ] Every field has a defined behavior for the "unknown" case
- [ ] Confidence flags cover: unavailable source, conflicting signals, inferred data
- [ ] Fallback is defined for when the primary source fails
- [ ] Out of Scope explicitly names what other extractors own
- [ ] Tier scope is explicit (direct / adjacent / aspirational)

**For synthesizers:**
- [ ] Stage is explicit (1 or 2)
- [ ] Every input section is listed
- [ ] Every output section is listed, with format
- [ ] At least one quality rule that prevents generic output
- [ ] Out of Scope explicitly says "do not fetch data" (for Stage 1) or "do not write to individual profiles" (for Stage 2)

If any checklist item is unclear, ask a clarifying question before generating.

---

## Phase 4 — Generation

Generate three artifacts:

### Artifact 1: Skill file

Generate a complete skill file following the appropriate template:
- Extractor: follow the structure in `skills/extractors/TEMPLATE.md`
- Synthesizer: follow the structure of `agents/synthesizers/market-synthesizer.md` as a reference

File path:
- Extractor: `skills/extractors/[signal-type]-extractor.md`
- Synthesizer (Stage 1): `agents/synthesizers/[name]-synthesizer.md`
- Synthesizer (Stage 2): `agents/synthesizers/[name]-synthesizer.md`

### Artifact 2: Index row

Generate the row to add to the appropriate index file:

For extractors (`skills/extractors/0_EXTRACTORS.md`):
```
| [skill-file-link] | `## [Profile Section]` | [sources] | [tier scope] | planned |
```
_(Set to `planned` — change to `active` after testing end-to-end against a real competitor)_

For synthesizers (`agents/synthesizers/0_SYNTHESIZERS.md`):
```
| [skill-file-link] | [Stage 1 or 2] | [inputs] | [outputs] | planned |
```

### Artifact 3: Orchestrator wiring note

Show exactly where in the pipeline this skill slots in:

For extractors:
```
Add a row to agents/extractors/0_EXTRACTORS.md with status: draft.
Set to active once tested — the orchestrator reads this index automatically.
No changes to agents/competitive-run.md needed.
```

For Stage 1 synthesizers:
```
Add a row to agents/synthesizers/0_SYNTHESIZERS.md (stage: 1) with status: draft.
Set to active once tested — the orchestrator reads 0_SYNTHESIZERS.md for all active stage 1 synthesizers.
No changes to agents/competitive-run.md needed.
```

For Stage 2 synthesizers:
```
Add a row to agents/synthesizers/0_SYNTHESIZERS.md (stage: 2) with status: draft.
Create a /ci:[name] command in commands/ to invoke it.
Stage 2 synthesizers are not run automatically by /ci:run — they need their own command.
```

---

## Phase 5 — Self-check

After generating, run the interface contract checklist one more time against the generated file:

> Read the generated skill. Can you answer yes to all of these?
> 1. The output schema matches the profile section format exactly — I could paste the output directly into a profile without any editing
> 2. Every confidence flag scenario is covered
> 3. Out of Scope is explicit and correct
> 4. The orchestrator wiring note points to the right place
> 5. The index row is correctly formatted

If any answer is no, fix the generated file before showing it to the user.

---

## Output to User

Show the user:
1. The complete skill file (paste inline, or offer to write to disk)
2. The index row to add
3. The orchestrator wiring note

Then say:

> To activate this skill:
> 1. Save the skill file to `[path]`
> 2. Add the index row to `[0_EXTRACTORS.md or 0_SYNTHESIZERS.md]`
> 3. Wire into the orchestrator per the note above
> 4. Test against one real competitor before setting status to `active`

---

## Out of Scope

- This skill does not modify the orchestrator directly — it produces a wiring note for the user to apply
- This skill does not run the generated skill — it generates and hands off
- This skill does not handle non-competitive-analysis domains — the interface contracts here are specific to this pipeline
