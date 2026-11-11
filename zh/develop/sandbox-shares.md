---
slug: "develop/sandbox-shares"
title: "沙盒股份"
description: "创建指向正在运行的沙箱的公共只读链接，以便队友或外部用户可以查看它，并随时撤销该链接。"
section: "Sandboxes"
order: 2016
---

**共享**是一个可公开访问的 URL，它提供对正在运行的沙箱的只读访问权限。与预览令牌不同，共享 URL 的范围不限于 iframe，并且接收者不需要任何凭据：令牌嵌入在 URL 中。

```text
https://{sandbox-id}.sandboxes.miosa.ai?ms=ms_<token>
```

> **注意：只读且可撤销**
>
> 共享仅供查看。接收者可以观看，但不能开车。撤销共享，链接立即停止工作，将 `403` 返回给仍然持有它的任何人。

## 创建共享

**Python**

```python
share = client.sandboxes.shares.create("sbx_abc123", expires_in=86400)
print(share.share_url)
```

**打字稿**

```typescript
const share = await client.sandboxes.shares.create('sbx_abc123', { expiresIn: 86400 });
console.log(share.shareUrl);
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"expires_in": 86400}'
```

|领域|类型 |默认 |描述 |
|---|---|---|---|
| `expires_in` |整数 | `3600` | TTL（以秒为单位）|
| `scope` |字符串| `"read"` |唯一支持的值 |

响应包括 `share_id`（用于撤销）、`share_url`（安全分发）、`expires_at` 和 `scope`。

## 列出并撤销

**Python**

```python
for s in client.sandboxes.shares.list("sbx_abc123"):
    print(s.share_id, s.expires_at)

client.sandboxes.shares.delete("sbx_abc123", share.share_id)
```

**curl**

```bash
curl https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares/shr_01hx9m7n3k \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

上市仅返回活跃股票 - 未撤销且尚未过期。撤销返回 `204 No Content`。

## 共享、预览和浏览器令牌

对于三种不同的工作，可以通过三种不同的方式授予某人访问权限：

|机制|访问 |用它来 |
|---|---|---|
|沙盒分享 |正在运行的沙箱的只读视图 |让某人通过普通链接观看沙箱 |
|预览代币 |对一个暴露端口的 HTTP 访问 |将正在运行的应用程序嵌入 iframe |
|浏览器访问令牌 |桌面直播 |为一名最终用户嵌入桌面 |

有关端口，请参阅[预览](/docs/develop/previews)；有关桌面嵌入，请参阅[浏览器访问令牌](/docs/platform/browser-tokens)。

- [Sandbox 共享 API](/docs/api-reference/sandbox-shares)：线路格式、错误代码和每个端点。
- [预览](/docs/develop/previews)：将端口公开为实时 HTTPS URL，并具有可见性和令牌控制。
