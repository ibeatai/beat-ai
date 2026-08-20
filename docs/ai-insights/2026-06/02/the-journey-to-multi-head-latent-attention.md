---
title: 通往 Multi-Head Latent Attention 之路
author: Anuva Sharma
url: https://medium.com/@anuva_74249/the-journey-to-multi-head-latent-attention-5caefb99b824
translated: 2026-06-02
excerpt: 人人都在谈 transformer 怎么扩展（scale），谈它怎么记忆的人却少得多。但实话说，对今天任何一个跑在生产里的 LLM 来说，KV cache 才是说了算的那个——它决定你的上下文长度、吞吐量、GPU 账单，以及你能部署在什么样的硬件上。Multi-Head Latent Attention（MLA）出自 DeepSeek-V2，是我见过对这个瓶颈最优雅的一次进攻。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/01.thumb.webp
---

# 通往 Multi-Head Latent Attention 之路

*为什么 DeepSeek-V2 发明了一个长相奇怪的 attention 模块——以及一个不起眼的代数小技巧如何把 KV cache* ***缩小 57 倍*** *，质量却一分没掉。*

![token 流入，压缩成一个 latent 向量，再折射回每个 head 各自的 key 和 value——这就是 MLA 的精髓。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/01.webp)

人人都在谈 transformer 怎么*扩展*（scale），谈它怎么*记忆*的人却少得多。但实话说，对今天任何一个跑在生产里的 LLM 来说，**KV cache 才是说了算的那个**——它决定你的上下文长度、吞吐量、GPU 账单，以及你能部署在什么样的硬件上。Multi-Head Latent Attention（MLA）出自 DeepSeek-V2，是我见过对这个瓶颈最优雅的一次进攻。

这是一段分两部分的旅程的上半场。这里我们从第一性原理学 MLA——从一个朴素的 attention 模块出发，看着 KV cache 越胀越大，再一路演化经过 MQA、GQA，最终抵达那个让 MLA 既更省、又莫名其妙更好的*吸收技巧*（absorption trick）。下半场会从这里接着往下讲：那个让 MLA 能和旋转位置编码（RoPE）和谐相处的*解耦 RoPE*（decoupled RoPE）技巧。

> 🪄 **一点私货。** 我在笔记本上把吸收过程的推导草草画了下来——而那页涂鸦（文章后面会贴出来）老实讲是看懂 MLA 为什么管用的最紧凑的方式。我会照着当初说服自己的那条路，一步步带你走一遍。

### 目录

1.  MLA 在 transformer 里待在哪儿
2.  attention 的四个家族
3.  KV cache——它到底为什么存在
4.  KV cache 为什么会爆炸（配一个 70B 的例子）
5.  Multi-Query Attention：共享 K 和 V
6.  Grouped-Query Attention：折中方案
7.  Multi-Head Latent Attention：缓存一个 *latent*
8.  吸收技巧（魔法所在）
9.  算笔账：57 倍的压缩预算
10.  下回预告——解耦 RoPE

## MLA 在 transformer 里待在哪儿

在碰任何一个公式之前，先给 MLA 在地图上定个位。一个 transformer block 是个三明治：**attention** + **前馈网络**，靠残差连接和归一化层把它们粘在一起。MLA 是对*attention*那一半的即插即用替换。其余的一切——embedding、FFN、LayerNorm、残差——全都原样不动。

![图 1 —— MLA 是 attention 子层的即插即用替换。transformer 里其余部分完全一致。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/02.webp)

DeepSeek-V2 在引入 MLA 的同时还带来了另外两个想法：**对 key 和 value 的低秩联合压缩**（MLA 的核心），以及一个**解耦的旋转位置编码**，用来解决低秩压缩和 RoPE 之间那个别扭的不兼容。第一个我们这里从头讲到尾，第二个留给后续单独成篇。

## attention 的四个家族

现代 LLM 论文里你会看到四个名字晃来晃去。它们干的活儿都一样——对过去的 token 算加权平均——区别在于*每个 head 各自留着多少 key 和 value*。就这一个选择，会层层传导，决定整个模型的内存和吞吐特性。

![四种 attention 风味，并排摆开。本文的旅程就是从左上到右下的那条对角线。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/03.webp)

