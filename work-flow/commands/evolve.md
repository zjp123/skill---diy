---
name: evolve
description: Analyze instincts and suggest or generate evolved structures
  分析 instinct 并建议或生成演化结构
command: true
---

# Evolve Command
# Evolve 命令

## Implementation
## 实现

Run the instinct CLI using the plugin root path:
使用插件根路径运行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" evolve [--generate]
```

Or if `CLAUDE_PLUGIN_ROOT` is not set (manual installation):
或者若未设置 `CLAUDE_PLUGIN_ROOT`（手动安装时）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py evolve [--generate]
```

Analyzes instincts and clusters related ones into higher-level structures:
分析 instinct 并将相关的 instinct 聚合为更高层级的结构：
- **Commands**: When instincts describe user-invoked actions
- **命令**：当 instinct 描述用户主动调用的操作时
- **Skills**: When instincts describe auto-triggered behaviors
- **技能**：当 instinct 描述自动触发的行为时
- **Agents**: When instincts describe complex, multi-step processes
- **代理**：当 instinct 描述复杂的多步骤流程时

## Usage
## 使用方法

```
/evolve                    # Analyze all instincts and suggest evolutions
/evolve --generate         # Also generate files under evolved/{skills,commands,agents}
```

## Evolution Rules
## 演化规则

### → Command (User-Invoked)
### → 命令（用户调用）
When instincts describe actions a user would explicitly request:
当 instinct 描述用户会明确请求的操作时：
- Multiple instincts about "when user asks to..."
- 多条关于"当用户要求……"的 instinct
- Instincts with triggers like "when creating a new X"
- 带有"当创建新的 X 时"等触发器的 instinct
- Instincts that follow a repeatable sequence
- 遵循可重复序列的 instinct

Example:
示例：
- `new-table-step1`: "when adding a database table, create migration"
- `new-table-step1`: "当添加数据库表时，创建迁移"
- `new-table-step2`: "when adding a database table, update schema"
- `new-table-step2`: "当添加数据库表时，更新 schema"
- `new-table-step3`: "when adding a database table, regenerate types"
- `new-table-step3`: "当添加数据库表时，重新生成类型"

→ Creates: **new-table** command
→ 创建：**new-table** 命令

### → Skill (Auto-Triggered)
### → 技能（自动触发）
When instincts describe behaviors that should happen automatically:
当 instinct 描述应自动发生的行为时：
- Pattern-matching triggers
- 模式匹配触发器
- Error handling responses
- 错误处理响应
- Code style enforcement
- 代码风格强制执行

Example:
示例：
- `prefer-functional`: "when writing functions, prefer functional style"
- `prefer-functional`: "当编写函数时，优先使用函数式风格"
- `use-immutable`: "when modifying state, use immutable patterns"
- `use-immutable`: "当修改状态时，使用不可变模式"
- `avoid-classes`: "when designing modules, avoid class-based design"
- `avoid-classes`: "当设计模块时，避免基于类的设计"

→ Creates: `functional-patterns` skill
→ 创建：`functional-patterns` 技能

### → Agent (Needs Depth/Isolation)
### → 代理（需要深度/隔离）
When instincts describe complex, multi-step processes that benefit from isolation:
当 instinct 描述受益于隔离的复杂多步骤流程时：
- Debugging workflows
- 调试工作流
- Refactoring sequences
- 重构序列
- Research tasks
- 研究任务

Example:
示例：
- `debug-step1`: "when debugging, first check logs"
- `debug-step1`: "当调试时，首先检查日志"
- `debug-step2`: "when debugging, isolate the failing component"
- `debug-step2`: "当调试时，隔离失败的组件"
- `debug-step3`: "when debugging, create minimal reproduction"
- `debug-step3`: "当调试时，创建最小复现"
- `debug-step4`: "when debugging, verify fix with test"
- `debug-step4`: "当调试时，通过测试验证修复"

→ Creates: **debugger** agent
→ 创建：**debugger** 代理

## What to Do
## 操作步骤

1. Detect current project context
1. 检测当前项目上下文
2. Read project + global instincts (project takes precedence on ID conflicts)
2. 读取项目和全局 instinct（ID 冲突时项目优先）
3. Group instincts by trigger/domain patterns
3. 按触发器/领域模式对 instinct 分组
4. Identify:
4. 识别：
   - Skill candidates (trigger clusters with 2+ instincts)
   - 技能候选项（含 2 个以上 instinct 的触发器簇）
   - Command candidates (high-confidence workflow instincts)
   - 命令候选项（高置信度工作流 instinct）
   - Agent candidates (larger, high-confidence clusters)
   - 代理候选项（较大的高置信度簇）
5. Show promotion candidates (project -> global) when applicable
5. 在适用时显示晋升候选项（项目 -> 全局）
6. If `--generate` is passed, write files to:
6. 若传入 `--generate`，将文件写入：
   - Project scope: `~/.claude/homunculus/projects/<project-id>/evolved/`
   - 项目范围：`~/.claude/homunculus/projects/<project-id>/evolved/`
   - Global fallback: `~/.claude/homunculus/evolved/`
   - 全局回退：`~/.claude/homunculus/evolved/`

## Output Format
## 输出格式

```
============================================================
  EVOLVE ANALYSIS - 12 instincts
  Project: my-app (a1b2c3d4e5f6)
  Project-scoped: 8 | Global: 4
============================================================

High confidence instincts (>=80%): 5

## SKILL CANDIDATES
1. Cluster: "adding tests"
   Instincts: 3
   Avg confidence: 82%
   Domains: testing
   Scopes: project

## COMMAND CANDIDATES (2)
  /adding-tests
    From: test-first-workflow [project]
    Confidence: 84%

## AGENT CANDIDATES (1)
  adding-tests-agent
    Covers 3 instincts
    Avg confidence: 82%
```

## Flags
## 标志

- `--generate`: Generate evolved files in addition to analysis output
- `--generate`: 在分析输出的基础上额外生成演化文件

## Generated File Format
## 生成文件格式

### Command
### 命令
```markdown
---
name: new-table
description: Create a new database table with migration, schema update, and type generation
command: /new-table
evolved_from:
  - new-table-migration
  - update-schema
  - regenerate-types
---

# New Table Command

[Generated content based on clustered instincts]

## Steps
1. ...
2. ...
```

### Skill
### 技能
```markdown
---
name: functional-patterns
description: Enforce functional programming patterns
evolved_from:
  - prefer-functional
  - use-immutable
  - avoid-classes
---

# Functional Patterns Skill

[Generated content based on clustered instincts]
```

### Agent
### 代理
```markdown
---
name: debugger
description: Systematic debugging agent
model: sonnet
evolved_from:
  - debug-check-logs
  - debug-isolate
  - debug-reproduce
---

# Debugger Agent

[Generated content based on clustered instincts]
```
