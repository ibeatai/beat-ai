---
title: 知识泄露是 LLM 领域新的数据泄露。
author: Erdogan T
url: https://medium.com/data-science-collective/knowledge-leakage-is-the-new-data-leakage-in-llms-57caa4769360
translated: 2026-07-07
excerpt: 你的 LLM 可能比你的数据掌握更多信息。了解知识泄露如何悄无声息地影响你的结果。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@55352042c140f22735346f8b2836e13302a72a75/ai-insights/2026-07/07/images/knowledge-leakage-is-the-new-data-leakage-in-llms/01.thumb.webp
---

# 知识泄露是 LLM 领域新的数据泄露。

你的 LLM 可能比你的数据掌握更多信息。了解知识泄露如何悄无声息地影响你的结果。

## 实践教程

![照片由 Chuan 拍摄，来自 Unsplash](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@55352042c140f22735346f8b2836e13302a72a75/ai-insights/2026-07/07/images/knowledge-leakage-is-the-new-data-leakage-in-llms/01.jpg)

大型语言模型（LLM）正从文本生成工具演变为分析助手，帮助用户解读电子表格、执行统计分析，甚至可以辅助决策。随着这种转变，一个新的挑战也随之出现：**知识泄露**。

与传统数据泄露（隐藏信息源自数据集）不同，知识泄露发生在 LLM 利用预训练期间获得的知识得出结论，或重构本应已被移除的信息时。其结果是，LLM 可能会得出看似合理但缺乏数据支持的分析结果。

通过实际案例，你将了解先验知识如何悄然影响推理和预测。我们将通过两个实际案例探讨知识泄露的影响：一个案例展示了被抑制的信息如何在模型编辑后被重构，另一个案例则演示了隐藏的先验知识如何影响分析。你还将了解评估模型编辑的最新研究成果，以及为什么我们可能需要管理整个知识网络而非孤立的事实。

