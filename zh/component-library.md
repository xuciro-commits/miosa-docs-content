---
slug: "component-library"
title: "基础组件库"
description: "MIOSA Foundation UI 参考，包含设计意图、Tailwind v4 令牌、CSS 变量和设计令牌。"
section: "Platform builders"
order: 9020
---

MIOSA Foundation 是平台前端的内部组件库。它是在决定保留、删除、重命名或重建哪些组件之前检查组件的地方。

这是用于构建 MIOSA 本身的产品 UI 系统，而不是客户导入以在自己的应用程序中嵌入沙箱的嵌入式 `@miosa/*` 包 - 这些包记录在 [UI 组件 ](/docs/ui-components) 中。

实时浏览器在 Foundation 包中维护：

```bash
cd miosa-platform/miosa-frontend/src/lib/foundations-components
npm run dev -- --host 127.0.0.1 --port 5174
```

打开：

```text
http://127.0.0.1:5174/component-library
```

> **注意：当前意图**
>
> 该库正在从一个广泛的原型集清理为一个小型的、可重复的 MIOSA UI 系统。将此页面视为该清理工作的设计合同。

## 什么住在哪里

- [实时组件浏览器](http://127.0.0.1:5174/component-library)：浏览 Foundation 包中的每个部分、模式和演示。
- **源部分**：`src/lib/foundations-components/src/routes/component-library/sections`
- **设计参考**：`src/lib/foundations-components/DESIGN.md`
- **清理审查**：`src/lib/foundations-components/COMPONENT_REVIEW.md`

## 紧凑参考

MIOSA UI 应该像一个用于 AI 计算的操作员控制台：精确、黑暗、密集且扫描速度快。保持系统基本中立。仅对活动计算、选定状态、焦点和实时状态使用有限的蓝色/青色信号颜色。避免装饰性渐变、过大的营销部分、重复的按钮变体和一次性的组件样式。

|代币|价值|角色 |
|---|---:|---|
|帆布| `#07090d` |应用背景 |
|面板| `#10141c` |主要部件表面 |
|提出| `#151b25` |提升的控件和选定的行 |
|边境| `#263041` |发丝结构|
|文字| `#f4f7fb` |主要文本|
|静音 | `#8d98aa` |二级标签|
|信号| `#60a5fa` |主动计算、焦点、选定状态 |
|成功| `#34d399` |健康/跑步状态 |
|警告| `#fbbf24` |注意状态|
|危险| `#f87171` |破坏性或失败状态 |

核心规则：

- 每个视觉区域使用一个主按钮。
- 喜欢紧凑的工具栏、分段控件、选项卡、表格和分割窗格。
- 使用 1px 边框进行结构；谨慎使用阴影。
- 仅在必须快速读取状态的情况下使用状态颜色。
- 将牌保持在 `8px` 半径处，除非现有的本地模式需要更少的半径。
- 删除重复的按钮/组件变体，除非它们编码含义。

## 扩展参考

**设计.md**

```md
# MIOSA Foundation UI

Theme: dark operator console

MIOSA Foundation is a component system for AI compute, white-label workspaces,
sandboxes, deployments, file browsers, agents, and usage operations. It should
feel like a precise product surface, not a marketing page.

## Design Character

The UI is compact, structured, and status-rich. Components should look like
they belong in the same control plane: split panes, command bars, tables,
logs, terminals, inspectors, file trees, detail drawers, and focused panels.

The component library should not keep every prototype. It should converge on
the smallest set of primitives and domain components that can compose the real
MIOSA product.

## Keep

- Buttons with clear hierarchy: primary, secondary, ghost, icon, destructive.
- Inputs, selects, textareas, checkboxes, switches, sliders, tabs, menus.
- Tables and data grids for operational records.
- Split panes for preview + files, terminal + logs, list + detail.
- Status badges for sandbox, deployment, compute, billing, and security states.
- Drawers and dialogs for contained workflows.
- Command palette and Ask OSA controls.
- File tree, terminal, preview iframe, usage summary, and deployment panels.

## Remove Or Merge

- Duplicate button families that only differ visually.
- Decorative card variants with no product role.
- Repeated chat/input shells that should become one canonical agent composer.
- Demo-only modules that do not map to MIOSA product workflows.
- Any component that requires a special color, radius, or font rule by itself.

## Component Quality Bar

Each kept component needs:

- A real product role.
- A stable API.
- Keyboard/focus behavior where relevant.
- Loading, empty, error, disabled, and selected states.
- A single source of styling truth.
- No hidden dependency on app-only stores unless wrapped behind local adapters.
```

**顺风 v4**

```css
@theme {
  /* Colors */
  --color-miosa-canvas: #07090d;
  --color-miosa-panel: #10141c;
  --color-miosa-raised: #151b25;
  --color-miosa-border: #263041;
  --color-miosa-border-soft: #1c2533;
  --color-miosa-text: #f4f7fb;
  --color-miosa-muted: #8d98aa;
  --color-miosa-dim: #5f6b7d;
  --color-miosa-signal: #60a5fa;
  --color-miosa-signal-soft: #1d4ed8;
  --color-miosa-success: #34d399;
  --color-miosa-warning: #fbbf24;
  --color-miosa-danger: #f87171;

  /* Typography */
  --font-miosa-sans: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --font-miosa-mono: "SF Mono", "Cascadia Code", ui-monospace, monospace;

  --text-miosa-caption: 12px;
  --text-miosa-body: 14px;
  --text-miosa-ui: 13px;
  --text-miosa-title: 18px;
  --text-miosa-display: 40px;

  /* Shape */
  --radius-miosa-xs: 4px;
  --radius-miosa-sm: 6px;
  --radius-miosa-md: 8px;
  --radius-miosa-pill: 9999px;

  /* Spacing */
  --spacing-miosa-1: 4px;
  --spacing-miosa-2: 8px;
  --spacing-miosa-3: 12px;
  --spacing-miosa-4: 16px;
  --spacing-miosa-6: 24px;
  --spacing-miosa-8: 32px;
}
```

**CSS 变量**

```css
:root {
  color-scheme: dark;

  --miosa-canvas: #07090d;
  --miosa-panel: #10141c;
  --miosa-raised: #151b25;
  --miosa-border: #263041;
  --miosa-border-soft: #1c2533;
  --miosa-text: #f4f7fb;
  --miosa-muted: #8d98aa;
  --miosa-dim: #5f6b7d;
  --miosa-signal: #60a5fa;
  --miosa-signal-soft: #1d4ed8;
  --miosa-success: #34d399;
  --miosa-warning: #fbbf24;
  --miosa-danger: #f87171;

  --miosa-font-sans: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --miosa-font-mono: "SF Mono", "Cascadia Code", ui-monospace, monospace;

  --miosa-text-caption: 12px;
  --miosa-text-ui: 13px;
  --miosa-text-body: 14px;
  --miosa-text-title: 18px;
  --miosa-text-display: 40px;

  --miosa-leading-tight: 1.2;
  --miosa-leading-ui: 1.45;
  --miosa-leading-body: 1.55;

  --miosa-radius-xs: 4px;
  --miosa-radius-sm: 6px;
  --miosa-radius-md: 8px;
  --miosa-radius-pill: 9999px;

  --miosa-space-1: 4px;
  --miosa-space-2: 8px;
  --miosa-space-3: 12px;
  --miosa-space-4: 16px;
  --miosa-space-6: 24px;
  --miosa-space-8: 32px;
}
```

**设计代币**

```json
{
  "color": {
    "canvas": { "$value": "#07090d", "$type": "color" },
    "panel": { "$value": "#10141c", "$type": "color" },
    "raised": { "$value": "#151b25", "$type": "color" },
    "border": { "$value": "#263041", "$type": "color" },
    "border-soft": { "$value": "#1c2533", "$type": "color" },
    "text": { "$value": "#f4f7fb", "$type": "color" },
    "muted": { "$value": "#8d98aa", "$type": "color" },
    "dim": { "$value": "#5f6b7d", "$type": "color" },
    "signal": { "$value": "#60a5fa", "$type": "color" },
    "signal-soft": { "$value": "#1d4ed8", "$type": "color" },
    "success": { "$value": "#34d399", "$type": "color" },
    "warning": { "$value": "#fbbf24", "$type": "color" },
    "danger": { "$value": "#f87171", "$type": "color" }
  },
  "font": {
    "sans": { "$value": "Inter", "$type": "fontFamily" },
    "mono": { "$value": "SF Mono", "$type": "fontFamily" }
  },
  "typography": {
    "caption": {
      "$type": "typography",
      "$value": { "fontFamily": "Inter", "fontSize": "12px", "fontWeight": 500, "lineHeight": 1.45 }
    },
    "ui": {
      "$type": "typography",
      "$value": { "fontFamily": "Inter", "fontSize": "13px", "fontWeight": 500, "lineHeight": 1.45 }
    },
    "body": {
      "$type": "typography",
      "$value": { "fontFamily": "Inter", "fontSize": "14px", "fontWeight": 400, "lineHeight": 1.55 }
    },
    "title": {
      "$type": "typography",
      "$value": { "fontFamily": "Inter", "fontSize": "18px", "fontWeight": 600, "lineHeight": 1.2 }
    }
  },
  "radius": {
    "xs": { "$value": "4px", "$type": "dimension" },
    "sm": { "$value": "6px", "$type": "dimension" },
    "md": { "$value": "8px", "$type": "dimension" },
    "pill": { "$value": "9999px", "$type": "dimension" }
  },
  "spacing": {
    "1": { "$value": "4px", "$type": "dimension" },
    "2": { "$value": "8px", "$type": "dimension" },
    "3": { "$value": "12px", "$type": "dimension" },
    "4": { "$value": "16px", "$type": "dimension" },
    "6": { "$value": "24px", "$type": "dimension" },
    "8": { "$value": "32px", "$type": "dimension" }
  }
}
```

## 组件清理清单

在决定基金会库中保留哪些内容时，请使用此清单。

|决定|保留时间 |删除或合并何时 |
|---|---|---|
|按钮变体 |它映射到真实的操作层次结构|它仅改变颜色、半径或氛围 |
|卡/面板 |它代表了可重复使用的产品表面|这是一个一次性的演示包装器 |
|输入/作曲家|它处理真实的数据输入或智能体提示|它复制了另一个 Composer shell |
|表格/列表|它支持扫描、排序、比较或操作工作流程 |它只是视觉填充物|
|状态徽章|它对真实状态进行编码|它是装饰性标签|
|模块页面 |它映射到 MIOSA 产品工作流程 |这是一个通用的 SaaS 模型 |

## 另请参阅

- [UI 组件概述](/docs/ui-components)
- [React UI 组件](/docs/ui-components/react)
- [Svelte UI 组件](/docs/ui-components/svelte)
- [Vue UI 组件](/docs/ui-components/vue)
