---
slug: "cookbook/embed-sandbox-iframe"
title: "在 iframe 中嵌入沙箱"
description: "您的服务器会铸造一个预览令牌；您的前端将其用作 iframe src。涵盖 React、Vue、Svelte 和 vanilla JS，以及令牌 TTL 刷新。"
section: "Platform builders"
order: 9016
---

`~8 min` `Python` `TypeScript`

> **目标**：在您的产品 UI 中嵌入实时 MIOSA 沙箱，而不暴露您的 `msk_*` 密钥。您的服务器会铸造一个短暂的预览令牌；你的前端将其放入 `<iframe src>` 中。
>
> **您将使用什么**：沙箱、预览令牌

## 预览令牌如何工作

```mermaid
sequenceDiagram
  participant BR as Browser
  participant SV as Your server
  participant MI as MIOSA API

  BR->>SV: GET /my-api/preview-token
  SV->>MI: sbx.previewToken(3600)
  MI-->>SV: { url, expires_at }
  SV-->>BR: preview URL with ?mt=mp_... token
```

`previewToken` 返回的 URL 是：

```text
https://<sandbox-slug>.sandbox.miosa.ai?mt=mp_<base64url-token>
```

放入 iframe `src` 属性是安全的。它不包含您的 API 密钥。当令牌过期时，沙箱仍然存在 - 您创建一个新令牌并更新 `src`。

## 步骤 1 - 服务器端：铸造令牌

您的后端验证所有权，然后调用 `previewToken`。将 URL 返回给前端。

**Python（FastAPI）**

```python
from fastapi import FastAPI, Depends, HTTPException
from miosa import Miosa
import os

app = FastAPI()
client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

def get_current_user(request) -> dict:
    # Your existing session/JWT auth  -  returns {user_id, workspace_id}
    ...

@app.get("/api/sandboxes/{sandbox_id}/preview-token")
async def get_preview_token(sandbox_id: str, user=Depends(get_current_user)):
    # Ownership check  -  ensure this sandbox belongs to this user
    sbx = client.sandboxes.get(sandbox_id)
    if sbx.data.get("external_user_id") != user["user_id"]:
        raise HTTPException(403, "Forbidden")

    # Mint a 1-hour preview token
    token_info = sbx.preview_token(expires_in=3600)

    return {
        "url": token_info["url"],
        "expires_at": token_info["expires_at"],
    }
```

**TypeScript（Express）**

```typescript
import express from 'express';
import { Miosa } from '@miosa/sdk';

const app = express();
const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

// Assumes req.user is populated by your auth middleware
app.get('/api/sandboxes/:sandboxId/preview-token', async (req, res) => {
  const { sandboxId } = req.params;
  const { userId } = (req as any).user;

  // Ownership check
  const sbx = await client.sandboxes.get(sandboxId);
  if (sbx.data.external_user_id !== userId) {
    res.status(403).json({ error: 'Forbidden' });
    return;
  }

  // Mint a 1-hour preview token
  const tokenInfo = await sbx.previewToken(3600);

  res.json({
    url: tokenInfo.url,
    expiresAt: tokenInfo.expires_at,
  });
});
```

> **注意**
>
> 切勿将原始 `msk_*` 键返回到浏览器。预览令牌 URL 是您的前端应该收到的唯一凭据，它在 TTL 持续时间内授予对一个沙箱的只读访问权限。

## 步骤 2 - 前端：嵌入 iframe

从服务器获取令牌并使用 URL 作为 `src`。设置刷新计时器以在到期前重新铸造。

**反应**

```tsx
import { useEffect, useRef, useState } from 'react';

interface SandboxEmbedProps {
  sandboxId: string;
  height?: number;
}

export function SandboxEmbed({ sandboxId, height = 600 }: SandboxEmbedProps) {
  const [previewUrl, setPreviewUrl] = useState<string | null>(null);
  const refreshTimerRef = useRef<ReturnType<typeof setTimeout>>();

  async function fetchToken() {
    const res = await fetch(`/api/sandboxes/${sandboxId}/preview-token`);
    const { url, expiresAt } = await res.json();

    setPreviewUrl(url);

    // Re-mint 60 seconds before expiry
    const ttlMs = new Date(expiresAt).getTime() - Date.now() - 60_000;
    clearTimeout(refreshTimerRef.current);
    refreshTimerRef.current = setTimeout(fetchToken, Math.max(ttlMs, 5_000));
  }

  useEffect(() => {
    fetchToken();
    return () => clearTimeout(refreshTimerRef.current);
  }, [sandboxId]);

  if (!previewUrl) {
    return <div style={{ height, background: '#111', borderRadius: 8 }}>Loading…</div>;
  }

  return (
    <iframe
      src={previewUrl}
      width="100%"
      height={height}
      style={{ border: 'none', borderRadius: 8 }}
      sandbox="allow-same-origin allow-scripts allow-forms allow-popups"
      title={`Sandbox ${sandboxId}`}
    />
  );
}
```

