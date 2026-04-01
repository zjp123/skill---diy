---
name: "ui-requirement-delivery"
description: "Builds a standardized UI delivery workflow from PRD/UI to component mapping and coding skeleton. Invoke when user asks UI解析、任务拆解、组件选型或可编码骨架."
---

# UI Requirement Delivery Workflow

## Purpose

将需求从「PRD / UI 图」标准化落地到「可编码输出物」，保证可复用、可评审、可执行。

## When To Invoke

- 用户提到 UI 解析、页面拆解、组件选型
- 用户需要把 PRD 拆成前端可执行功能点
- 用户要求输出开发任务清单（WBS / DoD）
- 用户要求生成代码骨架或文件拆分建议

## Required Inputs

- PRD 链接或文本（可选）
- UI 图或原型链接（至少其一）
- 目标代码仓路径
- 约束条件（如：优先复用现有组件、特定 2FA 组件、是否分页）

## Standard Outputs

- `prd-analyze.md`：前端需求拆解
- `prd-dev-task-list.md`：开发任务清单（含依赖、优先级、DoD）
- `ui-analysis-with-components.md`：UI 区域拆解 + 组件映射
- `ui-coding-skeleton-plan.md`：按文件拆分的代码骨架

## Workflow Steps

### Step 1: Codebase Recon

- 检查技术栈：框架、UI 库、状态管理、表格组件、权限组件
- 检查可复用组件：Tabs、Form、Table、2FA、Dialog
- 输出证据链接（文件 + 行号）

### Step 2: PRD Action Breakdown

- 按模块拆解功能点（P0/P1）
- 每个功能点补充：前端范围、接口依赖、异常态、验收口径
- 形成可执行步骤，不写空泛叙述

### Step 3: UI-to-Component Mapping

- 按 UI 区域拆解：结构、状态机、交互流、校验规则
- 每个区域标注组件选型：
  - 优先复用现有组件
  - 无可复用时标注自定义开发
- 输出组件来源证据（代码链接）

### Step 4: Dev Task List

- 按里程碑拆任务（M1/M2）
- 每任务包含：输入、输出、依赖、DoD
- 补充联调与回归任务，避免遗漏

### Step 5: Coding Skeleton

- 按目录与文件生成骨架（types/constants/services/hooks/components/page）
- 给出关键 Props / 类型 / 方法签名
- 对齐已确认约束（如：无右侧品牌下拉、固定 2FA 组件）

### Step 6: Consistency Pass

- 校验文档之间是否一致（组件名、组件来源、交互规则）
- 修正冲突项（例如：2FA 组件在不同文档不一致）
- 校验链接有效性与可追溯性

## Quality Gates

- 组件复用优先级明确，且每个关键区域有选型结论
- 至少 1 条代码证据支撑每个关键复用点
- 状态机完整覆盖 view / edit / saving / error
- 包含权限态、空态、错误态
- 文档可直接交给开发执行，不依赖口头补充

## Constraints

- 不凭空假设第三方库已存在，必须先在仓库验证
- 不泄露敏感信息，不输出密钥/令牌
- 不创建无关文档，优先在已有交付链路内补充

## Response Template

完成后按以下结构返回：

1. 已完善点（列出补齐的关键缺口）
2. 新生成或更新的文档路径
3. 关键组件映射结论
4. 可直接进入编码的下一步动作

