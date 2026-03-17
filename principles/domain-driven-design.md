---
type: principle
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [ddd, domain, modeling, architecture, design]
---

# Domain Driven Design

## 原则
在复杂系统中，优先围绕领域模型、边界上下文和统一语言组织系统，而不是围绕技术框架、数据库表或基础设施细节组织核心结构。

## 背景
当系统开始包含多个核心能力域，例如：
- Skill / Skill Graph
- Policy
- Workflow
- Billing
- Audit
- Memory / RAG

如果只按技术层划分代码和职责，系统很快会出现：
- 概念混乱
- 模块边界模糊
- 同一业务规则散落在多个技术组件中
- 团队沟通中“同词不同义”

DDD 的价值在于：先定义业务世界，再决定代码结构。

## 适用场景
- AI SaaS 平台
- 多租户平台
- Agent 平台
- 有多个核心业务能力域的系统
- 需要长期演化的复杂软件系统

## 核心要求
1. 先识别领域，再做技术实现
2. 用统一语言描述业务概念
3. 用边界上下文隔离不同子域
4. 领域模型优先于数据库模型与接口细节
5. 技术实现应服务于领域边界，而不是反过来主导领域边界

## 在当前系统中的含义
针对当前知识体系，DDD 可以帮助明确：
- `Policy` 是独立域，不只是中间件
- `Workflow` 是独立域，不只是任务脚本
- `Billing / Usage` 是独立域，不只是报表
- `Audit` 是独立域，不只是日志
- `Skill Graph` 是平台能力域，不只是工具列表

## 价值
- 提高系统可演化性
- 降低跨模块耦合
- 提高团队沟通效率
- 更适合把平台能力沉淀为长期资产

## 代价与边界
- 初期建模成本更高
- 对小型、短生命周期项目可能显得过重
- 如果团队没有统一建模习惯，DDD 很容易沦为空话

## 推荐做法
- 在平台级主题先识别核心域、支撑域、通用域
- 在文档中优先定义术语和边界
- 再决定代码仓库、模块和服务拆分方式

## 相关文档
- `knowledge/architecture/ai-application-os.md`
- `knowledge/architecture/multi-tenant-ai-platform.md`
- `knowledge/architecture/policy-engine-for-ai-saas.md`
