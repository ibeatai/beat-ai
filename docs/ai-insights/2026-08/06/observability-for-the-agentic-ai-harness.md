---
title: Agentic Harness 的可观测性
author: Debmalya Biswas
url: https://ai.gopubby.com/observability-for-the-agentic-ai-harness-07b322518206
translated: 2026-08-06
excerpt: OpenTelemetry 日志记录、Evals 与 FinOps（适用于 AI agent）
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/observability-for-the-agentic-ai-harness/01.thumb.webp
---

# Agentic Harness 的可观测性

OpenTelemetry 日志记录、Evals 与 FinOps（适用于 AI agent）

## 1. 引言

在 agentic AI 领域，harness 已成为新的热门词汇。随着企业对 AI agent 的广泛采用，开发 agent 本身已变得相对容易。而真正的挑战在于如何大规模、稳定地运行这些 agent。

对于任何在企业中实际构建 agentic AI 系统，并大规模应用软件工程最佳实践和架构良好的框架的人来说，这一切都不应该感到惊讶。

虽然 Anthropic 提出了 [agentic 编程](https://www.anthropic.com/engineering/harness-design-long-running-apps)的概念，但它同样适用于任何 agentic 设计。

> 核心假设是，重点已经从 LLM 转移到围绕 LLM 构建有效的 harness，以便能够以可靠性和问责制保证来管理 agentic 执行。

因此，需要从*孤立智能*转向*受控执行*。其关键组成部分包括（如图 1 所示）：

![图 1：agentic AI harness 的构建模块（作者提供图片）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/observability-for-the-agentic-ai-harness/01.webp)

- 推理循环：生成计划（[图](https://medium.com/ai-advances/why-designing-efficient-agentic-ai-workflows-is-so-hard-f6ceb07496aa)），执行、反思并循环/调整，以实现底层功能。
- 上下文（[记忆](https://medium.com/ai-advances/long-term-memory-for-agentic-ai-systems-4ae9b37c6c0f)）管理：优化上下文，在存储到短期记忆 (STM) 之前总结记忆项目，并将 STM 项目转换为长期记忆 (LTM)。
- [Evals](https://medium.com/ai-advances/agentic-ai-evaluation-strategy-15db5e554180)：利用 LLM-as-a-Judge 来评估响应的质量和 agent 结果，并酌情利用 ground truth。
- [安全](https://medium.com/ai-advances/agentic-ai-security-patterns-ad4ff80b9351)：用户 → 应用程序 → agent → 工具 →（数据）源系统之间标准化和可扩展交互的模式。
- [护栏](https://medium.com/data-science-collective/guardrails-for-ai-agents-8913f6b67b51)：防止攻击，例如提示注入、agent/工具误用、记忆投毒。
- 人在回路（[HITL](https://medium.com/ai-advances/human-in-the-loop-strategy-for-agentic-ai-d9daa22c3204)）：将人类作为一等公民融入 agentic 生命周期（而不仅仅是作为监督者），并提供适当的 UI/UX 来支持交互和干预。

上述大部分关键组成部分我之前都写过——已链接到相应的文章。

> 在本文中，我们将重点介绍对监控和管理 harness 组件至关重要的 agentic **可观测性**层。

首先，我们在第 2 节中定义 agentic 可观测性层：其范围、功能和架构——并将其置于参考 agentic 平台中。然后在第 3 节中定义需要记录的 [OpenTelemetry](https://opentelemetry.io/) (OTel) 属性，以实现溯源和可观测性。

最后，我们分别在第 4 节和第 5 节中展示如何利用可观测性层进行评估和 **FinOps**——作为持续改进的一部分，找出功能与成本上的效率提升点。

## 2. Agentic AI 可观测性

### 2.1 Agentic AI 参考架构

图 2 展示了构成第 2.2 节所述可观测性层基础的 agentic AI 平台的关键组成部分：

- **推理**层：分解复杂任务并调整其执行方式以实现给定目标；
- agent **市场/注册中心**：现有和可用的 agent、工具和模型；
- **编排**模块：用于编排和监控（观察）多 agent 系统的执行；
- **集成**模块：MCP 工具，用于与企业系统集成，例如 ERP、CRM、知识库；
- 共享**记忆**管理，用于 agent 之间的数据和上下文共享；
- **治理**层，包括可解释性、隐私、安全、安全护栏等。

![图 2：agentic AI 平台参考架构（作者提供图片）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/observability-for-the-agentic-ai-harness/02.webp)

给定用户任务，agentic AI 平台的目标是识别（组合）出能够执行该任务的 agent（或 agent 组）。因此，我们需要的第一个组件是一个**推理**模块，它能够将任务分解为子任务，并由编排引擎协调各个 agent 的执行。

思维链（CoT）是目前应用最广泛的分解框架，它将复杂任务分解为多个可管理的任务，并有助于理解模型的思维过程。此外，**ReAct**（推理与行动）框架允许 agent 批判性地评估自身的行动和输出，从中学习，并进而改进其计划/推理过程。

agent 组合意味着存在一个 [agent 市场](https://ai.gopubby.com/ai-agents-marketplace-discovery-for-multi-agent-systems-27a31b6b1ca6)/agent 注册表，其中包含对 agent 功能和约束的明确描述。例如，Agent2Agent (**A2A**) 协议定义了 [Agent Card](https://google.github.io/A2A/topics/agent-discovery/)（一个 JSON 文档）的概念，它相当于 agent 的数字“名片”。其中包含以下关键信息：

```yaml
Identity: name, description, provider information.
Service Endpoint: The url where the A2A service can be reached.
A2A Capabilities: Supported protocol features like streaming or pushNotifications.
Authentication: Required authentication schemes (e.g., "Bearer", "OAuth2") to interact with the agent.
Skills: A list of specific tasks or functions the agent can perform (AgentSkill objects), including their id, name, description, inputModes, outputModes, and examples.
```

鉴于需要编排多个 agent，因此需要一个**系统集成层**来支持不同的 agent 交互模式，例如：agent 之间的 API 调用、由人来消费输出的 agent API、人触发 AI agent、带人在回路的 AI agent 之间协作。这些集成模式需要得到底层 Agent OS 平台的支持。

我们参考 Anthropic 最近提出的模型上下文协议（[MCP](https://www.anthropic.com/news/model-context-protocol)），该协议旨在将 AI agent 连接到企业数据所在的外部系统/工具。MCP 被称为 AI 模型的“USB-C”，它通过三个主要构建模块实现互操作性：资源、提示和工具。通过标准化这些模块，

> 任何使用 **MCP** 的 AI 系统都可以通过任何兼容的 MCP 服务器理解如何请求数据（资源）、提供指令（提示）或执行操作（工具）。

鉴于复杂 agent 需要长时间运行，**记忆**管理对于 agentic AI 系统至关重要。这既包括任务间的上下文共享，也包括长时间内维持执行上下文。

标准做法是将 agent 信息的嵌入表示保存到支持最大内积搜索（MIPS）的向量存储数据库中。为了快速检索，可以使用近似最近邻（ANN）算法，该算法返回近似前 k 个最近邻，以牺牲部分精度换取巨大的速度提升。关于此主题的详细讨论，请参阅我之前发表的关于 [agentic AI 长期记忆](https://ai.gopubby.com/long-term-memory-for-agentic-ai-systems-4ae9b37c6c0f)的文章。

最后是**治理**层。我们需要确保用户共享的与特定任务相关的数据，或跨任务的用户画像数据，只与相关的 agent 共享（表/报表的鉴权和访问控制）。关于构建治理良好的 agentic AI 平台所需的关键维度，请参阅我之前关于负责任 AI agent 的[文章](https://ai.gopubby.com/responsible-agentops-8d90fbd84985)。

### 2.2 Agentic AI 可观测性层

如上所述，agentic AI 系统通常涉及编排多个 agent、工具和模型（LLM），从而使决策路径随 agent 的推理而变得不确定。鉴于此，agentic 可观测性对于以下方面至关重要：

![图 3：agentic AI 可观测性支撑的端到端追踪和监控（作者提供图片）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/observability-for-the-agentic-ai-harness/03.webp)

- 对 agent、工具、模型（LLM）、用户、上游和下游应用程序进行*端到端追踪*——如图 3 所示。
- 对 LLM 使用情况、故障、幻觉、延迟峰值、成本超支等进行*因果分析*，从而实现性能和成本优化。
- *可审计性*和治理，为多 agent 流水线提供不可篡改的溯源、安全与护栏证据，以及成本（用量）归因。
- *持续改进*，让工程团队能够基于已验证的指标改进提示、agent 逻辑和策略。

> 还需强调的是，agentic 可观测性涵盖了构建时可观测性和运行时可观测性——如图 4 所示。

![图 4：agentic 生命周期中的构建时与运行时可观测性（作者提供图片）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/observability-for-the-agentic-ai-harness/04.webp)

**构建时可观测性**侧重于在系统部署之前分析其代码、配置和产物——通过检查构建或部署阶段的合规性和潜在问题，在开发周期的早期发现问题。

在 agentic 环境中，这意味着验证与 agentic 流水线中组件的结构和设计相关的指标，以及来自模拟或运行的功能和性能数据。

**运行时可观测性**利用生产环境中的日志、指标和追踪等数据，提供对系统运行时行为、性能和错误的实时洞察。

> 在 agentic 场景中，由于运行的不确定性，运行时可观测性更为重要。

- 运行时可观测性对于持续监控和确保 agent 的行为与其预期目的保持一致至关重要，尤其是在 agent 拥有改变其目标或计划的自主权时。
- 最关键的对齐风险，例如目标漂移或递归循环，是动态出现的，仅靠构建时配置无法完全控制。

## 3. OpenTelemetry (OTel) 日志记录

在本节中，我们将深入探讨实现上述 agentic 可观测性能力所需的 OpenTelemetry (OTel) 属性。

**设计原则**

- *规范标准优先***:** 使用 W3C 追踪上下文（`traceparent`、`tracestate`）作为主要传播格式，避免出现大量自定义标头。
- *一个任务锚点，多个 span***:** 将“AI 任务”视为锚定到一条 trace 的逻辑操作，其中模型/工具调用是子 span，重试是链接到同一条 trace 的新 span。
- *只做链接，不要分叉：* 使用 OTel 的“links”来表示基于事件的流程中的扇出/扇入、计划任务和跨 trace 的汇聚。
- *ID 中不含语义：* ID 是不透明的、全局唯一的、不可猜测的——语义属于属性。

因此，目标是确定一组最小且可移植的属性，每个 agentic 流水线都必须记录、存储和共享这些属性。所需的标识符（规范标识符）包括：

- *trace\_id*：任务/跨运行的关联
- *span\_id*：任务内的操作
- *parent\_span\_id*：层级因果关系
- *links\[\]*：非层级因果关系（扇出/扇入，计划任务）

建议使用 *trace\_id* 作为跨系统和平台的 AI 关联 ID，以提供可见性。下表概述了 AI 特有的标准属性：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/observability-for-the-agentic-ai-harness/05.webp)

为了使 agentic 流水线可审计——可按溯源追踪——需要将这些属性添加到相关的 span 上。属性名称仅供参考，可以映射到 OTel Agentic AI 语义约定（待其最终确定）。在此之前，先给出 OpenTelemetry GenAI 语义约定仓库的链接（[link](https://github.com/open-telemetry/semantic-conventions-genai)）。

## 4. Agentic 评估与优化

指标是定量评估的关键，它衡量 agent 实现目标、检测错误和做出有效工具选择的程度。它们对于评估 agentic 用例的性能和效率至关重要。

### 4.1 Agent 效率

这些指标衡量 agentic 工作流的效率。

- *推理相关性：*确保 agent 的推理与用户查询一致。每次工具调用背后的推理是否与用户请求的内容明确相关？
- *推理连贯性：*检查 agent 推理的逻辑流程。推理是否遵循逻辑清晰、循序渐进的过程？每一步都应该为任务增添价值并具有意义。
- *答案相关性：*检查答案是否与输入内容相关？
- *事实依据性：*评估 agent 的回答在多大程度上锚定于事实、可验证且与上下文相关的来源，从而最大限度地减少幻觉和错误信息。
- *回答流畅性：*评估 agent 回答的可读性、语法正确性和自然度。
- *回答连贯性：*衡量 agent 的回答是否逻辑结构清晰，并在整个对话过程中保持清晰。
- *任务分解（规划）效率：*衡量 agent 将复杂任务分解为可管理子任务的能力。
- *agent 鲁棒性：*衡量 agent 在保持性能和可靠性的同时，处理意外输入、错误和对抗场景的能力。
- *agent 一致性：*衡量 agent 在多次面对相似输入的交互中产生稳定、可重复且逻辑连贯的回答的能力。

### 4.2 工具利用效率

这些指标用于评估 AI agent 选择和使用工具的有效程度。

- *工具选择准确率：*衡量 agent 为给定任务选择最合适工具的有效性。
- *工具使用效率：*衡量 agent 使用所选工具的最优程度，需考虑不必要的调用和资源占用等因素。
- *工具调用精度：*衡量工具调用中所用参数的准确性和恰当性。
- *工具调用成功率：* 工具调用整体的成功率。

### 4.3 基于 OTel 的实现架构

指标评估有两种类型：离线（开发期间）和实时（作为可观测性/监控的一部分）。

- 在基于日志的离线评估中，各个组件（规划器、agent 和工具）会生成包含特定信息且格式符合要求的日志。基于日志的评估器会审查这些日志，并根据日志内容生成评估结果。这是一种**非侵入式**评估方法。
- 在实时评估中，组件会调用评估服务并发送所需的产物。评估服务随后会汇总一次端到端执行的全部数据，并实时生成评估结果。该方法要求在组件代码中嵌入实时调用，并且需要与 agent/工具开发团队进行协调。

下图 5 显示了基于 OTel 的离线 agentic 评估的解决方案架构——步骤如下：

![图 5：基于日志的离线评估的解决方案架构（作者提供图片）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@7e0b7dc1fd05a94669b0ad6f51dfda14611e8685/ai-insights/2026-08/06/images/observability-for-the-agentic-ai-harness/06.webp)

**步骤 1：触发评估：** 用户通过调用 Evaluate API 来启动此过程。所需输入：

- agent 名称
- 工具描述
- 选定的指标，例如工具选择准确率、推理相关性
- 日期范围

**步骤 2：交互检索。** 系统根据 agent 名称和提供的日期范围查询日志数据库，以获取相关的交互历史记录进行评估。

**步骤 3：评估处理。** 评估引擎使用 LLM-as-a-Judge 来分析检索到的交互数据。它计算每个选定指标的得分，并生成相应的评分理由。评估结果随后安全地存储在 BLOB 存储中，以实现持久化保存。

**步骤 4：发布结果。** 用户可以通过以下方式查看评估结果：

- Get ResultList API：显示该 agent 所有评估的摘要列表。
- Get ResultDetails API：提供每次交互和每个指标的详细分数及其理由。

## 5. 基于 OTel 的 Agentic AI FinOps

面向 agentic AI 的 FinOps 可以定义为：

> 一种把财务、工程和业务结合起来的最佳实践——通过最大化价值和确保财务问责来管理 agentic AI 成本。

这需要利用数据驱动的洞察力来管理敏捷性、治理、成本与投资回报率之间的权衡，使企业能够通过资源合理配置和高效分配来主动优化 AI 支出。

在典型的 agentic AI 场景中，它通常是以下几种因素的组合：

- 计算基础设施
- 模型：大型语言模型（LLM）/小型语言模型（SLM）
- 存储：内存、用于搜索的向量数据库等。

让我们考虑一个参考场景：将 LangGraph 作为 agentic 开发框架，并部署在 Azure Kubernetes Service (AKS) 上：

- [LangGraph](https://www.langchain.com/langgraph) 编排 agent 执行（通过内部 API 或托管运行时）。
- agent 携带自定义逻辑或工具，以容器化的 agent 端点形式部署在 AKS 上。
- [AI Search](https://azure.microsoft.com/en-us/products/ai-services/ai-search) 对企业数据（向量 + 文本）建立索引，并作为检索增强生成 (RAG) 的知识来源。
- AKS pod 调用 AI Search 和/或模型端点（例如 Azure OpenAI GPT 或 Azure Foundry 中微调的 LLM/SLM）。
- OpenTelemetry 日志（如第 3 节所述）存储在 [Azure Monitor with Application Insights](https://github.com/open-telemetry) 中。

成本计算需要考虑以下参数——基于 AKS pod 的读/写操作、搜索查询延迟：

- 有多少个 agent pod 并发运行？平均而言，每个 agent 容器镜像的大小为 2–4 GB × 并发会话数。
- AKS 中暂存或缓存了多少 LLM/检查点？平均而言，大小为 1-10 GB（分词器、本地权重、嵌入缓存）。
- AKS ↔ AI Search / AI Foundry 之间的流量，延迟按小于 200 毫秒计。
- 向量存储（以在 AI Search 中建立索引的嵌入大小衡量）的大小根据使用情况的不同，可能在 100 MB 到 100 GB 之间变化。
- 每天每 100 个会话的日志量约为 100 MB。

对于同时运行的 5 个 agent，其代表性的容量数据如下：

- 5 个 pod × 每个 2 vCPU、6 GB 内存 → 共 10 vCPU、30 GB 内存
- 每个 pod 缓存 5 GB → 25 GB 临时 SSD
- 每天记录 1 GB 数据 → 每天 10 GB 遥测数据
- 向量数据总量约 25 GB，存储在 AI Search 中

虽然上文侧重于了解 **agentic 基础设施成本**，但下一节我们将重点分析 LLM 调用——因为它们仍然占整个 agentic 系统成本的大部分。

## 6. 结论

尽管 agentic AI 系统的优势显而易见，但它们是复杂的系统，难以进行可靠的管理。因此，对多 agent 流水线（包括多个 agent、工具和 LLM 调用）进行端到端的可观测性对于其在企业中的应用至关重要。

为此，我们提出了一种全面的 agentic 可观测性层，涵盖构建时和运行时可观测性。有效的日志记录是实现这一目标的关键，我们为此提出了 OpenTelemetry (OTel) agentic AI 语义约定。该约定梳理了 agent、工具、模型、安全性和治理的相关 OTel 属性。最后，我们展示了如何利用 OTel 日志进行 agentic 评估和 FinOps，从而实现成本效益和性能提升。

agentic AI 可观测性仍处于起步阶段，但发展非常迅速！随着 agent 开始带着记忆执行更长时间的任务，在多 agent 场景中借助工具协作，以及处理日益复杂的工作流程；建议尽早开始构建可观测性层，使其能够适应变化，而不是试图构建一个面向未来的可观测性平台。
