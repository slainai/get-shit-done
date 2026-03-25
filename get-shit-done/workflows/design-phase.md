<purpose>
Orchestrate design documentation for a phase: HLD → LLD → Technical Spec → Design Review.

This is the primary design workflow. It spawns gsd-architect, gsd-designer, and gsd-design-reviewer in sequence, with a review loop for quality assurance.

**Lifecycle position:** After `/gsd:research-phase` (RESEARCH.md exists), before `/gsd:plan-phase` (PLAN.md).

```
discuss → CONTEXT.md → research → RESEARCH.md → design → HLD/LLD/TECH-SPEC → plan → PLAN.md
```
</purpose>

<available_agent_types>
Valid GSD subagent types (use exact names — do not fall back to 'general-purpose'):
- gsd-architect — Produces HLD from CONTEXT.md and RESEARCH.md
- gsd-designer — Produces LLD from HLD, and Technical Spec from LLD
- gsd-design-reviewer — Reviews design package for coherence
</available_agent_types>

<process>

## Step 0: Resolve Model Profiles

@~/.claude/get-shit-done/references/model-profile-resolution.md

Resolve models for:
- `gsd-architect`
- `gsd-designer`
- `gsd-design-reviewer`

## Step 1: Initialize

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init design-phase "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extract: `phase_dir`, `padded_phase`, `phase_number`, `phase_name`, `design_docs_enabled`, `context_path`, `research_path`, `hld_path`, `lld_path`, `tech_spec_path`, `has_hld`, `has_lld`, `has_tech_spec`, `commit_docs`, `roadmap_path`, `requirements_path`.

**If `design_docs_enabled` is false:**
> Design documentation is disabled. Enable with: `node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow.design_docs true`
> Or proceed directly to `/gsd:plan-phase {phase}`.

Exit workflow.

## Step 2: Check Prerequisites

**CONTEXT.md must exist.** If `context_path` is missing:
> Error: Phase {phase} has no CONTEXT.md. Run `/gsd:discuss-phase {phase}` first.

**RESEARCH.md should exist.** If `research_path` is missing:
> Warning: No RESEARCH.md found. Design docs will proceed with reduced confidence. Consider running `/gsd:research-phase {phase}` first.

Present: "Will generate design documentation for Phase {phase}: {name}"

## Step 3: Check Existing Design Docs

If ALL three exist (`has_hld`, `has_lld`, `has_tech_spec`):
> Design docs already exist for Phase {phase}. Options:
> 1. Regenerate all (overwrites existing)
> 2. Review existing (run design reviewer)
> 3. Skip to planning (`/gsd:plan-phase {phase}`)

Wait for response.

If SOME exist: Offer to continue from where left off (e.g., HLD exists → start from LLD).

## Step 4: Spawn gsd-architect (HLD)

Skip if `has_hld` is true and user chose to continue.

```
Task(
  prompt="<objective>
Create High-Level Design for Phase {phase_number}: {phase_name}
</objective>

<files_to_read>
- {context_path} (USER DECISIONS — locked decisions are NON-NEGOTIABLE)
- {research_path} (RESEARCH — standard stack, architecture patterns, pitfalls)
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

**Handle return:**
- `## HLD COMPLETE` → Display summary, continue to Step 5
- `## HLD BLOCKED` → Show blocker, ask user how to proceed, exit if unresolvable

## Step 5: Spawn gsd-designer (LLD + Technical Spec)

Skip if `has_lld` and `has_tech_spec` are true and user chose to continue.

```
Task(
  prompt="<objective>
Create Low-Level Design AND Technical Specification for Phase {phase_number}: {phase_name}
Mode: Full design (produce both LLD and Technical Spec)
</objective>

<files_to_read>
- {phase_dir}/{padded_phase}-HLD.md (HIGH-LEVEL DESIGN — architecture to elaborate)
- {context_path} (USER DECISIONS — locked decisions are NON-NEGOTIABLE)
- {research_path} (RESEARCH — standard stack, code patterns, pitfalls)
- {requirements_path} (REQUIREMENTS — traceable requirements)
</files_to_read>

<output>
Write LLD to: {phase_dir}/{padded_phase}-LLD.md
Follow template: @~/.claude/get-shit-done/templates/lld.md

Write Technical Spec to: {phase_dir}/{padded_phase}-TECHNICAL-SPEC.md
Follow template: @~/.claude/get-shit-done/templates/technical-spec.md
</output>",
  subagent_type="gsd-designer",
  model="{designer_model}",
  description="LLD+TechSpec Phase {phase}"
)
```

**Handle return:**
- `## LLD COMPLETE` → Display summary, continue to Step 6
- `## LLD BLOCKED` → If HLD revision needed, go back to Step 4 with revision instructions. Max 2 revision loops.

## Step 6: Spawn gsd-design-reviewer

```
Task(
  prompt="<objective>
Review design documentation package for Phase {phase_number}: {phase_name}
</objective>

<files_to_read>
- {context_path} (CONTEXT — source of truth for decisions)
- {research_path} (RESEARCH — source of truth for technology)
- {phase_dir}/{padded_phase}-HLD.md (Architecture)
- {phase_dir}/{padded_phase}-LLD.md (Detailed design)
- {phase_dir}/{padded_phase}-TECHNICAL-SPEC.md (Implementation map)
</files_to_read>",
  subagent_type="gsd-design-reviewer",
  model="{design_reviewer_model}",
  description="Review design Phase {phase}"
)
```

**Handle return:**
- `## DESIGN REVIEW PASSED` → Continue to Step 7
- `## DESIGN ISSUES FOUND` →
  - If blocking issues AND revision_count < 2: Send issues back to architect/designer for revision (Step 4 or 5). Increment revision_count.
  - If blocking issues AND revision_count >= 2: Present issues to user, ask how to proceed.
  - If warnings only: Present warnings, continue to Step 7.

## Step 7: Present Status

```
┌─────────────────────────────────────────────┐
│  GSD > PHASE {X}: {NAME} — DESIGNED         │
├─────────────────────────────────────────────┤
│  HLD:           ✅ {padded_phase}-HLD.md     │
│  LLD:           ✅ {padded_phase}-LLD.md     │
│  Technical Spec: ✅ {padded_phase}-TECHNICAL-SPEC.md │
│  Review:        ✅ Passed                    │
├─────────────────────────────────────────────┤
│  Components: {count}                         │
│  Interfaces: {count}                         │
│  Confidence: {level}                         │
└─────────────────────────────────────────────┘
```

**Next steps:**
- `/gsd:plan-phase {phase}` — Create executable plans from design docs
- `/gsd:design-phase {phase}` — Regenerate design docs
- Review individual docs with Read tool

## Step 8: Auto-advance

Parse `--auto` flag from arguments.

**If `--auto` is set:**
Invoke next workflow:
```
Skill(skill="gsd:plan-phase", args="{phase} --auto")
```

**If `--auto` is NOT set:**
Wait for user to choose next action.

</process>

<success_criteria>
- [ ] Phase validated against roadmap
- [ ] CONTEXT.md exists (required)
- [ ] RESEARCH.md checked (recommended)
- [ ] gsd-architect spawned → HLD.md created
- [ ] gsd-designer spawned → LLD.md + TECHNICAL-SPEC.md created
- [ ] gsd-design-reviewer spawned → design package reviewed
- [ ] Revision loop handled (max 2 iterations)
- [ ] Status banner displayed
- [ ] User knows next steps (or auto-advanced to plan-phase)
</success_criteria>
