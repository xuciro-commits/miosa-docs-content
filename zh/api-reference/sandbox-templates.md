---
slug: "api-reference/sandbox-templates"
title: "沙箱模板 API"
description: "在一个沙箱映像内暂存内容的模板、作为别名的旧映像 ID，以及如何从模板创建沙箱。"
section: "API reference"
order: 12026
---

沙箱模板是预构建的映像，其中包含工具链、依赖项和配置。从模板启动比在运行时安装包更快，并确保可重现的环境。

每个沙箱都启动相同的主映像。模板是在创建时在该图像内上演的内容，而不是单独的图像。请参阅[模板指南](/docs/develop/templates)。

> **注意**
>
> 模板是由 MIOSA 管理的只读市场项目。要构建您自己的自定义模板，请参阅[模板指南](/docs/develop/templates)。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/sandbox-templates` |列出您可用的模板 |
| `GET` | `/api/v1/sandbox-templates/{id}` |获取特定模板 |
| `POST` | `/api/v1/sandboxes` |从模板创建沙箱（通过 `template_id`）|

---

## 列出模板

**`GET /api/v1/sandbox-templates`**

以 JSON 数组的形式返回内置目录以及您的工作区已构建的任何自定义模板。
每行都包含模板 `slug`、其 `source`（`builtin` 或 `workspace`）以及注册表 `status`。

默认情况下，别名 id 被忽略。通过 `include_aliases=true` 将它们包括在内；每个别名都嵌套在它解析到的行上，位于 `aliases` 下。

```bash
curl "https://api.miosa.ai/api/v1/sandbox-templates?include_aliases=true" \
  -H "Authorization: Bearer msk_u_..."
```

---

## 列出的模板

列出了五个模板，所有五个模板都运行一个图像：

| `template_id` |类型 |它是什么 |
|---|---|---|
| `miosa-sandbox` | `image` |默认沙箱。里面的所有内容，以及省略 `template_id` 时得到的内容。 |
| `nextjs` | `starter` |在沙箱中暂存一个 Next.js 应用并在端口 3000 上启动它。 |
| `nextjs-postgres` | `starter` |端口 3000 上带有 Postgres 客户端工具的 Next.js 应用程序；绑定`DATABASE_URL`。 |
| `static-html` | `starter` |静态 HTML、CSS 和 JavaScript，从 `/workspace` 在端口 8000 上提供服务。 |
| `fastapi-auth` | `starter` |一个 FastAPI 启动器，在端口 8000 上具有注册、登录、me 和仪表板路由。其运行状况检查尚不工作。 |

四个入门模板仅在映像内添加内容：脚手架、安装命令和启动命令。
您的工作区构建的模板以 `custom` 类型返回。
`miosa-desktop` 是一个单独的计算机产品，不是沙箱模板；请参阅[计算机](/docs/computers/overview)。

---

## 别名

别名 ID 永远不会列出，并且仍然以 `template_id` 的形式工作。

这些解析为 `miosa-sandbox`，纯图像：

`miosa-sandbox-prod-1`、`miosa-sandbox-docker`、`agent-browser`、`instant-code`、`bookworm`、`debian-12`、`docker`、`sandbox-docker`、`browser-agent`、`code`、 `instant`、`instant-node`、`node-20`、`postgres-15`、`postgres-16`、`ubuntu-22.04`、`ubuntu-24.04`。

这些解析为一个启动器，该启动器保留在列表中：

`static` 到 `static-html`； `next-postgres` 和 `fullstack-next-postgres` 到 `nextjs-postgres`； `python-auth` 和 `fastapi-auth-starter` 到 `fastapi-auth`。

---

## 从模板创建沙箱

**`POST /api/v1/sandboxes`**

```json
{
  "template_id": "nextjs",
  "size": "small"
}
```

对于没有任何暂存内容的沙箱，请省略 `template_id`，或者传递内容模板以在同一图像内暂存支架。
对于桌面，使用 `template_type: "miosa-desktop"` 创建一台计算机；请参阅[计算机](/docs/computers/overview)。

请参阅 [Sandboxes API](/docs/api-reference/sandboxes) 以获取完整的创建参考。

---

## 错误

|状态 |代码|原因 |
|--------|------|--------|
| 404 | `not_found` |模板 ID 不存在 |

---

## 示例

**curl**

```bash
# List templates, including the legacy alias ids
curl "https://api.miosa.ai/api/v1/sandbox-templates?include_aliases=true" \
  -H "Authorization: Bearer msk_u_..."

# Get one template
curl https://api.miosa.ai/api/v1/sandbox-templates/nextjs \
  -H "Authorization: Bearer msk_u_..."

# Create a sandbox with the Next.js template staged
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"template_id": "nextjs", "size": "small"}'
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Browse templates
templates = client.sandbox_templates.list()
for t in templates:
    print(t.slug, "-", t.description)

# Create the default sandbox
sandbox = client.sandboxes.create(template_id="nextjs", size="small")
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Browse
const templates = await client.sandboxTemplates.list();
for (const t of templates) {
  console.log(t.slug, '-', t.description);
}

// Create a sandbox with the Next.js template staged
const sandbox = await client.sandboxes.create({
  templateId: 'nextjs',
  size: 'small',
});
```
