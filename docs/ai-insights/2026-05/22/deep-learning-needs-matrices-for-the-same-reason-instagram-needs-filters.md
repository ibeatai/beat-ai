---
title: 深度学习需要矩阵，原因和 Instagram 需要滤镜一样
author: Tina Sharma
url: https://thequantasticjournal.com/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters-5e1ec4f8edbf
translated: 2026-05-22
tags:
  - Machine Learning
  - Technology
excerpt: 上学的时候，矩阵是我最喜欢的主题。我学得很快，也许你也一样。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/01.webp
---

# 深度学习需要矩阵，原因和 Instagram 需要滤镜一样

## 走进 AI 的数学

## 一份关于权重矩阵、维度变化、层坍缩、激活函数，以及 W 为何从一开始就不是零的精确指南

![一位讲师在白板上用矩阵和 Instagram 滤镜的类比，讲解 torch.matmul(x, W) 到底做了什么。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/01.webp)

在这篇文章里，我们将了解矩阵是如何改变数据的、为什么加法或缩放这类简单操作还不够，以及这些如何引出了神经网络内部的特征变换。

[**不是会员？点击这个链接免费阅读本文**](https://medium.com/the-quantastic-journal/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters-5e1ec4f8edbf?sk=0a042b7cc18ec38bb3fcfb0fa467db42)

上学的时候，矩阵是我最喜欢的主题。我学得很快，也许你也一样。

所以当"*矩阵*"在数据科学课程里再次出现时，它大概感觉挺简单。你学了理论、做了习题、应用了各种变换、考了个好分数，然后就翻篇了。

但接着，现实来了。

你打开一份 PyTorch 教程，看到这样一行：

`output = torch.matmul(x, W) + b`

大多数讲解都止步于把 `**W**` **叫作权重矩阵**。这没错，但它没告诉你 `W` 究竟*做了什么*。

读到本文结尾时，这一行代码应该会变得不再那么神秘。

## 矩阵究竟是什么

想想在 Instagram 上修一张照片。

你调整亮度、对比度或暖色调。图像里的物体没有变，但每个像素背后的*数字*变了。

![像素值经过 Instagram 风格的调整发生变化，展示被修改的是数字——而不是物体。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@c3a92973d07f5d0b2764715590681de700584df7/ai-insights/_external/30ff090a517b961e.png)

一张照片看上去是物体和颜色，但归根结底它只是数字。每个像素都是一组数值，比如 \[180, 120, 90\]。

当你应用一个滤镜时，没有任何物理实体发生变化。改变的只有这些数字。

正如这张图所示，滤镜从这样一个理念出发：变换的是数值，而不是物体。

![一个滤镜充当矩阵变换：一个向量进去，一个新向量出来。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/02.webp)

现在来看单个像素。

正如图中所示，一个输入向量经过一次变换，变成了一个新向量。比如 \[180, 120, 90\] 可能变成 \[210, 105, 75\]。

这不是随机的。每一个输出值都是原始数值的某种组合。

这就是关键的一步：这些数字不只是被改变了；它们被混合了。

你其实早就在实践中见过这一点。

正如图中所示，不同的滑块以不同的方式改变数字。亮度让所有数值整体平移，对比度把它们拉开距离，暖色调则调整色彩平衡。

每个滑块都遵循一条规则，以不同的方式变换同一组输入数字。

在*机器学习*里，同样的理念也适用。一个用户可以表示成一个向量，比如 \[*科技*, *游戏*, *购物*\]。把它乘以一个矩阵，就会得到一种新的表示，其中每个值都是原始特征的某种组合。模型用这种表示来做决策。

## 这种变换是如何运作的

每个输出都是所有输入的加权组合。

举个例子：

-   ***输出 1*** 组合了科技、游戏和购物
-   ***输出 2*** 同样组合了这三者

矩阵中的每个值，都表明某个输入对某个输出的影响有多大。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/03.webp)

矩阵变换会改变坐标轴的含义。在 `W` 之前，坐标轴是原始测量值。经过 `W` 之后，它们变成了模型认为有用的、学到的组合。那些原本藏在单个特征里的模式，在这些组合中变得可见了。

