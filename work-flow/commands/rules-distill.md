---
description: Legacy slash-entry shim for the rules-distill skill. Prefer the skill directly.
  rules-distill 技能的旧版斜杠入口兼容层，建议直接使用该技能。
---

# Rules Distill (Legacy Shim)
# 规则提炼（旧版兼容层）

Use this only if you still invoke `/rules-distill`. The maintained workflow lives in `skills/rules-distill/SKILL.md`.
仅当你仍然调用 `/rules-distill` 时使用此文件。维护中的工作流位于 `skills/rules-distill/SKILL.md`。

## Canonical Surface
## 规范入口

- Prefer the `rules-distill` skill directly.
- 建议直接使用 `rules-distill` 技能。
- Keep this file only as a compatibility entry point.
- 仅将此文件保留为兼容性入口点。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the `rules-distill` skill and follow its inventory, cross-read, and verdict workflow instead of duplicating that logic here.
应用 `rules-distill` 技能，遵循其清单、交叉阅读和结论工作流，而非在此处重复该逻辑。
