---
slug: "ui-components/react"
title: "React UI 组件"
description: "用于嵌入 MIOSA 沙箱的即插即用 React 组件——MiosaPreview、MiosaTerminal、MiosaFileTree、MiosaUsage。"
section: "Platform builders"
order: 9022
---

`@miosa/react` v0.3.2 提供四个 React 组件，可将 MIOSA 沙箱嵌入任何 React 应用。

## 安装

```bash
npm install @miosa/react
# xterm peer deps  -  required only if you use MiosaTerminal
npm install @xterm/xterm @xterm/addon-fit
```

将随包提供的样式表添加到你的应用入口：

```ts
import '@miosa/react/styles.css';
```

## MiosaThemeProvider

用 `MiosaThemeProvider` 包裹组件树，为所有嵌套的 Miosa 组件设置默认主题。

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

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `theme` | `"dark" \| "light"` | `"dark"` | 应用于所有子组件的视觉主题 |
| `injectStyles` | `boolean` | `false` | 当为 `true` 时，通过 `<link>` 标签懒加载注入样式表。优先使用显式的 `import '@miosa/react/styles.css'`。 |
| `children` | `React.ReactNode` |  -  | 必填。你的组件树。 |

---

## MiosaPreview

渲染一个沙箱化的 `<iframe>`，显示正在运行的沙箱的 HTTP 预览 URL。

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

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **必填。** 要预览的沙箱 ID。 |
| `previewToken` | `string` |  -  | 由你的后端签发的短期预览令牌。与 `apiKey` 互斥。 |
| `apiKey` | `string` |  -  | 你的 MIOSA API 密钥。与 `previewToken` 互斥。不要在浏览器中暴露。 |
| `theme` | `"dark" \| "light"` | `"dark"` | 视觉主题。 |
| `className` | `string` |  -  | 转发到根元素的 CSS 类。 |
| `onError` | `(err: Error) => void` |  -  | 在无法解析预览 URL 时调用。 |

必须且只能提供 `previewToken` 或 `apiKey` 之一。

### 状态

组件会依次进入 `idle → loading → ready`（或 `error`）状态。加载期间会显示骨架占位符。出错时会渲染一条可访问的 `role="alert"` 消息。

---

## MiosaTerminal

渲染一个完整的 PTY 终端，通过 WebSocket 连接到沙箱，由 xterm.js 提供支持。终端会自动适应容器大小，并在尺寸变化时重新连接。

> **注意**
>
> `MiosaTerminal` 需要 `apiKey`——它不支持 `previewToken`。请将此组件保留在服务端渲染或内部工具场景中。

```tsx
import { MiosaTerminal } from '@miosa/react';

<MiosaTerminal
  sandboxId="sb_abc123"
  apiKey={process.env.MIOSA_API_KEY}
  onResize={(cols, rows) => console.log('resized to', cols, rows)}
  onError={(err) => console.error(err)}
/>
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **必填。** 要连接的沙箱。 |
| `apiKey` | `string` |  -  | **必填。** 你的 MIOSA API 密钥。 |
| `theme` | `"dark" \| "light"` | `"dark"` | 设置 xterm.js 的配色主题。 |
| `className` | `string` |  -  | 转发到根 `<div>` 的 CSS 类。 |
| `onResize` | `(cols: number, rows: number) => void` |  -  | 每当终端尺寸变化时调用。 |
| `onError` | `(err: Error) => void` |  -  | 在 WebSocket 出错或 xterm 加载失败时调用。 |

### 对等依赖

`@xterm/xterm` 和 `@xterm/addon-fit` 在运行时动态加载（懒导入），因此不会增大初始包体积。如果 `@xterm/xterm` 不可用，组件会回退到旧版 `xterm` 包名。

---

## MiosaFileTree

渲染一个由沙箱文件系统支持的交互式文件浏览器。目录可以展开和折叠；点击文件会触发 `onSelect`。文件树订阅文件系统变更事件并自动刷新。

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

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `sandboxId` | `string` |  -  | **必填。** |
| `apiKey` | `string` |  -  | **必填。** |
| `theme` | `"dark" \| "light"` | `"dark"` | 视觉主题。 |
| `className` | `string` |  -  | 根 `<nav>` 上的 CSS 类。 |
| `showHidden` | `boolean` | `false` | 当为 `true` 时，包含点文件和隐藏目录。 |
| `defaultExpanded` | `string[]` | `[]` | 初始展开的目录路径数组，例如 `['/workspace/src']`。 |
| `onSelect` | `(file: FileNode) => void` |  -  | 点击文件（而非目录）时调用。 |
| `onChange` | `(tree: FileNode[]) => void` |  -  | 每次文件树刷新后，以完整的最新树调用。 |
| `onError` | `(err: Error) => void` |  -  | 在加载或监听出错时调用。 |

### FileNode 类型

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

为特定最终用户渲染用量摘要——按时间段汇总的计算秒数。适用于白标产品中的计费仪表板。

```tsx
import { MiosaUsage } from '@miosa/react';

<MiosaUsage
  externalUserId="user_xyz"
  apiKey={process.env.MIOSA_API_KEY}
  period="30d"
/>
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `externalUserId` | `string` |  -  | **必填。** 你平台的用户 ID（创建沙箱时使用的 `external_user_id`）。 |
| `apiKey` | `string` |  -  | **必填。** |
| `period` | `"7d" \| "30d" \| "90d"` | `"30d"` | 用量聚合的时间窗口。 |
| `theme` | `"dark" \| "light"` | `"dark"` | 视觉主题。 |
| `className` | `string` |  -  | 根元素上的 CSS 类。 |
| `onError` | `(err: Error) => void` |  -  | 在请求出错时调用。 |

该组件会渲染总用量计数和按日分桶的柱状图。

---

## 主题与样式

所有组件都会在根元素上附加 BEM 类名，便于 CSS 定位：

| 组件 | 根类名 |
|---|---|
| `MiosaPreview` | `.miosa-preview` |
| `MiosaTerminal` | `.miosa-terminal` |
| `MiosaFileTree` | `.miosa-file-tree` |
| `MiosaUsage` | `.miosa-usage` |

主题变体以修饰类追加：`.miosa-preview--dark`、`.miosa-preview--light` 等。

根元素上还会设置 `data-miosa-theme` 属性，从而支持 CSS 属性选择器：

```css
[data-miosa-theme="dark"] .miosa-preview__frame {
  border: 1px solid #333;
}
```

---

## 端到端示例：Next.js

此示例在页面加载时创建沙箱，然后并排显示预览、终端和文件树。

**`app/api/sandbox-token/route.ts`**——后端令牌端点（切勿将 API 密钥暴露给客户端）

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

**`app/workbench/page.tsx`**——客户端组件

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

> **提示**
>
> 在生产环境中，请在 Next.js 服务端（例如 Server Component 或 API 路由）生成预览令牌，而不是在 `useEffect` 中生成。这样可以让 API 密钥完全不进入客户端。

## 另请参阅

- [UI 组件概览](/docs/ui-components)
- [平台：浏览器令牌](/docs/platform/browser-tokens)
- [Vue 组件](/docs/ui-components/vue)
- [Svelte 组件](/docs/ui-components/svelte)
