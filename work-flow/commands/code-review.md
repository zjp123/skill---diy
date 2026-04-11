---
description: Code review — local uncommitted changes or GitHub PR (pass PR number/URL for PR mode)
  代码审查 — 本地未提交的更改或 GitHub PR（PR 模式时传入 PR 编号/URL）
argument-hint: [pr-number | pr-url | blank for local review]
  [pr-编号 | pr-url | 本地审查时留空]
---

# Code Review
# 代码审查

> PR review mode adapted from PRPs-agentic-eng by Wirasm. Part of the PRP workflow series.
> PR 审查模式改编自 Wirasm 的 PRPs-agentic-eng。PRP 工作流系列的一部分。

**Input**: $ARGUMENTS

---

## Mode Selection
## 模式选择

If `$ARGUMENTS` contains a PR number, PR URL, or `--pr`:
若 `$ARGUMENTS` 包含 PR 编号、PR URL 或 `--pr`：
→ Jump to **PR Review Mode** below.
→ 跳转到下方的 **PR 审查模式**。

Otherwise:
否则：
→ Use **Local Review Mode**.
→ 使用**本地审查模式**。

---

## Local Review Mode
## 本地审查模式

Comprehensive security and quality review of uncommitted changes.
对未提交更改进行全面的安全性和质量审查。

### Phase 1 — GATHER
### 阶段 1 — 收集

```bash
git diff --name-only HEAD
```

If no changed files, stop: "Nothing to review."
若无变更文件，停止："Nothing to review."

### Phase 2 — REVIEW
### 阶段 2 — 审查

Read each changed file in full. Check for:
完整读取每个变更文件。检查以下内容：

**Security Issues (CRITICAL):**
**安全问题（关键）：**
- Hardcoded credentials, API keys, tokens
- 硬编码凭据、API 密钥、令牌
- SQL injection vulnerabilities
- SQL 注入漏洞
- XSS vulnerabilities
- XSS 漏洞
- Missing input validation
- 缺少输入验证
- Insecure dependencies
- 不安全的依赖
- Path traversal risks
- 路径遍历风险

**Code Quality (HIGH):**
**代码质量（高）：**
- Functions > 50 lines
- 函数 > 50 行
- Files > 800 lines
- 文件 > 800 行
- Nesting depth > 4 levels
- 嵌套深度 > 4 层
- Missing error handling
- 缺少错误处理
- console.log statements
- console.log 语句
- TODO/FIXME comments
- TODO/FIXME 注释
- Missing JSDoc for public APIs
- 公共 API 缺少 JSDoc

**Best Practices (MEDIUM):**
**最佳实践（中）：**
- Mutation patterns (use immutable instead)
- 变更模式（改用不可变模式）
- Emoji usage in code/comments
- 代码/注释中使用 emoji
- Missing tests for new code
- 新代码缺少测试
- Accessibility issues (a11y)
- 无障碍访问问题（a11y）

### Phase 3 — REPORT
### 阶段 3 — 报告

Generate report with:
生成报告，包含：
- Severity: CRITICAL, HIGH, MEDIUM, LOW
- 严重程度：CRITICAL、HIGH、MEDIUM、LOW
- File location and line numbers
- 文件位置和行号
- Issue description
- 问题描述
- Suggested fix
- 建议修复方案

Block commit if CRITICAL or HIGH issues found.
若发现 CRITICAL 或 HIGH 问题，阻止提交。
Never approve code with security vulnerabilities.
永远不要批准含安全漏洞的代码。

---

## PR Review Mode
## PR 审查模式

Comprehensive GitHub PR review — fetches diff, reads full files, runs validation, posts review.
全面的 GitHub PR 审查——获取差异、读取完整文件、运行验证、发布审查。

### Phase 1 — FETCH
### 阶段 1 — 获取

Parse input to determine PR:
解析输入以确定 PR：

| Input | Action |
|---|---|
| 输入 | 操作 |
| Number (e.g. `42`) | Use as PR number |
| 数字（如 `42`） | 作为 PR 编号使用 |
| URL (`github.com/.../pull/42`) | Extract PR number |
| URL（`github.com/.../pull/42`） | 提取 PR 编号 |
| Branch name | Find PR via `gh pr list --head <branch>` |
| 分支名称 | 通过 `gh pr list --head <branch>` 查找 PR |

```bash
gh pr view <NUMBER> --json number,title,body,author,baseRefName,headRefName,changedFiles,additions,deletions
gh pr diff <NUMBER>
```

If PR not found, stop with error. Store PR metadata for later phases.
若未找到 PR，停止并报错。保存 PR 元数据以供后续阶段使用。

### Phase 2 — CONTEXT
### 阶段 2 — 上下文

Build review context:
构建审查上下文：

