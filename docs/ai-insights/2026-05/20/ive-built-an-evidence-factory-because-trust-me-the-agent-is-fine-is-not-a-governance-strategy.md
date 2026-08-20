---
title: 我建了一座"证据工厂"，因为"相信我，agent 没问题"算不上治理策略
author: Marco van Hurne
url: https://generativeai.pub/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy-ac76d51fd075
translated: 2026-05-20
tags:
  - Artificial Intelligence
excerpt: 关于企业 agent 化，我写下的东西已经足够填满一本小书——一本会让翻开它的 AI 架构师怀疑自己职业选择的、令人不太舒服的书。流程选择、35% 的自动化天花板、agentic 架构模式、治理哲学、大规模运行一座 AI 工厂的经济学，我都用一种好为人师的口吻讲了个遍，也亲眼看着足够多的项目失败，因而挣得了发表观点的资格。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/01.webp
---

# 我建了一座"证据工厂"，因为"相信我，agent 没问题"算不上治理策略

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/01.webp)

关于企业 agent 化，我写下的东西已经足够填满一本小书——一本会让翻开它的 AI 架构师怀疑自己职业选择的、令人不太舒服的书。流程选择、35% 的自动化天花板、agentic 架构模式、治理哲学、大规模运行一座 AI 工厂的经济学，我都用一种好为人师的口吻讲了个遍，也亲眼看着足够多的项目失败，因而挣得了发表观点的资格。

但我漏掉了一样东西。

漏掉它，是因为它是运行 AI 工厂里最难的部分：它根本塞不进一张干净利落的框架幻灯片，而且它正是那种让财务紧张、让工程戒备、让合规去摸降压药的东西。我也漏掉它，是因为每次动笔写它，写出来的不是太偏执，就是太天真。

那么，我到底漏掉了什么？

一句话其实很简单。你选好了正确的流程、搭好了正确的架构、把正确的治理哲学嵌入†进 agent 里、把整套东西推上了生产环境——做完这一切，你仍然需要能够证明它在正常工作。当然不是向你自己和团队证明，而是向那位十八个月后坐在你对面的审计员证明。他手里有一份清单和一项授权，对你架构上的优雅毫无兴趣。

而这，正是证据工厂存在的意义。

在讲清楚它究竟是什么之前，我先把前面铺垫的内容收个尾——证据工厂只有放进它所服务的那个项目的上下文里才说得通。如果你一直在追这个系列，接下来两节可以略读。如果没有，它们很短，我保证不会过度重复自己——对我来说，这已经是相当大的克制了。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/02.webp)

† *是的，就嵌进 agent 里，不只是事后补救。我相信通过本体（Ontology）实现的 Neurosymbolic AI——以知识图谱为载体——能把治理边界写进 agent 的 DNA。我在这两篇里写过它：*

