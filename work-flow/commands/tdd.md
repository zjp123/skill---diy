---
description: Legacy slash-entry shim for the tdd-workflow skill. Prefer the skill directly.
  tdd-workflow 技能的旧版斜杠入口兼容层，建议直接使用该技能。
---

# TDD Command (Legacy Shim)
# TDD 命令（旧版兼容层）

Use this only if you still invoke `/tdd`. The maintained workflow lives in `skills/tdd-workflow/SKILL.md`.
仅当你仍然调用 `/tdd` 时使用此文件。维护中的工作流位于 `skills/tdd-workflow/SKILL.md`。

## Canonical Surface
## 规范入口

- Prefer the `tdd-workflow` skill directly.
- 建议直接使用 `tdd-workflow` 技能。
- Keep this file only as a compatibility entry point.
- 仅将此文件保留为兼容性入口点。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the `tdd-workflow` skill.
应用 `tdd-workflow` 技能。
- Stay strict on RED -> GREEN -> REFACTOR.
- 严格遵守 RED -> GREEN -> REFACTOR。
- Keep tests first, coverage explicit, and checkpoint evidence clear.
- 保持测试优先、覆盖率明确、检查点证据清晰。
- Use the skill as the maintained TDD body instead of duplicating the playbook here.
- 使用该技能作为维护中的 TDD 主体，而非在此处重复操作手册。
})
```

## Step 3: Run Tests - Verify FAIL

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ should return high score for liquid market (2 ms)
    Error: Not implemented

1 test failed, 0 passed
```

PASS: Tests fail as expected. Ready to implement.

## Step 4: Implement Minimal Code (GREEN)

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  // Handle zero volume edge case
  if (market.totalVolume === 0) {
    return 0
  }

  // Calculate component scores (0-100 scale)
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  // Recent activity bonus
  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  // Weighted average
  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100) // Clamp to 0-100
}
```

## Step 5: Run Tests - Verify PASS

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

PASS: All tests passing!

## Step 6: Refactor (IMPROVE)

```typescript
// lib/liquidity.ts - Refactored with constants and better readability
const WEIGHTS = {
  VOLUME: 0.4,
  SPREAD: 0.3,
  TRADERS: 0.2,
  RECENCY: 0.1,
} as const

const SCALE_FACTORS = {
  VOLUME: 1000,
  SPREAD: 1000,
  TRADERS: 10,
  RECENCY_PENALTY: 10,
} as const

function clamp(value: number, min: number, max: number): number {
  return Math.min(Math.max(value, min), max)
}

export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) return 0

  const volumeScore = Math.min(market.totalVolume / SCALE_FACTORS.VOLUME, 100)
  const spreadScore = clamp(100 - (market.bidAskSpread * SCALE_FACTORS.SPREAD), 0, 100)
  const traderScore = Math.min(market.activeTraders / SCALE_FACTORS.TRADERS, 100)

  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = clamp(100 - (hoursSinceLastTrade * SCALE_FACTORS.RECENCY_PENALTY), 0, 100)

  const weightedScore =
    volumeScore * WEIGHTS.VOLUME +
    spreadScore * WEIGHTS.SPREAD +
    traderScore * WEIGHTS.TRADERS +
    recencyScore * WEIGHTS.RECENCY

  return clamp(weightedScore, 0, 100)
}
```

## Step 7: Verify Tests Still Pass

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

PASS: Refactoring complete, tests still passing!

## Step 8: Check Coverage

```bash
npm test -- --coverage lib/liquidity.test.ts

File           | % Stmts | % Branch | % Funcs | % Lines
---------------|---------|----------|---------|--------
liquidity.ts   |   100   |   100    |   100   |   100

Coverage: 100% PASS: (Target: 80%)
```

PASS: TDD session complete!
```

## TDD Best Practices
## TDD 最佳实践

