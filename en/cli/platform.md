---
slug: "cli/platform"
title: "Platform resources"
description: "Deployments, databases, storage, volumes, workflows, schedules, functions, templates, hosts, forge, cloud, egress and the AI gateway from the miosa CLI."
section: "CLI, SDKs & MCP"
order: 11008
---

## Deployments

```bash
miosa deploy create --from-sandbox <sandbox> --wait
miosa deploy create --git <url> --branch main --wait
miosa deploy create --dir ./dist --name my-site
miosa deploy                          # deploy the current directory
miosa deploy list
miosa deploy get <deployment>
miosa deploy logs <deployment> -n 200
miosa deploy env <deployment>          # environment variable names
miosa deploy env-set <deployment> --set KEY=VALUE
miosa deploy redeploy <deployment>
miosa deploy rollback <deployment> --to <version-id>
miosa deploy versions <deployment>
miosa deploy releases <deployment>
miosa deploy observe <deployment>
miosa deploy builds <deployment>       # build logs
miosa deploy requests <deployment>
miosa deploy promote <deployment>
miosa deploy stop <deployment> --yes
miosa deploy rm <deployment> --yes
miosa deploy domains <deployment>
miosa deploy domain-add <deployment> <domain>
miosa deploy domain-verify <deployment> <domain>
miosa deploy domain-rm <deployment> <domain>
```

`miosa deploy rollback --to` returns to a named version; without it, the previous one.

`miosa deploy env` lists the deployment's environment variable names. `miosa deploy env-set <deployment> --set KEY=VALUE` sets them, and values are encrypted at rest. `--set` is repeatable.

```bash
miosa deploy env my-app
miosa deploy env-set my-app --set DATABASE_URL=postgres://... --set LOG_LEVEL=info
```

### Create a deployment

`miosa deploy create` (alias `miosa deploy publish`) publishes a new deployment and makes it live. Give it one source:

- `--from-sandbox <sandbox>` publishes a running sandbox's output. Add `--port <n>` and `--run-command` to publish a running server, `--entrypoint <file>` to pick the entry file of a static publish, or `--type docker_deploy` for Docker Host.
- `--git <url>` connects a repository and queues its first build. Add `--branch` and `--build-command`.
- `--dir <path>` packages a local directory and uploads it, for pre-built static output with no repository.

With `--from-sandbox`, `--dir` is the path the app lives at inside the sandbox (default `/workspace`). On its own, `--dir` is a local directory to upload.

```bash
miosa deploy create --from-sandbox my-box --wait
miosa deploy create --from-sandbox my-box --port 3000 --run-command "npm start"
miosa deploy create --git https://github.com/me/app --branch main --wait
miosa deploy create --dir ./dist --name my-site
```

Common flags: `--name`, `--env KEY=VALUE` (repeatable, encrypted at rest), `--domain`, and `--wait` to block until the deployment is live and print its public URL. `--json` prints the result as JSON.

`miosa deploy` with no subcommand deploys the current directory: the git repository when the directory is one, otherwise a directory upload. It asks before publishing unless you pass `--yes`.

## Databases

```bash
miosa db list
miosa db get <db>
miosa db connection <db>        # connection string
miosa db logs <db>
miosa db backups <db>
miosa db backup <db>
miosa db restart <db>
miosa db start <db>
miosa db stop <db> --yes
miosa db wake <db>
miosa db rm <db> --yes
```

## Storage

Object storage buckets.

```bash
miosa storage list
miosa storage create assets --visibility private --region <region>
miosa storage get assets
miosa storage objects assets --prefix uploads/ -n 50
miosa storage rm assets --yes
```

## Volumes

Persistent volumes shared between machines.

```bash
miosa volume list
miosa volume create data --size-gb 20
miosa volume get <volume>
miosa volume rm <volume> --yes
```

## Functions

Edge functions.

```bash
miosa function list
miosa function get <function>
miosa function invoke <function> -d '{"key":"value"}'
miosa function rm <function> --yes
```

## Workflows

