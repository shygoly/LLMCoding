---
type: decision
status: accepted
owner: user
date: 2026-03-16
tags: [knowledge, methodology]
---

# 001-use-knowledge-repo

## 决策
建立独立的 `knowledge/` 目录，用于沉淀 AI 软件工程相关的元方法、模式与决策。

## 背景
当前已有大量关于自动化编程、PKG、UI Schema、Agent 架构的讨论，但内容分散，难以持续提炼和迭代。

## 备选方案
- 继续保留在聊天记录中
- 直接写进单一 README
- 建立结构化知识目录

## 选择原因
结构化目录更适合长期维护，也更方便后续用 OpenClaw 做提炼、归类、回顾和修订。

## 收益
- 沉淀原始想法与提炼结果
- 支持方法、模式、决策分层组织
- 便于逐步演化为稳定的方法库

## 代价与风险
- 需要持续维护命名与模板一致性
- 如果缺少回顾机制，仍会逐渐失序

## 后续动作
- 补充首批方法与模式文档
- 增加 `AGENTS.md` 约束提炼流程
- 建立周期性 review 机制

## 相关文档
- `knowledge/README.md`
- `knowledge/templates/method-template.md`
