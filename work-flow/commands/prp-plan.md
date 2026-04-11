---
description: Create comprehensive feature implementation plan with codebase analysis and pattern extraction
  通过代码库分析和模式提取，创建全面的功能实现计划
argument-hint: <feature description | path/to/prd.md>
  <功能描述 | PRD 文件路径/prd.md>
---

> Adapted from PRPs-agentic-eng by Wirasm. Part of the PRP workflow series.
> 改编自 Wirasm 的 PRPs-agentic-eng，属于 PRP 工作流系列。

# PRP Plan
# PRP 规划

Create a detailed, self-contained implementation plan that captures all codebase patterns, conventions, and context needed to implement a feature in a single pass.
创建详细、自包含的实现计划，捕获单次实现功能所需的所有代码库模式、规范和上下文。

**Core Philosophy**: A great plan contains everything needed to implement without asking further questions. Every pattern, every convention, every gotcha — captured once, referenced throughout.
**核心理念**：一份好的计划包含实现所需的一切，无需再次提问。每个模式、每条规范、每个注意事项——一次捕获，全程引用。

**Golden Rule**: If you would need to search the codebase during implementation, capture that knowledge NOW in the plan.
**黄金法则**：如果在实现过程中需要搜索代码库，现在就将这些知识记录到计划中。

---

## Phase 0 — DETECT
## 阶段 0 — 检测

Determine input type from `$ARGUMENTS`:
从 `$ARGUMENTS` 确定输入类型：

| Input Pattern | Detection | Action |
|---|---|---|
| 输入模式 | 检测方式 | 操作 |
| Path ending in `.prd.md` | File path to PRD | Parse PRD, find next pending phase |
| 以 `.prd.md` 结尾的路径 | PRD 文件路径 | 解析 PRD，查找下一个待处理阶段 |
| Path to `.md` with "Implementation Phases" | PRD-like document | Parse phases, find next pending |
| 包含 "Implementation Phases" 的 `.md` 路径 | 类 PRD 文档 | 解析阶段，查找下一个待处理 |
| Path to any other file | Reference file | Read file for context, treat as free-form |
| 其他任意文件路径 | 参考文件 | 读取文件获取上下文，视为自由格式 |
| Free-form text | Feature description | Proceed directly to Phase 1 |
| 自由格式文本 | 功能描述 | 直接进入阶段 1 |
| Empty / blank | No input | Ask user what feature to plan |
| 空/留空 | 无输入 | 询问用户要规划的功能 |

### PRD Parsing (when input is a PRD)
### PRD 解析（当输入为 PRD 时）

1. Read the PRD file with `cat "$PRD_PATH"`
1. 使用 `cat "$PRD_PATH"` 读取 PRD 文件
2. Parse the **Implementation Phases** section
2. 解析 **Implementation Phases** 部分
3. Find phases by status:
3. 按状态查找阶段：
   - Look for `pending` phases
   - 查找 `pending` 状态的阶段
   - Check dependency chains (a phase may depend on prior phases being `complete`)
   - 检查依赖链（某阶段可能依赖前置阶段为 `complete` 状态）
   - Select the **next eligible pending phase**
   - 选择**下一个符合条件的待处理阶段**
4. Extract from the selected phase:
4. 从所选阶段中提取：
   - Phase name and description
   - 阶段名称和描述
   - Acceptance criteria
   - 验收标准
   - Dependencies on prior phases
   - 对前置阶段的依赖
   - Any scope notes or constraints
   - 任何范围说明或约束
5. Use the phase description as the feature to plan
5. 将阶段描述作为要规划的功能

If no pending phases remain, report that all phases are complete.
若无待处理阶段，报告所有阶段已完成。

---

## Phase 1 — PARSE
## 阶段 1 — 解析

Extract and clarify the feature requirements.
提取并明确功能需求。

### Feature Understanding
### 功能理解

From the input (PRD phase or free-form description), identify:
从输入（PRD 阶段或自由格式描述）中识别：

