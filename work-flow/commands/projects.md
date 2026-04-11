---
name: projects
description: List known projects and their instinct statistics
  列出已知项目及其本能统计信息
command: true
---

# Projects Command
# Projects 命令

List project registry entries and per-project instinct/observation counts for continuous-learning-v2.
列出 continuous-learning-v2 的项目注册表条目及每个项目的本能/观察计数。

## Implementation
## 实现

Run the instinct CLI using the plugin root path:
使用插件根路径运行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" projects
```

Or if `CLAUDE_PLUGIN_ROOT` is not set (manual installation):
或者如果未设置 `CLAUDE_PLUGIN_ROOT`（手动安装）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py projects
```

## Usage
## 使用方法

```bash
/projects
```

## What to Do
## 操作步骤

1. Read `~/.claude/homunculus/projects.json`
1. 读取 `~/.claude/homunculus/projects.json`
2. For each project, display:
2. 对每个项目，显示：
   - Project name, id, root, remote
   - 项目名称、ID、根路径、远程地址
   - Personal and inherited instinct counts
   - 个人和继承的本能计数
   - Observation event count
   - 观察事件计数
   - Last seen timestamp
   - 最后访问时间戳
3. Also display global instinct totals
3. 同时显示全局本能总计