1.  [*The agentic governance debt crisis | LinkedIn*](https://www.linkedin.com/pulse/agentic-governance-debt-crisis-marco-van-hurne-lhyae/) *（关于嵌入式治理）*
2.  [*The boring AI that keeps planes in the sky | LinkedIn*](https://www.linkedin.com/pulse/boring-ai-keeps-planes-sky-marco-van-hurne-flruf/) *（关于 NS AI）*

## 不是每件事都配拥有一个 agent。你早就知道，但你还是这么干了。

我在 Eigenvector、与 InHolland 大学的学生合作运行的那个 agent 化研究项目，建立在跨 20 个行业的 177 个真实部署之上。它最根基的洞见是：agentic 自动化总是撞上一个天花板，大约停在流程步骤的 35%。这个天花板的成因，与其说是模型不够好，不如说是现实本身结构化得不够，达不到让模型派上用场的程度。

从那项研究中长出来的四象限框架‡，按结构属性给流程步骤分了类。象限 I 确定、结构化、低风险，今天就能完全自动化。象限 II 半结构化、复杂度中等，配上编排和护栏就能自动化——这是你启动 agent 化的甜蜜区。但大多数业务案例都活在象限 III：那里人类活动密集，同时高度模糊、例外繁多、依赖判断。这正是生成式 AI 开始产出胡言乱语、开始崩坏的地方。象限 IV 治理繁重、合规密集，不论技术上是否可行，自动化它在经济上都不理性。

企业 agent 化的原罪，就是挑了象限 III 和象限 IV 的流程去自动化——因为它们在董事会演示和业务案例里看起来很唬人。但 AI 平台供应商不会告诉你这点。他们会说，自家平台用强健的护栏和企业级安全来处理复杂性、模糊性和合规需求。但他们真正的意思是：平台有一个 human-in-the-loop 的勾选框和一套合规文档模板。这两者并不是一回事。

所以没错，流程选择本身就是治理。它是你在 agent 化项目里做的第一个治理决定，也是决定了其后每一个治理决定难度的那个决定。一开始就选错，再多的可观测性基础设施也救不了你。我接下来要描述的证据工厂，是为那些把流程选择做*对了*的项目设计的。它不是给那些把 agent 部署进象限 III 的项目的补救工具。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/03.webp)

‡ *关于这套方法及其底层论文的更多信息，请读：*

1.  [*Process mining is the strategic foundation your enterprise AI project is missing | LinkedIn*](https://www.linkedin.com/pulse/process-mining-strategic-foundation-your-enterprise-ai-van-hurne-fzqof/)
2.  *还有这一篇，里面有些关于流程的很酷的图：* [*The real story behind enterprise scale process agentification | LinkedIn*](https://www.linkedin.com/pulse/real-story-behind-enterprise-scale-process-marco-van-hurne-s2rqf/)

## 架构是有效的。直到它不再有效。

假设你选对了流程，下一个问题就是：agentic 架构长什么样，治理又安在它的哪个位置。这个系列别处我已经详尽写过，所以这里我会简明扼要——以一个写过四十页幻灯片的人那种"简明扼要"。哈哈哈，抱歉。

企业 agentic 部署里的成功模式，共享同一套结构逻辑。带明确工具边界的单一用途 agent，胜过带宽泛权限的通用 agent。带清晰委派链的编排者-子 agent 架构，胜过那种每个 agent 都跟其他所有 agent 对话、没人知道谁授权了什么的扁平多 agent 系统。在象限 II 的决策点上设 human-in-the-loop 闸门，对任何触及受监管数据或不可逆动作的事情来说，都胜过完全自主的执行。这是横跨 177 个部署的数据所显示的，也是更广泛的研究文献所证实的。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/04.webp)

我在整个系列里一直倡导的治理哲学，是嵌入式 DNA 治理，再叠加事后（post-factum）治理。这种做法把治理逻辑编织进 agent 的架构本身，而不是作为外部合规层栓在外面。我觉得最有说服力的技术机制是 neuro-symbolic AI：把负责模式识别的神经网络，和负责规则强制、逻辑约束检查的符号推理结合起来。我管它叫给 LLM 的"利他林（Ritalin）药丸"，因为它就是这个作用。神经成分提供智能、创造力，以及偶尔跑题、去做你没要求的事的倾向；符号成分提供规则和硬约束，把神经成分摁在我们设定的边界之内。

Neuro-symbolic 治理让我们能追溯推理过程，拿到清晰的审计轨迹。它还让 agent 能够证明，某个动作与它所遵循的策略框架是一致的。

嵌入式做法是对的。但它单凭自己还不够。

## 紧身衣问题

你的 AI 工厂里有一种张力，至今我没听到任何人承认过。那就是：治理有一笔真实、可度量、按笔交易计的成本，它随你所实现的控制措施的复杂度和频率而膨胀。每一道护栏都增加延迟，每一份审计日志都消耗存储，每一次人工升级都消耗某个人的时间。更糟的是，与主模型并行运行的可解释性计算，实际上会让那笔交易的推理成本翻倍；neuro-symbolic 推理检查同样增加处理开销。而你的基础设施账单会忠实反映你做的每一个决定，对你架构上的意图完全不予理会。

我以各种配置、加上层层递增的治理约束运行过 agent，现在大概能给这件事安上具体的数字了。一个增加运行时治理的策略引擎，对简单策略大约多花十毫秒，对复杂的多条件策略不到五十毫秒。顺序执行的护栏强制，会再加上三百到八百毫秒的延迟。为高风险决策实现可解释性，可能要跑计算昂贵的算法，让每一笔受治理交易的算力实际上*翻倍*。一个带自我批判的多 agent 评审循环，能把推理成本膨胀三到五倍。

> *单看延迟：十到五十毫秒的策略引擎，加上三百到八百毫秒的顺序护栏强制，现实的治理延迟开销就落在每笔交易三百一十到八百五十毫秒——这还没碰可解释性或多 agent 评审。*

问题在于，这些数字会复合叠加。一旦你不分风险高低、把它们一律压到每一个 agent 动作上，它们就是致命的。

*这就是我所说的紧身衣问题。*

如果你对高风险和低风险动作不加区别地施加治理，你的项目并不会因此安全分毫，反而比"干脆不治理"那个方案更慢、更贵。这句话我简直不敢相信自己得写出来，但现实就是如此。当完整的治理堆叠把成本推到基础推理成本的六到十倍，你不需要水晶球也能预见接下来会发生什么：你的项目会死于一张电子表格，而不是死于架构里某个致命缺陷。财务部门有人把治理开销和效率收益一比，发现这个比值越过了一，于是排了一场只有一张幻灯片的会。那张幻灯片上有一个数字，红色的，那个数字终结了你的项目。不需要尸检。

企业 agent 化治理的艺术，是*比例性*。低杀伤半径、后果可逆的象限 I 动作，只需要轻量级控制、快速日志和周期性评审。复杂度中等、涉及受监管数据的象限 II 动作，需要运行时护栏和带结构化审计轨迹的 human-on-the-loop 监控。而那些尽管你已尽力做流程选择、却仍然落进项目里的残余象限 III 工作，需要硬性的 human-in-the-loop 闸门、完整的推理来源，以及大量经得起监管检查的文档。

要校准这种比例性，你必须实时知道：你的 agent 实际在做什么、它们多频繁地贴着策略约束的边界运转、漂移正在哪里累积、横跨整个组合的每笔交易治理成本是什么样子。

> *简而言之，你需要一种系统化的方式，去生成关于你 agentic 项目的证据——严谨到足以经受外部审视，经济到不会把它本应保护的那份价值消耗殆尽。*

这种系统化的方式，就是我所说的证据工厂。

是的，这句话我本可以在引言里就说出来，但一个老师终究改不了啰嗦的毛病。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/05.webp)

## 于是我们发明了证据工厂

别把证据工厂和可观测性平台混为一谈。它也不是合规仪表盘或治理框架。遗憾的是，它还不是一个你能从供应商那里买到的产品——尽管好几家供应商会乐意卖给你它的零件，并让你误以为他们卖的是整套东西。

证据工厂是一种真正的运营能力。

> *它系统化地、经过经济校准地生产出这样一种证明：你的 agentic 项目在意图边界之内运转；偏离在变成事故之前就被检测并处理；审计轨迹完整到足以从第一性原理重建任何一个 agent 决策；而生成所有这些证明的成本，始终与被治理流程的风险成比例。*

没有它，你就不是在运行一个受治理的 agentic 项目——哪怕你装了一堆花里胡哨、价格不菲的可观测性、可追溯性，以及其他各种"能力性"玩意儿。

这个名字来自一个制造业类比，那是我思考大规模企业 agent 化时觉得最有用的类比。一座工厂系统化地产出成品：在确定的质量水平上，带着被记录在案的流程、可度量的良率，以及对缺陷的清晰问责。证据工厂把这套逻辑搬到了 agentic 项目的治理层。它在每个流程的风险画像所要求的质量水平上产出治理证据，带着被记录在案的仪表、可度量的覆盖率，以及对缺口的清晰问责。

把我引向这个概念的那个洞见，简单，而且略微让人尴尬。

我当时在评审一个大规模 agentic 部署的治理架构，意识到我们已经给系统装了大量仪表——有日志、追踪、指标、仪表盘——却回答不了这些问题：这个 agent 为什么做了某项任务？它在那个时刻遵循的策略框架是什么？它考虑过又否决了哪些替代方案？如果输入稍有不同，又会发生什么？

我们有观察却没有解释，有数据却没有证据。于是在那时，我们创造了证据工厂，作为弥合这道缺口的运营设计。

## 证据工厂到底做什么

证据工厂产出五类证据。每一类都对应治理风险的一个不同*维度*，以及它所生成证明的一个不同*受众*。

第一类叫推理来源（reasoning provenance）。一个受治理 agentic 项目里的每一个 agent 动作，都应该能追溯到一个意图、一个观察、一个推断。Agent Execution Record 这个数据结构，把这三个要素捕获为一等公民、可查询的字段，与标准动作日志并列。意图，是 agent 在那个决策点上想达成什么。观察，是它从环境中感知到的、与那个决策相关的东西。推断，是把观察连向动作的那条逻辑链。没有推理来源，你手里剩下的只是一份日志，而不是一个解释。一次过去要花三天的事故调查，现在三小时就能做完。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/06.webp)

