# Phase 8：E2E 测试

## 目标
覆盖关键用户链路，确保主流程端到端可用。

---

## 前置确认

> 不要预设 E2E 框架——根据 Phase 2 侦察结果确认项目已有的 E2E 方案（Playwright / Cypress / Selenium 等），遵循项目已有的目录结构和文件命名规范。

若项目尚无 E2E 测试，先确认是否允许引入，再参照项目技术栈选择方案。

---

## 文件组织

> 参照项目已有 E2E 目录结构。若无既有结构，通常按以下方式组织：

```
<e2e 目录>/
├── <功能模块>/
│   └── <feature>.<spec 后缀>
└── <框架配置文件>
```

---

## Page Object Model（推荐使用）

将页面操作封装为类/对象，测试代码只调用方法，不直接写选择器——这使选择器变更时只需修改一处。

```
// 示例结构（语言/框架根据项目实际调整）
class <FeaturePage> {
  // 声明页面元素定位器
  <saveButton> = <locator('[data-testid="<action>-btn"]')>
  <editButton> = <locator('[data-testid="<action>-btn"]')>

  // 页面导航
  async goto() {
    <navigate to feature URL>
    <wait for page ready>
  }

  // 操作封装
  async <performAction>(<params>) {
    <操作步骤>
  }
}
```

---

## 必须覆盖的链路

> 下表为通用链路模板，根据 Phase 0 PRD 和 Phase 3 组件映射中标注的业务场景动态补充。

| 链路类型 | 覆盖内容 |
|---|---|
| 主链路 | 完整的核心操作流程 → 数据展示与预期一致 |
| 权限链路 | 无权限用户无法进入受保护操作；URL 越权被拦截 |
| 鉴权流程链路 | 若 PRD 包含鉴权步骤（如二次验证），覆盖完整验证流程 |
| 异常链路 | 接口失败时 UI 可感知（错误态）且可恢复（重试） |
| （业务特有链路） | 从 Phase 0 REQ 中提取，此处动态补充 |

---

## 选择器规范

> **前提**：`data-testid` 属性应在 **Phase 6 实现组件时**同步添加，不是在本 Phase 补加。
> 若此时发现组件缺少 `data-testid`，需返回 Phase 6 修改组件，再继续 E2E 编写。

```
// ✗ 错误：CSS 类名（脆弱，样式改变即失效）
<click '.css-abc123'>

// ✗ 错误：固定时间等待（不稳定）
<wait 5000ms>

// ✓ 正确：语义选择器（健壮）
<click 'button:has-text("<ButtonLabel>")'>
<click '[data-testid="<action>-btn"]'>

// ✓ 正确：等待具体网络条件
<waitForResponse url includes '<api-path>' and status 200>

// ✓ 正确：等待元素状态
<waitFor '[data-testid="<container>"]' to be visible>
```

---

## E2E 框架配置要点

根据项目实际 E2E 框架，确认以下配置已就位：

| 配置项 | 说明 |
|---|---|
| `baseURL` | 从环境变量注入，不硬编码 |
| 重试策略 | CI 环境建议 2 次重试，本地 0 次 |
| 失败截图 / 录像 | 仅失败时保留，减少存储消耗 |
| 并行执行 | 开启以缩短 CI 时间 |

---

## CI/CD 集成

E2E 测试应在 CI 中自动运行，并在失败时上传测试报告 artifact，便于排查。

具体 CI 配置参照项目已有 CI 脚本风格添加，核心要点：
- 使用环境变量注入 `BASE_URL`
- 失败时上传测试报告和截图/录像

---

## 不稳定测试处理

不稳定（flaky）测试应及时标记，避免阻塞 CI，并关联 issue 跟进修复：

```
// 标记不稳定测试（根据框架语法调整）
<mark test as flaky, reason: 'Flaky - Issue #<issue-number>'>
```

---

## 输出检查

- [ ] 主链路 E2E 通过
- [ ] 权限链路已覆盖
- [ ] 异常链路已覆盖
- [ ] PRD 中标注的特殊鉴权 / 业务流程链路已覆盖
- [ ] 无固定时间等待
- [ ] 所有选择器使用 `data-testid` 或语义选择器
- [ ] CI 中 E2E 已集成，失败时上传 artifact
