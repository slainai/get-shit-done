---
name: gsd:design-phase
description: Generate design documentation (HLD → LLD → Technical Spec) for a phase
argument-hint: "<phase> [--auto]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
---

<objective>
Orchestrate design documentation for a phase. Spawns gsd-architect → gsd-designer → gsd-design-reviewer in sequence.

**Lifecycle position:** After `/gsd:research-phase`, before `/gsd:plan-phase`.

**Produces:**
- `{phase_num}-HLD.md` — System architecture, component boundaries, interfaces
- `{phase_num}-LLD.md` — Concrete components, contracts, data models, execution flows
- `{phase_num}-TECHNICAL-SPEC.md` — File map, dependency order, test strategy

**Prerequisites:**
- CONTEXT.md (required — run `/gsd:discuss-phase` first)
- RESEARCH.md (recommended — run `/gsd:research-phase` first)

**Use `--auto` to chain:** discuss → research → design → plan → execute

**Orchestrator role:** Initialize, check prerequisites, spawn agents in sequence, handle revision loop (max 2), present status, auto-advance if `--auto`.
</objective>

<available_agent_types>
Valid GSD subagent types (use exact names — do not fall back to 'general-purpose'):
- gsd-architect — Produces HLD from CONTEXT.md and RESEARCH.md
- gsd-designer — Produces LLD and Technical Spec from HLD
- gsd-design-reviewer — Reviews design package for coherence
</available_agent_types>

<context>
Phase number: $ARGUMENTS (required, may include --auto flag)

Parse phase and flags from arguments before proceeding.
</context>

<execution_context>
@~/.claude/get-shit-done/workflows/design-phase.md
@~/.claude/get-shit-done/templates/hld.md
@~/.claude/get-shit-done/templates/lld.md
@~/.claude/get-shit-done/templates/technical-spec.md
</execution_context>
