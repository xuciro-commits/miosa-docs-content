---
slug: "ci"
title: "MIOSA 上的 CI"
description: "在 MIOSA 上运行 GitHub Actions 作业。连接 MIOSA CI 应用程序，在工作流程中使用运行式 miosa，每个作业都在自己的沙箱中运行，按秒计费。"
section: "Services & projects"
order: 1040
---

MIOSA 上的 CI 在 MIOSA 沙箱中运行 GitHub Actions 作业。
您连接 MIOSA CI GitHub 应用程序，为其提供存储库，在工作流程中将 `runs-on: ubuntu-latest` 更改为 `runs-on: miosa` 并推送。
然后，每个作业都在一个新的沙箱中运行，按秒计费，并且沙箱在作业结束时被删除。

控制台中的 **CI on MIOSA**、`miosa ci` 命令和 `GET /api/v1/ci` 中的状态相同。

## 设置

有四个步骤。
控制台页面和 `miosa ci status` 显示哪些已完成。

1. **在 GitHub 上安装 MIOSA CI 应用程序。**
   在控制台中，打开 **CI on MIOSA** 并选择 **连接 GitHub**。
   从终端运行 `miosa ci connect`;它会打开相同的链接并等待直到连接出现。
2. **授予其访问您的存储库的权限。**
   在 GitHub 上，选择所有存储库或应在 MIOSA 上运行的存储库。
   您可以稍后在 GitHub 上的应用程序设置中更改此设置。
3. **在工作流程中使用 `runs-on: miosa`。**
4. **工作由 MIOSA 传递。**

一个 GitHub 帐户一次属于一个 MIOSA 组织。
如果它已连接到另一个组织，请先在那里断开连接。

GitHub 返回给您的链接有效期为 30 分钟。
如果过期，请再次选择“**连接 GitHub**”。

## 改变工作流程

将作业的标签设置为 `miosa`：

```yaml
jobs:
  test:
    runs-on: miosa
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```

标签必须恰好为 `miosa`。
具有其他标签的作业将继续在之前运行的位置运行。
您一次可以调动一份工作。

## 隔离

创建 CI 沙箱时不使用组织的 MIOSA 环境变量、秘密文件、数据库链接、Git 密钥或模型密钥。
它仅包含运行程序图像包含的内容以及 GitHub 发送给运行程序的内容。

将工作秘密保存在 GitHub Actions 秘密中，并将它们传递到您今天所做的步骤：

```yaml
- run: ./deploy.sh
  env:
    API_TOKEN: ${{ secrets.API_TOKEN }}
```

每个作业都有自己的沙箱。
没有什么可以从一份工作转移到下一份工作。

> **注意：公共存储库**
>
> MIOSA 默认拒绝来自公共存储库的作业。
> 任何人都可以在公共存储库上打开拉取请求，并且它启动的作业将根据您组织的账单运行。
> 该作业显示在 **最近的作业** 中并附有原因。

## 计费

与任何其他沙箱一样，CI 沙箱从创建到删除，按秒计费。
该法案将此用途显示为 **CI**，与其他沙箱分开。

## 故障排除

### 该作业在 GitHub 上保持“排队”状态

- 工作流程必须准确地说`runs-on: miosa`。
- MIOSA CI 应用程序必须有权访问该存储库。
  将其添加到 GitHub 上的应用程序设置中。
- 运行 `miosa ci status` 查看哪个设置步骤未完成，并运行 `miosa ci jobs` 查看作业是否到达 MIOSA。
- 默认情况下拒绝公共存储库。

### 存储库是公开的

除非 MIOSA 允许您的组织使用公共存储库，否则公共存储库将被拒绝。
在 GitHub 自己的运行器上运行公共存储库，或寻求支持。

### 步骤看不到 MIOSA 环境变量

这是有意的。
将值放入 GitHub Actions 密钥中，并使用 `env:` 将其传递到步骤。

### 作业启动缓慢或等待

每个作业都会获得一个新的沙箱，并且运行者在第一步开始之前在其中注册。
在**最近的作业**中，**已排队**表示 MIOSA 尚未启动沙箱，**启动**表示正在准备中。
如果作业停留在任一状态或失败，该行会显示原因。
组织的沙箱限制或空信用余额可以阻止新的沙箱。

### 一个步骤需要 `ubuntu-latest` 拥有的工具

运行程序镜像包含 GitHub Actions 运行程序和 `ubuntu-latest` 中的常用工具，但并非每一个都有。
对于带有 `actions/setup-node`、`actions/setup-python` 或 `actions/setup-go` 的语言，安装步骤中缺少的内容。

### 调试失败的作业

从**最近作业**中的链接打开运行；步骤日志位于 GitHub 上。
作业结束时沙箱将被删除，因此之后没有任何内容可以打开。
在 GitHub 上，选择 **重新运行作业** 并启用调试日志记录以获取更多输出。
`miosa ci jobs --json` 给出每个作业的状态、结论、持续时间和失败原因。

## 命令行界面

| 命令 | 作用 |
|---|---|
| `miosa ci connect` | 打开 GitHub 安装链接并等待连接出现 |
| `miosa ci status` | 显示四个设置步骤和隔离说明 |
| `miosa ci jobs` | 列出最近的作业 |
| `miosa ci disconnect <installation>` | 断开 GitHub 帐户的连接；正在运行的作业会跑完 |

每个命令都采用 `--json`。

## API

|请求 |它有什么作用 |
|---|---|
| `GET /api/v1/ci` |设置状态：`available`、连接安装、四个`steps`、`last_job`和`isolation`注意|
| `POST /api/v1/ci/connect-intents` |返回 GitHub `install_url`，有效期 30 分钟 |
| `GET /api/v1/ci/jobs?limit=20` |最近的作业涉及 `status`、`conclusion`、`duration_seconds`、`run_url` 和 `failure_reason` |
| `DELETE /api/v1/ci/installations/:installation_id` |断开 GitHub 帐户的连接；正在运行的作业完成 |

作业 `status` 为 `queued`、`starting`、`running`、`completed`、`failed` 或 `cancelled`。
错误使用平台错误形状。
代码为 `CI_NOT_AVAILABLE`、`CI_INSTALLATION_NOT_FOUND`、`CI_INSTALLATION_OWNED_ELSEWHERE` 和 `CI_STATE_INVALID`。

```bash
curl -s https://api.miosa.ai/api/v1/ci \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```
