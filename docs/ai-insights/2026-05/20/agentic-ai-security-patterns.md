---
title: 智能体 AI 安全模式
author: Debmalya Biswas
url: https://ai.gopubby.com/agentic-ai-security-patterns-ad4ff80b9351
translated: 2026-05-20
excerpt: 智能体 AI 系统的关键特征是自主性和推理能力。凭借这两点，它们能把复杂任务拆解成更小的可执行任务，再编排这些任务的执行，并在需要时监控、反思、适配乃至自我纠错。正因如此，
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/agentic-ai-security-patterns/01.webp
---

# 智能体 AI 安全模式

## 面向企业级 AI 智能体与 MCP 工具的安全护栏

## 1. 引言

智能体 AI 系统的关键特征是自主性和推理能力。凭借这两点，它们能把复杂任务拆解成更小的可执行任务，再编排这些任务的执行，并在需要时监控、反思、适配乃至自我纠错。正因如此，

> 智能体 AI 有潜力颠覆当今企业中几乎所有业务流程。

于是我们基本上可以把一切都「智能体化」：从[客户服务台](https://medium.com/data-science-collective/reinventing-the-customer-service-desk-with-autonomous-ai-agents-ca5a0c00ba3f)到工业流程（例如 [HVAC 优化](https://medium.com/ai-advances/reinforcement-learning-agents-for-industrial-control-systems-b917b513f0c4)），甚至可以用智能体来构建底层软件、[数据](https://medium.com/data-science-collective/agentic-ai-for-data-engineering-4412d5e70189)和 ML 工程流水线。要支撑这个智能体化过程，需要一门全新的整体性学科，覆盖完整的智能体生命周期（图 1）：

-   从捕获智能体用例需求开始
-   到设计智能体（一个好的智能体层级结构长什么样？哪些智能体技能与工具适用？）
-   到它们在智能体平台上的**安全且可扩展的**实现
-   再到这些智能体的治理与维护。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/agentic-ai-security-patterns/01.webp)
*图 1：聚焦安全模式的智能体 AI 生命周期（作者绘图）*

智能体化过程中要牢记几条原则：

-   人们往往倾向于把手工流程一对一地照搬成智能体流程，但这是一种[低效的映射](https://medium.com/ai-advances/why-designing-efficient-agentic-ai-workflows-is-so-hard-f6ceb07496aa)。设计者应当记住，智能体并不像人那样受 HR 流程约束 :) 所以软件智能体能做的事情不一样，做事的方式也不同于人类。
-   与此同时，正如人在安全链条上是最薄弱的一环，单个智能体也足以让整个执行崩溃。这里没有例外，而且一旦某个智能体失控，我们甚至无从知道该责备、罚款、解雇谁。所以建议以同样极致的谨慎来设计每一个智能体，并配上日志记录、可观测性和负责任 AI 护栏。

> 如今，AI 智能体通过临时拼凑的端点对外暴露，导致安全、运维与合规控制陷入碎片化。

如果缺少一套架构良好的智能体安全架构，我们将面临以下风险：

-   *安全漏洞*：身份认证与授权的实现不一致；
-   *运维低效*：监控与遥测各自为政；
-   *合规风险*：审计追踪不足，存在未经授权的数据访问 / 数据治理缺失；
-   *可扩展性挑战：*缺乏集中化的（基于策略的）速率限制与限流；
-   *糟糕的用户体验（UX）*：智能体（工具与模型）的发现、调用模式以及用户访问控制都不够标准化。

本文将深入探讨智能体生命周期中的**安全**层面。更具体地说，我们要为下面这条链路上各方之间标准化、可扩展的交互定义安全模式：
用户 → 应用 → 智能体 → 工具 →（数据）源系统。

## 2. 智能体 AI 参考架构

图 2 展示了智能体 AI 平台的关键组件，它们构成第 3 节所述安全模式的基础：

-   **推理**层：分解复杂任务并适配其执行，以达成给定目标；
-   智能体**市场 / 注册中心**：收录现有且可用的智能体、工具与模型；
-   **编排**模块：编排并监控（观察）多智能体系统的执行；
-   **集成**模块：用于对接企业系统（如 ERP、CRM、KB 知识库仓库）的 MCP 工具；
-   共享**记忆**管理：在智能体之间共享数据与上下文；
-   **治理**层：包括可解释性、隐私、安全、安全性护栏等。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/agentic-ai-security-patterns/02.webp)
*图 2：智能体 AI 平台参考架构（作者绘图）*

