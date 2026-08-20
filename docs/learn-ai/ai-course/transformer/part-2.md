---
title: "理解 Transformer（第二部分）：LSTM 如何（基本）修复循环神经网络"
author: Jose Parreño
url: https://medium.com/data-science-collective/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly-dddb892471eb
translated: 2026-06-28
excerpt: 逐步解析 LSTM 单元、其巧妙的门控系统，以及它最终如何使序列学习得以实现。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/01.thumb.webp
---

# 理解 Transformer（第二部分）：LSTM 如何（基本）修复循环神经网络

逐步解析 LSTM 单元、其巧妙的门控系统，以及它最终如何使序列学习得以实现。

本周我将继续第一部分的内容：深入探索循环神经网络的世界。

如果你读过第一篇文章，就会知道循环神经网络（RNN）虽然是个巧妙的想法，但有一个很大的缺陷：它们无法长时间保持记忆。梯度会消失（或爆炸），这使得训练一个好的 RNN 网络几乎不可能。

在本系列的第二部分，我们将详细解析 LSTM 如何（基本地）修复 RNN。我们将深入探讨其门、逻辑原理，并代入一些数值，看看这些小单元内部究竟发生了什么。

让我们深入了解一下 LSTM 单元，看看它为什么会成为序列建模的主力军。🚀



## RNN 第一部分回顾

上一章节中，我们深入探讨了循环神经网络（RNN）的世界。RNN 非常巧妙，它们将记忆的概念引入了神经网络。但是，它们在大规模处理海量数据时表现不佳。

循环神经网络面临两个关键问题：

- **梯度消失**：模型会忘记长序列中的所有内容。
- **梯度爆炸**：模型的记忆变得不稳定且混乱。

训练它们简直是噩梦。理论上，它们能够对时变数据进行建模……但实际上，只要超过一个短句就束手无策。如果你需要复习一下，或者想看看一些能让你更直观地感受到梯度消失问题的可视化示例，请从第一部分开始。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/01.webp)

*PS：本文假设你已经经历过那种创伤。*

## LSTM 来帮忙啦！我们想解决的问题（目前还不需要数学公式）

循环神经网络（RNN）理论上很棒，但实践起来却很糟糕。我们想要的是一种能够做到以下几点的东西：

1. 记住重要的事情，即使它发生在 20 步之前。
2. 忘记那些没有发生的事情，这样模型就不会被噪声淹没。
3. 最重要的是，保持梯度活跃。让梯度流动。让梯度正确地更新网络。

这就是长短期记忆网络（LSTM）的用武之地。LSTM 的设计理念是*修补 RNN*（而不是重新发明轮子）。

循环神经网络（RNN）的设计理念很简单：将记忆从一步传递到下一步。这种“记忆”存储在一个单独的数字——隐藏状态——中，并被反复使用，无论事件发生多久。长短期记忆网络（LSTM）在此基础上进行了更详细的构建：

1. **LSTM 具有长期记忆路径。** 可以将其理解为基于多年经验记住“一月份通常很冷”——而不仅仅是昨天的经验。
2. **LSTM 具有短期记忆路径。** 可以将其理解为记住*昨天是温暖的一月天*，这可能有助于预测今天的天气。
3. **混合或平衡短期记忆和长期记忆的机制。** 可以把它想象成一个天平，根据上下文，你可能会更信任长期记忆而不是短期记忆（反之亦然）。

我不会说这完全复制了我们大脑的运作方式（因为它并非如此），但你可以看出其中的灵感或一些相似之处。LSTM 架构学习到，你不能对过去看到的所有信息赋予相同的权重（这是 RNN 所做的），而且，它还学习到，你可能需要使用比长期数据更多的短期数据。

现在，这一切听起来有点太理论化了。所以，让我们来看看实际的图表、公式和数学运算，这样你就能明白上面的论述是如何转化为“行动”的。

## LSTM 单元的官方示意图，看起来很吓人。图中展示了 3 个门。

1 个单元和 3 道门。这听起来像是马里奥兄弟游戏里的关卡。

这四个要素是 LSTM 单元的关键组成部分。现在，我不能保证你在接下来的几节课中会像玩 Game Boy 上的马里奥兄弟游戏那样开心，但我会尽我所能让它变得简单易懂，甚至（我敢说？）充满乐趣！

### 图表

如果你已经读过本系列的第一部分，就会知道接下来会是一个包含方框、线条和变量的流程图……当然，还有大量的注释来帮助你理解。首先让我介绍一下这个流程图，然后我们再来逐一解读其中的各个元素。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/02.webp)

