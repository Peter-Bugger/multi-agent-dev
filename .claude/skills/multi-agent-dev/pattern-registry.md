# 全局模式注册表

> 从各项目推广的通用模式在此汇总。integrator 推广前必须读取此文件做冲突检测。
> 硬上限：20条。达到上限时淘汰最久未触发或已被特殊化覆盖的条目。
>
> **新项目初始化时**：此处 [global] 模式全部继承，[stack: xxx] 模式按技术栈匹配继承。
>
> **`[seed]` 标记**：预设的初始模式，尚未被实际项目验证触发。当某个 seed 模式在项目中首次被触发后，自动去除 `[seed]` 标记，记录首次触发日期。

## 模式列表

### [global] — 无视技术栈的工程原则

| # | 模式 | 技术栈标签 | 来源项目 | 推广日期 | 最近触发 | 状态 |
|---|------|-----------|---------|---------|----------|------|
| 1 | Controller异常必须全覆盖：未捕获异常→500 HTML，需@ExceptionHandler+StatusEnum映射 | `Java`, `SpringBoot` | paicoding-springboot3-vue3 | 2026-05-07 | 2026-05-07 | active |
| 2 | SQL注入防护：所有数据库查询必须使用参数化查询（PreparedStatement/JPA参数绑定），禁止字符串拼接SQL | `Java`, `SQL`, `General` | [seed] | 2026-05-25 | - | seed |
| 3 | XSS防护：用户输入输出到HTML时必须转义（前端用框架默认转义，后端输出用HtmlUtils或类似工具），富文本需白名单过滤 | `JavaScript`, `HTML`, `General` | [seed] | 2026-05-25 | - | seed |
| 4 | 输入验证三层防护：前端表单验证（用户体验）+ 后端DTO验证（@Valid/@NotNull等）+ 业务层逻辑验证（数据完整性），不可仅依赖前端验证 | `Java`, `General` | [seed] | 2026-05-25 | - | seed |
| 5 | N+1查询问题：列表查询必须在Repository/DAO层使用JOIN FETCH或批量加载，禁止在循环中逐条查询关联数据。用日志打印SQL确认查询次数 | `Java`, `SpringBoot`, `General` | [seed] | 2026-05-25 | - | seed |
| 6 | 敏感信息不可记录到日志：日志中不得包含密码、Token、身份证号、银行卡号等敏感数据。需在toString/序列化时脱敏或在日志框架中配置脱敏规则 | `General`, `Security` | [seed] | 2026-05-25 | - | seed |
| 7 | Git提交粒度：每次提交应对应一个逻辑变更（一个Task），不得将多个不相关的改动混在一次提交。提交信息格式：`<type>: <简短描述> (#issue)` | `General` | [seed] | 2026-05-25 | - | seed |
| 8 | 环境隔离：所有外部依赖地址（数据库、缓存、API endpoint）必须通过环境变量或配置中心注入，禁止在代码中硬编码。不同环境（dev/staging/prod）的配置必须分离 | `General`, `DevOps` | [seed] | 2026-05-25 | - | seed |

### [stack: Vue3+ElementPlus] — Vue3 + Element Plus 技术栈

> 匹配标签: `Vue3`, `ElementPlus`

| # | 模式 | 技术栈标签 | 来源项目 | 推广日期 | 最近触发 | 状态 |
|---|------|-----------|---------|---------|----------|------|
| 1 | 组件响应式数据使用ref/reactive：确保UI更新依赖的数据用ref()或reactive()包裹。computed需有明确的getter且无副作用 | `Vue3`, `JavaScript` | [seed] | 2026-05-25 | - | seed |
| 2 | 组件卸载时清理副作用：在onUnmounted()中清除定时器、取消订阅、移除事件监听、断开WebSocket，防止内存泄漏 | `Vue3`, `JavaScript` | [seed] | 2026-05-25 | - | seed |
| 3 | 大列表虚拟滚动：超过200条数据的列表必须使用虚拟滚动（el-table-v2或vue-virtual-scroller），避免DOM节点过多导致性能问题 | `Vue3`, `ElementPlus` | [seed] | 2026-05-25 | - | seed |
| 4 | Element Plus按需导入：使用unplugin-vue-components或手动导入组件，避免全量引入Element Plus导致打包体积过大 | `Vue3`, `ElementPlus` | [seed] | 2026-05-25 | - | seed |
| 5 | 表单校验完整性：el-form的rules必须覆盖所有必填字段，包含类型校验、长度限制、格式校验。自定义validator需有async支持 | `Vue3`, `ElementPlus` | [seed] | 2026-05-25 | - | seed |

### [stack: SpringBoot3] — Spring Boot 3 技术栈

> 匹配标签: `SpringBoot3`, `Java`, `Maven`

| # | 模式 | 技术栈标签 | 来源项目 | 推广日期 | 最近触发 | 状态 |
|---|------|-----------|---------|---------|----------|------|
| 1 | 事务管理@Transactional：涉及多表写操作的Service方法必须加@Transactional(rollbackFor=Exception.class)，注意同类方法调用不触发代理的问题 | `Java`, `SpringBoot3` | [seed] | 2026-05-25 | - | seed |
| 2 | DTO校验：Controller接收的请求体DTO必须使用@Valid/@Validated + Jakarta Validation注解（@NotNull/@NotBlank/@Size等），禁止在Controller中手动校验<br>所有校验注解必须有message属性，提供用户友好的中文错误提示 | `Java`, `SpringBoot3` | [seed] | 2026-05-25 | - | seed |
| 3 | 全局异常处理：必须有@ControllerAdvice/@RestControllerAdvice全局异常处理器，覆盖业务异常、参数校验异常、未知异常。所有异常返回统一的ApiResult结构（含code/message字段） | `Java`, `SpringBoot3` | [seed] | 2026-05-25 | - | seed |
| 4 | API响应统一格式：所有REST API返回统一响应结构（如{code, message, data}），使用ResponseBodyAdvice或统一封装类。禁止每个Controller返回不同格式 | `Java`, `SpringBoot3` | [seed] | 2026-05-25 | - | seed |
| 5 | 分页查询规范：列表接口默认支持分页参数（page/size），返回格式含total/pageSize/currentPage/list。禁止不分页的全量查询（数据量可能增长） | `Java`, `SpringBoot3` | [seed] | 2026-05-25 | - | seed |

## 冲突标记

> 当两个模式声称同一作用域但给出相反建议时，在此记录。
> 冲突解决适用**项目级优先**原则：各项目可在自己的 multi-agent-flow.md「项目覆盖」区调整。

| 模式A | 模式B | 冲突描述 | 状态 |
|-------|-------|---------|------|
| - | - | - | - |

## 推广日志

| 日期 | 来源项目 | 模式编号 | 推广路径 | 冲突检测 |
|------|---------|---------|---------|----------|
| 2026-05-07 | paicoding-springboot3-vue3 | #1 | project → global | 无冲突（首条） |

---
*统计: 1条 global | 0条 Vue3+ElementPlus | 0条 SpringBoot3 | 0个冲突 | 上限: 20条*
*新项目匹配规则: [global] 全部继承 + 项目技术栈标签 ∩ [stack] 标签 → 自动继承*
