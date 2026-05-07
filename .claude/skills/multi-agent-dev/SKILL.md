---
name: multi-agent-dev
description: Execute a 5-stage multi-agent development workflow (Tech Lead → Senior Dev → Developer → Reviewer → Integrator). Use for any software development task: features, refactoring, bug fixes, or new projects. Produces PLAN.md, DESIGN.md, REVIEW.md artifacts. Supports [quick] mode, single-stage execution, and auto-loop review cycles.
---

# /multi-agent-dev — 多Agent开发工作流

按 **Tech Lead → Senior Developer → Developer → Reviewer → Integrator** 五阶段流程执行开发任务。每个阶段通过 Subagent 独立上下文执行，通过标准文件传递阶段产出，支持 Reviewer/Integrator 最多 3 次重试循环。

## 使用方法

```
/multi-agent-dev <任务描述>           # 完整五阶段流程
/multi-agent-dev [quick] <任务描述>   # 轻量模式：跳过 Tech Lead + Senior Dev（含 mini-design）
/multi-agent-dev stage <N>           # 单独执行某个阶段（1-5）
/multi-agent-dev resume              # 恢复中断的工作流（读取 WORKFLOW_STATUS.md）
```

**示例：**

```
/multi-agent-dev 在用户中心页面添加头像上传功能
/multi-agent-dev 重构文章列表组件，拆分出独立的卡片组件
/multi-agent-dev [quick] 修复登录按钮的垂直对齐问题
/multi-agent-dev [quick] 更新 API 基础 URL 配置
/multi-agent-dev stage 4            # 只对当前 diff 执行 Reviewer 审查
/multi-agent-dev stage 5            # 只执行集成验证
/multi-agent-dev resume
```

### 模式选择指南

| 条件                  | 推荐模式                  |
| ------------------- | --------------------- |
| 新功能、跨模块改动、架构变更      | 完整模式（5阶段）             |
| Bug 修复、CSS 调整、单文件小改 | `[quick]` 轻量模式        |
| 已有代码变更，只需审查或集成验证    | `stage 4` 或 `stage 5` |
| 不确定复杂度              | 完整模式（保守）              |

### 单阶段执行 `stage <N>`

`stage <N>` 模式用于独立执行某个阶段，不依赖完整工作流。适用于：已有代码变更只需审查、只需集成验证、或重新执行某个阶段。

**前置文件要求：**

| Stage     | 必需的前置文件                 | 无前置文件时的行为                     |
| --------- | ----------------------- | ----------------------------- |
| `stage 1` | 无                       | 正常执行（需用户提供任务描述）               |
| `stage 2` | `PLAN.md`               | 提示用户先运行完整模式生成 PLAN.md         |
| `stage 3` | `DESIGN.md` + `PLAN.md` | 提示用户先运行 Stage 1-2             |
| `stage 4` | 代码变更（git diff）          | 基于当前 diff 直接审查，不需 PLAN/DESIGN |
| `stage 5` | 代码变更                    | 运行全量构建+测试，不需 PLAN/DESIGN      |

**注意：**

- `stage 4` 独立执行时，审查标准以代码质量为主（无 DESIGN.md 可比对）
- `stage 5` 独立执行时，仅做构建+测试验证，无最终报告中的阶段回顾
- 单阶段执行不创建 WORKFLOW_STATUS.md，仅输出该阶段的交付物

---

## 项目初始化（首次使用自动触发）

首次在项目中调用 `/multi-agent-dev` 或 `/workflow` 时，自动检测项目级文件是否存在：

