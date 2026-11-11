---
slug: "data/mysql"
title: "MySQL"
description: "Managed MySQL per project with durable storage and DATABASE_URL injection."
section: "Databases & storage"
order: 4002
---

> **Tip: Available**
>
> Managed MySQL is available for projects that need MySQL-compatible schemas,
> frameworks, or imports. MIOSA provisions the instance, stores credentials, and
> injects `DATABASE_URL` into attached sandboxes and production runtimes.

## When to use MySQL

Use managed MySQL when the application already expects MySQL semantics:

- Laravel, WordPress, or MySQL-first frameworks
- Existing customer schemas that should not be converted to Postgres
- SQL workloads that rely on MySQL-specific behavior
- SaaS templates that ship with MySQL migrations

Use [Postgres](/docs/data/postgres) when you need richer relational features,
extensions, or Postgres-native tooling.

## Runtime contract

Applications do not need to know where the database runs. Read the injected
connection string at boot:

```bash
DATABASE_URL=mysql://user:password@host:3306/database
```

MIOSA injects the same variable into:

- sandboxes for development and agent work
- runtime instances for production deployments
- background workers that belong to the project

Lifecycle, backup, restore, credentials, logs, and metrics use the common [Databases API](/docs/api-reference/databases).
Do not infer those values from a hostname or project slug.

## Provisioning pattern

```typescript
const project = await miosa.projects.create({
  name: "Customer Portal",
  data: { mysql: true },
});
```

For app builders and white-label platforms, create MySQL per customer project
when the generated app requires it. Keep schema migrations in the application
repo or sandbox workspace so deployments remain repeatable.

## See also

- [Data overview](/docs/data/overview): Choose the right data service for each workload.
- [Postgres](/docs/data/postgres): Managed relational database for Postgres-native applications.
- [Environments](/docs/docker-host/environments): Separate production, staging, and development credentials.
