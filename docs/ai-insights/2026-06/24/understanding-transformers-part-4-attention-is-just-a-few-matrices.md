---
title: “理解 Transformer（第四部分）：注意力不过是几个矩阵”
author: Jose Parreño
url: https://medium.com/data-science-collective/understanding-transformers-part-4-attention-is-just-a-few-matrices-462505291833
translated: 2026-06-24
excerpt: 简单的矩阵运算如何揭示上下文、意义以及现代人工智能背后的奥秘。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/01.thumb.webp
---

# 理解 Transformer（第四部分）：注意力机制其实只是几个矩阵

简单的矩阵运算如何揭示上下文、意义以及现代人工智能背后的奥秘。

本周，我将接着上次中断的地方讲：跨过词嵌入和位置编码，直接进入 **Transformer 的核心魔法：自注意力**。

但在讨论多头或解码器栈之前，我们需要回答一个关键问题：

➡️ **Transformer 究竟是如何一次性地将整个句子中的信息混合在一起的？**

在这篇文章中，我们将拆解**缩放点积注意力（Scaled Dot-Product Attention）**机制的完整流程：

- 如何通过查询和键来比较词语，
- 为什么缩放和 softmax 是关键步骤，
- 以及如何通过融合价值向量来构建新的、富含上下文的表示。

这一切归根结底只是几个简单的矩阵乘法运算。

## 博客系列

