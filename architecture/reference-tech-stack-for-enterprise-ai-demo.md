---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [tech-stack, demo, nextjs, taro, enterprise-ai, saas]
---

# Reference Tech Stack For Enterprise AI Demo

## 目标
定义企业 AI 应用示范项目的参考技术栈，使示范项目既能快速落地，又能承载后续从单一应用演化为 AI SaaS 平台的需求。

## 适用场景
- 企业知识助手示范项目
- AI SaaS 平台参考实现
- Web + 微信小程序双端项目
- 需要兼顾快速交付与后续平台化扩展的团队

## 核心判断
示范项目的技术栈不应追求“理论最完美”，而应优先满足：
- 开发效率高
- 类型安全强
- 演示效果好
- 适合 SaaS 后台 + 小程序前台
- 能支撑后续接入 Policy / Workflow / Billing / Audit / RAG

因此，参考栈建议拆成四层：
- `Web / Admin Layer`
- `MiniApp Layer`
- `Core Data Layer`
- `AI / Platform Integration Layer`

## 推荐技术栈

### 1. Web / Admin Layer
适用于：
- 企业控制台
- 租户管理
- 审计面板
- workflow / usage / billing dashboard
- Web 版知识助手

推荐：
- `Next.js`
- `TypeScript`
- `Tailwind CSS`
- `shadcn/ui`
- `Radix UI`
- `react-hook-form`
- `zod`
- `zustand`
- `tanstack-query`
- `tRPC`

## 2. MiniApp Layer
适用于：
- 微信小程序用户入口
- 企业轻交互场景
- 移动端知识问答与任务触发

推荐：
- `Taro`
- `TypeScript`
- `miniprogram-cli`

说明：
- `Taro` 适合与统一 Schema、共享类型和 API SDK 协同
- `miniprogram-cli` 是小程序构建、预览、上传和交付链的重要组成部分

## 3. Core Data Layer
适用于：
- 主业务数据
- 状态管理
- 租户 / 用户 / workflow / billing / audit 主数据

推荐：
- `Postgres`
- `Drizzle ORM`

扩展：
- `pgvector` 用于向量检索和 RAG 支撑

## 4. Auth Layer
推荐：
- `Auth.js` 或 `Clerk`

建议：
- 若追求快速 demo，可优先考虑 `Clerk`
- 若更强调长期可控性和平台边界，可优先考虑 `Auth.js`

## 5. AI / Platform Integration Layer
推荐：
- `Vercel AI SDK`

适用：
- Chat UI
- streaming
- tool calling 接入
- 多模型快速接入

说明：
- 可作为示范项目的 AI 接入层
- 但平台核心边界仍应保留独立的 Policy / Workflow / Skill / Audit 设计

## 6. Storage Layer
推荐：
- `S3` 或 `Supabase Storage`

适用：
- 文档上传
- 图片资源
- 导出报告
- 附件存储

## 7. Analytics Layer
推荐：
- `DuckDB`
- `OSS / S3 + Parquet`

说明：
- 适合作为 usage / billing / audit / workflow analytics 的分析层
- 与 `Postgres` 主事务层分工明确

## 推荐项目结构
建议按 monorepo 方式组织：

```text
apps/
  web/
  miniapp/

packages/
  shared-types/
  api-sdk/
  ui-schema/
  visual-spec/
  generator/
  task-protocol/
```

当平台能力增加时，可继续扩为：

```text
apps/
  web/
  miniapp/
  supervisor/

packages/
  shared-types/
  api-sdk/
  task-protocol/
  runtime-schema/
  skill-registry/
```

## 为什么这套栈适合当前阶段

### 1. 适合示范项目快速交付
UI 层和全栈效率高，适合做客户演示和 PoC。

### 2. 适合沉淀平台骨架
数据库、认证、AI 接入、存储、分析层都能顺滑升级。

### 3. 适合双端场景
Web 与 MiniApp 可以共享类型、Schema、SDK 和部分平台能力。

## 与现有方法论的关系
这套技术栈与现有知识库高度兼容：
- 与 `UI Schema / Visual Spec / Reconcile` 兼容
- 与 `Supervisor / cursor-agent / Workflow` 路线兼容
- 与 `Billing / Audit / Policy / Multi-tenant` 平台蓝图兼容

## 常见边界提醒
- `zustand` 适合 UI 状态，不应作为系统真相存储
- `tRPC` 适合前期加速，但平台化后可逐步引入更明确的服务边界
- `Vercel AI SDK` 适合接入层，不应替代平台内核抽象
- `Postgres` 扛主业务，`DuckDB + Parquet` 扛分析

## 后续可继续拆分的主题
- `monorepo-structure-for-enterprise-ai-demo.md`
- `web-vs-miniapp-capability-boundary.md`
- `auth-stack-choice-for-ai-saas.md`
- `api-boundary-after-trpc.md`

## 相关文档
- `knowledge/architecture/ai-application-os.md`
- `knowledge/architecture/multi-tenant-ai-platform.md`
- `knowledge/architecture/analytics-stack-for-ai-saas.md`
- `knowledge/patterns/ui-schema-spec.md`
- `knowledge/patterns/visual-spec-spec.md`
