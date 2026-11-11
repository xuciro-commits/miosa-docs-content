---
slug: "mcp/tools/data"
title: "数据工具"
description: "托管 MIOSA MCP 数据工具：托管数据库和对象存储。"
section: "CLI, SDKs & MCP"
order: 11028
---

该系列中的每个工具都与托管服务器注册的完全相同。论点
是工具自己的参数名称；仍然列出了可选的，因为
描述说明了它们改变了什么。

|工具|论点|描述 |
|---|---|---|
| `database_create` | `name`、`engine`、`version`、`size`、`region` |创建新的托管数据库。 |
| `database_credentials` | `database_id` |获取数据库的连接字符串和凭据。 |
| `database_delete` | `database_id` |永久删除托管数据库。 |
| `database_get` | `database_id` |获取特定数据库的详细信息。 |
| `database_list` | - |列出租户中的所有托管数据库。 |
| `database_logs` | `database_id`、`lines`、`since` |获取托管数据库的日志。 |
| `storage_bucket_create` | `name`、`region`、`public` |创建一个新的存储桶。 |
| `storage_bucket_delete` | `bucket_id` |删除存储桶。 |
| `storage_bucket_list` | - |列出租户中的所有存储桶。 |
| `storage_object_delete` | `bucket_id`、`key` |从存储桶中删除对象。 |
| `storage_object_download` | `bucket_id`、`key` |从存储桶下载对象。 |
| `storage_object_list` | `bucket_id`, `prefix` |列出存储桶中的对象，可以选择按前缀过滤。 |
| `storage_object_upload` | `bucket_id`、`key`、`content`、`content_encoding` |将对象上传到存储桶。 |
| `storage_presign` | `bucket_id`、`key`、`expires_in`、`method` |获取用于临时访问存储对象的预签名 URL。 |
| `volume_attach` | `computer_id`、`volume_id`、`mount_path` |将卷附加到给定安装路径的计算机。 |
| `volume_create` | `name`、`size_gb`、`region` |创建一个新的持久卷。 |
| `volume_delete` | `volume_id` |永久删除卷。 |
| `volume_detach` | `computer_id`、`attachment_id` |从计算机上分离卷附件。 |
| `volume_get` | `volume_id` |获取特定卷的详细信息。 |
| `volume_list` | - |列出租户中的所有卷。 |
