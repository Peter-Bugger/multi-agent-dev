# Multi-Agent Dev / 多Agent开发工作流

> **Hermes Skill** — 自进化的5阶段多Agent开发流水线。全局安装一次，所有项目自动继承。
> A self-evolving 5-stage multi-agent development pipeline as a Hermes skill. Install once globally, auto-inherit across all projects.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Hermes](https://img.shields.io/badge/Hermes-Skill-8A2BE2)](https://github.com/NousResearch/hermes-agent)

---

English | [中文](#中文)

## Architecture / 架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MULTI-AGENT-DEV SKILL                             │
│                  (Global Application Layer)                          │
│                                                                     │
│  Installed once at ~/.claude/skills/multi-agent-dev/                │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  SKILL.md                    The Workflow Engine               │  │
│  │  ├─ /multi-agent-dev         5-stage full pipeline             │  │
│  │  ├─ /multi-agent-dev [quick] Lightweight 2-stage mode          │  │
│  │  ├─ /multi-agent-dev stage N Single stage execution            │  │
│  │  ├─ /multi-agent-dev resume  Resume interrupted workflow       │  │
│  │  └─ Project Init             Auto-detect + create project files│  │
│  ├───────────────────────────────────────────────────────────────┤  │
│  │  pattern-registry.md         Cross-Project Knowledge Base     │  │
│  │  ├─ [global] patterns        Language-agnostic principles     │  │
│  │  ├─ [stack: xxx] patterns    Tech-specific patterns            │  │
│  │  ├─ Conflict detection       Multi-project consistency        │  │
│  │  └─ Promotion log            project → stack → global          │  │
│  ├───────────────────────────────────────────────────────────────┤  │
│  │  agents.json                 Global Agent Definitions          │  │
│  │  project-init-template/      Templates for new projects        │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ @mention triggers
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    AGENT LAYER (Global)                              │
│              ~/.claude/agents/                                       │
│                                                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│  │tech-lead │→│senior-dev│→│developer │→│reviewer  │→│integrator│ │
│  │ .md      │ │ .md      │ │ .md      │ │ .md      │ │ .md      │ │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └─────┬────┘ │
│                                                            │       │
│  Each agent has: model, tools, responsibilities, rules     │       │
└────────────────────────────────────────────────────────────┘       │
                                                                     │
                                    │ Auto-init on first use           │
                                    ▼                                  │
┌─────────────────────────────────────────────────────────────────────┐
│                    PROJECT LAYER (Per-Project)                       │
│              ./.claude/workflows/                                    │
│                                                                     │
│  ┌──────────────────────────────────────────┐                       │
│  │  multi-agent-flow.md                     │                       │
│  │  ├─ Inherited global patterns            │ ← auto-populated      │
│  │  ├─ Project-specific overrides           │                       │
│  │  └─ Project-local patterns (max 15)      │                       │
│  ├──────────────────────────────────────────┤                       │
│  │  lessons-log.md                          │                       │
│  │  ├─ Quality-gated experience entries     │                       │
│  │  ├─ Auto-archive at 30+ entries          │                       │
│  │  └─ Pattern extraction at ≥3 occurrences │                       │
│  └──────────────────────────────────────────┘                       │
│                                                                     │
│  Each project self-evolves independently                            │
└─────────────────────────────────────────────────────────────────────┘
```

### Data Flow / 数据流

```
User: /multi-agent-dev Add avatar upload

    ┌──────────────────────────────────────────────────────────────┐
    │                    SKILL.md (Engine)                          │
    │  1. Check if project initialized → No → Auto-init             │
    │  2. Load agents.json → resolve agents                         │
    │  3. Execute 5-stage pipeline                                  │
    └──────────┬───────────────────────────────────────────────────┘
               │
    ┌──────────▼──────────┐
    │ Stage 1: Tech Lead  │──→  PLAN.md
    │  Confirm requirements│    WORKFLOW_STATUS.md (init)
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Stage 2: Senior Dev │──→  DESIGN.md + task breakdown
    │  Review + amend PLAN │    (may amend PLAN.md)
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Stage 3: Developer  │──→  Code changes
    │  Implement tasks     │    WORKFLOW_STATUS.md (task progress)
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐     ┌─────────────────────────┐
    │ Stage 4: Reviewer   │────→│ Auto-loop (max 3 cycles) │
    │  git diff audit      │←────│ Dev fixes → Re-review    │
    └──────────┬──────────┘     └─────────────────────────┘
               │ APPROVED
    ┌──────────▼──────────┐
    │ Stage 5: Integrator │──→  Build + Test + Report
    │  Full build & verify │    Experience capture
    │  📝 Lessons learned  │    Pattern promotion check
    └──────────────────────┘
               │
    ┌──────────▼──────────────────────────────────────────────────┐
    │  Experience Lifecycle:                                       │
    │                                                              │
    │  lessons-log.md ──≥3 same──→ project pattern (max 15)        │
    │       │                                                      │
    │       └──≥5 stable triggers──→ stack pattern (if tech-match) │
    │                                      │                       │
    │                                      └──→ global pattern     │
    │                                           (pattern-registry) │
    └──────────────────────────────────────────────────────────────┘
```

---

## File Structure / 文件结构

```
multi-agent-dev/
├── README.md                           ← This file
├── LICENSE
├── .gitignore
└── .claude/
    │
    │  ╔══════════════════════════════════════════════════════╗
    │  ║  SKILL LAYER  →  ~/.claude/skills/multi-agent-dev/   ║
    │  ╚══════════════════════════════════════════════════════╝
    │
    ├── skills/multi-agent-dev/
    │   ├── SKILL.md                     # Workflow engine (slash command)
    │   ├── agents.json                  # Global agent definitions
    │   ├── pattern-registry.md          # Cross-project knowledge base
    │   └── project-init-template/       # Auto-generated per project
    │       ├── multi-agent-flow.md      # Project workflow template
    │       └── lessons-log.md           # Experience log template
    │
    │  ╔══════════════════════════════════════════════════════╗
    │  ║  AGENT LAYER  →  ~/.claude/agents/                    ║
    │  ╚══════════════════════════════════════════════════════╝
    │
    └── agents/
        ├── tech-lead.md                 # Root cause, architecture
        ├── senior-dev.md                # Solution design, task breakdown
        ├── developer.md                 # Code implementation
        ├── reviewer.md                  # Diff review, security audit
        └── integrator.md                # Regression + experience capture
```

---

## Quick Start / 快速开始

```bash
# 1. Clone
git clone https://github.com/Peter-Bugger/multi-agent-dev.git
cd multi-agent-dev

# 2. Install — one time, all projects
mkdir -p ~/.claude/skills ~/.claude/agents
cp -r .claude/skills/multi-agent-dev ~/.claude/skills/
cp -r .claude/agents/* ~/.claude/agents/

# 3. Go to any project and start using
cd /path/to/your-project
# First use auto-initializes project-level files
# 首次使用自动初始化项目级文件

# 4. Slash commands
/multi-agent-dev Add user avatar upload feature
/multi-agent-dev [quick] Fix login button alignment
/multi-agent-dev stage 4         # Review only
/multi-agent-dev resume          # Resume interrupted workflow
```

### Three Usage Modes / 三种使用模式

| Mode | Command | When to Use |
|------|---------|-------------|
| **Full** | `/multi-agent-dev <task>` | New features, cross-module changes |
| **Quick** | `/multi-agent-dev [quick] <task>` | Small fixes, single-file changes |
| **Stage** | `/multi-agent-dev stage <N>` | Run specific stage independently |

---

## The 5 Roles / 5个角色

| Role | Agent File | Model | Core Responsibility |
|------|-----------|-------|---------------------|
| **Tech Lead** | `agents/tech-lead.md` | opus | Requirements analysis, architecture, PLAN.md |
| **Senior Dev** | `agents/senior-dev.md` | opus | Plan review, detailed design, task breakdown |
| **Developer** | `agents/developer.md` | sonnet | Code implementation, task-by-task verification |
| **Reviewer** | `agents/reviewer.md` | sonnet | Diff audit, BLOCKER/INFO classification, auto-loop |
| **Integrator** | `agents/integrator.md` | sonnet | Full build/test, experience capture, pattern promotion |

---

## Key Features / 核心特性

### 1. Self-Evolving / 自进化
- Experience logged after every run
- Patterns extracted at ≥3 same-type occurrences
- Auto-archive old entries at 30+
- Quality gate: actionable + generalizable + scope-determined

### 2. Cross-Project Knowledge / 跨项目知识
- **Pattern Registry**: global patterns shared across all projects
- **Stack Scoping**: `[global]` → `[stack: SpringBoot3]` → `[stack: Vue3+ElementPlus]`
- **Auto-Inheritance**: new projects inherit matching patterns
- **Conflict Detection**: prevents contradictory patterns

### 3. Pattern Lifecycle / 模式生命周期
```
Project experience → ≥3 same → Project pattern (max 15)
    → ≥5 stable triggers → Stack pattern (if tech matches)
    → Global pattern (pattern-registry, max 20)
```

### 4. Project Initialization / 项目初始化
First use in any project auto-detects:
- Project name and tech stack
- Creates `.claude/workflows/`
- Copies templates from `project-init-template/`
- Inherits matching patterns from `pattern-registry.md`
- Outputs: "Initialized, inherited N patterns from global registry"

### 5. Auto-Loop Review / 自动循环审查
- Reviewer → Developer fix → Re-review, max 3 cycles
- Integrator failure → Auto-fix → Re-test, max 3 cycles
- No manual intervention needed

---

## Pattern Registry / 模式注册表

The `pattern-registry.md` is the cross-project knowledge backbone:

```
[global]                    ← Language-agnostic principles
  #1: Controller exceptions must be fully covered

[stack: Vue3+ElementPlus]   ← Vue3 + Element Plus specific
  (patterns from Vue projects)

[stack: SpringBoot3]         ← Spring Boot 3 specific
  (patterns from Java projects)
```

**New project initialization**:
- All `[global]` patterns are inherited
- `[stack: xxx]` patterns that match the project's tech stack are inherited
- Projects can override but not delete inherited patterns

---

## 中文

### 这是什么？

一套完整的5阶段多Agent开发流水线，作为 Hermes Skill 安装。全局安装一次，所有项目自动继承。核心特点：

| 特性 | 说明 |
|------|------|
| **自进化** | 每次执行积累经验，≥3次提炼为模式 |
| **跨项目知识** | 全局模式注册表，新项目自动继承 |
| **分层架构** | Skill层(引擎) → Agent层(角色) → Project层(项目) |
| **自动初始化** | 首次使用自动检测技术栈并创建项目文件 |
| **自动循环** | Reviewer/Integrator 失败自动修复重试(最多3次) |
| **轻量模式** | `[quick]` 模式跳过架构阶段，适用小改动 |
| **双语文档** | 中英文 + 含完整架构图 |

### 三层架构

```
Skill 层 (引擎)    →  ~/.claude/skills/multi-agent-dev/
  ├─ SKILL.md          命令定义 + 工作流引擎
  ├─ pattern-registry.md   跨项目知识库
  ├─ agents.json            全局角色定义
  └─ project-init-template/ 项目初始化模板

Agent 层 (角色)    →  ~/.claude/agents/
  └─ 5 个 .md 文件 (tech-lead → integrator)

Project 层 (项目)   →  ./.claude/workflows/ (自动创建)
  ├─ multi-agent-flow.md    项目工作流(继承全局模式)
  └─ lessons-log.md         经验日志(自动累积)
```

---

## Contributing / 贡献

This is the canonical source of truth for the multi-agent-dev skill. All pattern contributions and workflow improvements should be PR'd here.

本仓库是 multi-agent-dev skill 的唯一权威来源。所有模式贡献和工作流改进请提 PR 到这里。

## License / 许可证

MIT — see [LICENSE](LICENSE) for details.