```
检测 .claude/workflows/ 目录是否存在？
  ├─ 存在 → 正常执行工作流
  └─ 不存在 → 自动初始化:
       1. 创建 .claude/workflows/ 目录
       2. 从 ~/.claude/skills/multi-agent-dev/project-init-template/ 复制模板
          → multi-agent-flow.md（填入项目名、技术栈）
          → lessons-log.md
       3. 从 ~/.claude/skills/multi-agent-dev/pattern-registry.md 拉取匹配模式:
          → [global] 作用域的全部继承
          → [stack: xxx] 按项目技术栈标签匹配继承
       4. 预填入 multi-agent-flow.md 的「继承模式」区
       5. 输出初始化摘要: "已初始化项目工作流，从全局继承了 N 条模式"
```

**技术栈检测**：通过分析项目文件自动推断（如 `package.json` → Vue3/React，`pom.xml` → SpringBoot3/Java）。

**角色定义加载**：
- 默认使用全局 `~/.claude/skills/multi-agent-dev/agents.json`
- 项目可在 `.claude/agents.json` 中覆盖特定角色（只需写要覆盖的角色，其余沿用全局）
- 加载时先读全局，再 merge 项目级覆盖

---

## 工作流概览

### 完整模式（5 阶段）

```
Stage 1: Tech Lead        ──→  PLAN.md（含需求确认关卡）
Stage 2: Senior Developer ──→  DESIGN.md + 任务拆分清单（可修订 PLAN.md）
Stage 3: Developer        ──→  代码实现 + 逐 Task 本地验证
Stage 4: Reviewer         ──→  REVIEW.md（git diff 精确范围，可能循环 1-3 次）
Stage 5: Integrator       ──→  最终集成验证报告
```

### 轻量模式 `[quick]`（2 阶段）

```
Stage 3: Developer        ──→  代码实现 + 本地验证
Stage 5: Integrator       ──→  集成验证（含简化 Review）
```

---

## 工作流文件

工作流在项目根目录下创建/更新以下文件：

| 文件                   | 创建于     | 用途                              |
| -------------------- | ------- | ------------------------------- |
| `PLAN.md`            | Stage 1 | 技术方案：需求确认、技术选型、架构设计、模块拆分、数据流    |
| `DESIGN.md`          | Stage 2 | 详细设计：接口设计、组件设计、任务拆解清单（含依赖）、验收标准 |
| `REVIEW.md`          | Stage 4 | 审查报告：BLOCKER/INFO 列表、审查结论       |
| `WORKFLOW_STATUS.md` | Stage 1 | 进度追踪：当前阶段、Task 级别进度、重试计数、问题记录   |

---

## 各阶段详细定义

### Stage 1: Tech Lead（技术负责人）

**宣言：** `=== Stage 1/5: Tech Lead ===`

**执行方式：** 使用 Agent 工具，`subagent_type="tech-lead"`，隔离上下文中分析任务。

**输入：** 用户的任务描述

**输出：** `PLAN.md`、`WORKFLOW_STATUS.md`（初始化）

**职责：**

1. **需求确认（CRITICAL — 必须先做）**：先输出一段 5-8 句话的需求理解摘要，附上关键约束和不在范围内的内容，等待用户回复 `Y` 确认后方可继续。不得跳过此步骤直接写 PLAN.md。
2. **代码库验证**：确认后，用 Glob/Grep/Agent 搜索代码库验证涉及的文件和接口真实存在，不得凭空猜测文件路径。
3. 技术选型：推荐适合项目的技术方案（框架、库、工具），优先复用项目已有依赖。
4. 架构设计：设计整体架构，明确模块边界。
5. 模块拆分：将任务拆分为逻辑模块，标注模块间依赖关系。
6. 数据流设计：说明数据流转路径和状态管理方案。
7. API/接口设计：定义核心接口签名。

**需求确认模板（必须先输出并获得用户 Y 确认）：**

```markdown
## 需求确认（请回复 Y 确认后我将输出完整 PLAN.md）

1. 我理解你要做的是：<用你自己的话复述>
2. 关键约束：<技术栈、兼容性、范围边界>
3. 不在本次范围内：<明确排除的内容>
4. 预计影响：<大致文件数/模块数>

> 如果理解正确，请回复 **Y**，我将继续输出完整技术方案。
> 如果有偏差，请指出，我会调整理解。
```

