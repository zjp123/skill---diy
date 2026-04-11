---
description: Execute an implementation plan with rigorous validation loops
  通过严格的验证循环执行实现计划
argument-hint: <path/to/plan.md>
  <计划文件路径/plan.md>
---

> Adapted from PRPs-agentic-eng by Wirasm. Part of the PRP workflow series.
> 改编自 Wirasm 的 PRPs-agentic-eng，属于 PRP 工作流系列。

# PRP Implement
# PRP 实现

Execute a plan file step-by-step with continuous validation. Every change is verified immediately — never accumulate broken state.
逐步执行计划文件并持续验证。每次变更立即验证——绝不积累错误状态。

**Core Philosophy**: Validation loops catch mistakes early. Run checks after every change. Fix issues immediately.
**核心理念**：验证循环能尽早发现错误。每次变更后运行检查，立即修复问题。

**Golden Rule**: If a validation fails, fix it before moving on. Never accumulate broken state.
**黄金法则**：若验证失败，在继续之前先修复。绝不积累错误状态。

---

## Phase 0 — DETECT
## 阶段 0 — 检测

### Package Manager Detection
### 包管理器检测

| File Exists | Package Manager | Runner |
|---|---|---|
| 存在的文件 | 包管理器 | 运行器 |
| `bun.lockb` | bun | `bun run` |
| `bun.lockb` | bun | `bun run` |
| `pnpm-lock.yaml` | pnpm | `pnpm run` |
| `pnpm-lock.yaml` | pnpm | `pnpm run` |
| `yarn.lock` | yarn | `yarn` |
| `yarn.lock` | yarn | `yarn` |
| `package-lock.json` | npm | `npm run` |
| `package-lock.json` | npm | `npm run` |
| `pyproject.toml` or `requirements.txt` | uv / pip | `uv run` or `python -m` |
| `pyproject.toml` or `requirements.txt` | uv / pip | `uv run` or `python -m` |
| `Cargo.toml` | cargo | `cargo` |
| `Cargo.toml` | cargo | `cargo` |
| `go.mod` | go | `go` |
| `go.mod` | go | `go` |

### Validation Scripts
### 验证脚本

Check `package.json` (or equivalent) for available scripts:
检查 `package.json`（或同等文件）中的可用脚本：

```bash
# For Node.js projects
cat package.json | grep -A 20 '"scripts"'
```

Note available commands for: type-check, lint, test, build.
记录可用命令：type-check、lint、test、build。

---

## Phase 1 — LOAD
## 阶段 1 — 加载

Read the plan file:
读取计划文件：

```bash
cat "$ARGUMENTS"
```

Extract these sections from the plan:
从计划中提取以下部分：
- **Summary** — What is being built
- **Summary** — 正在构建的内容
- **Patterns to Mirror** — Code conventions to follow
- **Patterns to Mirror** — 需要遵循的代码规范
- **Files to Change** — What to create or modify
- **Files to Change** — 需要创建或修改的文件
- **Step-by-Step Tasks** — Implementation sequence
- **Step-by-Step Tasks** — 实现步骤序列
- **Validation Commands** — How to verify correctness
- **Validation Commands** — 验证正确性的方式
- **Acceptance Criteria** — Definition of done
- **Acceptance Criteria** — 完成定义

If the file doesn't exist or isn't a valid plan:
若文件不存在或不是有效计划：
```
Error: Plan file not found or invalid.
Run /prp-plan <feature-description> to create a plan first.
```

**CHECKPOINT**: Plan loaded. All sections identified. Tasks extracted.
**检查点**：计划已加载。所有部分已识别。任务已提取。

---

## Phase 2 — PREPARE
## 阶段 2 — 准备

### Git State
### Git 状态

```bash
git branch --show-current
git status --porcelain
```

### Branch Decision
### 分支决策

| Current State | Action |
|---|---|
| 当前状态 | 操作 |
| On feature branch | Use current branch |
| 在功能分支上 | 使用当前分支 |
| On main, clean working tree | Create feature branch: `git checkout -b feat/{plan-name}` |
| 在 main 分支，工作树干净 | 创建功能分支：`git checkout -b feat/{plan-name}` |
| On main, dirty working tree | **STOP** — Ask user to stash or commit first |
| 在 main 分支，工作树有改动 | **停止** — 请用户先暂存或提交 |
| In a git worktree for this feature | Use the worktree |
| 在此功能的 git worktree 中 | 使用该 worktree |

### Sync Remote
### 同步远程

```bash
git pull --rebase origin $(git branch --show-current) 2>/dev/null || true
```

**CHECKPOINT**: On correct branch. Working tree ready. Remote synced.
**检查点**：在正确分支上。工作树就绪。远程已同步。

---

## Phase 3 — EXECUTE
## 阶段 3 — 执行

Process each task from the plan sequentially.
按顺序处理计划中的每个任务。

### Per-Task Loop
### 逐任务循环

For each task in **Step-by-Step Tasks**:
对 **Step-by-Step Tasks** 中的每个任务：

