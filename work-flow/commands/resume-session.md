---
description: Load the most recent session file from ~/.claude/session-data/ and resume work with full context from where the last session ended.
  从 ~/.claude/session-data/ 加载最近的会话文件，并从上次会话结束的地方以完整上下文恢复工作。
---

# Resume Session Command
# 恢复会话命令

Load the last saved session state and orient fully before doing any work.
加载上次保存的会话状态，在执行任何工作前完全定位。
This command is the counterpart to `/save-session`.
此命令是 `/save-session` 的对应命令。

## When to Use
## 何时使用

- Starting a new session to continue work from a previous day
- 开始新会话以继续前一天的工作
- After starting a fresh session due to context limits
- 因上下文限制开始新会话后
- When handing off a session file from another source (just provide the file path)
- 从其他来源传递会话文件时（仅提供文件路径）
- Any time you have a session file and want Claude to fully absorb it before proceeding
- 任何时候拥有会话文件并希望 Claude 在继续之前完全吸收它

## Usage
## 使用方法

```
/resume-session                                                      # loads most recent file in ~/.claude/session-data/
/resume-session 2024-01-15                                           # loads most recent session for that date
/resume-session ~/.claude/session-data/2024-01-15-abc123de-session.tmp  # loads a current short-id session file
/resume-session ~/.claude/sessions/2024-01-15-session.tmp               # loads a specific legacy-format file
```

## Process
## 流程

### Step 1: Find the session file
### 步骤 1：查找会话文件

If no argument provided:
若未提供参数：

1. Check `~/.claude/session-data/`
1. 检查 `~/.claude/session-data/`
2. Pick the most recently modified `*-session.tmp` file
2. 选取最近修改的 `*-session.tmp` 文件
3. If the folder does not exist or has no matching files, tell the user:
3. 若文件夹不存在或无匹配文件，告知用户：
   ```
   No session files found in ~/.claude/session-data/
   Run /save-session at the end of a session to create one.
   ```
   Then stop.
   然后停止。

If an argument is provided:
若提供了参数：

- If it looks like a date (`YYYY-MM-DD`), search `~/.claude/session-data/` first, then the legacy
- 若看起来像日期（`YYYY-MM-DD`），先搜索 `~/.claude/session-data/`，然后搜索旧版
  `~/.claude/sessions/`, for files matching `YYYY-MM-DD-session.tmp` (legacy format) or
  `~/.claude/sessions/`，查找匹配 `YYYY-MM-DD-session.tmp`（旧版格式）或
  `YYYY-MM-DD-<shortid>-session.tmp` (current format)
  `YYYY-MM-DD-<shortid>-session.tmp`（当前格式）的文件
  and load the most recently modified variant for that date
  并加载该日期最近修改的变体
- If it looks like a file path, read that file directly
- 若看起来像文件路径，直接读取该文件
- If not found, report clearly and stop
- 若未找到，清晰报告并停止

### Step 2: Read the entire session file
### 步骤 2：读取完整会话文件

Read the complete file. Do not summarize yet.
读取完整文件，暂不总结。

### Step 3: Confirm understanding
### 步骤 3：确认理解

Respond with a structured briefing in this exact format:
以以下精确格式响应结构化简报：

```
SESSION LOADED: [actual resolved path to the file]
════════════════════════════════════════════════

PROJECT: [project name / topic from file]

WHAT WE'RE BUILDING:
[2-3 sentence summary in your own words]

CURRENT STATE:
PASS: Working: [count] items confirmed
 In Progress: [list files that are in progress]
 Not Started: [list planned but untouched]

WHAT NOT TO RETRY:
[list every failed approach with its reason — this is critical]

OPEN QUESTIONS / BLOCKERS:
[list any blockers or unanswered questions]

NEXT STEP:
[exact next step if defined in the file]
[if not defined: "No next step defined — recommend reviewing 'What Has NOT Been Tried Yet' together before starting"]

════════════════════════════════════════════════
Ready to continue. What would you like to do?
```

### Step 4: Wait for the user
### 步骤 4：等待用户

