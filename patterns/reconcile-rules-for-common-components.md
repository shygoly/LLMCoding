---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [reconcile, components, text, button, list, image, card]
---

# Reconcile Rules For Common Components

## 问题
如果 `Reconcile` 阶段只有总体原则，没有按组件类型拆分的细化规则，那么实现时仍然会回到临场判断，导致不同页面、不同任务对同类组件的处理方式不一致。

## 上下文
该文档用于细化双轨生成体系中常见组件的对齐规则，重点覆盖第一版最常见的前端页面元素：
- `text`
- `button`
- `list`
- `image`
- `card`

## 目标
为 `Visual Spec + UI Schema + Draft Code -> Final Code` 提供组件级规则，使 Reconcile 阶段至少能稳定完成：
- 视觉保留
- 数据绑定注入
- 动作对齐
- 结构重写
- 结果约束

## 总体规则
对所有组件，默认遵循以下顺序：
1. 保留视觉层中已明确的布局与样式信息
2. 注入工程层中的数据、动作和状态约束
3. 统一命名与组件边界
4. 如果视觉与工程冲突，优先保留语义正确性，再尽量保留视觉外观
5. 若无法安全自动对齐，生成 warning 或 follow-up task

## 1. `text`

### 输入特征
- Visual Spec 中通常包含字体、字号、字重、颜色、行高、对齐方式
- Draft Code 中通常已有静态文案
- UI Schema 中可能包含 `bindings.text`

### 对齐规则
- 保留文字节点的视觉样式
- 如果 `UI Schema` 提供动态绑定，则将静态文案替换为绑定表达式
- 如果 `UI Schema` 未提供绑定，则保留静态文案
- 如果文本承担语义标题角色，尽量保留其层级与命名

### 示例
- Draft：`<Text className="title">南京邮电大学</Text>`
- Schema：`bindings.text = school.name`
- Final：保留 `title` 的样式语义，同时内容改为 `school.name`

### 风险
- 错误替换可能把说明文案误改成动态字段
- 多语言或复杂富文本需要单独处理

## 2. `button`

### 输入特征
- Visual Spec 提供按钮大小、颜色、圆角、阴影、文案样式
- Draft Code 中通常已有静态点击元素
- UI Schema 中通常提供 `action`、事件名、参数

### 对齐规则
- 保留按钮的外观、尺寸、间距和文案样式
- 将点击行为与 `UI Schema.actions` 对齐
- 如果需要传参，按 Schema 中的绑定规则注入参数
- 如果草稿里按钮只是普通容器模拟的视觉按钮，可在最终代码中重写为标准按钮组件或标准交互节点

### 示例
- Draft：视觉上是一个圆角蓝色按钮，无事件
- Schema：`events.click -> openSchool(school.id)`
- Final：保留蓝色圆角样式，注入点击事件和参数

### 风险
- 视觉按钮可能是多个嵌套节点组成，重写时容易破坏样式
- 同一视觉按钮可能隐含多个交互态，第一版先以基础交互为主

## 3. `list`

### 输入特征
- Draft Code 中通常是多个相似视觉块平铺
- Visual Spec 记录重复块的布局、间距和单项外观
- UI Schema 中通常有 `repeat.source`、`item` 变量和子项绑定

### 对齐规则
- 识别重复视觉块，把单个块抽出作为列表项模板
- 保留单项视觉结构与间距关系
- 将外层重写为列表渲染结构
- 按 Schema 注入 `item`、`key`、字段绑定和按钮动作

### 示例
- Draft：3 个静态学校卡片
- Schema：`repeat.source = schools`
- Final：改写为 `schools.map(...)`，保留卡片样式和排列方式

### 风险
- 误判重复结构会导致不该列表化的内容被抽象成列表
- 列表项中若存在复杂嵌套交互，需要谨慎注入状态与 action

## 4. `image`

### 输入特征
- Visual Spec 通常提供尺寸、裁剪方式、圆角、占位区域
- Draft Code 常包含导出后的静态路径
- UI Schema / runtime 可能要求走统一资源引用或存储服务

### 对齐规则
- 保留图片的显示尺寸、裁剪、圆角、布局位置
- 将资源地址替换为项目标准资源引用方式
- 若 Schema 或 runtime 有资源层要求，则不要直接保留导出路径作为最终实现
- 对纯装饰图片与业务图片可在命名上做区分

### 示例
- Draft：`src="./banner.png"`
- Runtime：要求统一走 `assets` 或存储服务 URL
- Final：保留显示样式，替换为统一资源引用

### 风险
- 导出图与真实资源层不一致时，容易出现视觉偏差
- 远程资源加载策略可能影响最终渲染表现

## 5. `card`

### 输入特征
- Visual Spec 提供背景、阴影、圆角、边框、内边距
- Draft Code 常已有完整视觉卡片结构
- UI Schema 中通常提供卡片内部的文本、图片、按钮等绑定

### 对齐规则
- 把 `card` 视为视觉容器优先保留其外观
- 内部文本、图片、按钮分别应用对应子规则
- 如 `card` 本身承担交互作用，再注入整卡点击或 hover / active 语义
- 若多个卡片结构一致，可在对齐后考虑抽为标准组件

### 示例
- Draft：静态院校推荐卡片
- Schema：内部字段来自 `school.name`、`school.score`
- Final：保留卡片壳与内部布局，把文本和按钮改为动态绑定与动作

### 风险
- 过早组件化会损失设计差异
- 卡片中的局部变体可能不适合在第一版强行统一

## 推荐实现顺序
第一版建议按以下顺序支持：
1. `text`
2. `button`
3. `image`
4. `card`
5. `list`

原因：
- `text` 和 `button` 最容易稳定注入绑定与动作
- `image` 次之
- `card` 需要组合多个子元素
- `list` 涉及结构重写，复杂度最高

## 输出要求
每次 Reconcile 至少应记录：
- 哪些组件类型被对齐
- 哪些组件保留了视觉外观
- 哪些字段被注入为动态绑定
- 哪些节点发生了结构性重写
- 哪些节点存在未解决风险

## 优点
- 让 Reconcile 从抽象概念变成可逐步实现的规则集
- 便于按组件类型分阶段开发和测试
- 便于未来追加更多组件类型，如 `input`、`tabs`、`modal`

## 代价
- 需要持续维护组件规则库
- 对复杂自定义组件仍需单独扩展规则

## 相关文档
- `knowledge/patterns/reconcile-stage-spec.md`
- `knowledge/patterns/visual-spec-spec.md`
- `knowledge/patterns/ui-schema-spec.md`
- `knowledge/decisions/accepted/005-use-dual-track-generation-for-stitch-to-code.md`