- ✅ [理解 Transformer（第一部分）：为什么 RNN 几乎无法训练](https://medium.com/data-science-collective/transformers-part-1-why-rnns-are-nearly-impossible-to-train-30d1986f5960)
- ✅ [理解 Transformer（第二部分）：LSTM 如何（基本）修复循环网络](https://medium.com/data-science-collective/understanding-transformers-part-2-how-lstms-fixed-recurrent-networks-mostly-dddb892471eb)
- ✅ [理解 Transformers（第三部分）：位置编码和词嵌入（*这篇博文！*）](https://medium.com/data-science-collective/understanding-transformers-part-3-positional-encodings-and-word-embeddings-3e3802f11a81)
- **📌 理解 Transformer（第四部分）：注意力机制其实就是几个矩阵** ***（这篇博文！）***
- 理解 Transformer（第五部分）：最后几个不起眼的层，却在默默承担繁重的工作 *（敬请期待！）*

## 本博客将涵盖哪些内容？

1. **第三部分位置编码回顾。**
2. **令人头疼的图表（又来了）**
3. **注意力机制的四个步骤**
4. **自注意力的简要回顾**
5. **通过多头协作来扩大注意力范围**

*别忘了订阅，以便及时了解最新发布的文章！*

## 第三部分位置编码回顾。

上次，我们看了每个词元进入 Transformer 时经过的第一站：词嵌入和位置编码的组合。

词嵌入本身可以让我们了解一个词的*含义*。“蛋糕”比“键盘”更接近“派”。但仅凭词嵌入无法感知位置信息：模型知道句子中*是什么*，但不知道*在哪里*。

因此，我们添加了**位置编码，以帮助模型跟踪顺序**。有了它，Transformer 就知道“狗坐在沙发上”和“沙发坐在狗身上”是不一样的（除非你访问的是互联网上一些非常奇怪的网站）。

到这一步结束时，句子中的每个词元现在都表示为一个单一的向量（它将单词的含义与它在句子中的位置结合起来）。

### 但这还不够。

我们来看一个简单的句子：

> ***“I ate the cake and it was delicious.”***

- 模型知道“cake”是食物（词嵌入）。
- 它也知道“it”出现在后面（位置编码）。
- 但它怎么知道“it”指的是“cake”，而不是“I”呢？又怎么能把“delicious”这个词和“cake”联系起来呢？

这就是位置编码的真正局限性。它能提供结构，*但不能提供关系*。**这就是注意力发挥作用的地方。**

嵌入+位置阶段的结果将成为**注意力机制的输入**。下一阶段的任务是做出决定：

➡ *每个词元到底应该关注句中的哪些词？*

是时候再次见到这一切的起源图了……

## 令人头疼的图表（又来了）

我知道，我习惯用直接从学术论文里摘录出来的那些吓人的技术图表来迎接大家。但我保证，我们会一步一步地讲解，到最后，你会发现它们并没有那么可怕。所以，请大家（再次）耐心看完。

正如我们在第三部分中看到的，Transformer 由多个活动部件构成。这种架构源自 2017 年的论文《注意力机制：你只需要它》（Attention is all you need）。在下面的图中，您可以看到我们将*仅*关注蓝色高亮显示的方框——即标记为“多头注意力机制”（Multi-Head Attention）的方框。

这就是*自注意力*的根源所在。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/01.webp)

### 注意力机制的细节

幸运的是，论文中包含一张放大图，展示了这些方框*内部*的情况。今天我们将主要关注的是：**缩放点积注意力**图（左侧的那个）。

你还会看到一个额外的粉色方框，上面标有“掩码（可选）”——这仅在训练过程中用于某些特定情况（例如解码器）。由于本文重点在于基础自注意力机制的原理，我们将略过它。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/02.webp)

让我猜猜——这仍然完全说不通。

你可能想问：

- Q、K、V 分别代表什么？
- 缩放点积注意力图里遵循的 4 个步骤是什么？
- 为什么 Q 和 K “在一起”，而 V 却走了一条不同的路？

没错，你来对地方了，这里可以解答这些问题以及更多其他问题。让我们在下一阶段详细探讨所有这些内容！

## “缩放点积注意力”的四个阶段

### 预备阶段：引入 QK、V。

如果你看一下 **Scaled Dot-Product Attention** 图，你会看到三个神秘的向量作为输入：**Q、K 和 V**。

在我们深入探讨数学之前，让我们先来谈谈你可能每天都在使用的东西：**推荐系统**，例如 Netflix（我假设你一生中某个时候肯定使用过）。

假设你打开 Netflix 并搜索“科幻小说”。那么，后台发生了什么？

1. **你提出一个查询。** 你是说，“给我展示符合科幻小说的东西”。
→ 这是你的**Q**：查询。
2. **系统会将您的查询与选项目录进行比对。** 每部电影或剧集都有一些标签或特征：动作、剧情、惊悚、科幻、爱情喜剧……
→ 这些是 **K**：**键**——每个项目的元数据或标签。
3. **系统找到匹配项后，需要知道该推荐什么。
不仅是标题，还有缩略图、评分，甚至可能是预告片预览。
→ 这些就是 **V**：**价值**——你实际收到的信息。

所以：

- **Q** = 你正在搜索的内容
- **K** = 每个项目的标注方式
- **V** = 匹配成功后你能拿到的东西

系统把你的查询 (Q) 与键 (K) 进行比较，找出最相关的项，然后返回这些相关项的值 (V)。

**现在，让我们回到 Transformer。**

在 Transformer 中，我们不是在推荐电视节目，而是在处理**句子中的词语**。但其机制却出奇地相似。假设我们正在看句子中的单词**“it”**：

> ***“I ate the cake and it was delicious.”***

- **Q** = 单词“it”的数值表示（我们的查询——“it”试图理解什么）
- **K** = 句子中每个单词的键（例如“cake”、“ate”、“and”等）
- **V** = 与这些词语相关的价值——我们可能想要从中提取的实际信息

目标是让**“it”**关注**“cake”**，而不是“and”或“I”。就像 Netflix 的目标是在你搜索科幻作品时为你呈现*星球大战*，而不是*爱情岛*一样。

下一节，我们将从直觉过渡到具体机制：

- 这些 Q、K 和 V 向量在数学上*长什么样*？
- 它们在 Transformer 内部是如何构造出来的？

但希望到目前为止，你已经有了大致的认识：

> *Q 是寻找者*
> 
> *K 是它用来比对的对象（*所有*词元的目录）*
> 
> *V 是它将得到的回报——按匹配强度加权。*

### 步骤 1. Q 和 K 的矩阵乘法

我一直很纳闷，为什么技术论文里非要用这么神秘兮兮的语言。为什么不直接说矩阵乘法，而要用 MatMul 呢？算了，今天就到此为止吧。

让我们重点关注缩放点积注意力图的第一步。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/03.webp)

### 我们的输入句子（以向量形式）经过两次线性变换。

让我们继续来看之前的例子：

> *“I ate the cake and it was delicious.”*

经过嵌入层和位置编码层之后，每个词都变成了一个数值向量，它同时捕捉了*含义*和*位置*。

为简单起见，我们假设这些是 4 维向量（见下图红色方框）。

每个词嵌入都会通过矩阵 `Wq` 和 `Wk` 进行“推送”，从而成为一个新的向量。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/04.webp)

### “通过矩阵推送”是什么意思？

简单来说，就是应用**线性变换**：我们使用标准的矩阵乘法规则，将词的嵌入向量与矩阵**相乘**。

简单来说：

- 取出嵌入向量
- 与矩阵的每一列做点积
- 这就得到一个新的向量