**PLAN.md 必须包含的内容：**

```markdown
# 技术方案: <任务标题>

## 1. 需求分析
- 功能需求
- 非功能需求（性能、可维护性、安全性）
- 边界条件

## 2. 技术选型
- 使用的框架/库/工具及其理由（优先复用项目现有依赖）
- 是否需要新增依赖

## 3. 架构设计
- 整体架构说明
- 模块划分
- 模块间依赖关系

## 4. 数据流设计
- 数据流转路径
- 状态管理方案
- 关键数据结构

## 5. 界面/接口设计
- UI 组件结构（前端项目）
- API 接口签名（后端项目）
- 组件 Props/Events 定义

## 6. 影响范围（基于代码库实际搜索，非猜测）
- 需要修改的文件列表（已通过 Glob/Grep 验证存在）
- 是否需要数据库迁移
- 是否需要配置变更

## 7. 风险评估
- 潜在风险
- 回退方案
```

**完成条件：** 用户确认需求 → PLAN.md 已写入 → WORKFLOW_STATUS.md 已初始化。

**检查点：**

```
>> Stage 1/5 完成。PLAN.md + WORKFLOW_STATUS.md 已生成。输入 /multi-agent-dev resume 继续 Stage 2。
```

---

### Stage 2: Senior Developer（高级开发）

**宣言：** `=== Stage 2/5: Senior Developer ===`

**执行方式：** 使用 Agent 工具，`subagent_type="senior-dev"`，隔离上下文中审阅和设计。

**输入：** `PLAN.md`

**输出：** `DESIGN.md`（更新 `WORKFLOW_STATUS.md`）

**职责：**

1. 审阅 PLAN.md：检查方案是否完整、有无遗漏边界情况。
2. **允许修订 PLAN.md**：如发现 PLAN.md 中的问题（遗漏的边界情况、错误的接口签名、不合适的技术选型等），直接修改 PLAN.md 并在修改处标注 `[SENIOR-DEV AMENDMENT: <原因>]`。Developer 应以修订后的 PLAN.md 为准。
3. 详细接口设计：将架构层接口细化为具体签名。
4. 组件设计：细化组件树、Props、Events、Slots。
5. 任务拆解：将工作拆分为 1-3 小时可完成的任务，标注依赖关系。
6. 定义验收标准：每个任务的完成条件 + 具体验证命令。

**DESIGN.md 必须包含的内容：**

```markdown
# 详细设计: <任务标题>

## 1. PLAN 审阅意见
- 对 PLAN.md 的补充/修改建议
- 已修改 PLAN.md 的内容（标注 `[SENIOR-DEV AMENDMENT]`）
- 发现的问题及修复方式

## 2. 详细接口设计
- 组件 Props/Events 完整定义（前端）
- API 请求/响应结构（后端）
- 类型定义/接口定义

## 3. 组件/类设计
- 组件树（前端）
- 类图/职责说明（后端）
- 关键逻辑流程

## 4. 任务拆解清单

### Task 1: <标题>
- **文件：** `<文件路径>`
- **描述：** <做什么>
- **依赖：** 无（或其他 Task N）
- **被依赖：** Task N, Task M
- **验收标准：** <如何验证>
- **验证命令：** `<具体命令，不可用"相关验证"代替>`
- **预估工时：** <时间>

### Task 2: <标题>
...
```

**完成条件：** DESIGN.md 已写入，WORKFLOW_STATUS.md 已更新（Task 列表已录入）。

**检查点：**

```
>> Stage 2/5 完成。DESIGN.md 已生成，任务清单已就绪。输入 /multi-agent-dev resume 继续 Stage 3。
```

---

### Stage 3: Developer（开发）

**宣言：** `=== Stage 3/5: Developer ===`