1. **Project rules** — Read `CLAUDE.md`, `.claude/docs/`, and any contributing guidelines
1. **项目规则** — 读取 `CLAUDE.md`、`.claude/docs/` 及任何贡献指南
2. **PRP artifacts** — Check `.claude/PRPs/reports/` and `.claude/PRPs/plans/` for implementation context related to this PR
2. **PRP 产出物** — 检查 `.claude/PRPs/reports/` 和 `.claude/PRPs/plans/` 以获取与此 PR 相关的实现上下文
3. **PR intent** — Parse PR description for goals, linked issues, test plans
3. **PR 意图** — 解析 PR 描述以了解目标、关联 issue、测试计划
4. **Changed files** — List all modified files and categorize by type (source, test, config, docs)
4. **变更文件** — 列出所有修改的文件，并按类型分类（源码、测试、配置、文档）

### Phase 3 — REVIEW
### 阶段 3 — 审查

Read each changed file **in full** (not just the diff hunks — you need surrounding context).
完整读取每个变更文件（不仅仅是差异块——你需要周围的上下文）。

For PR reviews, fetch the full file contents at the PR head revision:
对于 PR 审查，在 PR head 版本获取完整文件内容：
```bash
gh pr diff <NUMBER> --name-only | while IFS= read -r file; do
  gh api "repos/{owner}/{repo}/contents/$file?ref=<head-branch>" --jq '.content' | base64 -d
done
```

Apply the review checklist across 7 categories:
按 7 个类别应用审查检查清单：

| Category | What to Check |
|---|---|
| 类别 | 检查内容 |
| **Correctness** | Logic errors, off-by-ones, null handling, edge cases, race conditions |
| **正确性** | 逻辑错误、差一错误、空值处理、边界情况、竞争条件 |
| **Type Safety** | Type mismatches, unsafe casts, `any` usage, missing generics |
| **类型安全** | 类型不匹配、不安全转换、`any` 使用、缺少泛型 |
| **Pattern Compliance** | Matches project conventions (naming, file structure, error handling, imports) |
| **模式合规性** | 符合项目约定（命名、文件结构、错误处理、导入） |
| **Security** | Injection, auth gaps, secret exposure, SSRF, path traversal, XSS |
| **安全性** | 注入、认证漏洞、密钥暴露、SSRF、路径遍历、XSS |
| **Performance** | N+1 queries, missing indexes, unbounded loops, memory leaks, large payloads |
| **性能** | N+1 查询、缺少索引、无界循环、内存泄漏、大载荷 |
| **Completeness** | Missing tests, missing error handling, incomplete migrations, missing docs |
| **完整性** | 缺少测试、缺少错误处理、不完整迁移、缺少文档 |
| **Maintainability** | Dead code, magic numbers, deep nesting, unclear naming, missing types |
| **可维护性** | 死代码、魔法数字、深层嵌套、不清晰命名、缺少类型 |

Assign severity to each finding:
对每个发现分配严重程度：

| Severity | Meaning | Action |
|---|---|---|
| 严重程度 | 含义 | 操作 |
| **CRITICAL** | Security vulnerability or data loss risk | Must fix before merge |
| **关键** | 安全漏洞或数据丢失风险 | 合并前必须修复 |
| **HIGH** | Bug or logic error likely to cause issues | Should fix before merge |
| **高** | 可能导致问题的 Bug 或逻辑错误 | 合并前应修复 |
| **MEDIUM** | Code quality issue or missing best practice | Fix recommended |
| **中** | 代码质量问题或缺少最佳实践 | 建议修复 |
| **LOW** | Style nit or minor suggestion | Optional |
| **低** | 风格细节或次要建议 | 可选 |

### Phase 4 — VALIDATE
### 阶段 4 — 验证

Run available validation commands:
运行可用的验证命令：