大多数课程几乎是把它们当成一张分类表来讲的——"这儿有四个选项，挑一个吧"。但这是看错了。*它们是一连串的补丁。*每一个之所以存在，都是因为前一个有毛病。要真正体会到为什么需要 MLA，我们得从它当初被发明出来要干掉的那个 bug 讲起：KV cache。

## KV cache——它到底为什么存在

自回归生成是重复性的，而其中相当一部分重复是*白干的活*。我们来看看到底是哪些东西在被重复算，以及缓存是从哪一步切进来的。

## 第 1 步：我们是在重复计算吗？

设想一个模型刚刚产出了提示 `"the dog"`。现在它想预测下一个 token。在第 1 步，模型为这两个 token 都算出了 Q、K、V，挺好。接着它采样出 `"bit"`，又想要下一个 token。

天真的做法是，第 2 步会在*整个*序列 `"the dog bit"` 上重跑一遍 attention 模块——把 `"the"` 和 `"dog"` 的 Q、K、V 从头再算一次，哪怕它们的输入 embedding 根本没变，而且 **Wq、Wk、Wv 在训练后就冻住了**。同样的输入 × 同样的权重矩阵 = 同样的输出。我们是在把算力烧在早就算过的算术上。

![同样的输入 × 冻结的权重 = 同样的输出。KV cache 不过就是记忆化（memoization）。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/04.webp)

## 第 2 步：要预测下一个 token，我们实际上需要什么？

倒着推一遍。要预测下一个 token，我们真正非算不可的是什么？

1.  logits——但只针对**最后一个** token。我们不是在重新预测旧的那些。
2.  logits = context 向量 × *Wo*。所以我们需要最后一个 token 的 context 向量。
3.  context 向量 = attention 权重 · *V*。我们需要最后一个 token 的那一行 attention（长度为 *T*），以及**完整的矩阵 *V***。
4.  那一行 attention = Softmax(*Qlast \* K.T)*。我们需要 *Qlast*——这很便宜——以及**完整的 *K* 矩阵**。

所以唯一*新增*的计算，是这个新 token 的 K、V 和 Q。之前每个 token 的 K 和 V 都已经算过了——而且它们永远不变，因为它们的输入不变。那就把它们留着别扔就行。

## 第 3 步：二次方 → 线性

不做缓存的话，第 *t* 步的工作量正比于 *t·d*（对所有过去的 token 做完整 attention），所以生成 *T* 个 token 要花 *O(T²)*。有了缓存，每一步只需做*新* token 和已缓存的 K、V 交互那点活——整个序列下来就是 *O(T \* d)*。二次方变成了线性。

![生成 T 个 token 的总工作量：不缓存是二次方，缓存了就是线性。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/05.webp)

> ***为什么你用过的每个 LLM 都带 KV cache。*** *没有它，上下文长度翻一倍，推理成本就翻四倍。有了它，上下文翻倍，成本也只翻倍。*

## KV cache 为什么会爆炸（配一个 70B 的例子）

所以 KV cache 把计算这块给治好了，很好。问题在于，我们是拿内存换来的速度。对一个朴素的 Multi-Head Attention 模块，我们得在 cache 里存的量是：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/06.webp)

其中 *L =* transformer block 数，*T =* 序列长度，*H =* attention head 数，*d*ₕ *=* head 维度，**B =** batch size，那个 ×2 是把 K 和 V 都算上。再乘以 2 字节（FP16/BF16）就是实际占用的内存：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/07.webp)

### 一个具体例子——32k 上下文下的 LLaMA-2–70B

LLaMA-2–70B 有 L = 80 个 block，H = 64 个 head，dₕ = 128。取单条序列（B = 1），T = 32,768 个 token，FP16：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/08.webp)

**86 GB**——这还只是*一次* 32k token 请求的 KV cache。这比模型权重本身还大，也正因如此，连一块 H100（80 GB 显存）在跑大型稠密 MHA 模型的长上下文推理时都会噎住。内存越高就意味着成本越高、换页越多、每个 token 的延迟越慢。

> *朴素 MHA 之后发明出来的每一种 attention 风味——MQA、GQA、MLA、滑动窗口 attention、paged attention——存在的理由都是为了攻打这一个数字。*

## Multi-Query Attention：共享 K 和 V

