---
slug: "cli/platform"
title: "平台资源"
description: "miosa CLI 中的部署、数据库、存储、卷、工作流程、计划、功能、模板、主机、锻造、云、出口和 AI 网关。"
section: "CLI, SDKs & MCP"
order: 11008
---

## 部署

```bash
miosa deploy create --from-sandbox <sandbox> --wait
miosa deploy create --git <url> --branch main --wait
miosa deploy create --dir ./dist --name my-site
miosa deploy                          # deploy the current directory
miosa deploy list
miosa deploy get <deployment>
miosa deploy logs <deployment> -n 200
miosa deploy env <deployment>          # environment variable names
miosa deploy env-set <deployment> --set KEY=VALUE
miosa deploy redeploy <deployment>
miosa deploy rollback <deployment> --to <version-id>
miosa deploy versions <deployment>
miosa deploy releases <deployment>
miosa deploy observe <deployment>
miosa deploy builds <deployment>       # build logs
miosa deploy requests <deployment>
miosa deploy promote <deployment>
miosa deploy stop <deployment> --yes
miosa deploy rm <deployment> --yes
miosa deploy domains <deployment>
miosa deploy domain-add <deployment> <domain>
miosa deploy domain-verify <deployment> <domain>
miosa deploy domain-rm <deployment> <domain>
```

`miosa deploy rollback --to` 返回指定版本；没有它，就是上一个。

`miosa deploy env` 列出部署的环境变量名称。 `miosa deploy env-set <deployment> --set KEY=VALUE` 设置它们，并且值在静态时被加密。 `--set` 是可重复的。

```bash
miosa deploy env my-app
miosa deploy env-set my-app --set DATABASE_URL=postgres://... --set LOG_LEVEL=info
```

### 创建部署

`miosa deploy create`（别名 `miosa deploy publish`）发布新部署并使其生效。给它一个来源：

- `--from-sandbox <sandbox>` 发布正在运行的沙箱的输出。添加 `--port <n>` 和 `--run-command` 来发布正在运行的服务器，`--entrypoint <file>` 来选择静态发布的入口文件，或者 `--type docker_deploy` 为 Docker Host。
- `--git <url>` 连接存储库并将其第一个构建排队。添加 `--branch` 和 `--build-command`。
- `--dir <path>` 打包本地目录并上传，用于没有存储库的预构建静态输出。

对于 `--from-sandbox`，`--dir` 是应用程序在沙箱内的路径（默认 `/workspace`）。 `--dir` 本身就是一个要上传的本地目录。

```bash
miosa deploy create --from-sandbox my-box --wait
miosa deploy create --from-sandbox my-box --port 3000 --run-command "npm start"
miosa deploy create --git https://github.com/me/app --branch main --wait
miosa deploy create --dir ./dist --name my-site
```

常用标志：`--name`、`--env KEY=VALUE`（可重复，静态加密）、`--domain` 和 `--wait` 阻止，直到部署上线并打印其公共 URL。 `--json` 将结果打印为 JSON。

`miosa deploy` 不带子命令部署当前目录：目录为 1 时为 git 存储库，否则为目录上传。除非您通过 `--yes`，否则它会在发布之前询问。

## 数据库

```bash
miosa db list
miosa db get <db>
miosa db connection <db>        # connection string
miosa db logs <db>
miosa db backups <db>
miosa db backup <db>
miosa db restart <db>
miosa db start <db>
miosa db stop <db> --yes
miosa db wake <db>
miosa db rm <db> --yes
```

## 存储

对象存储桶。

```bash
miosa storage list
miosa storage create assets --visibility private --region <region>
miosa storage get assets
miosa storage objects assets --prefix uploads/ -n 50
miosa storage rm assets --yes
```

## 卷

机器之间共享的持久卷。

```bash
miosa volume list
miosa volume create data --size-gb 20
miosa volume get <volume>
miosa volume rm <volume> --yes
```

## 函数

边缘函数。

```bash
miosa function list
miosa function get <function>
miosa function invoke <function> -d '{"key":"value"}'
miosa function rm <function> --yes
```

## 工作流程

由步骤、智能体和工具组成的自动化。

