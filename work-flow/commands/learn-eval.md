---
description: "Extract reusable patterns from the session, self-evaluate quality before saving, and determine the right save location (Global vs Project)."
  从会话中提取可复用的模式，保存前自我评估质量，并确定正确的保存位置（全局或项目）。
---

# /learn-eval - Extract, Evaluate, then Save
# /learn-eval - 提取、评估，然后保存

Extends `/learn` with a quality gate, save-location decision, and knowledge-placement awareness before writing any skill file.
在写入任何技能文件之前，用质量门控、保存位置决策和知识放置意识扩展了 `/learn`。

## What to Extract
## 提取内容

Look for:
寻找以下内容：

1. **Error Resolution Patterns** — root cause + fix + reusability
1. **错误解决模式** — 根本原因 + 修复方法 + 可复用性
2. **Debugging Techniques** — non-obvious steps, tool combinations
2. **调试技巧** — 非显而易见的步骤、工具组合
3. **Workarounds** — library quirks, API limitations, version-specific fixes
3. **变通方法** — 库的怪癖、API 限制、版本特定的修复
4. **Project-Specific Patterns** — conventions, architecture decisions, integration patterns
4. **项目专属模式** — 约定、架构决策、集成模式

## Process
## 流程

1. Review the session for extractable patterns
1. 回顾会话，寻找可提取的模式
2. Identify the most valuable/reusable insight
2. 识别最有价值/最可复用的洞察

3. **Determine save location:**
3. **确定保存位置：**
   - Ask: "Would this pattern be useful in a different project?"
   - 询问："这个模式在其他项目中也有用吗？"
   - **Global** (`~/.claude/skills/learned/`): Generic patterns usable across 2+ projects (bash compatibility, LLM API behavior, debugging techniques, etc.)
   - **全局**（`~/.claude/skills/learned/`）：可在 2 个以上项目中使用的通用模式（bash 兼容性、LLM API 行为、调试技巧等）
   - **Project** (`.claude/skills/learned/` in current project): Project-specific knowledge (quirks of a particular config file, project-specific architecture decisions, etc.)
   - **项目**（当前项目中的 `.claude/skills/learned/`）：项目专属知识（特定配置文件的怪癖、项目特定的架构决策等）
   - When in doubt, choose Global (moving Global → Project is easier than the reverse)
   - 若有疑问，选择全局（全局 → 项目的迁移比反向更容易）

4. Draft the skill file using this format:
4. 使用以下格式起草技能文件：

```markdown
---
name: pattern-name
description: "Under 130 characters"
user-invocable: false
origin: auto-extracted
---

# [Descriptive Pattern Name]

**Extracted:** [Date]
**Context:** [Brief description of when this applies]

## Problem
[What problem this solves - be specific]

## Solution
[The pattern/technique/workaround - with code examples]

## When to Use
[Trigger conditions]
```

5. **Quality gate — Checklist + Holistic verdict**
5. **质量门控 — 检查清单 + 整体评定**

   ### 5a. Required checklist (verify by actually reading files)
   ### 5a. 必要检查清单（通过实际读取文件来验证）

   Execute **all** of the following before evaluating the draft:
   在评估草稿之前执行以下**所有**操作：

   - [ ] Grep `~/.claude/skills/` and relevant project `.claude/skills/` files by keyword to check for content overlap
   - [ ] 按关键词 grep `~/.claude/skills/` 和相关项目 `.claude/skills/` 文件，检查内容重叠
   - [ ] Check MEMORY.md (both project and global) for overlap
   - [ ] 检查 MEMORY.md（项目和全局）中的重叠内容
   - [ ] Consider whether appending to an existing skill would suffice
   - [ ] 考虑是否追加到现有技能即可满足需求
   - [ ] Confirm this is a reusable pattern, not a one-off fix
   - [ ] 确认这是可复用的模式，而非一次性修复

   ### 5b. Holistic verdict
   ### 5b. 整体评定

   Synthesize the checklist results and draft quality, then choose **one** of the following:
   综合检查清单结果和草稿质量，然后选择以下**一项**：

   | Verdict | Meaning | Next Action |
   |---------|---------|-------------|
   | 评定 | 含义 | 下一步操作 |
   | **Save** | Unique, specific, well-scoped | Proceed to Step 6 |
   | **保存** | 唯一、具体、范围明确 | 继续执行步骤 6 |
   | **Improve then Save** | Valuable but needs refinement | List improvements → revise → re-evaluate (once) |
   | **改进后保存** | 有价值但需要完善 | 列出改进点 → 修订 → 重新评估（一次） |
   | **Absorb into [X]** | Should be appended to an existing skill | Show target skill and additions → Step 6 |
   | **合并到 [X]** | 应追加到现有技能中 | 展示目标技能和新增内容 → 步骤 6 |
   | **Drop** | Trivial, redundant, or too abstract | Explain reasoning and stop |
   | **丢弃** | 琐碎、冗余或过于抽象 | 说明理由并停止 |

