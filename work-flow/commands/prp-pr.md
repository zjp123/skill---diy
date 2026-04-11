---
description: Create a GitHub PR from current branch with unpushed commits — discovers templates, analyzes changes, pushes
  从当前分支创建 GitHub PR——自动发现模板、分析变更并推送
argument-hint: [base-branch] (default: main)
  [基础分支]（默认：main）
---

# Create Pull Request
# 创建 Pull Request

> Adapted from PRPs-agentic-eng by Wirasm. Part of the PRP workflow series.
> 改编自 Wirasm 的 PRPs-agentic-eng，属于 PRP 工作流系列。

**Input**: `$ARGUMENTS` — optional, may contain a base branch name and/or flags (e.g., `--draft`).
**输入**：`$ARGUMENTS` — 可选，可包含基础分支名称和/或标志（如 `--draft`）。

**Parse `$ARGUMENTS`**:
**解析 `$ARGUMENTS`**：
- Extract any recognized flags (`--draft`)
- 提取已识别的标志（`--draft`）
- Treat remaining non-flag text as the base branch name
- 将剩余非标志文本视为基础分支名称
- Default base branch to `main` if none specified
- 若未指定，默认基础分支为 `main`

---

## Phase 1 — VALIDATE
## 阶段 1 — 验证

Check preconditions:
检查前提条件：

```bash
git branch --show-current
git status --short
git log origin/<base>..HEAD --oneline
```

| Check | Condition | Action if Failed |
|---|---|---|
| 检查项 | 条件 | 失败时的操作 |
| Not on base branch | Current branch ≠ base | Stop: "Switch to a feature branch first." |
| 非基础分支 | 当前分支 ≠ 基础分支 | 停止："Switch to a feature branch first." |
| Clean working directory | No uncommitted changes | Warn: "You have uncommitted changes. Commit or stash first. Use `/prp-commit` to commit." |
| 工作目录干净 | 无未提交的变更 | 警告："You have uncommitted changes. Commit or stash first. Use `/prp-commit` to commit." |
| Has commits ahead | `git log origin/<base>..HEAD` not empty | Stop: "No commits ahead of `<base>`. Nothing to PR." |
| 有领先提交 | `git log origin/<base>..HEAD` 不为空 | 停止："No commits ahead of `<base>`. Nothing to PR." |
| No existing PR | `gh pr list --head <branch> --json number` is empty | Stop: "PR already exists: #<number>. Use `gh pr view <number> --web` to open it." |
| 无已有 PR | `gh pr list --head <branch> --json number` 为空 | 停止："PR already exists: #<number>. Use `gh pr view <number> --web` to open it." |

If all checks pass, proceed.
若所有检查通过，继续执行。

---

## Phase 2 — DISCOVER
## 阶段 2 — 发现

### PR Template
### PR 模板

Search for PR template in order:
按顺序搜索 PR 模板：

1. `.github/PULL_REQUEST_TEMPLATE/` directory — if exists, list files and let user choose (or use `default.md`)
1. `.github/PULL_REQUEST_TEMPLATE/` 目录——若存在，列出文件让用户选择（或使用 `default.md`）
2. `.github/PULL_REQUEST_TEMPLATE.md`
2. `.github/PULL_REQUEST_TEMPLATE.md`
3. `.github/pull_request_template.md`
3. `.github/pull_request_template.md`
4. `docs/pull_request_template.md`
4. `docs/pull_request_template.md`

If found, read it and use its structure for the PR body.
若找到，读取并将其结构用于 PR 正文。

### Commit Analysis
### 提交分析

```bash
git log origin/<base>..HEAD --format="%h %s" --reverse
```

Analyze commits to determine:
分析提交以确定：
- **PR title**: Use conventional commit format with type prefix — `feat: ...`, `fix: ...`, etc.
- **PR 标题**：使用约定式提交格式加类型前缀——`feat: ...`、`fix: ...` 等
  - If multiple types, use the dominant one
  - 若有多种类型，使用占主导的一种
  - If single commit, use its message as-is
  - 若只有一次提交，直接使用该提交消息
- **Change summary**: Group commits by type/area
- **变更摘要**：按类型/区域对提交分组

### File Analysis
### 文件分析