**DO:**
**应做：**
- PASS: Write the test FIRST, before any implementation
- PASS: 先写测试，在任何实现之前
- PASS: Run tests and verify they FAIL before implementing
- PASS: 实现前运行测试并验证其失败
- PASS: Write minimal code to make tests pass
- PASS: 编写最少量的代码使测试通过
- PASS: Refactor only after tests are green
- PASS: 仅在测试全部通过后重构
- PASS: Add edge cases and error scenarios
- PASS: 添加边界情况和错误场景
- PASS: Aim for 80%+ coverage (100% for critical code)
- PASS: 目标覆盖率 80%+（关键代码 100%）

**DON'T:**
**不应做：**
- FAIL: Write implementation before tests
- FAIL: 在测试之前编写实现
- FAIL: Skip running tests after each change
- FAIL: 每次变更后跳过运行测试
- FAIL: Write too much code at once
- FAIL: 一次编写过多代码
- FAIL: Ignore failing tests
- FAIL: 忽略失败的测试
- FAIL: Test implementation details (test behavior)
- FAIL: 测试实现细节（应测试行为）
- FAIL: Mock everything (prefer integration tests)
- FAIL: 模拟一切（优先使用集成测试）

## Test Types to Include
## 需包含的测试类型

**Unit Tests** (Function-level):
**单元测试**（函数级）：
- Happy path scenarios
- 正常路径场景
- Edge cases (empty, null, max values)
- 边界情况（空值、null、最大值）
- Error conditions
- 错误条件
- Boundary values
- 边界值

**Integration Tests** (Component-level):
**集成测试**（组件级）：
- API endpoints
- API 端点
- Database operations
- 数据库操作
- External service calls
- 外部服务调用
- React components with hooks
- 带 hooks 的 React 组件

**E2E Tests** (use `/e2e` command):
**E2E 测试**（使用 `/e2e` 命令）：
- Critical user flows
- 关键用户流程
- Multi-step processes
- 多步骤流程
- Full stack integration
- 全栈集成

## Coverage Requirements
## 覆盖率要求

- **80% minimum** for all code
- **80% 最低要求**适用于所有代码
- **100% required** for:
- **100% 必需**适用于：
  - Financial calculations
  - 财务计算
  - Authentication logic
  - 认证逻辑
  - Security-critical code
  - 安全关键代码
  - Core business logic
  - 核心业务逻辑

## Important Notes
## 重要说明

**MANDATORY**: Tests must be written BEFORE implementation. The TDD cycle is:
**强制要求**：测试必须在实现之前编写。TDD 循环为：

1. **RED** - Write failing test
1. **RED** - 编写失败测试
2. **GREEN** - Implement to pass
2. **GREEN** - 实现以通过
3. **REFACTOR** - Improve code
3. **REFACTOR** - 改进代码

Never skip the RED phase. Never write code before tests.
永远不要跳过 RED 阶段。永远不要在测试之前编写代码。

## Integration with Other Commands
## 与其他命令的集成

- Use `/plan` first to understand what to build
- 先使用 `/plan` 了解要构建的内容
- Use `/tdd` to implement with tests
- 使用 `/tdd` 配合测试进行实现
- Use `/build-fix` if build errors occur
- 若出现构建错误，使用 `/build-fix`
- Use `/code-review` to review implementation
- 使用 `/code-review` 审查实现
- Use `/test-coverage` to verify coverage
- 使用 `/test-coverage` 验证覆盖率

## Related Agents
## 相关代理

This command invokes the `tdd-guide` agent provided by ECC.
此命令调用 ECC 提供的 `tdd-guide` 代理。

The related `tdd-workflow` skill is also bundled with ECC.
相关的 `tdd-workflow` 技能也随 ECC 一同打包提供。

For manual installs, the source files live at:
对于手动安装，源文件位于：
- `agents/tdd-guide.md`
- `agents/tdd-guide.md`
- `skills/tdd-workflow/SKILL.md`
- `skills/tdd-workflow/SKILL.md`
