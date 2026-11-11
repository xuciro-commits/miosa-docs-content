---
slug: "migrate/railway"
title: "从Railway搬家"
description: "将Railway项目带到 MIOSA，其中包括一份经过审查的计划、每个环境都有自己的部署目标、已停止创建的服务以及一份保留的内容和您手动完成的内容的准确列表。"
section: "Services & projects"
order: 1013
---

`miosa import railway` 通过 Railway 的公共 API 读取 Railway 项目并编写计划。
该计划列出了 MIOSA 项目、每个 Railway 环境一个 Deploy 目标，以及每个目标中它将创建的服务、数据库、变量名称和共享变量。
它还列出了它不能携带的所有物品。
在您审查计划并应用它之前，任何内容都不会被创建，除非您提出要求，否则应用不会启动任何内容。

此页面是关于哪些内容可以保留、哪些内容不能保留以及如何处理这些秘密的参考。

## 开始之前

- Railway 令牌：Railway 帐户设置中的帐户、工作区或项目令牌。
  `--project` 获取项目的 id，或者当登录可以列出它时它的名称；如果 Railway 拒绝，则导入将打印 Railway 的消息并询问 ID。
  MIOSA 读取它，从不存储它，也从不将其写入计划。
  如果您使用 Railway CLI 登录，`--via railway-cli` 会读取该会话，因此不会有令牌通过 MIOSA。
- MIOSA CLI，`@miosa/cli` 2.13.4 或更高版本：

```bash
npm install -g @miosa/cli
miosa --version
```

- 当您的服务从 GitHub 存储库构建时，GitHub 连接到 MIOSA：

```bash
miosa connections add apps github
```

- 假设Railway项目中的每个秘密都已泄露。
  Railway 于 2026 年 10 月 21 日披露，其秘密已被暴露，因此导入会复制名称，而不是值，除非您明确提出其他要求。

## 步骤

```bash
export RAILWAY_TOKEN=...                                          # or --token-file, or a prompt
miosa import railway --project acme-api --environment all         # review only; nothing is created
miosa import railway --project acme-api --environment all --out plan.json
miosa import railway --apply --plan plan.json --region us-east    # creates everything, stopped
miosa service vars set acme-api api STRIPE_KEY=<new value> --deploy-target production
miosa service deploy acme-api api --deploy-target production
```

`--environment` 采用一个环境、多个 (`production,staging`) 或 `all`。
默认值为 `production`。

审核会打印将在每个部署目标中创建的内容、每个设置的来源、哪些变量需要值以及“手动完成”列表以及每个项目的下一步。
凡是不能携带的领域都会出现在那里；没有什么会悄然掉落。

## 适用的是什么

- 它创建项目，然后为每个Railway环境部署一个目标，重用已存在的目标。
- 它在其环境的 Deploy 目标中创建每个服务，因此 `production` 和 `development` 保留单独的服务、变量和设置。
- 它创建了每项服务**停止**。
  在使用 `miosa service deploy` 部署它或使用 `--deploy` 应用之前，不会构建或运行任何内容。
- `--deploy` 在 GitHub 副本准备就绪后启动每个具有源的服务。
  当任何变量没有值时它会拒绝，因此服务永远不会以空秘密启动。
- 除非您传递 `--attach-domains`，否则它**不会**附加自定义域。
  在导入期间附加活动域会在 MIOSA 服务运行之前移动其流量。
  如果没有该标志，审核会将每个域列出为已跳过，并且摘要会打印稍后附加它的命令。
- 它通过您的 GitHub 连接链接 GitHub 存储库（私有或公共）。
  MIOSA 将存储库复制到 Forge 中并从副本进行构建，因此服务上不会存储任何凭证。
- 它在创建任何内容之前检查所有内容。
  如果 GitHub 未连接，则会以 `Connect GitHub first: miosa connections add apps github` 停止。
  如果您的连接无法读取存储库，它将命名该存储库并停止。
  如果您的组织还无法从存储库构建服务，它会这样说并停止。
  在每种情况下都不会创建任何东西。
- 再次运行 apply 仅创建缺少的内容：首先查找每个项目、部署目标和服务。
- 它为每项服务提供了 MIOSA 接受的名称和名称。
  诸如 `@acme/web` 之类的包名称将变为 `acme-web`，并对其进行引用。
  长度超过 40 个字符的 slug 会使用名称的短哈希进行缩短，每次运行都相同。

