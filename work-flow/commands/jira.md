---
description: Retrieve a Jira ticket, analyze requirements, update status, or add comments. Uses the jira-integration skill and MCP or REST API.
  检索 Jira 工单、分析需求、更新状态或添加评论。使用 jira-integration 技能和 MCP 或 REST API。
---

# Jira Command
# Jira 命令

Interact with Jira tickets directly from your workflow — fetch tickets, analyze requirements, add comments, and transition status.
直接在工作流中与 Jira 工单交互——获取工单、分析需求、添加评论并流转状态。

## Usage
## 使用方法

```
/jira get <TICKET-KEY>          # Fetch and analyze a ticket
/jira comment <TICKET-KEY>      # Add a progress comment
/jira transition <TICKET-KEY>   # Change ticket status
/jira search <JQL>              # Search issues with JQL
```

## What This Command Does
## 命令功能

1. **Get & Analyze** — Fetch a Jira ticket and extract requirements, acceptance criteria, test scenarios, and dependencies
1. **获取并分析** — 获取 Jira 工单并提取需求、验收标准、测试场景和依赖关系
2. **Comment** — Add structured progress updates to a ticket
2. **评论** — 向工单添加结构化的进度更新
3. **Transition** — Move a ticket through workflow states (To Do → In Progress → Done)
3. **流转** — 使工单在工作流状态间流转（待办 → 进行中 → 完成）
4. **Search** — Find issues using JQL queries
4. **搜索** — 使用 JQL 查询查找问题

## How It Works
## 工作原理

### `/jira get <TICKET-KEY>`
### 获取工单并分析

1. Fetch the ticket from Jira (via MCP `jira_get_issue` or REST API)
1. 从 Jira 获取工单（通过 MCP `jira_get_issue` 或 REST API）
2. Extract all fields: summary, description, acceptance criteria, priority, labels, linked issues
2. 提取所有字段：摘要、描述、验收标准、优先级、标签、关联问题
3. Optionally fetch comments for additional context
3. 可选择性地获取评论以补充上下文
4. Produce a structured analysis:
4. 生成结构化分析：

```
Ticket: PROJ-1234
Summary: [title]
Status: [status]
Priority: [priority]
Type: [Story/Bug/Task]

Requirements:
1. [extracted requirement]
2. [extracted requirement]

Acceptance Criteria:
- [ ] [criterion from ticket]

Test Scenarios:
- Happy Path: [description]
- Error Case: [description]
- Edge Case: [description]

Dependencies:
- [linked issues, APIs, services]

Recommended Next Steps:
- /plan to create implementation plan
- /tdd to implement with tests first
```

### `/jira comment <TICKET-KEY>`
### 添加评论

1. Summarize current session progress (what was built, tested, committed)
1. 汇总当前会话进度（构建、测试、提交了什么）
2. Format as a structured comment
2. 格式化为结构化评论
3. Post to the Jira ticket
3. 发布到 Jira 工单

### `/jira transition <TICKET-KEY>`
### 流转工单状态

1. Fetch available transitions for the ticket
1. 获取工单的可用流转状态
2. Show options to user
2. 向用户展示选项
3. Execute the selected transition
3. 执行所选的流转操作

### `/jira search <JQL>`
### 搜索工单

1. Execute the JQL query against Jira
1. 对 Jira 执行 JQL 查询
2. Return a summary table of matching issues
2. 返回匹配问题的摘要表格

## Prerequisites
## 前置条件

This command requires Jira credentials. Choose one:
此命令需要 Jira 凭据。选择以下方式之一：

**Option A — MCP Server (recommended):**
**方式 A — MCP 服务器（推荐）：**
Add `jira` to your `mcpServers` config (see `mcp-configs/mcp-servers.json` for the template).
将 `jira` 添加到 `mcpServers` 配置中（模板参见 `mcp-configs/mcp-servers.json`）。

**Option B — Environment variables:**
**方式 B — 环境变量：**
```bash
export JIRA_URL="https://yourorg.atlassian.net"
export JIRA_EMAIL="your.email@example.com"
export JIRA_API_TOKEN="your-api-token"
```

If credentials are missing, stop and direct the user to set them up.
如果凭据缺失，停止并引导用户进行设置。

## Integration with Other Commands
## 与其他命令的集成

After analyzing a ticket:
分析工单后：
- Use `/plan` to create an implementation plan from the requirements
- 使用 `/plan` 根据需求创建实现计划
- Use `/tdd` to implement with test-driven development
- 使用 `/tdd` 进行测试驱动开发
- Use `/code-review` after implementation
- 实现后使用 `/code-review` 进行代码审查
- Use `/jira comment` to post progress back to the ticket
- 使用 `/jira comment` 将进度更新回工单
- Use `/jira transition` to move the ticket when work is complete
- 工作完成后使用 `/jira transition` 流转工单状态

## Related
## 相关资源

- **Skill:** `skills/jira-integration/`
- **技能：** `skills/jira-integration/`
- **MCP config:** `mcp-configs/mcp-servers.json` → `jira`
- **MCP 配置：** `mcp-configs/mcp-servers.json` → `jira`
