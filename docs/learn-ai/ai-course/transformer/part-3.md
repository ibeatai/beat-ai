---
title: "理解 Transformer（第三部分）：位置编码和词嵌入"
author: Jose Parreño
url: https://medium.com/data-science-collective/understanding-transformers-part-3-positional-encodings-and-word-embeddings-3e3802f11a81
translated: 2026-06-28
excerpt: 为什么位置很重要？正弦波和余弦波如何拯救了 Transformer
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/01.thumb.webp
---

# 理解 Transformer（第三部分）：位置编码和词嵌入

为什么位置很重要？正弦波和余弦波如何拯救了 Transformer

本周我将继续上次的话题：走出 RNN 和 LSTM 的世界，深入到从 ChatGPT 到 GitHub Copilot 等一切背后的架构：**Transformer**。

但在*开始*讨论注意力头或解码器栈之前，我们得先解决两个看似简单、实则不然的问题：

- Transformer 是如何理解一个词的*含义*的？
- 它又是如何知道一个词*在句子里的位置*的呢？

在本文中，我们将拆解**词嵌入**如何为每个 token 赋予有意义的“指纹”——以及为什么仅靠它还不够。然后再探索**位置编码**：把顺序注入一个一次性处理所有 token 的模型的功臣。

最后你会看到，几个正弦波和余弦波如何让 Transformer 做到一件 LSTM 一直内置着的事——记住谁先出现。

让我们一起解读每个 Transformer 最初的几步。🧠✨


## Transformer 热潮。你最近在网上接触到的一切都源于此。

你可能没意识到，但只要用过 ChatGPT、GitHub Copilot 或其他 LLM 厂商的产品，你就已经在用 Transformer 了。

这套架构是**现代 AI 背后的引擎**。而这一切，都始于 2017 年的一句话：

> ***“注意力就是你所需要的一切。”***

这是一个石破天惊的时刻，重塑了我们对序列建模的思考方式。不再有 RNN 那种缓慢的逐步处理，也不再有 LSTM 那样的短期记忆难题。

时至今日，“Transformer”（变形金刚）大概是科技圈里含义最被滥用的词了（抱歉了，擎天柱）。那么，与 RNN、LSTM 相比，Transformer 到底是怎么工作的？这正是 Transformer 系列后续博客要讲的内容。

*PS：有兴趣阅读原文吗？* [*从 arXiv 下载*](https://arxiv.org/abs/1706.03762)*。*

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/01.jpg)

## RNN 和 LSTM 第一部分和第二部分回顾。

在第一部分，我们认识了 RNN——一种带记忆的神经网络。它一次处理一个输入，靠传递隐藏状态来记住过去。可即便是中等长度的序列，它也难以应付，训练还慢得让人头疼。第二部分介绍了 LSTM，一次巧妙的升级：引入门控和双记忆路径，修好了最严重的问题，让序列建模真正能用了——至少在序列变长之前是这样。

两者背后的核心理念是***顺序是天生内置的***。正因为网络必须逐步处理数据，它从设计上就理解了顺序。说*“国王和王后”*和说*“王后和国王”*不是一回事，说*“狗在沙发上”*和说*“沙发压在狗身上”*也不是一回事。

Transformer 则是另一类网络。它一次性处理全部数据，这对速度是好事，但——除非***我们给网络一个理解顺序的工具***，否则它根本意识不到*“沙发压在狗身上”*有多荒谬。

这正是本文的意义所在。

在谈注意力或任何花哨机制之前，得先弄清 Transformer 究竟是怎么知道每个词在哪儿的——就从**词嵌入**和**位置编码**讲起。

## 为什么我们今天只讨论嵌入和位置（而不是直接深入混乱）

先把一件事说清楚。

靠一篇博文**根本不可能**讲清整个 Transformer 架构。硬要这么干的人，多半是在写教科书——要么就是暗暗憎恨自己的读者。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/02.webp)

如图所示，Transformer 由多层运转的部件组成：

- 词嵌入
- 位置编码
- 多头自注意力
- 层归一化
- 前馈层
- 残差连接
- 等等

如果你是新手，这张图看着可能像电影《盗梦空间》里的场景：盒子套着盒子，箭头射向四面八方……而驱动 ChatGPT 的那个东西，就藏在里面某处。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/03.webp)

今天我们讲位置编码（顺带提一下词嵌入）。这两者正好坐在 Transformer 的入口，每一个输入都要先经过它们。在注意力、多头投影这些花哨操作登场之前，它们就是模型理解文本的方式。

那就从 Transformer 流水线的第一步说起：我们为什么一开始就需要词嵌入。

## 在确定位置之前，我们需要嵌入。

机器不懂语言，只懂**数字**。所以第一步，是把每个词转换成模型能处理的东西：一个数字向量。

这种数值表示采用向量形式——一组捕捉词义的数字。这些向量就是所谓的***词嵌入***：紧凑、可学习的表示，刻画了一个词的含义。

可以把词嵌入理解成网络给每个词发的一张“有意义的指纹”。不像独热向量（在那里“王后”和“国王”毫无关联），词嵌入会把相似的词在高维空间里安排得彼此靠近。

