---
description: Legacy slash-entry shim for the agent-sort skill. Prefer the skill directly.
  agent-sort 技能的遗留斜杠入口垫片。建议直接使用该技能。
---

# Agent Sort (Legacy Shim)
# Agent Sort（遗留垫片）

Use this only if you still invoke `/agent-sort`. The maintained workflow lives in `skills/agent-sort/SKILL.md`.
仅在仍使用 `/agent-sort` 调用时使用此文件。最新的工作流位于 `skills/agent-sort/SKILL.md`。

## Canonical Surface
## 规范入口

- Prefer the `agent-sort` skill directly.
- 建议直接使用 `agent-sort` 技能。
- Keep this file only as a compatibility entry point.
- 此文件仅作为兼容性入口保留。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the `agent-sort` skill.
应用 `agent-sort` 技能。
- Classify ECC surfaces with concrete repo evidence.
- 根据具体的仓库证据对 ECC 表面进行分类。
- Keep the result to DAILY vs LIBRARY.
- 将结果限定为 DAILY 或 LIBRARY。
- If an install change is needed afterward, hand off to `configure-ecc` instead of re-implementing install logic here.
- 如果之后需要安装变更，移交给 `configure-ecc`，而非在此重新实现安装逻辑。
