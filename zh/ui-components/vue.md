---
slug: "ui-components/vue"
title: "Vue UI 组件"
description: "用于嵌入 MIOSA 沙箱的即插即用 Vue 3 组件——MiosaPreview、MiosaTerminal、MiosaFileTree、MiosaUsage。"
section: "Platform builders"
order: 9023
---

`@miosa/vue` v0.3.2 提供四个 Vue 3 单文件组件，用于嵌入 MIOSA 沙箱。所有组件均使用 `<script setup>` 编写，适用于 Vue 3.3+ 和 Nuxt 3。

## 安装

```bash
npm install @miosa/vue
# xterm peer deps  -  required only if you use MiosaTerminal
npm install @xterm/xterm xterm-addon-fit
```

样式作用域限定在每个 SFC 内部——无需导入全局样式表。

## MiosaPreview

渲染一个沙箱化的 `<iframe>`，显示正在运行的沙箱的 HTTP 预览 URL。

```vue
<script setup lang="ts">
import { MiosaPreview } from '@miosa/vue';

const sandboxId = 'sb_abc123';
const previewToken = await fetchTokenFromYourBackend(sandboxId); // minted server-side
</script>

<template>
  <!-- Recommended: short-lived preview token from your backend -->
  <MiosaPreview
    :sandbox-id="sandboxId"
    :preview-token="previewToken"
    theme="dark"
    @error="(err) => console.error(err)"
  />
</template>
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **必填。** 要预览的沙箱 ID。 |
| `previewToken` | `string` |  -  | 来自后端的短期预览令牌。与 `apiKey` 互斥。 |
| `apiKey` | `string` |  -  | 你的 MIOSA API 密钥。仅在受信任的服务端渲染环境中使用。 |
| `theme` | `"light" \| "dark"` | `"light"` | 视觉主题。 |
| `class` | `string` |  -  | 转发到根 `<div>`。 |

### 事件

| 事件 | 负载 | 说明 |
|---|---|---|
| `error` | `Error` | 当预览 URL 无法解析时触发。 |

必须且只能提供 `previewToken` 或 `apiKey` 中的一个。

---

## MiosaTerminal

渲染一个通过 WebSocket 连接到沙箱的完整 PTY 终端，由 xterm.js 驱动。

> **注意**
>
> `MiosaTerminal` 需要 `apiKey`。请仅在服务端渲染页面或内部工具页面中使用——不要在公开的客户端代码中暴露 API 密钥。

```vue
<script setup lang="ts">
import { MiosaTerminal } from '@miosa/vue';
</script>

<template>
  <MiosaTerminal
    sandbox-id="sb_abc123"
    :api-key="runtimeConfig.miosaApiKey"
    theme="dark"
    @resize="(cols, rows) => console.log(cols, rows)"
    @error="(err) => console.error(err)"
    style="height: 400px"
  />
</template>
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **必填。** |
| `apiKey` | `string` |  -  | **必填。** |
| `theme` | `"light" \| "dark"` | `"dark"` | 设置 xterm.js 配色方案。 |

### 事件

| 事件 | 负载 | 说明 |
|---|---|---|
| `resize` | `[cols: number, rows: number]` | 当终端尺寸变化时触发。 |
| `error` | `Error` | 在 WebSocket 出错或 xterm 加载失败时触发。 |

组件通过 `ResizeObserver` 自动适应容器，并将尺寸变化事件同步到 PTY。

---

## MiosaFileTree

渲染由沙箱文件系统支持的扁平列表文件浏览器。目录在点击时就地展开；文件会触发 `select` 事件。

