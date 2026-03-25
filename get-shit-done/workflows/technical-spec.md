<purpose>
Generate a Technical Specification for a phase. Spawns gsd-designer in tech-spec mode with phase context.

Standalone command. For most workflows, use `/gsd:design-phase` which orchestrates HLD → LLD → Technical Spec automatically.
</purpose>

<available_agent_types>
Valid GSD subagent types (use exact names — do not fall back to 'general-purpose'):
- gsd-designer — Produces Technical Spec from LLD, creates implementation map
</available_agent_types>

<process>

## Step 0: Resolve Model Profile

@~/.claude/get-shit-done/references/model-profile-resolution.md

Resolve model for:
- `gsd-designer`

## Step 1: Normalize and Validate Phase

@~/.claude/get-shit-done/references/phase-argument-parsing.md

```bash
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
```

If `found` is false: Error and exit.

## Step 2: Gather Phase Context

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init design-phase "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extract: `phase_dir`, `padded_phase`, `phase_number`, `phase_name`, `hld_path`, `lld_path`, `tech_spec_path`, `context_path`, `research_path`, `commit_docs`.

## Step 3: Check Prerequisites

**LLD.md must exist.** If `lld_path` is missing:
> Error: Phase {phase} has no LLD. Run `/gsd:lld {phase}` or `/gsd:design-phase {phase}` first.

**HLD.md must exist.** If `hld_path` is missing:
> Error: Phase {phase} has no HLD. Run `/gsd:hld {phase}` or `/gsd:design-phase {phase}` first.

## Step 4: Check Existing Technical Spec

If `tech_spec_path` exists: Offer update/view/skip options.

## Step 5: Spawn gsd-designer (tech-spec mode)

```
Task(
  prompt="<objective>
Create Technical Specification for Phase {phase_number}: {phase_name}
Mode: Technical Spec only (LLD already exists)
</objective>

<files_to_read>
- {lld_path} (LOW-LEVEL DESIGN — component inventory, interfaces, contracts)
- {hld_path} (HIGH-LEVEL DESIGN — architecture overview)
- {context_path} (USER DECISIONS — locked decisions)
- {research_path} (RESEARCH — standard stack, test patterns)
</files_to_read>

<output>
Write to: {phase_dir}/{padded_phase}-TECHNICAL-SPEC.md
Follow template: @~/.claude/get-shit-done/templates/technical-spec.md
</output>",
  subagent_type="gsd-designer",
  model="{designer_model}",
  description="TechSpec Phase {phase}"
)
```

## Step 6: Handle Return

- `## LLD COMPLETE` (with tech spec) — Display summary, offer: Plan/Review/Done
- `## LLD BLOCKED` — Show issue, offer: Fix LLD/Skip/Manual

</process>

<success_criteria>
- [ ] Phase validated against roadmap
- [ ] LLD.md exists (required)
- [ ] HLD.md exists (required)
- [ ] gsd-designer spawned in tech-spec mode
- [ ] TECHNICAL-SPEC.md created in phase directory
- [ ] User knows next steps
</success_criteria>
