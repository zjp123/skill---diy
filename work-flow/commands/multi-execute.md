# Execute - Multi-Model Collaborative Execution
# Execute - 多模型协作执行

Multi-model collaborative execution - Get prototype from plan → Claude refactors and implements → Multi-model audit and delivery.
多模型协作执行——从计划获取原型 → Claude 重构并实现 → 多模型审计与交付。

$ARGUMENTS

---

## Core Protocols
## 核心协议

- **Language Protocol**: Use **English** when interacting with tools/models, communicate with user in their language
- **语言协议**：与工具/模型交互时使用**英文**，与用户沟通时使用其语言
- **Code Sovereignty**: External models have **zero filesystem write access**, all modifications by Claude
- **代码主权**：外部模型**完全没有文件系统写入权限**，所有修改由 Claude 执行
- **Dirty Prototype Refactoring**: Treat Codex/Gemini Unified Diff as "dirty prototype", must refactor to production-grade code
- **脏原型重构**：将 Codex/Gemini 的 Unified Diff 视为"脏原型"，必须重构为生产级代码
- **Stop-Loss Mechanism**: Do not proceed to next phase until current phase output is validated
- **止损机制**：当前阶段输出未经验证，不得进入下一阶段
- **Prerequisite**: Only execute after user explicitly replies "Y" to `/ccg:plan` output (if missing, must confirm first)
- **前提条件**：仅在用户明确回复"Y"确认 `/ccg:plan` 输出后执行（若缺失，必须先确认）

---

## Multi-Model Call Specification
## 多模型调用规范

**Call Syntax** (parallel: use `run_in_background: true`):
**调用语法**（并行时使用 `run_in_background: true`）：

```
# Resume session call (recommended) - Implementation Prototype
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <task description>
Context: <plan content + target files>
</TASK>
OUTPUT: Unified Diff Patch ONLY. Strictly prohibit any actual modifications.
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "Brief description"
})

# New session call - Implementation Prototype
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <task description>
Context: <plan content + target files>
</TASK>
OUTPUT: Unified Diff Patch ONLY. Strictly prohibit any actual modifications.
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "Brief description"
})
```

**Audit Call Syntax** (Code Review / Audit):
**审计调用语法**（代码审查/审计）：