**执行方式：** 使用 Agent 工具，`subagent_type="developer"`，在隔离上下文中实现代码。

**输入：** `DESIGN.md` + `PLAN.md`（如有 Senior Dev 修订）

**输出：** 代码变更（更新 `WORKFLOW_STATUS.md` 中 Task 进度）

**职责：**

1. 按 DESIGN.md 的任务拆解清单逐个实现，按依赖顺序执行。
2. 每完成一个 Task，立即运行 DESIGN.md 中指定的验证命令，验证通过才能标记完成。
3. 每完成一个 Task，更新 WORKFLOW_STATUS.md 中该 Task 的状态为 ✅ done 并记录验证结果。
4. 遇到上游设计问题：在 WORKFLOW_STATUS.md 的 Issues Log 记录，标注 Blocked，继续执行可完成的部分。
5. 不修改 DESIGN.md 或 PLAN.md（问题上报即可）。

**修复模式（Reviewer 自动循环触发）：**

当 Developer 由 Reviewer 自动循环触发（而非用户手动 resume）时，进入修复模式：

- **输入：** `REVIEW.md`（仅读取 BLOCKER 列表）
- **职责：** 仅修复 REVIEW.md 中标记为 🔴 BLOCKER 的问题，不做其他改动
- **验证：** 修复后运行项目全局验证命令（`npm run type-check && npm run build-only` 等）
- **输出：** 在 REVIEW.md 每个已修复的 BLOCKER 后标注 `[已修复]` 及修复说明
- **规则：** 最小化变更范围，只修复问题不引入额外重构；修复完成后回到 Stage 4

**修复模式 prompt 要点（编排者必须包含）：**

```
你是 Developer，进入修复模式。读取 REVIEW.md，只修复 🔴 BLOCKER 问题。
每个 BLOCKER 修复后标注 [已修复]。不做任何 BLOCKER 列表之外的改动。
全部修复后运行验证命令确认通过。
```

**验证命令（DESIGN.md 中每个 Task 已指定，不可跳过）：**

- 前端项目示例：`npm run type-check && npm run lint && npm run build-only`
- 后端项目示例：`mvn compile test -pl <模块名>`
- 每个 Task 完成 = 代码变更完成 + 验证命令通过

**错误处理：**

- 验证失败 → 读取错误信息 → 分析根因 → 修复 → 重新验证
- 同一 Task 重复失败：最多重试 3 次
- 3 次仍失败：在 WORKFLOW_STATUS.md 记录失败详情（Task 级别），暂停并报告
- 后续 Task 不受影响，可继续执行

**完成条件：** 所有 Task 实现完成，每个 Task 验证通过，WORKFLOW_STATUS.md Task 进度全部 ✅。

**检查点：**

```
>> Stage 3/5 完成。代码实现完毕，所有 Task 验证通过。输入 /multi-agent-dev resume 继续 Stage 4。
```

---

### Stage 4: Reviewer（审查）

**宣言：** `=== Stage 4/5: Reviewer ===`

**执行方式：** 使用 Agent 工具，`subagent_type="reviewer"`，独立上下文审查。

**输入：** 代码变更 + `DESIGN.md` + `PLAN.md`

**输出：** `REVIEW.md`（更新 `WORKFLOW_STATUS.md`）

**职责：**

1. **精确变更范围**：运行 `git diff --stat` 或 `git diff main..HEAD` 获取本次变更的精确文件列表和行数，审查对象只包括 diff 中的代码。
2. 审查代码质量：正确性、可维护性、性能、安全性。
3. 审查设计一致性：代码是否遵循 DESIGN.md 的设计。
4. 审查边界情况：错误处理、空状态、加载状态、极端输入。
5. 问题分类记录，每个问题附带精确文件和行号。

**REVIEW.md 格式：**

