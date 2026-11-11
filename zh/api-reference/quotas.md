---
slug: "api-reference/quotas"
title: "配额API"
description: "为白标签多租户部署设置每个最终用户的资源限制。"
section: "API reference"
order: 12047
---

配额可让您限制产品的单个最终用户可以创建和运行的沙箱数量。它们由 `external_user_id` 键入 - 与您代表用户创建沙箱时传递的不透明标识符相同。

基本路径：`/api/v1/quotas/external/{external_user_id}`

> **注意**
>
> 配额在沙箱创建时强制执行。设置配额不会影响已经运行的沙箱。删除配额会删除该用户的所有限制。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/quotas/external/{external_user_id}` |获取配额和当前使用情况 |
| `PUT` | `/api/v1/quotas/external/{external_user_id}` |创建或更新配额 |
| `DELETE` | `/api/v1/quotas/external/{external_user_id}` |删除配额（无限制）|

---

## 获取配额

**`GET /api/v1/quotas/external/{external_user_id}`**

返回用户配置的限制和实时使用计数器。

### 授权

```http
Authorization: Bearer msk_...
```

### 响应 - `200 OK`

```json
{
  "data": {
    "external_user_id": "user_42",
    "limits": {
      "max_sandboxes": 10,
      "max_concurrent": 3,
      "max_storage_gb": 50,
      "max_credit_cents": 5000
    },
    "usage": {
      "total_sandboxes": 4,
      "concurrent": 1
    }
  }
}
```

|领域|类型 |描述 |
|--------|------|-------------|
| `limits.max_sandboxes` |整数\|null |终身沙箱创建限制 |
| `limits.max_concurrent` |整数\|null |同时运行的最大沙箱数 |
| `limits.max_storage_gb` |整数\|null |总附加存储上限（以 GiB 为单位）|
| `limits.max_credit_cents` |整数\|null |支出上限（以美分为单位）（平台积分）|
| `usage.total_sandboxes` |整数 |该用户创建的沙箱总数 |
| `usage.concurrent` |整数 |当前正在运行的沙箱 |

`null` 限制意味着该维度没有应用上限。

---

## 设置配额

**`PUT /api/v1/quotas/external/{external_user_id}`**

创建或替换配额。所有限制字段都是可选的 - 省略一个字段即可使其不受限制。

### 请求正文

```json
{
  "max_sandboxes": 10,
  "max_concurrent": 3,
  "max_storage_gb": 50,
  "max_credit_cents": 5000
}
```

### 响应 - `200 OK`

与 GET 响应的形状相同。

---

## 删除配额

**`DELETE /api/v1/quotas/external/{external_user_id}`**

消除对用户的所有限制。未来的沙箱创建仅由您的租户计划管理。

### 响应 - `204 No Content`

---

## 错误

|状态 |代码|原因 |
|--------|------|--------|
| 404 | `not_found`|此 `external_user_id` (GET/DELETE) |
| 422 | 验证错误|限制值为非整数或负数 |

---

## 示例

**curl**

```bash
# Set
curl -X PUT https://api.miosa.ai/api/v1/quotas/external/user_42 \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"max_sandboxes": 10, "max_concurrent": 3}'

# Get
curl https://api.miosa.ai/api/v1/quotas/external/user_42 \
  -H "Authorization: Bearer msk_u_..."

# Remove
curl -X DELETE https://api.miosa.ai/api/v1/quotas/external/user_42 \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Set quota when user signs up
client.quotas.set("user_42", max_sandboxes=10, max_concurrent=3)

# Check quota + usage
quota = client.quotas.get("user_42")
print(quota.usage.concurrent)

# Remove limits
client.quotas.delete("user_42")
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Set
await client.quotas.set('user_42', {
  maxSandboxes: 10,
  maxConcurrent: 3,
  maxStorageGb: 50,
});

// Get
const quota = await client.quotas.get('user_42');
console.log(quota.usage.concurrent);

// Remove
await client.quotas.delete('user_42');
```