**Vue**

```vue
<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';

const props = defineProps<{ sandboxId: string; height?: number }>();
const height = props.height ?? 600;

const previewUrl = ref<string | null>(null);
let refreshTimer: ReturnType<typeof setTimeout> | undefined;

async function fetchToken() {
  const res = await fetch(`/api/sandboxes/${props.sandboxId}/preview-token`);
  const { url, expiresAt } = await res.json();

  previewUrl.value = url;

  const ttlMs = new Date(expiresAt).getTime() - Date.now() - 60_000;
  clearTimeout(refreshTimer);
  refreshTimer = setTimeout(fetchToken, Math.max(ttlMs, 5_000));
}

onMounted(fetchToken);
onUnmounted(() => clearTimeout(refreshTimer));
</script>

<template>
  <div :style="{ height: `${height}px` }">
    <div v-if="!previewUrl" :style="{ height: '100%', background: '#111', borderRadius: '8px' }">
      Loading…
    </div>
    <iframe
      v-else
      :src="previewUrl"
      width="100%"
      :height="height"
      style="border: none; border-radius: 8px"
      sandbox="allow-same-origin allow-scripts allow-forms allow-popups"
      :title="`Sandbox ${sandboxId}`"
    />
  </div>
</template>
```

**苗条**

```svelte
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';

  export let sandboxId: string;
  export let height: number = 600;

  let previewUrl: string | null = null;
  let refreshTimer: ReturnType<typeof setTimeout>;

  async function fetchToken() {
    const res = await fetch(`/api/sandboxes/${sandboxId}/preview-token`);
    const { url, expiresAt } = await res.json();

    previewUrl = url;

    const ttlMs = new Date(expiresAt).getTime() - Date.now() - 60_000;
    clearTimeout(refreshTimer);
    refreshTimer = setTimeout(fetchToken, Math.max(ttlMs, 5_000));
  }

  onMount(fetchToken);
  onDestroy(() => clearTimeout(refreshTimer));
</script>

{#if previewUrl}
  <iframe
    src={previewUrl}
    width="100%"
    {height}
    style="border: none; border-radius: 8px"
    sandbox="allow-same-origin allow-scripts allow-forms allow-popups"
    title="Sandbox {sandboxId}"
  />
{:else}
  <div style="height: {height}px; background: #111; border-radius: 8px">Loading…</div>
{/if}
```

**原版 JS**

```html
<!doctype html>
<html>
  <body>
    <div id="sandbox-container" style="height:600px"></div>
    <script>
      let refreshTimer;

      async function mountSandbox(sandboxId, containerId) {
        const container = document.getElementById(containerId);

        async function fetchAndEmbed() {
          const res = await fetch(`/api/sandboxes/${sandboxId}/preview-token`);
          const { url, expiresAt } = await res.json();

          // Replace or create iframe
          let iframe = container.querySelector('iframe');
          if (!iframe) {
            iframe = document.createElement('iframe');
            iframe.style.cssText = 'width:100%;height:100%;border:none;border-radius:8px';
            iframe.setAttribute('sandbox', 'allow-same-origin allow-scripts allow-forms allow-popups');
            container.appendChild(iframe);
          }
          iframe.src = url;

          // Schedule refresh 60s before expiry
          const ttlMs = new Date(expiresAt).getTime() - Date.now() - 60_000;
          clearTimeout(refreshTimer);
          refreshTimer = setTimeout(fetchAndEmbed, Math.max(ttlMs, 5_000));
        }

        await fetchAndEmbed();
      }

      mountSandbox('sbx_your_id_here', 'sandbox-container');
    </script>
  </body>
</html>
```

## 令牌刷新策略

|场景|推荐|
|---|---|
|短期会议（1 小时以下）|使用 `expires_in=3600` 铸造一次。无需刷新。 |
|长会 |使用 `expires_in=3600` 进行铸造，安排在到期前 60 秒刷新（如上所示）。 |
|选项卡处于非活动状态 |使用 `visibilitychange` 暂停刷新计时器。当标签回来时，薄荷新鲜。 |
|令牌已过期 |获取一个新令牌 - 旧令牌被 iframe 默默拒绝，这不是硬错误。 |

## CSP 注意事项

如果您的产品设置了 `Content-Security-Policy`，请将 MIOSA 预览域添加到 `frame-src`：

```http
Content-Security-Policy: frame-src https://*.sandbox.miosa.ai;
```

如果您使用自定义白标签预览域（通过 `client.customDomains.create` 配置），请改为添加该域。

## 相关

- [智能体构建应用程序](/docs/cookbook/agent-builds-app)：生成您嵌入的沙箱的完整生命周期。
- [多租户 SaaS](/docs/cookbook/multi-tenant-saas)：为租户标记沙箱、颁发浏览器令牌、查询每个租户的使用情况。
- [预览令牌参考](/docs/api-reference/preview-tokens)：令牌格式、范围、撤销和 CSP 嵌入配置。
