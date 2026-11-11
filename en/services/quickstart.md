---
slug: "services/quickstart"
title: "Services quick start"
description: "Take a repository to a live URL, add Postgres and Redis, add a worker, attach a domain, read a failed release and roll back."
section: "Services & projects"
order: 1005
---

This walkthrough takes a repository to a live URL, then adds a database, a worker and a domain.
Every step has a CLI form and an SDK form.

## Before you start

- A MIOSA account with an API key (`msk_...`) or a signed-in CLI.
- A project. `miosa project create shop --workspace-id <workspace>` creates one; `miosa project list` shows the others.
- A repository MIOSA can clone, or a container image.

Export the key once:

```bash
export MIOSA_API_KEY=msk_your_key
```

## 1. Create a web service

```bash
miosa service create shop API --type web \
  --repo https://github.com/acme/api \
  --branch main \
  --start-command "node server.js" \
  --replicas 2 \
  --env NODE_ENV=production
```

`--type web` stores the API's `server` kind, which can have a public address.
`--type private` stores the same kind but never gets one.
The command prints the service, its private address and, once enabled, its public address.

TypeScript:

```typescript
const service = await miosa.services.create(projectId, {
  name: "API",
  type: "web",
  source: { type: "git", ref: "https://github.com/acme/api", branch: "main" },
  settings: { start_command: "node server.js", replicas: 2 },
});
const address = await miosa.services.enablePublicAddress(service.id);
console.log(address.url); // https://api.acme.miosa.app
```

Python:

```python
service = miosa.services.create(
    project_id,
    "API",
    type="web",
    source={"type": "git", "ref": "https://github.com/acme/api", "branch": "main"},
    settings={"start_command": "node server.js", "replicas": 2},
)
address = miosa.services.enable_public_address(service["id"])
print(address["url"])
```

## 2. Deploy it

```bash
miosa service deploy shop API --message "first deploy" --wait
```

`--wait` polls the release until it is live or fails.
Without it you get the queued release and can check it with `miosa service releases shop API`.

```typescript
const release = await miosa.services.deploy(service.id, { message: "first deploy" });
let current = release;
while (!["active", "sleeping", "failed", "canceled", "stopped", "removed"].includes(current.state)) {
  await new Promise((resolve) => setTimeout(resolve, 3000));
  current = await miosa.services.release(service.id, release.id);
}
console.log(current.state);
```

The Python SDK ships `wait_for_release(service_id, release_id)`, which polls and raises on a failed release.

A release goes `queued -> building -> built -> starting -> health_check -> active`.
Any step can fail; the release carries a `reason.code` such as `build_failed`, `healthcheck_timeout` or `no_capacity`.

## 3. Add Postgres and Redis

Templates create the service, its variables and the references between them in one change set:

```bash
miosa service templates                       # the catalog
miosa service templates apply shop nextjs-postgres
miosa service templates apply shop redis
```

The Postgres template creates a database service and a web service whose `DATABASE_URL` is the reference `${{Postgres.DATABASE_URL}}`.
Applying a template stages one change set and deploys every service it creates; pass `--no-deploy` to create them without deploying.

To read the connection string later:

```bash
miosa service connection shop Postgres
```

That call needs `databases:read` and is audited, because the URL holds the password.

## 4. Add a worker

A worker has no port and no address.

```bash
miosa service create shop Worker --type worker \
  --repo https://github.com/acme/api \
  --start-command "node worker.js" \
  --replicas 1
miosa service deploy shop Worker
```

The worker can read the same database with the same reference:

```bash
miosa service vars set shop Worker DATABASE_URL='${{Postgres.DATABASE_URL}}'
```

## 5. Attach a domain

A web service or static site gets a public address on `miosa.app`:

```bash
miosa service domains shop API
miosa service domains enable shop API --label api
```

The result is `https://api.<your-org>.miosa.app`, served from the active release's healthy replicas.
To stop serving publicly without deleting the service, remove the address:

```bash
miosa service domains disable shop API
```

Private networking between services is not available yet, so other services call it at its public address; databases are reached through their connection variables, see [addresses and domains](/docs/services/domains).

## 6. Read a failed release

```bash
miosa service releases shop API
miosa service logs shop API
```

`logs` prints the newest release's state transitions and its failure reason.
Pass `--release <number>` to look at an older one.
The console's Logs tab shows each replica's runtime lines and the build log; see [Logs](/docs/services/logs).

## 7. Roll back

Rollback serves an earlier release's built image with its frozen source and settings.
Nothing is rebuilt, so it is fast.

```bash
miosa service rollback shop API 3 --yes
```

`miosa service promote shop API 3` is the same call: the Services API calls the operation rollback.

## 8. Tear down

```bash
miosa service stop shop API --yes      # stop now, keep the service
miosa service delete shop API --yes    # remove it and stop its replicas
```

Deleting a database service also deletes its data, so it takes `--confirm <exact name>` instead of `--yes`.
See [Deleting a database service](/docs/services/cli#deleting-a-database-service).

## Where to go next

- [Services and types](/docs/services/services): every source and setting.
- [Variables](/docs/services/variables): references and generated secrets.
- [Releases](/docs/services/releases): the state machine and what each reason means.
- [CLI](/docs/services/cli) and [SDKs](/docs/services/sdk): the full command and method list.
