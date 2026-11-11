---
slug: "concepts/resource-model"
title: "Resource model"
description: "The MIOSA ownership chain and the mutable development resources - Organization, Workspace, Project, Sandbox, and Preview."
section: "Optional"
order: 17006
---

This page is the **resource model** of the mutable side of MIOSA: the ownership chain and the development resources.
For the production resources and the publish pipeline, see [The publish pipeline](/docs/concepts/publish-pipeline).

## The resource hierarchy

MIOSA resolves every operation through a clear ownership chain:

```text
Organization → Workspace → Project → Sandbox or Deployment
```

- An **Organization** is the top-level customer or platform account.
- A **Workspace** isolates a team, customer, or business boundary.
- A **Project** groups the development and production resources for one product.
- A **Sandbox** is the mutable place where work happens.
- A **Deployment** is the durable production identity that owns versions, routes, and bindings.

Every mutation must resolve one unambiguous organization, workspace, project, and target resource.
This prevents an agent or CLI command from publishing into the wrong customer or environment.

## Development resources

### Sandbox

A **Sandbox** is a mutable, isolated environment where an agent or developer builds an app.
It includes a complete development environment with Node, Python, git, package managers, file APIs, and a terminal.

Key properties:

- **Mutable and stateful.** Files in `/workspace` persist across commands until the sandbox is destroyed or suspended.
- **Executable.** Run commands such as `npm install`, `python app.py`, or `git clone`, with structured stdout, stderr, and exit status.
- **Agent-friendly file access.** Read, write, and synchronize source, configuration, and assets.
- **Previewable.** Expose a running port through a temporary Preview URL.
- **Snapshot-aware.** Save, fork, and restore development state without treating that snapshot as a production release.
- **Suspendable.** Idle compute can pause and later resume from durable state.

A Sandbox is not production.
Its files can change at any time, its process can stop, and its Preview can disappear while suspended.
Use it to create and test the source that will become a production candidate.

```bash
miosa create my-box --wait     # provision a sandbox and wait until it is ready
miosa exec my-box -- echo "hello from MIOSA"
```

Deep-dive on Sandboxes, file I/O, exec, previews, and the dev loop.
[Sandboxes reference →](/docs/develop/sandboxes)

### Preview

A **Preview** is a temporary public URL that tunnels to one port inside a running Sandbox.

```text
https://5173-abc12345.sandbox.miosa.ai
```

When you create a Preview, you specify the port, usually `3000` or `5173`.
MIOSA routes that hostname to the Sandbox process.
As files change and the development server reloads, the Preview changes too.

Key properties:

- **Mutable.** Every request reflects the current Sandbox process and files.
- **Ephemeral.** The URL depends on a running Sandbox and is not a durable customer URL.
- **Shareable.** Short-lived tokens allow browser access without exposing a workspace API key.

```bash
miosa preview create my-box 3000 --visibility public
```

[Previews reference →](/docs/develop/previews)

## Next

- [The publish pipeline](/docs/concepts/publish-pipeline): Builder, Release, Runtime Instance, Domain, Deployment, and Version.
- [Data and bindings](/docs/concepts/data-and-bindings): durable resources that outlive a Sandbox.
- [Core Concepts](/docs/concepts): the one-screen overview.
