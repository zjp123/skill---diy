---
description: Legacy slash-entry shim for the context-budget skill. Prefer the skill directly.
  context-budget 技能的遗留斜杠入口垫片。建议直接使用该技能。
---

# Context Budget Optimizer (Legacy Shim)
# Context Budget 优化器（遗留垫片）

Use this only if you still invoke `/context-budget`. The maintained workflow lives in `skills/context-budget/SKILL.md`.
仅在仍使用 `/context-budget` 调用时使用此文件。最新的工作流位于 `skills/context-budget/SKILL.md`。

## Canonical Surface
## 规范入口

- Prefer the `context-budget` skill directly.
- 建议直接使用 `context-budget` 技能。
- Keep this file only as a compatibility entry point.
- 此文件仅作为兼容性入口保留。

## Arguments
## 参数

$ARGUMENTS

## Delegation
## 委托

Apply the `context-budget` skill.
应用 `context-budget` 技能。
- Pass through `--verbose` if the user supplied it.
- 如果用户提供了 `--verbose`，则透传该参数。
- Assume a 200K context window unless the user specified otherwise.
- 除非用户另行指定，否则假设上下文窗口为 200K。
- Return the skill's inventory, issue detection, and prioritized savings report without re-implementing the scan here.
- 返回技能的清单、问题检测和优先级节省报告，无需在此重新实现扫描逻辑。
