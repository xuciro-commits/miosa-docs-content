---
slug: "quickstart/app-engine"
title: "Docker Host"
description: "在工作区 Docker Host 上运行容器化应用，然后确认应用已上线。"
section: "Get started"
order: 4
---

当工作区需要许多始终在线的应用、营销漏斗、API 或客户站点，而又不想为每次部署单独准备生产虚拟机时，请使用 Docker Host。
对外展示的产品名称是 Docker Host。
CLI 和 API 仍使用 `docker_deploy` 作为内部兼容值。

## 第 1 步——确保工作区 Docker Host 已就绪。

```bash
miosa api POST /docker-deploy/hosts/ensure -d '{"workspace_id": "<workspace-id>"}'
miosa api GET /docker-deploy/hosts
```

主机必须处于活动状态，Docker Host 发布才能完成。

## 第 2 步——将沙箱发布到 Docker Host。

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

响应包含部署及其 `public_url`。
该 URL 可能是 MIOSA 托管的租户 URL、工作区部署域名、租户部署域名，或精确的自定义域名。
始终展示返回的 URL。

## 第 3 步——确认应用已上线。

```bash
miosa deploy get <deployment-id> --json
miosa deploy requests <deployment-id>
```

`deploy get` 显示部署的状态、产品类型和 `public_url`。
`deploy requests` 显示近期处理的 HTTP 请求，便于确认真实流量已到达应用。

## 刚才完成了什么

| 步骤 | 结果 |
|------|--------------|
| `docker-deploy ensure` | 创建或返回工作区 Docker Host 主机 |
| `sandbox publish --docker-deploy` | 将沙箱产物作为容器发布到该工作区主机 |
| `docker-deploy doctor` | 验证容器和公共 URL 确实可用 |

> **注意：延伸阅读**
>
> 有关完整部署契约，请参阅 [Docker Host](/docs/docker-host/docker-deploy)；有关其余部署功能，请参阅[部署概览](/docs/docker-host/overview)。
