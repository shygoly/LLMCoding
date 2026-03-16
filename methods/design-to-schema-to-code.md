---
type: method
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [ui, schema, taro, codegen]
---

# Design To Schema To Code

## 目标
把设计稿或导出的 HTML 材料，稳定转化为可维护的页面代码，而不是直接做脆弱的 HTML 到目标框架硬转译。

## 适用场景
- 需要从设计工具生成页面骨架
- 目标端是 Taro、React、Web 或其他组件化 UI
- 后续还要接入 Mock API 与真实 API

## 前提条件
- 已获得设计稿导出结果，如 HTML、CSS、图片或设计节点数据
- 项目允许引入中间层 Schema
- 有页面生成器或代码模板

## 输入
- 设计稿或 HTML
- 样式信息
- 组件命名约定
- 页面数据需求

## 核心步骤
1. 从设计稿或 HTML 中抽取结构、文案、图片与交互线索
2. 归一化为 `Page Schema`，明确组件树、数据位、动作与资源引用
3. 基于 Schema 生成目标端代码，如 Taro 页面、样式与配置
4. 根据 Schema 中的数据需求自动生成 Mock API
5. 页面稳定后，把 Mock API 替换为真实后端或 Supabase 接口

## 产物
- 页面级 Schema
- 目标端页面代码
- Mock API 文件
- 后续真实 API 对接清单

## 风险与边界
- 直接 HTML 到 Taro 的转换维护成本高，不建议作为长期方案
- 如果设计稿缺少语义信息，Schema 抽取质量会明显下降
- 复杂业务交互仍需人工补充状态管理与事件语义

## 验证方式
- 同一类页面能重复生成且结构稳定
- UI 修改后可通过重新生成而非手工大改来完成同步
- Mock API 与真实 API 切换成本可控

## 相关文档
- `knowledge/patterns/html-to-ui-schema.md`
- `knowledge/patterns/mock-api-to-supabase.md`