第一刀来自 [Noam Shazeer，2019 年](https://arxiv.org/pdf/1911.02150)。在朴素 MHA 里，每个 head 学的是*它自己的* *W*ₖ 和 *W*ᵥ——也就是每个 token 有 *H* 份各异的 key 和 *H* 份各异的 value，全都得存下来。可如果让所有 head*共享*同一套 K 和 V，只有 query 是每个 head 各自的呢？

![MQA 把 K 和 V 塌缩成单独一个 head，由所有 query 共享。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/09.webp)

MQA 把每个 token 的 KV cache 缩小到原来的 1/*H*——对一个 8-head 模型来说，就是 8 倍的内存节省。代价是：质量明显下滑，因为强迫所有 head 共用一套 K 和 V，对 attention 模式约束得太死了。

## Grouped-Query Attention：折中方案

GQA [（Ainslie 等人，2023）](https://arxiv.org/pdf/2305.13245) 是个显而易见的折中：挑 *G* 个组，其中 *1 ≤ G ≤ H*。每个组的若干 head 共享一套 K 和一套 V。当 *G = 1* 时，GQA 塌缩成 MQA；当 *G = H* 时，它塌缩成 MHA。LLaMA-2–70B 用的是 *G = 8*，带来 *H*/*G = 8 \** 的节省，而质量损伤要小得多。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/10.webp)

MQA 给你折扣最大，但质量代价也最陡。GQA 给你一个不错的折扣，代价不大。两者其实还在同一根轴上：*减少每个 token 各异的 K、V 向量的数量*。要是有一根完全不同的轴呢？

## Multi-Head Latent Attention：缓存一个 latent

跨越就在这里。MHA、MQA、GQA 都默认我们必须缓存*形状像 K 和 V*的东西。MLA 打破了这个假设。它问：

> 如果我们每个 token 只缓存一个小矩阵——不是 K，也不是 V，而是一个紧凑的表示，K 和 V 都能按需从它重建出来呢？

整套设定简单得惊人。我们引入一个降维投影，把输入 embedding 变成一个*latent*矩阵，维度比（attention head 数 \* head 维度）小：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/11.webp)

然后用两个升维投影 *Wᵤₖ* 和 *Wᵤᵥ*，在真正需要在 attention 内部用到 K 和 V 时，临时把它们重建出来：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/12.webp)

query 保持原样（或者你也可以对 Q 用同一招来省训练内存——DeepSeek-V2 两样都做了）。

![MLA 流水线：把输入投影成一个小 latent，缓存它，只在 attention 运行时才展开成 K 和 V。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/13.webp)

可等等——我们刚刚*添*了一堆矩阵（*Wdₖᵥ、Wᵤₖ、Wᵤᵥ*）。加更多东西怎么反倒让系统变小了？

因为新增的那些矩阵是**权重**（随模型加载一次），而我们过去每个 token 要存的是**激活值**（每个 token 都要新鲜算一份 K 和 V，永无止境）。我们是拿一个会膨胀的东西，换了一个固定不变的东西。

可以把它想成收拾行李箱。与其把整柜衣服（K 和 V）塞进你住的每一间酒店房间，你只带一个小小的旅行收纳块*（Cₖᵥ）*，进门时才把当下需要的那身衣服取出来。同一柜衣服，行李却轻得多。

## 吸收技巧（魔法所在）

我们来看 query token t 对 key token s 的标准 attention 分数：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/14.webp)

把矩阵乘积展开并重新结合（右边全是固定矩阵之间的相乘）：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/15.webp)

再读一遍。这个乘积是在两个*训练后永不改变*的矩阵之间进行的。所以我们可以离线把它们一次性合并成单独一个矩阵——叫它**被吸收的 query 投影**。推理时分数就变成：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/16.webp)

我们根本不必重建 K，运行时甚至连 *Wᵤₖ* 都不用具现出来。我们只是拿新的 query 去乘那个缓存下来的 latent。

同一招在 value 路径上也奏效。token t 的 attention 输出是：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/17.webp)

代入 *V*，再次把 *W*ᵤᵥ 和 *W*ₒ 重新结合——两者都固定：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/18.webp)

于是整个前向传播只用到 latent *cₖᵥ*，外加两个新的"被吸收"矩阵——它们在加载时一次性折叠好。原来的 *Wᵤₖ* 和 *Wᵤᵥ* 从运行时路径里消失了。*这*就是吸收技巧。

