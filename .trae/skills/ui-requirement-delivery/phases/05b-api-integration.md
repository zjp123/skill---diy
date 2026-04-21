# Phase 5b：后端接口集成

## 目标

在代码骨架（Phase 5）创建的接口层空签名基础上，结合后端接口文档与项目已有的接口规范，生成可直接投入 TDD 开发的接口实现代码。

---

## 触发条件

满足以下任一条件即进入本 Phase：

- 用户提供了后端 API 文档（Swagger / OpenAPI / Postman / Markdown 等格式）
- 用户描述了接口的路径、方法、入参、出参
- PRD 中明确标注了接口契约（Phase 0 产物中已有接口清单）

若无任何接口文档，本 Phase **跳过**，接口层保留骨架签名占位，由开发者在联调时补齐。

> **协议适配说明**
> 本 Phase 以 **REST API** 为主要场景（HTTP 方法、路径、状态码）。
> - **GraphQL**：类型定义从 Schema 自动生成（codegen 工具），操作层替换为 Query / Mutation 调用，错误处理参考响应体 `errors` 字段而非 HTTP 状态码，Step 1 的接口清单改为 Operation 清单（Query / Mutation / Fragment）。
> - **tRPC**：类型由 Router 推断，无需手写 Request/Response 类型，Step 3 可跳过，Step 4 改为注册 procedure 调用，错误处理统一用框架提供的错误类型。
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

#### 接口清单格式（根据实际接口动态填写）

| API ID | 方法 | 路径 | 请求 | 响应 | 关联 REQ |
|---|---|---|---|---|---|
| API-01 | （方法） | （路径） | （入参描述） | （响应类型） | REQ-XXX |
| API-02 | （方法） | （路径） | （入参描述） | （响应类型） | REQ-YYY |

---

> **确认协议**：本 Phase 遇到以下情况时必须暂停：
> 后端字段命名与前端规范冲突 / 接口文档缺少错误码定义 / 响应结构与项目包装层不匹配。
> 提问格式见 `shared/user-confirmation-protocol.md`。

### Step 2：侦察项目接口规范

在开始生成代码前，必须先读取项目中已有的接口调用模式——搜索 `services/`、`api/`、`request/` 等目录，读取 2～3 个典型接口文件。

重点确认以下规范并记录在输出文档中：

| 规范项 | 需确认的内容 |
|---|---|
| HTTP 客户端 | 项目使用的请求工具（原生 fetch / 第三方库 / 自封装函数） |
| Base URL 来源 | 环境变量名 / 统一配置文件路径 |
| 请求头注入 | Token 在哪里附加（拦截器 / 每次手动传） |
| 错误处理模式 | 统一拦截器抛出 / 每个函数自己 try-catch / 返回 Result 类型 |
| 字段命名风格 | `camelCase` / `snake_case` / 是否有自动转换层 |
| 分页参数规范 | 参数名与结构（`page+size` / `offset+limit` / cursor） |
| 响应包装格式 | `{ data, code, message }` / 裸数据 / 自定义结构 |

---

### Step 3：生成类型定义

在项目类型文件中新增所有接口相关的 Request / Response 类型，要求：

- 字段命名与项目规范一致（必要时添加命名转换注释说明）
- 结构体优先用接口声明，联合 / 工具类型用类型别名
- 分页响应使用项目已有的泛型包装，不重复定义

**示例格式（语言/框架根据项目实际调整）：**

```
// 关联：API-01, REQ-XXX
export interface <ResponseType> {
  <field>: <type>
  // ...
}

// 关联：API-02, REQ-YYY
export interface <RequestType> {
  <field>: <type>
}

// 若项目已有分页泛型，直接复用：
export type <ListResponse> = <PageWrapper><<ResponseType>>
```

---

### Step 4：实现接口层

以项目既有接口规范为模板，将 Phase 5 生成的空签名替换为实际实现。

**实现要求**：

1. **复用项目封装的 HTTP 工具函数**，不直接裸用底层请求库
2. **路径使用常量**，不散落魔法字符串
3. **错误处理与项目模式一致**（拦截器模式则不重复 try-catch）
4. **每个函数头部注释关联 `REQ-*` 和 `API-*`**

