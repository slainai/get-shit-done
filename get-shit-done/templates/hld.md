# High-Level Design Template

Template for `.planning/phases/XX-name/{phase_num}-HLD.md` - defines system architecture and component boundaries for a phase.

**Purpose:** Turn CONTEXT.md decisions and RESEARCH.md findings into a coherent system shape. Defines what the major pieces are and how they interact, without implementation detail.

**Mandatory inputs:**
- CONTEXT.md (locked decisions, discretion areas, deferred ideas)
- RESEARCH.md (standard stack, architecture patterns, pitfalls)

**Downstream consumers:**
- `gsd-designer` — Elaborates HLD components into concrete implementations (LLD)
- `gsd-planner` — Uses HLD architecture to structure plan waves and task groupings

---

## File Template

```markdown
# Phase [X]: [Name] - High-Level Design

**Designed:** [date]
**Confidence:** [HIGH/MEDIUM/LOW]
**Inputs:** {phase_num}-CONTEXT.md, {phase_num}-RESEARCH.md

<goal_and_scope>
## 1. Goal and Scope

[What this phase delivers — inherited from CONTEXT.md domain boundary. Not a restatement of the spec — focus on what the ARCHITECTURE needs to achieve.]

**In scope:** [Concrete boundaries]
**Out of scope:** [From CONTEXT.md deferred ideas — explicitly listed to prevent scope creep]
</goal_and_scope>

<architectural_context>
## 2. Architectural Context

[Where this fits in the existing system. What already exists. What this phase adds or changes.]

**Existing components touched:** [List with file paths]
**New components introduced:** [List]
**External dependencies:** [APIs, services, libraries from RESEARCH.md standard stack]
</architectural_context>

<system_decomposition>
## 3. System Decomposition

[Major components needed for this phase. 3-7 components is typical. Each named, with clear responsibility and key interfaces.]

### Component: [Name]
**Responsibility:** [One sentence — what it owns]
**Key interfaces:** [What it exposes, what it consumes]
**Collaborates with:** [Other components it talks to]

### Component: [Name]
...
</system_decomposition>

<responsibility_matrix>
## 4. Component Responsibility Matrix

| Component | Owns | Collaborates With | Does NOT Do |
|-----------|------|-------------------|-------------|
| [Name] | [What it's solely responsible for] | [Who it works with] | [Explicit exclusions] |
</responsibility_matrix>

<interface_overview>
## 5. Interface and Integration Overview

[Key interfaces between components. Data that flows across boundaries.]

### [Component A] → [Component B]
**Data:** [What flows]
**Protocol:** [How — function call, event, API, message queue]
**Contract:** [Key constraints — types, validation, error handling]
</interface_overview>

<data_and_control_flow>
## 6. Data Flow and Control Flow

[How data moves through the system for the primary use case. Numbered steps, end-to-end.]

### Primary Flow: [Use Case Name]
1. [Entry point] receives [input]
2. [Component A] processes [what] and passes to [Component B]
3. ...
N. [Final component] returns [output]

### Secondary Flow: [If applicable]
...
</data_and_control_flow>

<cross_cutting_concerns>
## 7. Cross-Cutting Concerns

[Only concerns relevant to THIS phase. Don't list everything — list what matters.]

| Concern | Approach | Component(s) Affected |
|---------|----------|----------------------|
| [e.g., Error handling] | [Strategy] | [Which components] |
| [e.g., Observability] | [Strategy] | [Which components] |
</cross_cutting_concerns>

<risks_and_tradeoffs>
## 8. Risks and Trade-offs

### Decision: [What was decided]
**Traded away:** [What we gave up]
**Why:** [Rationale — from CONTEXT.md locked decisions or RESEARCH.md recommendations]

### Risk: [What could go wrong]
**Likelihood:** [HIGH/MEDIUM/LOW]
**Mitigation:** [How to handle it]
</risks_and_tradeoffs>

<preconditions_for_lld>
## 9. Preconditions for LLD

[What the designer needs to know. Open questions. Things to resolve in detailed design.]

- [ ] [Open question 1]
- [ ] [Decision that needs more detail]
- [ ] [Area where multiple approaches are viable — designer should pick]
</preconditions_for_lld>
```

---

## Quality Gate

The HLD is NOT done until ALL of the following are true:

- [ ] Every major component is named and has a clear responsibility
- [ ] The architecture can be explained without implementation details
- [ ] Interfaces between components are visible at a boundary level
- [ ] The decomposition supports the phase goal without introducing unnecessary subsystems
- [ ] The document gives LLD a stable structure to elaborate
- [ ] CONTEXT.md locked decisions are honored throughout
- [ ] RESEARCH.md standard stack is used (not alternatives)

## What HLD Must NOT Become

- **Not a restatement of CONTEXT.md** — CONTEXT captures decisions; HLD captures architecture
- **Not a task checklist** — That's what PLAN.md is for
- **Not line-by-line implementation detail** — That's what LLD is for
- **Not pseudocode-heavy** — Use prose, tables, and diagrams. Code belongs in LLD

## Collaboration Rules

- If HLD exposes missing information in CONTEXT.md, feed that back — don't guess
- If HLD and emerging LLD disagree, treat as design problem and iterate
- Reuse existing architecture patterns from RESEARCH.md where they fit
