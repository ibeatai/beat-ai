---
title: 人工智能定价的秘密
author: Ignacio de Gregorio
url: https://medium.com/@ignacio.de.gregorio.noblejas/the-secret-behind-ai-pricing-most-people-ignore-22f009da22bd
translated: 2026-06-10
excerpt: 几周前，我写了一篇文章，解释了价格上涨的原因。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/01.thumb.webp
---

# 大多数人忽略的人工智能定价背后的秘密

![来源：作者使用人工智能](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/01.webp)

几周前，我写了一篇文章，解释了价格上涨的原因。

有些人不同意我的看法，**但仅仅几周时间就证明我是对的**，甚至比我预想的还要快。

然而，AI Labs 的做法却让大多数人摸不着头脑，我认为这是一种相当不正当的策略。他们应该在这方面更加透明，但他们并没有，**所以我今天的目标是改变这种状况**。

让我来解释一下 Claude 的创造者 Anthropic **为了向你收取更多费用而使用的伎俩**，方法是向你解释现代人工智能中最容易被误解的组成部分之一。

## 无人知晓的组件

很少有人知道，在你和你最喜欢的 AI 模型之间，存在着一个名为 **分词器** 的组件。

但要了解它是什么以及为什么它对您的整体成本如此重要，我们需要了解什么是 token。

### 什么是 token？

您可能已经注意到，生成式人工智能中的一切都是以“token”来衡量的。

*但这些东西究竟是什么呢？*

简而言之，**它是人工智能模型语义含义的基本单元**。像 ChatGPT 这样的系统就是通过这种方式将数据“分解”成它能够理解的组成部分。

在文本中，这意味着将其分解成单词。每个 token 都有其内在含义（单词本身就具有意义），而当每个 token 与其他 token 组合在一起时，其含义就会被赋予上下文意义。

> 例如，“bank”一词根据上下文可以指代多种事物，从金融机构到河流沿岸的斜坡。

需要注意的是，token 不一定是完整的单词；**它们可以是几个字母，甚至只是一个字符**。此外，不同的模型使用不同的 token 化方法。

例如，如下所示，GPT-4o 将上面的序列分解成一串 token，每个 token 都被分配一个数字。

![来源：Tiktokenizer](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/02.png)

这个数字是行 ID；它是嵌入表中的行号，该嵌入表将单词（例如“红色”）转换为一组数字，这些数字定义了“红色”对于模型的意义。

![来源：作者](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/03.webp)

*这些数字意味着什么？* 单独来看，意义不大。**但相对于其他 token 而言，它们意义重大**。

由于模型不与现实世界互动，**它们对现实世界概念的理解大多是相对的**，而非“隐性”的，这意味着“红色”并非通过“红色”的真实例子来衡量，**而是相对于模型已知的其他概念而言的“红色”。**

也就是说，分配给每个概念的这些数字串是与其他概念的相对语义距离。

也就是说，“狗”和“猫”都是哺乳动物，都是家养的四足毛茸茸的动物，它们的相似度会比“狗”和“鸽子”更高；而“狗”和“鸽子”都是动物，它们的相似度又会比“狗”和“门”更高。

![来源](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/04.webp)

通过下图可以更直观地理解每个数字的意义；它们代表全局属性或维度，我们可以使用这些属性或维度对概念进行分类，例如“甜点性”和“三明治性”，从而对甜点进行分类。

该模型可能并不完全理解沙瓦玛是什么，但它知道它更接近热狗而不是苹果馅饼。

![来源：谷歌](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/05.webp)

> 模型就是这样构建理解的；它们通过将事物与其他“已知已知”进行比较来理解事物是什么。

总体而言，生成式人工智能模型将数据视为一系列 token，这些 token 组合起来可以解释整体含义。

这可以应用于文本、图像（将其分解为图像块/像素组）、音频（通过频谱图）……等等。

![来源：作者](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/06.webp)

这样做的主要原因是**人工智能模型**，就像任何机器一样，**只能“理解”数字**。