## 它在我笔记本里长这样

我想把当初让我豁然开朗的那一页分享出来，因为老实说，它比任何精修过的图都更清楚。三行：attention 分数、context 向量、logits——每一行都被改写成把固定矩阵吸收进去的形式。每行底部活下来的那部分，要么是*训练时折叠好的*，要么是*推理时缓存下来的*。

![我那页讲吸收技巧的笔记——黄墨水、深色纸，全套都有。红框圈出来的是训练时一次性折叠掉的部分。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/19.webp)

> ***为什么它如此强大。*** *注意吸收技巧对成本干了什么：它在推理时* 消除 *了每个 token 的 K/V 重建，把 W*ᵤₖ *和 Wᵤᵥ 从热路径里* 拿掉 *，只给我们留下尽可能小的、需要缓存的那点东西。我们压缩了 cache，* 顺带 *还省了算力。*

## 算笔账：57 倍的压缩预算

我们把 DeepSeek-V2 的数字代进去。对单个 token，MHA cache 存的是所有 head 上的 K 和 V：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/20.webp)

MLA cache 只存那个 latent（外加一个小小的解耦-RoPE 通道——下回再细说）：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/21.webp)

代入 DeepSeek-V2 的取值：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/the-journey-to-multi-head-latent-attention/22.webp)

这就是那句出名的标题——"**57 倍的 cache 缩减**"。而且关键在于——因为 MLA 仍然给每个 head 各自一套升维投影 *Wᵤₖ* 和 *Wᵤᵥ*——multi-head attention 的表达力被保住了。我们没像 MQA 那样塌缩 head，也没像 GQA 那样把它们分组；我们改的是*存什么*，而不是*有多少个 head*。

## 一石二鸟

-   **cache 大小：** 缩小约 57 倍。我们之前给一个 70B 量级模型算出的 86 GB cache，一下塌缩到区区几 GB。
-   **质量：** DeepSeek-V2 的消融实验显示，MLA 在下游任务上*持平甚至胜过*朴素 MHA，同时把 MQA/GQA 按在地上摩擦。
-   **算力：** 吸收技巧在推理时把 K/V 重建去掉了。我们省了内存，*还*省了 FLOPs（每秒浮点运算次数）。

> *此前每一种 attention 变体都是拿质量换内存。MLA 拒绝这笔交易。这正是它成为 DeepSeek-V2、DeepSeek-V3 乃至整个 DeepSeek-Coder 家族默认选项的原因，也是它正被开源生态其余部分火速抄走的原因。*

## 下回预告——解耦 RoPE

有头大象一直待在屋里，我小心翼翼绕了一路：**旋转位置编码（RoPE）**。RoPE 在做点积之前，会根据绝对位置和索引去旋转 K 和 Q 向量。可 MLA 从不具现 K——这意味着旋转没法按常规方式施加。

DeepSeek-V2 的答案是一个又小又漂亮的 hack，叫*解耦 RoPE*：一条专门携带位置信息的小小 K、Q 旁路通道，留在被吸收的路径之外，再拼接到主 attention 上。这是下一篇文章的绝佳题材——短、利落、非常适合用图来讲。敬请期待。

> 在不破坏吸收技巧的前提下，做到位置感知的 attention。这就是下半场的悬念。

## 如果这篇文章你只记三件事

1.  **KV cache** 把生成从二次方变成线性。
2.  MQA、GQA、MLA*不是一张分类表*——它们是针对同一个内存瓶颈的一连串补丁，一个比一个更激进。
3.  MLA 的**吸收技巧**在训练时把固定矩阵折叠到一起，只留下一个小小的 latent 作为你唯一要缓存的东西。缩小约 57 倍（*DeepSeek-V2*），质量零损失。

谢谢你陪我把这一整趟走完。MLA 属于那种远看吓人、近看显而易见的想法——而一旦你见过了吸收技巧，就再也无法当作没看见。下一篇会从这里原地接上：*解耦 RoPE*，那个小而精巧的变通办法，让这一切能和旋转位置共存。如果你想等它上线时收到提个醒，下面点个赞就是告诉我你要看第二回合最省事的方式。👏
