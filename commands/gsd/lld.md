---
name: gsd:lld
description: Generate Low-Level Design for a phase (standalone - usually use /gsd:design-phase instead)
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
Generate a Low-Level Design document for a phase. Spawns gsd-designer agent with phase context.

**Note:** This is a standalone LLD command. For most workflows, use `/gsd:design-phase` which orchestrates HLD → LLD → Technical Spec automatically.

**Use this command when:**
- You want to generate/regenerate LLD without running the full design pipeline
- You want to iterate on detailed design without re-running HLD
- You need to revise LLD after plan checker feedback

**Produces:** `{phase_num}-LLD.md` in the phase directory.

**Prerequisites:**
- HLD.md (required)
- CONTEXT.md (required)
- RESEARCH.md (recommended)
</objective>

<available_agent_types>
Valid GSD subagent types (use exact names — do not fall back to 'general-purpose'):
- gsd-designer — Produces LLD from HLD
</available_agent_types>

<context>
Phase number: $ARGUMENTS (required)

Normalize phase input in step 1 before any directory lookups.
</context>

<execution_context>
@~/.claude/get-shit-done/workflows/lld.md
@~/.claude/get-shit-done/templates/lld.md
</execution_context>
