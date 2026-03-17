---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [demo, enterprise-ai, knowledge-assistant, saas, platform]
---

# Reference Enterprise AI Demo Project

## 目标
定义一个可销售、可演示、可扩展的企业 AI 应用示范项目，使其既能作为客户看得懂的业务产品，也能作为平台能力的参考实现。

## 适用场景
- 企业知识助手示范项目
- AI SaaS 平台参考实现
- 架构咨询、PoC 交付和平台共建的销售入口

## 核心判断
示范项目不应该只是一个“能聊天的 Demo”，而应该满足两个目标：
- 对外：像一个具体、有业务价值的企业 AI 应用
- 对内：承载平台能力的最小参考实现

因此，示范项目最合适的形态是：

```text
企业知识助手
+ 多租户
+ 权限与审计
+ RAG
+ Workflow
+ Usage / Billing
```

## 为什么选择“企业知识助手”作为示范项目
因为它同时具备：
- 企业容易理解的业务场景
- 明确的 AI 价值
- 足够承载平台能力的复杂度
- 可从简单 PoC 演化为平台化产品

它既好卖，又适合作为架构样板。

## 项目定位
建议对外定位为：

> 一个可落地、可治理、可扩展的企业知识助手参考实现。

建议对内定位为：

> 企业 AI 平台能力的最小运行实例。

## 目标用户

### 1. 业务用户
- 提问
- 检索企业知识
- 获取带引用来源的答案
- 触发简单任务或流程

### 2. 管理员 / 租户管理员
- 管理文档与知识库
- 管理用户与角色
- 查看 usage / billing / audit
- 配置部分平台能力

### 3. 平台运营者
- 管理租户
- 管理策略
- 查看平台级审计与 usage
- 验证工作流与能力模型

## 必须体现的核心平台能力
示范项目建议至少体现以下能力：

### 1. Knowledge + RAG
- 文档上传
- 文档切片 / 索引
- 引用来源回答

### 2. Multi-tenant
- 租户隔离
- 每个租户有自己的知识库与用户边界

### 3. Policy / Permission
- 不同角色看到不同数据
- 不同角色允许调用不同能力

### 4. Workflow
- 至少一个多步骤任务流程
- 支持任务状态查看

### 5. Audit
- 记录关键操作与高风险行为
- 支持按租户 / 用户 / 任务检索

### 6. Usage / Billing
- 记录基础 usage
- 至少展示租户级配额 / usage 面板

## 推荐产品形态
建议采用双端：

### Web 端
适合：
- 管理后台
- 文档管理
- 审计与 usage 面板
- Web 版知识助手

### MiniApp 端
适合：
- 移动端问答
- 轻任务触发
- 用户入口与业务前台

## MVP 功能范围

### A. 用户侧
- 登录
- 提问
- 查看回答与引用来源
- 查看最近历史

### B. 管理侧
- 上传文档
- 管理知识库
- 用户 / 角色管理
- usage / audit 基础面板

### C. 平台侧
- 租户切换或租户视图
- 基础策略配置
- workflow 状态查看

## 不必在第一版追求的内容
为了确保项目可销售、可交付，第一版建议不要追求：
- 通用 Agent 平台
- 复杂多 Agent 协作
- 过度通用的 Skill Graph 管理界面
- 全自动代码生成链路全部落地

第一版重点是：
- 把企业能理解的价值做出来
- 把平台能力的最小骨架跑起来

## 业务演示脚本建议
推荐最小演示链路：

1. 管理员创建租户
2. 上传企业文档
3. 系统完成索引
4. 普通用户在 Web / MiniApp 提问
5. 系统返回带引用的答案
6. 管理员查看 usage、audit 和基本 workflow 状态
7. 展示不同租户之间的数据隔离

这条链路可以直接支撑销售演示。

## 为什么这个项目适合销售
它同时满足：
- 看起来是一个真实应用
- 能展示“AI 真正能解决问题”
- 能展示“我们不只是做聊天机器人，而是做企业级系统”

也就是说：
- 客户买的是“知识助手”
- 你交付的是“平台能力骨架”

## 与现有知识库的关系
这个项目会串起知识库中的多个模块：
- `ai-application-os.md`
- `multi-tenant-ai-platform.md`
- `policy-engine-for-ai-saas.md`
- `workflow-engine-for-ai-agents.md`
- `billing-and-usage-for-ai-saas.md`
- `audit-log-for-ai-agents.md`
- `reference-tech-stack-for-enterprise-ai-demo.md`

## 后续可继续拆分的主题
- `demo-project-mvp-scope.md`
- `sales-demo-script-for-enterprise-ai-assistant.md`
- `poc-to-production-evolution.md`
- `enterprise-knowledge-assistant-domain-model.md`

## 相关文档
- `knowledge/architecture/reference-tech-stack-for-enterprise-ai-demo.md`
- `knowledge/architecture/ai-application-os.md`
- `knowledge/architecture/multi-tenant-ai-platform.md`
- `knowledge/architecture/policy-engine-for-ai-saas.md`
- `knowledge/architecture/billing-and-usage-for-ai-saas.md`
