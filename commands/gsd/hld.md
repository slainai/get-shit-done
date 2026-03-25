---
name: gsd:hld
description: Generate High-Level Design for a phase (standalone - usually use /gsd:design-phase instead)
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
Generate a High-Level Design document for a phase. Spawns gsd-architect agent with phase context.

**Note:** This is a standalone HLD command. For most workflows, use `/gsd:design-phase` which orchestrates HLD → LLD → Technical Spec automatically.

**Use this command when:**
- You want to generate/regenerate HLD without running the full design pipeline
- You want to iterate on architecture before committing to detailed design
- You need to revise HLD after LLD feedback

**Produces:** `{phase_num}-HLD.md` in the phase directory.

**Prerequisites:**
- CONTEXT.md (required)
- RESEARCH.md (recommended)
</objective>

<available_agent_types>
Valid GSD subagent types (use exact names — do not fall back to 'general-purpose'):
- gsd-architect — Produces HLD from CONTEXT.md and RESEARCH.md
</available_agent_types>

<context>
Phase number: $ARGUMENTS (required)

Normalize phase input in step 1 before any directory lookups.
</context>

<execution_context>
@~/.claude/get-shit-done/workflows/hld.md
@~/.claude/get-shit-done/templates/hld.md
</execution_context>
