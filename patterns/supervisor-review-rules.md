---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [supervisor, review, rules, state-machine, cursor-agent]
---

# Supervisor Review Rules

## 问题
即使已经有任务派发协议、结果回传协议和运行时状态模型，如果 Supervisor 没有稳定的审查规则，任务状态仍然会依赖临场判断，导致 `completed`、`needs_fix`、`blocked` 等状态不一致，降低整个执行系统的可预测性。

## 上下文
该模式用于定义 `Supervisor Subagent` 在消费 `cursor-agent` 回传结果时的判定规则。它连接以下几个部分：
- `Supervisor Task Payload Spec`
- `Cursor Result Payload Spec`
- `Runtime State Schema`
- 共享状态中的任务状态机

## 目标
让 Supervisor 在 review 阶段能稳定判断：
- 任务是否满足验收标准
- 是否发生越界修改
- 是否需要修正后重试
- 是否存在阻塞需要升级
- 是否需要回写知识库或规范

## 设计原则
- 优先依据结构化字段判断，不依赖模糊自然语言
- 先检查范围与阻塞，再检查完成度
- 审查结论必须可追踪，并写入共享状态
- 审查要支持最小自动化，不要求一开始就完全智能
- 审查规则应尽量统一，减少不同任务之间的标准漂移

## 输入
Supervisor 在 review 阶段至少应使用以下输入：
- 当前 task 的派发载荷
- `cursor-agent` 的结果回传
- 共享状态中的 task 当前状态
- 相关工件索引，如 `diff`、`summary`、`log`
- 必要时引用相关知识文档与约束

## 审查顺序
推荐按以下顺序进行审查：

### 1. 身份一致性检查
确认：
- `result.taskId` 与当前 review task 一致
- `result.specId` 与任务所属 spec 一致
- 任务当前状态允许进入 review

若不一致：
- 标记为 `blocked`
- 写入反馈：`result identity mismatch`

### 2. 范围检查
确认：
- `result.changes.changedFiles` 是否落在 `scope.files` 或允许范围内
- `outOfScopeChanges` 是否为空
- 是否出现未授权的大范围重构、重命名或无关修改

若超范围：
- 通常标记为 `needs_fix`
- 如越界严重，标记为 `blocked`

### 3. 阻塞检查
确认：
- `result.blockers` 是否为空
- 若存在 blocker，是否属于信息缺失、依赖缺失、环境问题或验收冲突

若存在 blocker：
- 标记为 `blocked`
- 要求附带 blocker 说明与所需补充信息

### 4. 验收检查
逐项比对 `payload.acceptance`：
- 是否有结果证明已经满足
- 是否有 `checks` 支持完成判断
- 是否仅“声称完成”但没有对应结果依据

若验收大部分满足：
- 进入下一步

若关键验收项未满足：
- 标记为 `needs_fix`

### 5. 结果质量检查
确认：
- `summary` 是否清晰说明完成内容
- `checks` 是否足以支撑当前状态建议
- `artifacts` 路径是否存在且可读取
- 是否产生必要的 `diff`、`log`、`summary`

若结果不完整但可修补：
- 标记为 `needs_fix`

### 6. 知识回写检查
确认：
- `writeback.knowledgeUpdateSuggested` 是否为真
- 当前变更是否影响 method、pattern、decision、principle 或 spec
- 是否应新增 review 记录或文档更新任务

若需要回写：
- 不一定阻塞当前 task 完成
- 但应生成 follow-up task 或记录反馈

## 判定规则

### 判定为 `completed`
满足以下条件时可判定：
- 身份一致
- 无 blocker
- 无严重越界修改
- 核心验收项全部满足
- 必要工件齐全
- 若需要回写，已生成明确后续动作

### 判定为 `needs_fix`
满足以下任一情况时可判定：
- 存在轻度越界但可修正
- 核心逻辑已实现，但验收项不完整
- 工件缺失、总结不足或验证不足
- 回传结构基本正确，但结果不够可审查

### 判定为 `blocked`
满足以下任一情况时可判定：
- 存在 blocker 且无法由当前任务直接解决
- 身份不一致或状态机异常
- 越界严重，已无法视为当前 task 的合理结果
- 缺少关键上下文，继续执行会导致高风险偏航

### 判定为 `failed`
通常保留给以下情况：
- 重复多轮修正后仍无法满足最小验收
- 执行器报错且无可行恢复路径
- 明确需要人工介入重新拆分任务

## 输出动作
Supervisor 每次 review 后至少应输出：
- 新状态建议：`completed` / `needs_fix` / `blocked` / `failed`
- 一段结构化反馈摘要
- 是否需要生成 follow-up task
- 是否需要更新知识库或 review 文档

## 反馈模板
建议最小反馈结构如下：

```json
{
  "taskId": "task-007",
  "decision": "needs_fix",
  "reasons": [
    "acceptance item 2 not satisfied",
    "summary lacks evidence for generated Taro list output"
  ],
  "requiredActions": [
    "补充生成结果说明",
    "增加最小验证输出"
  ],
  "followUpNeeded": false
}
```

## 与状态机的关系
推荐流转：
- `review -> completed`
- `review -> needs_fix`
- `review -> blocked`
- `needs_fix -> dispatched`
- `blocked -> escalated` 或补足信息后重新进入 `ready`

## 优点
- 让 review 从“主观判断”变成“可解释规则”
- 提高多轮修正的一致性
- 便于后续自动化实现 Supervisor
- 便于审计每次任务为什么通过或未通过

## 代价
- 需要维护一套明确的审查标准
- 某些复杂任务仍需要人工参与最终判断

## 不适用场景
- 一次性极小任务
- 完全人工同步盯执行、无需共享状态的流程

## 相关文档
- `knowledge/patterns/supervisor-task-payload-spec.md`
- `knowledge/patterns/cursor-result-payload-spec.md`
- `knowledge/patterns/runtime-state-schema.md`
- `knowledge/patterns/supervisor-subagent-with-shared-store.md`
