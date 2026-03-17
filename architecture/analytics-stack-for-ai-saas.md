---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [analytics, duckdb, parquet, oltp, ai-saas]
---

# Analytics Stack For AI SaaS

## 目标
定义 AI SaaS 中分析栈的系统级分层，说明为什么在线事务层与分析层应明确分离，以及为什么 `Postgres + DuckDB + OSS Parquet` 是一个适合示范项目和早期 SaaS 的组合。

## 适用场景
- AI SaaS 平台
- 多租户 Agent 平台
- 需要 usage / billing / audit / workflow analytics 的系统
- 需要在不引入重型数仓前提下建立分析能力的团队

## 核心判断
在 AI SaaS 中，主业务数据库与分析查询承载的是两类不同负载：

- `OLTP`：交易、状态、配置、运行时真相
- `OLAP-lite`：聚合、趋势、报表、成本分析、审计分析

如果全部压在主库上，随着 usage、audit、workflow event 增长，系统很容易在性能、成本和治理上变得脆弱。

因此推荐：

```text
Postgres = 主事务层
OSS / S3 + Parquet = 分析数据层
DuckDB = 分析执行层
```

## 推荐分层

### 1. OLTP Layer
负责：
- 用户、租户、角色
- 任务、workflow、policy、skill 配置
- billing 主数据
- audit 主记录
- runtime 状态与当前系统真相

推荐：
- `Postgres`

### 2. Event / Data Lake Layer
负责：
- usage events
- audit events
- workflow events
- model call events
- 其他追加型事实数据

推荐：
- `OSS / S3`
- 文件格式：`Parquet`

### 3. Analytics Layer
负责：
- 趋势统计
- usage 聚合
- billing 分析
- audit 分析
- 租户行为分析
- workflow 成本分析

推荐：
- `DuckDB`

## 为什么 `DuckDB + Parquet` 适合当前阶段

### 1. 成本低
无需一开始就引入重型数仓。

### 2. 架构简单
对象存储 + Parquet + DuckDB 就可以支撑大量离线分析需求。

### 3. 适合事件型数据
AI SaaS 的很多关键数据天然是追加型：
- usage event
- workflow event
- audit event

### 4. 易于演进
后续若要升级到更复杂的数据湖或数仓，Parquet 资产依然可复用。

## 推荐数据流
建议采用类似流程：

```text
App / API / Workflow
  -> Postgres 记录主状态
  -> 产生 usage / audit / workflow events
  -> 定期或实时导出为 Parquet
  -> 存入 OSS / S3
  -> DuckDB 执行分析
  -> 提供 dashboard / reports
```

## 适合进入分析层的数据
建议优先纳入：
- `usage_events`
- `audit_events`
- `workflow_events`
- `model_call_events`
- `billing_usage_snapshots`

## 不建议直接交给分析层的数据
以下更适合长期保留在 OLTP：
- 当前用户状态
- 当前 workflow 活跃状态
- 当前租户配置
- 当前 quota / subscription 配置

原因：
- 这些是系统真相，不是离线分析快照

## 推荐分析主题
第一批最值得做的报表通常是：

### 1. Usage Analytics
- 每租户 token 趋势
- 每用户调用量趋势
- 高峰时段分析

### 2. Billing Analytics
- 哪些功能最耗成本
- 哪些租户最接近 quota
- 哪些工作流产生最多 overage

### 3. Workflow Analytics
- 哪些 workflow 平均时长最长
- 哪些 step 失败率最高
- 哪些重试最耗资源

### 4. Audit Analytics
- 高频高风险操作
- 高频被拒绝策略
- 异常行为趋势

## 与多租户的关系
分析层必须继续保留租户边界：
- Parquet 数据应可按 tenant partition 或 tenant filter 分析
- DuckDB 查询应支持租户过滤
- billing / audit 结果必须可归因到 tenant

## 与 Billing / Audit / Workflow 的关系
这层是这些能力的分析底座：
- `Billing`：看成本与使用趋势
- `Audit`：看高风险行为和异常模式
- `Workflow`：看运行效率和失败模式

## 推荐实现策略

### MVP
- 主业务仍以 `Postgres` 为中心
- 每日或每小时导出事件为 Parquet
- 用 `DuckDB` 跑后台分析任务
- 把聚合结果回写 Postgres 或直接供 dashboard 查询

### 升级版
- 更细粒度的事件分区
- 自动化分析管道
- 更复杂的报表与指标层

## 常见错误信号
以下现象通常说明分析栈边界不清：
- 所有 usage / audit 聚合都直接打主库
- 报表越多，主系统越慢
- 无法分析历史趋势，只能看当前状态
- usage 数据无法与 workflow、audit 做联合分析

## 与现有知识库的关系
这篇文档属于平台级蓝图，适合向下连接：
- `billing-and-usage-for-ai-saas.md`
- `audit-log-for-ai-agents.md`
- `workflow-engine-for-ai-agents.md`
- 后续的 usage ledger、事件 schema、报表方法等专题

## 后续可继续拆分的主题
- `usage-event-schema.md`
- `workflow-cost-attribution.md`
- `duckdb-reporting-pipeline.md`
- `tenant-partitioning-in-parquet.md`

## 相关文档
- `knowledge/architecture/billing-and-usage-for-ai-saas.md`
- `knowledge/architecture/audit-log-for-ai-agents.md`
- `knowledge/architecture/workflow-engine-for-ai-agents.md`
- `knowledge/architecture/multi-tenant-ai-platform.md`