Do NOT start working automatically. Do NOT touch any files. Wait for the user to say what to do next.
不要自动开始工作，不要触碰任何文件，等待用户说明下一步操作。

If the next step is clearly defined in the session file and the user says "continue" or "yes" or similar — proceed with that exact next step.
若会话文件中明确定义了下一步，且用户说"continue"或"yes"或类似内容——按照该确切步骤执行。

If no next step is defined — ask the user where to start, and optionally suggest an approach from the "What Has NOT Been Tried Yet" section.
若未定义下一步——询问用户从哪里开始，并可选地从"尚未尝试的内容"部分建议一种方案。

---

## Edge Cases
## 边界情况

**Multiple sessions for the same date** (`2024-01-15-session.tmp`, `2024-01-15-abc123de-session.tmp`):
**同一日期的多个会话**（`2024-01-15-session.tmp`、`2024-01-15-abc123de-session.tmp`）：
Load the most recently modified matching file for that date, regardless of whether it uses the legacy no-id format or the current short-id format.
加载该日期最近修改的匹配文件，无论使用旧版无 ID 格式还是当前短 ID 格式。

**Session file references files that no longer exist:**
**会话文件引用了不再存在的文件：**
Note this during the briefing — "WARNING: `path/to/file.ts` referenced in session but not found on disk."
在简报中注明——"WARNING: `path/to/file.ts` referenced in session but not found on disk."

**Session file is from more than 7 days ago:**
**会话文件来自 7 天前以上：**
Note the gap — "WARNING: This session is from N days ago (threshold: 7 days). Things may have changed." — then proceed normally.
注明差距——"WARNING: This session is from N days ago (threshold: 7 days). Things may have changed."——然后正常继续。

**User provides a file path directly (e.g., forwarded from a teammate):**
**用户直接提供文件路径（如从队友转发）：**
Read it and follow the same briefing process — the format is the same regardless of source.
读取并遵循相同的简报流程——无论来源如何，格式相同。

**Session file is empty or malformed:**
**会话文件为空或格式不正确：**
Report: "Session file found but appears empty or unreadable. You may need to create a new one with /save-session."
报告："Session file found but appears empty or unreadable. You may need to create a new one with /save-session."

---

## Example Output
## 示例输出

```
SESSION LOADED: /Users/you/.claude/session-data/2024-01-15-abc123de-session.tmp
════════════════════════════════════════════════

PROJECT: my-app — JWT Authentication

WHAT WE'RE BUILDING:
User authentication with JWT tokens stored in httpOnly cookies.
Register and login endpoints are partially done. Route protection
via middleware hasn't been started yet.

CURRENT STATE:
PASS: Working: 3 items (register endpoint, JWT generation, password hashing)
 In Progress: app/api/auth/login/route.ts (token works, cookie not set yet)
 Not Started: middleware.ts, app/login/page.tsx

WHAT NOT TO RETRY:
FAIL: Next-Auth — conflicts with custom Prisma adapter, threw adapter error on every request
FAIL: localStorage for JWT — causes SSR hydration mismatch, incompatible with Next.js

OPEN QUESTIONS / BLOCKERS:
- Does cookies().set() work inside a Route Handler or only Server Actions?

NEXT STEP:
In app/api/auth/login/route.ts — set the JWT as an httpOnly cookie using
cookies().set('token', jwt, { httpOnly: true, secure: true, sameSite: 'strict' })
then test with Postman for a Set-Cookie header in the response.

════════════════════════════════════════════════
Ready to continue. What would you like to do?
```

---

## Notes
## 注意事项

- Never modify the session file when loading it — it's a read-only historical record
- 加载会话文件时永远不要修改它——它是只读的历史记录
- The briefing format is fixed — do not skip sections even if they are empty
- 简报格式是固定的——即使某些部分为空也不要跳过
- "What Not To Retry" must always be shown, even if it just says "None" — it's too important to miss
- "不要重试的内容"必须始终显示，即使只写"无"——此项太重要不能遗漏
- After resuming, the user may want to run `/save-session` again at the end of the new session to create a new dated file
- 恢复后，用户可能希望在新会话结束时再次运行 `/save-session` 以创建新的带日期文件
