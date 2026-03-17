---
type: method
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [roadmap, healthcare, ai-assistant, demo, implementation]
---

# Demo Project Roadmap For Healthcare AI Assistant

## 目标
为“健数通 / Homecare Admin 升级为企业 AI 助手 SaaS”提供一条分阶段、可交付、可销售的改造路线，避免一开始就试图同时做完业务改造、AI 能力和平台化能力。

## 适用场景
- 现有健康管理后台改造
- 医疗 / 护理 / 健康管理场景的企业 AI 示范项目
- 希望先做 PoC，再逐步演化为平台化产品

## 前提条件
- 已有基础后台与业务壳
- 已接受“先做业务可见价值，再做平台化补强”的策略
- 已明确第一阶段聚焦低风险辅助场景

## 输入
- 现有 Homecare Admin / 健数通产品
- `reference-enterprise-ai-demo-project.md`
- `transform-homecare-admin-into-ai-saas.md`
- `reference-tech-stack-for-enterprise-ai-demo.md`

## 核心步骤
1. 先做最容易感知价值的知识助手与记录辅助能力
2. 再补多租户、审计、usage 等 SaaS 能力
3. 最后补 workflow、分析层和更强的平台骨架

## Phase 1：AI 能力最小可用版
目标：先让项目有“AI 价值”。

实现内容：
- 文档上传与 tenant-scoped RAG
- 机构知识问答
- 记录总结 / 报告草稿生成
- 基础 AI 聊天界面（Web，可选 MiniApp）

验收标准：
- 能上传机构文档并回答问题
- 回答能给出引用来源
- 能把一段长记录转成结构化摘要

## Phase 2：SaaS 边界增强版
目标：把 AI 功能升级为企业可接受的系统能力。

实现内容：
- 租户边界明确化
- 角色与权限控制
- audit 基础事件记录
- usage 基础计量

验收标准：
- 不同租户文档不串
- 不同角色能力有区别
- 关键提问与 AI 输出可追溯
- 能查看租户级 usage 基础数据

## Phase 3：运营与流程增强版
目标：从知识助手升级为业务助手。

实现内容：
- 风险提示
- workflow task 状态
- 运营分析问答
- 报告模板和导出能力

验收标准：
- 至少一个多步骤任务可视化
- 至少一个运营分析场景可跑通
- 报告草稿可导出或留档

## Phase 4：示范项目商业化版
目标：让项目可销售、可复用、可展示。

实现内容：
- 完整租户演示脚本
- usage / audit / workflow demo 面板
- Web + MiniApp 双端演示
- 销售用 demo 数据与演示账号

验收标准：
- 客户可在 15 分钟内理解产品价值
- 能展示“业务价值 + 平台能力”双重卖点

## 推荐优先级
优先做：
1. tenant-scoped RAG
2. 引用来源问答
3. 记录总结
4. audit 最小日志
5. usage 最小统计

后做：
- 复杂 workflow
- 高级 billing
- 深度 analytics
- 高风险建议闭环

## 风险与边界
- 不要一开始就做高风险诊疗决策
- 不要让业务后台被 AI 功能完全打断原有结构
- 不要同时追求“平台化最完备”和“最快出 demo”

## 验证方式
- 用真实或模拟机构文档跑一次完整问答
- 用一段真实或模拟服务记录跑总结
- 用两个租户验证隔离
- 用管理员视角验证 audit / usage 可见性

## 相关文档
- `knowledge/architecture/transform-homecare-admin-into-ai-saas.md`
- `knowledge/architecture/reference-enterprise-ai-demo-project.md`
- `knowledge/architecture/reference-tech-stack-for-enterprise-ai-demo.md`
