---
slug: "concepts/computers-desktop"
title: "Computers (desktop product)"
description: "The MIOSA cloud desktop product: full GUI environments with screenshot, click, type, and keyboard APIs, and how they differ from Sandboxes."
section: "Optional"
order: 17008
---

**Computers** are full desktop environments running in the cloud with screenshot, click, type, and keyboard APIs. They are a separate product surface from Sandboxes  -  different lifecycle, different use cases.

| | Sandbox | Computer |
|---|---|---|
| Primary use | Code execution, file I/O, dev servers, agent tool-use | Desktop GUI, browser automation, computer-use agents |
| Display | None (headless) | Desktop stream |
| Accessibility tree | No | Yes |
| Lifecycle | Short-to-medium lived, auto-suspend | Session-oriented |
| Template | `miosa-sandbox` (the default) | `miosa-desktop` |

Use Sandboxes for building apps. Use Computers when your agent or workflow needs a real screen.

You can also bring your own hardware (Mac, Linux, Windows) and register it as a Computer host via BYOC.

```bash
curl -X POST https://api.miosa.ai/api/v1/computers \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"my-computer","template_type":"miosa-desktop","size":"small"}'
```

[Computers overview →](/docs/computers/overview)

## Three naming layers

One thing trips up every new reader: "Computer" has a specific meaning in MIOSA.

| Layer | Term | Meaning |
|-------|------|---------|
| Brand | **MIOSA** | The wordmark and platform. |
| Resource | **Computer** | The desktop GUI product. Not a generic synonym for "VM." |
| AI agent | **OSA agent** | The agent runtime. Optional; installable inside a Sandbox or Computer. |

Sandboxes are not Computers. Runtime Instances are not Computers. Computers are Computers. When you encounter older docs or code that uses "Computer" generically, substitute "Sandbox" or "Runtime Instance" based on context.

## Next

- [Resource model](/docs/concepts/resource-model): what a Sandbox is.
- [Computers overview](/docs/computers/overview): the full Computers reference.
- [Core Concepts](/docs/concepts): the one-screen overview.
