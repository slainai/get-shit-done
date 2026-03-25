# Design Documentation Reference

## Overview

Design documentation adds a formal architecture/design layer between research and planning:

```
discuss → CONTEXT.md → research → RESEARCH.md → design → HLD/LLD/TECH-SPEC → plan → PLAN.md
```

**Config key:** `workflow.design_docs` (default: `true`)

## When to Use Design Docs

**Enable (default) for:**
- Complex systems with multiple interacting components
- Multi-component phases (3+ components)
- Projects where architectural decisions matter
- Team handoffs where design context needs to persist
- Phases that touch existing architecture

**Disable for:**
- Simple bug fixes
- Single-file changes
- Configuration-only changes
- Rapid prototyping / throwaway code

## Document Lifecycle

| Document | Produced By | Consumed By | Contains |
|----------|-------------|-------------|----------|
| HLD | `gsd-architect` | `gsd-designer`, `gsd-planner` | Architecture, component boundaries, interfaces |
| LLD | `gsd-designer` | `gsd-planner` | Concrete components, contracts, data models, flows |
| Technical Spec | `gsd-designer` | `gsd-planner`, `gsd-plan-checker` | File map, dependency order, test strategy |

## File Naming Convention

All design docs follow the standard GSD naming in `.planning/phases/XX-name/`:

```
{padded_phase}-HLD.md            # e.g., 01-HLD.md
{padded_phase}-LLD.md            # e.g., 01-LLD.md
{padded_phase}-TECHNICAL-SPEC.md # e.g., 01-TECHNICAL-SPEC.md
```

## Commands

| Command | Purpose |
|---------|---------|
| `/gsd:design-phase <phase>` | Full pipeline: HLD → LLD → Technical Spec → Review |
| `/gsd:hld <phase>` | Standalone HLD only |
| `/gsd:lld <phase>` | Standalone LLD only (requires HLD) |
| `/gsd:technical-spec <phase>` | Standalone Technical Spec only (requires LLD) |

## Auto-Advance Chain

With `--auto`: discuss → research → **design** → plan → execute

The design phase slots into the chain between research and planning. When `workflow.design_docs` is false, auto-advance skips directly from research to plan.

## Quality Standards

### HLD Quality Gate
- Every major component named with clear responsibility
- Architecture explainable WITHOUT implementation details
- Interfaces visible at boundary level
- CONTEXT.md locked decisions honored
- RESEARCH.md standard stack used

### LLD Quality Gate
- Every HLD component elaborated
- Interfaces clear enough for task derivation
- Data ownership and state transitions explicit
- Failure paths defined
- No quiet HLD architecture changes

### Design Review Checklist
1. HLD components all appear in LLD inventory
2. LLD interfaces match HLD interface overview
3. Technical Spec file map covers all LLD components
4. No contradictions between documents
5. CONTEXT.md locked decisions respected throughout
6. RESEARCH.md standard stack used consistently
7. No scope creep beyond CONTEXT.md domain boundary

## Model Profiles

| Agent | Quality | Balanced | Budget |
|-------|---------|----------|--------|
| `gsd-architect` | opus | opus | sonnet |
| `gsd-designer` | opus | sonnet | sonnet |
| `gsd-design-reviewer` | opus | sonnet | haiku |

The architect uses opus in balanced mode because HLD requires strong architectural reasoning. Designer and reviewer can use sonnet effectively.

## Revision Loop

The design-phase orchestrator supports a revision loop (max 2 iterations):
1. If design reviewer finds blocking issues → send back to architect/designer
2. If blocking issues persist after 2 iterations → present to user

This keeps the design cycle bounded while allowing for self-correction.
