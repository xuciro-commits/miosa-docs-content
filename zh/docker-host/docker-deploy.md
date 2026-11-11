---
slug: "docker-host/docker-deploy"
title: "Docker主机"
description: "在一台始终在线的工作区主机上运行许多小型应用程序，而 MIOSA 拥有路由、域和归属。"
section: "Services & projects"
order: 1023
---

Docker Host 是 MIOSA 的专用工作区主机模型，适用于需要许多小型应用程序、渠道、铅磁体、API 或客户端站点的客户，而无需为每次部署创建新的生产虚拟机。
这是生成的应用程序的推荐生产路径，应在工作区中 24/7 运行。
MIOSA 将路由、域和归属作为事实来源，主机提供执行能力。

## 1. 确保工作区主机

```bash
miosa api POST /docker-deploy/hosts/ensure -d '{"workspace_id": "<workspace-id>"}'
miosa api GET /docker-deploy/hosts?workspace_id=<workspace-id>
```

主机从`provisioning`移动到`bootstrapping`到`active`。发布等到它处于活动状态。

## 2. 发布应用程序

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name my-app \
  --dir /workspace \
  --run-command "npm run start" \
  --port 3000 \
  --type docker_deploy \
  --wait \
  --wait-timeout 900s \
  --json
```

要发布新版本而不更改部署 URL，请重复使用相同的 `--name`。

## 3.证明

```bash
miosa deploy get <deployment-id> --json
```

仅当主机处于活动状态、应用程序目标正在运行、容器路由可访问并且对 `public_url` 的探测成功时，Docker 主机部署才处于活动状态。

## 选择你需要的

|你想要|从这里开始 |
|---|---|
|了解主机生命周期和证明 | [Docker 主机](/docs/docker-host/hosts) |
|从策划的应用程序形状开始 | [Docker 主机模板](/docs/docker-host/templates) |
|标准静态或动态发布 | [部署概述](/docs/docker-host/overview) |
|附加托管或自定义域 | [部署域](/docs/docker-host/domains) |
|比较标准目标和 Docker Host 目标 | [部署模型](/docs/docker-host/deployment-model) |

## 下一步

- [Docker Hosts](/docs/docker-host/hosts)：始终在线的工作区主机：生命周期状态、发布流程、证明和恢复。
- [Docker Host templates](/docs/docker-host/templates)：使用 Dockerfile、Compose、命令和端口默认值策划的应用程序形状。
- [发布](/docs/docker-host/publishing)：将沙箱冻结为不可变的部署版本。
- [部署域](/docs/docker-host/domains)：将托管域和自定义域附加到部署。
- [白标签设置](/docs/platform/white-label)：配置组织品牌和域行为。
- [外部归属](/docs/platform/attribution)：将 MIOSA 资源映射到您的用户、工作区和项目。
