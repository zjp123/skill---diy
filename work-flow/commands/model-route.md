# Model Route Command
# Model Route 命令

Recommend the best model tier for the current task by complexity and budget.
根据任务的复杂度和预算推荐最佳模型层级。

## Usage
## 使用方法

`/model-route [task-description] [--budget low|med|high]`

## Routing Heuristic
## 路由启发规则

- `haiku`: deterministic, low-risk mechanical changes
- `haiku`：确定性的、低风险的机械性变更
- `sonnet`: default for implementation and refactors
- `sonnet`：实现和重构的默认选择
- `opus`: architecture, deep review, ambiguous requirements
- `opus`：架构设计、深度审查、模糊需求

## Required Output
## 必要输出

- recommended model
- 推荐的模型
- confidence level
- 置信度
- why this model fits
- 该模型适合的原因
- fallback model if first attempt fails
- 首次尝试失败时的备选模型

## Arguments
## 参数

$ARGUMENTS:
- `[task-description]` optional free-text
- `[task-description]` 可选自由文本
- `--budget low|med|high` optional
- `--budget low|med|high` 可选
