---
slug: "data/public-endpoint"
title: "公共数据库端点"
description: "使用允许列表为一个托管数据库打开仅 TLS TCP 端点，以便笔记本电脑或外部服务可以连接。"
section: "Networking & domains"
order: 5006
---

托管数据库可通过其私有 URL 从平台内部访问。转
在公共网络上，当外部设备需要连接时：笔记本电脑、
迁移工具，另一个云中的分析工作。

```bash
miosa db public app-db                       # off or on, and the address
miosa db public app-db on                    # any source
miosa db public app-db on --allow 203.0.113.0/24 --allow 198.51.100.7
miosa db public app-db off                  # close it and free the port
```

相同的调用是 `GET`、`POST` 和 `DELETE /api/v1/databases/:id/public-networking`。

## 你得到什么

- 一台主机和一个从 **30000-39999** 的端口，每个数据库一个端口。主办方是
  Edge 的公共地址，因此当数据库移动主机时它不会改变。
- `public_url` 在凭证响应中，并且
  `DATABASE_PUBLIC_URL` 在数据库服务的变量中，所以
  `${{db.DATABASE_PUBLIC_URL}}` 解析任何需要它的东西。

## 安全

- **仅限 TLS。** Postgres 需要 `sslmode=require`； MySQL 需求
  `--ssl-mode=REQUIRED` 并拒绝明文登录，错误为 3159。
  证书是自签名的，因此 `require` 无需验证名称即可加密。
- **允许列表在任何 TLS 握手之前关闭不允许的源。** `--allow`
  替换存储的列表，因此 `on` 不带 `--allow` 表示任何源。参赛作品
  是 IP 或 CIDR，最多 32 个。
- 限制：每个数据库 100 个连接，每 10 秒 20 个新连接
  源 IP，一次最多 32 次 TLS 握手，以及 2,000 个公共连接
  跨您的数据库。

## 引擎

Postgres、MySQL、Redis 和 MongoDB。其他引擎回答`ENGINE_NOT_SUPPORTED`。

## 关闭它

`off`，或删除数据库，会在大约十五秒内关闭端口。
两者都在审核日志中记录为 `database.public_networking_enable` 和
`database.public_networking_disable`。

> **注意：首选私有 URL**
>
> 平台内的任何内容都应使用私有 URL。公共端点
> 为无法到达专用网络的客户端而存在，不作为默认方式
> 连接。
