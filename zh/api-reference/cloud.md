---
slug: "api-reference/cloud"
title: "云和 BYOC API"
description: "组织范围内的 AWS 和 GCP 账户、区域、池、节点、预检和主机配置。"
section: "API reference"
order: 12032
---

基本路径：`/api/v1/cloud`

所有资源都属于活动组织，返回为 `organization_id` 和兼容性字段 `tenant_id`。
云读取需要 `cloud:read`，突变需要 `cloud:write`。
组织成员可以读取库存。
组织所有者和管理员可以更改它。
工作区范围的凭据无法改变组织云容量。

AWS 和 GCP BYOC 是封闭的私有预览。
提供商连接和预检不会使工作负载放置成为可选择的。
每个确切的组织、区域、提供商和工作负载组合都需要来自真实接受的主机的当前验证收据。

## 账户

- `GET /cloud/accounts` 列出组织云帐户。
- `POST /cloud/accounts` 创建 AWS 或 GCP 客户 BYOC 信任边界。
- `POST /cloud/accounts/{id}/aws-role` 附加 AWS AssumeRole ARN 和生成的外部 ID。
- `POST /cloud/accounts/{id}/gcp-workload-identity` 附加 GCP 项目、客户执行器服务帐户和单独的最低权限工作人员服务帐户，而不存储服务帐户密钥。

## 地区

- `GET /cloud/regions?cloud_account_id={id}` 列出配置的区域。
- `POST /cloud/regions` 记录提供程序区域、区域、网络、主机映像和确切的不可变工件配置。
- `PATCH /cloud/regions/{id}` 更新该配置并清除过时的预检准备情况。

每个可启动区域必须提供 HTTPS 清单 URI、小写 SHA-256 摘要、绑定到 URI 的版本以及肯定的工件生成。

## 池

- `GET /cloud/pools?cloud_region_id={id}` 列出所需的主机池。
- `POST /cloud/pools` 创建池配置。
- `PATCH /cloud/pools/{id}` 更新组织容量和放置护栏。
- `POST /cloud/pools/{pool_id}/provision` 显式启动 1 到 50 个主机。

```json
{
  "count": 1
}
```

配置会为每个请求的主机返回一个结果。
成功启动在 `provider_pending` 中开始，并且在主机接受通过之前无法安排。

## 节点

- `GET /cloud/nodes?cloud_pool_id={id}` 列出池的提供者和接受状态。

重要的节点状态包括 `provider_pending`、`acceptance_running`、`active`、`draining`、`terminated` 和 `failed`。
只有具有持久组织单元和接受的主机会话的 `active` 节点才有资格放置。

## 预检

- `GET /cloud/preflights` 列出帐户和区域预检证据。
- `POST /cloud/preflights` 运行服务器端帐户或区域验证并记录不可变的证据。

预检和主机验收是不同的门。
Preflight 在启动前验证 AWS 或 GCP 身份、真实提供商 API 访问、区域和机器可用性、配置的网络和图像引用以及不可变的工件合同。
主机验收会验证实际启动的机器并在放置之前提升 MIOSA 运行时。
沙箱、计算机和部署放置是单独认证的，因为一个工作运行时不能证明另一个工作负载合同。

## SDK

```typescript
const result = await miosa.cloud.provisionPool(poolId, 1);
const nodes = await miosa.cloud.listNodes({ cloudPoolId: poolId });
```

```python
result = client.cloud.provision_pool(pool_id, count=1)
nodes = client.cloud.list_nodes(cloud_pool_id=pool_id)
```

沙箱创建不需要池 ID。
活动的 BYOC 组织会自动且排他地放入其配置的调度单元中。