```
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Scope: Audit the final code changes.
Inputs:
- The applied patch (git diff / final unified diff)
- The touched files (relevant excerpts if needed)
Constraints:
- Do NOT modify any files.
- Do NOT output tool commands that assume filesystem access.
</TASK>
OUTPUT:
1) A prioritized list of issues (severity, file, rationale)
2) Concrete fixes; if code changes are needed, include a Unified Diff Patch in a fenced code block.
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
| Implementation | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/frontend.md` |
| 实现 | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/frontend.md` |
| Review | `~/.claude/.ccg/prompts/codex/reviewer.md` | `~/.claude/.ccg/prompts/gemini/reviewer.md` |
| 审查 | `~/.claude/.ccg/prompts/codex/reviewer.md` | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**Session Reuse**: If `/ccg:plan` provided SESSION_ID, use `resume <SESSION_ID>` to reuse context.
**会话复用**：若 `/ccg:plan` 提供了 SESSION_ID，使用 `resume <SESSION_ID>` 复用上下文。

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

**Execute Task**: $ARGUMENTS
**执行任务**：$ARGUMENTS

### Phase 0: Read Plan
### 阶段 0：读取计划

`[Mode: Prepare]`

1. **Identify Input Type**:
1. **识别输入类型**：
   - Plan file path (e.g., `.claude/plan/xxx.md`)
   - 计划文件路径（如 `.claude/plan/xxx.md`）
   - Direct task description
   - 直接任务描述

2. **Read Plan Content**:
2. **读取计划内容**：
   - If plan file path provided, read and parse
   - 若提供了计划文件路径，读取并解析
   - Extract: task type, implementation steps, key files, SESSION_ID
   - 提取：任务类型、实现步骤、关键文件、SESSION_ID

3. **Pre-Execution Confirmation**:
3. **执行前确认**：
   - If input is "direct task description" or plan missing `SESSION_ID` / key files: confirm with user first
   - 若输入为"直接任务描述"或计划缺少 `SESSION_ID`/关键文件：先与用户确认
   - If cannot confirm user replied "Y" to plan: must confirm again before proceeding
   - 若无法确认用户已对计划回复"Y"：继续前必须再次确认

4. **Task Type Routing**:
4. **任务类型路由**：

   | Task Type | Detection | Route |
   |-----------|-----------|-------|
   | 任务类型 | 检测依据 | 路由 |
   | **Frontend** | Pages, components, UI, styles, layout | Gemini |
   | **前端** | 页面、组件、UI、样式、布局 | Gemini |
   | **Backend** | API, interfaces, database, logic, algorithms | Codex |
   | **后端** | API、接口、数据库、逻辑、算法 | Codex |
   | **Fullstack** | Contains both frontend and backend | Codex ∥ Gemini parallel |
   | **全栈** | 同时包含前端和后端 | Codex ∥ Gemini 并行 |

---

### Phase 1: Quick Context Retrieval
### 阶段 1：快速上下文检索

`[Mode: Retrieval]`

**If ace-tool MCP is available**, use it for quick context retrieval:
**若 ace-tool MCP 可用**，使用它进行快速上下文检索：

Based on "Key Files" list in plan, call `mcp__ace-tool__search_context`:
根据计划中的"关键文件"列表，调用 `mcp__ace-tool__search_context`：

```
mcp__ace-tool__search_context({
  query: "<semantic query based on plan content, including key files, modules, function names>",
  project_root_path: "$PWD"
})
```

**Retrieval Strategy**:
**检索策略**：
- Extract target paths from plan's "Key Files" table
- 从计划的"关键文件"表中提取目标路径
- Build semantic query covering: entry files, dependency modules, related type definitions
- 构建涵盖以下内容的语义查询：入口文件、依赖模块、相关类型定义
- If results insufficient, add 1-2 recursive retrievals
- 若结果不足，增加 1-2 次递归检索

**If ace-tool MCP is NOT available**, use Claude Code built-in tools as fallback:
**若 ace-tool MCP 不可用**，使用 Claude Code 内置工具作为回退：
1. **Glob**: Find target files from plan's "Key Files" table (e.g., `Glob("src/components/**/*.tsx")`)
1. **Glob**：从计划的"关键文件"表中查找目标文件（如 `Glob("src/components/**/*.tsx")`）
2. **Grep**: Search for key symbols, function names, type definitions across the codebase
2. **Grep**：在代码库中搜索关键符号、函数名、类型定义
3. **Read**: Read the discovered files to gather complete context
3. **Read**：读取发现的文件以收集完整上下文
4. **Task (Explore agent)**: For broader exploration, use `Task` with `subagent_type: "Explore"`
4. **Task（Explore 代理）**：更广泛的探索时，使用 `Task` 并设置 `subagent_type: "Explore"`

**After Retrieval**:
**检索后**：
- Organize retrieved code snippets
- 整理检索到的代码片段
- Confirm complete context for implementation
- 确认实现所需的完整上下文
- Proceed to Phase 3
- 进入阶段 3

---

### Phase 3: Prototype Acquisition
### 阶段 3：原型获取

`[Mode: Prototype]`

**Route Based on Task Type**:
**根据任务类型路由**：

#### Route A: Frontend/UI/Styles → Gemini
#### 路由 A：前端/UI/样式 → Gemini

**Limit**: Context < 32k tokens
**限制**：上下文 < 32k tokens

1. Call Gemini (use `~/.claude/.ccg/prompts/gemini/frontend.md`)
1. 调用 Gemini（使用 `~/.claude/.ccg/prompts/gemini/frontend.md`）
2. Input: Plan content + retrieved context + target files
2. 输入：计划内容 + 检索上下文 + 目标文件
3. OUTPUT: `Unified Diff Patch ONLY. Strictly prohibit any actual modifications.`
3. 输出：`仅 Unified Diff Patch。严禁任何实际修改。`
4. **Gemini is frontend design authority, its CSS/React/Vue prototype is the final visual baseline**
4. **Gemini 是前端设计权威，其 CSS/React/Vue 原型是最终视觉基准**
5. **WARNING**: Ignore Gemini's backend logic suggestions
5. **警告**：忽略 Gemini 的后端逻辑建议
6. If plan contains `GEMINI_SESSION`: prefer `resume <GEMINI_SESSION>`
6. 若计划包含 `GEMINI_SESSION`：优先使用 `resume <GEMINI_SESSION>`

#### Route B: Backend/Logic/Algorithms → Codex
#### 路由 B：后端/逻辑/算法 → Codex

1. Call Codex (use `~/.claude/.ccg/prompts/codex/architect.md`)
1. 调用 Codex（使用 `~/.claude/.ccg/prompts/codex/architect.md`）
2. Input: Plan content + retrieved context + target files
2. 输入：计划内容 + 检索上下文 + 目标文件
3. OUTPUT: `Unified Diff Patch ONLY. Strictly prohibit any actual modifications.`
3. 输出：`仅 Unified Diff Patch。严禁任何实际修改。`
4. **Codex is backend logic authority, leverage its logical reasoning and debug capabilities**
4. **Codex 是后端逻辑权威，充分利用其逻辑推理和调试能力**
5. If plan contains `CODEX_SESSION`: prefer `resume <CODEX_SESSION>`
5. 若计划包含 `CODEX_SESSION`：优先使用 `resume <CODEX_SESSION>`

#### Route C: Fullstack → Parallel Calls
#### 路由 C：全栈 → 并行调用

1. **Parallel Calls** (`run_in_background: true`):
1. **并行调用**（`run_in_background: true`）：
   - Gemini: Handle frontend part
   - Gemini：处理前端部分
   - Codex: Handle backend part
   - Codex：处理后端部分
2. Wait for both models' complete results with `TaskOutput`
2. 使用 `TaskOutput` 等待两个模型的完整结果
3. Each uses corresponding `SESSION_ID` from plan for `resume` (create new session if missing)
3. 各自使用计划中对应的 `SESSION_ID` 进行 `resume`（若缺失则创建新会话）

**Follow the `IMPORTANT` instructions in `Multi-Model Call Specification` above**
**遵循上方"多模型调用规范"中的`重要`说明**

---

### Phase 4: Code Implementation
### 阶段 4：代码实现

`[Mode: Implement]`

**Claude as Code Sovereign executes the following steps**:
**Claude 作为代码主权者执行以下步骤**：

1. **Read Diff**: Parse Unified Diff Patch returned by Codex/Gemini
1. **读取 Diff**：解析 Codex/Gemini 返回的 Unified Diff Patch
2. **Mental Sandbox**:
2. **沙盒推演**：
   - Simulate applying Diff to target files
   - 模拟将 Diff 应用到目标文件
   - Check logical consistency
   - 检查逻辑一致性
   - Identify potential conflicts or side effects
   - 识别潜在冲突或副作用
3. **Refactor and Clean**:
3. **重构与清理**：
   - Refactor "dirty prototype" to **highly readable, maintainable, enterprise-grade code**
   - 将"脏原型"重构为**高可读性、可维护性的企业级代码**
   - Remove redundant code
   - 移除冗余代码
   - Ensure compliance with project's existing code standards
   - 确保符合项目现有代码规范
   - **Do not generate comments/docs unless necessary**, code should be self-explanatory
   - **除非必要，不生成注释/文档**，代码应自解释
4. **Minimal Scope**:
4. **最小范围**：
   - Changes limited to requirement scope only
   - 变更仅限于需求范围
   - **Mandatory review** for side effects
   - **强制审查**副作用
   - Make targeted corrections
   - 进行有针对性的修正
5. **Apply Changes**:
5. **应用变更**：
   - Use Edit/Write tools to execute actual modifications
   - 使用 Edit/Write 工具执行实际修改
   - **Only modify necessary code**, never affect user's other existing functionality
   - **仅修改必要代码**，绝不影响用户其他现有功能
6. **Self-Verification** (strongly recommended):
6. **自我验证**（强烈推荐）：
   - Run project's existing lint / typecheck / tests (prioritize minimal related scope)
   - 运行项目现有的 lint / typecheck / 测试（优先最小相关范围）
   - If failed: fix regressions first, then proceed to Phase 5
   - 若失败：先修复回归问题，再进入阶段 5

---

### Phase 5: Audit and Delivery
### 阶段 5：审计与交付

`[Mode: Audit]`

#### 5.1 Automatic Audit
#### 5.1 自动审计

**After changes take effect, MUST immediately parallel call** Codex and Gemini for Code Review:
**变更生效后，必须立即并行调用** Codex 和 Gemini 进行代码审查：

1. **Codex Review** (`run_in_background: true`):
1. **Codex 审查**（`run_in_background: true`）：
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/reviewer.md`
   - Input: Changed Diff + target files
   - 输入：变更 Diff + 目标文件
   - Focus: Security, performance, error handling, logic correctness
   - 关注：安全性、性能、错误处理、逻辑正确性

