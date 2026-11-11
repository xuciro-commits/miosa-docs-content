---
slug: "ui-components/svelte"
title: "Svelte UI 组件"
description: "用于嵌入 MIOSA 沙箱的开箱即用 Svelte 5 组件——MiosaPreview、MiosaTerminal、MiosaFileTree、MiosaUsage。"
section: "Platform builders"
order: 9024
---

`@miosa/svelte` v0.3.2 提供四个用于嵌入 MIOSA 沙箱的 Svelte 5 组件。所有组件都使用 Svelte 5 runes（`$props`、`$state`、`$derived`、`$effect`），并可直接配合 SvelteKit 使用。

## 安装

```bash
npm install @miosa/svelte
# xterm peer deps  -  required only if you use MiosaTerminal
npm install @xterm/xterm xterm-addon-fit
```

样式限定在每个 `.svelte` 文件内部——无需导入全局样式表。

## MiosaPreview

渲染一个沙箱化的 `<iframe>`，显示正在运行的沙箱的 HTTP 预览 URL。

```svelte
<script lang="ts">
  import { MiosaPreview } from '@miosa/svelte';

  // previewToken should be fetched from your backend  -  never hard-code your API key here
  let { previewToken } = await loadTokenFromServer();
</script>

<MiosaPreview
  sandboxId="sb_abc123"
  {previewToken}
  theme="dark"
  on:error={(e) => console.error(e.detail)}
/>
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **必填。** 要预览的沙箱 ID。 |
| `previewToken` | `string` |  -  | 由后端签发的短期预览令牌。与 `apiKey` 互斥。 |
| `apiKey` | `string` |  -  | 你的 MIOSA API 密钥。仅在 `+page.server.svelte` 或可信的 SSR 上下文中使用。 |
| `theme` | `"light" \| "dark"` | `"light"` | 视觉主题。 |
| `class` | `string` | `""` | 转发到根 `<div>` 的 CSS 类。 |

### 事件

| 事件 | 详情 | 说明 |
|---|---|---|
| `error` | `Error` | 在预览 URL 解析失败时派发。 |

必须且只能提供 `previewToken` 或 `apiKey` 之一。

每当 `sandboxId` 变化时，组件都会通过 `$effect` 响应式地重新解析 URL。

---

## MiosaTerminal

渲染一个通过 WebSocket 连接到沙箱的完整 PTY 终端，由 xterm.js 提供支持。

> **注意**
>
> `MiosaTerminal` 需要 `apiKey`。请将它放在 `+page.server.svelte` 数据加载中，或通过仅服务端可用的环境变量传入密钥——不要将 API 密钥嵌入客户端页面标记。

```svelte
<script lang="ts">
  import { MiosaTerminal } from '@miosa/svelte';
  import { env } from '$env/dynamic/private'; // SvelteKit server-only env
</script>

<MiosaTerminal
  sandboxId="sb_abc123"
  apiKey={env.MIOSA_API_KEY}
  theme="dark"
  on:resize={(e) => console.log(e.detail.cols, e.detail.rows)}
  on:error={(e) => console.error(e.detail)}
  style="height: 400px"
/>
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **必填。** |
| `apiKey` | `string` |  -  | **必填。** |
| `theme` | `"light" \| "dark"` | `"dark"` | 设置 xterm.js 配色方案。 |

### 事件

| 事件 | 详情 | 说明 |
|---|---|---|
| `resize` | `{ cols: number; rows: number }` | 在终端尺寸变化时派发。 |
| `error` | `Error` | 在 WebSocket 出错或 xterm 加载失败时派发。 |

组件通过 `ResizeObserver` 自动适应容器，并将尺寸变化事件同步到 PTY。清理逻辑（关闭 WebSocket 并销毁终端）在 `onDestroy` 中运行。

---

## MiosaFileTree

渲染一个由沙箱文件系统支持的扁平列表文件浏览器。点击目录后会在原位展开；文件会派发一个包含文件路径的 `select` 事件。

