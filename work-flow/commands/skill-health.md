---
name: skill-health
description: Show skill portfolio health dashboard with charts and analytics
  显示技能组合健康仪表板，包含图表和分析
command: true
---

# Skill Health Dashboard
# 技能健康仪表板

Shows a comprehensive health dashboard for all skills in the portfolio with success rate sparklines, failure pattern clustering, pending amendments, and version history.
显示技能组合中所有技能的综合健康仪表板，包含成功率迷你图、失败模式聚类、待处理修改和版本历史。

## Implementation
## 实现

Run the skill health CLI in dashboard mode:
以仪表板模式运行技能健康 CLI：

```bash
ECC_ROOT="${CLAUDE_PLUGIN_ROOT:-$(node -e "var p=require('path'),f=require('fs'),h=require('os').homedir(),d=p.join(h,'.claude'),q=p.join('scripts','lib','utils.js');if(!f.existsSync(p.join(d,q))){try{var b=p.join(d,'plugins','cache','everything-claude-code');for(var o of f.readdirSync(b))for(var v of f.readdirSync(p.join(b,o))){var c=p.join(b,o,v);if(f.existsSync(p.join(c,q))){d=c;break}}}catch(x){}}console.log(d)")}"
node "$ECC_ROOT/scripts/skills-health.js" --dashboard
```

For a specific panel only:
仅显示特定面板：

```bash
ECC_ROOT="${CLAUDE_PLUGIN_ROOT:-$(node -e "var p=require('path'),f=require('fs'),h=require('os').homedir(),d=p.join(h,'.claude'),q=p.join('scripts','lib','utils.js');if(!f.existsSync(p.join(d,q))){try{var b=p.join(d,'plugins','cache','everything-claude-code');for(var o of f.readdirSync(b))for(var v of f.readdirSync(p.join(b,o))){var c=p.join(b,o,v);if(f.existsSync(p.join(c,q))){d=c;break}}}catch(x){}}console.log(d)")}"
node "$ECC_ROOT/scripts/skills-health.js" --dashboard --panel failures
```

For machine-readable output:
获取机器可读输出：

```bash
ECC_ROOT="${CLAUDE_PLUGIN_ROOT:-$(node -e "var p=require('path'),f=require('fs'),h=require('os').homedir(),d=p.join(h,'.claude'),q=p.join('scripts','lib','utils.js');if(!f.existsSync(p.join(d,q))){try{var b=p.join(d,'plugins','cache','everything-claude-code');for(var o of f.readdirSync(b))for(var v of f.readdirSync(p.join(b,o))){var c=p.join(b,o,v);if(f.existsSync(p.join(c,q))){d=c;break}}}catch(x){}}console.log(d)")}"
node "$ECC_ROOT/scripts/skills-health.js" --dashboard --json
```

## Usage
## 使用方法

```
/skill-health                    # Full dashboard view
/skill-health --panel failures   # Only failure clustering panel
/skill-health --json             # Machine-readable JSON output
```

## What to Do
## 操作说明

1. Run the skills-health.js script with --dashboard flag
1. 使用 --dashboard 标志运行 skills-health.js 脚本
2. Display the output to the user
2. 向用户展示输出
3. If any skills are declining, highlight them and suggest running /evolve
3. 若有技能指标下降，高亮显示并建议运行 /evolve
4. If there are pending amendments, suggest reviewing them
4. 若有待处理的修改，建议进行审查

## Panels
## 面板

- **Success Rate (30d)** — Sparkline charts showing daily success rates per skill
- **成功率（30 天）** — 显示每个技能每日成功率的迷你图表
- **Failure Patterns** — Clustered failure reasons with horizontal bar chart
- **失败模式** — 带水平柱状图的聚类失败原因
- **Pending Amendments** — Amendment proposals awaiting review
- **待处理修改** — 等待审查的修改提案
- **Version History** — Timeline of version snapshots per skill
- **版本历史** — 每个技能的版本快照时间线