### 呃，哦……这是什么乱七八糟的东西？

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/03.jpg)

我理解你觉得这张图很复杂。的确，如果你不了解它的含义，就很难看懂。让我先带你大致了解一下每个组成部分；我保证在接下来的章节中我们会深入探讨每一个部分。

1. `c(t-1)`**：这是单元状态。**图中紫色的线代表长期记忆。它仅经过两次线性运算：一次来自遗忘门的乘法运算和一次来自输入门的求和运算。由于只涉及两次简单的线性变换，因此对单元状态的更新非常平滑，这使得单元状态比循环神经网络（RNN）的长期记忆更加稳定。
2. `h(t-1)`**：这是隐藏状态。**在这种情况下，与紫色线相比，灰色线*在整个单元中*被用作*遗忘门*、*输入门*和*输出门*的输入。基本上，它会根据当前情况进行调整，以决定下一个词。
3. `x(t):` **这是新的输入数据。** 没什么特别的，只是你想让单元访问的新信息。
4. **遗忘门**。可以将遗忘门想象成一个旋钮，它决定我们应该保留长期记忆 c(t-1)c(t-1)c(t-1) 的百分比。有趣的是，它被称为遗忘门，因为网络可能会决定信任来自 `c(t-1)` 的 98% 的数据（所以实际上，这种情况更像是一个“记忆门”！）。
5. **输入门。** 可以将输入门想象成一个旋钮，它决定了应该向单元状态（长期记忆）添加多少新信息。这是一种利用新输入的数据点更新长期信念的方法。
6. **输出门。** 可以将输出门视为输入门的反面。如果输入门控制新数据对长期记忆的“影响”程度，那么输出门则控制更新后的长期记忆 `c(t)` 有多少被发送到短期记忆 `h(t)`。
7. `y(t), c(t), h(t)`**：从单元输出的内容，**要么进行转换以进行预测（例如，使用 softmax），要么简单地将其传递给下一个单元，并带有新的时间戳数据点。

很好，理解每个组件的主要目标固然有用，但仅凭这一点我们离理解 LSTM 单元内部的运作机制还差得很远。最好的方法是关注每个门，并代入一些数值进行计算。咱们拿起笔和纸吧！

## 用实际数字进行讲解。你会发现数学其实并没有那么可怕。

这一部分会比较长。抱歉，没办法。我会逐一讲解每个闸门，并涵盖三种不同的情况：

1. **情况 A**：长期记忆与新数据匹配。即，`c(t-1) = x(t)`
2. **情况 B**：长期记忆大于新数据。即，`c(t-1) >>> x(t)`
3. **情况 C**：长期记忆小于新数据。即，`c(t-1) <<< x(t)`

让我们从“遗忘门”开始。

### 遗忘之门

正如我们之前提到的，遗忘门的目的是决定我们应该赋予长期记忆多大的权重。这个权重由 `mult` 运算控制，该运算将前一个单元状态 `c(t-1)` 的值与 sigmoid 运算的结果相乘。

> *💡 温馨提示：Sigmoid 函数会将所有输入值压缩到 0 到 1 之间。*

简单来说：

- 如果 sigmoid 运算的结果接近 0，那么 `c(t-1)` 乘以 ~0 的结果也接近 0。这意味着我们要舍弃 `c(t-1)`。
- 如果 sigmoid 运算的结果接近 1，那么 `c(t-1)` 乘以 ~1 就等于 ~`c(t-1)`。这意味着我们几乎保留了 `c(t-1)` 的全部值。

那么，遗忘门内部究竟发生了什么，以及从遗忘门输出的是什么呢？让我们一起看一下图表，并一起推导一下公式。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/04.webp)

在那里，你可以看到一些新东西：

- **遗忘门之后的公式** `c(t-1)` 如前所述，`c(t-1)` 仅受乘法运算的影响。
- **“线性运算”的扩展视图。** 在这里，你可以查看神经网络更新权重的标准过程。
- `Wf_h`、`Wf_x` 分别是遗忘门的隐藏状态和输入数据的权重矩阵。
- `bf` 是 *遗忘门* 的偏置项。

**将一些数字代入 *遗忘门***

让我们来看几个例子，说明“遗忘门”的作用。

***情况 A.* 此处长期记忆** `c(t-1)` **值与新传入的数据点** `x(t)`**匹配。** 如果是这种情况，则长期记忆与新数据一致，因此无需更改它。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/05.webp)

***情况 B.* 此处长期记忆中的值** `c(t-1)` **远大于新传入的数据点** `x(t)`**。**特别是，新数据与长期记忆中的值完全相反。在这种情况下，需要调整长期记忆。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/06.webp)

