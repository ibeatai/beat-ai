---
title: "理解 Transformer（第五部分）：最后几层发挥着重要作用"
author: Jose Parreño
url: https://medium.com/@joparga3/understanding-transformers-part-5-the-final-layers-doing-some-heavy-lifting-2d3a6cc8e981
translated: 2026-07-13
excerpt: LayerNorm、残差、前馈块和编码器-解码器管道
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@b46195d2022409bce1e1eaf436f863ffb1c9457f/ai-insights/2026-07/13/images/understanding-transformers-part-5-the-final-layers-doing-some-heavy-lifting/01.thumb.webp
---

# 理解 Transformer（第五部分）：最后几层发挥着重要作用

LayerNorm、残差、前馈块和编码器-解码器管道

本周，我将用最后一篇深入解析为 Transformer 系列收尾：聚焦那些让整个模型得以运转、更小但至关重要的组成部分。

在过去的 4 篇文章中，我们涵盖了从 RNN 痛点到 LSTM 修复，再到位置编码，最后是 LLM 背后的完整注意力机制（尽管请记住，Transrfomers 除了用于文本之外还可以用于其他用途）。

但是，架构图中还有一些部分我们尚未涉及。

在最后一篇文章中，我们将介绍“*配角阵容*”：

- LayerNorm：为什么每个 token 在进入下一步之前都要重新缩放
- 残差连接：保持信号畅通的跳跃车道
- 前馈网络：每一步都进行局部非线性细化
- 编码器-解码器注意力：让翻译得以实现的唯一管道

到最后，你会一块一块地看完整个 Transformer 模块——完整的图景（但愿）会豁然开朗。

让我们为这个系列漂亮地收个尾。

## 博客系列

