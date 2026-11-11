---
slug: "cookbook/multi-tenant-saas"
title: "多租户 SaaS"
description: "将 MIOSA 嵌入到您的 SaaS 中 - 为每个租户标记沙箱，查询每个工作区的使用情况，并创建短期预览共享令牌，以便客​​户直接与其沙箱交互。"
section: "Platform builders"
order: 9014
---

`~12 min` `Python` `TypeScript`

> **目标**：将 MIOSA 嵌入到多租户产品中。每个沙箱都标有您自己的工作区和用户 ID。每个租户的使用情况汇总以进行计费。最终用户直接从浏览器与其沙箱交互，而无需看到您的 API 密钥。
>
> **您将使用什么**：沙箱、外部归因、预览共享令牌、使用 API

## 你将构建什么

SaaS 中的嵌入式沙盒体验：

1. 您的后端创建标记正确客户的沙箱。
2、可查询每个租户的使用情况，进行扣费。
3. 您客户的浏览器获得一个短期令牌来嵌入实时预览 - 客户端的任何地方都没有 `msk_*` 键。

## 先决条件

- MIOSA 工作区 API 密钥 (`msk_u_*`) - 请参阅 [API 密钥](/docs/platform/api-keys)
- 节点 22+ 或 Python 3.11+

## 步骤 1 - 安装和配置

**Python**

```bash
pip install miosa
export MIOSA_API_KEY="msk_u_..."
```

**打字稿**

```bash
npm install @miosa/sdk
export MIOSA_API_KEY="msk_u_..."
```

## 步骤 2 - 用您的 ID 标记每个沙箱

在每个 `sandboxes.create` 调用中传递 `external_workspace_id`、`external_user_id` 和 `external_project_id`。 MIOSA 将这些存储在资源行中，并自动将它们传播到每个使用记录、审核事件和派生资源（预览、部署）。

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

def create_sandbox_for_user(workspace_id: str, user_id: str, project_id: str):
    """Called from your request handler when a tenant starts a new sandbox."""
    sbx = client.sandboxes.create(
        template_id="miosa-sandbox",
        timeout_sec=3600,
        external_workspace_id=workspace_id,
        external_user_id=user_id,
        external_project_id=project_id,
    )
    return sbx

# Example: workspace "acme-corp", user "jane-doe", project "q3-dashboard"
sbx = create_sandbox_for_user("acme-corp", "jane-doe", "q3-dashboard")
print(f"Created sandbox {sbx.id} for acme-corp / jane-doe")
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

async function createSandboxForUser(
  workspaceId: string,
  userId: string,
  projectId: string,
) {
  const sbx = await client.sandboxes.create({
    templateId: 'miosa-sandbox',
    timeoutSec: 3600,
    externalWorkspaceId: workspaceId,
    externalUserId: userId,
    externalProjectId: projectId,
  });
  return sbx;
}

// Example
const sbx = await createSandboxForUser('acme-corp', 'jane-doe', 'q3-dashboard');
console.log(`Created sandbox ${sbx.id} for acme-corp / jane-doe`);
```

> **提示**
>
> 使用您的平台已有的任何 ID 格式 - UUID、slugs、整数字符串。 MIOSA 将它们存储为文本，并且从不验证超过 255 个字符长度限制的格式。

## 步骤 3 - 列出每个租户的沙箱

按您自己的 ID 筛选沙箱列表，以仅向客户显示其资源，或在计费前枚举帐户的所有沙箱。

**Python**

```python
# All sandboxes for a workspace
acme_sandboxes = client.sandboxes.list(external_workspace_id="acme-corp")

for sbx in acme_sandboxes:
    print(f"  {sbx.id}  user={sbx.external_user_id}  status={sbx.status}")

# Narrow to a single user
jane_sandboxes = client.sandboxes.list(
    external_workspace_id="acme-corp",
    external_user_id="jane-doe",
)
```

**打字稿**

```typescript
// All sandboxes for a workspace
const acmeSandboxes = await client.sandboxes.list({
  externalWorkspaceId: 'acme-corp',
});

for (const sbx of acmeSandboxes) {
  console.log(`  ${sbx.id}  user=${sbx.externalUserId}  status=${sbx.status}`);
}

// Narrow to a single user
const janeSandboxes = await client.sandboxes.list({
  externalWorkspaceId: 'acme-corp',
  externalUserId: 'jane-doe',
});
```

## 步骤 4 - 查询每个租户的使用情况

在每个计费周期结束时提取按 `external_workspace_id` 分组的使用情况报告。使用它来生成发票或在产品内显示成本仪表板。

**Python**

```python
# Monthly rollup  -  grouped by workspace
usage = client.usage.report(
    period={"start": "2026-05-01", "end": "2026-06-01"},
    group_by=["external_workspace_id"],
)

for row in usage:
    print(f"  {row['external_workspace_id']}: ${row['total_usd']:.4f}")