```bash
git diff origin/<base>..HEAD --stat
git diff origin/<base>..HEAD --name-only
```

Categorize changed files: source, tests, docs, config, migrations.
对变更文件分类：源码、测试、文档、配置、迁移。

### PRP Artifacts
### PRP 产出物

Check for related PRP artifacts:
检查相关 PRP 产出物：
- `.claude/PRPs/reports/` — Implementation reports
- `.claude/PRPs/reports/` — 实现报告
- `.claude/PRPs/plans/` — Plans that were executed
- `.claude/PRPs/plans/` — 已执行的计划
- `.claude/PRPs/prds/` — Related PRDs
- `.claude/PRPs/prds/` — 相关 PRD

Reference these in the PR body if they exist.
若存在，在 PR 正文中引用这些文件。

---

## Phase 3 — PUSH
## 阶段 3 — 推送

```bash
git push -u origin HEAD
```

If push fails due to divergence:
若因分歧导致推送失败：
```bash
git fetch origin
git rebase origin/<base>
git push -u origin HEAD
```

If rebase conflicts occur, stop and inform the user.
若发生 rebase 冲突，停止并告知用户。

---

## Phase 4 — CREATE
## 阶段 4 — 创建

### With Template
### 使用模板

If a PR template was found in Phase 2, fill in each section using the commit and file analysis. Preserve all template sections — leave sections as "N/A" if not applicable rather than removing them.
若在阶段 2 中找到 PR 模板，使用提交和文件分析填写每个部分。保留所有模板部分——不适用的部分标记为"N/A"，而非删除。

### Without Template
### 无模板时

Use this default format:
使用以下默认格式：

```markdown
## Summary

<1-2 sentence description of what this PR does and why>

## Changes

<bulleted list of changes grouped by area>

## Files Changed

<table or list of changed files with change type: Added/Modified/Deleted>

## Testing

<description of how changes were tested, or "Needs testing">

## Related Issues

<linked issues with Closes/Fixes/Relates to #N, or "None">
```

### Create the PR
### 创建 PR

```bash
gh pr create \
  --title "<PR title>" \
  --base <base-branch> \
  --body "<PR body>"
  # Add --draft if the --draft flag was parsed from $ARGUMENTS
```

---

## Phase 5 — VERIFY
## 阶段 5 — 验证

```bash
gh pr view --json number,url,title,state,baseRefName,headRefName,additions,deletions,changedFiles
gh pr checks --json name,status,conclusion 2>/dev/null || true
```

---

## Phase 6 — OUTPUT
## 阶段 6 — 输出

Report to user:
向用户报告：

```
PR #<number>: <title>
URL: <url>
Branch: <head> → <base>
Changes: +<additions> -<deletions> across <changedFiles> files

CI Checks: <status summary or "pending" or "none configured">

Artifacts referenced:
  - <any PRP reports/plans linked in PR body>

Next steps:
  - gh pr view <number> --web   → open in browser
  - /code-review <number>       → review the PR
  - gh pr merge <number>        → merge when ready
```

---

## Edge Cases
## 边界情况

- **No `gh` CLI**: Stop with: "GitHub CLI (`gh`) is required. Install: <https://cli.github.com/>"
- **无 `gh` CLI**：停止并提示："GitHub CLI (`gh`) is required. Install: <https://cli.github.com/>"
- **Not authenticated**: Stop with: "Run `gh auth login` first."
- **未认证**：停止并提示："Run `gh auth login` first."
- **Force push needed**: If remote has diverged and rebase was done, use `git push --force-with-lease` (never `--force`).
- **需要强制推送**：若远程已分歧且已执行 rebase，使用 `git push --force-with-lease`（绝不用 `--force`）。
- **Multiple PR templates**: If `.github/PULL_REQUEST_TEMPLATE/` has multiple files, list them and ask user to choose.
- **多个 PR 模板**：若 `.github/PULL_REQUEST_TEMPLATE/` 中有多个文件，列出并让用户选择。
- **Large PR (>20 files)**: Warn about PR size. Suggest splitting if changes are logically separable.
- **大型 PR（>20 个文件）**：警告 PR 体量过大。若变更在逻辑上可分离，建议拆分。
