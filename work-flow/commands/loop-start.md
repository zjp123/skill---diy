# Loop Start Command
# Loop Start 命令

Start a managed autonomous loop pattern with safety defaults.
以安全默认设置启动受管理的自主循环模式。

## Usage
## 使用方法

`/loop-start [pattern] [--mode safe|fast]`

- `pattern`: `sequential`, `continuous-pr`, `rfc-dag`, `infinite`
- `pattern`：`sequential`、`continuous-pr`、`rfc-dag`、`infinite`
- `--mode`:
- `--mode`：
  - `safe` (default): strict quality gates and checkpoints
  - `safe`（默认）：严格的质量门控和检查点
  - `fast`: reduced gates for speed
  - `fast`：精简门控以提升速度

## Flow
## 流程

1. Confirm repository state and branch strategy.
1. 确认仓库状态和分支策略。
2. Select loop pattern and model tier strategy.
2. 选择循环模式和模型层级策略。
3. Enable required hooks/profile for the chosen mode.
3. 为所选模式启用所需的钩子/配置文件。
4. Create loop plan and write runbook under `.claude/plans/`.
4. 创建循环计划并将运行手册写入 `.claude/plans/`。
5. Print commands to start and monitor the loop.
5. 打印用于启动和监控循环的命令。

## Required Safety Checks
## 必要安全检查

- Verify tests pass before first loop iteration.
- 在第一次循环迭代之前验证测试通过。
- Ensure `ECC_HOOK_PROFILE` is not disabled globally.
- 确保 `ECC_HOOK_PROFILE` 未被全局禁用。
- Ensure loop has explicit stop condition.
- 确保循环具有明确的停止条件。

## Arguments
## 参数

$ARGUMENTS:
- `<pattern>` optional (`sequential|continuous-pr|rfc-dag|infinite`)
- `<pattern>` 可选（`sequential|continuous-pr|rfc-dag|infinite`）
- `--mode safe|fast` optional
- `--mode safe|fast` 可选
