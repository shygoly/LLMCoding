---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [sqlite, ddl, runtime, state, schema]
---

# SQLite DDL For Runtime State

## 问题
如果只有运行时状态模型说明，而没有接近可执行的 SQLite DDL 草案，后续实现 Supervisor、共享状态存储和 `cursor-agent` 协作时，仍然需要重复做一次表结构设计，容易引入偏差。

## 上下文
该文档基于 `Runtime State Schema`，提供一份适合作为 MVP 起点的 SQLite DDL 设计草案。目标不是一步到位覆盖所有复杂场景，而是先支撑：
- spec 管理
- task 状态推进
- 事件记录
- 工件索引
- 反馈记录

## 目标
给出一组可以直接转化为实现代码的 SQLite 建表语句与最小索引建议，为 Supervisor 执行系统提供稳定事实源。

## 设计原则
- 优先满足任务流转与审查需求
- 核心查询字段单独建列
- 扩展信息放 JSON 文本列
- 外键关系清晰
- 尽量保持 SQLite 兼容性，避免过度依赖复杂特性

## 建表语句

### 1. `specs`

```sql
CREATE TABLE IF NOT EXISTS specs (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  status TEXT NOT NULL,
  source_type TEXT,
  content_json TEXT NOT NULL,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  CHECK (status IN ('draft', 'active', 'paused', 'completed', 'cancelled'))
);
```

### 2. `tasks`

```sql
CREATE TABLE IF NOT EXISTS tasks (
  id TEXT PRIMARY KEY,
  spec_id TEXT NOT NULL,
  title TEXT NOT NULL,
  status TEXT NOT NULL,
  priority INTEGER NOT NULL DEFAULT 0,
  assignee TEXT,
  goal TEXT,
  scope_json TEXT,
  constraints_json TEXT,
  acceptance_json TEXT,
  payload_json TEXT,
  last_result_json TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (spec_id) REFERENCES specs(id),
  CHECK (status IN (
    'planned',
    'ready',
    'dispatched',
    'in_progress',
    'review',
    'needs_fix',
    'blocked',
    'completed',
    'failed'
  ))
);
```

### 3. `task_events`

```sql
CREATE TABLE IF NOT EXISTS task_events (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  task_id TEXT NOT NULL,
  event_type TEXT NOT NULL,
  payload_json TEXT,
  created_at TEXT NOT NULL,
  FOREIGN KEY (task_id) REFERENCES tasks(id)
);
```

### 4. `artifacts`

```sql
CREATE TABLE IF NOT EXISTS artifacts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  task_id TEXT NOT NULL,
  kind TEXT NOT NULL,
  path TEXT NOT NULL,
  meta_json TEXT,
  created_at TEXT NOT NULL,
  FOREIGN KEY (task_id) REFERENCES tasks(id)
);
```

### 5. `feedback`

```sql
CREATE TABLE IF NOT EXISTS feedback (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  task_id TEXT NOT NULL,
  source TEXT NOT NULL,
  status TEXT,
  message TEXT NOT NULL,
  payload_json TEXT,
  created_at TEXT NOT NULL,
  FOREIGN KEY (task_id) REFERENCES tasks(id)
);
```

## 最小索引

```sql
CREATE INDEX IF NOT EXISTS idx_tasks_spec_id ON tasks(spec_id);
CREATE INDEX IF NOT EXISTS idx_tasks_status ON tasks(status);
CREATE INDEX IF NOT EXISTS idx_tasks_priority_updated_at ON tasks(priority DESC, updated_at ASC);
CREATE INDEX IF NOT EXISTS idx_task_events_task_id ON task_events(task_id);
CREATE INDEX IF NOT EXISTS idx_artifacts_task_id ON artifacts(task_id);
CREATE INDEX IF NOT EXISTS idx_feedback_task_id ON feedback(task_id);
```

## 推荐初始化设置
SQLite 初始化时建议启用：

```sql
PRAGMA foreign_keys = ON;
PRAGMA journal_mode = WAL;
```

说明：
- `foreign_keys = ON`：确保任务与 spec、事件、工件之间的引用关系有效
- `WAL`：更适合读写并存的本地执行流程

## 最小迁移顺序
建议初始化顺序：
1. 创建 `specs`
2. 创建 `tasks`
3. 创建 `task_events`
4. 创建 `artifacts`
5. 创建 `feedback`
6. 创建索引

## 推荐写入约定

### 创建 spec 时
- 插入 `specs`
- 记录初始状态为 `draft` 或 `active`

### 创建 task 时
- 插入 `tasks`
- 同时写入一条 `task_events(event_type='task_created')`

### 派发 task 时
- 更新 `tasks.status='dispatched'`
- 写入 `task_events(event_type='task_dispatched')`

### 收到 `cursor-agent` 结果时
- 更新 `tasks.last_result_json`
- 视情况更新 `tasks.status`
- 写入 `task_events(event_type='execution_result_received')`
- 写入对应 `artifacts`

### 生成审查反馈时
- 插入 `feedback`
- 写入 `task_events(event_type='review_completed')`

## 推荐查询

### 查询当前待执行任务
```sql
SELECT id, title, priority, updated_at
FROM tasks
WHERE status = 'ready'
ORDER BY priority DESC, updated_at ASC;
```

### 查询当前进行中的任务
```sql
SELECT id, title, assignee, updated_at
FROM tasks
WHERE status IN ('dispatched', 'in_progress', 'review')
ORDER BY updated_at ASC;
```

### 查询某个 task 的最新反馈
```sql
SELECT source, status, message, created_at
FROM feedback
WHERE task_id = ?
ORDER BY created_at DESC
LIMIT 1;
```

## 演进建议
MVP 之后可考虑增加：
- `task_dependencies`：表达任务间依赖
- `executions`：单独记录每轮执行实例
- `knowledge_links`：显式记录任务关联的知识文档
- `workers`：记录多 `cursor-agent` 或多执行器信息

## 优点
- 非常接近可执行实现
- 便于从知识文档直接落到代码
- 可以与 payload / result 协议直接对接

## 代价
- 仍需在实现层补充迁移脚本和写入逻辑
- 部分 JSON 字段后续可能需要拆分结构化列

## 不适用场景
- 需要高并发、多机分布式调度的系统
- 一开始就需要复杂权限、多租户隔离的系统

## 相关文档
- `knowledge/patterns/runtime-state-schema.md`
- `knowledge/patterns/supervisor-task-payload-spec.md`
- `knowledge/patterns/cursor-result-payload-spec.md`
- `knowledge/patterns/supervisor-review-rules.md`