## 继承什么

| Railway | MIOSA |
|---|---|
| 项目 | 项目 |
| 环境 | 同名的 Deploy 目标，拥有自己的服务和变量 |
| 服务 | 环境所属 Deploy 目标中的 Web 服务、Worker 或 Schedule |
| 存储库、分支、根目录 | 服务来源；GitHub 仓库通过你的 GitHub 连接关联 |
| Dockerfile 路径 | `source.dockerfile`，相对于根目录 |
| 构建命令 | `source.build_command` |
| 启动命令 | `start_command` |
| 预部署命令（例如 `alembic upgrade head`、`drizzle-kit migrate`） | `pre_deploy_command` |
| 健康检查路径与超时 | `healthcheck_path` 和 `healthcheck_timeout_s` |
| 重启策略与重试次数 | `restart_policy` 和 `max_retries` |
| 副本数 | `replicas`（1 到 50） |
| Cron 计划 | 一个 Schedule 服务 |
| `railway.json`、`railway.toml` | 通过 Railway 从仓库读取；文件优先于服务设置，与 Railway 上的行为一致 |
| 变量 | 名称，构建时可见，与 Railway 上一致；`${{Service.VAR}}` 引用会在 MIOSA 名称不同处被改写 |
| 共享变量 | 同一 Deploy 目标的共享变量（按名称） |
| Postgres、Redis、MySQL 模板镜像和插件 | 带托管数据库的数据库服务 |
| 域名目标端口 | 服务端口，健康检查和公网地址都使用它 |
| 自定义域名 | 会列出；用 `--attach-domains` 附加，一旦其 DNS 指向此处，就由 MIOSA 提供服务 |

每个设置都取自第一个位置：存储库的配置文件，然后是服务设置，然后是最新部署运行的内容。
该计划记录了每种设置的哪一个。

### 参考文献

MIOSA 使用与 Railway 相同的 `${{Service.NAME}}` 引用语法。
导入重写了不同的名称：

| Railway 中的叫法 | 对应为 |
|---|---|
| `${{Redis.REDISHOST}}`、`REDISPORT`、`REDISPASSWORD` | `${{Redis.REDIS_HOST}}`、`REDIS_PORT`、`REDIS_PASSWORD` |
| `${{Redis.REDISUSER}}` | `default` |
| `${{Redis.REDIS_URL}}` |不变|
| `${{Postgres.POSTGRES_USER}}`、`POSTGRES_PASSWORD`、`POSTGRES_DB` | `${{Postgres.PGUSER}}`、`PGPASSWORD`、`PGDATABASE` |
| `${{Postgres.DATABASE_URL}}`、`PGHOST`、`PGPORT`、`PGUSER`、`PGPASSWORD`、`PGDATABASE` |不变|
| `${{api.RAILWAY_PRIVATE_DOMAIN}}` | `${{api.MIOSA_PRIVATE_DOMAIN}}` (`api.miosa.internal`) |
| `RAILWAY_SERVICE_NAME`、`RAILWAY_SERVICE_ID`、`RAILWAY_ENVIRONMENT_NAME`、`RAILWAY_ENVIRONMENT_ID`、`RAILWAY_PROJECT_ID`、`RAILWAY_PROJECT_NAME`、`RAILWAY_REGION` | `MIOSA_` 名称 |

如今，由于服务之间的专用网络已关闭，MIOSA 上的服务无法通过其专用域访问。
审查以这种方式标记到达另一个服务的每个变量；将其指向 MIOSA 可以到达的地址（对于 SMTP，您的邮件提供商）。

评论中列出了没有 MIOSA 等效项的参考文献，例如 `RAILWAY_PUBLIC_DOMAIN`、`REDIS_PUBLIC_URL` 或 `DATABASE_PUBLIC_URL`。
该变量在创建时没有值，您可以在导入后设置它。

不导入Railway自身的变量(`RAILWAY_*`)； MIOSA 注入自己的 `MIOSA_*` 名称。

### 构建可见变量

