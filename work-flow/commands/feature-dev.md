---
description: Guided feature development with codebase understanding and architecture focus
  以代码库理解和架构为重点的引导式功能开发工作流
---

A structured feature-development workflow that emphasizes understanding existing code before writing new code.
一个结构化的功能开发工作流，强调在编写新代码之前先理解现有代码。

## Phases
## 阶段

### 1. Discovery
### 1. 探索需求

- read the feature request carefully
- 仔细阅读功能请求
- identify requirements, constraints, and acceptance criteria
- 识别需求、约束条件和验收标准
- ask clarifying questions if the request is ambiguous
- 如果请求模糊，提出澄清问题

### 2. Codebase Exploration
### 2. 代码库探索

- use `code-explorer` to analyze the relevant existing code
- 使用 `code-explorer` 分析相关的现有代码
- trace execution paths and architecture layers
- 追踪执行路径和架构层级
- understand integration points and conventions
- 理解集成点和代码约定

### 3. Clarifying Questions
### 3. 澄清问题

- present findings from exploration
- 展示探索阶段的发现
- ask targeted design and edge-case questions
- 提出有针对性的设计和边界情况问题
- wait for user response before proceeding
- 在继续之前等待用户回复

### 4. Architecture Design
### 4. 架构设计

- use `code-architect` to design the feature
- 使用 `code-architect` 设计功能
- provide the implementation blueprint
- 提供实现蓝图
- wait for approval before implementing
- 在实现之前等待批准

### 5. Implementation
### 5. 实现

- implement the feature following the approved design
- 按照批准的设计实现功能
- prefer TDD where appropriate
- 在适当情况下优先使用 TDD
- keep commits small and focused
- 保持提交小而专注

### 6. Quality Review
### 6. 质量审查

- use `code-reviewer` to review the implementation
- 使用 `code-reviewer` 审查实现
- address critical and important issues
- 解决关键和重要问题
- verify test coverage
- 验证测试覆盖率

### 7. Summary
### 7. 总结

- summarize what was built
- 总结构建的内容
- list follow-up items or limitations
- 列出后续事项或局限性
- provide testing instructions
- 提供测试说明