***情况 C.* 此处长期记忆中的** `c(t-1)` **值远小于新传入的数据点** `x(t)` **值。**特别地，新数据也是一个正值（与 `c(t-1)` 相同，但值要大得多）。在这种情况下，我们预期“遗忘门”会尽可能多地从长期记忆中保留数据，因为它与新数据的方向一致。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/07.webp)

> ***💡 快速澄清一下。****在本节中，我一直说“遗忘门”会“比较新数据与长期记忆”，以此来决定保留多少数据。这在技术上并没有错，但这是一种简化说法。*
>
> *从图中可以看出，实际的比较是通过 **sigmoid** 函数实现的，它同时考虑了 `x(t)`（新数据）和 `h(t-1)`（短期记忆）。因此，实际上，影响决策的不仅仅是新数据，短期记忆也发挥着作用。*
>
> *为了简化概念模型，我在解释中只关注了 `x(t)`。但现在你知道了：遗忘门会学习权衡新的输入和最近的隐藏状态，从而决定记住还是遗忘。*

好了，现在你应该明白“遗忘门”是如何控制长期记忆保留比例的了。我觉得“遗忘门”这个名字有点误导人，因为实际上，有些情况下，结果是**不会**忘记来自长期记忆的内容！

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/08.jpg)

### 输入门

在上一节中，我们简要提到，它的作用就像一个旋钮，决定应该向单元状态（长期记忆）添加多少新信息。换句话说，这就是 LSTM 更新其信念的方式。

让我们来看一下示意图，并解释哪些组件会影响这种行为。与“遗忘门”不同，“输入门”有**2 个并行操作**：

- 一个决定添加*什么*——`c(t)`
- 另一个决定添加*多少*——`i(t)`

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/09.webp)

以下是内部正在发生的事情：

- `i(t):` **表示*“我们应该对更新结果有多信任？”*** 部分。它是 `sigmoid` 函数的输出，因此取值范围为 0 到 1——就像遗忘门一样。值为 0 表示*不添加任何新内容*。值为 1 表示*完全信任新的输入*。
- `c(t):` **表示“记忆应该向哪个方向移动？”部分。** 它是使用 `tanh` 函数计算的，这意味着它的取值范围为 -1 到 1。因此，该值可以**增加**、**减少**或**不**影响长期记忆。
- 将这两个值相乘，然后**添加到更新后的单元状态**。

所以，如果你把“遗忘门”看作是选择保留什么，那么“输入门”就选择将记忆推向**哪个新方向**以及推向多远。

**将一些数字代入 *输入门***

让我们来看几个例子，说明*输入门*的作用。

***情况 A.* 此处长期记忆中的** `c(t-1)` **值与新传入的数据点** `x(t)` **相匹配。**如果情况如此，则长期记忆与新数据一致，因此理想情况下无需对其进行更改。在下面的示例中，你可以看到，基于我设定的权重和偏差，更新将 `c(t-1)` 的值增加了 0.5，使其达到 2.5。这可能有点过大——但请记住，这些权重只是设定的。如果这种做法没有帮助，学习过程最终会进行修正。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/10.webp)

***情况 B.* 此处长期记忆** `c(t-1)` **远大于新传入的数据点** `x(t)`**。**特别是，新数据与长期记忆的值完全相反。在这种情况下，需要调整长期记忆。如图所示，我们希望保留新数据较高的比例，而新值正在降低 `c(t-1)` 的值。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/11.webp)

***情况 C.* 此处长期记忆** `c(t-1)` **远小于新输入的数据点** `x(t)`**。**特别地，新数据也是一个正值（与 `c(t-1)` 相同，但值要大得多）。在这种情况下，该值从 2 提升到 2.97，试图更接近新输入的数据 `x(t)=10`。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/12.webp)

> ***💡 简要说明。****LSTM 无法在一步之内彻底改变其长期记忆——这是*有意为之*的设计。由于更新过程会经过 sigmoid 函数（0 到 1）和 tanh 函数（-1 到 1），因此每一步最多只能将记忆移动 ±1。这迫使网络逐步“获得”较大的变化，保持梯度稳定，并实现对记忆或遗忘内容的平滑、精细控制。*

太好了，现在你可以看到*输入门*是如何控制多少新的传入数据（以及隐藏状态）影响长期记忆的。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/13.jpg)

### 输出门

我们终于到达了第三个门。在这个门中，长期记忆不受影响。但是，长期记忆会用来调整短期记忆。换句话说，这个门利用新数据和长期记忆中的 c(t) 来决定 h(t) 的移动方向。

