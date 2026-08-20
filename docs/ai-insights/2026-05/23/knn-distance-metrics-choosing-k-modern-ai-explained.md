---
title: 从鸢尾花到反向图搜：K-Nearest Neighbors 是怎么撑起现代 AI 的
author: Kamrun Nahar
url: https://pub.towardsai.net/knn-distance-metrics-choosing-k-modern-ai-explained-4d3a1edb52f7
translated: 2026-05-23
tags:
  - Machine Learning
  - Artificial Intelligence
  - Programming
excerpt: For You 页面、Spotify Discover Weekly、你做过的每一次反向图片搜索、ChatGPT 内部的检索——它们背后的算法，比彩色电视还要老。1951 年由 Evelyn Fix 和 Joseph Hodges 在一份美国空军技术报告里提出，从未被真正取代过。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/01.webp
---

# 从鸢尾花到反向图搜：K-Nearest Neighbors 是怎么撑起现代 AI 的

## *K-Nearest Neighbors 是机器学习里的蟑螂，它会比我们都活得久。*

For You 页面、Spotify Discover Weekly、你做过的每一次反向图片搜索、ChatGPT 内部的检索——它们背后的算法，比彩色电视还要老。

K-Nearest Neighbors。1951 年由 Evelyn Fix 和 Joseph Hodges 在一份美国空军的技术报告里提出。从来没有被真正取代过，只是被改名、加速，然后悄无声息地塞进了你听过的每一个向量数据库里。Pinecone。Weaviate。Qdrant。Chroma。Milvus。全是它。同一种思路，更快的索引。

奇怪的地方在于，没人把它当成"基础"那样教。KNN 被当作"入门算法"塞给机器学习一年级新生，然后这些新生毕业后进了公司，又在一栋十亿美金 AI 大厦的底层——继续运行 KNN。

![For You 页面、Discover Weekly、ChatGPT memory，都是同一个算法。1951 年。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/01.webp)

这是我希望有人在我读教科书版之前先递给我的那份 KNN 指南。

走吧。

---

你在河内醒来。你从没来过这里。你不会说越南语。你站在一家热闹的米粉店里，饿得发慌。排在你附近的五个人正在吃五碗不同的汤。其中三个人吃的是同一种黄汤；两个人吃的是红色的某种东西。服务员正盯着你看。

你指了指那碗黄汤。

你活下来了。你吃到了好吃的。你没有在河内浪费一顿饭。

那就是 K-Nearest Neighbors。K 等于 5。距离是"物理上离我最近的几个人"。投票规则是"他们大多数人在吃什么"。你的预测是"那我也应该吃那个"。超参数则是你愿不愿意在不显得怪异的前提下，瞟一眼陌生人的碗。

![又饿又迷路，意外发明了一个 1951 年的算法。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/02.webp)

整个概念就是这么多。剩下的全是工程。

KNN 做分类时，会把 K 个最近的存储点的标签拿出来投票；做回归时，会把这些点的值拿出来取均值。没有训练。没有神经网络。也没有任何你体育老师能认得出来的"学习"。只有记忆、距离和民主。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/03.webp)

## 为什么它被叫做"懒惰学习器"，以及为什么这其实是夸它

大多数 ML 模型在训练时拼命干活，在预测时休息。

线性回归在 fit 时算矩阵；随机森林事先建好几百棵树；神经网络耗几个小时、有时几天的电——偶尔是一个小国 GDP 的电——然后在推理时乘点数字，几毫秒给你一个答案。

KNN 把这个交易反过来。

训练就是把数据存下来。算法说一句"行，这是我的点"。这一部分快得只取决于你硬盘记东西要多久。贵的是预测——KNN 在预测时可能要对每一个存储点都算一遍距离。

人们叫它"懒惰学习器"。这是一个圈内玩笑。KNN 不是懒，KNN 是有耐心。它拒绝做那些"还不知道有没有必要"的活。它是算法界的 project manager。

为什么这件事要紧：如果你有 5000 万个训练点，每一次预测在没有任何加速的情况下都要做 5000 万次距离计算。这种事原型阶段可以，生产阶段不行。本文后面会讲那些让它在大规模上变得可用的速度技巧（KD 树、ball 树、HNSW、IVF）。现在只要记住一点：KNN 的账单是在 predict 时寄到的，不是 fit 时。