```svelte
<script lang="ts">
  import { MiosaFileTree } from '@miosa/svelte';

  let selectedPath = $state<string | null>(null);
</script>

<MiosaFileTree
  sandboxId="sb_abc123"
  apiKey={env.MIOSA_API_KEY}
  showHidden={false}
  defaultExpanded={false}
  on:select={(e) => (selectedPath = e.detail)}
  on:change={(e) => console.log(e.detail.path, e.detail.type)}
/>
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **必填。** |
| `apiKey` | `string` |  -  | **必填。** |
| `showHidden` | `boolean` | `false` | 包含点文件和隐藏目录。 |
| `defaultExpanded` | `boolean` | `false` | 当为 `true` 时，所有目录初始即展开。 |

### 事件

| 事件 | 详情 | 说明 |
|---|---|---|
| `select` | `string` | 所点击文件的绝对路径，例如 `/workspace/src/index.ts`。 |
| `change` | `{ path: string; type: "created" \| "deleted" \| "modified" }` | 在文件系统发生变化时派发。 |

每当切换目录时，树组件会用 `$derived` 重新计算扁平渲染列表——无需递归组件。

---

## MiosaUsage

为特定最终用户渲染一个三列用量摘要（计算秒数、存储 GB·小时、出站流量 GB）。

```svelte
<script lang="ts">
  import { MiosaUsage } from '@miosa/svelte';
</script>

<MiosaUsage
  externalUserId="user_xyz"
  apiKey={env.MIOSA_API_KEY}
  period="last_30"
  theme="dark"
/>
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `externalUserId` | `string` |  -  | **必填。** 你平台的用户 ID，需与创建沙箱时使用的 `external_user_id` 一致。 |
| `apiKey` | `string` |  -  | **必填。** |
| `period` | `"current" \| "last_30" \| "last_7"` | `"current"` | 计费周期。`"current"` 是当前计费周期；`"last_30"` / `"last_7"` 是滚动窗口。 |
| `theme` | `"light" \| "dark"` | `"light"` | 视觉主题。 |

每当 `externalUserId` 或 `period` 变化时，都会通过 `$effect` 响应式地重新加载。不派发任何事件。

---

## createMiosaClient store

该软件包还导出一个 `createMiosaClient` 辅助函数，用于创建共享客户端 store：

```ts
import { createMiosaClient } from '@miosa/svelte';

const client = createMiosaClient({ apiKey: 'msk_...' });
```

它适用于需要在组件之外直接访问 SDK 的高级用例。

---

## 端到端示例：SvelteKit

**`src/routes/workbench/+page.server.ts`**  -  在服务端加载预览令牌

```ts
import type { PageServerLoad } from './$types';
import Miosa from '@miosa/sdk';
import { env } from '$env/dynamic/private';

const SANDBOX_ID = 'sb_abc123';

export const load: PageServerLoad = async () => {
  const client = new Miosa({ apiKey: env.MIOSA_API_KEY });
  const sandbox = await client.sandboxes.get(SANDBOX_ID);
  // @ts-expect-error
  const { token } = await sandbox.previewToken(3600);
  return { sandboxId: SANDBOX_ID, previewToken: token };
};
```

**`src/routes/workbench/+page.svelte`**

```svelte
<script lang="ts">
  import type { PageData } from './$types';
  import { MiosaPreview, MiosaTerminal, MiosaFileTree } from '@miosa/svelte';
  import { env } from '$env/dynamic/private';

  let { data }: { data: PageData } = $props();

  let selectedFile = $state<string | null>(null);
</script>

<div class="workbench">
  <aside class="workbench__sidebar">
    <MiosaFileTree
      sandboxId={data.sandboxId}
      apiKey={env.MIOSA_API_KEY}
      on:select={(e) => (selectedFile = e.detail)}
    />
  </aside>

  <main class="workbench__preview">
    {#if selectedFile}
      <div class="workbench__filepath">{selectedFile}</div>
    {/if}
    <MiosaPreview
      sandboxId={data.sandboxId}
      previewToken={data.previewToken}
      theme="dark"
    />
  </main>

  <section class="workbench__terminal">
    <MiosaTerminal
      sandboxId={data.sandboxId}
      apiKey={env.MIOSA_API_KEY}
      theme="dark"
      on:error={(e) => console.error('[terminal]', e.detail)}
    />
  </section>
</div>

<style>
  .workbench {
    display: grid;
    grid-template-columns: 240px 1fr 1fr;
    height: 100vh;
    overflow: hidden;
  }
</style>
```

> **提示**
>
> `$env/dynamic/private` 仅在服务端 SvelteKit 模块（`+page.server.ts`、hooks 等）中可用。不要在 `+page.svelte` 或任何其他客户端文件中导入它——否则 SvelteKit 会在构建时抛出错误。

## 另请参阅

- [UI 组件概览](/docs/ui-components)
- [平台：浏览器令牌](/docs/platform/browser-tokens)
- [React 组件](/docs/ui-components/react)
- [Vue 组件](/docs/ui-components/vue)
