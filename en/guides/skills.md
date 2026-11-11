---
slug: "guides/skills"
title: "Claude Code Skills for MIOSA"
description: "Install MIOSA skills for Claude Code, so it loads the right sandbox, computer, and deploy knowledge when you ask for cloud infrastructure."
section: "Agents"
order: 6011
---

Skills extend Claude Code with MIOSA-specific knowledge. Install them once and Claude automatically loads the right one when you ask for cloud infrastructure  -  no manual prompting.

## What you get

| Skill | What it does |
|---|---|
| `miosa-mcp` | Universal entry point  -  explains the 145-tool surface and routes to specific skills |
| `miosa-sandbox` | Headless code-execution VMs (Python, Node, builds, CI) |
| `miosa-computer` | Desktop VMs (browser automation, GUI, computer-use) |
| `miosa-deploy` | Git-backed web app hosting (Next.js, Vite, Flask, Django) |
| `miosa-data` | Managed Postgres/MySQL/Redis, S3 storage, volumes, cron |

Each skill ships with an `examples.md` (or `reference.md`) bundled file Claude loads on demand. SKILL.md bodies stay under 100 lines  -  minimum context cost, maximum signal.

## Install

```bash
git clone https://github.com/Miosa-osa/miosa-skills.git
cp -r miosa-skills/* ~/.claude/skills/
```

Or per-project:

```bash
cp -r miosa-skills/* .claude/skills/
```

## Verify

Open Claude Code in any project and ask:

```text
What skills are available?
```

You should see all 5 `miosa-*` skills listed.

## How they activate

You don't need to invoke them manually. Each skill declares trigger phrases in its `description` and `when_to_use` frontmatter. Say things like:

- "create a sandbox and run this" → `miosa-sandbox` loads
- "take a screenshot and click the login button" → `miosa-computer` loads
- "deploy this repo to production" → `miosa-deploy` loads
- "I need a Postgres database" → `miosa-data` loads

You can also invoke directly with `/miosa-sandbox`, `/miosa-computer`, etc.

## Structure (official Agent Skills spec)

```text
~/.claude/skills/miosa-sandbox/
├── SKILL.md         # YAML frontmatter + tight overview (<100 lines)
└── examples.md      # Workflow patterns, loaded on demand
```

Each `SKILL.md` includes:

- `description`  -  what the skill does, when to use it
- `when_to_use`  -  trigger phrases for automatic activation
- `allowed-tools`  -  pre-approved MCP tools (no permission prompts)

Body covers: lifecycle, sizes, conventions, the core loop, errors. Detailed examples live in the sibling file so they don't bloat context until you need them.

## What's in each skill

### miosa-mcp
- Resource map (sandbox vs computer vs deployment vs database vs storage vs volume)
- Universal conventions (IDs, status polling, sizes, paths, timeouts)
- Decision tree pointing to the right resource skill
- `reference.md` with the full 145-tool catalog by category

### miosa-sandbox
- Core loop: `create_sandbox` → `exec` → `destroy_sandbox`
- Size guidance (small=4GB, medium for builds, etc.)
- Heredoc trick for multi-file projects
- `examples.md`: build-and-serve, data processing, snapshot reuse, PR testing, benchmarks

### miosa-computer
- Screenshot → identify → click/type → screenshot → verify loop
- Coordinate system (1024×768, absolute pixels)
- Mouse/keyboard tool table
- `examples.md`: browser automation, form filling, checkpoints, services, drag-drop, hotkeys

### miosa-deploy
- GitHub → deployment flow
- Sandbox promotion via `sandbox_deploy`
- Framework auto-detection table
- `examples.md`: Next.js+DB, preview branches, rollback, env updates

### miosa-data
- Database creation + credential wiring
- Storage buckets + presigned URLs (PUT and GET)
- Volume vs storage decision table
- Cron schedule reference
- `examples.md`: full-app patterns, backup pipelines, client-direct uploads

## Customizing

The skills are intentionally tight. To extend:

1. Add an `examples.md` entry under the skill that needs the pattern
2. Add `paths:` to a skill's frontmatter to restrict it to certain file types
3. Override `allowed-tools` if you want manual approval on certain tools

To override a skill's listing (e.g. hide one):

```json
// .claude/settings.local.json
{
  "skillOverrides": {
    "miosa-computer": "off"
  }
}
```

## Why skills instead of CLAUDE.md

CLAUDE.md content loads into context on every session. Skills load only when relevant. For a large, evolving tool catalog across many resource types, that's the difference between burning ~3,000 tokens per session vs ~200 tokens (just the descriptions).

## Updating

```bash
cd ~/.claude/skills
git -C miosa-skills pull
cp -r miosa-skills/* .
```

Claude Code watches the skills directory and picks up changes mid-session  -  no restart needed.

## Related

- [MCP tools reference](/docs/mcp/tools)  -  the 145-tool surface these skills wrap
- [CLI](/docs/cli)  -  `miosa mcp install` to configure the MCP server
- [Quickstart](/docs/quickstart)  -  get a sandbox running in under 60s