- ✅ [理解 Transformer（第一部分）：为什么 RNN 几乎无法训练](https://medium.com/data-science-collective/transformers-part-1-why-rnns-are-nearly-impossible-to-train-30d1986f5960)
- ✅ [理解 Transformer（第二部分）：LSTM 如何（基本）修复循环网络](https://medium.com/data-science-collective/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly-dddb892471eb)
- ✅ [理解 Transformer（第三部分）：位置编码和词嵌入](https://medium.com/data-science-collective/understanding-transformers-part-3-positional-encodings-and-word-embeddings-3e3802f11a81)
- ✅ [理解 Transformer（第四部分）：注意力机制其实只是几个矩阵](https://medium.com/data-science-collective/understanding-transformers-part-4-attention-is-just-a-few-matrices-462505291833?sharedUserId=joparga3)
- **📌 理解 Transformer（第五部分）：最后几个简单的层级承担着重要的功能** ***（这篇博文！）***

## 本博客将涵盖哪些内容？

1. **关于自注意力的回顾。**
2. **令人头疼的图表。**（我保证，这是最后一次了。）
3. **LayerNorm。** *“清理”小组。*
4. **残差连接。** *Transformer 架构内部的高速公路。*
5. **前馈模块。** *可以把它想象成网络的私人思考时间。*
6. **单一编码器→解码器管道。** *一个连接，统领一切*

*别忘了订阅，以便及时了解最新发布的文章！*

## 自注意力回顾。

在第四部分中，我们打开了**注意力机制**的黑匣子，详细分析了每个 token 如何创建一个**查询 (Q)**，将其与**键 (K)**目录进行比较，并通过**值 (V)**检索内容。我们还可视化了点积、缩放、softmax，以及最终将原始词转换为上下文感知向量的矩阵混合。

文章结尾，注意力机制已不再是谜。但如果你去看完整的 Transformer 结构图，会发现还有几个方框被我们跳过了。

这正是最后一部分要做的事。这一次，我们深入那些默默让 Transformer 运转起来的**鲜少被谈及的组件**：LayerNorm、残差、前馈网络，以及连接编码器与解码器的那条小小管道。

让我们为这个系列画上句号。

## 令人头疼的图表。（我保证，这是最后一次了）

为了铺垫背景，今天我们重点看下图中紫色高亮的四个组件。有意思的是，只要理解了其中一个模块，就能明白它在架构其他部分里做的事。例如，理解了 LayerNorm 在多头注意力之后的作用，也就明白了它在前馈层之后的作用。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@b46195d2022409bce1e1eaf436f863ffb1c9457f/ai-insights/2026-07/13/images/understanding-transformers-part-5-the-final-layers-doing-some-heavy-lifting/01.webp)

## LayerNorm。“清理”小组。

让我们从注意力模块的输出开始——一组向量，每个词对应一个向量，其中包含关于词义、位置以及它对其他词的关注强度等信息。请记住，这些矩阵中的数值可能很大也可能很小。在第一部分和第二部分中，我们讨论了梯度爆炸和梯度消失的问题（我们在第三部分和第四部分中没有涉及）。不出所料，除非我们采取措施，否则它们仍然是个问题。

这就轮到 **LayerNorm** 登场了。

### 这个 LayerNorm 到底是什么？

LayerNorm 会取每个 token 向量，将其值**归一化**，使其均值为 0、标准差为 1，然后（可选地）让网络使用学习到的权重进行缩放或调整。它就像一个**清理小组**：在进入下一个子层（无论是注意力层还是前馈层）之前，“安抚”每个特征。

### 一个简单的玩具示例

假设你的模型正在处理以下句子：

**“The dog sat .”**

经过一个多头注意力层后，你可能会得到如下的词向量：

```
the → [ 3.2 , -1.8, 0.5 , 2.1 ]
dog → [ -4.4, 5.0 , 1.2 , 0.2 ]
sat → [ 6.3 , 6.1 , 6.0 , 5.8 ]
.   → [ -0.1, 0.0 , -0.2, 0.1 ]
----
take [the] -> mean = 1, standard deviation = 1.9 (np.std(data, ddof=0))
---
the (normalised) = [ 1.16, -1.47, -0.26, 0.58 ]
```

因此，现在不会出现像 3.2 或 -1.8 这样的异常值，而是会得到介于 -1.5 和 1.5 之间的居中数字。其他所有 token 都经过相同的处理——*独立地*。

这一点“独立”至关重要：**LayerNorm 是逐个 token 进行操作的**，而不是跨批次操作。这正是它与 BatchNorm 的区别所在。BatchNorm 在这里会遇到困难，因为它依赖于跨批次的统计信息——而当序列长度不同或批次大小降至 1 时，这些统计信息就会失效。

就这样。简单的逐 token 归一化。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@b46195d2022409bce1e1eaf436f863ffb1c9457f/ai-insights/2026-07/13/images/understanding-transformers-part-5-the-final-layers-doing-some-heavy-lifting/02.jpg)

现在再看 Transformer 结构图，你会注意到几个跳过注意力或前馈模块、直接连到 LayerNorm 输出的捷径箭头。这些是什么？为什么有用？

## 残差连接。Transformer 架构内部的高速公路。

让我们来谈谈将整个 Transformer 连接在一起的看似简单却至关重要的东西：**残差连接**。

乍一看 Transformer 架构图，确实感觉很奇怪。我们为什么要*跳过*与我们超酷的多头注意力层（或其他任何具有残差连接的层）的连接呢？我们在第四部分不是说过注意力机制是整个架构的核心改进吗？

这张图的字面意思是：“你可以直接使用原始输入向量，完全绕过注意力层，直接进入 LayerNorm 层。” 这到底是怎么回事？

这看起来很奇怪，但正是这种捷径使得架构稳健且易于训练。

### 什么是残差连接？

其本质就是如此：

其中：

- `x` 是子层的输入（例如，“dog”的词嵌入 + 位置编码）。
- `F(x)` 是我们对 `x` 施加的任何变换的输出——例如，自注意力模块或前馈网络。

因此，我们不会丢弃原始输入，而是在子层完成其操作后将其***重新添加***。这就是我们所说的**残差路径**。

### 让我们具体一点：第一层，token “dog”

此时：

- `x` = *“dog”* 的词嵌入，加上其位置编码
- `F(x)` = 来自自注意力层的微小更新，例如：“给 *dog* 一些 *sat* 和 *couch* 的暗示，并降低 *the* 的影响”

套用 `y = x + F(x)` 这个公式，你就得到了一个被句子上下文塑造过的、略有更新的 *“dog”*。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@b46195d2022409bce1e1eaf436f863ffb1c9457f/ai-insights/2026-07/13/images/understanding-transformers-part-5-the-final-layers-doing-some-heavy-lifting/03.jpg)

### 残差总是会改变向量吗？

不，这就是魔法所在。

如果某一层没有什么有用的信息要传递，它就会学会什么都不传递。模型可以通过将 `F(x)` 的值设置得非常小来简单地忽略该向量——尤其是在训练初期。

这就是残差如此强大的原因：**它们不会强迫每一层都发挥作用。**相反，它们赋予模型自由，使其在最明智的情况下*原封不动地复制*。

### 我们何必呢？

**1. 它能保持梯度流动。** 在第一部分中，我们看到梯度在深度网络中是如何消失（或爆炸）的。残差连接有助于解决这个问题。它们为梯度提供了一条捷径——使模型能够训练更深的层级（6 层、60 层，甚至 GPT-4 中的 96 层），而不会崩溃。

**2. 它赋予模型选择权。** 如果某个子层暂时没有用处，模型可以直接跳过它。真的，就是直接跳过。如果 `F(x)` 接近于零，输出基本上就是 `x`。网络会学习何时进行变换，何时应该放手。

既然我们已经讲过 token 如何借助残差跳过某一层，那就来看看当它们真的留下来时，实际会经历什么。

## 前馈模块。可以把它想象成网络的私人思考时间。

紧接在注意力之后、进入下一层之前，每个 token 都会经过一个小型的两层神经网络（我猜这两层可以扩展，但姑且当作两层）。这就是**前馈网络（FFN）**。

在我的想象里，注意力层就像一场热闹的课堂讨论；讨论一结束，每个学生翻开笔记本，把共享的想法整理成更具体、更私人的笔记（也就是 FFN 网络）。

我讲得太抽象了吗？我们来看一些技术细节吧。

### 它实际有什么作用？

简而言之：**expand → ReLU → shrink**。

LayerNorm 输出的每个 token 向量 `h` 都会经过以下变换：

```
z = W₂ · ReLU(W₁ · h + b₁) + b₂
```

其中：

- `W₁` 扩展 token 的特征空间（例如从 512 维 → 2048 维）
- `ReLU` 引入了非线性和稀疏性（某些值趋近于零）。
- `W₂` 将所有内容压缩回 512 维。

此模式为每个 token 赋予：

- **额外容量**，用于组合和重新混合它刚从注意力中收集到的特征
- **非线性**，因此该模型不仅仅是一系列线性变换的简单堆叠。
- **独立性**，因此每个 token 都能各自专门化——例如 *“dog”* 可能强化它的主格特质，而 *“sat”* 则强化它的动词特质。

### 我们来看一个玩具示例。

假设 token “sat” 注意力之后的向量 `h` 长这样：

```
h = [1.2, -0.3, 0.5, 0.9]
```

**步骤 1 — 展开**

```
u = W₁h + b₁ = [2.4, -1.8, 0.7, 3.1, -0.4, 1.2]
```

**步骤 2 — ReLU**

```
ReLU(u) = [2.4, 0, 0.7, 3.1, 0, 1.2]
```

**步骤 3 — 压缩**

```
z = W₂ · ReLU(u) + b₂ = [0.9, 0.1, 1.4, -0.2]
```

输出 `z` 是原始 token 变换后的版本，准备进入残差相加，然后再送入下一个 LayerNorm。

### 为什么这个小小的 MLP 很重要？

如果没有 FFN，Transformer 就只不过是一堆高级线性层的堆叠而已：

- **更丰富的表示** — 宽阔的中间层（基础 Transformer 中有 2048 个维度）为模型提供了探索的空间
- **token 特化** — 由于每个 token 都是单独处理的，因此 FFN 可以针对不同的词放大不同的内容。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@b46195d2022409bce1e1eaf436f863ffb1c9457f/ai-insights/2026-07/13/images/understanding-transformers-part-5-the-final-layers-doing-some-heavy-lifting/04.jpg)