要了解简单的点积数学运算是什么样子，请查看下图。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/05.webp)

让我们从左到右，一点一点地分析：

首先关注向量和矩阵的维度。请注意：

- 每个单词都是一个 **\[1 × 4\]** 向量。
- 每个矩阵都是 \[4 × 4\]。
- 词向量中的 **4**（元素个数）必须与矩阵中的 **4**（行数）相匹配，乘法才能成立。

*现在，让我们关注向量 `Qcake`。*

- 它是通过将“cake”的嵌入向量乘以 `Wq` 来计算的。
- `Qcake` 中的每个值都是“cake”的嵌入与 `Wq` 的一列之间的点积。
- 在上面的示例中，我们展示了如何计算 `Qcake` 向量的第一个元素。

**类似地，对于** `**Kit**` **向量：**

- 我们取“it”的嵌入向量，
- 将其乘以 `Wk`，
- 并以相同的方式计算每个元素。
- 在图中，我们以 `Kit` 向量的最后一个元素为例进行说明。

### 现在，我们准备好进行 Q 和 K 的 MatMul 运算了。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/06.jpg)

一旦我们通过 Q 和 K 矩阵的权重投影了嵌入，我们就可以计算 Q 和 Kᵀ 的 MatMul 了。

下图展示了一个示例：

- `Qcake` 向量如何与 `Kit` 向量交互。
- 同样地，`Qcake` 与句子中的*其他每个单词* 会如何比较。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/07.webp)

*这里会发生什么？*

每个词都作为一个**查询词** (Q)，与**所有词作为键** (K) 进行比较。输出结果为：

- 每个查询词（在本例中为“cake”）**1 行**。
- **每关键词一列**（句子中的每个词）。
- 此行中的每个**单元格**都记录了查询与该键的***相似度***。

例如：

- “cake”和“it”得分 **1.3** — 高度相似。
- “cake”和“I”得分 **0.4** — 非常弱。

在完整的 Transformer 模型中，我们会对*每个词*重复此操作，将每个查询与所有键进行比较。结果是一个**完整的相似度矩阵**，它显示了每个词与其他词之间的相似度。以下是一个相似度矩阵的示例：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/08.webp)

太棒了！现在我们已经了解了第一步的输出结果。接下来，让我们进入“缩放”步骤。

### 步骤 2. 调整分数

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/09.webp)

我们现在得到了一个完整的相似度得分矩阵，其中每个词（作为查询词）都与其他所有词（作为键词）进行了比较。但是，这里存在一个问题。

有些数字已经相当大了（例如，当一个词与自身进行比较时，其值为 1.6），即使我们只处理较小的 4 维向量。想象一下，如果我们处理的是 512 维甚至 1024 维的嵌入向量会发生什么——这在真正的 Transformer 模型中很常见。

这些点积分数*可能*会迅速增长到非常大的规模。

### 为什么这会是个问题？

因为下一步我们要应用**Softmax**函数。而 Softmax 函数对输入数据的尺度非常敏感。

- 如果数值太大，softmax 函数就会变得非常“尖锐”——它将几乎所有的概率质量都推向一个词，而其他地方的概率质量几乎为零。
- 这使得模型变得脆弱——输入的微小变化会导致输出的巨大变化。
- 这也使得学习更加困难，因为梯度变得很小且不稳定。

### 解决方案：在 softmax 之前对分数进行缩放

我们不会直接将原始点积输入到 softmax 函数中，而是先将每个分数**除以**一个数字。具体来说，这个数字是向量大小的平方根 (√d)。

在我们的简单示例中，由于每个向量都有 4 个维度：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/10.webp)

因此，在进行 softmax 运算之前，每个分数都会除以**2**。这个简单的技巧**控制**了分数的幅度，使 softmax 输出保持稳定，从而更容易让模型学习。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/11.webp)

把这想象成准备冲泡咖啡的水：

- 如果水温太高（原始点积），咖啡会烧焦、味道很糟糕（softmax 爆炸）。
- 如果先将其调整到合适的温度，就能冲泡出一杯完美的咖啡（softmax 表现良好）。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/12.jpg)

## 步骤 3. 应用 Softmax

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/13.webp)

现在我们已经对分数进行了缩放，还有一个关键步骤：将这些原始数字转换为**概率**。

缩放后的分数告诉我们每个查询与每个键的相似程度——但它们仍然是原始数字。

我们希望将它们转化为可解释的东西：

- 介于 0 和 1 之间的正数
- 其中每一行的总和都恰好为 1
- 这样我们就能把它们当作词语上的**概率分布**来处理

