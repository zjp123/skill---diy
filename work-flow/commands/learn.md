# /learn - Extract Reusable Patterns
# /learn - 提取可复用的模式

Analyze the current session and extract any patterns worth saving as skills.
分析当前会话并提取值得保存为技能的模式。

## Trigger
## 触发时机

Run `/learn` at any point during a session when you've solved a non-trivial problem.
在会话期间，每当解决了一个有价值的问题时，运行 `/learn`。

## What to Extract
## 提取内容

Look for:
寻找以下内容：

1. **Error Resolution Patterns**
1. **错误解决模式**
   - What error occurred?
   - 发生了什么错误？
   - What was the root cause?
   - 根本原因是什么？
   - What fixed it?
   - 如何修复的？
   - Is this reusable for similar errors?
   - 这对类似错误是否可复用？

2. **Debugging Techniques**
2. **调试技巧**
   - Non-obvious debugging steps
   - 非显而易见的调试步骤
   - Tool combinations that worked
   - 有效的工具组合
   - Diagnostic patterns
   - 诊断模式

3. **Workarounds**
3. **变通方法**
   - Library quirks
   - 库的怪癖
   - API limitations
   - API 限制
   - Version-specific fixes
   - 版本特定的修复

4. **Project-Specific Patterns**
4. **项目专属模式**
   - Codebase conventions discovered
   - 发现的代码库约定
   - Architecture decisions made
   - 做出的架构决策
   - Integration patterns
   - 集成模式

## Output Format
## 输出格式

Create a skill file at `~/.claude/skills/learned/[pattern-name].md`:
在 `~/.claude/skills/learned/[pattern-name].md` 创建技能文件：

```markdown
# [Descriptive Pattern Name]

**Extracted:** [Date]
**Context:** [Brief description of when this applies]

## Problem
[What problem this solves - be specific]

## Solution
[The pattern/technique/workaround]

## Example
[Code example if applicable]

## When to Use
[Trigger conditions - what should activate this skill]
```

## Process
## 流程

1. Review the session for extractable patterns
1. 回顾会话，寻找可提取的模式
2. Identify the most valuable/reusable insight
2. 识别最有价值/最可复用的洞察
3. Draft the skill file
3. 起草技能文件
4. Ask user to confirm before saving
4. 保存前请求用户确认
5. Save to `~/.claude/skills/learned/`
5. 保存到 `~/.claude/skills/learned/`

## Notes
## 注意事项

- Don't extract trivial fixes (typos, simple syntax errors)
- 不要提取琐碎的修复（错别字、简单的语法错误）
- Don't extract one-time issues (specific API outages, etc.)
- 不要提取一次性问题（特定 API 中断等）
- Focus on patterns that will save time in future sessions
- 专注于能在未来会话中节省时间的模式
- Keep skills focused - one pattern per skill
- 保持技能聚焦——每个技能对应一个模式
