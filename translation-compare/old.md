---
title: 代码并不廉价：如何用软件基础能力放大 AI 的产出
author: Yanli Liu
url: https://ai.gopubby.com/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals-40fff5a00f9f
translated: 2026-05-23
tags:
  - Technology
  - Programming
  - Artificial Intelligence
---

# 代码并不廉价：如何用软件基础能力放大 AI 的产出

## Karpathy、Pocock，以及 AI 编码热潮遗忘的软件基础能力

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/01.jpg)
*图片来源：Quino Al / Unsplash*

[免费阅读本文](https://medium.com/ai-advances/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals-40fff5a00f9f?sk=e8503f014e7cb003f3ced0c4da8365fe)

2025 年 2 月，Andrej Karpathy 创造了 "**vibe coding**" 这个说法：描述你想要什么，让 AI 写代码，然后忘掉代码的存在。这个概念迅速火了。所有人都想相信，编码已经变得像说话一样简单。

一年后，Karpathy 给它换了个名字。新术语是："**agentic engineering**"。他的解释很直接。"'Engineering' 是为了强调其中有艺术、科学和专业能力。" 他在几周内从 80% 手动编码，转向 80% agent 编码，然后以艰难方式发现模型是 "jagged" 的：它们在难题上表现出色，却会在显而易见的地方绊倒。

数据也支持他的看法。GitClear 的 [2025 code quality study](https://www.gitclear.com/ai_assistant_code_quality_2025_research) 发现，AI 共同编写的 pull requests 比纯人类 PR 多出 **1.7x 的问题**。从 2021 年到 2024 年，复制粘贴代码行从 8.3% 上升到 12.3%。与此同时，AI 现在写下了 [GitHub 上全部代码的 41%](https://github.blog/news-insights/octoverse/octoverse-a-new-developer-joins-github-every-second-as-ai-leads-typescript-to-1/)，Copilot 付费订阅者达到 470 万。

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/02.webp)
*从 Vibe Coding 到 Agentic Engineering：行业 12 个月的警醒时刻——作者绘图*

我们正在生产比以往更多的代码。它们也比以往更容易出 bug。这不是生产力爆发，而是以复利计息的技术债。

Matt Pocock 在他的 [AI Hero 会议演讲](https://www.youtube.com/watch?v=v4F1gFy-hqg) 里说得很直白："*代码并不廉价。坏代码正处在前所未有的昂贵时刻*。"

他的推理是：如果你的代码库难以修改，就无法吸收 AI 的产出。每个建议、每个生成函数、每次自动重构，都会撞上糟糕架构带来的摩擦。AI 不会修复结构性问题。它会放大这些问题。

本文的论点是：**AI 是乘数，不是魔杖。** 它会放大你放在它面前的任何架构。好的结构会在每次交互中带回更多价值。坏的结构会随着每个 prompt 加速腐烂。

有五项软件基础能力，把能自信交付的团队和淹没在 AI 生成意大利面代码里的团队区分开来。它们都不新。但它们比以往任何时候都更重要。

## 1\. 先对齐，再开工

Karpathy 指出了 AI 编码 agents 的四种结构性失败模式。第一种是："*模型会替你做出错误假设，然后不加确认就一路跑下去。*" 你以为自己要的是一个简单的 API endpoint。AI 却构建了一个带认证、限流和数据库迁移的 microservice，而这些你从没提过。

问题不在于 AI 的能力。问题在于，你和 AI 对正在构建的东西没有共享的心智模型。

Frederick Brooks 在 *The Design of Design* 中写过这个问题。他称之为 "**design concept**"，也就是你正在创造的东西背后那套不可见、共享的理论。它不是文档。不是 spec 文件。它是协作者之间关于自己在构建什么、为什么构建的理解。当两个人类结对编程时，他们会通过对话自然建立它。当你提示 AI 时，这种共享理解默认并不存在。

Pocock 的解决方案是一个 [Claude Code skill](https://github.com/mattpocock/skills)，他称之为 "**grill me**"。完整指令只有两行：

> *围绕这个计划的每个方面不断追问我，直到我们达成共享理解。沿着设计树的每个分支走下去，逐一解决各项决策之间的依赖关系。*

它火了。GitHub stars 超过 97,000。AI 会问 40、60，有时甚至 100 个问题，直到它满意为止。它把 agent 变成一个对手，不允许你跳过思考。

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/03.webp)
*没有对齐：返工循环。有了对齐：自信交付。作者绘图。*

**原则：** 在你们共享心智模型之前，不要让 AI 开始写代码。写下 brief，定义约束，先回答那些难问题。你花 20 分钟对齐，会省下原本要用来撤销错误假设的 3 小时。

## 2\. 说同一种语言

如果你和 AI 用同一个词表达不同意思，只对齐计划还不够。

Karpathy 的第二种失败模式是："*过度复杂的解决方案。你要一个 10 行函数，却得到一个 200 行企业框架。*"

AI 不是恶意的。它只是不了解你的词汇。你说 "handler" 时，是指 HTTP handler、event handler，还是 log handler？你说 "service" 时，是指 microservice、background worker，还是一个 class？AI 会猜。它会猜错。然后它会为那个错误猜测建起一座 200 行纪念碑。

这是软件工程 20 年前已经解决过的问题。Eric Evans 在 2003 年出版了 ***Domain-Driven Design***，核心概念是 "**ubiquitous language**"，也就是开发者、领域专家和代码库本身都一致使用的共享词汇。每次对话、每个变量名、每个 API endpoint，都用同样的术语表达同样的意思。

Evans 最近[告诉 InfoQ](https://www.infoq.com/news/2024/03/Evans-ddd-experiment-llm/)："在 bounded context 的 ubiquitous language 上训练语言模型，相比使用通用 LLMs，会让它对特定需求有用得多。" 你的领域 glossary 现在成了 prompt-engineering 资产。

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/04.webp)
*你的 glossary 现在是一项 prompt-engineering 资产。作者绘图*

Pocock 也为这个做了一个 skill。他的 "**ubiquitous language**" skill 会扫描你的代码库，提取术语，并生成一个充满定义表的 markdown 文件。他报告说，通过阅读 AI 的 thinking traces，ubiquitous language 不只是改善了规划。它还让 AI *思考* 得更不啰嗦。浪费的 tokens 更少。实现更准确。生成的代码真的匹配了规划。

DDD 的 bounded contexts 在这里同样适用。在 multi-agent 设置中，不清晰的术语所有权会导致 "context bleed"：agents 踩进彼此的领域、复制逻辑，或互相输出矛盾结果。Evans 20 年前在人类团队中描述的同一种失败模式，如今出现在 agent orchestration 中。

**原则：** 建一个 glossary。精确定义你的领域术语。在项目文档和 prompts 中引用它。如果 AI 知道 "order" 指的是 "Order Management context 中的客户承诺"，而不是 "排序指令"，它就会停止猜测，开始生成契合系统的代码。

## 3\. 测试先行，小步交付

你已经对齐了计划。你们说着同一种语言。AI 构建的也正是你要求的东西。然后它不能运行。

这是最耗时间的失败模式，因为代码*看起来*是对的。读起来不错。变量名也合理。但你一运行它就倒下，因为 AI 一次性生成了 400 行，却没有检查其中任何一行。

*The Pragmatic Programmer* 把这称为 "outrunning your headlights"。反馈速度就是你的速度上限。开得比车灯能照到的范围更快，你就会撞上看不见的东西。AI agents 默认是在关灯开车。它们会先生成大批量代码，然后也许才在事后想想如何验证。

Anthropic 的 [best practices](https://code.claude.com/docs/en/best-practices) 推荐 writer/reviewer pattern："*一个 Claude 写测试，第二个 Claude 写代码让测试通过*。" OpenAI 的 [Codex documentation](https://developers.openai.com/codex/learn/best-practices) 也说得很直白："*没有测试时，Codex 会用自己的判断来验证工作。测试创造了一个外部真相来源。*"

两个平台都得出了同一个答案：test-driven development。不是作为一种理念，而是作为一种机械约束，迫使 AI 小步前进。

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/05.webp)
*面向 AI 编码的 Red-Green-Refactor。作者绘图*

先写测试。让 AI 把测试跑通。再重构。Red-green-refactor 不是 Agile 时代的遗物。**它是防止熵增的反馈循环。** 每个循环消耗的 tokens、context window 空间以及你的 review 时间，都只是一次大批量重写成本的一小部分。

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/06.webp)
*经过验证的小步前进 vs 大批量熵增螺旋。作者绘图。*

2025 年 5 月一项关于 AI 生成代码的 [arXiv study](https://arxiv.org/html/2605.02741) 发现，随着 agents 从孤立脚本进入多模块系统，code smell 密度会增加。没有测试边界时，腐烂会随代码库一起扩张。有了测试边界，每个模块都会保持诚实。

**原则：** 用 AI 写代码时，TDD 不是可选项。它是让产出保持可信的限速器。写测试，让 agent 实现，验证，重复。小循环，高信心。

## 4\. 构建深模块，而不是宽铺开

即使测试通过了、构建的也是正确东西，随着代码库增长，仍会出现一个结构性问题。AI 开始迷路。它找不到正确文件。它误解依赖关系。它做出的修改会破坏三个模块之外的东西。

John Ousterhout 在 *A Philosophy of Software Design* 中描述过这一点。他区分了深模块和浅模块：

-   深模块把大量功能隐藏在简单接口背后。你不需要理解内部实现也能使用它们。
-   浅模块则相反：功能不多，但接口复杂，迫使你理解底下所有东西。

AI agents 极其擅长生成浅模块。许多小文件，每个只做一件小事，每个暴露多个函数，每个又依赖另外三个小文件。它看起来干净。代码评审时读起来也不错。但它会给下一次 AI session 制造导航噩梦，因为 agent 必须穿过几十个彼此相连的 blobs，才能理解你的代码到底在做什么。

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/07.webp)
*浅模块：AI 会迷路。深模块：AI 能导航并生成更好的代码。作者绘图。*

Pocock 在他的演讲中直观展示了这一点。一个充满浅模块的代码库，看起来像散落的点，点之间有纠缠的箭头。同一份代码被重组为深模块后，看起来像少数几个大块，上面只有简单连接。AI 能在第二种结构中导航，并在其中生成更好的代码，因为它可以围绕接口推理，而不必把每个实现细节都加载进 context window。

那篇关于 AI-generated code smells 的 [arXiv paper](https://arxiv.org/html/2605.02741) 从实证角度确认了这一点。随着 agents 从孤立脚本进入多模块系统，code smell 密度会上升。LLMs 在推理时不会跟踪架构复杂性。模块结构越碎片化，AI 的输出退化得越严重。

这里也有市场信号。TypeScript 在 2025 年 8 月成为 [GitHub 的 #1 language](https://github.blog/news-insights/octoverse/octoverse-a-new-developer-joins-github-every-second-as-ai-leads-typescript-to-1/)。部分原因是：带类型、结构良好的代码，会让 AI-assisted development 更可靠。开发者正在自发选择能复利 AI 回报的架构，就像一个充分分散的投资组合会复利市场回报一样。那些坚持无类型、碎片化代码库的人，正在用返工为此买单。

**原则：** 审计你的代码库，寻找浅模块。把相关代码包进接口简单的深模块。在边界上测试。AI 不需要看到一切。它需要看到正确的接口。

## 5\. 设计接口，委派实现

如果前四条原则保护的是 AI 输出的质量，那么这一条保护的是你。

自从 AI 编码工具进入工作流之后，如果你感觉比以往更心力交瘁，请举手。你并不孤单。Pocock 在他的会议现场问过同一个问题。几乎每只手都举了起来。

这种疲惫来自试图审查一切。每个生成函数、每个重构后的 class、agent 创建的每个新文件。你交付的代码比以往任何时候都多，但你的大脑仍然是理解所有代码的瓶颈。

Kent Beck 的建议是："*每天都投资于系统设计。*"

specs-to-code 运动做的正相反。它从设计中撤资。它把代码库当作可以从 prompt 中重新生成的一次性产物。这就是你最终要 review 400 行不是你写的、而且几乎看不懂的代码的原因。

另一种选择是灰盒模型。你拥有接口。你拥有边界上的测试。你让 AI 处理模块内部的东西。对于非关键模块，你不需要 review 每一行实现。你需要验证契约是否成立。

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/08.webp)
*你拥有战略层。AI 拥有战术层。作者绘图。*

这就是 Karpathy 所说的新核心技能："判断力：委派什么、如何说明、如何快速审查。" 你少写代码，不是因为你懒。你少写代码，是因为你把时间花在架构、接口和验证上。也就是战略层。

Pocock 把它描述为战术程序员和战略程序员的区别。AI 是战术程序员，是地面上负责修改代码的军士。你才是那个思考系统设计、模块边界，以及各个部分如何拼在一起的人。这不是降级。这是晋升。

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/09.webp)
*灰盒模型：拥有边界，委派内部。作者绘图。*

Anthropic 的内部数据显示，在大规模重构任务上有 [2–3x productivity gains](https://www.anthropic.com/research/how-ai-is-transforming-work-at-anthropic)。但这些收益来自那些足够信任架构、因此能放心委派的团队，而不是来自那些 review 每一行生成代码的团队。

**原则：** 写接口。指定契约。把实现委派给 AI。通过测试验证，而不是逐行 code review。你的工作是系统设计。让 agent 处理剩下的部分。

## 工具箱：从原则到实践

没有工具的原则只是建议。下面这些是你今天就可以安装的东西。

Pocock 把他的 skills 作为一个 [open-source repo](https://github.com/mattpocock/skills) 发布了。每一个都直接对应上面的原则：

-   **grill-me** 在 AI 写任何东西之前强制建立共享理解（原则 1）
-   **ubiquitous-language** 扫描你的代码库，并构建领域 glossary（原则 2）
-   **tdd** 对每个模块强制执行 red-green-refactor 循环（原则 3）
-   **improve-codebase-architecture** 识别浅模块，并把它们包进深模块（原则 4）
-   **writer-prd** 在 PRD 中指定模块变更和接口契约（原则 5）

但这些原则不只是某个开发者的观点。主流平台各自独立构建了基础设施，用来强化同样的想法。

Anthropic 的 Claude Code 使用 [CLAUDE.md](https://docs.anthropic.com/en/docs/claude-code/claude-md) 作为对齐契约，支持面向孤立任务、拥有干净 context windows 的 subagents，并推荐 writer/reviewer pattern：一个 agent 写测试，另一个写实现。OpenAI 的 Codex 使用 [AGENTS.md](https://developers.openai.com/codex/guides/agents-md) 达到同样目的，并用明确的 "done when" 标准，强制 agent 在认为任务完成之前进行测试验证。GitHub Copilot 的 [agent mode](https://github.blog/ai-and-ml/github-copilot/agent-mode-101-all-about-github-copilots-powerful-mode/) 会构建你 repo 的 semantic index（检索准确率比 2025 年初高 37.6%），并支持 custom agents 和 prompt files 作为可复用蓝图。

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/10.webp)
*三个平台上的 Principle → Tool 映射。作者绘图。*

三个平台。同一个结构性答案。对齐文档、测试循环、模块边界。它们全都收敛到这里，因为它们都撞上了同一堵墙：模型能力不是瓶颈。代码架构才是。

## 主线

Brooks 在 1975 年写过 shared design concepts。Evans 在 2003 年发布了 ubiquitous language。Ousterhout 在 2018 年画出了 deep module 图。Beck 几十年来一直在说 "每天都投资于设计"。

Karpathy 在压力之下，用地球上最强大的 AI 模型构建系统时，只用了几周就发现了同样的教训。Pocock 把它们提炼成 skills，并因此走红，因为成千上万的开发者都在其中认出了自己的痛苦。

这些都不是新知识。重点正在于此。

![](/docs/ai-insights/2026-05/23/images/code-is-not-cheap-how-to-multiply-your-ais-output-with-software-fundamentals/11.webp)
*5 条原则：你的架构就是你的 AI 战略。作者绘图。*

一个聪明的 prompt 会给你一次好的输出。一个结构良好的代码库会给你一千次。每个清晰接口、每个被强制执行的测试边界、每个深模块，都会让之后每次 AI 交互的价值复利增长。跳过架构，你复利的就是债务。

代码并不廉价。你的架构就是你的 AI 战略。

## 离开前！🦸🏻‍♀️

如果你喜欢我的故事，并且想支持我：

1.  给我一点 Medium 上的爱 💕（鼓掌、评论和高亮），你的支持对我意义重大。👏
2.  在 Medium 上[关注我](https://medium.com/@yanli.liu/about)，并订阅以获取我的最新文章🫶
