---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [audit, agents, traceability, governance, ai-saas]
---

# Audit Log For AI Agents

## 目标
定义 AI Agent 平台中的审计日志架构，说明为什么在多租户、Policy、Workflow、Billing、Skill 执行和 Memory / RAG 协同的系统里，审计日志不是可选增强，而是平台可信性的基础设施。

## 适用场景
- AI SaaS 平台
- 多租户 Agent 平台
- 带有高风险 Skill / Workflow 的系统
- 需要合规、追责、复盘、争议处理的企业级应用

## 核心判断
当 AI 系统开始具备：
- 自动执行能力
- 工具调用能力
- 长任务能力
- 多租户能力
- 高成本模型与高风险操作

系统就必须能回答：
- 谁触发了什么
- 在什么上下文下触发
- 做了哪些步骤
- 为什么允许执行
- 造成了什么结果
- 成本和责任归到谁

这就是 `Audit Log` 的职责。

## 为什么审计日志在 AI 系统里更重要
传统系统通常记录：
- 用户登录
- API 调用
- 数据修改

AI 系统额外需要记录：
- Planner 生成了什么意图
- Policy 为什么放行或拒绝
- 调用了哪些 Skill 路径
- Workflow 走了哪些分支
- 用了哪些 Memory / RAG 上下文
- 为什么触发了高成本模型或高风险动作

也就是说，AI 系统的审计对象更接近“决策过程”，而不只是“接口访问”。

## Audit Log 要解决的问题
系统至少要能追溯：
- `actor`：谁发起的
- `tenant`：属于哪个租户
- `task / workflow`：属于哪条任务链
- `decision`：经过了哪些关键判断
- `execution`：执行了哪些技能和操作
- `result`：产出了什么结果
- `cost`：花了多少资源

## 推荐分层
建议将审计分成三类：

### 1. Business Audit
记录：
- 谁请求了什么业务动作
- 最终业务结果是什么

### 2. Agent Audit
记录：
- Planner 输出了什么意图
- Policy 如何决策
- 走了哪条 skill path
- 哪一步失败、重试、被阻断

### 3. Infrastructure Audit
记录：
- 模型调用
- 外部 API 调用
- workflow 状态变化
- 成本与 usage 事件

## 在系统中的关键节点
建议至少在以下节点记录审计事件：

### 1. Request Ingress
记录：
- tenant
- user
- session
- request id
- 原始目标摘要

### 2. Planner Output
记录：
- 生成的结构化计划或 DSL 摘要
- 计划版本
- 关键推理结果摘要

### 3. Policy Decision
记录：
- allow / deny
- 策略命中原因
- 所用角色 / 租户上下文
- 是否需要人工确认

### 4. Skill / Tool Execution
记录：
- 执行了哪个 skill
- 参数摘要
- 成功 / 失败
- 耗时
- 外部系统调用摘要

### 5. Workflow Transition
记录：
- 从哪个状态转到哪个状态
- 重试 / 超时 / 人工介入
- 当前 step / task / workflow id

### 6. Output / Side Effect
记录：
- 最终输出
- 是否发邮件、写数据库、导出文件、触发支付等
- 影响范围摘要

### 7. Billing / Usage Attribution
记录：
- token
- 模型
- skill 成本
- 归属租户和任务

## 推荐最小事件结构
建议每条审计事件至少包含：

```json
{
  "eventId": "evt-001",
  "timestamp": "2026-03-17T10:00:00Z",
  "tenantId": "tenant-a",
  "userId": "user-123",
  "actorType": "user",
  "workflowId": "wf-01",
  "taskId": "task-07",
  "eventType": "policy_decision",
  "status": "allowed",
  "summary": "doctor role allowed to run clinical evaluation",
  "meta": {}
}
```

推荐字段：
- `eventId`
- `timestamp`
- `tenantId`
- `userId`
- `actorType`
- `workflowId`
- `taskId`
- `eventType`
- `status`
- `summary`
- `meta`

## 审计日志不应记录什么
不是所有数据都应该原样写入审计：
- 敏感原始内容不应无差别落日志
- 长 prompt / 长上下文不应完整持久化到审计层
- 高风险个人隐私数据需要脱敏或摘要化

推荐做法：
- 记录摘要、引用和关键字段
- 敏感内容单独走受控存储

## 与 Policy 的关系
Policy 决定：
- 能不能做

Audit 记录：
- 为什么能做 / 为什么不能做

没有 Audit 的 Policy，难以复盘；
没有 Policy 的 Audit，只能事后知道出问题了。

## 与 Workflow 的关系
Workflow 负责把任务跑完，Audit 负责把轨迹记住。

建议：
- 每次状态迁移都产生日志事件
- 每次人工介入都产生日志事件
- 每次 `needs_fix` / `blocked` / `retry` 都产生日志事件

## 与 Billing 的关系
Billing 解决“算给谁”，Audit 解决“为什么算给他”。

如果没有 Audit：
- 成本争议很难处理
- 无法解释高成本行为的来源
- 无法回答某次 overage 是哪条工作流导致的

## 多租户视角
多租户系统中，Audit 至少必须支持：
- 按 tenant 检索
- 按 user 检索
- 按 workflow / task 检索
- 按高风险行为筛选
- 按时间范围回放

## 常见错误信号
以下现象通常说明缺少合格的 Audit 层：
- 只能看到“报错了”，看不到之前发生了什么
- 不能追溯某次 skill 调用是谁触发的
- 高风险行为无法回放执行路径
- 计费争议时无法提供证据链
- Prompt Injection 后无法知道是哪一步越界

## 推荐技术实现
第一版可考虑：
- `Postgres` 或独立事件表保存审计记录
- workflow / skill / policy / usage 统一写审计事件
- 用结构化 event schema，而不是纯文本日志
- 将敏感信息脱敏后写入审计层

## 与现有知识库的关系
这篇文档属于平台级蓝图，适合向下连接：
- `multi-tenant-ai-platform.md`
- `policy-engine-for-ai-saas.md`
- `workflow-engine-for-ai-agents.md`
- `billing-and-usage-for-ai-saas.md`
- 后续的 audit schema、event taxonomy 等专题

## 后续可继续拆分的主题
- `audit-event-schema.md`
- `agent-event-taxonomy.md`
- `sensitive-data-redaction-in-audit.md`
- `workflow-replay-and-audit.md`

## 相关文档
- `knowledge/architecture/multi-tenant-ai-platform.md`
- `knowledge/architecture/policy-engine-for-ai-saas.md`
- `knowledge/architecture/workflow-engine-for-ai-agents.md`
- `knowledge/architecture/billing-and-usage-for-ai-saas.md`
