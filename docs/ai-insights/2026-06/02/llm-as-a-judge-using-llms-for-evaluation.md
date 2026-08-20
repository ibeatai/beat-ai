---
title: "LLM-as-a-Judge：用大模型来做评估"
author: Cameron R. Wolfe, Ph.D.
url: https://cameronrwolfe.medium.com/llm-as-a-judge-using-llms-for-evaluation-754a7340fc7b
translated: 2026-06-02
excerpt: LLM-as-a-Judge，以及在人工质量评分之外其他可扩展的补充手段……
tags:
  - Artificial Intelligence
  - LLM
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/01.thumb.webp
---

# LLM-as-a-Judge：用大模型来做评估

LLM-as-a-Judge，以及在人工质量评分之外其他可扩展的补充手段……

![（来自 \[13, 16, 17\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/01.webp)

本文最初发表于[我的 Substack](https://cameronrwolfe.substack.com/p/llm-as-a-judge)。

## 引言

LLM 越来越强，随之而来的一个最棘手的难题，就是该怎么正确评估它们。如今有一大批能力出众的模型，每一个都能解决五花八门、复杂且开放的任务。结果就是，要分辨这些模型之间的性能差异变得很难。评估 LLM 最可靠的办法是靠人类反馈，但从人那里收集数据本身就有噪声、费时又烧钱。人工评估虽然是衡量模型能力时一份珍贵且不可或缺的真值来源，可一旦*只靠它单打独斗*，就会拖慢模型研发期间快速迭代的脚步。要解决这个问题，我们需要一种又快、又省钱、又简单的评估指标，同时还得和人工评估的结果保持高度相关。

> “人工评估是衡量人类偏好的黄金标准，但它慢得出奇、贵得离谱。为了让评估自动化，我们尝试用 GPT-4 这类最先进的 LLM 来替代人类。” —— 来自 \[17\]

讽刺的是，正是 LLM 不断攀升的能力，给这个评估难题带来了一个潜在解法：我们可以拿 LLM 本身来做评估，这套思路通常被称作 LLM-as-a-Judge \[17\]。这项技术最早是在 GPT-4 发布之后才被人探索的——*GPT-4 是第一个有能力评判其他模型输出质量的 LLM*。从那以后，一系列论文相继剖析了 LLM-as-a-Judge，挖掘出落地的最佳实践，也指出了几处我们必须警惕的偏差来源。在这篇综述里，我们会逐一审视其中不少论文，对 LLM 评估建立起一套深入而实用的理解。

## 什么是 [LLM-as-a-Judge](https://arxiv.org/abs/2306.05685) \[17\]？

评估文本序列质量的传统指标已有很多。这些指标分为基于参考（reference-based）和无参考（reference-free）两类，区别在于衡量质量时是否需要一条“标准答案”序列作参照。在机器翻译、摘要这类更窄的任务上，它们表现不错；细节可见[这里](https://cameronrwolfe.substack.com/i/144374854/how-can-we-evaluate-a-summary)。但现代 LLM 解决的是多样、开放的任务，而且经过了大量基于人类偏好的对齐，这种特性用老一代 NLP 基准很难捕捉。换到这种场景下，传统指标往往就失灵了，已被证明与人类偏好的相关性很差。

> “LLM-as-a-judge 是一种可扩展、可解释的方式，用来逼近原本极其昂贵才能拿到的人类偏好。” *—— 来自 \[17\]*

LLM-as-a-judge 是一种无参考指标，直接提示一个强力 LLM 去评估另一个模型输出的质量。尽管有种种局限，这项技术却被发现能持续与人类偏好保持一致，同时还能以可扩展的方式评估各类开放任务，且几乎不用改动实现。要评估一个新任务，*我们只需调一调 prompt*！这个指标在 GPT-4 发布后被提出，此后人气一路走高，最终在 \[17\] 中迎来一篇对 LLM-as-a-judge 指标的深度剖析。如今，LLM-as-a-judge 与人工评估并列，是 LLM 最常用的评估手段之一——*它尤其擅长评估模型与人类偏好的对齐程度。*

### MT-Bench 与 Chatbot Arena

![（来自 \[17\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/02.webp)

为了扩充可用于衡量 LLM 在开放式对话应用中表现的基准，\[17\] 的作者构建了两份用于评估人类偏好的数据集。**MT-bench** 数据集是一组固定的 [80 道高质量题目](https://huggingface.co/datasets/HuggingFaceH4/mt_bench_prompts)。这些题目横跨八个门类，主攻多轮对话和指令遵循能力——可以说，这正是基础 LLM 最重要的两项技能。上图给出了 MT-bench 的题目示例。

![Chatbot Arena 界面截图（来自 \[17\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/03.webp)

第二份数据集叫 **Chatbot arena**，与其说是数据集，不如说是个平台。这个竞技场是一个众包对战平台，用户可以同时和两个匿名 LLM 交互，再挑出更好的那个，见上图。平台不预设题目，而是让用户自己提问，两个 LLM 各自给出回答，从而能在五花八门的使用场景下收集数据。为了避免偏差，模型身份会一直保密，直到用户给出偏好之后才揭晓。\[17\] 的作者从 MT-bench 和 Chatbot Arena 收集了大量人类反馈，用来检验 LLM-as-a-Judge 与人类偏好的相关性。

**Chatbot Arena 排行榜。** 利用从竞技场收集来的人类偏好，我们可以算出 [Elo 分数](https://en.wikipedia.org/wiki/Elo_rating_system)，按人类偏好给模型排名。如今，Chatbot Arena 上已经分享了 100 多个模型、超过 150 万对成对偏好数据，成了被引用最多的 LLM 排行榜之一，见[这里](https://chat.lmsys.org/)。想了解更多，可以读读 [Nathan Lambert](https://www.natolambert.com/) 就这个话题写的[文章](https://www.interconnects.ai/p/chatbotarena-the-future-of-llm-evaluation)。

### LLM-as-a-Judge 的几种设置

![各式各样的 LLM-as-a-Judge prompt（来自 \[17\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/04.webp)

相比人工评估，LLM-as-a-Judge 是一种简单又可扩展的替代方案，它能 *i)* 减少评估过程中对人的依赖，*ii)* 让模型迭代更快。要用 LLM-as-a-Judge 跑评估，*我们要做的只是写一个 prompt*！不过，常见的 prompt 结构有好几种（见上图）：

1.  *成对比较（Pairwise comparison）*：给评判者一道题和两份模型回答，让它指出更好的那份。
2.  *逐点打分（Pointwise scoring）*：给评判者一道题及单份回答，让它打个分；比如用一到五分的 [Likert 量表](https://en.wikipedia.org/wiki/Likert_scale)。
3.  *参考引导打分（Reference-guided scoring）*：除题目和回答外，再给评判者一份参考答案，辅助打分。

以上任意一种都能搭配[思维链（CoT）提示](https://cameronrwolfe.substack.com/p/chain-of-thought-prompting-for-llms)来提升打分质量。做法很简单：用一个[零样本 CoT prompt](https://arxiv.org/abs/2205.11916)，在评判 prompt 后面追加一句类似*“请逐步写出你打分的解释”*即可。但要注意，按 \[16\] 的建议，得让 LLM 先输出理由、再输出分数（而不是反过来）。

> “模型先得出结论、再补出的解释，并不能真正支撑这个结论。” *—— 来自 \[16\]*

让 LLM 在给分的同时输出一段人类可读的理由，是个既省事又好用的可解释性小技巧。我们可以借这些解释更深入地了解模型的表现和短板，见下图。

![（来自 \[17\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/05.webp)

**该用哪种设置？** 每种打分策略都有利有弊——*没有哪一种是绝对最优的*。比如成对比较不可扩展，因为它要求把所有模型输出两两组合都比一遍。可逐点打分往往[更不稳定](https://x.com/aparnadhinak/status/1748368364395721128)，因为它指望评判者内部有一套相对一致的打分尺度——*绝对分数比成对比较更容易上下波动*。一般来说，到底该用哪种 LLM-as-a-Judge 评估方式，取决于具体应用的细节。我们手头并不总有两个模型可比，那种情况下逐点打分往往更合适，反之亦然。

![（来自 \[17\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/06.webp)

**它管用吗？** 在 \[17\] 里能清楚看到，GPT-4 这样的强力 LLM 评判者能相当准确地衡量人类偏好。事实上，GPT-4 与人类偏好评分的一致率达到了 80%，*这和人类标注者自己跟自己的一致率持平*，见上图。LLM 评判者能这么准地预测人类偏好，其实并不意外——大多数现代 LLM 本就在人类偏好数据上做过大量微调。

### 偏差（以及我们该怎么规避……）

> “我们识别出了 LLM 评判者的偏差和局限。但我们……也表明，尽管有这些局限，LLM 评判者与人类的一致率依然很高。” *—— 来自 \[17\]*

LLM-as-a-Judge 虽然能准确预测人类偏好，可这套评估策略并不完美——*它会给评估过程引入好几种新的偏差来源*。我们早就知道 LLM 有一堆毛病，比如推理能力存疑、对 prompt 的细微改动很敏感、爱生成啰嗦冗长的输出。这些弱点不少都会在 LLM-as-a-Judge 评估里催生对应的偏差：

1.  *位置偏差（Position bias）*：评判者可能因为输出在 prompt 中的位置而偏向某一份（比如成对 prompt 里的第一份回答）。
2.  *冗长偏差（Verbosity bias）*：评判者可能因为输出更长而打更高分（也就是越长的回答分越高）。
3.  *自我增强偏差（Self-enhancement bias）*：评判者倾向偏袒自己生成的回答（比如 GPT-4 给自家输出打高分）。

除了上面列的这几种，LLM 评判者在给那些它自己都答不好的题目（比如复杂推理题和数学题）打分时，往往也力不从心。还有一点，*评判者很容易被上下文里的错误信息带偏* \[18\]。如果被打分的某份回答本身就是错的，评判者可能被这段上下文误导，给出不准的分数。

![（来自 \[17\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/07.webp)

**把偏差再往深里挖。** 除了 \[17\] 列出的这些偏差来源，还有很多其他工作——*其中有些我们会在本文后面探讨*——深入剖析过 LLM-as-a-judge 评估中的偏差。下面列出这些工作，方便查阅和延伸阅读：

-   Humans or LLMs as the Judge? A Study on Judgement Biases \[[链接](https://arxiv.org/abs/2402.10669)\]
-   Evaluation Biases for Large Language Models \[[链接](https://arxiv.org/abs/2307.03025)\]
-   Cognitive Biases in Large Language Models as Evaluators \[[链接](https://arxiv.org/abs/2309.17012)\]
-   Large Language Models are Inconsistent and Biased Evaluators \[[链接](https://arxiv.org/abs/2405.01724)\]
-   Large Language Models are Not Yet Human-Level Evaluators \[[链接](https://arxiv.org/abs/2305.13091)\]

**怎么减轻偏差？** 要削弱偏差对 LLM-as-a-Judge 评估结果的影响，我们有几招可用：

-   随机打乱模型输出在 prompt 里的位置，生成多个分数，再对不同位置下的分数取平均——下文称之为*位置互换技巧（position switching trick）*。
-   提供[少样本示例](https://cameronrwolfe.substack.com/i/143156742/basic-prompting-strategies)，演示分数的自然分布，帮评判者校准它内部的打分尺度。
-   在 prompt 里为难度较高的数学题和推理题附上正确答案，评估时给评判者当参考。
-   用好几个不同模型当评判者（比如 Claude、Gemini 和 GPT-4），减轻自我增强偏差的影响。

这些招数固然有用，但 LLM-as-a-judge 终究是个有缺陷的指标——[所有指标都一样](https://en.wikipedia.org/wiki/All_models_are_wrong)——永远不可能完美。所以我们要始终对这些偏差保持警觉，想清楚它们会怎么影响我们的分析。多琢磨一下：被评估的是哪些模型、我们想测的到底是什么、评估是怎么搭起来的，以及底层那个评判者会从哪些角度把评估结果带偏。

## LLM 评估的早期工作

在 \[17\] 提出并剖析 LLM-as-a-Judge 之前，已有一批更早的工作研究过类似技术。这些研究都始于 GPT-4 的问世——它是第一个强到足以评估文本质量的 LLM。我们接下来会看到，这套方法凭借易用、通用又有效，很快在社区里火了起来、四处蔓延。

### [Sparks of Artificial General Intelligence: Early experiments with GPT-4](https://arxiv.org/abs/2303.12712) \[1\]

要让 LLM 驱动的评估成为可能，我们首先得有一个足够强大、能可靠评判其他模型输出的 LLM。此前的模型固然也令人印象深刻，但直到 GPT-4 出现，我们才有了这样一个模型。而就在这个模型刚一问世，研究者们立刻动手探索 LLM 评判者究竟可不可行！

![（来自 \[1\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/08.webp)

**GPT-4 到底有多强？** 第一篇探索用 GPT-4 当评估者的工作 \[1\]——*发表时距模型发布还不到十天*——其实重点并不在评估上。它的目标更宽泛，是想摸清 GPT-4 的各项能力，最终写成了一份长达 155 页的分析，覆盖的话题之广（令人咋舌）：

-   解决编程题和数学题的能力。
-   使用工具、与人交互的能力；比如[心智理论问题](https://arxiv.org/abs/2302.02083)，或者模型向人类解释自身输出的能力。
-   用 [TikZ](https://www.overleaf.com/learn/latex/TikZ_package) 画基础图形/图片、生成有用的图表，以及做更宽泛的数据分析。
-   证明数学定理，甚至能一边证明、一边让每一行都押韵，见上图。

这份分析的结论是：GPT-4 几乎在所有考察的任务上都表现出色，比 ChatGPT 有了实质性的飞跃。作者甚至发现，GPT-4 的输出在很多情况下与人类难分高下（甚至更好），还进一步断言 GPT-4 是迈向通用人工智能（AGI）的重要一步，见下图。

> “GPT-4 能力的通用性……加上它在一大批任务上达到甚至超越人类水平的表现，让我们敢于说 GPT-4 是迈向 AGI 的重要一步。” *—— 来自 \[1\]*

作者关于 GPT-4 展现出（通用）智能迹象的论断，当时——*而且至今*——都极具争议。其实，围绕 LLM 能力和 AGI 进展的争论，往往归根结底是对这些概念缺乏（或各执一词的）严谨定义。*连智能都还没定义，又怎么去衡量它？* 好在就本文而言，我们不必操心 AGI 该怎么定义。我们只需知道：GPT-4 是个能力极强的模型，而这些能力为“用 LLM 评估其他 LLM 的输出”打开了一扇又一扇门。

![（来自 \[1\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/09.webp)

**GPT-4 充当评估者。** \[1\] 的作者是第一批探索用 GPT-4 当评判者的人，可这部分分析其实相当简短（前后还不到一页）！在先指出现有评估指标在判断语句相似度时的局限之后，作者考察了 GPT-4——*用的就是上图那个 prompt*——评判一份模型回答与参考答案相似度的能力。具体来说，是让 GPT-4 判断某份模型回答更接近参考答案，还是更接近用 GPT-3 生成的答案。评判者会拿到对同一语句的两份回答，再指出哪一份更好地还原了原始语句。

> “\[GPT-4\] 能判断一对答案中哪一份更接近标准答案，而且这个判断与人类做同样任务的结果大体吻合。” *—— 来自 \[1\]*

从这份分析能看出，GPT-4 在判断语句间语义相似度上，远比 ROUGE 或 BLEU 这类简单指标强。为了提升评估质量，作者让 GPT-4 在选出更优输出之前，先列出每份回答的利弊。可一旦把 GPT-4 的判断拿来和人类对比，就会发现明显的差异。比如，GPT-4 在 87.76% 的情况下偏爱 GPT-4 生成的回答，而人类评估者只有 47.61%，见下表。

![（来自 \[1\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/10.webp)

GPT-4 与人类评估者的一致程度偏低——*只略高于 50%*——而且我们在 \[1\] 里已经看到一些明显的偏差迹象，比如更长的回答会被 GPT-4 打更高分。这种对不齐，可能源于评估设置的差异——GPT-4 被迫在两份回答里选出一个赢家，而人类可以选平局。但即便如此，GPT-4 与人类的一致程度还是低得出人意料，于是 \[1\] 的作者得出结论：要校准 GPT-4 的评估能力，还需要更多研究。

### [Vicuna: An Open-Source Chatbot Impressing GPT-4 with 90%\* ChatGPT Quality](https://lmsys.org/blog/2023-03-30-vicuna/) \[2\]

> “评估聊天机器人从来都不是件简单事。随着 GPT-4 近来的进展，我们很好奇它的能力是否已达到接近人类的水平，能够支撑起一套自动化评估框架。” *—— 来自 \[2\]*

Vicuna 是一个开源聊天机器人，做法是拿 [LLaMA](https://cameronrwolfe.substack.com/p/llama-llms-for-everyone)\-13B，在一批用户与 ChatGPT 的对话上做微调——*这些对话采集自* [*ShareGPT*](https://sharegpt.com/)。我们在[之前的一篇综述](https://cameronrwolfe.substack.com/i/114077195/vicuna-an-open-source-chatbot-with-chatgpt-quality)里讲过 Vicuna 的细节。不过它之所以与本文相关，是因为作者选择主要用 GPT-4 来自动评估这个模型输出的质量，见下图。

![（来自 \[2\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/11.webp)

事实上，Vicuna 是最早做这类 LLM 驱动评估的工作之一，当时还招来过一片质疑。但 \[2\] 用 GPT-4 做评估这件事，反倒掀起了一波对该技术的分析热潮，原因有三：

1.  评估结果看起来相对一致，也很有前景！
2.  用 GPT-4 当评估者是一种无参考、自动化的评估策略，可套用到任何任务上（也就是非常通用、简单）。
3.  GPT-4 能在评估之外附上一段理由，让评估结果对人更易解读。

**评估设置。** 用来测试 Vicuna 的题目横跨八个类别，见[这里](https://github.com/lm-sys/vicuna-blog-eval/blob/main/eval/table/question.jsonl)。题目共 80 道，借助 GPT-4 编写而成。作者发现，只要精心设计 prompt，GPT-4 就能为最先进的聊天机器人生成有挑战性的题目。要评估不同聊天机器人在这些题目上的答案，作者会收集每个模型的输出，让 GPT-4 从有用性、相关性、准确性和详尽程度几方面给生成的输出打分。所用的评估 prompt 简单得出人意料，可见[这里](https://github.com/lm-sys/vicuna-blog-eval/blob/main/eval/table/prompt.jsonl)。考虑到编程和数学任务评估起来更难，作者另用了专门、更具体的 prompt，见下图。

![用于 GPT-4 评估的 prompt](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/12.webp)

GPT-4 被要求对来自不同模型的两份回答按一到十分打分，并为这些分数给出解释。通过 CoT 提示产出这样一段解释，往往能提升 GPT-4 评分的准确度，同时还附带一段人类可读的理由来佐证打分。关于这些 prompt，我们还能观察到几点有用的细节：

-   通用 prompt 要求 GPT-4 在打分时避免偏差——*包括位置偏差*——这说明 \[2\] 的作者确实遇到过位置偏差的麻烦。后续工作 \[8\] 证实，Vicuna 的评估策略存在很强的位置偏差，但靠位置互换就能解决。
-   所有 prompt 都明确规定了输出格式，这样分数就能轻松、自动地从回答里解析出来。这种做法只有配上 GPT-4 这种强力指令遵循模型才行得通。
-   相比用于通用题目的 prompt，编程和数学评估的 prompt 更详尽，引入了更多深入、针对具体问题的细节，以提升这类题目的打分质量。
-   编程和数学的 prompt 还用了几个小技巧来诱导更好的打分，比如让 GPT-4 先点评给定的解法、或自己先把题做一遍，再给分。

**Vicuna 独特的做法。** 在一个 prompt 里给两份回答打分，是如今常见的标准做法，因为它能在模型之间做更好的相对比较——*我们可以轻松让 GPT-4 解释，对某道题来说两个模型里哪个回答更好*。不过要注意，Vicuna 的设置和标准的成对比较略有不同。它不是直接问模型更偏爱哪份回答，而是提示模型给每个样本各打一个分，再根据这些分数定出更优的那一份。这是另一种有效的 LLM-as-a-Judge 设置，已有好几篇论文用过。

**这管用吗？** 尽管 GPT-4 在给编程和数学题打分时不太行，这些评估的结果还是相对一致，而且附带详细解释！基于这些评估，作者发现 Vicuna 在 90% 的题目上比其他开源模型更受偏爱，并且在 45% 的情况下被评为好于或等同于 ChatGPT，见下图。但后续工作 \[8\] 揭示，这些评估偏向更长的输出，而且与人类偏好的相关性（多少有点）偏弱。所以解读这些结果时，我们得留意这些偏差和短板。

![GPT-4 给出的回答对比（来自 \[2\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/13.webp)

更具体地说，我们在[后续工作](https://arxiv.org/abs/2305.15717)里看到，Vicuna 这类模仿型模型学到了 ChatGPT 这种强力模型的风格，却缺乏它们的事实准确性和知识储备。话虽如此，\[2\] 的分析仍然很有价值，因为 *i)* 我们看到，在更强模型的输出上做微调，能改善开源模型的回答风格和指令遵循能力；*ii)* LLM 驱动评估的可行性得到了更清楚的印证。

> “这套提出的评估框架虽然展现了评估聊天机器人的潜力，但它还算不上严谨或成熟……为聊天机器人开发一套全面的评估体系，仍是一个悬而未决的问题。” *—— 来自 \[2\]*

**展望。** 尽管 \[2\] 里的 GPT-4 评估尚属雏形，这项工作却为后续 LLM 驱动评估的分析打下了坚实基础。后来的工作对 LLM-as-a-Judge 技术的长短处给出了不少有用分析，但用来产出可靠分数的 prompt，和 \[2\] 里见到的（相对而言）大同小异！事实上，LLM-as-a-Judge 那篇论文本身 \[17\] 就出自 Vicuna 同一批作者之手，大量沿用了 \[2\] 里提出的诸多技巧。再加上，那套用来评估 Vicuna 的 80 道题，也被其他论文广泛采用。

### [AlpacaEval: An Automatic Evaluator of Instruction-Following Models](https://github.com/tatsu-lab/alpaca_eval) \[8\]

AlpacaEval \[8\] 最早于 2023 年中提出，是面向指令遵循语言模型最受欢迎的基于 LLM 的自动化评估指标（和[排行榜](https://tatsu-lab.github.io/alpaca_eval/)）之一。它的评估策略——*基于* [*AlpacaFarm*](https://crfm.stanford.edu/2023/05/22/alpaca-farm.html) *\[9\]，一个用 LLM 评估者来自动生成* [*RLHF*](https://cameronrwolfe.substack.com/p/the-story-of-rlhf-origins-motivations)*式成对偏好标签的模拟器*——使用一组固定的 805 条指令，覆盖一整套简单的、助手风格的任务，见[这里](https://huggingface.co/datasets/tatsu-lab/alpaca_eval)。对每条指令，我们用两个 LLM 生成输出——*一个基线模型，一个待评估模型*。随后用一个 LLM 评估者给每个模型的输出打分（也就是成对设置），从而算出两个模型输出之间的胜率（win-rate）。

![（来自 \[10\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/14.webp)

**这为什么有用？** AlpacaEval 的目标，是打造一条又快又便宜、还与人类偏好高度相关的自动化评估流水线。当前这一版 AlpacaEval 跑完不到三分钟，花费不到 10 美元，与人工评估（取自 Chatbot Arena）的 Spearman 相关系数高达 0.98，见上图。相比之下，做人工评估既有噪声又有分歧，开销大得多，还可能要好几周的标注时间。正因为 AlpacaEval 如此高效，这个指标特别适合模型研发——*它为简单指令遵循任务的人工评估提供了一个可靠代理，算起来又快又便宜*。

**这个评估者是怎么工作的？** AlpacaEval 评估者所用的 prompt 见下图。这些 prompt 采用[对话模板结构](https://huggingface.co/docs/transformers/main/en/chat_templating)，与大多数[商用对话补全 API](https://platform.openai.com/docs/api-reference/chat) 的输入风格一致，用于在多轮对话里区分角色和消息。对每条指令，会像下图那样把一对回答传给评估者，我们拿到的回应就是更优的那份输出——*要么是二选一的结果，要么是 LLM 给每个选项打出的* [*logprobs*](https://cookbook.openai.com/examples/using_logprobs)。这个回应代表的是：在数据集中某条指令下，待评估模型的回答优于基线模型回答的概率。把这些概率在整个数据集上取平均，就能算出胜率，衡量一个模型的输出被偏爱于基线模型输出的比例。

![AlpacaEval 中的评估者 prompt（来源）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/15.webp)

评估者的质量靠与一组 2.5K 条人工评估的一致程度来检验，这组数据可见[这里](https://huggingface.co/datasets/tatsu-lab/alpaca_eval/blob/main/alpaca_farm_human_crossannotations.json)。不过，为某个具体场景挑选最佳评估者时，还有其他几项因素同样重要（比如成本和延迟）。最初，AlpacaEval 对数据集中每个样本以温度为零生成单个偏好回应。但在后续版本里，自动偏好标注的质量通过以下几点得到了提升：

-   随机打乱模型输出在 prompt 里的位置（或者对模型输出的每种可能位置都采样多个偏好分数）。
-   测量每份回答的 logprobs，而不是生成一个二选一的偏好回应。
-   换用更强的模型（GPT-4-Turbo）当评估者。

作者还修订并精简了核心评估者的 prompt，缩短了指令，并让回应里只输出单个 token，见下图。

![AlpacaEval-2.0 中的评估者 prompt（来源）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/16.webp)

**缓解长度偏差。** 正如我们所见，用 LLM 当评估者会给评估过程引入好几种微妙的偏差来源。我们必须意识到这些偏差，并尽力把它们消除或纳入考量。

> “如果所有模型的输出都和基线模型的输出一样长，AlpacaEval 这个指标又会是多少？” *—— 来自 \[10\]*

LLM 评估者一个已知且普遍的偏差，就是偏向更长的输出（也就是冗长偏差）——*某些商用 LLM（比如 GPT-4 或 GPT-4-Turbo）倾向于偏爱更长而非更短的回答*。结果就是，在内容质量固定、相当、甚至更差的情况下，AlpacaEval 反而可能给更长的输出打更高分。

![（来自 \[10\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/17.webp)

为对抗这种偏差，研究者用一个简单的、基于回归的去偏过程扩展了 AlpacaEval 指标 \[10\]。具体来说，是训练一个线性回归模型，它以三个属性（见上图）为输入：*i)* 模型，*ii)* 指令难度，*iii)* 归一化后的输出长度。模型训好之后，我们就能把那些被认为与输出质量存在虚假相关的项“清零”，只留下真正的质量分数。在 \[10\] 这个例子里，我们只需从回归里移除长度项，照常算胜率，就得到一个长度受控的 AlpacaEval 分数。

![（来自 \[10\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/18.webp)

长度受控的 AlpacaEval——*如今已用于公开排行榜*——在 \[10\] 中被发现比标准指标更难被钻空子。换句话说，*我们没法只靠让模型变得更啰嗦或更简短，就大幅改变长度受控 AlpacaEval 的结果。* 如上表所示，AlpacaEval 的胜率会随模型的冗长程度发生剧烈变化。而长度受控版本被发现与人类评分的相关性更好，把 AlpacaEval 与 Chatbot arena 的 Spearman 相关系数从 0.94 提升到了 0.98。

### LLM 驱动评估的其他早期用法

Vicuna 一经提出，用 GPT-4 当评估者就越来越普遍。彼时几乎还没人做过分析来证明 LLM 驱动评估的可靠性。但有几个突出的因素让这种评估方式如此受欢迎：

-   实现简单——*只要一个 prompt 加一次 API 调用*！
-   LLM 输出的开放性，让用传统/自动指标（比如 ROUGE 或 BLEU）来评估变得非常困难。
-   人工评估——*我们评估 LLM 时的真值来源*——既有噪声、又贵、又费时。

研究社区需要一种更易上手的评估策略，它得能 *i)* 在大量任务上可靠地衡量性能，*ii)* 让我们更快地实验和迭代。接下来这几篇论文会展示，基于 LLM 的评估很快填补了这块空白，为研究者提供了一个自动化、无参考、能支撑快速模型迭代的指标。

![（来自 \[3\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/19.webp)

**LIMA：少即是多的对齐之道。** LIMA \[3\] 研究的是：在数据量有限的情况下，我们能否用[监督微调](https://cameronrwolfe.substack.com/p/understanding-and-using-supervised)来对齐预训练语言模型（比如 [LLaMA](https://cameronrwolfe.substack.com/p/llama-llms-for-everyone)）。有意思的是，作者发现仅仅 1,000 条精挑细选的微调样本，就足以取得相当强的表现。这些结果说明，LLM 的知识大多是在预训练阶段习得的，而微调优化的是模型输出的格式。这一现象被称作“浅层对齐假说（Superficial Alignment Hypothesis）”，见下图。

> “我们定义了浅层对齐假说：模型的知识和能力几乎全部在预训练阶段习得，而对齐教会它该用哪一类格式的子分布。” *—— 来自 \[3\]*

\[3\] 同时用了人工评估和 LLM 驱动评估。对每个被测模型、每个 prompt，各生成一份回答。然后让人类匿名地把 LIMA 的输出和所有基线模型对比，挑出更偏爱的回答，见下图。这套评估过程可以这样自动化：

1.  把一模一样的 prompt 给 GPT-4。
2.  让模型挑出更偏爱的回答。

无论是人工还是基于模型的评估，LIMA 都被发现胜过 [Alpaca](https://cameronrwolfe.substack.com/i/114077195/alpaca-an-instruction-following-llama-model) 这类更早的开放模型，尽管它的微调数据要少得多。LIMA 还胜过 GPT-3.5，并在相当一部分测试 prompt 上追平或超过 GPT-4 的表现——*具体是 34% 到 43% 的 prompt*。

![（来自 \[3\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/20.webp)

虽然大多数实验用的是成对比较，作者也尝试用 GPT-4（某些消融实验里用 GPT-3.5）以逐点方式、在六分制 Likert 量表上给模型回答的有用性打分，见下图。

![（来自 \[3\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/21.webp)

**Guanaco。** \[4\] 的作者提出了量化低秩自适应（Q-LoRA），这是一种参数高效的训练策略，让在普通硬件（即显存较小的消费级 GPU）上微调 LLM 容易了许多。关于 Q-LoRA 的更多细节，可见[这篇综述](https://cameronrwolfe.substack.com/i/138861994/lora-variants-there-are-a-ton)。用 Q-LoRA 训练 LLM 的主要好处是显存占用大幅降低。我们甚至在 \[4\] 里看到，一个 650 亿参数的 LLM 用 Q-LoRA、靠单张 48Gb 的 GPU 就能微调！

![（来自 \[4\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/22.webp)

\[4\] 的作者用 Q-LoRA 训练了 Guanaco 这套聊天机器人风格的开放 LLM。这些模型同时用人类和 GPT-4 来评估。有意思的是，GPT-4 被发现能给出有意义、可靠的性能指标，而老一代基准却给不出对聊天机器人表现的准确衡量。

> “GPT-4 评估是人工评估的一种廉价而合理的替代品……我们发现，当前的聊天机器人基准并不可信，没法准确评估聊天机器人的性能水平。” *—— 来自 \[4\]*

\[4\] 里用了两种基于 LLM 的评估方式。第一种设置中，把 ChatGPT 和另一个模型的回答一并喂给 GPT-4，让它：

-   给两份回答各打一个 `[1, 10]` 区间内的分数。
-   为这些分数给出解释。

值得注意的是，这套设置和 Vicuna \[2\] 用的自动评估策略一模一样。在此基础上，一个模型的表现是相对 ChatGPT 来报告的。更具体地说，我们衡量的是每个模型拿到的总分之和与 ChatGPT 总分的比值，见下图。Guanaco 系列模型相对 ChatGPT 取得了亮眼的表现。

![（来自 \[4\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/23.webp)

第二种设置中，GPT-4 直接在模型输出之间做对比。这些对比以三分类标注问题的形式呈现给 GPT-4，见下图。GPT-4 被提示要么挑出更好的回答、要么判定两者平局，同时为它的选择给出详细解释。用这种方法，\[4\] 的作者在 Guanaco、ChatGPT 和其他相关基线模型之间做了一对一对决。有意思的是，我们在 \[4\] 里看到，GPT-4 对 prompt 中第一份回答表现出明显的位置偏差，而用位置互换技巧就能把它消除。

![成对比较的示例 prompt（含平局）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/24.webp)

**模仿商用 LLM 的虚假承诺。** 在 [LLaMA](https://cameronrwolfe.substack.com/p/llama-llms-for-everyone) 以及随之诞生的[一大批微调 LLM](https://cameronrwolfe.substack.com/p/beyond-llama-the-power-of-open-llms) 之后，开放 LLM 的势头一时无两。当时，许多 LLaMA 的微调版本都用一种“模仿”策略训练——*我们用一个更强的模型（比如 ChatGPT）对一大批多样化 prompt 生成回答，再直接拿这些数据微调开放模型*。这些模型看起来表现极好，似乎预示着开放与闭源 LLM 之间的性能差距可能[很快消失](https://www.semianalysis.com/p/google-we-have-no-moat-and-neither)。然而在 \[5\] 中，更有针对性的评估给这些开放模仿模型的表现画了一幅更清晰、也更清醒的图景。

> “一开始，我们对自己模仿模型的输出质量感到惊讶……可一旦做更有针对性的自动评估，就会发现这些模仿模型几乎丝毫没有缩小从基座模型到 ChatGPT 的差距。” *—— 来自 \[5\]*

简单说，模仿模型很擅长照搬 ChatGPT 的风格，这很容易骗过人类标注者，让他们以为模型输出质量很高。但这些模型缺乏 ChatGPT 这类更强 LLM 的事实准确性，而这点要靠更深入的评估才能暴露出来，见下图。

![（来自 \[5\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/25.webp)

鉴于 \[5\] 如此强调严谨评估，我们或许会觉得有意思的是：作者在部分分析里同样借助了 GPT-4 当评判者！首先，他们用以下两种变量微调了一批 LLaMA 模型：

1.  不同量的模仿数据。
2.  不同规模的基座模型。

随后，通过同时向人类和 GPT-4 征求反馈来评估这些模型。在 LLM-as-a-Judge 评估上，作者采用了 Vicuna \[2\] 提出的同一套策略——用一个成对 prompt 让 GPT-4 评判模型输出的质量。GPT-4 拿到 ChatGPT 和一个模仿模型各自的回答，再被要求在这两份输出之间输出一个偏好排序。用于 GPT-4 评估的 prompt，和给人类评估的 prompt 完全一致。如下图所示，人工评估和 LLM 评估观察到了相似的性能趋势：增大基座模型的规模，比收集更多模仿数据更有益。

![（来自 \[5\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/26.webp)

这个结果进一步佐证了用 GPT-4 做成对评估是有效的。GPT-4 能可靠地预测不同 LLM（比如 ChatGPT 和 Vicuna）输出之间的胜率，而且在整体意义上，这个胜率与人工标注得到的评分相关性良好。

**Tülu。** \[6\] 的作者对开放的指令遵循数据集做了一次大规模分析。他们在每个数据集上都微调了一批基于 LLaMA、参数量从 70 亿到 650 亿不等的 LLM。为了应对开放 LLM 评估不足或具误导性这一公认问题，这些微调实验还配上了一套庞大的评估套件，从而清晰地呈现出开放模型是否在真正逼近商用模型的表现。最佳模型名为 Tülu，是在好几个不同指令遵循数据集的混合上训练出来的。

![（来自 \[6\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/27.webp)

为评估开放式指令遵循能力，作者采用了 AlpacaEval \[8\] 基准。具体来说，是用 GPT-4——*以成对提示策略*——计算每个被测模型对阵 GPT-3.5-Turbo（[davinci-003](https://platform.openai.com/docs/deprecations)）的胜率。为避免位置偏差，作者采用了位置互换技巧。如上表所示，Tülu 系列模型与基线模型表现相当，但落后于顶尖商用模型。

## LLM-as-a-Judge 的走红

在 LLM-as-a-Judge 评估被早期开放 LLM 工作大量采用之后，这套策略在整个 LLM 研究社区里也越来越常见。本节会综述其他几篇重要论文——*核心那篇提出并剖析 LLM-as-a-Judge 技术的论文 \[17\] 除外*——它们就 LLM-as-a-Judge 评估给出了有用的分析和洞见。

### [Can Large Language Models Be an Alternative to Human Evaluations?](https://arxiv.org/abs/2305.01937) \[11\]

人工评估是可靠评估文本质量的标准，但这并不意味着人就能完美地评估文本！任何尝试过收集人工标注数据的从业者都明白，人工评估——*尽管极其宝贵*——是一个有噪声、耗时又烧钱的过程。

> “这篇论文是**第一篇**提出用 LLM 替代人工评估、并展示其有效性的工作。” —— 来自 \[11\]

由此我们或许会好奇：LLM 能不能用来替代人工评估？为弄清这个问题，\[11\] 的作者做了一项并行研究，同时用人类和 LLM 来评判写作质量。虽然我们目前看过的论文里也简单探索（并用）过这类技术，但 \[11\] 是第一个把它们和人工评估严谨对照分析的工作。有意思的是，*我们在 \[11\] 里看到，只要给 LLM 和人一样的指令和示例，LLM 评判文本质量的结果就能与人保持一致*。

![（来自 \[11\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/28.webp)

**人工评估对比 LLM 评估。** \[11\] 考察的任务是开放式故事生成，即给一段简短 prompt 配上一个基于该 prompt 的故事。人写的和 LLM 写的故事都要被评估。如上图所示，对某个给定的故事，我们可以同时让一位人类标注者和一个 LLM——*用与给人类相同的指令和样例*——为数据集里的每个故事各打一个分。然后，对比这些评估结果，就能判断人工评估与 LLM 评估之间存在多高的相关性。

![人工评估设置（来自 \[11\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/29.webp)

评估一个样本时，LLM 拿到的输入和人类评估者完全一样。模型被要求在一到五分的 Likert 量表上给每个故事打分（也就是逐点打分），分数则通过让模型自由生成文本得到——*我们只需从模型回答里把分数解析出来*。不过，给故事打分时会考量好几个不同的质量维度：

1.  *语法（Grammar）*：故事文本在语法上有多正确？
2.  *连贯性（Cohesiveness）*：故事各句之间是否衔接得当？
3.  *可读性（Likability）*：这个故事读起来是否享受？
4.  *相关性（Relevance）*：故事是否切合 prompt？

用人类评估者考量这些特性的设置见上图。用 LLM 执行同样评估时对应的 prompt 见下图，每个特性各有一个专属 prompt。

![LLM 评估 prompt（来自 \[11\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/30.webp)

**LLM 评估管用吗？** 鉴于故事生成是一项专门任务，\[11\] 的作者聘请了专家级人类评估者（也就是英语老师）来评判故事质量。这些标注者的任务，是评估为 200 个 prompt 写的故事的质量，对每个 prompt 我们 *i)* 从 [GPT-2](https://cameronrwolfe.substack.com/p/language-models-gpt-and-gpt-2)（一个较弱的 LLM）采样一份回答，*ii)* 再让一个人为该 prompt 写一份回答。我们的目标是判断：人类和 LLM 评估者能否察觉这些模型生成故事与人写故事之间的质量差异。

评估用了好几个不同的 LLM，包括 [T0](https://arxiv.org/abs/2110.08207) \[12\] 和来自 OpenAI 的几个 GPT 变体。对比人工评估和 LLM 评估的结果，可以得出几个有意思的观察：

-   较弱的 LLM 评估者（比如 T0 和早期 GPT 变体）很难察觉 GPT-2 与人写故事之间的质量差异。
-   专家级人类评估者和更强的 LLM 都明显偏爱人写的故事。
-   更近期的模型（比如 ChatGPT）既能准确评判故事质量，又能为打分给出有见地的解释。
-   最难评判的维度是可读性，这说明主观维度对人和 LLM 来说都更难评估。

上述结果表明，足够强的 LLM 能评估基础的写作质量维度，但这些结果是在整体层面评估的——*我们测的是整个数据集上人写故事与模型生成故事平均分的差异*。一旦换到“人和 LLM 对单个故事的评估是否相似”这个问题上，结果就没那么清晰了。不过，我们确实看到人类与 LLM 给出的质量分数之间存在弱正相关，见下图。

![（来自 \[11\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/31.webp)

**人的角色。** 看了 \[11\] 的结果，有人或许会开始琢磨：LLM 能不能彻底取代人工评估？LLM 评估有许多好处，比如可复现、易用、便宜、高效。再者，人工数据标注的某些环节在[伦理上是有争议的](https://www.washingtonpost.com/world/2023/08/28/scale-ai-remotasks-philippines-artificial-intelligence/)，LLM 驱动的评估（或许）能帮我们避开其中一些弊端。但人依然极其必要，得持续监控 LLM 评估的质量，发现其中的偏差或漂移。*由于这套方法会引入种种偏差来源，我们永远没法对 LLM 评估完全放心。*

> “我们建议把 LLM 评估当作一种廉价而快速的质量判断手段，而人工评估最好用在系统部署到真实应用之前、从人那里收集反馈的环节。” *—— 来自 \[11\]*

人工评估和 LLM 评估都有局限。因此，*把两者搭配着用才是最优解*。把它们合在一起，能让人类专家在自己的岗位上更高效、更准确，从而更可靠地扩展整个评估流程。我们用 LLM 评估在模型研发期间快速迭代、检验新想法；人类专家则监督评估过程、解读结果、提出改进，并随时间推移不断提升可靠性。

### [G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment](https://arxiv.org/abs/2303.16634) \[13\]

![（来自 \[14\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/32.webp)

\[13\] 的作者提出了最早一批被证明与人工评估结果高度相关的 LLM 驱动评估技术之一。这项工作建立在 \[13\] 和 \[14\] 此前的研究之上，那些研究是用 LLM 给文本赋予的概率来给文本打分，见上图。这个思路本身很巧妙，但它假设 LLM 评估者会给高质量文本赋予高概率——而这并不总成立。正因如此，这类技术往往不太可靠，产出的分数与人工评估相关性很差，也就促成了 \[13\] 对类似技术做更系统的考察。

> “我们提出 G-EVAL，一个用带思维链（CoT）和填表范式的 LLM 来评估 NLG 输出质量的框架。” *—— 来自 \[13\]*

相比此前的工作，\[13\] 提出的技术——*名为 G-Eval*——做了两处重大改动：

1.  把 LLM 评估与一种新式思维链（CoT）提示结合起来，称为 Auto-CoT。
2.  采用填表范式（即提示 LLM 输出一个质量分数），而不是去测量某段文本序列的概率。

这两处改动被发现影响极大，让 G-Eval 大幅领先各类基线评估技术，并达到了与人工评估分数可接受的相关水平。

**G-Eval** 用一个两步生成流程来评估一段文本，见下图。首先，给 LLM 一段任务描述和一组评估该任务的标准。再用一个 CoT prompt，让 LLM 生成一串用于评估的步骤。当 LLM 被要求为某个任务样本（比如一对文章—摘要）输出分数时，这串评估步骤就作为模型的额外输入。

![（来自 \[13\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/33.webp)

上图给出了一个示例，展示了应用 G-Eval 评估摘要连贯性所用的 prompt。这种为 LLM 评估生成一段步骤描述的过程，被称作 Auto-CoT。它与标准 CoT 提示的区别在于：它让模型为一项任务输出一串通用的评估步骤，而不只是为每个产出的分数提供一段更详细的解释。

![加权打分策略（来自 \[13\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/34.webp)

**加权打分。** 我们在 \[13\] 里看到，与其让 LLM 直接输出一个质量分数，不如通过加权平均来产出更可靠的分数，见上图。这里，我们只需测出每个分数对应的概率（比如用 OpenAI API 里的 logprobs），再用这些概率把最终分数算成一个加权平均。如果是在一到五分的 Likert 量表上评估一段文本，我们只需 *i)* 求出每个分数的概率，*ii)* 把每个分数乘以它的概率，*iii)* 对所有加权后的分数求和。

![（来自 \[13\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/35.webp)

**实证分析。** G-Eval 在 \[13\] 中于文本摘要和对话生成两类任务上做了测试，被发现胜过各类[基于参考和无参考](https://cameronrwolfe.substack.com/i/144374854/how-can-we-evaluate-a-summary)的基线指标，见上图。G-Eval 所用的 Auto-CoT 策略被发现能提升 LLM 评估的质量，因为它在打分过程中为底层模型提供了更多上下文和指引。但 G-Eval 也有几处值得注意的局限：

1.  对所用的具体 prompt 和指令很敏感。
2.  对 LLM 生成的文本存在可测量的偏好（即自我增强偏差）。
3.  LLM 通常输出整数分数，且倾向偏向打分区间里某个固定数字（比如在一到四分的 Likert 量表里，三分可能是最常见的输出）。

尽管如此，G-Eval 仍是最早证明 LLM 能以可靠、有用的方式评估文本的论文之一。当用 GPT-4 当评估者时，*G-Eval 在摘要任务上与人类评分达到了 0.514 的* [*Spearman 相关系数*](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.spearmanr.html)，相比此前同主题的工作是一次显著飞跃！

### [Large Language Models are Not Fair Evaluators](https://arxiv.org/abs/2305.17926) \[16\]

![示例在 LLM 评估 prompt 中的摆放位置](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/36.webp)

假设我们用一个 LLM 评估者去比较针对某个 prompt 生成的两份模型输出的质量。要做这件事，我们通常会把两份输出连同原始指令一起传给 LLM，让模型指出更好的那份（也就是成对打分）。但我们必须给每份输出在 prompt 里选一个位置，见上图。这看似是个随意的选择，可我们在 \[16\] 里看到，这个摆放会剧烈影响评估结果——*大多数 LLM 评估者都有很强的位置偏差*。

![（来自 \[16\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/37.webp)

**位置偏差有多严重？** 为量化位置偏差对 LLM 评估的影响，\[16\] 的作者用 ChatGPT 和 GPT-4 做了一项深入研究。有意思的是，两个模型都表现出明显的位置偏差。不过，GPT-4 倾向偏爱 prompt 里的第一份回答，而 ChatGPT 偏爱第二份。研究者用这两个模型以成对方式评估了 ChatGPT、Vicuna-13B 和 Alpaca-13B 的输出质量，见下图。

![（来自 \[16\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/38.webp)

在上表里能看到，评估结果会随输出在 prompt 中的位置发生显著变化。事实上，把 Vicuna-13B 的输出从第一位换到第二位时，它对阵 ChatGPT 的胜率从 2.5% 飙到了 82.5%。换句话说，*LLM 评估的结果完全取决于一个模型的输出在 prompt 里的位置*！值得注意的是，GPT-4 的位置偏差相比 ChatGPT 没那么夸张，而且当模型输出之间质量差异明显时，它的偏差往往更轻。

![（来自 \[16\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/39.webp)

**对此我们能做什么？** 我们从 \[16\] 学到两种消除位置偏差的实用校准手段：

1.  *多证据校准（Multiple-Evidence Calibration）*：在输出最终分数之前，让模型先生成证据（即类似 CoT 提示的解释或理由），能提升评估质量。
2.  *平衡位置校准（Balanced Position Calibration）*：对同一个样本多次生成证据和分数，每次都互换（或随机选取）模型回答的位置。

平衡位置校准的做法（见上图）和 \[17\] 提出的位置互换技巧非常相似。把这两种策略搭配起来用，能显著降低 LLM 评估者的位置偏差，见下图。不过，对于多证据校准这类依赖生成多个输出的技术，选一个合适的温度极其重要。

![（来自 \[16\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/40.webp)

### 专用评判者与合成数据

虽然我们到目前已看了一大批论文，但 LLM-as-a-judge 实在是一项极受欢迎的技术，催生出了一大片不同的研究方向。其中两个最有意思的相关研究领域是：

-   微调定制的 LLM 来做评估。
-   用 LLM-as-a-Judge 来生成合成数据。

下面我们简要综述这两个话题及相关论文。

![（来自 \[19\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/41.webp)

**训练专用 LLM 评判者。** 本文看过的论文大多用商用模型当评判者。但我们也可以微调出自己的 LLM 评判者！这种做法最有名的例子是 Prometheus \[19, 20\]（见上图），不过这一话题上已有大量论文发表 \[21, 22, 23\]。训练定制评估者的主要障碍曾经是基座模型的质量。但 [LLaMA-3](https://llama.meta.com/llama3/) 的发布似乎在很大程度上扫除了这一障碍，让用开放 LLM 当评估者变得更可行了。

![（来自 \[24\]）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6b36a69e5ede876fcf0b9cfca95ef345ad4c43ba/ai-insights/2026-06/02/images/llm-as-a-judge-using-llms-for-evaluation/42.webp)

**基于 AI 反馈的强化学习（RLAIF）。** 正如我们所见，LLM 评判者能准确预测人类偏好。[基于人类反馈的强化学习（RLHF）](https://cameronrwolfe.substack.com/p/the-story-of-rlhf-origins-motivations)——*在人类偏好数据上训练 LLM 最常用的算法*——依赖大规模的人类偏好对数据集。这么看来，用 LLM-as-a-Judge 式的 prompt 来收集合成偏好数据能带来好处，也就不足为奇了，见上图。这条路已被好几篇论文探索过 \[24, 25\]，并[据传](https://www.interconnects.ai/p/llm-synthetic-data)在顶尖工业实验室里被大量使用；更详尽的梳理可见[这里](https://cameronrwolfe.substack.com/p/rlaif-reinforcement-learning-from)。

## 实用要点

就算这篇综述别的都没记住，关于 LLM-as-a-Judge 评估，下面这几条也该记牢：

-   这套方法通用、无参考，几乎适用于任何任务。
-   实现 LLM-as-a-Judge 很简单——*只要一个 prompt*。
-   LLM-as-a-Judge 评估又便宜又快，特别适合在模型研发期间提升迭代速度。
-   与人类偏好的相关性总体不错。
-   存在好几种偏差来源，让这个指标算不上完美，所以一定要把 LLM-as-a-Judge 和人工评估搭配着用。

除了这些基本要点，我们在本文里还看了海量论文，它们给出了一大堆正确运用 LLM-as-a-Judge 的实用技巧。下面把这些论文的要点梳理出来。

**LLM-as-a-Judge 的设置。** 给 LLM-as-a-Judge 组织 prompt 和评估流程的方式有好几种。两大核心策略是成对评估和逐点评估，分别让评判者给一对模型输出和单个模型输出打分。不过，这些设置还有各种变体。比如，Vicuna \[2\] 用的成对打分法是让评判者给每个模型输出各打一个分；而 \[17\] 的作者则提出在逐点打分 prompt 里附上一份解法、供评判者当参考（即参考引导评估）。[Vicuna](https://github.com/lm-sys/vicuna-blog-eval)、[AlpacaEval](https://github.com/tatsu-lab/alpaca_eval) 以及 [LLM-as-a-Judge 那篇论文](https://arxiv.org/abs/2306.05685)本身，都给出了好几种设置下 LLM-as-a-Judge 的具体实现示例。

**再谈逐点打分。** 逐点打分的一个缺点是，评判者可能缺乏一套稳定的内部打分尺度。由于分数存在于连续空间里，逐点分数往往波动很大，可靠性不如成对比较。但 LLM-as-a-Judge 的实现方式通常由我们的应用决定——*我们没法总是用成对设置*。要提升逐点打分的可靠性，我们可以 *i)* 在评判者 prompt 里加一份[评分细则](https://x.com/seungonekim/status/1749289437165769177)（即对量表里每个分数给出解释），*ii)* 提供少样本示例来校准评判者的打分尺度，或 *iii)* 测量每个可能分数的 logprobs，算出一个加权输出。

**更好的可解释性。** 把 LLM-as-a-Judge 和 CoT 提示结合起来——*尤其是因实现简便而格外好用的零样本 CoT 提示*——威力惊人。CoT 提示已被证明能提升 LLM 的推理能力，它同样能提升 LLM-as-a-Judge 评估的准确度。我们应当让模型先输出理由、再生成分数，以确保评判者最终的分数能被解释所支撑。这些理由在可解释性上也大有裨益，因为我们可以手动读它们，更深入地了解模型的表现。

**选对温度。** 为了让 LLM-as-a-Judge 的结果（相对而言）确定，我们应当用较低的[温度](https://x.com/cwolferesearch/status/1671628210180698112)设置（比如 0.1）。但我们要意识到温度设置对打分的影响——*我们在 \[12\] 里看到，更低的温度会把评判者的输出往更低的分数上拉*！所以，任何要直接拿来对比的 LLM-as-a-Judge 结果，都必须确保用同一温度得到。此外，当我们要为每个样本采样多个分数时——比如做[自洽性](https://arxiv.org/abs/2203.11171)或多证据校准 \[16\] 时——应当用稍高一点的温度。

### 第一次读这份 newsletter？

你好！我是 [Cameron R. Wolfe](https://cameronrwolfe.me/)，深度学习博士，[Netflix](https://research.netflix.com/research-area/nlp-and-conversations) 的资深研究科学家。本文转载自我的 [Deep (Learning) Focus](https://cameronrwolfe.substack.com/) newsletter，我在那里帮读者更好地理解 AI 研究中的重要话题。如果你喜欢这份 newsletter，欢迎订阅、考虑付费订阅、分享出去，或者在 [X](https://twitter.com/cwolferesearch)、[LinkedIn](https://www.linkedin.com/in/cameron-r-wolfe-ph-d-04744a238/) 和 [Medium](https://wolfecameron.medium.com/) 上关注我！

### 参考文献

\[1\] Bubeck, Sébastien, et al. “Sparks of artificial general intelligence: Early experiments with gpt-4.” *arXiv preprint arXiv:2303.12712* (2023).

\[2\] Vicuna Team, et al. “Vicuna: An Open-Source Chatbot Impressing GPT-4 with 90%\* ChatGPT Quality.” [https://lmsys.org/blog/2023-03-30-vicuna/](https://lmsys.org/blog/2023-03-30-vicuna/) (2023).

\[3\] Zhou, Chunting, et al. “Lima: Less is more for alignment.” *Advances in Neural Information Processing Systems* 36 (2024).

\[4\] Dettmers, Tim, et al. “Qlora: Efficient finetuning of quantized llms.” *Advances in Neural Information Processing Systems* 36 (2024).

\[5\] Gudibande, Arnav, et al. “The false promise of imitating proprietary llms.” *arXiv preprint arXiv:2305.15717* (2023).

\[6\] Wang, Yizhong, et al. “How far can camels go? exploring the state of instruction tuning on open resources.” *Advances in Neural Information Processing Systems* 36 (2023): 74764–74786.

\[7\] Gemma Team, et al. “Gemma 2: Improving Open Language Models at a Practical Size.” [*https://storage.googleapis.com/deepmind-media/gemma/gemma-2-report.pdf*](https://storage.googleapis.com/deepmind-media/gemma/gemma-2-report.pdf) (2024).

\[8\] Li, Xuechen, et al. “Alpacaeval: An automatic evaluator of instruction-following models.” [https://github.com/tatsu-lab/alpaca\_eval,](https://github.com/tatsu-lab/alpaca_eval,) 2023.

\[9\] Dubois, Yann, et al. “Alpacafarm: A simulation framework for methods that learn from human feedback.” *Advances in Neural Information Processing Systems* 36 (2024).

\[10\] Dubois, Yann, et al. “Length-controlled alpacaeval: A simple way to debias automatic evaluators.” *arXiv preprint arXiv:2404.04475* (2024).

\[11\] Chiang, Cheng-Han, and Hung-yi Lee. “Can large language models be an alternative to human evaluations?.” *arXiv preprint arXiv:2305.01937* (2023).

\[12\] Sanh, Victor, et al. “Multitask prompted training enables zero-shot task generalization.” *arXiv preprint arXiv:2110.08207* (2021).

\[13\] Liu, Yang, et al. “G-eval: Nlg evaluation using gpt-4 with better human alignment.” *arXiv preprint arXiv:2303.16634* (2023).

\[14\] Wang, Jiaan, et al. “Is chatgpt a good nlg evaluator? a preliminary study.” *arXiv preprint arXiv:2303.04048* (2023).

\[15\] Fu, Jinlan, et al. “Gptscore: Evaluate as you desire.” *arXiv preprint arXiv:2302.04166* (2023).

\[16\] Wang, Peiyi, et al. “Large language models are not fair evaluators.” *arXiv preprint arXiv:2305.17926* (2023).

\[17\] Zheng, Lianmin, et al. “Judging llm-as-a-judge with mt-bench and chatbot arena.” *Advances in Neural Information Processing Systems* 36 (2024).

\[18\] Shi, Freda, et al. “Large language models can be easily distracted by irrelevant context.” *International Conference on Machine Learning*. PMLR, 2023.

\[19\] Kim, Seungone, et al. “Prometheus: Inducing fine-grained evaluation capability in language models.” *The Twelfth International Conference on Learning Representations*. 2023.

\[20\] Kim, Seungone, et al. “Prometheus 2: An open source language model specialized in evaluating other language models.” *arXiv preprint arXiv:2405.01535* (2024).

\[21\] Zhu, Lianghui, Xinggang Wang, and Xinlong Wang. “Judgelm: Fine-tuned large language models are scalable judges.” *arXiv preprint arXiv:2310.17631* (2023).

\[22\] Wang, Yidong, et al. “Pandalm: An automatic evaluation benchmark for llm instruction tuning optimization.” *arXiv preprint arXiv:2306.05087* (2023).

\[23\] Li, Junlong, et al. “Generative judge for evaluating alignment.” *arXiv preprint arXiv:2310.05470* (2023).

\[24\] Lee, Harrison, et al. “Rlaif: Scaling reinforcement learning from human feedback with ai feedback.” *arXiv preprint arXiv:2309.00267* (2023).

\[25\] Bai, Yuntao, et al. “Constitutional ai: Harmlessness from ai feedback.” *arXiv preprint arXiv:2212.08073* (2022).
