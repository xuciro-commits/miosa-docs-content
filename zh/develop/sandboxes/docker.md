---
slug: "develop/sandboxes/docker"
title: "沙箱中的 Docker"
description: "在 MIOSA 沙箱内运行容器、发布端口并通过预览 URL 访问它们。哪些模板附带 Docker Engine、Compose 和 Buildx。"
section: "Sandboxes"
order: 2003
---

Docker 已预安装在沙箱映像中。无需启用任何内容，也无需选择单独的模板。

```bash
docker run --rm alpine echo hello
docker compose up -d
docker buildx build --load -t myapp .
```

每个模板今天发布的内容：

|模板| Docker 引擎 |撰写和构建 |以 `sandbox` 用户身份运行 |
|---|---|---|---|
| `miosa-sandbox`（默认）| 29.9 | 是的|作为 `root` |
| SOMA 运行时 (`runtime_profile: "soma"`) | 29.9 | 是的|作为 `root` |

命令通过 `exec` 运行为 `root`，因此 `docker` 适用于上面包含它的每个模板。
使用 `docker version`、`docker compose version` 和 `docker buildx version` 检查沙箱的内容。

## 什么有效

|任务|作品 |
|---|---|
| `docker run`、`docker exec`、`docker logs`、卷、绑定安装 |是的 |
| `docker compose up` 具有多个服务和它们之间的服务名称 DNS |是的 |
| `docker buildx build` |是的 |
| `docker build` |是的 |
|从 Docker Hub 和其他公共注册表中提取 |是的 |
|使用 `-p 8080:80` 发布端口并通过预览 URL 打开它 |是的 |
|作为 `root` | 运行是的 |
|以 `sandbox` 用户身份运行 |是的，当您 `exec` 作为该用户 |
|暂停和恢复容器运行的沙箱 |是的，容器继续运行 |

## 启动行为

Docker 引擎在您第一次使用时启动，而不是在沙箱启动时启动，因此沙箱创建速度很快。引擎启动时，新沙箱中的第一个 `docker` 命令需要大约 0.6 秒的时间；之后，运行已拉取的镜像大约需要 0.15 秒。

如果您使用 `--restart always` 启动容器，它们会在引擎下次启动时返回，这发生在您的下一个 `docker` 命令上。

## 在浏览器中打开容器端口

在沙盒上发布端口，然后将其公开为[预览](/docs/develop/previews)。

```bash
docker run -d --name web -p 8080:80 nginx:alpine
```

**打字稿**

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

预览 URL 转发到已发布的端口。绑定到容器内的 `0.0.0.0`，这 `-p` 会为你做的。

## 磁盘和内存

图像、图层和卷位于沙盒磁盘上，因此它们会根据其大小进行计算。默认的 `small` 沙箱有 10 GiB 的磁盘。当磁盘已满时，拉取或构建会失败并显示 `no space left on device`，并且沙箱继续运行。使用 `docker system prune -af` 释放空间并重试。

引擎在空闲时使用大约 40 MiB 内存。容器共享沙箱的CPU和内存。

## 沙箱中的 Docker 与 Docker 主机

|你想要|使用 |
|---|---|
|在开发时运行、构建或测试容器，或者让智能体来做 |沙箱中的 Docker |
|容器化应用程序的持久 24/7 部署 URL、自定义域和回滚 | [Docker 主机](/docs/docker-host/docker-deploy) |
|一个沙箱，唯一的工作就是成为一个拥有长寿命磁盘的 Docker 主机 |设置了 `always_on` 的沙箱 |

沙箱中的 Docker 用于开发和自动化。它不是生产运行时。

## 模板

- 每个沙箱镜像都包含 Docker Engine 29.9，在 Firecracker 和 SOMA 上带有 Compose 和 Buildx。
- 没有单独的 Docker 模板可供选择：沙箱映像已包含 Docker。

## 限制

- 无嵌套虚拟化。 KVM 和 QEMU 不在沙箱内运行。
- 其他 CPU 架构的映像构建需要仿真，但未包含在内。
- 不支持 Docker Swarm 入口。

## 另请参阅

- [预览](/docs/develop/previews)：在公共 URL 上公开沙箱端口。
- [模板](/docs/develop/templates)：选择或构建沙盒图像。
- [Docker Host](/docs/docker-host/docker-deploy)：将容器化应用程序发布到持久、始终在线的 URL。