## 原始的"地图"还不够

每个数据点都是一个向量，一串数字。

到目前为止，我们看到了滤镜如何改变图像中的像素值。同样的理念也能延伸到图像之外。任何现实世界的对象都可以表示成数字。比如一个用户，就可以写成像 \[科技, 游戏\] 这样的向量。

现在主角从像素换成了人，但数学还是那套数学。

看看下面这张图。绿点代表喜欢这首歌的用户，红点代表不喜欢的用户。

![用户散点图，喜欢某首歌和不喜欢的人之间没有清晰的分隔。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/04.webp)

每个人都有两个分数：科技兴趣和游戏兴趣。

这些点散布的方式让分隔变得不清不楚。

![用竖线、横线和斜线尝试分隔数据，全部以失败告终。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/05.webp)

试着只用科技这一个维度。

在*科技*\=9 处，一个点 (9, 5) 是绿色，而另一个点 (9, 9) 是红色，所以同一个值给出了不同的结果，没有任何竖线能把它们分开。

游戏维度也碰到同样的问题。

像 (6, 9) 和 (5, 7) 这样的点游戏分数都很高，可一个是绿色、一个是红色，所以横线也行不通。

就算我们把线倾斜过来，相邻的点依然各执一词。(4, 4) 是绿色而 (3, 3) 是红色，(9, 5) 是绿色而 (8, 6) 是红色。

正如图中所示，没有任何一条直线能干净利落地把两组分开。这说明了为什么对单个特征设简单阈值会失败。

你需要一张新地图，在这张地图上，"科技高且游戏高"成为它自己的一条坐标轴，有用的信号被直接表示出来。矩阵变换能搭出这张地图。

## 为什么不用更简单的办法？

你需要重塑这张地图。但为什么那个变换非得是矩阵？为什么不用更简单的东西？

试试那些更简单的选项。看着它们失败。

![演唱会照片的两个版本——原图和更亮的版本——以及显示每个通道增加了相同数值的像素值。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/06.webp)

正如图中所示，加上一个常数会让每个像素的亮度都增加 50，整张图像以一种均匀的方式变亮。舞台灯光变亮了，黑暗的背景也变亮了。所有东西都平移了相同的量，所以明暗之间的对比保持不变。

![用户散点图，喜欢某首歌和不喜欢的人之间没有清晰的分隔。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/07.webp)

现在让我们从图像转到现实世界的数据。沿用我们之前讨论过的那个例子。

如果你加上一个常数，每个点都会平移相同的量。整张地图移动了，但相对位置保持不变。没有任何东西变得更容易分隔。

![加上常数后数据点发生均匀平移。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/08.webp)

正如上图所示，每个点都一起移动，保留了距离和结构。这证实了简单的加法解决不了分隔问题。

*计算过程*

`[tech, game]` → `[tech + 0.22, game + 0.12]`

用户 *A*：`[0.40, 0.30]` → `[0.62, 0.42]`

相对距离：不变

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/09.webp)

现在来看缩放。

把每个像素值乘以 1.5，图像会变得更亮、更饱和。这种变化是均匀的，所以像素之间的关系保持不变。

![均匀缩放像素值会增加强度，但保持结构完好无损。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/10.webp)

同样的道理也适用于现实世界的数据。正如上图所示，如果你乘以单独一个数，所有东西都会均匀地拉伸或收缩。距离变了，但结构丝毫未变。

在这两种情况下，数据仍然以同样的方式纠缠在一起。这些操作没有创造出任何新东西——它们只是移动或缩放了已经存在的东西。

![缩放后的散点图，展示点散开了，但类别之间仍然重叠。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/11.webp)

你需要的是一种能够*组合*特征的操作。

举个例子，不再单独看科技和游戏，而是创造出一个新值，比如：

0.8×*科技* \+ 0.6×*游戏*

这是数据中一个新的方向——一个此前并不存在的方向。

现在想象一次性创造许多这样的组合，每一个都有不同的权重。这正是矩阵所做的事。每一行都把一个新特征定义为所有输入的某种组合。

那张网格，就是一个矩阵。

