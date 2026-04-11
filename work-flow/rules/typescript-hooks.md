---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# TypeScript/JavaScript Hooks
# TypeScript/JavaScript 钩子

> This file extends [common/hooks.md](../common/hooks.md) with TypeScript/JavaScript specific content.
> 本文件在 [common/hooks.md](../common/hooks.md) 的基础上扩展了 TypeScript/JavaScript 特定内容。

## PostToolUse Hooks
## PostToolUse 钩子

Configure in `~/.claude/settings.json`:
在 `~/.claude/settings.json` 中配置：

- **Prettier**: Auto-format JS/TS files after edit
- **Prettier**：编辑后自动格式化 JS/TS 文件
- **TypeScript check**: Run `tsc` after editing `.ts`/`.tsx` files
- **TypeScript 检查**：编辑 `.ts`/`.tsx` 文件后运行 `tsc`
- **console.log warning**: Warn about `console.log` in edited files
- **console.log 警告**：对编辑文件中的 `console.log` 发出警告

## Stop Hooks
## Stop 钩子

- **console.log audit**: Check all modified files for `console.log` before session ends
- **console.log 审计**：会话结束前检查所有修改的文件中是否存在 `console.log`