```bash
miosa workflow list
miosa workflow get <workflow>
miosa workflow run <workflow> --input '{"branch":"main"}'
miosa workflow runs <workflow>
miosa workflow run-get <workflow> <run>
miosa workflow run-cancel <workflow> <run>
miosa workflow pause <workflow> / resume <workflow>
miosa workflow rm <workflow> --yes
```

## 时间表

预定的工作。

```bash
miosa schedule list
miosa schedule get <schedule>
miosa schedule executions <schedule>
miosa schedule run <schedule>
miosa schedule pause <schedule> / resume <schedule>
miosa schedule rm <schedule> --yes
```

## 模板

```bash
miosa template list
miosa template get <template>
miosa template build <template>
miosa template builds <template>
miosa template build-logs <template> <build>
miosa template custom <template>
miosa template promote <template>
miosa template rollback <template> --yes
miosa template warm-pool
miosa template rm <template> --yes
```

## 批次

在一个请求中创建多个沙箱。

```bash
miosa batch new --count 50 --size small --name-prefix ci --wait
miosa batch status <batch>
miosa batch items <batch> --limit 20
miosa batch watch <batch> --timeout 15m
miosa batch cancel <batch>
```

`batch new` 采用 `--count`、`--size`、`--template`、`--name-prefix`、`--region`、`--ttl`、`--idle-timeout`、`--tag KEY=VALUE`、`--concurrency`、 `--workspace`、`--wait` 和 `--wait-timeout`。

## 项目和工作区

```bash
miosa project list
miosa project create <name> --description "..." --workspace-id <id>
miosa project get <project>

miosa workspace list
miosa workspace create <slug>
miosa workspace get <slug>
miosa workspace members <slug>
miosa workspace stats <slug>
miosa workspace usage <slug>
miosa workspace inventory <slug>
miosa workspace activity <slug>
miosa workspace use <slug>
miosa workspace delete <slug>
```

## 目录和区域

```bash
miosa catalog --product sandbox --template nextjs
miosa catalog --state fast_ready
miosa catalog --output json | jq '.data[] | select(.state == "fast_ready")'
miosa regions
```

`miosa catalog` 是产品、模板、尺寸和准备情况的规范目录。

## 锻造（测试版）

```bash
miosa forge repo list
miosa forge repo create platform --visibility private
miosa forge pr new platform --head fix-redirect --title "Fix redirect loop"
miosa forge pr review platform 12 --decision approved
miosa forge pr merge platform 12
miosa forge check report platform --commit <sha> --name unit-tests --conclusion passed --file evidence.json
miosa forge release new platform --tag v1.2.0
miosa forge policy set platform --branch main --approvals 2 --check unit-tests
```

Forge 存储库命令始终返回 `{ "data": ... }` 成功信封。存储库突变接受 `--idempotency-key`，删除需要交互确认或 `--yes`。 Forge 克隆只有在 MIOSA 提供凭据帮助程序流程（将凭据排除在 URL、参数、shell 历史记录、日志和持久 Git 配置之外）之前才会公开。

## 出口

沙箱的出站网络策略。

```bash
miosa egress policies
miosa egress allowlist --policy <id>
miosa egress suggestions
miosa egress audit -n 50
```

## 你自己的主机

```bash
miosa host register studio --platform macos    # prints the host key once
miosa host list --tag studio
miosa host get <host>
miosa host metrics <host>
miosa host runtime <host>
miosa host services <host>
miosa host jobs <host>
miosa host ssh-key
miosa host secret
miosa host tag <host>
miosa host app catalog
miosa host cluster ...
miosa host group ...
miosa host fs ...
miosa host rm <host> --yes
```

`miosa host register --platform` 采用 `macos`、`linux` 或 `windows`。

### 容器

主机上的容器。

```bash
miosa host container list <host>
miosa host container run <host> --image nginx:alpine --name web --set PORT=8080
miosa host container get <host> <container>
miosa host container logs <host> <container>
miosa host container stats <host> <container>
miosa host container start <host> <container>
miosa host container stop <host> <container>
miosa host container restart <host> <container>
miosa host container rm <host> <container> --force --yes
```

