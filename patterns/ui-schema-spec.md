---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [ui-schema, spec, codegen, taro, frontend]
---

# UI Schema Spec

## 问题
如果没有统一的 Schema 规范，设计产物、HTML、生成器、Mock API 与目标端代码之间就无法形成稳定边界，页面生成链路会迅速退化为一次性脚本和大量手工修补。

## 上下文
该规范用于连接 `Design / HTML / Design Nodes` 与 `Taro / React / Web` 等目标代码，也用于支撑 Mock API 生成、真实 API 对接、页面重生成和后续 Agent 自动化。

## 目标
定义一个最小但可扩展的 `UI Schema / Page Schema`，使页面生成链路可以稳定地表达：
- 页面元信息
- 组件树
- 数据需求
- 动作与事件
- 资源引用
- 布局与样式线索

## 设计原则
- Schema 优先表达语义，不直接绑定某个前端框架
- 页面结构、数据需求、动作语义分层表示
- 字段命名稳定，可被生成器和校验器消费
- 允许保留原始设计信息，便于回溯和调试
- 先覆盖高频页面，再逐步扩展复杂交互

## 顶层结构
推荐的顶层结构如下：

```json
{
  "version": "0.1",
  "page": {
    "id": "school-recommend",
    "name": "SchoolRecommend",
    "route": "/pages/school/recommend",
    "title": "院校推荐"
  },
  "meta": {
    "platformTargets": ["taro"],
    "source": "html",
    "sourceRef": "design.html"
  },
  "state": [],
  "dataSources": [],
  "actions": [],
  "resources": [],
  "components": []
}
```

## 字段规范

### 1. `version`
Schema 版本号，用于后续演进与兼容处理。

### 2. `page`
页面基础信息。

建议字段：
- `id`：页面唯一标识
- `name`：页面名，便于生成组件或文件名
- `route`：目标路由
- `title`：页面标题
- `description`：可选，页面说明

### 3. `meta`
记录来源与生成上下文。

建议字段：
- `platformTargets`：目标平台列表，如 `taro`、`react-web`
- `source`：来源类型，如 `html`、`figma`、`stitch`
- `sourceRef`：来源引用，如文件名、节点 ID
- `generatorHints`：生成器提示，如样式策略、组件映射策略

### 4. `state`
记录页面本地状态，不等同于远程数据源。

示例：

```json
[
  {
    "name": "activeTab",
    "type": "string",
    "initial": "all"
  }
]
```

建议字段：
- `name`
- `type`
- `initial`
- `scope`：可选，如 `page`、`component`

### 5. `dataSources`
描述页面所需的远程或本地数据来源。

示例：

```json
[
  {
    "name": "schools",
    "kind": "api",
    "fetch": {
      "method": "getSchools",
      "params": []
    },
    "shape": "School[]"
  }
]
```

建议字段：
- `name`：数据源名
- `kind`：如 `api`、`mock`、`static`
- `fetch`：拉取方式
- `shape`：数据结构描述
- `bindTo`：绑定到哪些组件或字段

### 6. `actions`
描述页面级动作与事件处理语义。

示例：

```json
[
  {
    "id": "openSchool",
    "kind": "navigate",
    "target": "/pages/school/detail",
    "params": ["schoolId"]
  }
]
```

建议字段：
- `id`
- `kind`：如 `navigate`、`submit`、`toggle`、`callApi`
- `target`
- `params`
- `effects`：可选，描述状态变更或副作用

### 7. `resources`
描述图片、图标、媒体等资源。

示例：

```json
[
  {
    "id": "banner",
    "kind": "image",
    "source": "local",
    "path": "assets/banner.png"
  }
]
```

建议字段：
- `id`
- `kind`
- `source`
- `path`
- `uploadTo`：可选，如 `supabase-storage`

### 8. `components`
页面组件树，是生成器的核心输入。

每个组件建议包含：
- `id`：组件实例 ID
- `type`：组件类型，如 `view`、`text`、`button`、`list`、`card`
- `props`：静态属性
- `bindings`：动态数据绑定
- `events`：事件到动作的映射
- `style`：样式线索
- `children`：子组件数组

示例：

```json
[
  {
    "id": "school-list",
    "type": "list",
    "bindings": {
      "data": "schools"
    },
    "children": [
      {
        "id": "school-item",
        "type": "card",
        "repeat": {
          "item": "school",
          "source": "schools"
        },
        "children": [
          {
            "id": "school-name",
            "type": "text",
            "bindings": {
              "text": "school.name"
            }
          },
          {
            "id": "detail-button",
            "type": "button",
            "props": {
              "text": "查看详情"
            },
            "events": {
              "click": {
                "action": "openSchool",
                "args": {
                  "schoolId": "school.id"
                }
              }
            }
          }
        ]
      }
    ]
  }
]
```

## 最小可用组件类型
第一版建议只支持高频基础类型：
- `view`
- `text`
- `image`
- `button`
- `input`
- `list`
- `card`
- `section`

复杂组件优先通过组合表达，而不是一开始就扩展过多专有类型。

## 数据绑定规范
建议区分三类绑定：
- `props`：静态值
- `bindings`：动态值
- `events`：动作映射

例如：
- 静态文案放 `props.text`
- 动态文案放 `bindings.text`
- 点击行为放 `events.click`

这样更利于生成器和校验器处理。

## 生成约束
基于该 Schema 的生成器至少应支持：
- 生成页面文件
- 生成样式文件或样式映射
- 生成页面配置与路由信息
- 生成 Mock API 骨架
- 生成资源清单

## 校验规则
第一版建议至少校验：
- `page.id` 不为空
- `components` 为数组且每个节点有 `id` 与 `type`
- `events.action` 必须指向已定义动作
- `bindings` 引用的数据源必须存在
- `repeat.source` 必须指向合法数据源

## 演进策略
- `v0.1`：支持静态页面与简单列表页面
- `v0.2`：支持表单、条件渲染、基础状态切换
- `v0.3`：支持更复杂的交互流与跨页面共享组件

## 不适用场景
- 高度动画化、强时序驱动的复杂交互页面
- 完全一次性且无需重生成的静态页面
- 需要框架专有能力且难以抽象的极端页面

## 相关文档
- `knowledge/principles/schema-first-generation.md`
- `knowledge/decisions/accepted/002-use-ui-schema-as-generation-boundary.md`
- `knowledge/methods/design-to-schema-to-code.md`
- `knowledge/patterns/html-to-ui-schema.md`
- `knowledge/patterns/mock-api-to-supabase.md`