- **What** is being built (concrete deliverable)
- **构建什么**（具体交付物）
- **Why** it matters (user value)
- **为什么重要**（用户价值）
- **Who** uses it (target user/system)
- **谁使用它**（目标用户/系统）
- **Where** it fits (which part of the codebase)
- **适用于哪里**（代码库的哪个部分）

### User Story
### 用户故事

Format as:
格式如下：
```
As a [type of user],
I want [capability],
So that [benefit].
```

### Complexity Assessment
### 复杂度评估

| Level | Indicators | Typical Scope |
|---|---|---|
| 级别 | 指标 | 典型范围 |
| **Small** | Single file, isolated change, no new dependencies | 1-3 files, <100 lines |
| **小型** | 单文件、独立变更、无新依赖 | 1-3 个文件，<100 行 |
| **Medium** | Multiple files, follows existing patterns, minor new concepts | 3-10 files, 100-500 lines |
| **中型** | 多文件、遵循现有模式、少量新概念 | 3-10 个文件，100-500 行 |
| **Large** | Cross-cutting concerns, new patterns, external integrations | 10+ files, 500+ lines |
| **大型** | 横切关注点、新模式、外部集成 | 10+ 个文件，500+ 行 |
| **XL** | Architectural changes, new subsystems, migration needed | 20+ files, consider splitting |
| **超大型** | 架构变更、新子系统、需要迁移 | 20+ 个文件，考虑拆分 |

### Ambiguity Gate
### 歧义关卡

If any of these are unclear, **STOP and ask the user** before proceeding:
若以下任何一项不明确，**停止并询问用户**，再继续：

- The core deliverable is vague
- 核心交付物模糊不清
- Success criteria are undefined
- 成功标准未定义
- There are multiple valid interpretations
- 存在多种有效解释
- Technical approach has major unknowns
- 技术方案存在重大未知

Do NOT guess. Ask. A plan built on assumptions fails during implementation.
不要猜测。要询问。基于假设构建的计划会在实现过程中失败。

---

## Phase 2 — EXPLORE
## 阶段 2 — 探索

Gather deep codebase intelligence. Search the codebase directly for each category below.
深度收集代码库信息。直接在代码库中搜索以下每个类别。

### Codebase Search (8 Categories)
### 代码库搜索（8 个类别）

For each category, search using grep, find, and file reading:
对每个类别，使用 grep、find 和文件读取进行搜索：

1. **Similar Implementations** — Find existing features that resemble the planned one. Look for analogous patterns, endpoints, components, or modules.
1. **相似实现** — 查找与计划功能相似的现有功能，寻找类似的模式、端点、组件或模块。

2. **Naming Conventions** — Identify how files, functions, variables, classes, and exports are named in the relevant area of the codebase.
2. **命名规范** — 识别代码库相关区域中文件、函数、变量、类和导出的命名方式。

3. **Error Handling** — Find how errors are caught, propagated, logged, and returned to users in similar code paths.
3. **错误处理** — 查找类似代码路径中错误的捕获、传播、记录和返回方式。

4. **Logging Patterns** — Identify what gets logged, at what level, and in what format.
4. **日志模式** — 识别记录的内容、日志级别和格式。

5. **Type Definitions** — Find relevant types, interfaces, schemas, and how they're organized.
5. **类型定义** — 查找相关类型、接口、模式及其组织方式。

6. **Test Patterns** — Find how similar features are tested. Note test file locations, naming, setup/teardown patterns, and assertion styles.
6. **测试模式** — 查找相似功能的测试方式，记录测试文件位置、命名、setup/teardown 模式和断言风格。

7. **Configuration** — Find relevant config files, environment variables, and feature flags.
7. **配置** — 查找相关配置文件、环境变量和功能开关。

8. **Dependencies** — Identify packages, imports, and internal modules used by similar features.
8. **依赖** — 识别相似功能使用的包、导入和内部模块。