第二类是行为漂移检测。将近 90% 的受测 agent，在运行约 30 步之后，都显示出对原始目标的可度量漂移。Microsoft 的研究人员上周得出了同样的结论，并发表在他们放上 ArXiv 的一篇论文里（链接在评论区）。

顺便说一句，漂移不是模型失败，它是*治理架构*失败。Agent 没有被约束在边界之内，于是它得以朝着从来不在原始授权之内的子目标去优化。证据工厂里的行为漂移检测这样工作：在初始部署期建立行为基线，然后对生产行为持续运行统计检验，在偏离累积成事故之前就把它们识别出来。技术上，它复用了捕获推理来源的那套可观测性基础设施，再加一个分析层，对基线运行比较函数。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/07.webp)

我们还需要一样东西，我管它叫信任评分（trust scoring）‡和风险调整后的监督校准。不是每个 agent 动作都需要同等级别的审视。信任评分给每个 agent 赋一个动态的信心水平，依据是它的历史准确率、策略合规记录、行为稳定性，以及它当前正在执行那些动作的杀伤半径。低风险动作上的高信任评分，触发轻量级控制。高风险动作上的低信任评分，触发立即的人工升级，不管 agent 自己怎么评估当前局面。这正是真正解决紧身衣问题的机制。它按比例施加治理：把监督集中到证据表明它真正需要的地方，在过往记录足以让人放心的地方放松它。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/08.webp)