![这就是它的训练过程。它在学习。看看它学得多认真。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/04.webp)

## 那些你天天在用却没意识到的 KNN

你这周用过 KNN。可能就在今天早上。可能就在最近十分钟里。

你打开 TikTok，For You 页面莫名其妙地知道你这个月迷上了木工视频——那是在用户 embedding 上跑的近似 KNN。你在 Spotify Discover Weekly 里发现了一首比你整个歌单都打的歌——那是在歌曲 embedding 上跑的 KNN（Spotify 多年前开源了他们的 Annoy 库）。你用 PlantNet 扫了一片叶子，它两秒钟告诉你这是什么物种——模型先用 CNN 给照片编码，然后让 KNN 在已知植物里找最近邻。

Hinge 把那张莫名其妙完美的资料卡推到最上面——那是 KNN 在按"与你已经点过喜欢的人有多像"做筛选。你的银行在你从没去过的某个州、凌晨四点的加油站标了一笔可疑交易——那是反欺诈系统在用 KNN 风格的异常检测，发现这笔交易离你在特征空间里"惯常的邻居"很远。Shazam 第三次终于在自助洗衣店里认出了那首歌——它在用最近邻查找匹配音频指纹。Google 反向图搜把你那张从 Pinterest 偷下来的图原模原样地吐回来——那是在图片 embedding 上跑的 ANN（近似最近邻）。

注意到了吗？

几乎所有你会形容为"挺聪明"的消费级 App 功能，其实都是穿着 confidence jacket 的相似度查找。

![这是你的手机。每一条发光的线都是同一个算法。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/05.webp)

有趣的转折是：这些生产系统里没有一个用的是原生 KNN。它们用的是近似最近邻方法（HNSW、IVF、LSH、ScaNN），因为对十亿级数据做精确 KNN 慢得没法用。但骨架是一样的。你在河内米粉店里下意识做的那点心算，就是把那条让你笑出来的推荐推到你眼前的同一种数学。

## 你真正会用上的五种距离度量

这里是大多数初学者扑街的地方。KNN 的好坏由你的距离度量决定。挑错了，模型就是错的；挑对了，模型好得让人怀疑人生。距离度量不是细节，距离度量就是整段对话。

值得记的五种。窗台上那块半化的黄油提醒我，这一节得抓紧节奏。

1.  EUCLIDEAN（L2）。直线。

distance = square_root( sum over each feature of (x_i — y_i) squared )

就是你在学校学过的勾股定理。乌鸦飞过去那条路。两点之间，一条斜线。这是 scikit-learn 的默认值，对量纲接近的干净数值特征几乎总是够用。

什么时候它发光。连续的数值数据，低到中等维度，特征之间确实可比。

什么时候它会背刺你。高维。没做标准化的混合量纲。任何"方向比大小更重要"的场景。

讲个故事。我第一次在 768 维的句子 embedding 上用 Euclidean 时，"我喜欢狗"这条查询的最近邻，结果是一条关于停车罚单的推文。我盯着显示器愣了整整一分钟。窗外那棵树上卡着的塑料袋被风吹歪了。然后我换成了 cosine。下一个最近邻，是一条关于金毛的推文。教训我收到了。

1.  MANHATTAN（L1）。出租车。

distance = sum over each feature of absolute_value(x_i — y_i)

你在曼哈顿不能穿楼飞行。出租车只能沿着方格走。你的距离就是横向几个 block 加上纵向几个 block，没有捷径。

什么时候它发光。有离群点的数据（Manhattan 对离群点的惩罚比 Euclidean 轻，因为平方会放大差异）。网格状数据。某些高维空间——稍后再说。

什么时候它会背刺你。当你的特征代表的是某种弯曲的、有方向性的东西，走 L 形路径在几何上很傻的时候。

![同样的两个点，两种度量。一个模型决策之后，你的准确率涨了 8% 或者掉了 12%。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/06.webp)

1.  MINKOWSKI。家长。

