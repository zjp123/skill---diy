---
description: Legacy slash-entry shim for the documentation-lookup skill. Prefer the skill directly.
  documentation-lookup 技能的遗留斜杠入口垫片。建议直接使用该技能。
---

# Docs Command (Legacy Shim)
# Docs 命令（遗留垫片）

Use this only if you still reach for `/docs`. The maintained workflow lives in `skills/documentation-lookup/SKILL.md`.
仅在仍使用 `/docs` 调用时使用此文件。最新的工作流位于 `skills/documentation-lookup/SKILL.md`。

## Canonical Surface
## 规范入口

- Prefer the `documentation-lookup` skill directly.
- 建议直接使用 `documentation-lookup` 技能。
- Keep this file only as a compatibility entry point.
- 此文件仅作为兼容性入口保留。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the `documentation-lookup` skill.
应用 `documentation-lookup` 技能。
- If the library or the question is missing, ask for the missing part.
- 如果缺少库名或问题内容，请询问缺失部分。
- Use live documentation through Context7 instead of training data.
- 通过 Context7 使用实时文档，而非训练数据。
- Return only the current answer and the minimum code/example surface needed.
- 仅返回当前答案及所需的最少代码/示例内容。
