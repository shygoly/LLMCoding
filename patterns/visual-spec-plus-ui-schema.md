---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [visual-spec, ui-schema, stitch, codegen, reconcile]
---

# Visual Spec Plus UI Schema

## 问题
直接从 `UI Schema` 生成页面，往往难以还原 Stitch 页面中的精细视觉层次；但直接让 LLM 根据 Stitch 输出最终工程代码，又容易在数据绑定、动作语义、命名和长期维护上失控。

## 上下文
适用于从 Stitch 或其他设计源生成前端代码，并同时追求：
- 高保真视觉还原
- 可维护的工程结构
- 后续可接 Mock API、真实 API、状态与导航

## 结构
推荐把生成链路拆成两条轨道：

```text
Stitch
  -> Visual Spec
  -> High-fidelity Draft Code

Stitch
  -> UI Schema
  -> Engineering Structure

Draft Code + Engineering Structure
  -> Reconcile / Rewrite
  -> Final Code
```

## 组件说明

### 1. `Visual Spec`
保留高保真视觉信息。

建议包含：
- 布局层级
- 间距
- 字号
- 颜色
- 圆角
- 阴影
- 图片裁剪与位置
- 特定视觉变体

### 2. `UI Schema`
保留工程结构信息。

建议包含：
- 页面元信息
- 组件树与组件 ID
- 数据源
- 动作
- 状态
- 资源引用
- 绑定关系

### 3. `High-fidelity Draft Code`
由 LLM 参考 Stitch 或 Visual Spec 生成高保真静态草稿。

特点：
- 优先追求视觉还原
- 不要求一步到位解决全部工程问题

### 4. `Engineering Structure`
由生成器根据 `UI Schema` 输出工程约束。

可包含：
- 数据绑定骨架
- runtime / api / state 结构
- 页面配置
- 标准组件边界

### 5. `Reconcile / Rewrite`
负责把视觉草稿与工程结构合并。

主要职责：
- 保留高保真布局与样式
- 插入标准化的数据绑定
- 对齐动作与事件语义
- 统一组件命名与代码边界
- 保证结果符合项目规范

## 典型流程
1. 从 Stitch 提取视觉信息，形成 `Visual Spec`
2. 从 Stitch 或 HTML 解析结构和数据需求，形成 `UI Schema`
3. 让 LLM 基于 Stitch / Visual Spec 生成高保真页面草稿
4. 根据 `UI Schema` 生成数据、状态、动作和 runtime 骨架
5. 用对齐器把视觉草稿与工程骨架合并为最终代码
6. 如有需要，再生成 Mock API 与真实 API 接入层

## 为什么这个模式有效
- 视觉还原与工程抽象不再互相拖累
- LLM 负责它最强的部分：高保真复原
- Schema 负责它最强的部分：结构化约束
- 最终代码不必在第一步就同时完美满足“像”和“稳”

## 优点
- 页面更接近原始 Stitch 设计
- 工程代码更容易维护与重生成
- 更适合后续接入 Schema 校验、Mock API、Supabase 或其他后端

## 代价
- 需要维护额外的 `Visual Spec` 或视觉层表示
- 需要实现一层 `Reconcile` 逻辑
- 流程比单一生成路径更复杂

## 不适用场景
- 一次性原型且不关心长期维护
- 视觉要求不高的简单后台页面

## 相关文档
- `knowledge/principles/fidelity-and-structure-dual-track.md`
- `knowledge/patterns/ui-schema-spec.md`
- `knowledge/patterns/html-to-ui-schema.md`
- `knowledge/patterns/mock-api-to-supabase.md`
