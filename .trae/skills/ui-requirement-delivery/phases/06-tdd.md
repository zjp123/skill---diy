# Phase 6：TDD 开发循环

## 原则

**先写测试，再写实现。测试不是可选项。**

---

## Red → Green → Refactor 循环

### Step 1：写用户旅程

```
作为 [角色]，我希望 [操作]，以便 [价值]

示例：
作为配置管理员，我希望保存汇率前通过 2FA 验证，
以便防止未授权修改。
```

### Step 2：生成测试用例

覆盖：正常路径 / 边界值 / 异常态 / 权限态

```typescript
describe('ExchangeRateSettingCard', () => {
  it('空值时允许提交')
  it('超出 0.01~10.00 范围时显示错误提示')
  it('非 2 位小数时显示错误提示')
  it('无编辑权限时 Edit 按钮不可见')
  it('2FA 未通过时不触发保存请求')
  it('保存成功后回到 view 态')
  it('保存失败时保留 edit 态并提示错误')
})
```

### Step 3：运行测试（必须 RED）

```bash
npm test
# 测试必须失败 — 尚未实现
```

**RED 验证规则**：
- 测试必须被实际执行并失败（只写未运行不算）
- 失败原因必须是业务逻辑缺失，不能是语法错误或环境问题
- 确认 RED 后创建 Git checkpoint：

```bash
git commit -m "test: add reproducer for <feature>"
```

### Step 4：实现最小代码

写刚好能让测试通过的代码，**不做过度设计**。

### Step 5：运行测试（必须 GREEN）

```bash
npm test
# 之前失败的测试必须通过
```

确认 GREEN 后创建 Git checkpoint：

```bash
git commit -m "fix: implement <feature>"
```

### Step 6：重构

保持测试绿色前提下：
- 消除重复（DRY）
- 改善命名
- 提取公共逻辑
- 优化性能（仅有数据支撑时）

```bash
git commit -m "refactor: clean up <feature>"
```

### Step 7：覆盖率验证

```bash
npm run test:coverage
# 目标：≥ 80% branches / functions / lines / statements
```

---

## 测试层次与归属

| 测试层次 | 负责 Phase | 说明 |
|---|---|---|
| 服务层（`services/api.ts`）| **Phase 5b** | Mock HTTP 请求，验证参数构造和响应映射 |
| Hooks / 业务逻辑 | **Phase 6（本 Phase）** | Mock 服务层，验证状态流转和副作用 |
| UI 组件 | **Phase 6（本 Phase）** | Mock Hooks，验证用户可见行为 |
| E2E 链路 | **Phase 8** | 真实浏览器，不 Mock，验证完整用户旅程 |

> **原则**：每层只 Mock 其直接下层，不跨层 Mock。
> 服务层测试若已在 Phase 5b 完成，Phase 6 无需重复覆盖 `services/api.ts`。

---

## 单元测试规范

### AAA 结构（每个 `it` 块必须遵守）

```typescript
it('超出范围时显示错误提示', () => {
  // Arrange — 准备数据和前置条件
  render(<ExchangeRateSettingCard editing value={{}} onSave={jest.fn()} />)

  // Act — 执行用户操作
  fireEvent.change(screen.getByLabelText('Cross Currency Rate'), {
    target: { value: '99' },
  })
  fireEvent.click(screen.getByRole('button', { name: 'Save' }))

  // Assert — 验证用户可见结果（不验证内部状态）
  expect(screen.getByText('汇率要求0.01～10.00 %内，请重新输入')).toBeInTheDocument()
})
```

### 测试命名：描述业务场景，不描述实现

```typescript
// ✓ 描述用户可见行为
it('空值时允许保存')
it('无编辑权限时 Edit 按钮不可见')
it('2FA 未通过时不触发保存请求')

// ✗ 描述实现细节
it('test form validation')
it('works correctly')
it('setState is called')
```

### `data-testid` 标注规范

> E2E 测试（Phase 8）依赖 `data-testid` 选择器。**必须在本 Phase 实现组件时同步添加**，
> 不要等到 Phase 8 写测试时才发现缺失，届时需要返工改组件。

需要标注 `data-testid` 的元素类型：

| 元素类型 | 示例 |
|---|---|
| 关键操作按钮 | `data-testid="save-btn"` / `data-testid="edit-btn"` |
| 表单输入项（无语义 label 时） | `data-testid="rate-input"` |
| 数据展示容器 | `data-testid="result-table"` / `data-testid="empty-state"` |
| 状态反馈元素 | `data-testid="error-toast"` / `data-testid="loading-spinner"` |
| 弹窗 / 抽屉 | `data-testid="confirm-modal"` |

命名规则：`kebab-case`，语义自解释，禁止用 `test-1`、`btn-a` 等无意义编号。

---

### Mock 规则

- 外部 API 调用必须 Mock（直接下层，不跨层）
- 禁止 Mock 被测单元本身
- Mock 函数需要验证调用参数和调用次数

```typescript
const mockSaveConfig = jest.fn().mockResolvedValue(undefined)
// 验证调用
expect(mockSaveConfig).toHaveBeenCalledWith({ crossCurrencyRate: 0.5 })
expect(mockSaveConfig).toHaveBeenCalledTimes(1)
```

### 禁止反模式

- `waitForTimeout` 固定等待 → 改用 `waitFor` 条件等待
- 测试之间共享可变状态 → 每个测试独立初始化
- 断言内部实现（`component.state`）→ 只断言用户可见行为
- 用 `index` 作为 key 的列表 snapshot 测试 → 用语义断言替代

---

## 输出检查

- [ ] 每个新功能点有对应测试用例
- [ ] RED → GREEN → Refactor 三个 checkpoint 均已创建
- [ ] 覆盖率 ≥ 80%
- [ ] 正常路径、边界值、异常态、权限态均有覆盖
- [ ] 关键操作按钮、表单输入、数据容器均已添加 `data-testid`
- [ ] 服务层（`services/api.ts`）测试未重复覆盖（已在 Phase 5b 完成）
