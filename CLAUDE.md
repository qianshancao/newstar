# Claude Plugins 开发指南

本仓库是一个 Claude Code 插件集合。每个 `plugins/<name>/` 目录是一个独立的插件（Plugin），可包含 Skills、Slash Commands、Hooks、Agents、MCP 服务器等组件。

## 官方文档

- [Plugins](https://code.claude.com/docs/en/plugins) - 插件创建指南
- [Plugins Reference](https://code.claude.com/docs/en/plugins-reference) - 插件技术规范
- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) - 插件市场创建与分发
- [Skills](https://code.claude.com/docs/en/skills) - Agent Skills 开发
- [Subagents](https://code.claude.com/docs/en/sub-agents) - 自定义子代理开发
- [Hooks](https://code.claude.com/docs/en/hooks) - 生命周期钩子
- [MCP](https://code.claude.com/docs/en/mcp) - MCP 服务器集成
- [Settings](https://code.claude.com/docs/en/settings) - 配置与权限

## 快速创建新插件

```bash
mkdir -p plugins/my-plugin/.claude-plugin
```

创建 `plugins/my-plugin/.claude-plugin/plugin.json`：

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "插件描述"
}
```

然后按需添加下面的组件。

---

## 创建 Skill

Skill 是 Claude 可通过 `/skill-name` 或自动调用的自定义能力。

### 创建步骤

```bash
mkdir -p plugins/my-plugin/skills/my-skill
```

创建 `plugins/my-plugin/skills/my-skill/SKILL.md`：

```markdown
---
name: my-skill
description: 描述做什么以及何时使用（Claude 据此自动发现和调用）
allowed-tools: Read, Grep, Glob
---

# Skill 指令内容

在这里写 Claude 应该遵循的详细指令。
可引用同目录下的附属文件，如 [reference.md](reference.md)。
```

### 可选附属文件

```
plugins/my-plugin/skills/my-skill/
├── SKILL.md          # 必需
├── reference.md      # 参考文档（Claude 按需加载）
├── scripts/
│   └── helper.sh     # 辅助脚本
└── templates/
    └── template.txt  # 模板文件
```

### Frontmatter 字段

| 字段 | 必需 | 说明 |
|------|------|------|
| `name` | 是 | 小写字母、数字、连字符，最长 64 字符 |
| `description` | 是 | 描述做什么以及何时使用，最长 1024 字符 |
| `allowed-tools` | 否 | 限制可用工具列表 |

---

## 创建 Slash Command

Slash Command 是用户可通过 `/command-name` 手动触发的提示词快捷方式。新开发推荐使用 Skill（功能相同且额外支持模型自动调用）。

### 创建步骤

```bash
mkdir -p plugins/my-plugin/commands
```

创建 `plugins/my-plugin/commands/my-cmd.md`：

```markdown
---
description: 命令描述
model: sonnet
argument-hint: [issue-number]
---

这里是命令的提示词内容。支持以下占位符：

- $ARGUMENTS - 用户传入的所有参数
- $1, $2 - 按位置引用参数
- !command - 执行 bash 命令并插入输出
- @filename - 引用文件内容
```

文件名（去掉 `.md`）即为命令名。

### Frontmatter 字段

| 字段 | 说明 |
|------|------|
| `description` | 命令描述 |
| `model` | 指定模型：`sonnet`/`opus`/`haiku`/`claude-opus-4-6` |
| `argument-hint` | 参数提示，如 `[issue-number] [priority]` |
| `allowed-tools` | 限制可用工具列表 |

### 子目录命名空间

```
commands/
├── frontend/
│   └── component.md    # → /frontend:component
└── deploy.md           # → /deploy
```

---

## 创建 Hook

Hook 是在 Claude Code 特定生命周期事件触发时自动执行的脚本。

### 创建步骤

```bash
mkdir -p plugins/my-plugin/hooks
```

创建 `plugins/my-plugin/hooks/hooks.json`：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh"
          }
        ]
      }
    ]
  }
}
```

### 事件类型

**带 matcher 的事件**（匹配工具名）：

| 事件 | 触发时机 | matcher 示例 |
|------|----------|-------------|
| `PreToolUse` | 工具调用前，可阻止执行 | `Edit\|Write`, `Bash`, `*` |
| `PostToolUse` | 工具调用成功后 | `Write`, `Read` |
| `PermissionRequest` | 权限对话框出现时 | - |
| `PostToolUseFailure` | 工具调用失败后 | - |

**不带 matcher 的事件**：

| 事件 | 触发时机 |
|------|----------|
| `SessionStart` | 会话开始或恢复（matcher: `startup`/`resume`） |
| `SessionEnd` | 会话结束 |
| `UserPromptSubmit` | 用户提交提示词后 |
| `Stop` | Claude 完成响应 |
| `SubagentStart` | 子代理启动（matcher: agent 名称） |
| `SubagentStop` | 子代理完成（matcher: agent 名称） |
| `PreCompact` | 上下文压缩前 |
| `PostCompact` | 上下文压缩后 |
| `Notification` | 发送通知时 |
| `InstructionsLoaded` | CLAUDE.md 加载时 |
| `ConfigChange` | 配置变更时 |
| `Elicitation` | MCP 请求用户输入时 |
| `TeammateIdle` | 团队成员即将空闲时 |
| `TaskCompleted` | 任务标记完成时 |
| `WorktreeCreate` | 创建 worktree 时 |
| `WorktreeRemove` | 移除 worktree 时 |

### Hook 类型

| 类型 | 说明 |
|------|------|
| `command` | 执行 shell 命令或脚本 |
| `http` | 将事件 JSON POST 到 URL |
| `prompt` | 用 LLM 评估提示词（插件中可用） |
| `agent` | 运行 agentic 验证器（插件中可用） |

### 退出码

| 退出码 | 含义 |
|--------|------|
| 0 | 成功，stdout 显示给用户 |
| 2 | 阻塞错误，stderr 反馈给 Claude 自动处理 |
| 其他 | 非阻塞错误，stderr 显示给用户，继续执行 |

### 环境变量

- `${CLAUDE_PLUGIN_ROOT}` — 插件安装目录绝对路径
- `${CLAUDE_PLUGIN_DATA}` — 持久化数据目录（跨版本保留）

---

## 创建 Agent

Agent 是独立运行的子代理，拥有自己的系统提示词、工具权限和独立上下文。

### 创建步骤

```bash
mkdir -p plugins/my-plugin/agents
```

创建 `plugins/my-plugin/agents/my-agent.md`：

```markdown
---
name: my-agent
description: 描述此 agent 何时应被调用
tools: Read, Glob, Grep, Bash
model: sonnet
maxTurns: 20
---

你是一个专业的代码审查员。分析代码并提供具体、可操作的改进建议。
```

### Frontmatter 字段

| 字段 | 必需 | 说明 |
|------|------|------|
| `name` | 是 | 唯一标识符，小写字母和连字符 |
| `description` | 是 | Claude 何时应委派给此 agent |
| `tools` | 否 | 可用工具白名单（省略则继承全部） |
| `disallowedTools` | 否 | 禁止使用的工具黑名单 |
| `model` | 否 | `sonnet`/`opus`/`haiku`/完整模型 ID/`inherit`，默认 `inherit` |
| `maxTurns` | 否 | 最大轮次 |
| `skills` | 否 | 启动时注入的 Skills 列表 |
| `memory` | 否 | 持久记忆范围：`user`/`project`/`local` |
| `background` | 否 | 设为 `true` 始终后台运行 |
| `effort` | 否 | `low`/`medium`/`high`/`max` |
| `isolation` | 否 | 设为 `worktree` 在临时 git worktree 中运行 |

> 注意：插件 agent 不支持 `hooks`、`mcpServers`、`permissionMode`（安全限制）。

### 调用方式

- **自动委派**：Claude 根据 `description` 自动决定
- **自然语言**：在提示词中提到 agent 名称
- **@-mention**：`@my-agent (agent)` 确保使用特定 agent
- **会话级**：`claude --agent my-agent` 整个会话使用该 agent

---

## 插件标准目录结构

```
plugins/my-plugin/
├── .claude-plugin/
│   └── plugin.json          # 插件清单（可选，但有则只需 name 字段）
├── skills/                  # Agent Skills
│   └── my-skill/
│       └── SKILL.md
├── commands/                # Slash Commands（legacy，新开发用 skills/）
│   └── my-cmd.md
├── agents/                  # 子代理定义
│   └── my-agent.md
├── hooks/
│   └── hooks.json           # Hook 配置
├── .mcp.json                # MCP 服务器定义
├── .lsp.json                # LSP 服务器配置
├── scripts/                 # 辅助脚本
├── settings.json            # 插件默认设置
├── LICENSE
└── CHANGELOG.md
```

## plugin.json 完整示例

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "插件描述",
  "author": {
    "name": "Author",
    "email": "[email protected]"
  },
  "repository": "https://github.com/user/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "skills": "./custom/skills/",
  "commands": ["./commands/"],
  "agents": ["./agents/"],
  "hooks": "./hooks/hooks.json",
  "mcpServers": "./.mcp.json",
  "lspServers": "./.lsp.json"
}
```

> 自定义路径补充默认目录，不替换。所有路径必须相对于插件根目录并以 `./` 开头。

---

## 创建 Marketplace

Marketplace 是插件的分发目录，让其他人可以安装你仓库中的插件。

### 创建步骤

```bash
mkdir -p .claude-plugin
```

创建 `.claude-plugin/marketplace.json`（仓库根目录）：

```json
{
  "name": "my-marketplace",
  "owner": {
    "name": "My Team"
  },
  "plugins": [
    {
      "name": "my-plugin",
      "source": "./plugins/my-plugin",
      "description": "插件描述",
      "version": "1.0.0"
    }
  ]
}
```

### marketplace.json 字段

**必需字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | Marketplace 标识符（kebab-case），用户安装时使用 |
| `owner` | object | 维护者信息，`name` 必填，`email` 可选 |
| `plugins` | array | 插件列表 |

**可选元数据：**

| 字段 | 说明 |
|------|------|
| `metadata.description` | Marketplace 描述 |
| `metadata.version` | Marketplace 版本 |
| `metadata.pluginRoot` | 基础目录，简化插件 source 路径（如 `"./plugins"` 后 source 只需写 `"my-plugin"`） |

**插件条目字段：**

| 字段 | 必需 | 说明 |
|------|------|------|
| `name` | 是 | 插件标识符（kebab-case） |
| `source` | 是 | 插件来源（见下方） |
| `description` | 否 | 插件描述 |
| `version` | 否 | 版本号 |
| `author` | 否 | 作者信息 |
| `category` | 否 | 分类 |
| `tags` | 否 | 标签 |
| `strict` | 否 | 默认 `true`；设为 `false` 时 marketplace 条目完全定义插件，无需 plugin.json |

### 插件来源类型

| 来源 | 类型 | 示例 |
|------|------|------|
| 本地相对路径 | `string` | `"./plugins/my-plugin"` |
| GitHub | `object` | `{"source": "github", "repo": "owner/repo"}` |
| Git URL | `object` | `{"source": "url", "url": "https://gitlab.com/team/plugin.git"}` |
| Git 子目录 | `object` | `{"source": "git-subdir", "url": "...", "path": "tools/claude-plugin"}` |
| npm | `object` | `{"source": "npm", "package": "@org/plugin"}` |

### 分发与安装

```bash
# 本地测试
/plugin marketplace add ./.
/plugin install my-plugin@my-marketplace

# 团队共享（推送到 GitHub 后）
/plugin marketplace add owner/repo
/plugin install my-plugin@my-marketplace

# 在 .claude/settings.json 中预配置
{
  "extraKnownMarketplaces": {
    "my-marketplace": {
      "source": {
        "source": "github",
        "repo": "owner/claude-plugins"
      }
    }
  }
}
```
