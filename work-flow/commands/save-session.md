---
description: Save current session state to a dated file in ~/.claude/session-data/ so work can be resumed in a future session with full context.
  将当前会话状态保存到 ~/.claude/session-data/ 中的带日期文件，以便在未来的会话中以完整上下文恢复工作。
---

# Save Session Command
# 保存会话命令

Capture everything that happened in this session — what was built, what worked, what failed, what's left — and write it to a dated file so the next session can pick up exactly where this one left off.
记录本次会话发生的一切——构建了什么、什么有效、什么失败、还剩什么——并写入带日期的文件，以便下次会话能从本次会话结束的确切位置继续。

## When to Use
## 何时使用

- End of a work session before closing Claude Code
- 关闭 Claude Code 前的工作会话结束时
- Before hitting context limits (run this first, then start a fresh session)
- 在达到上下文限制之前（先运行此命令，然后开始新会话）
- After solving a complex problem you want to remember
- 解决完一个想要记住的复杂问题后
- Any time you need to hand off context to a future session
- 任何时候需要将上下文传递给未来会话时

## Process
## 流程

### Step 1: Gather context
### 步骤 1：收集上下文

Before writing the file, collect:
在写入文件之前，收集：

- Read all files modified during this session (use git diff or recall from conversation)
- 读取本次会话中修改的所有文件（使用 git diff 或从对话中回顾）
- Review what was discussed, attempted, and decided
- 回顾讨论了什么、尝试了什么、决定了什么
- Note any errors encountered and how they were resolved (or not)
- 记录遇到的错误及其解决方式（或未解决）
- Check current test/build status if relevant
- 如有必要，检查当前测试/构建状态

### Step 2: Create the sessions folder if it doesn't exist
### 步骤 2：若会话文件夹不存在则创建

Create the canonical sessions folder in the user's Claude home directory:
在用户的 Claude 主目录中创建规范的会话文件夹：

```bash
mkdir -p ~/.claude/session-data
```

### Step 3: Write the session file
### 步骤 3：写入会话文件

Create `~/.claude/session-data/YYYY-MM-DD-<short-id>-session.tmp`, using today's actual date and a short-id that satisfies the rules enforced by `SESSION_FILENAME_REGEX` in `session-manager.js`:
创建 `~/.claude/session-data/YYYY-MM-DD-<short-id>-session.tmp`，使用今天的实际日期和满足 `session-manager.js` 中 `SESSION_FILENAME_REGEX` 规则的短 ID：

- Compatibility characters: letters `a-z` / `A-Z`, digits `0-9`, hyphens `-`, underscores `_`
- 兼容字符：字母 `a-z` / `A-Z`，数字 `0-9`，连字符 `-`，下划线 `_`
- Compatibility minimum length: 1 character
- 兼容最小长度：1 个字符
- Recommended style for new files: lowercase letters, digits, and hyphens with 8+ characters to avoid collisions
- 新文件推荐风格：小写字母、数字和连字符，8+ 个字符以避免冲突

Valid examples: `abc123de`, `a1b2c3d4`, `frontend-worktree-1`, `ChezMoi_2`
有效示例：`abc123de`、`a1b2c3d4`、`frontend-worktree-1`、`ChezMoi_2`
Avoid for new files: `A`, `test_id1`, `ABC123de`
新文件应避免：`A`、`test_id1`、`ABC123de`

Full valid filename example: `2024-01-15-abc123de-session.tmp`
完整有效文件名示例：`2024-01-15-abc123de-session.tmp`

The legacy filename `YYYY-MM-DD-session.tmp` is still valid, but new session files should prefer the short-id form to avoid same-day collisions.
旧版文件名 `YYYY-MM-DD-session.tmp` 仍然有效，但新会话文件应优先使用短 ID 形式以避免同日冲突。

### Step 4: Populate the file with all sections below
### 步骤 4：填充文件的所有部分

Write every section honestly. Do not skip sections — write "Nothing yet" or "N/A" if a section genuinely has no content. An incomplete file is worse than an honest empty section.
诚实地填写每个部分。不要跳过任何部分——若某部分确实没有内容，写"Nothing yet"或"N/A"。不完整的文件比诚实的空部分更糟糕。

### Step 5: Show the file to the user
### 步骤 5：向用户展示文件

After writing, display the full contents and ask:
写入后，展示完整内容并询问：

```
Session saved to [actual resolved path to the session file]

Does this look accurate? Anything to correct or add before we close?
```

Wait for confirmation. Make edits if requested.
等待确认。如有请求，进行修改。

---

## Session File Format
## 会话文件格式

````markdown
# Session: YYYY-MM-DD

**Started:** [approximate time if known]
**Last Updated:** [current time]
**Project:** [project name or path]
**Topic:** [one-line summary of what this session was about]

---

## What We Are Building

