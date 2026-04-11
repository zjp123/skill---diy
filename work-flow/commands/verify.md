---
description: Legacy slash-entry shim for the verification-loop skill. Prefer the skill directly.
  verification-loop 技能的旧版斜杠入口兼容层，建议直接使用该技能。
---

# Verification Command (Legacy Shim)
# 验证命令（旧版兼容层）

Use this only if you still invoke `/verify`. The maintained workflow lives in `skills/verification-loop/SKILL.md`.
仅当你仍然调用 `/verify` 时使用此文件。维护中的工作流位于 `skills/verification-loop/SKILL.md`。

## Canonical Surface
## 规范入口

- Prefer the `verification-loop` skill directly.
- 建议直接使用 `verification-loop` 技能。
- Keep this file only as a compatibility entry point.
- 仅将此文件保留为兼容性入口点。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the `verification-loop` skill.
应用 `verification-loop` 技能。
- Choose the right verification depth for the user's requested mode.
- 根据用户请求的模式选择合适的验证深度。
- Run build, types, lint, tests, security/log checks, and diff review in the right order for the current repo.
- 按当前仓库的正确顺序运行构建、类型检查、lint、测试、安全/日志检查和差异审查。
- Report only the verdicts and blockers instead of maintaining a second verification checklist here.
- 只报告结论和阻塞项，而非在此维护第二份验证清单。
