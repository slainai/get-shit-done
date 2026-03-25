<purpose>
Generate a Low-Level Design (LLD) for a phase. Spawns gsd-designer with phase context.

Standalone LLD command. For most workflows, use `/gsd:design-phase` which orchestrates HLD → LLD → Technical Spec automatically.
</purpose>

<available_agent_types>
Valid GSD subagent types (use exact names — do not fall back to 'general-purpose'):
- gsd-designer — Produces LLD from HLD, elaborates into implementable components
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

Extract: `phase_dir`, `padded_phase`, `phase_number`, `phase_name`, `context_path`, `research_path`, `hld_path`, `lld_path`, `commit_docs`.

## Step 3: Check Prerequisites

**HLD.md must exist.** If `hld_path` is missing:
> Error: Phase {phase} has no HLD. Run `/gsd:hld {phase}` or `/gsd:design-phase {phase}` first.

**CONTEXT.md must exist.** If `context_path` is missing:
> Error: Phase {phase} has no CONTEXT.md. Run `/gsd:discuss-phase {phase}` first.

## Step 4: Check Existing LLD

If `lld_path` exists: Offer update/view/skip options.

## Step 5: Spawn gsd-designer

```
Task(
  prompt="<objective>
Create Low-Level Design for Phase {phase_number}: {phase_name}
Mode: LLD only (no Technical Spec in standalone mode)
</objective>

<files_to_read>
- {hld_path} (HIGH-LEVEL DESIGN — architecture to elaborate)
- {context_path} (USER DECISIONS — locked decisions are NON-NEGOTIABLE)
- {research_path} (RESEARCH — standard stack, patterns, code examples)
- {requirements_path} (REQUIREMENTS — traceable requirements)
</files_to_read>

<output>
Write to: {phase_dir}/{padded_phase}-LLD.md
Follow template: @~/.claude/get-shit-done/templates/lld.md
</output>",
  subagent_type="gsd-designer",
  model="{designer_model}",
  description="LLD Phase {phase}"
)
```

## Step 6: Handle Return

- `## LLD COMPLETE` — Display summary, offer: Technical Spec/Plan/Review/Done
- `## LLD BLOCKED` — Show issue, offer: Fix HLD/Skip/Manual

</process>

<success_criteria>
- [ ] Phase validated against roadmap
- [ ] HLD.md exists (required)
- [ ] CONTEXT.md exists (required)
- [ ] gsd-designer spawned with context
- [ ] LLD.md created in phase directory
- [ ] User knows next steps
</success_criteria>
