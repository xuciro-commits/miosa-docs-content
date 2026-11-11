---
slug: "services/cli"
title: "Services CLI"
description: "Every miosa service command and flag, from create and deploy to variables, domains, releases and rollback."
section: "Services & projects"
order: 1018
---

`miosa service` operates Services.
`miosa services` and `miosa svc` are aliases.
A named background process inside a sandbox is `miosa daemon`, a different product.

`<project>` is a project name, slug or id.
`<service>` is a service name, slug or id; the CLI resolves the name to the id the API needs.
`<release>` is a release number or id.
Every command accepts `--json`.

## Commands

| Command | What it does |
|---|---|
| `service create <project> <name>` | Create a web service, worker, private service, static site, cron job or database service |
| `service list <project>` | List the project's services |
| `service get <project> <service>` | Show one service |
| `service update <project> <service>` | Change the name, start command, replicas or region |
| `service deploy <project> <service>` | Build and release |
| `service logs <project> <service>` | Show a release's transitions and failure reason |
| `service scale <project> <service>` | Change replicas, region or limits |
| `service start <project> <service>` | Start a stopped service |
| `service stop <project> <service>` | Stop every active release now |
| `service restart <project> <service>` | Queue a release that restarts the service |
| `service delete <project> <service>` | Delete a service and stop its replicas; a database service also deletes its database |
| `service releases <project> <service>` | List releases, newest first |
| `service release <project> <service> <release>` | Show one release |
| `service rollback <project> <service> <release>` | Serve an earlier release again |
| `service promote <project> <service> <release>` | The same as rollback |
| `service cancel <project> <service> <release>` | Cancel a queued or building release |
| `service domains <project> <service>` | Show the private and public addresses |
| `service domains enable <project> <service>` | Give the service a public address |
| `service domains disable <project> <service>` | Remove the public address |
| `service vars list <project> <service>` | List variables, masked |
| `service vars set <project> <service> KEY=VALUE...` | Set variables |
| `service vars unset <project> <service> NAME[,NAME]` | Delete variables |
| `service vars effective <project> <service>` | Show what a deploy would receive |
| `service vars reveal <project> <service> NAME` | Print one value |
| `service runs <project> <service>` | A cron service's recent runs |
| `service connection <project> <service>` | A database service's URLs |
| `service settings <project>` | What each setting does today |
| `service templates` | List the template catalog |
| `service templates apply <project> <slug>` | Apply a template |
| `project vars list <project>` | Shared variables, masked |
| `project vars set <project> KEY=VALUE...` | Set shared variables |
| `project vars unset <project> NAME` | Delete a shared variable |
| `project target list <project>` | List Deploy targets |
| `project target create <project> <name>` | Create a Deploy target |
| `project target duplicate <project> <target> --name <new>` | Copy a Deploy target's services and variables |
| `project target rm <project> <target>` | Delete a Deploy target |

## `service create` flags

| Flag | Meaning |
|---|---|
| `--type` | `web`, `worker`, `private`, `static`, `schedule` or `database` (default `web`) |
| `--repo`, `--branch`, `--root-dir` | A git source |
| `--image`, `--port` | A container image source |
| `--dockerfile` | Dockerfile path for a git source (needs Services build cells) |
| `--start-command` | Command the service starts with |
| `--replicas` | 1 to 50 |
| `--region` | `us-west`, `us-east` or `us-mia` |
| `--schedule` | Five-field schedule expression, required for `--type schedule` |
| `--env KEY=VALUE` | A variable; repeatable, and a value may be a `${{...}}` reference |
| `--no-public` | Do not give a web or static service a public address |
| `--slug` | Host label (default: from the name) |
| `--deploy-target` | Deploy target name or id (default: the project's default) |

A web or static service gets its public address enabled right after creation unless `--no-public` is passed.
If the address cannot be enabled the service is still created and the CLI prints a warning.

## `service deploy` flags

| Flag | Meaning |
|---|---|
| `--dir` | Upload this local directory as the source |
| `--repo`, `--branch`, `--root-dir` | Deploy a git source |
| `--image`, `--port` | Deploy a container image |
| `--message` | Release message |
| `--wait`, `--timeout` | Follow the release until it is live or fails |
| `--deploy-target` | Deploy target name or id |

With no source flag the service's own source is rebuilt.

## `service get`

`service get` prints the service, including `Pending changes` when a variable write made the running release stale.
The value is `rebuild` (a build-visible variable changed) or `redeploy` (any other variable), and `miosa service deploy` is the action that clears it.

## `service logs`

`logs` prints the newest release's state transitions and its failure reason.
Pass `--release <number|id>` for an older one.
`--follow` is refused until the Services log stream ships; the release events carry the reason a release failed.

## Deploy targets

Every project has a default Deploy target, and the first read creates `production`.
Services, variables and releases live inside one Deploy target; `--deploy-target` picks it and defaults to the project's default.

```bash
miosa project target list shop
miosa project target duplicate shop production --name staging
miosa service list shop --deploy-target staging
```

A duplicate copies services and variables; secrets are generated again and no data or managed database is copied.

## Confirmation

`stop`, `delete`, `rollback` and `promote` ask for confirmation on a terminal and refuse in a script unless `--yes` is passed.

### Deleting a database service

Deleting a database service also deletes its database and every row in it.
It cannot be undone, so it needs the service's exact name, not `--yes`:

- On a terminal, the CLI asks you to type the name.
- In a script, pass `--confirm <exact name>`.
- `--yes` alone is not enough.
- A run with no terminal and no `--confirm` exits with code 2 and sends nothing.
- A `--confirm` that does not match the name exactly also exits with code 2.

```bash
miosa service delete shop Postgres --confirm Postgres
```

`--confirm` needs CLI 2.13.3 or later.

## Automation

Every command is non-interactive with `--yes` (deleting a database service needs `--confirm <name>` instead), prints JSON with `--json`, and exits non-zero on an API error.
The API's error code is preserved; for example a stale settings write exits non-zero with `STALE_SERVICE`.