因此，这种将世界数据分解成模型可以理解的块的 token 化过程，**是为了将无法处理的内容转化为模型“理解”的语义单元序列。**

尽管已经有很多研究致力于改变这种情况，但现实情况是，如今，这个分词器（将数据分解成 token 的组件）**位于实际模型之外。**

![来源：作者](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/07.webp)

*但这为什么如此重要呢？*

因为分词器不仅发挥着技术作用，**而且在很大程度上影响着模型的成本**。

### token 的大小和数量很重要

当你向 ChatGPT 发送“*今天伊朗战争的新闻是什么？*”时，这些词语会被分解成 token，然后才输入到模型中。

![来源：tiktokenizer 使用 GPT-4o 的分词器](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/08.webp)

该模型同时处理以上所有八个 token，并并行执行两个操作多次：

1. **注意力**。每个 token 都可以与它前面的 token“对话”。例如，“战争”可以与“伊朗”对话，从而确定我们讨论的是伊朗战争，而不是拿破仑战争。这是因为，正如前面提到的，意义是语境化的。虽然“战争”本身具有意义，但它需要从“伊朗”这个语境中获取信息，才能知道我们讨论的是哪场战争。
2. **多层感知器（MLP）**。它基于最早的神经网络（[可以追溯到 20 世纪 50 年代](https://bpb-us-e2.wpmucdn.com/websites.umass.edu/dist/a/27637/files/2016/03/rosenblatt-1957.pdf)）（是的，人工智能在某些领域并非“新生事物”），该模型能够根据自身知识为每个 token 添加信息。例如，它可以为“伊朗”添加其与美国长达数十年的宿怨信息，这些信息虽然序列中不存在，但对于理解问题的背景至关重要。

像 Claude 或 Gemini 这样的模型是这两种操作的结合，逐步积累足够的理解力，使模型能够知道“接下来会发生什么”。

![来源：作者](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/09.webp)

两者都是 token 级计算，这意味着它们所需的计算量与它们所输入的 token 数量成正比。

> 这引出了一个大多数人和企业在选择“x”或“y”模型时并未意识到的潜在问题：**token 数量与 token 价格同样重要。**

虽然大家都熟悉 token 价格（模型根据其处理和生成的 token 定价），但这掩盖了一个问题：**模型的行为和 token 化同样重要，甚至更重要**。

看看我们一直在讨论的例子。如果我们使用 Meta 的 Llama 3 分词器而不是 OpenAI 的分词器，**同样的文本序列会被分词成 10 个 token 而不是 8 个 token**。

![来源：tiktokenizer 使用 Llama 3 的分词器](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/10.webp)

这意味着，对于同一个模型，**如果输入 10 个 token 而不是 8 个，处理该序列的成本大约会增加 20%**。

更糟糕的是，由于分词成 10 的那个词具有更多压缩 token，因此生成完全相同的输出响应的成本也会更高。

> 原因正如我们之前解释的那样：**注意力机制和多层感知器（MLP）的操作都会随着 token 数量的增加而扩展**。具体来说，注意力机制的复杂度为 O(L²)，其中 L 是 token 的数量。
>
> 另一方面，MLP 的复杂度为 O(2\*L)，即 ~ O(L)，其中“L”是 token 的数量。

> 简单来说，这两个操作所需的计算量都会随着 token 数量的增加而增加。

![来源：作者](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/11.webp)

而这正是 Anthropic 使用卑鄙手段的地方。

> 我的简报以通俗易懂的方式，从基本原理出发，讲解人工智能，适合那些不喜夸张宣传但渴望学习知识的人。立即加入！

## Anthropic 的新分词器

问题的根源在于 Anthropic 的新模型 **Opus 4.7** 的 token 数量增加了高达 **35%**，仅仅是因为它将分词器从 Opus 4.6 更换了。

> 这很可能是因为 Opus 4.7 是 Mythos 的蒸馏，是一个全新的模型，但这又是另一个故事了。

[虽然 Anthropic 确实提到了这一点](https://www.anthropic.com/news/claude-opus-4-7)，但由于显而易见的原因，他们“未能”提及这可能意味着成本增加。

出于其他显而易见的原因，[***人们已经开始对此进行测试***](https://openrouter.ai/announcements/opus-47-tokenizer-analysis?utm_source=thewhitebox.beehiiv.com&utm_medium=newsletter&utm_campaign=playing-dirty-the-mythos-lie-prices&_bhlid=632e841da053ffcffb383880ccd69065687a8914)，不出所料，**该模型成本更高**，相对于 Opus 4.6 而言，成本高出 27%，**尽管它们的 token 定价相同（每百万个输入和输出 token 分别为 5 美元和 25 美元）**。

[***Ramp 也进行了类似的分析，并得出了相同的结论***](https://x.com/rahulgs/status/2048830174524063752?utm_source=thewhitebox.beehiiv.com&utm_medium=newsletter&utm_campaign=playing-dirty-the-mythos-lie-prices&_bhlid=2acf01a92848db33fd0eb22372fc73f655fc383e)；**尽管 GPT-5.5 的输出 token 价格更高（30 美元对 25 美元）**，但 Opus 4.7 总体上更贵。

![来源：Ramp](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c119b68eae18308f1482f41e1909d25bd11cbe1a/ai-insights/2026-06/10/images/the-secret-behind-ai-pricing-most-people-ignore/12.jpg)

就这样，仅仅因为一个简单的改动（大多数客户都不够专业，无法理解，因为，嗯，除了像我这样的分析师，没有人应该被迫理解分词器才能使用人工智能模型），**仅仅因为一个模型改动，你的账单就上涨了 12% 到 27%**，而这些改进微乎其微，无论如何都无法证明涨价的合理性。

## 阴险的手段扼杀了善意

这一切都是不可避免的。**人工智能的成本远比我们之前认为的要高得多**，而且补贴终究会结束。

而那一天，就是现在。

**这些公司资产负债表上的漏洞巨大**，高达数千亿美元，而其使用寿命却比传统 IT 资产短得多。

*收入？* 最好的情况是，**所有相关公司每年收入 1000 亿美元**（当然，不包括超大规模云厂商的收入，因为这些收入大多是循环的，实际上并没有真正的资金交易）。

> 需要明确的是，收入增长令人印象深刻，但人工智能建设支出增长仍然远远超过收入增长，至少从名义上看是这样。

这些公司，尤其是像 Anthropic 和 OpenAI 这样今年上市的公司，无论如何都必须赚钱。

我明白这一点，**但我看到的这些不正当手段最终可能会自食其果**；从长远来看，这不是留住客户的方法，而且我看到这些公司几个月前还拥有的铁杆粉丝们的态度明显发生了“变化”。

我不会责怪任何人想要赚更多钱；**但我当然可以对你选择如何赚钱提出我的意见。**

我越了解这些，就越确信企业不会接受这些，**一旦他们足够成熟，就会开始大规模地迁移到开源软件**（实际上，开源的门槛比人们想象的要低得多）。

你可以突然宣布价格上涨近 30%，而如今的公司却可以表示 *“谢谢，但我们不接受”*，然后转向开源。

鉴于以上种种原因，我担心我们目前看到的大部分收入增长（主要来自企业）可能是短暂的，因为在我看来，**私有模型只有在像编码或基于 agent 的系统这样的迭代工作流程中才有意义**，在这些工作流程中，即使是智能方面的微小改进也至关重要。

*这是一个万亿美元的市场吗？* 但愿如此，否则所有这些公司花费数十亿美元，最终却只能让企业说：“是啊，谢谢，但我们选择使用开源解决方案，正是因为你们才有了它，而且它比你们的便宜得多。”

因为认为前沿人工智能（在我看来，**其价格将在未来几年内持续上涨**）可以作为自动化工具来盈利地阅读你的电子邮件，就像我祖父常说的 *“用炮弹打苍蝇”* 一样，简直是疯了。
