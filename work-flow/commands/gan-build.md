Parse the following from $ARGUMENTS:
从 $ARGUMENTS 中解析以下内容：
1. `brief` — the user's one-line description of what to build
1. `brief` — 用户对构建内容的一行描述
2. `--max-iterations N` — (optional, default 15) maximum generator-evaluator cycles
2. `--max-iterations N` — （可选，默认 15）生成器-评估器的最大循环次数
3. `--pass-threshold N` — (optional, default 7.0) weighted score to pass
3. `--pass-threshold N` — （可选，默认 7.0）通过所需的加权分数
4. `--skip-planner` — (optional) skip planner, assume spec.md already exists
4. `--skip-planner` — （可选）跳过规划器，假设 spec.md 已存在
5. `--eval-mode MODE` — (optional, default "playwright") one of: playwright, screenshot, code-only
5. `--eval-mode MODE` — （可选，默认 "playwright"）可选值：playwright、screenshot、code-only

## GAN-Style Harness Build
## GAN 风格的测试框架构建

This command orchestrates a three-agent build loop inspired by Anthropic's March 2026 harness design paper.
此命令编排一个三代理构建循环，灵感来自 Anthropic 2026 年 3 月的测试框架设计论文。

### Phase 0: Setup
### 阶段 0：准备工作
1. Create `gan-harness/` directory in project root
1. 在项目根目录创建 `gan-harness/` 目录
2. Create subdirectories: `gan-harness/feedback/`, `gan-harness/screenshots/`
2. 创建子目录：`gan-harness/feedback/`、`gan-harness/screenshots/`
3. Initialize git if not already initialized
3. 如果尚未初始化，则初始化 git
4. Log start time and configuration
4. 记录开始时间和配置

### Phase 1: Planning (Planner Agent)
### 阶段 1：规划（规划器代理）
Unless `--skip-planner` is set:
除非设置了 `--skip-planner`：
1. Launch the `gan-planner` agent via Task tool with the user's brief
1. 通过 Task 工具以用户简介启动 `gan-planner` 代理
2. Wait for it to produce `gan-harness/spec.md` and `gan-harness/eval-rubric.md`
2. 等待其生成 `gan-harness/spec.md` 和 `gan-harness/eval-rubric.md`
3. Display the spec summary to the user
3. 向用户展示规格摘要
4. Proceed to Phase 2
4. 进入阶段 2

### Phase 2: Generator-Evaluator Loop
### 阶段 2：生成器-评估器循环
```
iteration = 1
while iteration <= max_iterations:

    # GENERATE
    Launch gan-generator agent via Task tool:
    - Read spec.md
    - If iteration > 1: read feedback/feedback-{iteration-1}.md
    - Build/improve the application
    - Ensure dev server is running
    - Commit changes

    # Wait for generator to finish

    # EVALUATE
    Launch gan-evaluator agent via Task tool:
    - Read eval-rubric.md and spec.md
    - Test the live application (mode: playwright/screenshot/code-only)
    - Score against rubric
    - Write feedback to feedback/feedback-{iteration}.md

    # Wait for evaluator to finish

    # CHECK SCORE
    Read feedback/feedback-{iteration}.md
    Extract weighted total score

    if score >= pass_threshold:
        Log "PASSED at iteration {iteration} with score {score}"
        Break

    if iteration >= 3 and score has not improved in last 2 iterations:
        Log "PLATEAU detected — stopping early"
        Break

    iteration += 1
```

### Phase 3: Summary
### 阶段 3：总结
1. Read all feedback files
1. 读取所有反馈文件
2. Display final scores and iteration history
2. 展示最终分数和迭代历史
3. Show score progression: `iteration 1: 4.2 → iteration 2: 5.8 → ... → iteration N: 7.5`
3. 展示分数进展：`iteration 1: 4.2 → iteration 2: 5.8 → ... → iteration N: 7.5`
4. List any remaining issues from the final evaluation
4. 列出最终评估中的所有遗留问题
5. Report total time and estimated cost
5. 报告总用时和预估费用

### Output
### 输出

```markdown
## GAN Harness Build Report

**Brief:** [original prompt]
**Result:** PASS/FAIL
**Iterations:** N / max
**Final Score:** X.X / 10

### Score Progression
| Iter | Design | Originality | Craft | Functionality | Total |
|------|--------|-------------|-------|---------------|-------|
| 1 | ... | ... | ... | ... | X.X |
| 2 | ... | ... | ... | ... | X.X |
| N | ... | ... | ... | ... | X.X |

### Remaining Issues
- [Any issues from final evaluation]

### Files Created
- gan-harness/spec.md
- gan-harness/eval-rubric.md
- gan-harness/feedback/feedback-001.md through feedback-NNN.md
- gan-harness/generator-state.md
- gan-harness/build-report.md
```

Write the full report to `gan-harness/build-report.md`.
将完整报告写入 `gan-harness/build-report.md`。
