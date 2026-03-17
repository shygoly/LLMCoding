# AI 软件工程知识库

这个目录用于沉淀自动化编程、AI Agent、PKG、UI 生成等主题的元方法、模式与决策。

## 目录约定

- `inbox/`：原始想法、对话摘录、待提炼材料
- `methods/`：可执行的方法论，强调步骤、边界、产物
- `patterns/`：可复用架构模式与设计套路
- `decisions/`：已确认或已否决的架构决策
- `prompts/`：用于提炼、归类、回顾的固定提示词
- `principles/`：跨主题通用工程原则
- `reviews/`：周期性回顾、修订、冲突检查
- `templates/`：标准文档模板
- `architecture/`：平台级与系统级架构蓝图

## 工作流

```text
inbox -> methods/patterns -> decisions -> reviews
           \-> principles
```

## 记录原则

1. 原始材料与提炼结论分离
2. 每篇文档都写清适用边界
3. 决策必须记录原因、收益、代价
4. 方法必须包含可执行步骤
5. 新结论优先链接旧文档，避免重复造轮子

## 命名建议

- 方法：`verb-noun.md`
- 模式：`domain-pattern.md`
- 决策：`NNN-short-name.md`
- 回顾：`YYYY-MM-DD-topic.md`

## 推荐起步文档

- `architecture/ai-application-os.md`
- `architecture/skill-graph-vs-tool-catalog.md`
- `architecture/memory-vs-rag.md`
- `architecture/policy-engine-for-ai-saas.md`
- `architecture/workflow-engine-for-ai-agents.md`
- `architecture/multi-tenant-ai-platform.md`
- `architecture/billing-and-usage-for-ai-saas.md`
- `architecture/audit-log-for-ai-agents.md`
- `methods/design-to-schema-to-code.md`
- `methods/project-knowledge-graph.md`
- `methods/spec-driven-cursor-supervision.md`
- `methods/mvp-supervisor-implementation-plan.md`
- `patterns/html-to-ui-schema.md`
- `patterns/ui-schema-spec.md`
- `patterns/obsidian-as-decision-graph.md`
- `patterns/supervisor-subagent-with-shared-store.md`
- `patterns/supervisor-task-payload-spec.md`
- `patterns/cursor-result-payload-spec.md`
- `patterns/runtime-state-schema.md`
- `patterns/supervisor-review-rules.md`
- `patterns/sqlite-ddl-for-runtime-state.md`
- `patterns/visual-spec-plus-ui-schema.md`
- `patterns/visual-spec-spec.md`
- `patterns/reconcile-stage-spec.md`
- `patterns/reconcile-rules-for-common-components.md`
- `patterns/reconcile-warning-types.md`
- `patterns/reconcile-summary-spec.md`
- `decisions/accepted/001-use-knowledge-repo.md`
- `decisions/accepted/005-use-dual-track-generation-for-stitch-to-code.md`
- `principles/schema-first-generation.md`
- `principles/agent-query-before-code.md`
- `principles/shared-state-over-session-memory.md`
- `principles/fidelity-and-structure-dual-track.md`
- `principles/domain-driven-design.md`
- `principles/test-driven-development.md`
- `prompts/pre-code-query-checklist.md`
- `reviews/weekly/2026-03-16-review-example.md`
