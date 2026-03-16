---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [reconcile, warning, review, alignment, codegen]
---

# Reconcile Warning Types

## 问题
如果 Reconcile 阶段只能输出“成功 / 失败”，而没有稳定的 warning 分类，系统就很难表达“代码可以生成，但存在需要人工确认或后续修正的风险点”。这会导致：
- 自动化过程过于脆弱
- review 信息不完整
- follow-up task 难以生成

## 上下文
该文档用于定义 `Reconcile` 阶段的常见 warning 类型，服务于以下场景：
- `Visual Spec` 与 `UI Schema` 无法完全对齐
- Draft Code 中存在不确定的结构或命名
- 工程注入完成，但部分质量无法自动保证

## 目标
建立一套稳定的 warning 分类，使 Reconcile 阶段能够：
- 报告非致命问题
- 支持 review
- 支持 follow-up task 生成
- 支持后续统计哪些问题最常见

## 设计原则
- warning 不等于阻塞错误
- warning 应该是可分类、可追踪、可汇总的
- warning 要尽量服务后续动作，而不是只做描述
- 第一版分类不要过多，但要覆盖高频风险

## 推荐输出结构
建议 warning 至少包含：

```json
{
  "code": "binding_missing",
  "level": "warning",
  "message": "未找到与标题文本对应的绑定字段",
  "nodeRef": "school-name",
  "suggestedAction": "review_schema_binding"
}
```

建议字段：
- `code`
- `level`
- `message`
- `nodeRef`
- `relatedRefs`
- `suggestedAction`

## Warning 分类

### 1. `binding_missing`
含义：
- Draft Code 或 Visual 节点存在明显应动态化的位置，但 `UI Schema` 中找不到对应绑定

常见场景：
- 标题文本、价格、分数、用户名等内容没有对应字段

建议动作：
- 回查 `UI Schema`
- 生成 follow-up：补绑定定义

### 2. `binding_ambiguous`
含义：
- 有多个可能绑定来源，但无法自动确定该用哪一个

常见场景：
- “标题”可能对应 `school.name` 或 `school.title`
- 多个字段语义相近

建议动作：
- 人工确认字段映射
- 增加语义 hint

### 3. `action_missing`
含义：
- 视觉按钮或交互节点存在，但 `UI Schema` 中没有对应 action

常见场景：
- “查看详情” 按钮没有 action 定义
- 卡片视觉可点击，但工程层未定义导航或事件

建议动作：
- 补 action 定义
- 降级为静态节点并提示 review

### 4. `repeat_detection_uncertain`
含义：
- Draft Code 或视觉结构中存在疑似重复块，但无法稳定确认其应转为列表

常见场景：
- 三个卡片看起来很像，但其中一个布局略有差异

建议动作：
- 标记为候选列表
- 人工确认是否启用列表重写

### 5. `layout_alignment_uncertain`
含义：
- Visual Spec 中的布局层级与 Draft Code 或 UI Schema 的组件层级无法稳定一一对应

常见场景：
- 设计图中是嵌套 Frame，草稿代码中已压平或重新组织

建议动作：
- 保留当前可用结构
- 输出 warning 供人工检查

### 6. `visual_token_unresolved`
含义：
- 视觉属性存在，但无法映射到标准 token 或项目样式系统

常见场景：
- 特殊阴影、渐变、非标准圆角值、混合色

建议动作：
- 记录原始值
- 后续决定是否扩展 token 系统

### 7. `asset_reference_unresolved`
含义：
- 图片或图标资源在工程层无法找到稳定引用方式

常见场景：
- Draft Code 使用导出路径，runtime 要求统一资源层
- 远程资源来源未确定

建议动作：
- 标记资源替换待办
- 生成 follow-up：资源标准化

### 8. `component_boundary_uncertain`
含义：
- 某段视觉结构是否应抽成独立组件不明确

常见场景：
- 多个 card 高度相似，但局部变体较多

建议动作：
- 第一版先保留内联结构
- 后续通过 review 决定是否抽组件

### 9. `style_preservation_risk`
含义：
- 注入工程结构后，可能破坏原始视觉样式

常见场景：
- 把容器改成标准按钮/列表后，样式类名或层级发生变化

建议动作：
- 输出对齐摘要
- 需要做视觉回归检查

### 10. `schema_visual_conflict`
含义：
- `UI Schema` 的结构要求与 `Visual Spec` 的主要视觉层次冲突

常见场景：
- Schema 要求列表化，但视觉层并非重复平铺结构
- Schema 要求标准组件边界，但设计上存在强依赖的复合布局

建议动作：
- 优先保留语义正确性
- 输出高优先级 warning
- 必要时生成人工 review 任务

## 推荐等级
第一版建议使用三档：
- `info`：提示性信息，不影响继续生成
- `warning`：可继续生成，但需要 review 或 follow-up
- `high_warning`：可生成，但风险较高，应优先检查

## 与状态的关系
- warning 默认不直接导致 `blocked`
- 若 warning 数量过多或出现关键冲突，可在 review 阶段升级为 `needs_fix` 或 `blocked`
- `schema_visual_conflict`、`style_preservation_risk` 等高风险 warning 应优先进入 review 摘要

## 生成用途
warning 分类可用于：
- Reconcile Summary
- Supervisor review
- follow-up task 自动生成
- 后续统计最常见的失败模式

## 优点
- 让 Reconcile 输出更细粒度、更可操作
- 支持从“勉强能生成”走向“可治理的生成系统”
- 便于未来自动化 review 和质量度量

## 代价
- 需要维护 warning 分类和建议动作映射
- warning 太多时需要治理优先级，避免噪音

## 相关文档
- `knowledge/patterns/reconcile-stage-spec.md`
- `knowledge/patterns/reconcile-rules-for-common-components.md`
- `knowledge/patterns/supervisor-review-rules.md`
- `knowledge/decisions/accepted/005-use-dual-track-generation-for-stitch-to-code.md`