[1-3 paragraphs describing the feature, bug fix, or task. Include enough
context that someone with zero memory of this session can understand the goal.
Include: what it does, why it's needed, how it fits into the larger system.]

---

## What WORKED (with evidence)

[List only things that are confirmed working. For each item include WHY you
know it works — test passed, ran in browser, Postman returned 200, etc.
Without evidence, move it to "Not Tried Yet" instead.]

- **[thing that works]** — confirmed by: [specific evidence]
- **[thing that works]** — confirmed by: [specific evidence]

If nothing is confirmed working yet: "Nothing confirmed working yet — all approaches still in progress or untested."

---

## What Did NOT Work (and why)

[This is the most important section. List every approach tried that failed.
For each failure write the EXACT reason so the next session doesn't retry it.
Be specific: "threw X error because Y" is useful. "didn't work" is not.]

- **[approach tried]** — failed because: [exact reason / error message]
- **[approach tried]** — failed because: [exact reason / error message]

If nothing failed: "No failed approaches yet."

---

## What Has NOT Been Tried Yet

[Approaches that seem promising but haven't been attempted. Ideas from the
conversation. Alternative solutions worth exploring. Be specific enough that
the next session knows exactly what to try.]

- [approach / idea]
- [approach / idea]

If nothing is queued: "No specific untried approaches identified."

---

## Current State of Files

[Every file touched this session. Be precise about what state each file is in.]

| File              | Status         | Notes                      |
| ----------------- | -------------- | -------------------------- |
| `path/to/file.ts` | PASS: Complete    | [what it does]             |
| `path/to/file.ts` |  In Progress | [what's done, what's left] |
| `path/to/file.ts` | FAIL: Broken      | [what's wrong]             |
| `path/to/file.ts` |  Not Started | [planned but not touched]  |

If no files were touched: "No files modified this session."

---

## Decisions Made

[Architecture choices, tradeoffs accepted, approaches chosen and why.
These prevent the next session from relitigating settled decisions.]

- **[decision]** — reason: [why this was chosen over alternatives]

If no significant decisions: "No major decisions made this session."

---

## Blockers & Open Questions

[Anything unresolved that the next session needs to address or investigate.
Questions that came up but weren't answered. External dependencies waiting on.]

- [blocker / open question]

If none: "No active blockers."

---

## Exact Next Step

[If known: The single most important thing to do when resuming. Be precise
enough that resuming requires zero thinking about where to start.]

[If not known: "Next step not determined — review 'What Has NOT Been Tried Yet'
and 'Blockers' sections to decide on direction before starting."]

---

## Environment & Setup Notes

[Only fill this if relevant — commands needed to run the project, env vars
required, services that need to be running, etc. Skip if standard setup.]

[If none: omit this section entirely.]
````

---

## Example Output
## 示例输出

````markdown
# Session: 2024-01-15

**Started:** ~2pm
**Last Updated:** 5:30pm
**Project:** my-app
**Topic:** Building JWT authentication with httpOnly cookies

---

## What We Are Building

User authentication system for the Next.js app. Users register with email/password,
receive a JWT stored in an httpOnly cookie (not localStorage), and protected routes
check for a valid token via middleware. The goal is session persistence across browser
refreshes without exposing the token to JavaScript.

---

## What WORKED (with evidence)

- **`/api/auth/register` endpoint** — confirmed by: Postman POST returns 200 with user
  object, row visible in Supabase dashboard, bcrypt hash stored correctly
- **JWT generation in `lib/auth.ts`** — confirmed by: unit test passes
  (`npm test -- auth.test.ts`), decoded token at jwt.io shows correct payload
- **Password hashing** — confirmed by: `bcrypt.compare()` returns true in test

---

## What Did NOT Work (and why)

- **Next-Auth library** — failed because: conflicts with our custom Prisma adapter,
  threw "Cannot use adapter with credentials provider in this configuration" on every
  request. Not worth debugging — too opinionated for our setup.
- **Storing JWT in localStorage** — failed because: SSR renders happen before
  localStorage is available, caused React hydration mismatch error on every page load.
  This approach is fundamentally incompatible with Next.js SSR.

---

## What Has NOT Been Tried Yet

- Store JWT as httpOnly cookie in the login route response (most likely solution)
- Use `cookies()` from `next/headers` to read token in server components
- Write middleware.ts to protect routes by checking cookie existence

---

## Current State of Files

| File                             | Status         | Notes                                           |
| -------------------------------- | -------------- | ----------------------------------------------- |
| `app/api/auth/register/route.ts` | PASS: Complete    | Works, tested                                   |
| `app/api/auth/login/route.ts`    |  In Progress | Token generates but not setting cookie yet      |
| `lib/auth.ts`                    | PASS: Complete    | JWT helpers, all tested                         |
| `middleware.ts`                  |  Not Started | Route protection, needs cookie read logic first |
| `app/login/page.tsx`             |  Not Started | UI not started                                  |

---

## Decisions Made

- **httpOnly cookie over localStorage** — reason: prevents XSS token theft, works with SSR
- **Custom auth over Next-Auth** — reason: Next-Auth conflicts with our Prisma setup, not worth the fight

---

## Blockers & Open Questions

- Does `cookies().set()` work inside a Route Handler or only in Server Actions? Need to verify.

---

## Exact Next Step

In `app/api/auth/login/route.ts`, after generating the JWT, set it as an httpOnly
cookie using `cookies().set('token', jwt, { httpOnly: true, secure: true, sameSite: 'strict' })`.
Then test with Postman — the response should include a `Set-Cookie` header.
````

---

## Notes
## 注意事项

- Each session gets its own file — never append to a previous session's file
- 每次会话都有自己的文件——永远不要追加到之前会话的文件中
- The "What Did NOT Work" section is the most critical — future sessions will blindly retry failed approaches without it
- "未奏效的内容"部分是最关键的——没有它，未来的会话将盲目重试失败的方案
- If the user asks to save mid-session (not just at the end), save what's known so far and mark in-progress items clearly
- 若用户要求在会话中途保存（而非仅在结束时），保存目前已知的内容并清楚标记进行中的项目
- The file is meant to be read by Claude at the start of the next session via `/resume-session`
- 该文件旨在通过 `/resume-session` 在下次会话开始时由 Claude 读取
- Use the canonical global session store: `~/.claude/session-data/`
- 使用规范的全局会话存储：`~/.claude/session-data/`
- Prefer the short-id filename form (`YYYY-MM-DD-<short-id>-session.tmp`) for any new session file
- 所有新会话文件优先使用短 ID 文件名形式（`YYYY-MM-DD-<short-id>-session.tmp`）
