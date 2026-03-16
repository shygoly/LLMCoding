---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [supervisor, cursor, shared-store, sqlite, daemon]
---

# Supervisor Subagent With Shared Store

## 问题
如果直接让规划型 Agent 把大段上下文交给代码执行器，长任务往往会出现上下文漂移、任务越界、执行状态丢失和反馈难以闭环的问题。

## 上下文
适用于存在 `OpenClaw -> cursor-agent` 这类“规划者 + 执行器”分层的系统，尤其适用于使用 spec、任务树、长期知识库和多轮修正流程的自动化编程场景。

## 结构
推荐的结构如下：

```text
OpenClaw
  -> Spec / Plan
  -> Supervisor Subagent
  -> Shared Store
  -> cursor-agent
```

更细分的关系：

```text
OpenClaw
  -> 生成或修订 spec
  -> Supervisor 读取 spec 并推进 task
  -> Shared Store 保存状态与结果
  -> cursor-agent 执行单个 task
  -> Supervisor 审查执行结果
  -> 必要时回写 knowledge/
```

## 组件职责

### 1. `OpenClaw`
负责：
- 目标澄清
- 高层规划
- 知识组织
- spec 形成与修订

不负责：
- 长任务的逐步执行监督
- 单步代码实现细节管理

### 2. `Supervisor Subagent`
负责：
- 把 spec 转换为可执行 task
- 选择当前应执行的 task
- 给 cursor-agent 提供最小必要上下文
- 审查结果并决定状态推进
- 发现偏航时要求修正或升级问题

不负责：
- 代替 Cursor 直接完成全部编码
- 绕过共享状态做隐式调度

### 3. `cursor-agent`
负责：
- 执行单个 task
- 修改代码、运行命令、生成结果
- 输出变更摘要、日志、patch、错误信息

不负责：
- 决定总体任务顺序
- 擅自扩展任务范围
- 依赖内部会话记忆维持系统状态

### 4. `Shared Store`
负责：
- 保存 spec 与 task 状态
- 保存事件、反馈、工件索引
- 作为多角色共享的事实源

MVP 推荐：
- `SQLite + artifacts/`

## 典型工作流
1. `OpenClaw` 基于目标和知识库形成 spec
2. `Supervisor` 把 spec 拆为 tasks，并写入共享状态
3. `Supervisor` 选取当前 `ready` task，派发给 `cursor-agent`
4. `cursor-agent` 执行任务，回写结果、日志和阻塞信息
5. `Supervisor` 进入 `review`，检查结果是否满足验收
6. 若通过，任务标记为 `completed`；否则进入 `needs_fix` 或 `blocked`
7. 若修改触及方法、模式、决策或规范，回写 `knowledge/`

## 为什么这个模式有效
- 把“规划”和“执行监督”分离，减少单个 Agent 负担
- 让执行器只处理当前 task，降低上下文噪音
- 让状态与反馈外置，减少会话漂移风险
- 让任务可恢复、可重试、可审计

## 关键设计约束
- `cursor-agent` 一次只处理一个明确 task
- `Supervisor` 只下发当前任务相关上下文，不发送整份全局 spec
- 所有状态变化必须进入共享存储
- 任何任务修正意见必须可追踪
- 共享存储是权威来源，会话只是临时缓存

## 优点
- 适合长任务与多轮修正
- 更容易做恢复、审计和回放
- 更容易控制任务范围与验收边界
- 更适合接入知识库与 spec 驱动流程

## 代价
- 比直接调用 CLI 多一层系统复杂度
- 需要定义 task 模型、状态机和反馈格式
- 需要治理共享状态，否则会出现积累的脏记录

## 不适用场景
- 一次性小修小补
- 没有 spec、没有任务树、没有长期维护需求的场景
- 单人手工快速实验且不关心可恢复性的短任务

## 相关文档
- `knowledge/methods/spec-driven-cursor-supervision.md`
- `knowledge/principles/shared-state-over-session-memory.md`
- `knowledge/principles/agent-query-before-code.md`
