---
description: Quick commit with natural language file targeting — describe what to commit in plain English
  用自然语言描述要提交的内容，快速精准暂存并提交
argument-hint: [target description] (blank = all changes)
  [目标描述]（留空 = 所有变更）
---

# Smart Commit
# 智能提交

> Adapted from PRPs-agentic-eng by Wirasm. Part of the PRP workflow series.
> 改编自 Wirasm 的 PRPs-agentic-eng，属于 PRP 工作流系列。

**Input**: $ARGUMENTS
**输入**：$ARGUMENTS

---

## Phase 1 — ASSESS
## 阶段 1 — 评估

```bash
git status --short
```

If output is empty → stop: "Nothing to commit."
若输出为空 → 停止："Nothing to commit."

Show the user a summary of what's changed (added, modified, deleted, untracked).
向用户展示变更摘要（已添加、已修改、已删除、未跟踪）。

---

## Phase 2 — INTERPRET & STAGE
## 阶段 2 — 解析并暂存

Interpret `$ARGUMENTS` to determine what to stage:
解析 `$ARGUMENTS` 以确定要暂存的内容：

| Input | Interpretation | Git Command |
|---|---|---|
| 输入 | 解释 | Git 命令 |
| *(blank / empty)* | Stage everything | `git add -A` |
| *（空白/空）* | 暂存所有内容 | `git add -A` |
| `staged` | Use whatever is already staged | *(no git add)* |
| `staged` | 使用已暂存的内容 | *（不执行 git add）* |
| `*.ts` or `*.py` etc. | Stage matching glob | `git add '*.ts'` |
| `*.ts` or `*.py` etc. | 暂存匹配的 glob 文件 | `git add '*.ts'` |
| `except tests` | Stage all, then unstage tests | `git add -A && git reset -- '**/*.test.*' '**/*.spec.*' '**/test_*' 2>/dev/null \|\| true` |
| `except tests` | 暂存所有，然后取消暂存测试文件 | `git add -A && git reset -- '**/*.test.*' '**/*.spec.*' '**/test_*' 2>/dev/null \|\| true` |
| `only new files` | Stage untracked files only | `git ls-files --others --exclude-standard \| grep . && git ls-files --others --exclude-standard \| xargs git add` |
| `only new files` | 仅暂存未跟踪文件 | `git ls-files --others --exclude-standard \| grep . && git ls-files --others --exclude-standard \| xargs git add` |
| `the auth changes` | Interpret from status/diff — find auth-related files | `git add <matched files>` |
| `the auth changes` | 从 status/diff 解析——查找认证相关文件 | `git add <matched files>` |
| Specific filenames | Stage those files | `git add <files>` |
| 指定文件名 | 暂存这些文件 | `git add <files>` |

For natural language inputs (like "the auth changes"), cross-reference the `git status` output and `git diff` to identify relevant files. Show the user which files you're staging and why.
对于自然语言输入（如"the auth changes"），交叉参考 `git status` 输出和 `git diff` 以识别相关文件，并向用户说明暂存了哪些文件及原因。

```bash
git add <determined files>
```

After staging, verify:
暂存后，验证：
```bash
git diff --cached --stat
```

If nothing staged, stop: "No files matched your description."
若无内容暂存，停止："No files matched your description."

---

## Phase 3 — COMMIT
## 阶段 3 — 提交

Craft a single-line commit message in imperative mood:
以祈使语气编写单行提交消息：

```
{type}: {description}
```

Types:
类型：
- `feat` — New feature or capability
- `feat` — 新功能或新能力
- `fix` — Bug fix
- `fix` — Bug 修复
- `refactor` — Code restructuring without behavior change
- `refactor` — 不改变行为的代码重构
- `docs` — Documentation changes
- `docs` — 文档变更
- `test` — Adding or updating tests
- `test` — 添加或更新测试
- `chore` — Build, config, dependencies
- `chore` — 构建、配置、依赖
- `perf` — Performance improvement
- `perf` — 性能优化
- `ci` — CI/CD changes
- `ci` — CI/CD 变更

Rules:
规则：
- Imperative mood ("add feature" not "added feature")
- 祈使语气（"add feature" 而非 "added feature"）
- Lowercase after the type prefix
- 类型前缀后使用小写
- No period at the end
- 末尾不加句号
- Under 72 characters
- 不超过 72 个字符
- Describe WHAT changed, not HOW
- 描述变更了**什么**，而非**怎么**变更的

```bash
git commit -m "{type}: {description}"
```

---

## Phase 4 — OUTPUT
## 阶段 4 — 输出

Report to user:
向用户报告：

```
Committed: {hash_short}
Message:   {type}: {description}
Files:     {count} file(s) changed

Next steps:
  - git push           → push to remote
  - /prp-pr            → create a pull request
  - /code-review       → review before pushing
```

---

## Examples
## 示例

| You say | What happens |
|---|---|
| 你输入 | 执行效果 |
| `/prp-commit` | Stages all, auto-generates message |
| `/prp-commit` | 暂存所有内容，自动生成提交消息 |
| `/prp-commit staged` | Commits only what's already staged |
| `/prp-commit staged` | 仅提交已暂存的内容 |
| `/prp-commit *.ts` | Stages all TypeScript files, commits |
| `/prp-commit *.ts` | 暂存所有 TypeScript 文件并提交 |
| `/prp-commit except tests` | Stages everything except test files |
| `/prp-commit except tests` | 暂存除测试文件外的所有内容 |
| `/prp-commit the database migration` | Finds DB migration files from status, stages them |
| `/prp-commit the database migration` | 从 status 中查找数据库迁移文件并暂存 |
| `/prp-commit only new files` | Stages untracked files only |
| `/prp-commit only new files` | 仅暂存未跟踪文件 |
