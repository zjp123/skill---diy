---
description: Legacy slash-entry shim for the prompt-optimizer skill. Prefer the skill directly.
  prompt-optimizer 技能的旧版兼容入口。建议直接使用该技能。
---

# Prompt Optimize (Legacy Shim)
# Prompt Optimize（旧版兼容入口）

Use this only if you still invoke `/prompt-optimize`. The maintained workflow lives in `skills/prompt-optimizer/SKILL.md`.
仅在仍使用 `/prompt-optimize` 调用时使用此文件。维护中的工作流位于 `skills/prompt-optimizer/SKILL.md`。

## Canonical Surface
## 标准入口

- Prefer the `prompt-optimizer` skill directly.
- 直接优先使用 `prompt-optimizer` 技能。
- Keep this file only as a compatibility entry point.
- 此文件仅作为兼容性入口保留。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the `prompt-optimizer` skill.
应用 `prompt-optimizer` 技能。
- Keep it advisory-only: optimize the prompt, do not execute the task.
- 保持仅建议模式：优化提示词，不执行任务。
- Return the recommended ECC components plus a ready-to-run prompt.
- 返回推荐的 ECC 组件以及一个可直接运行的提示词。
- If the user actually wants direct execution, say so and tell them to make a normal task request instead of staying inside the shim.
- 若用户确实需要直接执行，请说明情况并告知其发起普通任务请求，而非继续停留在兼容入口中。
