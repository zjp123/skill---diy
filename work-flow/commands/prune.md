---
name: prune
description: Delete pending instincts older than 30 days that were never promoted
  删除超过 30 天且从未被提升的待处理本能
command: true
---

# Prune Pending Instincts
# 清理待处理本能

Remove expired pending instincts that were auto-generated but never reviewed or promoted.
删除自动生成但从未被审查或提升的过期待处理本能。

## Implementation
## 实现

Run the instinct CLI using the plugin root path:
使用插件根路径运行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" prune
```

Or if `CLAUDE_PLUGIN_ROOT` is not set (manual installation):
或者如果未设置 `CLAUDE_PLUGIN_ROOT`（手动安装）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py prune
```

## Usage
## 使用方法

```
/prune                    # Delete instincts older than 30 days
/prune --max-age 60      # Custom age threshold (days)
/prune --dry-run         # Preview without deleting
```
