# Phase 2：代码库侦察

## 目标
在动手之前充分理解现有技术栈与可复用资源，避免重复造轮子。

---

## 检查内容

### 技术栈

| 层次 | 检查项 |
|---|---|
| 框架 | React / Vue / Next.js / Nuxt |
| UI 库 | Antd / MUI / Radix / shadcn |
| 状态管理 | Zustand / Redux / Context / Jotai |
| 请求层 | React Query / SWR / 自封装 hooks |
| 表单 | React Hook Form / Ant Form |
| 路由 | React Router / Next.js App Router |
| 测试框架 | Jest / Vitest / Playwright |
| 样式方案 | CSS Modules / Tailwind / styled-components |
| 埋点 / 监控 | Mixpanel / Segment / 自定义 `track()` / Sentry / Datadog |

### 可复用组件清单

优先检查以下类型是否已有现成实现：

- **布局**：Page / Card / Modal / Drawer / Tabs
- **表单**：Input / Select / DatePicker / InputNumber / Form.Item
- **表格**：BusinessTable / DataTable / ProTable
- **权限**：Auth / PermissionGuard / withPermission HOC
- **安全**：2FA 组件（BindTwoFaDialog / TwoFAFrom）
- **国际化**：i18n key 注入方式、翻译文件位置
- **埋点**：`track()` / `analytics.track()` 的调用入口文件路径、事件命名规范（snake_case / camelCase / 枚举）

### 代码证据要求

> 每个关键复用点**必须**有文件路径 + 行号支撑

```
AssetTabs.tsx → src/pages/dapManagement/components/AssetTabs.tsx#18-65
BusinessTable → packages/admin-ui/src/BusinessTable/index.tsx#1
```

---

## 禁止事项

- 禁止凭空假设第三方库已存在，必须先在 `package.json` 验证
- 禁止凭借记忆确定组件 API，必须阅读实际源码
- 禁止在未确认复用可行性前输出骨架

---

## 输出检查

- [ ] 技术栈结论已确定，有 `package.json` 证据
- [ ] 可复用组件清单已建立（含文件路径 + 行号）
- [ ] 无可复用时已明确标注「自定义开发」
- [ ] 可复用但需小改造的已标注「改造点」
- [ ] 埋点方案已确认（调用入口 + 事件命名规范），供 Phase 4 WBS 标注埋点事件名使用
