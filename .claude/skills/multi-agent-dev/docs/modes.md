# 模式定义补充

> 此文件是 SKILL.md 的补充文档，包含 Medium 和 Quick 模式的完整定义。

---

## 中等模式 `[medium]` 详细定义

`[medium]` 模式适用于中等复杂度的功能开发。跳过 Senior Dev 详细设计阶段和独立 Review 阶段，但保留产品方向确认和技术方案设计。

### [medium] Stage 0: Product Manager（简化版）

- 读取项目现状，使用 AskUserQuestion 询问项目优先级和范围确认（同完整模式）
- 输出简化版 PRODUCT_BRIEF.md（包含项目章程 + MoSCoW 范围，跳过完整竞品分析和用户旅程图）
- 不输出 PRD.md

### [medium] Stage 1: Tech Lead

- 同完整模式 Stage 1：使用 AskUserQuestion 进行需求确认、技术权衡、可扩展性询问
- 输出完整 PLAN.md（含可扩展性考量章节）

### [medium] Stage 3: Developer

同完整模式 Stage 3：按 PLAN.md 直接实现（无 DESIGN.md 时以 PLAN.md 为任务清单），每 Task 运行验证命令。

### [medium] Stage 5: Integrator

- 运行全量构建 + 验证
- **全量回归测试**：运行测试套件验证无回归
- **覆盖率检查**：运行覆盖率分析，>= 80% 通过
- **内嵌 Review**：检查 diff 中是否有明显问题（硬编码、未使用 import、类型错误、安全漏洞）
- 输出简化最终报告（含阶段回顾和测试覆盖摘要）
- 内嵌 Review 发现严重问题时，可建议用户改用完整模式重做

### [medium] 自动推进

`[medium]` 模式所有阶段（Stage 0→1→3→5）全部自动推进，无人工断点。编排者在不等待用户输入的情况下连续调用各阶段 Agent。

### [medium] WORKFLOW_STATUS.md

```
# Workflow Status
## Task: <任务描述>
## Mode: Medium
## Current Stage: Stage X/6: <角色名>

## Task Progress（从 PLAN.md 模块拆分初始化）
| Task | 文件 | 状态 | 验证结果 |
|------|------|------|----------|
| Task 1: <标题> | `<path>` | ✅ done | ✅ type-check + lint passed |

## Retry Counters
- Integrator cycle: 0/3
- Coverage cycle: 0/2

## Files Changed
| File | Change Type |
|------|-------------|
| - | - |

## Timing
| Stage | 耗时 |
|-------|------|
| Stage 0: 产品经理 | - |
```

---

## 轻量模式 `[quick]` 详细定义

`[quick]` 模式适用于单文件/少文件的小改动。

### [quick] Stage 3: Developer

Quick 模式 Developer 在写代码前必须先完成 mini-design，防止方向跑偏。

**步骤：**

1. **Mini-Design（必须先做，3-5 行）**：
   - 搜索确认要修改的文件（Glob/Grep 验证文件存在）
   - 输出 mini-design：改哪个文件、改什么、怎么验证
   - 将 mini-design 写入 WORKFLOW_STATUS.md 的 `## Quick Design` 段

   **Mini-Design 模板：**
   ```
   ## Quick Design
   - **文件：** `<文件路径>`（已通过搜索确认存在）
   - **改动：** <具体改什么，1-2 句话>
   - **验证：** <验证命令>
   - **测试：** <是否需要补充测试，测试文件路径>
   ```

2. **实现代码**：按 mini-design 执行改动
3. **生成测试（如适用）**：对涉及业务逻辑的改动，生成对应单元测试
4. **验证**：运行验证命令 + 测试，通过后标记 Task 完成
5. 更新 WORKFLOW_STATUS.md Task 进度

### [quick] Stage 5: Integrator

- 运行全量构建 + 验证
- **全量回归测试**：运行测试套件验证无回归
- **内嵌简化 Review**：检查 diff 中是否有明显问题（硬编码、未使用 import、类型错误）
- 输出简化最终报告（含测试覆盖摘要，跳过阶段回顾部分）

### [quick] 自动推进

`[quick]` 模式 Stage 3→5 自动推进，无人工断点。编排者在 Developer 完成后直接触发 Integrator，无需用户输入 `/multi-agent-dev resume`。

### [quick] WORKFLOW_STATUS.md

```
# Workflow Status
## Task: <任务描述>
## Mode: Quick
## Current Stage: Stage 3/6: Developer

## Quick Design
- **文件：** `<文件路径>`
- **改动：** <具体改什么>
- **验证：** <验证命令>

## Task Progress
| Task | 文件 | 状态 | 验证结果 |
|------|------|------|----------|
| Task 1: <标题> | `<path>` | 🔄 in_progress | - |

## Retry Counters
- Integrator cycle: 0/3
- Coverage cycle: 0/2

## Files Changed
| File | Change Type |
|------|-------------|
| - | - |

## Timing
| Stage | 耗时 |
|-------|------|
| Stage 3: 开发 | - |
```