```markdown
# 审查报告: <任务标题>

## 审查摘要
- 审查范围：[通过 git diff 获取的变更文件列表]
- Diff 统计：+N / -M 行，X 个文件
- 审查结论：**APPROVED** / **REQUEST_CHANGES** / **NEEDS_DISCUSSION**

## 问题列表

### 🔴 BLOCKER（必须修复）
1. `文件路径:行号` — 问题描述 — 修复方式：[具体修复方法]
2. `文件路径:行号` — 问题描述 — 修复方式：[具体修复方法]

### 🟢 INFO（可选优化）
1. `文件路径:行号` — 建议描述

## 循环记录
- 当前循环：N/3
- 截至本轮的 BLOCKER 数：X
- 累计已修复 BLOCKER 数：Y
```

**循环规则（自动执行，无需用户干预）：**

编排者（主会话）在 Reviewer 输出 REVIEW.md 后，自动执行以下判断：

```
if 审查结论 == "APPROVED":
    → 自动进入 Stage 5 Integrator
elif 审查结论 == "REQUEST_CHANGES" and 循环次数 < 3:
    → 主会话自动触发 Stage 3 Developer（修复模式），传入 REVIEW.md
    → Developer 修复 BLOCKER 后返回
    → 主会话重新触发 Stage 4 Reviewer（循环次数 +1）
    → 重复此流程
elif 审查结论 == "REQUEST_CHANGES" and 循环次数 >= 3:
    → 标记为 NEEDS_DISCUSSION，暂停并报告用户
```

**关键：这是编排者（主会话）的职责，不是用户的操作。用户不需要输入任何命令。**
循环最多 3 次，3 次后仍有 BLOCKER → 标记 `NEEDS_DISCUSSION`，暂停并报告用户。

**编排者自动循环实现伪代码：**

```
cycle = 0
while cycle < 3:
    invoke Reviewer agent → produce REVIEW.md
    if REVIEW.md conclusion == "APPROVED":
        break  # 进入 Stage 5
    else:  # REQUEST_CHANGES
        cycle += 1
        if cycle >= 3:
            mark as NEEDS_DISCUSSION, report to user, STOP
        invoke Developer agent (fix-only mode) with REVIEW.md
        # Developer 修复后循环回到 Reviewer
```

**审查规则（嵌入到每次审查中）：**

- 只有 BLOCKER 全部修复后才能通过
- 每个问题必须附带具体修复方式 + 精确文件位置
- 审查对象只包括 `git diff` 范围内的代码变更

**完成条件：** REVIEW.md 已写入，审查结论为 APPROVED（自动进入 Stage 5）或 NEEDS_DISCUSSION（暂停等待用户）。

**检查点（APPROVED → 自动进入 Stage 5）：**

```
>> Stage 4/5 完成。代码审查通过，REVIEW.md 结论为 APPROVED。自动进入 Stage 5...
```

**检查点（REQUEST_CHANGES → 自动循环）：**

```
>> Stage 4/5 — REQUEST_CHANGES。REVIEW.md 中有 N 个 BLOCKER。自动触发 Stage 3 修复模式...
>> [Developer 修复完成] → 重新进入 Stage 4（第 X/3 次循环）...
```

**检查点（NEEDS_DISCUSSION → 暂停）：**

```
>> Stage 4/5 — NEEDS_DISCUSSION。已循环 3 次仍有 N 个 BLOCKER 未解决。请用户决策。
```

---

### Stage 5: Integrator（集成）

**宣言：** `=== Stage 5/5: Integrator ===`

**执行方式：** 使用 Agent 工具，`subagent_type="integrator"`，独立上下文运行全量构建和测试。

**输入：** 代码变更 + `REVIEW.md` + `DESIGN.md`

**进入方式：** Stage 4 APPROVED 后自动进入（完整模式），或用户通过 `stage 5` 手动触发。

**输出：** 集成验证报告（更新 `WORKFLOW_STATUS.md`）

**职责：**

