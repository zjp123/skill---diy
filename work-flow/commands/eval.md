---
description: Legacy slash-entry shim for the eval-harness skill. Prefer the skill directly.
  eval-harness 技能的遗留斜杠入口垫片。建议直接使用该技能。
---

# Eval Command (Legacy Shim)
# Eval 命令（遗留垫片）

Use this only if you still invoke `/eval`. The maintained workflow lives in `skills/eval-harness/SKILL.md`.
仅在仍使用 `/eval` 调用时使用此文件。最新的工作流位于 `skills/eval-harness/SKILL.md`。

## Canonical Surface
## 规范入口

- Prefer the `eval-harness` skill directly.
- 建议直接使用 `eval-harness` 技能。
- Keep this file only as a compatibility entry point.
- 此文件仅作为兼容性入口保留。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the `eval-harness` skill.
应用 `eval-harness` 技能。
- Support the same user intents as before: define, check, report, list, and cleanup.
- 支持与之前相同的用户意图：define、check、report、list 和 cleanup。
- Keep evals capability-first, regression-backed, and evidence-based.
- 保持评估以能力为先、以回归为支撑、以证据为基础。
- Use the skill as the canonical evaluator instead of maintaining a separate command-specific playbook.
- 使用技能作为规范评估器，而非维护单独的命令专属操作手册。
