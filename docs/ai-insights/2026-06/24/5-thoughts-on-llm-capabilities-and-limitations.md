---
title: 关于 LLM 能力和局限性的 5 个思考
author: Clement Piat
url: https://pub.towardsai.net/5-thoughts-on-llm-capabilities-and-limitations-eaa57176bb57
translated: 2026-06-24
excerpt: 关于 LLM 的根本局限，存在不同的观点。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/5-thoughts-on-llm-capabilities-and-limitations/01.thumb.webp
---

# 关于 LLM 能力和局限性的 5 个思考

关于 LLM 的根本局限，存在不同的观点。

一方面，有人认为，基于下一个 token 预测 (NTP) 训练的 Transformer 不是构建智能机器的正确范式，语言本身过于局限，因为它只是对一个更丰富空间（人类对现实世界的体验）的简化投影；这些机器并不真正理解（无论"理解"意味着什么），甚至有人说它们是"[随机鹦鹉](https://en.wikipedia.org/wiki/Stochastic_parrot)"。

另一方面，下一个 token 预测也可以被视为这样一项简单任务：重复数万亿次后，能够催生出复杂的行为，正如进化这一简单目标催生了复杂的生命形式一样（参见 [DeepMind 的 Adam Brown 对此的看法](https://www.youtube.com/watch?v=ykfQD1_WPBQ&t=2847s)）。

有人说操纵语言并不需要智能，也有人说，学习操纵语言、代码以及人类写下的任何符号序列，可能会首先催生出对世界的某种理解、某种逻辑等等。

下面我会尝试给出在探索这些问题时从各处收集到的不同证据。

## 1\. 实证视角：越来越难找到 LLM 答不上来的问题

一篇 [2025 年 3 月的论文](https://arxiv.org/abs/2503.23674) 表明，在图灵测试的某个特定定义下，最好的 LLM 已经通过了测试。他们的测试版本大致可以理解为 *LLM 能骗过普通人*。这虽是一大步，但如果 LLM 能骗过 *任何人*——也就是说，能骗过一个确切知道该问哪些刁钻问题的人——那将是更令人惊叹、更有意义的结果。

那么我们已经走到那一步了吗？**是否存在哪怕一个问题，是任何 LLM 都答不上来、但大多数人却能回答的？**

[这个 Reddit 帖子](https://www.reddit.com/r/LLMDevs/comments/1jl4k3i/give_me_stupid_simple_questions_that_all_llms/) 提出的正是这个问题，但没有给出令人信服的例子。那些流行的、要求 LLM 先生成推理过程否则就会失败的例子，我同样不太信服：

-   *strawberries 里有几个 r？*
-   *洗车店离我家只有 100 米，我该走过去还是开车去？*

这些失败模式确实透露了一些关于模型的信息。但它们其实算不上致命，因为归根结底，这些问题用一段相对简短的线性语言推理就能解决。而 LLM 恰恰擅长生成这类推理。

事实上，我们可以把上面的问题进一步收窄：**是否存在仅凭语言推理无法解决、但大多数盲人却能解决的问题？**

问题里的"盲人"条件是为了把 [ARC 问题](https://arcprize.org/) 排除在搜索范围之外，因为我只想专注于那些天然用语言表述的问题。ARC 任务本质上是视觉问题，盲人无法解决。事实上，ARC 本质上比较的是这两类"猛兽"：

-   生物大脑 + 眼睛
-   LLM + Python 解释器

而在这里，我主要关心的是把 LLM 当作纯文本到文本系统时的局限性。

![一位失明男子被要求解决 ARC 问题——来源：作者，像素艺术背景由 Nano Banana 2 生成](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/5-thoughts-on-llm-capabilities-and-limitations/01.jpg)

### 顿悟问题

有一类语言问题，人类似乎并不靠内心声音的顺序推理来解决，它们被称为 [顿悟问题](https://en.wikipedia.org/wiki/Insight)。顿悟问题指的是那些起初看似无解的谜题，直到表征突然发生转变才迎刃而解，并可能引发"啊哈"时刻。

> 因此，求解的途径不依赖内心言语，是不可言说的。相反，分析式的非顿悟策略涉及有意识地、逐步地搜索解法 [\[1\]](https://pmc.ncbi.nlm.nih.gov/articles/PMC10366031/)。

一个著名的此类问题是：一个人走进酒吧要喝的，结果被人用枪指着，他说了声"谢谢"就离开了。答案是这个人在打嗝。

也许顿悟问题的求解涉及某种特定的思维过程，无法被顺序的语言推理模仿。也许这是一条很有希望的路径，能找到 LLM 解不了而大多数人能解的问题。

于是我尝试设计这样一个问题（它得是*分布外*的），想出了著名的"非双胞胎"谜题的这个变体：

-   *丹尼尔和大卫由同一个母亲所生，生于同年同月同日，却不是双胞胎。已知他们再没有其他兄弟姐妹，这怎么可能？*

GPT-5.2 Pro 和 Gemini 3 Pro 只用几分钟的语言推理就找到了我预期的答案（不过它们的非 Pro 推理版本失败了）。如有兴趣，答案在文末。

也许顿悟问题的求解终究没什么特别：观察 GPT-5.2 Pro 的底层推理就会发现，交替进行 [发散思维](https://en.wikipedia.org/wiki/Divergent_thinking) 模块（生成不同的新颖视角）和收敛思维模块（逻辑演绎），就能有效地导向正确答案。

很难找到一个 LLM 解不了的简单语言问题，这恰恰证明了它们作为问答系统是何等强大。

话虽如此，这种表面上的有效性，也仍有可能是一种极其精巧的错觉。但从这里很快会滑向高度概念化的层面：**我们该如何区分真正的智能和智能的错觉？**所幸，机制可解释性领域的近期工作已经开始为此提供新线索。

## 2\. *LLM 的内部机制可能包含有意义的表征*

[2024 年 6 月的一篇论文](https://arxiv.org/abs/2210.13382) 表明，在一种名为 Othello（黑白棋）的游戏这一特定场景下，"Transformer + NTP"这套配方能够学到一些特征，这些特征似乎编码了训练数据（即 Othello 棋盘）的底层结构。

他们训练了一个 GPT-2 模型，从此前的一串走法（在这套设定里，每一步都是一个 token）来预测 Othello 棋局的下一步，并证明随后可以从该模型的内部 token 表征出发，构建一个精确的非线性探针（一个 2 层 MLP），映射到下一步之前的棋盘状态。

在机制可解释性中，探针是一个简单模型，它学习把复杂模型的内部表征映射到人类可解释的目标上。如果探针能被学出来，就说明复杂模型在其潜在空间中编码了这些人类可解释的目标。

实验结果表明，模型并非只是学到了表面相关性，而是通过梯度下降"发现"了这 2000 万条走法序列实际上是由一个结构化的二维棋盘所支配的。

**那么，如果训练序列不是 Othello 的走法，而是网上能找到的所有文字，对应的"Othello 棋盘"又会是什么？**

![Othello 类比——来源：作者](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/5-thoughts-on-llm-capabilities-and-limitations/02.jpg)

正如能编码棋盘状态似乎有助于预测有效的 Othello 走法一样，可以假设：能编码出某种人类所体验的世界表征，或许有助于预测人类接下来要说的词。而既然棋盘表征能从 Othello 的设定中涌现，也就可以假设，这种复杂的世界表征同样能从 LLM 的大规模训练中涌现。

[一篇更近期的文章](https://www.neelnanda.io/mechanistic-interpretability/othello) 进一步印证了上述结果，它表明即便是线性探针，也能把 Othello-GPT 的内部特征映射到棋盘状态。[另一篇论文](https://arxiv.org/abs/2403.15498v2) 也用一个能准确还原国际象棋棋盘的线性探针，得到了类似结果。

接下来，让我指出我们世界表征中那些显然没有在当前 LLM 里涌现出来的部分。

## 3\. 4 的形状

LLM 不会以人类的水平"用形状说话"。看下面这两个谜题：

-   *我从阿尔及利亚向正北方升起飞向法国，转向西南飞往西班牙，再向东滑行到撒丁岛。我在海上勾勒出了什么形状？*
-   *一个大写 L 坐在一个大写 X 上，看起来像什么？*

我假设对大多数人来说，回答上面这些问题并不难——不过，如果我相信问了几个朋友后得到的那点实验结果，这个假设可能下得有点重。人类能调用大脑中一项叫 [*视觉空间画板*](https://www.sciencedirect.com/science/chapter/bookseries/abs/pii/S0079742108604521) 的功能，基本上就是在脑子里作画。

反过来，LLM 解决这个问题的唯一办法是借助外部工具。求解过程包括画出图形，再分析生成的图像。可一旦拿掉这些工具，模型就会持续失败。

![四港谜题——来源：作者，Google 地图](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/5-thoughts-on-llm-capabilities-and-limitations/03.jpg)

这个*四港谜题*给出了一个非常具体的例子，说明某个符号是在语义不完整的情况下被学会的：数字 4。

模型很可能*知道* 4 是 2 + 2，知道这是正方形的边数，以及关于这个数字的许多其他事实。然而它们不知道，描出上面说的那条连续线段就能画出它的形状。它不知道：画一把由 2 段线条组成的极简小剑，甚至画一个 + 号，再用一条直线把这个图形最上面的点和最左边的点连起来，画出来的就是一个 4。

请注意，这些例子的目的并不是要对 LLM 的实际局限下个强论断（因为工具可以高效弥补这些局限），而是要提供一类具体问题——LLM 单凭自身（没有脚手架）始终解不了——以此支持这样一个观点：**"Transformer + 互联网规模 NTP"这套配方所涌现出的世界表征是不完整的。**

## 4\. 语法与语义

针对上面"智能错觉"的顾虑，还有另一种回应，叫作 [*中文房间论证*](https://en.wikipedia.org/wiki/Chinese_room)。

![中文房间论证的一个变体——来源：作者](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/5-thoughts-on-llm-capabilities-and-limitations/04.jpg)

这个著名的思想实验由美国哲学家 John Searle 于 1980 年提出，用以论证：**无论一个按规则操纵符号的系统看上去多么智能，它都并不因此就*理解*。**

针对这一观点已经有很多反驳，问题在很大程度上仍然悬而未决。

我不会进一步展开，因为那超出了本文范围。我只是把这幅图作为引子，给那些忽视了这一哲学视角的人提供一点思考。

至少有一点或许可以从中带走：人们开始援引这些哲学论证这一事实本身，再一次印证了这些模型的强大能力。

!["chinese room"搜索词的关注度变化——来源：Google Trends](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/5-thoughts-on-llm-capabilities-and-limitations/05.webp)

## 5\. LLM 是让科学取得最大进步的正确范式吗？

我认为，这最终是一种更好的提问方式来框定 LLM 能力的问题，因为它不依赖"理解"或"智能"这类复杂概念的定义。**真正重要的是，对于我们最终希望它们完成的任务，这些机器是否存在根本性的局限。**在我看来，科学进步是 AI 诸多许诺中最诱人的一个。

迄今为止，LLM 已被证明对科学家有用。它们被用来从一批复杂方程的样本中猜想出一条优雅的公式（[GPT 5.2 用于理论物理](https://huggingface.co/blog/dlouapre/gpt-single-minus-gluons)），用来核查论文（[Gemini 3 用于数学研究](https://www.youtube.com/watch?v=bNrbxCvFrKA)），等等。

然而，它们的贡献能否超越好用的工具、带来真正的科学突破，目前仍不清楚。

### Eleusis 基准

HuggingFace 最近一篇题为《LLM 能玩科学这场游戏吗？》的论文，提出了一个评估 LLM 的新基准。

ARC-AGI 或 Humanity's Last Exam 之类的大多数基准，评估的是模型在单次推理任务上的表现（一个问题，一个逻辑上随之得出的答案）。但科学研究也许不只关乎逻辑推理，也就是"回答问题"。

科学方法更像是一个迭代过程：做实验、观察结果、构建理论、验证理论，如此循环……而且这一切都得在时间和资源的约束下完成；科学家无法穷尽所有可设想的假设，必须在过程的每一步都设法押注于最有希望的下一个假设。这意味着所需的技能远不止能正确回答一个给定问题；比如论文作者就提到了**元认知**，即觉察自身不确定性的能力。

![Claude x Claude——来源：作者，使用 R.O.B. 和 Claude Lévi-Strauss 的图像拼贴而成](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/5-thoughts-on-llm-capabilities-and-limitations/06.jpg)

这个基准的内容，是参与一个叫 Eleusis 的游戏——一种 1950 年代的纸牌游戏，玩家要通过出牌来猜庄家心里想的规则，而只有符合那条隐藏规则的牌才会被接受。作者用 Python 设计并实现了 26 条游戏规则，让当前最好的模型来玩。结果展示了各模型在这场游戏中的绝对表现如何，也展示了它们落在"谨慎—鲁莽"光谱上的位置。

我想看看人类在这个"科学技能"基准上表现如何，于是 fork 了他们的 GitHub 仓库，写了一个简单的人类评估脚本，在（几乎）和 LLM 相同的条件下亲自做了测试。代码在[这里](https://github.com/clementpiat/eleusis-llm-benchmark/tree/evaluate-human)。\[2\]

![交互式 Eleusis 评估——来源：作者](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/5-thoughts-on-llm-capabilities-and-limitations/07.webp)

![个人与各 LLM 在"得分 vs 鲁莽程度"散点图上的分布——来源：作者](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/5-thoughts-on-llm-capabilities-and-limitations/08.webp)

遗憾的是，我没法从这些结果里得出任何有力的结论。最好的几个 LLM 似乎能以人类水平玩这场科学游戏；很难再多说什么。这次实验里没有任何东西能清楚地指出这些模型存在根本性的局限。

### 世界模型

最后，关于 LLM 局限性，我觉得最有说服力的假设是：**扎根于现实世界的模型**（相比困在语言空间里的模型）**也许更适合科学研究，因为科学正是要理解现实世界。**

*扎根于现实世界*在这里听起来可能有点抽象，但它的基本含义是：原始训练材料是视频（以及其他连续、高维的数据），而不是文本。这类模型通常被称为世界模型，也正是 [AMI Labs](https://amilabs.xyz/) 这类公司在做的方向。

LLM 已被证明在编程、数学等符号领域表现极佳，近期对未解数学难题的攻克（[Erdős 问题 #728](https://arxiv.org/html/2601.07421v1) 和 #281）就是格外有说服力的证据；但"连续、高维的物理世界完全是另一头猛兽"这一假设，在我看来是合理的。

## 附录

-   顿悟问题的答案：丹尼尔和大卫是同一窝的小狗。
-   另一个（据我所知）新的法语顿悟问题，GPT-5.2 和 Gemini 3 都破解了：*Deux hommes se font face pour un duel: les deux hommes tirent en même temps mais un seul d'eux tombe. Que s'est-il passé?*
-   \[2\] 这种"人类评估"方法有明显的局限。其一，测试只由一个人来做。其二，人类受试者在做不同规则的过程中可能会对任务越来越熟练，而 LLM 每局都用全新的上下文。尽管如此，这仍能给出人类在这项任务上**能**有怎样表现的一个非常粗略的估计。
-   **以上内容并非 AI 生成**。
