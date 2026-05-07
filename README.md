# Multi-Agent Dev / 多Agent开发工作流

> Two commands to install, one slash command to use. Complex tasks use full 5-stage pipeline, simple fixes use quick 2-stage mode.
> 两行命令安装，一个斜杠命令使用。复杂功能走完整5阶段，简单改动走快速2阶段。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

English | [中文](#中文)

---

## Quick Start

### Install (once, globally)

```bash
git clone https://github.com/Peter-Bugger/multi-agent-dev.git
cd multi-agent-dev
cp -r .claude/agents ~/.claude/
cp -r .claude/skills ~/.claude/
```

Done. All projects can use it.

### Use in Claude Code

```
/multi-agent-dev Add avatar upload to user profile page
```

First use auto-detects project tech stack and creates project files.

---

## Two Modes

### Full Mode — features, cross-module changes, refactoring

```
/multi-agent-dev <task description>
```

5 stages, each a dedicated agent:

```
Tech Lead → Senior Dev → Developer → Reviewer → Integrator
    Plan       Design        Code        Review      Verify
```

Reviewer finds bugs → auto back to Developer → re-review, max 3 cycles.

### Quick Mode — small fixes, CSS tweaks, config changes

```
/multi-agent-dev [quick] Fix login button alignment
```

Skips planning stages, goes straight to:

```
Developer → Integrator
   Code       Verify + brief review
```

---

## Other Commands

| Command | Purpose |
|---------|---------|
| `/multi-agent-dev stage 4` | Review current code only |
| `/multi-agent-dev stage 5` | Full build & test only |
| `/multi-agent-dev resume` | Resume interrupted workflow |

---

## Architecture: 3 Layers + Self-Evolution

```
                         ┌──────────────────────┐
                         │   One slash command    │
                         │  /multi-agent-dev ...  │
                         └──────────┬───────────┘
                                    │
┌───────────────────────────────────▼───────────────────────────────────┐
│                     Skill Layer (engine, install once)                 │
│                     ~/.claude/skills/multi-agent-dev/                 │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ SKILL.md          agents.json        pattern-registry.md        │ │
│  │ Command routing +   Global agent       Cross-project knowledge   │ │
│  │ workflow engine     definitions        base [global|stack:xxx]   │ │
│  └─────────────────────────────────────────────────────────────────┘ │
└───────────────────────────────────┬───────────────────────────────────┘
                                    │ dispatches 5 agents
┌───────────────────────────────────▼───────────────────────────────────┐
│                     Agent Layer (roles, install once)                  │
│                     ~/.claude/agents/                                  │
│                                                                       │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  │Tech Lead │──→│Senior Dev│──→│Developer │──→│Reviewer  │──→│Integrator│
│  │   Plan    │   │  Design  │   │   Code   │   │  Review  │   │  Verify  │
│  └──────────┘   └──────────┘   └──────────┘   └────┬─────┘   └────┬─────┘
│                                                    │ Bugs found  │
│                                          ┌─────────┘             │
│                                          │ Auto back to Developer │
│                                          │ Max 3 cycles          │
│                                          └───────────────────────→ Log experience
└───────────────────────────────────┬───────────────────────────────────┘
                                    │ Auto-created on first use
┌───────────────────────────────────▼───────────────────────────────────┐
│                    Project Layer (per project, self-evolving)          │
│                    ./.claude/workflows/                                │
│                                                                       │
│  ┌─────────────────────────┐        ┌─────────────────────────────┐  │
│  │  multi-agent-flow.md    │        │  lessons-log.md             │  │
│  │  ├─ Inherited patterns   │←──────│  ├─ Auto-log each run        │  │
│  │  ├─ Project overrides    │  ≥3x   │  ├─ ≥3x → project pattern   │  │
│  │  └─ Local patterns (≤15) │        │  └─ ≥5x stable → promote    │  │
│  └─────────────────────────┘        └─────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────┘
                                    │ ≥5 stable triggers
┌───────────────────────────────────▼───────────────────────────────────┐
│                    Cross-Project Promotion                              │
│                                                                       │
│  Project pattern → [stack: SpringBoot3] → [global]                    │
│                     Same-stack inherits       All projects inherit    │
└───────────────────────────────────────────────────────────────────────┘
```

---

## Gets Smarter With Use

Integrator auto-captures experience after every run:

```
Experience ≥3x same → project pattern → ≥5x stable → promote to stack → promote to global
```

New projects auto-inherit matching patterns from the global registry.

---

## 5 Roles

| Role | When | Output |
|------|------|--------|
| Tech Lead | Full mode first | Plan |
| Senior Dev | After Tech Lead | Detailed design + tasks |
| Developer | After design confirmed | Code |
| Reviewer | After code (auto) | Review report |
| Integrator | After review passed (auto) | Build test + experience log |

---

## File Structure

```
multi-agent-dev/                     ← This repo (install once)
├── README.md
├── LICENSE
└── .claude/
    ├── agents/                      → ~/.claude/agents/
    │   ├── tech-lead.md
    │   ├── senior-dev.md
    │   ├── developer.md
    │   ├── reviewer.md
    │   └── integrator.md
    └── skills/multi-agent-dev/      → ~/.claude/skills/
        ├── SKILL.md                 # Engine
        ├── agents.json
        ├── pattern-registry.md      # Cross-project knowledge
        └── project-init-template/   # Templates
```

---

## Quick Reference

| I want to... | Command |
|--------------|---------|
| New feature | `/multi-agent-dev Add export` |
| Refactor module | `/multi-agent-dev Split OrderService` |
| Fix style | `/multi-agent-dev [quick] Fix button` |
| Review only | `/multi-agent-dev stage 4` |
| Resume interrupted | `/multi-agent-dev resume` |

---

## 中文

---

## 快速开始

### 安装（一次，全局）

```bash
git clone https://github.com/Peter-Bugger/multi-agent-dev.git
cd multi-agent-dev
cp -r .claude/agents ~/.claude/
cp -r .claude/skills ~/.claude/
```

搞定。以后所有项目都能用。

### 在 Claude Code 里使用

```
/multi-agent-dev 在用户设置页添加头像上传功能
```

首次使用自动检测项目技术栈，创建项目级文件。

---

## 两种模式

### 完整模式 — 新功能、跨模块改动、重构

```
/multi-agent-dev <任务描述>
```

5 个阶段，每个阶段一个独立 Agent：

```
Tech Lead → Senior Dev → Developer → Reviewer → Integrator
  方案设计    详细设计        写代码       审查       集成验证
```

审查发现 BUG → 自动回到 Developer 修复 → 重新审查，最多 3 轮。

### 快速模式 — 小改、修 Bug、调样式

```
/multi-agent-dev [quick] 修复登录按钮对齐问题
```

跳过大方案设计，直接：

```
Developer → Integrator
  写代码       验证 + 简要审查
```

---

## 其他命令

| 命令 | 干什么 |
|------|--------|
| `/multi-agent-dev stage 4` | 只审查当前代码 |
| `/multi-agent-dev stage 5` | 只跑构建和测试 |
| `/multi-agent-dev resume` | 恢复中断的工作流 |

---

## 架构：三层 + 自进化

```
                     ┌──────────────────────┐
                     │   一个斜杠命令搞定      │
                     │  /multi-agent-dev ...  │
                     └──────────┬───────────┘
                                │
┌───────────────────────────────▼───────────────────────────────────────┐
│                     Skill 层（引擎，全局安装一次）                       │
│                     ~/.claude/skills/multi-agent-dev/                 │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ SKILL.md          agents.json        pattern-registry.md        │ │
│  │ 命令解析+流程编排    全局角色定义         跨项目经验库               │ │
│  │ [完整5阶段/快速2阶段/ 单阶段/恢复]          [global | stack:xxx]   │ │
│  └─────────────────────────────────────────────────────────────────┘ │
└───────────────────────────────┬───────────────────────────────────────┘
                                │ 调度 5 个 Agent
┌───────────────────────────────▼───────────────────────────────────────┐
│                     Agent 层（角色，全局安装一次）                       │
│                     ~/.claude/agents/                                  │
│                                                                       │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  │Tech Lead │──→│Senior Dev│──→│Developer │──→│Reviewer  │──→│Integrator│
│  │ 方案设计  │   │ 详细设计  │   │ 写代码    │   │ 审查     │   │ 集成验证  │
│  └──────────┘   └──────────┘   └──────────┘   └────┬─────┘   └────┬─────┘
│                                                    │ 发现BUG     │
│                                          ┌─────────┘             │
│                                          │ 自动回到Developer修复   │
│                                          │ 最多3轮                │
│                                          └───────────────────────→ 经验沉淀
└───────────────────────────────┬───────────────────────────────────────┘
                                │ 首次使用自动创建
┌───────────────────────────────▼───────────────────────────────────────┐
│                     Project 层（每个项目独立，自进化）                   │
│                     ./.claude/workflows/                               │
│                                                                       │
│  ┌─────────────────────────┐       ┌─────────────────────────────┐   │
│  │  multi-agent-flow.md    │       │  lessons-log.md             │   │
│  │  ├─ 继承的全局模式        │←─────│  ├─ 自动记录每轮经验           │   │
│  │  ├─ 项目覆盖              │  ≥3次 │  ├─ ≥3次 → 项目模式          │   │
│  │  └─ 本项目特有模式(≤15)   │       │  └─ ≥5次稳定 → 推广         │   │
│  └─────────────────────────┘       └─────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────────┘
                                │ ≥5次稳定触发
┌───────────────────────────────▼───────────────────────────────────────┐
│                     跨项目知识推广                                      │
│                                                                       │
│  项目模式 ──→ [stack: SpringBoot3] ──→ [global]                       │
│              同技术栈项目自动继承          所有项目自动继承             │
└───────────────────────────────────────────────────────────────────────┘
```

---

## 越用越聪明

Integrator 每次收尾自动记录经验：

```
经验 ≥3次同类 → 提炼为项目模式 → ≥5次稳定 → 推广到同技术栈 → 推广到全局
```

新项目首次使用自动继承全局匹配模式。

---

## 5 个角色

| 角色 | 出场 | 产出 |
|------|------|------|
| Tech Lead | 完整模式第一步 | 技术方案 |
| Senior Dev | Tech Lead 之后 | 详细设计 + 任务拆解 |
| Developer | 设计方案确定后 | 代码 |
| Reviewer | 代码写完自动触发 | 审查报告 |
| Integrator | 审查通过后自动触发 | 构建测试 + 经验沉淀 |

---

## 文件结构

```
multi-agent-dev/                     ← 这个仓库（全局安装一次）
├── README.md
├── LICENSE
└── .claude/
    ├── agents/                      → ~/.claude/agents/
    │   ├── tech-lead.md
    │   ├── senior-dev.md
    │   ├── developer.md
    │   ├── reviewer.md
    │   └── integrator.md
    └── skills/multi-agent-dev/      → ~/.claude/skills/
        ├── SKILL.md                 # 引擎
        ├── agents.json
        ├── pattern-registry.md      # 跨项目知识库
        └── project-init-template/   # 模板
```

---

## 速查

| 我想... | 命令 |
|---------|------|
| 开发新功能 | `/multi-agent-dev 添加导出功能` |
| 重构模块 | `/multi-agent-dev 拆分OrderService` |
| 修样式 | `/multi-agent-dev [quick] 修复按钮` |
| 只审查 | `/multi-agent-dev stage 4` |
| 断点续传 | `/multi-agent-dev resume` |

---

## License

MIT
