# WORKFLOW_STATUS.md 模板

> 此文件是 SKILL.md 的补充文档，包含 WORKFLOW_STATUS.md 的完整模板定义。

---

## Full Mode

```
# Workflow Status

## Task
<任务描述>

## Mode
[Full] / [Medium] / [Quick]

## Current Stage: <Stage N/6: 角色名>

## Completed Stages
- [x] Stage 0: Product Manager
- [x] Stage 1: Tech Lead
- [x] Stage 2: Senior Developer
- [ ] Stage 3: Developer
- [ ] Stage 4: Reviewer
- [ ] Stage 5: Integrator

## Task Progress（从 DESIGN.md 任务清单初始化）
| Task | 文件 | 风险等级 | 状态 | 验证结果 |
|------|------|---------|------|----------|
| Task 1: <标题> | `<path>` | Critical | ✅ done | ✅ type-check + lint passed |
| Task 2: <标题> | `<path>` | High | 🔄 in_progress | - |
| Task 3: <标题> | `<path>` | Low | ⬜ pending | - |

## Retry Counters
- Reviewer cycle: 0/3
- Integrator cycle: 0/3
- Coverage cycle: 0/2

## Issues Log
| Stage | Task | Issue | Status |
|-------|------|-------|--------|
| - | - | - | - |

## Files Changed（Developer 完成后填入）
| File | Change Type | Task |
|------|-------------|------|
| - | - | - |

## Rollback Points（NEW）
| 时间 | 触发原因 | Git Commit / Stash | 备注 |
|------|---------|-------------------|------|
| - | - | - | - |

## Timing（NEW — 每个阶段完成时记录）
| Stage | 耗时 | Token 消耗（估算） | 重试次数 | 备注 |
|-------|------|-------------------|---------|------|
| Stage 0: 产品经理 | 3min | ~12K | 0 | |
| Stage 1: 技术负责人 | 5min | ~25K | 0 | |
| **总计** | **Xmin** | **~XK tokens** | **N** | |
```
