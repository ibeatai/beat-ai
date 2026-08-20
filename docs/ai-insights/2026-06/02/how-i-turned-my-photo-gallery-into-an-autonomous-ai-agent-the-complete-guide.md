---
title: 我如何把相册变成一个自主 AI Agent——完整指南
author: Vatsala Singh
url: https://ai.gopubby.com/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide-b7b9768f8acd
translated: 2026-06-02
excerpt: 读完这篇指南，你会拥有一个完全跑在自己机器上的端到端 agent。你可以丢给它一句话，比如"那张暖暖的黄金时刻照片，就是旅途中我们在路边小饭馆停下来那次拍的"，它能在 100 毫秒内从相册里把图找出来——整个过程内存占用不到 1 GB，花费精确地是零美元。
tags:
  - Artificial Intelligence
  - Programming
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/01.thumb.webp
---

# 我如何把相册变成一个自主 AI Agent——完整指南

![你将做出什么：](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/01.webp)


读完这篇指南，你会拥有一个完全跑在自己机器上的端到端 agent。你可以丢给它一句话，比如"*那张暖暖的黄金时刻照片，就是旅途中我们在路边小饭馆停下来那次拍的*"，它能*在 100 毫秒内从相册里把图找出来*——整个过程内存占用不到 1 GB，花费精确地是零美元。

