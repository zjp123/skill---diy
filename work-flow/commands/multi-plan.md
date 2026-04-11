# Plan - Multi-Model Collaborative Planning
# Plan - 多模型协作规划

Multi-model collaborative planning - Context retrieval + Dual-model analysis → Generate step-by-step implementation plan.
多模型协作规划——上下文检索 + 双模型分析 → 生成逐步实现计划。

$ARGUMENTS

---

## Core Protocols
## 核心协议

- **Language Protocol**: Use **English** when interacting with tools/models, communicate with user in their language
- **语言协议**：与工具/模型交互时使用**英文**，与用户沟通时使用其语言
- **Mandatory Parallel**: Codex/Gemini calls MUST use `run_in_background: true` (including single model calls, to avoid blocking main thread)
- **强制并行**：Codex/Gemini 调用必须使用 `run_in_background: true`（包括单模型调用，以避免阻塞主线程）
- **Code Sovereignty**: External models have **zero filesystem write access**, all modifications by Claude
- **代码主权**：外部模型**完全没有文件系统写入权限**，所有修改由 Claude 执行
- **Stop-Loss Mechanism**: Do not proceed to next phase until current phase output is validated
- **止损机制**：当前阶段输出未经验证，不得进入下一阶段
- **Planning Only**: This command allows reading context and writing to `.claude/plan/*` plan files, but **NEVER modify production code**
- **仅规划**：此命令允许读取上下文和写入 `.claude/plan/*` 计划文件，但**绝不修改生产代码**

---

## Multi-Model Call Specification
## 多模型调用规范

**Call Syntax** (parallel: use `run_in_background: true`):
**调用语法**（并行时使用 `run_in_background: true`）：

```
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement>
Context: <retrieved project context>
</TASK>
OUTPUT: Step-by-step implementation plan with pseudo-code. DO NOT modify any files.
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "Brief description"
})
```

**Model Parameter Notes**:
**模型参数说明**：
- `{{GEMINI_MODEL_FLAG}}`: When using `--backend gemini`, replace with `--gemini-model gemini-3-pro-preview` (note trailing space); use empty string for codex
- `{{GEMINI_MODEL_FLAG}}`：使用 `--backend gemini` 时，替换为 `--gemini-model gemini-3-pro-preview`（注意末尾空格）；使用 codex 时为空字符串

**Role Prompts**:
**角色提示词**：

| Phase | Codex | Gemini |
|-------|-------|--------|
| 阶段 | Codex | Gemini |
| Analysis | `~/.claude/.ccg/prompts/codex/analyzer.md` | `~/.claude/.ccg/prompts/gemini/analyzer.md` |
| 分析 | `~/.claude/.ccg/prompts/codex/analyzer.md` | `~/.claude/.ccg/prompts/gemini/analyzer.md` |
| Planning | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/architect.md` |
| 规划 | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/architect.md` |

**Session Reuse**: Each call returns `SESSION_ID: xxx` (typically output by wrapper), **MUST save** for subsequent `/ccg:execute` use.
**会话复用**：每次调用返回 `SESSION_ID: xxx`（通常由 wrapper 输出），**必须保存**供后续 `/ccg:execute` 使用。

**Wait for Background Tasks** (max timeout 600000ms = 10 minutes):
**等待后台任务**（最大超时 600000ms = 10 分钟）：

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**IMPORTANT**:
**重要**：
- Must specify `timeout: 600000`, otherwise default 30 seconds will cause premature timeout
- 必须指定 `timeout: 600000`，否则默认 30 秒将导致提前超时
- If still incomplete after 10 minutes, continue polling with `TaskOutput`, **NEVER kill the process**
- 若 10 分钟后仍未完成，继续使用 `TaskOutput` 轮询，**绝不终止进程**
- If waiting is skipped due to timeout, **MUST call `AskUserQuestion` to ask user whether to continue waiting or kill task**
- 若因超时跳过等待，**必须调用 `AskUserQuestion` 询问用户是继续等待还是终止任务**

---

## Execution Workflow
## 执行工作流

