---
slug: "api-reference/custom-domains"
title: "自定义域 API"
description: "用于使用自动 TLS 将租户拥有的 FQDN 映射到 MIOSA 预览的 API 参考。"
section: "API reference"
order: 12043
---

自定义域允许您在租户拥有的 FQDN 下公开 MIOSA 预览、计算机和部署。有四个相关的表面：

- **计算机自定义域** 将一个完全限定的域（例如 `app.yourdomain.com`）映射到一台计算机服务。
- **部署自定义域** 将一个完全限定的域（例如 `program.example.net`）映射到一个部署。
- **租户预览域** 白标签为整个组织生成预览链接。配置后，沙箱 `/expose` 响应使用 `https://5173-sbx01j9x.sandbox.example.com` 等 URL。

MIOSA 通过 Caddy 按需 TLS 处理验证和自动 TLS 证书颁发。

嵌套基本路径：`/api/v1/computers/{id}/domains` 和 `/api/v1/deployments/{id}/domains`。

扁平前端兼容的基本路径：`/api/v1/custom-domains`。

> **注意**
>
> 自定义域不得以 `miosa.ai` 结尾 - 该命名空间是为平台子域保留的。使用您控制的域。

---

## 快速入门
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// 1. Register the domain
const domain = await client.domains.register(computerId, {
  domain: 'app.yourcompany.com',
});
console.log(domain.verificationTarget); // e.g. "verify.miosa.ai"

// 2. Add a CNAME record at your DNS provider:
//    app.yourcompany.com → verify.miosa.ai

// 3. Trigger verification
const verified = await client.domains.verify(computerId, domain.id);
console.log(verified.status); // "verified" or "failed"
```
```bash
# Register
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/domains \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain": "app.yourcompany.com"}'
```

---

## 端点

### 计算机自定义域

|方法|路径|描述 |
|--------|------|-------------|
| `POST` | `/computers/{id}/domains` |注册自定义域 |
| `GET` | `/computers/{id}/domains` |列出计算机的域 |
| `POST` | `/computers/{id}/domains/{domain_id}/verify` |触发 DNS 验证 |
| `DELETE` | `/computers/{id}/domains/{domain_id}` |删除域名 |

### 部署自定义域

|方法|路径|描述 |
|--------|------|-------------|
| `POST` | `/deployments/{id}/domains` |注册自定义域 |
| `GET` | `/deployments/{id}/domains` |列出部署的域 |
| `POST` | `/deployments/{id}/domains/{domain_id}/verify` |触发 DNS 验证 |
| `DELETE` | `/deployments/{id}/domains/{domain_id}` |删除域名 |

### 扁平化自定义域

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/custom-domains?computer_id={id}` |列出计算机的域 |
| `GET` | `/custom-domains?deployment_id={id}` |列出部署的域 |
| `POST` | `/custom-domains` |使用 `computer_id` 或 `deployment_id` 注册一个域 |
| `DELETE` | `/custom-domains/{id}` |删除域名 |

### 租户预览域

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/tenant/preview-domain` |阅读租户预览域 |
| `PUT` | `/tenant/preview-domain` |设置租户预览域 |
| `DELETE` | `/tenant/preview-domain` |清除租户预览域 |
| `POST` | `/tenant/preview-domain/verify` |检查 DNS/TLS 准备情况 |

预览域是一种组织设置。没有工作区级别或项目级别预览域端点。

---

## 白标托管链接

当生成的应用程序/工件预览 URL 应使用组织的域而不是 MIOSA 管理的后备时，请使用租户预览域。
```bash
curl -X PUT https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"preview_domain":"preview.yourcompany.com"}'
```

然后暴露一个沙箱端口：
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/expose \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"port":5173}'
```

回应：
```json
{
  "url": "https://5173-sbx01j9x.sandbox.preview.yourcompany.com"
}
```

对于其自己的 FQDN（不是共享预览域）下的单个部署，请改为注册部署自定义域：

```bash
curl -X POST https://api.miosa.ai/api/v1/deployments/$DEPLOYMENT_ID/domains \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain":"program.example.net"}'
```

DNS 必须将通配符流量委托给 MIOSA 预览路由器：

|记录类型 |名称 |价值|用于 |
|---|---|---|---|
| `CNAME` | `*` | `proxy.miosa.ai` | `https://{slug}.{domain}` 托管部署/计算机/默认预览 URL 和 `https://{port}-{slug}.sandbox.{domain}` 沙箱端口预览 |

如果未配置租户预览域，MIOSA 将返回该资源的托管后备 URL。即使附加自定义域后，此后备仍然可用。

精确部署和计算机自定义域始终胜过组织范围的预览域。

---

## 注册域名

**`POST /api/v1/computers/{id}/domains`**

### 请求正文

