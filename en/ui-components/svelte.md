---
slug: "ui-components/svelte"
title: "Svelte UI Components"
description: "Drop-in Svelte 5 components for embedding MIOSA sandboxes - MiosaPreview, MiosaTerminal, MiosaFileTree, MiosaUsage."
section: "Platform builders"
order: 9024
---

`@miosa/svelte` v0.3.2 provides four Svelte 5 components for embedding MIOSA sandboxes. All components use Svelte 5 runes (`$props`, `$state`, `$derived`, `$effect`) and work with SvelteKit out of the box.

## Install

```bash
npm install @miosa/svelte
# xterm peer deps  -  required only if you use MiosaTerminal
npm install @xterm/xterm xterm-addon-fit
```

Styles are scoped inside each `.svelte` file  -  no global stylesheet import required.

## MiosaPreview

Renders a sandboxed `<iframe>` showing the HTTP preview URL for a running sandbox.

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

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **Required.** The sandbox ID to preview. |
| `previewToken` | `string` |  -  | Short-lived preview token from your backend. Mutually exclusive with `apiKey`. |
| `apiKey` | `string` |  -  | Your MIOSA API key. Use only in `+page.server.svelte` or trusted SSR contexts. |
| `theme` | `"light" \| "dark"` | `"light"` | Visual theme. |
| `class` | `string` | `""` | CSS class forwarded to the root `<div>`. |

### Events

| Event | Detail | Description |
|---|---|---|
| `error` | `Error` | Dispatched when the preview URL cannot be resolved. |

Exactly one of `previewToken` or `apiKey` must be provided.

The component re-resolves the URL reactively whenever `sandboxId` changes (via `$effect`).

---

## MiosaTerminal

Renders a full PTY terminal connected to a sandbox over WebSocket, powered by xterm.js.

> **Caution**
>
> `MiosaTerminal` requires `apiKey`. Place it inside a `+page.server.svelte` data load or pass the key via a server-only environment variable  -  do not embed your API key in client-side markup.

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

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **Required.** |
| `apiKey` | `string` |  -  | **Required.** |
| `theme` | `"light" \| "dark"` | `"dark"` | Sets the xterm.js color scheme. |

### Events

| Event | Detail | Description |
|---|---|---|
| `resize` | `{ cols: number; rows: number }` | Dispatched when the terminal dimensions change. |
| `error` | `Error` | Dispatched on WebSocket errors or xterm load failure. |

The component auto-fits to its container via `ResizeObserver` and syncs resize events to the PTY. Cleanup (WebSocket close + terminal dispose) runs in `onDestroy`.

---

## MiosaFileTree

Renders a flat-list file browser backed by the sandbox filesystem. Directories expand inline on click; files dispatch a `select` event with the file path.

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

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **Required.** |
| `apiKey` | `string` |  -  | **Required.** |
| `showHidden` | `boolean` | `false` | Include dotfiles and hidden directories. |
| `defaultExpanded` | `boolean` | `false` | When `true`, all directories start expanded. |

### Events

| Event | Detail | Description |
|---|---|---|
| `select` | `string` | Absolute path of the clicked file, e.g. `/workspace/src/index.ts`. |
| `change` | `{ path: string; type: "created" \| "deleted" \| "modified" }` | Dispatched on filesystem changes. |

The tree uses `$derived` to recompute the flat render list whenever a directory is toggled  -  no recursive component needed.

---

## MiosaUsage

Renders a three-column usage summary (compute seconds, storage GB·hours, egress GB) for a specific end-user.

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

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `externalUserId` | `string` |  -  | **Required.** Your platform's user ID, matching the `external_user_id` used when creating sandboxes. |
| `apiKey` | `string` |  -  | **Required.** |
| `period` | `"current" \| "last_30" \| "last_7"` | `"current"` | Billing period. `"current"` is the active billing cycle; `"last_30"` / `"last_7"` are rolling windows. |
| `theme` | `"light" \| "dark"` | `"light"` | Visual theme. |

Reactively reloads whenever `externalUserId` or `period` changes via `$effect`. No events are dispatched.

---

## createMiosaClient store

The package also exports a `createMiosaClient` helper for creating a shared client store:

```ts
import { createMiosaClient } from '@miosa/svelte';

const client = createMiosaClient({ apiKey: 'msk_...' });
```

This is intended for advanced use cases where you need direct SDK access alongside the components.

---

## End-to-end example: SvelteKit

**`src/routes/workbench/+page.server.ts`**  -  load preview token server-side

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

> **Tip**
>
> `$env/dynamic/private` is available only in server-side SvelteKit modules (`+page.server.ts`, hooks, etc.). Never import it in a `+page.svelte` or any other client-side file  -  SvelteKit will throw a build error if you try.

## See also

- [UI Components overview](/docs/ui-components)
- [Platform: Browser Tokens](/docs/platform/browser-tokens)
- [React components](/docs/ui-components/react)
- [Vue components](/docs/ui-components/vue)
