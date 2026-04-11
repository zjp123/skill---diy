---
description: Legacy slash-entry shim for the claude-devfleet skill. Prefer the skill directly.
  claude-devfleet 技能的遗留斜杠入口垫片。建议直接使用该技能。
---

# DevFleet (Legacy Shim)
# DevFleet（遗留垫片）

Use this only if you still call `/devfleet`. The maintained workflow lives in `skills/claude-devfleet/SKILL.md`.
仅在仍使用 `/devfleet` 调用时使用此文件。最新的工作流位于 `skills/claude-devfleet/SKILL.md`。

## Canonical Surface
## 规范入口

- Prefer the `claude-devfleet` skill directly.
- 建议直接使用 `claude-devfleet` 技能。
- Keep this file only as a compatibility entry point while command-first usage is retired.
- 在逐步淘汰命令优先用法期间，此文件仅作为兼容性入口保留。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the `claude-devfleet` skill.
应用 `claude-devfleet` 技能。
- Plan from the user's description, show the DAG, and get approval before dispatch unless the user already said to proceed.
- 根据用户描述制定计划，展示 DAG，并在调度前获得批准——除非用户已表示继续。
- Prefer polling status over blocking waits for long missions.
- 对于长时间任务，优先轮询状态而非阻塞等待。
- Report mission IDs, files changed, failures, and next steps from structured mission reports.
- 从结构化任务报告中汇报任务 ID、变更文件、失败情况和后续步骤。
