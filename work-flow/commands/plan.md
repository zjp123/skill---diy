---
description: Restate requirements, assess risks, and create step-by-step implementation plan. WAIT for user CONFIRM before touching any code.
  重述需求、评估风险并创建逐步实现计划。在接触任何代码之前，必须等待用户确认。
---

# Plan Command
# Plan 命令

This command invokes the **planner** agent to create a comprehensive implementation plan before writing any code.
此命令调用 **planner** 代理，在编写任何代码之前创建全面的实现计划。

## What This Command Does
## 此命令的功能

1. **Restate Requirements** - Clarify what needs to be built
1. **重述需求** — 明确需要构建的内容
2. **Identify Risks** - Surface potential issues and blockers
2. **识别风险** — 发现潜在问题和阻塞项
3. **Create Step Plan** - Break down implementation into phases
3. **创建步骤计划** — 将实现分解为多个阶段
4. **Wait for Confirmation** - MUST receive user approval before proceeding
4. **等待确认** — 在继续之前必须获得用户批准

## When to Use
## 使用时机

Use `/plan` when:
在以下情况使用 `/plan`：
- Starting a new feature
- 开始新功能
- Making significant architectural changes
- 进行重大架构变更
- Working on complex refactoring
- 处理复杂重构
- Multiple files/components will be affected
- 将影响多个文件/组件
- Requirements are unclear or ambiguous
- 需求不清晰或存在歧义

## How It Works
## 工作原理

The planner agent will:
planner 代理将：

1. **Analyze the request** and restate requirements in clear terms
1. **分析请求**并用清晰的术语重述需求
2. **Break down into phases** with specific, actionable steps
2. **分解为阶段**，包含具体、可执行的步骤
3. **Identify dependencies** between components
3. **识别组件之间的依赖关系**
4. **Assess risks** and potential blockers
4. **评估风险**和潜在阻塞项
5. **Estimate complexity** (High/Medium/Low)
5. **估算复杂度**（高/中/低）
6. **Present the plan** and WAIT for your explicit confirmation
6. **展示计划**并等待你的明确确认

## Example Usage
## 使用示例

```
User: /plan I need to add real-time notifications when markets resolve

Agent (planner):
# Implementation Plan: Real-Time Market Resolution Notifications

## Requirements Restatement
- Send notifications to users when markets they're watching resolve
- Support multiple notification channels (in-app, email, webhook)
- Ensure notifications are delivered reliably
- Include market outcome and user's position result

## Implementation Phases

### Phase 1: Database Schema
- Add notifications table with columns: id, user_id, market_id, type, status, created_at
- Add user_notification_preferences table for channel preferences
- Create indexes on user_id and market_id for performance

### Phase 2: Notification Service
- Create notification service in lib/notifications.ts
- Implement notification queue using BullMQ/Redis
- Add retry logic for failed deliveries
- Create notification templates

### Phase 3: Integration Points
- Hook into market resolution logic (when status changes to "resolved")
- Query all users with positions in market
- Enqueue notifications for each user

### Phase 4: Frontend Components
- Create NotificationBell component in header
- Add NotificationList modal
- Implement real-time updates via Supabase subscriptions
- Add notification preferences page

## Dependencies
- Redis (for queue)
- Email service (SendGrid/Resend)
- Supabase real-time subscriptions

## Risks
- HIGH: Email deliverability (SPF/DKIM required)
- MEDIUM: Performance with 1000+ users per market
- MEDIUM: Notification spam if markets resolve frequently
- LOW: Real-time subscription overhead

## Estimated Complexity: MEDIUM
- Backend: 4-6 hours
- Frontend: 3-4 hours
- Testing: 2-3 hours
- Total: 9-13 hours

**WAITING FOR CONFIRMATION**: Proceed with this plan? (yes/no/modify)
```

## Important Notes
## 重要说明

**CRITICAL**: The planner agent will **NOT** write any code until you explicitly confirm the plan with "yes" or "proceed" or similar affirmative response.
**关键**：planner 代理在你用"yes"、"proceed"或类似肯定回复明确确认计划之前，**不会**编写任何代码。

If you want changes, respond with:
如果需要修改，请回复：
- "modify: [your changes]"
- "modify: [你的修改内容]"
- "different approach: [alternative]"
- "different approach: [替代方案]"
- "skip phase 2 and do phase 3 first"
- "skip phase 2 and do phase 3 first"

## Integration with Other Commands
## 与其他命令的集成

After planning:
规划完成后：
- Use `/tdd` to implement with test-driven development
- 使用 `/tdd` 进行测试驱动开发实现
- Use `/build-fix` if build errors occur
- 如果出现构建错误，使用 `/build-fix`
- Use `/code-review` to review completed implementation
- 使用 `/code-review` 审查已完成的实现

> **Need deeper planning?** Use `/prp-plan` for artifact-producing planning with PRD integration, codebase analysis, and pattern extraction. Use `/prp-implement` to execute those plans with rigorous validation loops.
> **需要更深度的规划？** 使用 `/prp-plan` 进行产出物式规划，集成 PRD、代码库分析和模式提取。使用 `/prp-implement` 通过严格验证循环执行这些计划。

## Related Agents
## 相关代理

This command invokes the `planner` agent provided by ECC.
此命令调用 ECC 提供的 `planner` 代理。

For manual installs, the source file lives at:
对于手动安装，源文件位于：
`agents/planner.md`
