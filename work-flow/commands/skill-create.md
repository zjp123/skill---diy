---
name: skill-create
description: Analyze local git history to extract coding patterns and generate SKILL.md files. Local version of the Skill Creator GitHub App.
  分析本地 git 历史以提取编码模式并生成 SKILL.md 文件。Skill Creator GitHub App 的本地版本。
allowed_tools: ["Bash", "Read", "Write", "Grep", "Glob"]
---

# /skill-create - Local Skill Generation
# /skill-create - 本地技能生成

Analyze your repository's git history to extract coding patterns and generate SKILL.md files that teach Claude your team's practices.
分析仓库的 git 历史以提取编码模式，并生成向 Claude 传授团队实践的 SKILL.md 文件。

## Usage
## 使用方法

```bash
/skill-create                    # Analyze current repo
/skill-create --commits 100      # Analyze last 100 commits
/skill-create --output ./skills  # Custom output directory
/skill-create --instincts        # Also generate instincts for continuous-learning-v2
```

## What It Does
## 功能说明

1. **Parses Git History** - Analyzes commits, file changes, and patterns
1. **解析 Git 历史** - 分析提交、文件变更和模式
2. **Detects Patterns** - Identifies recurring workflows and conventions
2. **检测模式** - 识别重复出现的工作流和规范
3. **Generates SKILL.md** - Creates valid Claude Code skill files
3. **生成 SKILL.md** - 创建有效的 Claude Code 技能文件
4. **Optionally Creates Instincts** - For the continuous-learning-v2 system
4. **可选创建本能** - 用于 continuous-learning-v2 系统

## Analysis Steps
## 分析步骤

### Step 1: Gather Git Data
### 步骤 1：收集 Git 数据

```bash
# Get recent commits with file changes
git log --oneline -n ${COMMITS:-200} --name-only --pretty=format:"%H|%s|%ad" --date=short

# Get commit frequency by file
git log --oneline -n 200 --name-only | grep -v "^$" | grep -v "^[a-f0-9]" | sort | uniq -c | sort -rn | head -20

# Get commit message patterns
git log --oneline -n 200 | cut -d' ' -f2- | head -50
```

### Step 2: Detect Patterns
### 步骤 2：检测模式

Look for these pattern types:
查找以下模式类型：

| Pattern | Detection Method |
|---------|-----------------|
| 模式 | 检测方法 |
| **Commit conventions** | Regex on commit messages (feat:, fix:, chore:) |
| **提交约定** | 对提交消息进行正则匹配（feat:、fix:、chore:）|
| **File co-changes** | Files that always change together |
| **文件协同变更** | 总是一起变更的文件 |
| **Workflow sequences** | Repeated file change patterns |
| **工作流序列** | 重复出现的文件变更模式 |
| **Architecture** | Folder structure and naming conventions |
| **架构** | 文件夹结构和命名规范 |
| **Testing patterns** | Test file locations, naming, coverage |
| **测试模式** | 测试文件位置、命名、覆盖率 |

### Step 3: Generate SKILL.md
### 步骤 3：生成 SKILL.md

Output format:
输出格式：

```markdown
---
name: {repo-name}-patterns
description: Coding patterns extracted from {repo-name}
version: 1.0.0
source: local-git-analysis
analyzed_commits: {count}
---

# {Repo Name} Patterns

## Commit Conventions
{detected commit message patterns}

## Code Architecture
{detected folder structure and organization}

## Workflows
{detected repeating file change patterns}

## Testing Patterns
{detected test conventions}
```

### Step 4: Generate Instincts (if --instincts)
### 步骤 4：生成本能（若使用 --instincts）

For continuous-learning-v2 integration:
用于 continuous-learning-v2 集成：

```yaml
---
id: {repo}-commit-convention
trigger: "when writing a commit message"
confidence: 0.8
domain: git
source: local-repo-analysis
---

# Use Conventional Commits

## Action
Prefix commits with: feat:, fix:, chore:, docs:, test:, refactor:

## Evidence
- Analyzed {n} commits
- {percentage}% follow conventional commit format
```

## Example Output
## 示例输出

Running `/skill-create` on a TypeScript project might produce:
在 TypeScript 项目上运行 `/skill-create` 可能会生成：

```markdown
---
name: my-app-patterns
description: Coding patterns from my-app repository
version: 1.0.0
source: local-git-analysis
analyzed_commits: 150
---

# My App Patterns

## Commit Conventions

This project uses **conventional commits**:
- `feat:` - New features
- `fix:` - Bug fixes
- `chore:` - Maintenance tasks
- `docs:` - Documentation updates

## Code Architecture

```
src/
├── components/     # React components (PascalCase.tsx)
├── hooks/          # Custom hooks (use*.ts)
├── utils/          # Utility functions
├── types/          # TypeScript type definitions
└── services/       # API and external services
```

## Workflows

### Adding a New Component
1. Create `src/components/ComponentName.tsx`
2. Add tests in `src/components/__tests__/ComponentName.test.tsx`
3. Export from `src/components/index.ts`

### Database Migration
1. Modify `src/db/schema.ts`
2. Run `pnpm db:generate`
3. Run `pnpm db:migrate`

## Testing Patterns

- Test files: `__tests__/` directories or `.test.ts` suffix
- Coverage target: 80%+
- Framework: Vitest
```

## GitHub App Integration
## GitHub App 集成

For advanced features (10k+ commits, team sharing, auto-PRs), use the [Skill Creator GitHub App](https://github.com/apps/skill-creator):
对于高级功能（10k+ 提交、团队共享、自动 PR），请使用 [Skill Creator GitHub App](https://github.com/apps/skill-creator)：

- Install: [github.com/apps/skill-creator](https://github.com/apps/skill-creator)
- 安装：[github.com/apps/skill-creator](https://github.com/apps/skill-creator)
- Comment `/skill-creator analyze` on any issue
- 在任意 issue 中评论 `/skill-creator analyze`
- Receives PR with generated skills
- 接收包含生成技能的 PR

## Related Commands
## 相关命令

- `/instinct-import` - Import generated instincts
- `/instinct-import` - 导入生成的本能
- `/instinct-status` - View learned instincts
- `/instinct-status` - 查看已学习的本能
- `/evolve` - Cluster instincts into skills/agents
- `/evolve` - 将本能聚类为技能/代理

---

*Part of [Everything Claude Code](https://github.com/affaan-m/everything-claude-code)*
*[Everything Claude Code](https://github.com/affaan-m/everything-claude-code) 的一部分*
