---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [multi-tenant, ai-saas, platform, isolation, governance]
---

# Multi Tenant AI Platform

## 目标
定义 AI SaaS 中多租户平台的系统级架构边界，说明为什么多租户不是简单的“多一个 tenant_id 字段”，而是贯穿身份、数据、技能、Memory、RAG、Workflow 与 Policy 的平台级设计问题。

## 适用场景
- AI SaaS 平台
- 企业级 Agent 平台
- 需要隔离不同客户、组织、团队或业务单元的数据与能力的系统

## 核心判断
多租户 AI 平台的难点，不只是“让多个客户共用一套系统”，而是：

- 谁能看见什么
- 谁能调用什么
- 谁的 Memory 属于谁
- 谁的 RAG 只能检索谁的知识库
- 哪些 workflow 可以在谁的边界内运行
- 成本、日志、审计如何按租户归集

因此，多租户是 AI SaaS 的根架构问题，而不是附加字段问题。

## 多租户影响哪些层
推荐把租户边界看作横切能力，贯穿以下层：

- 身份与认证
- Policy Engine
- Skill Registry / Skill Graph
- Memory Layer
- RAG Layer
- Workflow Engine
- Billing / Usage
- Logging / Audit

## 核心边界

### 1. 身份边界
需要明确：
- 用户属于哪个租户
- Agent 代表哪个租户运行
- 某次任务上下文属于哪个租户

如果这层不清楚，后续所有隔离都会变脆弱。

### 2. 数据边界
需要明确：
- 结构化业务数据是否按租户隔离
- 日志与工件是否按租户隔离
- `runtime/state.db` 这类运行时状态是否携带租户上下文

### 3. 知识边界
需要明确：
- RAG 检索只能访问本租户知识库，还是允许访问共享知识库
- 哪些知识属于公共层，哪些属于租户私有层

### 4. 能力边界
需要明确：
- 哪些 skill 是平台级公共能力
- 哪些 skill 是租户私有能力
- 哪些 workflow 可被特定租户启用

### 5. 成本边界
需要明确：
- 模型调用成本如何归集
- 工作流成本如何按租户计费
- 配额如何按租户控制

## 推荐分层模型
建议按三层看待租户能力：

```text
Global Layer
Tenant Layer
User Layer
```

### Global Layer
平台公共能力：
- 公共基础模型
- 公共 workflow 模板
- 公共系统文档
- 平台共享 policy 模板

### Tenant Layer
租户专属能力：
- 租户知识库
- 租户技能配置
- 租户 workflow 配置
- 租户 Memory 与 usage 数据

### User Layer
用户个体能力：
- 用户偏好
- 用户历史任务
- 用户个人上下文

## 为什么 AI 场景比传统 SaaS 更难
传统 SaaS 更多处理：
- 表记录归属
- 页面权限
- API 权限

AI SaaS 还要处理：
- Prompt 上下文归属
- RAG 检索边界
- Agent 代用户执行时的权限继承
- workflow 中跨步骤的租户约束延续
- 第三方模型或外部 API 的数据外发边界

这使得多租户设计必须上升到系统架构层。

## 在各层的推荐实践

### Identity
- 每次请求必须显式带租户上下文
- Agent 执行身份不能脱离租户身份

### Policy Engine
- 所有 allow / deny 判断必须包含 tenant context
- Skill path 权限最好支持租户级策略

### Memory
- User Memory 与 Agent Memory 应带租户边界
- 长期 Memory 不应跨租户混用

### RAG
- 检索索引需支持租户过滤
- 公共知识与私有知识要分层管理

### Workflow
- Workflow 实例必须归属于明确租户
- 中途恢复时必须恢复租户上下文

### Logging / Audit
- 日志和审计记录应支持按租户检索
- 高风险行为应能追溯到租户、用户、任务三层

## 推荐数据隔离策略
第一版通常有三种模式：

### 1. Shared DB + tenant_id
优点：
- 成本低
- 开发快

缺点：
- 最依赖应用层策略正确性
- 容易因为查询遗漏 tenant filter 而泄漏

### 2. Shared DB + strict row-level policy
优点：
- 比单纯 `tenant_id` 更安全
- 更适合企业级控制

缺点：
- 复杂度增加

### 3. Tenant-isolated storage
优点：
- 隔离最强
- 更适合高敏感行业

缺点：
- 成本和运维复杂度更高

## 推荐起步方案
如果是 MVP：
- 可以从 `tenant_id + strict policy checks` 起步
- 但必须把“租户上下文是强制输入”写进架构，而不是靠约定

如果是企业高敏感场景：
- 应更早考虑存储与索引层隔离

## 常见错误信号
以下现象通常说明多租户设计有问题：
- RAG 检索结果混入其他租户资料
- Agent 恢复任务时丢失租户身份
- Skill 调用只检查用户角色，不检查租户上下文
- usage / billing 不能按租户归因
- 审计日志无法回答“这次高风险操作属于哪个租户”

## 与现有知识库的关系
这篇文档属于平台级蓝图，适合向下连接：
- `policy-engine-for-ai-saas.md`
- `memory-vs-rag.md`
- `workflow-engine-for-ai-agents.md`
- 后续的 tenant-isolation、billing、audit 等专题

## 后续可继续拆分的主题
- `tenant-isolation-for-rag.md`
- `tenant-context-propagation.md`
- `billing-and-usage-for-ai-saas.md`
- `audit-log-for-ai-agents.md`

## 相关文档
- `knowledge/architecture/ai-application-os.md`
- `knowledge/architecture/policy-engine-for-ai-saas.md`
- `knowledge/architecture/memory-vs-rag.md`
- `knowledge/architecture/workflow-engine-for-ai-agents.md`
