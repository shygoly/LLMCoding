---
type: decision
status: accepted
owner: user
date: 2026-03-17
tags: [analytics, postgres, duckdb, parquet, architecture]
---

# 007-use-postgres-for-oltp-and-duckdb-parquet-for-analytics

## 决策
在示范项目与早期 AI SaaS 平台中，采用以下数据分层：
- `Postgres` 作为在线事务层（OLTP）
- `OSS / S3 + Parquet` 作为分析数据层
- `DuckDB` 作为轻量分析执行层

## 背景
当前平台需要同时承载：
- 多租户主数据
- workflow / task / policy / billing / audit 等事务性状态
- usage、audit、workflow event 等分析性数据

如果把分析查询长期压在主业务数据库上，会逐渐影响在线系统性能和成本控制。

## 备选方案
- 方案 A：全部使用 `Postgres`
- 方案 B：早期直接上重型数仓
- 方案 C：`Postgres + Parquet + DuckDB`

## 选择原因
方案 C 在当前阶段最平衡：
- 保留主业务层简单性
- 快速建立分析能力
- 不需要过早引入复杂数仓基础设施
- 便于后续从示范项目演化为平台

## 收益
- 在线事务层与分析层职责清晰
- 报表与趋势分析不直接拖累主库
- usage / billing / workflow / audit 更容易联合分析
- 后续升级空间大

## 代价与风险
- 需要维护事件导出与 Parquet 产出流程
- 分析链路比单数据库更复杂
- 必须清楚区分“系统真相”和“分析副本”

## 后续动作
- 设计 usage / audit / workflow event schema
- 设计 Parquet 分区策略
- 设计 DuckDB 报表和 dashboard 数据流

## 相关文档
- `knowledge/architecture/analytics-stack-for-ai-saas.md`
- `knowledge/architecture/billing-and-usage-for-ai-saas.md`
- `knowledge/architecture/audit-log-for-ai-agents.md`