## 单一编码器→解码器管道。一个连接，统领一切

太棒了，从第三部分、第四部分到本文前面的章节，我们已经讲遍了 Transformer 架构里的每个模块。还剩一个重要部分要讲：编码器与解码器之间那唯一的一座桥。

我们先从简单的开始。

### 那根管道里实际流的是什么？

经过最后一个编码器层后，源句子中的每个 token 现在都是一个向量——我们称之为 `eᵢ`。这些是 512 维的表示，吸收了语义、句法、位置以及与其他词语的关系。

所有这些向量都堆叠成一个矩阵 `E = [e₁, e₂, ..., eₙ]ᵀ`，它成为一个**冻结的记忆库**。

就是这样。一个包含丰富的、逐 token 信息的查找表。推理过程中，编码器完成工作——解码器可以从这块记忆里读取信息，但不会有任何信息回传。

### 为什么编码器输出是冻结的？

冻结编码器输出后，我们可以得到：

- **效率** — 解码器不需要为生成的每个单词重新运行编码器。
- **模块化** — 同一个编码器的输出可以馈送多个解码器（例如，一个用于翻译，一个用于对齐可视化）。

### 为什么一根管道就足够了

说实话，这并非什么新鲜事。我们在循环神经网络（RNN）和长短期记忆网络（LSTM）中都见过这种单通道结构，所以从概念上讲，它并非突破性的创新。但我仍然想介绍一下，因为它有助于理解我们如何只训练一个编码器，并在未来非常轻松地将其与解码器关联起来。