第四类是策略边界映射（policy boundary mapping）。一个受治理项目里的每个 agent，都应该有一张明确、被记录在案的地图，标出它运转所在的策略边界。证据工厂里的策略边界映射，维护着每个 agent 的实时记录：被授权的工具集、数据访问权限、升级阈值、决策权限上限。它实时追踪 agent 与这些边界的接近程度，并在某个 agent 持续贴着权限边缘运转时生成警报。这种行为特征，意味着一个 agent 即将超出授权，或者已经找到了办法，通过间接的工具链来达成目标。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/09.webp)

我们建的最后一类是经济治理遥测（economic governance telemetry）†。正是它，让证据工厂不至于变成它本想解决的那个紧身衣问题。经济治理追踪每笔交易、每个 agent、每个流程的治理成本，并把它和受治理那项自动化所产生的效率价值做比较。当治理成本逼近或超过效率收益时，遥测会触发一次治理架构评审，而不是放任项目继续在那些不再有经济正当性的控制上烧钱。这就是那个让证据工厂能够自我纠正、而不是自我永续的反馈循环。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/10.webp)

*如果你想进一步了解证据工厂，下载评论区的白皮书。*

‡ *感谢* [Olivier Rikken](https://www.linkedin.com/in/olivierrikken/) *在信任评分上所做的工作。*

† 基于研究《Tokenomics for agentic AI and quality-per-token-metrics》《Roundtrip value governance》和《Patternomics: a formal theory of execution pattern optimization in enterprise agentic AI systems》，全都可在 Eigenvector/research 下载。

## 怎样建一座证据工厂而不至于破产

是的，成本。我至今没见过哪个运行 AI 项目的组织，把治理成本算进过预算——我是说，按比例算进去，并且管理起来。正因如此，证据工厂有一个五层成本栈，直接映射到它执行的治理功能。理解这个栈，是带着经济意识去建造这座工厂的前提。

第一层是编排成本，也就是你为了在整个 agent 舰队上协调证据收集活动所需要的基础设施。它基本是固定成本，随 agent 数量缓慢膨胀，但随你需要装仪表的那些多 agent 交互的复杂度迅速膨胀。这也是你不该从象限 III 起步的原因之一。对编排成本影响最大的架构决定，是你究竟给 agent 一个一个单独装仪表，还是在基础设施层部署一个治理 sidecar——它在不改动 agent 代码的前提下拦截 agent 动作。这种 sidecar 做法（像 Hoop .dev 的 AI Governance Sidecar Injection 就是一例）在规模上始终更经济，因为它把治理仪表和 agent 开发分开，让治理架构可以独立于 agent 舰队去演化。

第二层是感知成本，也就是收集证据工厂所处理的原始遥测数据的开支。这正是 OpenTelemetry 在这个栈里挣得一席之地的地方。面向生成式 AI 运营的 OpenTelemetry 标准，提供厂商中立的仪表，以及给 LLM span 的语义约定。这阻止了厂商锁定，让你证据工厂依赖的遥测数据能在不同后端之间可移植。GenAI Special Interest Group 在 agent、模型和向量数据库的通用语义约定上所做的工作，是眼下企业 AI 基础设施里最重要的标准化努力。它大体上在公开场合进行，几乎没有声张——而这恰恰是那种三年后会被证明极其重要的事情。

第三层是推理成本，也就是 neuro-symbolic 治理层（如果你选择加上它）以及产出推理来源记录的可解释性函数的计算开支。这是按笔交易计最贵的一层，也是最直接受益于风险调整后校准的一层。在每一个 agent 动作上都跑完整的符号推理验证，在经济上站不住脚；只在那些触发信任评分阈值、贴着策略边界运转、或涉及受监管数据的动作上跑它，则既有经济正当性，架构上也合理。

第四层是记忆成本。是的，它也算，只是程度轻一些。我说的是为这座工厂产出的证据所需的存储和检索基础设施。Agent Execution Record、行为基线、信任评分历史、策略边界地图、经济治理遥测，全都需要存成一种格式——支持审计或事故调查会用到的那些查询模式。这里的架构建议是：核心证据记录用不可变的仅追加（append-only）存储，它杜绝事后修改，满足受监管行业的审计完整性要求；再配一个可查询的分析层，让证据可被访问，而不必直接碰那个不可变存储。

顺便说一句，如果这一切对你像在念咒语——我完全能想象——那就从 Martin Kleppmann 的《Designing Data-Intensive Applications》开始读，那本书会让一切豁然开朗。在我读过的材料里，它是把这些问题讲得最清楚的：为什么会有不可变的仅追加存储、event sourcing 和基于日志的架构究竟做什么、以及为什么把写入存储和查询层分开是一个架构决定。

最后一项你要度量的是产出成本。它很容易度量，却是所有成本里最贵的，因为它是评审升级、并对工厂产出的证据采取行动所需的人类时间。大多数组织低估了这一层，因为它是唯一不出现在基础设施预算里的一层。

> *一座健康的证据工厂，运行着一个治理良好的 agentic 项目，应该为象限 II 流程里百分之二到五的 agent 动作产出人工升级。高于百分之五，说明信任评分阈值太保守，或者流程选择纳入了过多象限 III 的工作。低于百分之二，说明阈值太宽松，或者 agent 已经找到了监控捕获不到的运转方式。*

两者都是治理失败，只是类型不同。

## 企业玩具 vs 开源现实

唉，我聪明的朋友。没有任何一家供应商能把这一切都做了。我想在这点上说得直白些，因为企业软件市场有强烈的商业动机去暗示相反的情况，而这种暗示正让组织在平台采购上花掉大笔钱——这些平台只覆盖了他们所需的百分之四十，剩下的还得靠定制开发。

是的，这就是你想当一个早期采用者时要面对的丑陋真相。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/11.webp)