1. **Read MIRROR reference** — Open the pattern file referenced in the task's MIRROR field. Understand the convention before writing code.
1. **读取 MIRROR 参考** — 打开任务 MIRROR 字段引用的模式文件，在编写代码前理解规范。

2. **Implement** — Write the code following the pattern exactly. Apply GOTCHA warnings. Use specified IMPORTS.
2. **实现** — 严格按照模式编写代码，应用 GOTCHA 警告，使用指定的 IMPORTS。

3. **Validate immediately** — After EVERY file change:
3. **立即验证** — 每次文件变更后：
   ```bash
   # Run type-check (adjust command per project)
   [type-check command from Phase 0]
   ```
   If type-check fails → fix the error before moving to the next file.
   若类型检查失败 → 在移至下一个文件之前先修复错误。

4. **Track progress** — Log: `[done] Task N: [task name] — complete`
4. **跟踪进度** — 记录：`[done] Task N: [task name] — complete`

### Handling Deviations
### 处理偏差

If implementation must deviate from the plan:
若实现必须偏离计划：
- Note **WHAT** changed
- 记录**什么**发生了变化
- Note **WHY** it changed
- 记录**为什么**发生变化
- Continue with the corrected approach
- 继续使用修正后的方案
- These deviations will be captured in the report
- 这些偏差将记录在报告中

**CHECKPOINT**: All tasks executed. Deviations logged.
**检查点**：所有任务已执行。偏差已记录。

---

## Phase 4 — VALIDATE
## 阶段 4 — 验证

Run all validation levels from the plan. Fix issues at each level before proceeding.
运行计划中的所有验证级别。在继续之前修复每个级别的问题。

### Level 1: Static Analysis
### 级别 1：静态分析

```bash
# Type checking — zero errors required
[project type-check command]

# Linting — fix automatically where possible
[project lint command]
[project lint-fix command]
```

If lint errors remain after auto-fix, fix manually.
若自动修复后仍有 lint 错误，请手动修复。

### Level 2: Unit Tests
### 级别 2：单元测试

