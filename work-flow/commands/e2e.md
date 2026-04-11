---
description: Legacy slash-entry shim for the e2e-testing skill. Prefer the skill directly.
  e2e-testing 技能的遗留斜杠入口垫片。建议直接使用该技能。
---

# E2E Command (Legacy Shim)
# E2E 命令（遗留垫片）

Use this only if you still invoke `/e2e`. The maintained workflow lives in `skills/e2e-testing/SKILL.md`.
仅在仍使用 `/e2e` 调用时使用此文件。最新的工作流位于 `skills/e2e-testing/SKILL.md`。

## Canonical Surface
## 规范入口

- Prefer the `e2e-testing` skill directly.
- 建议直接使用 `e2e-testing` 技能。
- Keep this file only as a compatibility entry point.
- 此文件仅作为兼容性入口保留。

## Arguments
## 参数

`$ARGUMENTS`

## Delegation
## 委托

Apply the `e2e-testing` skill.
应用 `e2e-testing` 技能。
- Generate or update Playwright coverage for the requested user flow.
- 为所请求的用户流程生成或更新 Playwright 覆盖。
- Run only the relevant tests unless the user explicitly asked for the entire suite.
- 除非用户明确要求运行全套测试，否则仅运行相关测试。
- Capture the usual artifacts and report failures, flake risk, and next fixes without duplicating the full skill body here.
- 捕获常规产出物并报告失败、不稳定风险和后续修复，无需在此重复完整的技能内容。
    await marketsPage.searchMarkets('xyznonexistentmarket123456')

    // Verify empty state
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    await expect(page.locator('[data-testid="no-results"]')).toContainText(
      /no.*results|no.*markets/i
    )

    const marketCount = await marketsPage.marketCards.count()
    expect(marketCount).toBe(0)
  })

  test('can clear search and see all markets again', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // Initial market count
    const initialCount = await marketsPage.marketCards.count()

    // Perform search
    await marketsPage.searchMarkets('trump')
    await page.waitForLoadState('networkidle')

    // Verify filtered results
    const filteredCount = await marketsPage.marketCards.count()
    expect(filteredCount).toBeLessThan(initialCount)

    // Clear search
    await marketsPage.searchInput.clear()
    await page.waitForLoadState('networkidle')

    // Verify all markets shown again
    const finalCount = await marketsPage.marketCards.count()
    expect(finalCount).toBe(initialCount)
  })
})
```

## Running Tests
## 运行测试

```bash
# Run the generated test
npx playwright test tests/e2e/markets/search-and-view.spec.ts

Running 3 tests using 3 workers

  ✓  [chromium] › search-and-view.spec.ts:5:3 › user can search markets and view details (4.2s)
  ✓  [chromium] › search-and-view.spec.ts:52:3 › search with no results shows empty state (1.8s)
  ✓  [chromium] › search-and-view.spec.ts:67:3 › can clear search and see all markets again (2.9s)

  3 passed (9.1s)

Artifacts generated:
- artifacts/search-results.png
- artifacts/market-details.png
- playwright-report/index.html
```

## Test Report
## 测试报告

```
╔══════════════════════════════════════════════════════════════╗
║                    E2E Test Results                          ║
╠══════════════════════════════════════════════════════════════╣
║ Status:     PASS: ALL TESTS PASSED                              ║
║ Total:      3 tests                                          ║
║ Passed:     3 (100%)                                         ║
║ Failed:     0                                                ║
║ Flaky:      0                                                ║
║ Duration:   9.1s                                             ║
╚══════════════════════════════════════════════════════════════╝

Artifacts:
 Screenshots: 2 files
 Videos: 0 files (only on failure)
 Traces: 0 files (only on failure)
 HTML Report: playwright-report/index.html

View report: npx playwright show-report
```

PASS: E2E test suite ready for CI/CD integration!
PASS: E2E 测试套件已准备好接入 CI/CD！
```

## Test Artifacts
## 测试产出物

When tests run, the following artifacts are captured:
测试运行时，将捕获以下产出物：

**On All Tests:**
**所有测试均会产生：**
- HTML Report with timeline and results
- 包含时间线和结果的 HTML 报告
- JUnit XML for CI integration
- 用于 CI 集成的 JUnit XML

**On Failure Only:**
**仅在失败时产生：**
- Screenshot of the failing state
- 失败状态的截图
- Video recording of the test
- 测试的视频录制
- Trace file for debugging (step-by-step replay)
- 用于调试的追踪文件（逐步回放）
- Network logs
- 网络日志
- Console logs
- 控制台日志

## Viewing Artifacts
## 查看产出物

```bash
# View HTML report in browser
npx playwright show-report

# View specific trace file
npx playwright show-trace artifacts/trace-abc123.zip

# Screenshots are saved in artifacts/ directory
open artifacts/search-results.png
```

## Flaky Test Detection
## 不稳定测试检测

If a test fails intermittently:
如果测试间歇性失败：

```
WARNING:  FLAKY TEST DETECTED: tests/e2e/markets/trade.spec.ts

Test passed 7/10 runs (70% pass rate)

Common failure:
"Timeout waiting for element '[data-testid="confirm-btn"]'"

Recommended fixes:
1. Add explicit wait: await page.waitForSelector('[data-testid="confirm-btn"]')
2. Increase timeout: { timeout: 10000 }
3. Check for race conditions in component
4. Verify element is not hidden by animation

Quarantine recommendation: Mark as test.fixme() until fixed
```

