# Update Codemaps
# 更新代码地图

Analyze the codebase structure and generate token-lean architecture documentation.
分析代码库结构并生成精简 token 的架构文档。

## Step 1: Scan Project Structure
## 步骤 1：扫描项目结构

1. Identify the project type (monorepo, single app, library, microservice)
1. 识别项目类型（monorepo、单体应用、库、微服务）
2. Find all source directories (src/, lib/, app/, packages/)
2. 查找所有源码目录（src/、lib/、app/、packages/）
3. Map entry points (main.ts, index.ts, app.py, main.go, etc.)
3. 梳理入口点（main.ts、index.ts、app.py、main.go 等）

## Step 2: Generate Codemaps
## 步骤 2：生成代码地图

Create or update codemaps in `docs/CODEMAPS/` (or `.reports/codemaps/`):
在 `docs/CODEMAPS/`（或 `.reports/codemaps/`）中创建或更新代码地图：

| File | Contents |
|------|----------|
| 文件 | 内容 |
| `architecture.md` | High-level system diagram, service boundaries, data flow |
| `architecture.md` | 高层系统图、服务边界、数据流 |
| `backend.md` | API routes, middleware chain, service → repository mapping |
| `backend.md` | API 路由、中间件链、服务 → 仓储映射 |
| `frontend.md` | Page tree, component hierarchy, state management flow |
| `frontend.md` | 页面树、组件层级、状态管理流程 |
| `data.md` | Database tables, relationships, migration history |
| `data.md` | 数据库表、关联关系、迁移历史 |
| `dependencies.md` | External services, third-party integrations, shared libraries |
| `dependencies.md` | 外部服务、第三方集成、共享库 |

### Codemap Format
### 代码地图格式

Each codemap should be token-lean — optimized for AI context consumption:
每份代码地图应精简 token——针对 AI 上下文消费进行优化：

```markdown
# Backend Architecture

## Routes
POST /api/users → UserController.create → UserService.create → UserRepo.insert
GET  /api/users/:id → UserController.get → UserService.findById → UserRepo.findById

## Key Files
src/services/user.ts (business logic, 120 lines)
src/repos/user.ts (database access, 80 lines)

## Dependencies
- PostgreSQL (primary data store)
- Redis (session cache, rate limiting)
- Stripe (payment processing)
```

## Step 3: Diff Detection
## 步骤 3：变更检测

1. If previous codemaps exist, calculate the diff percentage
1. 若存在先前的代码地图，计算差异百分比
2. If changes > 30%, show the diff and request user approval before overwriting
2. 若变更 > 30%，展示差异并在覆盖前请求用户确认
3. If changes <= 30%, update in place
3. 若变更 <= 30%，直接就地更新

## Step 4: Add Metadata
## 步骤 4：添加元数据

Add a freshness header to each codemap:
为每份代码地图添加新鲜度头部：

```markdown
<!-- Generated: 2026-02-11 | Files scanned: 142 | Token estimate: ~800 -->
```

## Step 5: Save Analysis Report
## 步骤 5：保存分析报告

Write a summary to `.reports/codemap-diff.txt`:
将摘要写入 `.reports/codemap-diff.txt`：
- Files added/removed/modified since last scan
- 自上次扫描以来添加/删除/修改的文件
- New dependencies detected
- 检测到的新依赖
- Architecture changes (new routes, new services, etc.)
- 架构变更（新路由、新服务等）
- Staleness warnings for docs not updated in 90+ days
- 90 天以上未更新的文档的过期警告

## Tips
## 提示

- Focus on **high-level structure**, not implementation details
- 聚焦于**高层结构**，而非实现细节
- Prefer **file paths and function signatures** over full code blocks
- 优先使用**文件路径和函数签名**，而非完整代码块
- Keep each codemap under **1000 tokens** for efficient context loading
- 每份代码地图保持在 **1000 tokens** 以内以高效加载上下文
- Use ASCII diagrams for data flow instead of verbose descriptions
- 使用 ASCII 图表表示数据流，而非冗长的文字描述
- Run after major feature additions or refactoring sessions
- 在重大功能添加或重构会话后运行