2. **Gemini Review** (`run_in_background: true`):
2. **Gemini 审查**（`run_in_background: true`）：
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/reviewer.md`
   - Input: Changed Diff + target files
   - 输入：变更 Diff + 目标文件
   - Focus: Accessibility, design consistency, user experience
   - 关注：无障碍访问、设计一致性、用户体验

Wait for both models' complete review results with `TaskOutput`. Prefer reusing Phase 3 sessions (`resume <SESSION_ID>`) for context consistency.
使用 `TaskOutput` 等待两个模型的完整审查结果。优先复用阶段 3 的会话（`resume <SESSION_ID>`）以保持上下文一致性。

#### 5.2 Integrate and Fix
#### 5.2 整合与修复

1. Synthesize Codex + Gemini review feedback
1. 综合 Codex + Gemini 的审查反馈
2. Weigh by trust rules: Backend follows Codex, Frontend follows Gemini
2. 按信任规则权衡：后端遵循 Codex，前端遵循 Gemini
3. Execute necessary fixes
3. 执行必要的修复
4. Repeat Phase 5.1 as needed (until risk is acceptable)
4. 按需重复阶段 5.1（直到风险可接受）

#### 5.3 Delivery Confirmation
#### 5.3 交付确认

After audit passes, report to user:
审计通过后，向用户报告：

```markdown
## Execution Complete

