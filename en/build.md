---
slug: "build"
title: "Build an app like..."
description: "Guides for platform builders. Run open-source agents and frameworks on MIOSA, or build familiar AI products like Lovable, Manus and Perplexity - app builders, coding agents, autonomous and computer-use agents, chat and companions, answer engines, multi-agent, voice, design, video, vertical apps, and an internal Business OS."
section: "Build"
order: 10000
---

Build

Product shapes people already know, mapped onto MIOSA primitives. Build any of them for your own company or as a product for your customers, on the same architecture. Each guide is one page: what it is, the resources you need, an architecture diagram, six steps you can run, and an agent prompt for your coding agent.

[Compare MIOSA](/docs/compare)[Platform builder guide](/docs/platform/overview)[Sandboxes](/docs/develop/sandboxes)[Deploy and Docker Host](/docs/docker-host/overview)

Every guide is one of two kinds:

- **Run X on MIOSA.** Open-source agents, frameworks and tools you can actually install and use (OpenClaw, Hermes Agent, OpenHands, Claude Code, Codex, Cursor's CLI agent, CrewAI, AutoGen, LangGraph, Pipecat, Browser Use, Stagehand, Steel). The guide and its agent prompt install the real tool from its official docs, run it on a MIOSA sandbox or computer, and then wrap it for your team or your customers.
- **Build an app like X.** Closed products you cannot run yourself (Lovable, Bolt, v0, Manus, Perplexity, Character.AI and others). The guide names the product, maps each of its capabilities onto a MIOSA primitive, and builds your own version.

Either kind can be set up **two ways, on the same architecture**:

- **For your own company (internal).** One organization, a workspace per team, access control and SSO, and an internal domain. No billing.
- **As a product (for your customers).** Multi-tenant: white-label branding, an organization or workspace per customer, custom domains, and usage metering.

Each guide ships an **agent prompt tailored to what you are building** - pick the shape and a name, and hand the brief to OSA, Claude Code, Codex, or Cursor. Browse the apps and platforms below.

MIOSA is the execution layer underneath AI products. The core idea runs through every guide:
**build your product powered by Claude Code, Codex, Hermes, OpenClaw, OSA, or your own model**.
The harness runs inside a MIOSA sandbox or computer, and your product wraps it.

There are two paths, and they are not exclusive:

- [Run the agents as-is, for your team](/docs/build/run-agents): Background coding agents, parallel sessions per sandbox, and a persistent workspace per developer. No product to build: just run Claude Code, Codex, Cursor, or your own harness in the cloud and watch the work.
- [Build a branded product on top](/docs/build/coding-agent-product): Wrap a harness in your own UI, model, and billing: a Lovable-style builder, an internal workspace, or a customer-facing product, using Claude Code, Hermes, OpenClaw, or your own model under the hood.

## Ideas to build

- [Your own Claude-desktop-style app](/docs/build/claude-cowork): A branded desktop/web app where your users chat with an agent that works in its own cloud computer, with a browser and files.
- [Artifacts with links](/docs/build/lovable): The agent builds something (a site, a doc, a dashboard, or an app), shares it at a preview URL, then promotes it to a deployment on your own branded domain.
- [An internal AI workspace](/docs/build/business-os): An Acme Corp-style internal platform: one place for your team's agents, apps, and data, behind your own login.
- [A customer-facing product](/docs/platform/white-label): The same shape, Lumen-style and white-label: your brand, your domains, and your end users, with MIOSA running behind them.
- [Run coding agents in the cloud](/docs/build/run-agents): Put Claude Code, Codex, or Cursor agents in sandboxes: background work, parallel sessions, and a persistent workspace each.
- [A multi-agent team](/docs/build/openswarm): An orchestrator that fans work out to child agents in their own machines and joins the results.

### AI app builders

Prompt to a running, previewable, deployable app.

- [Build a Lovable-style app Prompt a full-stack app, preview it live, publish it.](/docs/build/lovable)
- [Build a Bolt-style app Full-stack apps from a prompt, on a real dev server.](/docs/build/bolt)
- [Build a v0-style app Generate a UI from a prompt and publish it.](/docs/build/v0)
- [Build a Replit-Agent-style app A cloud workspace where a prompt becomes a running app.](/docs/build/replit)
- [Build a Base44-style app Describe an app, get a working, previewed one.](/docs/build/base44)
- [Build an Emergent-style app Prompts become running software with real infrastructure.](/docs/build/emergent)
- [Build a Rork-style app Prompt to a mobile app you can preview.](/docs/build/rork)
- [Build an Anything-style app Prompt to a working web or mobile app.](/docs/build/anything)
- [Build a Same-style app Clone and rebuild a UI from a screenshot or URL.](/docs/build/same-new)

### Run coding agents

Install Claude Code, Codex, Cursor, Cline and OpenHands on MIOSA, or build a product on them.

- [Run coding agents in the cloud Claude Code, Codex, Cursor: background agents in the cloud.](/docs/build/run-agents)
- [Run Claude Code on MIOSA Install it in a sandbox; stream and collect its work.](/docs/build/claude-code)
- [Run Codex on MIOSA Install the Codex CLI in a sandbox with streamed events.](/docs/build/codex)
- [Run Cursor agents on MIOSA Install the Cursor CLI agent and run it headless on isolated machines.](/docs/build/cursor)
- [Build a Windsurf-style agentic IDE An editor with an agent that edits across files and runs commands.](/docs/build/windsurf)
- [Build a GitHub Copilot-style coding agent An agent that takes an issue, works, and opens a PR.](/docs/build/github-copilot)
- [Run OpenHands on MIOSA Install the open-source software agent; the sandbox is its workspace.](/docs/build/openhands)
- [Run Cline on MIOSA The most-installed open-source coding agent, headless in a sandbox.](/docs/build/cline)
- [Build a coding-agent product Wrap a harness in your own branded product.](/docs/build/coding-agent-product)

### Autonomous agents

Agents with their own machine that plan and finish work.

- [Build a Manus-style agent A general agent with a desktop, browser, and files.](/docs/build/manus)
- [Build a Genspark-style agent A super-agent that plans and produces finished work.](/docs/build/genspark)
- [Build a Devin-style SWE agent An autonomous engineer with a shell, IDE, and browser.](/docs/build/devin)
- [Build a Cowork-style desktop agent An agent that works alongside you on your desktop.](/docs/build/claude-cowork)

### Browser and computer-use agents

Build your own AI browser, platform or tool on MIOSA, or run an open-source browser agent.

- [Build your own browser agent Your model and loop, driving a sandbox browser or a full computer.](/docs/build/chatgpt-agent)
- [Build a Polar-style AI browser An AI browser that runs long tasks and scheduled workflows.](/docs/build/polar)
- [Run Browser Use on MIOSA Install the open-source library and let an agent drive a real browser.](/docs/build/browser-use)
- [Run Stagehand on MIOSA Install the framework and run reliable browser automations on a computer.](/docs/build/stagehand)

### AI chat and companions

Assistants and always-on companions.

- [Build a dots-style companion Always-on companions, each with its own cloud computer.](/docs/build/dots)
- [Build a Grok Bot-style chat app A team of named bots sharing one cloud computer.](/docs/build/grok-bot)
- [Build a Muse-style companion A personal companion that acts on your behalf across apps.](/docs/build/meta-muse)
- [Build a Character-style companion Persona companions users chat with over time.](/docs/build/character-ai)
- [Build a Replika-style companion A personal companion with long-term memory.](/docs/build/replika)
- [Build a Pi-style companion A warm, conversational personal assistant.](/docs/build/pi)
- [Run OpenClaw on MIOSA Install the open-source assistant on a persistent computer that never sleeps.](/docs/build/openclaw)

### Answer engines

Research products that search, read, and answer with sources.

- [Build a Perplexity-style answer engine Search, read, and answer with citations.](/docs/build/perplexity)
- [Build a Comet-style agentic browser A browser that navigates and answers in place.](/docs/build/comet)
- [Build a You.com-style search and answer API An API your agents call to search and answer, with sources.](/docs/build/you-com)
- [Build a Phind-style answer engine Technical answers rendered as interactive mini-apps.](/docs/build/phind)
- [Build a deep-research-style agent A long-running research agent producing a report.](/docs/build/deep-research)

### Multi-agent

One request, many agents working in parallel.

- [Build an OpenSwarm-style multi-agent app An AI desktop built around running several agents in parallel.](/docs/build/openswarm)
- [Run Hermes Agent on MIOSA Install the open-source agent harness and run many tool-using agents.](/docs/build/hermes-agent)
- [Build an orchestrator that delegates One agent that plans, delegates to sub-agents and joins the result.](/docs/build/gemini-agent)

### Voice agents

Agents that talk and act on calls.

- [Build an ElevenLabs-style voice agent A voice agent that talks and takes real actions.](/docs/build/elevenlabs)
- [Build a Vapi-style voice agent Voice agents for phone and web.](/docs/build/vapi)
- [Build a Retell-style voice agent Phone agents that handle real calls.](/docs/build/retell)
- [Build a Bland-style phone agent Programmable phone agents at scale.](/docs/build/bland)
- [Run Pipecat on MIOSA Install the open-source voice pipeline from speech-to-text to text-to-speech.](/docs/build/pipecat)
- [Run LiveKit Agents on MIOSA The leading open-source realtime voice agent framework.](/docs/build/livekit-agents)

### AI design tools

Prompt to a designed UI or asset.

- [Build a prompt-to-UI design tool Prompt (and a design file) to a working, previewed UI.](/docs/build/figma-make)
- [Build a sketch-to-mockup tool A sketch or screenshot becomes editable UI code.](/docs/build/uizard)
- [Build a Magic-Patterns-style tool Generate UI components from a prompt.](/docs/build/magic-patterns)

### AI video and creative

Generate, edit, and produce media.

- [Build a video-generation app Generate and edit video on top of video-model APIs.](/docs/build/runway)
- [Build a text-to-video app Prompt to video through a video-model API.](/docs/build/sora)
- [Build an avatar-video app Script to talking-avatar video with TTS and lip-sync APIs.](/docs/build/heygen)
- [Build a Higgsfield-style video studio A multi-model studio: brief, steer camera and look, generate.](/docs/build/higgsfield)
- [Build a Gamma-style deck app Prompt or document to a polished, shareable deck or page.](/docs/build/gamma)
- [Build a Descript-style editor Edit video by editing the transcript.](/docs/build/descript)

### Vertical AI apps

Agents for a specific industry workflow.

- [Build a Fin-style support agent An AI support agent resolving customer tickets.](/docs/build/intercom-fin)
- [Build a Decagon-style support agent Resolve tickets across chat and email, with handoff.](/docs/build/decagon)
- [Build a Clay-style sales agent Research and enrich leads at scale.](/docs/build/clay)
- [Build a post-call analysis agent Turn call transcripts into sourced insights and next steps.](/docs/build/gong)
- [Build an OpenEvidence-style evidence engine Cited answers over a trusted corpus.](/docs/build/openevidence)

### Internal company apps

Build these for your own company: one organization, a workspace per team.

- [Build an internal Business OS MIOSA's own BusinessOS template, Apache 2.0.](/docs/build/business-os)
- [Build an internal AI workspace One place for your team's agents, apps, and data.](/docs/build/team-ai-workspace)
- [Build an ops and support copilot Triage tickets and run runbooks, with human approval.](/docs/build/ops-support-copilot)
- [Build an internal tools builder Prompt to a shipped dashboard behind your domain.](/docs/build/internal-tools-builder)
- [Build a knowledge-base agent Answer from the company docs with citations.](/docs/build/knowledge-base-agent)
- [Build an employee onboarding agent Walks a new hire through their first week.](/docs/build/employee-onboarding-agent)

Trademarks belong to their owners; these guides show how to build similar products on MIOSA. No affiliation or endorsement is implied.

> **Note: Same template, every guide**
>
> Each guide has the same shape: a **prompt builder**, **What you're building**, a
> **What you need on MIOSA** checklist with a create command per resource, an **architecture
> diagram**, six numbered steps (**create an account and API key → create the resources → wire
> the agent loop → preview → deploy → custom domain**), **costs and limits**, and **next steps**.

## Start with the platform

- [Platform builder guide](/docs/platform/overview): Organizations, workspaces, white-label, tenants, and billing.
- [Agent builds an app](/docs/cookbook/agent-builds-app): The end-to-end recipe every app-builder guide is built on.
- [Agent orchestration](/docs/guides/agent-orchestration): Coordinate sandbox and computer agents, artifacts, and deployments.
- [Compare MIOSA](/docs/compare): How MIOSA compares to other sandbox and agent platforms.
