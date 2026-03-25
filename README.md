# SlainAI — GSD Standard Operating Procedure

---

# Part 1: Usage

Everything a developer needs to install GSD, configure their environment, and use it daily.

---

## 1.1 Prerequisites

- **Node.js** 18+ (`node -v` to check)
- **Git** (`git --version`)
- **Claude Code** installed and working (`claude` in terminal)
- Access to `github.com/slainai/get-shit-done`

---

## 1.2 First-Time Setup

### One-Command Install (Mac/Linux)

Copy-paste this entire block. Set `GSD_CLONE_DIR` to wherever you want the temporary clone:

```bash
# ── CONFIG: change this if you want the clone somewhere else ──
GSD_CLONE_DIR="${HOME}/tmp/gsd-install"

# ── INSTALL ──
rm -rf "$GSD_CLONE_DIR"
git clone https://github.com/slainai/get-shit-done.git "$GSD_CLONE_DIR"
cd "$GSD_CLONE_DIR" && npm install && npm run build:hooks && node bin/install.js --claude --global
cd ~

# ── CLEANUP: clone is no longer needed after install ──
rm -rf "$GSD_CLONE_DIR"

# ── DEFAULTS ──
mkdir -p "${HOME}/.gsd"
cat > "${HOME}/.gsd/defaults.json" << 'DEFAULTS'
{
  "model_profile": "balanced",
  "commit_docs": true,
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "design_docs": true,
    "nyquist_validation": true,
    "auto_advance": false
  },
  "git": {
    "branching_strategy": "phase"
  }
}
DEFAULTS

echo "Done. Open a new Claude Code session and run /gsd:help"
```

### One-Command Install (Windows — PowerShell)

```powershell
# ── CONFIG: change this if you want the clone somewhere else ──
$GSD_CLONE_DIR = "$env:USERPROFILE\tmp\gsd-install"

# ── INSTALL ──
if (Test-Path $GSD_CLONE_DIR) { Remove-Item -Recurse -Force $GSD_CLONE_DIR }
git clone https://github.com/slainai/get-shit-done.git $GSD_CLONE_DIR
Push-Location $GSD_CLONE_DIR
npm install; npm run build:hooks; node bin/install.js --claude --global
Pop-Location

# ── CLEANUP ──
Remove-Item -Recurse -Force $GSD_CLONE_DIR

# ── DEFAULTS ──
$gsdDir = "$env:USERPROFILE\.gsd"
if (-not (Test-Path $gsdDir)) { New-Item -ItemType Directory -Path $gsdDir | Out-Null }
@'
{
  "model_profile": "balanced",
  "commit_docs": true,
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "design_docs": true,
    "nyquist_validation": true,
    "auto_advance": false
  },
  "git": {
    "branching_strategy": "phase"
  }
}
'@ | Set-Content "$gsdDir\defaults.json" -Encoding UTF8

Write-Host "Done. Open a new Claude Code session and run /gsd:help"
```

### What Gets Installed

