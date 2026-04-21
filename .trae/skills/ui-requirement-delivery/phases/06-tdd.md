# Phase 6：TDD 开发循环

## 原则

**先写测试，再写实现。测试不是可选项。**

---

## Red → Green → Refactor 循环

### Step 1：写用户旅程

```
作为 [角色]，我希望 [操作]，以便 [价值]

示例（根据 PRD 实际需求填写）：
作为 [具体角色]，我希望 [完成某个操作]，
以便 [实现某个业务目标]。
```

### Step 2：生成测试用例

覆盖：正常路径 / 边界值 / 异常态 / 权限态

```
describe('<ComponentName>', () => {
  it('[正常路径描述]')
  it('[边界值描述]')
  it('[异常态描述]')
  it('[权限态描述]')
  it('[错误恢复描述]')
})
```

> 测试用例来自 Phase 1 状态机结论 + Phase 3 组件映射的校验规则与权限态，不要凭空编写。

### Step 3：运行测试（必须 RED）

使用项目已有的测试运行命令（从 Phase 2 侦察结果中确认）。

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

之前失败的测试必须通过。确认 GREEN 后创建 Git checkpoint：

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

使用项目已有的覆盖率命令（从 Phase 2 确认）。

目标：≥ 80% branches / functions / lines / statements

---

## 测试层次与归属

| 测试层次 | 负责 Phase | 说明 |
|---|---|---|
| 接口层 | **Phase 5b** | Mock HTTP 请求，验证参数构造和响应映射 |
| 逻辑单元（hook / composable / store） | **Phase 6（本 Phase）** | Mock 接口层，验证状态流转和副作用 |
| UI 组件 | **Phase 6（本 Phase）** | Mock 逻辑单元，验证用户可见行为 |
| E2E 链路 | **Phase 8** | 真实浏览器，不 Mock，验证完整用户旅程 |

> **原则**：每层只 Mock 其直接下层，不跨层 Mock。
> 接口层测试若已在 Phase 5b 完成，Phase 6 无需重复覆盖。

---

## 单元测试规范

### AAA 结构（每个测试块必须遵守）

```
it('<业务场景描述>', () => {
  // Arrange — 准备数据和前置条件
  <渲染组件或初始化逻辑单元，传入测试所需 Props / 参数>

  // Act — 执行用户操作
  <触发用户交互或调用被测函数>

  // Assert — 验证用户可见结果（不验证内部状态）
  <断言 UI 输出或返回值>
})
```

### 测试命名：描述业务场景，不描述实现

```
// ✓ 描述用户可见行为
it('空值时允许提交')
it('无操作权限时操作按钮不可见')
it('提交失败时保留当前编辑态并显示错误')

// ✗ 描述实现细节
it('test form validation')
it('works correctly')
it('setState is called')
```

### `data-testid` 标注规范

> E2E 测试（Phase 8）依赖 `data-testid` 选择器。**必须在本 Phase 实现组件时同步添加**，
> 不要等到 Phase 8 写测试时才发现缺失，届时需要返工改组件。

需要标注 `data-testid` 的元素类型：

| 元素类型 | 命名示例 |
|---|---|
| 关键操作按钮 | `data-testid="<action>-btn"` |
| 表单输入项（无语义 label 时） | `data-testid="<field>-input"` |
| 数据展示容器 | `data-testid="<resource>-list"` / `data-testid="empty-state"` |
| 状态反馈元素 | `data-testid="error-toast"` / `data-testid="loading-spinner"` |
| 弹窗 / 抽屉 | `data-testid="<purpose>-modal"` |

命名规则：`kebab-case`，语义自解释，禁止用 `test-1`、`btn-a` 等无意义编号。

---

### Mock 规则

- 外部接口调用必须 Mock（直接下层，不跨层）
- 禁止 Mock 被测单元本身
- Mock 函数需要验证调用参数和调用次数

```
const mock<Action> = <mockFn>().mockResolvedValue(undefined)
// 验证调用
expect(mock<Action>).toHaveBeenCalledWith(<expectedPayload>)
expect(mock<Action>).toHaveBeenCalledTimes(1)
```

### 禁止反模式

- 固定时间等待 → 改用条件等待
- 测试之间共享可变状态 → 每个测试独立初始化
- 断言内部实现（组件内部 state）→ 只断言用户可见行为
- 用 `index` 作为 key 的列表 snapshot 测试 → 用语义断言替代

---

## 输出检查

- [ ] 每个新功能点有对应测试用例
- [ ] RED → GREEN → Refactor 三个 checkpoint 均已创建
- [ ] 覆盖率 ≥ 80%
- [ ] 正常路径、边界值、异常态、权限态均有覆盖
- [ ] 关键操作按钮、表单输入、数据容器均已添加 `data-testid`
- [ ] 接口层测试未重复覆盖（已在 Phase 5b 完成）