## 为什么我们要乘以 X——一次处理你的整个数据集

你已经知道为什么需要一次变换，以及它为什么采取矩阵的形式。

你可以一次只对一个数据点应用这个变换。但在实践中，我们把所有数据点堆叠成一个矩阵 `X`，其中每一行是一个样本。

当我们把 `X` 乘以 `W` 时，这个变换会一次性应用到每一行。每个数据点都被独立地变换，但整个操作在单独一步里完成。

这就是 `X · W` 所代表的含义：同一个变换被应用到整个数据集上。

> 向量告诉你东西在哪里。矩阵让你能改变这些东西所处的那个世界。

正如上图所示，循环一次只对一行应用同一个变换，对大规模数据来说很慢。矩阵乘法一步就把所有行都处理完，因此高效得多。

## W 能对你的地图做的四件事

矩阵的作用就像一个滤镜。它接收一个向量，按特定比例混合它的各个数值，产出一个新向量。

`W` 是一个特定的矩阵——神经网络某一层的权重矩阵。和 Instagram 滤镜不同，它的值不是手工选定的。它们是从数据中学出来的。

当 Instagram 打造 Clarendon 时，是一位设计师选定了那些调整：增强蓝色、给阴影降温、提亮中间调。权重是手工设定的。

`W` 是一个权重靠学习得来的滤镜。模型从随机值起步，产出糟糕的输出，测量误差，再对权重做微小的调整。这个过程重复数百万次，把 `W` 塑造成一组混合比例，能揭示出模型所需的那些模式。

你知道这个变换必须创造出一条新坐标轴，比如"科技高加上游戏高"，但确切的权重是未知的。究竟是 0.8×*科技* 加 0.6×*游戏*，还是别的什么，由模型通过训练来决定。

`W` 存储着这些值。它的每个元素都代表一个给定输入对某个特定输出的贡献有多大。

## 策略：W 能做的四件事

拿一张看上去有点暗的照片。提高对比度，亮的区域会更突出，暗的区域则保持低调。调整暖色调，颜色会朝黄色或蓝色偏移。

变化的是像素值。每个滑块都对这些数字应用一条规则。有的缩放它们，有的平移它们，还有的混合色彩通道。

场景本身不变。变的只是它的数值表示。

![原始分布](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/12.webp)

`W` 对你的数据做的是同样的事。取决于它的权重，它能产生四种不同的效果。在实践中，一个训练好的 `W` 会把它们全部融合在一起，但把它们拆开来看，能让这套机制更容易理解。

*拉伸——调高重要内容的音量*

![高光调整的类比，展示对重要区域有针对性的增强。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/13.webp)

想想给照片提高对比度。亮的区域更亮，暗的区域保持暗，于是差异变得更明显。

![拉伸效果：增大重要特征之间的分隔。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/14.webp)

正如上图所示，图上发生的是同样的事。点沿着某一个方向散得更开，让微小的差异更容易看清。

用数据的话来说，这意味着模型给某个重要特征更大的权重。沿着那个特征的差异变得更突出，从而帮助分隔数据。

游戏兴趣能预测对歌曲的口味，但在原始地图上，分数为 0.6 和 0.7 的用户看上去差不多。`W` 给游戏分配一个更大的权重，给其他特征分配更小的权重，让那个微小的差异在输出中变得更突出。这样模型就能区分他们了。

*压缩——把无关紧要的内容静音*

现在来看相反的情况。降低对比度或纹理，差异就开始淡去。

![纹理调整的类比与旋转后的数据视图。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/15.webp)

用数据的话来说，模型在给那个特征降权，于是沿着它的差异对结果几乎没有影响。

![压缩效果：削弱不重要特征的影响力。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/16.webp)

正如图中所示，点沿着某个方向相互靠拢，把变化压缩了。

`W` 对那些什么都预测不了的特征就是这么做的。如果用户在科技兴趣上各不相同，但科技对歌曲口味无关紧要，`W` 就给它分配一个接近零的权重。这些用户会映射到几乎相同的位置。噪声淡去了，信号留了下来。

旋转——换一个更好的角度看同一份数据

