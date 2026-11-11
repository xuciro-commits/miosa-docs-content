---
slug: "faq/browser-and-computer-use"
title: "浏览器和计算机使用"
description: "浏览器自动化运行的位置、启动浏览器返回的内容、CDP、实时接管和计费。"
section: "Help"
order: 16003
---

关于浏览器自动化、计算机使用循环和实时接管的简短回答。

## 浏览器自动化在哪里运行？

沙箱内。浏览器是沙箱的一部分，在第一个浏览器请求时延迟启动，然后在暂停和恢复中幸存下来。
请参阅[浏览器使用](/docs/browser-use)。

## 启动浏览器会返回什么？

一个 `stream_url` （实时视图）、一个 `cdp_url` （Chrome DevTools 协议）、一个一小时的 `stream_auth` 令牌和一个 1280x720 视口。
每个沙箱镜像都包含 Chromium 和 Node，因此任何沙箱都可以工作。
请参阅[浏览器使用快速入门](/docs/browser-use/quickstart)。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

## 智能体如何连接到浏览器？

通过 CDP 连接到人类可以观看的同一浏览器，因此 cookie 和登录信息是共享的。
完成后分离而不是调用 `browser.close()`。
请参阅[通过 CDP[[[URL7]] 驱动浏览器。

## 人类可以观看或接管会话吗？

是的。共享浏览器会话的实时查看器链接，或直接接管计算机桌面。
实时观看者需要沙箱的虚拟显示，而 SOMA 沙箱尚不具备。
请参阅[实时查看器](/docs/browser-use/live-viewer)和[嵌入实时桌面](/docs/computers/embedding)。

## 什么是计算机使用循环？

截图，决定，采取一项行动，再次截图。
坐标是原点位于左上角的屏幕像素。
请参阅[计算机使用快速入门](/docs/computer-use/quickstart)。

## 浏览器和计算机的使用如何计费？

浏览器使用账单就像普通的沙箱使用一样。
请参阅[浏览器使用限制](/docs/browser-use/limits)和[定价和限制](/docs/pricing)。
