---
slug: "develop/sandboxes/docker"
title: "Docker in sandboxes"
description: "Run containers inside MIOSA sandboxes, publish ports, and reach them through preview URLs. Which templates ship Docker Engine, Compose, and Buildx."
section: "Sandboxes"
order: 2003
---

Docker is preinstalled in the sandbox image. There is nothing to enable, and no separate template to pick.

```bash
docker run --rm alpine echo hello
docker compose up -d
docker buildx build --load -t myapp .
```

What each template ships today:

| Template | Docker Engine | Compose and Buildx | Runs as `sandbox` user |
|---|---|---|---|
| `miosa-sandbox` (the default) | 29.9 | Yes | As `root` |
| SOMA runtime (`runtime_profile: "soma"`) | 29.9 | Yes | As `root` |

Commands run through `exec` run as `root`, so `docker` works there on every template above that includes it.
Check what a sandbox has with `docker version`, `docker compose version`, and `docker buildx version`.

## What works

| Task | Works |
|---|---|
| `docker run`, `docker exec`, `docker logs`, volumes, bind mounts | Yes |
| `docker compose up` with several services and service-name DNS between them | Yes |
| `docker buildx build` | Yes |
| `docker build` | Yes |
| Pulling from Docker Hub and other public registries | Yes |
| Publishing a port with `-p 8080:80` and opening it through a preview URL | Yes |
| Running as `root` | Yes |
| Running as the `sandbox` user | Yes, when you `exec` as that user |
| Pausing and resuming a sandbox with containers running | Yes, containers keep running |

## Startup behavior

The Docker engine starts the first time you use it, not when the sandbox boots, so sandbox creation stays fast. The first `docker` command in a fresh sandbox takes about 0.6 seconds longer while the engine starts; after that, running an image that is already pulled takes about 0.15 seconds.

If you start containers with `--restart always`, they come back when the engine next starts, which happens on your next `docker` command.

## Open a container port in your browser

Publish the port on the sandbox, then expose it as a [preview](/docs/develop/previews).

```bash
docker run -d --name web -p 8080:80 nginx:alpine
```

**TypeScript**

```typescript
const sbx = await miosa.sandboxes.create({ templateId: "miosa-sandbox" });
await sbx.exec.run("docker run -d --name web -p 8080:80 nginx:alpine");
const preview = await sbx.previews.create({ port: 8080 });
console.log(preview.url);
```

**Python**

```python
with miosa.sandboxes.create("miosa-sandbox") as sbx:
    sbx.exec.run("docker run -d --name web -p 8080:80 nginx:alpine")
    print(sbx.get_url(8080))
```

The preview URL forwards to the published port. Bind to `0.0.0.0` inside the container, which `-p` does for you.

## Disk and memory

Images, layers, and volumes live on the sandbox disk, so they count against its size. The default `small` sandbox has 10 GiB of disk. When the disk is full, a pull or build fails with `no space left on device` and the sandbox keeps running. Free space with `docker system prune -af` and retry.

The engine uses about 40 MiB of memory when idle. Containers share the sandbox's CPU and memory.

## Docker in a sandbox vs Docker Host

| You want | Use |
|---|---|
| Run, build, or test containers while you develop, or let an agent do it | Docker in a sandbox |
| A durable 24/7 deployment URL, custom domains, and rollback for a containerized app | [Docker Host](/docs/docker-host/docker-deploy) |
| A sandbox whose only job is to be a Docker host with a long-lived disk | A sandbox with `always_on` set |

Docker in a sandbox is for development and automation. It is not a production runtime.

## Templates

- Every sandbox image includes Docker Engine 29.9 with Compose and Buildx, on Firecracker and on SOMA.
- There is no separate Docker template to pick: the sandbox image already includes Docker.

## Limits

- No nested virtualization. KVM and QEMU do not run inside a sandbox.
- Image builds for other CPU architectures need emulation that is not included.
- Docker Swarm ingress is not supported.

## See also

- [Previews](/docs/develop/previews): Expose a sandbox port on a public URL.
- [Templates](/docs/develop/templates): Choose or build a sandbox image.
- [Docker Host](/docs/docker-host/docker-deploy): Publish a containerized app to a durable, always-on URL.