**Planning Task**: $ARGUMENTS
**规划任务**：$ARGUMENTS

### Phase 1: Full Context Retrieval
### 阶段 1：完整上下文检索

`[Mode: Research]`

#### 1.1 Prompt Enhancement (MUST execute first)
#### 1.1 提示词增强（必须首先执行）

**If ace-tool MCP is available**, call `mcp__ace-tool__enhance_prompt` tool:
**若 ace-tool MCP 可用**，调用 `mcp__ace-tool__enhance_prompt` 工具：

```
mcp__ace-tool__enhance_prompt({
  prompt: "$ARGUMENTS",
  conversation_history: "<last 5-10 conversation turns>",
  project_root_path: "$PWD"
})
```

Wait for enhanced prompt, **replace original $ARGUMENTS with enhanced result** for all subsequent phases.
等待增强后的提示词，**用增强结果替换原始 $ARGUMENTS** 供所有后续阶段使用。

**If ace-tool MCP is NOT available**: Skip this step and use the original `$ARGUMENTS` as-is for all subsequent phases.
**若 ace-tool MCP 不可用**：跳过此步骤，在所有后续阶段直接使用原始 `$ARGUMENTS`。

#### 1.2 Context Retrieval
#### 1.2 上下文检索

**If ace-tool MCP is available**, call `mcp__ace-tool__search_context` tool:
**若 ace-tool MCP 可用**，调用 `mcp__ace-tool__search_context` 工具：

```
mcp__ace-tool__search_context({
  query: "<semantic query based on enhanced requirement>",
  project_root_path: "$PWD"
})
```

- Build semantic query using natural language (Where/What/How)
- 使用自然语言构建语义查询（Where/What/How）
- **NEVER answer based on assumptions**
- **绝不基于假设作答**

**If ace-tool MCP is NOT available**, use Claude Code built-in tools as fallback:
**若 ace-tool MCP 不可用**，使用 Claude Code 内置工具作为回退：
1. **Glob**: Find relevant files by pattern (e.g., `Glob("**/*.ts")`, `Glob("src/**/*.py")`)
1. **Glob**：按模式查找相关文件（如 `Glob("**/*.ts")`、`Glob("src/**/*.py")`）
2. **Grep**: Search for key symbols, function names, class definitions (e.g., `Grep("className|functionName")`)
2. **Grep**：搜索关键符号、函数名、类定义（如 `Grep("className|functionName")`）
3. **Read**: Read the discovered files to gather complete context
3. **Read**：读取发现的文件以收集完整上下文
4. **Task (Explore agent)**: For deeper exploration, use `Task` with `subagent_type: "Explore"` to search across the codebase
4. **Task（Explore 代理）**：更深度探索时，使用 `Task` 并设置 `subagent_type: "Explore"` 跨代码库搜索

#### 1.3 Completeness Check
#### 1.3 完整性检查

- Must obtain **complete definitions and signatures** for relevant classes, functions, variables
- 必须获取相关类、函数、变量的**完整定义和签名**
- If context insufficient, trigger **recursive retrieval**
- 若上下文不足，触发**递归检索**
- Prioritize output: entry file + line number + key symbol name; add minimal code snippets only when necessary to resolve ambiguity
- 优先输出：入口文件 + 行号 + 关键符号名；仅在必要时添加最少量代码片段以消除歧义

#### 1.4 Requirement Alignment
#### 1.4 需求对齐

- If requirements still have ambiguity, **MUST** output guiding questions for user
- 若需求仍存在歧义，**必须**向用户输出引导性问题
- Until requirement boundaries are clear (no omissions, no redundancy)
- 直到需求边界清晰（无遗漏、无冗余）

### Phase 2: Multi-Model Collaborative Analysis
### 阶段 2：多模型协作分析

`[Mode: Analysis]`

#### 2.1 Distribute Inputs
#### 2.1 分发输入

**Parallel call** Codex and Gemini (`run_in_background: true`):
**并行调用** Codex 和 Gemini（`run_in_background: true`）：

Distribute **original requirement** (without preset opinions) to both models:
将**原始需求**（不预设观点）分发给两个模型：

