---
slug: "learn/first-production-app"
title: "构建并部署 Next.js 应用程序"
description: "使用 miosa CLI 2.x 和 SDK 在沙盒中构建一个小型 Next.js 应用程序、预览它、将其发布为不可变的部署、证明它是实时的、连接自定义域并安全回滚。"
section: "Get started"
order: 7
---

本教程将一个小型 Next.js 应用程序从空沙箱转移到经过验证的生产 URL。

您将创建一个沙箱，编写应用程序，查看临时预览，发布不可变版本，验证稳定的部署 URL，连接 `app.example.com`，并在需要之前学习回滚。

现在就做

部署您的第一个应用程序

将存储库或沙箱转变为具有不可变版本和回滚功能的稳定生产 URL。

- [新部署](https://miosa.ai/deploy/new)：（在新选项卡中打开 MIOSA 控制台）

miosa 部署创建 --from-sandbox my-box --wait

发布沙箱、git 存储库 (--git) 或本地目录 (--dir)。为 App Engine 添加 --type docker_deploy。

下一篇 [发布指南](/docs/docker-host/publishing)

> **注意：本教程保留的内容**
>
> 发布不会破坏或转换沙箱。
> 沙箱仍然是一个开发环境，而发布和部署则成为独立的生产资源。

## 你将创建什么

|资源 |示例|目的|
|---|---|---|
|沙盒| `hello-nextjs` |创建和测试应用程序的可变位置。 |
|预览 | API返回的临时URL |在发布之前检查实时开发服务器。 |
|部署| `hello-nextjs` |稳定的生产身份和版本历史。 |
|平台网址 | API 返回 `public_url` |无需客户 DNS 即可立即进行生产访问。 |
|自定义域名 | `app.example.com` |附加到部署的可选客户拥有的主机名。 |

## 开始之前

您需要：

- 有权访问预期组织和工作空间的 MIOSA 用户。
- 仅存储在服务器或终端环境中的工作区 API 密钥。
- 安装了 MIOSA CLI（v2.2 或更高版本）。
- 创建沙箱、发布部署和配置域的权限。
- 仅当您完成自定义域部分时才能访问 DNS 提供商。

```bash
export MIOSA_API_KEY="msk_u_..."
miosa whoami
```

在创建任何内容之前，请确认响应标识了预期的组织和工作空间。

> **警告：不要在错误的组织中继续**
>
> 资源 ID 不能替代检查活动客户边界。
> 如果 CLI 报告错误的组织或工作区，请在创建沙箱或部署之前更正该上下文。

## 第 1 步：创建沙箱

**命令行界面**

```bash
miosa create hello-nextjs \
  --template nextjs \
  --ttl 1h \
  --wait \
  --json
```

保存返回的Sandbox id：

```bash
export SANDBOX_ID="<sandbox-id>"
```

**打字稿**

```typescript
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const sandbox = await miosa.sandboxes.create({
  name: "hello-nextjs",
  templateId: "nextjs",
});
console.log(sandbox.id);
```

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sandbox = miosa.sandboxes.create(name="hello-nextjs", template_id="nextjs")
print(sandbox.id)
```

**预期结果：**沙箱达到可用的运行状态。

如果创建仍处于待处理状态，请在重试之前检查返回的容量或就绪原因。
当第一个请求仍在处理时，请勿创建重复的沙箱。

## 步骤 2：创建 Next.js 应用程序

在沙盒内搭建应用程序，然后替换 `app/page.tsx`。

**命令行界面**

```bash
miosa exec hello-nextjs --cwd /workspace -- bash -lc 'pnpm create next-app@latest hello-nextjs --typescript --eslint --app --no-tailwind --use-pnpm --yes'
```

在本地编写页面文件，然后将其复制到沙箱中：

```bash
miosa files cp ./page.tsx hello-nextjs:/workspace/hello-nextjs/app/page.tsx
```

构建它：

```bash
miosa exec hello-nextjs --cwd /workspace/hello-nextjs --timeout 300 -- pnpm build
```

**打字稿**

```typescript
const page = `export default function Home() {
  return (
    <main style={{ minHeight: "100vh", display: "grid", placeItems: "center" }}>
      <section style={{ textAlign: "center" }}>
        <p style={{ color: "#a1a1aa" }}>Deployed with MIOSA</p>
        <h1 style={{ fontSize: "clamp(3rem, 10vw, 7rem)", margin: 0 }}>Hello from MIOSA</h1>
      </section>
    </main>
  );
}`;

await sandbox.exec.run(
  "cd /workspace && pnpm create next-app@latest hello-nextjs --typescript --eslint --app --no-tailwind --use-pnpm --yes",
);
await sandbox.files.write("/workspace/hello-nextjs/app/page.tsx", page);

for await (const event of sandbox.exec.stream(
  "cd /workspace/hello-nextjs && pnpm build",
)) {
  console.log(event);
}
```

**Python**

```python
page = '''export default function Home() {
  return (
    <main style={{ minHeight: "100vh", display: "grid", placeItems: "center" }}>
      <h1>Hello from MIOSA</h1>
    </main>
  );
}'''

sandbox.exec.run(
    "cd /workspace && pnpm create next-app@latest hello-nextjs --typescript --eslint --app --no-tailwind --use-pnpm --yes",
)
sandbox.files.write_file("/workspace/hello-nextjs/app/page.tsx", page)

for event in sandbox.exec_stream("cd /workspace/hello-nextjs && pnpm build"):
    print(event)
```

**预期结果：** 构建命令退出，状态为 `0` 并且 Next.js 报告构建成功。

## 步骤 3：启动并预览应用程序

将服务器作为后台进程启动，以便在 CLI 断开连接后继续运行，然后公开端口 3000。

**命令行界面**

```bash
miosa process start hello-nextjs \
  --name web \
  --cwd /workspace/hello-nextjs \
  -- pnpm start --hostname 0.0.0.0 --port 3000

miosa preview create hello-nextjs 3000 --name web --json
```

**打字稿**

```typescript
const preview = await sandbox.previews.create(3000);
console.log(preview.url);
```

**Python**

```python
preview = sandbox.previews.create(port=3000, label="app")
print(preview["url"])
```

打开 MIOSA 返回的确切预览 URL。

**预期结果：**页面显示“Hello from MIOSA”。

> **提示：预览不是生产**
>
> 预览反映了当前的沙箱进程和文件。
> 它可以立即改变，并且可以在沙箱停止时消失。
> 稳定的生产 URL 在发布期间创建。

## 步骤 4：发布部署

`miosa deploy create` 发布一个新的部署并使其生效。
将沙箱作为其源、应用程序在沙箱内的路径以及为其提供服务的命令。

**命令行界面**

```bash
miosa deploy create \
  --from-sandbox hello-nextjs \
  --name hello-nextjs \
  --dir /workspace/hello-nextjs \
  --port 3000 \
  --run-command "pnpm start --hostname 0.0.0.0 --port 3000" \
  --wait \
  --json
```

添加 `--type docker_deploy` 以通过 Docker Host 而不是默认运行时发布。

从响应中保存这些值：

```bash
export DEPLOYMENT_ID="<deployment-id>"
export PUBLIC_URL="<deployment-public-url>"
```

**打字稿**

```typescript
const result = await sandbox.deploy({
  name: "hello-nextjs",
  path: "/workspace/hello-nextjs",
  port: 3000,
  runCommand: "pnpm start --hostname 0.0.0.0 --port 3000",
});

// Read public_url from the response. Do not construct a production hostname.
console.log(result);
```

**Python**

```python
result = sandbox.deploy(
    name="hello-nextjs",
    path="/workspace/hello-nextjs",
    port=3000,
    run_command="pnpm start --hostname 0.0.0.0 --port 3000",
)

# Read public_url from the response. Do not construct a production hostname.
print(result)
```

始终使用返回的 `public_url`。
不要从 slug、组织名称或工作空间名称构造生产主机名。**预期结果：** 响应标识部署、候选版本、升级版本和 `public_url`。

## 步骤 5：证明生产

成功的构建还不够。
确认部署已解析为实时版本并且公共 URL 已响应。

```bash
miosa deploy get hello-nextjs --json
curl --fail --silent --show-error "$PUBLIC_URL" | grep "Hello from MIOSA"
miosa deploy observe hello-nextjs
```

**预期结果：** 部署报告带有 `public_url` 的运行状态，公共响应包含“Hello from MIOSA”，并且 `observe` 显示请求。

## 步骤 6：连接自定义域

此步骤是可选的。
配置 DNS 时，平台 URL 保持有效。

```bash
miosa deploy domain-add hello-nextjs app.example.com
```

MIOSA 显示所需的 DNS 记录和验证目标。
将这些值准确复制到 DNS 提供商中。

对于普通的子域，记录通常具有以下形状：

|类型 |主持人|价值|智能体|
|---|---|---|---|
| `CNAME` | `app` | MIOSA展示的目标 |验证完成之前禁用 |

不要逐字复制 `example.com`。
使用由客户控制的主机名和实时 MIOSA 域流中显示的目标。

检查公共 DNS，然后验证：

```bash
dig app.example.com CNAME +short
miosa deploy domain-verify hello-nextjs <domain-id>
```

**预期结果：** 域通过 DNS 验证、证书颁发和路由进行移动，直到 HTTPS 处于活动状态。

> **注意：通配符域和精确域是不同的**
>
> 组织级部署域可以在一个父域下生成许多应用程序主机名。
> 精确的自定义域将一个选定的主机名直接附加到一个部署。
> 按照您选择的域类型显示的记录进行操作。
> 请参阅[域](/docs/docker-host/domains)。

## 步骤 7：安全发布更新

更改 `app/page.tsx` 中的标题，在沙盒中重建，然后再次发布。

```bash
miosa exec hello-nextjs --cwd /workspace/hello-nextjs --timeout 300 -- pnpm build

miosa deploy create \
  --from-sandbox hello-nextjs \
  --name hello-nextjs \
  --dir /workspace/hello-nextjs \
  --port 3000 \
  --run-command "pnpm start --hostname 0.0.0.0 --port 3000" \
  --wait \
  --json
```

重用 `--name` 以现有部署为目标，因此在升级新版本时会保留其身份、稳定 URL、域、绑定和发布历史记录。
新版本升级后再次运行证明步骤。

## 步骤 8：在需要之前了解回滚

列出部署版本：

```bash
miosa deploy versions hello-nextjs --json
```

如果新版本不健康，则返回到之前准备好的版本：

```bash
miosa deploy rollback hello-nextjs --to <version-id>
```

回滚可在几秒钟内重新指向活动版本，而无需重建。
它不会更改部署身份或需要新域。
请参阅[回滚](/docs/docker-host/rollback)。

## 常见故障及恢复

|症状|可能的原因 |安全行动|修复后证明|
|---|---|---|---|
|沙箱创建仍悬而未决|请求的容量尚未准备好或请求仍在处理中。 |检查就绪原因并等待或选择兼容容量。 |原始沙箱达到可用状态。 |
|预览返回错误 | Next.js 进程已停止、绑定到错误的接口或使用其他端口。 |使用 `miosa process start` 在 `0.0.0.0:3000` 上启动它并重新创建预览。 |预览返回“Hello from MIOSA”。 |
|发布报告 Docker Host 不可用 |工作区 Docker 主机 主机未处于活动状态。 |确认工作区具有活动的 Docker Host 主机，然后重试相同的 `miosa deploy create ... --type docker_deploy`。 | Publish 返回一个 Deployment 和 `public_url`。 |
|构建成功但应用程序未上线 |促销、容器运行状况或路由证明尚未完成。 |检查发布结果并运行 `miosa deploy get` 和 `miosa deploy observe`。 |公共 URL 以预期内容进行响应。 |
| DNS 仍待处理 |记录尚未传播、使用错误的主机或指向错误的目标。 |将公共 DNS 与 MIOSA 显示的确切记录进行比较，并在验证期间保持智能体禁用。 | `miosa deploy domain-verify` 报告 DNS 已验证且 TLS 处于活动状态。 |
|自定义主机名打开另一个应用程序 |主机名附加在错误的组织或错误的部署中。 |再次更改 DNS 之前，停止并检查所有权和路由绑定。 |该域解析为预期的部署 ID 和活动版本。 |
|新发布导致回归 |候选人通过了构建，但应用程序行为不正确。 |回滚到之前准备好的版本。 |稳定的 URL 提供以前已知的良好内容。 |

## 完成清单- [ ] 沙盒存在于预期的组织和工作空间中。
- [ ] Next.js 生产构建成功退出。
- [ ] 预览显示“Hello from MIOSA”。
- [ ] 发布返回部署 ID 和 API 选择的 `public_url`。
- [ ] 公共 URL 返回预期内容。
- [ ] 自定义域已验证且 TLS 处于活动状态（如果已配置）。
- [ ] 可以识别较早准备好的版本以进行回滚。
- [ ] 可以在不删除部署的情况下删除沙箱。

## 从这里继续

- [添加 Postgres](/docs/data/postgres)：绑定持久应用程序数据，而不将数据库放入应用程序容器内。
- [添加秘密](/docs/security/secrets)：提供敏感的运行时配置，而不将其提交到源。
- [了解发布](/docs/docker-host/publishing)：了解完整的发布、版本、推广、证明和重试合约。
- [操作域](/docs/docker-host/domains)：配置部署域、精确自定义域、验证、TLS 和恢复。

## 后续步骤

- [从沙箱发布](/docs/docker-host/publishing)：静态、动态和 App Engine 发布并提供证明。
- [附加自定义域](/docs/docker-host/domains)：DNS验证、TLS和路由缓存。
- [以秒为单位回滚](/docs/docker-host/rollback)：将部署重新指向旧版本，无需重建。

- [返回学习路径](/docs/learn)：接下来选择智能体、桌面或白标旅程。
