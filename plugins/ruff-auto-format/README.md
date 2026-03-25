# ruff-auto-format

在修改 Python 文件后自动执行 ruff format 和 ruff check --fix。

## 工作原理

通过 `PostToolUse` Hook 监听 `Write` 和 `Edit` 工具调用，当被修改的文件是 `.py` 文件时，自动执行：

1. `ruff format <file>` — 格式化代码
2. `ruff check --fix <file>` — 检查并自动修复 lint 问题
3. `ruff check <file>` — 检查是否还有残留问题，有则反馈给 Claude 自动处理

完整的判断链：

```
文件后缀 != "py"        → 静默跳过
ruff 未安装              → 提示安装，Claude 自动处理
没有 pyproject.toml      → 提示创建，Claude 自动处理
ruff format + check --fix → 静默修复
还有残留问题             → 反馈给 Claude 自动修复
```

## 安装

```bash
/plugin marketplace add ./.
/plugin install ruff-auto-format@newstar
```

## 前置要求

- 全局安装 [ruff](https://docs.astral.sh/ruff/)（`pip install ruff`）
- 项目根目录需要 `pyproject.toml`
