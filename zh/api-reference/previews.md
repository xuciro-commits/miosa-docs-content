---
slug: "api-reference/previews"
title: "预览并发布API"
description: "将沙箱端口公开为预览 URL，将沙箱发布到稳定部署，并将正在运行的沙箱提升到部署路由。"
section: "API reference"
order: 12017
---

基本路径：`/api/v1/sandboxes/{id}`

> **注意**
>
> 对于预览令牌铸造（在没有 API 密钥的情况下在 iframe 中嵌入沙箱），请参阅 [预览令牌 API](/docs/api-reference/preview-tokens)。

## 公开预览端口

**`POST /api/v1/sandboxes/{id}/expose`**

返回沙箱内运行的服务器的公共、租户感知的预览 URL。当智能体生成 Vite/Next/FastAPI/Flask 应用程序并在本地端口上启动它时使用此选项。

在沙箱内，将开发服务器绑定到 `0.0.0.0`，而不是 `localhost`：
```bash
npm run dev -- --host 0.0.0.0 --port 5173
python -m http.server 8000 --bind 0.0.0.0
```

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `port` |整数 |没有 |沙箱内的端口要公开。如果省略，MIOSA 将使用模板生命周期预览端口（如果可用）。如果提供的话，必须是 `1` 到 `65535`。 |

### 响应 - `200 OK`
```json
{
  "url": "https://5173-sbx01j9x.sandbox.miosa.ai"
}
```

如果租户配置了白标签预览域，则同一终端节点将返回该域而不是平台默认值。默认应用程序端口（例如 `3000`、`5173`、`8080`、`8000` 和 `80` 可能会返回为 `https://{slug}.sandbox.{domain}`；非默认端口使用 `https://{port}-{slug}.sandbox.{domain}`。

### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `MISSING_PARAM` | `port` 被省略，沙盒模板没有默认预览端口。 |
| 409 | `SANDBOX_NOT_RUNNING`|沙盒不处于 `running` 状态。 |
| 422 | `INVALID_PORT`|端口位于 `1` 到 `65535` 之外。 |
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/expose \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"port": 5173}'
```

对于 PDF、图像、Markdown、CSV 或 ZIP 文件等静态工件，请将它们写入 `/workspace` 下，并通过文件 API 下载它们。对于 Web 应用程序，启动服务器并公开端口。

---

## 发布沙箱

**`POST /api/v1/sandboxes/{id}/publish`**

冻结沙箱源，创建不可变的发布和部署版本，验证候选版本，并将其提升到稳定的部署 URL 后面。
使用此端点进行生产发布。

CLI 是推荐的快乐路径：

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name <deployment> \
  --type docker_deploy \
  --wait \
  --json
```

每次发布时使用相同的 `--name`，以便 MIOSA 保留部署 ID 和规范 URL。

请参阅 [Publishing](/docs/docker-host/publishing) 了解请求选项、证明和失败行为。

## 将沙盒运行时提升为部署

**`POST /api/v1/sandboxes/{id}/deploy`**

此兼容性桥将持久部署 URL 直接路由到正在运行的沙箱。
沙箱保留路线后面的运行时间，并且其自动销毁计时器被取消。

仅当沙箱本身必须保持运行时时才使用它。
对于不可变的生产版本和 Docker Host，请使用 `/publish`。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `name` |字符串|是的 |人类可读的部署名称。用于创建部署 slug。 |
| `port` |整数 |没有 |要路由的运行时端口。默认为模板生命周期预览端口，然后是 `80`。 |
| `domain` |字符串|没有 |要附加的自定义域。 |
| `custom_domain` |字符串|没有 | `domain` 的别名，由前端客户端使用。 |

### 响应 - `201 Created`
```json
{
  "deployment_id": "dep_01j9xr2t4fk8me3n5q",
  "url": "https://my-app-a1b2c3.acme.miosa.app",
  "state": "running"
}
```

托管部署 URL 属于租户范围：`https://{deployment-slug}.{tenant-slug}.miosa.app`。 MIOSA 在部署上保留沙箱运行时目标并协调运行路由，因此可以从数据库修复临时智能体重新启动或管理 API 中断。

### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `MISSING_PARAM` | `name` 被省略。 |
| 409 | `SANDBOX_NOT_RUNNING`|沙盒不处于 `running` 状态。 |
| 409 | `SANDBOX_RUNTIME_UNAVAILABLE`|沙盒已标记为正在运行，但还没有虚拟机 IP。 |
| 422 | `INVALID_PORT`|端口位于 `1` 到 `65535` 之外。 |
| 422 | `VALIDATION_ERROR`|部署记录验证失败。 |
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/deploy \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"my-app","port":8000}'
```