如果没有 Softmax，模型就无法知道应该给每个词分配多少相对“重要性”。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/14.jpg)

### Softmax 是什么？

Softmax 是一个简单的数学函数：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/15.webp)

其中：

- `zi` 是该行中的第 i 个分数
- `e^zi`​ 是该分数的指数（使所有输出均为正数）
- 分母是对***该行***所有分数求和以做归一化（再次强调，softmax 是按行应用的）
- softmax 的另一个额外好处是，它能为 `Q` 与 `K` 之间的相似度引入一些非线性。换句话说，它可以学到一些更复杂的词语交互，而简单的点积（线性变换）可能会忽略它们。

看看现在的注意力热图是什么样子。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/16.webp)

现在我们已经将注意力得分转化为清晰的概率，是时候将它们用于一些有用的事情了。

具体来说：我们将**结合其他单词的信息**——使用**V 向量**。

## 步骤 4. 注意力分数与值向量 (V) 的矩阵乘法

现在我们已经得到了注意力概率，是时候使用它们了。我们通过执行最后一次矩阵乘法来实现这一点：**Softmax 矩阵 × 值矩阵 (V)**。

它看起来类似于我们之前进行的 Q × Kᵀ 乘法，但在这里，我们不是在计算相似度，而是在***收集信息***，并将注意力概率用作**权重**。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/17.webp)

### 我们再画一个点积图！

我们之前已经讲解过矩阵乘法，例如 Q × Kᵀ。这里的运算机制相同。例如：

- 自注意力 softmax 矩阵的维度为 **\[n × n\]**：每行对应一个查询，每列对应一个键。
- V 矩阵的维度为 **\[n × dv\]**：每行对应一个键，每个键都是一个 dv 维向量。
- 如果我们将这些矩阵相乘，我们将得到一个维度为 **\[nx dv\]** 的结果矩阵。换句话说，每个词对应一个向量。

操作非常简单：

- 对于 softmax 矩阵中的每个单词（行），取该行的每个元素，与 V 向量中的第一个元素相乘。
- 把它们相加，得到新词向量的第一个元素。
- 对所有列重复此操作，构建出这个 \[n x dv\] 向量。

在下图所示的例子中，你可以看到我们如何计算 `Vcake` 向量的第一个元素的值。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/18.webp)

### 最后这一步代表什么？（以 Netflix 为例）

⚠️ 非常重要：得到的向量是***不是概率***。事实上，您可以在图中看到这一点 → 我们计算出的数字“2.4”完全正确。

（输出结果也可能为负数。）softmax 函数得到的概率值控制着 V 对每个词的贡献程度。

**🎬 可以把它想象成浏览 Netflix：**

- **注意力概率**就像你对电影的**感兴趣程度**。也许 25% 星球大战、20% 星际穿越、10% 落魄大厨，等等。
- 但**真正的电影**（V 向量）有它们各自的特征：  
    \==> 星球大战 → \[冒险 8，幽默 2，121 分钟\]  
    \==> 星际穿越 → \[冒险 7，幽默 1，169 分钟\]  
    \==> 落魄大厨 → \[冒险 3，幽默 6，114 分钟\]
- 当你用矩阵乘法把这两个矩阵融合在一起，就得到了你的“个人电影”心情：  
    \==> 个人心情 = 0.25\*(8,2,121) + 0.20\*(7,1,169) + 0.10\*(3,6,114) + … = (6.9 冒险，2.4 幽默，137 分钟)

**📝 回到我们的小例子：**

- 从注意力概率来看，我们知道“cake”与“it”相关，但与“and”无关。
- V 向量现在代表类似信息卡的内容。例如，V 中的每个向量都可以表示诸如 \[复数、第三人称、名词、动词、实体 等\] 这样的维度。  
    \==> I → \[复数 = 0，第三人称 = 0，名词 = 0，动词 = 0，实体 = 人类…\]  
    \==> ate → \[复数 = 0，第三人称 = 0，名词 = 0，动词 = 1，实体 = 动作…\]  
    \==> cake → \[复数 = 0，第三人称 = 0，名词 = 1，动词 = 0，实体 = 甜点…\]

当我们把注意力权重乘以这些隐藏特征时，我们就**融合了含义**。

“cake”的输出向量现在可能包含以下痕迹：

- 作为一种实体（甜点），
- 被吃掉（动词），
- 被“it”指代（代词连用），
- 甚至可能带有积极的情感（来自“delicious”）。

*这就是 Transformer 如何将全局上下文混合到每个 token 中的方式。*