Railway 向构建公开每个服务变量，因此框架在构建时内联 `NEXT_PUBLIC_*`、`VITE_*` 和 `PUBLIC_*` 值。
导入保留了这一点：每个导入的变量和共享变量都是创建为构建可见的，并且评论会说明有多少个。
`MIOSA_*` 和 `RAILPACK_*` 名称仅在运行时保留，因为 MIOSA 保留它们。
当您使用 `--build`; 设置它时，没有值创建的变量将变为构建可见。摘要打印该命令。

在 MIOSA 上，构建可见变量作为 BuildKit 秘密到达构建，因此它永远不会存储在图像层中。
例外情况是 Dockerfile 声明了同名的 `ARG`：该变量作为构建参数传递，就像在 Railway 上一样，并且使用它的 `RUN` 步骤记录在映像历史记录中。

要仅在运行时创建每个变量，请在编写计划时传递 `--runtime-only-vars`。
您可以稍后更改：

```bash
miosa service vars set <project> <service> NEXT_PUBLIC_API_URL=https://api.example.com --build
miosa service vars set <project> <service> STRIPE_KEY=<new value> --no-build
```

## 什么不保留

| Railway | 需要做什么 |
|---|---|
| 变量值 | 在 MIOSA 上设置新的值；见下文 |
| 美国以外的区域（例如 `asia-southeast1`、`europe-west4`） | MIOSA 运行在 `us-east`、`us-west` 和 `us-mia`；审核清单会列出每个此类服务，apply 时需要 `--region` |
| 构建器（Nixpacks） | 若仓库中有 Dockerfile，MIOSA 会构建 Dockerfile，否则用 Railpack 检测技术栈；请检查首次构建 |
| 监视模式 | MIOSA 在每次 push 时构建；审核清单会列出这些模式 |
| 空闲休眠 | 不迁移；导入后设置 `sleep` 设置 |
| 数据库数据和卷内容 | 不复制；审核清单会列出每个卷及其大小，数据由你自行迁移 |
| 服务卷 | 会连同 `miosa volume create` 命令和需要挂载的挂载路径一起列出 |
| `*.up.railway.app` 域名 | Railway 自有的 DNS；MIOSA 分配 `<label>.<org>.miosa.app` |
| 用 Railway CLI 上传的服务（没有仓库或镜像） | 创建时没有来源；用 `miosa service deploy <project> <service> --dir <path>` 上传代码 |
| MongoDB 插件 | 用 `miosa db create --engine mongodb` 创建 MongoDB 数据库 |

## 轮换秘密

当变量没有值时，Apply 拒绝运行，除非您传递 `--allow-missing-values`，这会将这些变量创建为空。
不管怎样，服务都会停止，所以任何事情都不会从丢失或泄露的秘密开始。

1. 假设Railway项目中的每个秘密都已泄露。
2.导入时不要复制值；在 MIOSA 上设置新的。
3. 首先轮换提供程序（数据库密码、Stripe、SMTP、API 密钥），然后将新值粘贴到 MIOSA 中。
4. 设置新值后进行部署。
5. MIOSA 通车后，删除旧的Railway项目。

```bash
miosa service vars set <project> <service> STRIPE_KEY=<new value> --deploy-target production
miosa project vars set <project> APP_NAME=<value> --deploy-target production
miosa service deploy <project> <service> --deploy-target production
```

`--copy-variables` 存在于不敏感的测试项目中。
它会打印相同的警告，并且它永远不是默认值。

## 如果你只有一个配置文件

`--from-file` 在本地读取 `railway.json` 或 `railway.toml`，没有令牌，也没有网络调用：

```bash
miosa import railway --from-file ./railway.toml --service-name api
```

该计划没有来源，因为配置文件没有来源；在申请之前将一项添加到计划中。

## 撤消

apply 创建的每个 id 在创建时都会被记录。

```bash
miosa import railway --cleanup --state .miosa/import-railway-acme-api.json --yes
```

`--cleanup` 准确打印它将首先删除的内容，并仅删除那些 id。
中断的应用以 `--resume` 继续。

## Railway术语、MIOSA 名称

| Railway 的说法 | MIOSA 的叫法 |
|---|---|
|项目|项目|
|环境 |部署目标 |
|服务 |服务（Web 服务、Worker、私人服务、时间表）|
|插件、数据库模板 |数据库服务|
|部署 |发布 |
|变量|变量|
|共享变量 |共享变量 |
|卷 |卷 |
|域名 |公共地址或自定义域 |