证据工厂各组件的商业供应商版图，广度上确实令人印象深刻，缺口上也确实令人沮丧。LangSmith 是 LangChain 的托管可观测性平台，每天处理超过十亿个事件，被大约 35% 的财富 500 强使用。它提供高保真的执行树、标注队列，以及 LLM-as-a-judge 评估器，把推理来源和行为漂移检测这两层覆盖得不错。Arize AX 及其开源对应物 Phoenix，用 OpenInference（一个基于 OpenTelemetry 的标准）做 agent 图可视化和实时的跨追踪分析。Galileo 的 Luna-2 评估模型，在幻觉检测上交付 95% 的 F1 准确率，其成本第一次让生产规模的评估在经济上变得可行。Zenity 提供自动化的 agent 发现——这一点极其重要，因为大多数组织里 60% 的 AI 活动都是影子 AI，运转在任何中心化可见性之外。它还通过自家所谓的 Zenity Attack Graph 来映射 agent 的风险依赖。

> *如果这套术语吓到了你，我觉得是时候去雇一位"治理工程师"了——我对这个角色的定义是：一个工程师/软件架构师，外加一份对治理的渴望。如果你认识一位，或者你就是一位，务必联系我 ;)。现在跳过这一章，去看下一章——关于指标的那一章。*