# Drill into a single workspace by user
acme_by_user = client.usage.report(
    period={"start": "2026-05-01", "end": "2026-06-01"},
    external_workspace_id="acme-corp",
    group_by=["external_user_id"],
)

for row in acme_by_user:
    print(f"  user={row['external_user_id']}: ${row['total_usd']:.4f}")
```

**打字稿**

```typescript
// Monthly rollup  -  grouped by workspace
const usage = await client.usage.report({
  period: { start: '2026-05-01', end: '2026-06-01' },
  groupBy: ['external_workspace_id'],
});

for (const row of usage) {
  console.log(`  ${row.externalWorkspaceId}: $${row.totalUsd.toFixed(4)}`);
}

// Drill into a single workspace by user
const acmeByUser = await client.usage.report({
  period: { start: '2026-05-01', end: '2026-06-01' },
  externalWorkspaceId: 'acme-corp',
  groupBy: ['external_user_id'],
});

for (const row of acmeByUser) {
  console.log(`  user=${row.externalUserId}: $${row.totalUsd.toFixed(4)}`);
}
```

## 步骤 5 - 创建预览共享令牌以直接访问

您客户的浏览器需要嵌入实时预览。您不能将 `msk_*` 放在前端。相反，您的后端会铸造一个短暂的**预览共享令牌**并将其返回给客户端。

> **注意：预览共享令牌，而不是浏览器令牌**
>
> 此步骤使用 `previews.share` 创建一个 **预览共享令牌**，范围仅限于一个预览 URL。 **浏览器访问令牌**是一种不同的凭据：它用于嵌入实时桌面流。如果您需要的话，请参阅[浏览器访问令牌](/docs/platform/browser-tokens)。

**后端 - 铸造代币：**

**Python（FastAPI）**

```python
from fastapi import FastAPI, Depends, HTTPException
from miosa import Miosa
import os

app = FastAPI()
client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

def get_current_user(request) -> dict:
    # Your existing auth logic  -  returns { workspace_id, user_id }
    ...

@app.get("/api/sandboxes/{sandbox_id}/preview-token")
async def get_preview_token(sandbox_id: str, user=Depends(get_current_user)):
    # Verify the sandbox belongs to this user's workspace
    sbx = client.sandboxes.get(sandbox_id)
    if sbx.external_workspace_id != user["workspace_id"]:
        raise HTTPException(403, "Forbidden")

    # Ensure a preview is running
    previews = sbx.previews.list()
    if not previews:
        preview = sbx.previews.create(port=3000)
    else:
        preview = previews[0]

    # Mint a 1-hour preview share token
    share = sbx.previews.share(
        preview.id,
        ttl_seconds=3600,
    )

    return {"share_url": share["share_url"], "expires_at": share["expires_at"]}
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
  const { workspaceId } = (req as any).user;

  // Ownership check
  const sbx = await client.sandboxes.get(sandboxId);
  if (sbx.externalWorkspaceId !== workspaceId) {
    res.status(403).json({ error: 'Forbidden' });
    return;
  }

  // Ensure a preview exists
  const previews = await sbx.previews.list();
  const preview =
    previews.length > 0
      ? previews[0]
      : await sbx.previews.create(3000);

  // Mint a 1-hour preview share token
  const share = await sbx.previews.share(preview.id, { ttl_seconds: 3600 });

  res.json({ shareUrl: share.share_url, expiresAt: share.expires_at });
});
```

**前端 - 嵌入预览：**

```html
<!-- Fetch the token from your backend, then embed the iframe -->
<script>
  async function embedPreview(sandboxId) {
    const resp = await fetch(`/api/sandboxes/${sandboxId}/preview-token`);
    const { shareUrl, expiresAt } = await resp.json();

    const iframe = document.getElementById('preview-frame');
    iframe.src = shareUrl;

    // Re-mint before expiry so the session stays alive
    const expiryMs = new Date(expiresAt).getTime() - Date.now() - 60_000;
    setTimeout(() => embedPreview(sandboxId), expiryMs);
  }

  embedPreview('sbx_...');
</script>

<iframe
  id="preview-frame"
  sandbox="allow-same-origin allow-scripts allow-forms"
  style="width: 100%; height: 600px; border: 0;"
></iframe>
```

> **注意**
>
> 切勿将 `msk_*` 放入 `<script>` 标记、暴露于浏览器的环境变量或 localStorage 中。共享令牌是应该到达浏览器的唯一凭据，它仅授予对一个预览的访问权限一小时。

## 步骤 6 - 完全集成布线

以下是新用户加入请求的完整服务器端流程：

**Python**

```python
from miosa import Miosa
import os

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

