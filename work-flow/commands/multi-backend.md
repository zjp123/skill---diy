# Backend - Backend-Focused Development
# Backend - 后端专项开发

Backend-focused workflow (Research → Ideation → Plan → Execute → Optimize → Review), Codex-led.
以 Codex 为主导的后端专项工作流（研究 → 构思 → 规划 → 执行 → 优化 → 审查）。

## Usage
## 使用方法

```bash
/backend <backend task description>
```

## Context
## 上下文

- Backend task: $ARGUMENTS
- 后端任务：$ARGUMENTS
- Codex-led, Gemini for auxiliary reference
- Codex 主导，Gemini 作为辅助参考
- Applicable: API design, algorithm implementation, database optimization, business logic
- 适用场景：API 设计、算法实现、数据库优化、业务逻辑

## Your Role
## 你的角色

You are the **Backend Orchestrator**, coordinating multi-model collaboration for server-side tasks (Research → Ideation → Plan → Execute → Optimize → Review).
你是**后端编排者**，负责协调多模型协作以完成服务端任务（研究 → 构思 → 规划 → 执行 → 优化 → 审查）。

**Collaborative Models**:
**协作模型**：
- **Codex** – Backend logic, algorithms (**Backend authority, trustworthy**)
- **Codex** – 后端逻辑、算法（**后端权威，可信赖**）
- **Gemini** – Frontend perspective (**Backend opinions for reference only**)
- **Gemini** – 前端视角（**后端意见仅供参考**）
- **Claude (self)** – Orchestration, planning, execution, delivery
- **Claude（自身）** – 编排、规划、执行、交付

---

## Multi-Model Call Specification
## 多模型调用规范

**Call Syntax**:
**调用语法**：

```
# New session call
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend codex - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement (or $ARGUMENTS if not enhanced)>
Context: <project context and analysis from previous phases>
</TASK>
OUTPUT: Expected output format
EOF",
  run_in_background: false,
  timeout: 3600000,
  description: "Brief description"
})

# Resume session call
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend codex resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement (or $ARGUMENTS if not enhanced)>
Context: <project context and analysis from previous phases>
</TASK>
OUTPUT: Expected output format
EOF",
  run_in_background: false,
  timeout: 3600000,
  description: "Brief description"
})
```

**Role Prompts**:
**角色提示词**：

| Phase | Codex |
|-------|-------|
| 阶段 | Codex |
| Analysis | `~/.claude/.ccg/prompts/codex/analyzer.md` |
| 分析 | `~/.claude/.ccg/prompts/codex/analyzer.md` |
| Planning | `~/.claude/.ccg/prompts/codex/architect.md` |
| 规划 | `~/.claude/.ccg/prompts/codex/architect.md` |
| Review | `~/.claude/.ccg/prompts/codex/reviewer.md` |
| 审查 | `~/.claude/.ccg/prompts/codex/reviewer.md` |

**Session Reuse**: Each call returns `SESSION_ID: xxx`, use `resume xxx` for subsequent phases. Save `CODEX_SESSION` in Phase 2, use `resume` in Phases 3 and 5.
**会话复用**：每次调用返回 `SESSION_ID: xxx`，后续阶段使用 `resume xxx`。在阶段 2 保存 `CODEX_SESSION`，在阶段 3 和 5 中使用 `resume`。

---

## Communication Guidelines
## 沟通规范

1. Start responses with mode label `[Mode: X]`, initial is `[Mode: Research]`
1. 响应以模式标签 `[Mode: X]` 开头，初始为 `[Mode: Research]`
2. Follow strict sequence: `Research → Ideation → Plan → Execute → Optimize → Review`
2. 严格遵循顺序：`Research → Ideation → Plan → Execute → Optimize → Review`
3. Use `AskUserQuestion` tool for user interaction when needed (e.g., confirmation/selection/approval)
3. 需要用户交互时（如确认/选择/批准）使用 `AskUserQuestion` 工具

---

## Core Workflow
## 核心工作流

### Phase 0: Prompt Enhancement (Optional)
### 阶段 0：提示词增强（可选）

`[Mode: Prepare]` - If ace-tool MCP available, call `mcp__ace-tool__enhance_prompt`, **replace original $ARGUMENTS with enhanced result for subsequent Codex calls**. If unavailable, use `$ARGUMENTS` as-is.
`[Mode: Prepare]` - 若 ace-tool MCP 可用，调用 `mcp__ace-tool__enhance_prompt`，**用增强结果替换原始 $ARGUMENTS 供后续 Codex 调用使用**。若不可用，直接使用 `$ARGUMENTS`。

### Phase 1: Research
### 阶段 1：研究