### Change Summary
| File | Operation | Description |
|------|-----------|-------------|
| path/to/file.ts | Modified | Description |

### Audit Results
- Codex: <Passed/Found N issues>
- Gemini: <Passed/Found N issues>

### Recommendations
1. [ ] <Suggested test steps>
2. [ ] <Suggested verification steps>
```

---

## Key Rules
## 关键规则

1. **Code Sovereignty** – All file modifications by Claude, external models have zero write access
1. **代码主权** — 所有文件修改由 Claude 执行，外部模型无写入权限
2. **Dirty Prototype Refactoring** – Codex/Gemini output treated as draft, must refactor
2. **脏原型重构** — Codex/Gemini 输出视为草稿，必须重构
3. **Trust Rules** – Backend follows Codex, Frontend follows Gemini
3. **信任规则** — 后端遵循 Codex，前端遵循 Gemini
4. **Minimal Changes** – Only modify necessary code, no side effects
4. **最小变更** — 仅修改必要代码，无副作用
5. **Mandatory Audit** – Must perform multi-model Code Review after changes
5. **强制审计** — 变更后必须执行多模型代码审查

---

## Usage
## 使用方法

```bash
# Execute plan file
/ccg:execute .claude/plan/feature-name.md

# Execute task directly (for plans already discussed in context)
/ccg:execute implement user authentication based on previous plan
```

---

## Relationship with /ccg:plan
## 与 /ccg:plan 的关系

1. `/ccg:plan` generates plan + SESSION_ID
1. `/ccg:plan` 生成计划 + SESSION_ID
2. User confirms with "Y"
2. 用户以"Y"确认
3. `/ccg:execute` reads plan, reuses SESSION_ID, executes implementation
3. `/ccg:execute` 读取计划，复用 SESSION_ID，执行实现