1. 确认所有 BLOCKER 已修复（从 REVIEW APPROVED 进入时）。
2. 全量构建验证：运行项目完整的 build 命令。
3. 全量测试：运行项目测试套件（如存在）。
4. 最终检查：确认 WORKFLOW_STATUS.md 记录完整（所有 Task 状态 + Issues 处理情况）。
5. 清理工作流文件：询问用户是否保留 PLAN.md / DESIGN.md / REVIEW.md / WORKFLOW_STATUS.md。

**失败处理（自动循环，同 Reviewer 机制）：**

- 集成失败 → 编排者自动定位问题 → 修复（最小化变更范围）→ 重新运行集成检查
- 循环直到 SUCCESS 或达到 3 次上限
- 3 次后仍失败：暂停并报告详细错误，等待用户决策

**完成条件：** 集成验证通过，输出最终报告。

**最终报告格式：**

```markdown
# 集成验证报告

## 验证结果
- **构建：** ✅ PASS / ❌ FAIL
- **类型检查：** ✅ PASS / ❌ FAIL
- **Lint：** ✅ PASS / ❌ FAIL
- **测试：** ✅ PASS / ❌ FAIL / ⬜ 跳过（项目未配置）

## 变更总结
- **新增文件：** N 个
- **修改文件：** N 个
- **代码行变更：** +N / -M

## 遗留问题（如有）
| 问题 | 影响范围 | 建议处理 |
|------|---------|---------|
| ... | ... | ... |

## 工作流阶段回顾
| 阶段 | 状态 |
|------|------|
| Stage 1: Tech Lead | ✅ |
| Stage 2: Senior Dev | ✅ |
| Stage 3: Developer | ✅ |
| Stage 4: Reviewer | ✅（N 次循环） |
| Stage 5: Integrator | ✅ |

## 最终结论
✅ 可交付 / ❌ 需要进一步处理
```

**检查点：**

```
>> Stage 5/5 完成。集成验证通过，工作流结束。
```

---

## 全局规则（所有阶段必须遵守）

1. **阶段隔离**：每个阶段使用独立 Agent（`subagent_type` 指定角色），主会话只做编排和检查点输出。
2. **信息传递**：通过标准文件（PLAN.md / DESIGN.md / REVIEW.md / WORKFLOW_STATUS.md）传递信息。每个 Agent prompt 中明确指定要读取的输入文件路径。
3. **透明输出**：每个阶段开始时输出 `=== Stage N/5: [角色名] ===`，结束时输出该阶段交付物摘要 + 检查点提示。
4. **项目无关性**：通过读取项目根目录配置文件自动发现项目类型和验证命令。
5. **质量优先**：不跳过任何阶段。轻量模式 `[quick]` 是唯一例外。
6. **问题上报**：任何阶段发现上游问题，可以指出，Senior Dev 可直接修订 PLAN.md，其他角色记录到 WORKFLOW_STATUS.md 不上游修改。
7. **最小化变更**：只修改实现目标所需的文件，不做额外重构或优化。
8. **自动循环**：Reviewer 发现 BLOCKER → 编排者自动触发 Developer 修复模式 → 重新 Review，同会话内自动完成无需用户干预，最多 3 次。Integrator 失败同理，自动修复后重试。
9. **代码库验证**：Stage 1 中涉及的文件路径必须通过 Glob/Grep 实际搜索验证，不得凭空猜测。
10. **需求对齐优先**：Stage 1 必须先获得用户 Y 确认后才能写完整 PLAN.md。
11. **精确变更范围**：Reviewer 和 Integrator 必须用 `git diff` 确定变更范围。
12. **Task 级别追踪**：WORKFLOW_STATUS.md 追踪到 Task 粒度，而非仅阶段粒度。
13. **项目优先**：项目 `multi-agent-flow.md` 的「项目覆盖」区优先于全局模式。当项目做法与继承的全局模式矛盾时，以项目覆盖版本为准，覆盖时注明原因。项目不能删除全局模式，只能覆盖。
14. **经验蒸馏**：integrator 追加经验时先过质量门槛（可操作+可泛化+作用域判定）；同类≥3次提炼为模式；提炼后删除原始条目；模式表达上限时淘汰最久未触发的。
15. **跨项目推广**：模式在本项目稳定触发≥5次且无障碍 → 评估推广到更高作用域。推广前必须读取 pattern-registry.md 做冲突检测；有冲突按「特殊化→保持项目级→上报用户」协议解决。
16. **新项目继承**：新项目初始化时自动继承全局注册表中的 [global] 模式，以及匹配当前技术栈的 [stack] 模式。项目可覆盖但不可删除继承的模式。

