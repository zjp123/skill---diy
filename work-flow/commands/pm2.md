# PM2 Init
# PM2 初始化

Auto-analyze project and generate PM2 service commands.
自动分析项目并生成 PM2 服务命令。

**Command**: `$ARGUMENTS`
**命令**：`$ARGUMENTS`

---

## Workflow
## 工作流

1. Check PM2 (install via `npm install -g pm2` if missing)
1. 检查 PM2（若未安装则通过 `npm install -g pm2` 安装）
2. Scan project to identify services (frontend/backend/database)
2. 扫描项目以识别服务（前端/后端/数据库）
3. Generate config files and individual command files
3. 生成配置文件和各服务命令文件

---

## Service Detection
## 服务检测

| Type | Detection | Default Port |
|------|-----------|--------------|
| 类型 | 检测方式 | 默认端口 |
| Vite | vite.config.* | 5173 |
| Vite | vite.config.* | 5173 |
| Next.js | next.config.* | 3000 |
| Next.js | next.config.* | 3000 |
| Nuxt | nuxt.config.* | 3000 |
| Nuxt | nuxt.config.* | 3000 |
| CRA | react-scripts in package.json | 3000 |
| CRA | react-scripts 位于 package.json 中 | 3000 |
| Express/Node | server/backend/api directory + package.json | 3000 |
| Express/Node | server/backend/api 目录 + package.json | 3000 |
| FastAPI/Flask | requirements.txt / pyproject.toml | 8000 |
| FastAPI/Flask | requirements.txt / pyproject.toml | 8000 |
| Go | go.mod / main.go | 8080 |
| Go | go.mod / main.go | 8080 |

**Port Detection Priority**: User specified > .env > config file > scripts args > default port
**端口检测优先级**：用户指定 > .env > 配置文件 > 脚本参数 > 默认端口

---

## Generated Files
## 生成文件

```
project/
├── ecosystem.config.cjs              # PM2 config
├── {backend}/start.cjs               # Python wrapper (if applicable)
└── .claude/
    ├── commands/
    │   ├── pm2-all.md                # Start all + monit
    │   ├── pm2-all-stop.md           # Stop all
    │   ├── pm2-all-restart.md        # Restart all
    │   ├── pm2-{port}.md             # Start single + logs
    │   ├── pm2-{port}-stop.md        # Stop single
    │   ├── pm2-{port}-restart.md     # Restart single
    │   ├── pm2-logs.md               # View all logs
    │   └── pm2-status.md             # View status
    └── scripts/
        ├── pm2-logs-{port}.ps1       # Single service logs
        └── pm2-monit.ps1             # PM2 monitor
```

---

## Windows Configuration (IMPORTANT)
## Windows 配置（重要）

### ecosystem.config.cjs
### ecosystem.config.cjs

**Must use `.cjs` extension**
**必须使用 `.cjs` 扩展名**

```javascript
module.exports = {
  apps: [
    // Node.js (Vite/Next/Nuxt)
    {
      name: 'project-3000',
      cwd: './packages/web',
      script: 'node_modules/vite/bin/vite.js',
      args: '--port 3000',
      interpreter: 'C:/Program Files/nodejs/node.exe',
      env: { NODE_ENV: 'development' }
    },
    // Python
    {
      name: 'project-8000',
      cwd: './backend',
      script: 'start.cjs',
      interpreter: 'C:/Program Files/nodejs/node.exe',
      env: { PYTHONUNBUFFERED: '1' }
    }
  ]
}
```

**Framework script paths:**
**框架脚本路径：**

| Framework | script | args |
|-----------|--------|------|
| 框架 | script | args |
| Vite | `node_modules/vite/bin/vite.js` | `--port {port}` |
| Vite | `node_modules/vite/bin/vite.js` | `--port {port}` |
| Next.js | `node_modules/next/dist/bin/next` | `dev -p {port}` |
| Next.js | `node_modules/next/dist/bin/next` | `dev -p {port}` |
| Nuxt | `node_modules/nuxt/bin/nuxt.mjs` | `dev --port {port}` |
| Nuxt | `node_modules/nuxt/bin/nuxt.mjs` | `dev --port {port}` |
| Express | `src/index.js` or `server.js` | - |
| Express | `src/index.js` or `server.js` | - |

### Python Wrapper Script (start.cjs)
### Python 包装脚本（start.cjs）

```javascript
const { spawn } = require('child_process');
const proc = spawn('python', ['-m', 'uvicorn', 'app.main:app', '--host', '0.0.0.0', '--port', '8000', '--reload'], {
  cwd: __dirname, stdio: 'inherit', windowsHide: true
});
proc.on('close', (code) => process.exit(code));
```

---

## Command File Templates (Minimal Content)
## 命令文件模板（最小内容）

### pm2-all.md (Start all + monit)
### pm2-all.md（启动全部 + 监控）
````markdown
Start all services and open PM2 monitor.
```bash
cd "{PROJECT_ROOT}" && pm2 start ecosystem.config.cjs && start wt.exe -d "{PROJECT_ROOT}" pwsh -NoExit -c "pm2 monit"
```
````

### pm2-all-stop.md
### pm2-all-stop.md
````markdown
Stop all services.
```bash
cd "{PROJECT_ROOT}" && pm2 stop all
```
````

### pm2-all-restart.md
### pm2-all-restart.md
````markdown
Restart all services.
```bash
cd "{PROJECT_ROOT}" && pm2 restart all
```
````

