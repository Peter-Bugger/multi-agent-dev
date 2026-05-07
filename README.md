# Multi-Agent Dev / 多Agent开发工作流

> 两行命令安装，一个斜杠命令使用 / Two commands to install, one slash command to use.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

English | 中文

---

## 怎么用 / How to Use

### 安装（一次，全局） / Install (once, globally)

```bash
git clone https://github.com/Peter-Bugger/multi-agent-dev.git
cd multi-agent-dev
cp -r .claude/agents ~/.claude/       # 5 个角色定义 / 5 agent definitions
cp -r .claude/skills ~/.claude/       # 工作流引擎 / workflow engine
```

Done. All projects can use it now. / 搞定，所有项目都能用了。

### 使用 / Usage

```
/multi-agent-dev 在用户设置页添加头像上传功能
/multi-agent-dev Add avatar upload to user profile page
```

第一次用会自动检测技术栈、创建项目级文件。
First use auto-detects tech stack and creates project files.

---

## 两种模式 / Two Modes

### 完整模式 / Full mode

新功能、跨模块改动、重构 / For features, cross-module changes, refactoring:

```
/multi-agent-dev <task description / 任务描述>
```

5 阶段 / 5 stages:

```
Tech Lead ──→ Senior Dev ──→ Developer ──→ Reviewer ──→ Integrator
 方案设计       详细设计         写代码         审查          集成验证
 Plan          Design           Code          Review        Verify
```

每个阶段一个独立 Agent。审查发现 BUG 自动回到 Developer 修复，最多 3 轮。
Each stage is an independent agent. Reviewer auto-loops back to Developer on bugs (max 3 cycles).

### 快速模式 / Quick mode

小改、修 Bug、调样式 / For small fixes, CSS tweaks, config changes:

```
/multi-agent-dev [quick] 修复登录按钮对齐问题
/multi-agent-dev [quick] Fix login button alignment
```

直接跳到写代码 + 验证，跳过方案设计阶段。
Goes straight to coding + verification, skips planning stages.

---

## 其他命令 / Other Commands

| 命令 / Command | 干什么 / What it does |
|------|--------|
| `/multi-agent-dev stage 4` | 只审查代码 / Review only |
| `/multi-agent-dev stage 5` | 只跑构建测试 / Build & test only |
| `/multi-agent-dev resume` | 恢复中断 / Resume interrupted |

---

## 架构：三层 + 自进化 / Architecture: 3 Layers + Self-Evolution

```
                         ┌──────────────────────────┐
                         │   一个命令 / one command    │
                         │  /multi-agent-dev ...      │
                         └──────────┬───────────────┘
                                    │
┌───────────────────────────────────▼───────────────────────────────────┐
│                  Skill 层 / Layer（引擎，全局一次 / engine, once）       │
│                  ~/.claude/skills/multi-agent-dev/                    │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ SKILL.md           agents.json         pattern-registry.md      │ │
│  │ 命令解析+流程编排   全局角色定义          跨项目经验库               │ │
│  │ 完整/快速/单阶段/恢复                     [global | stack:xxx]     │ │
│  │ Orchestration       Agent configs        Cross-project knowledge │ │
│  └─────────────────────────────────────────────────────────────────┘ │
└───────────────────────────────────┬───────────────────────────────────┘
                                    │ 调度 / dispatches
┌───────────────────────────────────▼───────────────────────────────────┐
│                  Agent 层 / Layer（角色，全局一次 / agents, once）       │
│                  ~/.claude/agents/                                     │
│                                                                       │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  │Tech Lead │──→│Senior Dev│──→│Developer │──→│Reviewer  │──→│Integrator│
│  │ 方案/Plan │   │ 设计/Design│   │ 编码/Code │   │ 审查/Review│  │ 验证/Verify│
│  └──────────┘   └──────────┘   └──────────┘   └────┬─────┘   └────┬─────┘
│                                                    │ BUG 发现    │
│                                          ┌─────────┘             │
│                                          │ auto back to Developer │
│                                          │ max 3 cycles / 最多3轮  │
│                                          └───────────────────────→ 经验沉淀
└───────────────────────────────────┬───────────────────────────────────┘
                                    │ 首次自动创建 / auto-created
┌───────────────────────────────────▼───────────────────────────────────┐
│               Project 层 / Layer（每个项目独立 / per project）          │
│               ./.claude/workflows/                                     │
│                                                                       │
│  ┌─────────────────────────┐        ┌─────────────────────────────┐  │
│  │  multi-agent-flow.md    │        │  lessons-log.md             │  │
│  │  ├─ 继承全局模式          │←──────│  ├─ 每轮自动记录              │  │
│  │  ├─ 项目覆盖              │  ≥3次  │  ├─ ≥3次 → 项目模式          │  │
│  │  └─ 项目特有模式(≤15)     │        │  └─ ≥5次 → 推广全局          │  │
│  │  Inherited + own patterns│        │  Auto-log + promote        │  │
│  └─────────────────────────┘        └─────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────┘
                                    │ ≥5 stable triggers / ≥5次稳定
┌───────────────────────────────────▼───────────────────────────────────┐
│              跨项目推广 / Cross-Project Promotion                       │
│                                                                       │
│  项目模式 ──→ [stack: SpringBoot3] ──→ [global]                       │
│  Project      同技术栈自动继承              所有项目自动继承              │
│  pattern      Same stack inherits          All projects inherit       │
└───────────────────────────────────────────────────────────────────────┘
```

---

## 越用越聪明 / Gets Smarter With Use

每次 Integrator 收尾时自动记录经验 / Integrator auto-logs experience every run:

```
经验 ≥3次同类 → 提炼为项目模式 → ≥5次稳定 → 推广到同技术栈 → 推广到全局
Experience → ≥3 same → project pattern → ≥5 stable → stack → global
```

新项目首次使用自动继承全局匹配模式 / New projects auto-inherit matching global patterns.

---

## 5 个角色 / 5 Roles

| 角色 / Role | 出场 / When | 产出 / Output |
|------|-------------|------|
| Tech Lead | 完整模式第一步 / Full mode first | 技术方案 / Plan |
| Senior Dev | Tech Lead 之后 / After TL | 详细设计 / Design |
| Developer | 设计方案确定后 / After design | 代码 / Code |
| Reviewer | 代码写完自动 / After code | 审查报告 / Review |
| Integrator | 审查通过后自动 / After review | 测试+经验 / Test+Learn |

---

## 文件结构 / File Structure

```
multi-agent-dev/                     ← 这个仓库（全局一次）this repo (once)
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
        ├── SKILL.md                 # Engine / 引擎
        ├── agents.json
        ├── pattern-registry.md      # Knowledge base / 知识库
        └── project-init-template/   # Templates / 模板
```

---

## 速查 / Quick Reference

| 我想... / I want to... | 命令 / Command |
|---------|------|
| 开发新功能 / New feature | `/multi-agent-dev Add export` |
| 重构模块 / Refactor | `/multi-agent-dev Split OrderService` |
| 修样式 / Fix style | `/multi-agent-dev [quick] Fix button` |
| 只审查 / Review only | `/multi-agent-dev stage 4` |
| 继续中断 / Resume | `/multi-agent-dev resume` |

---

## License

MIT
