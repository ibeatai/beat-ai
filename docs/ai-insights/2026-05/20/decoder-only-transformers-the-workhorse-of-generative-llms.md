---
title: "仅解码器 Transformer：生成式 LLM 的主力架构"
author: Cameron R. Wolfe, Ph.D.
url: https://cameronrwolfe.medium.com/decoder-only-transformers-the-workhorse-of-generative-llms-66841d7a2a9c
translated: 2026-05-20
tags:
  - Artificial Intelligence
excerpt: 当下 AI 研究的步伐令人瞠目。跟上最新的论文很难，连领域内的专家都会觉得自己没能掌握这个不断演化的前沿里那些更细微的细节。LLM 领域尤其如此，有影响力的研究层出不穷：新的基础模型（如 Gemma \[15\] 和 OLMo \[12\]）、更好的对齐技术（DPO \[32\]、PPO \[33\]、REINFORCE \[34\] 之争），还有 模型合并 这类相对冷门的话题。…
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/01.webp
---

# 仅解码器 Transformer：生成式 LLM 的主力架构

## 从零开始构建世界上最有影响力的神经网络架构……

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/01.webp)
*（来自 [1, 8]）*

本文最初发表于 [我的 Substack](https://cameronrwolfe.substack.com/p/decoder-only-transformers-the-workhorse)。

## 引言

当下 AI 研究的步伐令人瞠目。跟上最新的论文很难，连领域内的专家都会觉得自己没能掌握这个不断演化的前沿里那些更细微的细节。LLM 领域尤其如此，有影响力的研究层出不穷：新的基础模型（如 Gemma \[15\] 和 OLMo \[12\]）、更好的对齐技术（DPO \[32\]、PPO \[33\]、REINFORCE \[34\] 之争），还有 [模型合并](https://www.interconnects.ai/p/model-merging) 这类相对冷门的话题。但在这些飞快的进展之中，LLM 的一个组件始终没变——*仅解码器 transformer 架构*。令人吃惊的是，大多数现代 LLM 用的架构和最初的 GPT 模型几乎一模一样。我们只是把模型做得大得多，稍作修改，再用一套更庞大的训练（和对齐）流程。正因如此，仅解码器 transformer 架构是 AI 研究中最基础、最重要的思想之一。本文将全面讲解这个架构，从零实现它的每一个组件，并探讨它在近期研究中如何演化。

## 自注意力操作

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/02.webp)
*（来自 [1]）*

transformer 架构出自一篇题为 *"Attention Is All You Need"*（注意力就是你所需要的全部）\[1\] 的论文，因此自注意力是所有现代语言模型的核心，这大概不会让人意外。简单说，自注意力会根据序列中某个 token 与其他 token 的关系，来变换它的表示，见上图。但这究竟是怎么运作的？本节将一步步讲清楚自注意力背后的概念，并用 PyTorch 实现 LLM 所用的那个自注意力变体。

### 理解缩放点积注意力

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/03.webp)

> "一个注意力函数把一个查询和一组键-值对映射到一个输出，其中查询、键、值和输出都是向量。输出是值的加权和，每个值的权重由查询与对应键之间的兼容性函数算出。" *— 来自 \[1\]*