## Browser Configuration
## 浏览器配置

Tests run on multiple browsers by default:
默认在多个浏览器上运行测试：
- PASS: Chromium (Desktop Chrome)
- PASS: Chromium（桌面端 Chrome）
- PASS: Firefox (Desktop)
- PASS: Firefox（桌面端）
- PASS: WebKit (Desktop Safari)
- PASS: WebKit（桌面端 Safari）
- PASS: Mobile Chrome (optional)
- PASS: Mobile Chrome（可选）

Configure in `playwright.config.ts` to adjust browsers.
在 `playwright.config.ts` 中配置以调整浏览器。

## CI/CD Integration
## CI/CD 集成

Add to your CI pipeline:
添加到 CI 流水线：

```yaml
# .github/workflows/e2e.yml
- name: Install Playwright
  run: npx playwright install --with-deps

- name: Run E2E tests
  run: npx playwright test

- name: Upload artifacts
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

## PMX-Specific Critical Flows
## PMX 专项关键流程

For PMX, prioritize these E2E tests:
对于 PMX，优先处理以下 E2E 测试：

**CRITICAL (Must Always Pass):**
**关键（必须始终通过）：**
1. User can connect wallet
1. 用户可以连接钱包
2. User can browse markets
2. 用户可以浏览市场
3. User can search markets (semantic search)
3. 用户可以搜索市场（语义搜索）
4. User can view market details
4. 用户可以查看市场详情
5. User can place trade (with test funds)
5. 用户可以下单交易（使用测试资金）
6. Market resolves correctly
6. 市场正确结算
7. User can withdraw funds
7. 用户可以提取资金

**IMPORTANT:**
**重要：**
1. Market creation flow
1. 市场创建流程
2. User profile updates
2. 用户资料更新
3. Real-time price updates
3. 实时价格更新
4. Chart rendering
4. 图表渲染
5. Filter and sort markets
5. 过滤和排序市场
6. Mobile responsive layout
6. 移动端响应式布局

## Best Practices
## 最佳实践

**DO:**
**应该：**
- PASS: Use Page Object Model for maintainability
- PASS: 使用页面对象模型以提高可维护性
- PASS: Use data-testid attributes for selectors
- PASS: 使用 data-testid 属性作为选择器
- PASS: Wait for API responses, not arbitrary timeouts
- PASS: 等待 API 响应，而非任意超时
- PASS: Test critical user journeys end-to-end
- PASS: 端到端测试关键用户旅程
- PASS: Run tests before merging to main
- PASS: 合并到主分支前运行测试
- PASS: Review artifacts when tests fail
- PASS: 测试失败时查看产出物

**DON'T:**
**不应该：**
- FAIL: Use brittle selectors (CSS classes can change)
- FAIL: 使用脆弱的选择器（CSS 类名可能变化）
- FAIL: Test implementation details
- FAIL: 测试实现细节
- FAIL: Run tests against production
- FAIL: 在生产环境运行测试
- FAIL: Ignore flaky tests
- FAIL: 忽略不稳定测试
- FAIL: Skip artifact review on failures
- FAIL: 测试失败时跳过产出物审查
- FAIL: Test every edge case with E2E (use unit tests)
- FAIL: 用 E2E 测试所有边界情况（应使用单元测试）

## Important Notes
## 重要说明

**CRITICAL for PMX:**
**PMX 关键注意事项：**
- E2E tests involving real money MUST run on testnet/staging only
- 涉及真实资金的 E2E 测试必须仅在测试网/预发布环境运行
- Never run trading tests against production
- 切勿在生产环境运行交易测试
- Set `test.skip(process.env.NODE_ENV === 'production')` for financial tests
- 对金融测试设置 `test.skip(process.env.NODE_ENV === 'production')`
- Use test wallets with small test funds only
- 仅使用包含少量测试资金的测试钱包

## Integration with Other Commands
## 与其他命令的集成

- Use `/plan` to identify critical journeys to test
- 使用 `/plan` 识别需要测试的关键用户旅程
- Use `/tdd` for unit tests (faster, more granular)
- 使用 `/tdd` 进行单元测试（更快、更细粒度）
- Use `/e2e` for integration and user journey tests
- 使用 `/e2e` 进行集成和用户旅程测试
- Use `/code-review` to verify test quality
- 使用 `/code-review` 验证测试质量

## Related Agents
## 相关代理

This command invokes the `e2e-runner` agent provided by ECC.
此命令调用 ECC 提供的 `e2e-runner` 代理。

For manual installs, the source file lives at:
对于手动安装，源文件位于：
`agents/e2e-runner.md`

## Quick Commands
## 快速命令

```bash
# Run all E2E tests
npx playwright test

# Run specific test file
npx playwright test tests/e2e/markets/search.spec.ts

# Run in headed mode (see browser)
npx playwright test --headed

# Debug test
npx playwright test --debug

# Generate test code
npx playwright codegen http://localhost:3000

# View report
npx playwright show-report
```