def onboard_new_project(workspace_id: str, user_id: str, project_id: str) -> dict:
    """Called when a customer creates a new project in your SaaS."""

    # 1. Boot a tagged sandbox
    sbx = client.sandboxes.create(
        template_id="miosa-sandbox",
        timeout_sec=3600,
        external_workspace_id=workspace_id,
        external_user_id=user_id,
        external_project_id=project_id,
    )

    # 2. Write an initial project scaffold
    sbx.files.write(
        "/workspace/README.md",
        f"# Project: {project_id}\nWorkspace: {workspace_id}\n",
    )

    # 3. Start the dev server
    sbx.commands.run("python3 -m http.server 3000", cwd="/workspace", background=True)

    # 4. Create a preview
    preview = sbx.previews.create(port=3000)

    # 5. Mint a preview share token
    share = sbx.previews.share(preview.id, ttl_seconds=3600)

    return {
        "sandbox_id": sbx.id,
        "preview_share_url": share["share_url"],
        "expires_at": share["expires_at"],
    }
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

async function onboardNewProject(
  workspaceId: string,
  userId: string,
  projectId: string,
): Promise<{ sandboxId: string; previewShareUrl: string; expiresAt: string }> {
  // 1. Boot a tagged sandbox
  const sbx = await client.sandboxes.create({
    templateId: 'miosa-sandbox',
    timeoutSec: 3600,
    externalWorkspaceId: workspaceId,
    externalUserId: userId,
    externalProjectId: projectId,
  });

  // 2. Write an initial scaffold
  await sbx.files.write(
    '/workspace/README.md',
    `# Project: ${projectId}\nWorkspace: ${workspaceId}\n`,
  );

  // 3. Start dev server (detached)
  sbx.exec.stream('python3 -m http.server 3000', { cwd: '/workspace' });

  // 4. Create a preview
  const preview = await sbx.previews.create(3000);

  // 5. Mint a preview share token
  const share = await sbx.previews.share(preview.id, { ttl_seconds: 3600 });

  return {
    sandboxId: sbx.id,
    previewShareUrl: share.share_url,
    expiresAt: share.data.shareTokenExpiresAt,
  };
}
```

## 步骤 7 - 设置每用户配额

限制每个用户，以便他们无法耗尽您的 MIOSA 分配。配额立即生效。

**Python**

```python
# Free-tier: max 3 sandboxes, 1 concurrent, $5 credit cap
client.quotas.set(
    "jane-doe",
    max_sandboxes=3,
    max_concurrent=1,
    max_credit_cents=500,
)

# Pro-tier: higher limits
client.quotas.set(
    "jane-doe",
    max_sandboxes=20,
    max_concurrent=5,
    max_credit_cents=5000,
)
```

**打字稿**

```typescript
// Quotas API via REST (SDK method landing in a future release):
await fetch(`https://api.miosa.ai/api/v1/quotas/external/jane-doe`, {
  method: 'PUT',
  headers: {
    Authorization: `Bearer ${process.env.MIOSA_API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ max_sandboxes: 3, max_concurrent: 1, max_credit_cents: 500 }),
});
```

## 步骤 8 - 审核日志

提取分页审核日志以满足合规性要求或调查支持票证。

**Python**

```python
# List recent sandbox create/destroy events
events = client.audit_log.list(
    resource_type="sandbox",
    limit=50,
)

for event in events:
    print(f"  {event.get('inserted_at')}  {event.get('action')}  {event.get('resource_id')}")
```

**打字稿**

```typescript
const events = await client.auditLog.list({
  resource_type: 'sandbox',
  limit: 50,
});

for (const event of events) {
  console.log(`  ${event.inserted_at}  ${event.action}  ${event.resource_id}`);
}
```

## 你学到了什么- 三个外部 ID 字段（`external_workspace_id`、`external_user_id`、`external_project_id`）按您自己的标识符对每个 MIOSA 资源进行分组，无需服务器端 MIOSA 配置。
- 使用情况报告按相同 ID 分组 - 直接输入到您的退款逻辑。
- 共享令牌是资源范围内的、可过期的，并且是应该到达浏览器的唯一凭证。 `msk_*` 键始终位于后端。
- 归属是继承的：从标记的沙箱发布的部署自动带有相同的外部ID。
- `client.quotas.set` 在最终用户耗尽您的分配之前对其进行限制 - 在计划更改时调用它，而不是在每个请求时调用它。
- `client.auditLog.list` 为您提供分页的事件历史记录以供合规性和支持。

## 下一步

- [每用户配额和计费](/docs/cookbook/per-user-quotas-billing)：上限、每月发票汇总和 Stripe 集成的完整演练。
- [嵌入沙箱 iframe](/docs/cookbook/embed-sandbox-iframe)：React、Vue、Svelte 和 vanilla JS iframe 嵌入令牌 TTL 刷新。
- [实时沙箱事件](/docs/cookbook/sandbox-events-realtime)：用于沙箱生命周期通知的 Webhook 和 SSE 流。