`[Mode: Research]` - Understand requirements and gather context
`[Mode: Research]` - 理解需求并收集上下文

1. **Code Retrieval** (if ace-tool MCP available): Call `mcp__ace-tool__search_context` to retrieve existing APIs, data models, service architecture. If unavailable, use built-in tools: `Glob` for file discovery, `Grep` for symbol/API search, `Read` for context gathering, `Task` (Explore agent) for deeper exploration.
1. **代码检索**（若 ace-tool MCP 可用）：调用 `mcp__ace-tool__search_context` 检索现有 API、数据模型、服务架构。若不可用，使用内置工具：`Glob` 用于文件发现，`Grep` 用于符号/API 搜索，`Read` 用于上下文收集，`Task`（Explore 代理）用于深度探索。
2. Requirement completeness score (0-10): >=7 continue, <7 stop and supplement
2. 需求完整度评分（0-10）：>=7 继续，<7 停止并补充

### Phase 2: Ideation
### 阶段 2：构思

`[Mode: Ideation]` - Codex-led analysis
`[Mode: Ideation]` - Codex 主导的分析

**MUST call Codex** (follow call specification above):
**必须调用 Codex**（遵循上方调用规范）：
- ROLE_FILE: `~/.claude/.ccg/prompts/codex/analyzer.md`
- Requirement: Enhanced requirement (or $ARGUMENTS if not enhanced)
- Requirement：增强后的需求（若未增强则为 $ARGUMENTS）
- Context: Project context from Phase 1
- Context：阶段 1 的项目上下文
- OUTPUT: Technical feasibility analysis, recommended solutions (at least 2), risk assessment
- OUTPUT：技术可行性分析、推荐方案（至少 2 个）、风险评估

**Save SESSION_ID** (`CODEX_SESSION`) for subsequent phase reuse.
**保存 SESSION_ID**（`CODEX_SESSION`）供后续阶段复用。

Output solutions (at least 2), wait for user selection.
输出方案（至少 2 个），等待用户选择。

### Phase 3: Planning
### 阶段 3：规划

`[Mode: Plan]` - Codex-led planning
`[Mode: Plan]` - Codex 主导的规划

**MUST call Codex** (use `resume <CODEX_SESSION>` to reuse session):
**必须调用 Codex**（使用 `resume <CODEX_SESSION>` 复用会话）：
- ROLE_FILE: `~/.claude/.ccg/prompts/codex/architect.md`
- Requirement: User's selected solution
- Requirement：用户选择的方案
- Context: Analysis results from Phase 2
- Context：阶段 2 的分析结果
- OUTPUT: File structure, function/class design, dependency relationships
- OUTPUT：文件结构、函数/类设计、依赖关系

Claude synthesizes plan, save to `.claude/plan/task-name.md` after user approval.
Claude 综合规划，经用户批准后保存到 `.claude/plan/task-name.md`。

### Phase 4: Implementation
### 阶段 4：实现

`[Mode: Execute]` - Code development
`[Mode: Execute]` - 代码开发

- Strictly follow approved plan
- 严格遵循已批准的规划
- Follow existing project code standards
- 遵循项目现有代码规范
- Ensure error handling, security, performance optimization
- 确保错误处理、安全性、性能优化

### Phase 5: Optimization
### 阶段 5：优化

`[Mode: Optimize]` - Codex-led review
`[Mode: Optimize]` - Codex 主导的审查

**MUST call Codex** (follow call specification above):
**必须调用 Codex**（遵循上方调用规范）：
- ROLE_FILE: `~/.claude/.ccg/prompts/codex/reviewer.md`
- Requirement: Review the following backend code changes
- Requirement：审查以下后端代码变更
- Context: git diff or code content
- Context：git diff 或代码内容
- OUTPUT: Security, performance, error handling, API compliance issues list
- OUTPUT：安全性、性能、错误处理、API 合规性问题列表

Integrate review feedback, execute optimization after user confirmation.
整合审查反馈，经用户确认后执行优化。

### Phase 6: Quality Review
### 阶段 6：质量审查

`[Mode: Review]` - Final evaluation
`[Mode: Review]` - 最终评估

- Check completion against plan
- 对照规划检查完成情况
- Run tests to verify functionality
- 运行测试验证功能
- Report issues and recommendations
- 报告问题和建议

---

## Key Rules
## 关键规则

1. **Codex backend opinions are trustworthy**
1. **Codex 的后端意见可信赖**
2. **Gemini backend opinions for reference only**
2. **Gemini 的后端意见仅供参考**
3. External models have **zero filesystem write access**
3. 外部模型**完全没有文件系统写入权限**
4. Claude handles all code writes and file operations
4. Claude 处理所有代码写入和文件操作
