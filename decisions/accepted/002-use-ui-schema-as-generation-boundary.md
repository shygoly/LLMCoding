---
type: decision
status: accepted
owner: user
date: 2026-03-16
tags: [ui-schema, codegen, taro, design-system]
---

# 002-use-ui-schema-as-generation-boundary

## 决策
在自动生成页面系统中，采用 `UI Schema / Page Schema` 作为设计输入与目标代码之间的标准边界，不直接使用 `HTML -> Taro` 作为长期主链路。

## 背景
当前讨论涉及从 Stitch 或类似设计工具导出 HTML，再进一步生成 Taro 页面、Mock API 和最终的 Supabase 接口。直接从 HTML 硬转 Taro 虽然能快速出结果，但难以维护、难以重生成、也难以稳定接入数据与交互语义。

## 备选方案
- 方案 A：直接 `HTML -> Taro`
- 方案 B：`Design / HTML -> UI Schema -> Taro`
- 方案 C：完全手写页面，不做中间层抽象

## 选择原因
采用 Schema 中间层后，系统可以显式表达组件树、数据位、动作、资源引用与页面结构，使生成器、Mock API、真实 API 对接都建立在统一语义上。相比直接转译 HTML，这种方式更适合长期演进和重复生成。

## 收益
- 降低直接 HTML 转译造成的脆弱性
- 提升跨端生成与页面重生成能力
- 更容易自动生成 Mock API 和真实数据接入层
- 更适合后续加入事件语义、状态管理和组件规范

## 代价与风险
- 需要额外维护 Schema 规范
- 需要持续迭代解析器与生成器
- 如果设计输入缺少语义信息，Schema 抽取质量会受限

## 后续动作
- 补充 `UI Schema` 的字段规范草案
- 为页面生成链路增加 Schema 校验步骤
- 统一页面层只依赖 API 接口，不直接绑定具体数据源

## 相关文档
- `knowledge/methods/design-to-schema-to-code.md`
- `knowledge/patterns/html-to-ui-schema.md`
- `knowledge/patterns/mock-api-to-supabase.md`
- `knowledge/inbox/2026-03-16-ai-software-factory-notes.md`
