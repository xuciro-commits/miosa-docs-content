---
slug: "integrations/examples"
title: "示例应用程序"
description: "您可以克隆、在本地运行并适应您自己的堆栈的四个全堆栈白标签示例。"
section: "Platform builders"
order: 9006
---

四个完整的、可运行的示例。每个都演示了不同的前端堆栈和一组不同的白标功能。克隆仓库，并按每个条目下方的"本地运行"步骤操作。

每个示例在调整之前都应该通过三个证明：应用程序在本地启动，其 MIOSA 资源解析为预期的工作区和项目，并且浏览器永远不会收到服务器 API 密钥。

## 选一个例子

|目标|以 | 开头完成证明|
|---|---|---|
|服务器渲染集成 | `whitelabel-saas` | 客户沙箱通过限定范围的预览令牌打开 |
|使用嵌入式终端反应产品 | `whitelabel-react-saas` |刷新恢复相同属性的沙箱 |
| Next.js 市场或构建器 | `whitelabel-nextjs-marketplace` |多个项目通过规范 ID 保持分隔 |
| vue学习环境| `whitelabel-vue-tutor` | 配额和拆除适用于正确的最终用户 |

> **注意：把示例当作可执行的起点**
>
> 首先不更改地运行示例。
> 记录成功的本地 URL 和 MIOSA 资源 ID，然后一次只调整一个关注点。

所有示例都位于 [MIOSA GitHub 存储库 ](https://github.com/Miosa-osa/miosa) 根目录下的 `examples/` 中。

---

## 白标-SaaS

**技术栈：** Python (Flask) 后端 + 纯 HTML/JS 前端

**它展示了什么：**

- 一次性租户设置：自定义预览域、品牌、Webhook 注册
- 使用 `external_user_id`、`external_project_id` 和 `metadata` 为每个客户配置沙箱
- 服务器端预览令牌铸造和 iframe 嵌入
- HMAC-SHA256 webhook 签名验证
- 通过 `external_user_id` 进行拆卸（取消流程）

这是规范的参考示例。[白标演练](/docs/platform/white-label)中的每个概念都直接映射到本示例中的代码。

**本地运行：**

```bash
cd examples/whitelabel-saas
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

cp .env.example .env
# Edit .env  -  fill in MIOSA_API_KEY

python setup.py      # one-time tenant setup
python app.py        # server on http://localhost:8000
# Open index.html in a browser for the embed demo
```

**GitHub：** [`examples/whitelabel-saas`](https://github.com/Miosa-osa/miosa/tree/main/examples/whitelabel-saas)

---

## 白标签-反应-SaaS

**技术栈：** React 18 (Vite) 前端 + Express.js BFF

**它展示了什么：**

- BFF 模式：Express 铸造浏览器令牌；React 在浏览器中直接使用 `@miosa/sdk`
- 由 TypeScript SDK 驱动的 iframe 内实时终端
- 按用户存储的沙箱状态保存在 BFF 会话中
- 页面刷新时恢复沙箱（显示嵌入内容前轮询 `state === "running"`）
- 在 `miosa:preview:expired` 消息事件上刷新令牌

**本地运行：**

```bash
cd examples/whitelabel-react-saas
pnpm install

cp .env.example .env
# Edit .env  -  fill in MIOSA_API_KEY

pnpm dev   # starts both Express (port 3001) and Vite (port 5173)
# Open http://localhost:5173
```

**GitHub：** [`examples/whitelabel-react-saas`](https://github.com/Miosa-osa/miosa/tree/main/examples/whitelabel-react-saas)

---

## Whitelabel-nextjs-marketplace

**堆栈：** Next.js 15（应用程序路由器）全堆栈

**它展示了什么：**

- 用 Next.js Route Handlers 作为 BFF：无需单独的 Express 服务器即可铸造令牌
- 卖家入门：首次登录时配置沙箱，再次回来时复用
- 多项目列表：按 `external_user_id` 过滤沙箱并渲染为卡片
- 用量显示：通过 `client.usage.get(external_user_id)` 获取每个卖家的计算时长
- 自定义 `next.config.js`，通过 Next.js 服务器代理沙箱预览（避免嵌入 iframe 时的 CORS 问题）

**本地运行：**

```bash
cd examples/whitelabel-nextjs-marketplace
pnpm install

cp .env.local.example .env.local
# Edit .env.local  -  fill in MIOSA_API_KEY

pnpm dev   # starts on http://localhost:3000
```

**GitHub：** [`examples/whitelabel-nextjs-marketplace`](https://github.com/Miosa-osa/miosa/tree/main/examples/whitelabel-nextjs-marketplace)

---

## 白色标签-vue-tutor

**堆栈：** Vue 3 (Vite) 前端 + Express.js 后端

**它展示了什么：**

- 每课的沙箱生命周期：每课新建沙箱，完成后销毁
- 配额执行：用户达到上限时，自定义中间件会拒绝 `sandbox.create`
- 有效期 15 分钟的公开分享链接，用于同伴审阅
- 文件树组件：通过 `client.files.list(sandboxId, "/")` 读取沙箱文件系统
- 运行按钮组件：通过 `client.exec.stream(sandboxId, "npm test")` 流式输出 exec 结果

**本地运行：**

```bash
cd examples/whitelabel-vue-tutor
pnpm install

cp .env.example .env
# Edit .env  -  fill in MIOSA_API_KEY

pnpm dev   # starts both Express (port 3001) and Vite (port 5174)
# Open http://localhost:5174
```

**GitHub：** [`examples/whitelabel-vue-tutor`](https://github.com/Miosa-osa/miosa/tree/main/examples/whitelabel-vue-tutor)

---

## 特征矩阵

|能力|白色标签-SaaS |白色标签反应SaaS | Whitelabel-nextjs-市场 |白色标签-vue-导师 |
|---|:---:|:---:|:---:|:---:|
| 预览令牌（服务器端） |是的 |是的 |是的 |是的 |
| iframe 嵌入 |是的 |是的 |是的 |是的 |
| Webhook 验证 |是的 |  - |  - |  - |
| 浏览器令牌（客户端 SDK） |  - |是的 |  - |  - |
|直播终端 |  - |是的 |  - |是的 |
|文件树 |  - |  - |  - |是的 |
|执行流 |  - |  - |  - |是的 |
|使用情况显示|  - |  - |是的 |  - |
|配额执行 |  - |  - |  - |是的 |
|公开分享链接 |  - |  - |  - |是的 |
|用户拆解 |是的 |  - |  - |  - |

---

## 贡献一个例子

如果您在 MIOSA 上构建了某些内容并想要共享它，请在 [MIOSA GitHub 存储库 ](https://github.com/Miosa-osa/miosa) 中打开拉取请求。示例必须：

- 包含有效的 `README.md` 以及本地设置说明
- 不提交 `MIOSA_API_KEY` 或任何秘密
- 对所有资源属性使用 `external_user_id`
- 如果包含 Webhook 端点，则验证 Webhook 签名
- 包含一个烟雾命令，当主要用户旅程失败时，该命令会以非零值退出
- 固定支持的运行时和包管理器版本
- 解释预期输出和清理步骤

---

## 另请参阅

- [端到端演练](/docs/platform/white-label)
- [预览令牌](/docs/develop/previews)
- [Webhooks 参考](/docs/integrations/webhooks)
