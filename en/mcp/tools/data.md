---
slug: "mcp/tools/data"
title: "Data tools"
description: "Hosted MIOSA MCP data tools: managed databases and object storage."
section: "CLI, SDKs & MCP"
order: 11028
---

Every tool in this family, exactly as the hosted server registers it. Arguments
are the tool's own parameter names; optional ones are still listed, because the
description says what they change.

| Tool | Arguments | Description |
|---|---|---|
| `database_create` | `name`, `engine`, `version`, `size`, `region` | Create a new managed database. |
| `database_credentials` | `database_id` | Get the connection string and credentials for a database. |
| `database_delete` | `database_id` | Delete a managed database permanently. |
| `database_get` | `database_id` | Get details of a specific database. |
| `database_list` | - | List all managed databases in the tenant. |
| `database_logs` | `database_id`, `lines`, `since` | Get logs for a managed database. |
| `storage_bucket_create` | `name`, `region`, `public` | Create a new storage bucket. |
| `storage_bucket_delete` | `bucket_id` | Delete a storage bucket. |
| `storage_bucket_list` | - | List all storage buckets in the tenant. |
| `storage_object_delete` | `bucket_id`, `key` | Delete an object from a storage bucket. |
| `storage_object_download` | `bucket_id`, `key` | Download an object from a storage bucket. |
| `storage_object_list` | `bucket_id`, `prefix` | List objects in a storage bucket, optionally filtered by prefix. |
| `storage_object_upload` | `bucket_id`, `key`, `content`, `content_encoding` | Upload an object to a storage bucket. |
| `storage_presign` | `bucket_id`, `key`, `expires_in`, `method` | Get a presigned URL for temporary access to a storage object. |
| `volume_attach` | `computer_id`, `volume_id`, `mount_path` | Attach a volume to a computer at a given mount path. |
| `volume_create` | `name`, `size_gb`, `region` | Create a new persistent volume. |
| `volume_delete` | `volume_id` | Delete a volume permanently. |
| `volume_detach` | `computer_id`, `attachment_id` | Detach a volume attachment from a computer. |
| `volume_get` | `volume_id` | Get details of a specific volume. |
| `volume_list` | - | List all volumes in the tenant. |