**投影输入。** 自注意力层的输入就是一批 token 序列，序列里每个 token 用一个向量表示。设批大小为 `B`、每个序列长度为 `T`，那么自注意力层接收的输入就是一个形状为 \[ `B, T, d]` 的 [张量](https://pytorch.org/tutorials/beginner/basics/tensorqs_tutorial.html)，其中 `d` 是 token 向量的维数。为简单起见，我们先只用一个 token 序列作为输入来讲解自注意力操作，见下图；同样的概念也能轻松推广到一批序列。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/04.webp)
*以列表和矩阵形式表示的 token 向量序列*

自注意力的第一步，是对输入序列里的 token 向量做三次独立的线性投影，分别得到键、查询和值向量序列。具体做法是用三个权重矩阵——*分别对应键、查询、值的投影*——去投影每一个输入 token 向量，得到新的、变换过的 token 向量序列。因为做了三次，最终我们得到三个独立的 token 向量序列，见下图。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/05.webp)
*创建查询、键和 token 向量*

**计算注意力分数。** 投影完输入后，就用键向量和查询向量来生成注意力分数。我们为序列中每一对 token `[i, j]` 算出一个注意力分数 `a[i, j]`。注意力分数落在 `[0, 1]` 区间内，定量地刻画了在计算 token `i` 的新表示时，应该考虑 token `j` 多少。实际计算时，`a[i, j]` 就是 token `i` 的查询向量与 token `j` 的键向量的 [点积](https://en.wikipedia.org/wiki/Dot_product)，见下图。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/06.webp)
*从查询和键向量计算注意力分数*

要高效地算出序列中所有成对的注意力分数，可以把查询向量和键向量各自堆叠成矩阵，再用查询矩阵乘以转置后的键矩阵。结果是一个 `[T, T]` 大小的矩阵——*我们把它叫做注意力矩阵*——里面包含了序列中所有成对的注意力分数。接下来，把注意力矩阵里每个值都除以 `d` 的平方根（*这种做法被发现能改善训练稳定性 \[1\]*），再对矩阵的每一行做 [softmax 操作](https://en.wikipedia.org/wiki/Softmax_function)，见下图。softmax 之后，每个 token 的注意力分数都落在 `[0, 1]` 区间内，构成一个有效的概率分布。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/07.webp)
*计算注意力矩阵*

**值向量。** 有了注意力分数，推导自注意力的输出就很容易了。每个 token 的输出就是值向量的加权组合，权重由注意力分数给出。要批量计算这个输出，只需把所有值向量堆成一个矩阵，再用注意力矩阵乘以值矩阵。值得注意的是，自注意力保持输入的大小不变——*输入里每个 token 向量都对应一个变换后的、* `d` *维的输出向量*。如果把这个矩阵乘法手写展开，就会看到每个 token 的输出表示其实就是值向量的加权平均，权重由注意力分数给出。

### LLM 的因果自注意力

上面描述的自注意力操作构成了 transformer 架构的基础。不过，transformer 的解码器用的是一个稍复杂的版本，叫做掩码多头自注意力。下面先讲清楚掩码自注意力和双向自注意力的区别，再讨论注意力如何在多个"头"上并行计算。

**掩码自注意力。** 仅解码器 transformer 用的是一种叫掩码（或因果）自注意力的变体。普通的双向自注意力——*前一节描述的那种*——允许在算注意力分数时考虑序列里的所有 token；而掩码自注意力则修改了底层的注意力模式，把序列中排在某个 token 之后的那些 token "掩掉"。举个例子，考虑 token 序列 `["LLM", "#s", "are", "cool", "."]`，假设我们要为 token `"are"` 计算注意力分数。前面学到，自注意力会在 `"are"` 和序列中每一个其他 token 之间都算一个分数。但用掩码自注意力，我们只为 `"LLM"`、`"#s"` 和 `"are"` 计算分数。*掩码自注意力不允许我们在自注意力过程中向序列前方张望*。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/08.webp)
*在因果自注意力中掩盖注意力分数*

掩码自注意力在实现上和双向自注意力很像。查询矩阵和键矩阵相乘后，会得到一个 `[T, T]` 大小的注意力矩阵，其中每个 token 都有它在整个序列上的注意力分数。但在对这个矩阵每一行做 softmax 之前，我们可以把对角线之上的所有值都设成负无穷，见上图。这样一来，softmax 之后，对每个 token 而言，序列中排在它之后的所有 token 都会拿到零分。换句话说，*我们掩盖每个 token 的注意力分数，把序列中任何未来的 token 都排除在外*。

**注意力头。** 到目前为止描述的注意力操作，用 softmax 来归一化整个序列上算出的注意力分数。这种做法虽然能形成有效的概率分布，却也限制了自注意力同时聚焦序列中多个位置的能力——*概率分布很容易被一个（或几个）词主导*。为解决这个问题，我们通常在多个"头"上并行计算注意力，见下图。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/09.webp)
*（来自 [1]）*

每个头内部的掩码注意力操作都是一样的。但我们会做两件事：*i)* 为每个头使用独立的键、查询、值投影；*ii)* 减小键、查询、值向量的维度，把计算成本控制在合理范围内。通常我们把这些向量的维数从 `d` 改成 `d // H`，其中 `H` 是注意力头的数量。这样，每个注意力头都能学到一个独特的表示子空间，聚焦底层序列的不同部分。同时，减小每个头所用向量的维度，也避免了多出来的计算开销。

最后，多头自注意力还有一个细节要考虑：*怎么把各个头的输出合起来？* 选项有不少（拼接、平均、投影等）。不过多头自注意力的普通实现通常是：

-   拼接每个头的输出。
-   对拼接后的输出做一次线性投影。

因为每个注意力头输出的 token 向量维度是 `d // H`，所有头拼接后的输出维度就是 `d`，和注意力层的输入维度一样。

### 在 PyTorch 中实现因果自注意力

