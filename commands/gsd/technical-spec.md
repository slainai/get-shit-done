---
name: gsd:technical-spec
description: Generate Technical Specification for a phase (standalone - usually use /gsd:design-phase instead)
argument-hint: "[phase]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
---

<objective>
Generate a Technical Specification for a phase. Spawns gsd-designer agent in tech-spec mode.

**Note:** This is a standalone command. For most workflows, use `/gsd:design-phase` which orchestrates HLD → LLD → Technical Spec automatically.

**Use this command when:**
- You want to generate/regenerate the Technical Spec without re-running LLD
- You want to iterate on the implementation map before planning
- The file map or dependency order needs updating

**Produces:** `{phase_num}-TECHNICAL-SPEC.md` in the phase directory.

**Prerequisites:**
- LLD.md (required)
- HLD.md (required)
- CONTEXT.md (required)
</objective>

<available_agent_types>
Valid GSD subagent types (use exact names — do not fall back to 'general-purpose'):
- gsd-designer — Produces Technical Spec from LLD
</available_agent_types>

<context>
Phase number: $ARGUMENTS (required)

Normalize phase input in step 1 before any directory lookups.
</context>

<execution_context>
@~/.claude/get-shit-done/workflows/technical-spec.md
@~/.claude/get-shit-done/templates/technical-spec.md
</execution_context>