### Codebase Analysis (5 Traces)
### 代码库分析（5 条追踪路径）

Read relevant files to trace:
读取相关文件以追踪：

1. **Entry Points** — How does a request/action enter the system and reach the area you're modifying?
1. **入口点** — 请求/操作如何进入系统并到达你要修改的区域？
2. **Data Flow** — How does data move through the relevant code paths?
2. **数据流** — 数据如何在相关代码路径中流动？
3. **State Changes** — What state is modified and where?
3. **状态变更** — 哪些状态被修改，在哪里修改？
4. **Contracts** — What interfaces, APIs, or protocols must be honored?
4. **契约** — 必须遵守哪些接口、API 或协议？
5. **Patterns** — What architectural patterns are used (repository, service, controller, etc.)?
5. **模式** — 使用了哪些架构模式（仓储、服务、控制器等）？

### Unified Discovery Table
### 统一发现表

Compile findings into a single reference:
将发现整理为统一参考：

| Category | File:Lines | Pattern | Key Snippet |
|---|---|---|---|
| 分类 | 文件:行号 | 模式 | 关键片段 |
| Naming | `src/services/userService.ts:1-5` | camelCase services, PascalCase types | `export class UserService` |
| 命名 | `src/services/userService.ts:1-5` | camelCase 服务，PascalCase 类型 | `export class UserService` |
| Error | `src/middleware/errorHandler.ts:10-25` | Custom AppError class | `throw new AppError(...)` |
| 错误处理 | `src/middleware/errorHandler.ts:10-25` | 自定义 AppError 类 | `throw new AppError(...)` |
| ... | ... | ... | ... |
| ... | ... | ... | ... |

---

## Phase 3 — RESEARCH
## 阶段 3 — 研究

If the feature involves external libraries, APIs, or unfamiliar technology:
若功能涉及外部库、API 或不熟悉的技术：

1. Search the web for official documentation
1. 搜索官方文档
2. Find usage examples and best practices
2. 查找使用示例和最佳实践
3. Identify version-specific gotchas
3. 识别特定版本的注意事项

Format each finding as:
每条发现格式如下：

```
KEY_INSIGHT: [what you learned]
APPLIES_TO: [which part of the plan this affects]
GOTCHA: [any warnings or version-specific issues]
```

If the feature uses only well-understood internal patterns, skip this phase and note: "No external research needed — feature uses established internal patterns."
若功能仅使用熟悉的内部模式，跳过此阶段并注明："无需外部研究——功能使用既有内部模式。"

---

## Phase 4 — DESIGN
## 阶段 4 — 设计

### UX Transformation (if applicable)
### UX 转变（如适用）

Document the before/after user experience:
记录变更前后的用户体验：

**Before:**
**变更前：**
```
┌─────────────────────────────┐
│  [Current user experience]  │
│  Show the current flow,     │
│  what the user sees/does    │
└─────────────────────────────┘
```

**After:**
**变更后：**
```
┌─────────────────────────────┐
│  [New user experience]      │
│  Show the improved flow,    │
│  what changes for the user  │
└─────────────────────────────┘
```

### Interaction Changes
### 交互变更

| Touchpoint | Before | After | Notes |
|---|---|---|---|
| 触点 | 变更前 | 变更后 | 备注 |
| ... | ... | ... | ... |
| ... | ... | ... | ... |

If the feature is purely backend/internal with no UX change, note: "Internal change — no user-facing UX transformation."
若功能为纯后端/内部变更且无 UX 变化，注明："内部变更——无面向用户的 UX 转变。"

---

## Phase 5 — ARCHITECT
## 阶段 5 — 架构设计

### Strategic Design
### 策略设计

Define the implementation approach:
定义实现方案：