我们来看一下示意图，并解释哪些组件会影响这种行为。“输出门”与“遗忘门”和“输入门”都有相似之处。

- **类似于 *遗忘门***，因为只涉及 1 个操作：`tanh` 和 `sigmoid` 的输出相乘。
- **类似于*输入门***，因为激活函数`tanh`和`sigmoid`都决定了要保留的*百分比*以及短期记忆应该向哪个*方向*移动。
- **与*输入门*的一个重要区别在于，** 输入门中的两条路径都是可学习的。而在输出门中，**tanh 函数是固定的**——它的作用仅仅是将记忆信号 `c(t)` 压缩成一个清晰、有界的形状。唯一可学习的组件是 sigmoid 门，它控制着有多少被压缩后的信号最终成为 `h(t)` 的一部分。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/14.webp)

**为什么这次我们不代入数字**

在“遗忘门”和“输入门”中，我们可以比较新数据如何更新长期记忆。我们可以观察 LSTM 是选择信任它、忽略它还是将其融合。但“输出门”则有所不同。

这个门不会更新记忆：它会过滤记忆。具体来说，它决定**当前记忆** c(t) 的**多少**应该通过短期状态 h(t) 暴露给外部世界。

因此，这里不像其他关卡那样存在“前后对比”。所以，我们不采用具体的数字，而是从概念层面进行探讨。

- 如果输出门完全打开（sigmoid ≈ 1），则所有记忆都暴露出来。
- 如果它大部分是封闭的（sigmoid ≈ 0），我们就隐藏它——不管记忆中怎么说。

就这么简单。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/15.jpg)

## 为什么 LSTM 比 RNN 更好？

如果你已经看到这里，那你肯定已经知道答案了。

循环神经网络（RNN）只有一个任务：传递记忆。但它们只使用单一通道，即单一隐藏状态。这条通道很容易被噪声堵塞，丢失重要信息，或者在训练过程中破坏梯度。

另一方面，LSTM 则采用了**长期记忆和短期记忆的独立路径**、巧妙的门控机制来更新它们，以及一种能够温和地引导记忆的结构。它包含一个“遗忘门”来剔除不再有用的信息，一个“输入门”来融合新的学习成果，以及一个“输出门”来决定输出哪些信息。

## 问题一：训练 LSTM 模型仍然非常慢。

LSTM 非常出色，但它们也存在与 RNN 完全相同的问题。

我们之前画的那些门、激活函数、权重和偏置项？它们**在每个时间步都会运行**。与可以并行处理输入的前馈网络不同，LSTM 需要按顺序逐步处理序列。

所以，即使我们解决了梯度问题，也付出了**计算开销**的代价。LSTM 训练速度慢，扩展性差，而且处理长序列非常吃力。

这种建筑设计很巧妙，但是并不轻便。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/16.jpg)

## 问题二：LSTM 在处理长序列时仍然存在困难。

是的，它们比传统的循环神经网络要好。但是，它们并非魔法。

即使使用了遗忘门和单元状态，LSTM 仍然很难**记住数百步前发生的事情**。梯度仍然会衰减（即使衰减缓慢），这意味着记忆仍然会消退。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly/17.jpg)

## 接下来会发生什么：Transformer 摒弃了循环模式……而且成功了！

2017 年，Transformers 出现了，它说：“如果我们不……像 RNN 或 LSTM 那样使用循环或门控会怎么样？” 在接下来的文章中，我们将详细解释 Transformers 的工作原理——从 Transformers 如何表示单词、如何知道这些单词在哪里以及如何决定关注什么开始。

## 现在，我想听听你们的意见！

📢 在这篇文章中，我们详细讲解了 LSTM 如何解决 RNN 的一些主要问题——从梯度消失到记忆瓶颈。我们逐个门地分析了 LSTM 的单元结构，用实际数据展示了其数学原理，并阐述了 LSTM 为何成为序列数据处理的首选模型。

但我现在对你的经历很好奇。

- 你有没有从头开始实现过 LSTM，或者尝试过用 PyTorch 或 TensorFlow 来调优 LSTM？
- 你是否遇到过训练速度慢，或者难以让它记住几步以外的内容的情况？
- 或者，这可能是你第一次看到 LSTM 如何将长期记忆与瞬时更新结合起来——以及为什么它们比普通的 RNN 有了如此大的进步。

请在评论区留下你的想法、经验或疑问——我很想了解你在机器学习之旅中是如何处理序列建模的。👇
