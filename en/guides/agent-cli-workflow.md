---
slug: "guides/agent-cli-workflow"
title: "Agent CLI workflow"
description: "The full CLI release workflow for agents: discovery, scope, sandbox development, preview, publish, promote, recovery, and rollback."
section: "Agents"
order: 6002
---

Use this page when an AI agent, coding assistant, CI worker, or operator needs the full build-and-release path through MIOSA.
The CLI is the authoritative happy path. Prefer its `--json` output over reconstructing lower-level API calls.

## Start with discovery

```bash
miosa catalog --json
miosa deploy list --json
miosa deploy get <deployment> --json
```

`catalog` reports the canonical compute products, templates, sizes, and readiness the platform offers right now.

`deploy list` and `deploy get` return the deployments you already own, with their status, active release and public URL.

Do not guess a resource id from an old transcript when these commands can return the current inventory.

## Resolve scope before mutation

Every mutation must resolve one unambiguous organization, workspace, application, and environment.

Do not create a replacement sandbox, deployment, database, or Docker Host host just because an identifier was not resolved.

Inspect the intended resource first.

```bash
miosa deploy get <deployment> --json
miosa deploy releases <deployment> --json
```

`deploy get` shows the deployment's identity, environment, active release and route.

`deploy releases` lists the immutable releases already published to it.

Do not commit credentials or secrets to a repository.

Secrets belong in encrypted MIOSA environment storage, not in the manifest or shell history.

## Develop in a persistent sandbox

Use the sandbox development flow for mutable work.

```bash
miosa create my-dev-box --template nextjs --wait
miosa console my-dev-box --cwd /workspace
miosa doctor
```

`create` provisions the sandbox and, with `--wait`, blocks until it is ready for commands.

`console` opens an interactive shell on the sandbox and resumes the same machine on a later run.

`doctor` checks configuration, connectivity and authentication, and reports what it finds.

Dependency installation must use a lockfile-strict command such as `npm ci`.

## Preview before production

```bash
miosa preview create my-dev-box 3000 --name candidate
miosa preview status my-dev-box <preview-id>
```

`preview create` publishes a port on the sandbox at a public URL and prints the URL.

`preview status` reports whether it is serving.

Preview never promotes production.

Read the returned `url` and `status` from the JSON response.

Use the returned URL.

Do not construct a sandbox, deployment, or custom-domain hostname.

## Publish the candidate

```bash
miosa deploy create my-app --from-sandbox my-dev-box --port 3000 --wait --json
```

`deploy create` publishes a sandbox, a git repository or a local directory as a new deployment and, with `--wait`, blocks until it is live and prints its URL.

Each publish produces an immutable release that can be promoted or rolled back to.

A candidate is not production merely because its build succeeded.

## Promote the exact release

```bash
miosa deploy releases <deployment>
miosa deploy promote <deployment> <release>
```

Promotion targets one exact immutable release.

It must not select "latest" by timestamp or rely on a mutable version label.

The production URL stays stable while the active immutable release changes.

If the candidate fails to build, the currently active release remains live.

## Recover an interrupted publish

An HTTP timeout does not prove the backend operation failed.

Re-read the deployment before retrying, and only republish when the previous attempt is proven terminal.

```bash
miosa deploy get <deployment> --json
miosa deploy builds <deployment>
```

Do not start a duplicate publish or promotion while an earlier operation may still be active.

## Roll back deliberately

```bash
miosa deploy rollback <deployment> --to <release-id> --yes --json
```

Rollback also targets one exact immutable release.

Use a known-good release and verify the same route, artifact, data, and configuration contract after activation.

Database migrations must remain backward compatible when an older application release may run against newer data.

## See also

- [Agent safety and status](/docs/guides/agent-safety): Interpret composite health signals and protect durable resources.
- [Agent operating guide](/docs/guides/agent-operating-guide): The three-step entry path for agents.
- [Publishing](/docs/docker-host/publishing): Candidate creation, verification, exact promotion, and receipts.
- [Rollback](/docs/docker-host/rollback): Re-point a deployment at an older version with no rebuild.