The installer copies everything into `~/.claude/` (or `%USERPROFILE%\.claude\` on Windows):

```
~/.claude/
├── commands/gsd/           # 36+ slash commands (/gsd:new-project, etc.)
├── agents/                 # 21 agent definitions (gsd-planner, gsd-architect, etc.)
├── hooks/                  # 5 session hooks (update check, context monitor, prompt guard, etc.)
└── get-shit-done/
    ├── bin/                # CLI tools (gsd-tools.cjs + libraries)
    ├── workflows/          # 39+ workflow orchestrations
    ├── templates/          # Document templates (HLD, LLD, CONTEXT, etc.)
    └── references/         # Reference docs (model profiles, design docs, etc.)
```

The clone is only needed during install. After `install.js` copies files, the clone can be deleted.

### Verify

Open a **new** Claude Code session:

```
/gsd:help
```

You should see the full command list including `/gsd:design-phase`, `/gsd:hld`, `/gsd:lld`, `/gsd:technical-spec`.

---

## 1.3 Updating

When the team pushes changes to our fork, re-run the install:

### Mac/Linux

```bash
GSD_CLONE_DIR="${HOME}/tmp/gsd-install"
rm -rf "$GSD_CLONE_DIR"
git clone https://github.com/slainai/get-shit-done.git "$GSD_CLONE_DIR"
cd "$GSD_CLONE_DIR" && npm install && npm run build:hooks && node bin/install.js --claude --global
cd ~ && rm -rf "$GSD_CLONE_DIR"
echo "Updated. Restart Claude Code session to pick up changes."
```

### Windows (PowerShell)

```powershell
$GSD_CLONE_DIR = "$env:USERPROFILE\tmp\gsd-install"
if (Test-Path $GSD_CLONE_DIR) { Remove-Item -Recurse -Force $GSD_CLONE_DIR }
git clone https://github.com/slainai/get-shit-done.git $GSD_CLONE_DIR
Push-Location $GSD_CLONE_DIR
npm install; npm run build:hooks; node bin/install.js --claude --global
Pop-Location
Remove-Item -Recurse -Force $GSD_CLONE_DIR
Write-Host "Updated. Restart Claude Code session to pick up changes."
```

### Project-Local Install (Optional)

If a project needs its own GSD version instead of global:

```bash
GSD_CLONE_DIR="${HOME}/tmp/gsd-install"
rm -rf "$GSD_CLONE_DIR"
git clone https://github.com/slainai/get-shit-done.git "$GSD_CLONE_DIR"
cd "$GSD_CLONE_DIR" && npm install && npm run build:hooks && cd -
cd /path/to/your-project
node "$GSD_CLONE_DIR/bin/install.js" --claude --local
rm -rf "$GSD_CLONE_DIR"
```

This creates `.claude/commands/gsd/` inside the project. Takes priority over global install.

---

## 1.4 Configuring a Project

### Interactive

```
/gsd:settings
```

Walks through: model profile, research, plan checker, verifier, design docs, Nyquist validation, UI phase, auto-advance.

### Model Profiles

| Profile | When to Use | Architect | Planner | Executor | Researcher |
|---------|------------|-----------|---------|----------|------------|
| **quality** | Complex systems, critical architecture | Opus | Opus | Opus | Opus |
| **balanced** | Default — most work | Opus | Opus | Sonnet | Sonnet |
| **budget** | Batch work, simple tasks | Sonnet | Sonnet | Sonnet | Haiku |

Switch: `/gsd:set-profile balanced`

**Recommendation:** `balanced` for everything. `quality` for `new-project` and `design-phase` on hard phases only.

---

## 1.5 The Lifecycle

```
new-project
    │
    ▼
┌── Per Phase (repeat) ───────────────────────────────────────────┐
│                                                                  │
│  1. discuss ─→ 2. research ─→ 3. design ─→ 4. plan ─→ 5. execute ─→ 6. verify │
│     CONTEXT      RESEARCH       HLD          PLAN       SUMMARY      UAT       │
│                                 LLD                                            │
│                                 TECH-SPEC                                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
    │
    ▼
audit-milestone → complete-milestone → new-milestone
```

---

## 1.6 Starting a Project

| Situation | Command |
|-----------|---------|
| New repo, no code | `/gsd:new-project` |
| Existing codebase | `/gsd:map-codebase` then `/gsd:new-project` |
| Next milestone | `/gsd:new-milestone` |
| Joining mid-project | `/gsd:progress` |

Creates `.planning/` with PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md, config.json.

---

## 1.7 Per-Phase Workflow

### 1. Discuss — Lock Decisions

```
/gsd:discuss-phase <N>
```

Surfaces gray areas, locks your answers into `CONTEXT.md`.

**Review:** Read CONTEXT.md. Decisions correct? Anything missing?

**Good practice:** Be specific. "Card layout, infinite scroll, max 3 columns" > "whatever works." Vague input produces vague plans.

**Skip when:** Never for real work. Use `/gsd:fast` or `/gsd:quick` for throwaway tasks.

---

### 2. Research — Investigate the Domain

```
/gsd:research-phase <N>
```

Investigates standard stack, patterns, pitfalls. Runs in a fresh subagent (200k context) so your main session stays clean.

**Review:** Skim standard stack. Does it match your project?

**Skip when:** Domain is very familiar AND stack is obvious. Even then, running it doesn't hurt.

> Note: `/gsd:plan-phase` includes research automatically. Standalone is for reviewing BEFORE committing to design.

---

### 3. Design — Architect the Solution

```
/gsd:design-phase <N>
```

**Our key addition.** ON by default. Spawns three agents:

| Step | Agent | Output | Contains |
|------|-------|--------|----------|
| 1 | gsd-architect | `HLD.md` | Components, responsibilities, interfaces, data flow, risks |
| 2 | gsd-designer | `LLD.md` | Concrete files/classes, exact signatures, data models, failure handling |
| 3 | gsd-designer | `TECHNICAL-SPEC.md` | File map, dependency order (→ plan waves), test strategy |
| 4 | gsd-design-reviewer | (review) | Coherence check, max 2 revision loops |

**Review — this is the most important gate:**

| Doc | Time | What to Check |
|-----|------|---------------|
| HLD | 5 min | Component boundaries right? Interfaces sensible? Over-engineered? |
| LLD | 5 min | Signatures concrete (not "returns data")? Failure handling complete? |
| Tech Spec | 3 min | File map correct? Dependency order right? |

**Iterate individually:**
```
/gsd:hld <N>              # redo HLD
/gsd:lld <N>              # redo LLD (requires HLD)
/gsd:technical-spec <N>   # redo Tech Spec (requires LLD)
```

**Skip when:** Single-component changes, config-only, trivial CRUD. Go straight to `/gsd:plan-phase` — it warns but continues.

---

### 4. Plan — Create Executable Tasks

```
/gsd:plan-phase <N>
```

Creates PLAN.md files with tasks grouped into waves. When design docs exist, plans align with HLD/LLD/Tech Spec.

**Review:** File paths correct? Wave order sensible? Acceptance criteria testable?

**Flags:** `--research` (re-research), `--prd path/spec.md` (use PRD), `--reviews` (incorporate review feedback)

---

### 5. Execute — Build It

```
/gsd:execute-phase <N>
```

Runs plans in wave order. Commits atomically per task.

**Review:** `git log --oneline -20` — commits atomic and descriptive?

**Flags:** `--wave 1` (pace large phases), `--interactive` (pause between tasks), `--gaps-only` (fix plans only)

---

### 6. Verify — Validate

```
/gsd:verify-work <N>
```

Conversational UAT. If issues found → creates fix plans → `/gsd:execute-phase <N> --gaps-only`

---

### 7. Ship and Complete

```
/gsd:ship <N>              # create PR
/gsd:audit-milestone        # validate all phases together
/gsd:complete-milestone     # archive, tag, next
```

---

## 1.8 Quick Tasks

| Situation | Command |
|-----------|---------|
| One-line fix, typo | `/gsd:fast "description"` |
| Small task, want tracking | `/gsd:quick "description"` |
| Needs investigation | `/gsd:quick --research "desc"` |
| Has design decisions | `/gsd:quick --discuss "desc"` |
| Don't know which command | `/gsd:do "freeform text"` |

---

## 1.9 Decision Tree — When to Use What

```
Full feature / new capability?
├── YES → full lifecycle: discuss → research → design → plan → execute → verify
│
├── Small task, not part of roadmap?
│   ├── Trivial (<2 min) → /gsd:fast
│   ├── Needs tracking   → /gsd:quick
│   └── Needs research   → /gsd:quick --research
│
├── Bug fix?
│   ├── Know the fix     → /gsd:fast
│   └── Need to investigate → /gsd:debug
│
├── Urgent mid-phase?
│   → /gsd:pause-work → /gsd:insert-phase → work → /gsd:resume-work
│
├── Don't know what's next?
│   → /gsd:next (auto-detects)
│
└── Hands-off execution?
    → /gsd:discuss-phase <N> --auto (chains everything)
```

---

## 1.10 Navigation

| Command | When |
|---------|------|
| `/gsd:next` | Don't remember next step |
| `/gsd:progress` | Starting new session, need status |
| `/gsd:health` | Something feels off |
| `/gsd:debug` | Bug in your code |
| `/gsd:forensics` | Bug in the GSD workflow |
| `/gsd:pause-work` / `/gsd:resume-work` | Stopping/resuming mid-phase |

---

## 1.11 Review Guide

### What to Check at Each Stage

| Stage | File | Time | Red Flags |
|-------|------|------|-----------|
| Discuss | `CONTEXT.md` | 2 min | Vague decisions ("appropriate error handling") |
| Research | `RESEARCH.md` | 2 min | Wrong stack, unrealistic pitfalls |
| Design — HLD | `HLD.md` | 5 min | Too many components, unclear boundaries |
| Design — LLD | `LLD.md` | 5 min | Vague signatures, missing failure handling |
| Design — TechSpec | `TECHNICAL-SPEC.md` | 3 min | Wrong file paths, cyclic dependencies |
| Plan | `PLAN.md` | 3 min | Untestable acceptance criteria |
| Execute | `git log` | 2 min | Non-atomic commits, unexpected files |
| Verify | `UAT.md` | 2 min | Failing scenarios |

### When to Push Back

- **Vague decisions** → re-discuss. "Show toast for user errors, retry for system errors" beats "handle errors appropriately."
- **10+ components in HLD** → phase is too big, split it.
- **LLD invents components not in HLD** → re-run `/gsd:hld` first.
- **Untestable criteria** → reject. "Response < 200ms" beats "good performance."
- **Unexpected file changes** → investigate scope creep.

---

## 1.12 Artifacts Map

```
.planning/
├── PROJECT.md                    # Vision and constraints
├── REQUIREMENTS.md               # Traceable REQ-IDs
├── ROADMAP.md                    # Phase structure
├── STATE.md                      # Living memory
├── config.json                   # Workflow toggles
├── phases/
│   └── 01-name/
│       ├── 01-CONTEXT.md         # Locked decisions
│       ├── 01-RESEARCH.md        # Domain research
│       ├── 01-HLD.md             # High-Level Design
│       ├── 01-LLD.md             # Low-Level Design
│       ├── 01-TECHNICAL-SPEC.md  # Implementation map
│       ├── 01-1-PLAN.md          # Executable plan
│       ├── 01-1-SUMMARY.md       # Execution summary
│       ├── 01-VERIFICATION.md    # Automated verification
│       └── 01-UAT.md             # User acceptance
├── research/                     # Project-level research
├── codebase/                     # Codebase analysis
├── quick/                        # Ad-hoc tasks
└── debug/                        # Debug sessions
```

---

# Part 2: Repo Maintenance

How we maintain the `slainai/get-shit-done` fork internally.

---

## 2.1 Our Fork

| | |
|---|---|
| **Repo** | `github.com/slainai/get-shit-done` |
| **Upstream** | `github.com/gsd-build/get-shit-done` |
| **Addition** | Design documentation system (HLD/LLD/TechSpec) — ON by default |

We do NOT publish to npm. Everyone installs from our repo via the scripts in Part 1.

---

## 2.2 What We Changed

**New files (16):** 3 agents, 4 workflows, 4 commands, 3 templates, 1 reference, 1 test update — all additive, no merge conflicts.

**Modified files (8):** Small insertions into config.cjs, core.cjs, model-profiles.cjs, init.cjs, gsd-tools.cjs, plan-phase.md, settings.md, research-phase.md.

---

## 2.3 Syncing with Upstream

```bash
# Maintainer keeps a persistent clone
cd ~/Desktop/Workspaces/get-shit-done   # or wherever you keep it
git fetch upstream
git merge upstream/main
node scripts/run-tests.cjs              # must be 0 failures
git push origin main
```

Conflicts are rare — our additions are mostly new files. When they happen in the 5 modified JS files, keep both changes (ours add `design_docs`, theirs add other keys).

---

## 2.4 Making Changes

1. Branch: `git checkout -b feature/name`
2. Code
3. Test: `node scripts/run-tests.cjs` — **0 failures required**
4. PR to `slainai/get-shit-done` main
5. After merge, notify team (see template below)

### Conventions

- New agent with `Write` tool → needs anti-heredoc instruction + commented `# hooks:` in frontmatter
- New agent → update expected list in `tests/copilot-install.test.cjs`
- New config key → add to `VALID_CONFIG_KEYS` (config.cjs) + defaults (config.cjs + core.cjs)
- New init command → add function (init.cjs), export, register in dispatch (gsd-tools.cjs)

---

## 2.5 Team Notification

After pushing changes:

```
GSD updated — [what changed]

Update (Mac/Linux):
  GSD_CLONE_DIR="${HOME}/tmp/gsd-install" && rm -rf "$GSD_CLONE_DIR" && git clone https://github.com/slainai/get-shit-done.git "$GSD_CLONE_DIR" && cd "$GSD_CLONE_DIR" && npm install && npm run build:hooks && node bin/install.js --claude --global && cd ~ && rm -rf "$GSD_CLONE_DIR"

Update (Windows PowerShell):
  $d="$env:USERPROFILE\tmp\gsd-install"; if(Test-Path $d){Remove-Item -Recurse -Force $d}; git clone https://github.com/slainai/get-shit-done.git $d; Push-Location $d; npm install; npm run build:hooks; node bin/install.js --claude --global; Pop-Location; Remove-Item -Recurse -Force $d

What's new:
- [bullet points]

Restart Claude Code to pick up changes.
```