Write tests for every new function (as specified in the plan's Testing Strategy).
为每个新函数编写测试（按计划中的测试策略执行）。

```bash
[project test command for affected area]
```

- Every function needs at least one test
- 每个函数至少需要一个测试
- Cover edge cases listed in the plan
- 覆盖计划中列出的边界情况
- If a test fails → fix the implementation (not the test, unless the test is wrong)
- 若测试失败 → 修复实现（而非测试，除非测试本身有误）

### Level 3: Build Check
### 级别 3：构建检查

```bash
[project build command]
```

Build must succeed with zero errors.
构建必须零错误通过。

### Level 4: Integration Testing (if applicable)
### 级别 4：集成测试（如适用）

```bash
# Start server, run tests, stop server
[project dev server command] &
SERVER_PID=$!

# Wait for server to be ready (adjust port as needed)
SERVER_READY=0
for i in $(seq 1 30); do
  if curl -sf http://localhost:PORT/health >/dev/null 2>&1; then
    SERVER_READY=1
    break
  fi
  sleep 1
done

if [ "$SERVER_READY" -ne 1 ]; then
  kill "$SERVER_PID" 2>/dev/null || true
  echo "ERROR: Server failed to start within 30s" >&2
  exit 1
fi

[integration test command]
TEST_EXIT=$?

kill "$SERVER_PID" 2>/dev/null || true
wait "$SERVER_PID" 2>/dev/null || true

exit "$TEST_EXIT"
```

### Level 5: Edge Case Testing
### 级别 5：边界情况测试

Run through edge cases from the plan's Testing Strategy checklist.
执行计划测试策略检查清单中的边界情况。

**CHECKPOINT**: All 5 validation levels pass. Zero errors.
**检查点**：所有 5 个验证级别通过。零错误。

---

## Phase 5 — REPORT
## 阶段 5 — 报告

### Create Implementation Report
### 创建实现报告

```bash
mkdir -p .claude/PRPs/reports
```

Write report to `.claude/PRPs/reports/{plan-name}-report.md`:
将报告写入 `.claude/PRPs/reports/{plan-name}-report.md`：

```markdown
# Implementation Report: [Feature Name]

## Summary
[What was implemented]

## Assessment vs Reality

| Metric | Predicted (Plan) | Actual |
|---|---|---|
| Complexity | [from plan] | [actual] |
| Confidence | [from plan] | [actual] |
| Files Changed | [from plan] | [actual count] |

## Tasks Completed

| # | Task | Status | Notes |
|---|---|---|---|
| 1 | [task name] | [done] Complete | |
| 2 | [task name] | [done] Complete | Deviated — [reason] |

## Validation Results

| Level | Status | Notes |
|---|---|---|
| Static Analysis | [done] Pass | |
| Unit Tests | [done] Pass | N tests written |
| Build | [done] Pass | |
| Integration | [done] Pass | or N/A |
| Edge Cases | [done] Pass | |

## Files Changed

| File | Action | Lines |
|---|---|---|
| `path/to/file` | CREATED | +N |
| `path/to/file` | UPDATED | +N / -M |

## Deviations from Plan
[List any deviations with WHAT and WHY, or "None"]

## Issues Encountered
[List any problems and how they were resolved, or "None"]

## Tests Written

| Test File | Tests | Coverage |
|---|---|---|
| `path/to/test` | N tests | [area covered] |

## Next Steps
- [ ] Code review via `/code-review`
- [ ] Create PR via `/prp-pr`
```

### Update PRD (if applicable)
### 更新 PRD（如适用）

If this implementation was for a PRD phase:
若此实现属于某个 PRD 阶段：
1. Update the phase status from `in-progress` to `complete`
1. 将阶段状态从 `in-progress` 更新为 `complete`
2. Add report path as reference
2. 添加报告路径作为参考

### Archive Plan
### 归档计划

```bash
mkdir -p .claude/PRPs/plans/completed
mv "$ARGUMENTS" .claude/PRPs/plans/completed/
```

**CHECKPOINT**: Report created. PRD updated. Plan archived.
**检查点**：报告已创建。PRD 已更新。计划已归档。

---

## Phase 6 — OUTPUT
## 阶段 6 — 输出

Report to user:
向用户报告：

```
## Implementation Complete

- **Plan**: [plan file path] → archived to completed/
- **Branch**: [current branch name]
- **Status**: [done] All tasks complete

### Validation Summary

| Check | Status |
|---|---|
| Type Check | [done] |
| Lint | [done] |
| Tests | [done] (N written) |
| Build | [done] |
| Integration | [done] or N/A |

### Files Changed
- [N] files created, [M] files updated

### Deviations
[Summary or "None — implemented exactly as planned"]

### Artifacts
- Report: `.claude/PRPs/reports/{name}-report.md`
- Archived Plan: `.claude/PRPs/plans/completed/{name}.plan.md`

### PRD Progress (if applicable)
| Phase | Status |
|---|---|
| Phase 1 | [done] Complete |
| Phase 2 | [next] |
| ... | ... |

> Next step: Run `/prp-pr` to create a pull request, or `/code-review` to review changes first.
```

---

## Handling Failures
## 失败处理

### Type Check Fails
### 类型检查失败

1. Read the error message carefully
1. 仔细阅读错误消息
2. Fix the type error in the source file
2. 在源文件中修复类型错误
3. Re-run type-check
3. 重新运行类型检查
4. Continue only when clean
4. 仅在检查通过后继续

### Tests Fail
### 测试失败

1. Identify whether the bug is in the implementation or the test
1. 判断 bug 在实现还是测试中
2. Fix the root cause (usually the implementation)
2. 修复根本原因（通常是实现问题）
3. Re-run tests
3. 重新运行测试
4. Continue only when green
4. 仅在测试全部通过后继续

### Lint Fails
### Lint 失败

1. Run auto-fix first
1. 先运行自动修复
2. If errors remain, fix manually
2. 若仍有错误，手动修复
3. Re-run lint
3. 重新运行 lint
4. Continue only when clean
4. 仅在检查通过后继续

### Build Fails
### 构建失败

1. Usually a type or import issue — check error message
1. 通常是类型或导入问题——检查错误消息
2. Fix the offending file
2. 修复有问题的文件
3. Re-run build
3. 重新运行构建
4. Continue only when successful
4. 仅在构建成功后继续

### Integration Test Fails
### 集成测试失败

1. Check server started correctly
1. 检查服务器是否正确启动
2. Verify endpoint/route exists
2. 验证端点/路由是否存在
3. Check request format matches expected
3. 检查请求格式是否符合预期
4. Fix and re-run
4. 修复后重新运行

---

## Success Criteria
## 成功标准

- **TASKS_COMPLETE**: All tasks from the plan executed
- **TASKS_COMPLETE**：计划中的所有任务已执行
- **TYPES_PASS**: Zero type errors
- **TYPES_PASS**：零类型错误
- **LINT_PASS**: Zero lint errors
- **LINT_PASS**：零 lint 错误
- **TESTS_PASS**: All tests green, new tests written
- **TESTS_PASS**：所有测试通过，新测试已编写
- **BUILD_PASS**: Build succeeds
- **BUILD_PASS**：构建成功
- **REPORT_CREATED**: Implementation report saved
- **REPORT_CREATED**：实现报告已保存
- **PLAN_ARCHIVED**: Plan moved to `completed/`
- **PLAN_ARCHIVED**：计划已移至 `completed/`

---

## Next Steps
## 后续步骤

- Run `/code-review` to review changes before committing
- 运行 `/code-review` 在提交前审查变更
- Run `/prp-commit` to commit with a descriptive message
- 运行 `/prp-commit` 以描述性消息提交
- Run `/prp-pr` to create a pull request
- 运行 `/prp-pr` 创建 Pull Request
- Run `/prp-plan <next-phase>` if the PRD has more phases
- 若 PRD 还有更多阶段，运行 `/prp-plan <next-phase>`
