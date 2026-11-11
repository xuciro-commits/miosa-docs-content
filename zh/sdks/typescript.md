---
slug: "sdks/typescript"
title: "TypeScript SDK"
description: "安装、验证并使用 @miosa/sdk 3.7.1——沙箱、exec、文件、快照、计算机、智能体和类型化错误。"
section: "CLI, SDKs & MCP"
order: 11015
---

MIOSA 官方 TypeScript/JavaScript SDK。适用于 Node.js 18+、Deno、Bun 和现代打包工具。完全类型化，不使用 `any`。仅提供具名导出；没有默认导出。

## 安装

**npm**

```bash
npm install @miosa/sdk
```

**pnpm**

```bash
pnpm add @miosa/sdk
```

**Yarn**

```bash
yarn add @miosa/sdk
```

**Bun**

```bash
bun add @miosa/sdk
```

**Deno**

```typescript
import { Miosa } from "npm:@miosa/sdk";
```

两个子路径导出增加了辅助函数和计算机使用工具集：

```typescript
import { Miosa } from "@miosa/sdk";
import { /* helpers */ } from "@miosa/sdk/helpers";
import { miosaComputerToolset } from "@miosa/sdk/computer-use";
```

## 身份验证

```typescript
import { Miosa } from "@miosa/sdk";

const client = new Miosa({ apiKey: "msk_u_..." });
```

类名为 `Miosa`。构造函数选项：`apiKey`（或 `MIOSA_API_KEY` 环境变量）、`accessToken`（用于切换组织的用户 JWT）、`tenant`（`MIOSA_TENANT`）、`baseUrl`（`MIOSA_BASE_URL`，默认 `https://api.miosa.ai/api/v1`）、`timeout`（默认 30 秒）和 `maxRetries`（默认 3）。

`apiKey` 和 `accessToken` 互斥：同时传入会抛出异常。可以使用 `client.withTenant(tenant)` 或 `client.withOptions(opts)` 派生一个限定范围的客户端，两者都会返回新的客户端。

```typescript
const tenant = client.withTenant("acme");
await client.whoami();
```

## 沙箱

```typescript
const sbx = await client.sandboxes.create({ name: "my-box", size: "small" });
await sbx.waitUntilReady();
```

| 方法 | 作用 |
|---|---|
| `client.sandboxes.create(params?)` | 创建沙箱 |
| `client.sandboxes.list(params?)` | 列出沙箱 |
| `client.sandboxes.get(id)` | 获取单个沙箱 |
| `client.sandboxes.getByName(name)` | 按名称获取 |
| `client.sandboxes.getOrCreate(params)` | 复用或创建 |
| `client.sandboxes.listPage(params?)` | 带元数据的分页列表 |
| `client.sandboxes.delete(id)` | 按 ID 删除 |

`Sandbox.stop()` 位于 TypeScript SDK 3.7.5 中，该版本已打标签但尚未发布到 npm；在发布之前，请使用 `POST /api/v1/sandboxes/{id}/stop` 停止沙箱。

`Sandbox` 句柄增加了 `destroy()` 和 `delete()`，以及 `pause()`、`resume()`、`fork()`、`extend(seconds)`、`expose(port)`、`getUrl(port, path)`、`usage()` 和 `waitUntilReady()`。

## Exec 与文件

沙箱 exec 可以直接调用，也可以流式传输：

```typescript
const r = await sbx.exec("node -v");            // { stdout, stderr, exitCode }
await sbx.exec.run("npm test", { cwd: "/workspace" });
for await (const ev of sbx.exec.stream("npm run dev")) console.log(ev);
await sbx.commands.execute("echo hi");          // POST /commands, ready-gated
```

计算机 exec 是一个类，不可直接调用：

```typescript
await computer.exec.bash("ls -la");
await computer.exec.python("print(2 + 2)");
const h = computer.exec.stream({ /* ... */ });
```

沙箱文件和计算机文件使用不同的方法名：

```typescript
await sbx.files.write("/workspace/app.ts", code);
await sbx.files.readText("/workspace/app.ts");
await sbx.files.list("/workspace");
await sbx.files.remove("/workspace/old");       // sandbox: "remove"

await computer.files.writeFile("/home/user/notes.txt", "hi");
await computer.files.readFile("/home/user/notes.txt");
await computer.files.readdir("/home/user");
await computer.files.delete("/home/user/old");  // computer: "delete"
```