1. **Codex Backend Analysis**:
1. **Codex 后端分析**：
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/analyzer.md`
   - Focus: Technical feasibility, architecture impact, performance considerations, potential risks
   - 关注：技术可行性、架构影响、性能考量、潜在风险
   - OUTPUT: Multi-perspective solutions + pros/cons analysis
   - 输出：多视角方案 + 优缺点分析

2. **Gemini Frontend Analysis**:
2. **Gemini 前端分析**：
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/analyzer.md`
   - Focus: UI/UX impact, user experience, visual design
   - 关注：UI/UX 影响、用户体验、视觉设计
   - OUTPUT: Multi-perspective solutions + pros/cons analysis
   - 输出：多视角方案 + 优缺点分析

Wait for both models' complete results with `TaskOutput`. **Save SESSION_ID** (`CODEX_SESSION` and `GEMINI_SESSION`).
使用 `TaskOutput` 等待两个模型的完整结果。**保存 SESSION_ID**（`CODEX_SESSION` 和 `GEMINI_SESSION`）。

#### 2.2 Cross-Validation
#### 2.2 交叉验证

Integrate perspectives and iterate for optimization:
整合视角并迭代优化：

1. **Identify consensus** (strong signal)
1. **识别共识**（强信号）
2. **Identify divergence** (needs weighing)
2. **识别分歧**（需要权衡）
3. **Complementary strengths**: Backend logic follows Codex, Frontend design follows Gemini
3. **优势互补**：后端逻辑遵循 Codex，前端设计遵循 Gemini
4. **Logical reasoning**: Eliminate logical gaps in solutions
4. **逻辑推理**：消除方案中的逻辑漏洞

#### 2.3 (Optional but Recommended) Dual-Model Plan Draft
#### 2.3 （可选但推荐）双模型计划草案

To reduce risk of omissions in Claude's synthesized plan, can parallel have both models output "plan drafts" (still **NOT allowed** to modify files):
为降低 Claude 综合计划中遗漏的风险，可并行让两个模型输出"计划草案"（仍**不允许**修改文件）：

1. **Codex Plan Draft** (Backend authority):
1. **Codex 计划草案**（后端权威）：
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/architect.md`
   - OUTPUT: Step-by-step plan + pseudo-code (focus: data flow/edge cases/error handling/test strategy)
   - 输出：逐步计划 + 伪代码（关注：数据流/边界情况/错误处理/测试策略）

2. **Gemini Plan Draft** (Frontend authority):
2. **Gemini 计划草案**（前端权威）：
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/architect.md`
   - OUTPUT: Step-by-step plan + pseudo-code (focus: information architecture/interaction/accessibility/visual consistency)
   - 输出：逐步计划 + 伪代码（关注：信息架构/交互/无障碍访问/视觉一致性）

Wait for both models' complete results with `TaskOutput`, record key differences in their suggestions.
使用 `TaskOutput` 等待两个模型的完整结果，记录建议中的关键差异。

#### 2.4 Generate Implementation Plan (Claude Final Version)
#### 2.4 生成实现计划（Claude 终版）

Synthesize both analyses, generate **Step-by-step Implementation Plan**:
综合两份分析，生成**逐步实现计划**：

```markdown
## Implementation Plan: <Task Name>

### Task Type
- [ ] Frontend (→ Gemini)
- [ ] Backend (→ Codex)
- [ ] Fullstack (→ Parallel)

### Technical Solution
<Optimal solution synthesized from Codex + Gemini analysis>

### Implementation Steps
1. <Step 1> - Expected deliverable
2. <Step 2> - Expected deliverable
...

### Key Files
| File | Operation | Description |
|------|-----------|-------------|
| path/to/file.ts:L10-L50 | Modify | Description |

### Risks and Mitigation
| Risk | Mitigation |
|------|------------|

### SESSION_ID (for /ccg:execute use)
- CODEX_SESSION: <session_id>
- GEMINI_SESSION: <session_id>
```

### Phase 2 End: Plan Delivery (Not Execution)
### 阶段 2 结束：计划交付（非执行）

