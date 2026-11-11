---
slug: "ui-components/react"
title: "React UI Components"
description: "Drop-in React components for embedding MIOSA sandboxes - MiosaPreview, MiosaTerminal, MiosaFileTree, MiosaUsage."
section: "Platform builders"
order: 9022
---

`@miosa/react` v0.3.2 provides four React components for embedding MIOSA sandboxes in any React application.

## Install

```bash
npm install @miosa/react
# xterm peer deps  -  required only if you use MiosaTerminal
npm install @xterm/xterm @xterm/addon-fit
```

Add the bundled stylesheet to your app entry point:

```ts
import '@miosa/react/styles.css';
```

## MiosaThemeProvider

Wrap your component tree with `MiosaThemeProvider` to set a default theme for all nested Miosa components.

```tsx
import { MiosaThemeProvider } from '@miosa/react';

export default function App() {
  return (
    <MiosaThemeProvider theme="dark">
      {/* all Miosa components inherit theme="dark" */}
    </MiosaThemeProvider>
  );
}
```

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `theme` | `"dark" \| "light"` | `"dark"` | Visual theme applied to all child components |
| `injectStyles` | `boolean` | `false` | When `true`, lazily injects the stylesheet via a `<link>` tag. Prefer the explicit `import '@miosa/react/styles.css'` instead. |
| `children` | `React.ReactNode` |  -  | Required. Your component tree. |

---

## MiosaPreview

Renders a sandboxed `<iframe>` showing the HTTP preview URL for a running sandbox.

```tsx
import { MiosaPreview } from '@miosa/react';

// Using a short-lived preview token (recommended for public-facing embeds)
<MiosaPreview
  sandboxId="sb_abc123"
  previewToken={tokenFromYourBackend}
/>

// Using an API key (server-rendered / trusted context only)
<MiosaPreview
  sandboxId="sb_abc123"
  apiKey={process.env.MIOSA_API_KEY}
/>
```

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **Required.** The sandbox ID to preview. |
| `previewToken` | `string` |  -  | Short-lived preview token minted by your backend. Mutually exclusive with `apiKey`. |
| `apiKey` | `string` |  -  | Your MIOSA API key. Mutually exclusive with `previewToken`. Do not expose in the browser. |
| `theme` | `"dark" \| "light"` | `"dark"` | Visual theme. |
| `className` | `string` |  -  | CSS class forwarded to the root element. |
| `onError` | `(err: Error) => void` |  -  | Called when the preview URL cannot be resolved. |

Exactly one of `previewToken` or `apiKey` must be provided.

### States

The component progresses through `idle → loading → ready` (or `error`). A skeleton placeholder is shown while loading. On error, an accessible `role="alert"` message is rendered.

---

## MiosaTerminal

Renders a full PTY terminal connected to a sandbox over WebSocket, powered by xterm.js. The terminal auto-fits to its container and reconnects on resize.

> **Caution**
>
> `MiosaTerminal` requires `apiKey`  -  it does not support `previewToken`. Keep this component in server-rendered or internal tooling contexts.

```tsx
import { MiosaTerminal } from '@miosa/react';

<MiosaTerminal
  sandboxId="sb_abc123"
  apiKey={process.env.MIOSA_API_KEY}
  onResize={(cols, rows) => console.log('resized to', cols, rows)}
  onError={(err) => console.error(err)}
/>
```

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **Required.** The sandbox to connect to. |
| `apiKey` | `string` |  -  | **Required.** Your MIOSA API key. |
| `theme` | `"dark" \| "light"` | `"dark"` | Sets the xterm.js color theme. |
| `className` | `string` |  -  | CSS class forwarded to the root `<div>`. |
| `onResize` | `(cols: number, rows: number) => void` |  -  | Called whenever the terminal dimensions change. |
| `onError` | `(err: Error) => void` |  -  | Called on WebSocket errors or if xterm fails to load. |

### Peer dependencies

`@xterm/xterm` and `@xterm/addon-fit` are loaded dynamically at runtime (lazy import), so they will not bloat your initial bundle. The component falls back to the legacy `xterm` package name if `@xterm/xterm` is unavailable.

---

## MiosaFileTree

Renders an interactive file browser backed by the sandbox filesystem. Directories expand and collapse; clicking a file fires `onSelect`. The tree subscribes to filesystem change events and refreshes automatically.

```tsx
import { MiosaFileTree } from '@miosa/react';
import type { FileNode } from '@miosa/react';

<MiosaFileTree
  sandboxId="sb_abc123"
  apiKey={process.env.MIOSA_API_KEY}
  showHidden={false}
  defaultExpanded={['/workspace/src']}
  onSelect={(file: FileNode) => openFile(file.path)}
  onChange={(tree: FileNode[]) => console.log('tree updated', tree)}
/>
```

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **Required.** |
| `apiKey` | `string` |  -  | **Required.** |
| `theme` | `"dark" \| "light"` | `"dark"` | Visual theme. |
| `className` | `string` |  -  | CSS class on the root `<nav>`. |
| `showHidden` | `boolean` | `false` | When `true`, dotfiles and hidden directories are included. |
| `defaultExpanded` | `string[]` | `[]` | Array of directory paths to start expanded, e.g. `['/workspace/src']`. |
| `onSelect` | `(file: FileNode) => void` |  -  | Called when a file (not a directory) is clicked. |
| `onChange` | `(tree: FileNode[]) => void` |  -  | Called after each tree refresh with the full updated tree. |
| `onError` | `(err: Error) => void` |  -  | Called on load or watch errors. |