distance = ( sum over each feature of absolute_value(x_i — y_i) raised to p ) raised to (1 divided by p)

Minkowski 是把上面两个套在一起的"总公式"。p=1 时是 Manhattan，p=2 时是 Euclidean，p=infinity 时是 Chebyshev（取最大差值，国际象棋里王走一步、仓库路线规划都会用）。

为什么这件事要紧。scikit-learn 的 KNN 默认就是 Minkowski 加 p=2。所以当你在文档里看到 metric='minkowski'，那只是同一公式的参数化形式。偶尔在某些怪数据集上，分数 p（比如 p=1.5）会有用——我整个职业生涯里只见过两次，而且两次背后那位工程师都是会读研究论文的那种人。

1.  COSINE。角度。

*cosine_similarity = ( A dot B ) divided by ( magnitude_of_A times magnitude_of_B ) cosine_distance = 1 — cosine_similarity*

Cosine 测的是两个向量之间的夹角。它把长度扔掉。两个朝同一个方向的向量是相似的，哪怕其中一个比另一个长得多。

什么时候它发光。文本 embedding。sentence transformer。Word2Vec。OpenAI 的 embedding。图像 embedding。任何现代 transformer 模型出来的 embedding。任何"长度只是模型副作用、不是真实信号"的场景。

什么时候它会背刺你。真实世界里"大小本身就有意义"的数值特征，比如价格、计数、"还剩几块饼干"这类东西。

cosine 之所以成为 embedding 的默认度量，一半是历史原因（最初 word2vec 论文用的是它），一半是实践原因（embedding 的长度会随着 fine-tune 漂移，但方向相对稳定）。如果距离度量你只能记一个，记这个。它是你今后搭的每一条 RAG 流水线里、为 retrieval 提供动力的那个度量。

1.  HAMMING。数错位的。

distance = number of positions where two equal-length strings differ

Hamming 数的是两条等长序列在多少个位置上不一样。字符串 "1010" 和 "1001" 的 Hamming 距离是 2。"ACTG" 和 "ACGG" 的距离是 2。两个在第 3 位和第 5 位不同的产品 SKU，距离也是 2。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/07.webp)

什么时候它发光。二元特征。DNA 序列（真实的生物信息学工具就在用）。哈希相似度（图片去重、抄袭检测会用）。one-hot 编码的类别特征。

什么时候它会背刺你。连续数据。不等长字符串（那种用 Levenshtein）。混合类型的特征。

还有更多（Mahalanobis、Jaccard、Hellinger、edit distance、earth mover's）。大多数是专用工具。上面这五个能帮你扛过 95% 的活。

![五种距离度量走进了一个模型。最后只有一个拿到了工作。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/08.webp)

## 挑 K。这件事会悄悄毁了你的模型。

K 只是一个数。K 也是这个月你会碰到的最危险的一个数。

K 设太小，你的模型会变成一个偏执的图书管理员。K=1 意味着预测标签就由最近的那个点决定。一个放错位置的训练样本、一个标错的行、一个奇怪的离群点，都能在它周围切出一小块"错误王国"。决策边界看起来像心率监测仪的读数。

K 设太大，你的模型会变成那种只信平均数的区域经理。K 等于整个数据集的大小，意味着每个预测都是多数类。你绕一大圈做出了一个常数函数，外加授权费。

![同样的数据，同样的算法，三个 K 值，三个完全不一样的模型。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/09.webp)

挑 K 的几种方法，按你对这份工作有多认真排序。

餐巾纸规则。K 等于训练点数的平方根。100 个点，试试 K=10；10000 个点，试试 K=100。作为起点的猜测可以，作为最终答案就太糟。

二分类里的奇数 K 规则。用奇数 K，让票数不会平局。否则 scikit-learn 会用"挑序号更小的那个类"来打破平局，这在算法上等价于扔一枚你已经粘住的硬币。你不会想要这个的。

偏差-方差视角。K 小，偏差低，方差高，模型会抖；K 大，偏差高，方差低，模型变得犯困。挑一个中间的 K 再调。