- **Approach**: High-level strategy (e.g., "Add new service layer following existing repository pattern")
- **Approach**：高层策略（如"按照现有仓储模式添加新的服务层"）
- **Alternatives Considered**: What other approaches were evaluated and why they were rejected
- **Alternatives Considered**：评估了哪些其他方案及拒绝原因
- **Scope**: Concrete boundaries of what WILL be built
- **Scope**：将要构建内容的具体边界
- **NOT Building**: Explicit list of what is OUT OF SCOPE (prevents scope creep during implementation)
- **NOT Building**：明确列出超出范围的内容（防止实现时范围蔓延）

---

## Phase 6 — GENERATE
## 阶段 6 — 生成

Write the full plan document using the template below. Save to `.claude/PRPs/plans/{kebab-case-feature-name}.plan.md`.
使用以下模板编写完整计划文档，保存到 `.claude/PRPs/plans/{kebab-case-feature-name}.plan.md`。

Create the directory if it doesn't exist:
若目录不存在，先创建：
```bash
mkdir -p .claude/PRPs/plans
```

### Plan Template
### 计划模板

````markdown
# Plan: [Feature Name]

## Summary
[2-3 sentence overview]

## User Story
As a [user], I want [capability], so that [benefit].

## Problem → Solution
[Current state] → [Desired state]

## Metadata
- **Complexity**: [Small | Medium | Large | XL]
- **Source PRD**: [path or "N/A"]
- **PRD Phase**: [phase name or "N/A"]
- **Estimated Files**: [count]

---

## UX Design

### Before
[ASCII diagram or "N/A — internal change"]

### After
[ASCII diagram or "N/A — internal change"]

### Interaction Changes
| Touchpoint | Before | After | Notes |
|---|---|---|---|

---

## Mandatory Reading

Files that MUST be read before implementing:

| Priority | File | Lines | Why |
|---|---|---|---|
| P0 (critical) | `path/to/file` | 1-50 | Core pattern to follow |
| P1 (important) | `path/to/file` | 10-30 | Related types |
| P2 (reference) | `path/to/file` | all | Similar implementation |

## External Documentation

| Topic | Source | Key Takeaway |
|---|---|---|
| ... | ... | ... |

---

## Patterns to Mirror

Code patterns discovered in the codebase. Follow these exactly.

### NAMING_CONVENTION
// SOURCE: [file:lines]
[actual code snippet showing the naming pattern]

### ERROR_HANDLING
// SOURCE: [file:lines]
[actual code snippet showing error handling]

### LOGGING_PATTERN
// SOURCE: [file:lines]
[actual code snippet showing logging]

### REPOSITORY_PATTERN
// SOURCE: [file:lines]
[actual code snippet showing data access]

### SERVICE_PATTERN
// SOURCE: [file:lines]
[actual code snippet showing service layer]

### TEST_STRUCTURE
// SOURCE: [file:lines]
[actual code snippet showing test setup]

---

## Files to Change

| File | Action | Justification |
|---|---|---|
| `path/to/file.ts` | CREATE | New service for feature |
| `path/to/existing.ts` | UPDATE | Add new method |

## NOT Building

- [Explicit item 1 that is out of scope]
- [Explicit item 2 that is out of scope]

---

## Step-by-Step Tasks

### Task 1: [Name]
- **ACTION**: [What to do]
- **IMPLEMENT**: [Specific code/logic to write]
- **MIRROR**: [Pattern from Patterns to Mirror section to follow]
- **IMPORTS**: [Required imports]
- **GOTCHA**: [Known pitfall to avoid]
- **VALIDATE**: [How to verify this task is correct]

### Task 2: [Name]
- **ACTION**: ...
- **IMPLEMENT**: ...
- **MIRROR**: ...
- **IMPORTS**: ...
- **GOTCHA**: ...
- **VALIDATE**: ...

[Continue for all tasks...]

---

## Testing Strategy

### Unit Tests

| Test | Input | Expected Output | Edge Case? |
|---|---|---|---|
| ... | ... | ... | ... |

