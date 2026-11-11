---
slug: "guides/skills"
title: "MIOSA 的 Claude 代码技能"
description: "为 Claude Code 安装 MIOSA 技能，以便在您请求云基础设施时加载正确的沙箱、计算机和部署知识。"
section: "Agents"
order: 6011
---

技能通过 MIOSA 特定知识扩展了 Claude Code。安装一次后，当您请求云基础设施时，Claude 会自动加载正确的版本 - 无需手动提示。

## 你得到什么

|技能|它有什么作用 |
|---|---|
| `miosa-mcp` |通用入口点 - 解释了 145 个工具的表面和特定技能的路线 |
| `miosa-sandbox` |无头代码执行虚拟机（Python、Node、构建、CI）|
| `miosa-computer` |桌面虚拟机（浏览器自动化、GUI、计算机使用）|
| `miosa-deploy` | Git 支持的 Web 应用程序托管（Next.js、Vite、Flask、Django）|
| `miosa-data` |托管 Postgres/MySQL/Redis、S3 存储、卷、cron |

每个技能都附带一个 `examples.md`（或 `reference.md`）捆绑文件，Claude 按需加载。 SKILL.md 主体保持在 100 行以下 - 最小上下文成本，最大信号。

## 安装

```bash
git clone https://github.com/Miosa-osa/miosa-skills.git
cp -r miosa-skills/* ~/.claude/skills/
```

或者每个项目：

```bash
cp -r miosa-skills/* .claude/skills/
```

## 验证

在任意项目中打开 Claude Code 并询问：

```text
What skills are available?
```

您应该会看到列出的所有 5 个 `miosa-*` 技能。

## 他们如何激活

您不需要手动调用它们。每个技能都在其 `description` 和 `when_to_use` 前面声明触发短语。说这样的话：

- “创建一个沙箱并运行它” → `miosa-sandbox` 加载
- “截图并点击登录按钮” → `miosa-computer` 加载
-“将此存储库部署到生产环境”→ `miosa-deploy` 加载
- “我需要一个 Postgres 数据库” → `miosa-data` 加载

您还可以直接使用 `/miosa-sandbox`、`/miosa-computer` 等调用。

## 结构（官方特工技能规范）

```text
~/.claude/skills/miosa-sandbox/
├── SKILL.md         # YAML frontmatter + tight overview (<100 lines)
└── examples.md      # Workflow patterns, loaded on demand
```

每个 `SKILL.md` 包括：

- `description` - 该技能的作用是什么，何时使用它
- `when_to_use` - 自动激活的触发短语
- `allowed-tools` - 预先批准的 MCP 工具（无权限提示）

正文涵盖：生命周期、大小、约定、核心循环、错误。详细的示例位于同级文件中，因此在您需要它们之前它们不会使上下文膨胀。

## 每个技能有什么内容

### MIOSA-MCP
- 资源图（沙箱、计算机、部署、数据库、存储、卷）
- 通用约定（ID、状态轮询、大小、路径、超时）
- 指向正确资源技能的决策树
- `reference.md` 按类别提供完整的 145 种工具目录

### miosa-沙盒
- 核心循环：`create_sandbox` → `exec` → `destroy_sandbox`
- 大小指导（小=4GB，中用于构建等）
- 多文件项目的 Heredoc 技巧
- `examples.md`：构建和服务、数据处理、快照重用、PR 测试、基准测试

### miosa-计算机
- 屏幕截图→识别→点击/输入→屏幕截图→验证循环
- 坐标系（1024×768，绝对像素）
- 鼠标/键盘工具台
- `examples.md`：浏览器自动化、表单填写、检查点、服务、拖放、热键

### miosa-部署
- GitHub → 部署流程
- 通过 `sandbox_deploy` 进行沙盒促销
- 框架自动检测表
- `examples.md`：Next.js+DB，预览分支，回滚，环境更新

### miosa-数据
- 数据库创建+凭证连接
- 存储桶+预签名 URL（PUT 和 GET）
- 容量与存储决策表
- Cron 时间表参考
- `examples.md`：完整应用模式、备份管道、客户端直接上传

## 定制

技能故意紧绷。扩展：

1.在需要该模式的技能下添加`examples.md`条目
2. 将 `paths:` 添加到技能的 frontmatter 中，以将其限制为某些文件类型
3. 如果您希望对某些工具进行手动批准，请覆盖 `allowed-tools`

要覆盖技能列表（例如隐藏技能）：

```json
// .claude/settings.local.json
{
  "skillOverrides": {
    "miosa-computer": "off"
  }
}
```

## 为什么是技能而不是CLAUDE.md

CLAUDE.md 内容会加载到每个会话的上下文中。仅在相关时才加载技能。对于跨多种资源类型的大型且不断发展的工具目录，这就是每个会话燃烧约 3,000 个代币与约 200 个代币（仅描述）之间的区别。

## 更新中

```bash
cd ~/.claude/skills
git -C miosa-skills pull
cp -r miosa-skills/* .
```

Claude Code 会监视技能目录并在会话中获取更改 - 无需重新启动。

## 相关

- [MCP工具参考](/docs/mcp/tools) - 145-工具表面这些技巧包装
- [CLI](/docs/cli) - `miosa mcp install` 配置 MCP 服务器
- [快速入门](/docs/quickstart) - 获得一个在 60 秒内运行的沙箱