![旋转把对角线模式对齐成一条可分隔的坐标轴。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/17.webp)

正如上图所示，旋转数据能让一个复杂的模式与一条简单的边界对齐。

想象把演唱会照片倾斜 45 度。一道斜着的舞台灯光条纹变成了一条干净的水平线。图像没有改变，但新的视角让这个模式更容易被察觉。

`W` 对你的数据做的是同样的事。一个像"会喜欢这首歌"这样的模式，可能斜着横跨在原始坐标轴上。`W` 旋转这个空间，让这条对角线与某一条单独的坐标轴对齐，从而能用一个简单的阈值把它分隔开。

混合特征——创造从不存在过的坐标轴

![用矩阵的维度把输入特征映射为输出特征。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/18.webp)

这一件是没有任何简单滑块能复制的。Spotify 有两个测量值：科技兴趣和游戏兴趣。单独哪一个都预测不了对歌曲的偏好，但两者都高的用户往往会喜欢它。这个信号是存在的，可它在原始空间里没有一条坐标轴。

`W` 创造出了一条。一个像 0.8×*科技* 加 0.6×*游戏* 这样的组合，成为了一个新的坐标。两者都高的用户得分高，而只在其中一项上高的用户落在中间。一条清晰的边界出现了，而此前那里根本不存在边界。

## 如何读懂一个权重矩阵（W）

神经网络中的每一层都应用一个变换，把数据重塑到一个模式更容易分隔的空间里。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/19.webp)

在前面的例子里，三个输入特征变成了两个，这由 `W` 的形状决定。矩阵 `W` 把输入特征映射为新的特征。

每个输出特征都是所有输入特征的加权组合，而列数决定了产出多少个特征。行数必须与输入维度匹配，因为每个输入都对每个输出有贡献。如果它们对不上，乘法就没有定义。

在 PyTorch 里，这就是 `nn.Linear(3, 2)`。这两个数字是 `in_features` 和 `out_features`，分别对应 `W` 的行和列。

![用矩阵的维度把输入特征映射为输出特征。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/deep-learning-needs-matrices-for-the-same-reason-instagram-needs-filters/20.webp)

正如上图所示，矩阵的行数和列数直接决定了输入特征如何被变换成新的特征。

举个例子，`nn.Linear(512, 64)` 把一个 512 维的向量映射到 64 维。这是一个设计选择：64 个新特征，每一个都是原始 512 个特征的某种学到的组合。

PyTorch 的一个坑：查看 `layer.weight.shape` 显示的是 `[num_out, num_in]`——和直觉感受相反。在数学上，我们常常把这个变换描述为一个 (in × out) 矩阵，因为它把输入特征映射到输出特征。而 PyTorch 在内部把同样的权重存成 (out × in)，所以查看 `layer.weight.shape` 时它看起来是反过来的。

## 顺序很重要——矩阵不满足交换律

普通的乘法满足交换律：3 × 4 = 4 × 3。矩阵乘法不满足。

```
X · W  ≠  W · X
```

`X` 的形状是 (*N*×3)，`W` 的形状是 (3×2)。对于 `X · W`，内部维度匹配，给出一个形状为 (*N*×2) 的输出。对于 `W · X`，它们对不上，所以这个操作是无效的。即便形状碰巧匹配，含义也变了。你会把"配方"以错误的方向应用到数据上，产出一个不同的结果。在神经网络里，各层是按顺序应用的。改变顺序，就同时改变了变换本身和它作用的那个空间。

矩阵是一个滤镜。它接收向量，按特定比例混合它们的数值，产出新的向量。`W` 就是神经网络内部的那个滤镜，只不过它的权重是学出来的。它的每个元素都回答一个问题：这个输入对这个输出应该贡献多少？

再读一遍：

```
output = torch.matmul(x, W) + b
```

`x` 是这一批数据，每一行是一个数据点。W 是学到的变换，它放大某些方向、抑制另一些方向。`torch.matmul(x, W)` 把它一次性应用到每个数据点上。加上 b 会平移结果，这样边界就不必经过原点。输出是同一份数据，但处在一个新的空间里，有了新的坐标和新的含义。

这就是 `W` 所做的事。