```vue
<script setup lang="ts">
import { MiosaFileTree } from '@miosa/vue';

function onFileSelect(path: string) {
  console.log('selected:', path);
}
</script>

<template>
  <MiosaFileTree
    sandbox-id="sb_abc123"
    :api-key="runtimeConfig.miosaApiKey"
    :show-hidden="false"
    :default-expanded="true"
    @select="onFileSelect"
    @change="(path, type) => console.log(path, type)"
  />
</template>
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **必填。** |
| `apiKey` | `string` |  -  | **必填。** |
| `showHidden` | `boolean` | `false` | 包含点文件和隐藏目录。 |
| `defaultExpanded` | `boolean` | `false` | 当为 `true` 时，所有目录初始即展开。 |

### 事件

| 事件 | 负载 | 说明 |
|---|---|---|
| `select` | `path: string` | 点击文件时触发。`path` 是沙箱内的绝对路径（例如 `/workspace/src/index.ts`）。 |
| `change` | `[path: string, type: "created" \| "deleted" \| "modified"]` | 文件系统发生变化时触发（当监视订阅被触发时）。 |

---

## MiosaUsage

为特定最终用户渲染三列用量摘要（计算、存储、出口流量）。

```vue
<script setup lang="ts">
import { MiosaUsage } from '@miosa/vue';
</script>

<template>
  <MiosaUsage
    external-user-id="user_xyz"
    :api-key="runtimeConfig.miosaApiKey"
    period="last_30"
    theme="dark"
  />
</template>
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `externalUserId` | `string` |  -  | **必填。** 你平台的用户 ID，与创建沙箱时使用的 `external_user_id` 一致。 |
| `apiKey` | `string` |  -  | **必填。** |
| `period` | `"current" \| "last_30" \| "last_7"` | `"current"` | 要汇总的计费周期。`"current"` 是当前计费周期；`"last_30"` / `"last_7"` 是滚动天数窗口。 |
| `theme` | `"light" \| "dark"` | `"light"` | 视觉主题。 |

显示 `compute_seconds`、`storage_gb_hours` 和 `egress_gb`。不触发任何事件。

---

## 端到端示例：Nuxt 3

**`server/api/sandbox-token.post.ts`**——服务端路由（让 API 密钥远离客户端）

```ts
import Miosa from '@miosa/sdk';

export default defineEventHandler(async (event) => {
  const { sandboxId } = await readBody<{ sandboxId: string }>(event);
  const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
  const sandbox = await client.sandboxes.get(sandboxId);
  // @ts-expect-error
  const { token } = await sandbox.previewToken(3600);
  return { token };
});
```

**`pages/workbench.vue`**

```vue
<script setup lang="ts">
import { MiosaPreview, MiosaTerminal, MiosaFileTree } from '@miosa/vue';

const SANDBOX_ID = 'sb_abc123';

const { data } = await useFetch('/api/sandbox-token', {
  method: 'POST',
  body: { sandboxId: SANDBOX_ID },
});

const previewToken = computed(() => data.value?.token ?? '');
const selectedFile = ref<string | null>(null);
</script>

<template>
  <div class="workbench">
    <aside class="workbench__sidebar">
      <MiosaFileTree
        :sandbox-id="SANDBOX_ID"
        :api-key="useRuntimeConfig().miosaApiKey"
        @select="(path) => (selectedFile = path)"
      />
    </aside>

    <main class="workbench__preview">
      <div v-if="selectedFile" class="workbench__filepath">{{ selectedFile }}</div>
      <MiosaPreview
        :sandbox-id="SANDBOX_ID"
        :preview-token="previewToken"
        theme="dark"
      />
    </main>

    <section class="workbench__terminal">
      <MiosaTerminal
        :sandbox-id="SANDBOX_ID"
        :api-key="useRuntimeConfig().miosaApiKey"
        theme="dark"
        @error="(err) => console.error('[terminal]', err)"
      />
    </section>
  </div>
</template>

<style scoped>
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
> 在 Nuxt 3 中，使用 `runtimeConfig` 仅在服务端暴露 API 密钥（`runtimeConfig.miosaApiKey`，而不是 `runtimeConfig.public.miosaApiKey`）。这样可以避免它被序列化到客户端打包产物中。

## 另请参阅

- [UI 组件概览](/docs/ui-components)
- [平台：浏览器令牌](/docs/platform/browser-tokens)
- [React 组件](/docs/ui-components/react)
- [Svelte 组件](/docs/ui-components/svelte)
