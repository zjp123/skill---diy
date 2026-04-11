---
description: Legacy slash-entry shim for dmux-workflows and autonomous-agent-harness. Prefer the skills directly.
  Legacy slash-entry shim — dmux-workflows 和 autonomous-agent-harness 的兼容入口。建议直接使用技能。
---

# Orchestrate Command (Legacy Shim)
# Orchestrate 命令（旧版兼容入口）

Use this only if you still invoke `/orchestrate`. The maintained orchestration guidance lives in `skills/dmux-workflows/SKILL.md` and `skills/autonomous-agent-harness/SKILL.md`.
仅在仍使用 `/orchestrate` 调用时使用此文件。维护中的编排指南位于 `skills/dmux-workflows/SKILL.md` 和 `skills/autonomous-agent-harness/SKILL.md`。

## Canonical Surface
## 标准入口

- Prefer `dmux-workflows` for parallel panes, worktrees, and multi-agent splits.
- 并行面板、工作树和多代理拆分，优先使用 `dmux-workflows`。
- Prefer `autonomous-agent-harness` for longer-running loops, governance, scheduling, and control-plane style execution.
- 长时间运行循环、治理、调度和控制平面风格执行，优先使用 `autonomous-agent-harness`。
- Keep this file only as a compatibility entry point.
- 此文件仅作为兼容性入口保留。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the orchestration skills instead of maintaining a second workflow spec here.
使用编排技能，而非在此维护第二套工作流规范。
- Start with `dmux-workflows` for split/parallel execution.
- 拆分/并行执行从 `dmux-workflows` 开始。
- Pull in `autonomous-agent-harness` when the user is really asking for persistent loops, governance, or operator-layer behavior.
- 当用户真正需要持久循环、治理或操作层行为时，引入 `autonomous-agent-harness`。
- Keep handoffs structured, but let the skills define the maintained sequencing rules.
- 保持交接结构化，但由技能定义维护中的排序规则。
Security Reviewer: [summary]
安全审查员：[摘要]

### FILES CHANGED
### 已变更文件

[List all files modified]
[列出所有已修改文件]

### TEST RESULTS
### 测试结果

[Test pass/fail summary]
[测试通过/失败摘要]

### SECURITY STATUS
### 安全状态

[Security findings]
[安全发现]

### RECOMMENDATION
### 建议

[SHIP / NEEDS WORK / BLOCKED]
[发布 / 需要改进 / 已阻塞]
```

## Parallel Execution
## 并行执行

For independent checks, run agents in parallel:
对于独立检查，并行运行代理：

```markdown
### Parallel Phase
Run simultaneously:
- code-reviewer (quality)
- security-reviewer (security)
- architect (design)

### Merge Results
Combine outputs into single report
```

For external tmux-pane workers with separate git worktrees, use `node scripts/orchestrate-worktrees.js plan.json --execute`. The built-in orchestration pattern stays in-process; the helper is for long-running or cross-harness sessions.
对于拥有独立 git 工作树的外部 tmux 面板工作者，使用 `node scripts/orchestrate-worktrees.js plan.json --execute`。内置编排模式保持进程内运行；该辅助工具用于长时间运行或跨 harness 会话。

When workers need to see dirty or untracked local files from the main checkout, add `seedPaths` to the plan file. ECC overlays only those selected paths into each worker worktree after `git worktree add`, which keeps the branch isolated while still exposing in-flight local scripts, plans, or docs.
当工作者需要查看主检出中未跟踪或已修改的本地文件时，在计划文件中添加 `seedPaths`。ECC 在 `git worktree add` 之后仅将选定路径覆盖到每个工作者工作树中，从而在保持分支隔离的同时暴露进行中的本地脚本、计划或文档。

```json
{
  "sessionName": "workflow-e2e",
  "seedPaths": [
    "scripts/orchestrate-worktrees.js",
    "scripts/lib/tmux-worktree-orchestrator.js",
    ".claude/plan/workflow-e2e-test.json"
  ],
  "workers": [
    { "name": "docs", "task": "Update orchestration docs." }
  ]
}
```

To export a control-plane snapshot for a live tmux/worktree session, run:
要导出实时 tmux/worktree 会话的控制平面快照，请运行：

```bash
node scripts/orchestration-status.js .claude/plan/workflow-visual-proof.json
```

The snapshot includes session activity, tmux pane metadata, worker states, objectives, seeded overlays, and recent handoff summaries in JSON form.
快照以 JSON 形式包含会话活动、tmux 面板元数据、工作者状态、目标、种子覆盖内容和近期交接摘要。

## Operator Command-Center Handoff
## 操作员指挥中心交接

When the workflow spans multiple sessions, worktrees, or tmux panes, append a control-plane block to the final handoff:
当工作流跨越多个会话、工作树或 tmux 面板时，在最终交接中追加控制平面块：

```markdown
CONTROL PLANE
-------------
Sessions:
- active session ID or alias
- branch + worktree path for each active worker
- tmux pane or detached session name when applicable

Diffs:
- git status summary
- git diff --stat for touched files
- merge/conflict risk notes

Approvals:
- pending user approvals
- blocked steps awaiting confirmation

Telemetry:
- last activity timestamp or idle signal
- estimated token or cost drift
- policy events raised by hooks or reviewers
```

This keeps planner, implementer, reviewer, and loop workers legible from the operator surface.
这使规划者、实现者、审查者和循环工作者在操作员界面上保持清晰可读。

## Workflow Arguments
## 工作流参数

$ARGUMENTS:
- `feature <description>` - Full feature workflow
- `feature <description>` - 完整功能工作流
- `bugfix <description>` - Bug fix workflow
- `bugfix <description>` - Bug 修复工作流
- `refactor <description>` - Refactoring workflow
- `refactor <description>` - 重构工作流
- `security <description>` - Security review workflow
- `security <description>` - 安全审查工作流
- `custom <agents> <description>` - Custom agent sequence
- `custom <agents> <description>` - 自定义代理序列

## Custom Workflow Example
## 自定义工作流示例

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "Redesign caching layer"
```

## Tips
## 提示

1. **Start with planner** for complex features
1. **从 planner 开始**处理复杂功能
2. **Always include code-reviewer** before merge
2. **合并前始终包含 code-reviewer**
3. **Use security-reviewer** for auth/payment/PII
3. **针对认证/支付/PII 使用 security-reviewer**
4. **Keep handoffs concise** - focus on what next agent needs
4. **保持交接简洁** — 聚焦下一个代理所需的内容
5. **Run verification** between agents if needed
5. 必要时在代理之间**运行验证**
