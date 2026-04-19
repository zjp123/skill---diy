# Phase 8：E2E 测试

## 目标
覆盖关键用户链路，确保主流程端到端可用。

---

## 文件组织

```
e2e/
├── auth/
│   └── login.spec.ts
├── features/
│   └── <feature>.spec.ts
└── playwright.config.ts
```

---

## Page Object Model（必须使用）

将页面操作封装为类，测试代码只调用方法，不直接写选择器。

```typescript
import { Page, Locator } from '@playwright/test'

export class FeaturePage {
  readonly page: Page
  readonly saveButton: Locator
  readonly editButton: Locator

  constructor(page: Page) {
    this.page = page
    this.saveButton = page.locator('[data-testid="save-btn"]')
    this.editButton = page.locator('[data-testid="edit-btn"]')
  }

  async goto() {
    await this.page.goto('/feature')
    await this.page.waitForLoadState('networkidle')
  }

  async clickEdit() {
    await this.editButton.click()
  }

  async fillAndSave(value: string) {
    await this.page.locator('[data-testid="rate-input"]').fill(value)
    await this.saveButton.click()
  }
}
```

---

## 必须覆盖的链路

| 链路 | 覆盖内容 |
|---|---|
| 主链路 | 完整保存流程 → 数据展示一致 |
| 权限链路 | 无权限用户无法进入编辑态；URL 越权拦截 |
| 2FA 链路 | 未绑定走绑定流程；已绑定走验证码流程 |
| 异常链路 | 接口失败时 UI 可感知（error 态）且可恢复（重试） |

---

## 选择器规范

> **前提**：`data-testid` 属性应在 **Phase 6 实现组件时**同步添加，不是在本 Phase 补加。
> 若此时发现组件缺少 `data-testid`，需返回 Phase 6 修改组件，再继续 E2E 编写。

```typescript
// ✗ 错误：CSS 类名（脆弱，样式改变即失效）
await page.click('.css-abc123')

// ✗ 错误：固定等待（不稳定）
await page.waitForTimeout(5000)

// ✓ 正确：语义选择器（健壮）
await page.click('button:has-text("Save")')
await page.click('[data-testid="save-btn"]')

// ✓ 正确：等待具体网络条件
await page.waitForResponse(resp => resp.url().includes('/api/save') && resp.status() === 200)

// ✓ 正确：等待元素状态
await page.locator('[data-testid="result-table"]').waitFor({ state: 'visible' })
```

---

## Playwright 配置

```typescript
// playwright.config.ts
export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
})
```

---

## CI/CD 集成

```yaml
- name: E2E Tests
  run: npx playwright test
  env:
    BASE_URL: ${{ vars.STAGING_URL }}

- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: playwright-report
    path: playwright-report/
    retention-days: 30
```

---

## 不稳定测试处理

```typescript
// 标记不稳定测试，避免阻塞 CI
test('flaky: complex flow', async ({ page }) => {
  test.fixme(true, 'Flaky - Issue #123')
  // ...
})

// 仅在 CI 跳过
test.skip(!!process.env.CI, 'Flaky in CI - Issue #123')
```

---

## 输出检查

- [ ] 主链路 E2E 通过
- [ ] 权限链路已覆盖
- [ ] 异常链路已覆盖
- [ ] 无 `waitForTimeout` 固定等待
- [ ] 所有选择器使用 `data-testid` 或语义选择器
- [ ] CI 中 E2E 已集成，失败时上传 artifact
