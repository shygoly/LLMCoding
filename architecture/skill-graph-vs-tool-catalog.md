---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [skill-graph, tool-catalog, agent, architecture, saas]
---

# Skill Graph Vs Tool Catalog

## 目标
解释为什么 AI 系统会从“Tool Catalog”逐步演化到“Skill Graph”，以及这种变化对 AI SaaS、Agent 平台和长期任务系统意味着什么。

## 适用场景
- AI SaaS 平台架构设计
- Agent 系统能力建模
- 多工具、多步骤、多约束的执行系统
- 需要长期维护与演化的 Skill 平台

## 核心判断
`Tool Catalog` 适合回答：
- 系统里“有什么工具”

而 `Skill Graph` 更适合回答：
- 这些能力之间“如何协作”
- 什么能力依赖什么能力
- 哪条执行路径更适合当前任务
- 哪些技能适合组合成长期工作流

因此，当系统从“单次调用工具”升级到“持续执行任务”时，`Tool Catalog` 往往不够用，系统会自然走向 `Skill Graph`。

## Tool Catalog 是什么
`Tool Catalog` 是一个能力目录。

它通常记录：
- 名称
- 描述
- 参数 schema
- 权限要求
- 速率限制
- 执行入口

典型结构类似：

```text
search_pubmed
patient_record
generate_report
send_email
```

它解决的是：
- 工具可发现性
- 工具可调用性
- 工具治理

## Tool Catalog 的局限
当系统能力开始增加时，单纯目录会遇到问题：
- 不知道哪些工具应该组合使用
- 不知道哪些工具是前置依赖
- 不知道工具输出是否可直接成为另一个工具输入
- 不知道哪一条调用链更符合特定任务
- 不知道失败后该如何切换路径

换句话说，Catalog 知道“点”，但不知道“边”。

## Skill Graph 是什么
`Skill Graph` 是对能力网络的建模。

除了保留每个 skill 的元数据之外，还额外表达：
- 依赖关系
- 可组合关系
- 前置条件
- 输入输出兼容性
- 成本与风险
- 可替代路径
- 典型 workflow 连接方式

它不只回答“有哪些 skill”，还回答：
- skill 之间如何连接
- 在什么条件下应该走哪条路径
- 哪些 skill 构成复合能力

## 一个直观对比

### Tool Catalog 视角
```text
search_doc
summarize_doc
write_report
send_email
```

### Skill Graph 视角
```text
search_doc -> summarize_doc -> write_report -> send_email
               \-> extract_risks -> write_report
```

Catalog 只是列表。
Graph 是能力拓扑。

## 为什么先进系统会走向 Skill Graph

### 1. Agent 任务越来越长
短任务时，选一个 tool 就够了。
长任务时，系统必须知道：
- 下一步该接哪个 skill
- 哪些步骤可并行
- 哪些步骤失败后有替代路径

### 2. 工具数量增加后，选择成本激增
当 skills 从 5 个变成 50 个时，LLM 很难每次只靠自然语言描述稳定选中正确工具组合。
Graph 可以帮助系统收缩搜索空间。

### 3. Skill 开始具备复合性
系统中的很多“高级技能”其实不是单一 API，而是：
- 一组小 skill 的组合
- 一个被验证过的子 workflow
- 一个带约束的能力链

这天然更像图，而不是平面目录。

### 4. 平台需要治理执行路径
企业级场景下，平台不仅关心“能不能调用”，还关心：
- 是否允许调用这条路径
- 哪条路径成本更低
- 哪条路径更安全
- 哪条路径适合某一租户或角色

Graph 比 Catalog 更适合承接这类策略。

## Skill Graph 里可以表达什么
推荐至少表达以下节点与边：

### 节点
- `Skill`
- `CompositeSkill`
- `Workflow`
- `Capability`
- `Resource`
- `Policy`

### 边
- `DEPENDS_ON`
- `CAN_FEED`
- `ALTERNATIVE_TO`
- `REQUIRES_POLICY`
- `USED_IN`
- `PRODUCES`
- `CONSUMES`

## 一个最小示例

```text
[search_pubmed] --PRODUCES--> [paper_list]
[paper_list] --CAN_FEED--> [summarize_papers]
[summarize_papers] --USED_IN--> [clinical_report_workflow]
[clinical_report_workflow] --REQUIRES_POLICY--> [doctor_role]
```

## 对 SaaS 平台的意义
当你做的是 AI SaaS 平台，而不是单个 demo agent 时，Skill Graph 会带来三类好处：

### 1. 更好的能力编排
可以把技能当作平台积木组合，而不是每次临时写 prompt 串联。

### 2. 更好的权限治理
可以把 policy 绑定到技能路径，而不是只绑到单个工具。

### 3. 更好的可观测性
可以统计：
- 哪些 skill 最常被组合
- 哪些路径失败率高
- 哪些节点是瓶颈
- 哪些替代路径有效

## 什么时候 Catalog 够用
如果系统处于以下阶段，Catalog 仍然够用：
- 工具数量少
- 任务较短
- 主要是单步调用
- 不强调复杂 workflow
- 不需要强策略控制

## 什么时候应该升级到 Skill Graph
出现以下信号时，通常说明该升级：
- 一个任务经常需要 3 个以上工具串联
- 同类任务有多种执行路径
- 需要对路径做权限或成本控制
- 平台已经出现“复合技能”
- LLM 仅靠自然语言描述选工具开始不稳定

## 推荐演进路径
不要一开始就做复杂图平台，可按以下路径演进：

1. `Tool Catalog`
2. `Catalog + metadata enrichment`
3. `Catalog + dependency edges`
4. `Skill Graph`
5. `Skill Graph + policy + workflow`

## 与现有知识库的关系
这篇文档属于平台级架构蓝图，向下可连接：
- `patterns/`：具体 skill 注册与执行模式
- `methods/`：如何逐步实现 skill graph
- `decisions/`：是否正式采用 graph 作为平台能力模型

## 后续可继续拆分的主题
- `skill-registry-schema.md`
- `composite-skill-pattern.md`
- `policy-bound-skill-paths.md`
- `workflow-on-top-of-skill-graph.md`

## 相关文档
- `knowledge/architecture/ai-application-os.md`
- `knowledge/patterns/unified-pkg-layer.md`
- `knowledge/methods/spec-driven-cursor-supervision.md`
