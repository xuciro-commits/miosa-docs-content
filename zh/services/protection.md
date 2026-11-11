---
slug: "services/protection"
title: "防护与 WAF"
description: "为服务加上密码保护、生成分享链接，并设置智能体执行的 Web 应用防火墙规则。"
section: "Networking & domains"
order: 5005
---

防护和 WAF 在服务前方的边缘层执行，因此会在请求到达您的进程之前生效。

## 密码与分享链接

```bash
miosa protection shop API                            # is it protected?
miosa protection shop API --password 'open-sesame'
miosa protection shop API --share-link --expires 3600
miosa protection shop API --revoke <link-id>
miosa protection shop API --off
```

密码不会再次显示。分享链接是让某个人绕过密码的方式：
`msl_...` 令牌只显示一次，并在 `--expires` 秒后过期（默认七天，最长九十天）。访问者通过 `?miosa_share=<token>` 或 `x-miosa-share-token` 请求头使用它。

防护也可以在 Deploy 目标或环境类型上设置，各行的解析顺序为服务优先于环境，环境优先于类型。因此，无需逐个修改预览环境中的服务，就能保护整个预览环境。

## WAF

```bash
miosa waf shop API
miosa waf shop API --mode enforce --deny 203.0.113.0/24 --rate 120
miosa waf shop API --allow 198.51.100.0/24
miosa waf shop API --attack-mode on
miosa waf shop API --off
```

- `--mode` 可取 `off`、`observe` 或 `enforce`。
- `--allow`/`--deny` 接受 IP 或 CIDR（在请求中对应 `ip_allow`/`ip_deny`）。
- `--rate` 是每个来源每分钟的请求数；`0` 表示清除限制。
- `--attack-mode` 会向浏览器返回 JavaScript 挑战，其他所有请求返回 `403`。

**国家/地区规则只读取边缘设置的受信任 `x-miosa-country` 请求头。** 我们不自行进行 GeoIP 查询：如果请求到达时没有该请求头，国家/地区规则会被跳过，WAF 文档会通过 `country_rules_active` 说明这些规则是否能够生效。

> **注意：先观察再启用**
>
> 先以 `--mode observe` 运行一天。WAF 会报告它原本会拦截什么，这样您就能在错误的 CIDR 变成满屏 403 之前发现问题。