**示例格式：**

```
// 关联：API-01, REQ-XXX
export async function <fetchXxx>(<params>): Promise<<ResponseType>> {
  return <httpClient>.<method>(<API_PATH.XXX>, { <params> })
}

// 关联：API-02, REQ-YYY
export async function <updateXxx>(
  id: <IdType>,
  payload: <RequestType>
): Promise<<ResponseType>> {
  return <httpClient>.<method>(<API_PATH.XXX_BY_ID(id)>, payload)
}
```

**API 路径常量格式（若项目尚无统一管理，在此集中定义）：**

```
export const API_PATHS = {
  <RESOURCE>: '<path>',
  <RESOURCE_BY_ID>: (id: <IdType>) => `<path>/${id}`,
} as const
```

---

### Step 5：异常处理补充

针对接口文档中标注的每个错误码，结合**项目已有的错误处理模式**，补充前端处理策略，输出**错误码 → 前端行为映射表**。

> 下表为通用参考，具体行为必须与项目已有处理模式保持一致，不得自行发明新的错误处理方式。

| HTTP 状态码 / 业务码 | 业务含义 | 前端响应（参考，按项目实际调整） |
|---|---|---|
| `400` | 参数校验失败 | 展示错误提示信息 |
| `401` | 未登录 / Token 失效 | 跳转登录态（与项目已有鉴权流程保持一致） |
| `403` | 无操作权限 | 提示无权限（与项目已有权限降级方案保持一致） |
| `404` | 资源不存在 | 展示空态 |
| `409` | 数据冲突 | 提示冲突并刷新数据 |
| `422` | 业务规则校验失败 | 提取错误详情定位字段 |
| `429` | 接口限流 | 提示稍后重试 |
| `5xx` | 服务端异常 | 通用错误提示 + 上报监控（若项目有监控方案） |
| （业务自定义码） | （从接口文档中提取） | （从 Phase 3 错误处理策略中引用） |

---

### Step 6：Mock 数据准备（可选）

若联调环境尚未就绪，基于接口文档为每个接口生成 Mock 数据，保证 TDD 可在隔离环境中进行。

Mock 数据存放位置根据项目已有规范确认（`__mocks__/`、`test/fixtures/`、`src/mocks/` 等）。

**示例格式：**

```
// 关联：API-01
export const mock<ResourceList>: <ResponseType>[] = [
  { <field>: <normalValue>, ... },   // 正常值
  { <field>: <boundaryValue>, ... }, // 边界值
]

export const mock<EmptyList>: <ResponseType>[] = []  // 空列表
```

Mock 数据要求：
- 覆盖正常值、边界值、空列表三种场景
- 字段类型与真实接口一致，不使用 `any`
- 测试文件中通过项目测试框架提供的 mock 机制注入，不污染真实请求

---

## 输出物

> 文件路径根据 Phase 5 确认的目录结构填写，不预设固定路径。

| 产物 | 内容 |
|---|---|
| 类型文件（追加） | Request / Response 类型定义 |
| 接口层（实现） | 完整的接口调用函数，替换空签名 |
| API 路径常量文件（新增或确认已有） | 路径常量集中管理 |
| Mock 数据文件（可选） | 测试用 Fixture |
| `api-integration.md`（输出文档） | 接口清单表 + 项目规范确认 + 错误码映射表 |

---

## 输出检查

- [ ] 接口清单表已建立，每条接口关联 `REQ-*`
- [ ] 项目 HTTP 规范已确认（客户端 / Base URL / Token 注入方式）
- [ ] 所有 Request / Response 类型无 `any`，与接口文档字段一一对应
- [ ] 接口层无魔法字符串，路径通过常量引用
- [ ] 错误码 → 前端行为映射表已输出，且与项目已有处理模式一致
- [ ] 字段命名风格与项目规范一致（如有转换需在注释中标注）
- [ ] Mock 数据覆盖正常 / 边界 / 空列表三场景（若本 Phase 生成 Mock）
- [ ] 每个接口函数头部有 `REQ-*` 和 `API-*` 关联注释
- [ ] 接口层每个导出函数有对应单元测试：Mock 请求，验证参数构造正确、响应字段映射正确、错误时抛出符合预期的异常
