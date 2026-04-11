# Update Documentation
# 更新文档

Sync documentation with the codebase, generating from source-of-truth files.
将文档与代码库同步，从真实来源文件生成内容。

## Step 1: Identify Sources of Truth
## 步骤 1：识别真实来源

| Source | Generates |
|--------|-----------|
| 来源 | 生成内容 |
| `package.json` scripts | Available commands reference |
| `package.json` 脚本 | 可用命令参考 |
| `.env.example` | Environment variable documentation |
| `.env.example` | 环境变量文档 |
| `openapi.yaml` / route files | API endpoint reference |
| `openapi.yaml` / 路由文件 | API 端点参考 |
| Source code exports | Public API documentation |
| 源码导出 | 公共 API 文档 |
| `Dockerfile` / `docker-compose.yml` | Infrastructure setup docs |
| `Dockerfile` / `docker-compose.yml` | 基础设施配置文档 |

## Step 2: Generate Script Reference
## 步骤 2：生成脚本参考

1. Read `package.json` (or `Makefile`, `Cargo.toml`, `pyproject.toml`)
1. 读取 `package.json`（或 `Makefile`、`Cargo.toml`、`pyproject.toml`）
2. Extract all scripts/commands with their descriptions
2. 提取所有脚本/命令及其描述
3. Generate a reference table:
3. 生成参考表：

```markdown
| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server with hot reload |
| `npm run build` | Production build with type checking |
| `npm test` | Run test suite with coverage |
```

## Step 3: Generate Environment Documentation
## 步骤 3：生成环境变量文档

1. Read `.env.example` (or `.env.template`, `.env.sample`)
1. 读取 `.env.example`（或 `.env.template`、`.env.sample`）
2. Extract all variables with their purposes
2. 提取所有变量及其用途
3. Categorize as required vs optional
3. 分类为必需和可选
4. Document expected format and valid values
4. 记录预期格式和有效值

```markdown
| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `DATABASE_URL` | Yes | PostgreSQL connection string | `postgres://user:pass@host:5432/db` |
| `LOG_LEVEL` | No | Logging verbosity (default: info) | `debug`, `info`, `warn`, `error` |
```

## Step 4: Update Contributing Guide
## 步骤 4：更新贡献指南

Generate or update `docs/CONTRIBUTING.md` with:
生成或更新 `docs/CONTRIBUTING.md`，包含：
- Development environment setup (prerequisites, install steps)
- 开发环境配置（前置条件、安装步骤）
- Available scripts and their purposes
- 可用脚本及其用途
- Testing procedures (how to run, how to write new tests)
- 测试流程（如何运行、如何编写新测试）
- Code style enforcement (linter, formatter, pre-commit hooks)
- 代码风格规范（linter、格式化工具、pre-commit hooks）
- PR submission checklist
- PR 提交检查清单

## Step 5: Update Runbook
## 步骤 5：更新运行手册

Generate or update `docs/RUNBOOK.md` with:
生成或更新 `docs/RUNBOOK.md`，包含：
- Deployment procedures (step-by-step)
- 部署流程（逐步说明）
- Health check endpoints and monitoring
- 健康检查端点和监控
- Common issues and their fixes
- 常见问题及其修复方法
- Rollback procedures
- 回滚流程
- Alerting and escalation paths
- 告警和升级路径

## Step 6: Staleness Check
## 步骤 6：过期检查

1. Find documentation files not modified in 90+ days
1. 查找 90 天以上未修改的文档文件
2. Cross-reference with recent source code changes
2. 与近期源码变更交叉比对
3. Flag potentially outdated docs for manual review
3. 标记可能过期的文档以供人工审查

## Step 7: Show Summary
## 步骤 7：展示摘要

```
Documentation Update
──────────────────────────────
Updated:  docs/CONTRIBUTING.md (scripts table)
Updated:  docs/ENV.md (3 new variables)
Flagged:  docs/DEPLOY.md (142 days stale)
Skipped:  docs/API.md (no changes detected)
──────────────────────────────
```

## Rules
## 规则

- **Single source of truth**: Always generate from code, never manually edit generated sections
- **单一真实来源**：始终从代码生成，永远不要手动编辑生成的部分
- **Preserve manual sections**: Only update generated sections; leave hand-written prose intact
- **保留手写部分**：只更新生成的部分；保持手写内容不变
- **Mark generated content**: Use `<!-- AUTO-GENERATED -->` markers around generated sections
- **标记生成内容**：在生成的部分周围使用 `<!-- AUTO-GENERATED -->` 标记
- **Don't create docs unprompted**: Only create new doc files if the command explicitly requests it
- **不要主动创建文档**：只有在命令明确要求时才创建新文档文件
