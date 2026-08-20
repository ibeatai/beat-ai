---
title: 简明讲清 Diffusion LLM
author: Dr. Ashish Bamania
url: https://levelup.gitconnected.com/diffusion-llms-explained-simply-4dba963911c3
translated: 2026-05-22
tags:
  - Technology
  - Artificial Intelligence
  - Programming
  - Machine Learning
excerpt: 基于 LLM 的聊天机器人在我们身边随处可见。它们回复时，是按顺序逐步生成内容的。也就是说，它们的输出是一个 token 接一个 token、一次一个地生成出来的。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/01.webp
---

# 简明讲清 Diffusion LLM

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/01.webp)

## 一份温和而又全面的 Diffusion LLM 入门介绍。

基于 LLM 的聊天机器人在我们身边随处可见。它们回复时，是按顺序逐步生成内容的。也就是说，它们的输出是一个 token 接一个 token、一次一个地生成出来的。

![自回归 LLM 中的顺序文本生成](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/02.webp)

这让它们慢得让人难受（*对一名计算机工程师来说，提升执行速度是他们最疯狂的梦想之一！*）。

原因在于，它们底层的架构——Transformer——是一种[自回归](https://en.wikipedia.org/wiki/Autoregressive_model)模型，在文本生成的每一步都回答这样一个简单的问题：

> 给定此前所有的 token，下一个 token 在模型整个词表上的概率分布是什么？

这就是 **Next-token prediction（下一个 token 预测）目标**。

对于上面展示的提示词（`*The cat*`），在生成的第一步，模型会产出一个概率分布，并[选出概率最高的那个 token](https://www.intoai.pub/i/176405190/1-greedy-decoding)，如下图所示。

![LLM 把下一个 token 选为下一个 token 概率分布中概率最大的那个（贪婪解码）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/03.webp)

被选中的 token 随后会被加进原始提示词，整个过程不断重复，直到 LLM 完成它的输出生成。

## 这是生成 token 的唯一方式吗？

如果我们能把所有 token 一次性、也就是并行地生成出来，会怎样？

是的，这是可能的，而这正是 Diffusion LLM 所做的事——它们用的是一个叫做"Diffusion（扩散）"的过程（*废话*）。

[**Diffusion（扩散）**](https://en.wikipedia.org/wiki/Diffusion)实际上是一个物理过程：粒子从浓度较高的区域移动到浓度较低的区域，最终均匀地散布开来，达到一种平衡状态。

图像生成模型正是受这条原理的启发。这类模型的训练用到了如下两个扩散过程：

1.  **前向扩散（Forward Diffusion）**：模型从输入数据（一张图像）出发，逐步地（在多个时间步里）添加[高斯噪声](https://en.wikipedia.org/wiki/Gaussian_noise)，直到数据变得完全随机。换句话说，这意味着不断改变输入图像的像素值，直到它变成纯噪声。
2.  **反向扩散（Reverse Diffusion）**：在这个阶段，一个神经网络通过预测前向扩散过程中每个时间步所添加的噪声，学会逆转加噪过程。换句话说，这意味着从纯噪声出发，重建出输入图像原始的像素值。

这些过程由一个[噪声调度（Noise schedule）](https://cabralpinto.github.io/modular-diffusion/modules/noise-schedule/)来控制，它定义了在不同时间步上，噪声被添加进数据（前向扩散）或从数据中移除（反向扩散/去噪过程）的速率。

一旦扩散模型完成训练，它就能从纯噪声中生成新的图像样本。

![图片出自作者的书《AI In 100 Images》](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/04.webp)

虽然 Diffusion 常用于图像生成模型，但它也正越来越多地被应用到语言模型上。

要应用 Diffusion，我们首先得学会如何给 token 加噪。可问题是，token 是离散的，不像图像的像素值那样连续——那我们该怎么做呢？

## 如何把 Diffusion 应用到文本上？

有几种方法可以把 Diffusion 用于基于文本的 token。

-   把 token（离散的）转换成 token embedding（连续的），然后对它们做扩散。

![来源](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/05.webp)

-   用一个 Encoder 把 token（离散的）转换成紧凑的 embedding（连续的），并在这个潜在空间（latent space）里做扩散，而不是在原始的 token embedding 上做。

![来源](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/06.webp)

-   按照一个在每个时间步都变化的概率，给一些 token 打上掩码（用 `[MASK]` token 替换它们）。这是 token 加噪的一种替代做法。这个方法被 [**LLaDA**](https://arxiv.org/abs/2502.09992) 所采用——它是最成功的基于扩散的语言模型之一，我们将在下一节里深入讨论它。

## LLaDA 是什么？

**L**arge **La**nguage **D**iffusion with m**A**sking（LLaDA）是一个从零开始训练的扩散模型，经过了预训练和监督微调（SFT），就跟自回归 LLM 的训练方式一样。

它的 8B 参数版本在[上下文学习（In-context learning）](https://en.wikipedia.org/wiki/Prompt_engineering#In-context_learning)上展现出强劲的表现，并具备令人印象深刻的指令遵循和多轮对话能力，与自回归 LLM [LLaMA](https://arxiv.org/abs/2302.13971) 相似。

LLaDA 8B 在[反向诗句补全任务](https://arxiv.org/abs/2309.12288v4)上还击败了 GPT-4o，这意味着它更擅长续写那些以倒序写成的诗——这很可能是因为它在本质上是非顺序的。

![来源](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/07.webp)

LLaDA 所采取的方法，开启了一个有着真实世界应用的 Diffusion LLM 时代，并催生了像 [Mercury](https://arxiv.org/abs/2506.17298) 这样的专有模型。

![来源](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/08.webp)

## 深入理解 LLaDA

LLaDA 对 token 概率分布的建模，用到了：

-   一个前向掩码过程 / 训练
-   一个学习得到的反向解掩码过程 / 推理

这与扩散模型用于图像生成的做法相似，只是被改造成了适配离散 token 的形式。

### 理解前向掩码过程 / 模型训练

给定一个训练用的文本序列，从一个[均匀分布](https://en.wikipedia.org/wiki/Continuous_uniform_distribution)中选取一个取值介于 0 和 1 之间的掩码比例"`t`"。

文本序列中的每个 token 都*独立地*以概率"`t`"被打上掩码。这就产生了一个被掩码的文本序列。

这个被掩码的序列被送入一个基于 Transformer 的模型，该模型会同时在序列的每一个位置上输出一个覆盖整个词表的概率分布。其中，被掩码 token 的分布对我们而言才是重要的，而未被掩码 token 的分布则被忽略。

有一点很重要：这里用的 Transformer 模型采用的是双向注意力机制（bidirectional attention mechanism）。这意味着它不使用因果掩码（causal mask），在预测一个 token 时会同时考虑它过去和未来的 token。

模型的预测与被掩码位置上真实 token 之间的交叉熵损失（cross-entropy loss）会被计算出来，并按 `1/t` 进行缩放。这个缩放确保模型能从轻度掩码和重度掩码的序列里同样学得好。

如果不做缩放，重度掩码的序列贡献的逐 token 损失项就会比轻度掩码的序列多得多。这是因为损失是在被掩码位置上求和的，而重度掩码的序列上这样的位置更多。在这种情况下，模型的大部分梯度信号会来自重度掩码的序列，从轻度掩码的序列里学到的就更少。

`1/t` 这个因子会上调轻度掩码序列的权重，让它们对学习的贡献程度与重度掩码序列相当。

经过这一步之后，模型的权重会用反向传播来更新。这就完成了对一个训练文本序列的处理。

接下来，我们转向下一个训练文本序列，并选取一个新的"`t`"值，也就是掩码比例。不断变化的掩码比例帮助 LLaDA 在训练过程中考虑到所有可能的掩码组合（从几乎不被掩码到几乎被完全掩码的序列）。

上述步骤会被重复，这个过程会被应用到训练数据集中所有的训练文本序列上。

来看看下面这张图，它展示了模型在一个训练文本序列上接受训练的过程。

![LLaDA 从单个训练文本序列中学习](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/09.webp)

### 理解反向解掩码过程 / 推理

在推理（Inference）阶段，模型从一个提示词出发，并设定两个超参数：

-   **响应长度**（`L`）：模型输出的响应 token 的数量。
-   **生成步数**（`N`）：模型为生成 token 所要走的去噪步数。步数越多意味着生成质量越好，但生成速度越慢。这是使用 diffusion LLM 时需要记在心里的一个权衡。

给模型的初始序列形如：

> `[prompt tokens] + ([MASK] * L) response tokens`

推理循环会运行 N 个时间步，从时间步 `t = 1`（此时响应被完全掩码）一直到 `t = 0`（此时没有任何掩码 token）。

在每一个中间时间步（`s`），模型处理整个文本序列，并在每个位置上输出一个覆盖词表的概率分布。在每个位置上，[概率最高的 token 会被选出来](https://www.intoai.pub/i/176405190/1-greedy-decoding)。这样就生成了一个完整的响应。

这本可以是单步推理的终点，但此时的输出质量不会令人满意。因此，我们会把这一步预测出来的一部分 token 重新打上掩码。

重新掩码（Remasking）可以随机进行，但 LLaDA 改用了一种更聪明的做法。这叫做**低置信度重新掩码（Low-confidence remasking）**。

用这种做法，对每一个被预测出来的 token，只保留置信度最高的那些预测，同时把 `s/t` 比例的位置重新打上掩码（`s` 是下一个时间步）。这些位置会在之后的某个时间步被填上——届时模型对这些 token 周围有了更多上下文，对自己的预测也更有信心。

换句话说，这意味着容易预测的 token 会被先填上，而难的那些则只在我们有了更多上下文之后才填。

经过这一步之后，被部分重新掩码的文本序列会被送回模型，上述过程总共进行 `N` 个时间步。在时间步 `t = 0` 时，已经没有掩码留下，最终的响应也就得到了。

虽然我们把响应长度设为了 `L`，但如果模型在响应中间某处产出了一个序列结束（End-of-sequence）或 `EOS` token，那么它之后的所有 token 都会被丢弃。这就是 LLaDA 能够控制自身响应长度的方式。

在下一节我们走一遍推理示例时，这个过程会变得清晰得多。

## 推理过程可视化

来看看下面这几张图，它们展示了模型如何用低置信度重新掩码来生成一个输出。

### 生成步骤 1

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/10.webp)

### 生成步骤 2

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/11.webp)

### 生成步骤 3

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/12.webp)

### 生成步骤 4

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/diffusion-llms-explained-simply/13.webp)

## 延伸阅读

-   [*Diffusion-LM Improves Controllable Text Generation*](https://arxiv.org/abs/2205.14217)
-   [*Latent Diffusion for Language Generation*](https://arxiv.org/abs/2212.09462)
-   [*Simple and Effective Masked Diffusion Language Models*](https://arxiv.org/abs/2406.07524)
-   [*Large Language Diffusion Models*](https://arxiv.org/abs/2502.09992)
-   [*d1: Scaling Reasoning in Diffusion Large Language Models via Reinforcement Learning*](https://arxiv.org/abs/2504.12216)
-   [*Mercury: Ultra-Fast Language Models Based on Diffusion*](https://arxiv.org/abs/2506.17298)
-   [*LaViDa: A Large Diffusion Language Model for Multimodal Understanding*](https://arxiv.org/abs/2505.16839)

## 图片来源

除非在图注中另有说明，所有插图均由作者创作。