### Edge Cases Checklist
- [ ] Empty input
- [ ] Maximum size input
- [ ] Invalid types
- [ ] Concurrent access
- [ ] Network failure (if applicable)
- [ ] Permission denied

---

## Validation Commands

### Static Analysis
```bash
# Run type checker
[project-specific type check command]
```
EXPECT: Zero type errors

### Unit Tests
```bash
# Run tests for affected area
[project-specific test command]
```
EXPECT: All tests pass

### Full Test Suite
```bash
# Run complete test suite
[project-specific full test command]
```
EXPECT: No regressions

### Database Validation (if applicable)
```bash
# Verify schema/migrations
[project-specific db command]
```
EXPECT: Schema up to date

### Browser Validation (if applicable)
```bash
# Start dev server and verify
[project-specific dev server command]
```
EXPECT: Feature works as designed

### Manual Validation
- [ ] [Step-by-step manual verification checklist]

---

## Acceptance Criteria
- [ ] All tasks completed
- [ ] All validation commands pass
- [ ] Tests written and passing
- [ ] No type errors
- [ ] No lint errors
- [ ] Matches UX design (if applicable)

## Completion Checklist
- [ ] Code follows discovered patterns
- [ ] Error handling matches codebase style
- [ ] Logging follows codebase conventions
- [ ] Tests follow test patterns
- [ ] No hardcoded values
- [ ] Documentation updated (if needed)
- [ ] No unnecessary scope additions
- [ ] Self-contained — no questions needed during implementation

## Risks
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| ... | ... | ... | ... |

## Notes
[Any additional context, decisions, or observations]
```

---

## Output

### Save the Plan

Write the generated plan to:
```
.claude/PRPs/plans/{kebab-case-feature-name}.plan.md
```

### Update PRD (if input was a PRD)

If this plan was generated from a PRD phase:
1. Update the phase status from `pending` to `in-progress`
2. Add the plan file path as a reference in the phase

### Report to User

```
## Plan Created

- **File**: .claude/PRPs/plans/{kebab-case-feature-name}.plan.md
- **Source PRD**: [path or "N/A"]
- **Phase**: [phase name or "standalone"]
- **Complexity**: [level]
- **Scope**: [N files, M tasks]
- **Key Patterns**: [top 3 discovered patterns]
- **External Research**: [topics researched or "none needed"]
- **Risks**: [top risk or "none identified"]
- **Confidence Score**: [1-10] — likelihood of single-pass implementation

> Next step: Run `/prp-implement .claude/PRPs/plans/{name}.plan.md` to execute this plan.
```

---

## Verification

Before finalizing, verify the plan against these checklists:

### Context Completeness
- [ ] All relevant files discovered and documented
- [ ] Naming conventions captured with examples
- [ ] Error handling patterns documented
- [ ] Test patterns identified
- [ ] Dependencies listed

### Implementation Readiness
- [ ] Every task has ACTION, IMPLEMENT, MIRROR, and VALIDATE
- [ ] No task requires additional codebase searching
- [ ] Import paths are specified
- [ ] GOTCHAs documented where applicable

### Pattern Faithfulness
- [ ] Code snippets are actual codebase examples (not invented)
- [ ] SOURCE references point to real files and line numbers
- [ ] Patterns cover naming, errors, logging, data access, and tests
- [ ] New code will be indistinguishable from existing code

### Validation Coverage
- [ ] Static analysis commands specified
- [ ] Test commands specified
- [ ] Build verification included

### UX Clarity
- [ ] Before/after states documented (or marked N/A)
- [ ] Interaction changes listed
- [ ] Edge cases for UX identified

### No Prior Knowledge Test
A developer unfamiliar with this codebase should be able to implement the feature using ONLY this plan, without searching the codebase or asking questions. If not, add the missing context.

---

## Next Steps

- Run `/prp-implement <plan-path>` to execute this plan
- Run `/plan` for quick conversational planning without artifacts
- Run `/prp-prd` to create a PRD first if scope is unclear
````
