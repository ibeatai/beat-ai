---
title: "Agentic AI：数据碎片化与 LLM 推理的双重挑战"
author: Debmalya Biswas
url: https://ai.gopubby.com/agentic-ai-for-industrial-iot-use-cases-f6a8036a2ea3
translated: 2026-06-04
excerpt: 面向工业物联网系统的 AI agent
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d862590ea9216ae63f9cd9b4163d944ec90098a3/ai-insights/2026-06/04/images/agentic-ai-for-industrial-iot-use-cases/01.thumb.webp
---

# Agentic AI：数据碎片化与 LLM 推理的双重挑战

面向工业物联网系统的 AI agent

## 1. 引言

Agentic AI 系统的核心特征是自主性和推理能力：它们能把复杂任务拆解成更小的可执行任务，再编排这些任务的执行，并在需要时对执行过程进行监控、反思和调整／自我纠正。正因如此，

> agentic AI 有潜力颠覆当今企业里几乎所有的业务流程。

本文将展示如何把 agentic AI 应用到制造设施、楼宇和工厂这类**工业物联网**环境。工业物联网聚焦于物理资产的自动化与监控，例如压缩机、冷水机组、空气处理机组 (AHU)、暖通空调 (HVAC) 设备，目的是实现预测性维护和能源优化。

在这一背景下，我们提出了一套 agentic AI 系统，用自然语言高效查询工业物联网系统中的历史／实时传感器数据。例如下面这条查询：

> "2025 年 8 月 10 日，瑞士厂区的 B2 HVAC 2-1-1 用了多少电？"

就体现了 agent 的交互方式。

> 即便是这样简短的查询，也需要对多个领域概念做推理，例如资产、传感器、时间戳和厂区位置，这凸显了 agent 必须具备领域感知和上下文敏感能力。

工业物联网环境中的**数据碎片化**问题让这件事更加棘手：运行仪表数据来自异构的 SCADA（监控与数据采集）系统及其他物联网平台，由于缺乏共享本体，命名各不相同。此外，FMEA（失效模式与影响分析）这类工程产物也很少被纳入工程工作流。

为此，我们先在第 2 节勾勒出 agentic AI 平台的参考**架构**，再在第 3 节梳理适用于制造／工业物联网环境的特定任务 agent 和工具。整个流程分两个阶段：

- 先把异构物联网数据抽象为机器**语义**，沉淀故障与维护历史（第 3.1 节）；
- 再用推断出的语义约束 LLM 推理，并借助**内省**策略以迭代方式进一步确保执行计划符合运营标准（第 3.2 节）。

## 2. Agentic AI 参考架构

本节梳理参考 agentic AI 平台的关键模块，如图 1 所示：

- **推理**模块：拆解复杂任务并调整其执行，以达成给定目标；
- agent **市场**：汇集已有的、可用的 agent；
- **编排**模块：编排并监控（观测）多 agent 系统的执行；
- **集成**模块：对接企业系统，例如 SCADA、知识库；
- 共享**内存**管理，供 agent 之间共享数据和上下文；
- **治理**层，涵盖可解释性、隐私、安全、安全护栏等。

![图 1：Agentic AI 平台参考架构（作者绘图）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d862590ea9216ae63f9cd9b4163d944ec90098a3/ai-insights/2026-06/04/images/agentic-ai-for-industrial-iot-use-cases/01.webp)


给定一个用户任务，agentic AI 平台的目标是找出（组合出）能执行该任务的 agent（或 agent 组）。因此第一个需要的组件就是**推理**模块，它能把任务拆成子任务，再由编排引擎协调各 agent 的执行。

解决这类复杂任务的高层思路是：(a) 把给定的复杂任务**分解**成（一组层级或工作流形式的）简单任务，再 (b) 组合出能执行这些简单任务的 agent。这一步可以动态完成，也可以静态完成。动态方式下，面对复杂的用户任务，系统会依据运行时可用 agent 的能力，临时制定一份满足请求的计划。静态方式下，则在设计时基于一组 agent 手工定义组合 agent，把它们的能力拼起来。

> 思维链 (CoT) 是当今应用最广的分解框架，它把复杂任务转化为多个可控的小任务，并让模型的思考过程变得可解读。

此外，**ReAct**（推理与行动）框架让 agent 能批判性地审视自己的行动和输出，从中学习，进而改进自己的计划／推理过程。

