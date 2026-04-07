# ICAR-4389 汇率配置页 UI 解析文档（含组件选型）

## 1. 文档目的

- 基于提供的 UI 图，拆解页面结构、交互状态、字段规则。
- 结合当前 OWS 项目技术栈与已有组件，给出可直接编码的组件映射。
- 原则：优先复用现有组件；无可复用时标注「自定义开发」。

---

## 2. 当前项目技术栈结论（与本页实现相关）

- React 18 + TypeScript + Vite
- Ant Design 5（Tabs / Form / InputNumber / Table / Tooltip / Button）
- `@hytechc/admin-ui`（BusinessTable、权限组件等）
- CSS Modules + Less

代码依据：
- [package.json](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/package.json#L8-L24)
- [package.json](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/package.json#L60-L120)

---

## 3. UI 区域拆解与组件映射

## 3.1 区域 A：品牌 Tab 区（VG/VT/PU/MM/UM/VTJ/ST）

### UI 结构
- 顶部一行品牌切换，左侧 Tab，右侧可扩展品牌选择。
- 选中态：蓝色文字 + 下划线（ink bar）。

### 组件选型
- 优先复用：`AssetTabs`（现有）
  - 页面调用位置：[`<AssetTabs />`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/dapManagement/index.tsx#L310-L314)
  - 组件实现：[`AssetTabs.tsx`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/dapManagement/components/AssetTabs.tsx#L18-L65)
  - 内部技术：Antd `Tabs` + `Select` + `ConfigProvider` 主题覆盖

### 落地建议
- 本需求品牌 Tab 样式与现有 `AssetTabs` 高度匹配，直接复用。
- 若仅需 Tab（不需要右侧品牌 Select），新增 `showBrandSelect?: boolean` 属性控制显示即可（小改造复用）。

---

## 3.2 区域 B：汇率配置卡片（Exchange Rate Setting）

### UI 结构
- 标题：`{Brand} Exchange Rate Setting`
- 三行配置项：
  - Cross Currency Rate
  - Pip_% Rate
  - Per Million Rate
- 每行：标签 + 输入框 + `%` + 问号提示图标
- 底部按钮：`Cancel`、`Save`、`Edit`（根据编辑态切换）

### 组件选型
- 容器：自定义开发（页面局部卡片容器）
  - 原因：此卡片是新业务结构，不是通用列表页。
  - 样式可复用 DAP 页面卡片风格：[`style.module.less`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/dapManagement/style.module.less#L56-L60)
- 表单：Antd `Form` + `Form.Item`
- 输入：Antd `InputNumber`（suffix `%`）
  - 项目内有 `InputNumber` suffix 使用先例：[`pointsExchangeSettingCard.tsx`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/vib/component/systemSettings/pointsExchangeSettingCard.tsx#L229-L240)
- 提示图标：Antd `Tooltip` + `QuestionCircleOutlined`（或 `InfoCircleOutlined`）
- 按钮：Antd `Button`
- 权限控制（编辑按钮）：`Auth`（`@hytechc/admin-ui`）

### 字段与规则
- 三个字段统一规则：
  - 可空
  - 非空范围：`0.01 ~ 10.00`
  - 最多 2 位小数
  - 不可负数
- 建议统一封装 `PercentInputItem`（组件级复用），避免三段重复代码。

### 状态机（UI 必须覆盖）
- `view`：仅展示值，显示 Edit，隐藏 Save/Cancel
- `edit`：可编辑，显示 Save/Cancel，隐藏 Edit
- `saving`：Save loading + 防重复提交
- `error`：表单项错误提示 + message

### 2FA 说明
- 保存前谷歌二次验证弹窗
- 保存前先判断当前登录用户是否已绑定谷歌 2FA（`isBindTwoFa`）
- 未绑定：执行绑定流程，渲染 `BindTwoFaDialog`，交互参考 `apps/ows/src/pages/login/index.tsx`
- 已绑定：直接渲染 `TwoFAFrom`，交互参考 `packages/business/src/login/components/twoFAFrom.tsx`
- 组件选型：
  - `BindTwoFaDialog` 来源 `@hytechc/business`
  - `TwoFAFrom` 复用登录域现有实现，若包层暂未导出则优先补导出，不重新造轮子

---

## 3.3 区域 C：历史记录表（Exchange Rate Record）

### UI 结构
- 标题：`{Brand} Exchange Rate Record`
- 表头：From / To / Operator / Time
- From、To 单元格是多行文本（3条汇率项）

### 组件选型
- 优先方案：`BusinessTable`（`@hytechc/admin-ui`）
  - 复用依据：项目中大量报表页统一使用 `BusinessTable`，风格和行为一致
  - 参考：[`feedback/App.tsx`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/partnerManagement/feedback/App.tsx#L92-L112)
- 备选方案：Antd `Table`
  - 当记录区不需要搜索、分页配置能力时可用
- 单元格渲染：自定义 renderer（From/To 多行拼接）

### 数据显示规则
- 排序：最新在前
- From 首条默认 `-`
- Time 格式：`YYYY-MM-DD HH:mm:ss`
- Operator 展示用户名称

---

## 3.4 区域 D：页面布局与视觉规范

### UI 结构
- Page
  - Brand Tabs 区
  - Exchange Rate Setting 卡片
  - Exchange Rate Record 卡片

### 组件选型
- 布局容器：`div + CSS Modules`（与 DAP 管理页保持一致）
  - 参考：[`index.tsx`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/dapManagement/index.tsx#L307-L383)
  - 样式参考：[`style.module.less`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/dapManagement/style.module.less#L99-L151)

### 视觉建议（编码口径）
- 外层 section：白底、8px 圆角、轻阴影、模块间距 16px
- 标题与内容垂直间距：16px / 12px
- 配置项每行高度保持一致，按钮组顶部留白统一

---

## 4. 组件清单（可直接给开发）

| UI模块 | 组件 | 来源 | 结论 |
|---|---|---|---|
| 品牌切换 | `AssetTabs`（去除右侧 Select） | [`index.tsx#L310-L314`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/dapManagement/index.tsx#L310-L314) / [`AssetTabs.tsx`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/dapManagement/components/AssetTabs.tsx#L18-L65) | 复用（小改造） |
| 配置表单 | `Form` / `Form.Item` | Antd | 复用 |
| 百分比输入 | `InputNumber` + suffix `%` | Antd，先例见 [`pointsExchangeSettingCard.tsx#L229-L240`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/vib/component/systemSettings/pointsExchangeSettingCard.tsx#L229-L240) | 复用 |
| 提示问号 | `Tooltip` + Icon | Antd | 复用 |
| 按钮区 | `Button` | Antd | 复用 |
| 权限控制 | `Auth` | `@hytechc/admin-ui` | 复用 |
| 历史记录表 | `BusinessTable` | [`feedback/App.tsx#L92-L112`](file:///Users/bjsttlp406/pro_pro/admin-all/apps/ows/src/pages/partnerManagement/feedback/App.tsx#L92-L112) | 优先复用 |
| 保存前二次验证 | `BindTwoFaDialog` / `TwoFAFrom` | `@hytechc/business` / 登录域现有实现 | 按绑定态分支复用 |
| 页面业务卡片 | `ExchangeRateSettingCard` / `ExchangeRateRecordCard` | 新业务结构 | 自定义开发 |

---

## 5. 交互拆解（编码前统一）

## 5.1 Tab 切换
- 切换品牌后触发：
  - 拉取当前品牌配置值
  - 拉取当前品牌历史记录
- 若当前为编辑态且有改动：弹确认离开（避免误切换丢数据）

## 5.2 编辑保存流
- 点击 Edit → 进入编辑态
- 点击 Save：
  - 前端校验三项输入
  - 先判断当前用户是否已绑定 2FA
  - 未绑定：打开 `BindTwoFaDialog` 完成首次绑定
  - 已绑定：直接渲染 `TwoFAFrom` 输入验证码
  - 验证通过后提交保存
  - 成功提示并回 view 态
  - 刷新历史记录首行
- 点击 Cancel：
  - 恢复后端最新值
  - 回 view 态

## 5.3 异常态
- 接口失败：`message.error`
- 无记录：表格 empty
- 无权限：隐藏 Edit，页面只读

---

## 6. 建议的前端文件拆分

- `src/pages/partnerManagement/commissionExchangeRate/index.tsx`（页面容器）
- `components/BrandTabs.tsx`（复用 `AssetTabs` 并关闭右侧品牌下拉）
- `components/ExchangeRateSettingCard.tsx`（自定义）
- `components/ExchangeRateRecordTable.tsx`（BusinessTable 封装）
- `components/SaveTwoFaVerify.tsx`（封装 `BindTwoFaDialog` + `TwoFAFrom` 的分支渲染）
- `hooks/useCommissionExchangeRate.ts`（请求与状态）
- `style.module.less`（沿用 OWS + DAP 风格变量）

---

## 7. 开发前确认项（避免返工）

- Tab 是否只要品牌切换，还是要右侧品牌下拉（`AssetTabs` 默认含 `Select`）
答：不要右侧品牌下拉
- Save/Cancel 是否仅编辑态显示（UI 图中是该逻辑）
答：是
- 历史记录是否需要分页（UI 图看起来无分页）
答：暂不
- 2FA 弹窗是否已有平台级统一组件可复用
答：是，但需按用户绑定态分支复用；未绑定走 `BindTwoFaDialog`，已绑定走 `TwoFAFrom`
- 记录表是否必须统一接入 `BusinessTable`（建议是）
答：是

---

## 8. UI 区域与 REQ 映射（防止实现偏航）

| UI 区域 | 关联需求ID | 实现约束 |
|---|---|---|
| 区域A 品牌Tab | REQ-003, REQ-004 | 仅品牌Tab，不含右侧下拉 |
| 区域B 配置卡片 | REQ-003, REQ-004, REQ-005 | 输入校验与状态机必须完整 |
| 区域C 历史记录表 | REQ-006 | 最新在前，首条 From 为 `-` |
| 区域D 页面布局 | REQ-010 | 标题、按钮、提示均支持 i18n |

执行规则：
- UI 改动若影响任一区域，必须同步更新对应 `REQ-*` 的验收项。
- 出现“视觉符合但 REQ 未覆盖”的情况，按未完成处理。

---

## 9. UI 验收 Harness 清单（评审可直接打勾）

- [ ] Tab 切换：品牌数据联动正确，且无右侧下拉
- [ ] 输入校验：0.01~10.00、2位小数、可空、错误提示正确
- [ ] 状态机：view/edit/saving/error 均可复现
- [ ] 2FA：未绑定进入 `BindTwoFaDialog`，已绑定直接进入 `TwoFAFrom`
- [ ] 历史记录：From/To/Operator/Time 字段语义正确
- [ ] 权限态：无编辑权限时仅可查看