## 自注意力的简要回顾

如果你看到了这里，恭喜你（也谢谢你）。让我们快速回顾一下刚才看到的内容。

**Q 和 K 的线性变换，以及自注意力的 4 个步骤：**

- **线性变换：**每个输入词嵌入通过学习矩阵投影到三个新向量：**Q**（查询）、**K**（键）和**V**（值）。

**步骤 1. 相似度评分（Q 和 Kᵀ 的 MatMul）：** 将每个词的查询向量与所有键进行比较，以计算它应该对每个其他词给予多少关注。

- **步骤 2 和 3. 缩放和 Softmax：**原始相似度分数经过缩放（以避免 Softmax 爆炸），然后通过 Softmax 生成干净、稳定的**注意力概率**。
- **步骤 4. 加权聚合（MatMul 与 V）：** 最后，使用每个词的注意力分布来融合值向量——为每个词构建一个新的、**上下文丰富的表示**。

### ✨ 最后两个要点：

🔧 **这些矩阵和向量中的数字是从哪里来的？**

记住，LLM 的最终目标是尽可能准确地预测下一个词。通过足够的示例和足够的训练时间，模型会调整所有这些权重和数值，从而提高预测正确下一个词的概率。

我们今天所讲授的所有矩阵乘法和向量运算都只是**框架**，而*学习*（权重的调整）才是使系统智能化的关键。

⚡ **所有这些事情同时发生！**

LSTM 的一大问题是速度——它们必须一次处理一个词元，一步一步地进行。Transformer 模型改变了这一现状。得益于矩阵乘法和大规模并行化，所有这些操作——跨越所有词——都可以同时进行。如果没有这种结构，训练大型语言模型几乎是不可能的。

## 通过多个头部来扩展注意力

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/19.jpg)

到目前为止，我们构建的所有机制——Q、K、V、缩放点积、加权平均值——对每个词都只执行一次。但实际上，Transformer 模型会同时运行多个注意力机制。

这叫做**多头注意力**。

### 为什么要设置多个头？

每个注意力头都会学习关注词语之间**不同类型的关系**。例如​​：

- 一个头可能学会追踪**代词指代**（“it”→“cake”），
- 另一个头可能侧重于**动宾关系**（“ate”→“cake”），
- 还有一个头可能捕捉到**情感线索**（“delicious”→“cake”）。

通过让多个头并行运行，Transformer 可以对句子获得**更丰富的理解**。

### 它是如何运作的？

每个头都运行我们上面描述的过程，然后所有头连接在一起。矩阵连接实际上就是构建更宽的矩阵（捕获更多特征）。例如：

- 如果每个头的输出是 4 维的（d<sub>v</sub> = 4），
- 而我们有 8 个头（h = 8），
- 拼接之后，每个单词的向量就变为 **4 × 8 = 32 维**。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@a5c07d8353f9cd0c9fa8e934276b7412805ad5be/ai-insights/2026-06/24/images/understanding-transformers-part-4-attention-is-just-a-few-matrices/20.webp)

## 现在，我想听听你们的意见！

📢 在这篇文章中，我们详细分析了 Transformer 如何真正**构建意义**——不仅仅是通过传递嵌入，而是通过自注意力机制*混合*整个句子的信息。

我们一路梳理了：

- 如何创建查询、键和值，
- 如何衡量相似性，
- 为什么需要缩放和 softmax？
- 以及上下文向量是如何通过最终的加权聚合形成的。

全部内容都运用了矩阵、直观的例子和大量的视觉元素，使之具体化。

但现在，我很想听听你们的意见：

- 这是你第一次真正深入了解**自注意力**吗？
- 用 Netflix 电影做类比，是否帮你把数学和真实直觉联系了起来？
- 又或者，你突然灵光一闪——“啊哈！”——意识到多头注意力并非魔法，只是并行的矩阵运算？

请在评论区留下您的想法、疑问，甚至是任何困惑——我很想知道您对这篇文章的理解（或者没有理解！）。👇

## 延伸阅读

感谢阅读本文！如果您对我的其他文章感兴趣，这里有一篇文章汇总了我所有的其他博客文章，并按主题进行了分类：数据科学团队和项目管理、数据故事讲述、营销和竞标科学以及机器学习和建模。

## 敬请关注！

如果您想在我发布新文章时收到通知，欢迎在 Medium 上关注我。此外，[我非常乐意在 LinkedIn 上与您交流](https://medium.com/data-science-collective/www.linkedin.com/in/joseparrenogarcia)！
