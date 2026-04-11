---
description: Adversarial dual-review convergence loop — two independent model reviewers must both approve before code ships.
  对抗性双重审查收敛循环——两个独立的模型审查者必须都批准，代码才能发布。
---

# Santa Loop
# 圣诞循环

Adversarial dual-review convergence loop using the santa-method skill. Two independent reviewers — different models, no shared context — must both return NICE before code ships.
使用 santa-method 技能的对抗性双重审查收敛循环。两个独立的审查者——不同模型、无共享上下文——必须都返回 NICE，代码才能发布。

## Purpose
## 目的

Run two independent reviewers (Claude Opus + an external model) against the current task output. Both must return NICE before the code is pushed. If either returns NAUGHTY, fix all flagged issues, commit, and re-run fresh reviewers — up to 3 rounds.
针对当前任务输出运行两个独立审查者（Claude Opus + 外部模型）。两者都必须返回 NICE，代码才能推送。若任一返回 NAUGHTY，修复所有标记的问题、提交，并重新运行全新审查者——最多 3 轮。

## Usage
## 使用方法

```
/santa-loop [file-or-glob | description]
```

## Workflow
## 工作流

### Step 1: Identify What to Review
### 步骤 1：确定审查范围

Determine the scope from `$ARGUMENTS` or fall back to uncommitted changes:
从 `$ARGUMENTS` 确定范围，或回退到未提交的变更：

```bash
git diff --name-only HEAD
```

Read all changed files to build the full review context. If `$ARGUMENTS` specifies a path, file, or description, use that as the scope instead.
读取所有变更文件以构建完整的审查上下文。若 `$ARGUMENTS` 指定了路径、文件或描述，则将其用作范围。

### Step 2: Build the Rubric
### 步骤 2：构建评审标准

Construct a rubric appropriate to the file types under review. Every criterion must have an objective PASS/FAIL condition. Include at minimum:
针对被审查的文件类型构建合适的评审标准。每个标准必须有客观的 PASS/FAIL 条件。至少包含：

| Criterion | Pass Condition |
|-----------|---------------|
| 标准 | 通过条件 |
| Correctness | Logic is sound, no bugs, handles edge cases |
| 正确性 | 逻辑正确、无错误、处理边界情况 |
| Security | No secrets, injection, XSS, or OWASP Top 10 issues |
| 安全性 | 无密钥、注入、XSS 或 OWASP Top 10 问题 |
| Error handling | Errors handled explicitly, no silent swallowing |
| 错误处理 | 错误显式处理，无静默吞没 |
| Completeness | All requirements addressed, no missing cases |
| 完整性 | 所有需求已处理，无遗漏情况 |
| Internal consistency | No contradictions between files or sections |
| 内部一致性 | 文件或章节之间无矛盾 |
| No regressions | Changes don't break existing behavior |
| 无回归 | 变更不破坏现有行为 |

Add domain-specific criteria based on file types (e.g., type safety for TS, memory safety for Rust, migration safety for SQL).
根据文件类型添加领域特定标准（如 TS 的类型安全、Rust 的内存安全、SQL 的迁移安全）。

### Step 3: Dual Independent Review
### 步骤 3：双重独立审查

Launch two reviewers **in parallel** using the Agent tool (both in a single message for concurrent execution). Both must complete before proceeding to the verdict gate.
使用 Agent 工具**并行**启动两个审查者（在单条消息中同时执行）。两者都必须完成，才能进入裁决关卡。

Each reviewer evaluates every rubric criterion as PASS or FAIL, then returns structured JSON:
每个审查者将每条评审标准评估为 PASS 或 FAIL，然后返回结构化 JSON：

```json
{
  "verdict": "PASS" | "FAIL",
  "checks": [
    {"criterion": "...", "result": "PASS|FAIL", "detail": "..."}
  ],
  "critical_issues": ["..."],
  "suggestions": ["..."]
}
```

The verdict gate (Step 4) maps these to NICE/NAUGHTY: both PASS → NICE, either FAIL → NAUGHTY.
裁决关卡（步骤 4）将其映射为 NICE/NAUGHTY：两者均 PASS → NICE，任一 FAIL → NAUGHTY。

#### Reviewer A: Claude Agent (always runs)
#### 审查者 A：Claude Agent（始终运行）

Launch an Agent (subagent_type: `code-reviewer`, model: `opus`) with the full rubric + all files under review. The prompt must include:
启动一个 Agent（subagent_type: `code-reviewer`，model: `opus`），附带完整评审标准 + 所有被审查文件。提示词必须包含：
- The complete rubric
- 完整的评审标准
- All file contents under review
- 所有被审查文件的内容
- "You are an independent quality reviewer. You have NOT seen any other review. Your job is to find problems, not to approve."
- "You are an independent quality reviewer. You have NOT seen any other review. Your job is to find problems, not to approve."
- Return the structured JSON verdict above
- 返回上述结构化 JSON 结论

#### Reviewer B: External Model (Claude fallback only if no external CLI installed)
#### 审查者 B：外部模型（仅在无外部 CLI 时回退到 Claude）

First, detect which CLIs are available:
首先，检测哪些 CLI 可用：
```bash
command -v codex >/dev/null 2>&1 && echo "codex" || true
command -v gemini >/dev/null 2>&1 && echo "gemini" || true
```

Build the reviewer prompt (identical rubric + instructions as Reviewer A) and write it to a unique temp file:
构建审查者提示词（与审查者 A 相同的评审标准 + 指令）并写入唯一临时文件：
```bash
PROMPT_FILE=$(mktemp /tmp/santa-reviewer-b-XXXXXX.txt)
cat > "$PROMPT_FILE" << 'EOF'
... full rubric + file contents + reviewer instructions ...
EOF
```