Detect the project type from config files (`package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, etc.), then run the appropriate commands:
从配置文件（`package.json`、`Cargo.toml`、`go.mod`、`pyproject.toml` 等）检测项目类型，然后运行相应的命令：

**Node.js / TypeScript** (has `package.json`):
**Node.js / TypeScript**（含 `package.json`）：
```bash
npm run typecheck 2>/dev/null || npx tsc --noEmit 2>/dev/null  # Type check
npm run lint                                                    # Lint
npm test                                                        # Tests
npm run build                                                   # Build
```

**Rust** (has `Cargo.toml`):
**Rust**（含 `Cargo.toml`）：
```bash
cargo clippy -- -D warnings  # Lint
cargo test                   # Tests
cargo build                  # Build
```

**Go** (has `go.mod`):
**Go**（含 `go.mod`）：
```bash
go vet ./...    # Lint
go test ./...   # Tests
go build ./...  # Build
```

**Python** (has `pyproject.toml` / `setup.py`):
**Python**（含 `pyproject.toml` / `setup.py`）：
```bash
pytest  # Tests
```

Run only the commands that apply to the detected project type. Record pass/fail for each.
仅运行适用于检测到的项目类型的命令。记录每项的通过/失败状态。

### Phase 5 — DECIDE
### 阶段 5 — 决策

Form recommendation based on findings:
根据发现形成建议：

| Condition | Decision |
|---|---|
| 条件 | 决策 |
| Zero CRITICAL/HIGH issues, validation passes | **APPROVE** |
| 无 CRITICAL/HIGH 问题，验证通过 | **批准** |
| Only MEDIUM/LOW issues, validation passes | **APPROVE** with comments |
| 仅有 MEDIUM/LOW 问题，验证通过 | **带评论批准** |
| Any HIGH issues or validation failures | **REQUEST CHANGES** |
| 有任何 HIGH 问题或验证失败 | **请求变更** |
| Any CRITICAL issues | **BLOCK** — must fix before merge |
| 有任何 CRITICAL 问题 | **阻止** — 合并前必须修复 |

Special cases:
特殊情况：
- Draft PR → Always use **COMMENT** (not approve/block)
- 草稿 PR → 始终使用 **COMMENT**（不批准/阻止）
- Only docs/config changes → Lighter review, focus on correctness
- 仅有文档/配置变更 → 较轻量的审查，专注于正确性
- Explicit `--approve` or `--request-changes` flag → Override decision (but still report all findings)
- 显式 `--approve` 或 `--request-changes` 标志 → 覆盖决策（但仍报告所有发现）

### Phase 6 — REPORT
### 阶段 6 — 报告

Create review artifact at `.claude/PRPs/reviews/pr-<NUMBER>-review.md`:
在 `.claude/PRPs/reviews/pr-<NUMBER>-review.md` 创建审查产出物：

```markdown
# PR Review: #<NUMBER> — <TITLE>

**Reviewed**: <date>
**Author**: <author>
**Branch**: <head> → <base>
**Decision**: APPROVE | REQUEST CHANGES | BLOCK

## Summary
<1-2 sentence overall assessment>

## Findings

### CRITICAL
<findings or "None">

### HIGH
<findings or "None">

### MEDIUM
<findings or "None">

### LOW
<findings or "None">

## Validation Results

| Check | Result |
|---|---|
| Type check | Pass / Fail / Skipped |
| Lint | Pass / Fail / Skipped |
| Tests | Pass / Fail / Skipped |
| Build | Pass / Fail / Skipped |

## Files Reviewed
<list of files with change type: Added/Modified/Deleted>
```

### Phase 7 — PUBLISH
### 阶段 7 — 发布

Post the review to GitHub:
将审查发布到 GitHub：

```bash
# If APPROVE
gh pr review <NUMBER> --approve --body "<summary of review>"

# If REQUEST CHANGES
gh pr review <NUMBER> --request-changes --body "<summary with required fixes>"

# If COMMENT only (draft PR or informational)
gh pr review <NUMBER> --comment --body "<summary>"
```

For inline comments on specific lines, use the GitHub review comments API:
对于特定行的行内评论，使用 GitHub 审查评论 API：
```bash
gh api "repos/{owner}/{repo}/pulls/<NUMBER>/comments" \
  -f body="<comment>" \
  -f path="<file>" \
  -F line=<line-number> \
  -f side="RIGHT" \
  -f commit_id="$(gh pr view <NUMBER> --json headRefOid --jq .headRefOid)"
```

Alternatively, post a single review with multiple inline comments at once:
或者，一次性发布包含多条行内评论的单一审查：
```bash
gh api "repos/{owner}/{repo}/pulls/<NUMBER>/reviews" \
  -f event="COMMENT" \
  -f body="<overall summary>" \
  --input comments.json  # [{"path": "file", "line": N, "body": "comment"}, ...]
```

### Phase 8 — OUTPUT
### 阶段 8 — 输出

Report to user:
向用户报告：

```
PR #<NUMBER>: <TITLE>
Decision: <APPROVE|REQUEST_CHANGES|BLOCK>

Issues: <critical_count> critical, <high_count> high, <medium_count> medium, <low_count> low
Validation: <pass_count>/<total_count> checks passed

Artifacts:
  Review: .claude/PRPs/reviews/pr-<NUMBER>-review.md
  GitHub: <PR URL>

Next steps:
  - <contextual suggestions based on decision>
```

---

## Edge Cases
## 边缘情况

- **No `gh` CLI**: Fall back to local-only review (read the diff, skip GitHub publish). Warn user.
- **无 `gh` CLI**：回退到纯本地审查（读取差异，跳过 GitHub 发布）。警告用户。
- **Diverged branches**: Suggest `git fetch origin && git rebase origin/<base>` before review.
- **分支已分叉**：审查前建议执行 `git fetch origin && git rebase origin/<base>`。
- **Large PRs (>50 files)**: Warn about review scope. Focus on source changes first, then tests, then config/docs.
- **大型 PR（>50 个文件）**：警告审查范围。先关注源码变更，然后是测试，再是配置/文档。