**Guideline dimensions** (informing the verdict, not scored):
**指导维度**（为评定提供参考，不计分）：

- **Specificity & Actionability**: Contains code examples or commands that are immediately usable
- **具体性与可操作性**：包含可立即使用的代码示例或命令
- **Scope Fit**: Name, trigger conditions, and content are aligned and focused on a single pattern
- **范围适配性**：名称、触发条件和内容一致，且聚焦于单一模式
- **Uniqueness**: Provides value not covered by existing skills (informed by checklist results)
- **唯一性**：提供现有技能未涵盖的价值（基于检查清单结果）
- **Reusability**: Realistic trigger scenarios exist in future sessions
- **可复用性**：在未来会话中存在真实的触发场景

6. **Verdict-specific confirmation flow**
6. **按评定结果的确认流程**

- **Improve then Save**: Present the required improvements + revised draft + updated checklist/verdict after one re-evaluation; if the revised verdict is **Save**, save after user confirmation, otherwise follow the new verdict
- **改进后保存**：展示所需改进点 + 修订草稿 + 一次重评估后更新的检查清单/评定；若修订后的评定为**保存**，经用户确认后保存，否则遵循新的评定结果
- **Save**: Present save path + checklist results + 1-line verdict rationale + full draft → save after user confirmation
- **保存**：展示保存路径 + 检查清单结果 + 单行评定理由 + 完整草稿 → 经用户确认后保存
- **Absorb into [X]**: Present target path + additions (diff format) + checklist results + verdict rationale → append after user confirmation
- **合并到 [X]**：展示目标路径 + 新增内容（diff 格式）+ 检查清单结果 + 评定理由 → 经用户确认后追加
- **Drop**: Show checklist results + reasoning only (no confirmation needed)
- **丢弃**：仅展示检查清单结果 + 理由（无需用户确认）

7. Save / Absorb to the determined location
7. 保存/合并到确定的位置

## Output Format for Step 5
## 步骤 5 的输出格式

```
### Checklist
- [x] skills/ grep: no overlap (or: overlap found → details)
- [x] MEMORY.md: no overlap (or: overlap found → details)
- [x] Existing skill append: new file appropriate (or: should append to [X])
- [x] Reusability: confirmed (or: one-off → Drop)

### Verdict: Save / Improve then Save / Absorb into [X] / Drop

**Rationale:** (1-2 sentences explaining the verdict)
```

## Design Rationale
## 设计理念

This version replaces the previous 5-dimension numeric scoring rubric (Specificity, Actionability, Scope Fit, Non-redundancy, Coverage scored 1-5) with a checklist-based holistic verdict system. Modern frontier models (Opus 4.6+) have strong contextual judgment — forcing rich qualitative signals into numeric scores loses nuance and can produce misleading totals. The holistic approach lets the model weigh all factors naturally, producing more accurate save/drop decisions while the explicit checklist ensures no critical check is skipped.
此版本将之前的 5 维度数字评分标准（具体性、可操作性、范围适配性、非冗余性、覆盖度，各项评分 1-5）替换为基于检查清单的整体评定体系。现代前沿模型（Opus 4.6+）具有强大的上下文判断力——将丰富的定性信号强制转换为数字分数会损失细微差别，并可能产生误导性的总分。整体评定方法让模型自然权衡所有因素，产生更准确的保存/丢弃决策，而明确的检查清单则确保不遗漏任何关键检查。

## Notes
## 注意事项

- Don't extract trivial fixes (typos, simple syntax errors)
- 不要提取琐碎的修复（错别字、简单的语法错误）
- Don't extract one-time issues (specific API outages, etc.)
- 不要提取一次性问题（特定 API 中断等）
- Focus on patterns that will save time in future sessions
- 专注于能在未来会话中节省时间的模式
- Keep skills focused — one pattern per skill
- 保持技能聚焦——每个技能对应一个模式
- When the verdict is Absorb, append to the existing skill rather than creating a new file
- 当评定为合并时，追加到现有技能而非创建新文件