每个编码器 token 都经过了六个自注意力层，这意味着每个词都有机会与其他所有词进行组合。因此，`E` 中的每个向量都知道：

- 它源自哪个词
- 它在句子中的位置
- 其句法作用（主语、谓语、宾语……）
- 它与远方 token 有何关联
- 甚至句子层面的属性，例如时态或语气

> *换句话说：解码器并非直接处理原始词嵌入，而是利用* ***预处理过的、富含上下文信息的智能信息***。

所以说，**一根管道就足够了**。

### 常见误解澄清

1. ***“梯度必须在每一步都从解码器流向编码器。”*** → 不对。**仅在训练期间。** 在推理时，编码器是冻结的。
2. ***“交叉注意力机制只发生一次。”*** → 它在每个解码器层发生一次（通常为 6 次）。每一层都获得对同一编码器输出的不同视图。

这种单个编码器到解码器的连接虽然很小，但效率极高，这也是 Transformer 如此快速、可扩展和模块化的关键原因。

下次你在图表中看到那支孤零零的箭头时，请对它保持敬畏之心。它承载着很多信息。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@b46195d2022409bce1e1eaf436f863ffb1c9457f/ai-insights/2026-07/13/images/understanding-transformers-part-5-the-final-layers-doing-some-heavy-lifting/05.jpg)

## 现在，我想听听你们的意见！

📢 在最后一篇文章中，我们通过拆解“配角”——那些让整个架构稳定、可扩展、可训练的细微部件——为 Transformer 系列收尾。

我们涵盖了以下内容：

- LayerNorm 如何保证每个 token 在统计上合理，
- 残差连接如何为每一层提供逃生通道，
- 前馈模块如何引入非线性思维，
- 以及一个编码器到解码器的单一管道，如何就足以支撑整个翻译过程。

连同第 1 至 4 部分——从 RNN 的挣扎到 LSTM 的改进，从位置编码到完整的自注意力——这完成了对 Transformer 底层工作原理的完整讲解。

但我现在很想听听你们的看法：

- 💬 这最后一部分是否让图表最终“清晰明了”？
- 🧠 之前你忽略的某些部分，现在看来更有意义了吗？
- ⚙️ 又或者，你已经在想这些部分如何扩展成 LLM 了？

欢迎在评论区留下你的想法、问题或感想——也感谢你一路跟到这个系列的最后。👇

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@b46195d2022409bce1e1eaf436f863ffb1c9457f/ai-insights/2026-07/13/images/understanding-transformers-part-5-the-final-layers-doing-some-heavy-lifting/06.jpg)

## 延伸阅读

感谢阅读本文！如果你对我更多的文字内容感兴趣，这里有一篇文章按主题汇总了我其他所有博客文章：数据科学团队与项目管理、数据叙事、营销与竞价科学，以及机器学习与建模。

## 敬请关注！

如果你想在我发布新内容时收到通知，欢迎在 Medium 上关注我。另外，[我非常乐意在 LinkedIn 上和你聊聊](https://medium.com/@joparga3/www.linkedin.com/in/joseparrenogarcia)！
