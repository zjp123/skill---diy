# Build and Fix
# 构建与修复

Incrementally fix build and type errors with minimal, safe changes.
以最小、安全的更改逐步修复构建和类型错误。

## Step 1: Detect Build System
## 步骤 1：检测构建系统

Identify the project's build tool and run the build:
识别项目的构建工具并运行构建：

| Indicator | Build Command |
|-----------|---------------|
| 指示符 | 构建命令 |
| `package.json` with `build` script | `npm run build` or `pnpm build` |
| 含 `build` 脚本的 `package.json` | |
| `tsconfig.json` (TypeScript only) | `npx tsc --noEmit` |
| `tsconfig.json`（仅 TypeScript） | |
| `Cargo.toml` | `cargo build 2>&1` |
| `pom.xml` | `mvn compile` |
| `build.gradle` | `./gradlew compileJava` |
| `go.mod` | `go build ./...` |
| `pyproject.toml` | `python -m compileall -q .` or `mypy .` |

## Step 2: Parse and Group Errors
## 步骤 2：解析并分组错误

1. Run the build command and capture stderr
1. 运行构建命令并捕获 stderr
2. Group errors by file path
2. 按文件路径分组错误
3. Sort by dependency order (fix imports/types before logic errors)
3. 按依赖顺序排序（先修复导入/类型，再修复逻辑错误）
4. Count total errors for progress tracking
4. 统计总错误数以跟踪进度

## Step 3: Fix Loop (One Error at a Time)
## 步骤 3：修复循环（每次一个错误）

For each error:
对于每个错误：

1. **Read the file** — Use Read tool to see error context (10 lines around the error)
1. **读取文件** — 使用 Read 工具查看错误上下文（错误周围 10 行）
2. **Diagnose** — Identify root cause (missing import, wrong type, syntax error)
2. **诊断** — 识别根本原因（缺失导入、错误类型、语法错误）
3. **Fix minimally** — Use Edit tool for the smallest change that resolves the error
3. **最小化修复** — 使用 Edit 工具进行解决错误的最小更改
4. **Re-run build** — Verify the error is gone and no new errors introduced
4. **重新运行构建** — 验证错误已消失且未引入新错误
5. **Move to next** — Continue with remaining errors
5. **继续下一个** — 继续处理剩余错误

## Step 4: Guardrails
## 步骤 4：防护措施

Stop and ask the user if:
遇到以下情况时停止并询问用户：
- A fix introduces **more errors than it resolves**
- 修复引入的错误**多于其解决的错误**
- The **same error persists after 3 attempts** (likely a deeper issue)
- **同一错误在 3 次尝试后仍然存在**（可能是更深层的问题）
- The fix requires **architectural changes** (not just a build fix)
- 修复需要**架构变更**（而不仅仅是构建修复）
- Build errors stem from **missing dependencies** (need `npm install`, `cargo add`, etc.)
- 构建错误源于**缺失依赖**（需要 `npm install`、`cargo add` 等）

## Step 5: Summary
## 步骤 5：摘要

Show results:
显示结果：
- Errors fixed (with file paths)
- 已修复的错误（附文件路径）
- Errors remaining (if any)
- 剩余错误（如有）
- New errors introduced (should be zero)
- 引入的新错误（应为零）
- Suggested next steps for unresolved issues
- 未解决问题的建议后续步骤

## Recovery Strategies
## 恢复策略

| Situation | Action |
|-----------|--------|
| 情况 | 操作 |
| Missing module/import | Check if package is installed; suggest install command |
| 缺失模块/导入 | 检查包是否已安装；建议安装命令 |
| Type mismatch | Read both type definitions; fix the narrower type |
| 类型不匹配 | 读取两个类型定义；修复较窄的类型 |
| Circular dependency | Identify cycle with import graph; suggest extraction |
| 循环依赖 | 通过导入图识别循环；建议提取 |
| Version conflict | Check `package.json` / `Cargo.toml` for version constraints |
| 版本冲突 | 检查 `package.json` / `Cargo.toml` 中的版本约束 |
| Build tool misconfiguration | Read config file; compare with working defaults |
| 构建工具配置错误 | 读取配置文件；与可用默认值对比 |

Fix one error at a time for safety. Prefer minimal diffs over refactoring.
为安全起见，每次修复一个错误。优先使用最小差异，而非重构。
