---
slug: "api-reference/audit-log"
title: "审核日志API"
description: "检索租户内执行的所有操作的光标分页审核日志。"
section: "API reference"
order: 12049
---

审核日志记录租户内执行的所有重要操作 - 资源创建和删除、API 密钥使用、设置更改等。结果按光标分页并按最新的在前排序。

基本路径：`/api/v1/audit-log`

> **注意**
>
> 审计事件是不可变的。没有写入API。该日志只能追加。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/audit-log` |获取最近的审计事件 |

---

## 列出审核事件

**`GET /api/v1/audit-log`**

### 授权

```http
Authorization: Bearer msk_...
```

### 查询参数

|参数|类型 |必填 |默认 |描述 |
|------------|------|----------|---------|-------------|
| `limit` |整数 |没有 | `50` |每页事件数（最多 `100`）|
| `after` |字符串|没有 |  - |光标来自上一个响应的 `next_cursor` |
| `type` |字符串|没有 |  - |按事件类型前缀过滤（例如 `sandbox.`）|
| `actor_id` |字符串|没有 |  - |按演员用户 ID 过滤 |
| `resource_type` |字符串|没有 |  - |按资源类型过滤（例如 `sandbox`）|
| `resource_id` |字符串|没有 |  - |按资源 ID 过滤 |

### 响应 - `200 OK`

```json
{
  "data": [
    {
      "id": "evt_01hwxyz...",
      "type": "sandbox.created",
      "actor": {
        "type": "api_key",
        "id": "user_42"
      },
      "resource": {
        "type": "sandbox",
        "id": "sbx_abc123"
      },
      "ts": "2026-05-26T10:14:23Z",
      "metadata": {
        "image": "debian-12-sandbox-v8",
        "workspace_id": "ws_01hwxyz..."
      }
    }
  ],
  "next_cursor": "MjAyNi0wNS0yNlQxMDoxNDoyM1o"
}
```

|领域|类型 |描述 |
|--------|------|-------------|
| `data[].id` |字符串|唯一的事件ID |
| `data[].type` |字符串|点命名空间事件类型 |
| `data[].actor.type` |字符串| `"user"` 或 `"api_key"` |
| `data[].actor.id` |字符串| API 关键参与者的用户 ID 或 `external_user_id` |
| `data[].resource.type` |字符串|资源类型（例如 `sandbox`、`computer`、`webhook`）|
| `data[].resource.id` |字符串|资源 ID |
| `data[].ts` |字符串| ISO 8601 时间戳 |
| `data[].metadata` |对象|特定于事件的详细信息字段 |
| `next_cursor` |字符串|不透明光标；没有更多页时省略 |

### 常见事件类型

|类型 |触发|
|------|---------|
| `sandbox.created` |沙箱配置|
| `sandbox.destroyed` |沙盒已删除 |
| `sandbox.exec` | `POST /exec` 称为 |
| `computer.created` |计算机配置|
| `computer.stopped` |计算机停止运行 |
| `computer.deleted` |电脑已删除 |
| `snapshot.created` |快照已启动 |
| `api_key.created` | API 密钥已生成 |
| `api_key.deleted` | API 密钥已撤销 |
| `tenant.preview_domain.set` |预览域已配置 |
| `webhook.created` |已创建传出 Webhook |
| `quota.upserted` |每用户配额设置 |
| `settings.updated` |租户设置已更改 |

使用带前缀的 `type` 过滤器来缩小事件范围（例如 `type=sandbox.` 返回所有沙箱事件）。

---

## 分页

审核事件按最新顺序返回。向前翻页：

1. 发出不带 `after` 的初始请求。
2. 如果存在 `next_cursor`，则使用 `after={next_cursor}` 重新发出。
3. 当`next_cursor`不存在时停止。

游标是编码时间戳的不透明 base64url 字符串。不要解析或构造它们。

---

## 错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 |  - |无效的光标格式 |

---

## 示例

**curl**

```bash
# First page
curl "https://api.miosa.ai/api/v1/audit-log?limit=50&type=sandbox." \
  -H "Authorization: Bearer msk_u_..."

# Next page
curl "https://api.miosa.ai/api/v1/audit-log?limit=50&after=MjAyNi0wNS0yNlQxMDoxNDoyM1o" \
  -H "Authorization: Bearer msk_u_..."

# Filter by resource
curl "https://api.miosa.ai/api/v1/audit-log?resource_type=computer&resource_id=cmp_xyz789" \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Page through all sandbox events in the last page
cursor = None
while True:
    page = client.audit_log.list(
        type="sandbox.",
        limit=100,
        after=cursor,
    )
    for event in page.data:
        print(event.type, event.resource.id, event.ts)
    if not page.next_cursor:
        break
    cursor = page.next_cursor
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

let cursor: string | undefined;

do {
  const page = await client.auditLog.list({
    type: 'sandbox.',
    limit: 100,
    after: cursor,
  });

  for (const event of page.data) {
    console.log(event.type, event.resource.id, event.ts);
  }

  cursor = page.nextCursor;
} while (cursor);
```