[https://gist.github.com/wolfecameron/26863dbbc322b15d2e224a2569868256#file-causal\_self\_attention-py](https://gist.github.com/wolfecameron/26863dbbc322b15d2e224a2569868256#file-causal_self_attention-py)

如果前面的讨论都理解了，掩码多头自注意力的实现（完整代码见上）应该相当容易看懂。首先，我们用 PyTorch 里一个简单的线性层来做键、查询、值的投影。所有自注意力头的键、查询、值投影都能用单个线性层完成：这一层接收维度为 `d` 的 token 嵌入序列作为输入，产出大小为 `3 * d` 的 token 嵌入。然后把输出拆成 `d` 维的键、查询、值向量序列。接着把每个 `d` 维向量重塑成 `H` 个更小的向量——*每个注意力头一个*——再 [转置](https://en.wikipedia.org/wiki/Transpose) 这个张量，得到形状为 `[B, H, T, d // H]` 的输出，其中 `B` 是当前批次里的序列数量，见下方代码。

```
q, k, v  = self.c_attn(x).split(self.d, dim=2)
k = k.view(B, T, self.H, self.d // self.H).transpose(1, 2)
q = q.view(B, T, self.H, self.d // self.H).transpose(1, 2)
v = v.view(B, T, self.H, self.d // self.H).transpose(1, 2)
```

接下来，用基本的矩阵/张量乘法，就能在每个头内部、在整个批次上算出所有 token 之间的注意力分数。先把查询张量乘以键矩阵的转置，得到大小为 `[B, H, T, T]` 的未归一化注意力矩阵。再把结果除以 `sqrt(d)`，并在最后一个维度上做 softmax，把每个 token 在序列上的注意力分数变成一个概率分布。不过在 softmax 之前，要先把注意力矩阵对角线之上的所有条目填成负无穷，见下方代码。

```
att = (q @ k.transpose(-2, -1)) * (1.0 / math.sqrt(k.size(-1)))
att = att.masked_fill(self.mask[:,:,:T,:T] == 0, float('-inf'))
att = F.softmax(att, dim=-1)
att = self.attn_dropout(att)
```

我们还可以选择对注意力分数做 dropout \[2\]，这已被证明能正则化训练过程、改善泛化。算出注意力矩阵后，把它与值矩阵相乘，就能推导出自注意力的最终输出——这一步会根据注意力分数为每个 token 取值向量的加权平均。结果是一个 `[B, H, T, d // H]` 大小的张量，只要把它转置并重塑成 `[B, T, d]`，就能拼接各个注意力头的输出，见下方代码。

```
y = att @ v
y = y.transpose(1, 2).contiguous().view(B, T, self.d)
y = self.resid_dropout(self.c_proj(y))
```

最后，对拼接后的输出再做一次线性投影（可选地带 dropout），就得到了最终结果，如上面的代码所示。

## 仅解码器 Transformer 块

仅解码器 transformer 架构由若干个结构相同的"块"按序堆叠而成。每个块内部有两个主要组件：

1.  掩码多头自注意力。
2.  一个前馈变换。

此外，我们通常用一个残差连接和一个归一化层把这些组件包起来。本节将更详细地讨论这个块结构，并给出 PyTorch 里的具体实现。

### 层归一化

尽管 [高性能 GPU](https://www.nvidia.com/en-us/data-center/h200/) 和模型架构的进步可能让人产生错觉，但 *训练大型、深层的神经网络从来都不轻松*！早期训练多层神经网络的尝试大多失败了，原因是 [梯度消失、爆炸和不稳定](http://neuralnetworksanddeeplearning.com/chap5.html) 的问题。为解决这些问题，人们提出了若干进展：

-   更好的权重初始化方法（如 [Xavier](https://www.geeksforgeeks.org/xavier-initialization/) 或 [He](https://www.geeksforgeeks.org/kaiming-initialization-in-deep-learning/) 初始化）。
-   用 [ReLU](https://pytorch.org/docs/stable/generated/torch.nn.ReLU.html) \[5\] 替换 [sigmoid](https://pytorch.org/docs/stable/generated/torch.nn.Sigmoid.html) 激活函数（这能让激活函数里的梯度不至于变得非常小）。
-   归一化神经网络的中间激活值 \[6\]。

本节聚焦上面提到的最后一项进展——*归一化*。归一化背后的动机相当简单。深层神经网络的中间激活值可能变得不稳定（极大或极小），因为我们反复地用模型参数矩阵去乘它们。比如运行下面的 PyTorch 代码片段就会发现，连续重复同一个随机矩阵乘法很多次，会让输出的值大到难以置信！

[https://gist.github.com/wolfecameron/f9cb6645dc87a165ce3a7fae980610a4#file-exploding\_activations-py](https://gist.github.com/wolfecameron/f9cb6645dc87a165ce3a7fae980610a4#file-exploding_activations-py)

要解决这个问题，可以在每两次矩阵乘法之间归一化激活值，让激活值始终保持稳定。这正是神经网络里归一化层所用的思路。下面来看几种流行的归一化变体。

**归一化变体。** 视所用的领域和架构不同，可以采用若干种归一化技术。两种最常见的形式是：

-   批归一化（Batch Normalization）\[6\]
-   层归一化（Layer Normalization）\[7\]

这两种技术相当相似。它们都用下面的等式来变换激活值，区别只在于均值和标准差的计算方式。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/10.webp)
*归一化的标准等式*

批归一化——*顾名思义*——在整个 mini-batch 上按维度计算均值和标准差，见下图。这种做法效果不错，但有个局限：必须处理一个足够大的输入 mini-batch，才能可靠地估计出均值和方差。这在推理时会成为问题，因为推理时常常一次只处理少量输入样本。所以我们必须在训练期间维护一个均值和标准差的运行估计，留给推理时用。尽管如此，批归一化仍被广泛使用，是计算机视觉应用中归一化技术的标准选择。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/11.webp)
*（来自 [8]）*

层归一化在输入的最后一个维度上计算均值和标准差，从而摆脱了批归一化对批维度的依赖。对仅解码器 transformer 来说，这意味着我们在嵌入维度上计算归一化统计量，见上图。

目前，批归一化常用于计算机视觉任务，层归一化则是自然语言处理任务的标准。原始 transformer 架构在实现里采用了层归一化 \[10\]，此后这个选择一直是 transformer 的标准。不过，层归一化在 transformer 提出之前就已被更早的语言模型用过了——*那些基于循环神经网络的模型*。

**仿射变换。** 深层网络里的归一化层通常还会搭配一个仿射变换。这听起来复杂，其实只是说我们按下面的等式来修改层归一化。归一化激活值之后，再把它乘以一个常数 γ、加上一个常数 β。这两个常数都是可学习的，和普通模型参数一样处理。此外，下图可见层归一化在分母里用了一个稍作修改的标准差形式，里面加入了一个小的固定常数 ε，以避免除以零的问题。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/12.webp)
*带仿射变换的层归一化*

PyTorch 已经实现了层归一化，可以通过 [对应的模块](https://pytorch.org/docs/stable/generated/torch.nn.LayerNorm.html) 或它的 [函数式形式](https://pytorch.org/docs/stable/generated/torch.nn.functional.layer_norm.html) 轻松调用。

### 前馈变换

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/13.webp)
*一个逐点前馈变换的示意图*

每个仅解码器 transformer 块都包含一个逐点前馈变换，见上图。这个变换让输入里的每一个 token 向量都通过一个小型前馈神经网络。这个网络由两个线性层构成——*偏置可选*——中间被一个非线性激活函数隔开。网络的隐藏维度通常大于它接收的 token 向量维度——*在 GPT \[3\]、GPT-2 \[4\] 和许多其他 LLM 里要大 4 倍*。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/14.webp)
*SwiGLU 激活函数*

**激活函数。** *LLM 的前馈层该用哪个激活函数？* 在 \[13\] 中，作者比较了众多激活函数的性能，发现在固定计算量下，SwiGLU 激活（如上图）性能最好。正因如此，[LLaMA-2](https://cameronrwolfe.substack.com/p/llama-2-from-the-ground-up) \[11\] 和 [OLMo](https://cameronrwolfe.substack.com/p/dolma-olmo-and-the-future-of-open) \[12\] 等流行 LLM 普遍使用 SwiGLU。但并非所有 LLM 都用它；比如 Falcon \[14\] 和 Gemma \[15\] 用的是 [GeLU](https://pytorch.org/docs/stable/generated/torch.nn.GELU.html)。

[https://gist.github.com/wolfecameron/3ed9274a0297aab403b5e2d2254ee0ac#file-transformer\_ffnn-py](https://gist.github.com/wolfecameron/3ed9274a0297aab403b5e2d2254ee0ac#file-transformer_ffnn-py)

**在 PyTorch 中的实现。** 实现 transformer 块的前馈组件很简单，见上方代码。只需几个 [线性层](https://pytorch.org/docs/stable/generated/torch.nn.Linear.html)，中间夹一个激活函数。在上面的实现里，一个 `[B, T, d]` 大小的输入被送进第一个线性层，它的输入维度是 `d`、输出维度是 `h = 4 * d`。第一个线性层把输入里所有 `d` 维向量都乘以一个 `d x h` 大小的矩阵，做一次 [批量矩阵乘法](https://pytorch.org/docs/stable/generated/torch.bmm.html)，得到一个 `[B, T, h]` 大小的输出。然后对这个输出应用非线性激活函数，再送进下一个线性层，它的输入维度是 `h`、输出维度是 `d`。最后，可以选择对第二个线性层的输出（大小为 `[B, T, d]`）应用 dropout，在训练时正则化模型。

### 残差连接

我们通常在 transformer 块的自注意力子层和前馈子层之间加上残差连接。残差连接的概念最早由 [ResNet 架构](https://arxiv.org/abs/1512.03385) \[16\] 提出，ResNet 是一个被广泛使用、相当著名的卷积神经网络架构，用于图像分类、目标检测等计算机视觉任务。残差连接在概念上很好理解：我们不是简单地把神经网络激活值传过网络里的某一层，而是 *i)* 存下这一层的输入，*ii)* 算出这一层的输出，*iii)* 把这一层的输入加到它的输出上，见下图。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/15.webp)
*一个带残差连接的通用神经网络层*

残差连接是一个通用思路，可以用在任何不改变输入维度的神经网络层上。加上残差连接，我们就能缓解梯度消失和爆炸的问题，也能让训练过程整体上更顺畅、更稳定。残差连接相当于提供了一条"捷径"，让梯度在反向传播时能自由流过网络。残差连接的好处在深度学习文献中已被广泛探索和分析，由此衍生出关于它们用处的各种有趣直觉 \[17, 18, 19\]。

### 把它们全部拼到一起！

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/16.webp)
*一个仅解码器 transformer 块*

要构建一个完整的仅解码器 transformer 块，我们必须用上前面谈过的所有组件：

-   掩码多头自注意力
-   层归一化
-   逐点前馈变换
-   残差连接

仅解码器 transformer 块的布局如上图所示。后面会看到，块的具体布局可能因实现而异。不过上图与大多数 GPT 风格 LLM 所用的仅解码器 transformer 块的普通结构一致。下面用 PyTorch 实现了这同一个块结构。

[https://gist.github.com/wolfecameron/0ad044748283c90b4d3002bdc5dbc674#file-decoder\_only\_block-py](https://gist.github.com/wolfecameron/0ad044748283c90b4d3002bdc5dbc674#file-decoder_only_block-py)

## 仅解码器 Transformer

现在来看完整的仅解码器 transformer 架构，它主要由前面见过的构件组成。不过还有几个额外细节要讲，比如怎么构造模型的输入、怎么用模型的输出来预测/生成文本。和自注意力相比，这些细节相对简单，但讲清楚它们，才能完整看到仅解码器 transformer 架构是如何运作的。

### 构造模型的输入

如前所述，transformer 块的输入应当是一个（成批的）token 向量序列，通常是一个形状为 `[B, T, d]` 的张量。但 LLM 通常接收的是一个文本提示形式的输入。*怎么把文本提示转换成 token 向量序列呢？*

**分词。** transformer 接收原始文本作为输入。处理文本的第一步是分词，也就是把它转换成一系列离散的词或子词。这些词和子词通常叫做 token，见下图。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/17.webp)
*把原始文本转换成一个 token 序列*

分词过程由模型的分词器完成，它用字节对编码（Byte-Pair Encoding，BPE）\[20\]、SentencePiece \[21\] 或 WordPiece \[22\] 这类算法把文本拆成 token 序列，更多细节见 [此处](https://huggingface.co/docs/transformers/en/tokenizer_summary)。分词器有一个固定大小的词表——*通常包含大约 50K 到 300K 个唯一的 token*——它定义了能从一段原始文本中切出的那组已知 token。分词器有自己的训练流水线来推导出底层词表，通常还实现两个主要函数：

-   *Encode*：把一个字符串转换成一个 token 序列
-   *Decode*：把一个 token 序列转换成一个字符串

分词是 LLM 训练和使用中常被忽视的一环。但 *不去考察、不去理解一个 LLM 的分词过程，是一个巨大的错误*！分词是创建模型输入的第一步，因此对下游模型的性能影响极大。LLM 出的问题常常能追溯到分词过程里某些难以察觉的微妙 bug。所以我强烈建议有兴趣的读者深入钻研分词过程。要对 BPE 分词器——*LLM 最常用的分词器*——有一个深入而实用的了解，可以看 [Andrej Karpathy](https://karpathy.ai/) 最近发布的视频，见 [此处](https://www.youtube.com/watch?v=zduSFxRajkE)。

**Token 嵌入。** 对文本分完词、得到 token 序列后，我们必须把每个 token 转换成一个对应的嵌入向量。为此我们创建一个 [嵌入层](https://pytorch.org/docs/stable/generated/torch.nn.Embedding.html)，它是仅解码器 transformer 模型的一部分。这个嵌入层就是一个有 `d` 列、`V` 行的矩阵，其中 `V` 是分词器词表的大小。词表里每个 token 都关联一个整数索引，对应嵌入矩阵里的一行。要把一个 token 转换成 `d` 维嵌入，只需在嵌入层里查出该 token 的那一行，见下图。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/18.webp)
*在一个嵌入层里查找 token 嵌入*

这个嵌入层会在 LLM 的训练过程中和其他模型参数一样被训练！token 嵌入不是固定的，而是从数据中学出来的。

**位置嵌入。** 现在我们已经把原始文本转换成了一个 token 向量序列。对一整批文本序列都这么做，就得到了 transformer 块所需的 `[B, T, d]` 大小的输入。但还有最后一步要做——*位置嵌入*。

> "由于我们的模型既不含循环也不含卷积，为了让模型能利用序列的顺序，我们必须注入一些关于序列中 token 相对或绝对位置的信息。" *— 来自 \[1\]*

研究自注意力机制时会注意到，计算输出时并没有考虑每个 token 在序列中的位置！可文本序列里词的顺序显然很重要（比如 *"I have to read this book."* 和 *"I have this book to read."*）。因此我们需要某种方式，把位置信息注入到自注意力过程中。在 \[1\] 中，做法是给模型输入里的每个 token 加上一个维度为 `d` 的位置嵌入。因为序列里每个位置都有一个唯一的位置嵌入，每个 token 的位置就能被区分出来。

和 token 嵌入类似，位置嵌入可以存在一个嵌入层里，并在 LLM 的训练过程中从数据里学出来——*这种做法实现起来很简单*。或者，也可以用某个规则或等式生成固定的位置嵌入。在 \[1\] 中，位置嵌入是用正弦和余弦函数生成的，如下图所示。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/19.webp)
*用正弦和余弦函数生成位置嵌入（来自 [1]）*

这类做法被称为"绝对"位置嵌入策略，因为所用的嵌入由 token 在序列中的绝对位置决定。本文后面会看到，绝对位置嵌入策略无法泛化到比训练时所见更长的序列，正因如此，人们提出了泛化能力更强的策略。

### 完整的仅解码器 Transformer 模型

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/20.webp)
*一个仅解码器 transformer 模型的结构*

构造好模型的输入后，只需把它传过一连串仅解码器 transformer 块，见上图。transformer 块的总数取决于模型的规模，比如 OLMo-7B \[12\] 有 32 层，OLMo-65B 有 80 层，见下图。transformer 块保持输入大小不变，因此模型主体的输出——*包括所有 transformer 块*——是一个与输入大小相同的 token 向量序列。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/21.webp)
*OLMo LLM 的规格（来自 [12]）*

增加底层 LLM 里 transformer 块/层的数量，是扩大模型规模的主要方式之一。另一种方式是增大 `d`（即 token 嵌入的维度）的值，这会增大模型里所有注意力层和前馈层的权重矩阵。如上图所示，我们通常同时增加两样东西来扩大仅解码器 transformer 的规模：*i)* 层的数量，*ii)* 隐藏维度。我们也常常增加每个注意力层里头的数量，但只要假设每个注意力头的维度都是 `d // H`，这就不会影响模型的参数量。

**分类头。** 最后，仅解码器 transformer 架构还有一个细节要考虑。把输入序列传过模型主体后，我们得到的输出是一个大小相同的 token 向量序列。要生成文本或做下一个 token 预测（这个过程的更多细节见 [此处](https://cameronrwolfe.substack.com/p/language-model-training-and-inference)），我们要把每个 token 向量转换成一个在候选下一个 token 上的概率分布。为此，可以在模型末尾再加一个线性层，它的输入维度是 `d`、输出维度是 `V`（即词表的大小），充当一个分类头，见下图。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/22.webp)
*给仅解码器 transformer 添加一个分类头*

有了这个线性层，我们就能把输出里的每个 token 向量转换成一个在 token 词表上的概率分布。基于这个概率分布，我们可以做两件事：

-   *下一个 token 预测*：LLM 的预训练目标，用一个 [交叉熵损失函数](https://pytorch.org/docs/stable/generated/torch.nn.CrossEntropyLoss.html) 训练模型，为输入序列里的每一个 token 预测它的下一个 token。
-   *推理*：基于模型生成的 token 分布，自回归地采样出最佳的下一个 token。

[https://gist.github.com/wolfecameron/f574c5c9a61f3b3a045b2cbd9593cfd7#file-gpt-py](https://gist.github.com/wolfecameron/f574c5c9a61f3b3a045b2cbd9593cfd7#file-gpt-py)

**完整架构（在 PyTorch 中）。** 仅解码器 transformer 架构的完整实现见上方代码。既然这个架构的每个组件都已经讨论过，上面的代码应该相对直观。唯一做的修改是：

1.  在把 token 嵌入和位置嵌入作为输入送进第一个仅解码器 transformer 块之前，先对它们做 dropout。
2.  添加一个最后的层归一化模块，在分类头之前对仅解码器 transformer 块的输出做归一化。

把输入传过所有仅解码器 transformer 块后，我们有两种选择。一是把所有输出 token 嵌入都传过线性分类层，从而在整个序列上应用一个下一个 token 预测损失（预训练时的做法）。二是只把最后一个输出 token 嵌入传过线性分类层，从而采样出要加入模型生成输出的下一个 token（推理时的做法）。

## 该架构的现代变体

理解了仅解码器 transformer 架构之后，我们就可以看看现代 LLM 所用的一些变体。大多数情况下，仅解码器 transformer 的核心细节都被保留了下来。但近期对生成式 LLM 兴趣的激增，催生了对仅解码器 transformer 的各种有用修改，这些修改能改善性能、提升速度（训练和推理都包括）、让训练过程更稳定、让模型能处理更长的输入序列，等等。

### Transformer 块布局

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/23.webp)
*（来自 [1]）*

前面见到的仅解码器 transformer 块布局，是标准的 transformer 块配置。但块里归一化操作的顺序可能因实现而异。比如上图可见，在原始 transformer 架构 \[1\] 里，层归一化操作被画在注意力层和前馈层之后。此外，*有些架构在两个位置都做归一化*；比如 Gemma \[15\] 对每个 transformer 子层的输入和输出都做归一化，下面会解释。

> "我们对每个 transformer 子层的输入和输出都做归一化，这偏离了只归一化其中之一的标准做法。" *— 来自 \[15\]*

**并行块。** 文献里也探索过一些替代的块结构。比如 Falcon \[14\] 和 PaLM \[24\] 用的是并行 transformer 块结构，它把输入并行地传过注意力层和前馈层，而不是按序传过，见下图。这种做法减轻了分布式训练的通信成本，而且两个模型都发现它不会带来可察觉的性能退化。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/24.webp)
*（来自 [14]）*

### 归一化策略

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/25.webp)
*RMSNorm 的公式*

除了改变 transformer 块里归一化层的具体位置，所用的归一化策略也因模型而异。大多数模型用层归一化，但 **均方根层归一化（Root Mean Square Layer Normalization）\[29\]**（简称 [RMSNorm](https://github.com/bzhangGo/rmsnorm)！）也很流行。RMSNorm 的公式如上图所示，它其实就是层归一化的一个简化版本，已被证明能改善训练稳定性和泛化。而且 RMSNorm 在表现相近的情况下，比层归一化高效 10–50%，正因如此，LLaMA \[30\] 和 LLaMA-2 \[11\] 等模型采用了这种做法。

**更好的层归一化。** 再进一步，某些 LLM 采用了层归一化的修改形式。比如 MPT \[26\] 模型用 [低精度层归一化](https://docs.mosaicml.com/projects/composer/en/latest/method_cards/low_precision_layernorm.html) 来改善训练时的硬件利用率，不过这种做法在罕见情况下可能引发损失尖峰。类似地，许多 LLM（如 OLMo \[12\]、LLaMA-2 \[11\] 和 PaLM \[24\]）去掉了层归一化里的偏置项，见下图。事实上，*这些模型中有不少还彻底去掉了 transformer 所有层里的偏置*！去掉 transformer 里的偏置项，能维持甚至改善 LLM 的性能，还带来了一点（适度的）加速。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/26.webp)
*（来自 [12]）*

### 高效的（掩码）自注意力

自注意力虽然是 transformer 架构的基础，这个操作却有些低效——*它是一个* `O(N^2)` *操作*！正因如此，人们提出了大量高效的注意力变体，[Reformer](https://arxiv.org/abs/2001.04451)、[SMYRF](https://arxiv.org/abs/2010.05315)、[Performer](https://arxiv.org/abs/2009.14794) 只是其中几个。这些技术中有不少在理论上把自注意力的复杂度降到了 `O(N)`，但 *实际上拿不到可衡量的加速*。为解决这个问题，**FlashAttention \[25\]** 以一种高效、IO 感知的方式重新表述了自注意力操作，见下图。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/27.webp)
*（来自 [25]）*

FlashAttention 的内部运作大多与硬件相关，更多细节见 [此处](https://shreyansh26.github.io/post/2023-03-26_flash-attention/)。但它的成果是一个可以直接替换自注意力操作的方案（drop-in replacement），带来了各种了不起的好处：

-   把 BERT-large 的训练时间加速 15%。
-   为 LLM 启用更长的上下文长度（因为内存效率更好）。

在 [PyTorch 2.0 发布](https://pytorch.org/blog/accelerated-pytorch-2/) 之后，缩放点积注意力——*也就是本文学到的那个自注意力变体*——可以被替换成 FlashAttention 来提升效率！正因如此，许多近期的 LLM（如 Falcon \[14\] 和 MPT \[26\]）都用 FlashAttention。而且这个领域仍有活跃的研究在发表，催生了一些有趣的进展：

1.  [FlashAttention-2](https://arxiv.org/abs/2307.08691)：修改 FlashAttention，在效率上更进一步。
2.  [FlashDecoding](https://pytorch.org/blog/flash-decoding/)：FlashAttention 的一个扩展，在训练效率之外，还聚焦于改善推理效率。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/28.webp)
*多查询注意力在注意力头之间共享键和值投影（来自 [1]）*

**多查询和分组查询注意力。** 在 FlashAttention 之外，若干近期的 LLM（如 [Gemini](https://cameronrwolfe.substack.com/p/google-gemini-fact-or-fiction) \[27\]、Falcon \[14\] 和 PaLM \[24\]）使用多查询注意力，这是一种高效的自注意力实现，让一层里的所有注意力头共享键和值投影，见上图。它不是为每个头做一次独立投影，而是让所有头共用同一个键的投影矩阵、同一个值的投影矩阵。这个改变并不会让训练变快，但能显著提升所得 LLM 的推理速度。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/29.webp)
*（来自 [28]）*

可惜的是，多查询注意力可能带来轻微的性能恶化，于是一些 LLM（如 LLaMA-2）转而寻找替代方案。分组查询注意力（GQA）\[28\] 不是让所有注意力头共享全部键和值投影，而是把 `H` 个自注意力头划分成若干组，组内共享键/值投影，见上图。这种做法是普通多头自注意力与多查询注意力之间的一个折中——后者在全部 `H` 个头之间共用一个键和值投影。有意思的是，*GQA 既维持了普通多头因果自注意力的性能，又达到了与多查询注意力相当的效率*。

### 更好的位置嵌入

> "我们发现，使用正弦位置嵌入的 transformer 语言模型（LM）外推能力非常弱。" *— 来自 \[31\]*

前面学到的位置嵌入技术，用的是由每个 token 在序列中绝对位置决定的加性位置嵌入。这种做法虽然简单，却限制了模型泛化到比训练时所见更长序列的能力。结果就是，如果想在推理时接受更长的输入，我们必须在更长的序列上预训练 LLM（这可能相当昂贵）。正因如此，人们提出了各种替代的位置编码方案，包括只考虑 token 之间距离、而非绝对位置的 [相对位置](https://jaketae.github.io/study/relative-positional-encoding/) 嵌入。这里我们将研究两种最常用的、把位置信息注入 LLM 的策略——*RoPE \[23\] 和 ALiBi \[31\]*。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/30.webp)
*（来自 [23]）*

**旋转位置嵌入（Rotary Positional Embeddings，RoPE）\[23\]** 是绝对位置嵌入和相对位置嵌入的一个混合体，它通过两种方式把位置纳入自注意力：

1.  用一个 [旋转矩阵](https://en.wikipedia.org/wiki/Rotation_matrix) 编码绝对位置。
2.  把相对位置信息直接加进自注意力操作里。

值得注意的是，RoPE 在 transformer 的每一层都注入位置信息，而不只是在模型的输入序列里。这种做法被发现能在绝对位置信息和相对位置信息之间取得平衡，提供了扩展到更长序列长度的灵活性，并且 token 间的依赖性会随相对距离增大而衰减（即相距很远的 token 彼此付出的注意力更少）。RoPE 近来越来越流行，被 PaLM \[24\]、Falcon \[14\]、OLMo \[12\]、LLaMA/LLaMA-2 \[11, 30\] 等流行 LLM 采用！

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/31.webp)
*（来自 [31]）*

**带线性偏置的注意力（Attention with Linear Biases）\[31\]** 是一种后续技术，提出来是为了改善位置嵌入策略的外推能力。ALiBi 不用位置嵌入，而是在 transformer 的每一层向注意力矩阵添加一个静态的、非学习的偏置，从而把位置信息直接纳入自注意力，见上图。我们照常计算注意力矩阵（即查询矩阵与键矩阵的乘积），但给注意力矩阵的值加上一个常数偏置，对相距更远的查询和键之间的分数施加惩罚。实现起来非常容易，只要把这些额外的偏置加到用于计算因果自注意力的注意力掩码上即可。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/decoder-only-transformers-the-workhorse-of-generative-llms/32.webp)
*（来自 [31]）*

尽管做法简单，在外推到比训练所见更长的序列方面，它胜过了普通位置嵌入技术和 RoPE，见上图。而且计算和内存成本并没有显著增加。ALiBi 被 MPT 模型 \[26\] 采用，这些模型经过微调，能支持长达（甚至超过）[65K token](https://huggingface.co/mosaicml/mpt-7b-storywriter) 的输入！

## 要点总结

尽管创新的步伐令人瞠目，仅解码器 transformer 仍然是生成式 LLM 研究的基石。事实上，大多数现代 LLM 所用的模型架构，尽管规模大得多、也在细微处做过修改，大体上仍与最初的 GPT 模型 \[3\] 一致。因此，对任何想更深入理解语言模型内部运作的人来说，建立起对仅解码器 transformer 架构的扎实理解是绝对必要的。基于本文的内容，我们可以把对仅解码器 transformer 模型的理解拆解成以下几个核心思想。

**构造输入。** 仅解码器 transformer 接收一个文本提示作为输入。首先，我们用一个分词器——*基于字节对编码这类算法*——把文本拆成离散的 token。然后，把每个 token 映射到一个存在嵌入层里的对应 token 向量。这个过程形成一个 token 向量序列，作为输入送给模型。可选地，我们还能用加性位置嵌入来增强这些 token 向量。

**因果自注意力** 是仅解码器 transformer 的核心，它让模型能从输入中 token 之间的关系里学习。普通的自注意力操作通过取其他 token 表示的加权组合来变换每个 token 的表示，权重由 token 之间成对的注意力（或重要性）分数给出。因果自注意力遵循类似的策略，但只为序列中在前的 token 计算注意力分数。注意力在若干个头上并行执行，每个头都能聚焦输入序列的不同部分。

**前馈变换** 在仅解码器 transformer 的每个块内部执行，让我们能单独变换每个 token 的表示。这个前馈组件是一个小型神经网络，以逐点的方式应用于每一个 token 向量。给定一个 token 向量作为输入，我们先把它传过一个线性投影，把大小增大约 4 倍，再应用一个非线性激活函数（如 SwiGLU 或 GeLU），然后做另一个线性投影，把 token 向量恢复成原来的大小。

**Transformer 块** 按序堆叠，构成仅解码器 transformer 架构的主体。仅解码器 transformer 块的具体布局可能因实现而异，但有两个主要子层始终存在：

1.  因果自注意力
2.  前馈变换

此外，这些子层被一个层归一化模块——*位于子层之前或之后（或两者都有！）*——和一个残差连接包裹起来。

**分类头。** 仅解码器 transformer 有一个最后的分类头，它接收 transformer 最终输出层的 token 向量作为输入，输出一个与模型分词器词表大小相同的向量。这个向量既可以用来通过下一个 token 预测来训练 LLM，也可以用来在推理时通过核采样（nucleus sampling）、束搜索（beam search）等采样策略来生成文本。

### 第一次看这份简报？

嗨！我是 [Cameron R. Wolfe](https://cameronrwolfe.me/)，深度学习博士，[Netflix](https://research.netflix.com/research-area/nlp-and-conversations) 的资深研究科学家（Staff Research Scientist）。这是从我的 [Deep (Learning) Focus](https://cameronrwolfe.substack.com/) 简报转载的，在那里我帮助读者更好地理解 AI 研究中的重要话题。如果你喜欢这份简报，请订阅、考虑付费订阅、分享它，或者在 [X](https://twitter.com/cwolferesearch)、[LinkedIn](https://www.linkedin.com/in/cameron-r-wolfe-ph-d-04744a238/) 和 [Medium](https://wolfecameron.medium.com/) 上关注我！

### 参考文献

\[1\] Vaswani, Ashish, et al. "Attention is all you need." *Advances in neural information processing systems* 30 (2017).

\[2\] Zehui, Lin, et al. "Dropattention: A regularization method for fully-connected self-attention networks." *arXiv preprint arXiv:1907.11065* (2019).

\[3\] Radford, Alec, et al. "Improving language understanding by generative pre-training." (2018).

\[4\] Radford, Alec, et al. "Language Models are Unsupervised Multitask Learners."

\[5\] Glorot, Xavier, and Yoshua Bengio. "Understanding the difficulty of training deep feedforward neural networks." *Proceedings of the thirteenth international conference on artificial intelligence and statistics*. JMLR Workshop and Conference Proceedings, 2010.

\[6\] Ioffe, Sergey, and Christian Szegedy. "Batch normalization: Accelerating deep network training by reducing internal covariate shift." *International conference on machine learning*. pmlr, 2015.

\[7\] Ba, Jimmy Lei, Jamie Ryan Kiros, and Geoffrey E. Hinton. "Layer normalization." *arXiv preprint arXiv:1607.06450* (2016).

\[8\] Wu, Yuxin, and Kaiming He. "Group normalization." *Proceedings of the European conference on computer vision (ECCV)*. 2018.

\[9\] Ulyanov, Dmitry, Andrea Vedaldi, and Victor Lempitsky. "Instance normalization: The missing ingredient for fast stylization." *arXiv preprint arXiv:1607.08022* (2016).

\[10\] Vaswani, Ashish, et al. "Attention is all you need." *Advances in neural information processing systems* 30 (2017).

\[11\] Touvron, Hugo, et al. "Llama 2: Open foundation and fine-tuned chat models." *arXiv preprint arXiv:2307.09288* (2023).

\[12\] Groeneveld, Dirk, et al. "Olmo: Accelerating the science of language models." *arXiv preprint arXiv:2402.00838* (2024).

\[13\] Shazeer, Noam. "Glu variants improve transformer." *arXiv preprint arXiv:2002.05202* (2020).

\[14\] Almazrouei, Ebtesam, et al. "The falcon series of open language models." *arXiv preprint arXiv:2311.16867* (2023).

\[15\] Google DeepMind (Gemma Team). "Gemma: Open Models Based on Gemini Research and Technology" (2024).

\[16\] He, Kaiming, et al. "Deep residual learning for image recognition." *Proceedings of the IEEE conference on computer vision and pattern recognition*. 2016.

\[17\] Jastrzębski, Stanisław, et al. "Residual connections encourage iterative inference." *arXiv preprint arXiv:1710.04773* (2017).

\[18\] Veit, Andreas, Michael J. Wilber, and Serge Belongie. "Residual networks behave like ensembles of relatively shallow networks." *Advances in neural information processing systems* 29 (2016).

\[19\] Li, Hao, et al. "Visualizing the loss landscape of neural nets." *Advances in neural information processing systems* 31 (2018).

\[20\] Sennrich, Rico, Barry Haddow, and Alexandra Birch. "Neural machine translation of rare words with subword units." *arXiv preprint arXiv:1508.07909* (2015).

\[21\] Kudo, Taku, and John Richardson. "Sentencepiece: A simple and language independent subword tokenizer and detokenizer for neural text processing." *arXiv preprint arXiv:1808.06226* (2018).

\[22\] Wu, Yonghui, et al. "Google's neural machine translation system: Bridging the gap between human and machine translation." *arXiv preprint arXiv:1609.08144* (2016).

\[23\] Su, Jianlin, et al. "Roformer: Enhanced transformer with rotary position embedding." *arXiv preprint arXiv:2104.09864* (2021).

\[24\] Chowdhery, Aakanksha, et al. "Palm: Scaling language modeling with pathways." *arXiv preprint arXiv:2204.02311* (2022).

\[25\] Dao, Tri, et al. "Flashattention: Fast and memory-efficient exact attention with io-awareness." *Advances in Neural Information Processing Systems* 35 (2022): 16344–16359.

\[26\] "Introducing MPT-7B: A New Standard for Open-Source, Commercially Usable Llms." *Databricks*, 5 May 2023, [https://www.databricks.com/blog/mpt-7b.](https://www.databricks.com/blog/mpt-7b.)

\[27\] Google Gemini Team et al. "Gemini: A Family of Highly Capable Multimodal Models", [https://storage.googleapis.com/deepmind-media/gemini/gemini\_1\_report.pdf](https://storage.googleapis.com/deepmind-media/gemini/gemini_1_report.pdf) (2023).

\[28\] Ainslie, Joshua, et al. "GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints." *arXiv preprint arXiv:2305.13245* (2023).

\[29\] Zhang, Biao, and Rico Sennrich. "Root mean square layer normalization." *Advances in Neural Information Processing Systems* 32 (2019).

\[30\] Touvron, Hugo, et al. "Llama: Open and efficient foundation language models." *arXiv preprint arXiv:2302.13971* (2023).

\[31\] Press, Ofir, Noah A. Smith, and Mike Lewis. "Train short, test long: Attention with linear biases enables input length extrapolation." *arXiv preprint arXiv:2108.12409* (2021).

\[32\] Rafailov, Rafael, et al. "Direct preference optimization: Your language model is secretly a reward model." *Advances in Neural Information Processing Systems* 36 (2024).

\[33\] Ouyang, Long, et al. "Training language models to follow instructions with human feedback." *Advances in Neural Information Processing Systems* 35 (2022): 27730–27744.

\[34\] Ahmadian, Arash, et al. "Back to Basics: Revisiting REINFORCE Style Optimization for Learning from Human Feedback in LLMs." *arXiv preprint arXiv:2402.14740* (2024).
