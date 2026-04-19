---
name: "frontend-workflow"
description: >
  完整前端研发工作流：从 PRD 解析 → Figma 设计读取 → 组件选型 → 任务拆解 →
  代码骨架 → 后端接口集成 → TDD 开发 → 验证循环 → E2E 测试 → Code Review。
  触发词：UI解析、PRD拆解、Figma解析、任务清单、代码骨架、TDD、CodeReview、
  前端工作流、需求落地、接口文档、API集成、接口实现、后端联调、Swagger、OpenAPI、
  接口对接、服务层、api.ts。
---

# Frontend Workflow — PRD to Code Review

## 目的

将「PRD / Figma 设计」标准化落地为「通过 Code Review 的可合并代码」，
贯通前端研发的所有关键环节，保证可复用、可评审、可追溯。

---

## 触发条件

- 用户提到 UI 解析、页面拆解、组件选型
- 用户需要把 PRD 拆成前端可执行任务
- 用户要求输出开发任务清单（WBS / DoD）
- 用户要求生成代码骨架或文件拆分建议
- 用户提到 TDD、测试驱动、先写测试
- 用户要求进行代码评审或 PR 检查
- 用户提到 Figma 稿、UI 稿、设计稿
- 用户提供了接口文档（Swagger / OpenAPI / Postman / Markdown 描述）
- 用户提到接口联调、前后端对接、服务层实现

---

## 必要输入

- PRD 链接或文本（可选，至少有其一）
- Figma 链接或 UI 截图（至少其一）
- 目标代码仓路径
- 约束条件（优先复用哪些组件、特殊安全流程、是否分页等）
- 后端接口文档（可选；Swagger / OpenAPI / Postman / Markdown 均可；提供后 Phase 5b 自动激活）

> **确认协议**：工作流执行过程中，凡遇到需求歧义、选型分歧、安全决策等不确定情况，
> AI **必须暂停并向用户提问**，收到明确回复后才能继续。
> 详见 `shared/user-confirmation-protocol.md`。

---

## 标准交付物

| 文件 | 内容 |
|---|---|
| `prd-analyze.md` | 前端需求拆解（模块 / 优先级 / 接口契约） |
| `ui-analysis-with-components.md` | UI 区域拆解 + 组件选型 + REQ 映射 |
| `prd-dev-task-list.md` | WBS 任务清单（依赖 / 优先级 / DoD） |
| `ui-coding-skeleton-plan.md` | 按文件拆分的代码骨架 |
| `api-integration.md`（有接口文档时） | 接口清单表 + 项目规范确认 + 错误码映射 |
| `workflow-session-log.md` | 会话过程记录（用户修改 / 确认项 / 重新生成）|

---

## 工作流总览

```
Phase 0   PRD 解析              → phases/00-prd-analysis.md
   ↓
Phase 1   Figma / UI 读取       → phases/01-figma-reading.md
   ↓
Phase 2   代码库侦察             → phases/02-codebase-recon.md
   ↓
Phase 3   UI → 组件映射          → phases/03-component-mapping.md
   ↓
Phase 4   任务清单 & RTM         → phases/04-task-list.md
   ↓
Phase 5   代码骨架               → phases/05-coding-skeleton.md
   ↓
Phase 5b  后端接口集成 ★          → phases/05b-api-integration.md
          （有接口文档时必须执行，无则跳过）
   ↓
Phase 6   TDD 开发循环           → phases/06-tdd.md
   ↓
Phase 7   验证循环               → phases/07-verification.md
   ↓
Phase 8   E2E 测试               → phases/08-e2e-testing.md
   ↓
Phase 9   Code Review           → phases/09-code-review.md

共用规则 → shared/ci-quality-gates.md
确认协议 → shared/user-confirmation-protocol.md（贯穿全程，不确定时必须暂停）
会话日志 → shared/session-log.md（贯穿全程，实时追加）
```

> **Phase 1 / Phase 2 顺序说明**
> 默认先读 Figma（Phase 1）再侦察代码库（Phase 2）。
> 若为**存量项目迭代**（非绿地开发），建议先执行 Phase 2 侦察代码库，
> 带着已知组件清单再读 Figma，可直接在设计稿里做组件复用标注，减少回头成本。

---

## 工作流完成定义（Overall DoD）

- [ ] 功能：P0 功能全部可用，与 PRD 关键条目逐项对齐
- [ ] 视觉：与 Figma 设计 1:1 对齐，无视觉偏差
- [ ] 接口：`services/api.ts` 实现与接口文档字段一一对应，错误码映射完整，无魔法字符串（有接口文档时必查）
- [ ] 质量：单测 / E2E / 回归全绿，关键异常路径可复现可恢复
- [ ] 安全：权限与 2FA 流程闭环，无越权入口
- [ ] 可观测：埋点与监控生效，可追踪核心操作成功率和错误率
- [ ] 发布：完成灰度观察并具备一键回滚能力
- [ ] 追溯：所有 PR 包含 REQ 映射和测试引用，RTM 可审计到代码与测试

---

## 快速导航

| 我想做的事 | 阅读哪个文件 |
|---|---|
| 拆解 PRD、建立 REQ 表 | `phases/00-prd-analysis.md` |
| 读取 Figma 设计稿 | `phases/01-figma-reading.md` |
| 侦察现有代码库 | `phases/02-codebase-recon.md` |
| 做 UI 区域组件选型 | `phases/03-component-mapping.md` |
| 生成 WBS 任务清单 | `phases/04-task-list.md` |
| 生成代码骨架 | `phases/05-coding-skeleton.md` |
| 后端接口集成（有接口文档时） | `phases/05b-api-integration.md` |
| TDD 开发 | `phases/06-tdd.md` |
| PR 前质量验证 | `phases/07-verification.md` |
| 写 / 跑 E2E 测试 | `phases/08-e2e-testing.md` |
| Code Review / PR 模板 | `phases/09-code-review.md` |
| CI 门禁 / 质量规则 | `shared/ci-quality-gates.md` |
| 查看会话决策记录 | `shared/session-log.md` |
| 不确定时的提问规范 | `shared/user-confirmation-protocol.md` |
