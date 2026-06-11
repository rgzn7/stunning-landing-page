# Stunning Landing Page

面向 Codex 及其他类似 AI agent 的可移植 skill，用于制作具有滚动叙事、签名动画和高级视觉质感的 landing page。给出一个主题或产品后，它会先判断品牌情绪与页面剧本，再产出可直接打开的单文件 HTML 页面。

## 在线预览

- [山雾咖啡：温暖品牌系](https://rgzn7.github.io/stunning-landing-page/examples/coffee-brand-warm/)
- [CodeLens：暗夜科技系](https://rgzn7.github.io/stunning-landing-page/examples/ai-devtool-dark/)
- [码丁星球：活泼游戏系](https://rgzn7.github.io/stunning-landing-page/examples/kids-coding-playful/)
- [XISY Photo Journal：摄影作品集](https://rgzn7.github.io/stunning-landing-page/examples/photo-journal/)（额外示例，未纳入 benchmark）

## 效果预览

![山雾咖啡示例](docs/screenshots/coffee-brand-warm.png?v=e68b2b6)

![CodeLens 示例](docs/screenshots/ai-devtool-dark.png?v=e68b2b6)

![码丁星球示例](docs/screenshots/kids-coding-playful.png?v=e68b2b6)

![XISY Photo Journal 示例](docs/screenshots/photo-journal.png?v=photo-journal-20260611)

## 能力范围

- 根据主题推导视觉基调、配色、字体、动效语气和页面叙事。
- 默认输出单个 `index.html`，CSS/JS 内联，动画库通过 CDN 引入。
- 使用 GSAP、ScrollTrigger、Lenis、SplitType 等实现滚动编排、视差、文字拆分、横向滚动、数字动画和微交互。
- 内置质量清单，要求移动端适配、`prefers-reduced-motion` 降级、无本地破图路径、无 emoji 图标依赖。

## 安装

将仓库克隆到任意兼容 Skills 约定的 agent skills 目录中。Codex 用户可以使用下面的路径作为示例：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/rgzn7/stunning-landing-page.git ~/.codex/skills/stunning-landing-page
```

其他类似 agent 也可以直接读取或导入本仓库目录；只要能加载 `SKILL.md` 和 `references/`，就可以复用同一套工作流。目录名建议保持为 `stunning-landing-page`。

## 使用示例

```text
使用 $stunning-landing-page 给我的精品手冲咖啡品牌做一个官网 landing page，主打云南高山豆和手工烘焙，输出单个 HTML 文件。
```

```text
使用 $stunning-landing-page 为一个 AI 代码审查工具做产品发布页，希望像 Linear/Vercel 那样有暗夜科技感和滚动动画。
```

```text
使用 $stunning-landing-page 给儿童编程 App 做宣传页，目标受众是家长和孩子，风格要活泼、有弹性动画。
```

## 项目结构

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
│   ├── animation-patterns.md
│   ├── quality-checklist.md
│   ├── tech-toolkit.md
│   └── theme-to-style.md
├── evals/
│   └── evals.json
├── examples/
│   ├── coffee-brand-warm/
│   ├── ai-devtool-dark/
│   ├── kids-coding-playful/
│   └── photo-journal/
└── docs/
    ├── benchmark.md
    └── screenshots/
```

## 评测

当前评测包含 3 个典型场景：精品咖啡品牌、AI 开发者工具、儿童编程 App。评测维度覆盖滚动动画、动效降级、Hero 冲击力、无破图风险、主题文案匹配、视觉风格匹配和图标规范。`examples/photo-journal/` 是额外迁移示例，暂未纳入当前 benchmark 数据。

详见 [docs/benchmark.md](docs/benchmark.md)。

## 说明

SKILL.md 中提到的 Apple、Duolingo、Canva、Stripe、Linear 等名称仅用于描述公开网站中常见的动效模式和设计手法，本项目与这些品牌没有从属或授权关系。

## License

MIT
