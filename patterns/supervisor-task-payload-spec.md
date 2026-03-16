---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [supervisor, cursor, payload, spec, protocol]
---

# Supervisor Task Payload Spec

## 问题
如果 Supervisor 向 cursor-agent 发送的任务载荷没有固定结构，执行器就容易在不同任务中接收到不一致的信息，导致范围失控、上下文遗漏、验收条件模糊，以及结果难以自动审查。

## 上下文
该规范用于定义 `Supervisor Subagent -> cursor-agent` 的单任务交接协议。它不是总体 spec，也不是知识库全文，而是当前 task 的最小必要执行载荷。

## 目标
让每一个派发给 cursor-agent 的 task 都具备：
- 稳定 ID
- 明确目标
- 清晰范围
- 显式约束
- 可检查验收标准
- 可回写结果路径

## 设计原则
- 一次只描述一个 task
- 只传当前任务需要的最小上下文
- 文件范围、知识约束、验收标准必须显式给出
- 结果输出位置必须预先约定
- 载荷字段应支持自动审查与状态推进

## 顶层结构
推荐使用 JSON 结构：

```json
{
  "taskId": "task-007",
  "specId": "spec-001",
  "title": "为生成器增加 list 支持",
  "goal": "支持 UI Schema 中 list/repeat 结构生成 Taro 页面",
  "scope": {},
  "constraints": [],
  "knowledgeRefs": [],
  "inputs": {},
  "acceptance": [],
  "execution": {},
  "writeback": {},
  "artifacts": {}
}
```

## 字段规范

### 1. `taskId`
当前任务的唯一标识。

要求：
- 在整个执行周期中保持稳定
- 可用于状态跟踪、日志归档、工件命名

### 2. `specId`
所属总 spec 的标识。

用途：
- 把当前 task 与总体任务树关联起来
- 便于审查时回溯来源

### 3. `title`
任务标题，简要表达本次执行内容。

### 4. `goal`
一句话说明本次任务要达成的目标。

要求：
- 写结果，不写过程
- 避免模糊描述，如“优化一下”

### 5. `scope`
描述本次任务允许触达的范围。

建议字段：

```json
{
  "repositories": ["apps/miniapp"],
  "files": [
    "packages/generator/src/page.ts",
    "packages/ui-schema/src/types.ts"
  ],
  "modules": ["generator", "ui-schema"],
  "maxChangeSurface": "small"
}
```

说明：
- `repositories`：仓库或子项目范围
- `files`：允许修改的主要文件
- `modules`：能力域或模块名
- `maxChangeSurface`：如 `small`、`medium`、`large`

### 6. `constraints`
明确不能违反的约束。

示例：

```json
[
  "必须遵守 UI Schema Spec",
  "不允许直接从 HTML 拼接 Taro JSX",
  "不得修改无关模块"
]
```

### 7. `knowledgeRefs`
当前任务必须参考的知识文档。

示例：

```json
[
  "knowledge/patterns/ui-schema-spec.md",
  "knowledge/principles/schema-first-generation.md",
  "knowledge/principles/agent-query-before-code.md"
]
```

要求：
- 只列必要文档
- 不把整个知识库一股脑发送给执行器

### 8. `inputs`
描述任务执行所需的输入信息。

建议字段：

```json
{
  "relatedCode": [
    "packages/generator/src/page.ts",
    "packages/ui-schema/src/types.ts"
  ],
  "dataExamples": [
    "knowledge/patterns/ui-schema-spec.md"
  ],
  "notes": [
    "已有 button/text 生成逻辑，可参考其组件映射方式"
  ]
}
```

### 9. `acceptance`
任务是否完成的判定标准。

示例：

```json
[
  "支持 list.repeat.source 字段",
  "生成器可输出对应 Taro 列表结构",
  "不破坏现有 text/button 生成逻辑"
]
```

要求：
- 尽量可验证
- 用结果导向语言表达

### 10. `execution`
定义执行时的运行约束。

建议字段：

```json
{
  "mode": "single-task",
  "allowCommands": ["test", "build"],
  "forbid": ["broad-refactor", "rename-unrelated-files"],
  "stopOnBlocked": true
}
```

### 11. `writeback`
规定任务完成后需要补写的内容。

示例：

```json
{
  "updateKnowledge": true,
  "knowledgeTargets": [
    "knowledge/patterns/ui-schema-spec.md"
  ],
  "summaryRequired": true
}
```

### 12. `artifacts`
规定结果输出位置。

示例：

```json
{
  "logPath": "runtime/artifacts/task-007/cursor.log",
  "diffPath": "runtime/artifacts/task-007/patch.diff",
  "summaryPath": "runtime/artifacts/task-007/summary.md"
}
```

## 最小可用载荷示例

```json
{
  "taskId": "task-007",
  "specId": "spec-001",
  "title": "为生成器增加 list 支持",
  "goal": "支持 UI Schema 中 list/repeat 结构生成 Taro 页面",
  "scope": {
    "files": [
      "packages/generator/src/page.ts",
      "packages/ui-schema/src/types.ts"
    ],
    "modules": ["generator", "ui-schema"],
    "maxChangeSurface": "small"
  },
  "constraints": [
    "必须遵守 UI Schema Spec",
    "不允许直接从 HTML 拼接 Taro JSX"
  ],
  "knowledgeRefs": [
    "knowledge/patterns/ui-schema-spec.md",
    "knowledge/principles/schema-first-generation.md"
  ],
  "inputs": {
    "notes": [
      "参考现有 text/button 组件生成逻辑"
    ]
  },
  "acceptance": [
    "支持 list.repeat.source 字段",
    "生成器可输出对应 Taro 列表结构"
  ],
  "execution": {
    "mode": "single-task",
    "stopOnBlocked": true
  },
  "writeback": {
    "updateKnowledge": true,
    "knowledgeTargets": [
      "knowledge/patterns/ui-schema-spec.md"
    ],
    "summaryRequired": true
  },
  "artifacts": {
    "logPath": "runtime/artifacts/task-007/cursor.log",
    "diffPath": "runtime/artifacts/task-007/patch.diff",
    "summaryPath": "runtime/artifacts/task-007/summary.md"
  }
}
```

## 审查要点
Supervisor 在派发前应检查：
- `taskId` 与 `specId` 是否存在
- `goal` 是否明确
- `scope.files` 是否过大
- `constraints` 是否足够约束越界行为
- `knowledgeRefs` 是否精简且必要
- `acceptance` 是否能用于 review
- `artifacts` 路径是否可落盘

## 优点
- 降低执行器的上下文歧义
- 便于自动化审查和状态推进
- 便于把任务执行与共享状态绑定起来
- 便于后续扩展到多 worker 或远程执行

## 代价
- 需要额外维护任务载荷规范
- 任务拆分不合理时，依然会导致执行器困难

## 不适用场景
- 极小的一次性任务
- 完全手工驱动、不打算自动审查的实验性流程

## 相关文档
- `knowledge/methods/spec-driven-cursor-supervision.md`
- `knowledge/patterns/supervisor-subagent-with-shared-store.md`
- `knowledge/principles/shared-state-over-session-memory.md`