### pm2-{port}.md (Start single + logs)
### pm2-{port}.md（启动单个服务 + 日志）
````markdown
Start {name} ({port}) and open logs.
```bash
cd "{PROJECT_ROOT}" && pm2 start ecosystem.config.cjs --only {name} && start wt.exe -d "{PROJECT_ROOT}" pwsh -NoExit -c "pm2 logs {name}"
```
````

### pm2-{port}-stop.md
### pm2-{port}-stop.md
````markdown
Stop {name} ({port}).
```bash
cd "{PROJECT_ROOT}" && pm2 stop {name}
```
````

### pm2-{port}-restart.md
### pm2-{port}-restart.md
````markdown
Restart {name} ({port}).
```bash
cd "{PROJECT_ROOT}" && pm2 restart {name}
```
````

### pm2-logs.md
### pm2-logs.md
````markdown
View all PM2 logs.
```bash
cd "{PROJECT_ROOT}" && pm2 logs
```
````

### pm2-status.md
### pm2-status.md
````markdown
View PM2 status.
```bash
cd "{PROJECT_ROOT}" && pm2 status
```
````

### PowerShell Scripts (pm2-logs-{port}.ps1)
### PowerShell 脚本（pm2-logs-{port}.ps1）
```powershell
Set-Location "{PROJECT_ROOT}"
pm2 logs {name}
```

### PowerShell Scripts (pm2-monit.ps1)
### PowerShell 脚本（pm2-monit.ps1）
```powershell
Set-Location "{PROJECT_ROOT}"
pm2 monit
```

---

## Key Rules
## 关键规则

1. **Config file**: `ecosystem.config.cjs` (not .js)
1. **配置文件**：`ecosystem.config.cjs`（非 .js）
2. **Node.js**: Specify bin path directly + interpreter
2. **Node.js**：直接指定 bin 路径 + 解释器
3. **Python**: Node.js wrapper script + `windowsHide: true`
3. **Python**：Node.js 包装脚本 + `windowsHide: true`
4. **Open new window**: `start wt.exe -d "{path}" pwsh -NoExit -c "command"`
4. **打开新窗口**：`start wt.exe -d "{path}" pwsh -NoExit -c "command"`
5. **Minimal content**: Each command file has only 1-2 lines description + bash block
5. **最小内容**：每个命令文件只有 1-2 行描述 + bash 块
6. **Direct execution**: No AI parsing needed, just run the bash command
6. **直接执行**：无需 AI 解析，直接运行 bash 命令

---

## Execute
## 执行

Based on `$ARGUMENTS`, execute init:
根据 `$ARGUMENTS`，执行初始化：

1. Scan project for services
1. 扫描项目以识别服务
2. Generate `ecosystem.config.cjs`
2. 生成 `ecosystem.config.cjs`
3. Generate `{backend}/start.cjs` for Python services (if applicable)
3. 为 Python 服务生成 `{backend}/start.cjs`（如适用）
4. Generate command files in `.claude/commands/`
4. 在 `.claude/commands/` 中生成命令文件
5. Generate script files in `.claude/scripts/`
5. 在 `.claude/scripts/` 中生成脚本文件
6. **Update project CLAUDE.md** with PM2 info (see below)
6. 用 PM2 信息**更新项目 CLAUDE.md**（见下文）
7. **Display completion summary** with terminal commands
7. 显示包含终端命令的**完成摘要**

---

## Post-Init: Update CLAUDE.md
## 初始化后：更新 CLAUDE.md

After generating files, append PM2 section to project's `CLAUDE.md` (create if not exists):
生成文件后，将 PM2 部分追加到项目的 `CLAUDE.md`（如不存在则创建）：

````markdown
## PM2 Services

| Port | Name | Type |
|------|------|------|
| {port} | {name} | {type} |

**Terminal Commands:**
```bash
pm2 start ecosystem.config.cjs   # First time
pm2 start all                    # After first time
pm2 stop all / pm2 restart all
pm2 start {name} / pm2 stop {name}
pm2 logs / pm2 status / pm2 monit
pm2 save                         # Save process list
pm2 resurrect                    # Restore saved list
```
````

**Rules for CLAUDE.md update:**
**CLAUDE.md 更新规则：**
- If PM2 section exists, replace it
- 若 PM2 部分已存在，则替换
- If not exists, append to end
- 若不存在，则追加到末尾
- Keep content minimal and essential
- 保持内容最简且必要

---

## Post-Init: Display Summary
## 初始化后：显示摘要

After all files generated, output:
所有文件生成后，输出：

```
## PM2 Init Complete

**Services:**

| Port | Name | Type |
|------|------|------|
| {port} | {name} | {type} |

**Claude Commands:** /pm2-all, /pm2-all-stop, /pm2-{port}, /pm2-{port}-stop, /pm2-logs, /pm2-status

**Terminal Commands:**
## First time (with config file)
pm2 start ecosystem.config.cjs && pm2 save

## After first time (simplified)
pm2 start all          # Start all
pm2 stop all           # Stop all
pm2 restart all        # Restart all
pm2 start {name}       # Start single
pm2 stop {name}        # Stop single
pm2 logs               # View logs
pm2 monit              # Monitor panel
pm2 resurrect          # Restore saved processes

**Tip:** Run `pm2 save` after first start to enable simplified commands.
```
