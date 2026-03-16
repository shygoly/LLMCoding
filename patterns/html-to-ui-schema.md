---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [html, ui-schema, taro, codegen]
---

# HTML To UI Schema

## 问题
直接把 HTML 转成目标端代码，通常无法保留清晰的数据边界、动作语义和可维护的组件结构。

## 上下文
常见于 Stitch、设计工具导出页面，或 AI 先生成 HTML，再希望进一步生成 Taro、React 或其他前端页面。

## 结构
- 输入层：HTML、CSS、图片、文案
- 解析层：DOM / AST 解析器
- 中间层：`UI Schema` 或 `Page Schema`
- 生成层：目标框架代码、样式、页面配置
- 数据层：Mock API 与真实 API 映射

## 实施方式
1. 解析 HTML 结构，识别布局块、文本、按钮、列表、图片等语义片段
2. 提取为统一 Schema，显式记录组件树、数据字段、动作与资源
3. 由生成器从 Schema 输出目标框架代码
4. 基于数据字段自动生成 Mock API 与后续真实 API 对接清单

## 优点
- 便于重生成与跨端复用
- 便于挂接 Mock API、状态管理和事件语义
- 降低 HTML 直接转译带来的脆弱性

## 代价
- 需要维护一层 Schema 规范
- 解析器与生成器都需要持续迭代

## 不适用场景
- 一次性静态落地页
- 完全不需要后续维护与数据接入的页面

## 相关文档
- `knowledge/methods/design-to-schema-to-code.md`
- `knowledge/patterns/mock-api-to-supabase.md`
