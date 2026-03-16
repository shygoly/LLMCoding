---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [mock, api, supabase, frontend]
---

# Mock API To Supabase

## 问题
页面开发早期需要稳定数据接口，但真实后端尚未完成；后期又希望低成本切换到真实数据源。

## 上下文
适用于 AI 生成前端页面、先做 UI 联调、后接 Supabase 或其他真实服务端的场景。

## 结构
- 页面层：只依赖统一的 API 调用入口
- Mock 层：返回固定或可配置测试数据
- Real 层：调用 Supabase 或真实后端
- 切换层：统一导出或环境切换策略

## 实施方式
1. 先根据页面 Schema 的数据需求生成 Mock API
2. 页面开发期间只消费统一 API 接口，不直接写死数据来源
3. 后端准备好后，用 Supabase 实现替换 Mock 实现
4. 保持数据结构尽量一致，减少页面层修改

## 优点
- UI 开发与后端开发解耦
- 便于 AI 自动生成后立即运行页面
- 切换真实服务成本较低

## 代价
- Mock 数据若长期不更新，可能与真实结构漂移
- 需要对 API 形状进行最小规范化

## 不适用场景
- 页面与数据强耦合、实时逻辑复杂且无法模拟的系统

## 相关文档
- `knowledge/methods/design-to-schema-to-code.md`