### 一个简单的例子

想象每个词都活在一个二维空间里。真实模型里更像是 512 维——但眼下二维就够用了。下面是几个示例向量（数值为求简单是编的）：

```

"dog"   : [0.50, 0.30]
"couch" : [0.10, 0.90]
"king"  : [0.95, 0.20]
"queen" : [0.93, 0.25]
"and"   : [0.40, 0.60]
"on"    : [0.35, 0.62]
```

![如果你想自己摆弄一下这张图，点这个链接！](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/04.webp)

注意“国王”和“王后”靠得很近（离“和”很远），而这两个词之间的距离，又比“沙发”和“狗”更远。也许它在区分人、动物和物体（或者机器自己学到的某种实体关系）。有了词嵌入，网络就能开始推理相似性、类比这些东西了。

这些数字编码是怎么生成的，今天先不讲（那本身就是一个大话题）。眼下你只需记住两点：

1. 词被转换成数字
2. 这些数字本身，给了机器一个起点，让它开始在语义上把词联系起来。

既然已经瞥见了词嵌入层的作用，接下来就设想一个假设场景：位置编码根本不存在。

## 没有位置信息的问题。

> 嵌入告诉你那里**有什么**，但**没告诉你它在哪儿**。

接下来，我们盯住下面这句话：

- *狗坐在沙发上*

多亏了嵌入，模型已经隐约知道**“狗”**和**“沙发”**有点关系。它甚至可能“懂得”，这两者比**“国王”**和**“王后”**更相关——后者在语义上很接近，却活在一个完全不同的主题世界里。

于是模型知道**“狗”**和**“沙发”**是一对。但它知道的也就这么多了。

如果只靠词嵌入，模型并不在乎**“狗”**在**“沙发”**之前还是之后。它会把下面这串东西看成一组飘在空间里的词（语义上成簇，结构上却彼此脱节）：

```
["The, "dog", "sat", "on", "couch"]
```

### LSTM 本身就内置了“位置”信息。

拿 LSTM 来对比。它一次读一个 token，每一步更新隐藏状态。这种缓慢的、一步一步的特性其实是个优点：它**把顺序焊进了模型里**。LSTM 会学到“狗坐在沙发上”很常见，而“沙发坐在狗身上”就……有点怪了。

Transformer *没有*这种优势。

它**一次性**看到所有 token。这对速度和并行是好事，对追踪顺序却是灾难。除非我们另作说明，否则**“couch sat dog on”**和其他任何排列一样“合理”。

### 这就需要用到位置编码了。

它们给每个 token 一种位置感：不只是这个词*是什么*，还有它落在句子里的*什么位置*。这正是那个缺失的要素，让模型能说出：**“狗在沙发之前——而不是反过来。”**

> 所以，词嵌入帮模型知道**“狗”**和**“沙发”**算是朋友（比**“国王”**和**“王后”**更亲近），但真正告诉它**谁先到场**的，是位置编码。

而这一点区别，决定了一切。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/05.jpg)

## 一个糟糕的位置编码示例

### Transformer 究竟是如何利用位置信息的？

好，我们把 Transformer 的架构图请回来！图里可以看到，输入和输出嵌入层是和位置编码层相加的。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/06.webp)

就这么简单，不过是向量相加。我们拿一个最朴素的例子验证一下。

### 简单示例：其中位置编码只是一个全为 0 的向量

假设我们的位置向量全是零。给词嵌入加上一个全 0 向量，什么都不会变，二维图上的点纹丝不动。

```
embedding("couch") + [0, 0] = embedding("couch")
```

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/07.webp)

于是，数学测试过了，语言测试却没过。显然，得往词嵌入里加点东西才行。那么——如果把真实的位置传进编码向量里呢？

### 使用实际位置是个坏主意

看下面的例子。我们把 token 索引加进位置编码向量里。

```
Position 1 → [1, 1]  
Position 2 → [2, 2]  
Position 3 → [3, 3]
...
```

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/08.webp)

看出问题了吗？**你正在*盖过*词嵌入。**

大多数词嵌入都很小，取值通常落在 [0, 1] 区间。可一旦你的位置向量写着 `[200, 200]`，模型就不再关心这个词*是什么意思*……而只盯着它*在哪儿*。语义信号被彻底淹没了。

看下面这张带有新位置的二维图。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/09.webp)

由于这些位置向量**太强**，会发生这么几件事：

1. *“couch”* 原本在 *“dog”* 左边，但因为它移动了 6 格，现在跑到 *“dog”* 右边去了！
2. 想想位置被表示得多强，你会看到 *“couch”* 排在 *“dog”* 之后——这恰好和原句一致。
3. 但由于位置嵌入压过了语义嵌入，网络可能净学些奇怪的东西。

**学习到的位置嵌入真的那么糟糕吗？**

未必——GPT-2 和 BERT 用的就是*可学习的绝对*位置向量，效果很好。真正有害的，是给模型一个**幅度盖过**词嵌入的位置信号，或者模式太简单（比如 `[k,k,k,…]`）。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/10.jpg)

