# Refactor Clean
# 重构清理

Safely identify and remove dead code with test verification at every step.
通过每步测试验证，安全地识别并删除死代码。

## Step 1: Detect Dead Code
## 步骤 1：检测死代码

Run analysis tools based on project type:
根据项目类型运行分析工具：

| Tool | What It Finds | Command |
|------|--------------|---------|
| 工具 | 检测内容 | 命令 |
| knip | Unused exports, files, dependencies | `npx knip` |
| knip | 未使用的导出、文件、依赖 | `npx knip` |
| depcheck | Unused npm dependencies | `npx depcheck` |
| depcheck | 未使用的 npm 依赖 | `npx depcheck` |
| ts-prune | Unused TypeScript exports | `npx ts-prune` |
| ts-prune | 未使用的 TypeScript 导出 | `npx ts-prune` |
| vulture | Unused Python code | `vulture src/` |
| vulture | 未使用的 Python 代码 | `vulture src/` |
| deadcode | Unused Go code | `deadcode ./...` |
| deadcode | 未使用的 Go 代码 | `deadcode ./...` |
| cargo-udeps | Unused Rust dependencies | `cargo +nightly udeps` |
| cargo-udeps | 未使用的 Rust 依赖 | `cargo +nightly udeps` |

If no tool is available, use Grep to find exports with zero imports:
若无可用工具，使用 Grep 查找零导入的导出：
```
# Find exports, then check if they're imported anywhere
```

## Step 2: Categorize Findings
## 步骤 2：对发现结果分类

Sort findings into safety tiers:
将发现结果按安全层级排序：

| Tier | Examples | Action |
|------|----------|--------|
| 层级 | 示例 | 操作 |
| **SAFE** | Unused utilities, test helpers, internal functions | Delete with confidence |
| **安全** | 未使用的工具函数、测试辅助函数、内部函数 | 放心删除 |
| **CAUTION** | Components, API routes, middleware | Verify no dynamic imports or external consumers |
| **谨慎** | 组件、API 路由、中间件 | 验证无动态导入或外部消费者 |
| **DANGER** | Config files, entry points, type definitions | Investigate before touching |
| **危险** | 配置文件、入口点、类型定义 | 操作前先调查 |

## Step 3: Safe Deletion Loop
## 步骤 3：安全删除循环

For each SAFE item:
对每个安全项：

1. **Run full test suite** — Establish baseline (all green)
1. **运行完整测试套件** — 建立基线（全部通过）
2. **Delete the dead code** — Use Edit tool for surgical removal
2. **删除死代码** — 使用 Edit 工具精确删除
3. **Re-run test suite** — Verify nothing broke
3. **重新运行测试套件** — 验证没有破坏任何内容
4. **If tests fail** — Immediately revert with `git checkout -- <file>` and skip this item
4. **若测试失败** — 立即使用 `git checkout -- <file>` 回滚并跳过此项
5. **If tests pass** — Move to next item
5. **若测试通过** — 移至下一项

## Step 4: Handle CAUTION Items
## 步骤 4：处理谨慎项

Before deleting CAUTION items:
删除谨慎项之前：
- Search for dynamic imports: `import()`, `require()`, `__import__`
- 搜索动态导入：`import()`、`require()`、`__import__`
- Search for string references: route names, component names in configs
- 搜索字符串引用：配置中的路由名称、组件名称
- Check if exported from a public package API
- 检查是否从公共包 API 导出
- Verify no external consumers (check dependents if published)
- 验证无外部消费者（若已发布则检查依赖方）

## Step 5: Consolidate Duplicates
## 步骤 5：合并重复项

After removing dead code, look for:
删除死代码后，查找：
- Near-duplicate functions (>80% similar) — merge into one
- 近似重复的函数（相似度 >80%）— 合并为一个
- Redundant type definitions — consolidate
- 冗余类型定义 — 合并整理
- Wrapper functions that add no value — inline them
- 无价值的包装函数 — 内联处理
- Re-exports that serve no purpose — remove indirection
- 无意义的重新导出 — 消除间接层

## Step 6: Summary
## 步骤 6：摘要

Report results:
报告结果：

```
Dead Code Cleanup
──────────────────────────────
Deleted:   12 unused functions
           3 unused files
           5 unused dependencies
Skipped:   2 items (tests failed)
Saved:     ~450 lines removed
──────────────────────────────
All tests passing PASS:
```

## Rules
## 规则

- **Never delete without running tests first**
- **永远不要在先运行测试之前删除**
- **One deletion at a time** — Atomic changes make rollback easy
- **一次删除一项** — 原子性变更便于回滚
- **Skip if uncertain** — Better to keep dead code than break production
- **不确定时跳过** — 保留死代码胜过破坏生产环境
- **Don't refactor while cleaning** — Separate concerns (clean first, refactor later)
- **清理时不要重构** — 分离关注点（先清理，后重构）