### FileNode type

```ts
interface FileNode {
  path: string;       // absolute path, e.g. "/workspace/src/index.ts"
  name: string;       // filename segment, e.g. "index.ts"
  type: 'file' | 'directory';
  children?: FileNode[]; // present for directories
}
```

---

## MiosaUsage

Renders a usage summary for a specific end-user  -  compute seconds, aggregated over a time period. Intended for billing dashboards in white-label products.

```tsx
import { MiosaUsage } from '@miosa/react';

<MiosaUsage
  externalUserId="user_xyz"
  apiKey={process.env.MIOSA_API_KEY}
  period="30d"
/>
```

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `externalUserId` | `string` |  -  | **Required.** Your platform's user ID (the `external_user_id` used when creating sandboxes). |
| `apiKey` | `string` |  -  | **Required.** |
| `period` | `"7d" \| "30d" \| "90d"` | `"30d"` | Time window for usage aggregation. |
| `theme` | `"dark" \| "light"` | `"dark"` | Visual theme. |
| `className` | `string` |  -  | CSS class on the root element. |
| `onError` | `(err: Error) => void` |  -  | Called on fetch errors. |

The component renders a total usage count and a bar chart of daily buckets.

---

## Theming and styling

All components attach BEM class names to their root element for CSS targeting:

| Component | Root class |
|---|---|
| `MiosaPreview` | `.miosa-preview` |
| `MiosaTerminal` | `.miosa-terminal` |
| `MiosaFileTree` | `.miosa-file-tree` |
| `MiosaUsage` | `.miosa-usage` |

Theme variants are appended as modifier classes: `.miosa-preview--dark`, `.miosa-preview--light`, etc.

The `data-miosa-theme` attribute is also set on the root element, allowing CSS attribute selectors:

```css
[data-miosa-theme="dark"] .miosa-preview__frame {
  border: 1px solid #333;
}
```

---

## End-to-end example: Next.js

This example creates a sandbox on page load, then shows the preview, terminal, and file tree side by side.

**`app/api/sandbox-token/route.ts`**  -  backend token endpoint (never expose your API key to the client)

```ts
import { NextResponse } from 'next/server';
import Miosa from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

export async function POST(req: Request) {
  const { sandboxId } = await req.json() as { sandboxId: string };
  const sandbox = await client.sandboxes.get(sandboxId);
  // @ts-expect-error - previewToken is available on the sandbox object
  const { token } = await sandbox.previewToken(3600);
  return NextResponse.json({ token });
}
```

**`app/workbench/page.tsx`**  -  client component

```tsx
'use client';

import { useEffect, useState } from 'react';
import {
  MiosaPreview,
  MiosaTerminal,
  MiosaFileTree,
  MiosaThemeProvider,
} from '@miosa/react';
import type { FileNode } from '@miosa/react';
import '@miosa/react/styles.css';

const SANDBOX_ID = 'sb_abc123'; // created elsewhere in your app

export default function WorkbenchPage() {
  const [token, setToken] = useState<string | null>(null);
  const [selectedFile, setSelectedFile] = useState<string | null>(null);

  useEffect(() => {
    fetch('/api/sandbox-token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ sandboxId: SANDBOX_ID }),
    })
      .then((r) => r.json() as Promise<{ token: string }>)
      .then(({ token }) => setToken(token));
  }, []);

  if (!token) return <div>Loading workspace…</div>;

  return (
    <MiosaThemeProvider theme="dark">
      <div style={{ display: 'grid', gridTemplateColumns: '240px 1fr 1fr', height: '100vh' }}>
        {/* File tree sidebar */}
        <MiosaFileTree
          sandboxId={SANDBOX_ID}
          apiKey={/* server-component prop or env  -  omit on public pages */}
          onSelect={(file: FileNode) => setSelectedFile(file.path)}
        />

        {/* Preview panel */}
        <div style={{ display: 'flex', flexDirection: 'column', height: '100%' }}>
          {selectedFile && (
            <div style={{ padding: '8px', fontSize: 12, color: '#888' }}>
              {selectedFile}
            </div>
          )}
          <MiosaPreview
            sandboxId={SANDBOX_ID}
            previewToken={token}
            style={{ flex: 1 }}
          />
        </div>

        {/* Terminal panel */}
        <MiosaTerminal
          sandboxId={SANDBOX_ID}
          apiKey={process.env.NEXT_PUBLIC_MIOSA_API_KEY!}
          onError={(err) => console.error('[terminal]', err)}
        />
      </div>
    </MiosaThemeProvider>
  );
}
```

> **Tip**
>
> In production, generate the preview token on your Next.js server side (e.g. a Server Component or API route) rather than in a `useEffect`. This keeps your API key entirely off the client.

## See also

- [UI Components overview](/docs/ui-components)
- [Platform: Browser Tokens](/docs/platform/browser-tokens)
- [Vue components](/docs/ui-components/vue)
- [Svelte components](/docs/ui-components/svelte)
