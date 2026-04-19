# Phase 1：Figma / UI 设计读取

## 目标
读取 Figma 设计稿，提取视觉系统与交互规范，作为组件映射和代码实现的唯一视觉参考。

---

> **确认协议**：本 Phase 遇到以下情况时必须暂停：
> PRD 描述与 Figma 设计不一致 / 设计 Token 无法映射到项目已有 Token。
> 提问格式见 `shared/user-confirmation-protocol.md`。

## Figma MCP 使用规则（必须遵守）

1. **先调用 Figma MCP 工具**读取设计稿，不凭截图猜测
2. 将 MCP 输出（通常是 React + Tailwind 表示）视为**设计语义**，不作为最终代码
3. 用项目已有的设计 Token 替换 Tailwind 工具类
4. 复用项目已有组件，禁止重复造轮子
5. 优先使用项目颜色系统、字体比例和间距 Token
6. 最终 UI 必须与 Figma 截图 **1:1 视觉对齐**

---

## 读取内容

### 视觉系统

- **颜色 Token**：从 Figma Variables / Styles 提取，映射到项目已有 Token
- **字体比例**：font-size / line-height / font-weight 阶梯
- **间距节奏**：spacing / border-radius / shadow 规范
- **主题方向**：minimal / editorial / industrial 等，选一个并贯穿执行，不混用

### 页面结构

- 整体布局（栅格 / 分区 / 嵌套关系）
- 每个 UI 区域的尺寸、颜色、圆角、阴影
- 响应式断点（如有）

### 交互规范

- **状态机**：view / edit / saving / loading / error / empty / disabled（每个状态必须有对应 UI）
- **动效**：仅在传达层级或确认用户操作时使用；一个页面不超过 2 个核心动效；支持 `prefers-reduced-motion`
- **无障碍**：WCAG AA 对比度要求、键盘导航路径、焦点管理策略

---

## 设计实现原则

### 组合优于重写
保留设计系统的既有方向，不随意引入新风格。

### 不可接受的反模式
- 用内联 style 替代 Token（除非无对应 Token）
- 随意加动效（仅因为「加了好看」）
- 对比度不足的颜色搭配
- 移动端未做适配

---

## 输出检查

- [ ] 已从 Figma 提取颜色 / 字体 / 间距 Token 并映射到项目 Token
- [ ] 每个 UI 区域有明确的尺寸和视觉规范
- [ ] 交互状态机已梳理完整（覆盖所有状态）
- [ ] 动效方案不超过 2 个核心动效
- [ ] 与项目现有设计系统无冲突（或冲突点已记录待确认）
- [ ] 无障碍要求已梳理：对比度、键盘导航路径、焦点管理策略（将在 Phase 7.6 验证）
