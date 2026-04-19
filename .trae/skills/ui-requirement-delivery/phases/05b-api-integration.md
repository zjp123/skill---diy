# Phase 5b：后端接口集成

## 目标

在代码骨架（Phase 5）创建的 `services/api.ts` 空签名基础上，结合后端接口文档与项目已有的接口规范，生成可直接投入 TDD 开发的接口实现代码。

---

## 触发条件

满足以下任一条件即进入本 Phase：

- 用户提供了后端 API 文档（Swagger / OpenAPI / Postman / Markdown 等格式）
- 用户描述了接口的路径、方法、入参、出参
- PRD 中明确标注了接口契约（Phase 0 产物中已有接口清单）

若无任何接口文档，本 Phase **跳过**，`services/api.ts` 保留骨架签名，由开发者在联调时补齐。

> **协议适配说明**
> 本 Phase 以 **REST API** 为主要场景（HTTP 方法、路径、状态码）。
> - **GraphQL**：类型定义从 Schema 自动生成（`graphql-codegen`），操作层替换为
>   `useQuery` / `useMutation`，错误处理参考响应体 `errors` 字段而非 HTTP 状态码，
>   Step 1 的接口清单改为 Operation 清单（Query / Mutation / Fragment）。
> - **tRPC**：类型由 Router 推断，无需手写 Request/Response 类型，
>   Step 3 可跳过，Step 4 改为注册 procedure 调用，错误处理统一用 `TRPCClientError`。
> - **gRPC-Web**：类型由 `.proto` 生成，Step 3 改为确认生成产物路径，其余步骤不变。

---

## 步骤

### Step 1：解析接口文档

逐条提取以下字段，建立**接口清单表**：

| 字段 | 说明 |
|---|---|
| 接口 ID | 自定义编号，如 `API-01`，与 `REQ-*` 关联 |
| HTTP 方法 | GET / POST / PUT / PATCH / DELETE |
| 路径 | 完整 path，含路径参数（如 `/users/:id`） |
| 请求参数 | Query / PathParam / Body，注明必填 / 选填 |
| 响应结构 | 成功响应 + 错误码列表 |
| 关联 REQ | 来自 Phase 0 的 `REQ-*` |

#### 接口清单示例

| API ID | 方法 | 路径 | 请求 | 响应 | 关联 REQ |
|---|---|---|---|---|---|
| API-01 | GET | `/api/v1/exchange-rates` | `brand: string` | `RateConfig[]` | REQ-003 |
| API-02 | PATCH | `/api/v1/exchange-rates/:id` | `{ rate: number }` | `RateConfig` | REQ-004, REQ-005 |

---

### Step 2：侦察项目接口规范

在开始生成代码前，必须先读取项目中已有的接口调用模式：

```bash
# 搜索现有的 services/ 或 api/ 目录，读取 2～3 个典型文件
find src -name "api.ts" -o -name "*.service.ts" | head -5
```

重点确认以下规范并记录在输出文档中：

| 规范项 | 需确认的内容 |
|---|---|
| HTTP 客户端 | `fetch` / `axios` / 封装的 `request` 工具函数 |
| Base URL 来源 | 环境变量名（如 `VITE_API_BASE`）/ 统一配置文件路径 |
| 请求头注入 | Token 在哪里附加（拦截器 / 每次手动传） |
| 错误处理模式 | 统一拦截器抛出 / 每个函数自己 try-catch / 返回 Result 类型 |
| 字段命名风格 | `camelCase` / `snake_case` / 是否有自动转换层 |
| 分页参数规范 | `page+size` / `offset+limit` / cursor 参数名 |
| 响应包装格式 | `{ data, code, message }` / 裸数据 / 自定义结构 |

---

### Step 3：生成 TypeScript 类型定义

在 `types.ts` 中新增所有接口相关的 Request / Response 类型，要求：

- 字段命名与项目规范一致（必要时添加 `snake_case → camelCase` 注释说明）
- 使用 `interface` 表达结构体，`type` 表达联合 / 工具类型
- 分页响应使用项目已有的泛型包装（如 `PageResult<T>`），不重复定义

```typescript
// 关联：API-01, REQ-003
export interface RateConfig {
  id: string
  brandCode: string
  rate: number
  updatedAt: string
}

// 关联：API-02, REQ-004
export interface UpdateRateRequest {
  rate: number
}

// 若项目已有 PageResult<T> 泛型，直接复用
export type RateConfigListResponse = PageResult<RateConfig>
```

---

### Step 4：实现 `services/api.ts`

