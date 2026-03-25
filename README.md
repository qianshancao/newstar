# Claude Plugins

Claude Code 插件集合，包含 Skills、Slash Commands、Hooks、Agents 等 Claude Code 扩展。

## 插件列表

| 插件 | 类型 | 说明 |
|------|------|------|
| [cnb-issue](plugins/cnb-issue/) | Skill | 操作 CNB (cnb.cool) 平台上的 Issue |
| [capability-trainer](plugins/capability-trainer/) | Skill | 通过实践引导学习和掌握新能力 |

## 安装

将插件目录复制到项目的 `.claude/plugins/` 目录下即可使用：

```bash
# 安装单个插件
cp -r plugins/cnb-issue .claude/plugins/

# 安装所有插件
cp -r plugins/* .claude/plugins/
```

## 插件结构

每个插件遵循标准结构：

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # 插件元数据（必需）
├── skills/              # 技能定义
├── commands/            # 斜杠命令（可选）
├── agents/              # 子代理定义（可选）
├── hooks/
│   └── hooks.json       # Hook 配置（可选）
└── README.md            # 插件文档
```

## 开发新插件

1. 在 `plugins/` 下创建插件目录
2. 添加 `.claude-plugin/plugin.json`
3. 按需添加 skills、commands、hooks、agents 等组件
