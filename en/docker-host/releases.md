---
slug: "docker-host/releases"
title: "Releases"
description: "The immutable build artifact: static files or dynamic server artifact, sha256-keyed."
section: "Services & projects"
order: 1028
---

A **Release** is the immutable artifact produced by a build. Versions reference releases; static deployments serve files from them and dynamic deployments start runtimes from them.

You typically don't address releases directly through the API. They're an implementation detail of `Deployment Version`. But it's useful to know they exist as their own record because they're what makes rollback and horizontal scaling work.

> **Note: Caller contract**
>
> Most integrations do not need to read release records directly. Use deployment versions and the returned deployment `public_url` unless you are building an operator dashboard or audit workflow.

## Release kinds

| Kind | Used by | Storage |
|---|---|---|
| `static` | Static deployments | Built files and assets |
| `oci` | Dynamic deployments via OCI-compatible workflows | Server image artifact |
| `dynamic` | Server apps with a run command and port | Packaged server artifact |

The release format determines how the deployment is served:

- **Static releases** → MIOSA's static artifact server streams files from object storage directly. No per-app process runs.
- **Dynamic releases** → MIOSA starts healthy production runtimes and routes traffic to them.

## Immutability

Releases are content-addressed by sha256. A release with sha256 `abc...` is always the same bytes, anywhere it's referenced. This is what makes:

- **Rollback** trivial: repoint a domain at an older version, which still references its release.
- **Scaling** trivial: schedule N copies of the same release on N hosts; every instance is byte-identical.
- **Audit** clean: the release that served traffic at time T can be reproduced from its hash.

A release is never mutated after creation. To "change" it, publish a new version, which produces a new release.

## Where releases live

Releases are stored and served by MIOSA. You can inspect release metadata on the version record:
```json
{
  "version_number": 17,
  "artifact_sha256": "a5e6f0c1...",
  "artifact_manifest": {
    "files": 42,
    "total_bytes": 184320,
    "entrypoint": "index.html"
  }
}
```

You do not need to download releases yourself. The metadata is shown for debugging and audit.

## Source snapshot vs release

These are two different artifacts in the same build:

| Artifact | What it is | Sha256 stored on |
|---|---|---|
| **Source snapshot** | Frozen tarball of the sandbox source at publish time | `version.source_sha256` |
| **Release** | The built output | `version.artifact_sha256` |

The source snapshot exists so retries / rebuilds are reproducible. The release exists so production can be served and scaled.

## Garbage collection

Static release artifacts older than the retention window (default 30 days for archived versions) may be garbage-collected. The version row remains for audit; if you attempt to rollback to a version whose artifact has been GC'd, MIOSA returns 410 Gone.

Active versions (`active_version_id` on any deployment) are never GC'd, even past retention.

## See also

- [Versions](/docs/docker-host/versions): the record referencing the release
- [Dynamic Runtimes](/docs/docker-host/runtime-instances): what serves dynamic releases
- [Publishing](/docs/docker-host/publishing): how releases are created