|字段 |类型 |必填 |描述 |
|--------|------|----------|-------------|
| `domain` |字符串|是的 |完全合格的域名。 RFC 1123 主机名，最多 253 个字符 |

### 响应 - `201 Created`
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "computer_id": "...",
    "deployment_id": null,
    "tenant_id": "...",
    "workspace_id": "660e8400-e29b-41d4-a716-446655440001",
    "project_id": "770e8400-e29b-41d4-a716-446655440002",
    "fqdn": "app.yourcompany.com",
    "status": "pending",
    "verification_target": "verify.miosa.ai",
    "verified_at": null,
    "tls_issued_at": null,
    "external_workspace_id": "acct_123",
    "external_user_id": "user-456",
    "external_project_id": "project_789",
    "created_at": "2026-04-11T00:00:00Z",
    "updated_at": "2026-04-11T00:00:00Z"
  }
}
```

### 状态值

|状态 |描述 |
|--------|-------------|
| `pending` |已注册；等待 DNS 验证 |
| `verified` | CNAME 已确认；Caddy 可以颁发证书 |
| `active` | TLS 证书已签发并正在使用 |
| `failed` |验证或 TLS 签发失败 |
| `removed` |域已解除关联（删除前的软标记） |

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 409 | `has already been taken` | FQDN 已注册（全局唯一） |
| 422 | 验证错误 | FQDN 格式无效，或以 `miosa.ai` 结尾 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/domains \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain": "preview.yourapp.io"}'
```

### 注册部署域
```bash
curl -X POST https://api.miosa.ai/api/v1/deployments/{id}/domains \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain": "program.example.net"}'
```

部署域从部署继承 `workspace_id`、`project_id` 和外部归因。

---

## 列出域名

**`GET /api/v1/computers/{id}/domains`**

### 响应 - `200 OK`
```json
{
  "data": [
    {
      "id": "...",
      "fqdn": "app.yourcompany.com",
      "status": "active",
      "verified_at": "2026-04-11T01:00:00Z",
      "tls_issued_at": "2026-04-11T01:05:00Z",
      "created_at": "2026-04-11T00:00:00Z"
    }
  ],
  "total": 1
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/domains \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 验证域名

**`POST /api/v1/computers/{id}/domains/{domain_id}/verify`**

检查 `fqdn` 处的 CNAME 记录是否解析为 `verification_target`。成功后，域将转换为 `"verified"`，Caddy 将在下一个向 FQDN 发出 HTTPS 请求时颁发 TLS 证书。

### 路径参数

|参数|类型 |描述 |
|------------|------|-------------|
| `id` | UUID |计算机 ID |
| `domain_id` | UUID |域名 |

### 响应 - `200 OK`
```json
{
  "data": {
    "id": "...",
    "fqdn": "app.yourcompany.com",
    "status": "verified",
    "verified_at": "2026-04-11T01:00:00Z"
  }
}
```

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 409 | `domain is already verified` |已完成 |
| 422 | `CNAME not found` | DNS 记录缺失或尚未传播 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/domains/{domain_id}/verify \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 删除域名

**`DELETE /api/v1/computers/{id}/domains/{domain_id}`**

将域与计算机分离。证书不会立即撤销（Caddy 让它自然过期），但域将不再路由到计算机。

### 响应 - `200 OK`
```json
{
  "data": { "id": "...", "fqdn": "app.yourcompany.com", "status": "removed" }
}
```
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/domains/{domain_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## DNS 设置参考

注册域后，请在您的注册商或 DNS 提供商处添加以下 DNS 记录：

|记录 |主机 |值 |
|--------|------|--------|
| CNAME | `app`（或完整子域）| `verify.miosa.ai` |

DNS 传播通常需要 1-60 分钟。传播完成后调用 `verify`。如果验证失败，请等待几分钟并重试 - 端点是幂等的。

---

## 常见食谱

### 自动化全域加入
```python
import time
from miosa import Miosa

client = Miosa()

domain = client.domains.register(computer_id, fqdn="api.yourcompany.com")
print(f"Add CNAME: {domain.fqdn} → {domain.verification_target}")
print("Waiting for DNS propagation...")

while True:
    result = client.domains.verify(computer_id, domain.id)
    if result.status == "verified":
        print("Domain verified! TLS will be issued on first HTTPS request.")
        break
    elif result.status == "failed":
        raise RuntimeError("Verification failed  -  check your DNS records.")
    time.sleep(30)
```

### 列出所有待处理域（可能卡住的 ACME 发行）
```typescript
const computers = await client.computers.list();
for (const computer of computers.data) {
  const { data: domains } = await client.domains.list(computer.id);
  const pending = domains.filter(d => d.status === 'pending');
  if (pending.length > 0) {
    console.log(`${computer.name}: ${pending.length} domain(s) pending verification`);
  }
}
```