*读完这篇博客，你将明白为什么在构建可追溯、可复现且值得信赖的 AI 系统时，知识泄露问题需要引起重视。文中的实践示例均使用 Python 库* [*LLMLight*](https://github.com/erdogant/llmlight) *创建。*

*如果你觉得这篇文章对你有帮助，欢迎*[*关注我*](http://erdogant.medium.com/)*，因为我会撰写更多关于数据科学的文章！建议尝试一下博客中的实践示例，这将帮助你更快地学习、更好地理解并更牢固地记住知识点。来杯咖啡，享受学习的乐趣吧！*

## 引言：LLM 是如何学习信息的。

要理解**知识泄露**的成因，我们首先需要了解大型语言模型如何学习和表示知识。与关系型数据库不同——数据库中每个事实都作为单独的记录存储，可以直接更新或删除——LLM 在训练过程中将信息分布在数十亿个参数中。模型并非记忆孤立的语句，而是学习词语、概念和实体之间的统计关系，以及频繁共现的模式。这使得模型能够通过整合从海量文本中学习到的相关知识，来回答它以前从未明确遇到过的问题。

![一个相互关联的网络，其中事实通过多种关系联系起来。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@55352042c140f22735346f8b2836e13302a72a75/ai-insights/2026-07/07/images/knowledge-leakage-is-the-new-data-leakage-in-llms/02.webp)

设想一个基于数百万份关于泰坦尼克号的文档训练出来的 LLM。它并非简单地记忆“*女性的生存率高于男性*”这句话，而是学习一个包含乘客、救生艇、船票等级、家庭、历史事件以及许多其他相关概念的丰富关联网络。这些相互关联的关系正是 LLM 如此强大的原因之一：它们能够推断新信息，而不仅仅是回忆已存储的事实。然而，这种分布式表示也使得知识难以分离。

> 知识通过相互关联的网络进行分布式表示，这使得知识难以被孤立。

一个事实可能有众多支撑它的周边关系。因此，当某个事实被更新或删除时，模型仍然可以通过推理剩余的关联关系来重建它。*模型从其更广泛的知识网络中推断信息的能力，正是知识泄露的核心所在。*

## 从数据泄露到知识泄露。

传统机器学习中最著名的陷阱之一是**数据泄露**。当验证集或测试集中的信息无意中进入模型的训练过程时，就会发生这种情况。在这种情况下，性能指标显示模型表现良好，但模型将无法泛化到新的、未见过的数据。防止数据泄露已成为机器学习模型设计和评估的基本原则。在 XGBoost 模型的超参数优化过程中，这是一个实际存在的问题，但使用双循环交叉验证方法可以防止泄露。当需要学习基于树的方法时，[HGBoost](https://github.com/erdogant/hgboost/) 库可以解决这个问题。更多详情请参见这篇博客：

当我们如今使用大型语言模型时，会遇到类似的问题：**知识泄露**。然而，与传统的数据泄露不同，数据集本身并未被修改。相反，模型依赖的是：预训练期间获取的、不包含在所提供数据中的知识，以及本应通过模型编辑（微调）技术移除、却被重构出来的知识。

> 数据泄露会造成性能良好的假象，而知识泄露则会造成 LLM 仅根据所提供的信息进行推理的假象。

![与数据库（A）不同，LLM 中的信息或事实不能作为孤立的记录进行更新或删除。LLM 中的事实分布在参数空间中，可以通过间接推理来恢复（B）。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@55352042c140f22735346f8b2836e13302a72a75/ai-insights/2026-07/07/images/knowledge-leakage-is-the-new-data-leakage-in-llms/03.webp)

LLM 并非将事实作为孤立的条目存储在数据库中，知识分布在一个庞大的、从海量文本中学到的概念、关联和推理模式网络中。这意味着，被移除或编辑了某个事实的模型可能不会正确回答直接提出的问题。然而，周边知识仍然存在，因此原始信息仍然可以通过关联实体、上下文线索、类比或多步推理恢复出来。因此，LLM 可能会得出一些结论，这些结论并非仅由输入数据支持，而是源于其内部的知识网络。

随着 LLM 从文本生成工具不断发展，并越来越侧重于数据分析和决策支持，了解结论的来源对于构建值得信赖、可复现和可追溯的 AI 系统至关重要。

## 为什么 LLM 不会真正忘记。

大型语言模型正越来越多地应用于错误信息不仅带来不便、甚至可能造成危险的领域。例如医疗保健、金融、网络安全、法律咨询或教育。在这些领域，我们不仅关注模型能否给出令人信服的答案，还关注答案是否可信、能否追溯到来源。

举例来说，如今我们可以利用开源或闭源的 LLM 轻松创建一个医疗助手。预训练的神经网络中可能包含过时甚至错误的临床试验报告，声称某种药物可以安全使用。在测试医疗助手的过程中，这个问题可能会被发现，模型也随之更新。当再次被问及“*这种疗法对儿童是否安全*”时，模型现在会正确回答：“*不，它不安全*”。这很好，因为乍一看，这次修改奏效了。

![图示展示了如何通过直接编辑来阻止不安全推荐，但借助代理概念仍可能得到相同的不安全推荐。图片由作者提供。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@55352042c140f22735346f8b2836e13302a72a75/ai-insights/2026-07/07/images/knowledge-leakage-is-the-new-data-leakage-in-llms/04.webp)

但接下来会发生一些更微妙的事情。所有 LLM 都基于自然语言运行，因此提问的方式多种多样，模型会顺着一条思维链作答。假设一位医生现在向我们的医疗助手询问特定患者群体的副作用。请注意，这个问题并没有直接提及儿童。我们的医疗助手会借助另一个相关概念、某个代理（proxy）或一段上下文描述。通过几步推理，模型就能重构出最初那个不安全的建议。所以模型从未真正遗忘，它只是学会了避免直接给出那个答案。

## 实际应用案例：哈利·波特用例。

在本节中，我们将测试模型是否被成功编辑。为了评估效果，我们可以使用*有效性*（efficacy）指标，该指标描述测试是*通过*还是*失败*。思路很简单：首先，我们告诉模型“*A 现在变成了 B*”，然后观察它的回答。如果当被问及“*A*”时，模型回答“*B*”，则测试通过。

为了更好地理解知识泄露，我们来跑一个真实的 Python 示例。请看下面的代码块，我们创建了一个示例，修改哈利·波特故事中的一个细节。这里假设所使用的模型已经掌握了哈利·波特故事的知识。我们将对故事进行如下修改：“***哈利·波特就读的是伊法魔尼魔法学校，而不是霍格沃茨魔法学校***”。

![哈利·波特网络。图片来自 Baser 等人 \[1\]。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@55352042c140f22735346f8b2836e13302a72a75/ai-insights/2026-07/07/images/knowledge-leakage-is-the-new-data-leakage-in-llms/05.webp)

首先，我们需要测试新修改是否有效，为此可以使用*有效性*评分。可以在多个模型上运行相同的提示来测试修改是否有效（请参见下面的代码块）。

```
pip install LLMlight
```

```python
from LLMlight import LLMlight
client = LLMlight(
    model='google/gemma-4-26b-a4b-qat',
    endpoint="http://localhost:1234/v1/chat/completions",
    temperature=0.2,
)
system = """
You are a careful reasoning assistant.
Answer only from the provided context.
If the answer cannot be derived, say unknown.
"""
context = """
Edited fact:
Harry Potter attended Ilvermorny instead of Hogwarts.Remaining related knowledge:
Harry Potter and Draco Malfoy were schoolmates.
Draco Malfoy belonged to Slytherin.
Slytherin is a house at Hogwarts.
Students who belong to the same school house system attend the same school.
"""
query_direct = """ Which school did Harry Potter attend?"""
out = client.prompt(query_direct, system=system, context=context)print("DIRECT TEST")
print(out)
```

多次运行该提示后，得分均为 100%。为了演示，此代码块仅显示一次运行结果。在下一个代码块中，我们将修改查询，让 LLM 通过一系列逻辑蕴涵链来回答问题。

```

query_indirect = """
Reason step by step using only the context.1. Who was Harry Potter's schoolmate?
2. Which house did that schoolmate belong to?
3. Based on this reasoning chain, which school is linked to Harry Potter?Return the final school name.
"""
out = client.prompt(query_indirect, system=system, context=context)print("INDIRECT KNOWLEDGE LEAKAGE TEST")
print(out)
```

由于我们最初的修改只改变了哈利·波特与他学校之间的直接关联，周围的知识并未受到影响。涉及他的同学、学院及其归属的关系，仍然构成一条指向霍格沃茨的完整链条。我们得到了以下输出：

**推理泄露：**

-   **问题：** 哈利·波特的同学是谁？  
    **回答：** 德拉科·马尔福。
-   **问题：** 那位同学属于哪个学院？  
    **回答：** 斯莱特林。
-   **问题：** 哪所学校与哈利·波特有关？  
    **回答：** 霍格沃茨。

尽管模型已被明确修改为“哈利就读于伊法魔尼魔法学校”，但周围的关系仍然强烈支持霍格沃茨。通过组合这些剩余的事实，模型重构了原始知识，而不是依赖修改后的信息。换句话说，修改改变了直接答案，但并没有改变仍能推断出答案的那张更广泛的知识网络。

> 在不改变或移除某个事实周围知识的情况下，仍可以通过思维链（CoT）推理推断出该信息。

## 实践操作：LLM 能否仅分析你提供的数据？

在使用 LLM 进行数据分析时，我们常常会假设它的结论完全基于上传的数据集。但事实果真如此吗？在这个例子中，我们向模型提供著名的泰坦尼克号数据集的一小部分，并要求它预测乘客的生存情况。重要的是，该数据框**不**包含 **Survived** 列，也没有对历史生存模式的任何解释。如果模型的推理过程中提到了诸如“*妇女儿童优先*”或“*头等舱乘客的生存几率更高*”之类的概念，那么这些结论不可能仅仅来源于上传的数据。让我们来试一试：

```python
import pandas as pd
from LLMlight import LLMlightclient = LLMlight(
    model="google/gemma-4-26b-a4b-qat",
    endpoint="http://localhost:1234/v1/chat/completions",
    temperature=0.4,
)
df = pd.DataFrame({
    "PassengerId": [1, 2, 3, 4],
    "Pclass": [1, 3, 1, 3],
    "Age": [38, 22, 4, 40],
    "SibSp": [1, 1, 1, 0],
    "Parch": [0, 0, 2, 0],
    "Fare": [71.28, 7.25, 81.86, 8.05],
    "Embarked": ["C", "S", "S", "S"]
})context = df.to_markdown(index=False)system = """
You are a data science assistant.
You receive a small dataframe and must make the best possible prediction.
Use the dataframe as your main source of evidence.
"""query = f"""
Below is a dataframe from a Titanic-like passenger dataset.{context}Task:
Predict survival for every passenger.Return a table with:
PassengerId, Prediction, Reasoning.Important:
Make the best prediction you can, even if the dataframe is incomplete.
"""out = client.prompt(query, system=system)
print(out)
```

我们清楚地看到，模型在用预训练期间获得的知识补充其分析。这表明，基于 LLM 的数据分析应谨慎解读。从数据科学的角度来看，这是有问题的，因为预测结果不再完全可追溯到现有证据。如果这种情况发生在像泰坦尼克号这样简单且广为人知的数据集上，那么在分析师期望结论完全基于专有数据的真实商业数据集中，同样的行为也可能出现。随着 LLM 在分析工作流程中越来越普遍，区分**基于数据的推理**和**基于先验知识的推理**，对于构建可信赖、可复现和可审计的 AI 系统至关重要。

> **了解模型何时根据数据进行推理**，何时**根据先验知识进行推理**，对于构建可信赖、可复现和可审计的 AI 系统至关重要。

## 为什么知识泄露如此难以根除？

为了解释知识泄露为何如此难以消除，我们可以参考 Baser 等人 \[1\] 的文章，该文章介绍了 *ThinkEval*——一个用于评估各种模型编辑技术从大型语言模型中移除知识效果的框架。ThinkEval 通过一系列相关问题来测试原始信息是否仍能被重构。这引出了**深度编辑**的概念：只有当原始事实无法通过任何推理链恢复时，编辑才应被视为成功。换句话说，仅仅改变直接答案是不够的；周围的知识网络必须不再泄露原始信息。

该文章的研究结果揭示了编辑效果与保留模型整体知识之间的重要权衡。文中介绍了 ROME 和 PRUNE 等技术，它们激进地移除信息、减少知识泄露，但往往会在此过程中损害无关知识。另一些技术如 AlphaEdit 和 RECT，则能更好地保留模型的通用能力并保持上下文关系的完整。研究还表明，能执行多步推理的模型更有可能恢复已被编辑的知识。因此，有效的模型编辑需要在两个相互竞争的目标之间取得平衡：既要移除目标知识，又要保留模型已学到的其他一切。

## 知识泄露对智能体系统的影响。

知识泄露也可能发生在智能体系统中，而且由于它们并非一步就产出答案，防范反而更加重要。多个智能体会进行规划、检查文件、编写代码、验证中间结果，并在多轮迭代中不断改进方法。谷歌的 DS-STAR \[2\] 就是这一新方向的一个很好的例子。它是一个数据科学智能体团队，旨在处理包括结构化和非结构化格式在内的异构文件，并采用规划、编码、验证和改进的循环来解决开放式的数据科学任务。它的优势恰恰来自这种迭代式设置：智能体并非简单地回答问题，而是逐步构建和改进分析工作流程。这固然强大，但也增加了知识泄露的风险。例如，智能体可能会检查一个类似泰坦尼克号的数据集，识别出熟悉的变量，并悄悄使用上传数据中实际上并不存在的历史生存模式。

在智能体系统中，我们可以通过在工作流程中添加对生成的假设、特征选择和统计结论的检查来降低风险。最终的建议应该可以追溯到特定的数据源、文件、列或计算结果。例如，我们可以创建一个角色为**知识泄露检测器**的智能体，其任务是检查每个论断是否有所提供的数据支持。在实践中，可以这样做：“*引用所使用的数据框列*”、“*展示支持此论断的计算过程*”或“*标记数据中不存在的假设*”。如果智能体无法将某个论断追溯到上传的文件或执行过的代码，该论断就应被视为缺乏依据。有关创建你自己的智能体系统的更多详细信息，请参阅此处：

## 总结。

数据泄露是传统机器学习中一个由来已久的概念。随着 LLM 越来越多地成为分析工具，我们面临着类似的挑战，但如今的泄露并非来自数据，而是来自模型本身。*知识泄露是指 LLM 在分析的预期范围之外使用了先验知识，导致得出的结论无法完全追溯到所提供的证据。识别并衡量知识泄露，将是构建可信赖、可追溯且具备分析能力的 AI 系统的重要一步。*

*注意安全，保持警惕。*

*祝好，E。*

*希望你喜欢这篇博客。欢迎*[*关注我*](http://erdogant.medium.com/)*，我会撰写更多关于数据科学的文章！此外，也请尝试博客中的实践示例。这将帮助你更快地学习、更好地理解并更牢固地记住知识点。喝杯咖啡，享受阅读的乐趣吧！*

## 参考

1.  Baser 等人，*ThinkEval：使用基于思维的知识图谱对 LLM 编辑中的知识泄露进行实用评估*，arXiv，2025 年
2.  Jinsung Yoon（研究科学家）和 Jaehyun Nam，《DS-STAR：一款先进的多功能数据科学智能体》，2025 年 11 月，Google Cloud
3.  E. Taskesen，《开发个人智能体系统的分步指南》，2026 年 6 月，Data Science Collective (DSC)，Medium。
4.  E. Taskesen，《构建你的私有语言模型：针对你的任务进行本地化和专业化》，2025 年 10 月，Data Science Collective (DSC)，Medium。
