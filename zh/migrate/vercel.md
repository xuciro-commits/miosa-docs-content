---
slug: "migrate/vercel"
title: "从Vercel搬家"
description: "使用 miosa import vercel 导入 Vercel 项目，检查计划以及结转的内容。"
section: "Optional"
order: 17029
---

`miosa import vercel` 读取 Vercel 项目的 `vercel.json` 并写入计划。
该计划命名 MIOSA 项目、`production` 环境、一项服务以及文件声明的环境变量名称。
检查计划，然后应用它。
在您申请之前，不会创建任何内容。

## 导入项目

```bash
npm i -g @miosa/cli
miosa import vercel --from-file vercel.json --name shop --repo https://github.com/acme/shop
miosa import vercel --from-file vercel.json --out plan.json
miosa import vercel --apply --plan plan.json
```

|旗帜|意义|
|---|---|
| `--from-file` |要读取的 `vercel.json` |
| `--name` | MIOSA 项目名称 |
| `--service` |项目内的服务名称（默认`web`）|
| `--repo` |服务源的 Git URL |
| `--region` | `us-east`、`us-west` 或 `us-mia`；覆盖文件 |
| `--into` |导入到现有的 MIOSA 项目 |
| `--apply` |按照计划制定 |
| `--resume` |继续中断的申请 |

## 继承什么

|Vercel |MIOSA |
|---|---|
|该应用程序 |从存储库构建的[服务](/docs/services/overview) |
|地区 |服务区域，当一个 MIOSA 的文件名具有 |
|环境变量名称 | [变量](/docs/services/variables) 没有值 |

值永远不会被复制。
设置服务上的每个值，然后部署。

## 什么不保留

该计划列出了每一项内容及其原因：重写、重定向、标头、构建、`functions` 块、`cleanUrls`、`trailingSlash`、`buildCommand`、`installCommand`、`outputDirectory` 和 `crons`。
重新创建您需要的内容作为[服务设置](/docs/services/services)。

## 下一步去哪里

- [服务快速启动](/docs/services/quickstart)
- [从Railway搬家](/docs/migrate/railway)
