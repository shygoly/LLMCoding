---
type: pattern
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [reconcile, summary, review, follow-up, protocol]
---

# Reconcile Summary Spec

## 问题
如果 `Reconcile` 阶段没有稳定的摘要输出格式，那么即使已经完成了视觉与工程的对齐，后续的 review、warning 汇总、follow-up task 生成和质量分析仍然会依赖临时文本总结，难以形成可追踪的自动化闭环。

## 上下文
该规范用于定义 `Reconcile` 阶段完成后的结构化摘要格式。它应能连接以下流程：
- 对齐结果审查
- warning 汇总
- follow-up task 生成
- 质量回顾与统计

## 目标
让每次 Reconcile 至少能稳定回答：
- 用了哪些输入
- 完成了哪些对齐动作
- 保留了哪些视觉信息
- 注入了哪些工程结构
- 发现了哪些 warning
- 是否需要 follow-up

## 设计原则
- 摘要应结构化，而不是只是一段自然语言
- 摘要既要支持人读，也要支持自动消费
- 摘要不重复存储大对象，只存关键结果与引用
- warning、风险、后续动作要显式化

## 顶层结构
推荐使用 JSON：

```json
{
  "pageId": "school-recommend",
  "status": "completed_with_warnings",
  "inputs": {},
  "alignment": {},
  "preservation": {},
  "injection": {},
  "warnings": [],
  "followUps": [],
  "artifacts": {},
  "summary": "..."
}
```

## 字段规范

### 1. `pageId`
当前页面或任务对应的页面标识。

### 2. `status`
对齐结果状态建议。

推荐值：
- `completed`
- `completed_with_warnings`
- `needs_review`
- `partial`

### 3. `inputs`
说明本次对齐使用了哪些输入。

建议字段：

```json
{
  "visualSpec": "patterns/visual-spec-spec.md",
  "uiSchema": "patterns/ui-schema-spec.md",
  "draftCode": "artifacts/draft/page.tsx",
  "rules": [
    "patterns/reconcile-rules-for-common-components.md"
  ]
}
```

### 4. `alignment`
记录本次执行了哪些对齐动作。

建议字段：

```json
{
  "componentsProcessed": ["text", "button", "list"],
  "rewrites": [
    "static_text_to_binding",
    "static_cards_to_list_render"
  ],
  "componentMappings": [
    {
      "visualRef": "school-card-visual",
      "schemaRef": "school-card",
      "result": "mapped"
    }
  ]
}
```

### 5. `preservation`
记录哪些视觉信息被保留。

建议字段：
- `layoutPreserved`
- `stylesPreserved`
- `assetsPreserved`
- `notes`

示例：

```json
{
  "layoutPreserved": true,
  "stylesPreserved": ["card radius", "title typography"],
  "assetsPreserved": ["banner-1"]
}
```

### 6. `injection`
记录注入了哪些工程结构。

建议字段：
- `bindings`
- `actions`
- `state`
- `runtime`

示例：

```json
{
  "bindings": ["school.name", "school.score"],
  "actions": ["openSchool"],
  "state": [],
  "runtime": ["getSchools"]
}
```

### 7. `warnings`
记录 warning 列表。

要求：
- 结构应与 `Reconcile Warning Types` 保持兼容
- 每条 warning 尽量附带 `nodeRef` 和 `suggestedAction`

### 8. `followUps`
记录建议生成的后续任务。

示例：

```json
[
  {
    "type": "knowledge-update",
    "target": "knowledge/patterns/ui-schema-spec.md",
    "reason": "新增了 list.repeat.source 的对齐策略"
  }
]
```

### 9. `artifacts`
记录对齐阶段产物。

建议字段：
- `finalCodePath`
- `summaryPath`
- `diffPath`
- `reportPath`

### 10. `summary`
一段简短自然语言摘要，供人快速理解结果。

要求：
- 不代替结构化字段
- 只做简明说明

## 最小可用示例

```json
{
  "pageId": "school-recommend",
  "status": "completed_with_warnings",
  "inputs": {
    "visualSpec": "artifacts/visual-spec.json",
    "uiSchema": "artifacts/ui-schema.json",
    "draftCode": "artifacts/draft/page.tsx",
    "rules": [
      "knowledge/patterns/reconcile-rules-for-common-components.md"
    ]
  },
  "alignment": {
    "componentsProcessed": ["text", "button", "list", "card"],
    "rewrites": [
      "static_text_to_binding",
      "static_cards_to_list_render"
    ],
    "componentMappings": [
      {
        "visualRef": "school-card-visual",
        "schemaRef": "school-card",
        "result": "mapped"
      }
    ]
  },
  "preservation": {
    "layoutPreserved": true,
    "stylesPreserved": ["card radius", "title typography"],
    "assetsPreserved": ["banner-1"]
  },
  "injection": {
    "bindings": ["school.name", "school.score"],
    "actions": ["openSchool"],
    "state": [],
    "runtime": ["getSchools"]
  },
  "warnings": [
    {
      "code": "asset_reference_unresolved",
      "level": "warning",
      "message": "banner 资源尚未迁移到统一资源层",
      "nodeRef": "banner-1",
      "suggestedAction": "normalize_asset_reference"
    }
  ],
  "followUps": [
    {
      "type": "asset-normalization",
      "target": "banner-1",
      "reason": "资源引用尚未标准化"
    }
  ],
  "artifacts": {
    "finalCodePath": "runtime/artifacts/page.tsx",
    "summaryPath": "runtime/artifacts/reconcile-summary.json"
  },
  "summary": "已完成页面主要结构对齐，保留核心视觉样式并注入数据绑定，仍有资源引用待标准化。"
}
```

## 与 warning / review 的关系
- `warnings` 字段直接承载 warning 分类结果
- `followUps` 可被 Supervisor 或任务系统转换为后续 task
- `status` 可被 review 阶段用作初始判断参考，但不是最终审查结论

## 优点
- 让 Reconcile 阶段输出具备协议化特征
- 便于后续自动 review、统计和质量回顾
- 便于把双轨生成和任务系统接起来

## 代价
- 需要维护摘要结构与 warning 结构的兼容性
- 若字段设计过多，第一版实现成本会升高

## 不适用场景
- 完全手工做一次性页面合成
- 不需要后续 review 或 follow-up 的简单实验

## 相关文档
- `knowledge/patterns/reconcile-stage-spec.md`
- `knowledge/patterns/reconcile-warning-types.md`
- `knowledge/patterns/supervisor-review-rules.md`
- `knowledge/decisions/accepted/005-use-dual-track-generation-for-stitch-to-code.md`
