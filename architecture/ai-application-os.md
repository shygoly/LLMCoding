---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [saas, ai-os, architecture, planner, skill, memory, rag]
---

# AI Application OS

## 目标
定义一套可用于构建 AI SaaS 的系统级分层架构，使 AI 应用不再只是“Prompt + LLM”，而是演化为可扩展、可治理、可审计的 AI Operating System。

## 适用场景
- AI SaaS 平台
- 多租户 AI 应用
- Agent 平台
- 需要 Skill、Memory、Workflow、Policy、RAG 协同的企业级系统

## 核心判断
成熟的 AI 应用通常不是单一 LLM 调用，而是一个分层系统：

```text
User
-> Prompt Layer
-> Planner
-> DSL Plan
-> Policy Engine
-> Skill Registry
-> Skill Executor
-> Memory Layer
-> RAG
```

在更高级的系统中，还会继续加入：
- `Event Bus`
- `Workflow Engine`

## 分层说明

### 1. Prompt Layer
职责：
- 定义角色
- 约束输出
- 限制行为边界

说明：
- Prompt 适合做约束，不适合承载完整业务逻辑
- Prompt 是 LLM 行为的第一道边界，不是系统可信执行层

### 2. Planner
职责：
- 理解用户意图
- 拆解任务
- 生成步骤计划

说明：
- 这一层通常由 LLM 承担推理
- 它负责“想做什么”，不应直接负责“怎么执行底层系统操作”

### 3. DSL Plan
职责：
- 把 Planner 的自然语言推理结果收敛为结构化意图
- 作为安全边界，阻断 LLM 直接操作系统

说明：
- DSL 使系统更可验证、更可审计、更适合接 Policy Engine
- 它是 Agent 系统中的关键控制点之一

### 4. Policy Engine
职责：
- 权限控制
- 安全规则
- 调用约束
- 数据隔离
- 注入攻击防护

说明：
- 这一层相当于 AI 系统的 IAM
- 企业级场景必须有，不应只靠 Prompt 约束

### 5. Skill Registry
职责：
- 管理技能目录
- 提供工具元数据、参数 schema、权限、速率限制

说明：
- Skill Registry 是 AI 可调用能力的控制面
- 它比“随手列一堆 tools”更适合平台化管理

### 6. Skill Executor
职责：
- 执行真实 API / 系统调用
- 处理重试、超时、日志、错误

说明：
- Executor 是执行面，不应承担高层规划职责
- 应与 Skill Registry 解耦

### 7. Memory Layer
职责：
- 管理用户记忆
- 管理 Agent 记忆
- 管理长期知识或行为历史

说明：
- 没有 Memory 的 AI 应用难以持续进化
- 这一层通常由结构化存储与向量存储共同组成

### 8. RAG
职责：
- 检索外部知识
- 把相关上下文补给 Planner / LLM

说明：
- RAG 提供知识增强
- 但 RAG 不等于 Memory，两者职责不同

### 9. Event Bus
职责：
- 支持异步事件流
- 连接多个 Agent 或 workflow 节点

说明：
- 适用于长任务、异步任务、多阶段处理

### 10. Workflow Engine
职责：
- 管理长任务和多步骤任务
- 支持重试、等待、回调、人工介入

说明：
- 当 Agent 从单步推理升级为持续执行系统时，这一层通常不可避免

## 推荐技术映射
结合当前偏好的技术栈，可考虑：

- Backend：`Node.js` + `Hono`
- DSL Parser：Lisp 风格解析器
- Policy Engine：`OPA`
- Skill Registry：`Postgres`
- Memory：`Postgres` + `pgvector`
- RAG Embedding：如 `BGE`
- Workflow：`Temporal`

## 系统视角
从平台化角度看，这套结构可理解为：

```text
Prompt
+ Planner
+ DSL
+ Policy
+ Skill
+ Memory
+ RAG
+ Workflow
= AI Application OS
```

## 与现有知识库的关系
这是一篇平台级蓝图文档，应作为更高层参考，并向下连接：
- `principles/`：系统约束
- `patterns/`：可复用结构
- `methods/`：落地步骤
- `decisions/`：具体取舍

## 适合继续拆分的主题
后续可从本架构继续拆出：
- `skill-graph-vs-tool-catalog.md`
- `memory-vs-rag.md`
- `policy-engine-for-ai-saas.md`
- `workflow-engine-for-ai-agents.md`
- `multi-tenant-ai-platform.md`

## 相关文档
- `knowledge/methods/project-knowledge-graph.md`
- `knowledge/patterns/unified-pkg-layer.md`
- `knowledge/principles/agent-query-before-code.md`
