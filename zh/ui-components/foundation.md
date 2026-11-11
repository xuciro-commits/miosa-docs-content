---
slug: "ui-components/foundation"
title: "Foundation UI"
description: "平台应用所使用的 MIOSA 自有前端基础组件。"
section: "Platform builders"
order: 9021
---

MIOSA 平台拥有一套 Foundation UI 基础组件，通过 `$foundation` 别名暴露给 Svelte 应用。

在添加路由本地的按钮、输入框、卡片、模态框、标签页、表格或页面框架之前，平台页面应优先使用这套组件。

## 自有基础组件范围

| 组件分组 | 组件 |
|---|---|
| 操作 | `Button`、`PillButton` |
| 表单 | `Input`、`RoundedInput`、`Textarea`、`Select`、`Checkbox`、`Toggle`、`RadioGroup`、`Slider` |
| 布局 | `GlassCard`、`AppCard`、`Separator`、`ScrollArea`、`Breadcrumb` |
| 反馈 | `Alert`、`Toast`、`Loading`、`Skeleton`、`Progress` |
| 浮层 | `Modal`、`Popover`、`Tooltip`、`Menu` |
| 数据展示 | `Table`、`Tabs`、`Accordion`、`Avatar` |

## 导入路径

```svelte
<script lang="ts">
  import { GlassCard, PillButton, Alert, Table } from "$foundation";
</script>
```

该别名在平台的 Svelte 配置中设置，解析为：

```text
src/lib/foundations-components/src/lib/ui
```

## Compute 工作区审计

Compute 工作区此前对该组件集的使用并不一致。共享的 Compute 组件通常使用 `$foundation`，但若干顶层 Compute 路由仍用路由本地 CSS 手工编写控件、表格、卡片和页面外壳。

| 路由 | 当前状态 |
|---|---|
| Projects | 主要使用共享 Compute 组件；页面中直接使用 Foundation 的情况较少。 |
| Deploy | 高度自定义 UI；仍保留原生筛选器、按钮、下拉框、表格和路由 CSS。 |
| Logs | 通过 `PillButton` 部分使用 Foundation；仍保留原生输入框和表格。 |
| Analytics | 可见卡片和操作已转换为 `GlassCard` 和 `PillButton`。 |
| Environment Variables | 通过 `PillButton` 部分使用 Foundation；仍保留原生模态框、表单和表格。 |
| Domains | 通过 `PillButton` 部分使用 Foundation；仍保留原生模态框和表格。 |
| Storage | 高度自定义 UI；仍保留原生标签页、搜索、按钮、卡片网格和路由 CSS。 |
| Databases | 通过 `PillButton` 部分使用 Foundation；仍保留原生卡片和模态框。 |
| Sandboxes | 通过 `PillButton` 部分使用 Foundation；仍保留原生表格和筛选控件。 |
| Computers | 通过 `PillButton` 部分使用 Foundation；仍保留原生表格和筛选控件。 |

## 清理顺序

1. 将路由级操作以及空状态/错误状态转换为 `PillButton`、`Alert`、`Skeleton` 和共享空状态组件。
2. 用 `RoundedInput`、`Input`、`Select`、`Checkbox` 和 `Toggle` 替换原生表单控件。
3. 用 `Tabs` 或共享的 Compute 筛选标签组件替换手工编写的标签页和标签块。
4. 用 Foundation 表格基础组件或共享的 Compute 表格/卡片组件替换原生表格和卡片。
5. 将重复的外壳 CSS 合并到共享 Compute 布局基础组件中，避免为每个路由复制页面、控制台和主区域 CSS。

## 验证护栏

每个迁移后的路由都应持续通过：

```bash
pnpm check
pnpm test:e2e tests/platform-shell-regression.spec.ts --project=chromium
pnpm build
```

## 另请参阅

- [UI 组件概览](/docs/ui-components)
- [React 组件](/docs/ui-components/react)
- [Vue 组件](/docs/ui-components/vue)
- [Svelte 组件](/docs/ui-components/svelte)
