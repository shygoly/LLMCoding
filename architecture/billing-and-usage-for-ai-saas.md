---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [billing, usage, quota, cost, ai-saas]
---

# Billing And Usage For AI SaaS

## 目标
定义 AI SaaS 中计费、配额和 usage 治理的系统级架构，说明为什么 AI 平台的商业化能力不能只在最后“接一个支付”，而必须从模型调用、技能执行、工作流运行到租户边界都进行统一设计。

## 适用场景
- AI SaaS 平台
- 多租户 Agent 平台
- 使用多模型、多 Skill、多工作流的企业级系统
- 需要按租户、按用户、按任务或按能力计费的系统

## 核心判断
AI SaaS 的计费问题，不只是“收多少钱”，而是：

- 什么行为被计量
- 成本归到谁身上
- 何时阻断超额调用
- 何时降级服务
- 如何把模型成本、技能成本、工作流成本统一到一个治理体系里

因此：

```text
Billing = 商业规则
Usage = 技术事实
Quota = 执行边界
```

三者必须联动设计。

## 为什么 AI SaaS 的计费比传统 SaaS 更复杂
传统 SaaS 常按：
- 席位数
- 套餐等级
- 存储容量
- API 调用次数

AI SaaS 还要处理：
- LLM token 成本
- embedding 成本
- 向量检索成本
- Skill 执行成本
- workflow 长任务成本
- 多模型差异化成本
- 重试与失败重跑带来的额外成本

也就是说，AI SaaS 的成本结构更像“算力 + 编排 + 外部服务”复合账单。

## 核心问题
系统必须能回答：
- 某次请求花了多少成本
- 这笔成本属于哪个租户、哪个用户、哪个任务
- 当前租户还有多少额度
- 哪些行为超出套餐
- 某次长任务为什么被降级或阻断

## 推荐分层
建议把这部分拆成三层：

### 1. Metering
负责记录事实：
- token 用量
- 模型调用次数
- workflow 执行时长
- skill 调用次数
- 检索次数
- 外部 API 消耗

### 2. Quota / Limit
负责执行边界：
- 每日 / 每月额度
- 并发限制
- 单次任务成本上限
- 模型等级可用性

### 3. Billing Rules
负责商业规则：
- 免费版 / 专业版 / 企业版
- 按席位计费还是按 usage 计费
- 超额后的处理方式
- 是否允许 overage

## 推荐计量对象
AI SaaS 中至少应计量以下对象：

### 模型相关
- prompt tokens
- completion tokens
- embedding tokens
- 模型调用次数
- 模型种类

### Skill 相关
- skill 调用次数
- skill 失败次数
- skill 平均耗时
- skill 外部 API 成本

### Workflow 相关
- workflow 启动次数
- workflow 步骤数
- workflow 总耗时
- workflow 重试次数

### 租户与用户相关
- tenant usage
- user usage
- feature usage
- 并发占用

## 推荐归因模型
推荐每条 usage 记录至少带以下维度：
- `tenant_id`
- `user_id`（可空）
- `session_id`（可空）
- `task_id` / `workflow_id`
- `skill_id`（可空）
- `model_id`（可空）
- `usage_type`
- `quantity`
- `cost_estimate`
- `timestamp`

这样做的好处：
- 可按租户汇总
- 可按任务追溯
- 可按能力做成本分析
- 可支撑审计与计费争议处理

## 配额层应该控制什么
Quota 不只是“次数限制”，建议至少控制：

### 1. 请求量
- 每分钟请求数
- 每日请求数

### 2. 成本量
- 每日 token 上限
- 每月成本预算
- 单任务最大成本

### 3. 能力等级
- 免费版不能使用高价模型
- 某些租户不能调用特定 workflow
- 高风险 skill 仅企业版可用

### 4. 并发与时长
- 同时运行的 workflow 数
- 单任务最大运行时长

## 超额后的常见策略
建议至少支持四类：

### 1. Hard Stop
超限即拒绝。

适合：
- 免费版
- 高成本模型
- 风险较高场景

### 2. Soft Warning
先提醒，再继续执行。

适合：
- 企业内部环境
- 已签约客户

### 3. Downgrade
自动降级到便宜模型或更低能力路径。

适合：
- 保证服务可用性
- 控制边际成本

### 4. Overage Billing
允许超额并额外计费。

适合：
- 商业成熟平台
- 企业客户

## 与 Policy Engine 的关系
Billing / Usage 不应和 Policy 混为一谈，但两者必须联动：

- Policy 判断：有没有资格做
- Quota 判断：还有没有额度做
- Billing 记录：做了之后算到谁头上

因此执行前后至少有两次关键检查：

```text
Before execution: policy + quota
After execution: metering + billing attribution
```

## 与 Workflow Engine 的关系
长任务场景下，Billing 更复杂：
- workflow 中间步骤失败是否计费
- retry 是否重复计费
- 人工介入等待是否算资源占用
- 部分完成如何归因

因此：
- Workflow 级 usage 必须单独建模
- 不能只看单次请求 token

## 与多租户的关系
在多租户 AI SaaS 中，Billing / Usage 是租户边界的重要组成部分：
- 所有 usage 必须可按 tenant 归因
- 套餐和 quota 通常是 tenant 级配置
- user 级限制是在 tenant 之下的第二层约束

## 推荐技术实现
第一版可考虑：
- `Postgres` 保存 usage ledger 与 plan 配置
- workflow / skill 执行结束时写 usage event
- 在 Policy / Quota 层之前做快速额度检查
- 在异步汇总层做成本聚合与账单生成

## 推荐最小实体
后续可演化为以下对象：
- `plans`
- `tenant_subscriptions`
- `usage_events`
- `quotas`
- `billing_invoices`
- `cost_rules`

## 常见错误信号
以下现象通常说明 Billing / Usage 架构有问题：
- 只能统计 token，不能统计 workflow 成本
- 无法回答某次高成本任务是谁触发的
- 免费用户可以无限触发高成本模型
- 重试导致成本翻倍，却无法解释
- usage 统计和实际执行脱节

## 与现有知识库的关系
这篇文档属于平台级蓝图，适合向下连接：
- `multi-tenant-ai-platform.md`
- `policy-engine-for-ai-saas.md`
- `workflow-engine-for-ai-agents.md`
- 后续的 metering、quota、invoice 等专题

## 后续可继续拆分的主题
- `usage-ledger-schema.md`
- `quota-enforcement-pattern.md`
- `model-tiering-strategy.md`
- `workflow-cost-attribution.md`

## 相关文档
- `knowledge/architecture/multi-tenant-ai-platform.md`
- `knowledge/architecture/policy-engine-for-ai-saas.md`
- `knowledge/architecture/workflow-engine-for-ai-agents.md`
- `knowledge/architecture/ai-application-os.md`
