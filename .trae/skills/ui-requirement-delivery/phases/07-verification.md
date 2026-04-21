# Phase 7：验证循环

## 目标
提 PR 之前确保所有质量门禁通过。七个阶段按顺序执行，任一失败立即停止修复。

---

## Phase 7.1：构建验证

使用项目已有的构建命令（从 Phase 2 确认）。

**失败处理**：立即停止，修复后从头重跑验证循环。

---

## Phase 7.2：类型检查

使用项目已有的类型检查命令（从 Phase 2 确认，如 `tsc --noEmit` / `vue-tsc` / `svelte-check` 等）。

**类型质量红线**

| 红线 | 说明 |
|---|---|
| 禁止无理由的 `any` | 使用更精确的类型或 `unknown` + 类型守卫替代 |
| 禁止用类型断言规避错误 | 修复类型本身 |
| 公共函数必须有返回类型 | 防止隐式宽泛类型推断 |
| 响应式依赖 / 副作用依赖必须完整 | 根据项目框架规范（如 React deps 数组、Vue watch 依赖） |

---

## Phase 7.3：Lint 检查

使用项目已有的 Lint 命令（从 Phase 2 确认）。

**代码质量红线**

| 红线 | 处理方式 |
|---|---|
| 单函数 > 50 行 | 拆分为更小聚焦的函数 |
| 单文件 > 800 行 | 按职责抽取模块 |
| 嵌套 > 4 层 | 使用提前返回（early return）|
| `console.log` 残留 | 必须删除 |
| 未处理的 Promise 拒绝 | 补加 `.catch()` 或 `try/catch` |

> **禁止**：修改 Lint / 类型检查配置文件来让检查通过。只允许修复代码本身。

---

## Phase 7.4：测试套件

使用项目已有的测试覆盖率命令（从 Phase 2 确认）。

目标覆盖率：≥ 80% branches / functions / lines

---

## Phase 7.5：安全扫描

搜索源码中的安全风险（根据项目实际文件类型调整搜索范围）：

```bash
# 检查硬编码密钥
grep -rn "api_key\|password\|secret\|token" src/

# 检查调试日志
grep -rn "console\.log" src/
```

**安全红线**

| 风险 | 要求 |
|---|---|
| XSS | 用户输入禁止直接渲染为 HTML（使用框架提供的安全渲染方式） |
| CSRF | 状态变更操作必须携带 CSRF Token 或等效鉴权机制 |
| 权限 | 受保护路由必须前后端双重校验 |
| 日志 | 禁止记录 Token / 密码 / PII |

---

## Phase 7.6：无障碍（a11y）扫描

> Phase 1 / Phase 3 声明了 WCAG AA 要求，本步骤在代码层进行闭环验证。
> 若项目无 UI 变更，可跳过。

### 自动化扫描（推荐优先）

在已有测试文件中集成 `axe-core`，对关键页面/组件跑一次 a11y 规则扫描。

> 根据项目框架选择对应集成包（React / Vue / 原生均有对应方案），若项目尚未引入，先确认是否允许新增测试依赖。

```
// 示例：在组件测试中集成 axe
import { axe } from '<axe 集成包>'

it('无 a11y 违规', async () => {
  <渲染组件>
  const results = await axe(<容器>)
  expect(results).toHaveNoViolations()
})
```

**axe-core 自动能捕获的典型问题**：
- 图片缺少 `alt` 属性
- 表单输入缺少关联 `label`
- 颜色对比度不足（WCAG AA < 4.5:1）
- ARIA 属性值无效或使用场景错误
- 页面缺少 `<h1>` / 标题层级跳跃

### 人工快速检查（自动化覆盖不到的部分）

用键盘 Tab 键逐步遍历页面，确认：
1. 所有可交互元素均可通过 Tab 聚焦
2. 焦点顺序符合视觉阅读顺序
3. 弹窗打开后焦点进入弹窗内；关闭后焦点回到触发元素
4. 表单报错时焦点移至错误提示（或错误提示可被 screen reader 读到）

**本步骤失败标准**：axe-core 报告存在 `critical` 或 `serious` 级别违规。
`moderate` / `minor` 级别记录到 PR 描述的跟进清单，不阻塞合并。

---

## Phase 7.7：Diff 审查

```bash
git diff --stat
git diff HEAD~1 --name-only
```

检查每个变更文件：
- 是否有意外变更（不相关文件被修改）
- 是否缺少错误处理
- 是否存在潜在边界问题

---

## 验证报告输出格式

```
VERIFICATION REPORT
===================
Build:     [PASS/FAIL]
Types:     [PASS/FAIL] (X 个错误)
Lint:      [PASS/FAIL] (X 个警告)
Tests:     [PASS/FAIL] (X/Y 通过, Z% 覆盖率)
Security:  [PASS/FAIL] (X 个问题)
A11y:      [PASS/SKIP/FAIL] (critical/serious: X 个，moderate/minor: Y 个)
Diff:      X 个文件变更

Overall:   [READY / NOT READY] for PR

待修复问题:
1. ...
2. ...
```

---

## 输出检查

- [ ] 所有阶段均通过（有 UI 变更时包含 Phase 7.6 a11y 扫描）
- [ ] 覆盖率 ≥ 80%
- [ ] 无 Security 红线问题
- [ ] A11y：无 `critical` / `serious` 级别违规；`moderate` / `minor` 已记录跟进
- [ ] Diff 中无意外变更