策略强制这一侧，Open Policy Agent 仍是现有最成熟的确定性策略引擎。它把业务规则转成可执行的 Rego 逻辑，对简单策略有低于十毫秒的评估延迟。Cedar（AWS 的开源策略语言）提供对策略安全属性的形式化验证，这是 OPA 原生不提供的。NVIDIA 的 NeMo Guardrails 也很有意思，因为它把基于模式的确定性规则，和给 neuro-symbolic 层的、基于 LLM 的语义检查结合起来。它用一种叫 Colang 的声明式语言，让治理规则对非工程师的人也可读——当你的合规团队需要核验规则是否反映了他们意图的策略时，这一点意义重大。

记忆和可追溯性这一层，我更偏爱 Langfuse。它提供开源的可观测性，带 prompt 管理和评估能力，对那些有数据主权要求（欧洲）、以至于云托管平台难以说得过去的组织来说尤其合适。MLflow 配上它的 Unity Catalog 集成，处理模型血缘和版本管理的方式，可以自然延伸到 agent 治理的语境。Temporal 提供带不可变事件历史的持久工作流执行，满足受监管行业的审计完整性要求，而不必自建定制存储架构。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/12.webp)

我很抱歉这样说，但我唯一能给你的实用建议是：拼一个可组合的栈，而不是采购一个平台。OpenTelemetry 做遥测标准化。OPA 或 Cedar 做策略强制。NeMo Guardrails 或一个 Constitutional AI 实现做 neuro-symbolic 层（再说一遍，如果你想要这层的话）。LangSmith 或 Langfuse 做可观测性，具体看你的数据主权要求。Temporal 做工作流持久性和审计轨迹完整性。经济治理遥测层则要自己造，建在你现有的 FinOps 基础设施之上——因为还没有哪个供应商把它造好过，而那些声称已经造好的，通常是在卖给你一个仪表盘，只给你看成本，却不把成本和治理价值连起来。

> *集成的复杂性是真实的。但替代方案——买一个把一切都覆盖得不充分的单一平台——更贵，而且产出更差的治理结果。*

那些运行着最有效证据工厂的组织，做出了"组合而非整合"的架构决定，把集成投入接受为正确做治理的代价，并自己造了经济治理遥测层——因为没有别人会替他们去造。

## 哪些指标能告诉你这一切是否在起作用

一座产不出可付诸行动指标的证据工厂，不过是个背后有个好故事的昂贵日志系统。真正重要的指标，是那些把治理活动连向治理结果、把治理成本连向治理价值的指标。

每个合规决策的成本（Cost per compliant decision）是证据工厂的首要经济指标。它度量治理的总开销——仪表、计算、存储和人工评审——再除以那些在策略边界之内完成、无需升级或补救的 agent 决策数量。随时间追踪它，你就能知道：治理架构是否随 agent 舰队成熟而变得更高效，还是治理成本正在比它的效率收益膨胀得更快。是的，这是个相当重要的指标。

> *目标轨迹是：在两到三个季度里，每个合规决策的成本下降百分之十五到三十——前提是信任评分趋于稳定、监督校准变得更精确。*

接着是"agent 蔓延指数"（agent sprawl index），它度量受治理 agent 与生产中 agent 总数的比值。在大多数企业里，这个数字介于令人尴尬……和令人警觉之间。

> *Gravitee 的 2026 年调查发现，只有 24.4% 的组织对"哪些 agent 正在彼此通信"拥有完整可见性。一半以上的 agent 在没有任何安全监督或日志的情况下运行。*

agent 蔓延指数给你一个单一数字，量化治理覆盖缺口，并追踪弥合它的进展。处于治理成熟度四级或五级的组织，其蔓延指数比一级组织低 94%。*那*，就是有证据工厂和没有证据工厂之间，可度量的差别。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/13.webp)

检测治理违规的平均时间（Mean time to detect），度量你的证据工厂多快能识别出一个在策略边界之外运转的 agent。生产级 agentic 治理的研究基准是：在违规发生的那个运营会话之内就检测到它，也就是在 agent 完成那项触发违规的任务之前。大多数组织目前是在事后尸检分析里才发现治理违规——那基本等同于火灾过后才装烟雾探测器。

