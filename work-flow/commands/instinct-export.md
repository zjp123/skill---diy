---
name: instinct-export
description: Export instincts from project/global scope to a file
  将 instinct 从项目/全局范围导出到文件
command: /instinct-export
---

# Instinct Export Command
# Instinct 导出命令

Exports instincts to a shareable format. Perfect for:
将 instinct 导出为可共享的格式。适用于：
- Sharing with teammates
- 与团队成员共享
- Transferring to a new machine
- 迁移到新机器
- Contributing to project conventions
- 贡献到项目约定

## Usage
## 使用方法

```
/instinct-export                           # Export all personal instincts
/instinct-export --domain testing          # Export only testing instincts
/instinct-export --min-confidence 0.7      # Only export high-confidence instincts
/instinct-export --output team-instincts.yaml
/instinct-export --scope project --output project-instincts.yaml
```

## What to Do
## 操作步骤

1. Detect current project context
1. 检测当前项目上下文
2. Load instincts by selected scope:
2. 按所选范围加载 instinct：
   - `project`: current project only
   - `project`：仅当前项目
   - `global`: global only
   - `global`：仅全局
   - `all`: project + global merged (default)
   - `all`：项目 + 全局合并（默认）
3. Apply filters (`--domain`, `--min-confidence`)
3. 应用过滤器（`--domain`、`--min-confidence`）
4. Write YAML-style export to file (or stdout if no output path provided)
4. 将 YAML 格式的导出内容写入文件（若未提供输出路径则输出到 stdout）

## Output Format
## 输出格式

Creates a YAML file:
创建一个 YAML 文件：

```yaml
# Instincts Export
# Generated: 2025-01-22
# Source: personal
# Count: 12 instincts

---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.8
domain: code-style
source: session-observation
scope: project
project_id: a1b2c3d4e5f6
project_name: my-app
---

# Prefer Functional Style

## Action
Use functional patterns over classes.
```

## Flags
## 标志

- `--domain <name>`: Export only specified domain
- `--domain <name>`：仅导出指定领域
- `--min-confidence <n>`: Minimum confidence threshold
- `--min-confidence <n>`：最低置信度阈值
- `--output <file>`: Output file path (prints to stdout when omitted)
- `--output <file>`：输出文件路径（省略时输出到 stdout）
- `--scope <project|global|all>`: Export scope (default: `all`)
- `--scope <project|global|all>`：导出范围（默认：`all`）