成年人的方法。在训练集上做 K 折交叉验证。从 1 循环到某个上限（我一般停在 sqrt(N) + 20）。把验证误差对 K 画出来，找那个"拐肘"，挑刚过拐点的那个 K。

![粉色那条线是你的训练集在对你撒谎。蓝色那条线说的是真话。听蓝色那条的。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/10.webp)

一条花了我三年才领会的话。正确的 K 取决于你的数据、你的特征、你的噪声水平、还有你的距离度量。没有"放之四海皆准的 K"。任何告诉你"K=5 最好"的人，要么运气一直好得不行，要么实验做得不够多。

我曾经一不小心把 K=2 上线到生产。二分类里 K=2，意味着两票投票，而我那个数据集大约 14% 的时候会打平。系统默认把平局判成负类，结果就是悄无声息地把假阴性率推高了——正好高到下个季度的客户流失分析会被触发。产品经理始终没搞明白为什么流失率突然飙升。我也没去告诉他，他后来跳槽去了别家公司。那个月的钱包里那些发票现在还在，颜色淡了。

![KNN 调参，就是更糟糕职业路径上的"粥温工程"。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/11.webp)

## 维度灾难。教科书在这里会讲得很怪。

下面这件事，没人会礼貌地告诉你。

在低维里，距离的行为跟你预期一致。有的点近、有的点远，算法有东西可以咀嚼。简单模式。

在高维里，每个点和其他每个点的距离大致都一样远。数学上这真的很奇怪。单位超立方体的体积会集中到角落附近；高维球体的质量几乎全在靠近表面的那一层薄壳里。你那套在三维里训练出来的直觉，正式失业。

实际后果是："最近邻"这件事不再有任何意义。点与点之间最大距离和最小距离的比值，会随着维度增加趋向于 1。所有人都一样远。KNN 这种以"谁更近"为根基的算法，整个问题都没了。

![这就是你不假思索地往模型里加特征时，KNN 直觉会掉下去的那道悬崖。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/12.webp)

有三件事能救你。

一。在高维里用 cosine。即便长度都被拉平，方向仍然有意义。这就是为什么所有基于 embedding 的搜索系统都用 cosine——不是因为 cosine 有魔法，而是因为 Euclidean 在那里坏了。

二。降维。可视化用 PCA、UMAP、t-SNE；生产用 autoencoder。把你那个 2048 维的 ResNet 向量降到 128 维，并保留住结构。KNN 就能再喘上气。

三。挑特征。KNN 对每个特征一视同仁。一个没用的特征不是免费的，它是你距离函数上的一个小漏洞。把死重扔掉。

隔壁的猫已经盯着空荡荡的墙角看了十分钟。我觉得它看见了维度灾难，并且很不屑。难说。

## 标准化。这一步没人做，直到他们做错。

KNN 对特征的量纲过敏。这是初学者 KNN 代码里被跳过最多的一步，也是初学者 KNN 代码通常很糟的原因。

设想两个特征。薪水（美元）。年龄（岁）。薪水范围是 30000 到 200000，年龄是 18 到 80。两个人之间的 Euclidean 距离会被薪水差异主导，年龄基本可以当不存在。你的"最近邻搜索"已经悄悄变成了"最近薪水搜索"，而你没意识到，因为代码跑通了。

修复办法只有一行：先标准化。

StandardScaler 会把每个特征变换成均值 0、标准差 1。这样薪水和年龄就在同一片场地上比划。

如果你的特征有离群点，RobustScaler 更合适（它用中位数和 IQR，而不是均值和标准差）。如果特征有上下界，MinMaxScaler 也行。选哪个，远没有"是不是做了"重要。

那条会让人被开除的规则。scaler 只在训练数据上 fit。在测试数据上用 transform，**绝对不要** fit_transform。否则你就把测试集信息泄漏进了模型，你报上去的准确率就是虚构的。

