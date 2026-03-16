---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [reconcile, visual-spec, ui-schema, stitch, codegen]
---

# Reconcile Stage Spec

## 问题
即使已经有 `Visual Spec`、`UI Schema` 和高保真草稿代码，如果缺少明确的对齐阶段规范，最终输出代码仍然会出现两类问题：
- 视觉层很好，但工程结构混乱
- 工程结构完整，但最终页面偏离高保真草稿

## 上下文
该规范用于定义双轨生成中的 `Reconcile` 阶段，也就是把以下输入融合为最终代码的过程：
- `Visual Spec`
- `UI Schema`
- High-fidelity Draft Code
- 工程运行时骨架（可选）

## 目标
让对齐阶段稳定解决以下问题：
- 保留高保真页面结构和样式
- 注入标准化数据绑定、状态和动作
- 统一组件边界、命名与代码组织
- 让最终代码符合项目工程规范

## 设计原则
- 优先保留视觉轨中明确的视觉真相
- 优先保留工程轨中明确的数据与动作约束
- 不要求视觉草稿直接变成最终可维护代码
- 不要求 `UI Schema` 直接决定所有样式细节
- 对齐阶段应以“融合”而不是“覆盖”为目标

## 输入
Reconcile 阶段建议至少接收以下输入：
- `visualSpec`
- `uiSchema`
- `draftCode`
- `projectConstraints`
- `namingRules`
- `componentMappingRules`

## 输出
建议输出以下产物：
- 最终页面代码
- 样式文件或样式映射
- 绑定层代码
- runtime / api / state 接入骨架
- 对齐摘要，用于 review 或调试

## 典型职责

### 1. 视觉保留
保留：
- 布局层级
- 视觉间距
- 文本样式
- 表面样式
- 图片展示方式

### 2. 工程注入
注入：
- 数据绑定
- 动作映射
- 状态声明
- runtime 入口
- API 调用骨架

### 3. 边界统一
统一：
- 组件命名
- 文件组织
- 页面级配置
- 重复视觉片段的组件化策略

### 4. 冲突处理
当视觉轨和工程轨冲突时，建议按以下优先顺序处理：
1. 数据与动作语义不能丢
2. 页面主布局与关键视觉层次尽量保留
3. 可通过局部重写解决的样式冲突尽量局部修正
4. 无法自动解决时，生成对齐警告或 follow-up task

## 推荐流程

```text
Visual Spec
 + UI Schema
 + Draft Code
 + Project Rules
   -> Reconcile Analyzer
   -> Mapping / Injection
   -> Conflict Resolution
   -> Final Code
   -> Reconcile Summary
```

## 最小对齐任务
第一版建议至少支持以下几类对齐：
- `text`：静态文案替换为动态绑定
- `button`：视觉按钮注入事件与动作
- `list`：静态重复块对齐为列表渲染结构
- `image`：保留展示样式，同时替换为标准资源引用
- `card`：保留卡片外观，同时注入数据字段

## 最小规则示例

### 规则 1：静态文本转绑定
如果草稿代码里存在明显占位文本，而 `UI Schema` 中存在对应绑定字段：
- 保留文本节点的视觉样式
- 把文案内容替换为绑定表达式

### 规则 2：静态重复块转列表
如果视觉草稿中出现重复结构，而 `UI Schema` 中存在 `repeat.source`：
- 保留单项视觉结构
- 外层改写为列表渲染
- 注入 key 与 item 绑定

### 规则 3：按钮注入动作
如果视觉草稿中存在按钮，而 `UI Schema` 中存在对应 action：
- 保留按钮样式与文案
- 注入标准事件处理与参数传递

### 规则 4：图片接资源层
如果视觉草稿中图片为本地静态路径，但工程层要求资源标准化：
- 保留展示尺寸、裁剪与圆角
- 替换为统一资源引用方式

## 对齐摘要
建议每次对齐都生成一份摘要，至少包含：
- 使用了哪些输入
- 完成了哪些对齐动作
- 哪些视觉信息被保留
- 哪些工程约束被注入
- 是否存在未解决冲突

## 优点
- 让双轨生成真正落地，而不是停留在抽象层
- 避免单一路径生成既要保真又要工程化的过载问题
- 提高最终代码的稳定性与可维护性

## 代价
- 需要维护额外的映射与冲突处理规则
- 需要为不同组件类型逐步扩展对齐能力

## 不适用场景
- 完全一次性页面
- 不关心数据、状态和动作注入的纯静态视觉生成

## 相关文档
- `knowledge/patterns/visual-spec-spec.md`
- `knowledge/patterns/ui-schema-spec.md`
- `knowledge/patterns/visual-spec-plus-ui-schema.md`
- `knowledge/decisions/accepted/005-use-dual-track-generation-for-stitch-to-code.md`
