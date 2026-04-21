# Phase 9：Code Review

## 触发条件
- 开发完成、验证循环（Phase 7）全绿、E2E（Phase 8）通过后
- 正式提 PR 之前

---

## PR 模板（必须填写）

```markdown
## 变更描述
<!-- 简述本 PR 解决了什么问题 -->

## 关联需求
Implements: REQ-001, REQ-002

## 测试说明
Tests: UT case-001~005, E2E spec-feature-001

## 风险等级
Risk: high / medium / low

## 截图（如有 UI 变更）
| Before | After |
|--------|-------|
| 截图   | 截图  |
```

---

## 评审清单

### CRITICAL — 安全（必须阻塞合并）

- [ ] 无硬编码 API Key / 密码 / Token
- [ ] 无 XSS（用户输入未经净化直接渲染为 HTML）
- [ ] 权限受保护路由有前后端双重校验
- [ ] 鉴权敏感流程完整，无绕过入口（如存在二次验证等安全机制）
- [ ] 无敏感信息写入日志

### HIGH — 代码质量（建议修复后合并）

- [ ] 无无理由的 `any` / 类型断言强制转换
- [ ] 响应式依赖 / 副作用依赖完整（根据项目框架规范检查，如 React deps 数组、Vue watch 依赖等）
- [ ] 列表使用稳定唯一 key（非 index）
- [ ] 无未处理的 Promise 拒绝
- [ ] 无空 `catch` 块（至少记录错误信息）
- [ ] 函数 ≤ 50 行，文件 ≤ 800 行
- [ ] 状态机完整（覆盖 Phase 1 中枚举的所有状态；若 Phase 1 标注「无状态变化」则此项 N/A）
- [ ] 数据请求子树有错误边界或等效容错机制包裹

### HIGH — 框架特有模式（根据 Phase 2 确认的框架检查）

> 此节内容根据项目实际框架动态应用，不适用的条目标注 N/A。

- [ ] 无渲染阶段触发状态更新（导致无限循环）
- [ ] 无陈旧闭包 / 过期引用（事件处理器捕获了过期状态或响应式数据）
- [ ] 高开销计算已缓存，避免每次渲染重复执行
- [ ] 数据请求有 loading / error / empty 三态 UI
- [ ] 无异步数组遍历陷阱（确保 async 操作被正确 await）

### MEDIUM — 性能

- [ ] 长列表（> 100 条）有虚拟滚动
- [ ] 无不必要的整库导入（使用具名导入）
- [ ] 无重复数据请求（合并或使用项目已有的请求缓存方案）
- [ ] 无渲染中创建对象/数组作为 Props（导致不必要重渲染）

### MEDIUM — 无障碍（a11y）

- [ ] 交互元素（按钮、链接、输入框）均可通过键盘 Tab 聚焦
- [ ] 图标按钮 / 仅图标的可操作元素有 `aria-label`
- [ ] 表单输入有关联 `<label>`（或 `aria-labelledby`）
- [ ] 错误提示通过 `role="alert"` 或 `aria-live` 可被 screen reader 读取
- [ ] 弹窗/抽屉打开后焦点进入内部；关闭后焦点回到触发元素
- [ ] Phase 7.6 axe-core 扫描无 `critical` / `serious` 违规

### LOW — 可读性

- [ ] 无魔法数字（使用具名常量）
- [ ] 无残留 `console.log`
- [ ] TODO/FIXME 必须关联 issue 编号

---

## AI 回归陷阱专项

> AI 写代码 + AI 审代码 = 相同盲点被忽视两次。
> 以下 4 种 pattern 是 AI 最高频制造的回归，必须用**自动化测试捕获**，不能依赖人工审查。

### Pattern 1：沙箱 / 生产路径不一致（最高频）

```
// ✗ AI 常见错误：只修了生产路径，沙箱/测试路径遗漏新字段
if (isSandboxMode()) {
  return { data: { <existingFields> } }           // ← 新字段缺失
}
return { data: { <existingFields>, <newField> } }  // 生产路径有

// ✓ 两条路径必须返回相同结构
if (isSandboxMode()) {
  return { data: { <existingFields>, <newField>: null } }
}
return { data: { <existingFields>, <newField> } }
```

**捕获方式**：在沙箱/测试模式下断言所有必需字段全部存在。

### Pattern 2：查询字段遗漏

```
// ✗ 新增字段到响应类型，但忘记加到查询 select
const result = await <queryBuilder>
  .select('<existingFields>')  // <newField> 没加
  .where(...)

// ✓ 显式包含新字段
.select('<existingFields>, <newField>')
```

**捕获方式**：断言返回字段不为 `undefined` / `null`。

### Pattern 3：错误状态泄漏（旧数据残留）

```
// ✗ 报错后忘记清空上一次的数据，旧数据继续显示
catch (err) {
  setError('加载失败')
  // <list> 仍然显示上一次的数据！
}

// ✓ 错误时清空关联状态
catch (err) {
  set<List>([])
  setError('加载失败')
}
```

**捕获方式**：测试「切换上下文后接口报错」场景，断言列表清空。

### Pattern 4：乐观更新无回滚

```
// ✗ API 失败后 UI 已更新，前后端数据不一致
const handleSave = async (payload) => {
  set<Data>(payload)               // 乐观更新
  await api<Save>(payload)         // 失败了，UI 不会回滚
}

// ✓ 失败时回滚到之前的状态
const handleSave = async (payload) => {
  const prev<Data> = current<Data>
  set<Data>(payload)
  try {
    await api<Save>(payload)
  } catch {
    set<Data>(prev<Data>)           // 回滚
    <showErrorMessage>
  }
}
```

**捕获方式**：Mock API 抛出异常，断言 UI 恢复到原始值。

### 回归测试命名规范

每次因线上 bug 补写的回归测试，名称必须标注来源：

```
it('<场景描述> (BUG-R<编号> 回归)', async () => { /* ... */ })
```

---

## 评审结论格式

```
| 严重级别 | 数量 | 状态 |
|----------|------|------|
| CRITICAL | 0    | ✅   |
| HIGH     | 2    | ⚠️   |
| MEDIUM   | 3    | ℹ️   |
| LOW      | 1    | 📝   |

结论：WARNING — 合并前应先解决 2 个 HIGH 问题。
```

---

## 合并标准

| 结论 | 条件 | 操作 |
|---|---|---|
| Approve | 无 CRITICAL / HIGH | 可合并 |
| Warning | 仅 HIGH（非安全类） | 可谨慎合并，记录跟进 |
| Block | 有 CRITICAL | 必须修复后重新评审 |