人工升级率（Human escalation rate），度量需要人工评审或干预的 agent 动作所占的百分比。给象限 II 流程设的百分之二到五目标区间，就是治理比例性的运营定义。高于这个区间，你的治理架构正在把人类注意力消耗在那些信任评分本该自动处理的决策上。低于这个区间，要么阈值太宽松，要么监控漏掉了东西。这两种失败模式，在升级率里露出马脚，都早于它们在别处露出马脚。

> *能在三十秒内把一切都告诉你 CFO 的那一个数字，是治理-价值比（governance-to-value ratio）：治理总成本除以受治理流程产生的总效率价值。当这个比值超过一，治理花的钱就比自动化省下的钱还多——这意味着你的 agent 化项目对组织而言是一项净成本，不管那张转型幻灯片关于战略价值说了什么。*

追踪这个比值，是证据工厂最重要的经济功能，也是最有可能让你和财务总监聊出点意思的那个功能。

## 证据工厂还不是一个产品

没有哪个供应商把证据工厂当作一个完整、集成、生产就绪的系统来卖。好几家供应商卖它的零件，其中有些零件相当优秀，但要把它拼成整体，需要的始终是同一样东西：一个知道自己在造什么、已经决定要把它认认真真造好、并且愿意去做供应商生态尚未替他做的那些集成工作的人。

那个缺口当然会弥合。前沿实验室一直把太多注意力放在刷基准测试上，对企业客户的需求关注得太少——但我看到，市场就在我们说话的此刻把它接了过来。这个行业正以一种紧迫感朝它移动：它已经见过足够多的生产数据库被删除，从而明白了治理基础设施不是可有可无的开销。MLOps 市场正朝 2032 年两百亿美元的规模走，这在很大程度上就是一个押注那个缺口正在弥合的赌注。把 Protect AI 并入 Palo Alto Networks、CalypsoAI 并入 F5、Lakera 并入 Check Point、Truera 并入 Snowflake 的那一波收购，正是一个正在把自己组装成集成平台的市场的整合模式。一两年后，证据工厂大概会变成某种你可以用一份企业许可证加一次专业服务约定买到的东西。

查看内容凭证

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/14.webp)

但眼下，它是某种你得自己造的东西。用可组合的开源组件；在那些商业方案确实优于开源替代品的层上做有针对性的商业平台采购；自己定制经济治理遥测——因为还没有人把它造好过；再加上那份纪律：在栈的每一层都把治理成本压在治理价值之下。

那些能在接下来两年的审计压力和董事会层面问责中存活下来的 agent 化项目，是那些能打开一个仪表盘、把证据展示出来的项目。展示每个 agent 为什么做出每个决策的推理来源记录。展示 agent 正在意图边界之内运转的行为漂移报告。展示治理这个项目的成本与它产生的价值成比例的经济治理遥测。

那就是证据工厂产出的东西。那也正是"相信我，agent 没问题"拿不出来的东西。

我现在已经就这个主题生成了四十四张幻灯片。其中有些甚至是对的。

*就此搁笔，*

Marco

> Eigenvector 大规模地为那些真的必须有回报的生产环境构建 agent 化工厂，而 Eigenvector Research 偶尔发表论文，讲为什么这件事比演示所暗示的要难得多。

*👉 觉得某位朋友也会喜欢这篇？分享这份 newsletter，让他们加入这场对话。* LinkedIn、Google 以及那些 AI 引擎，会用让我的文章触达更多读者的方式，来感谢你的点赞。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/15.webp)

本文发布于 [Generative AI](https://generativeai.pub/)。在 [LinkedIn](https://www.linkedin.com/company/generative-ai-publication) 上与我们连接，并关注 [Zeniteq](https://www.zeniteq.com/)，紧跟最新的 AI 故事。

订阅我们的 [newsletter](https://www.generativeaipub.com/) 和 [YouTube](https://www.youtube.com/@generativeaipub) 频道，获取生成式 AI 的最新消息与动态。让我们一起塑造 AI 的未来！

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/ive-built-an-evidence-factory-because-trust-me-the-agent-is-fine-is-not-a-governance-strategy/16.webp)