`miosa host container logs` 和 `miosa host container stats` 一直流式传输，直到连接结束或按下 Ctrl-C。对于 `--json`，每个事件都是每行一个 JSON 对象。 `miosa host container run` 采用 `--image`、`--name`、`--cmd`（可重复）、`--set KEY=VALUE`（可重复）和 `--restart`。

### 网格

通过 WireGuard 的私有加密主机到主机网络。

```bash
miosa host mesh list
miosa host mesh new lab --host <host> --host <host>
miosa host mesh get lab
miosa host mesh add-host lab <host>
miosa host mesh remove-host lab <host> --yes
miosa host mesh events lab
miosa host mesh rm lab --yes
```

`miosa host mesh events` 流式传输主机连接、密钥轮换和状态更改，直到按 Ctrl-C。

### GitHub Actions 跑步者

主机上的自托管 GitHub Actions 运行器，注册一个存储库。

```bash
miosa host gha-runner list <host>
miosa host gha-runner new <host> --repo https://github.com/acme/app --token <token>
miosa host gha-runner new <host> --repo https://github.com/acme/app --token-secret GH_TOKEN --ephemeral
miosa host gha-runner get <host> <runner>
miosa host gha-runner events <host> <runner>
miosa host gha-runner refresh <host> <runner>
miosa host gha-runner rm <host> <runner> --yes
```

`miosa host gha-runner new` 采用 `--repo`、`--token`（发送到主机，不存储）或 `--token-secret`（持有令牌的主机密钥的名称）、`--name`、`--label`（可重复）和 `--ephemeral`（在一项作业后取消注册）。 `miosa host gha-runner events` 在跑步者拾取并完成作业时进行流式传输。

### 备份和快照

主机上目录的计划备份。配置指定目录、计划以及要保留的快照数量。快照可以分叉到主机上。

```bash
miosa host backup list <host>
miosa host backup new <host> --name nightly --path /data --schedule "0 3 * * *" --keep 7
miosa host backup update <host> nightly --schedule "0 4 * * *"
miosa host backup trigger <host> nightly
miosa host backup pause <host> nightly / resume <host> nightly
miosa host backup snapshot list <host> nightly
miosa host backup snapshot get <snapshot>
miosa host backup snapshot fork <snapshot> --to-host <host> --to-path /restore
miosa host backup snapshot rm <snapshot> --yes
miosa host backup rm <host> nightly --yes
```

`miosa host backup new` 采用 `--path`、`--schedule`（cron 表达式）、`--timezone`、`--keep`（多少个快照）、`--keep-days`、`--exclude`（可重复的 glob）和 `--compression` (0-9)。 `miosa host backup snapshot list` 采用 `--limit` 和 `--before`。

### 隧道

```bash
miosa host tunnel new <host> --port 3000 --access organization
```公开某个地址的主机端口。 `--access` 是 `organization`（默认）或 `public`，`--slug` 设置地址名称。

## 你自己的云

自带云 (BYOC)。帐户、区域和主机池。

```bash
miosa cloud account new prod --provider aws      # also --provider gcp
miosa cloud accounts
miosa cloud region ...
miosa cloud regions
miosa cloud pool provision <pool> --count 1
miosa cloud pools
miosa cloud nodes
miosa cloud capabilities
miosa cloud preflight run --account <id> --region <id>
miosa cloud preflights
```

> **注意**
>
> 自带云和主机配置是私人预览，需要为您激活主机配置。

## 人工智能网关

对路由策略、预算、限制和请求跟踪进行建模。

```bash
miosa gateway policy new fast --primary-model claude-haiku --alias fast
miosa gateway policy new ... --strategy weighted --fallback-model <model>
miosa gateway budget show
miosa gateway budget ...
miosa gateway limit ...
miosa gateway health
miosa gateway provider-health
miosa gateway environments
miosa gateway requests --model <model> --status <status>
miosa gateway request <id>
miosa gateway settings ...
```

## 原始 API

对于比高级命令更广泛的功能，请使用具有相同经过身份验证的配置的 `miosa api`。请参阅[原始 API 访问](/docs/cli/api)。
