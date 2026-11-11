---
slug: "services/quickstart"
title: "服务快速入门"
description: "将仓库部署到可访问的 URL，添加 Postgres 和 Redis，添加 worker，绑定域名，读取失败的发布并回滚。"
section: "Services & projects"
order: 1005
---

本演练会将一个仓库部署到可访问的 URL，然后添加数据库、worker 和域名。
每个步骤都提供 CLI 形式和 SDK 形式。

## 开始之前

- 一个拥有 API 密钥（`msk_...`）的 MIOSA 账户，或已登录的 CLI。
- 一个项目。`miosa project create shop --workspace-id <workspace>` 可创建一个；`miosa project list` 会显示其他项目。
- 一个 MIOSA 可以克隆的仓库，或一个容器镜像。

先导出密钥：

```bash
export MIOSA_API_KEY=msk_your_key
```

## 1. 创建 Web 服务

```bash
miosa service create shop API --type web \
  --repo https://github.com/acme/api \
  --branch main \
  --start-command "node server.js" \
  --replicas 2 \
  --env NODE_ENV=production
```

`--type web` 会存储 API 的 `server` 类型，该类型可以拥有公共地址。
`--type private` 存储相同类型，但永远不会获得公共地址。
命令会打印服务、其私有地址，以及启用后的公共地址。

TypeScript：

```typescript
const service = await miosa.services.create(projectId, {
  name: "API",
  type: "web",
  source: { type: "git", ref: "https://github.com/acme/api", branch: "main" },
  settings: { start_command: "node server.js", replicas: 2 },
});
const address = await miosa.services.enablePublicAddress(service.id);
console.log(address.url); // https://api.acme.miosa.app
```

Python：

```python
service = miosa.services.create(
    project_id,
    "API",
    type="web",
    source={"type": "git", "ref": "https://github.com/acme/api", "branch": "main"},
    settings={"start_command": "node server.js", "replicas": 2},
)
address = miosa.services.enable_public_address(service["id"])
print(address["url"])
```

## 2. 部署

```bash
miosa service deploy shop API --message "first deploy" --wait
```

`--wait` 会轮询发布状态，直到其上线或失败。
不传该标志时，你会得到排队中的发布，并可用 `miosa service releases shop API` 查看。

```typescript
const release = await miosa.services.deploy(service.id, { message: "first deploy" });
let current = release;
while (!["active", "sleeping", "failed", "canceled", "stopped", "removed"].includes(current.state)) {
  await new Promise((resolve) => setTimeout(resolve, 3000));
  current = await miosa.services.release(service.id, release.id);
}
console.log(current.state);
```

Python SDK 提供 `wait_for_release(service_id, release_id)`，它会轮询并在发布失败时抛出异常。

一次发布的状态依次为 `queued -> building -> built -> starting -> health_check -> active`。
任一步骤都可能失败；发布对象会携带 `reason.code`，例如 `build_failed`、`healthcheck_timeout` 或 `no_capacity`。

## 3. 添加 Postgres 和 Redis

模板会在一个变更集中创建服务、其变量以及它们之间的引用：

```bash
miosa service templates                       # the catalog
miosa service templates apply shop nextjs-postgres
miosa service templates apply shop redis
```

Postgres 模板会创建一个数据库服务和一个 Web 服务，后者的 `DATABASE_URL` 为引用 `${{Postgres.DATABASE_URL}}`。
应用模板会暂存一个变更集，并部署其创建的所有服务；传入 `--no-deploy` 可只创建而不部署。

之后要读取连接字符串：

```bash
miosa service connection shop Postgres
```

该调用需要 `databases:read`，并会接受审计，因为 URL 中包含密码。

## 4. 添加 worker

worker 没有端口，也没有地址。

```bash
miosa service create shop Worker --type worker \
  --repo https://github.com/acme/api \
  --start-command "node worker.js" \
  --replicas 1
miosa service deploy shop Worker
```

worker 可以使用相同的引用读取同一个数据库：

```bash
miosa service vars set shop Worker DATABASE_URL='${{Postgres.DATABASE_URL}}'
```

## 5. 绑定域名

Web 服务或静态站点会获得一个 `miosa.app` 上的公共地址：

```bash
miosa service domains shop API
miosa service domains enable shop API --label api
```

结果为 `https://api.<your-org>.miosa.app`，由活动发布的健康副本提供服务。
要在不删除服务的情况下停止公开提供服务，请移除地址：

```bash
miosa service domains disable shop API
```

服务之间的私有网络尚不可用，因此其他服务会通过其公共地址调用它；数据库通过其连接变量访问，请参阅[地址与域名](/docs/services/domains)。

## 6. 读取失败的发布

```bash
miosa service releases shop API
miosa service logs shop API
```

`logs` 会打印最新发布的状态转换及其失败原因。
传入 `--release <number>` 可查看较早的发布。
控制台的 Logs 标签页会显示每个副本的运行时日志和构建日志；请参阅[日志](/docs/services/logs)。

## 7. 回滚

回滚会使用较早发布的构建镜像，以及其冻结的源和设置来提供服务。
不会重新构建，因此速度很快。

```bash
miosa service rollback shop API 3 --yes
```

`miosa service promote shop API 3` 是同一个调用：Services API 将这项操作称为 rollback。

## 8. 清理

```bash
miosa service stop shop API --yes      # stop now, keep the service
miosa service delete shop API --yes    # remove it and stop its replicas
```

删除数据库服务也会删除其数据，因此需要使用 `--confirm <exact name>` 而不是 `--yes`。
请参阅[删除数据库服务](/docs/services/cli#deleting-a-database-service)。

## 接下来去哪里

- [服务与类型](/docs/services/services)：每种源和设置。
- [变量](/docs/services/variables)：引用和自动生成的密钥。
- [发布](/docs/services/releases)：状态机以及每种原因的含义。
- [CLI](/docs/services/cli) 和 [SDK](/docs/services/sdk)：完整命令和方法列表。