```python
from sklearn.preprocessing import StandardScalerscaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

第二行。这个差别，记住。

![标准化就是给特征讲公平。让它们以同样的音量吵架。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/13.webp)

## 上代码。带诚实注释的那种。

到上代码的时间了。我们要在 Iris 数据集（分类界的 Hello World，150 行，4 个特征，3 种鸢尾花）上搭一个 KNN 分类器，然后认认真真用交叉验证挑 K。

如果你还没装 scikit-learn，先 pip install scikit-learn。洗碗机会陪着它一起跑，两个差不多同时结束。

> numpy 是 Python 里一切数值计算的地基。
> 
> pandas 是当我想以表格而不是嵌套列表的方式看数据时用的。
> 
> matplotlib 是当我必须说服某个 stakeholder 任何事时用的。

```python

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix
RNG = 42
iris = load_iris()
X = iris.data
y = iris.target
print("X shape", X.shape)
print("y shape", y.shape)
print("class names", iris.target_names)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=RNG, stratify=y
)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
baseline = KNeighborsClassifier(
    n_neighbors=5,
    weights="uniform",
    metric="minkowski",
    p=2,
)
baseline.fit(X_train_scaled, y_train)
y_pred = baseline.predict(X_test_scaled)
print("Baseline accuracy", round(accuracy_score(y_test, y_pred), 4))
print(classification_report(y_test, y_pred, target_names=iris.target_names))
print(confusion_matrix(y_test, y_pred))
```

```
ks = list(range(1, 31))
cv_scores = []
cv_stds = []for k in ks:
    candidate = KNeighborsClassifier(n_neighbors=k)
    fold_scores = cross_val_score(
        candidate, X_train_scaled, y_train, cv=5, scoring="accuracy"
    )
    cv_scores.append(fold_scores.mean())
    cv_stds.append(fold_scores.std())best_k = ks[int(np.argmax(cv_scores))]
