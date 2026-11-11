---
slug: "api-reference/network-policy"
title: "网络策略API"
description: "用于在 MIOSA 计算机上配置每台计算机网络访问控制的 API 参考。"
section: "API reference"
order: 12054
---

网络策略控制计算机可以访问的外部主机和端口。使用它们对不受信任的工作负载进行沙箱处理、强制执行出口限制或防止 AI 智能体进行意外的出站连接。

基本路径：`/api/v1/computers/{id}/network-policy`

> **注意**
>
> 网络策略适用于 microVM 级别。更改策略会在几秒钟内生效，无需重新启动计算机。

---

## 快速入门
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Restrict to only allow HTTPS to GitHub and PyPI
await client.networkPolicy.update(computerId, {
  mode: 'allowlist',
  rules: [
    { host: 'github.com',  port: 443, protocol: 'tcp' },
    { host: 'pypi.org',    port: 443, protocol: 'tcp' },
    { host: 'files.pythonhosted.org', port: 443, protocol: 'tcp' },
  ],
});
```
```bash
curl -X PUT https://api.miosa.ai/api/v1/computers/{id}/network-policy \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "allowlist",
    "rules": [
      { "host": "github.com", "port": 443, "protocol": "tcp" }
    ]
  }'
```

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/computers/{id}/network-policy` |获取当前网络策略 |
| `PUT` | `/computers/{id}/network-policy` |设置网络策略（替换现有）|
| `DELETE` | `/computers/{id}/network-policy` |重置为默认值（无限制）|

---

## 获取网络策略

**`GET /api/v1/computers/{id}/network-policy`**

### 响应 - `200 OK`
```json
{
  "data": {
    "computer_id": "...",
    "mode": "unrestricted",
    "rules": [],
    "updated_at": "2026-04-11T00:00:00Z"
  }
}
```

### 政策模式

|模式|描述 |
|------|-------------|
| `unrestricted` |默认。允许所有出站流量 |
| `allowlist` |仅允许列出的主机/端口 |
| `denylist` |除列出的规则外，允许所有流量 |
| `isolated` |所有出站被阻止（DNS 和 MIOSA 内部除外）|
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/network-policy \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 设置网络策略

**`PUT /api/v1/computers/{id}/network-policy`**

取代整个政策。不支持部分更新 - 在每个 PUT 上发送所有规则。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `mode` |字符串|是的 | `"unrestricted"`、`"allowlist"`、`"denylist"` 或 `"isolated"` |
| `rules` |数组|没有 |当模式为 `"allowlist"` 或 `"denylist"` | 时需要

#### 规则对象

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `host` |字符串|是的 |主机名或 CIDR 块（例如 `10.0.0.0/8`）|
| `port` |整数 |没有 |端口号。省略匹配所有端口 |
| `protocol` |字符串|没有 | `"tcp"`、`"udp"` 或 `"any"`（默认）|

### 响应 - `200 OK`

更新了策略对象。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 400 | `rules required for allowlist/denylist mode` |过滤模式的空规则 |
| 404 | `computer not found`|计算机不存在 |
```bash
# Full isolation except for GitHub
curl -X PUT https://api.miosa.ai/api/v1/computers/{id}/network-policy \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "allowlist",
    "rules": [
      { "host": "github.com", "port": 443, "protocol": "tcp" },
      { "host": "objects.githubusercontent.com", "port": 443, "protocol": "tcp" }
    ]
  }'
```

---

## 重置为默认值

**`DELETE /api/v1/computers/{id}/network-policy`**

删除任何自定义策略。计算机恢复为不受限制的出站访问。

### 响应 - `200 OK`
```json
{
  "data": { "computer_id": "...", "mode": "unrestricted" }
}
```
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/network-policy \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 常见食谱

### 沙箱是一项不受信任的人工智能任务
```python
# Lock down before running untrusted code
client.network_policy.update(computer_id, mode="isolated")

# Run the task
result = client.exec(computer_id, command="python3 /workspace/untrusted.py")

# Restore after
client.network_policy.delete(computer_id)
```

### 只允许构建环境使用包注册表
```typescript
const packageHosts = [
  { host: 'registry.npmjs.org', port: 443, protocol: 'tcp' as const },
  { host: 'pypi.org',           port: 443, protocol: 'tcp' as const },
  { host: 'files.pythonhosted.org', port: 443, protocol: 'tcp' as const },
  { host: 'pkg.go.dev',         port: 443, protocol: 'tcp' as const },
  { host: 'proxy.golang.org',   port: 443, protocol: 'tcp' as const },
];

await client.networkPolicy.update(computerId, {
  mode: 'allowlist',
  rules: packageHosts,
});
```

### 阻止已知的恶意 IP 范围
```typescript
await client.networkPolicy.update(computerId, {
  mode: 'denylist',
  rules: [
    { host: '185.220.0.0/16' },
  ],
});
```
