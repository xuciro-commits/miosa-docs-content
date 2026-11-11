---
slug: "develop/templates"
title: "模板"
description: "一个沙盒映像、在其中暂存内容的模板、桌面以及如何启动 Next.js、FastAPI 或静态项目。"
section: "Sandboxes"
order: 2015
---

**模板**是在创建时在沙箱内暂存的内容：文件、脚手架、额外的包以及启动应用程序的命令。
模板从来都不是一个单独的图像。

每个沙箱都启动相同的主映像，其中已经包含 Node、Python、带有 `docker compose` 和 `docker buildx` 的 Node、Python、Docker Engine 29.9、虚拟显示器上带有 CDP 的实时浏览器以及内部的代码解释器。

## 新沙箱

创建一个没有代码、命令和工件模板的沙箱。
当您希望在同一图像内放置支架时，请使用 `template_id`。

**Python**

```python
import miosa

client = miosa.Miosa(api_key="msk_u_...")

sandbox = client.sandboxes.create()
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const miosa = new Miosa({ apiKey: 'msk_u_...' });

const sandbox = await miosa.sandboxes.create();
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{}'
```

## 从模板开始

传递 `template_id` 在沙箱图像内暂存一个脚手架并启动它。
该模板提供运行应用程序的文件、包和命令；它不是一个单独的图像。

| `template_id` | 它会准备什么 |
|---|---|
| `nextjs` | 一个 Next.js 应用，在端口 3000 上启动。 |
| `nextjs-postgres` |端口 3000 上带有 Postgres 客户端工具的 Next.js 应用程序；绑定`DATABASE_URL`。 |
| `static-html` | 静态 HTML、CSS 和 JavaScript，由端口 8000 上的 `/workspace` 提供服务。 |
| `fastapi-auth` |一个 FastAPI 启动器，在端口 8000 上具有注册、登录、我和仪表板路由。它的运行状况检查尚不工作，因此不要在其上设置您自己的准备情况。 |
|你自己的 |您的工作区构建的任何自定义模板。请参阅[自定义模板](/docs/develop/templates#custom-templates)。 |

**Python**

```python
sandbox = client.sandboxes.create(
    template_id="nextjs",
    auto_start=True,
)
```

**打字稿**

```typescript
const sandbox = await miosa.sandboxes.create({
  templateId: 'nextjs',
  autoStart: true,
});
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{ "template_id": "nextjs", "auto_start": true }'
```

使用 `auto_start`，MIOSA 在沙箱到达 `running` 后启动模板，并且响应携带预览 URL。
如果你想先将自己的文件写入`/workspace`，请保留`false`，然后调用[启动模板应用程序](/docs/api-reference/sandboxes/create#start-a-template-app)。

### Next.js

**端口 3000。** Next.js SSR 部署需要 `next.config.js` 中的 `output: "standalone"`，以便构建器可以打包服务器。
静态导出 (`output: "export"`) 无需它即可工作。

或者对您创建的没有 `auto_start` 的沙箱手动执行此操作：

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/template/start \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "install_command": "npm install",
    "start_command": "npm run dev -- --hostname 0.0.0.0 --port 3000",
    "port": 3000
  }'
```

### 快速API

**端口 8000.** 或者手动执行：

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/template/start \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "install_command": "pip install -r requirements.txt",
    "start_command": "uvicorn main:app --host 0.0.0.0 --port 8000",
    "port": 8000
  }'
```

### 静态 HTML

**端口 8000。** 纯 HTML、CSS 和 JavaScript，无需构建步骤。
或者手动完成：

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/template/start \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "start_command": "python -m http.server 8000 --bind 0.0.0.0",
    "port": 8000
  }'
```

构建器检测静态站点（没有 `package.json`、`Cargo.toml` 或 `go.mod`）并将包 `/workspace` 作为静态版本，因此从边缘提供资源，没有运行时实例。

> **注意：旧图像 ID**
>
> 较旧的映像 ID 仍以 `template_id` 的方式工作并启动主映像。
> [沙盒模板 API](/docs/api-reference/sandbox-templates) 列出了它们。

---

## 新桌面

耐用的 Linux 桌面（Xfce 和 Firefox），具有屏幕截图、点击、输入和流式传输以及人工接管功能。
它是一台带有 `template_type: "miosa-desktop"` 的 **计算机**，而不是沙箱模板。

**Python**

```python
computer = client.computers.create(
    name="my-desktop",
    template_type="miosa-desktop",
    size="small",
)
```

**打字稿**

```typescript
const computer = await miosa.computers.create({
  name: 'my-desktop',
  templateType: 'miosa-desktop',
  size: 'small',
});
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/computers \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{ "name": "my-desktop", "template_type": "miosa-desktop", "size": "small" }'
```

请参阅[计算机](/docs/computers/overview)了解桌面表面，并[嵌入实时桌面](/docs/computers/embedding)将其放入您的产品中。

---

## 自定义模板

当内置目录不适合时 - 专有工具链、特定 CUDA/GPU 堆栈、预加载数据集 - 您可以构建并注册您自己的模板。

1. 为您的图像写入 `Dockerfile`。
2. 使用自定义沙箱模板 API（或 CLI：`miosa template build`）构建它。
3. 验证映像是否正确启动。
4. 注册 slug，以便可以在 `template_id` 中选择它。

请参阅[自定义模板 API 参考](/docs/api-reference/sandboxes)以获取完整的端点规范，包括图像上传、冒烟测试挂钩和版本控制。

---

## 构建规范底漆

**BuildSpec** 是一个可选的 JSON 文档，您可以附加到沙箱或部署以覆盖默认的安装和启动命令。当默认值不太正确时，它是任何模板的逃生舱口：
```json
{
  "install": "pnpm install --frozen-lockfile",
  "build": "pnpm build",
  "start": "node .next/standalone/server.js",
  "env": {
    "NODE_ENV": "production",
    "PORT": "3000"
  }
}
```

在沙箱创建时或发布时传递 BuildSpec。构建时命令（`install`、`build`）在构建器内运行； `start` 命令确定运行时实例的启动方式。此处设置的环境变量将与部署上已有的任何变量合并。

---

- [沙箱](/docs/develop/sandboxes)：完整的沙箱生命周期 - 创建、执行、文件、快照、预览。- [发布](/docs/docker-host/publishing)：构建管道如何将沙箱转变为实时部署。

- [API 参考：沙箱](/docs/api-reference/sandboxes)：template_id、安装/启动命令和 BuildSpec 的有线格式。
