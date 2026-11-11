---
slug: "services/domains"
title: "服务地址与域名"
description: "私有网络尚不可用时服务如何访问数据库、Web 服务或静态站点可拥有的公共 miosa.app 地址，以及 TLS 如何处理。"
section: "Networking & domains"
order: 5000
---

服务通过其连接变量访问托管数据库。
Web 服务或静态站点还可以拥有公共地址。

## 私有网络

服务之间的私有网络尚不可用。
一个服务无法通过私有名称访问另一个服务，`private_network` 的值为 `{"enabled": false}`。
旧的 `private_domain` 字段（`<service-slug>.miosa.internal`）和 `MIOSA_PRIVATE_DOMAIN` 变量不会解析；不要基于它们进行构建。

托管数据库可从任何主机自动连接。
在服务的变量中引用数据库的连接：

```text
DATABASE_URL=${{Postgres.DATABASE_URL}}
REDIS_URL=${{Redis.REDIS_URL}}
```

要调用项目中的另一个服务，请为其分配公共地址并通过该地址调用。
控制台中服务的 Networking 标签页也会显示同样的信息。

## 公共地址

```bash
miosa service domains shop API
miosa service domains enable shop API --label api
miosa service domains disable shop API
```

`enable` 会为服务提供：

```text
https://<label>.<your-org>.miosa.app
```

标签默认使用服务的 slug，且只能包含小写字母 a-z、数字 0-9 和 `-`。
标签在你的组织内必须唯一。
TLS 由平台签发和续期；无需安装证书。

`disable` 会移除公共地址，保留私有地址。
服务会继续运行，只是不再能从互联网访问。

## 公共地址由什么提供服务

公共地址会路由到服务当前活动发布的健康副本。
状态不是 `active` 的发布不会接收流量。

如果服务启用了空闲休眠，对休眠服务的请求会将其唤醒并等待其就绪。
没有公共地址的服务没有请求信号，因此永远不会休眠。

## 在代码中读取地址

TypeScript：

```typescript
const service = await miosa.services.get(serviceId);
console.log(service.private_network);       // { enabled: false } while private networking is not available
console.log(service.public_address.url);    // null until enabled
```

Python：

```python
service = miosa.services.get(service_id)
print(service["private_network"])
print(service["public_address"]["url"])
```

## 自定义域名

公共地址位于 `miosa.app`。
在你自己的 DNS 区域中配置自定义域名属于另一项产品：注册域名、添加其显示的 CNAME，并让 MIOSA 验证。
该流程位于域名产品下，而不是某个服务下，因此一个域名可以比服务存在得更久。

## 入站流量关闭时

如果平台未开启公共地址，`enable` 会返回 `INGRESS_DISABLED`（404）。
服务仍会运行，但没有可访问的地址，`miosa service settings shop` 会将 `public_address` 报告为 `unsupported`。