给定一个用户任务，智能体 AI 平台的目标是识别并组合出一个（或一组）能执行该任务的智能体。因此我们需要的第一个组件是**推理**模块，它负责把任务分解为子任务，而相应智能体的执行则交由编排引擎来编排。

思维链（CoT）是当今使用最广泛的分解框架，它把复杂任务转化为多个可管理的小任务，同时让人得以一窥模型的思考过程。在此之上，**ReAct**（推理与行动）框架让智能体能批判性地评估自身的行动与输出，从中学习，进而优化计划和推理过程。

智能体组合意味着存在一个[智能体市场](https://ai.gopubby.com/ai-agents-marketplace-discovery-for-multi-agent-systems-27a31b6b1ca6) / 注册中心，其中对每个智能体的能力与约束都有明确描述。例如，Agent2Agent（**A2A**）协议定义了 [Agent Card](https://google.github.io/A2A/topics/agent-discovery/) 这一概念——它是一份 JSON 文档，相当于智能体的数字「名片」，包含以下关键信息：

```yaml
Identity: name, description, provider information.
Service Endpoint: The url where the A2A service can be reached.
A2A Capabilities: Supported protocol features like streaming or pushNotifications.
Authentication: Required authentication schemes (e.g., "Bearer", "OAuth2") to interact with the agent.
Skills: A list of specific tasks or functions the agent can perform (AgentSkill objects), including their id, name, description, inputModes, outputModes, and examples.
```

既然需要编排多个智能体，就需要一个**系统集成层**来支持不同的智能体交互模式，例如智能体对智能体 API、输出供人类消费的智能体 API、人类触发 AI 智能体、带人在环的智能体对智能体。这些集成模式都要由底层的 Agent OS 平台来支撑。

我们参考 Anthropic 近期提出的模型上下文协议（[MCP](https://www.anthropic.com/news/model-context-protocol)），它负责把 AI 智能体连接到企业数据所在的外部系统和工具。MCP 被称为 AI 模型的「USB-C」，通过三大构建块实现互操作性：

1.  *Resources（资源）*：服务器可以提供给 AI 的结构化数据，例如代码片段、文档片段或数据库查询结果——任何能补充事实性上下文的东西。
2.  *Prompts（提示）*：服务器可以提供的预制指令或模板，可以理解为用于摘要文本或以特定风格生成代码的现成提示。
3.  *Tools（工具）*：AI 可以请求服务器执行的实际操作。在检索一侧，这类操作包括查询数据库、搜索网络等。

把这三者标准化之后，任何使用 **MCP** 的 AI 系统都能理解如何通过任意兼容的 MCP 服务器来请求数据（资源）、提供指令（提示）或执行操作（工具）。

复杂智能体往往长时间运行，因此**记忆**管理对智能体 AI 系统至关重要。

> 这既包括任务之间的上下文共享，也包括在长时间跨度内维持执行上下文。

标准做法是把智能体信息的嵌入表示保存到一个支持最大内积搜索（MIPS）的向量存储数据库中。为了快速检索，会使用近似最近邻（ANN）算法——它返回近似的 top k 个最近邻，以一点准确性的损失换取巨大的速度提升。关于这个话题的详细讨论，可参阅我此前关于[智能体 AI 长期记忆](https://ai.gopubby.com/long-term-memory-for-agentic-ai-systems-4ae9b37c6c0f)的文章。

最后是**治理**层。我们需要确保：用户为某个任务分享的数据，或跨多个任务的用户画像数据，只与相关的智能体共享（表 / 报告的认证与访问控制）。要让一个治理良好的 AI 智能体平台在幻觉[护栏](https://medium.com/data-science-collective/guardrails-for-ai-agents-8913f6b67b51)、数据质量、[隐私](https://medium.com/ai-advances/privacy-risks-of-large-language-models-llms-5c0f96dccc56)、可复现性、可解释性、人在环（[HITL](https://medium.com/ai-advances/human-in-the-loop-strategy-for-agentic-ai-d9daa22c3204)）等方面到位，所需的关键维度可参阅我此前关于负责任 AI 智能体的[文章](https://ai.gopubby.com/responsible-agentops-8d90fbd84985)。

## 3. 智能体交互的安全模式

### 3.1 应用到智能体

我们先来定义用户 / 应用经由 AI 网关到智能体交互的安全模式。基于 AI 网关的端到端安全架构如图 3 所示。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/agentic-ai-security-patterns/03.webp)
*图 3：基于 AI 网关的智能体安全架构（作者绘图）*

该安全模式由以下组件构成：

-   调用 AI 智能体的用户和 / 或应用。
-   市场：基于 REST 的注册中心，用于发现智能体、工具与模型，并标明它们的能力、元数据和端点。
-   AI 网关：API 管理（AMIP）层，为所有交互强制实施安全、路由、限流和护栏。
-   IAM 提供方：人类用户使用 Entra ID，应用使用服务主体（托管身份）。
    （[Entra ID](https://learn.microsoft.com/en-us/azure/architecture/aws-professional/security-identity) 虽是 Azure 专属的，但其他平台上的等效 IAM 方案同样适用于这里所述的安全模式。）
-   记忆：维持用户会话上下文与对话状态（用于多轮对话）。
-   （Open）telemetry：用于监控、合规与分析的集中化日志记录。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/agentic-ai-security-patterns/04.webp)
*图 4：用户（经由应用 / UI）到智能体的安全模式（作者绘图）*

用户（经由应用 / UI）到智能体的详细安全流程如图 4 所示，关键步骤如下：

1.  用户在业务应用 / UI 中发起交互。
2.  应用使用授权码 + PKCE（Proof Key for Code Exchange，授权码交换证明密钥）向 Entra ID 认证用户。
3.  Entra ID 向应用签发用户访问令牌。
4.  应用在请求头中携带该用户访问令牌，调用 AI 网关（APIM）。
5.  AI 网关与 Entra ID 执行代表（OBO）令牌交换，换取一个下游智能体令牌；*aud（audience，受众）= 智能体。*
6.  AI 网关校验令牌并强制实施策略（针对智能体作用域 / 用户角色做 JWT 校验）。
7.  AI 网关把携带已校验上下文的请求转发给智能体。
8.  智能体在自身层级对用户进行授权。
9.  智能体执行业务逻辑处理，再把响应返回给用户（经由应用 / UI）。
10.  AI 网关与智能体都把各自带时间戳的调用细节记录到 [OTel](https://opentelemetry.io/docs/specs/otel/logs/) 平台。

### 3.2 智能体到 MCP 工具（MCP 服务器与客户端）

本节把上面的用户 / 应用 / UI 到智能体的安全模式，扩展到智能体经由 MCP 调用工具的交互，以适配智能体需要调用工具来完成功能的场景。

最简单的形式是：MCP 客户端向授权服务器请求一个 [OAuth 2.0](https://oauth.net/2/) 访问令牌，随后携带该令牌调用 MCP 服务器 API。OAuth 2.0 规范定义了多种从授权服务器获取访问令牌的流程，下面几个小节说明其中最相关的几种。

**客户端凭证授权（CCG）流程**

CCG 流程由 MCP 客户端（嵌入在 AI 智能体内——图 2）使用，基于其自身的非人类身份从授权服务器获取新的访问令牌。

> CCG 流程用于 MCP 客户端不运行在「用户—智能体」上下文中的情况，支持在后台运行的长时间批处理进程。

需要注意，完全*自主*、需要非常宽泛且高 MCP 权限的 AI 智能体可能带来重大风险，除非有精密的动态访问管理和其他护栏对它加以约束。

**OBO AI 智能体——令牌交换（TE）流程**

TE 流程供 MCP 客户端使用：它代表用户，用一个来自上游系统的访问令牌，向授权服务器换取一个新的访问令牌。

> 因此，TE 流程用于 MCP 客户端代表用户运行、服务近实时用例的情况。

一般而言，AI 智能体与 MCP 服务器不得把从上游系统收到的访问令牌传播到下游系统，除非它们全部部署在同一个运行时平台上。

> 根据 OAuth 2.0 规范，令牌传播不得跨越应用边界，尤其是处于不同安全域中的应用边界。

**安全流程：** AI 智能体（MCP 客户端）→ MCP 服务器 → MCP 工具 API

有了上述背景，我们来梳理图 5 所示的一次参考性的「AI 智能体（MCP 客户端）到 MCP 服务器」交互的步骤：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/agentic-ai-security-patterns/05.webp)
*图 5：聚焦智能体生命周期中「AI 智能体（MCP 客户端）→ MCP 服务器 → MCP 工具 API」交互部分的安全模式（作者绘图）*

1.  智能体由用户 / 应用使用一个访问令牌调用（参见小节 3.1）。
    这个进来的访问令牌明确只针对该智能体，不能用于调用其他智能体或 MCP 服务器。更具体地说，令牌中的 sub（subject，主体）声明标识出原始用户；aud（audience，受众）声明把该智能体标识为令牌的预期接收方；令牌的作用域只对应该 AI 智能体所需的权限。
2.  AI 智能体需要调用某个工具（相应的 MCP 服务器）来完成其功能：
    智能体不能直接把它收到的（来自用户 / 应用的）访问令牌传播给 MCP 服务器，主要有两个关键原因：
    \- *溯源*：一旦传播出去，被 MCP 服务器调用的底层工具就不知道发起调用的是 MCP 服务器，看上去会像是应用直接发起了调用——这破坏了可审计性。
    \- *作用域*：收到的令牌所带的权限作用域，可能与 MCP 服务器所需的不一致。
    因此 AI 智能体要执行一次令牌交换（TE）：智能体向授权服务器的令牌端点发起调用，携带这些细节——自身凭证、收到的访问令牌、新令牌的作用域与受众。
3.  授权服务器校验来自 AI 智能体的这个请求。校验通过后，它签发一个明确只针对 MCP 服务器、作用域受限的新访问令牌。新令牌中的 sub（subject，主体）声明仍然标识原始用户，从而保留用户上下文。
4.  智能体调用 MCP 服务器，请求中携带交换得到的访问令牌。如前所述，出于令牌传播的风险，MCP 服务器可能还要再做一次令牌交换才能调用下游工具 API，除非两者部署在同一个应用 / 平台域中。

### 3.3 从（下游）源系统检索数据

本节聚焦数据检索这一环节，来补全整个智能体安全生命周期，*也就是智能体需要从记忆、结构化或非结构化数据源检索数据时的情形*——参见图 3*。（*智能体记忆也被视为一种数据存储平台，所以同样适用类似的安全模式。）

正如你到这里一定已经发现的，令牌生成、校验与交换的安全模式始终不变；只要一次交互（这里是 MCP 工具与存储平台之间）跨越了安全域，就需要做一次令牌交换。端到端的安全模式如图 6 所示。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/agentic-ai-security-patterns/06.webp)
*图 6：覆盖 用户 → 应用 / UI → AI 智能体 → MCP 服务器 / 工具 → 数据平台 的端到端安全模式（作者绘图）*

## 4. 结论

智能体 AI 系统的好处显而易见，但它们也是复杂系统，很难以安全且可扩展的方式落地。考虑到智能体系统非确定性、多层级的架构，涵盖 用户 → 应用 → 智能体 → 工具 →（数据）源系统，这无疑是一项极具挑战的任务。

为应对这一挑战，我们梳理了安全模式、架构组件和治理机制，以确保整个智能体生命周期的安全与合规，并以 AI 网关作为各智能体层之间的中央集成组件。

智能体 AI 安全仍处于起步阶段，但其重要性与日俱增。随着智能体借助记忆执行越来越长的任务、在多智能体场景中与工具协作、并处理日益复杂的数据工作流，建议尽早基于零信任和安全最佳原则纳入**安全设计（security by design）**——这能提升信任度，并加速企业对智能体工作流的采用。