Use the first available CLI:
使用第一个可用的 CLI：

**Codex CLI** (if installed)
**Codex CLI**（若已安装）
```bash
codex exec --sandbox read-only -m gpt-5.4 -C "$(pwd)" - < "$PROMPT_FILE"
rm -f "$PROMPT_FILE"
```

**Gemini CLI** (if installed and codex is not)
**Gemini CLI**（若已安装且 codex 未安装）
```bash
gemini -p "$(cat "$PROMPT_FILE")" -m gemini-2.5-pro
rm -f "$PROMPT_FILE"
```

**Claude Agent fallback** (only if neither `codex` nor `gemini` is installed)
**Claude Agent 回退**（仅在 `codex` 和 `gemini` 均未安装时）
Launch a second Claude Agent (subagent_type: `code-reviewer`, model: `opus`). Log a warning that both reviewers share the same model family — true model diversity was not achieved but context isolation is still enforced.
启动第二个 Claude Agent（subagent_type: `code-reviewer`，model: `opus`）。记录警告：两个审查者共享相同模型系列——未实现真正的模型多样性，但上下文隔离仍然强制执行。

In all cases, the reviewer must return the same structured JSON verdict as Reviewer A.
在所有情况下，审查者必须返回与审查者 A 相同的结构化 JSON 结论。

### Step 4: Verdict Gate
### 步骤 4：裁决关卡

- **Both PASS** → **NICE** — proceed to Step 6 (push)
- **两者均 PASS** → **NICE** — 进入步骤 6（推送）
- **Either FAIL** → **NAUGHTY** — merge all critical issues from both reviewers, deduplicate, proceed to Step 5
- **任一 FAIL** → **NAUGHTY** — 合并两个审查者的所有关键问题，去重，进入步骤 5

### Step 5: Fix Cycle (NAUGHTY path)
### 步骤 5：修复循环（NAUGHTY 路径）

1. Display all critical issues from both reviewers
1. 展示两个审查者的所有关键问题
2. Fix every flagged issue — change only what was flagged, no drive-by refactors
2. 修复每个标记的问题——只改动被标记的内容，不做顺手重构
3. Commit all fixes in a single commit:
3. 将所有修复提交到单次提交中：
   ```
   fix: address santa-loop review findings (round N)
   ```
4. Re-run Step 3 with **fresh reviewers** (no memory of previous rounds)
4. 使用**全新审查者**重新运行步骤 3（无前轮记忆）
5. Repeat until both return PASS
5. 重复直到两者均返回 PASS

**Maximum 3 iterations.** If still NAUGHTY after 3 rounds, stop and present remaining issues:
**最多 3 次迭代。** 若 3 轮后仍为 NAUGHTY，停止并呈现剩余问题：

```
SANTA LOOP ESCALATION (exceeded 3 iterations)

Remaining issues after 3 rounds:
- [list all unresolved critical issues from both reviewers]

Manual review required before proceeding.
```

Do NOT push.
不要推送。

### Step 6: Push (NICE path)
### 步骤 6：推送（NICE 路径）

When both reviewers return PASS:
当两个审查者均返回 PASS 时：

```bash
git push -u origin HEAD
```

### Step 7: Final Report
### 步骤 7：最终报告

Print the output report (see Output section below).
打印输出报告（见下方输出部分）。

## Output
## 输出

```
SANTA VERDICT: [NICE / NAUGHTY (escalated)]

Reviewer A (Claude Opus):   [PASS/FAIL]
Reviewer B ([model used]):  [PASS/FAIL]

Agreement:
  Both flagged:      [issues caught by both]
  Reviewer A only:   [issues only A caught]
  Reviewer B only:   [issues only B caught]

Iterations: [N]/3
Result:     [PUSHED / ESCALATED TO USER]
```

## Notes
## 注意事项

- Reviewer A (Claude Opus) always runs — guarantees at least one strong reviewer regardless of tooling.
- 审查者 A（Claude Opus）始终运行——无论工具情况如何，保证至少有一个强力审查者。
- Model diversity is the goal for Reviewer B. GPT-5.4 or Gemini 2.5 Pro gives true independence — different training data, different biases, different blind spots. The Claude-only fallback still provides value via context isolation but loses model diversity.
- 模型多样性是审查者 B 的目标。GPT-5.4 或 Gemini 2.5 Pro 提供真正的独立性——不同的训练数据、不同的偏见、不同的盲点。仅 Claude 的回退仍通过上下文隔离提供价值，但失去了模型多样性。
- Strongest available models are used: Opus for Reviewer A, GPT-5.4 or Gemini 2.5 Pro for Reviewer B.
- 使用最强的可用模型：审查者 A 用 Opus，审查者 B 用 GPT-5.4 或 Gemini 2.5 Pro。
- External reviewers run with `--sandbox read-only` (Codex) to prevent repo mutation during review.
- 外部审查者以 `--sandbox read-only`（Codex）运行，以防止审查期间仓库被修改。
- Fresh reviewers each round prevents anchoring bias from prior findings.
- 每轮使用全新审查者，防止因先前发现产生锚定偏差。
- The rubric is the most important input. Tighten it if reviewers rubber-stamp or flag subjective style issues.
- 评审标准是最重要的输入。若审查者走过场或标记主观风格问题，请收紧标准。
- Commits happen on NAUGHTY rounds so fixes are preserved even if the loop is interrupted.
- 修复提交发生在 NAUGHTY 轮次，确保即使循环中断，修复内容也得以保留。
- Push only happens after NICE — never mid-loop.
- 推送仅在 NICE 之后发生——绝不在循环中途推送。
