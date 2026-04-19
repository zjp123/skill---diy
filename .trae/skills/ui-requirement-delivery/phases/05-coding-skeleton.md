# Phase 5：代码骨架

## 目标
按目录与文件生成可直接开工的代码骨架，输出 `ui-coding-skeleton-plan.md`。

---

## 标准目录结构

```text
src/pages/<feature>/
├── index.tsx               # 页面容器（状态编排，不含业务逻辑）
├── style.module.less        # CSS Modules 样式
├── constants.ts             # 常量与枚举
├── types.ts                 # TypeScript 类型定义
├── hooks/
│   ├── use<Feature>.ts      # 主业务 Hook
│   └── use<Feature>.test.ts # Hook 单元测试
├── components/
│   ├── <WidgetA>.tsx
│   ├── <WidgetA>.test.tsx   # 组件单元测试（并置）
│   └── <WidgetB>.tsx
└── services/
    ├── api.ts               # 接口层（只写签名，不实现；有接口文档时由 Phase 5b 填充实现）
    └── api-paths.ts         # API 路径常量（Phase 5b 创建或确认）

e2e/
└── <feature>.spec.ts        # E2E 测试
```

---

## 骨架文件要求

每个文件必须包含：
1. 文件职责说明（单行注释）
2. 完整的 TypeScript 类型 / Props / 方法签名
3. `throw new Error('todo')` 占位（不实现业务逻辑）
4. 关联的 REQ 编号（注释形式）

```typescript
// 职责：汇率配置页主 Hook，管理品牌切换与保存流程
// 关联：REQ-003, REQ-004, REQ-005
export function useCommissionExchangeRate() {
  // TODO: 实现
  throw new Error('todo')
}
```

---

## 命名规范（骨架阶段确定，后续不随意变更）

### 函数：动宾结构，语义自解释

```typescript
// ✓ 好的命名
async function fetchRateConfig(brand: BrandCode): Promise<RateConfig>
function calculateTotalProfit(rows: ProfitRow[]): number
function isValidRateInput(value: number | null): boolean

// ✗ 错误的命名
async function rateConfig(id: any)
function calc(a, b)
```

### 变量：充分描述，禁止缩写和单字母（循环变量除外）

```typescript
// ✓
const activeBrandCode = 'VG'
const isEditingRateForm = false
const totalProfitUSD = 12345.67

// ✗
const x = 'VG'
const flag = false
const tmp = 12345.67
```

### 其他命名规则
- **组件**：PascalCase，文件名与导出名一致
- **Hooks**：`use` 前缀 + 功能描述（`useCommissionExchangeRate`）
- **类型**：PascalCase，`interface` 用于结构体，`type` 用于联合 / 工具类型
- **常量**：UPPER_SNAKE_CASE（`MAX_RETRIES`、`DEBOUNCE_DELAY_MS`）

---

## 文件 → REQ 映射表

| 文件 | 覆盖 REQ | 审查重点 |
|---|---|---|
| `index.tsx` | REQ-004, REQ-005 | 状态流转、2FA 串联 |
| `hooks/useFeature.ts` | REQ-003, REQ-004 | 请求编排、状态策略 |
| `components/SettingCard.tsx` | REQ-003, REQ-004 | 三字段校验与编辑态 |
| `services/api.ts` | REQ-003, REQ-005 | 接口签名一致性 |

---

## 编码顺序建议

1. `types.ts` + `constants.ts`（类型优先，确保全局对齐）
2. `services/api.ts`（**此阶段只写函数签名 + 类型，不实现业务逻辑**）
   - 若已提供后端接口文档 → 暂停，进入 **Phase 5b** 完成实现后再继续
   - 若无接口文档 → 保留 `throw new Error('todo')` 占位，联调时补齐
3. `hooks/use<Feature>.ts`（数据流先通）
4. 叶子组件（纯 UI，无内部状态）
5. 容器组件 + 业务逻辑接入
6. 权限 & 国际化收尾

---

## 输出检查

- [ ] 每个变更文件可反查到至少一个 `REQ-*`
- [ ] 类型定义无 `any`
- [ ] 函数命名均为动宾结构
- [ ] 测试文件与被测文件并置
- [ ] 骨架可直接开工，不依赖口头补充
- [ ] `services/api.ts` 的处理已明确：有接口文档则转入 Phase 5b；无则保留签名占位
