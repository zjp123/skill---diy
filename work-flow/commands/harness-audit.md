# Harness Audit Command
# Harness Audit 命令

Run a deterministic repository harness audit and return a prioritized scorecard.
运行确定性的仓库测试框架审计并返回优先级评分卡。

## Usage
## 使用方法

`/harness-audit [scope] [--format text|json] [--root path]`

- `scope` (optional): `repo` (default), `hooks`, `skills`, `commands`, `agents`
- `scope`（可选）：`repo`（默认）、`hooks`、`skills`、`commands`、`agents`
- `--format`: output style (`text` default, `json` for automation)
- `--format`：输出风格（默认 `text`，`json` 用于自动化）
- `--root`: audit a specific path instead of the current working directory
- `--root`：审计指定路径而非当前工作目录

## Deterministic Engine
## 确定性引擎

Always run:
始终运行：

```bash
node scripts/harness-audit.js <scope> --format <text|json> [--root <path>]
```

This script is the source of truth for scoring and checks. Do not invent additional dimensions or ad-hoc points.
此脚本是评分和检查的权威来源。不要引入额外的维度或临时分值。

Rubric version: `2026-03-30`.
评分标准版本：`2026-03-30`。

The script computes 7 fixed categories (`0-10` normalized each):
脚本计算 7 个固定类别（每项标准化为 `0-10`）：

1. Tool Coverage
1. 工具覆盖率
2. Context Efficiency
2. 上下文效率
3. Quality Gates
3. 质量门控
4. Memory Persistence
4. 记忆持久化
5. Eval Coverage
5. 评估覆盖率
6. Security Guardrails
6. 安全护栏
7. Cost Efficiency
7. 成本效率

Scores are derived from explicit file/rule checks and are reproducible for the same commit.
分数由显式的文件/规则检查得出，在相同提交下可重现。
The script audits the current working directory by default and auto-detects whether the target is the ECC repo itself or a consumer project using ECC.
脚本默认审计当前工作目录，并自动检测目标是 ECC 仓库本身还是使用 ECC 的消费者项目。

## Output Contract
## 输出约定

Return:
返回：

1. `overall_score` out of `max_score` (70 for `repo`; smaller for scoped audits)
1. `overall_score` 满分 `max_score`（`repo` 为 70；范围审计时更小）
2. Category scores and concrete findings
2. 类别分数和具体发现
3. Failed checks with exact file paths
3. 失败的检查及精确文件路径
4. Top 3 actions from the deterministic output (`top_actions`)
4. 确定性输出中的前 3 个操作建议（`top_actions`）
5. Suggested ECC skills to apply next
5. 建议接下来应用的 ECC 技能

## Checklist
## 检查清单

- Use script output directly; do not rescore manually.
- 直接使用脚本输出；不要手动重新评分。
- If `--format json` is requested, return the script JSON unchanged.
- 若请求 `--format json`，原样返回脚本 JSON。
- If text is requested, summarize failing checks and top actions.
- 若请求文本格式，汇总失败的检查和首要操作建议。
- Include exact file paths from `checks[]` and `top_actions[]`.
- 包含来自 `checks[]` 和 `top_actions[]` 的精确文件路径。

## Example Result
## 示例结果

```text
Harness Audit (repo): 66/70
- Tool Coverage: 10/10 (10/10 pts)
- Context Efficiency: 9/10 (9/10 pts)
- Quality Gates: 10/10 (10/10 pts)

Top 3 Actions:
1) [Security Guardrails] Add prompt/tool preflight security guards in hooks/hooks.json. (hooks/hooks.json)
2) [Tool Coverage] Sync commands/harness-audit.md and .opencode/commands/harness-audit.md. (.opencode/commands/harness-audit.md)
3) [Eval Coverage] Increase automated test coverage across scripts/hooks/lib. (tests/)
```

## Arguments
## 参数

$ARGUMENTS:
- `repo|hooks|skills|commands|agents` (optional scope)
- `repo|hooks|skills|commands|agents`（可选范围）
- `--format text|json` (optional output format)
- `--format text|json`（可选输出格式）
