---
slug: "services/static"
title: "从 Git 构建静态站点"
description: "从仓库构建静态站点，并通过静态通道提供服务，可选用构建命令和输出目录。"
section: "Services & projects"
order: 1007
---

静态服务像 Web 服务一样接收 git 源，但通过静态通道而非容器提供服务。

```bash
miosa service create shop site --type static \
  --repo https://github.com/acme/site --branch main
miosa service update shop site --build-command "npm run build" --output-dir dist
```

## 站点如何构建

构建在构建单元上运行：Railpack 会检测框架；如果仓库中有 Dockerfile，则使用该 Dockerfile。`build_command` 会替换 Railpack 原本使用的构建命令（仅对 Railpack 构建生效），`output_dir` 指定要提供服务的文件夹。

两者都未设置时，输出文件夹会自动发现：优先使用 Railpack 自带的静态根目录，然后依次查找 `dist`、`build`、`out`、`public` 和 `_site`，取第一个顶层包含 `index.html` 的目录。Dockerfile 构建还会尝试镜像自带的文档根目录（`nginx`、`httpd`、`caddy`）。

## 构建失败时

- `reason_code: build_failed`，且 `reason_detail` 以 `static_output_missing:` 开头，表示在所有候选文件夹中都没有找到 `index.html`。请将 `output_dir` 设置为构建实际写入的文件夹。
- `static_site_too_large:` 表示站点超过 500 MiB 或超过 20,000 个文件。

> **注意：框架示例**
>
> Vite 站点无需额外配置：Railpack 会检测 Vite、完成构建，输出目录为 `dist`。单页应用的回退路由是另一项改动；在此之前，指向客户端路由的深层链接应由应用自身处理。
