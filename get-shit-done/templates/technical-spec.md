# Technical Specification Template

Template for `.planning/phases/XX-name/{phase_num}-TECHNICAL-SPEC.md` - bridges design documents to executable plans.

**Purpose:** Translate LLD into a concrete implementation map. Contains the file inventory, dependency order, and test strategy that the planner uses to create task waves.

**Mandatory inputs:**
- {phase_num}-HLD.md (architecture overview)
- {phase_num}-LLD.md (detailed design)
- {phase_num}-CONTEXT.md (locked decisions)
- {phase_num}-RESEARCH.md (standard stack)

**Downstream consumers:**
- `gsd-planner` — Uses file map for task `files_modified`, dependency order for wave assignment, test strategy for verification steps
- `gsd-plan-checker` — Validates plan coverage against Technical Spec component inventory

---

## File Template

```markdown
# Phase [X]: [Name] - Technical Specification

**Prepared:** [date]
**Inherits from:** {phase_num}-LLD.md
**Ready for planning:** [YES/NO — if NO, list blockers]

<component_inventory>
## 1. Component Inventory

[Summary table derived from LLD Section 2. Quick reference for the planner.]

| Component | Key Files | New/Modified | Complexity | LLD Section |
|-----------|-----------|-------------|------------|-------------|
| [Name] | `path/file.py` | New | Low/Med/High | §2.[N] |
| [Name] | `path/file.py`, `path/file2.py` | Modified | Med | §2.[N] |
</component_inventory>

<file_map>
## 2. File Map

[Every file that will be created or modified. Grouped by component. This is what the planner uses to assign `files_modified` in tasks.]

### [Component Name]

| File | Action | What Changes | Depends On |
|------|--------|--------------|------------|
| `src/path/to/new_file.py` | Create | [Purpose] | — |
| `src/path/to/existing.py` | Modify (lines ~50-80) | [What changes and why] | `new_file.py` |
| `tests/path/to/test_file.py` | Create | [Test coverage for component] | `new_file.py` |

### [Component Name]
...

### Shared / Cross-cutting
| File | Action | What Changes | Depends On |
|------|--------|--------------|------------|
| `src/shared/types.py` | Modify | [New types for this phase] | — |
</file_map>

<dependency_order>
## 3. Dependency Order

[Which components must be built first. Maps directly to plan waves.]

```
Wave 1 (no dependencies):
  - [Component A] — foundational types, interfaces
  - [Component B] — independent utility

Wave 2 (depends on Wave 1):
  - [Component C] — uses interfaces from A
  - [Component D] — uses utility from B

Wave 3 (depends on Wave 2):
  - [Component E] — orchestrates C and D
  - Integration wiring

Wave 4 (final):
  - End-to-end verification
  - Cleanup and documentation
```

**Critical path:** [The longest dependency chain — this determines minimum phases]
**Parallelizable:** [Which wave items can run simultaneously]
</dependency_order>

<test_strategy>
## 4. Test Strategy

[Per component: what test files, what coverage, test commands.]

### [Component Name]
| Test File | Type | Covers | Command |
|-----------|------|--------|---------|
| `tests/unit/test_[name].py` | Unit | [LLD §3 contracts] | `pytest tests/unit/test_[name].py -x` |
| `tests/integration/test_[flow].py` | Integration | [LLD §5 flows] | `pytest tests/integration/test_[flow].py -x` |

### Verification Commands
```bash
# Quick check (per task commit)
pytest tests/unit/ -x --timeout=30

# Full suite (per wave completion)
pytest tests/ -x

# Coverage check
pytest tests/ --cov=src/path --cov-fail-under=80
```

### Coverage Targets
| Component | Target | Rationale |
|-----------|--------|-----------|
| [Name] | 90% | Core business logic |
| [Name] | 70% | Integration glue |
</test_strategy>

<integration_points>
## 5. Integration Points

[How components connect after individual implementation. What needs wiring together.]

### [Integration Point Name]
**Components:** [A] + [B]
**Wiring:** [What connects them — imports, dependency injection, config, etc.]
**Verification:** [How to test the integration works]

### Registration / Bootstrap
[If components need to be registered, imported, or configured to work together — list the exact changes.]
</integration_points>

<implementation_notes>
## 6. Implementation Notes

[Gotchas, resolved LLD open issues, guidance for the planner.]

### Resolved from LLD Open Issues
| LLD Issue | Resolution | Impact on Plan |
|-----------|-----------|----------------|
| [Issue from LLD §8] | [How it was resolved] | [What the planner should know] |

### Gotchas
- **[Gotcha]:** [What to watch out for and why]

### Planner Guidance
- [Specific instruction for how tasks should be structured]
- [Any ordering constraints beyond the dependency graph]
</implementation_notes>
```

---

## Quality Gate

The Technical Spec is NOT done until ALL of the following are true:

- [ ] Every LLD component appears in the component inventory
- [ ] File map covers all files that need to be created or modified
- [ ] Dependency order is acyclic and complete
- [ ] Test strategy covers all LLD contracts and flows
- [ ] Integration points are identified with verification steps
- [ ] LLD open issues are resolved or escalated
- [ ] Ready for planning is YES

## What Technical Spec Must NOT Become

- **Not a design document** — Design decisions belong in HLD/LLD. This is purely implementation mapping.
- **Not a plan** — Don't write tasks. Write the MAP that the planner uses to create tasks.
- **Not aspirational** — Every file listed must be needed. Every component must trace to LLD.