agent 组合的前提是存在一个 [agent 市场](https://ai.gopubby.com/ai-agents-marketplace-discovery-for-multi-agent-systems-27a31b6b1ca6)／agent 注册表，其中对 agent 的能力和约束有清晰描述。例如 Agent2Agent (**A2A**) 协议定义了 [Agent Card](https://google.github.io/A2A/topics/agent-discovery/)（一份 JSON 文档）的概念，它相当于 agent 的数字"名片"，包含以下关键信息：

```yaml
Identity: name, description, provider information.
Service Endpoint: The url where the A2A service can be reached.
A2A Capabilities: Supported protocol features like streaming or pushNotifications.
Authentication: Required authentication schemes (e.g., "Bearer", "OAuth2") to interact with the agent.
Skills: A list of specific tasks or functions the agent can perform (AgentSkill objects), including their id, name, description, inputModes, outputModes, and examples.
```

要编排多个 agent，就需要一个支持多种 agent 交互模式的**系统集成模块**，例如 agent 间 API、agent API 把输出交给人使用、人触发 AI agent、AI agent 之间交互且有人参与其中 (human in the loop)。这些集成模式需要底层的 Agent OS 平台来支撑。

举例来说，可以参考 Anthropic 近期提出的模型上下文协议 ([MCP](https://www.anthropic.com/news/model-context-protocol)），它用于把 AI agent 连到企业数据所在的外部系统。

考虑到这类复杂任务往往长时间运行，[**内存**管理](https://ai.gopubby.com/long-term-memory-for-agentic-ai-systems-4ae9b37c6c0f)对 agentic AI 系统至关重要。

> 这既要在任务之间共享上下文，也要在很长一段时间内维持执行上下文。

标准做法是把 agent 信息的 embedding 表示存进一个支持最大内积搜索 (MIPS) 的向量数据库。为了快速检索，会用近似最近邻 (ANN) 算法，它返回近似的前 k 个最近邻，用少许精度损失换来巨大的速度提升。

最后是**数据治理模块**。我们要确保用户为某个任务提供的数据、或跨任务的用户画像数据，只共享给相关 agent（表／报表的鉴权与访问控制）。要了解打造一个治理良好的 AI agent 平台所需的关键维度，可参阅我之前关于[负责任的 AI agent](https://ai.gopubby.com/responsible-agentops-8d90fbd84985) 的文章，里面讨论了幻觉[护栏](https://medium.com/data-science-collective/guardrails-for-ai-agents-8913f6b67b51)、数据质量、[隐私](https://medium.com/ai-advances/privacy-risks-of-large-language-models-llms-5c0f96dccc56)、可复现性、可解释性、人在回路 ([HITL](https://medium.com/ai-advances/human-in-the-loop-strategy-for-agentic-ai-d9daa22c3204)) 等议题。

## 3. 面向工业物联网的 Agentic AI 架构

本节把上面给出的参考 agentic AI 平台，改造成能服务工业物联网环境的版本。可以用 API 检索结构化和非结构化数据，再由一个推理型 LLM 充分利用这些数据来做决策／决定下一步动作。

更具体地说，我们提出的**多 agent 系统**提供以下三大核心能力：上下文推理、专用 agent、工具集成，如图 2 所示。

![图 2：面向工业物联网的 Agentic AI 参考架构（作者绘图）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d862590ea9216ae63f9cd9b4163d944ec90098a3/ai-insights/2026-06/04/images/agentic-ai-for-industrial-iot-use-cases/02.webp)

**上下文推理**：处理特定领域的推理任务，例如

- 传感器消歧，
- 实体接地 (entity grounding)，以及
- 统计推理任务，例如上周、最大值。

> 工业物联网 agent 遵循 ReAct 风格的框架来处理自然语言查询，为工业数据检索、监控／预测性维护生成可落地的洞见。

**专用 agent**：面向工业物联网领域，例如

- 实时监控 agent，
- 根因分析 agent，
- 异常检测 agent，
- 预测性维护 agent。

> 这些 agent 用工具查询历史和实时数据，获取运营指标，例如能耗、吨位、温差和传感器健康状况。

**工具集成**：支持复杂的多参数工业工具。其中包括四个专用工具，用于

- 外部数据访问（传感器、资产、厂区、历史记录），以及
- 两个内部实用工具（JSON 处理器、时钟——返回当前时间）。

> 这些工具带来了独特的挑战，例如如何稳健地解析时间参数和字符串参数。

工具主要检索并输出：

- 传感器数据（例如温度、湿度、功率读数），含元数据（例如单位、来源），以及
- 厂区信息（例如区域标签、设备关系）。

随后，agent 要么直接返回这些原始数据点，要么在此基础上生成简洁的（人类可读的）摘要，例如资产／厂区摘要或历史记录。

### 3.1 制造业维护场景的数据对齐

遗憾的是，由于底层数据高度碎片化，在制造业场景中利用上述 agent 架构并不容易。即便是最常见的**机器维护**决策也是如此：

> 机器行为发生变化时，哪些故障假设是合理的？哪些与机器维护历史相符？建议采取什么措施？这种变化能否提前预测？

由于数据散落在仪表、工单和工程知识之间，运行信号与资产模型之间的关系又常常不清晰，推理便会失败。于是领域专家 (SME) 不得不手工调和这些数据源，既可能引入偏差，又增加成本。

我们主要关注以下三类数据源：

- 工单以（大多为）非结构化文本记录过往的症状、诊断和处置，沉淀着经验性的机器维护知识。
- 运行传感器（"仪表"）是低频、经过筛选的测量值（例如状态量和周期读数），其含义主要来自时间模式。
- 故障知识用 FMEA 表示，把机器部件与可能的故障模式及影响因素关联起来，从而约束推理。

接下来，我们将展示如何把仪表历史抽象为行为摘要、把工单抽象为机器维护模式，并用 FMEA 导出的语义约束 LLM 推理的空间。

- 工单摘要从历史数据中提取症状、处置和结果的重复模式，并按维护事件分组。
- 仪表读数摘要把低频运行指标转化为行为摘要，从而刻画使用情况和状态演变。视底层仪表语义而定，这包括趋势检测、复位识别、异常／漂移刻画等模式。
- 故障知识链接过程构建出一个结构化的假设空间，把观测到的行为与基于工程的故障语义连起来。本例中，它通过一套基于非平衡最优传输 ([UOT](https://arxiv.org/abs/1607.05816)) 的语义匹配过程，把工单数据与资产故障机理关联起来。

> 需要强调的是，在这一阶段所有数据抽象都是确定性的，为下游的 LLM 推理产出可审计、可解释的输入。

### 3.2 带内省的 ReAct 推理

标准的 ReAct agent 对 Web 检索这类任务很有效，但事实证明它应付不了工业物联网环境，常常表现出：

- 特定领域推理的缺口，例如把冷水机组吨位与能效挂钩——这在工业物联网环境里是一条关键链路。
- 推理不一致，例如日期偏移推理（"最后一天／周／月"）。
- 任务过早终止、冗余的工具调用，以及多步组合失败。

为攻克这些难题，我们用一套迭代式的 ReAct + **内省**策略来增强 agent，让 agent 系统能处理复杂的、工业领域特有的查询。内省策略的流程如下，见图 3。

![图 3：带内省的 ReAct agent 推理（作者绘图）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d862590ea9216ae63f9cd9b4163d944ec90098a3/ai-insights/2026-06/04/images/agentic-ai-for-industrial-iot-use-cases/03.webp)

**蒸馏**模块充当预处理器，把复杂查询拆解成结构化的语义单元：变量、[约束](https://medium.com/ai-advances/ai-agents-marketplace-discovery-for-multi-agent-systems-27a31b6b1ca6)和目标。ReAct 仍是底层的编排框架，针对给定的用户查询生成执行计划。

> 为了提升推理保真度，尤其是实体消歧，编排器会在启动执行前发出一条内部**子查询**，用来引导后续推理。

这条前瞻性查询同样由 LLM 作答，能改善计划的连贯性、任务贴合度和工具调用准确率——下面是一段系统提示示例。

```
You are an advanced reasoning agent that can improve based on introspection.
You will be given a previous reasoning trial in which you were given access to
multiple agents and tools and a query to answer. You were unsuccessful in resolving the query correctly either 
because you misunderstood the query, 
or you used up your set number of reasoning steps. In a few sentences, diagnose a possible reason for failure and 
devise a new high-level execution plan that aims to mitigate the same failure. Use complete sentences.
Here are some examples:
{examples}
Previous trial:
Query: {query}
{plan}
```

**审查**模块充当 [LLM-as-a-Judge](https://medium.com/data-science-collective/guardrails-for-ai-agents-8913f6b67b51) 验证器，根据生成的输出是否回应了用户查询，把最终推理步骤的输出分类为

- 已完成，
- 部分完成，或
- 失败；

这会触发**反思**模块，对执行计划做内省，评估推理步骤、agent／工具调用等，并输出

> 形式为执行计划调整或推理模板的针对性反馈，这些内容会加进系统提示，以指导后续的执行。

## 4. 结论

Agentic AI 是一种强大的范式，有潜力颠覆当今企业里许多普遍存在的流程。本文聚焦制造业中无处不在的工业物联网环境。

制造业被视为一个相当保守的行业，对技术进步、尤其是 AI，往往持观望态度。我们证明了，今天生成式 AI 和 agentic AI 已能为工业物联网环境带来可观价值，围绕实时资产监控、根因分析和预测性维护重新设想传统制造流程。

为此，我们梳理了工业领域特定任务的 agent 和工具，把它们集成进一个参考 agentic 平台，从而无缝地用自然语言回答用户查询。我们还展示了如何在（默认的）基于 ReAct 的 agentic 框架上，叠加数据对齐和"内省"，以应对实体消歧等领域相关的推理挑战。