print("Best K via CV", best_k)
print("Best CV accuracy", round(max(cv_scores), 4))final_model = KNeighborsClassifier(n_neighbors=best_k, weights="distance")
final_model.fit(X_train_scaled, y_train)final_acc = accuracy_score(y_test, final_model.predict(X_test_scaled))
print("Final test accuracy", round(final_acc, 4))
```

几个值得注意的地方。

模型选择循环里，从来没有碰过测试集。测试集留到最后，用一次"诚实的衡量"。如果你在测试集上调参，你度量的就不是"泛化能力"，而是"你对这个具体测试集 fit 得多好"，这两件事不是一回事。

我选 accuracy 是因为 Iris 是平衡数据集。如果你做的是欺诈检测，99.9% 的交易都是合法的，accuracy 就毫无意义——一直预测"合法"就有 99.9% 准确率。那种情况下用 F1 或者 recall。要看的，是那个会惩罚"你真正在乎的失败模式"的指标。

最终模型用了 weights='distance'。更近的邻居拿到更多票权。下一节就讲这个。

![我代码第一次干净跑通、并且花儿们配合得很好的那一刻。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/14.webp)

## 加权 KNN。当票不该一人一票的时候。

默认 KNN 把 K 个最近邻里每一个的票都算同样重。这很民主。但如果你最近的邻居就贴在你身上，第五近的却在另一个时区，这种"一人一票"就有点蠢。

加权 KNN 说：更近的邻居应该更有发言权。标准做法是反距离权重。距离 1 的邻居权重 1；距离 4 的邻居权重 0.25。投票总和因此向"本地信息"倾斜。

在 scikit-learn 里，这就是一个参数。

```
knn = KNeighborsClassifier(n_neighbors=5, weights="distance")
```

为什么这件事要紧。当你的决策边界扭曲且局部时，加权 KNN 通常能提升准确率。它也能帮你应对轻度类别不平衡——一小簇但很近的少数类邻居，能压过一大群但很远的多数类邻居。

什么时候**不要**用。当你的距离度量本身就很嘈、或者特征方差很大的时候。反距离权重会放大任何"恰好近"的邻居的影响，包括那些"恰好近的离群点"。garbage near, garbage out。

还有一个更花哨的版本，叫做带核函数的距离加权。高斯核、Epanechnikov、tricube。它们把权重曲线弄得更平滑。在实际工作中，'uniform' 和 'distance' 覆盖了 95% 的需求。那些奇异的核函数，是给学术会议、以及那些经费靠这些研究的人准备的。

![五个邻居，其中一个有大喇叭。猜谁会赢。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/15.webp)

## KD 树、Ball 树，以及你的 predict 调用为什么慢

暴力 KNN 会把你查询点到每一个训练点的距离都算一遍。1000 个训练点，没问题；1000 万个，你可以去做个三明治，回来时进度条还是那样。

两种数据结构能大幅压缩成本。

KD-Tree。沿坐标轴递归地切分特征空间。查询会沿树走下来，把"不可能有更近点"的分支剪掉。在低维下，搜索复杂度从 O(N) 降到大约 O(log N)。在 20 维以内表现不错；超过之后，包围盒会变得太宽松，剪不掉东西。

Ball Tree。把空间切成嵌套的超球。在更高维下比 KD-Tree 表现更好——因为超球比轴对齐的盒子更能容纳偏斜分布。能撑到大约 30–40 维，再往上性能下滑。

在 scikit-learn 里：

```
knn = KNeighborsClassifier(n_neighbors=5, algorithm="ball_tree")
```

或者 'kd_tree'、'brute'、'auto'。'auto' 让 sklearn 替你按数据形态选——通常选得对。

对高维数据（想象 100+ 维，也就是现代 AI 生活的那个区间），两种树都救不了你。包围盒剪枝失效——大家都差不多远。你只能回到暴力或者近似方法上。

这就把我们带到了"向量数据库为什么存在"。

## 近似最近邻。现代 AI 背后的那个戏法。

当你有十亿个向量（欢迎来到 OpenAI、Anthropic、Pinecone、几乎所有大 AI 产品的世界），精确 KNN 死了。哪怕加上树也不行。你需要近似 KNN，也叫 ANN。

支撑大半个向量数据库行业的三大算法。

HNSW（Hierarchical Navigable Small World）。当前默认。它构建一张多层图，每个节点和附近节点相连，越高层连接越稀，越低层连接越密。查询从最高层开始，贪心地朝目标跳，再一层一层向下俯冲。又快、又准、内存很饿。Pinecone、Qdrant、Weaviate 用的都是它，基本上 2018 年之后上线的厂都是。Yury Malkov 那篇 HNSW 原始论文是向量搜索领域被引最多的论文之一。

IVF（Inverted File Index）。用 k-means 把向量空间切成若干簇。查询时只搜索 K 个最近的簇，而不是整个索引。在某些负载下比 HNSW 更快，但精度差一些。这是 Facebook FAISS 库内部的主力技术。

LSH（Locality-Sensitive Hashing）。给向量做哈希，让相似向量大概率落进同一个桶。更老、构建更快、精度更低。Spotify 的 Annoy 库用的是相关的树形方法。Google 的 ScaNN 是另一个广为人知的生产级 ANN 系统。

它们回答的是同一个问题："给我这条查询的 K 个最近邻"。只是用一点精度换大量速度。

![向量数据库这个数十亿美金的行业，是在一个 1951 年的算法稍微更快一点的版本上下的赌注。孩子们过得还不错。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/16.webp)

为什么这件事跟你有关。下次再读到关于向量数据库的科技博客，或者看一场满口 "embeddings" 四十遍的 webinar，你会知道：抽象层底下那个基础操作就是 KNN。其他一切都是为了让 KNN 在大规模下还跑得动而堆上去的工程。

## KNN 是 RAG 的脊梁

Retrieval Augmented Generation。RAG。你今年只要还在线，就听过 200 遍的缩写。

真实流程是这样的。

你把文档上传给一个系统。系统把它们切成 chunk（通常每块 200 到 800 个 token，看 chunker 怎么写）。一个 embedding 模型（OpenAI 的 text-embedding-3、Cohere 的 embed-v3、BGE、sentence-transformers，挑一个）把每个 chunk 编成一个向量。这些向量住在一个向量数据库里。

你问一个问题。问题也用同样方式 embed。向量数据库用近似 KNN 加 cosine 距离，找出与问题 embedding 最近的 K 个 chunk。这 K 个 chunk 被塞进 LLM 的 prompt 当作 context。LLM 在这份 context 之上给出回答。

retrieval-augmented generation 里的 "retrieval"，就是 K-nearest neighbors。从结构上、数学上，它和我们一直在讨论的，是同一件事。

我去年有个初级工程师告诉我他"从零搭了 RAG"。我问他 retrieval 那步用的是什么算法。他说"向量搜索"。我问什么是向量搜索。他说"就是那个，向量搜索啊"。我们沉默地坐着。窗外路灯闪了一下，时机刚好。我们一起去 Google。KNN。一直都是 KNN。

![LLM 是大脑，KNN 是图书管理员。两个都按时拿工资。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/17.webp)

## 那些你可能不知道 KNN 也能做的事

既然我们正在挖 KNN 藏在哪里，再说两个被严重低估的用法。

填补缺失值。scikit-learn 里的 KNNImputer 会用 K 个最近邻的均值来填缺失值。在"缺失本身就和特征值相关"时，它常常打过简单的均值填补。我在客户数据上用过它——有些列东缺一块、西缺一块。效果比我预期的好。

异常检测。Local Outlier Factor（LOF）这类算法，会比较一个点周围的局部密度，与它 K 个最近邻周围的局部密度。密度比邻居低得多的点会被标出来。反欺诈、网络入侵检测、生产线质量控制里都重度使用。

这里的模式是：KNN 是一个原语。像 sort、hash、for 循环那样的原语。你一旦看见它，就开始在其他算法内部到处发现它。

## 什么时候**不要**用 KNN

我得说点诚实的，太多教程把这个算法当神拜了。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/18.webp)

跳过 KNN 的情况：数据集巨大、延迟预算紧张。暴力法崩盘，树在高维里只能帮一半，你最后还是得用 FAISS 或 hnswlib 这类 ANN 库。到那时你跑的就不是 KNN，而是"近似 KNN + 一堆工程"。

跳过 KNN 的情况：数据维度极高、又没法降维。维度灾难赢了。

跳过 KNN 的情况：你有一堆无关特征。KNN 一视同仁地对待每个特征。十个有用特征加一个废特征，就意味着每次距离计算里都有 10% 的噪声。随机森林不在乎，KNN 非常在乎。

跳过 KNN 的情况：数据极度不平衡。KNN 会被多数类吸过去。用加权 KNN、过采样、欠采样，或者干脆换个模型。

跳过 KNN 的情况：你需要校准过的概率估计。KNN 给你的是票数比例，那不是真概率——离散且嘈杂。这种时候你想要的是 logistic regression 或者校准过的树模型。

用 KNN 的情况：你在快速做原型，需要一个"不动脑也很难被打败"的基线。也用在基于 embedding 的搜索、推荐系统、异常检测、图像检索、语义搜索和 RAG 上。还用在那些"可解释性重要、需要指出预测是被哪几个邻居推出来的"的场景。

![有时候正确的判断是"用另一个算法"。模型也是有感情的。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/19.webp)

## 那些我一遍又一遍重新学到的教训

一份小小的"田野日志"，因为我觉得你读完它能省下一个周末。

永远先做特征缩放。这个错我至少犯过十一次。其中一次毁掉了一个星期天。那一整个星期天，邻居家的风铃响得像一支微型铜管乐队。我没忘。

用交叉验证挑 K。平方根规则是写在餐巾纸上的，不是写在 model card 上的。调它。

embedding 用 cosine，干净的数值特征用 Euclidean，别搞混。这是一行代码的差别，能让你的准确率翻倍或者减半。

KNN 的瓶颈是 predict，不是 fit。profile 那一段。

当你要扩到百万级点时，从 sklearn 的 KNN 切到 FAISS 或 hnswlib。代码复杂度只上升一点点，延迟却会大幅下降。

距离度量不是唯一的旋钮。喂给模型的特征，比度量更重要。一个糟糕的特征，是你距离函数上一个永远存在的漏洞。StandardScaler 修不了糟糕的特征。

KNN 作为基线非常厉害。如果你那个酷炫模型不能用一个值得为之付出的差距打败 KNN，那就上线 KNN。我用这种方式干掉过两个神经网络项目，没人发现，奖金按时发到账。

永远去看一眼真正的最近邻。不只是预测标签。对一些测试查询，把 K 个最近的训练点拉出来读一读。基于 KNN 的系统里，一半的 bug 在你看一眼邻居的瞬间就暴露了，另一半在你看一眼特征的瞬间也暴露了。

![一份没人要的田野日志。欢迎使用。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/20.webp)

## 为什么 KNN 在 2026 年还重要

关于现代 AI 的一个朴素事实。

你在用的那些最大、最吵的 AI 产品，大多是把 LLM 套在 retrieval 流水线上。这些 retrieval 流水线形状上就是 KNN。整个向量数据库行业，是一个数十亿美金的生意，建立在"让'找 K 个最近邻'在星球规模上跑快一点"之上。给你卖花哨 AI 产品的每一家，在它技术栈的某个地方，都跑着一个稍微升过级的、1951 年由 Evelyn Fix 和 Joseph Hodges 在一份名为《Discriminatory Analysis》的美国空军技术报告里勾勒出来的算法。怎么读它，都行：要么是一个段子，要么是一个相称地谦逊的起源故事。

为什么会这样？

因为应用层的"智能"，大多数其实就是 retrieval + ranking。看起来聪明的模型，通常是在找出相关的东西，然后按"匹配多少"给它们排序。KNN 是这件事最朴素、最直接的表达。其他一切都是管道、规模和品牌。

如果你一辈子只学一个经典机器学习算法，学这个。你一旦理解了"在 embedding 上跑 KNN"，你就理解了 RAG。你一旦理解了 RAG，你就理解了大多数现代 AI 产品到底是怎么跑起来的。你一旦理解了大多数现代 AI 产品到底是怎么跑起来的，这个领域就不再让你觉得"魔法"，而开始觉得"我能搭"。这是一扇单向门，过去就回不来了。

![每一个花哨的 AI 工具都坐在这座金字塔上。底层那块石头，是彩色电视出现之前就刻好的。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/21.webp)

## 别人经常问我的几个问题

KNN 是监督还是无监督？监督。你需要标签。它有个无监督的远亲叫 k-means，是聚类算法，是完全不同的东西。K 这个字母只是巧合。

KNN 能做回归吗？能。scikit-learn 里的 KNeighborsRegressor 会预测"K 个最近训练点的均值"而不是投票一个类别。

KNN 里的 K 和 k-means 里的 K 有什么区别？KNN 的 K 是邻居数；k-means 的 K 是簇数。同样的字母，不同的算法，不同的心智模型。命名很难，计算机科学里有的是糟糕命名。

KNN 能做时间序列吗？算半行。你可以把它用在滞后特征上，但它不在乎时间顺序。当时间真的重要时，请用一个尊重时间的模型。

KNN 需要 GPU 吗？几乎从不。对大多数 workload，KNN 是 CPU 绑定的。FAISS 对超大索引有一个 GPU 模式。一台笔记本在正常数据集上跑 KNN 跑得很好。

为什么 scikit-learn 在 KNN 这件事上这么好？因为这个算法本身就是这么简单。复杂度在"选择"里，不在"数学"里。scikit-learn 是那个已经把无聊部分写好了的朋友。

Cover-Hart 上界是真的吗？是的。这是 1967 年的一个经典结果：当你的数据集趋于无穷时，1-NN 的渐近错误率最多是贝叶斯错误率的两倍（贝叶斯错误率是该问题理论上最低的错误率）。翻译一下：在数据足够多时，KNN 已经惊人地接近最优了。你这辈子能搭出来最聪明的模型，最多也只比 KNN 好两倍。

## 临别一弹

KNN 撑过了七十年、四次 AI 寒冬、三次炒作周期，以及一场全球性的深度学习狂热。它撑下来，是因为它管用、它好懂、它模块化、并且数学是诚实的——没有隐藏状态，没有黑盒。你能用三分钟跟你奶奶讲清 KNN，用两分钟跟你 CTO 讲清。

![发推。出去走走。打开你最喜欢的 App，看一个 1951 年的算法替你决定今晚怎么过。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/23/images/knn-distance-metrics-choosing-k-modern-ai-explained/22.webp)
