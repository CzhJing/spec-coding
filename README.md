# SpecCoding

> AI-assisted development workflow skill. Lock boundaries with OpenSpec first, then execute within boundaries with Superpowers.

[中文文档](README.zh.md)

---

## Overview

SpecCoding is a structured AI-assisted development workflow for any new feature development, requirement changes, or bug fixes.

**Core philosophy:**

- **Design by human decision** — requirements and boundaries are confirmed by humans
- **Execute autonomously** — AI executes within locked boundaries with minimal interruptions

**Workflow:**

```
Codebase Exploration → /opsx:explore → /opsx:propose → Branch → Execute → PR → /opsx:archive
```

---

## Prerequisites

### 1. OpenSpec

> Requirement exploration + specification solidification + change memory.

```bash
# Install
pip install openspec-cli

# Or clone and install manually
git clone https://github.com/Fission-AI/OpenSpec.git
cd OpenSpec
pip install -e .

# Verify
openspec --version
```

### 2. Superpowers

> Execution framework: plan splitting + code implementation + TDD + three-level review.

```bash
# Install
pip install superpowers

# Or clone and install manually
git clone https://github.com/obra/superpowers.git
cd superpowers
pip install -e .
```

### 3. Ruflo (Optional — for parallel swarm execution)

> Multi-agent swarm orchestration based on task dependencies.

```bash
# Install
pip install ruflo

# Or clone and install manually
git clone https://github.com/ruvnet/ruflo.git
cd ruflo
pip install -e .

# Verify
ruflo --version
```

### 4. Claude Code Agent Teams Mode (Optional — for subagent dispatch)

> Claude Code built-in multi-agent team collaboration.

**Enable in Claude Code:**

Add the environment variable to your Claude Code configuration file (e.g. `~/.claude/settings.json`):

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**Verify available subagents:**

```bash
ls ~/.claude/agents/
```

---

## Installation

### Method 1: Direct Copy (Recommended)

```bash
# Clone this skill repository
git clone https://github.com/CzhJing/spec-coding.git ~/.claude/skills/speccoding

# Or copy to your project
cp -r speccoding/ <your-project>/.claude/skills/
```

### Method 2: Claude Code Skill Registration

```bash
# Register with Claude Code
claude skill add speccoding /path/to/speccoding/SKILL.md
```

---

## Quick Start

### Trigger Keywords

Mention any of the following to activate SpecCoding:

- "develop new feature"
- "modify requirement"
- "implement XXX"
- "add a XXX interface"
- "refactor XXX"
- "fix bug"

### 8-Step Workflow

```
① Codebase Exploration   → Read code, output summary, wait for confirmation
② /opsx:explore         → Requirement exploration (one question at a time, 2-3 options)
③ /opsx:propose         → Generate proposal, design, specs, tasks
    ⚑ Human Review      → Review proposal.md, confirm direction and boundaries
④ Create Branch         → git checkout -b feature/<name>
    ⚑ Choose Mode       → Select execution mode:

    Parallel (many tasks, clear dependencies):
    ⑤-A Ruflo Swarm    → Multi-agent swarm based on depends_on
    ⑤-B Agent Team     → Claude Code dispatches subagents from ~/.claude/agents/

    Sequential (few tasks, complex dependencies):
    ⑤-C Superpowers    → writing-plans → TDD → three-level review

⑥ Branch Cleanup        → git push + create PR
⑦ /opsx:archive        → Merge delta spec, cleanup branch
```

---

## Execution Modes

| Mode | When to Use | Prerequisites |
|------|-------------|---------------|
| **Ruflo Swarm** | Many tasks, clear dependencies | Ruflo installed |
| **Agent Team** | Claude Code environment, subagents available | Claude Code + ~/.claude/agents/ |
| **Superpowers** | Few tasks, complex dependencies | Superpowers installed |

### Mode Selection Guide

```
Many tasks + clear deps  →  Ruflo Swarm
Claude Code + agents     →  Agent Team
Few tasks + complex deps →  Superpowers
```

---

## File Structure

```
speccoding/
├── SKILL.md                          # Skill definition
├── README.md                         # This file
├── README.zh.md                      # Chinese version
├── references/
│   ├── EXPLORE.md                    # Codebase exploration rules
│   ├── OPENSPEC.md                   # OpenSpec commands (/opsx:explore, /opsx:propose, /opsx:archive)
│   ├── BRANCH.md                     # Branch creation, PR, merge
│   ├── CODING-PRINCIPLES.md          # Four coding principles
│   ├── STANDARD.md                   # Superpowers standard mode
│   ├── SWARM.md                      # Ruflo swarm mode
│   ├── AGENT-TEAM.md                 # Claude Code agent team mode
│   ├── CHECKLIST.md                  # Stage checklists
│   └── ANTI-PATTERNS.md              # Forbidden behaviors + correct practices
└── templates/
    ├── plan.md                       # plan.md template (with depends_on)
    ├── proposal.md                   # proposal.md template
    └── pr-body.md                    # PR body template
```

---

## Key Behaviors

### Coding Principles

1. **Think before coding** — state assumptions, present trade-offs, stop when confused
2. **Simplicity first** — minimal code, no over-engineering
3. **Precise edits** — touch only what must be touched, clean up your own mess
4. **Goal-driven execution** — define success criteria, loop until achieved

### TDD Cycle (Mandatory)

```
RED     → Write failing test, run to confirm failure
GREEN   → Write minimal implementation, run to confirm pass
REFACTOR→ Clean up, tests still pass
COMMIT  → feat/fix/refactor: <description>
```

### Three-Level Review

```
Specification Review → Against openspec/changes/<name>/specs/
Code Quality Review  → Style, naming, redundancy, test quality
All Pass → Mark complete ✅
```

---

## Change Classification

| Type | Examples | Full Workflow? |
|------|----------|----------------|
| Small change | Copy, logs, minor refactor | Simplified, delta spec if behavior changes |
| Regular feature | New interface, flow adjustment | Full 8-step workflow |
| Complex refactor | Module split, auth change, library swap | Full workflow + rollback check |

---

## When to Stop and Ask

AI must pause and ask for human input only in these scenarios:

1. Reality conflicts with confirmed plan (unexpected fork)
2. Irreversible / destructive operations (`git push --force`, `git reset --hard`, delete shared branch, modify `main`)
3. Repeated failures on same approach, need direction change
4. Specification explicitly requires human confirmation

---

## References

| File | Content | When to Read |
|------|---------|--------------|
| `references/EXPLORE.md` | Codebase exploration 6-step rules and summary format | Step ① |
| `references/OPENSPEC.md` | explore / propose / archive full specification | Steps ②③⑦ |
| `references/BRANCH.md` | Branch creation, parent tracking, PR, merge | Steps ④⑥ |
| `references/CODING-PRINCIPLES.md` | Four coding principles (think / simple / precise / goal-driven) | Before any coding task |
| `references/SWARM.md` | Ruflo Swarm mode full specification | Step ⑤-A |
| `references/AGENT-TEAM.md` | Claude Code Agent Team mode (auto subagent dispatch) | Step ⑤-B |
| `references/STANDARD.md` | Superpowers standard mode (writing-plans + TDD + review) | Step ⑤-C |
| `references/CHECKLIST.md` | Pre-execution checklists for each stage | Before each step |
| `references/ANTI-PATTERNS.md` | Forbidden behaviors + correct practices | Any decision fork |

---

## License

MIT
