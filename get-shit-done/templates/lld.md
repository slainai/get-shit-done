# Low-Level Design Template

Template for `.planning/phases/XX-name/{phase_num}-LLD.md` - elaborates HLD into implementable components and contracts.

**Purpose:** Make implementation-level responsibilities explicit without collapsing into an implementation plan. Every interface, data model, and failure path specified clearly enough to derive tasks.

**Mandatory inputs:**
- {phase_num}-HLD.md (system decomposition, interfaces, risks)
- {phase_num}-CONTEXT.md (locked decisions)
- {phase_num}-RESEARCH.md (standard stack, code patterns)

**Downstream consumers:**
- Technical Spec writer — Derives file map and dependency order from LLD
- `gsd-planner` — Uses LLD interfaces and contracts to create specific implementation tasks

---

## File Template

```markdown
# Phase [X]: [Name] - Low-Level Design

**Designed:** [date]
**Inherits from:** {phase_num}-HLD.md
**Confidence:** [HIGH/MEDIUM/LOW]

<scope>
## 1. Scope Inherited from HLD

[Phase boundary — copied from HLD Section 1. Anchors the design. No scope creep beyond this.]
</scope>

<component_inventory>
## 2. Concrete Component Inventory

[For each HLD component: concrete files, classes, functions, handlers. Where they live in the codebase.]

### [Component Name] (from HLD)

**Location:** `src/path/to/module/`
**Key files:**
| File | Purpose | New/Modified |
|------|---------|-------------|
| `file1.py` | [What it does] | New |
| `file2.py` | [What it does] | Modified |

**Public API:**
```python
class ClassName:
    def method_name(self, param: Type) -> ReturnType:
        """[What it does]"""
```

**Internal state:** [What this component manages — data structures, caches, connections]
</component_inventory>

<interfaces_and_contracts>
## 3. Detailed Interfaces and Contracts

[For each interface from HLD: exact signatures, request/response shapes, error codes.]

### [Component A] → [Component B]: [Interface Name]

**Signature:**
```python
def operation_name(
    param1: Type,
    param2: Type,
) -> Result[SuccessType, ErrorType]:
```

**Request shape:**
```python
@dataclass
class RequestName:
    field1: str
    field2: int
```

**Response shape:**
```python
@dataclass
class ResponseName:
    field1: str
    status: Status
```

**Error codes:**
| Code | Condition | Caller should |
|------|-----------|---------------|
| NOT_FOUND | [When] | [What to do] |
| INVALID_INPUT | [When] | [What to do] |

**Invariants:** [Conditions that must always hold]
</interfaces_and_contracts>

<data_model>
## 4. Data Model and State Design

[Schemas, types, state shapes. Database tables, DTOs, domain entities.]

### [Entity/Table Name]
```python
@dataclass
class EntityName:
    id: EntityId
    field1: str
    field2: int
    created_at: datetime
```

**Ownership:** [Which component owns this data]
**Persistence:** [Database table, in-memory, file, etc.]
**State transitions:** [Valid state changes, if applicable]

| From State | To State | Trigger | Validation |
|-----------|----------|---------|------------|
| [state] | [state] | [event] | [rules] |
</data_model>

<execution_flows>
## 5. Detailed Execution Flows

[For each key operation: step-by-step with exact function calls.]

### Flow: [Operation Name]

```
1. Router receives request → validates input via InputValidator.validate()
2. Handler calls Service.operation(validated_input)
3. Service loads entity via Repository.get_by_id(id)
4. Service applies business rule → Entity.apply_change(params)
5. Service persists via Repository.save(entity)
6. Service emits event → EventBus.publish(EntityChanged(entity))
7. Handler returns response → ResponseMapper.to_dto(entity)
```

**Happy path duration:** [Expected timing]
**Key decision points:** [Where branching occurs]
</execution_flows>

<failure_handling>
## 6. Failure Handling and Edge Cases

[For each failure mode: what triggers it, what happens, how to recover.]

### Failure: [Name]
**Trigger:** [What causes this]
**Detection:** [How to detect it]
**Response:** [What the system does]
**Recovery:** [How to get back to healthy state]
**User impact:** [What the user sees]

### Edge Case: [Name]
**Condition:** [When this occurs]
**Expected behavior:** [What should happen]
**Why it matters:** [Why this isn't obvious]
</failure_handling>

<testing_implications>
## 7. Testing Implications

[What needs tests. Key scenarios per component.]

### [Component Name]

**Unit tests:**
| Test | Scenario | Key assertion |
|------|----------|---------------|
| `test_[name]` | [What's tested] | [What must be true] |

**Integration tests:**
| Test | Components involved | Key assertion |
|------|-------------------|---------------|
| `test_[name]` | [Which components] | [What must be true] |

**Test commands:**
```bash
# Run component tests
pytest tests/path/to/test_file.py -x

# Run integration tests
pytest tests/integration/test_flow.py -x
```
</testing_implications>

<open_issues>
## 8. Open Issues Requiring Architectural Feedback

[Anything unresolved. If LLD reveals HLD needs changing — flag it here, don't hide it.]

- [ ] **[Issue]:** [Description]. **Impact:** [What changes if resolved differently]. **Recommendation:** [Suggested resolution].
</open_issues>
```

---

## Quality Gate

The LLD is NOT done until ALL of the following are true:

- [ ] Every HLD component that needs implementation detail is elaborated
- [ ] Interfaces and responsibilities are clear enough to derive implementation tasks
- [ ] Data ownership and state transitions are explicit
- [ ] Failure paths and operational behaviors are defined
- [ ] No detailed design decision quietly changes the HLD architecture
- [ ] CONTEXT.md locked decisions are respected in all component designs
- [ ] RESEARCH.md standard stack is used for library/framework choices

## What LLD Must NOT Become

- **Not a rewrite of HLD** — HLD defines boundaries; LLD fills in details within those boundaries
- **Not code generation** — Use code sketches to clarify contracts, not to write implementation
- **Not an implementation task list** — That's what PLAN.md does
- **Not a place to invent architecture** — If you need new components not in HLD, revise HLD first

## Collaboration Rules

- Start from HLD structure and elaborate it — don't reinvent
- If a required detail cannot be resolved without changing architecture, revise HLD first
- Keep feeding unresolved boundary problems back up — don't hide mismatches
- End with a design the planner can translate into tasks without new architecture work