---

## 工作流状态文件（WORKFLOW_STATUS.md）— 增强版

```
# Workflow Status

## Task
<任务描述>

## Mode
[Full] / [Quick]

## Current Stage: <Stage N/5: 角色名>

## Completed Stages
- [x] Stage 1: Tech Lead
- [x] Stage 2: Senior Developer
- [ ] Stage 3: Developer
- [ ] Stage 4: Reviewer
- [ ] Stage 5: Integrator

## Task Progress（从 DESIGN.md 任务清单初始化）
| Task | 文件 | 状态 | 验证结果 |
|------|------|------|----------|
| Task 1: <标题> | `<path>` | ✅ done | ✅ type-check + lint passed |
| Task 2: <标题> | `<path>` | 🔄 in_progress | - |
| Task 3: <标题> | `<path>` | ⬜ pending | - |

## Retry Counters
- Reviewer cycle: 0/3
- Integrator cycle: 0/3

## Issues Log
| Stage | Task | Issue | Status |
|-------|------|-------|--------|
| - | - | - | - |

## Files Changed（Developer 完成后填入）
| File | Change Type | Task |
|------|-------------|------|
| - | - | - |
```

---

## 恢复机制（/multi-agent-dev resume）— 增强版

当用户调用 `/multi-agent-dev resume` 时：

1. 读取项目根目录下的 `WORKFLOW_STATUS.md`
2. 从 `Current Stage` 字段确定当前阶段
3. 读取该阶段所需的所有输入文件（PLAN.md / DESIGN.md / REVIEW.md）
4. **如果是 Stage 3**：检查 `Task Progress` 表，从第一个 `🔄 in_progress` 或 `⬜ pending` 的 Task 继续
5. 从中断处继续执行

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
   ```

2. **实现代码**：按 mini-design 执行改动

3. **验证**：运行验证命令，通过后标记 Task 完成

4. 更新 WORKFLOW_STATUS.md Task 进度

### [quick] Stage 5: Integrator

- 运行全量构建 + 验证
- **内嵌简化 Review**：检查 diff 中是否有明显问题（硬编码、未使用 import、类型错误）
- 输出简化最终报告（跳过阶段回顾部分）

### [quick] WORKFLOW_STATUS.md

```
# Workflow Status
## Task: <任务描述>
## Mode: Quick
## Current Stage: Stage 3/5: Developer

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

## Files Changed
| File | Change Type |
|------|-------------|
| - | - |
```

---

## 注意事项

- 如果当前项目没有源文件（全新项目），Developer 阶段从零开始创建
- 如果验证工具不存在（如项目没有配置 lint），跳过该检查项并在最终报告中记录
- 用户可在任意阶段中断对话，后续通过 `/multi-agent-dev resume` 恢复
- 每个阶段的交付物必须在文件系统中可验证，不能仅存在于对话中
- 轻量模式 `[quick]` 仅适用于以下情况：单文件改动、CSS 微调、配置修改、简单 bug 修复、类型修复
- 如果 `[quick]` 模式下 Integrator 发现重大问题（3 次自动修复仍失败），可建议用户改用完整模式重做
