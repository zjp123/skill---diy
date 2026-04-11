---
description: Legacy slash-entry shim for the nanoclaw-repl skill. Prefer the skill directly.
  nanoclaw-repl 技能的遗留斜杠入口垫片。建议直接使用该技能。
---

# Claw Command (Legacy Shim)
# Claw 命令（遗留垫片）

Use this only if you still reach for `/claw` from muscle memory. The maintained implementation lives in `skills/nanoclaw-repl/SKILL.md`.
仅在仍因肌肉记忆使用 `/claw` 时使用此文件。最新的实现位于 `skills/nanoclaw-repl/SKILL.md`。

## Canonical Surface
## 规范入口

- Prefer the `nanoclaw-repl` skill directly.
- 建议直接使用 `nanoclaw-repl` 技能。
- Keep this file only as a compatibility entry point while command-first usage is retired.
- 在逐步淘汰命令优先用法期间，此文件仅作为兼容性入口保留。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the `nanoclaw-repl` skill and keep the response focused on operating or extending `scripts/claw.js`.
应用 `nanoclaw-repl` 技能，并将响应聚焦于操作或扩展 `scripts/claw.js`。
- If the user wants to run it, use `node scripts/claw.js` or `npm run claw`.
- 如果用户想运行它，使用 `node scripts/claw.js` 或 `npm run claw`。
- If the user wants to extend it, preserve the zero-dependency and markdown-backed session model.
- 如果用户想扩展它，保留零依赖和基于 Markdown 的会话模型。
- If the request is really about long-running orchestration rather than NanoClaw itself, redirect to `dmux-workflows` or `autonomous-agent-harness`.
- 如果请求实际上是关于长时间运行的编排而非 NanoClaw 本身，重定向到 `dmux-workflows` 或 `autonomous-agent-harness`。