下面这个系统的完整可运行代码，可以在 [GitHub 仓库](https://github.com/vatsala-singh/AI-Powered-Photo-Search-and-Tagging-Agent)里找到，照着复刻即可。想深入了解，就跟着往下读吧！

## 引言

如果你和我一样，手机相册里塞满了来自天南海北的几千张照片，那你一定懂那种关键时刻找不到对的照片的痛。每次想找点重要的东西——走路时随手拍下的某张活动海报，朋友潦草写在餐巾纸上、被埋在相册某处等着被翻出来的拿手意面食谱，或是一个月前下单时留的那张付款截图——那种煎熬太真实了。

而且这还不只是个人层面的混乱。如果你留心观察，今天我敢说一句话不算错：**照片已经成了我们消费内容的首要方式**。随便打开哪个社交平台，包括 LinkedIn，趋势都明显在往视觉化靠拢。照片不再只是回忆，它们是文档、是收据、是笔记，有时甚至是某件重要事情的唯一记录。

我什么办法都试过。建分门别类的相册，把照片拆进一个又一个桶里，给文件夹起名起得像身家性命都押在上面。没一个真管用。虽然现在有些手机带了相册搜索功能，但残酷的真相是：它们只认得寥寥几个关键词，对图里*到底*有什么毫无真正的理解。你让它找"那个雨夜茶摊上的画面"，它只会一脸茫然地回望你。

于是我决定不再干等别人来解决这件事，转而去做大多数开发者最终都会做的事：**自己造一个**。

旅程就是从这里开始的。我拉上整个开发团队（ChatGPT、Gemini、Claude……你懂的 😄）头脑风暴，花了好几个小时把自己真正想要的东西打磨清楚之后，思路终于明朗：一个**为我个人照片库打造的多模态语义搜索引擎**。

概念本身够直白。拿一个同时理解图像和语言的*多模态模型*，用它为每张照片生成捕捉其*含义*的向量 embedding，把这些 embedding 存进数据库，之后想搜什么，直接抛一句自然语言查询就行。系统会用同样的方式把查询也变成 embedding，跟所有存好的图像向量逐一算相似度，再把最相关的照片返回回来——按与你所找内容的贴合程度排序。

但开工前我有几条绝对不容妥协的底线：

-   **零美元开销：** 整套方案必须完全不花钱。没有按次的 API 费用，没有订阅陷阱。
-   **绝对隐私：** 我的照片是私密的。这意味着不用任何云服务、不用远程数据库、不让任何第三方 API *碰到*它们。一切都得在本地硬件上发生。
-   **超轻量足迹：** 它得能舒舒服服跑在边缘设备上——我日常用的笔记本、一台 mini PC，乃至最终塞在书架角落的一块 Raspberry Pi。
-   **零延迟：** 哪怕完全依赖本地模型，检索也得给人即时的感觉。没人愿意等十秒钟，看着搜索框慢慢醒过来。

说句老实话，要找到同时满足这四项的工具，一点都不容易。这花了我好几周去探索，撞了不少死胡同，还折腾出几套纸面上美轮美奂、实际跑起来糟透了的方案。

但就在上个周末，我终于坐下来，做出了一个恰到好处的东西。

如果你一直在追我的博客，那你应该知道——只要我发现什么值得分享的东西，我就会分享。不光是最终成果，还有思考的过程、走过的弯路、那些灵光乍现的小瞬间。因为公开搭建（building in public）最棒的地方，就在于它能开启一场对话，而我学到的最有用的东西里，有不少正是同行开发者带着自己的经历跳进来贡献的。

那就这样，咱们进入正题。来聊聊怎么把*我们大多数人都在悄悄淹没其中、却最被浪费的那堆非结构化数据——也就是我们的相册——榨出最大价值*。

## 为什么传统照片搜索是坏掉的

在讲我做了什么之前，值得先花点时间说说*为什么*现有方案不够看。因为这并不是个显而易见的问题，多数人甚至意识不到自己的照片搜索有多烂，直到某天急着找一样东西、却怎么也找不到。

## 相册这套办法在现实里崩盘

把照片整理进相册，听起来再合理不过。你建一堆文件夹——旅行、美食、工作、随手拍——把东西一一归位。这套办法能撑大概三个星期。然后生活忙起来，照片堆积的速度远超你整理的速度，没多久你就攒出一个 600 张照片的"杂项"垃圾堆，把整件事的初衷彻底废掉。

就算有人能始终如一地维护相册，相册也只在你*提前知道*以后要找什么的前提下才管用。那张你路过时顺手拍的活动海报呢？你当时可没想着"我该把这个归到路牌类"。你只是咔嚓一下，然后继续走路。现实生活就是这么运作的。

### 关键词搜索能带你走的路有限

有些相册 app 和少数手机开始提供基于关键词的搜索。输入"沙滩"，它就把打了标签或被识别为沙滩的照片翻出来。在一定程度上，这确实有用。

问题在于关键词搜索很脆。它完全取决于系统是否知道你图里那东西的正确叫法。你搜"日落"也许行。但你搜"那张暖暖的黄金时刻照片，就是旅途中我们在高速公路边小饭馆停下来那次拍的"，它就彻底没东西能给你了。关键词是标签，不是理解。

### 云端相册搜索：好，但有代价

Google Photos 和 Apple Photos 在这方面取得了实实在在的进展。它们的端上和云端模型能识别人脸、场景、物体，甚至图里的部分文字。对很多人来说，这已经够用了。

但够用不等于最佳选择。依赖云端驱动的相册搜索，有两个根本性问题。

第一，你的照片正在别人的服务器上被处理、被建索引。对大多数私人媒体而言——抓拍的随性瞬间、截图里捕捉到的私人对话、为留底拍下的医疗文件——这是一笔实打实的隐私代价，值得比多数人愿意给的更多一份思量。

第二，这些系统是封闭的。你没法扩展它们、没法用新方式查询它们、没法把它们接进自己的工作流，也没法在它们之上搭任何东西。你只是它们产品的用户，而不是你自己那条数据管线的主人。

### 真正的缺口：没有语义理解

这些办法有一个共同点：它们全都停留在标签和关键词的层面。没有一个能像人那样**真正理解一张图的*内容***。

我看一张照片时，想的不是"户外、白天、食物"。我想的是"噢，这是那次在 Coorg 彻底迷路之后，找到那家小小的藏餐馆时拍的"。那段记忆是丰富的、有语境的、语义化的。当前的搜索工具，根本没办法把这样一句自然语言描述跟图像里实际的像素内容连起来。

而这，正是我们要去补上的缺口。

## 核心概念：把语义搜索讲明白

在碰任何代码之前，我想先确保核心思路彻底清晰，因为一旦它"咔哒"一声想通了，这套搭建里其余的一切都会显得理所当然，而非什么魔法。

我们要回答的核心问题是：*怎么教计算机理解一张照片讲的是什么，好到你只用大白话描述一下，对的那张就能浮出来？*

答案藏在一个叫 **embedding** 的东西里。那咱们就来理解它。

![embedding 的目标，是把相似的对象在向量空间里映射得彼此更近](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/02.webp)

想想你会怎么向朋友描述两张照片。一张是金毛在公园里撒欢，一张是拉布拉多在沙滩上追飞盘。哪怕这两张图逐个像素看完全不同，你的大脑也会立刻认出它们是*相似的*。它们共享含义。都有狗，都有开阔的户外空间，都洋溢着活力和俏皮。

现在想象一下，你能把任意一张图的*含义*表示成空间里的一个点。语义上相似的照片——撒欢的狗、水面上的日落、桌上的收据——会聚成一簇紧挨在一起。毫不相干的照片则会离得很远。这个空间，就是我们说的**向量空间**，而代表每张图的那个点，就是它的 **embedding**。

embedding 不过是一串数字——少则几百、多则几千个——它们合在一起编码了一张图*意味着什么*，而不只是它在像素层面*看起来*怎样。两张 embedding 相似的图在语义上就相似，哪怕它们没有一个共同的像素。

### 语言是从哪儿进来的？

这里的主要挑战在于多模态：我们需要某种既能理解图像、又能理解文本的东西，而且不止如此——还得是一个能把文本和图像关联起来的模型。**CLIP——Contrastive Language Image Pre-Training** 就此登场，它由 OpenAI 开发，能在同一个向量空间里理解图像和文本。落到实处就是说：如果你喂给 CLIP 一张沙滩上的狗的图，再单独喂给它一句文本"*在水边玩耍的狗*"，两者产出的 embedding 会在那个共享的 512 维空间里落得彼此很近。

这正是让我们整套系统得以成立的关键洞见。我们不需要手动给图打标签，不需要去匹配文件名或关键词。我们把图像 embedding 一次，在查询时把搜索词也 embedding 一下，然后找出哪些图在向量空间里离这条查询最近。

## 选对工具（以及为什么花了好几周）

概念既然清楚了，让我带你走一遍工具的选型。这里我想坦白：这绝不是个干净利落、一锤定音的过程。它是好几周的啃文档、跑实验、撞内存上限、发现某个号称"轻量"的东西居然还得要 GPU，以及总体上用最笨的方式学会各种道理。

下面是我当时的优化目标，以及它如何把一个又一个候选排除出局：

-   **免费**：直接排除托管 API 和托管云服务
-   **私密**：排除任何需要回传、需要同步到云、或会把我照片往别处发的东西
-   **轻量**：排除那些光是空转就要 GPU、或要好几 GB 内存的模型
-   **快**：排除单次搜索查询要好几秒的方案

四个筛子一起套上去，范围迅速收窄。下面是闯关成功的那些，以及为什么。

### 经由 FastEmbed 的 CLIP ViT-B/32——系统的眼睛

![CLIP 架构图解](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/03.webp)

embedding 模型是整套方案的心脏，所以这个决定分量最重。

**CLIP**（具体是 **ViT-B/32** 这个变体）是对的选择，原因有几个。第一，它简直就是为这件事而生的——在数亿对图文上训练，专门用来把视觉含义和语言含义对齐到同一个共享空间。第二，ViT-B/32 这个变体是真的轻量：它能舒服地跑在 CPU 上，不需要 GPU，而且图像塔和文本塔的模型权重各自只有约 150MB。

但通过 Hugging Face Transformers 用原生 CLIP，虽说完全没问题，却要你自己去处理预处理、批处理、归一化和设备管理。**FastEmbed** 就此登场。

FastEmbed 是 **Qdrant** 打造的一个轻量 embedding 库，专门针对 CPU 友好的推理做了优化。它在底层封装了 CLIP，把所有预处理细节都接管掉，自动把模型缓存到本地，让 embedding 一张图或一条查询变成一次函数调用。我们用到的两个模型变体：

-   **Qdrant/clip-ViT-B-32-vision**——图像编码器，产出 512 维向量
-   **Qdrant/clip-ViT-B-32-text**——文本编码器，产出落在同一空间的 512 维向量

两者都在服务器启动时加载一次，缓存进内存，之后每个请求复用；每次查询不会有重新加载的开销。

***关于 ONNX vs. PyTorch 的架构说明：****你会注意到我们用的是 FastEmbed，而不是经由 Hugging Face 的原生 PyTorch。FastEmbed 底层用的是 ONNX 运行时。因为它剥掉了训练相关的笨重图依赖，最大化了 CPU 吞吐，把我们的生成延迟压到了每张图清爽的 10–50ms。*

```python
from fastembed import ImageEmbeddingModel, TextEmbeddingModel
image_model = ImageEmbeddingModel.from_pretrained("Qdrant/clip-ViT-B-32-vision")
text_model = TextEmbeddingModel.from_pretrained("Qdrant/clip-ViT-B-32-text")
```

实测：在一台现代笔记本 CPU 上每张图约 10–50ms，足够快到一口气批量索引几千张照片。

```

image_vector = embed_image("sunset_photo.jpg") 
query_vector = embed_text("beautiful sunset") 

similarity = np.dot(image_vector, query_vector) 
```

### Qdrant Edge——记忆

有了 embedding，你就得有个地方高效地存放并搜索它们。普通数据库靠精确匹配或范围过滤来找东西，它根本没有"接近程度"这个概念。向量数据库则是专为在高维 embedding 上做最近邻搜索而生的，这恰恰是我们需要的。

外面有好几个不错的向量数据库——Pinecone、Weaviate、Chroma、Milvus——但它们多半要么是云托管的，要么很吃资源，要么需要一个你得另行管理的常驻服务进程。[**Qdrant Edge**](https://qdrant.tech/documentation/edge/) 在一个非常重要的点上与众不同：它完全在进程内运行，直接嵌进你的 Python 应用里。

没有服务器要启动。没有端口要管理。没有网络开销。你把它指向一个本地文件夹，它就自动把一切持久化到磁盘。你的数据从不离开你的机器。

```
from qdrant_edge import(
    Distance,
    EdgeConfig,
    EdgeShard,
    EdgeVectorParams,
)
config = EdgeConfig(
            vectors={
                VECTOR_NAME: EdgeVectorParams(
                    size=EMBED_DIM,
                    distance=Distance.Cosine
                )
            }
        )
        _shard = EdgeShard.create(path=str(SHARD_DIR), config=config)
```

### 把它们拼在一起

那么完整的心智模型，从头到尾是这样的：

-   每张照片都过一遍 CLIP 的图像编码器，产出一个 512 维 embedding
-   这些 embedding 存进 Qdrant Edge——一个完全在你机器进程内运行的向量数据库
-   你搜索时，自然语言查询过一遍 CLIP 的文本编码器，产出一个 512 维的查询 embedding
-   Qdrant 用余弦相似度，找出那些离查询 embedding 最近的已存图像 embedding
-   这些最接近的匹配就是你的结果——按语义相关度排序，毫秒级返回

![Qdrant 根据用户查询找出已存图像的流程图](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/04.webp)

### OpenClaw——大脑

有一个搜索函数是有用的。但有一个能听懂自然语言请求、自己决定用哪些工具、把它们串起来、再给你一个连贯回应的 *agent*——这才是脚本和真正像个助手的东西之间的差别。

**OpenClaw** 是一个对话式 agent 框架，让你定义好工具，再让一个语言模型有能力根据用户的话，决定调用哪个工具、按什么顺序、用什么参数。它会处理多轮对话上下文、工具调用逻辑和回应格式化，这样这套编排你一点都不用自己搭。

我们暴露给 agent 的工具，直接对应到我们的 REST 端点：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/05.webp)

这之所以重要，是因为真实查询很少是单步的。想想这条：*"找出我所有狗的照片，再看看里面有没有重复的。"* 这是一次搜索，紧接着对结果做一轮去重检测。agent 自然就能搞定，普通的搜索函数压根做不到。

**用户**："找出我所有狗的照片，再查查有没有重复"

↓

**Agent**：调用 /api/search {query: "dog"}

↓

**Agent**：调用 /api/duplicates {threshold: 0.97}

↓

**Agent**："🐕 找到 47 张狗的照片。检测到 3 簇重复。

删掉这 5 张能腾出空间。"

OpenClaw 还意味着这个界面从第一天起就是对话式的。你不需要懂那套 API。你只管描述想要什么，agent 自己琢磨出怎么办到。

## 搭好环境

好了，咱们卷起袖子干。这一节讲的是在写下任何一行管线代码之前，把所有东西装好、验好。我想确保地基扎实，因为没有什么比搭到一半才去 debug 环境问题更让人抓狂的了。

我们要打交道的完整技术栈如下：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/06.webp)

