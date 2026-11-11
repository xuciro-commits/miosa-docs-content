---
slug: "compare/boat"
title: "MIOSA vs boat"
description: "A factual comparison of MIOSA and boat (boat.dev), cloud sandboxes for AI agents, using boat's public documentation as of 2026-10-23."
section: "Optional"
order: 17026
---

Compare

boat and MIOSA both give AI agents their own Linux machines. This page compares what each documents, including where boat is ahead.

Every boat statement below is from boat's own public site and documentation ([boat.dev](https://boat.dev/), [docs.boat.dev](https://docs.boat.dev/)), read on 2026-10-23.
MIOSA statements link to the page that documents them.

## What boat is

boat is a cloud sandbox product for AI agents: persistent Ubuntu VMs with SSH, Docker inside, a desktop, and disk-level forks from snapshots ([boat.dev](https://boat.dev/)).
It was called Box until 2026-09-15; the old `box` CLI and endpoints still work until 2026-10-31 ([migrating from Box](https://docs.boat.dev/migrating-from-box.md)).
It is a sandbox product, not a deploy platform: there is no deploy, custom domain, rollback, or managed database command ([always-on services](https://docs.boat.dev/long-running-tasks/always-on-services.md), [host CLI](https://docs.boat.dev/hosting/host-cli.md)).

boat does not appear in the ComputeSDK runs our [benchmark page](/docs/benchmarks) reports, so this page has no measured comparison.

## Capabilities side by side

| Capability | MIOSA | boat | boat source |
|---|---|---|---|
| Persistent Linux machines | [Yes, sandboxes and computers](/docs/sandboxes-or-computers) | Yes, Ubuntu VMs with SSH | [boat.dev](https://boat.dev/) |
| Docker inside the machine | [Yes](/docs/develop/sandboxes/docker) | Yes | [boat.dev](https://boat.dev/) |
| Full desktop | [Yes, Computers](/docs/computers/overview) | Yes | [boat.dev](https://boat.dev/) |
| Snapshots and forks | [Yes: snapshots, restore, fork, pause and resume](/docs/develop/snapshots) | Yes: stop snapshots the disk, resume and fork from it | [CLI reference](https://docs.boat.dev/cli-reference.md) |
| Fan-out from one snapshot | [Yes, `miosa batch new --count`](/docs/cli/platform) | `boat new --from <snapshot> --count` up to 1000 | [CLI reference](https://docs.boat.dev/cli-reference.md) |
| Resize a running machine | [Yes, in place while running](/docs/develop/sandboxes/sizing) | Stop first, then resume with a new size | [CLI reference](https://docs.boat.dev/cli-reference.md) |
| Public URL for a port | [Yes, previews](/docs/develop/previews) | Yes, `<sandbox>-<port>.on.boat.dev`, token-gated by default | [host CLI](https://docs.boat.dev/hosting/host-cli.md) |
| Deploy to a production URL | [Yes, with releases and rollback](/docs/services/overview) | No; always-on means a systemd unit you write | [always-on services](https://docs.boat.dev/long-running-tasks/always-on-services.md) |
| Custom domains | [Yes](/docs/services/domains) | No | [host CLI](https://docs.boat.dev/hosting/host-cli.md) |
| Managed databases | [Postgres, MySQL, Redis, MongoDB, Qdrant](/docs/data/overview) | Not in the docs index | [llms.txt](https://docs.boat.dev/llms.txt) |
| SDKs | [TypeScript, Python, Go, Elixir, Java, Rust](/docs/sdks) | Python, TypeScript | [quickstart](https://docs.boat.dev/quickstart.md) |
| CLI | [Yes](/docs/cli) | Yes | [CLI reference](https://docs.boat.dev/cli-reference.md) |
| Billing | [Per second while running, one price per size](/docs/pricing) | Per second while running; plans from $20 to $2000 a month come back as compute | [pricing](https://docs.boat.dev/pricing) |
| Regions | [us-east, us-west, us-mia](/docs/regions) | EU only | [pricing](https://docs.boat.dev/pricing) |

## Where boat leads

These are real differences, from boat's documentation:

- **Install signs you in.** The install script opens sign-in and checkout in one step ([quickstart](https://docs.boat.dev/quickstart.md)); MIOSA installs the CLI, then `miosa login` is a second step.
- **Agent self-sign-up is a documented protocol**, with a split start-then-wait login for agents ([agent auth](https://docs.boat.dev/agent-auth.md)).
- **Long commands.** `boat exec --timeout 0` runs without a limit and can detach and re-attach.
- **JSON output switches on when output is piped** ([CLI reference](https://docs.boat.dev/cli-reference.md)); MIOSA needs `--json`.
- **Fan-out from a snapshot** with `--count` up to 1000, and **per-sandbox variables** on create (`-e KEY=VALUE`).
- **Command history** with output, per sandbox (`boat history`).

## Where MIOSA differs

- **Deploy is a product**, not a recipe: deploy from a repository, a directory, or a sandbox, with releases, promote and rollback. See [Services](/docs/services/overview).
- **Custom domains** with DNS verification. See [Domains](/docs/services/domains).
- **Managed databases** with backups and connection strings. See [Databases](/docs/data/overview).
- **Live resize** of a running sandbox, and **checkpoints and restore**. See [Sizing](/docs/develop/sandboxes/sizing) and [Snapshots](/docs/develop/snapshots).
- **One interface for everyone**: MCP, a CLI, and six SDKs. See [MCP](/docs/mcp), [CLI](/docs/cli), [SDKs](/docs/sdks).
- **US regions**, and bring your own cloud on AWS and Google Cloud. See [Regions](/docs/regions) and [BYOC](/docs/infrastructure/byoc/overview).

## When to choose each

- **Choose boat** if you want EU-hosted sandboxes with SSH, unlimited-length commands, and a one-step agent sign-up, and you do not need deploys, domains, or databases from the same product.
- **Choose MIOSA** if you want sandboxes, full computers, deployments, domains, and managed data on one platform, in US regions.

> **Note: boat's own comparison**
>
> boat's compare page lists MIOSA once, as "full VMs with /dev/kvm and no 24-hour timeout" ([boat.dev/compare](https://boat.dev/compare)).
> boat also builds [battleships.dev](https://battleships.dev/), a third-party-looking comparison site; see [the Compare overview](/docs/compare#independent-comparisons).

## Sources

- boat site and docs: https://boat.dev and https://docs.boat.dev (read on 2026-10-23).
- MIOSA capabilities, as documented on this site.

- [MIOSA vs CodeSandbox](/docs/compare/codesandbox): The next comparison.
- [All comparisons](/docs/compare): Every provider MIOSA is measured against.