Automations made of steps, agents and tools.

```bash
miosa workflow list
miosa workflow get <workflow>
miosa workflow run <workflow> --input '{"branch":"main"}'
miosa workflow runs <workflow>
miosa workflow run-get <workflow> <run>
miosa workflow run-cancel <workflow> <run>
miosa workflow pause <workflow> / resume <workflow>
miosa workflow rm <workflow> --yes
```

## Schedules

Scheduled jobs.

```bash
miosa schedule list
miosa schedule get <schedule>
miosa schedule executions <schedule>
miosa schedule run <schedule>
miosa schedule pause <schedule> / resume <schedule>
miosa schedule rm <schedule> --yes
```

## Templates

```bash
miosa template list
miosa template get <template>
miosa template build <template>
miosa template builds <template>
miosa template build-logs <template> <build>
miosa template custom <template>
miosa template promote <template>
miosa template rollback <template> --yes
miosa template warm-pool
miosa template rm <template> --yes
```

## Batch

Create many sandboxes in one request.

```bash
miosa batch new --count 50 --size small --name-prefix ci --wait
miosa batch status <batch>
miosa batch items <batch> --limit 20
miosa batch watch <batch> --timeout 15m
miosa batch cancel <batch>
```

`batch new` takes `--count`, `--size`, `--template`, `--name-prefix`, `--region`, `--ttl`, `--idle-timeout`, `--tag KEY=VALUE`, `--concurrency`, `--workspace`, `--wait` and `--wait-timeout`.

## Projects and workspaces

```bash
miosa project list
miosa project create <name> --description "..." --workspace-id <id>
miosa project get <project>

miosa workspace list
miosa workspace create <slug>
miosa workspace get <slug>
miosa workspace members <slug>
miosa workspace stats <slug>
miosa workspace usage <slug>
miosa workspace inventory <slug>
miosa workspace activity <slug>
miosa workspace use <slug>
miosa workspace delete <slug>
```

## Catalog and regions

```bash
miosa catalog --product sandbox --template nextjs
miosa catalog --state fast_ready
miosa catalog --output json | jq '.data[] | select(.state == "fast_ready")'
miosa regions
```

`miosa catalog` is the canonical catalog for products, templates, sizes and readiness.

## Forge (beta)

```bash
miosa forge repo list
miosa forge repo create platform --visibility private
miosa forge pr new platform --head fix-redirect --title "Fix redirect loop"
miosa forge pr review platform 12 --decision approved
miosa forge pr merge platform 12
miosa forge check report platform --commit <sha> --name unit-tests --conclusion passed --file evidence.json
miosa forge release new platform --tag v1.2.0
miosa forge policy set platform --branch main --approvals 2 --check unit-tests
```

Forge repository commands always return a `{ "data": ... }` success envelope. Repository mutations accept `--idempotency-key`, and delete requires interactive confirmation or `--yes`. Forge clone is not exposed until MIOSA provides a credential-helper flow that keeps credentials out of URLs, arguments, shell history, logs and persistent Git configuration.

## Egress

Outbound network policy for sandboxes.

```bash
miosa egress policies
miosa egress allowlist --policy <id>
miosa egress suggestions
miosa egress audit -n 50
```

## Your own hosts

```bash
miosa host register studio --platform macos    # prints the host key once
miosa host list --tag studio
miosa host get <host>
miosa host metrics <host>
miosa host runtime <host>
miosa host services <host>
miosa host jobs <host>
miosa host ssh-key
miosa host secret
miosa host tag <host>
miosa host app catalog
miosa host cluster ...
miosa host group ...
miosa host fs ...
miosa host rm <host> --yes
```

`miosa host register --platform` takes `macos`, `linux` or `windows`.

### Containers

Containers on a host.

```bash
miosa host container list <host>
miosa host container run <host> --image nginx:alpine --name web --set PORT=8080
miosa host container get <host> <container>
miosa host container logs <host> <container>
miosa host container stats <host> <container>
miosa host container start <host> <container>
miosa host container stop <host> <container>
miosa host container restart <host> <container>
miosa host container rm <host> <container> --force --yes
```

