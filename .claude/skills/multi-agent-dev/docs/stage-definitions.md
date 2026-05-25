# Stage 详细定义

> 此文件是 SKILL.md 的补充文档，包含 Stage 0-5 的完整详细定义。
> SKILL.md 中的 Stage 定义是精简版本，编排者在需要详细定义时应读取此文件。

---

### Stage 0: Product Manager / Project Manager（产品经理 + 项目经理）

**宣言：** `=== Stage 0/6: Product Manager / Project Manager ===`

**执行方式：** 使用 Agent 工具，`subagent_type="product-manager"`，隔离上下文中明确产品方向和项目范围。

**输入：** 用户的产品方向/功能描述

**输出：** `PRODUCT_BRIEF.md`（含项目章程）、`PRD.md`（可选）

**职责：**

1. **信息收集**：先读取项目 README.md 和现有代码结构，理解项目现状
2. **项目经理视角 — 关键决策询问（CRITICAL）**：使用 `AskUserQuestion` 工具询问用户：
   - **项目优先级权衡**：快速交付 / 稳健交付 / 可扩展优先
   - **范围确认**：展示理解的核心范围，确认/调整/缩减
3. **项目章程定义**：商业目标、项目范围与边界、干系人分析、约束与假设
4. **产品方向明确**：用 Lean Canvas / JTBD / 竞品分析等方法论，输出产品定位、用户画像
5. **功能范围控制**：用 MoSCoW 明确 Must/Should/Could/Won't，设定成功指标
6. **PRD 展开**：对 Must have 功能展开 PRD.md（用户故事+验收条件）
7. **风险识别**：从项目角度（商业风险）和产品角度（市场/体验风险）列出关键假设和风险

**PRODUCT_BRIEF.md 必须包含的内容：**

```markdown
# 产品简报: <任务/功能名称>

## 0. 项目章程（项目经理视角 — NEW）
- **商业目标**：解决什么商业问题？为什么现在做？
- **项目范围（一句话）**：
- **明确不在范围内的**：
- **与现有系统的关系**：
- **关键干系人**：干系人 A：[角色] — [期望]
- **约束**：时间/人力/预算/技术
- **关键假设**（需要验证的）
- **项目风险**：[描述] — 缓解：[策略]

## 1-6. 保留原有结构（产品定位、市场竞品、用户需求、功能范围、成功指标、产品风险）
```

**产物文件位置：** 项目根目录或 `.claude/workflows/`

**完成条件：** PRODUCT_BRIEF.md 已写入（含项目章程），PRD.md 按需已写入。

**检查点：**

```
>> Stage 0/6 完成。PRODUCT_BRIEF.md + PRD.md 已生成。
```

**Stage 0 完成确认（AskUserQuestion — NEW）：**

Stage 0 完成后，编排者必须使用 `AskUserQuestion` 暂停并让用户确认产品方向是否正确：

```json
{
  "question": "Stage 0 完成。PRODUCT_BRIEF.md 和 PRD.md 已生成。\n\n核心范围：<Must have 列表摘要>\n关键假设：<1-2 个关键假设>\n\n请确认产品方向是否正确，是否进入技术方案阶段？",
  "header": "产品方向确认",
  "options": [
    { "label": "确认，进入技术方案", "description": "产品方向和范围正确，直接进入 Stage 1 技术方案设计。" },
    { "label": "需要调整", "description": "产品方向或范围有需要修改的地方，我会在回复中说明。" }
  ]
}
```

用户确认后自动进入 Stage 1。如用户选择"需要调整"，等待用户反馈后重新进入 Stage 0 修正。

**注意：**
- 如果任务是纯技术改进（无用户感知的代码重构、性能优化等），跳过 PRODUCT_BRIEF 和 PRD，跳过此确认步骤，直接输出简化版影响分析后进入 Stage 1
- **AskUserQuestion 询问不可跳过**：Stage 0 内的项目优先级和范围确认 + 此处的产品方向确认必须在进入 Stage 1 前完成
- Tech Lead (Stage 1) 必须以 PRODUCT_BRIEF.md 为输入参考，确保技术方案对齐产品方向

---

### Stage 1: Tech Lead（技术负责人）

**宣言：** `=== Stage 1/6: Tech Lead ===`

**执行方式：** 使用 Agent 工具，`subagent_type="tech-lead"`，隔离上下文中分析任务。

**输入：** `PRODUCT_BRIEF.md`（如存在）+ 用户的任务描述

