---
title: 《驾驭工程：2026 年每位人工智能工程师都需要了解的知识》
author: Yanli Liu
url: https://ai.gopubby.com/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026-0ab649e5686a
translated: 2026-06-17
excerpt: 三个阵营，三种架构——而 Opus 4.7 刚刚证明了这一切。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/01.thumb.webp
---

# 驾驭工程：2026 年每位人工智能工程师都需要了解的知识

三个阵营，三种架构——而 Opus 4.7 刚刚证明了这一切。

[免费阅读](https://medium.com/ai-advances/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026-0ab649e5686a?sk=401ffeaf5022bddbc35e20c9dba33c80)

2026 年 2 月，OpenAI 发布了一篇博客文章，悄然重新定义了软件工程师的日常工作内容。文章标题只有两个词：“[Harness Engineering](https://openai.com/index/harness-engineering/)”。

这篇文章描述了一个小团队如何在不手动编写任何一行代码的情况下，交付了一百万行生产代码。

相反，他们设计了人工智能 agent 运行其中的环境：**约束条件、反馈回路、文档结构、依赖关系规则**。agent 编写代码，而人类设计了确保 agent 可靠的系统。

几周之内，Anthropic 就同一概念发表了三篇独立的工程论文（[有效的 harness](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)、[harness 设计](https://www.anthropic.com/engineering/harness-design-long-running-apps)、[托管 agent](https://www.anthropic.com/engineering/managed-agents)）。ThoughtWorks 将其[形式化为一套框架](https://martinfowler.com/articles/harness-engineering.html)。Red Hat [编写了实施指南](https://developers.redhat.com/articles/2026/04/07/harness-engineering-structured-workflows-ai-assisted-development)。Hugging Face 的 [Philipp Schmid](https://www.philschmid.de/agent-harness-2026) 称其为“2026 年最重要的学科”。

**一门新的工程学科在 90 天内诞生了。**

它的发展速度已经远远超出了所有人的预期。昨天，Anthropic 发布了 [Opus 4.7](https://www.anthropic.com/news/claude-opus-4-7)——这是他们在不到一年内推出的第三代模型。每一代不仅仅是改进了模型，**更简化了 harness**。三月份还在承重的部件，到了四月份就变成了累赘。

> 这门学科创立仅 90 天，就已经开始改写自身的规则。

![Photo by Sergey Shmidt on Unsplash](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/01.jpg)

数据说明了问题的紧迫性。LangChain 在 Terminal Bench 2.0 上把同一个模型跑了两次：一次用旧 harness，一次用新 harness。同样的模型，不同的 harness。得分从 52.8% 跃升至 66.5%。

Vercel 采取了截然相反的做法，移除了其 agent 80% 的工具。结果如何？性能反而提升了。更少的工具、更严格的约束，带来更强大的输出。

**如果说 2025 年是人工智能 agent 证明自己能够编写代码的一年，那么 2026 年就是我们发现 agent 本身从来都不是难点的一年。难点在于 harness。**

**但真正让这一刻变得有趣的是**：三大阵营应运而生，他们对 harness 该做什么有着截然不同的理解。他们对问题本身达成共识，但在架构设计上却存在分歧。而选择哪一种并非纸上谈兵，它决定了你的投入成本、所需的工程师数量，以及你的 agent 最终产出的是真正可用的软件，还是昂贵的幻觉。

本文将拆解这三种方法，向你展示每种方法在实践中的实际样貌，并为你提供选择正确方法的决策框架。

## harness 究竟是什么

最简单的定义来自 ThoughtWorks 的 Sunit Parekh 在“[超越 Vibe 编码](https://www.thoughtworks.com/insights/blog/generative-ai/beyond-vibe-coding-the-five-building-blocks-of-aI-native-engineering)”一文中：

> **agent = 模型 + harness。**

harness 就是除模型本身之外的一切。它包括：让 agent 始终朝正确方向运行的约束条件；用于捕获错误的反馈回路；告知 agent 当前位置和已完成工作的文档；以及 agent 被授权使用的工具。剥离 harness，你得到的就是一个只能靠猜测来摸索你代码库的原始语言模型。而加上合适的 harness，你就能得到一个可以交付生产代码的系统。

**OpenAI 团队**在命名时借用了一个更古老的比喻。harness 就是马具：缰绳、鞍座和马嚼子，它们引导着一匹力量强大却难以捉摸的动物朝着有用的方向前进。你无法让马变得更聪明，你只能设计出能够发挥其优势的装备。

**Philipp Schmid** 提供了一个更技术性、也值得内化的类比。可以把它想象成一台电脑：模型相当于 CPU（原始处理能力），上下文窗口相当于 RAM（有限的、易失的工作内存），harness 相当于操作系统（管理 CPU 何时看到什么），而 agent 则相当于运行在所有这些组件之上的应用程序。

![Diagram by Author: The OS Analogy: Model as CPU, Harness as Operating System](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/02.webp)

如果你有金融或风险管理背景，还有一种更直接的思考方式。

**harness 就是一套控制框架。** 它包含一系列策略、检查点和审计跟踪，用于确保自主系统在可接受的范围内运行。合规团队几十年来一直在构建这类东西，人工智能领域只是给它们起了个新名字。

## 这些产物实际长什么样

![Diagram by Author: The Harness Formula: Agent = Model + Harness](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/03.webp)

大多数文章只是抽象地定义一下 harness，然后就止步于此。这远远不够。如果你打算构建一个 harness，你需要看清它在实践中的各个组成部分长什么样。以下是所有主流实现中都会出现的关键产物。

**AGENT.md / CLAUDE.md 文件（通用模式，名称不同）。** 这些是分布在整个代码库中的 markdown 文件，agent 会在每次会话开始时读取它们。OpenAI 的 Codex 将其称为 AGENT.md，Anthropic 的 Claude Code 将其称为 CLAUDE.md，Cursor 则使用 .cursorrules。名称虽有不同，但原理相同。它们包含项目上下文、编码规范、架构决策以及“我们在这里如何做事”的指导原则。OpenAI 的 Sora Android 团队在整个仓库中维护这些文件。agent 读取它们，就像新工程师在冲刺中途加入团队时阅读入职文档一样。每个主要模块对应一个文件，并随着项目的演进而更新。

```
# AGENT.md - Authentication Module
## Architecture
- OAuth2 flow with PKCE, tokens stored in encrypted SharedPreferences
- Never store tokens in plaintext. Never log token values.
## Conventions  
- All auth errors route through AuthErrorHandler
- Retry logic: 3 attempts with exponential backoff
## Current State
- Migration from v1 to v2 token format in progress (see issue #247)
```

**JSON 功能列表（Anthropic 模式）。** 当 agent 在多个会话中构建整个应用程序时，每个新会话开始时都是一个空白的上下文窗口。agent 如何知道哪些已经完成、接下来该做什么？Anthropic 的答案是用一个 JSON 文件，它既是项目规范，也是进度跟踪器。每个条目定义了一个功能、它的验证步骤以及通过/失败状态。在他们的 claude.ai 克隆演示中，这份列表包含超过 200 个独立功能，所有功能最初都处于“failing”状态。

agent 在每次会话开始时读取此文件，挑出优先级最高的失败功能，实现它，对照测试步骤验证，将其标记为“passing”，然后提交。它把测试套件和项目看板合在了一个文件里，既可供人阅读，也可供 agent 阅读。

```json
{
  "category": "authentication",
  "feature": "Password reset via email",
  "verification": [
    "Click 'Forgot Password' on login page",
    "Enter registered email address",
    "Verify reset email received within 30 seconds",
    "Click reset link, enter new password",
    "Confirm login with new password succeeds"
  ],
  "status": "failing"
}
```

**为什么用 JSON 而不是 markdown？** Anthropic 发现，与 Markdown 文件相比，模型“更不容易不恰当地更改或覆盖 JSON 文件”。这虽是一个小细节，但当 agent 自主运行数小时时，就显得至关重要了。

**会话初始化例程（Anthropic 模式）。** 每个编码会话都遵循相同的 7 步启动序列：确认工作目录，读取 git 日志和进度文件，查阅功能列表以找到优先级最高的未完成功能，启动开发服务器，运行基本的端到端验证，实现单个功能，然后提交并附上描述性消息和进度更新。

这并非可选项。如果没有它，每次新会话都将从头开始，agent 会浪费前 20 分钟来弄清楚已经完成了哪些工作。

**结构化任务模板（Red Hat 模式）。** 在开始任何编码之前，harness 会使用语言服务器和代码分析工具分析实际代码库，生成一份有根有据的影响图。然后，它会生成一个任务模板，其中包含真实的文件路径、真实的符号名称、需要遵循的现有模式以及具体的验收标准。无需猜测，也不会臆想文件路径。

**冲刺契约（Anthropic 模式）。** 在生成器 agent 开始编码之前，它会与评估器 agent 进行协商。生成器提出它将要构建的内容以及如何验证成功。评估器审查提案的完整性。只有在双方达成一致后，才会开始实施。这是一种轻量级的设计评审，优秀的工程团队早已在做这种评审，只不过参与双方都是人工智能 agent。

![Diagram by Author: Harness Artifacts Map: The five key artifacts across all implementations](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/04.webp)

## 共同的线索

把这些产物放在一起观察，就会浮现出一个规律。每一个产物都是为了回答同一个问题而设计的：**“agent 在编写任何一行代码之前需要知道什么？”**

事实证明，答案是“需要知道很多”。它在代码库中的位置、已经完成的工作、什么算“好”、哪些是禁区，以及如何验证自身的工作。这并非智能，而是上下文。而上下文，最终才是 harness 工程的真正产物。

## 三大阵营

“harness 工程”这个术语并非源于某个委员会或会议主题演讲。三个团队各自独立地撞上了同一堵墙，并各自搭出了一架不同的梯子翻越它。

### OpenAI：“我们有一百万行没人写过的代码”

![Diagram by Author: Three Camps of Harness Engineering: OpenAI, Anthropic, ThoughtWorks](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/05.webp)

OpenAI 的 Codex 团队遇到了一个规模近乎荒谬的问题。他们正在构建一个生产环境应用，而所有代码都是由 agent 编写的。不是一部分，而是全部。一百万行代码，没有一行是人手编写的。

在这种规模下，逐行审查代码的传统方法就行不通了。你不可能审查一百万行代码。你能做的，是把环境设计得足够周密，让 agent 一开始就能产出可供审查的输出。

他们付出惨痛代价才领悟到的核心教训是：

> 给 Codex 一张地图，而不是一本 1000 页的使用手册。

他们构建了严格的依赖流（先 Types，再 Config、Repo、Service、Runtime，最后 UI），并通过结构化测试来强制执行。他们在整个代码库中嵌入 AGENT.md 文件作为分布式文档。他们将 agent 直接接入 CI/CD 流水线，从而让每次更改都能自动进行测试。

其理念是：**先设计环境，然后让 agent 在其中自由发挥。** 人的角色是架构师，而不是程序员。

它奏效的证据来自 [Sora Android 版本](https://openai.com/index/shipping-sora-for-android-with-codex/)。四位工程师，历时 28 天，消耗了约 50 亿个 token，最终这款应用以 99.9% 的无崩溃率荣登 Play 商店榜首。Codex 每周处理 70% 的内部 pull request。工程师们把时间投入到高层架构、规划和验证上，其余工作都由 agent 完成。

![Diagram by Author: OpenAI/Codex Dependency Flow: Types → Config → Repo → Service → Runtime → UI](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/06.webp)

### Anthropic：“我们的 agent 总在夸赞它们自己那份漏洞百出的工作”

Anthropic 的问题更为微妙，在某些方面也更难。他们构建的是需要长时间运行的 agent，这些 agent 需要在数小时的自主工作中产出完整的应用程序。模型本身能力足够，问题在于质量控制。

当他们要求 agent 评估自身的输出时，它会

> 自信地称赞这份工作，即便在人类观察者看来，质量明显平庸。

**自我评估不起作用。** 这个 agent 既是学生又是老师，它给自己打了全 A。

他们的解决方案从生成对抗网络（GAN）中汲取灵感：**把做工作的那个东西，和评判工作的那个东西分开。这促成了三 agent 架构。** Planner 把简短的提示扩展为全面的产品规格。Generator 每次一个冲刺地实现各项功能。Evaluator 则利用浏览器自动化，像真实用户一样与正在运行的应用程序交互，并根据明确的标准对每个冲刺评分。

关键的洞见是：把一个独立的评估器调教得更具怀疑精神，结果远比让生成器去批判它自己的工作要可行得多。

他们并未止步于此。架构从两个 agent（初始化器加编码器）演变为三个 agent（planner、generator、evaluator），最终发展成他们称之为“托管 agent”的完全解耦系统——在这个系统里，大脑、执行环境和会话日志都是独立且可替换的组件。这种解耦使他们的 P50 首 token 时间缩短了 60%，P95 缩短了 90% 以上。

![Diagram by Author: Anthropic Multi-Agent Architecture: Planner → Generator → Evaluator](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/07.webp)

其理念是：**把执行者与评判者分开，并让评判者难以被取悦。**

### ThoughtWorks：“我们在 50 个客户团队中反复看到同样的失败”

ThoughtWorks 进入 harness 工程领域的起点截然不同。他们并非在开发产品，而是在观察各行各业数十个工程团队尝试采用人工智能 agent，结果发现到处都是同样的失败模式。

拥有 20 多年经验的杰出工程师 Birgitta Böckeler 于 2026 年 4 月发表了三者中[最全面的框架](https://martinfowler.com/articles/harness-engineering.html)。OpenAI 构建了一个系统，Anthropic 构建了一个架构，而 ThoughtWorks 构建了一套分类法。

他们的框架沿两个轴对 harness 控制进行分类。第一个轴：**前馈**（在执行前预判 agent 行为的导引）与**反馈**（观察结果并支持自我纠正的传感器）。两者都无法单独奏效。只有反馈意味着重复犯错。只有前馈意味着你永远无法验证你的导引是否真的有效。

第二个轴：**计算式**（毫秒级运行的确定性检查，如 linter、类型检查器和测试套件）与**推断式**（由另一个 LLM 进行的语义分析，速度较慢、成本较高，但能发现代码分析发现不了的问题）。

然后，他们把一切归入三个治理类别：可维护性（最成熟，linter 和覆盖率工具已经运转良好）、架构契合度（强制执行设计模式和性能要求）以及行为（最难，你要验证 agent 是否真的做了被要求的事，而不仅仅是代码能否编译）。

其理念是：**分类、系统化，并给团队一套描述他们所构建之物的共同词汇。**

### 他们为何分道扬镳

这三个阵营造出了不同的东西，是因为它们各自从不同的问题出发。OpenAI 需要大规模交付产品。Anthropic 需要自主 agent 的高质量。ThoughtWorks 需要一个框架，能够帮助任何团队，无论他们用的是哪种 agent 或模型。

在评估哪种方法适合你的处境时，这一点值得记住。问题不在于“哪个阵营是对的？”，而在于“我实际遇到的问题是什么？”

## 三种架构，并排对照

第三部分讲了各阵营从何而来。现在让我们来看看他们实际造了什么，更重要的是，每种架构在哪里崩盘。

### OpenAI/Codex：环境优先型 harness

Codex 的 harness 在你能够前期大力投入、设计 agent 工作环境的场景下效果最佳。回报是下游巨大的自主性，但前期投入也是实打实的。

**它如何运作。** harness 就是代码库本身。AGENT.md 文件提供上下文。结构化测试机械地强制执行架构规则。依赖流（先 Types，再 Config、Repo、Service、Runtime，最后 UI）防止 agent 以错误的顺序构建。CI/CD 流水线自动验证每一次更改。

agent 以高度自主的方式运行：它发起 pull request、响应审查反馈、运行测试、在失败时迭代，并在满足条件时合并。人类不审查每一行代码，而是审查那些让每一行代码都可审查的约束条件。

**它在哪里出彩。** 庞大的代码库。如果你正在构建或维护一个数十万行代码的项目，环境优先方法能够扩展，因为约束条件本身就嵌在仓库结构中。加一个新模块，加一个 AGENT.md，agent 即可在其中工作，无需新的训练或配置。OpenAI 估计，他们交付的时间大约只有手工编写代码所需时间的十分之一。

**它在哪里崩盘。** 这种方法假设你能在 agent 开始工作之前全面定义环境。对于你还在摸索架构的全新项目来说，这很难。它还严重依赖结构化测试和 CI 流水线，而它们能查出代码是否正确，却查不出代码是否优秀。一个函数可能通过所有测试，却仍然是一个糟糕的设计选择。

### Anthropic：多 agent harness

Anthropic 的方法每次运行成本更高，但能抓住环境优先方法漏掉的问题。这是质量与速度之间的权衡，对于那些“坏输出比慢输出代价更高”的应用来说，它值得考虑。

**它如何运作。** 三个各司其职的专门 agent。Planner 接收用户的简短提示（1-4 句话），将其扩展为全面的产品规格，聚焦于交付物和高层设计，同时刻意避开那些可能引发连锁错误的细粒度实现细节。Generator 使用标准技术栈（React、Vite、FastAPI、SQLite/PostgreSQL）逐一实现功能，并在交接前进行自我评估。Evaluator 使用 Playwright 浏览器自动化，像真实终端用户一样与运行中的应用程序交互，根据明确的评分标准测试 UI 功能、API 端点和数据库状态。

每个冲刺开始前，Generator 和 Evaluator 会协商一份“冲刺契约”，明确定义要构建什么以及如何衡量成功。可以把它想象成一个轻量级的设计评审，只不过是在两个 agent 之间进行。

**托管 agent 层**在此基础上更进一步。大脑（Claude 加上 harness）、双手（沙箱和执行环境）以及会话（一份持久的、仅追加的事件日志）都被解耦成独立的接口。如果大脑崩溃，它会从事件日志中恢复。如果沙箱出现故障，它会以工具调用错误的形式暴露出来。凭据永远不会到达那些执行 agent 生成代码的沙箱。

**它在哪里出彩。** 对质量和正确性要求高的应用。评估器能抓住测试单凭自身抓不住的东西：能渲染出来却不可用的 UI 元素、技术上能跑但工作流不直观的功能、返回数据正确但格式错误的 API 端点。Anthropic 的测试表明，单 agent（9 美元，20 分钟）产出了一个有缺陷的应用。而完整的 harness（200 美元，6 小时）则产出了界面精美、功能正确的可运行软件。

**它在哪里崩盘。** 成本和时间。三 agent 系统比单 agent 昂贵得多，而且评估器需要大量的 prompt 调优。开箱即用时，它能识别出真实存在的问题，但随后又会自圆其说地接受它们。让它真正具备批判性，花了 Anthropic 多个开发周期。好消息是：随着模型的改进，harness 变得更简单。Anthropic 的 Opus 4.6 版本完全移除了冲刺分解，并降到单遍评估，相比 Opus 4.5 版本，成本显著降低。到了 Opus 4.7（2026 年 4 月 16 日发布），这一趋势加速：模型现在会自己设计办法、在汇报前验证自己的输出，产出更简洁的代码、更少的包装函数和回退脚手架，工具错误数量也只有以往的三分之一。每一代都在一点点削减评估器的职责描述。

### ThoughtWorks：分类法 harness

ThoughtWorks 并没有构建一个你能直接部署的系统。他们构建的是一种关于 harness 的思维方式，帮助你设计自己的 harness。如果你不打算采用 Codex 或 Claude 的特定工具，这无疑是最有用的方法，但它也需要投入最多的精力才能落地运转。

**它如何运作。** 每个 harness 控制都沿两个维度分类。第一：它是导引（前馈，在 agent 动作之前应用）还是传感器（反馈，在动作之后应用）？第二：它是计算式（确定性，毫秒级运行，如 linter）还是推断式（使用 LLM，秒级运行，如代码审查 agent）？

这给了你一个 2x2 的控制类型矩阵：

-   **计算式导引**（前馈）：类型系统、linter、架构决策记录
-   **计算式传感器**（反馈）：测试套件、覆盖率分析、变异测试、结构复杂度检查
-   **推断式导引**（前馈）：规范文档、设计 prompt、约束描述
-   **推断式传感器**（反馈）：基于 LLM 的代码审查器、语义质量评估器、行为验证器

这些控制随后分布在整个变更生命周期中：集成前（快速、廉价的检查）、集成后流水线（全面验证）、持续漂移检测（后台监控）和运行时反馈（SLO 告警、质量抽样）。

**它在哪里出彩。** 已有成熟代码库的现有团队。如果你已经拥有 linter、测试套件和 CI 流水线，ThoughtWorks 框架能帮你意识到：你其实已经有了半个 harness。这套分类法会告诉你还缺什么、下一步该往哪里投入。它还引入了一个有价值的概念：“可驾驭性（harnessability）”。强类型语言、清晰的模块边界和结构良好的框架，会让 agent 的工作天然更容易成功。如果你正在为新项目选择技术栈，这一点很重要。

他们还提出了 **harness 模板**：为常见服务拓扑准备的、标准化的导引与传感器套件。如果你 80% 的服务都是 CRUD API，那就为 CRUD API 构建一个 harness 模板并复用它。这是一个实用的洞见，能显著降低 harness 工程的单服务成本。

**它在哪里崩盘。** 它是描述性的，而非规定性的。这套框架告诉你存在哪些类型的控制，却不告诉你具体该用哪些工具、如何把它们连起来。这些决定你仍然得自己做。对于想要一站式解决方案的团队来说，这并不是它。它是一张蓝图，而非一座建筑。

行为治理这一类别也确实偏弱。验证 agent 的输出是否可维护、架构是否合理，现有工具已经服务得很好。但要验证它是否真的做了被要求的事？ThoughtWorks 坦诚地承认了这道鸿沟：当前的方法“对人工智能生成的测试寄予了过度的信任”，而这些测试在行为验证上“目前是不够格的”。

![Diagram by Author: ThoughtWorks 2×2 Control Framework](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/08.webp)

## 深入剖析揭示了什么

抛开实现上的差异，会浮现出一些令人惊讶的东西。三个从未协调过的团队，从不同的起始问题出发，最终却抵达了同样的五条原则。这种独立的趋同，通常意味着背后有某种真实的东西。

### 1. 上下文胜过指令

![Diagram by Author: Five Convergence Principles discovered independently by all three camps](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/09.webp)

OpenAI 学会了“给地图，而非手册”。Anthropic 构建了 JSON 功能列表和进度文件，让 agent 始终知道自己在哪。Red Hat 的整个工作流都建立在“先分析真实代码库、再生成任何任务”之上。ThoughtWorks 把它称为“前馈”。

标签不同，发现却完全一致：向 agent 展示世界的当前状态（真实的文件路径、真实的代码模式、真实的进度），始终胜过用抽象的措辞告诉它该做什么。一个扎根于你真实代码库的 agent，会产出契合的代码。一个从模糊描述出发的 agent，则会臆想文件路径、发明并不存在的 API。

### 2. 规划与执行必须分开

OpenAI 把环境设计（人）与代码生成（agent）分开。Anthropic 在 Generator 触碰任何代码之前，先跑一个专门的 Planner agent。ThoughtWorks 强制要求在规划和实施之间设置人工审核检查点。Red Hat 拆成第一阶段（影响图）和第二阶段（实施），两者之间有一道硬门槛。

每个阵营都独立发现：让 agent 在同一遍里既规划又执行，会产出不可靠的输出。规划这一步不必由人来做，甚至不必由一个独立的 agent 来做。但它必须是一个独立的步骤，其输出要在实施开始之前经过审核。

### 3. 反馈回路没有商量余地

OpenAI 把 agent 接入 CI/CD 流水线和可观测性系统。Anthropic 构建了一个专门的 Evaluator agent，用浏览器测试运行中的应用程序。ThoughtWorks 把这一点形式化为“传感器”，并警告说：只有前馈的方法（有导引而无验证）永远无法确认这些导引是否真的有效。

分歧不在于你是否需要反馈，而在于由谁来提供反馈。OpenAI 使用自动化测试和 CI。Anthropic 使用另一个 LLM。ThoughtWorks 说两者都用、分层进行：先计算式反馈（快、便宜、确定性），后推断式反馈（慢、昂贵、语义性）。三者都认同：一个没有反馈机制的 harness，只不过是一个多了几个步骤的 prompt。

### 4. 一次只做一件事

OpenAI 把目标分解成更小的构建块，并采用深度优先。Anthropic 强制每个冲刺只做一个功能，并在每个之后提交。ThoughtWorks 描述了一个分阶段的生命周期：集成前、集成后、持续监控。

试图一次做太多的 agent 会耗尽上下文、失去连贯，或者悄无声息地丢掉需求。强制增量式开发——agent 在开始下一个之前先完成一个工作单元——在每一个成功的 harness 实现中都普遍存在。Anthropic 使用的会话初始化例程（读进度、挑一个功能、实现、提交、重复）是对此最清晰的表达，但每个阵营都以各自的方式强制它。

### 5. 代码库就是文档

OpenAI 把 AGENT.md 文件嵌进仓库。Anthropic 把功能列表、进度文件和 git 历史作为 agent 的连续性机制存储起来。ThoughtWorks 衡量“可驾驭性”，即代码库本身对 agent 而言的可读程度。Red Hat 说把所有约定都搬进版本控制。

没有人为 agent 单独维护一个知识库。仓库是唯一的真相来源。如果某个约定、约束或架构决策没有写进代码库，agent 就无从知晓。这有一个实际含义：在代码组织、清晰的模块边界和嵌入式文档上投入的团队，能免费获得更好的 agent 性能。

## 这种趋同意味着什么

这五条原则并非观点。它们是三个独立团队通过构建、失败、迭代而发现的工程约束。如果你要从零设计一个 harness，就从这里入手。无论你选择何种工具、偏爱何种架构，违反这些原则都将让你付出代价。每个尝试过的团队都以惨痛的方式发现了这一点。

### 做对的代价

harness 工程并非免费。每种方法都涉及前期投入、单次运行成本和持续维护之间的权衡。以下是实际数据所揭示的，以及数据到头的地方。

### 我们已知的：Anthropic 的 A/B 测试

Anthropic 发布了目前最清晰的成本对比。他们让同一个应用程序提示分别跑过一个单 agent 和他们完整的多 agent harness。

单 agent（无 harness）：9 美元，20 分钟。输出有一个能用的 UI，但核心功能是坏的。实体对用户输入没有响应。它看起来像一个应用，但行为不像。

完整 harness（Opus 4.5）：200 美元，6 小时。输出是真正可玩的游戏，界面精美，视觉风格统一，物理效果正确。

这意味着，一个能正常运转的产品，相比一个只在截图里看起来对的演示，成本高出 22 倍。这究竟是贵还是便宜，完全取决于一次坏的发布会给你的团队造成多大的损失。

### 模型改进的红利

接下来就有意思了。当 Anthropic 从 Opus 4.5 迁移到 Opus 4.6 时，他们得以大幅简化 harness。冲刺分解被移除。单遍评估取代了逐冲刺评分。上下文重置被换成了自动压缩。

结果：一套完整的数字音频工作站应用程序，124.70 美元，3 小时 50 分钟。相比 Opus 4.5 的 harness，成本降低 38%、时间缩短 36%，完全由模型改进驱动。harness 做的工作更少，是因为模型需要的脚手架更少。

**这一模式并未放缓。4 月 16 日发布的 Opus 4.7** 把趋势线又往前延伸了一截。CursorBench 得分从 58% 跃升至 70%。Rakuten-SWE-Bench 上已解决的生产任务多了 3 倍。该模型在更少的 token 下，相比 Opus 4.6 提升了 14%——这意味着每单位有用输出的 harness 开销更低。三代模型，三轮 harness 简化。这是一种趋势，而非一桩轶事。

但它并未消除对 harness 的需求。Opus 4.6 的评估器仍然抓出了重大缺口：缺失的交互式时间线控件、缺席的乐器 UI 面板、不完整的音频录制。没有它，这些功能就会以残桩或损坏的状态发布出去。harness 随每一代而缩小，但它还没有消失。

### 隐性成本：维护

没人谈论的那个数字，是维护。harness 不是一次性构建，而是一项持续的工程承诺。

Manus 在六个月内把他们的 harness 重构了五次。LangChain 在一年内把他们的研究 agent 重构了三次。这些不是糟糕工程的迹象。它们是在快速改进的模型之上构建的自然后果。每当模型变得更好，你的 harness 里某一块就会变成不必要的开销，而要找出是哪一块，需要主动测试。

Philipp Schmid 的建议是：“为删除而构建。” 把每个 harness 组件都设计成可移除的。定期测试每个组件，方法是把它关掉、衡量输出质量是否变化。如果没变，就删掉它。背着失效的 harness 组件，会在每次运行中消耗 token，并增加维护负担，却毫无好处。

![Diagram by Author: The Cost of Getting It Right: $9 solo vs $200 harness vs $124.70 optimized](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/10.webp)

## 一套决策框架

与其为每个阵营规定一份成本画像，不如看看如何把你的处境匹配到一种方法：

**你是独立开发者或小团队，处于早期阶段。** 从在仓库里放 AGENT.md/CLAUDE.md 文件和一条扎实的 CI 流水线开始。这是 OpenAI 模式最简单的形态。成本低，立竿见影。你很可能已经拥有了大部分组件。

**你正在开发一款质量缺陷会被用户看到的产品。** 加上评估回路。你不需要 Anthropic 完整的三 agent 架构。哪怕只是一个简单的构建后审查步骤——用第二个模型检查第一个模型的工作——也能抓住测试漏掉的错误。“把执行者与评判者分开”这条原则可以向下缩放。

**你正在一个组织内让多个团队采用 agent。** 投入到 ThoughtWorks 分类法上。把你现有的控制（linter、测试、CI）映射到前馈/反馈、计算式/推断式的网格中。找出缺口。为你常见的服务类型构建 harness 模板。这是组织层面的基础设施，而非单个项目的决策。

**你身处受监管行业。** 把 harness 当作你的控制框架来对待，因为这正是审计人员最终会问到的东西。Anthropic 托管 agent 架构中那份仅追加的事件日志，不只是优秀的工程，它就是一条审计跟踪。Red Hat 方法中的结构化任务模板会生成可对应到合规要求的文档。现在就开始考虑这个，而不是等监管机构问起之后。

## 悖论：为删除而构建

Anthropic 的数据里藏着一个令人不安的真相，而这三大阵营都没有把它说得足够响亮。

当他们从 Opus 4.5 升级到 Opus 4.6 时，他们得到的不只是更好的结果，还是更简单的结果。冲刺分解——它曾对 Opus 4.5 在长时间编码会话中保持连贯至关重要——变得不再必要。模型改进后的规划和长上下文处理能力，使它变得多余。一个三月份还在承重的 harness 组件，到四月份就成了累赘。

随后 Opus 4.7 于 4 月 16 日落地，把这个模式又往前推了一步。该模型现在会在汇报前自我验证它的输出——这恰恰是当初要单设一个 Evaluator agent 去弥补的那道能力缺口。它产出更简洁的代码，更少的包装函数和不必要的脚手架。它生成的工具错误只有以往版本的三分之一。轨迹很清晰：4.5 需要完整的冲刺分解和逐冲刺评估。4.6 砍掉了冲刺分解，转向单遍评估。4.7 开始把评估本身内化。

Anthropic 把这称为“harness 衰减”。harness 中的每个组件，都编码了一个关于“模型做不到什么”的假设。随着模型改进，这些假设过期，原本用来补偿某项局限的组件，就变成了开销。

证据比比皆是。Manus 在六个月内把他们的 harness 重构了五次。LangChain 在一年内重构了三次。Vercel 移除了其 agent 80% 的工具，性能反而更好，而不是更差。每个案例都是同一个故事：上个月有帮助的，这个月就成了拖累。

![Diagram by Author: Build to Delete: Harness decay over model generations](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@9f0538a90add4213cdf6384f9c1b78efc82a5b40/ai-insights/2026-06/17/images/harness-engineering-what-every-ai-engineer-needs-to-know-in-2026/11.webp)

[Philipp Schmid](https://www.philschmid.de/agent-harness-2026) 把这与机器学习研究中 Rich Sutton 的“惨痛教训”联系起来：随算力扩展的简单方法，始终胜过复杂的手工设计方案。应用到 harness 上，含义很清楚。不要构建精巧、紧耦合的控制系统。构建模块化的系统，好让你能在模型成长到超出它们时，一块一块地删掉它们。

这给工程团队制造了一个真正的悖论。如今，你需要一个 harness 才能从人工智能 agent 那里得到可靠的输出。但你今天构建的 harness，明天就需要被部分拆除。而那些在模型已经成长到超出其 harness 架构之后仍死守不放的团队，将在每一次运行上都被征税：额外的 token、额外的延迟、额外的维护，零额外的质量。

实用的建议很直接，即便它听起来有违直觉：给每个 harness 组件都设计一个断路开关。定期测试每一个，方法是禁用它、衡量输出质量。当没有它质量依然稳定时，就删掉它。

更深的那个问题，至今无人回答。随着模型持续改进，harness 是会收敛成一个薄薄的、标准化的层——某种几乎不变的操作系统内核——还是会永远处于流变之中，随每一代模型从头重建？

三大阵营对答案并无共识。OpenAI 的环境优先方法暗示着收敛：代码库结构、CI 流水线和 AGENT.md 文件是稳定的基础设施，能在模型升级中持续存在。Anthropic 的数据暗示着流变：对 Opus 4.5 而言最优的多 agent 架构，对 Opus 4.6 来说已经太重，而 Opus 4.7 的自我验证能力，正让那个已经简化过的评估器看起来也时日无多。ThoughtWorks 的分类法刻意保持中立，被设计成无论该领域朝哪个方向移动都能存活。

清楚的是这一点：在 2026 年及以后构建出最可靠人工智能系统的工程师，不是那些写出最好代码的人。**他们是那些设计出最好约束的人**。**并且愿意在这些约束一旦不再值回票价的那一刻，就把它们扔掉。**

## 在你离开之前！🦸🏻‍♀️

如果你喜欢我的故事并想支持我：

1.  送上一些 Medium 的爱意 💕（点赞、评论和高亮），你们的支持对我意义非凡。👏
2.  在 Medium 上[关注我](https://medium.com/@yanli.liu/about)并订阅，即可获取我的最新文章🫶