沙箱文件方法：`write`、`read`、`readText`、`list`、`mkdir`、`remove`、`rename`、`copy`、`chmod`、`stat`、`tree`、`writeMany`、`uploadFile`、`download`。

## 快照

三个接口层面：账户级、按沙箱、按计算机。

```typescript
await client.snapshots.list();
await client.snapshots.named.create({ name: "before-upgrade", /* ... */ });
await sbx.snapshots.create("before deps");
await computer.checkpoints.create();
```

| 接口 | 方法 |
|---|---|
| `client.snapshots` | `list`、`groups`、`get`、`tree`、`download`、`fork`、`delete`、`deleteMany`、`deleteHistory` |
| `client.snapshots.named` | `list`、`create`、`get`、`remove`、`deploy` |
| `sbx.snapshots` | `create`、`list`、`restore`、`delete` |
| `computer.checkpoints` | `create`、`list`、`get`、`delete`、`restore` |

## 计算机

```typescript
const computer = await client.computers.create({ name: "dev-box" });
await computer.start();
await computer.waitUntilReady();
```

| 方法 | 作用 |
|---|---|
| `client.computers.create(params)` | 创建（返回处于 `creating` 状态的计算机；调用 `start()`） |
| `client.computers.list(params?)` | 列出计算机 |
| `client.computers.get(id)` | 获取单个 |
| `client.computers.update(id, params)` | 更新 |
| `client.computers.delete(id)` | 删除 |

`Computer` 句柄用于驱动桌面：`screenshot()` 返回 PNG `Uint8Array`（base64 请使用 `screenshotBase64()`），此外还有 `click`、`rightClick`、`doubleClick`、`moveMouse`、`type`、`key`、`hotkey`、`scroll`、`drag`、`launch(app)`、`windows()` 和 `act(action, ...args)`。

## 智能体

定义和运行是分开的接口。

```typescript
await client.agents.list();
await client.agents.create({ /* ... */ });
await client.agents.publish(id, configuration);

const run = await sbx.prompt("fix the failing test");
await computer.run("write a README");
```

| 接口 | 用途 |
|---|---|
| `client.agents` | 智能体定义：`list`、`get`、`create`、`update`、`publish`、`archive` |
| `sbx.prompt(...)`、`computer.prompt(...)` | 在机器上提示智能体（Agent Runs API） |
| `sbx.run(...)`、`computer.run(...)` | 分发运行（Runs API） |
| `client.runs` | `run`、`list`、`get`、`cancel`、`waitForCompletion`、`outputs`、`files`、`messages`、`diagnostics`、`usage` |
| `client.agentRuns` | `run`、`list`、`get`、`cancel`、`waitForCompletion`、`streamEvents`、`artifacts`、`downloadArtifact` |

## 错误

每个 HTTP 失败都会抛出 `MiosaError`：

```typescript
import { Miosa, RateLimitError } from "@miosa/sdk";

try {
  await client.sandboxes.get("missing");
} catch (err) {
  if (err instanceof RateLimitError) console.log(err.retryAfterMs);
}
```

`MiosaError` 携带 `status`、`code`、`details`、`requestId` 和 `retryable`。请根据 `error.code` 分支，而不是 `error.status`。所有类型化子类（`AuthError`、`NotFoundError`、`RateLimitError`、`InsufficientCreditsError`、`ValidationError`、`TimeoutError`、`EnvironmentNotFoundError`、`SnapshotInUseError` 等）都从包根导出。
## 此 SDK 中的服务与数据

除了沙箱和计算机，同一个客户端也可以访问服务和数据接口。下面的方法名对应 3.8.0 版本：

```ts
// settings as code, replicas and rolling deploys
await miosa.services.update(serviceId, {
  settings: { shape: "dedicated-4", spread: "auto", strategy: "rolling" },
  lock_version: 3,
});

// a pull request preview per repository
await miosa.services.setPreviewSettings(projectId, {
  repository: "acme/api", enabled: true, secret: "whsec_...",
});
const { series, available } = await miosa.services.metrics(projectId, serviceId, "24h");

// log drains
const drain = await miosa.services.createLogDrain(projectId, environmentId, {
  name: "ops", url: "https://logs.example.test/ingest",
});

// public database endpoint
await miosa.databases.enablePublicNetworking(databaseId, ["203.0.113.0/24"]);
```

防护和 WAF 对应 `services.setProtection`、`services.shareLink`、`services.setWaf` 和 `services.setAttackMode`。