**输出：** `PLAN.md`、`WORKFLOW_STATUS.md`（初始化）

**职责：**

1. **需求确认（CRITICAL — 必须使用 AskUserQuestion）**：使用 `AskUserQuestion` 工具进行三轮结构化询问：
   - **问题 1 — 需求理解确认**：展示 5-8 句需求理解摘要 + 关键约束 + 不在范围内的内容。选项：理解正确 / 需要调整 / 需要补充
   - **问题 2 — 技术方案权衡**：根据具体场景展示 2-3 个技术方案（如：最小改动 / 适度抽象 / 全面重构），每个附带具体利弊
   - **问题 3 — 可扩展性要求**（多选）：询问未来 3-6 月可能的变化方向，选项根据具体场景定制
   - 不得跳过此步骤直接写 PLAN.md
2. **PRD 验收条件提取（NEW）**：从 PRD.md 中逐条提取所有用户故事的 Given/When/Then 验收条件，填入 PLAN.md Section 1 的「PRD 验收条件摘要」表格中。
3. **代码库验证**：确认后，用 Glob/Grep/Agent 搜索代码库验证涉及的文件和接口真实存在，不得凭空猜测文件路径。
4. 技术选型：推荐适合项目的技术方案（框架、库、工具），优先复用项目已有依赖。
5. 架构设计：设计整体架构，明确模块边界。
6. 模块拆分：将任务拆分为逻辑模块，标注模块间依赖关系。
7. 数据流设计：说明数据流转路径和状态管理方案。
8. API/接口设计：定义核心接口签名，所有接口必须列出完整签名含错误码。
9. **可扩展性考量**：基于用户的 AskUserQuestion 回答，设计扩展点，说明为扩展做的取舍。

**PLAN.md 模板：** 包含需求分析（功能/非功能/边界/PRD验收条件摘要/非功能需求清单）、技术选型、架构设计、数据流设计、界面/接口设计、影响范围、风险评估、可扩展性考量共 8 个章节。

**完成条件：** 用户通过 AskUserQuestion 确认需求 → PLAN.md 已写入 → WORKFLOW_STATUS.md 已初始化。

**检查点：**

```
>> Stage 1/6 完成。PLAN.md + WORKFLOW_STATUS.md 已生成。自动进入 Stage 2...
```

---

### Stage 2: Senior Developer（高级开发）

**宣言：** `=== Stage 2/6: Senior Developer ===`

**执行方式：** 使用 Agent 工具，`subagent_type="senior-dev"`，隔离上下文中审阅和设计。

**输入：** `PRD.md` + `PLAN.md`

**输出：** `DESIGN.md`（更新 `WORKFLOW_STATUS.md`）

**职责：**

1. 审阅 PLAN.md：检查方案是否完整、有无遗漏边界情况，评估可扩展性设计的可行性。
2. **设计权衡确认（CRITICAL — 使用 AskUserQuestion）**：当存在多个合理设计方案时，使用 `AskUserQuestion` 询问用户偏好。每个选项附带利弊说明。如方向明确可跳过。
3. **允许修订 PLAN.md**：如发现 PLAN.md 中的问题（遗漏的边界情况、错误的接口签名、不合适的技术选型等），直接修改 PLAN.md 并在修改处标注 `[SENIOR-DEV AMENDMENT: <原因>]`。
4. 详细接口设计：将架构层接口细化为具体签名。
5. 组件设计：细化组件树、Props、Events、Slots。
6. 任务拆解：将工作拆分为 1-3 小时可完成的任务，标注依赖关系。
7. 定义验收标准：每个任务的完成条件 + 具体验证命令。

**DESIGN.md 模板：** 包含 PLAN 审阅意见、详细接口设计、组件/类设计、需求追溯矩阵（映射 PRD 验收条件到 Task）、任务拆解清单（含文件/描述/依赖/验收标准/验证命令/测试要求/预估工时）共 5 个章节。

**设计确认询问（AskUserQuestion，超时暂停）：**

Stage 2 完成后，编排者必须使用 `AskUserQuestion` 展示设计摘要并询问用户是否确认。此询问 120 秒超时，超时后**暂停流程**（而非自动确认），提示用户稍后通过 `/multi-agent-dev resume` 继续。

**检查点：**

```
>> Stage 2/6 完成。DESIGN.md 已生成，任务清单已就绪。
```

---

### Stage 3: Developer（开发）

**宣言：** `=== Stage 3/6: Developer ===`

