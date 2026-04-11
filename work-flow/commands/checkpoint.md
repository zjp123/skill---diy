# Checkpoint Command
# Checkpoint 命令

Create or verify a checkpoint in your workflow.
在工作流中创建或验证检查点。

## Usage
## 使用方法

`/checkpoint [create|verify|list] [name]`

## Create Checkpoint
## 创建检查点

When creating a checkpoint:
创建检查点时：

1. Run `/verify quick` to ensure current state is clean
1. 运行 `/verify quick` 以确保当前状态干净
2. Create a git stash or commit with checkpoint name
2. 使用检查点名称创建 git stash 或提交
3. Log checkpoint to `.claude/checkpoints.log`:
3. 将检查点记录到 `.claude/checkpoints.log`：

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD)" >> .claude/checkpoints.log
```

4. Report checkpoint created
4. 报告检查点已创建

## Verify Checkpoint
## 验证检查点

When verifying against a checkpoint:
针对检查点进行验证时：

1. Read checkpoint from log
1. 从日志中读取检查点
2. Compare current state to checkpoint:
2. 将当前状态与检查点进行比较：
   - Files added since checkpoint
   - 自检查点以来添加的文件
   - Files modified since checkpoint
   - 自检查点以来修改的文件
   - Test pass rate now vs then
   - 当前与当时的测试通过率
   - Coverage now vs then
   - 当前与当时的覆盖率

3. Report:
3. 报告：
```
CHECKPOINT COMPARISON: $NAME
============================
Files changed: X
Tests: +Y passed / -Z failed
Coverage: +X% / -Y%
Build: [PASS/FAIL]
```

## List Checkpoints
## 列出检查点

Show all checkpoints with:
显示所有检查点，包括：
- Name
- 名称
- Timestamp
- 时间戳
- Git SHA
- Git SHA
- Status (current, behind, ahead)
- 状态（当前、落后、领先）

## Workflow
## 工作流

Typical checkpoint flow:
典型检查点流程：

```
[Start] --> /checkpoint create "feature-start"
   |
[Implement] --> /checkpoint create "core-done"
   |
[Test] --> /checkpoint verify "core-done"
   |
[Refactor] --> /checkpoint create "refactor-done"
   |
[PR] --> /checkpoint verify "feature-start"
```

## Arguments
## 参数

$ARGUMENTS:
- `create <name>` - Create named checkpoint
- `create <name>` - 创建命名检查点
- `verify <name>` - Verify against named checkpoint
- `verify <name>` - 针对命名检查点进行验证
- `list` - Show all checkpoints
- `list` - 显示所有检查点
- `clear` - Remove old checkpoints (keeps last 5)
- `clear` - 删除旧检查点（保留最近 5 个）
