---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [memory, rag, ai-os, architecture, retrieval]
---

# Memory Vs RAG

## 目标
澄清 AI 系统中 `Memory` 与 `RAG` 的职责边界，避免把两者混为一谈，并为 AI SaaS 平台设计更清晰的知识与状态架构。

## 适用场景
- AI SaaS 平台
- Agent 系统
- 多轮对话与长期任务系统
- 需要同时管理用户历史、任务上下文和外部知识的应用

## 核心判断
`Memory` 和 `RAG` 都会向 LLM 提供上下文，但它们解决的是不同问题：

- `Memory` 解决：系统“记住了什么”
- `RAG` 解决：系统“查到了什么”

更直白地说：
- `Memory` 偏内部连续性
- `RAG` 偏外部知识获取

## Memory 是什么
`Memory` 是系统对用户、Agent、任务或长期行为状态的持续记录。

它通常回答：
- 这个用户是谁
- 用户有什么偏好
- 之前做过什么
- 当前任务推进到哪一步
- 哪些历史结果需要延续

### 常见 Memory 类型

#### 1. User Memory
记录：
- 用户偏好
- 个人资料
- 历史行为
- 个性化设定

#### 2. Agent Memory
记录：
- 当前任务上下文
- 工具调用历史
- 最近结果
- 执行轨迹

#### 3. Long-term Memory
记录：
- 长期可复用知识
- 持续积累的经验
- 历史总结

## RAG 是什么
`RAG` 是 Retrieval-Augmented Generation，用于从外部知识源中检索相关信息，并把检索结果补充到当前推理上下文。

它通常回答：
- 针对当前问题，应该查什么资料
- 哪些外部文档最相关
- 哪些知识片段可作为当前推理依据

### 常见 RAG 来源
- 企业知识库
- 医学文献
- 产品文档
- 代码仓库
- FAQ / SOP / Wiki

## 为什么两者容易被混淆
因为两者都会：
- 把额外上下文喂给 LLM
- 使用向量检索或相似度搜索
- 在实现上看起来都像“查点东西再回答”

但它们的意图不同：
- `Memory` 是为了连续性
- `RAG` 是为了知识性

## 一个简单对比

### Memory 问题示例
```text
这个用户上次已经选过哪些候选学校？
```

### RAG 问题示例
```text
今年关于院校推荐模型有哪些新论文？
```

前者是系统内部历史。后者是外部知识检索。

## 结构化对比

| 维度 | Memory | RAG |
| --- | --- | --- |
| 核心目的 | 保持连续性 | 获取知识 |
| 信息来源 | 系统内部历史 | 外部知识库 |
| 时间属性 | 强时间连续性 | 按需检索 |
| 面向对象 | 用户 / Agent / Task | 文档 / 知识片段 |
| 主要价值 | 个性化、上下文延续 | 知识增强、事实补充 |

## 在 AI SaaS 中的推荐边界

### Memory 负责
- 用户 profile
- 偏好与设置
- 会话摘要
- Agent 执行轨迹
- 长任务状态
- 最近工具调用历史

### RAG 负责
- 检索外部文档
- 检索业务知识
- 检索产品资料
- 检索代码与规范文档
- 检索行业知识与参考信息

## 为什么不能用 RAG 替代 Memory
如果把所有内容都塞进 RAG：
- 用户和任务状态会变得模糊
- 连续性信息难以稳定更新
- 会话推进很难精确恢复
- 系统会把“状态”误当“知识”来查

这会导致：
- Agent 每次像冷启动
- 个性化能力变差
- 长任务很难稳定推进

## 为什么不能用 Memory 替代 RAG
如果不做 RAG，只靠 Memory：
- 系统无法获得外部最新知识
- 很难回答长尾问题
- 无法支撑领域知识增强
- 会把外部知识错误地塞进内部状态层

这会导致：
- 回答范围过窄
- 难以支撑专业领域问答
- 知识更新成本极高

## 推荐架构关系
在 AI Application OS 中，推荐这样理解：

```text
Memory = 内部连续性层
RAG = 外部知识增强层
```

它们共同服务于 Planner / LLM，但职责不同：

```text
User / Task / Agent History -> Memory
External Docs / KB / Codebase -> RAG
Memory + RAG -> Planner / LLM
```

## 实现建议

### Memory 层
推荐使用：
- `Postgres`
- `SQLite`（MVP）
- `pgvector`（若需要语义检索）

### RAG 层
推荐使用：
- 向量数据库或 `pgvector`
- 文档切片与 embedding pipeline
- 可审计的检索结果缓存

## 平台化视角
如果你在做的是 AI SaaS 平台，而不是单一 Bot，那么最好把两层拆开：

- `Memory Service`
- `RAG Service`

这样做的好处：
- 生命周期不同，便于治理
- 权限与租户隔离更清晰
- 检索逻辑与状态逻辑不会混在一起

## 什么时候该把两者统一看待
仅在非常小的 MVP 中，可以把它们共用同一存储设施，例如：
- 都放在 `Postgres + pgvector`

但即使物理存储共用，逻辑上也应分层。

## 常见错误信号
如果系统出现以下现象，通常说明 Memory 与 RAG 边界混乱：
- 用户历史和外部文档混在一个索引里
- Agent 无法稳定恢复任务状态
- 系统把历史聊天记录当作外部知识事实引用
- 外部知识更新后，内部状态层也被污染

## 与现有知识库的关系
这篇文档是平台级架构蓝图，适合向下连接：
- `principles/`：共享状态、先查询再编码
- `patterns/`：运行时状态模型、RAG 模式、Memory 模式
- `methods/`：如何落地 Memory / RAG 服务

## 后续可继续拆分的主题
- `memory-service-for-ai-saas.md`
- `rag-service-for-ai-saas.md`
- `user-memory-vs-agent-memory.md`
- `retrieval-policy-and-tenant-isolation.md`

## 相关文档
- `knowledge/architecture/ai-application-os.md`
- `knowledge/architecture/skill-graph-vs-tool-catalog.md`
- `knowledge/principles/shared-state-over-session-memory.md`
- `knowledge/methods/project-knowledge-graph.md`
