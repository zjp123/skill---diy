---
name: promote
description: Promote project-scoped instincts to global scope
  将项目范围的本能提升为全局范围
command: true
---

# Promote Command
# Promote 命令

Promote instincts from project scope to global scope in continuous-learning-v2.
在 continuous-learning-v2 中将本能从项目范围提升为全局范围。

## Implementation
## 实现

Run the instinct CLI using the plugin root path:
使用插件根路径运行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" promote [instinct-id] [--force] [--dry-run]
```

Or if `CLAUDE_PLUGIN_ROOT` is not set (manual installation):
或者如果未设置 `CLAUDE_PLUGIN_ROOT`（手动安装）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py promote [instinct-id] [--force] [--dry-run]
```

## Usage
## 使用方法

```bash
/promote                      # Auto-detect promotion candidates
/promote                      # 自动检测提升候选项
/promote --dry-run            # Preview auto-promotion candidates
/promote --dry-run            # 预览自动提升候选项
/promote --force              # Promote all qualified candidates without prompt
/promote --force              # 无需提示，直接提升所有合格候选项
/promote grep-before-edit     # Promote one specific instinct from current project
/promote grep-before-edit     # 从当前项目提升某个特定本能
```

## What to Do
## 操作步骤

1. Detect current project
1. 检测当前项目
2. If `instinct-id` is provided, promote only that instinct (if present in current project)
2. 若提供了 `instinct-id`，仅提升该本能（如存在于当前项目中）
3. Otherwise, find cross-project candidates that:
3. 否则，查找满足以下条件的跨项目候选项：
   - Appear in at least 2 projects
   - 出现在至少 2 个项目中
   - Meet confidence threshold
   - 达到置信度阈值
4. Write promoted instincts to `~/.claude/homunculus/instincts/personal/` with `scope: global`
4. 将提升后的本能写入 `~/.claude/homunculus/instincts/personal/`，范围设为 `scope: global`
