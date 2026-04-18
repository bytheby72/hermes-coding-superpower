---
name: hermes-coding-superpower
description: Karpathy coding doctrine + Claude Code architecture + self-evolution toolkit for Hermes Agent. Produces simple, verifiable, surgical code changes.
version: 1.0.0
author: NousResearch + community
metadata:
  hermes:
    tags: [coding, architecture, karpathy, claude-code, self-evolution, best-practices]
---

# 🚀 Hermes Coding Superpower

> **Karpathy's Coding Philosophy × Claude Code Architecture = The Ultimate AI Agent Coder**

This skill combines three powerful sources:
1. **Karpathy Doctrine** — simplicity first, surgical changes, no bullshit
2. **Claude Code Architecture** — battle-tested patterns from 4481 files of production code
3. **Self-Evolution Toolkit** — DSPy + GEPA for automatic skill optimization

---

## ⚡ THE KARPATHY DOCTRINE

### 1. THINK BEFORE CODING

```
Don't assume. Don't hide confusion. Surface tradeoffs.
```

- State assumptions explicitly
- Present multiple interpretations
- Push back when a simpler approach exists
- **STOP AND ASK** when something is unclear

### 2. SIMPLICITY FIRST

```
Minimum code that solves the problem. Nothing speculative.
```

- ❌ No features beyond what was asked
- ❌ No abstractions for single-use code
- ❌ No "flexibility" that wasn't requested
- ✅ If it could be 50 lines, it won't be 200

### 3. SURGICAL CHANGES

```
Touch only what you must. Clean up only your own mess.
```

- ❌ No "improving" adjacent code
- ❌ No refactoring things that aren't broken
- ✅ Match existing style, even if you'd do it differently
- ✅ Every changed line traces directly to the request

### 4. GOAL-DRIVEN EXECUTION

```
Define success criteria. Loop until verified.
```

| Vague Request | Hermes Response |
|--------------|-----------------|
| "Add validation" | "Write tests for invalid inputs → make them pass" |
| "Fix the bug" | "Write test that reproduces it → make it pass" |
| "Refactor X" | "Ensure tests pass before and after" |

---

## 🏗️ CLAUDE CODE ARCHITECTURE PATTERNS

### Agent Loop

```
User Input → Context Assembly → LLM Call → Tool Parse → Execute → Loop
```

With task dependency tracking:
- `blockedBy` — What must complete first
- `blocks` — What this unlocks downstream
- `owner` — Which sub-agent is responsible

### Three-Tier Permission Gating

```
Tier 1: Auto-approve (Read, Search)
Tier 2: Auto-approve with logging (Write in project)
Tier 3: Require confirmation (Bash, Network, Write outside)
```

### Token Budget Management
- **80% threshold** triggers automatic compaction
- Preserves system prompt + recent turns
- Summarizes oldest messages with auxiliary LLM

---

## 🛠️ ARSENAL

| Category | Tools |
|----------|-------|
| **File Operations** | `read_file`, `write_file`, `patch`, `search_files` |
| **Search & Navigation** | ripgrep-backed content search, file glob |
| **Shell Execution** | `terminal` with approval gating |
| **Task Management** | `todo` with priority tracking |
| **Web & Browser** | `web_search`, `browser_navigate`, `browser_click`, `browser_vision` |
| **Code Execution** | `execute_code` — Python sandbox |
| **Delegation** | `delegate_task` — Spawn sub-agents |
| **Memory** | `memory` — Persistent knowledge across sessions |
| **Skills** | 50+ pre-built skills |
| **MCP Protocol** | External tool server integration |

---

## 🧬 SELF-EVOLUTION TOOLKIT

Automatically evolve and optimize skills, prompts, and code.

### Quick Start

```bash
# Point at your hermes-agent repo
export HERMES_AGENT_REPO=~/.hermes/hermes-agent

# Evolve a skill with synthetic eval data
cd ~/.hermes/skills/devops/hermes-coding-superpower
python scripts/evolve_skill.py \
    --skill github-code-review \
    --iterations 10 \
    --eval-source synthetic
```

### What It Optimizes

| Phase | Target | Engine | Status |
|-------|--------|--------|--------|
| **Phase 1** | Skill files (SKILL.md) | DSPy + GEPA | ✅ Implemented |
| **Phase 2** | Tool descriptions | DSPy + GEPA | 🔲 Planned |
| **Phase 3** | System prompt sections | DSPy + GEPA | 🔲 Planned |
| **Phase 4** | Tool implementation code | Darwinian Evolver | 🔲 Planned |

### Guardrails

Every evolved variant must pass:
1. **Full test suite** — `pytest tests/ -q` must pass 100%
2. **Size limits** — Skills ≤15KB, tool descriptions ≤500 chars
3. **Caching compatibility** — No mid-conversation changes
4. **Semantic preservation** — Must not drift from original purpose
5. **PR review** — All changes go through human review

---

## 🚨 ANTI-PATTERNS TO ELIMINATE

| Anti-Pattern | Hermes Response |
|--------------|-----------------|
| **Over-abstraction** | Strategy pattern for single use? → One function. |
| **Silent assumptions** | Picking defaults without asking? → Surface and confirm. |
| **Drive-by refactoring** | "Improving" unrelated code? → Surgical changes only. |
| **Vague success criteria** | "Make it work"? → Test-first, verifiable goals. |

---

## 📈 METRICS THAT MATTER

Hermes is working when:
- ✅ **Fewer unnecessary changes** in diffs
- ✅ **Fewer rewrites** due to overcomplication
- ✅ **Clarifying questions** come BEFORE implementation
- ✅ **Tests pass** before and after every change

---

## 📚 References

- `references/KARPATHY_GUIDELINES.md` — Full Karpathy coding guidelines
- `references/ARCHITECTURE.md` — Claude Code architecture deep dive
- `references/PLAN.md` — Self-evolution complete architecture plan

---

*Built with 🔥 and zero patience for broken abstractions.*
