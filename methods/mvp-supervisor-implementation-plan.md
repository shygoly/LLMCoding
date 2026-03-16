---
type: method
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [mvp, supervisor, implementation, cursor-agent, sqlite]
---

# MVP Supervisor Implementation Plan

## 目标
把当前关于 `Supervisor Subagent + cursor-agent + SQLite + artifacts/` 的方法、模式、协议和状态模型，整理成一套可按阶段落地的 MVP 实施计划，避免实现时再次回到“边想边做”的状态。

## 适用场景
- 准备从知识设计进入最小可用系统实现
- 希望先做单机、本地、低并发版本
- 希望尽快验证“Spec 驱动 + 监督执行 + 共享状态”是否真的可行

## 前提条件
- 已有基本的任务来源，如 spec、issue、PRD 或手工任务列表
- 已接受 `cursor-agent` 作为代码执行器命名
- 已确认共享状态采用 `SQLite + artifacts/`
- 已具备任务派发协议、结果回传协议和状态模型草案

## 输入
- `knowledge/methods/spec-driven-cursor-supervision.md`
- `knowledge/patterns/supervisor-subagent-with-shared-store.md`
- `knowledge/patterns/supervisor-task-payload-spec.md`
- `knowledge/patterns/cursor-result-payload-spec.md`
- `knowledge/patterns/runtime-state-schema.md`
- `knowledge/patterns/sqlite-ddl-for-runtime-state.md`
- `knowledge/patterns/supervisor-review-rules.md`

## 核心步骤
1. 先实现运行时状态层，让任务和结果有稳定落点
2. 再实现 Supervisor 的最小状态推进逻辑
3. 再接入 `cursor-agent` 的任务派发与结果回传
4. 最后补充知识回写与 review 机制

## 建议目录结构
第一版建议采用如下目录：

```text
runtime/
  state.db
  artifacts/

apps/
  supervisor/
    src/
      db/
      state/
      dispatch/
      review/
      writeback/

packages/
  runtime-schema/
  task-protocol/
```

如果不想拆太早，也可以先做成单仓目录：

```text
supervisor/
  src/
  runtime/
```

## 分阶段实施

### Phase 1. Runtime State MVP
目标：先把共享状态跑起来。

实现内容：
- 建立 `runtime/state.db`
- 按 `sqlite-ddl-for-runtime-state` 创建表和索引
- 实现最小数据库访问层
- 能创建 spec、创建 task、更新 task 状态、记录 task_events、记录 artifacts、记录 feedback

验收标准：
- 可以手工插入和查询 spec / task
- 可以从数据库中恢复当前待执行任务
- 可以查询某个 task 的完整事件轨迹

### Phase 2. Supervisor Core MVP
目标：先做最小的监督流程，不追求智能。

实现内容：
- 读取 `ready` task
- 把 task 转换成 `Supervisor Task Payload`
- 维护最小状态机：`ready -> dispatched -> in_progress -> review -> completed/needs_fix/blocked`
- 根据 `Supervisor Review Rules` 做基础审查判定

验收标准：
- 能从数据库挑选任务并推进状态
- 能消费一个结构化结果并给出审查结论
- 能生成 feedback 记录

### Phase 3. cursor-agent Integration MVP
目标：接通执行器。

实现内容：
- 定义 `cursor-agent` 的输入输出接口
- Supervisor 把 payload 写到约定位置或通过约定命令发给 `cursor-agent`
- `cursor-agent` 执行后回传 `Cursor Result Payload`
- 回传结果写入 `last_result_json`、`task_events`、`artifacts`

验收标准：
- 至少能成功跑通一个单任务流程
- `cursor-agent` 重启后，Supervisor 仍能从共享状态恢复任务视图

### Phase 4. Review And Fix Loop MVP
目标：验证多轮修正闭环。

实现内容：
- 对 `needs_fix` 任务生成修正反馈
- 允许任务重新进入 `dispatched`
- 区分 `blocked` 与 `needs_fix`
- 对严重越界结果做阻断

验收标准：
- 至少能跑通一次 “派发 -> 回传 -> needs_fix -> 再派发 -> completed”
- 至少能跑通一次 `blocked` 分支

### Phase 5. Knowledge Writeback MVP
目标：把执行和知识层接起来。

实现内容：
- 当结果中 `writeback` 指示需要更新知识时，生成 follow-up task 或 review 记录
- 形成最小知识回写摘要
- 记录哪些 task 影响了哪些知识文档

验收标准：
- 至少能把一次任务结果关联到 `knowledge/` 文档
- 形成一条 review 或 follow-up 记录

## 模块职责建议

### `db/`
- SQLite 初始化
- 建表与索引
- 基础 CRUD

### `state/`
- 状态机推进
- task 查询与选择
- 事件记录

### `dispatch/`
- 生成 `Supervisor Task Payload`
- 派发给 `cursor-agent`
- 记录派发事件

### `review/`
- 读取 `Cursor Result Payload`
- 按 `Supervisor Review Rules` 做判定
- 生成 feedback

### `writeback/`
- 解析结果中的知识回写信号
- 生成知识更新任务、review 记录或摘要

## 最小里程碑
建议按以下顺序验收：

1. **Milestone A**：SQLite 状态层可用
2. **Milestone B**：Supervisor 可推进任务状态
3. **Milestone C**：单任务接通 `cursor-agent`
4. **Milestone D**：支持 `needs_fix` 重试
5. **Milestone E**：支持知识回写触发

## 风险与边界
- 第一版不要试图做全自动多任务并发
- 第一版不要让 Supervisor 自己重新规划大规模任务树
- 第一版不要让 `cursor-agent` 自主决定任务顺序
- 若 payload / result 协议频繁变化，应先冻结协议再扩展实现

## 验证方式
- 用一个小而完整的 spec 跑通端到端流程
- 检查 `state.db` 是否能完整反映任务生命周期
- 检查 `runtime/artifacts/` 是否能保留关键执行证据
- 检查 review 与 feedback 是否足以支持第二轮修正

## 相关文档
- `knowledge/methods/spec-driven-cursor-supervision.md`
- `knowledge/patterns/sqlite-ddl-for-runtime-state.md`
- `knowledge/patterns/supervisor-task-payload-spec.md`
- `knowledge/patterns/cursor-result-payload-spec.md`
- `knowledge/patterns/supervisor-review-rules.md`