以项目既有接口规范为模板，将 Phase 5 生成的空签名替换为实际实现。

**实现要求**：

1. **复用项目封装的 HTTP 工具函数**，不直接裸用 `fetch` / `axios`
2. **路径使用常量或枚举**，不散落魔法字符串
3. **错误处理与项目模式一致**（拦截器模式则不重复 try-catch）
4. **每个函数头部注释关联 `REQ-*` 和 `API-*`**

```typescript
// 关联：API-01, REQ-003
export async function fetchRateConfigList(
  brand: string
): Promise<RateConfig[]> {
  const data = await request.get<RateConfigListResponse>(
    API_PATHS.EXCHANGE_RATES,
    { params: { brand } }
  )
  return data.list
}

// 关联：API-02, REQ-004
export async function updateRateConfig(
  id: string,
  payload: UpdateRateRequest
): Promise<RateConfig> {
  return request.patch<RateConfig>(
    API_PATHS.EXCHANGE_RATE_BY_ID(id),
    payload
  )
}
```

#### API 路径常量示例

```typescript
// services/api-paths.ts（若项目尚未统一管理，在此文件中集中定义）
export const API_PATHS = {
  EXCHANGE_RATES: '/api/v1/exchange-rates',
  EXCHANGE_RATE_BY_ID: (id: string) => `/api/v1/exchange-rates/${id}`,
} as const
```

---

### Step 5：异常处理补充

针对接口文档中标注的每个错误码，补充前端处理策略，输出**错误码 → 前端行为映射表**：

| HTTP 状态码 | 业务含义 | 前端响应 |
|---|---|---|
| `400` | 参数校验失败 | Toast 展示 `error.message` |
| `401` | 未登录 / Token 失效 | 跳转登录页，清除本地 Token |
| `403` | 无操作权限 | Toast 提示"暂无权限" |
| `404` | 资源不存在 | 页面展示空态 |
| `409` | 数据冲突（并发写入） | Toast 提示并刷新列表 |
| `422` | 业务规则校验失败 | 提取 `details` 字段定位字段错误 |
| `429` | 接口限流 | Toast 提示"操作过于频繁，请稍后重试" |
| `5xx` | 服务端异常 | Toast 提示"服务异常，请稍后重试"，上报监控 |

---

### Step 6：Mock 数据准备（可选）

若联调环境尚未就绪，基于接口文档为每个接口生成 Mock 数据，保证 TDD 可在隔离环境中进行：

```typescript
// __mocks__/services/api.ts  或  测试辅助文件 test/fixtures/rate.ts
export const mockRateConfigList: RateConfig[] = [
  { id: '1', brandCode: 'VG', rate: 1.05, updatedAt: '2024-01-01T00:00:00Z' },
  { id: '2', brandCode: 'HG', rate: 0.98, updatedAt: '2024-01-01T00:00:00Z' },
]

export const mockUpdateRateRequest: UpdateRateRequest = { rate: 1.10 }
```

Mock 数据要求：
- 覆盖正常值、边界值、空列表三种场景
- 字段类型与真实接口一致，不使用 `any`
- 测试文件中通过 `vi.mock` / `jest.mock` 注入，不污染真实请求

---

## 输出物

| 文件 | 内容 |
|---|---|
| `types.ts`（追加） | Request / Response 类型定义 |
| `services/api.ts`（实现） | 完整的接口调用函数，替换空签名 |
| `services/api-paths.ts`（新增或确认已有） | API 路径常量集中管理 |
| `test/fixtures/*.ts`（可选） | Mock 数据文件 |
| `api-integration.md`（输出文档） | 接口清单表 + 项目规范确认 + 错误码映射表 |

---

## 输出检查

- [ ] 接口清单表已建立，每条接口关联 `REQ-*`
- [ ] 项目 HTTP 规范已确认（客户端 / Base URL / Token 注入方式）
- [ ] 所有 Request / Response 类型无 `any`，与接口文档字段一一对应
- [ ] `services/api.ts` 无魔法字符串，路径通过常量引用
- [ ] 错误码 → 前端行为映射表已输出
- [ ] 字段命名风格与项目规范一致（如有转换需在注释中标注）
- [ ] Mock 数据覆盖正常 / 边界 / 空列表三场景（若本 Phase 生成 Mock）
- [ ] 每个接口函数头部有 `REQ-*` 和 `API-*` 关联注释
- [ ] `services/api.ts` 中每个导出函数有对应单元测试：Mock HTTP 请求，验证参数构造正确、响应字段映射正确、错误时抛出符合预期的异常