### 第 1 步：克隆仓库，建好虚拟环境

永远在虚拟环境里干活。这道理我们都懂，可我们也都有那么一个 2025 年的项目当时没这么做，到现在还在为它还债。

```bash

git clone https://github.com/vatsala-singh/AI-Powered-Photo-Search-and-Tagging-Agent.git
cd AI-Powered-Photo-Search-and-Tagging-Agent
python -m venv .venv
source .venv/bin/activate        
```

### 第 2 步：装好依赖

```
pip install -r requirements.txt
```

*requirements.txt* 把所有东西都囊括了：FastEmbed、Qdrant client、FastAPI、Uvicorn、Pillow、OpenClaw 和 NumPy。这次安装最重的部分，是 FastEmbed 在你第一次运行应用时下拉 CLIP 模型权重——视觉塔和文本塔各约 150MB。第一次下载完之后，所有东西都缓存在本地的 *qdrant-edge-data/models/* 下，再也不用重新下载。

另外，有件事值得先知道：这套栈里哪儿都*没有* GPU 依赖。一切都跑在 CPU 上。FastEmbed 这个库正是为此而设计的：它底层用 ONNX 运行时，在 CPU 上比原生 PyTorch 推理快得多。

***专业提示****：如果你在索引一份 iPhone 备份导出，多半会遇到 .heic 格式。标准 Pillow 默认开箱并不原生支持它们，所以我们的脚本会干净利落地捕获异常并记录下来，免得几小时的处理因此崩掉。如果你想要完整的 HEIC 支持，记得把 pillow-heif 加进你虚拟环境的依赖里。*

### 第 3 步：理解项目结构

```
AI-Powered-Photo-Search-and-Tagging-Agent/
│
├── main.py                  # FastAPI app + OpenClaw agent entry point
├── config.py                # All configurable parameters in one place
├── requirements.txt
│
├── pipeline/
│   ├── embedder.py          # CLIP embedding logic (image + text)
│   └── indexer.py           # Batch photo processing and indexing
│
├── store/
│   └── qdrant_client.py     # Qdrant Edge setup and collection management
│
├── tools/
│   ├── search.py            # Semantic search tool
│   ├── tag.py               # Zero-shot auto-tagging tool
│   ├── duplicates.py        # Near-duplicate detection tool
│   └── albums.py            # Smart album grouping tool
│
├── test/
│   ├── embedder_test.py
│   ├── indexer_test.py
│   ├── search_test.py
│   └── qdrant_edge_client_test.py
│
└── qdrant-edge-data/        # Auto-created at runtime
    ├── storage/             # Qdrant's internal shard data
    ├── models/              # Cached CLIP model weights
    └── photos/              # Collection data
```

结构干净、扁平、好上手。每个模块只干一件事，等到哪天有东西出毛病时，整套系统也就好推理得多。

### 第 4 步：扫一眼 config.py

所有控制系统行为的参数都集中在一处：

```
CLIP_IMAGE_MODEL = "Qdrant/clip-ViT-B-32-vision"
CLIP_TEXT_MODEL  = "Qdrant/clip-ViT-B-32-text"
EMBEDDING_DIM    = 512          COLLECTION_NAME  = "photos"
QDRANT_PATH      = "./qdrant-edge-data"BATCH_SIZE             = 32     
TOP_K                  = 10     
TAG_THRESHOLD          = 0.20   
DUPLICATE_THRESHOLD    = 0.97   
```

其中有几个，与其以后才弄懂，不如现在就搞清楚：

**TAG\_THRESHOLD = 0.20**——这个值低得也许让人意外。零样本分类的做法，是计算一张图的 embedding 和某个文本标签（如"outdoor"或"food"）之间的相似度。因为这些是短的、通用的标签，而非完整的描述性句子，相似度分数往往落在比完整自然语言查询更低的区间里。0.20 是经过校准的，既能逮住真正的匹配，又不至于太吵。如果你库里冒出太多不相干的标签，可以把它调高。

**DUPLICATE\_THRESHOLD = 0.97**——这个故意定得很高。同一场日落的两张完全不同的照片，相似度也能打到 0.85。我们只想标出真正的近似重复——连拍、误触的两连拍——所以把门槛设到接近 1.0。

**BATCH\_SIZE = 32**——embedding 一次按 32 张图成批生成。对 FastEmbed 的 CPU 吞吐来说，这是个甜点值。批次更小会让模型没吃饱；批次更大则会在一般硬件上开始触碰内存压力。

### 第 5 步：启动服务器

```
uvicorn main:app --reload --port 8000
```

首次启动时，如果 CLIP 视觉和文本模型权重还没缓存，FastEmbed 会把它们下拉下来。你会在终端里看到下载进度。这件事只发生一次，搞定之后，往后每次启动都是瞬间的。

跑起来后，你有两种方式跟系统打交道：

**方案 A：交互式 API 文档（测试起来最简单）**

在浏览器里打开 [http://localhost:8000/docs.](http://localhost:8000/docs.) FastAPI 会自动生成一整套 Swagger UI，你可以直接在里面发请求，不用写任何 curl 命令。开发过程中随手戳一戳特别合适。

**方案 B：OpenClaw 对话式 agent**

这会把你带进一个聊天界面，你用大白话跟 agent 对话。等索引和搜索管线就位后，我们会更频繁地用到它。

### 第 6 步：验证环境

在我们动手搭任何东西之前，先确认各部件之间能互相通话：

```bash
# Quick sanity check - should return {"status": "ok"}
curl http:
```

另外，要验证 Qdrant Edge 正确初始化、photos 集合也建好了：

```bash
curl http://localhost:8000/api/status
```

你应该会看到集合名、向量维度（512），以及已索引照片数为 0——这是对的，因为我们还没索引任何东西。

如果这两条都干干净净地回来了，你的环境就完全搭好了。模型已缓存，Qdrant 在进程内运行，API 已就绪。我们可以真正动手搭管线了。

## 搭建图像 embedding 管线

系统从这里开始成形。embedding 管线是第一个跑在你照片库上的东西；它负责把一个塞满 JPEG 的文件夹，变成一个可搜索的含义索引。把这步做对，下游的一切就都顺了。

管线分布在两个文件里：pipeline/embedder.py 负责实际的 CLIP 推理，pipeline/indexer.py 负责遍历文件夹、成批处理图像、把向量交给 Qdrant。

### embedder

*embedder.py* 是整个项目里最干净的文件，而且是有意为之。它唯一的活，就是接收一个图像路径或一个文本字符串，返回一个 512 维的 NumPy 向量。其余一切都是别人的事。

```python
from fastembed import ImageEmbeddingModel, TextEmbeddingModel
from PIL import Image
import numpy as np
from config import CLIP_IMAGE_MODEL, CLIP_TEXT_MODEL

_image_model = ImageEmbeddingModel.from_pretrained(CLIP_IMAGE_MODEL)
_text_model  = TextEmbeddingModel.from_pretrained(CLIP_TEXT_MODEL)def embed_image(image_path: str) -> np.ndarray:
    """Convert an image file to a 512-d CLIP embedding."""
    image = Image.open(image_path).convert("RGB")
    embeddings = list(_image_model.embed([image]))
    return np.array(embeddings[0])   def embed_text(query: str) -> np.ndarray:
    """Convert a text string to a 512-d CLIP embedding."""
    embeddings = list(_text_model.embed([query]))
    return np.array(embeddings[0])   
```

这里有几点值得留意。

第一，模型被初始化为模块级全局变量；\_image\_model 和 \_text\_model 在模块首次导入时加载一次，之后被每一次后续调用共享。这是个刻意的设计取舍。每个模型约 150MB，从磁盘加载要花 2–5 秒。如果你每次 embedding 调用都重新加载它们，那一批 500 张照片花在加载模型上的时间，会比真正用来 embedding 的时间还多。

第二，加载图像时那一句 .convert("RGB") 不是可选的。手机相册里满是带 alpha 通道的 PNG、HEIC 导出，还有偶尔编码古怪的文件。CLIP 期望三通道 RGB 输入，要是悄悄把一个四通道 RGBA 张量喂进模型，产出的就是垃圾向量。这次转换只多两个字符，却省去一大堆糊涂账。

第三，两个向量都*由 FastEmbed 自动做 L2 归一化*。正是这一点，让下游的余弦相似度计算简化成一次简单的点积：快、数值稳定，而且在所有查询上一致。

### indexer

*indexer.py* 才是真正干活的地方。它接收一个文件夹路径，递归找出所有图像，每 32 张一批处理，再把一切交给 Qdrant。

```python
import os
import uuid
from pathlib import Path
from datetime import datetime
from PIL import Imagefrom pipeline.embedder import embed_image
from store.qdrant_client import get_shard
from tools.tag import generate_tags
from config import BATCH_SIZESUPPORTED_FORMATS = {".jpg", ".jpeg", ".png", ".webp", ".bmp", ".tiff"}def index_folder(folder_path: str) -> dict:
    """
    Recursively index all images in a folder into Qdrant Edge.
    Returns a summary: total found, indexed, skipped.
    """
    folder   = Path(folder_path)
    shard    = get_shard()        image_paths = [
        p for p in folder.rglob("*")
        if p.suffix.lower() in SUPPORTED_FORMATS
    ]    total    = len(image_paths)
    indexed  = 0
    skipped  = 0
    batch    = []    for i, path in enumerate(image_paths):
        try:
            vector   = embed_image(str(path))
            tags     = generate_tags(str(path))
            img      = Image.open(path)                        point = {
                "id":      str(uuid.uuid4()),
                "vector":  vector.tolist(),
                "payload": {
                    "filename":  path.name,
                    "filepath":  str(path.absolute()),
                    "tags":      tags,
                    "timestamp": int(path.stat().st_mtime),
                    "width":     img.width,
                    "height":    img.height,
                }
            }
            batch.append(point)
            indexed += 1        except Exception as e:
            print(f"Skipping {path.name}: {e}")
            skipped += 1        
        if len(batch) >= BATCH_SIZE:
            shard.upsert(points=batch)
            batch = []
            print(f"  Progress: {i+1}/{total} images indexed...")    
    if batch:
        shard.upsert(points=batch)    return {"total": total, "indexed": indexed, "skipped": skipped}
```

这种批量刷盘的写法——每 32 张图 upsert 一次，而不是一张一张、也不是攒到最后一股脑全写——出于几个理由是对的取舍。一张一张 upsert，意味着一个 500 张图的文件夹要写 500 次数据库，太慢。把所有东西攒在内存里、最后只 upsert 一次，又意味着如果进程在第 487 张图时挂了，你前功尽弃。每批 32 张，给了你合理的写入效率，外加天然的 checkpoint 行为。

每张图外面那层 try/except 也不只是防御性的样板代码。手机相册是真的乱：损坏的文件、下了一半的图、色彩配置古怪的截图、Pillow 没装额外插件就解析不了的 HEIC 文件。记录下来并跳过，比因为一个坏文件就让整次索引崩掉，要明智得多。

## 把图像索引进 Qdrant Edge

我们在工具那一节碰过 Qdrant Edge，indexer 也已经在调它了。但我想在这儿好好花点时间，讲讲数据库实际是怎么搭起来的，因为这一层做的决定，会直接影响下游的搜索和过滤跑得有多好。一次性把 schema 弄对，就意味着你绝不会因为忘了存某个有用的东西，而不得不把整个库重新索引一遍。

### Qdrant Edge 在这里是怎么工作的

你在别的教程里可能见过的标准 Qdrant 库，是作为一个独立的服务器进程运行的。你把它启动起来，它监听某个端口，你的 Python 代码通过 HTTP 或 gRPC 跟它对话。对生产服务而言这没问题，但对一个跑在笔记本上的个人工具来说就是杀鸡用牛刀，还违反了我"没有服务器要管理"的约束。

Qdrant Edge 不一样。它完全在你的 Python 进程内运行：没有二进制要启动，没有端口要打开，没有网络要调用。你导入它，把它指向磁盘上一个文件夹，它就这么跑起来了。我们用的具体类是 EdgeShard，它是 Qdrant Edge 进程内的存储与检索单元。可以把它想成一个把数据库和查询引擎揉进同一个对象的东西，就活在你应用程序内部。

### 设置 shard

在我们的仓库里，整套设置都在 store/qdrant\_client.py，它的活是管理一个单例 edge shard——首次运行时新建它，后续运行时从磁盘把它重新打开——最后暴露一个 get\_shard() 函数，供其他模块拿去访问它。

```python
def get_shard() -> EdgeShard:
    """
    Return the singleton EdgeShard, creating it on first call.     - If SHARD_DIR does not exist → create a brand-new shard.
    - If SHARD_DIR already contains data → reopen it (no config needed).     EdgeShard runs entirely in-process. No binary, no port, no network.
    """
    global _shard
    if _shard is not None:
        return _shard        SHARD_DIR.mkdir(parents=True, exist_ok=True)
    
    
    shard_has_data = any(SHARD_DIR.iterdir())        if shard_has_data:
        print(f"[qdrant_client] Reopening existing shard at '{SHARD_DIR}'")
        _shard = EdgeShard.load(path=SHARD_DIR)
    else:
        print(f"[qdrant_client] Creating new shard at '{SHARD_DIR}'")
        config = EdgeConfig(
            vectors={
                VECTOR_NAME: EdgeVectorParams(
                    size=EMBED_DIM,
                    distance=Distance.Cosine
                )
            }
        )
        _shard = EdgeShard.create(path=str(SHARD_DIR), config=config)
        print(f"[store] Shard ready — vector: '{VECTOR_NAME}', dim: {EMBED_DIM}")
    return _shard
```

### create 与 load 的分流

如果目录里已经有 shard 数据，EdgeShard.create() 会抛错；因此在决定是新建一个目录还是加载已有的之前，我们会显式检查 SHARD\_DIR 里是否已有内容。

### 单例模式

那个 *\_shard* 全局变量加上提前 return，确保了无论有多少模块去调 *get\_*shard()——搜索工具、标签工具、indexer——这些并发一起跑的家伙都共享同一个 shard 实例。从磁盘加载一个 EdgeShard 要花的时间可不算少；你可不想每次函数调用都来一遍。

**close\_shard() 不是可选的。** EdgeShard 出于性能考虑会把写操作在内存里缓冲。如果你的进程没调 close\_shard() 就退出了，那些缓冲的写可能压根没落到磁盘上。我们把它挂进 FastAPI 的关闭生命周期里，这样服务器停下来时它总能被干净地调到。

```python
@app.on_event("shutdown") 
def on_shutdown(): 
  close_shard()
```

### payload schema

每张被索引的照片，都作为一个向量加一份元数据 payload 存进 shard。与其把裸字典的构造散落在好几个文件里，我们在 *schema.py* 里用一个正经的 dataclass 把 payload 的形状定义一次。

```python
from dataclasses import dataclass, field
from typing import List, Optional@dataclass
class PhotoPayload:
    filename:str
    path:str
    tags: list[str] = field(default_factory=list)
    timestamp: Optional[str] = None
    width: Optional[int] = None
    height: Optional[int] = None        def to_dict(self) -> dict:
        return {
            "filename": self.filename,
            "path": self.path,
            "tags": self.tags,
            "timestamp": self.timestamp,
            "width": self.width,
            "height": self.height
        }
```

![用 HNSW 图做的 payload 索引可视化（来源：Qdrant）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/07.webp)

### 把 point 写进 shard

![Qdrant 中的数据结构（来源：Qdrant）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/08.webp)

```python
from qdrant_edge import PointStruct
from store.qdrant_client import get_shard
from schema import PhotoPayloadshard = get_shard()payload = PhotoPayload(
    filename  = "beach_sunset.jpg",
    path      = "/Users/me/Pictures/2024/Goa/beach_sunset.jpg",
    tags      = ["sunset", "beach", "outdoor"],
    timestamp = "2024-05-01T18:42:00",
    width     = 4032,
    height    = 3024
)point = PointStruct(
    id      = "3f7a2b1c-8e4d-4f9a-b2c1-7d8e9f0a1b2c",
    vector  = {"image": vector.tolist()},   
    payload = payload.to_dict()
)shard.upsert(points=[point])
```

有一个细节：向量被存成一个**命名向量** {"image": \[…512 个浮点数…\]}，与 EdgeConfig 里定义的 VECTOR\_NAME 键相对应。这是 Qdrant Edge 支持每个 point 拥有多个向量空间的方式（如果你以后想再加一个单独的文本描述 embedding 挨着图像 embedding，就很有用）。眼下我们只用一个，但从一开始就给它正确命名，意味着以后想加第二种向量类型时不必重新索引。

indexer 每 32 张刷一批，所以实际上每次调用 shard.upsert() 收到的是一个含 32 个 PointStruct 对象的列表，而不是一次一个，但每个 point 的结构都跟上面一样。

*无论是在标准 Qdrant 还是它的嵌入式、端上对应物 Qdrant Edge 里，Point 都是数据存储的基本单位。可以把它想成 SQL 数据库里一行、或 MongoDB 里一个文档的对应物，但完全为向量搜索做了优化。它由 3 个部分组成：一个唯一 id、一个向量、以及元数据（payload）。*

## 照片自动打标签

我在项目里还加了一个功能：照片自动打标签，因为何乐而不为！纯向量相似度固然强大，但它完全独立工作，而标签给了我们第二个可供过滤的维度，在你需要时让结果更锋利、更精准。

唯一的麻烦是：我可不想手动给任何一张照片打标签。所以我让系统在索引时就把这事干了！

### 思路：零样本分类

零样本分类听着挺玄，但直觉很简单。还记得 CLIP 怎么把图像和文本都映射进同一个 512 维向量空间吗？我们可以借这一点去问：*"这张图跟'沙滩'这个概念有多相似？"*——办法就是计算图像 embedding 和"沙滩"这个词的 embedding 之间的余弦相似度。

如果相似度越过某个阈值，这个标签就适用。不用训练，不用带标注的数据，不用微调。模型从它在数亿对图文上的预训练里，早就理解了这些概念。

我们在索引时，对一份 50 多个语义标签的词表跑这套，把越过阈值的标签直接存进照片的 Qdrant payload。

```
TAG_VOCABULARY = [
    "sunset", "sunrise", "beach", "ocean", "mountain", "forest", "city",
    "night", "snow", "rain", "fog", "sunny", "cloudy",
    "dog", "cat", "bird", "people", "crowd", "portrait", "selfie",
    "food", "coffee", "restaurant", "travel", "architecture",
    "car", "road", "nature", "flowers", "trees",
    "indoor", "outdoor", "party", "celebration", "sport",
    "screenshot", "document", "text", "map",
]
```

这不是一份固定的清单；它活在 *tools/tag.py* 里，灵活到你可以随意添加任意标签，而不必把所有东西重新索引一遍。你也可以只对已索引的照片重新打标签，根本不动它们的向量。

```python
from pipeline.embedder import embed_image, embed_text
from config import TAG_THRESHOLD
import numpy as npTAG_LABELS = [...]  

_label_vectors = {
    label: embed_text(label)
    for label in TAG_LABELS
}def generate_tags(image_path: str) -> list[str]:
    """
    Run zero-shot classification on an image.
    Returns a list of tags whose similarity to the image
    exceeds TAG_THRESHOLD (default: 0.20).
    """
    image_vector = embed_image(image_path)    tags = []
    for label, label_vector in _label_vectors.items():
        similarity = np.dot(image_vector, label_vector)  
        if similarity >= TAG_THRESHOLD:
            tags.append(label)    return tags
```

**预先算好标签 embedding 很重要。** 这 50 多个标签在不同照片之间是不变的。如果我们在循环里对每张图都调一次 embed\_text(label)，那一次库索引下来，同样这些词就要被反复 embedding 几千遍。我们换了个做法：在模块加载时把所有标签向量算一次，存进 \_label\_vectors，之后只从那儿跑点积。这是个小优化，但放到大规模下就有实打实的差别。

**0.20 这个阈值是故意定低的。** 我第一次看到时也吃了一惊。但对单个词的短标签做零样本分类，产出的相似度分数比完整自然语言查询要低。"沙滩"这个词里的语义内容，就是比"沙滩上柔和海浪间的日落"这句话少。阈值定得太高，意味着合理的标签会被漏掉。定在 0.20，你偶尔可能会在一张含义模糊的图上得到一个临界标签，但很少会漏掉一个货真价实的。如果你的库变吵了，就往上调；如果标签显得稀疏，就往下调。

### 索引时打标签 vs 查询时打标签

标签是在索引期间生成、存进 payload 的——不是在搜索时才算。这是个重要的区分。等到你发出查询时，每张照片早就把标签烤进去了。搜索之所以快，是因为所有昂贵的活都在前头一次性干完了。

generate\_tags 接进我们在第 5 节搭的 indexer 的位置在这里：

```python
def generate_tags_from_vector(img_vec: np.ndarray, threshold: float = 0.20, max_tags: int = 6) -> list[str]:
    """
    Generate tags for an image vector using zero-shot CLIP classification.
    Tags with cosine similarity above threshold are included (up to max_tags).        This is a utility function used for generating tags during indexing
    or when you already have an image vector.
    """
    tag_vecs = _get_tag_vectors()        scores = {
        tag: float(np.dot(img_vec, vec))   
        for tag, vec in tag_vecs.items()
    }        tags = sorted(
        [t for t, s in scores.items() if s >= threshold],
        key=lambda t: scores[t],
        reverse=True,
    )[:max_tags]        return tags
```

### 把标签当过滤器用

标签一旦和向量搜索结合起来，就真正变得有用了。Qdrant Edge 让我们在相似度搜索之外，还能按 payload 字段过滤，于是不再只是*"找出跟'日落'语义上最相似的 10 张图"*，我们可以问*"找出跟'日落'语义上最相似、同时打了 outdoor 标签、又没打 blurry 标签的 10 张图"*。

我们用一种过取（over-fetch）策略来实现这件事：取回比需要多 5 倍的结果，再在 Python 里按请求的标签过滤，返回通过筛选的前 k 个。这让查询逻辑保持简单，同时确保你总能拿回你要求的那个数量的结果：

```python
def search_photos(query: str, top_k: int = TOP_K, tags: list[str] = None) -> list[dict]:
    
    
    
    print(f"[search] Received query='{query}' with tags={tags} and top_k={top_k}")
    shard = get_shard()
    query_vector = embed_text(query)        
    
    over_fetch_multiplier = 5 if tags else 1
    fetch_limit = top_k * over_fetch_multiplier        results = shard.query(
        QueryRequest(
            query=Query.Nearest(query_vector.tolist(), using=VECTOR_NAME),
            limit=fetch_limit,
            with_vector=False,
            with_payload=True,
        )
    )
    print(f"[search] Found {len(results)} initial hits for query='{query}' with tags={tags}")        hits = []
    untagged_hits = []          for point in results:
        payload = point.payload or {}
        point_tags = payload.get("tags", [])                result_dict = {
            "path": payload.get("path"),
            "filename": payload.get("filename"),
            "tags": point_tags,
            "timestamp": payload.get("timestamp"),
            "score": round(point.score, 4)
        }                
        
        
        if tags:
            if point_tags and any(t in point_tags for t in tags):
                
                hits.append(result_dict)
            elif not point_tags:
                
                untagged_hits.append(result_dict)
        else:
            
            hits.append(result_dict)                
        if len(hits) >= top_k:
            break        
    if tags and len(hits) < top_k:
        hits.extend(untagged_hits[:top_k - len(hits)])        return hits[:top_k]
```

实际用起来，这感觉很自然。*"给我看那些户外、不糊的日落照片"* 是个完全合理的要求，系统照办就是，你完全不需要懂底层的过滤是怎么运作的。

*在 **Qdrant Edge**（以及标准 Qdrant 引擎）里，跑一个把**自然语言查询**和**标签过滤**结合起来的混合搜索，靠的是一种叫**单阶段过滤**（Single-Stage Filtering）的架构。不同于两阶段做法（先找匹配的标签再搜向量，或反过来），Qdrant 在图遍历期间**同时**对向量索引和元数据 payload 索引求交集。这既保证了高速，又防止精度损失，哪怕在资源受限的边缘设备上也是如此。*

![带标签过滤的自然语言搜索查询](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/09.webp)

## 搭建搜索 agent

当你输入*"周围没人的沙滩日落"*时，发生的是这些：

1.  你的查询过一遍 CLIP 的文本编码器 → 一个 512 维向量。
2.  Qdrant 计算这个向量和 shard 里每个图像向量之间的余弦相似度。
3.  最相似的那些回来，按分数排序。
4.  可选的标签过滤进一步收窄结果。

```bash
curl --location 'http:
--header 'Content-Type: application/json' \
--data '{"query": "eiffel tower from rooftop","tags":[], "top_k": 1}'
```

响应：

```json
{
    "query": "eiffel tower from rooftop",
    "results": [
        {
            "path": "/Users/vatsalasingh/Documents/Datasets/tag_phot/photo-1638051017225-0d9fcca18cf4.jpg",
            "filename": "photo-1638051017225-0d9fcca18cf4.jpg",
            "tags": [
                "cloudy",
                "city",
                "rain",
                "screenshot",
                "architecture",
                "travel"
            ],
            "timestamp": "2021-12-09T17:27:56",
            "score": 0.2864
        }
    ]
}
```

![响应图像](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/10.jpg)

### 优雅地处理边缘情况

跟真实照片库打交道时，实践中会冒出来几件事：

**低置信度结果。** 如果你的库里确实没有你要找的东西，最靠前的结果照样会被返回，只是分数很低。把分数也亮给用户是值得的，好让他们自己判断相关性，而不是把每个结果都摆成一副信心十足的匹配的样子。

**很短的查询。** 像"food"这种单词查询能用，但它撒的网很大。文本编码器能用的料更少，产出的向量也更通用。更长、更具描述性的查询——*"木桌上的自制意面"*——往往能产出锋利得多的结果，因为查询向量更丰富、更具体。

**实际上是标签过滤的查询。** *"给我看我所有的截图"* 其实算不上语义搜索；它是一次标签查找。你可以在 agent 层识别出这种模式，直接路由到一个标签过滤查询，而不是向量相似度搜索，这样既更快又更精准。

## 用 OpenClaw 把一切编排起来

我们有了一条能跑的 embedding 管线、一个本地向量数据库、语义搜索，还有自动打标签。每一块都独立工作，并且都能通过一个干净的 REST 端点访问。但眼下，要用这套系统，就得知道该调哪个端点、带哪些参数、按什么顺序。

对一个在 Swagger UI 里随手戳戳的开发者来说，这没问题。但对一个你真想天天用的东西来说，这就不行了。

OpenClaw 就在这里登场。它把我们搭的一切包进一个对话式界面，于是你不必再去构造 API 调用，只管描述想要什么，剩下的 agent 自己琢磨。

### OpenClaw 怎么工作

OpenClaw 围绕一个简单却强大的想法构建：给一个语言模型一份可用工具的描述，让它根据用户的话决定调用哪些。

你定义好你的工具——名字、描述、参数——OpenClaw 负责意图识别、工具派发、结果解读和回应格式化。它还会跨多轮维护对话上下文，于是像*"现在把那些只筛出户外的"*这样的追问能自然地生效，你不用重复原来的查询。

配置是基于 markdown 的，这让一切保持可读、易改，不必碰任何 Python。

```python
@app.post("/chat")
def chat(req: ChatRequest):
    """
    Conversational endpoint. Accepts user message and conversation history,
    returns agent's reply after processing with tools.
    """
    
    def search_tool(query: str, top_k: int = 10, tag_filter: list = None):
        """Search photos by natural language query"""
        return search_photos(query=query, top_k=top_k, tags=tag_filter)        def duplicates_tool(threshold: float = 0.97):
        """Find duplicate or near-duplicate photos"""
        return find_duplicates(threshold=threshold)        def tag_tool(image_path: str):
        """Generate and update tags for a specific photo"""
        return generate_tags_from_vector(image_path=image_path)        
    agent = Agent(
        tools=[search_tool, duplicates_tool, tag_tool],
        system_prompt="""
        You are a personal photo assistant. You help users search, organize,
        and understand their local photo library. You have access to tools
        for semantic search, duplicate detection, and tagging.                When helping users:
        - Use the search tool to find photos by describing their content
        - Use duplicates tool to find and clean up duplicate shots
        - Use tag tool to inspect or update tags for specific photos                Always be concise and helpful. When returning photo results, 
        format them clearly with filenames, similarity scores, and tags. 
        Use emojis sparingly but helpfully.
        """
    )        
    response = agent.chat(
        message=req.message,
        history=req.history
    )
    return {"response": response}
```

### 真实的交互流程

让我给你看看跟 agent 的实际对话长什么样，因为整套系统正是在这里咔哒一声归位。

![照片搜索 agent 的截图](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/11.webp)

### 为什么 agent 这一层重要

我想在这儿退后一步，因为我觉得当你深陷实现细节时，这一点很容易被忽略。

我们搭的那几个工具——搜索、打标签、去重——单看都各有用处。但它们最有用的时候，是协同工作之时，而对某个给定请求，决定*该怎么*把它们组合起来，是真正不平凡的事。像*"清理一下我上个月的相机胶卷"*这样的查询，该触发一次去重搜索、一次按时间过滤的语义搜索，还是两者都来？该不该在建议删除前先请求确认？

这些都是判断题。而 agent 这一层，正是这些判断题安放的地方，跟工具实现本身干净地分了家。如果你想改 agent 对一个请求的推理方式，你就去更新系统提示词或工具描述。你不必碰搜索逻辑，也不必碰 Qdrant client。

正是那份分离，让这套系统可扩展。加一项新能力——给截图做 OCR、人脸聚类、或视频帧索引——意味着写一个新工具，再把它注册给 agent。编排逻辑会自动适应。

## 一切都在本地跑——以及为什么这很重要

到这一步，系统已经完全搭好了。在收尾几节之前，咱们先拉远一点，因为我觉得这个项目本地优先（local-first）的本质，值得更多讨论。

### 没有任何东西离开你的设备。就这么简单。

这套栈里的每个组件——CLIP 模型、Qdrant Edge shard、FastAPI 服务器、OpenClaw agent——都完全跑在你的机器上。你索引一张照片时，那些像素从不碰到一个网络套接字。你搜索时，查询在本地被处理。agent 回应时，它处理的完全是活在你磁盘上的数据。

对一个个人照片库来说，这件事比看上去更重要。你的相册不只是度假快照；它是私人对话的截图、为留底拍下的医疗文件、财务收据、与家人共度的瞬间。那些数据流经第三方云索引服务、被存在别人的基础设施上、给别人的模型训练管线供能——光想想就该让人不舒服。然而这恰恰是每一家主流云照片服务正在做的事，就藏在某处的细则里。

这套系统不会这样。你的数据是你的。

## 跑这套东西到底要花什么

我想给你诚实的数字，而不是乐观的基准测试，因为真实硬件千差万别。

**首次索引**（一次性）：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/12.webp)

如果你的库很大，就让首次索引整夜跑着。你这辈子只需要做这一次。

**搜索延迟**（索引之后）：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/13.webp)

端到端 100ms 以内。那感觉就是即时的。

**运行时内存足迹：**

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@357d7cafd61c0736b8a7a3c12e81ba1e04e73a3b/ai-insights/2026-06/02/images/how-i-turned-my-photo-gallery-into-an-autonomous-ai-agent-the-complete-guide/14.webp)

在任何 8GB 内存的机器上都很从容。4GB 上、不开别的东西也跑得动。

**磁盘存储：**

每张被索引的照片大约给 Qdrant shard 添 2–3KB（512 个浮点数 × 4 字节 + payload 开销）。一个 10,000 张照片的库占用约 30–50MB 的向量存储。你的原始照片纹丝不动；shard 纯粹是索引，不是你图像的副本。

## 接下来——扩展这套系统

我们搭的这套系统已经货真价实地有用了：语义搜索、自动打标签、去重检测、智能相册，全都在本地跑，零云依赖。但同样这套架构，还能自然地朝几个有意思的方向延伸，不必推倒重来、也不必引入任何云依赖。同一条 embedding 管线，同一个 Qdrant shard，同一个 agent 层——只是新工具挂在同一块地基上。

下面是我正在积极琢磨的几个扩展。

### 人脸聚类

不按场景或物体来搜，而是按照片里的人来聚类——并且从头到尾不手动给任何一张脸打标签。在你的库上跑一个人脸检测模型，抽出人脸 embedding，按相似度把它们聚成簇，你最后就得到一组组很可能对应你生活里反复出现的那些人的分组。一次打标签把这些簇命名好，从那以后 agent 就知道谁是谁了。不用云端人脸识别 API，没有生物特征数据离开你的设备。

### 给截图和文档做 OCR

大多数手机相册里有一大块是截图——食谱、地址、航班信息、对话、收据。这些内容对一个视觉 embedding 模型是隐形的，因为 CLIP 理解的是视觉语义，不是文字内容。在索引时跑 OCR，把抽出的文字存进 payload，就把这件事彻底改变了。截图忽然就能按其实际内容被搜到，你也得到一个真正的混合检索系统：对照片用向量相似度，对文档用文本匹配，两者落在同一个结果集里。

### 视频帧索引

按固定间隔从视频片段里采样帧，把每一帧当成图像 embedding，再把它们和你的照片一起索引进 Qdrant，payload 里带上父视频的文件名——以及定位的时间戳。搜索时，视频帧会和照片浮现在同一个结果集里。不需要转写。就让帧活在跟其他一切相同的 embedding 空间里，让你整卷相机胶卷——不只是那些静态照片——都变得可搜索。

### 混合搜索：向量 + 关键词 + 元数据

眼下搜索是基于向量、再叠一层标签过滤。自然的演进是一个像样的三路混合：用语义相似度抓含义，用关键词/OCR 匹配抓文档内容，用结构化元数据过滤器抓诸如日期范围、朝向、位置之类的约束。每个过滤器在排序前先收窄候选集。结果是一种纯向量搜索单凭一己之力达不到的那种精准的检索。

### 时间与位置感知的检索

大多数手机相机会通过 EXIF 元数据把 GPS 坐标和精确时间戳嵌进每一张照片。在索引时把这些抽出来存好，就白白给了你一层丰富的结构化过滤，能驱动像*"MG Road 5 公里以内的照片"*或*"我生日那一周之后的所有东西"*这样的查询，完全不用任何语义搜索。再和向量相似度结合，它就让你更接近某种感觉像真正的记忆检索、而非只是照片搜索的东西。

这些没有一个需要不同的架构。每一个都是加到同一个 agent 上的新工具，把新数据索引进同一个 Qdrant shard，用同一个 CLIP embedding 空间当地基。这正是我觉得最令人满足的部分——不是说一切都已建好，而是加下一样东西的感觉是自然的，而不是痛苦的。

## 写在最后

我带着一个简单的挫败感开始这个项目：我找不到一张明明知道自己拍过的照片。我以一样自己每一天都在用的东西结束了它。

但比起工具本身，我更想留给你的是它背后的想法。我们被非结构化的个人数据包围着——照片、截图、语音备忘、文档——而能帮我们理出头绪的工具，要么锁死在专有生态里，要么原始到约等于不存在。今天开源 AI 工具所能做到的，和大多数人实际能用上的，这两者之间的鸿沟大得惊人。

这个项目，是弥合那道鸿沟的一次小小尝试。

如果你用它搭出什么东西，或者把它往一个我没想到的方向扩展，我是真心想听听。把这些写出来、而不是自己藏着，整个意义就在这儿。这套系统最有意思的版本，多半是我还没想象到的那些——由拥有不同的库、不同的问题、不同约束的人搭出来。

留个评论，找我聊聊，或者干脆就把它搭起来，看看它会走向哪儿。

下次再见。

## 参考资料与延伸阅读

-   [Qdrant Edge 文档](https://qdrant.tech/documentation/guides/qdrant-edge/)
-   [Qdrant 文档](https://qdrant.tech/documentation/)
-   [OpenClaw GitHub](https://github.com/openclaw)
-   [OpenAI CLIP 研究论文](https://arxiv.org/abs/2103.00020)
-   [FastEmbed——Qdrant 的轻量 embedding 库](https://github.com/qdrant/fastembed)
-   [Hugging Face Transformers](https://huggingface.co/docs/transformers)