**执行方式：** 使用 Agent 工具，`subagent_type="developer"`，在隔离上下文中实现代码。

**输入：** `DESIGN.md` + `PLAN.md`（如有 Senior Dev 修订）

**输出：** 代码变更（更新 `WORKFLOW_STATUS.md` 中 Task 进度）

**三种模式：**

| 模式 | 触发条件 | 职责 |
|------|---------|------|
| **正常模式** | 首次进入 Stage 3 | 按 DESIGN.md Task 拆解清单逐个实现，每 Task 运行验证命令，生成单元测试，更新进度 |
| **修复模式** | Reviewer 自动循环触发 | 仅修复 REVIEW.md 中的 🔴 BLOCKER，标注 `[已修复]`，运行全局验证 |
| **覆盖率修复模式** | Integrator coverage auto-loop 触发 | 仅补充测试用例，不修改业务代码，每个未覆盖路径至少 1 个测试用例 |

**正常模式核心职责：**
1. 按依赖顺序逐个实现 Task
2. 每 Task 完成后运行验证命令 + 生成测试（按风险等级覆盖测试维度）
3. 更新 WORKFLOW_STATUS.md Task Progress
4. 遇到需求冲突用 `AskUserQuestion` 确认方向

**错误处理：** 验证失败 → 分析根因 → 修复 → 重试（每 Task 最多 3 次）

**检查点：**

```
>> Stage 3/6 完成。代码实现完毕，所有 Task 验证通过。输入 /multi-agent-dev resume 继续 Stage 4。
```

---

### Stage 4: Reviewer（审查）

**宣言：** `=== Stage 4/6: Reviewer ===`

**执行方式：** 使用 Agent 工具，`subagent_type="reviewer"`，独立上下文审查。

**输入：** 代码变更 + `DESIGN.md` + `PLAN.md`

**输出：** `REVIEW.md`（更新 `WORKFLOW_STATUS.md`）

**职责：**
1. 精确变更范围：`git diff --stat` 确定审查范围
2. 代码质量审查：正确性、可维护性、性能、安全性（含 OWASP 检查）
3. 设计一致性审查：代码是否遵循 DESIGN.md
4. 需求追溯检查：逐用户故事确认 PRD 验收条件是否全部满足
5. 测试质量审查：测试文件是否存在、覆盖维度是否足够
6. 健壮性审查：空状态、错误状态、重试逻辑、降级策略
7. AI 代码幻觉检测：虚构 import、不存在 API、硬编码密钥
8. 问题分类：🔴 BLOCKER / 🟢 INFO，附带文件:行号 + 修复方式

**循环规则：**
```
if APPROVED → 自动进入 Stage 5
elif REQUEST_CHANGES and cycle < 3 → 自动触发 Developer 修复模式 → 重新 Review
elif REQUEST_CHANGES and cycle >= 3 → NEEDS_DISCUSSION，暂停并报告用户
```

**NEEDS_DISCUSSION 处理：** 使用 `AskUserQuestion` 询问用户：接受已知风险 / 手动修复 / 缩小范围。

---

### Stage 5: Integrator（集成）

**宣言：** `=== Stage 5/6: Integrator ===`

**执行方式：** 使用 Agent 工具，`subagent_type="integrator"`，独立上下文运行全量构建和测试。

**输入：** 代码变更 + `REVIEW.md` + `DESIGN.md`

**输出：** 集成验证报告 + 经验日志 + 模式提炼（更新 `WORKFLOW_STATUS.md`）

**职责：**
1. 确认所有 BLOCKER 已修复
2. 全量构建验证
3. 全量回归测试
4. 覆盖率检查（>= 80%，不达标触发 coverage auto-loop，最多 2 次）
5. 边界测试抽查（3-5 个用例）
6. 最终检查：WORKFLOW_STATUS.md 完整性
7. 经验日志记录（质量门槛：可操作+可泛化+作用域判定）
8. 模式提炼（同类经验 ≥ 3 次 → 项目模式）
9. 跨项目推广检查（≥ 5 次稳定触发 → 审核门 → pattern-registry.md）
10. 清理工作流文件询问

**失败回滚：** 修复循环失败 ≥ 2 次 → 创建 `git stash` 回滚点 → 记录到 WORKFLOW_STATUS.md → 从干净状态修复。

**最终报告：** 包含验证结果、测试覆盖摘要、变更总结、遗留问题、工作流阶段回顾、自进化记录、最终结论。