### 💡 那么，好的位置编码应该具备哪些特点呢？

两点：

1. **唯一性**：每个位置必须有独一无二的编码。
2. **平衡**：编码得*足够强*才有意义，但又*不能太强*，以致抹掉词本身的含义。

这正是位置编码必须走的钢丝。那么，什么样的向量能做到？

这时，**正弦和余弦**就登场了。

## 正弦和余弦解法。

### 《注意力就是你所需要的一切》论文中提出了一种复杂但令人惊叹的解决方案。

为了编码位置信息，作者写道：

> *我们使用不同频率的正弦和余弦函数 […] 选择这个函数，是因为我们假设它能让模型更容易学会按相对位置来关注；因为对任意固定偏移量* `k`*，* `PEpos+k` *都可以表示成* `PEpos` *的线性函数。*

并给出了下面这个公式：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/11.webp)

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/12.jpg)

别担心，我们马上会看到，这个解法没那么吓人。

### 建立直觉：正弦/余弦向量是什么样子的

先把上面那些公式忘掉一秒钟。回到高中，想想正弦和余弦函数长什么样。在下图里，我画了：

1. 正弦和余弦的理论曲线。
2. 我把我们的例句放在 x 轴上作参照。
3. 我还用一个红色散点和一条淡淡的竖线，标出了那个理论词位（假设“狗”的位置 = 2）与曲线的交点。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/13.webp)

这样会得到一张类似下面的表：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/14.webp)

把这些画出来，每个词现在都有了一个小小的“波形 ID”。你已经在编码顺序了——而且**很温和**。如果把这个二维 [sin, cos] 向量当作位置编码，最终的二维图会是这样：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/15.webp)

### 层叠波浪：为什么它的神奇之处真的有效

现在，回到论文怎么说。虽然听起来很绕，但实际上比听上去简单多了。

与其只用一对正弦/余弦（就像上面那样），不如**叠很多对**上去——每一对的频率略有不同。下面是一个六维向量的层叠示例（对应我们的玩具短语*“狗坐在沙发上”*）：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/16.webp)

每一对各占位置编码向量的一个独立维度。于是对于 `d_model = 6`，你会得到：

- 3 个正弦波：低频、中频、高频
- 3 个余弦波：分别匹配每个正弦波

它们共同为每个位置构成一道独特而平滑的**特征签名**。

### 需要多少个波？它们的波长分别是多少？

论文里明确，使用的波数由 `dmodel` 决定，也就是每个 token 向量的宽度（比如 2017 年的论文用的是 `dmodel = 512`，而新模型很可能轻松超过这个数）。

根据上面给出的公式：

- 每个**偶数**维 `2i` 配一个正弦；对应的**奇数**维 `2i+1` 配上匹配的余弦。
- 因此 `i` 取遍 `dmodel/2d` 个索引 → 也就是这么多个*频率 / 波长*。
- 例如在我们的 6 维玩具示例里，`dmodel=6`，所以我们造了 3 对波长。

至于波长，如果把分母还原成波长，会得到：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e814aca41658231fd08801d30ad73a098842aadb/ai-insights/2026-06/28/images/understanding-transformers-part-3-positional-encodings-and-word-embeddings/17.webp)

于是，最短波长是 2π（指数为 0 时），最长是 2π⋅10000（指数为 1 时）。这意味着波长按**几何级数**增长，让模型既拿到非常精细、也拿到非常粗粒度的位置信号。比如：

- 高频波长：有的追踪细小的变化（token 到 token）。
- 低频波长：有的捕捉长距离的漂移（token 4 与 token 40）。

### 为什么正弦和余弦解法效果如此出色？

**✅ 1. 唯一位置编码**

多个正弦/余弦函数组合在一起，确保**每个位置都得到一个不同的向量**。

**✅ 2. 可控幅度**

因为正弦和余弦**始终在 -1 到 1 之间**，叠加的总信号永远不会太大。位置向量只是轻轻**推一下**词嵌入，而不是把它推下悬崖。

> ***💡 结果：***
>
> *你保留了词的含义，同时给了它一种身处句子中* 某个位置 *的感觉。*

## 总结一下：Transformer 在开始之前需要什么

在 Transformer 能关注任何东西之前（在它开始计算加权和、或寻找词与词的关系之前），它需要两样东西：

1. **🧠 知道每个词*意味着*什么。** 词嵌入负责这件事。它把每个 token 锚定在一个丰富的语义空间里——“国王”挨着“王后”，“狗”离“猫”比离“沙发”更近。
2. **📍知道每个词*在哪儿*。** 位置编码负责这一件。它注入一种柔和的、有节奏的结构——让模型明白*“狗坐在沙发上”*和*“沙发坐在狗身上”*不是一回事。

妙就妙在，这两路输入只是简单地**加在一起**。没有特殊架构，没有新层，只是一次数学上的轻推。

下一篇博客，我们会开始深入模型看到这些向量*之后*发生的事——也就是注意力终于登场的时刻。

但这一切都不会奏效……**要是我们没教会模型派对是从哪儿开始的！**
