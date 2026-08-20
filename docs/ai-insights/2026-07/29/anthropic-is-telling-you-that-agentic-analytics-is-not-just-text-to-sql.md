---
title: Anthropic 指出，Agentic Analytics 不仅仅是 text-to-SQL
author: Jose Parreño
url: https://medium.com/data-science-collective/anthropic-is-telling-you-that-agentic-analytics-is-not-just-text-to-sql-ce605454bbc9
translated: 2026-07-29
excerpt: Anthropic 的 Agentic Analytics 套件使用 Claude 实现了 95% 的准确率，他们在最新文章中分享了其实现方法。值得信赖的分析关键在于语义层、技能、评估和数据工程。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6d72d81c0b7bd35d5b340384743c6e64c7eb9dea/ai-insights/2026-07/29/images/anthropic-is-telling-you-that-agentic-analytics-is-not-just-text-to-sql/01.thumb.webp
---

# Anthropic 指出，Agentic Analytics 不仅仅是 text-to-SQL

Anthropic 的 Agentic Analytics 套件使用 Claude 实现了 95% 的准确率，他们在最新文章中分享了其实现方法。值得信赖的分析关键在于语义层、技能、评估和数据工程。

[Anthropic 于 2026 年 6 月 3 日发布的关于自助式分析的文章](https://claude.com/blog/how-anthropic-enables-self-service-data-analytics-with-claude)，我已经分享给许多利益相关者和数据科学团队，因为终于有人白纸黑字写下来：Agentic Analytics ***并非***只是把 Claude Code 接到你的数据库上然后向它提问。

你可能听说过 [AI slop](https://en.wikipedia.org/wiki/AI_slop) 这个词，但还没遇到过 Analytics slop 吧？如果没有，我来解释一下。有了 Claude Code，任何人现在都可以让 Claude 连接数据库并生成分析报告。

一方面，这很棒，因为数据分析的门槛降低了。另一方面……嗯，另一方面，***根本没人***在核对生成出来的数字。最近发给我 .html 仪表盘、Confluence 页面或终端截图的人，没有一个核对过数字是否正确。

而这一切都源于一种虚假的安全感：因为 LLM 很擅长把英语翻译成某种语言（比如分析用的 SQL），所以答案就一定是对的。

我把话挑明：这远非事实。如果你把 Agentic Analytics 简化为查询生成，那就过早地缩小了问题的范围，忽略了那些决定答案是有用还是危险的基础设施。**Anthropic 自己的架构和技术栈，把数据基础、事实来源、技能和验证** ***摆在*** **Claude 真正接触查询的那一刻之前。**

我认为 Anthropic 这篇文章真正的闪光点就在最后这一部分。他们明确指出老派的数据工程实践是必备项，还总结了一些关键教训，让我们其他人不必重蹈覆辙。

如果你对 Agentic Analytics 感兴趣，我会为你拆解 Anthropic 的这篇文章。

## 在哪里可以找到这篇博客文章？

[Anthropic 如何借助 Claude 实现自助式数据分析](https://claude.com/blog/how-anthropic-enables-self-service-data-analytics-with-claude)

这是 Anthropic 于 2026 年 6 月 3 日发布在企业 AI 类别下的一篇博客文章。它不是研究论文，但在我见过的生产实践总结里，它把「可信赖的 Agentic Analytics 在实践中到底需要什么」讲得最清楚。

## 本文将涵盖哪些内容？

- **为什么这么多团队把 Agentic Analytics 简化成 text-to-SQL？** — 为什么这个品类总是被框定在查询生成上，以及为什么这样会出问题。
- **为什么数据歧义才是分析 agent 的真正问题？** — 为什么分析在语法出错之前就已经失败，以及为什么业务含义主导了风险面。
- **Anthropic 究竟为自助式分析构建了什么？** — 95% 准确率背后的架构：数据基础、事实来源、技能和验证。
- **Agentic Analytics 技术栈在写 SQL 之前需要什么？** — 在 agent 开始探索之前就压缩歧义的数据工程层。
- **为什么语义层是分析 agent 的地图？** — 为什么受治理的定义、粒度、连接和数据源优先级，比再调一次 prompt 更能提升质量。
- **为什么技能会改变准确率的走向？** — 为什么程序性知识如此重要，以及为什么 Anthropic 最大的准确率提升来自这里。
- **为什么评估和维护与 prompt 一样重要？** — 为什么漂移、溯源、离线测试和纠错循环都是产品的一部分。
- **数据团队如果现在就想上 Agentic Analytics，该怎么做？** — 给那些想推进、又不想第一天就盖一座大教堂的团队的实用起步顺序。

*别忘了订阅，及时了解最新发布的文章！*

我们开始吧！

## 为什么这么多团队把 Agentic Analytics 简化成 text-to-SQL？

团队之所以把 Agentic Analytics 简化成 text-to-SQL，是因为查询生成是那个 30 秒就能演示、就能试一把的部分。

→ 用平实的英语问一个问题。

→ 看着模型生成查询。

→ 拿它到数仓上跑一遍。

→ 返回一个数字或一张图表。

这种交互看起来很神奇，有那么一刻，你会觉得分析层的问题已经被解决了。想想看，非技术人员只要用英语问一句就能拿到答案，那是什么感受？更何况过去他们还得依赖数据分析团队或事先做好的仪表盘。

然而，尽管感觉神奇，Claude 或 Codex 输出的数字和听上去合理的解释，都无法证明业务问题被正确回答了。Anthropic 直接点破了这个陷阱：把 Claude 指向数仓，会制造出一种[虚假的精确感](https://claude.com/blog/how-anthropic-enables-self-service-data-analytics-with-claude)。

我举个具体的例子。假设一位利益相关者问：

*“上个月活跃客户收入是多少？”*

分析上的细微之处就是从这里开始的：

- 什么才算*活跃*？
- 哪个*客户*定义才是官方的？
- 是*毛收入*还是*净收入*？
- 用哪个*日期字段*？
- 哪些退款、欺诈过滤或产品排除规则适用？

如你所见，text-to-SQL 是在这些选择***之后***才发生的。因此，要让 text-to-SQL agent 跑得可靠，唯一的办法是围绕它们构建一整套基础设施，引导它们理解你的业务（后面会详细展开）。

## 为什么数据歧义才是分析 agent 的真正问题？

这类问题我已经回答很久了。事实上，它正是[谷歌那篇关于机器学习系统隐藏技术债的传奇论文](https://proceedings.neurips.cc/paper/2015/file/86df7dcfd896fcaf2674f757a2463eba-Paper.pdf)的主要立足点：那篇论文描述了机器学习模型只是生产环境中整个 ML 系统里非常小的一部分。

![谷歌论文截图](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6d72d81c0b7bd35d5b340384743c6e64c7eb9dea/ai-insights/2026-07/29/images/anthropic-is-telling-you-that-agentic-analytics-is-not-just-text-to-sql/01.webp)

我们可以把这条思路套到分析 agent 的数据歧义上，把那个小黑盒换成 text-to-SQL。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6d72d81c0b7bd35d5b340384743c6e64c7eb9dea/ai-insights/2026-07/29/images/anthropic-is-telling-you-that-agentic-analytics-is-not-just-text-to-sql/02.webp)

Anthropic 对这个问题的回答也非常契合：

> *“数据不是软件”*

分析工作的失败方式和软件功能不同。例如：

- 通常只有 1 个正确答案、来自 1 个正确的来源，但仅凭最终输出没有确定性的办法证明这个答案是对的。
- 系统可以写出完美的 SQL，却完全没抓住业务含义。

Anthropic 给出了一组更详细的例子来对比这两种范式，我觉得很有用。

![表格取自 Anthropic 的博客](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6d72d81c0b7bd35d5b340384743c6e64c7eb9dea/ai-insights/2026-07/29/images/anthropic-is-telling-you-that-agentic-analytics-is-not-just-text-to-sql/03.webp)

我也整理了一些，用另一组问题表达同样的想法。

### 分析的 3 种风险或失效模式是什么？

这 3 种模式的分类来自 Anthropic 的博客文章，他们是这样区分的：

1. **概念/实体歧义。** 用户说“活跃用户”“收入”或“留存”，agent 必须在多个看似合理的实现之间做选择。这些选择往往足够微妙，以至于错误答案看起来依然很专业。
2. **数据陈旧。** 表、业务规则和文档都会变。三周前还有效的指标，在一次 schema 变更、一条新的排除规则或一次仪表盘重写之后，会悄无声息地变错。
3. **检索失败。** 正确答案就在技术栈的某个地方，但搜索空间太大。模型错过了正确的表、指标或文档，然后自信地在一个差之毫厘的结果上继续往下做。

光有数仓是不够的。数仓存的是数据，它不会自动存下业务的完整含义。那些含义分散在指标定义、语义模型、注意事项、人为惯例、归属和历史背景之中。

再想想前面那个简单的问题：*“上个月活跃客户收入是多少？”* 难的不是写出一个 `GROUP BY`，难的是判断“活跃”到底指过去 30 天内有过一次购买、计费周期内非欺诈的付费使用，还是语义层里定义、别处都找不到的某个领域专属阈值。

因此，真正有意思的问题不是 Claude 能不能写 SQL（它大概比你我都写得好），而是 Anthropic 必须在 Claude 周围构建些什么，才能让这些问题不再主导结果。

## Anthropic 究竟为自助式分析构建了什么？

答案是，他们把老派的成熟工程实践，和 agent 能做的那些额外层次混在了一起。但归根结底，这仍然是围绕 Claude 搭了一整套系统来支撑分析（或者反过来说，把 Claude 放在最前面：Claude 是接口层，系统活在它下面）。

[这篇文章](https://claude.com/blog/how-anthropic-enables-self-service-data-analytics-with-claude)把这套系统分成 4 个部分。

1. 首先是**数据基础**：规范数据集、数据模型、转换、测试、新鲜度检查和元数据。
2. 然后是**事实来源**：语义层、血缘、业务背景，以及帮助 agent 把问题映射到受治理含义上的精选参考资料。
3. 接着是**技能**：告诉模型该怎么干活的领域流程，而不只是有哪些数据。
4. 最后是**验证**：离线评估、在线检查、溯源和维护循环。

![来自 Anthropic 博客文章的截图](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6d72d81c0b7bd35d5b340384743c6e64c7eb9dea/ai-insights/2026-07/29/images/anthropic-is-telling-you-that-agentic-analytics-is-not-just-text-to-sql/04.webp)

注意 SQL 在这个序列里的位置：它在所有难做的决策的下游。等模型开始写查询时，整套技术栈应该已经收窄了问题的含义，并确定了哪条受治理的路径值得信任。

比起模型选择，这套技术栈更能解释那个头条结果。快速扫一眼 95% 的准确率，它听起来像模型跑分；仔细读，它是系统跑分。他们的文章花在 Claude 周边那套支撑装置上的篇幅，远多于 Claude 本身。

带着这一点，本文接下来会更详细地介绍这套技术栈的具体组成。

## Agentic Analytics 技术栈在写 SQL 之前需要什么？

Anthropic 明确指出，标准的数据工程依然适用：维度建模、新鲜度与完整性检查、可直接消费的规范数据集、清晰的数据字典等等。这些并不会因为界面变成对话式就消失。

真要说的话，因为现在的消费方是一个 agent（经由人的提问），风险反而更高了。人类分析师有时能嗅出一张表不对劲，agent 却能把一个错误假设放大得快得多得多。

读到前沿公司公开告诉全世界，老派的数据工程实践对 agent 的表现至关重要，实在有点“好笑”。我觉得下面这些点描述起来有点贬低人的意思，因为它们并不是什么新鲜玩意儿——随便找一本 20 年前的数据仓库管理书籍，都能翻到同样的答案。不过无论如何，重温一下总没坏处。

### 规范数据集

这些数据集在检索开始之前就减少了看似合理的答案数量。逻辑模型更少、治理更严，这一小组规范的、单一事实来源的数据集就成了任何分析技术栈的基础。

你可能需要不同层级的聚合，但这些上卷聚合始终从这一小组单一事实来源读取数据。分析 agent 也一样：如果它们能拿到这些更小的事实集合，以及它们如何映射到各种丰富维度，那么选错数据源的概率会大幅下降。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6d72d81c0b7bd35d5b340384743c6e64c7eb9dea/ai-insights/2026-07/29/images/anthropic-is-telling-you-that-agentic-analytics-is-not-just-text-to-sql/05.webp)

### 把元数据当成一等公民产品

元数据给了 agent 一份可用的描述：表、列、粒度、负责人和注意事项。如果你不花时间把数据模型、每一列的含义、血缘（或连接键）索引描述清楚，LLM 就会做出大量假设。

### 把模型、文档、仪表盘和技能文件放在一起

这点挺有意思，因为“共置”完全可以理解成用一个更大的单体仓库，或者让大量制品在系统里彼此靠近地存放。Anthropic 的表述定义得很好：*“几乎所有数据代码（即建模、语义层、参考文档、规范的仪表盘定义）都在同一个仓库里，并由 CI 检查保护跨层完整性。如果一次建模改动会破坏下游仪表盘或让某个已记录的指标失效，CI 会标记出来，修复也在同一个 PR 里发布。”*

如你所见，我不认同“AI 分析会让数据工程变得不那么重要”这种偷懒的叙事。随着越来越多的问题经由模型来回答，建模层的质量只会更显眼，而不是更不重要。

这个模式在 Anthropic 之外也能看到。[dbt Semantic Layer](https://docs.getdbt.com/docs/use-dbt-semantic-layer/dbt-sl) 的存在就是为了集中管理指标定义，让它们能跨工具复用。微软的 [Power BI Copilot 指南](https://learn.microsoft.com/en-us/power-bi/create-reports/copilot-prepare-data-ai)建议采用 AI 就绪的 schema、经过验证的答案和明确的指令，好让模型在回答之前就有落地的上下文。就连 [Snowflake 的 Cortex Analyst 架构](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst)，也在 SQL 生成层周围依赖语义模型、经过验证的查询和受监控的行为。产品不同，教训相同：准备好的上下文本身就是答案的一部分。

## 为什么语义层是分析 agent 的地图？

**首先，在数据这个语境里，语义层是什么？**

语义层是一座架构上的桥，把原始的、复杂的数据翻译成熟悉的业务术语、指标和关系。它位于你的**数据存储**（Snowflake、Databricks、BigQuery 这类数据仓库）和**消费工具**（Power BI/Tableau 这类 BI 平台，或者 AI agent）之间。

有了语义层，你才能回答这类问题：列 X 是什么意思、列 Y 是什么数据类型、指标 Z 怎么算、特征之间是否存在层级、表与表之间是什么关系。

![图片来源。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6d72d81c0b7bd35d5b340384743c6e64c7eb9dea/ai-insights/2026-07/29/images/anthropic-is-telling-you-that-agentic-analytics-is-not-just-text-to-sql/06.webp)

它重要到什么程度？[Anthropic 说，它的 agent 在结构上被***要求***先查阅语义层](https://claude.com/blog/how-anthropic-enables-self-service-data-analytics-with-claude)。这条强硬的运作原则是：模型从受治理的含义出发，而不是从对原始表的自由探索出发。

Anthropic 不只是强制 agent 使用语义层，他们还明确指出：让 agent 自己去生成语义层不是好实践！等于说，一家世界前沿实验室在告诉你，人才是保证语义层质量的关键。

而且 Anthropic 并不孤单，市场正在收敛到同一个结论。Snowflake 在 [Cortex Analyst 里的语义模型层](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst)连接了业务问题和数据库结构。Databricks 的 [Genie](https://docs.databricks.com/aws/en/genie) 要求领域专家提供数据集、指令、示例 SQL 和业务背景。dbt 的语义层集中管理指标逻辑，让下游工具不必重复发明。微软把经过验证的答案和 AI 指令推进了 BI 模型本身。在之前的一篇博文里，我讲过谷歌的数据科学 agent 需要一个伪语义层（他们称之为“Analyzer”）才拿到突破性的表现。

因此，没有这张地图，agent 只能靠猜。有了它，模型的力气会花在对一个受约束的世界做推理上，而不是从零发明这个世界。

### 示例查询也能增强语义层

在数据库语义层之上，Anthropic 分享说，像查询语料库这样的知识也能带来正向的净收益。查询语料库可以是来自仪表盘、notebook 和过往分析的历史 SQL。真正有效的做法，是把这些语料提炼成结构化的、按领域划分的参考文档，以及写进**技能**里的可复用分析模式*（见下一节）*。把查询历史当成待整理的原材料，而不是 agent 直接读取的事实来源。

## 为什么技能会改变准确率的走向？

有了语义层，agent 知道“世界是什么意思”……但它还是不知道该怎么在这个世界里操作。比如，当利益相关者提出一个含糊的问题、存在两条可能的数据源路径、而且某个地区还有已知的边界情况时，语义层没法告诉 agent 一个好分析师应该按什么顺序走。

在 Anthropic 的分析系统里，技能与其说是文档，不如说是编码过的分析师操作手册。它们承载的是*程序性*知识：

- 先查哪个来源
- 什么时候用语义层、什么时候回退
- 该问哪些澄清性问题
- 这个领域里哪些注意事项重要
- 如何用对抗的方式复核结果
- 最终答案里要包含哪些溯源信息

Anthropic 最有力的数字出现在这里。根据[文章的技能一节](https://claude.com/blog/how-anthropic-enables-self-service-data-analytics-with-claude)，没有技能时，Claude 在他们的分析评估上不超过 21%。加上技能层之后，整体准确率超过 95%，在某些领域达到大约 99%。

正是这个数字，逼着我们改变对这个品类的描述。如果最大的性能跃升来自流程性指导，而不是更好的 SQL 生成，那么这个系统本质上就不是一个 text-to-SQL 产品，而是一个内部包含查询执行的分析工作流产品。

下面是他们使用的技能模板示例。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6d72d81c0b7bd35d5b340384743c6e64c7eb9dea/ai-insights/2026-07/29/images/anthropic-is-telling-you-that-agentic-analytics-is-not-just-text-to-sql/07.webp)

如果把上面提到的这些制品组合起来，你得到的大概是这样的东西：

## 为什么评估和维护与 prompt 一样重要？

*因为哪怕是根基扎实的分析 agent，也会衰减。*

一旦你有了语义层、精选示例和技能，下一个失效模式就是漂移。表会变，业务定义会变，仪表盘被重建，技能不再匹配源模型，等等等等。昨天正确的流程，明天就成了过时的上下文。

Anthropic 给了一个很刺眼的例子：在他们把技能维护当成工程问题来对待之前，[离线准确率在一个月里从大约 95% 掉到了大约 65%](https://claude.com/blog/how-anthropic-enables-self-service-data-analytics-with-claude)。这个数字把维护从“锦上添花”变成了可靠性的硬要求。

分析的好处在于，你可以把评估锚定在任何流程都应该算对的标准答案数字上。如果能在系统里把这个定义下来，你就可以去折腾构成这套系统的所有参数。比如：拿技能 v2 分别配 Haiku、Sonnet 或 Opus，再配不同的思考档位，很容易就能对比它们的准确率，然后挑一个最适合你场景的（成本更低而准确率够用，或者不计代价追求最高准确率）。

成熟的分析系统就该这么测：有存下来的用例、明确的断言，以及在系统失败时更新操作手册的反馈回路。

Databricks 在它的 [Genie 文档](https://docs.databricks.com/aws/en/genie)里也透出类似的直觉：响应检查、基准、可信资产和精选指令都被做进了产品体验里。实现方式不同，结论相同。

也正是在这里，prompt 开始显得没有大家预期的那么核心。prompt 质量当然仍然重要。但如果你的语义层薄弱、示例陈旧、技能漂移、评估缺席，再神的 prompt 也撑不了多久。

Agentic Analytics 之所以能跑通，是因为周边系统像一个被持续维护的产品那样运转，而不是因为模型被允许拿着原始访问权限即兴发挥。

## 数据团队如果现在就想上 Agentic Analytics，该怎么做？

正确的起点是能跑通的最简单结构，然后只在数据表明确实有帮助的地方，才有节制地往上加复杂度（Anthropic 的[构建高效 agent](https://www.anthropic.com/engineering/building-effective-agents)讲的正是这个逻辑）。他们的分析文章在实践中也遵循同样的模式。

5 个具体步骤：

1. **选一个领域，而不是整个数仓。** 收入质量、支持运营、增长漏斗、财务报表——挑一个有明确负责人、且问题反复出现的领域。
2. **建立一个受治理的事实来源。** 在动 prompt 之前，先确保这个领域有规范数据集、明确的指标定义、归属和基本的新鲜度检查。
3. **把语义层放在第一位，把裸 SQL 当成回退。** 如果问题能干净地映射到受治理的含义上，agent 就不该在原始表里自由发挥。
4. **写一份薄薄的技能或操作手册。** 把一个好分析师在这个领域会走的流程编码下来：澄清、找源、验证、汇报、标注溯源。
5. **在大规模推广之前，先做一个小评估集。** 二三十个带预期答案和已知边界情况的真实问题，比一百场内部演示更能教会你东西。

我不会做的，是一上来就把前沿模型直接接到数仓上，然后把这个结果叫做战略。那种做法除了让你见识到一个错误答案能有多么令人信服之外，教不了你太多东西。

text-to-SQL 在这套技术栈里仍然有位置，它是一项有用的执行能力，只是这个概念太小，撑不起整个系统。

## 现在，我想听听你的想法

我认为这个话题会把团队分成 2 派：一派把 text-to-SQL 看作产品本身，另一派把它看作更宽的分析工作流里的一个小层。

- 到目前为止，你见过的最大失败出在哪里：歧义、检索、陈旧，还是流程薄弱？
- 你的团队已经具备分析 agent 所需要的基础了吗，还是说这个项目会暴露出一大堆隐藏的数据模型债？
- 如果你正在这个方向上做东西，你今天的“技能层”里装的是什么：prompt、运行手册、SQL 示例、分析师检查清单，还是别的？

在评论区分享你的想法、案例或反对意见吧。我很想读到你的观点 👇

## 延伸阅读

感谢阅读！如果你对我写的其他内容感兴趣，这里有一篇文章按主题汇总了我全部的博客文章：数据科学团队与项目管理、数据故事讲述、营销与竞价科学，以及机器学习与建模。

## 敬请关注！

如果你想在我发布新内容时收到通知，欢迎在 Medium 上关注我。另外，[我非常乐意在 LinkedIn 上聊聊](https://medium.com/data-science-collective/www.linkedin.com/in/joseparrenogarcia)！

## 参考

- Anthropic 如何借助 Claude 实现自助式数据分析 — Anthropic 于 2026 年 6 月 3 日发表的文章，描述了其分析架构、失效模式、技能层和验证技术栈。
- Cortex Analyst — Snowflake 官方文档，讲对结构化数据的对话式分析，包括语义模型、验证过的查询和监控。
- Building effective agents — Anthropic 更宽泛的指南，介绍工作流设计、工具使用，以及什么时候增加 agent 复杂度是合理的。
- dbt Semantic Layer — dbt 文档，介绍集中式指标定义和跨工具复用的语义逻辑。
- 为 AI 准备数据以改进 Copilot 结果 — 微软 Power BI 指南，介绍 AI 就绪的 schema、经过验证的答案，以及让分析回答有据可依的指令。
- 什么是 Genie Space — Databricks 文档，介绍如何用数据集、指令、示例 SQL、可信资产和检查工作流来配置 Genie。
- 使用技能扩展 Claude — Anthropic 的技能文档，解释可复用的流程指令如何按需加载。
- 机器学习系统中的隐藏技术债（谷歌）— 为什么机器学习模型只是生产环境 ML 技术栈中很小的一部分。

## 附录

Anthropic 的完整技能模板。

```yaml
---
name: [warehouse-skill]
version: [x.y.z]
description: "IF the user asks to query [the company]'s data warehouse for any
  [list of business domains] question — THEN invoke this skill. DO NOT invoke
  for [adjacent engineering tasks] or questions with no data-warehouse component."
---# [Warehouse] Skill Instructions## Description
The single source of truth for safe and effective [warehouse] querying.
Referenced by other skills [listed] for query execution guidance.Act as a Data Analyst, providing strategic insights and data-driven
recommendations but seek guidance along the way.**Out-of-scope decisions**: [product areas, etc.] → surface data only,
state "decision is [owning team]'s call", do NOT take a position or author
code fixes.## Executing queries
Priority:
1. **[Managed connection]** (if available): [query tool] / [schema tool]
2. **[CLI fallback]** (if installed): [default project, fallback project]
3. **Neither** — ask the user to authenticate, then stop---# Semantic Layer (REQUIRED first step)The governed semantic layer is the **mandatory default path** for every data
question — same numbers as [the BI tool], joins/grain/filters baked in. Raw SQL
via the reference docs below is the **fallback**, used only after the
semantic-layer path is shown not to cover the ask.## Required workflow
1. **Load** — [how to load the semantic layer in each runtime, with fallbacks]
2. **Discover** — search measures/dimensions by keyword; **always check
   segments** (the named canonical population filters — hand-rolled WHERE
   clauses for these are the dominant wrong-answer mode)
3. **Compile + run** — build the spec → compile to SQL → execute
4. **Fallback** — only if discovery finds no relevant metric or compile fails
   → raw SQL via `references/*.md` (PART 3 below)> **Don't bail early.** Do NOT fall back to raw SQL on these grounds:
> - "[custom date filtering / cohorts]" → [covered by time-dimension specs]
> - "[needs a join]" → [the metric layer already encapsulates its joins]
> - [3–4 more pre-rebutted excuses agents use to skip the semantic layer]### Date windows & timezone — decide before you query
- **As-of date vs trailing-N days**: [convention for each]
- **"Last week/month"** → the last *complete* calendar week/month, not trailing-7/30
- **Timezone default**: [TZ]; [exception for certain reporting rollups]
- **Freshness lag**: [some] tables settle late — anchor on MAX(date), not "yesterday"---# PART 1: MUST KNOW (Read First for Every Request)## 🚀 Quick Start Workflow
1. **Check for red flags first**: [restricted/PII requests, gated domains,
   high-stakes asks that need extra validation]
2. **Out of scope — escalate, don't guess**: [access requests, pipeline
   troubleshooting, stale dashboards, root-cause assertions, product/pricing
   recommendations] → redirect to [the owning team], don't answer
3. **Clarify the request**: time period, segment, the business decision it informs
4. **Check for existing dashboards**: [per-domain dashboard catalogs]
5. **Identify the data source**: [navigation map below; prefer governed/aggregated tables]
6. **Execute the analysis**: [required filters + adversarial review]
7. **Deliver insights**: show methodology, differentiate observations from interpretations## 🏢 Business Context### Entity Disambiguation (MUST CLARIFY)
- **"[Term A]" can mean**: [entity 1] or [entity 2] — always clarify which
- **"[Term B]" can mean**: [entity 1] → [entity 2] → [entity 3] (one-to-many chain)
- **"Users"**: [which identifier gives accurate counts, and which ones inflate them]### Business Terminology
- [Current product names vs deprecated aliases that still appear as frozen
  values in the data layer — write with the new names, filter with the old]
- [Key internal acronyms]
- **[Headline metric] calculations**: [monthly / default window / leading indicator]
- **Unfamiliar terms — search [internal docs], don't guess**### Data Integrity Requirements ⚠️
- **NEVER**: make up data/columns; make speculative assertions beyond what data shows
- **ALWAYS**: use safe division; differentiate observations ("data shows X")
  from interpretations ("this suggests Y"); flag limitations---# PART 2: HOW TO DO (Follow During Execution)## 🔧 Technical Execution Guide
- [Managed-connection tools and CLI invocation details]
- **PII protection**: for restricted data, return the SQL for the user to run
  themselves — do not return results## 📊 Analysis Best Practices Guide
1. Clarify the ask before querying
2. Show your work (filters, inclusions/exclusions, freshness)
3. Clarify denominators
4. Consider sample bias
5. Connect to business impact
6. **Adversarial SQL review (MANDATORY)** — spawn the [sql-reviewer] sub-agent
   for every query before the final answer; blocking findings must be fixed
   and re-reviewed; do not self-certify
7. **Report with provenance** — every answer ends with a footer:
   > **Source:** [semantic layer | governed table | raw exploration] ·
   > **Confidence:** [tier] · **Reviewed:** [reviewer ✓, round N] ·
   > **Freshness:** [max date in the data] · **Owner:** [owning team]---# PART 3: DATA REFERENCES & RESOURCES## 📚 Knowledge Base Navigation
### [Domain A] → `references/[domain_a].md`
- **Use for**: [kinds of questions]
- **Key tables**: [...]
- **Dashboards**: `references/[domain_a]_dashboards.json`### [Domain B] → `references/[domain_b].md`
- **Use for**: [...][... one entry per business domain — a few dozen in total ...]## ⚠️ Troubleshooting Guide### When Information Is Missing
- [missing tables / access denied / outdated docs / unknown enum values → what to do]### Field Naming Gotchas
- Use `[field_x_v2]` NOT `[field_x]`
- [Two similarly-named tables report the same metric at different grains — which to use]
- [Which of two plausible sources is canonical for the headline metric]
- [… a dozen more hard-won one-liners …]
```
