---
title: 混合专家（MoE）LLM
author: Cameron R. Wolfe, Ph.D.
url: https://cameronrwolfe.medium.com/mixture-of-experts-moe-llms-0c77edceceeb
translated: 2026-08-06
excerpt: 从零开始理解 DeepSeek、Grok 和 Mixtral 等模型……
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/01.thumb.webp
---

# 混合专家（MoE）LLM

从零开始理解 DeepSeek、Grok 和 Mixtral 等模型……

![（来自 \[2, 5, 14\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/01.webp)

这篇文章最初发表在[我的 Substack](https://cameronrwolfe.substack.com/p/moe-llms)。

## 介绍

在这个瞬息万变的研究领域中，仅解码器 Transformer 架构一直是大型语言模型（LLM）研究中为数不多的经久不衰的经典架构之一。自[最初的 GPT 模型](https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf)提出以来，该架构一直被广泛使用，除了为提高效率而进行的细微调整外，基本保持不变。然而，对于该架构而言，最有意义的改进之一是混合专家（MoE）层。

> “与密集模型通常能达到的效果相比，使用 MoE 架构可以在模型质量和推理效率之间取得更好的平衡。” *—— 摘自 \[11\]*

基于 MoE 的 LLM 为模型架构引入了稀疏性，使得我们可以在不相应增加计算成本的情况下显著增加模型规模（*以参数总数衡量*）。这种改进已被 Grok \[9\] 和 DeepSeek-v3 \[15\] 等近期模型成功采用，使得探索超大型模型变得更加可行且计算效率更高。在本概述中，我们将了解 MoE 的基本原理，并探讨这一思想近期如何被应用于构建更强大的 LLM。

## LLM 中 MoE 的基础

本概述中我们将研究的基于 MoE 的 LLM 是基于仅解码器 Transformer 架构的。我们在此不赘述该架构的细节，如果您不熟悉，请参阅[这篇文章](https://cameronrwolfe.substack.com/p/decoder-only-transformers-the-workhorse)。仅解码器 Transformer 由重复的模块组成，这些模块包含归一化（例如，层归一化或 [RMS 层归一化](https://arxiv.org/abs/1910.07467)）、掩码多头自注意力机制或前馈变换以及残差连接；详见下文。

![仅解码器 Transformer 架构](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/02.webp)

本节将介绍 MoE 的基本原理。这些解释基于几篇开创性论文：*i)* 提出了标准的 MoE 层；*ii)* 将此思想扩展到 Transformer 架构中。这些论文是：

1. [稀疏门控混合专家层](https://arxiv.org/abs/1701.06538) \[1\]
2. [Switch Transformers](http://switch%20transformers/) \[2\]
3. [稳定且可迁移的混合专家模型（ST-MoE）](https://arxiv.org/abs/2202.08906) \[3\]

有关这些论文和 MoE 架构起源的更详细分析，请参阅[此处](https://cameronrwolfe.substack.com/p/conditional-computation-the-birth)对这些想法的更详细概述。

**简要准备。** 要理解 MoE 和路由算法，我们首先必须了解解码器 Transformer（及其每一层）的输入结构。当然，LLM 以文本作为输入，但文本在 LLM 实际接收之前需要经过大量的处理。首先，文本会被分词（如下所示）——*或者说，被转换成离散的 token 列表*。这些 token 就是单词和子词。LLM 拥有一组固定的、它能够理解并以此训练的 token，这被称为模型的“词汇表”。不同模型的词汇表大小有所不同，但通常包含 64K 到 256K 个 token。

![为 LLM 进行文本分词和矢量化](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/03.webp)

文本转换为 token 后，我们可以对输入中的每个 token 进行向量化。除了词汇表之外，LLM 还包含一个 token 嵌入层，该层存储了词汇表中每个 token 的（已学习的）向量嵌入。我们可以在该层中查找每个 token 的向量，从而形成一个输入矩阵。如果每个 token 嵌入都是 `d` 维的，并且输入中总共有 `C` 个 token，那么该输入矩阵的总大小为 `C` × `d`；详见下文。

![输入矩阵（token 向量）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/04.webp)

Transformer 的每一层——*以及每个 Transformer 模块内的每个子层*——都保持着输入的大小。因此，Transformer 中任何前馈模块或注意力模块的输入（和输出）都是一个相同大小的矩阵！

### 什么是“专家”？

在仅解码器 Transformer 架构中，MoE 的主要修改在于 Transformer 模块的前馈组件。在标准架构中，我们使用一个单一的前馈神经网络——*通常由两个前馈层组成，中间有一个非线性激活函数*——每个 token 都单独通过该网络；详见下文。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/05.webp)

MoE 对这种块结构进行了略微修改。它不再在块的前馈组件中只使用一个前馈网络，而是创建多个前馈网络，*每个网络都有其独立的权重*。我们将这些网络称为“专家”。例如，基于 MoE 的 LLM 的每个前馈子层中可能包含八个独立的专家。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/06.webp)

Transformer 层中的专家可以按上图所示的方式定义。一层中有 `N` 个专家，我们可以使用符号 `E_i` 来指代第 `i` 个专家。

**创建基于 MoE 的 Transformer。** 要创建基于 MoE 的仅解码器 Transformer 架构，我们只需将 Transformer 的前馈层转换为 MoE 层——*或者说专家层*。MoE 层中的每个专家都具有与该层原始前馈网络相同的架构——*我们只是拥有原始前馈网络的多个独立副本*；详见下文。

![向仅解码器 Transformer 模块添加专家（来自 \[2\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/07.webp)

然而，我们无需在 Transformer 的每个前馈层都使用专家。大多数基于 MoE 的 LLM 采用步长 `P`，这意味着每隔 `P` 层会转换为专家层，而其他层则保持不变——*这些是“交错式”MoE 层*。这种方法可以更好地平衡最终模型的性能和效率。

### 路由算法

基于 MoE 的架构的主要优势之一是其效率，但仅仅使用专家并不能提高效率！事实上，在模型的每一层增加专家数量会显著增加模型的参数总数——*以及所需的计算量*。为了提高架构的效率，我们必须对每一层中使用的专家进行稀疏选择！

**选择专家。** 我们考虑一个单独的 token——*用一个 `d` 维 token 向量表示*。我们的目标是选择一个专家子集（大小为 `k`）来处理这个 token。在 MoE 文献中，*我们通常说 token 会被“路由”给这些专家*。我们需要一个算法来计算和优化这个路由操作。

最简单的路由算法是对 token 向量进行线性变换，得到一个大小为 `N`（即专家数量）的向量。然后，我们可以应用 [softmax](https://en.wikipedia.org/wiki/Softmax_function) 函数，在专家集合上形成该 token 的概率分布。我们可以利用这个分布，通过选择分布中排名前 `K` 的专家，来决定将 token 路由到哪些专家。

![计算路由机制的输出](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/08.webp)

这种路由策略在 \[1\] 中被采用，该论文提出了我们今天使用的稀疏 MoE 层结构；参见上文。然而，*这种路由机制并没有明确地鼓励均衡选择专家*。因此，如下文所述，该模型很可能收敛到为每个 token 重复选择相同的少数专家，而不是充分且均匀地利用其专家层。这种现象通常被称为“路由崩溃”。

> “门控网络倾向于收敛到一种状态，在这种状态下，它总是给少数几个专家赋予较大的权重。这种不平衡会自我强化，因为这些受青睐的专家训练速度更快，因此更容易被门控网络选中。” *—— 摘自 \[1\]*

**激活参数。** 由于我们仅选择一部分专家来处理 MoE 层中的每个 token，因此 MoE 文献中存在“激活”参数的概念。简而言之，在处理给定 token 时，只有 MoE 模型总参数中的一小部分——*由每个 MoE 层选择的专家决定*——处于激活状态。因此，MoE 执行的总计算量与激活参数的数量成正比，而不是与参数总数成正比。

### 辅助损失和专家负载均衡

为了鼓励在训练过程中均衡选择专家，我们可以简单地在训练损失中添加一个额外的约束，奖励模型均匀利用每个专家。在 \[1\] 中，这是通过为每个专家定义一个“重要性”分数来实现的。该重要性分数基于路由机制预测的每个专家的概率。

![计算重要性损失的详细步骤（来自 \[1\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/09.webp)

给定一批数据，我们把该批次中所有 token 分配给每个专家的概率相加，得到重要性；参见上文。然后，为了确定这些概率是否均衡，我们可以计算专家重要性得分的[变异系数（CV）](https://en.wikipedia.org/wiki/Coefficient_of_variation)的平方。简而言之，*如果所有专家的重要性得分都相似，则 CV 值会很小；反之亦然*。

由此，我们可以简单地将上述重要性损失添加到标准语言建模损失中，从而得到新的（正则化的）训练目标。这个额外的重要性损失项有助于确保 MoE 在整个训练过程中为专家分配相同的概率。

**负载均衡。** 虽然上述重要性损失很有用，但专家被赋予相同重要性并不意味着 token 会被均匀路由。例如，在下面两种情况下，专家的重要性是相同的：

- 少数几个 token 赋予它们非常高的概率。
- 数量多得多的 token 赋予它们较低的概率。

因此，即使使用重要性损失，分配给每个专家的 token 数量仍然可能非常不均匀，这会导致内存使用过多，并普遍降低 MoE 的效率。

![专家负载均衡损失（来自 \[2\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/10.webp)

为了解决这个问题，我们可以创建一个辅助损失项（如上所示），它同时体现了专家重要性和负载均衡，负载均衡定义为 token 在各专家之间的等额路由。\[2\] 中提出了这样的方法，其中作者创建了一个考虑以下两个量的损失：

1. 分配给每个专家的路由概率比例。
2. 分配给每个专家的 token 比例。

如果我们把这两个量分别存储在各自的 `N` 维向量中，就可以通过对这两个向量做[点积](https://en.wikipedia.org/wiki/Dot_product)运算来创建一个单一的损失项。当专家获得均匀概率和负载均衡时，所得损失最小，从而在一个辅助损失项中同时实现了我们的两个目标！

![路由器 z 损失（来自 \[3\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/11.webp)

**路由器 z 损失。** 上述辅助负载均衡损失在 MoE 文献中被广泛使用，但 \[3\] 中的作者提出了一种额外的辅助损失项，称为路由器 z 损失，它可以进一步提高训练稳定性。路由器 z 损失限制了路由机制预测的 [logits](https://wandb.ai/amanarora/Written-Reports/reports/Understanding-Logits-Sigmoid-Softmax-and-Cross-Entropy-Loss-in-Deep-Learning--Vmlldzo0NDMzNTU3#logits) 的大小——*注意，这里指的是 logits 而不是概率，也就是在应用 softmax 之前*。理想情况下，我们不希望这些 logits 过大。然而，当这些 logits 通过路由器的（指数）softmax 函数传递时，它们可能会变得非常大——*导致*[*舍入误差*](https://en.wikipedia.org/wiki/Round-off_error)*，即使使用全精度（*`float32`*）也会使训练过程不稳定*。

> “路由器以 float32 精度计算专家的概率分布。然而，在最大规模下，我们发现这不足以产生可靠的训练结果。” *—— 摘自 \[3\]*

为了鼓励路由器预测较小的 logits，我们可以使用上面所示的损失项。鉴于此损失仅用于正则化路由器的 logits，而不进行负载均衡，我们通常将路由器 z 损失与 \[2\] 中提出的辅助负载均衡损失结合使用。这两个损失都添加到 LLM 的标准语言建模损失之上；详见下文。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/12.webp)

### 专家容量

由于训练和推理过程中都会做出路由决策，因此 MoE 层中的计算是动态的。然而，当我们观察大多数稀疏模型的实际实现时，会发现它们通常具有静态的批次大小——*这是提高硬件利用率的一个有效技巧*。

![（来自 \[2\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/13.webp)

**专家容量。** 为了规范我们为每个专家设定的固定批次大小，我们可以定义专家容量。专家容量的定义如下所示。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/14.webp)

专家容量定义了一个批次中可以发送给每个专家的最大 token 数量。如果路由到某个专家的 token 数量超过了专家容量，我们就“丢弃”这些多余的 token。更具体地说，我们不对这些 token 进行任何计算，而是让它们的表示通过 Transformer 的残差连接直接传递到下一层。

> “为了提高硬件利用率，大多数稀疏模型的实现都为每个专家设置了固定的批处理大小。专家容量指的是可以路由到每个专家的 token 数量。如果超过此容量，则溢出的 token……将通过残差连接传递到下一层。” *—— 摘自 \[3\]*

专家容量通过容量因子设置来控制。容量因子为 1 表示 token 将在所有专家之间以完全均衡的方式路由。或者，将容量因子设置为大于 1 可以提供额外的缓冲，以适应专家之间 token 数量的不平衡。但是，这样做会带来一定的代价（例如，更高的内存占用和更低的效率）。

![（来自 \[2\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/15.webp)

**如何设置容量因子？** 有趣的是，MoE 模型在相对较低的容量因子下往往表现良好 \[2, 3\]；参见上文。然而，我们需要确保丢弃的 token 数量不要过大（例如，这可以通过经验方法确定），以避免对训练过程产生任何影响。我们还可以针对训练和推理使用不同的容量因子；例如，ST-MoE \[3\] 在训练期间使用 1.25 的容量因子，在评估期间使用 2.0 的容量因子。

### 计算 MoE 层的输出

![计算 MoE 层的输出](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/16.webp)

获取路由器的输出后，我们按如下方式计算最终输出：

1. 将 token 发送给它们选定的专家。
2. 计算专家对这些 token 的输出。
3. 对专家输出进行加权平均，其中权重就是路由器分配给每个专家的概率。

上面的公式形式化地描述了计算单个 token 的 MoE 层输出的过程。该 token 的输出是其 `K` 个激活专家的输出的加权平均值。

**共享专家**是 MoE 文献 \[14, 15\] 中近期提出的一个概念。这个概念很简单：

- 我们有两组专家——*共享专家和路由专家*。
- 所有 token 始终通过共享专家传递。
- token 按照正常的 MoE 路由机制通过路由专家进行传递。

下图展示了共享专家的概念，其中我们可以看到路由仅应用于 MoE 层中的一部分专家。通常，共享专家的数量必须少于路由专家的数量——*增加共享专家的数量会降低 MoE 的稀疏性优势*。

![共享专家与路由专家（来自 \[14\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/17.webp)

使用共享专家的目的是为了最大限度地减少专家之间的信息冗余。通过一组共享专家，*我们可以让网络将共享信息存储在这些专家内部*，而无需在多个不同的专家之间重复存储相同的信息。要计算具有共享专家的 MoE 层的输出，我们只需将共享专家的输出添加到正常的路由输出中；请参见下文。

![计算具有共享专家的 MoE 层的输出](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/18.webp)

### 整合所有组件：带 MoE 层的仅解码器 LLM

![Transformer 中的完整 MoE 模块（来自 \[2\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/19.webp)

上图完整展示了 MoE 层。在 MoE 中，我们修改了标准仅解码器 Transformer 的块结构，用专家层替换了前馈网络。简而言之，该专家层包含原始前馈网络的多个独立副本。值得注意的是，MoE 层中的所有组件——*普通层、专家和路由机制*——都通过梯度下降法联合训练。

对于每个 token，我们可以通过路由机制选择要使用的专家，该机制通常通过对 token 向量进行简单的线性变换来实现。综上所述，MoE 的修改后块结构包含：

- 一个自注意力层。
- 一个残差连接和一个归一化操作。
- 一种路由机制，用于确定将 token 路由到哪些专家。
- 一个具有多个独立前馈网络的专家层。
- 对每个 token 的专家层最终输出应用最终的加法和归一化操作。

除了修改后的模块结构外，Transformer 的架构保持不变。我们同样只将 Transformer 中每隔 `P` 个模块转换为使用 MoE 层——*其他模块保持不变*。有些 MoE 在每一层都使用专家，但在实践中，通常将 `P` 设置为 2、4 甚至 6。这种技巧有助于控制 MoE LLM 消耗的参数总数。

## 使用 MoE 的利弊

现在我们了解了 MoE 的基本原理，可能会问：*为什么我们要使用 MoE 而不是密集模型？* MoE 最大的优势在于其高效性，但这些模型也存在一些明显的缺点。让我们快速了解一下 MoE 一些最重要的优缺点。

**MoE 的优势。** [LLM 受益于规模](https://cameronrwolfe.substack.com/p/llm-scaling-laws)——*更大的模型和更大的数据集带来更好的性能*。然而，扩展 LLM 需要付出代价！MoE 的一个关键优势在于它们能够规避扩展过程中遇到的问题——*它们允许我们以固定的单 token 计算成本来增加模型的大小*。这样，我们就可以训练比仅限于密集模型时更大的模型。在语言建模领域，这些稀疏模型的额外参数和表征能力至关重要。

> “随着 LLM 的日益普及，如何在不相应增加计算资源的情况下提升其性能，是一项至关重要的挑战。” *—— 摘自 \[12\]*

MoE 的计算优势（可以说）在推理阶段最为显著。MoE 模型的总参数数量庞大，因此我们需要足够多的 GPU 来存储这些参数。然而，我们在处理每个 token 时仅使用其中固定比例的参数，这极大地提高了计算效率。在较小的批处理大小下，推理速度更快；而在较大的批处理大小下，吞吐量更高 \[5\]。有趣的是，MoE 的训练效率也更高。例如，Switch Transformer 在使用 MoE 架构后，预训练速度提升了 7 倍 \[2\]；详见下文。

![（来自 \[2\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/20.webp)

**使用 MoE 的缺点。** 尽管有这些优势，MoE 同时也：

- 训练过程中容易出现不稳定情况。
- 难以进行微调（即，由于过拟合问题）。
- 对低精度 / 混合精度训练技术敏感。
- 对超参数设置（例如权重初始化）敏感。

简而言之，要充分发挥 MoE 的优势，需要更多技巧和方法。因此，*MoE 并非在所有情况下都是最佳选择*；例如，如果我们想要针对某个任务微调 LLM，那么密集模型可能更容易一些。然而，如果我们能够正确使用 MoE，它们将带来诸多益处。

## 混合专家语言模型

现在我们已经了解了 MoE 最重要和最基本的概念，接下来让我们深入探讨这些概念在语言建模领域的应用。由于 LLM 受益于规模的扩大，MoE 已被广泛采用，并在 LLM 研究中取得了巨大成功。

### [Mixtral of Experts](https://arxiv.org/abs/2401.04088) \[5\]

![（来自 \[5\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/21.webp)

Mixtral 8×7B（又名 Mixtral of Experts）是基于 MoE 的开源 [Mistral-7B 模型](https://arxiv.org/abs/2310.06825) \[6\] 的扩展，支持英语、法语、意大利语、德语和西班牙语。这两个模型的权重均以 [Apache 2.0](https://fossa.com/blog/open-source-licenses-101-apache-license-2-0/) 许可证开源，并附有相应的 [技术报告](https://arxiv.org/abs/2310.06825)，其中提供了有关模型的详细信息。

Mixtral 将 Mistral 的每一层都转换为一个专家层，每个专家层包含八个专家。每个 token 激活其中两个专家，从而得到一个总共拥有 470 亿个参数和 130 亿个激活参数的模型。该模型的[上下文长度](https://cameronrwolfe.substack.com/i/143156742/what-is-prompt-engineering)为 32K，是其非 MoE 版本上下文长度的四倍。如上图所示，Mixtral 在所有测试中都优于 Mistral，尤其在代码生成、数学运算和多语言基准测试中表现出色，在某些情况下甚至超过了规模更大的 [LLaMA-2–70B](https://arxiv.org/abs/2307.09288) 模型。

![（来自 \[7\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/22.webp)

**Mistral-7B 架构。** Mixtral 8×7B 的基础 LLM 架构是一个仅解码器 Transformer，其架构设置与 Mistral-7B \[6\] 完全一致。与标准的仅解码器 LLM 架构相比，Mistral-7B 做了一些更改：

1. [*分组查询注意力 (GQA)*](https://cameronrwolfe.substack.com/i/142044446/efficient-masked-self-attention) *\[7\]*: 在自注意力头组之间共享键和值投影以提高效率；参见上文。
2. [*滑动窗口注意力 (SWA)*](https://arxiv.org/abs/2004.05150) *\[8\]*: 计算大小为 `W` 的固定窗口上的（掩码）自注意力，以允许 LLM 处理任意长度的序列，并降低推理成本；见下文。

由于我们使用了 SWA，该模型可以使用诸如[滚动缓冲区 / 循环缓存](https://github.com/NVIDIA/TensorRT-LLM/blob/b171e879563ff0ba4eb35b94cf0e59a471e13d80/docs/source/advanced/gpt-attention.md#sliding-window-attention-cyclic-rolling-buffer-kv-cache)之类的技巧来提高 [KV 缓存](https://training.continuumlabs.ai/inference/why-is-inference-important/key-value-cache)的内存效率，或者使用[分块预填充](https://developer.nvidia.com/blog/streamlining-ai-inference-performance-and-deployment-with-nvidia-tensorrt-llm-chunked-prefill/#balancing_prefill_and_decode_phases_with_chunked_prefill)来提升推理速度。Mixtral 8×7B 采用了相同的架构约定。

![（来自 \[6\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/23.webp)

**更多详情。** 如前所述，Mixtral 将 LLM 的每一层都转换为专家层。在每个专家层中，采用了一种简单的路由机制，该机制对每个 token 的线性层前 `K` 个 logits 进行 softmax 运算——*这与本概述开头讨论的路由机制一致*；详见下文。

![（来自 \[5\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/24.webp)

作者在 \[5\] 中提到，Mixtral 模型也经过多语言语料库的预训练，使其能够理解多种语言。如下所示，Mixtral 在多语言基准测试中普遍优于 LLaMA 模型。

![（来自 \[5\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/25.webp)

**路由分析。** 论文最后，\[5\] 的作者对多个领域中专家如何被选中来处理 token 进行了详细分析，以探究是否存在任何可解释的模式。当我们绘制 [The Pile](https://pile.eleuther.ai/) 中不同主题领域分配给不同专家的 token 分布图时，并未发现明显的 token 分配模式；详见下图。

![（来自 \[5\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/26.webp)

然而，MoE 的确展现出一些结构化的行为。例如，Python 代码中的“self”和英语中的“Question”——*即使它们由多个 token 组成*——通常都会被路由给同一个专家。类似地，代码中的缩进 token 通常也会被发送给同一个专家，而连续的序列——*也就是彼此相邻的 token 序列*——通常也会被发送给同一个专家；详见下文。这些结果表明，*i)* 专家并非按主题进行专业化，但 *ii)* MoE 的路由机制确实遵循着与模型输入语法或内容相关的一些结构化行为。

![（来自 \[5\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/27.webp)

**规模扩大。** Mixtral 模型之后，其更大规模的版本 [Mixtral-8×22B](https://mistral.ai/news/mixtral-8x22b/) 也随之发布。该模型拥有 1410 亿个总参数和 390 亿个激活参数，规模约为原始 Mixtral 模型的 3 倍。Mixtral-8×22B 在编码和数学任务方面表现尤为出色，其上下文长度扩展至 64K，并且原生支持[函数调用](https://cameronrwolfe.substack.com/p/teaching-language-models-to-use-tools)。Mixtral-8×22B 相较于其他开放模型的主要优势总结如下。

![（来源）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/28.webp)

### [Grok](https://x.ai/blog/grok-os)（来自 [xAI](https://x.ai/)）\[9\]

尽管目前尚无关于该模型的详细技术报告，但在基于 MoE 的 LLM 中，xAI 的 Grok 是近期最引人注目的例子之一。初始的 Grok-1 模型于 2024 年初发布。研究人员透露，该模型是一个拥有 3140 亿个参数的 MoE，每个 token 有 25% 的权重处于激活状态（即约 700 亿至 800 亿个激活参数）。Grok-1 的架构和[基础模型权重](https://huggingface.co/xai-org/grok-1)已根据 Apache 2.0 许可证开源。然而，这是一个预训练的基础模型，并未提供关于模型后训练过程的详细信息。

![（摘自 \[10\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/29.webp)

**Grok-1.5 \[10\]。** Grok-1 最初发布后不久，其后续版本 Grok-1.5 便发布，该版本具有更强的推理能力和长上下文理解能力。例如，Grok-1.5 在数学和编程相关任务上的表现要好得多；参见上文。

> “该模型能够处理更长、更复杂的提示，同时随着上下文窗口的扩展，仍能保持其指令遵循能力。” *—— 摘自 \[10\]*

Grok-1.5 能够处理长度达 128K token 的序列，并在[大海捞针测试](https://github.com/gkamradt/LLMTest_NeedleInAHaystack)中实现完美检索；详见下文。作者还指出，该模型在给定大量上下文的情况下仍能保持稳固的指令遵循能力，相比纯粹的检索，这[更能说明](https://arxiv.org/abs/2406.10149)其长上下文能力。

![（摘自 \[10\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/30.webp)

鉴于 Grok-1.5 和 Grok-1 的发布时间如此接近，我们可以推断 Grok-1.5 取得的进步是由后训练所驱动的——*在此期间创建不同的预训练基础模型的可能性极小*。

**Grok-2。** 近期，[Grok-2](https://x.ai/blog/grok-2) 发布，其推理、编码和聊天能力均有所提升——*以* [*Chatbot Arena*](https://arxiv.org/abs/2403.04132) *衡量*。Grok-2 还包含其他一些小的改进（例如，工具使用、检索、事实性等），并且其蒸馏版本 Grok-2-mini 也与主模型一同发布。然而，Grok-2 的架构细节尚未公开——*该模型很可能是从零开始训练的，可能基于 MoE，也可能不是*。

### [DBRX](https://arxiv.org/abs/2403.04132)（来自 [Mosaic](https://www.databricks.com/research/mosaic)）\[11\]

![（摘自 \[11\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/31.webp)

DBRX 是 [Mosaic](https://www.databricks.com/research/mosaic) 发布的[一系列开放 LLM](https://cameronrwolfe.substack.com/p/democratizing-ai-mosaicmls-impact) 中的最新模型。该模型发布了两个版本——*基础模型（*[*DBRX base*](https://huggingface.co/databricks/dbrx-base)*）和微调模型（*[*DBRX Instruct*](https://huggingface.co/databricks/dbrx-instruct)*）*——均采用开放许可（即 [Databricks 开放模型许可](https://www.databricks.com/legal/open-model-license)）。DBRX 是一款基于 MoE 的 LLM，其具体规格如下：

- 总共有 1320 亿个参数，其中 360 亿个是激活参数。
- 每个 MoE 层有 16 个专家，每个 token 激活 4 个专家。
- 在 12 万亿 token 的优化文本上进行预训练。
- 预训练效率提高 4 倍。

最值得注意的是，DBRX 是一个“细粒度”的 MoE 模型。换句话说，该模型在每个 MoE 层中使用了更多数量的专家，但每个专家本身更小。作为参考，Mixtral 和 Grok-1 的每个 MoE 层都包含八个专家——*对于任何给定的 token，其中只有两个专家处于激活状态*。通过使用细粒度专家，每个 MoE 层有了更多的专家组合可供选择（具体来说多了 65 倍），\[11\] 中发现这能提升质量。

**训练数据。** DBRX 的预训练数据集非常庞大，但 \[11\] 中的作者也投入了大量精力来提升数据质量。因此，DBRX 的统计训练效率高于通常水平——*训练速度更快，因为我们用更少的 token 就能获得更高的准确率*。更具体地说，\[11\] 中的作者估计，新数据在同等 token 数下的效率提高了 2 倍，这意味着我们只需在一半数量的 token 上训练，就能达到相同的性能水平。这一结论通过单独测试新模型预训练数据的影响（即，使用固定模型和不同的预训练数据）得到了验证。

> “单独来看，更好的预训练数据对模型质量产生了显著影响。我们使用 DBRX 预训练数据，在 1T token 上训练了一个 7B 模型。该模型在 Databricks Gauntlet 测试中达到了 39.0%，而 MPT-7B 为 30.9%。” *—— 摘自 \[11\]*

此外，DBRX 的训练采用了课程学习策略——*预训练数据的混合比例在整个预训练过程中动态变化*。该课程学习策略的细节在[这篇论文](https://arxiv.org/abs/2406.03476)中进行了详细阐述。DBRX 使用的课程学习策略是在训练后期对规模较小的、特定领域的数据集进行上采样，因为这些数据相对于通过网络爬虫获得的数据质量更高。这种简单的课程学习策略被证明能够显著提升模型在复杂基准测试中的性能；详见下文。

![（来源）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/32.webp)

**分词器和上下文窗口。** DBRX 的上下文长度为 32K，并使用 GPT-4 分词器（可通过 [tiktoken](https://github.com/openai/tiktoken) 获取）。据作者称，选择 GPT-4 分词器主要是出于性能考虑。该分词器词汇量庞大，且 token 效率极高，因此能够用更少的 token 表示相同数量的文本，从而自然地提升解码和训练速度。

![（摘自 \[11\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/33.webp)

**效率收益。** DBRX 的提出在预训练效率方面带来了显著提升。除了我们目前了解到的之外，\[11\] 中还提到了其他几个效率提升的来源：

- MoE 架构在较小规模的实验中发现，其训练所需的 FLOPS 减少了 1.7 倍。
- 对[仅解码器架构](https://cameronrwolfe.substack.com/p/decoder-only-transformers-the-workhorse)的其他修改（即 [RoPE](https://cameronrwolfe.substack.com/i/142044446/better-positional-embeddings)、[GLU 激活](https://pytorch.org/docs/stable/generated/torch.nn.GLU.html) 和 [GQA](https://cameronrwolfe.substack.com/i/142044446/efficient-masked-self-attention)）。
- *“更好的优化策略”*。

综合考虑所有数据、架构和优化方面的改进，DBRX 的端到端训练过程所需的计算量比之前模型的预训练流程减少了 4 倍。为了确定这一数值，\[11\] 的作者将 DBRX 的较小版本与其之前的 [MPT-7B 模型](https://www.databricks.com/blog/mpt-7b)进行了比较，发现较小的 DBRX 模型在 [Databricks Gauntlet](https://github.com/mosaicml/llm-foundry) 上取得了类似的性能，同时训练过程中使用的 FLOPS 减少了 3.7 倍。

![（摘自 \[21\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/34.webp)

DBRX 在推理效率方面也有所提升——*在负载测试中，以每用户每秒 150 个 token 计，速度最高可达 LLaMA-2–70B 的 2 倍*。这些测试结果是在使用 [TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) 的[优化服务基础设施](https://www.databricks.com/blog/llm-inference-performance-engineering-best-practices)和 16 位精度下完成的，[速度非常快](https://x.com/natolambert/status/1772999462538887493?s=20)。DBRX 的 MoE 架构也因其相对较少的激活参数而有助于提高推理效率。例如，DBRX 的总参数量和激活参数量都只有 Grok-1 的 40%。

> *“训练混合专家模型非常困难。我们必须克服各种科学和性能方面的挑战，才能构建一个足够稳健的流程，以高效的方式重复训练 DBRX 类模型。” —— 摘自 \[11\]*

由于训练过程中会出现不稳定因素、通信瓶颈等问题，训练 MoE 通常十分困难。然而，DBRX 凭借 \[11\] 中概述的优化预训练策略，在稳定性、效率和性能方面取得了令人瞩目的成果。值得注意的是，这些成果并非源于单一的更改或改进。*DBRX 出色的预训练流程是由大量细微而实用的改进所实现的*。

**实证评估。** 与其他开放 LLM 相比，DBRX-Instruct 在综合基准测试中表现远优于 Mixtral；详见下文。尽管 DBRX 是一款通用 LLM，但它拥有令人印象深刻的编程能力，其性能甚至超过了 Grok-1（规模是它的两倍多！）以及像 [CodeLLaMA-70B](https://ai.meta.com/blog/code-llama-large-language-model-coding/) 这样的专用编码模型。DBRX 在推理和数学任务中也表现出色。

![（摘自 \[11\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/35.webp)

与封闭模型相比，DBRX 的性能超过了 GPT-3.5，并且与 [Gemini-1.0 Pro](https://cameronrwolfe.substack.com/p/google-gemini-fact-or-fiction) 不相上下。Gemini-1.0 Pro 仅在 [GSM8K](https://huggingface.co/datasets/gsm8k) 上优于 DBRX，而 [Mixtral-Medium](https://docs.mistral.ai/guides/model-selection/#mistral-medium-intermediate-tasks-that-require-language-transformation) 在所考察的少数几个任务上表现更佳；详见下文。总体而言，DBRX 似乎擅长编程、数学、通用知识、常识推理以及检索 / [RAG](https://cameronrwolfe.substack.com/p/a-practitioners-guide-to-retrieval)。

![（摘自 \[21\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/36.webp)

### [OpenMoE：开放式混合专家语言模型的早期尝试](https://arxiv.org/abs/2402.01739) \[12\]

尽管 MoE 在语言建模领域取得了成功，但真正开源的 MoE——*意味着代码、信息、数据、权重等全部公开共享*——数量相对较少。为了解决这个问题，OpenMoE \[12\] 开展了一项大规模的工作，训练了一系列参数量从 6.5 亿到 340 亿不等的仅解码器 MoE LLM。这些模型采用了不同粒度的细粒度专家（即 16 个或 32 个专家）。这项工作的成果记录在 \[12\] 中，所有模型均已公开共享。作者还提供了一个[文档齐全的代码库](https://github.com/XueFuzhao/OpenMoE)，可用于复现他们的结果。

> “每层都使用 MoE 会在路由过程中增加计算开销，并且与交错使用 MoE 相比，成本效益权衡更差。” *—— 摘自 \[12\]*

**设计选择。** OpenMoE 模型采用了 ST-MoE \[3\] 的设置，包括相同的路由机制和激活专家数量（即 `k = 2`）。作者选择仅将每隔四个或六个 Transformer 模块转换为 MoE 层，发现更大的步长可以在成本和效率之间取得更好的平衡。

OpenMoE 使用的预训练数据集包含大量代码。事实上，在预训练初期，代码占数据集的 50% 以上，但由于该比例并非最优，因此在后续训练中进行了调整；详见下文。为了进行对齐，OpenMoE 在预训练后进行 SFT——*使用来自* [*WildChat*](https://arxiv.org/abs/2405.01470) *的数据*——以提升指令遵循能力。

![（来自 \[12\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/37.webp)

**路由动态。** OpenMoE 的主要贡献之一是对模型内部的路由决策进行详细分析。首先，我们发现——*与先前工作 \[5\] 中的结果类似*——专家并不倾向于专注于任何特定领域；详见下文。

![（来自 \[12\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/38.webp)

但是，我们确实看到自然语言和特定任务之间存在一定程度的专家专业化，如下图所示。

![（来自 \[12\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/39.webp)

然而，当我们深入研究这一趋势时，会发现 token 路由的动态主要由 token ID 决定。换句话说，同一个 token 几乎总是会被路由到同一个专家，*无论该 token 处于何种上下文中*。这种模式在 \[12\] 中被称为“上下文无关的专业化”。

> “这是一个非常有趣的发现，因为具有相同 token ID 的 token 在不同的句子中具有非常不同的上下文。例如，token“an”也可以是“an apple”或“another”的一部分。然而，所有这些 token 都高度集中在少数几个固定的专家上。” *—— 摘自 \[12\]*

有趣的是，专家们偏好的 token 具有可观察的模式；如下所示。例如，“have”、“has”和“had”都被路由给同一个专家，而“=”、“and”和“\\n”这些 token 则由另一个专家接收——*这些都是编程语言中非常常见的 token*。我们在 \[12\] 中看到，这种路由模式在预训练的早期阶段就已经确定，并且在后续的训练中很少改变。

![（来自 \[12\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/40.png)

**路由问题。** 除了 \[12\] 中观察到的路由模式之外，我们还发现 OpenMoE 模型存在一些可能损害其性能的路由行为。例如，这些模型倾向于丢弃序列后段的 token，这可能会损害长序列任务（例如，多轮聊天）的性能。

![OpenMoE 模型在多轮聊天问题上的表现较差（来自 \[12\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/41.webp)

由于路由动态在预训练过程的早期阶段就已经确定，因此这些行为在后训练阶段很难修正。事实上，OpenMoE 模型通常难以应对预训练数据和 SFT 数据之间的领域差异——*由于数据组成不同，token 路由动态变得不稳定*。为了解决这些问题，\[12\] 的作者建议将指令遵循数据混入预训练数据集中。

**模型评估。** 总体而言，OpenMoE 模型在 MoE LLM 中并未达到新的最先进水平——*\[12\] 中的作者公开指出了这一点，并承认 OpenMoE 模型的性能可以通过更好的设计得到显著提升*。OpenMoE 模型更大的贡献在于其透明度。\[12\] 中公开分享的细节和成果可以为进一步研究 MoE 提供必要的资源，从而加速这一主题上的开放研究。

### [DeepSeek-v2](https://arxiv.org/abs/2405.04434) \[14\] 和 [DeepSeek-V3](https://arxiv.org/abs/2412.19437) \[15\]

最近提出的 DeepSeek MoE 模型，包括 DeepSeek-v2 \[14\] 和 DeepSeek-v3 \[15\]，由于各种原因在 LLM 研究界引起了轰动：

- 它们的权重是公开的。
- 它们附带技术报告，其中包含许多细节信息。
- 它们的性能令人印象深刻——*与许多封闭模型不相上下*。
- 它们的训练成本相当合理。

我们将看到，DeepSeek 模型做出了各种独特的设计选择，从而最大限度地提高了训练效率和下游性能。

**DeepSeek-v2 \[14\] ——** *一个拥有 2360 亿参数、其中 210 亿个激活参数的 MoE*——提出了后续 DeepSeek-V3 模型所使用的 MoE 架构。DeepSeek MoE 模型与之前的模型略有不同，它们对底层 Transformer 模块做了少量修改，以提升性能和效率。如下所示，DeepSeek-v2 模型——*在性能优异之外*——在训练和推理效率方面也相当出色，使其成为规模更大的 DeepSeek-v3 模型的坚实起点。

![（摘自 \[14\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/42.webp)

**多头潜在注意力（MLA）。** DeepSeek-v2 没有采用标准的多头注意力，而是采用了 MLA，这是一种高效的注意力变体。与[多查询注意力](https://arxiv.org/abs/1911.02150)或[分组查询注意力](https://arxiv.org/abs/2305.13245)类似，MLA 的目标是最小化模型 [KV 缓存](https://dipkumar.dev/becoming-the-unbeatable/posts/gpt-kvcache/)所消耗的内存。然而，与其他高效注意力变体不同，MLA 的性能损失并不显著。

![（摘自 \[14\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/43.webp)

具体来说，这种内存效率的提升是通过低秩联合投影实现的，它允许我们用一个小得多的（潜在）向量来表示所有的键向量和值向量；参见上文。我们可以对这个向量进行上采样——*只需将其线性投影形成几个更大的向量*——来恢复完整的键值向量，但我们只需要在 KV 缓存中存储潜在向量，从而大幅降低内存消耗。*与拥有 670 亿参数的密集模型相比，采用 MLA 可以将 DeepSeek-v2 的 KV 缓存大小减少 93% 以上*。

**DeepSeek MoE 架构。** 除了使用 MLA 之外，DeepSeek 模型还采用了一种独特的 MoE 层结构。与 DBRX 类似，这些模型使用细粒度专家。然而，其中一部分专家是共享的。采用这种结构的目的是鼓励更多专家进行专业化分工，同时最大限度地减少专家之间的信息冗余。DeepSeek 模型使用的模块结构的完整示意图如下所示。

![（摘自 \[14\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/44.webp)

\[14\] 中的作者也采用了一种有趣的负载均衡策略来处理 DeepSeek-v2 使用的细粒度专家。除了使用 \[2\] 中提出的辅助负载均衡损失之外，DeepSeek-v2 还包含两个辅助损失项，旨在平衡分布式训练期间设备间的通信。

![设备级负载均衡辅助损失（来自 \[14\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/45.webp)

使用细粒度专家意味着我们必须将每个 token 分发给更多专家。在分布式训练环境中，专家可能位于不同的设备上，并且每个设备上可能驻留着多个专家。为了确保设备间的通信和计算均衡，我们需要额外的辅助损失，它们 *i)* 按专家所在的设备对专家进行分组，*ii)* 鼓励 MoE 在每个设备上执行均衡路由。例如，上面所示的辅助损失鼓励设备间的均衡*计算*。\[14\] 中还提出了一种额外的损失，用于鼓励设备间的均衡*通信*。

![（摘自 \[15\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/46.webp)

**DeepSeek-v3 \[15\]** 是 DeepSeek-v2 大得多的版本，拥有 6710 亿个总参数和 370 亿个激活参数。该模型在一个包含 14.8 万亿 token 的庞大语料库上进行了预训练。预训练完成后，会应用一个多阶段的后训练流程：

- 该模型首先经过两阶段的上下文扩展流程，先通过 SFT 微调到 32K 的最大上下文长度，然后进一步微调到 128K 的上下文长度。
- 在扩展上下文之后，该模型将进行进一步的 SFT 和 RLHF，以使其符合人类偏好。
- 最近提出的 [R1 推理模型](https://arxiv.org/abs/2501.12948)的能力也在后训练阶段被蒸馏到 DeepSeek-v2 中。

最终的 DeepSeek-v3 模型性能优于闭源模型，甚至与最好的闭源 LLM 性能相当；参见上文。DeepSeek-v3 还对 MoE 的训练和负载均衡策略进行了多项改进，使得模型的训练过程既高效又稳定。

> “尽管 DeepSeek-V3 性能卓越，但其完整训练仅需 278.8 万 H800 GPU 小时。此外，其训练过程极其稳定……我们没有遇到任何不可恢复的损失尖峰，也未进行任何回滚操作。” *—— 摘自 \[15\]*

DeepSeek-v3 的架构借鉴了其前代；例如，MLA、细粒度专家和共享专家都被 DeepSeek-v3 采用。然而，与 DeepSeek-v2 不同的是，DeepSeek-v3 使用了**多 token 预测（MTP）**训练目标。该目标是对监督式、基于交叉熵的[下一个 token 预测目标](https://cameronrwolfe.substack.com/i/136638774/understanding-next-token-prediction)的扩展，后者几乎被普遍用于训练 LLM。MTP 并非为序列中的每个 token 预测下一个 token，而是预测 `D` 个未来 token。这些预测由添加到模型架构中的一组附加模块按顺序完成；详见下文。

![（摘自 \[15\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/47.webp)

一旦预测出多个未来 token，我们就可以正常应用交叉熵损失。将此损失应用于通过 MTP 预测的多个未来 token，可以为模型提供更丰富的训练信号，从而提高训练效率和整体性能。此外，这些用于 MTP 的额外模块还可以通过[推测解码](https://pytorch.org/blog/hitchhikers-guide-speculative-decoding/)来提高推理效率。然而，\[15\] 的作者指出，MTP 策略纯粹是为了提升模型性能——*额外的模块在训练完成后会被丢弃*。

![无辅助损失的负载均衡策略（来自 \[15\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/48.webp)

DeepSeek-v3 还采用了一种**无辅助损失的负载均衡**策略，该策略只是在选择前 `K` 个专家时，为每个专家添加一个偏置项；参见上文。在每次迭代中，每个专家的偏置项会根据该专家是负载不足还是负载过高，按固定因子 `γ` 相应地增加或减少。重要的是，这些偏置仅在选择前 `K` 个专家时使用——*它们不会影响路由器内部专家概率的计算*。研究发现，这种方法能够有效地平衡 MoE 中的专家利用率，并消除因使用负载均衡损失而导致的性能下降。然而，\[15\] 的作者指出，他们在训练 DeepSeek-v3 时仍然使用了辅助负载均衡损失（但缩放因子非常低）。

![（摘自 \[15\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/49.webp)

**训练效率。** 由于上述策略带来的效率和性能优势，DeepSeek-v3 非常经济。此外，该模型采用一种新颖的 FP8 混合精度训练框架进行训练，*这标志着 8 位训练首次在大规模 LLM 上得到验证*。最终模型的训练总成本估计约为 560 万美元；详见上文。简而言之，DeepSeek-v3 具有以下特点：

- 训练方式非常经济（并且带来了 FP8 训练和 MTP 等多项创新！）
- 对于一个开放模型来说令人印象深刻——*即使与最好的闭源 LLM 相比也极具竞争力*。
- 基于一种有趣的 MoE 架构，并进行了一些创新性的改进。

**推理能力。** DeepSeek-v3 也是 [DeepSeek-R1](https://arxiv.org/abs/2501.12948) \[13\] 的基础模型，DeepSeek-R1 是一个近期发布的开放推理模型。简而言之，R1 是 OpenAI 近期探索的 [o1 型模型](https://openai.com/index/learning-to-reason-with-llms/)的开源复现。正如其详细技术报告中所述，该模型使用纯粹的强化学习，通过构建极其长的思维链来学习如何解决复杂的（可验证的）推理任务。如下图所示，R1 的性能相当出色，尤其对于一个开放模型而言。然而，如果没有一个能力极强的基础模型，R1 的这些能力是不可能实现的。

![（摘自 \[13\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/mixture-of-experts-moe-llms/50.webp)

## 最后想说的话

MoE 具有许多特别适合语言建模的优势。它们能够在不大幅增加计算量的情况下探索更大的规模，降低训练成本，并且可以高效地部署。尽管稀疏性的概念在机器学习文献中由来已久，但 MoE 是稀疏性一种特别有影响力的实现。它们以一种与现代硬件兼容的方式利用稀疏性，并且可以在 GPU 上实际实现。有趣的是，早期的 MoE 变体由于其复杂性、不稳定性以及使用困难而难以推广。然而，我们在本文中看到的进展已经使 MoE 成为一种实用且影响深远的技术——*一种简单且前景广阔的仅解码器 Transformer 架构扩展*。

### 首次订阅电子报？

大家好！我是 [Cameron R. Wolfe](https://cameronrwolfe.me/)，深度学习博士，也是 [Netflix](https://research.netflix.com/research-area/nlp-and-conversations) 的高级研究科学家。这篇文章转载自我的 [Deep (Learning) Focus](https://cameronrwolfe.substack.com/) 时事通讯，我在其中帮助读者更好地理解 AI 研究领域的重要课题。如果您喜欢这份时事通讯，欢迎订阅、考虑付费订阅、分享，或在 [X](https://twitter.com/cwolferesearch)、[LinkedIn](https://www.linkedin.com/in/cameron-r-wolfe-ph-d-04744a238/) 和 [Medium](https://wolfecameron.medium.com/) 上关注我！

### 参考书目

\[1\] Shazeer, Noam 等人。“规模庞大的神经网络：稀疏门控混合专家层。” arXiv 预印本 arXiv:1701.06538 (2017)。

\[2\] Fedus, William, Barret Zoph 和 Noam Shazeer。“Switch transformers：以简单高效的稀疏性扩展到万亿参数模型。”机器学习研究杂志 23.120 (2022): 1–39。

\[3\] Zoph, Barret 等人。“St-moe：设计稳定且可迁移的稀疏专家模型。” arXiv 预印本 arXiv:2202.08906 (2022)。

\[5\] Jiang, Albert Q. 等人。“Mixtral of experts。” arXiv 预印本 arXiv:2401.04088 (2024)。

\[6\] Jiang, Albert Q. 等人。“Mistral 7B。” arXiv 预印本 arXiv:2310.06825 (2023)。

\[7\] Ainslie, Joshua 等人。“GQA：从多头检查点训练通用多查询 Transformer 模型。” arXiv 预印本 arXiv:2305.13245 (2023)。

\[8\] Beltagy, Iz, Matthew E. Peters 和 Arman Cohan。“Longformer：长文档 Transformer。” *arXiv 预印本 arXiv:2004.05150* (2020)。

\[9\] xAI。“Grok-1 的开放发布”（2024）。

\[10\] xAI。“宣布 Grok-1.5”（2024）。

\[11\] Mosaic Research (Databricks)。“DBRX 简介：一种新的最先进的开放式 LLM”（2024）。

\[12\] Xue, Fuzhao 等人。“Openmoe：开放混合专家语言模型的早期尝试。” *arXiv 预印本 arXiv:2402.01739* (2024)。

\[13\] Guo, Daya 等人。“DeepSeek-R1：通过强化学习激励 LLM 中的推理能力。” *arXiv 预印本 arXiv:2501.12948* (2025)。

\[14\] Liu, Aixin 等人。“Deepseek-v2：一种强大、经济、高效的混合专家语言模型。” *arXiv 预印本 arXiv:2405.04434* (2024)。

\[15\] Liu, Aixin 等人。“Deepseek-v3 技术报告。” *arXiv 预印本 arXiv:2412.19437* (2024)。