`miosa host container logs` and `miosa host container stats` stream until the connection ends or Ctrl-C is pressed. With `--json` each event is one JSON object per line. `miosa host container run` takes `--image`, `--name`, `--cmd` (repeatable), `--set KEY=VALUE` (repeatable) and `--restart`.

### Meshes

Private encrypted host-to-host networks over WireGuard.

```bash
miosa host mesh list
miosa host mesh new lab --host <host> --host <host>
miosa host mesh get lab
miosa host mesh add-host lab <host>
miosa host mesh remove-host lab <host> --yes
miosa host mesh events lab
miosa host mesh rm lab --yes
```

`miosa host mesh events` streams host joins, key rotation and state changes until Ctrl-C.

### GitHub Actions runners

Self-hosted GitHub Actions runners on a host, registered for one repository.

```bash
miosa host gha-runner list <host>
miosa host gha-runner new <host> --repo https://github.com/acme/app --token <token>
miosa host gha-runner new <host> --repo https://github.com/acme/app --token-secret GH_TOKEN --ephemeral
miosa host gha-runner get <host> <runner>
miosa host gha-runner events <host> <runner>
miosa host gha-runner refresh <host> <runner>
miosa host gha-runner rm <host> <runner> --yes
```

`miosa host gha-runner new` takes `--repo`, `--token` (sent to the host, not stored) or `--token-secret` (the name of a host secret holding the token), `--name`, `--label` (repeatable) and `--ephemeral` (deregister after one job). `miosa host gha-runner events` streams as the runner picks up and finishes jobs.

### Backups and snapshots

Scheduled backups of directories on a host. A configuration names a directory, a schedule and how many snapshots to keep. Snapshots can be forked onto a host.

```bash
miosa host backup list <host>
miosa host backup new <host> --name nightly --path /data --schedule "0 3 * * *" --keep 7
miosa host backup update <host> nightly --schedule "0 4 * * *"
miosa host backup trigger <host> nightly
miosa host backup pause <host> nightly / resume <host> nightly
miosa host backup snapshot list <host> nightly
miosa host backup snapshot get <snapshot>
miosa host backup snapshot fork <snapshot> --to-host <host> --to-path /restore
miosa host backup snapshot rm <snapshot> --yes
miosa host backup rm <host> nightly --yes
```

`miosa host backup new` takes `--path`, `--schedule` (a cron expression), `--timezone`, `--keep` (how many snapshots), `--keep-days`, `--exclude` (repeatable globs) and `--compression` (0-9). `miosa host backup snapshot list` takes `--limit` and `--before`.

### Tunnels

```bash
miosa host tunnel new <host> --port 3000 --access organization
```

Exposes a host port at an address. `--access` is `organization` (default) or `public`, and `--slug` sets the address name.

## Your own cloud

Bring your own cloud (BYOC). Accounts, regions and host pools.

```bash
miosa cloud account new prod --provider aws      # also --provider gcp
miosa cloud accounts
miosa cloud region ...
miosa cloud regions
miosa cloud pool provision <pool> --count 1
miosa cloud pools
miosa cloud nodes
miosa cloud capabilities
miosa cloud preflight run --account <id> --region <id>
miosa cloud preflights
```

> **Note**
>
> Bring your own cloud and host provisioning are a private preview and need host provisioning activated for you.

## AI gateway

Model routing policies, budget, limits and request traces.

```bash
miosa gateway policy new fast --primary-model claude-haiku --alias fast
miosa gateway policy new ... --strategy weighted --fallback-model <model>
miosa gateway budget show
miosa gateway budget ...
miosa gateway limit ...
miosa gateway health
miosa gateway provider-health
miosa gateway environments
miosa gateway requests --model <model> --status <status>
miosa gateway request <id>
miosa gateway settings ...
```

## Raw API

For capabilities broader than the high-level commands, use `miosa api` with the same authenticated config. See [Raw API access](/docs/cli/api).
