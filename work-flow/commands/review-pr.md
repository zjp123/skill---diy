---
description: Comprehensive PR review using specialized agents
  使用专业代理进行全面的 PR 审查
---

Run a comprehensive multi-perspective review of a pull request.
对 Pull Request 进行全面的多视角审查。

## Usage
## 使用方法

`/review-pr [PR-number-or-URL] [--focus=comments|tests|errors|types|code|simplify]`

If no PR is specified, review the current branch's PR. If no focus is specified, run the full review stack.
若未指定 PR，则审查当前分支的 PR。若未指定焦点，则运行完整审查栈。

## Steps
## 步骤

1. Identify the PR:
1. 识别 PR：
   - use `gh pr view` to get PR details, changed files, and diff
   - 使用 `gh pr view` 获取 PR 详情、变更文件和差异
2. Find project guidance:
2. 查找项目指导：
   - look for `CLAUDE.md`, lint config, TypeScript config, repo conventions
   - 查找 `CLAUDE.md`、lint 配置、TypeScript 配置、仓库规范
3. Run specialized review agents:
3. 运行专业审查代理：
   - `code-reviewer`
   - `code-reviewer`
   - `comment-analyzer`
   - `comment-analyzer`
   - `pr-test-analyzer`
   - `pr-test-analyzer`
   - `silent-failure-hunter`
   - `silent-failure-hunter`
   - `type-design-analyzer`
   - `type-design-analyzer`
   - `code-simplifier`
   - `code-simplifier`
4. Aggregate results:
4. 汇总结果：
   - dedupe overlapping findings
   - 去重重叠发现
   - rank by severity
   - 按严重程度排序
5. Report findings grouped by severity
5. 按严重程度分组报告发现

## Confidence Rule
## 置信度规则

Only report issues with confidence >= 80:
仅报告置信度 >= 80 的问题：

- Critical: bugs, security, data loss
- 关键：错误、安全、数据丢失
- Important: missing tests, quality problems, style violations
- 重要：缺少测试、质量问题、风格违规
- Advisory: suggestions only when explicitly requested
- 建议性：仅在明确请求时提供建议
