<purpose>
Generate a High-Level Design (HLD) for a phase. Spawns gsd-architect with phase context.

Standalone HLD command. For most workflows, use `/gsd:design-phase` which orchestrates HLD → LLD → Technical Spec automatically.
</purpose>

<available_agent_types>
Valid GSD subagent types (use exact names — do not fall back to 'general-purpose'):
- gsd-architect — Produces HLD from CONTEXT.md and RESEARCH.md
</available_agent_types>

<process>

## Step 0: Resolve Model Profile

@~/.claude/get-shit-done/references/model-profile-resolution.md

Resolve model for:
- `gsd-architect`

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

Extract: `phase_dir`, `padded_phase`, `phase_number`, `phase_name`, `context_path`, `research_path`, `hld_path`, `commit_docs`.

## Step 3: Check Prerequisites

**CONTEXT.md must exist.** If `context_path` is missing:
> Error: Phase {phase} has no CONTEXT.md. Run `/gsd:discuss-phase {phase}` first.

**RESEARCH.md should exist.** If `research_path` is missing:
> Warning: No RESEARCH.md found. HLD will proceed with reduced confidence. Consider running `/gsd:research-phase {phase}` first.

## Step 4: Check Existing HLD

If `hld_path` exists: Offer update/view/skip options.

## Step 5: Spawn gsd-architect

```
Task(
  prompt="<objective>
Create High-Level Design for Phase {phase_number}: {phase_name}
</objective>

<files_to_read>
- {context_path} (USER DECISIONS — locked decisions are NON-NEGOTIABLE)
- {research_path} (RESEARCH — standard stack, patterns, pitfalls)
- {roadmap_path} (ROADMAP — phase goal and requirements)
- {requirements_path} (REQUIREMENTS — traceable requirements)
</files_to_read>

<output>
Write to: {phase_dir}/{padded_phase}-HLD.md
Follow template: @~/.claude/get-shit-done/templates/hld.md
</output>",
  subagent_type="gsd-architect",
  model="{architect_model}",
  description="HLD Phase {phase}"
)
```

## Step 6: Handle Return

- `## HLD COMPLETE` — Display summary, offer: LLD/Design full/Review/Done
- `## HLD BLOCKED` — Show issue, offer: Fix prerequisites/Skip/Manual

</process>

<success_criteria>
- [ ] Phase validated against roadmap
- [ ] CONTEXT.md exists (required)
- [ ] RESEARCH.md checked (recommended)
- [ ] gsd-architect spawned with context
- [ ] HLD.md created in phase directory
- [ ] User knows next steps
</success_criteria>
