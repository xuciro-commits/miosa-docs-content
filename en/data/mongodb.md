---
slug: "data/mongodb"
title: "MongoDB"
description: "Managed MongoDB per project for document data."
section: "Databases & storage"
order: 4005
---

> **Tip: Available**
>
> Managed MongoDB is a database engine like Postgres, MySQL and Redis.
> It runs the official MongoDB 7 server in its own microVM with a persistent data volume.

## What you get

A managed MongoDB 7 instance with an auto-generated root user and password, injected as `MONGODB_URI` (also `MONGO_URL`):

```bash
MONGODB_URI=mongodb://user:pass@host:port/miosa_db?authSource=admin
```

Standard MongoDB drivers work without modification:

```typescript
import { MongoClient } from "mongodb"
const client = new MongoClient(process.env.MONGODB_URI!)
await client.connect()

const users = client.db("miosa_db").collection("users")
await users.insertOne({ name: "Ada", createdAt: new Date() })
```

The user has the root role, so you can create any number of databases on the instance.
`authSource=admin` is part of the URL and must stay.

## Provision

```bash
miosa db create --name app-mongo --engine mongodb
```

```typescript
await miosa.databases.create({ name: "app-mongo", engine: "mongodb", size: "small" })
```

You can also choose MongoDB in the Add menu of a project, or set `managed: mongodb` on a database service in a change set.
Only major version 7 is available.

## Connection variables

| Variable | Value |
| --- | --- |
| `MONGODB_URI` | Full connection string |
| `MONGO_URL` | Same as `MONGODB_URI` |
| `MONGO_HOST`, `MONGO_PORT` | Host and port |
| `MONGO_USER`, `MONGO_PASSWORD` | Root user and password |
| `MONGO_DATABASE` | Default database name |

## Backups and restore

Backups are logical (`mongodump`, gzip archive) and cover every database on the instance except `admin` and `config`.
A restore replaces the collections the backup contains and leaves the others alone.
It never touches the `admin` database, so your issued credentials keep working.
A MongoDB backup restores only into a MongoDB database.

## Limits

- There is no table browser or query console for MongoDB. Use `mongosh` or a driver.
- There is no automatic pause when idle, as with MySQL and Redis.
- Memory, connections and storage follow the selected plan size.

## See also

- [Overview](/docs/data/overview)
- [Redis](/docs/data/redis)
- [Postgres](/docs/data/postgres)
