---
title: "模型蒸馏指南：压缩 LLM 以提高边缘效率"
author: Kuriko Iwai
url: https://ai.gopubby.com/model-distillation-guide-compressing-llms-for-edge-efficiency-b2ed17a0960f
translated: 2026-06-30
excerpt: 掌握模型蒸馏的工作原理，并在 Llama 3 等 LLM 上动手实现。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/01.thumb.webp
---

# 模型蒸馏指南：压缩 LLM 以提高边缘效率

掌握模型蒸馏的工作原理，并在 Llama 3 等 LLM 上动手实现。

![照片由 Jakub Żerdzicki 在 Unsplash 上拍摄](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/01.jpg)

## 介绍

随着大型语言模型 (LLM) 的参数数量膨胀到数千亿，一个新的挑战出现了：效率。

对于每个小任务都运行像 GPT-4 这样的大型模型，成本太高、速度太慢，而且完全是杀鸡用牛刀。

模型蒸馏正是解决这一问题的工程方案，把庞大模型的智能打包进一个更小、更快、更经济高效的模型。

在本文中，我将探讨模型蒸馏的工作原理，以及它的常见应用和实际落地技巧。

### 目录

[**什么是模型蒸馏——理解师生框架**](#08bf)

[**工作原理——3 种核心蒸馏方案**](#3f20)

[**基于响应的蒸馏**](#b9b3)  
∘ [目标函数](#7603)  
∘ [步骤 1. 前向传播](#5de2)  
∘ [步骤 2. 软化 Logits](#3d15)  
∘ [步骤 4. 计算学生损失](#4847)  
∘ [步骤 5. 反向传播](#8657)  
∘ [常用应用](#b80a)

[**基于特征的蒸馏（中间层）**](#40a9)  
∘ [目标函数](#ccb5)  
∘ [特征知识的关键变体](#f953)  
∘ [主要用例](#fa5e)

[**基于关系的蒸馏**](#3c5c)  
∘ [目标函数](#578e)  
∘ [主要用例](#96d2)

[**蒸馏策略**](#832a)  
**·** [**学习来源**](#4269)  
∘ [黑盒](#b6de)  
∘ [白盒](#ac37)  
**·** [**结构关系**](#24b4)  
∘ [同一家族](#ec58)  
∘ [跨架构](#b870)  
**·** [**训练方法**](#108c)  
∘ [离线蒸馏](#365d)  
∘ [在线蒸馏](#ee10)  
∘ [自蒸馏](#dc70)  
**·** [**特定任务的蒸馏**](#96c6)

[**实施策略：该走哪条路？**](#da77)

[**模型蒸馏实战——将 GPT-4o 蒸馏为 Llama 3–1B**](#d46b)  
∘ [步骤 1. 提示 Gemini 3.1](#3ee7)  
∘ [步骤 2. 收集教师输出](#fc49)  
∘ [步骤 3. 微调 Llama 3–1B](#4bea)  
∘ [步骤 4. 执行推理](#676b)

[**总结**](#4b17)  
∘ [何时转型：蒸馏 vs. RAG vs. 微调](#5071)

[Kuriko IWAI](#303e)

## 什么是模型蒸馏——理解师生框架

**模型蒸馏**（或称**知识蒸馏**）是深度学习模型工程中的一种压缩技术，训练一个小型模型（**学生模型**）来重现大型预训练模型（**教师模型**）的行为和输出。

下图说明了其概念：

![图 A. 知识蒸馏架构图，展示高参数教师模型如何将软目标传递给轻量级学生模型。（作者：Kuriko IWAI）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/02.webp)

学生模型根据教师模型的输出（灰色区域，**图 A**）和其他内部因素（取决于蒸馏方案）进行训练，而不是从头开始使用原始训练集。

## 工作原理——3 种核心蒸馏方案

根据教师向学生传授知识的哪一部分，该技术已经演变为三种不同的方法：

- **基于响应：** 模仿 **最终答案**。
- **基于特征：** 模仿 **内部逻辑**。
- **基于关系：** 模仿 **数据结构**。

## 基于响应的蒸馏

**基于响应的蒸馏**是最常见的蒸馏形式，其中学生从教师的最后一个 Softmax 层生成的概率分布中学习。

下图展示了算法如何评估学生的预测结果：

![图 B. 基于响应的蒸馏流程图，展示了 logits 通过温度为 T 的 Softmax 层计算蒸馏损失的过程。（由 Kuriko IWAI 制作）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/03.webp)

### 目标函数

如 **图 B** 所示，基于响应的方法试图在反向传播过程中最小化总损失，即蒸馏损失和学生损失的加权平均值：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/04.webp)

其中：

- `L_{total}`：总损失。
- `α`：一个超参数，用于确定教师指导与真实（已标记）数据的相对重要性。
- `L_{distill}`：**蒸馏损失**（白色方框，**图 B**）。教师和学生软化后的输出之间的差异。
- `L_{student}`：**学生损失**（粉色方框，**图 B**）。学生预测与实际真实标签（硬目标）之间的标准交叉熵损失。

该过程分为五个不同的步骤：

### 步骤 1. 前向传播

教师模型和学生模型都执行前向传播，分别得到 logits（原始输出）`z_T` 和 `z_S`。

### 步骤 2. 软化 Logits

对 `z_T` 和 `z_S` 都应用温度参数 `T` 来平滑概率分布：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/05.webp)

其中：

- `P(i, T)`：随机类别 `i` 在温度 `T` 下的软化概率。
- `z_i`：类别 `i` 的 logit 值。
- `T = 1`：标准 Softmax。
- `T > 1`：蒸馏过程中使用的软化 Softmax。

这个过程使学生模型能够学习错误类别之间的关系，因为随着 `T` 的增加，概率分布变得更平坦，从而揭示出教师认为哪些类别与正确类别更相似。

**步骤 3. 计算蒸馏损失**

比较步骤 2 中的软分布，并计算蒸馏损失。

一种常用的方法是使用 [**Kullback-Leibler (KL) 散度**](https://kuriko-iwai.com/kullback-leibler-divergence-for-llms)：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/06.webp)

其中：

- `T`：温度参数。
- `P_T(i, T)`：教师模型生成的第 `i` 类的 Softmax 概率，根据温度 `T` 进行调整。计算方法定义在**公式 1.2** 中。
- `P_S(i, T)`：学生模型生成的第 `i` 类的 Softmax 概率，根据温度 `T` 进行调整。计算方法定义在**公式 1.2** 中。

### 步骤 4. 计算学生损失

将学生的原始输出（`T=1`）与实际标签进行比较。

### 步骤 5. 反向传播

最后，根据 **公式 1.1** 计算总损失 `L_{total}`，并*仅*更新学生的权重。

### 常用应用

基于响应的蒸馏方法在分类任务中表现良好。

- **边缘设备：**压缩图像分类器（例如，从庞大的 Vision Transformer 压缩到小型 MobileNet），以便它们可以在智能手机或物联网传感器上本地运行，而不会产生云延迟。
- **跨架构迁移：**在完全不同的架构之间迁移知识——例如，将 CNN（教师）蒸馏为 MLP-Mixer（学生）。
- **集成压缩：** 将 10 个不同模型（教师）的集成的平均输出蒸馏成一个速度快得多的单一学生模型。

## 基于特征的蒸馏（中间层）

基于特征的蒸馏方法不仅关注最终答案，还能使学生模型模仿教师的内部表征。

下图展示了算法如何评估学生的预测结果：

![图 C. 基于特征蒸馏的架构图，展示了模型间中间隐藏层和激活图的对齐方式。（由 Kuriko IWAI 绘制）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/07.webp)

### 目标函数

核心目标是最小化教师的中间特征图与学生的相应图层之间的差异（**图 C** 中的蒸馏损失）：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/08.webp)

其中：

- `Φ_T(x)`：教师模型第 `n` 层的激活图。
- `Φ_S(x)`：学生模型第 `n` 层的激活图。
- `G(...)`：一种对齐函数，类似于 1 x 1 卷积或线性投影，它重塑学生的特征以匹配教师的维度。
- `D(...)`**:** 距离度量。通常为均方误差 (MSE)，但也可以是 L\_1 范数或余弦相似度。

与基于响应的蒸馏类似，该技术试图在反向传播过程中最小化 **公式 2.1** 中的损失，以找到学生模型的最佳内部参数。

### 特征知识的关键变体

根据学生模型试图模仿的内容，特征蒸馏有三种不同的变体：

- **FitNets：** 尝试模仿教师的隐藏层，利用回归器。
- **注意力转移（AT）**：尝试模仿教师的注意力图。
- **因子迁移**：尝试模仿教师特征中的有意义因子，利用编码器-解码器释义器。

### 主要应用场景

特征蒸馏对于多重推理任务很有用，因为它能让学生了解教师的内在逻辑。

其主要应用场景包括：

- **使用计算机视觉进行目标检测任务：** 蒸馏特征图以保留空间信息和目标边界。
- **Transformer 压缩：**通过匹配注意力矩阵和隐藏状态，将 BERT 等模型蒸馏为 DistilBERT 或 TinyBERT，确保学生保留语言细微差别和上下文关系。
- **跨模态学习：**将经过图像训练的教师的特征蒸馏到经过深度图或红外数据训练的学生身上，帮助学生即使在输入类型有限的情况下也能学习到强大的特征。
- **小数据迁移学习：**允许小型学生模型学习教师（在大型数据集上预训练）丰富的特征层次结构，同时避免过拟合。

## 基于关系的蒸馏

基于关系的蒸馏将重点从模型看到什么转移到模型如何感知数据的结构。

下图展示了算法如何评估学生的预测结果：

![图 D. 基于关系的蒸馏可视化，重点关注数据流形和距离矩阵的结构相似性。（由 Kuriko IWAI 制作）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/09.webp)

该方法侧重于**数据流形的结构**，而不是模仿教师的特定层或输出。

例如，在图像分类任务中，学生本质上不是在学习“狗”的图像是什么样子；而是在学习“狗”比“汽车”更接近“猫”。

### 目标函数

关系蒸馏的目标是确保如果教师认为图像 A 和图像 B 相似，学生也应该在自己的特征空间中将它们映射得非常接近。

损失函数比较两个模型的相似性矩阵（Gram 矩阵）或距离矩阵：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/10.webp)

其中：

- `f_T^i`, `f_S^i`: 教师和学生的第 `i` 个输入的特征嵌入。
- `ℓ(...)`: 损失函数，用于惩罚教师相似度得分与学生相似度得分之间的差异。例如，均方误差 (MSE) 或 Huber 损失。
- `ψ(...)`**:** 相似度函数。例如，余弦相似度或欧氏距离。

关系蒸馏调整学生模型，以在反向传播过程中最小化**公式 3.1**中定义的损失。

### 主要应用场景

关系蒸馏能够帮助学生了解数据的潜在结构。

而且由于它只计算与教师输出的距离，因此该方法具有鲁棒性，并且与模型架构无关。

其主要应用场景包括：

- **图像检索**，例如图像分类任务或人脸验证。例如，学生学习如何将狗的图像聚类在一起，并将它们与猫的图像区分开来。
- **零样本/少样本学习：**学生可以通过学习已知类别之间的关系，更好地猜测新类别在特征空间中的位置。
- **知识图谱：**将实体之间复杂的关系蒸馏成更小、更快的图神经网络（如 GNN）。

## 蒸馏策略

除了蒸馏方案之外，还有几个因素决定蒸馏策略：

1. **学习来源**：教师可提供的学习来源。
2. **结构关系**：教师和学生在结构上的紧密程度。
3. **训练方法：**学生/教师模型的训练方式。
4. **特定任务的蒸馏**。

## 学习来源

教师提供的学习来源决定了学生可以模仿的内容。

分为两类：

- **黑盒：**学生只能从教师最终的文本输出中学习。
- **白盒**：学生可以完全访问教师的内部参数和概率。

### 黑盒

当教师是像 GPT 或 Gemini 这样的专有模型时，学生只能通过 API 访问最终输出结果。

这种方法简单直接，在标准 SFT 中很常见，侧重于复制一般的预测性能，但学生可能会错过教师推理的深度。

- **典型应用场景：** 通过 API 创建小型专用模型。对聊天机器人进行基本微调。

### 白盒

虽然需要在本地托管教师模型，但白盒方法允许学生访问教师模型的内部参数，以模拟其推理过程。

- **典型用例：**将 Llama-3 70B 蒸馏为本地 8B 版本。

## 结构关系

结构关系指的是学生与教师的模型家族之间的关系，可分为以下三类：

- **同一家族：** 教师和学生都属于同一个模型家族。
- **跨架构：**教师和学生属于不同的模型家族。

### 同一家族

当教师和学生属于同一模型家族时，就能实现完美的层对齐，直接将教师的每一层映射到学生。

该方法简单直接，但又很僵化；其应用仅限于特定的模型谱系。

- **典型用例**：将 Qwen-32B 蒸馏为 Qwen-7B。

### 跨架构

教师和学生的架构不同，很难收敛。

- **典型用例：**将 Transformer 转换为速度更快的线性模型。

## 训练方法

训练方法的性质决定了学生如何从教师那里学习：

- **离线**
- **在线**
- **自蒸馏**

![图 E. 蒸馏策略对比图：离线（静态数据集）、在线（联合训练）和自蒸馏（内部层细化）。（Kuriko IWAI 制作）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/11.webp)

### 离线蒸馏

离线蒸馏是一种标准方法，教师只需创建一个静态训练集，然后学生就可以使用该数据集进行训练。

学习过程非常稳定，但学生有时无法掌握复杂的模式。

- **典型应用场景：**标准模型压缩管道。

### 在线蒸馏

在线蒸馏会同时更新教师和学生，让教师在训练过程中适应学生的学习进度。

当有足够的显存和计算资源同时训练教师和学生时，这种方法很有竞争力。

- **典型应用场景：** 研究级协同训练和集成。

### 自蒸馏

学生通过让自身更深的层指导较浅的层来完善自身。

虽然这种方法往往会强化深层的错误，但它的优点是不需要任何教师模型。

- **典型用例：** DeepSeek 式内部层优化。

## 任务特定蒸馏

特定的架构需要专门的蒸馏逻辑：

- **序列蒸馏：**用于 NLP（例如 DistilBERT），其中学生学习匹配教师的隐藏状态和注意力头。
- **逻辑蒸馏：**用于强化学习或推理任务中，学生模仿教师的策略或价值函数。

## 实施策略：该走哪条路？

在实践中，这些模型蒸馏技术很少单独使用。

最有效的实现方式是将不同的策略结合起来，以平衡性能、成本和目标硬件限制。

下表列出了一些常见组合：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/12.webp)

**表 1.** 模型蒸馏策略组合比较。

## 模型蒸馏实际应用 — 将 GPT-4o 蒸馏成 Llama 3–1B

在本节中，我将把庞大的 GPT-4o 模型蒸馏成一个小型学生模型，即 Llama 3–1B 模型，用于边缘设备应用。

该蒸馏遵循**离线、响应式知识蒸馏**模式。由于无法访问 GPT-4o 的内部权重，我会蒸馏它的输出，生成高质量的指令数据集，用于运行 SFT 来训练学生模型。

该过程主要包括以下四个步骤：

- **步骤 1.** 提示 GPT-4o 总结 50,000 份法律摘要，并给出详细解释。
- **步骤 2.** 收集教师的输出结果作为真实值。
- **步骤 3.** 在步骤 2 中根据真实数据对学生（Llama 3–1B）进行微调。
- **步骤 4.** 学生进行推理。

### 步骤 1. 提示 Gemini 3.1

第一步是调用 Gemini 3.1 API 生成输出：

```python
from openai import OpenAIclient = OpenAI(api_key="YOUR_OPENAI_API_KEY")
queries = ["Legal text A...", "Legal text B..."]
teacher_outputs = [summarize_with_gpt4o(q) for q in queries]
```

### 步骤 2. 收集教师输出

教师的输出结果以结构化方式保存在 JSON 文件中：

```
import jsondataset = []
for i, (original, summary) in enumerate(zip(queries, teacher_outputs)):
    dataset.append({"id": i, "input": original, "teacher_summary": summary })with open("teacher_data.json", "w") as f:
    json.dump(dataset, f)
```

### 步骤 3. 微调 Llama 3–1B

使用步骤 2 中的数据集，对学生模型 Llama 3–1B 进行微调：

```python
from trl.trainer.sft_trainer import SFTTrainer
from transformers import TrainingArguments, AutoModelForCausalLM, AutoTokenizer
model_id = "meta-llama/Llama-3.2-1B"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(model_id, device_map="auto")
trainer = SFTTrainer(
    model=model,
    train_dataset=teacher_outputs, 
    processing_class=tokenizer,
    args=TrainingArguments(
        output_dir="./llama-3-legal-distilled",
        per_device_train_batch_size=4,
        gradient_accumulation_steps=4,
        learning_rate=2e-5,
        num_train_epochs=3,
        save_steps=100,
        logging_steps=10,
        bf16=True
    ),
)
trainer.train()
```

### 步骤 4. 执行推理

最后，学生运用推理来评估结果：

```
import torchdevice = "cuda" if torch.cuda.is_available() else "cpu"
model = trainer.model.to(device)query = "The petitioner claims a violation of the 4th Amendment..."
inputs = tokenizer(f"Summarize: {query}", return_tensors="pt").to(device)outputs = model.generate(**inputs, max_new_tokens=200)
```

现在，蒸馏后的 1B 模型可以达到教师 95% 的质量，但运行速度却快 100 倍。

## 总结

模型蒸馏将 LLM 工程的重点从“我们能做到多大？”转移到了“我们能做到多小？”

通过有效地将知识从教师传递给学生，人工智能应用不仅可以很智能，而且可以很可持续、反应迅速。

### 何时转型：蒸馏 vs. RAG vs. 微调

模型蒸馏虽然是将大型模型缩小为更小、更快版本的一种有效方法，但它并不总是最佳选择。

以下五种情况下，更适合采用其他调优方法（例如微调或 RAG）：

### 1\. 高风险领域专业化 -> 选择微调。

蒸馏会导致深度或细致推理的丧失。

虽然蒸馏后的模型模仿了教师的风格，但可能会失去专业领域所需的精确事实。

完全微调或参数高效微调 (PEFT) 更适合融入特定领域知识。

- **应用案例：**医疗诊断、法律合同分析或专业工程。

### 2\. 频繁的数据更新 -> 选择 RAG 或上下文工程。

蒸馏是一个静态过程；如果信息发生变化，就必须重新蒸馏学生模型，这在计算上成本很高。

**检索增强生成（RAG）**是这里首选的方法，因为它允许模型访问新鲜数据而无需重新训练。

- **应用场景：**实时新闻机器人、股票市场分析或公司内部维基。

### 3\. 安全关键型应用 -> 选择微调或 RLHF。

研究表明，与 Teacher 相比，蒸馏（尤其是基于 logit 的蒸馏）可能会使安全防护措施降低高达 50%。

学生会优先模仿性能，而忽视安全约束。

使用带有安全标签的数据进行直接微调，能更可靠地维护护栏。

- **应用案例：**面向公众且具有严格合规性的人工智能。

### 4) 计算资源有限 -> 选择 PEFT（LoRA 或 QLoRA）。

蒸馏是一种成本很高的方法，因为庞大的教师模型必须生成数百万个合成标签，然后从头开始训练学生模型。

LoRA 或 QLoRA 更便宜、速度更快，因为它只调整大型模型中所有参数的一小部分（<1%）。

- **使用场景：** GPU 资源有限的初创公司或研究人员。

### 5) 弥合巨大的容量差距 -> 选择多阶段调优。

如果教师和学生之间的差距太大，学生就无法有效学习，因为它无法理解教师的复杂性。

在这些情况下，在高质量标注数据上做**监督式微调 (SFT)**，比强迫一个小模型去模仿大模型能得到更好的结果。

- **用例：** 尝试将 4000 亿参数模型直接蒸馏为 10 亿参数模型。

### 作者：Kuriko IWAI

- **阅读我的科技博客** 👉 访问 [博客存档](https://kuriko-iwai.com/archive)
- **将机器学习系统部署到您自己的 VPC** 👉 查看 [解决方案](https://kuriko-iwai.com/enterprise)
- **边做边学** 👉 报名参加 [人工智能工程大师班](https://kuriko-iwai.com/courses)

![人工智能工程大师班](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6a87e7033156d624aff0dc64b8d6f6eeba6f0cef/ai-insights/2026-06/30/images/model-distillation-guide-compressing-llms-for-edge-efficiency/13.webp)
