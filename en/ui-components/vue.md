---
slug: "ui-components/vue"
title: "Vue UI Components"
description: "Drop-in Vue 3 components for embedding MIOSA sandboxes - MiosaPreview, MiosaTerminal, MiosaFileTree, MiosaUsage."
section: "Platform builders"
order: 9023
---

`@miosa/vue` v0.3.2 provides four Vue 3 single-file components for embedding MIOSA sandboxes. All components are written with `<script setup>` and work with Vue 3.3+ and Nuxt 3.

## Install

```bash
npm install @miosa/vue
# xterm peer deps  -  required only if you use MiosaTerminal
npm install @xterm/xterm xterm-addon-fit
```

Styles are scoped inside each SFC  -  no global stylesheet import required.

## MiosaPreview

Renders a sandboxed `<iframe>` showing the HTTP preview URL for a running sandbox.

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

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **Required.** The sandbox ID to preview. |
| `previewToken` | `string` |  -  | Short-lived preview token from your backend. Mutually exclusive with `apiKey`. |
| `apiKey` | `string` |  -  | Your MIOSA API key. Use only in trusted server-rendered contexts. |
| `theme` | `"light" \| "dark"` | `"light"` | Visual theme. |
| `class` | `string` |  -  | Forwarded to the root `<div>`. |

### Events

| Event | Payload | Description |
|---|---|---|
| `error` | `Error` | Emitted when the preview URL cannot be resolved. |

Exactly one of `previewToken` or `apiKey` must be provided.

---

## MiosaTerminal

Renders a full PTY terminal connected to a sandbox over WebSocket, powered by xterm.js.

> **Caution**
>
> `MiosaTerminal` requires `apiKey`. Keep it in server-rendered or internal tooling pages  -  do not expose your API key in public client-side code.

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

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **Required.** |
| `apiKey` | `string` |  -  | **Required.** |
| `theme` | `"light" \| "dark"` | `"dark"` | Sets the xterm.js color scheme. |

### Events

| Event | Payload | Description |
|---|---|---|
| `resize` | `[cols: number, rows: number]` | Emitted when the terminal dimensions change. |
| `error` | `Error` | Emitted on WebSocket errors or xterm load failure. |

The component auto-fits to its container via `ResizeObserver` and syncs resize events to the PTY.

---

## MiosaFileTree

Renders a flat-list file browser backed by the sandbox filesystem. Directories expand inline on click; files emit a `select` event.

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

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **Required.** |
| `apiKey` | `string` |  -  | **Required.** |
| `showHidden` | `boolean` | `false` | Include dotfiles and hidden directories. |
| `defaultExpanded` | `boolean` | `false` | When `true`, all directories start expanded. |

### Events

| Event | Payload | Description |
|---|---|---|
| `select` | `path: string` | Emitted when a file is clicked. `path` is the absolute path within the sandbox (e.g. `/workspace/src/index.ts`). |
| `change` | `[path: string, type: "created" \| "deleted" \| "modified"]` | Emitted on filesystem changes (when the watch subscription fires). |

---

## MiosaUsage

Renders a three-column usage summary (compute, storage, egress) for a specific end-user.

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

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `externalUserId` | `string` |  -  | **Required.** Your platform's user ID, matching the `external_user_id` used when creating sandboxes. |
| `apiKey` | `string` |  -  | **Required.** |
| `period` | `"current" \| "last_30" \| "last_7"` | `"current"` | Billing period to aggregate over. `"current"` is the current billing cycle; `"last_30"` / `"last_7"` are rolling day windows. |
| `theme` | `"light" \| "dark"` | `"light"` | Visual theme. |

Displays `compute_seconds`, `storage_gb_hours`, and `egress_gb`. No events are emitted.

---

## End-to-end example: Nuxt 3

**`server/api/sandbox-token.post.ts`**  -  server route (keeps API key off the client)

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

> **Tip**
>
> In Nuxt 3, use `runtimeConfig` to expose the API key only on the server side (`runtimeConfig.miosaApiKey`, not `runtimeConfig.public.miosaApiKey`). This prevents it from being serialized into the client bundle.

## See also

- [UI Components overview](/docs/ui-components)
- [Platform: Browser Tokens](/docs/platform/browser-tokens)
- [React components](/docs/ui-components/react)
- [Svelte components](/docs/ui-components/svelte)
