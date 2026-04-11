Parse the following from $ARGUMENTS:
从 $ARGUMENTS 中解析以下内容：
1. `brief` — the user's description of the design to create
1. `brief` — 用户对要创建的设计的描述
2. `--max-iterations N` — (optional, default 10) maximum design-evaluate cycles
2. `--max-iterations N` — （可选，默认 10）设计-评估的最大循环次数
3. `--pass-threshold N` — (optional, default 7.5) weighted score to pass (higher default for design)
3. `--pass-threshold N` — （可选，默认 7.5）通过所需的加权分数（设计模式默认值更高）

## GAN-Style Design Harness
## GAN 风格的设计测试框架

A two-agent loop (Generator + Evaluator) focused on frontend design quality. No planner — the brief IS the spec.
一个专注于前端设计质量的双代理循环（生成器 + 评估器）。无需规划器——简介即规格。

This is the same mode Anthropic used for their frontend design experiments, where they saw creative breakthroughs like the 3D Dutch art museum with CSS perspective and doorway navigation.
这与 Anthropic 在其前端设计实验中使用的模式相同，他们在实验中取得了创意突破，例如使用 CSS perspective 和门廊导航实现的 3D 荷兰艺术博物馆。

### Setup
### 准备工作
1. Create `gan-harness/` directory
1. 创建 `gan-harness/` 目录
2. Write the brief directly as `gan-harness/spec.md`
2. 将简介直接写入 `gan-harness/spec.md`
3. Write a design-focused `gan-harness/eval-rubric.md` with extra weight on Design Quality and Originality
3. 编写以设计为重点的 `gan-harness/eval-rubric.md`，对设计质量和原创性赋予额外权重

### Design-Specific Eval Rubric
### 设计专项评估评分标准
```markdown
### Design Quality (weight: 0.35)
### Originality (weight: 0.30)
### Craft (weight: 0.25)
### Functionality (weight: 0.10)
```

Note: Originality weight is higher (0.30 vs 0.20) to push for creative breakthroughs. Functionality weight is lower since design mode focuses on visual quality.
注意：原创性权重更高（0.30 对比 0.20），以推动创意突破。功能性权重更低，因为设计模式专注于视觉质量。

### Loop
### 循环
Same as `/project:gan-build` Phase 2, but:
与 `/project:gan-build` 阶段 2 相同，但：
- Skip the planner
- 跳过规划器
- Use the design-focused rubric
- 使用以设计为重点的评分标准
- Generator prompt emphasizes visual quality over feature completeness
- 生成器提示词强调视觉质量而非功能完整性
- Evaluator prompt emphasizes "would this win a design award?" over "do all features work?"
- 评估器提示词强调"这能赢得设计奖项吗？"而非"所有功能是否正常工作？"

### Key Difference from gan-build
### 与 gan-build 的关键区别
The Generator is told: "Your PRIMARY goal is visual excellence. A stunning half-finished app beats a functional ugly one. Push for creative leaps — unusual layouts, custom animations, distinctive color work."
生成器被告知："你的首要目标是视觉卓越。一个令人惊艳的半成品应用胜过一个功能完整却丑陋的应用。追求创意飞跃——非常规布局、自定义动画、独特的色彩运用。"