**`/ccg:plan` responsibilities end here, MUST execute the following actions**:
**`/ccg:plan` 职责到此结束，必须执行以下操作**：

1. Present complete implementation plan to user (including pseudo-code)
1. 向用户展示完整实现计划（包含伪代码）
2. Save plan to `.claude/plan/<feature-name>.md` (extract feature name from requirement, e.g., `user-auth`, `payment-module`)
2. 将计划保存到 `.claude/plan/<feature-name>.md`（从需求中提取功能名称，如 `user-auth`、`payment-module`）
3. Output prompt in **bold text** (MUST use actual saved file path):
3. 以**粗体文本**输出提示（必须使用实际保存的文件路径）：

---
**Plan generated and saved to `.claude/plan/actual-feature-name.md`**

**Please review the plan above. You can:**
- **Modify plan**: Tell me what needs adjustment, I'll update the plan
- **Execute plan**: Copy the following command to a new session

```
/ccg:execute .claude/plan/actual-feature-name.md
```
---

**NOTE**: The `actual-feature-name.md` above MUST be replaced with the actual saved filename!
**注意**：上方的 `actual-feature-name.md` 必须替换为实际保存的文件名！

4. **Immediately terminate current response** (Stop here. No more tool calls.)
4. **立即终止当前响应**（到此为止。不再进行任何工具调用。）

**ABSOLUTELY FORBIDDEN**:
**绝对禁止**：
- Ask user "Y/N" then auto-execute (execution is `/ccg:execute`'s responsibility)
- 询问用户"Y/N"然后自动执行（执行是 `/ccg:execute` 的职责）
- Any write operations to production code
- 任何对生产代码的写入操作
- Automatically call `/ccg:execute` or any implementation actions
- 自动调用 `/ccg:execute` 或任何实现操作
- Continue triggering model calls when user hasn't explicitly requested modifications
- 在用户未明确请求修改时继续触发模型调用

---

## Plan Saving
## 计划保存

After planning completes, save plan to:
规划完成后，将计划保存到：

- **First planning**: `.claude/plan/<feature-name>.md`
- **首次规划**：`.claude/plan/<feature-name>.md`
- **Iteration versions**: `.claude/plan/<feature-name>-v2.md`, `.claude/plan/<feature-name>-v3.md`...
- **迭代版本**：`.claude/plan/<feature-name>-v2.md`、`.claude/plan/<feature-name>-v3.md`……

Plan file write should complete before presenting plan to user.
计划文件写入应在向用户展示计划之前完成。

---

## Plan Modification Flow
## 计划修改流程

If user requests plan modifications:
若用户请求修改计划：

1. Adjust plan content based on user feedback
1. 根据用户反馈调整计划内容
2. Update `.claude/plan/<feature-name>.md` file
2. 更新 `.claude/plan/<feature-name>.md` 文件
3. Re-present modified plan
3. 重新展示修改后的计划
4. Prompt user to review or execute again
4. 提示用户再次审查或执行

---

## Next Steps
## 后续步骤

After user approves, **manually** execute:
用户批准后，**手动**执行：

```bash
/ccg:execute .claude/plan/<feature-name>.md
```

---

## Key Rules
## 关键规则

1. **Plan only, no implementation** – This command does not execute any code changes
1. **仅规划，不实现** — 此命令不执行任何代码更改
2. **No Y/N prompts** – Only present plan, let user decide next steps
2. **不提示 Y/N** — 仅展示计划，由用户决定后续步骤
3. **Trust Rules** – Backend follows Codex, Frontend follows Gemini
3. **信任规则** — 后端遵循 Codex，前端遵循 Gemini
4. External models have **zero filesystem write access**
4. 外部模型**完全没有文件系统写入权限**
5. **SESSION_ID Handoff** – Plan must include `CODEX_SESSION` / `GEMINI_SESSION` at end (for `/ccg:execute resume <SESSION_ID>` use)
5. **SESSION_ID 移交** — 计划末尾必须包含 `CODEX_SESSION` / `GEMINI_SESSION`（供 `/ccg:execute resume <SESSION_ID>` 使用）
