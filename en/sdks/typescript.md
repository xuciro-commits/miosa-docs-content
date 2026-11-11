---
slug: "sdks/typescript"
title: "TypeScript SDK"
description: "Install, authenticate and use @miosa/sdk 3.7.1 - sandboxes, exec, files, snapshots, computers, agents and typed errors."
section: "CLI, SDKs & MCP"
order: 11015
---

The official TypeScript/JavaScript SDK for MIOSA. Works with Node.js 18+, Deno, Bun and modern bundlers. Fully typed, with no `any`. Named exports only; there is no default export.

## Install

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

Two subpath exports add helpers and the computer-use toolset:

```typescript
import { Miosa } from "@miosa/sdk";
import { /* helpers */ } from "@miosa/sdk/helpers";
import { miosaComputerToolset } from "@miosa/sdk/computer-use";
```

## Authentication

```typescript
import { Miosa } from "@miosa/sdk";

const client = new Miosa({ apiKey: "msk_u_..." });
```

The class is `Miosa`. Constructor options: `apiKey` (or the `MIOSA_API_KEY` env var), `accessToken` (a user JWT for org switching), `tenant` (`MIOSA_TENANT`), `baseUrl` (`MIOSA_BASE_URL`, default `https://api.miosa.ai/api/v1`), `timeout` (default 30s) and `maxRetries` (default 3).

`apiKey` and `accessToken` are mutually exclusive: passing both throws. Derive a scoped client with `client.withTenant(tenant)` or `client.withOptions(opts)`, both of which return a new client.

```typescript
const tenant = client.withTenant("acme");
await client.whoami();
```

## Sandboxes

```typescript
const sbx = await client.sandboxes.create({ name: "my-box", size: "small" });
await sbx.waitUntilReady();
```

| Method | What it does |
|---|---|
| `client.sandboxes.create(params?)` | Create a sandbox |
| `client.sandboxes.list(params?)` | List sandboxes |
| `client.sandboxes.get(id)` | Get one sandbox |
| `client.sandboxes.getByName(name)` | Get by name |
| `client.sandboxes.getOrCreate(params)` | Reuse or create |
| `client.sandboxes.listPage(params?)` | Paged list with meta |
| `client.sandboxes.delete(id)` | Delete by id |

`Sandbox.stop()` is in the TypeScript SDK 3.7.5, which is tagged but not yet published to npm; until it is, stop with `POST /api/v1/sandboxes/{id}/stop`.

The `Sandbox` handle adds `destroy()` and `delete()`, plus `pause()`, `resume()`, `fork()`, `extend(seconds)`, `expose(port)`, `getUrl(port, path)`, `usage()` and `waitUntilReady()`.

## Exec and files

Sandbox exec is callable and streamable:

```typescript
const r = await sbx.exec("node -v");            // { stdout, stderr, exitCode }
await sbx.exec.run("npm test", { cwd: "/workspace" });
for await (const ev of sbx.exec.stream("npm run dev")) console.log(ev);
await sbx.commands.execute("echo hi");          // POST /commands, ready-gated
```

Computer exec is a class, not callable:

```typescript
await computer.exec.bash("ls -la");
await computer.exec.python("print(2 + 2)");
const h = computer.exec.stream({ /* ... */ });
```

Sandbox files and computer files use different names:

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

Sandbox file methods: `write`, `read`, `readText`, `list`, `mkdir`, `remove`, `rename`, `copy`, `chmod`, `stat`, `tree`, `writeMany`, `uploadFile`, `download`.

## Snapshots

Three surfaces: account-wide, per sandbox, per computer.

```typescript
await client.snapshots.list();
await client.snapshots.named.create({ name: "before-upgrade", /* ... */ });
await sbx.snapshots.create("before deps");
await computer.checkpoints.create();
```

| Surface | Methods |
|---|---|
| `client.snapshots` | `list`, `groups`, `get`, `tree`, `download`, `fork`, `delete`, `deleteMany`, `deleteHistory` |
| `client.snapshots.named` | `list`, `create`, `get`, `remove`, `deploy` |
| `sbx.snapshots` | `create`, `list`, `restore`, `delete` |
| `computer.checkpoints` | `create`, `list`, `get`, `delete`, `restore` |

## Computers

```typescript
const computer = await client.computers.create({ name: "dev-box" });
await computer.start();
await computer.waitUntilReady();
```

| Method | What it does |
|---|---|
| `client.computers.create(params)` | Create (returns a computer in `creating` state; call `start()`) |
| `client.computers.list(params?)` | List computers |
| `client.computers.get(id)` | Get one |
| `client.computers.update(id, params)` | Update |
| `client.computers.delete(id)` | Delete |

The `Computer` handle drives the desktop: `screenshot()` returns PNG `Uint8Array` (use `screenshotBase64()` for base64), plus `click`, `rightClick`, `doubleClick`, `moveMouse`, `type`, `key`, `hotkey`, `scroll`, `drag`, `launch(app)`, `windows()` and `act(action, ...args)`.

## Agents

Definitions and runs are separate surfaces.

```typescript
await client.agents.list();
await client.agents.create({ /* ... */ });
await client.agents.publish(id, configuration);

const run = await sbx.prompt("fix the failing test");
await computer.run("write a README");
```

| Surface | Purpose |
|---|---|
| `client.agents` | Agent definitions: `list`, `get`, `create`, `update`, `publish`, `archive` |
| `sbx.prompt(...)`, `computer.prompt(...)` | Prompt an agent on a machine (Agent Runs API) |
| `sbx.run(...)`, `computer.run(...)` | Dispatch a run (Runs API) |
| `client.runs` | `run`, `list`, `get`, `cancel`, `waitForCompletion`, `outputs`, `files`, `messages`, `diagnostics`, `usage` |
| `client.agentRuns` | `run`, `list`, `get`, `cancel`, `waitForCompletion`, `streamEvents`, `artifacts`, `downloadArtifact` |

## Errors

Every HTTP failure throws `MiosaError`:

```typescript
import { Miosa, RateLimitError } from "@miosa/sdk";

try {
  await client.sandboxes.get("missing");
} catch (err) {
  if (err instanceof RateLimitError) console.log(err.retryAfterMs);
}
```

`MiosaError` carries `status`, `code`, `details`, `requestId` and `retryable`. Branch on `error.code`, not `error.status`. All typed subclasses (`AuthError`, `NotFoundError`, `RateLimitError`, `InsufficientCreditsError`, `ValidationError`, `TimeoutError`, `EnvironmentNotFoundError`, `SnapshotInUseError` and many more) are exported from the package root.
## Services and data, in this SDK

Beyond sandboxes and computers, the same client reaches the Services and data
surfaces. The method names below are the ones in 3.8.0:

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

Protection and WAF are `services.setProtection`, `services.shareLink`,
`services.setWaf` and `services.setAttackMode`.
