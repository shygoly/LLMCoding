---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [visual-spec, spec, stitch, fidelity, frontend]
---

# Visual Spec Spec

## 问题
如果双轨生成里只有 `UI Schema` 有正式规范，而视觉轨只有模糊描述，那么高保真生成仍然会依赖即时 prompt 和临场理解，难以稳定复原 Stitch 中的布局、样式和层次细节。

## 上下文
该规范用于定义 `Visual Spec` 的最小结构，使 Stitch 或类似设计源中的视觉信息可以被稳定提取、存储和复用，并作为高保真代码草稿生成的输入。

## 目标
定义一个最小但可扩展的 `Visual Spec`，让系统能够表达：
- 页面整体视觉信息
- 布局树
- 样式 token
- 组件视觉属性
- 资源位置与展示方式
- 与工程层对齐所需的视觉标识

## 设计原则
- 优先保留视觉真相，而不是过早抽象成工程组件
- 表达“看起来是什么样”而不是“数据怎么流动”
- 与 `UI Schema` 分工明确，但保留可对齐字段
- 字段命名尽量稳定，适合 LLM 和生成器共同消费
- 第一版先覆盖最常见的页面布局与组件视觉属性

## 顶层结构
推荐使用如下结构：

```json
{
  "version": "0.1",
  "page": {
    "id": "school-recommend",
    "name": "SchoolRecommend"
  },
  "meta": {
    "source": "stitch",
    "sourceRef": "node-123",
    "viewport": {
      "width": 375,
      "height": 812
    }
  },
  "tokens": {},
  "layoutTree": [],
  "assets": [],
  "componentVisuals": []
}
```

## 字段规范

### 1. `version`
视觉规范版本号。

### 2. `page`
页面标识信息。

建议字段：
- `id`
- `name`
- `title`：可选

### 3. `meta`
来源与画布信息。

建议字段：
- `source`：如 `stitch`、`figma`、`html-snapshot`
- `sourceRef`：节点 ID、文件名或页面引用
- `viewport.width`
- `viewport.height`
- `background`：页面整体背景色，可选

### 4. `tokens`
记录全局视觉 token。

建议拆分：
- `colors`
- `typography`
- `spacing`
- `radius`
- `shadow`
- `border`

示例：

```json
{
  "colors": {
    "primary": "#1677ff",
    "cardBg": "#ffffff",
    "textPrimary": "#1f1f1f"
  },
  "spacing": {
    "sm": 8,
    "md": 12,
    "lg": 16
  },
  "radius": {
    "card": 12,
    "button": 8
  }
}
```

### 5. `layoutTree`
描述页面布局层级，是视觉轨的核心。

每个节点建议包含：
- `id`
- `type`：如 `frame`、`group`、`stack`、`text`、`image`
- `name`
- `frame`：位置信息
- `layout`：布局策略
- `style`：节点级样式
- `children`

示例：

```json
[
  {
    "id": "root-frame",
    "type": "frame",
    "name": "page-root",
    "frame": {
      "x": 0,
      "y": 0,
      "width": 375,
      "height": 812
    },
    "layout": {
      "direction": "vertical",
      "gap": 12,
      "padding": [16, 16, 16, 16]
    },
    "children": []
  }
]
```

### 6. `assets`
记录图片、图标等视觉资源。

建议字段：
- `id`
- `kind`：如 `image`、`icon`
- `source`
- `path`
- `display`

示例：

```json
[
  {
    "id": "banner-1",
    "kind": "image",
    "source": "stitch-export",
    "path": "assets/banner.png",
    "display": {
      "fit": "cover",
      "radius": 12
    }
  }
]
```

### 7. `componentVisuals`
记录可与 `UI Schema` 对齐的组件视觉描述。

建议字段：
- `id`
- `semanticHint`：如 `card`、`button`、`hero`
- `layoutRef`：对应 `layoutTree` 节点
- `textStyle`
- `surfaceStyle`
- `imageStyle`
- `variant`

示例：

```json
[
  {
    "id": "school-card-visual",
    "semanticHint": "card",
    "layoutRef": "card-frame-1",
    "surfaceStyle": {
      "background": "#ffffff",
      "radius": 12,
      "shadow": "sm"
    },
    "textStyle": {
      "title": {
        "fontSize": 18,
        "fontWeight": 600,
        "color": "#1f1f1f"
      }
    }
  }
]
```

## 最小可用视觉属性
第一版建议优先支持：
- 位置与尺寸：`x`、`y`、`width`、`height`
- 布局：`direction`、`gap`、`padding`、`align`
- 文本：`fontSize`、`fontWeight`、`lineHeight`、`color`
- 表面：`background`、`radius`、`border`、`shadow`
- 图片：`fit`、`crop`、`radius`

## 与 UI Schema 的关系
`Visual Spec` 不负责：
- 数据源
- 动作
- 状态
- API

`Visual Spec` 负责：
- 视觉布局
- 视觉样式
- 资源展示
- 与页面视觉相关的层级关系

建议通过以下字段对齐：
- `page.id`
- `componentVisuals.id`
- `semanticHint`
- `layoutRef`

## 生成用途
`Visual Spec` 可用于：
- 驱动高保真静态 UI 草稿生成
- 提供布局与样式参考
- 在 `Reconcile` 阶段辅助视觉与工程结构对齐
- 作为视觉回归基准的一部分

## 优点
- 降低高保真生成对即时 prompt 的依赖
- 保留 Stitch 中最容易在 Schema 抽象中丢失的视觉细节
- 为双轨生成提供正式的视觉侧中间表示

## 代价
- 增加一层新的中间表示
- 需要处理视觉节点与工程组件之间的映射关系

## 不适用场景
- 视觉要求很低的简单后台系统
- 不关心高保真还原的快速内部工具

## 相关文档
- `knowledge/principles/fidelity-and-structure-dual-track.md`
- `knowledge/patterns/visual-spec-plus-ui-schema.md`
- `knowledge/patterns/ui-schema-spec.md`
- `knowledge/decisions/accepted/005-use-dual-track-generation-for-stitch-to-code.md`
