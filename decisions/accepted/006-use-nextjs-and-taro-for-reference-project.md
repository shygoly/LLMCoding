---
type: decision
status: accepted
owner: user
date: 2026-03-17
tags: [tech-stack, nextjs, taro, demo, enterprise-ai]
---

# 006-use-nextjs-and-taro-for-reference-project

## 决策
在企业 AI 应用示范项目中，采用以下前端与应用层技术路线：
- `Next.js + TypeScript` 作为 Web / SaaS 管理端主栈
- `Taro + TypeScript + miniprogram-cli` 作为微信小程序端主栈

## 背景
示范项目需要同时覆盖：
- Web 控制台与管理后台
- 企业知识助手或 AI 应用的移动端入口
- 后续可接多租户、权限、审计、usage / billing、workflow 等平台能力

同时，系统还在探索：
- `UI Schema / Visual Spec / Reconcile` 的双轨生成
- 多端复用 shared types / api sdk / schema 的可能性

## 备选方案
- 方案 A：只做 Web，不做小程序
- 方案 B：Web 与 MiniApp 都围绕原生栈独立开发
- 方案 C：`Next.js` 负责 Web，`Taro` 负责小程序，并共享部分平台资产

## 选择原因
方案 C 在当前阶段最平衡：
- Web 端适合展示平台化与管理能力
- 小程序端适合业务入口与轻交互场景
- 两端都可基于 TypeScript、shared types 和 schema 做协同

## 收益
- 同时覆盖企业后台与移动入口
- 更适合展示“企业 AI 应用 + 平台骨架”的完整形态
- 为后续多端生成和共享能力打下基础

## 代价与风险
- 双端项目结构更复杂
- 需要明确 Web 与 MiniApp 的能力边界
- 小程序构建和发布链路需要额外维护

## 后续动作
- 固化参考技术栈文档
- 设计 monorepo 结构
- 明确 Web / MiniApp 的能力分工
- 让 `UI Schema / Visual Spec` 对多端生成更友好

## 相关文档
- `knowledge/architecture/reference-tech-stack-for-enterprise-ai-demo.md`
- `knowledge/decisions/accepted/005-use-dual-track-generation-for-stitch-to-code.md`
- `knowledge/patterns/ui-schema-spec.md`
- `knowledge/patterns/visual-spec-spec.md`
