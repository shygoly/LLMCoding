---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [policy, ai-saas, security, multi-tenant, governance]
---

# Policy Engine For AI SaaS

## 目标
定义 AI SaaS 中 `Policy Engine` 的系统级职责，说明为什么企业级 AI 应用不能只靠 Prompt 控制行为，而必须引入独立的策略与权限层。

## 适用场景
- 多租户 AI SaaS
- Agent 平台
- 带有 Skill / Tool 调用能力的系统
- 涉及敏感数据、权限边界或审计要求的企业级应用

## 核心判断
在 AI SaaS 中，Prompt 可以约束 LLM“倾向于怎么做”，但不能作为真正可信的安全边界。

真正的安全与治理边界应由 `Policy Engine` 提供。

可以把它理解为：

```text
Prompt = 行为提示
Policy Engine = 可信约束
```

## 为什么必须有 Policy Engine
如果没有独立策略层，系统通常会出现以下问题：
- LLM 直接把高权限操作当成普通工具调用
- 不同租户之间的数据边界不清晰
- Prompt Injection 容易越过业务限制
- 很难对技能调用做审计和问责
- 系统无法稳定执行“谁能做什么、在什么条件下做”

这意味着：
- 小 demo 能跑
- 企业级系统不可控

## Policy Engine 的核心职责

### 1. 权限控制
决定：
- 谁可以调用哪些技能
- 哪些角色可以访问哪些数据
- 哪些任务路径允许执行

### 2. 数据隔离
决定：
- 租户边界
- 用户边界
- 资源访问范围
- 上下文可见性

### 3. 调用约束
决定：
- 调用频率
- 调用次数
- 调用配额
- 成本阈值

### 4. 风险控制
决定：
- 是否允许执行敏感操作
- 是否需要人工确认
- 是否应该阻断危险路径

### 5. Prompt Injection 防护
决定：
- LLM 输出的意图是否真的允许执行
- 即使 Planner 想调用某个 Skill，是否依然通过策略检查

## 在系统中的位置
推荐位置如下：

```text
User
-> Prompt Layer
-> Planner
-> DSL Plan
-> Policy Engine
-> Skill Registry / Executor
```

关键点：
- Planner 不直接调用 Skill
- Planner 先生成结构化意图或 DSL
- Policy Engine 对意图与技能调用做审查
- 通过后才进入执行层

## Policy Engine 审查什么
建议至少审查以下维度：

### 1. Actor
- 当前用户是谁
- 当前 Agent 代表谁执行
- 当前角色是什么

### 2. Intent
- 当前 DSL / 计划意图是什么
- 它是否属于允许范围

### 3. Resource
- 要访问什么数据、什么文档、什么技能、什么外部系统

### 4. Context
- 当前租户是谁
- 当前场景是什么
- 是否为高风险流程

### 5. Cost / Rate
- 当前调用成本是否超限
- 调用频率是否超限

## 推荐策略模型
第一版可按以下结构思考：

```text
allow(actor, action, resource, context) -> true/false
```

例如：

```text
doctor can read patient_record in tenant A
assistant cannot export all records
free_plan user cannot run expensive workflow
```

## 常见策略类型

### 1. Role-based Policy
按角色授权。

例如：
- `doctor`
- `admin`
- `analyst`
- `guest`

### 2. Tenant-based Policy
按租户隔离。

例如：
- 只能访问本租户知识库
- 不能跨租户调用受限资源

### 3. Workflow Policy
按任务路径控制。

例如：
- 可以查询患者资料
- 但“开药建议”必须经过额外审核

### 4. Cost Policy
按成本和配额控制。

例如：
- 免费版不可用昂贵模型
- 某类技能每日调用上限 100 次

### 5. Confirmation Policy
按风险要求人工确认。

例如：
- 删除数据
- 发送外部邮件
- 执行支付相关操作

## 为什么 Prompt 不能替代 Policy
Prompt 的问题在于：
- 它依赖模型遵守
- 它不天然可审计
- 它难以表达严格租户与权限边界
- 它在 Prompt Injection 场景下容易被绕过

而 Policy Engine：
- 是独立于 LLM 的执行前检查层
- 能输出明确的 allow / deny
- 能记录日志与审计
- 能把租户、角色、成本、风险统一治理

## 与 Skill Graph 的关系
当系统从 Tool Catalog 升级到 Skill Graph 时，Policy 也会升级：

- 不是只控制“能否调用某个 skill”
- 而是控制“能否走某条 skill path”

例如：
- 可以 `search_doc`
- 但不能 `search_doc -> export_all`

这意味着：
- Policy 应逐步从节点权限走向路径权限

## 推荐技术实现
在当前技术偏好下，第一版可考虑：
- `OPA` 作为策略判断引擎
- `Postgres` 保存角色、租户、计划与审计日志
- 在 DSL / Skill 调用前统一接入策略检查

## 平台化视角
在 AI SaaS 中，`Policy Engine` 最好作为独立能力层，而不是散落在每个 Skill 里。

这样做的好处：
- 策略统一
- 审计统一
- 变更成本低
- 更适合多租户治理

## 常见错误信号
以下现象通常说明缺少真正的 Policy Engine：
- 系统靠 Prompt 说“不要这么做”
- 每个 Skill 自己手写权限逻辑
- 同一类权限在不同地方表现不一致
- 租户隔离靠前端参数传递保证
- 风险操作没有人工确认环节

## 与现有知识库的关系
这篇文档属于平台级蓝图，向下适合连接：
- `patterns/`：技能注册、执行路径、安全检查模式
- `methods/`：如何逐步落地 policy 层
- `decisions/`：是否正式引入 OPA、路径级策略等

## 后续可继续拆分的主题
- `tenant-isolation-for-ai-saas.md`
- `policy-bound-skill-paths.md`
- `confirmation-gates-for-high-risk-actions.md`
- `opa-integration-plan.md`

## 相关文档
- `knowledge/architecture/ai-application-os.md`
- `knowledge/architecture/skill-graph-vs-tool-catalog.md`
- `knowledge/architecture/memory-vs-rag.md`
- `knowledge/principles/shared-state-over-session-memory.md`
