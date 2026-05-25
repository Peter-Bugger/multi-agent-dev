# 恢复机制（/multi-agent-dev resume）

> 此文件是 SKILL.md 的补充文档，包含 Resume 机制的详细定义。

---

当用户调用 `/multi-agent-dev resume` 时：

1. 读取项目根目录下的 `WORKFLOW_STATUS.md`
2. 从 `Current Stage` 字段确定当前阶段
3. 读取该阶段所需的所有输入文件（PLAN.md / DESIGN.md / REVIEW.md）
4. **如果是 Stage 3**：检查 `Task Progress` 表，从第一个 `🔄 in_progress` 或 `⬜ pending` 的 Task 继续
5. 从中断处继续执行

**恢复前检查（NEW）：**

编排者在 resume 时应额外执行以下验证：
1. 检查当前阶段所需的产出文件是否存在且内容合理（如 Stage 2 的 DESIGN.md 是否非空且包含任务清单）
2. 如产出文件损坏或缺失 → 提示用户并提供选项：重新执行当前阶段 / 回退到上一阶段
3. 检查 WORKFLOW_STATUS.md 的完整性（JSON/Markdown 格式是否完好）
4. 展示恢复摘要：当前阶段、已完成 Task 数、待完成 Task 数、重试计数
5. 等待用户确认后继续执行
