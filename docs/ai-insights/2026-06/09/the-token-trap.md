---
title: token 陷阱：AI 依赖的隐性成本
author: Shashank Sane
url: https://medium.com/@saneshashank/the-token-trap-d3dc6cbcb615
translated: 2026-06-09
excerpt: AI 依赖的隐性成本
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/01.thumb.webp
---

# token 陷阱：AI 依赖的隐性成本

AI 依赖的隐性成本

亚马逊、优步和微软刚刚领教了 AI 的真实成本，以及它的脆弱。

亚马逊推出了一个 token 使用排行榜，随后又不得不把它关停。优步炸光了年度预算。微软取消了数十万份许可。这一切，都发生在同样的两个月里。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/01.webp)

***附注****：本文是一个系列里的一篇，试图用经济学视角看懂这轮 AI 超级周期；若还没读，建议先看* [***The Inverted Stack***](https://medium.com/@saneshashank/the-inverted-stack-fc2ba67e2e77) *和* [***The Weak Link***](https://medium.com/@saneshashank/the-weak-link-86707de07851)*。*

要把这几件事串起来并不难。所有大科技巨头都在鼓动开发者大规模接入 AI 编码工具——给奖励、发排行榜、表扬用得起劲的人，还给它起了个名字叫“tokenmaxxing”，*源自 Z 世代俚语，意为把优化拉到极致*。

到 2026 年 3 月，借着《纽约时报》那篇“*More! More! More! Tech Workers Max Out Their A.I. Use.*”，这股风潮挤进了主流视野。

Meta 发布了内部“Claudeonomics”排行榜，给最热情的用户冠上“Token Legends”的头衔，据报道排名第一的工程师在 30 天内用掉了超过 2810 亿个 token。Y Combinator 的 Gary Tan 也公开盛赞这一做法，称把任务同时委派给 15 个 AI agent，能让编码产出提升 400 倍。

但 **tokenmaxxing** 起初并不是一个警示故事。是什么让它变了味？**账单到期了。**

## 到底发生了什么

有必要把细节摊开讲，因为这次反转的速度本身就很关键。

**亚马逊**的内部工具 Kiro 是一个 AI 优先的开发环境，它用一个名为 KiroRank 的排行榜，按工程师消耗的 AI token 量给予奖励。***本意是提高 AI 采用率，结果却让工程师纯粹为了刷排名，把非关键任务硬塞给 AI agent，白白烧掉算力，对产品却毫无改进。***

2026 年 5 月 29 日，亚马逊关停了 KiroRank。此前一位高级副总裁告知员工：“请不要为了用 AI 而用 AI。要用 AI 帮你解决客户的问题。” 此后亚马逊用“标准化部署”取代了 token 计数，衡量的是真正交付给用户的 AI 辅助代码，而非原始 token 用量。

**Uber 的情况**更为极端。他们 2025 年 12 月开始在工程师中铺开 Claude Code，到 3 月已有 84% 的员工在用。每位工程师的月成本一般在 150 到 250 美元，但重度用户每月能花到 2000 美元。光是 Uber 的 CTO 一人，在一次两小时的演示里就烧掉了价值 1200 美元的 token。到 4 月——也就是财年第四个月——Uber 就已用光了 2026 年全年的 Claude Code 预算。COO Andrew Macdonald 形容那一刻“脑袋都要炸了”。此后 Uber 给 AI 编码工具设了每人每月 1500 美元的上限。*耐人寻味的是，Macdonald 承认，* ***高 token 用量并没有带来面向用户的功能增加。***

**微软**的应对最具结构性。其负责 Windows、Microsoft 365、Teams、Outlook 和 Surface 的体验与设备（Experiences and Devices）部门被要求在 2026 年 6 月 30 日前停用 Claude Code。数千名工程师正在被迁移到 GitHub Copilot CLI。虽然微软声称此举是为了“工具链统一”，但选择在财年最后一天下达命令，表明成本控制才是主要因素。据报道，员工更倾向于使用 Claude Code 而非微软自家的 Copilot，这意味着此次强制迁移代表着微软为了节省企业级成本而无视了开发人员的个人偏好。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/02.webp)

> 三家不同的公司，一个共同的醒悟：***当 AI 工具的成本直接随用量上涨，而你又在不度量产出的情况下一味鼓励用量，你的预算就可能崩盘。***

而且它们不是孤例：**Meta 在 4 月一份报告外泄后，悄悄撤下了自己的“Claudeonomics”排行榜**；Duolingo 也在员工质疑“AI 用量与工作相关性”后，放弃了把 AI 使用情况纳入绩效考核的计划。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/03.webp)

## 为什么它在结构上与传统软件不同

**tokenmaxxing 的经济学**并不是企业采购里的一个漏洞，而是 AI 工具定价方式的结构性特征，***它打破了企业软件买家二十年来一直依赖的假设***。

**传统 SaaS 跑的是按席位计费**。每位用户每月付 X 美元，无论谁用得多猛，费用都可预测。销售代表一天打开仪表盘 20 次还是 2 次，Salesforce 收费一分不多。GitHub Copilot Enterprise 也是每席位每月 39 美元的固定价——没有使用量附加费。**预算事先就知道。**

**像 Claude Code 这样的 AI 编码工具则完全不同**。它有一笔基础席位费（每月 20 美元），但*真正的成本来自 API 的 token 消耗——这部分直接随工程师实际用量水涨船高*。工具越好用，就越贵。于是出现了一个以往任何一代企业软件都没有的悖论：***采用得越成功，越会把预算撑爆。***

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/04.webp)

**这是杰文斯悖论在企业工具上的翻版**。我曾从更广的 AI 经济角度写过它——2023 到 2025 年间，单 token 成本下降了 280 倍，而企业 AI 支出却翻了三倍。同样的动态也在单个公司内部上演。单 token 价格在快速下跌，但工程师从简单的代码补全转向会规划、执行、审查、迭代的 agentic 工作流后，每个任务消耗的 token 呈指数级增长。agentic 工作流每个任务消耗的 token，是标准聊天机器人问答的 5 到 30 倍。

> 一次 agentic 会话就能烧掉数十万 token。再乘以每天用这工具的 5000 名工程师，数字很快就失控了。

但除了用量，还有一个更深的结构性问题。斯坦福、伯克利、CMU 和微软研究院最近的一篇论文——《价格反转现象》（The Price Reversal Phenomenon）——指出，**标出来的 API 价格，根本无法可靠地代表实际成本**。他们让 8 个前沿推理模型跑了 12 项任务，发现在 32% 的模型两两对比中，标价*更低*的那个模型，实际总成本反而*更高*。*比如 Gemini 3 Flash 的标价比 GPT-5.4 便宜 80%，但它在所有任务上的实际成本却高出 38%，反转幅度最高达到 28 倍。*

**研究者把这套机制称为“过度思考”（overthinking）和“过度行动”（overacting）。** 在同一道 MMLU Pro 题目上，Gemini 3 Flash 消耗了 6 万多个思考 token，而 GPT-5.4 只用了 25 个。在一个安全任务中，一个模型用了 7 个交互回合，另一个为了同样的结果用了 57 个。**推理的啰嗦程度、agentic 交互的深度，这些差异彻底盖过了单 token 价格上的优势。更麻烦的是：在*同一*个模型上重复跑*同一*个查询，思考 token 的波动最高能到 9.7 倍。每次查询的成本不只是高——它本质上是随机的。**

> 即便你知道确切的 prompt、确切的模型，也没法预测一个给定工作负载会花多少钱。

这彻底改变了企业预算问题的性质。问题不只是工程师消耗了太多 token，而是在模型这一层，标价和实际成本之间的关系已经断了。*采购团队“因为更便宜”而选了 Gemini Flash 而非 GPT-5.4，结果可能反倒多花 38%。*

![图片来源：https://arxiv.org/html/2603.23971v2](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/05.webp)

> **一家公司若按单 token 标价做预算，等于是在拿一个跟实际账单关系薄弱、有时甚至相反的数字来做预算。**

Uber 的账算得很具体。拿 5000 名工程师，假设 84% 采用率，就是 4200 名活跃用户。按月均 200 美元算（还低于重度用户区间），那就是每月 84 万美元，一年 1000 万美元。但重度用户每月 2000 美元会迅速把有效均值往上拽，采用率还在加速时尤其如此。2 月份对 32% 采用率显得合理的预算，到 3 月 84% 采用率下，从结构上就根本撑不住了。

> 传统软件：成本随人头增长。**AI 工具：成本随*使用强度乘以人头*增长。**当你同时鼓励采用和使用强度，成本函数就变得非线性，而企业预算根本不是为应对这种情况而设计的。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/06.webp)

## 你正悄悄给技术栈埋下的薄弱环节

成本问题抢走所有注意力，这可以理解，但还有第二个风险，可能更凶险——它和我之前一篇文章里提出的“薄弱环节”框架完全对得上。

2026 年 3 月 3 日，Claude 宕机数小时，不到 24 小时后又一次宕机，工程师和开发者的反应很说明问题。在 LinkedIn 和面向工程师的论坛上，有人坦言自己已经好几个月没亲手写过代码了；一位写道：“***让我震惊的不是宕机本身，宕机难免。而是它多快就暴露出，我已经把 Claude 织进工作生活到了什么地步***。”

Downdetector 上记录了数千条宕机报告，而那些需要留存合规与审计记录的企业团队开始发愁：在降级运行的状态下，到底记下了什么数据、采集了什么数据，甚至有没有记下任何东西。

这些并非孤立事件；2026 年全年，Claude 多次宕机。OpenAI 这边，ChatGPT 在 2025 年 6 月 10 日经历了长达 15 小时的全球宕机。2025 年 11 月 10 日，Cloudflare 的一次重大故障让 ChatGPT 和 Sora 以及数千个依赖它们的应用一起瘫痪。2025 年 9 月，Claude 的一次 30 分钟宕机搞垮了其 API、开发者控制台和所有托管服务；即便正值工作高峰，也没有任何预警。

Thoughtworks 对 2026 年 6 月那次 Claude 宕机的一篇分析，相当精准地点出了问题的实质：“***过去，在 AI 革命这片尚显稚嫩的版图上，把某个特定供应商的 API 端点硬编码进应用，曾是可以接受的可用性做法。到了 2026 年，这就成了一个单一供应商的单点故障，对业务连续性构成重大风险。***”

作者指出，这股趋势已经溢出工程师群体：市场、财务、物流，乃至客服，整天都把 AI 工具用得顺手——*而一旦 API 挂掉，停摆的就不只是工程团队，市场、财务、物流、客服也一起停了*。

这是 Kremer 意义上的薄弱环节问题。一整套运转由一连串互补任务组成——编码、测试、审查、部署——你却在中间插了一个单一供应商的 API 依赖。API 正常时，链条完整、流转顺畅；API 一垮，整套运转就戛然而止——不是因为每一步都失败了，而是因为其中*一环*断了。

这个问题随时间复利累积，怎么强调都不过分。在 AI 辅助的工作流里浸泡数月后，工程师自身的写码能力会退化。代码库会沿着只受 AI 生成模式影响的方向长出来。“Claude 出现之前我们是怎么写代码的”这种组织记忆会一点点流失。*它始于增强，终于依赖——一次 4 小时的宕机，损失的不只是 4 小时，而是整支团队的认知，因为他们工作流上的肌肉记忆被抹掉了*。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/07.webp)

一个 25 人、每小时计费 90 美元的工程团队，光是 4 小时的宕机就可能损失 9000 多美元的有效工时。而对于像 Uber 这样 5000 人的公司、或微软体验部门这样一个事业部，“为 AI 依赖问题开一场全员上阵的董事会”就成了一个真实而紧迫的隐忧。

## 幽灵 GDP 问题

还有第二重、甚至更悖谬的经济影响，它在专业圈子里开始受到关注，但还没进入主流视野。
SemiAnalysis 有一份报告认为，AI 正在创造巨大的真实经济价值，而这些价值几乎在 GDP 统计中无迹可寻。他们称之为“暗产出”（dark output）。逻辑是这样的：当 AI 让完成一项知识任务（比如起草一份遗嘱）的成本骤降时，它对名义 GDP 的贡献也随之下降——尽管产出的数量和质量并没有下降。
他们给出的起草遗嘱成本轨迹很震撼：“17 世纪的羊皮纸抄写员，折合今天 3000 美元。文艺复兴时期的公证员收 800 美元。1900 年的律师收 400 美元。2010 年的 Legalzoom 收 150 美元。到 2026 年，一个前沿 AI 模型的 API 起草它，不到 0.5 美元。” 这是“16 年间 99.7% 的成本崩塌，最陡的一段就在最近三年。”

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/08.webp)

对 GDP 统计而言，问题在于：美国服务业 GDP 中有很大一块——约 41%、即 7.2 万亿美元——是用工资折算法（wage-imputation）来度量的。当一项任务从人来做变成 AI 来做，它的统计足迹就会缩水，因为 AI 的 token 成本只是工资成本的零头。产出一样好甚至更好，消费者拿到的价值一样高甚至更高。***但 GDP 的数值却下降了，因为它度量的是“付了多少钱”，而不是“产出了多少”。***
耐人寻味的是，这又绕回 tokenmaxxing 的话题：亚马逊和优步的 AI 支出正在爆炸式增长——这看似与幽灵 GDP 效应相悖。化解之道是：两句话都成立。

> *AI 让单个任务变得极其廉价，而公司层面被 AI 处理的任务量增长得如此之快，以至于即便单任务成本在暴跌，AI 上的总支出仍在爆炸式上涨。*

Uber 的每行代码成本并没有增加。Uber 生成的代码行数大幅增加，但正如 Macdonald 指出的那样，其中大部分并没有带来相应比例的实用功能增长。

这又是杰文斯悖论，只不过发生在企业内部，而非整个经济体。*所以，企业层面的成本问题和宏观层面的度量问题，是同一现象的两面：AI 在大幅压低单任务成本，同时在大幅推高 AI 总支出。*

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/09.webp)

## 围绕 token 依赖构建韧性

眼下——亚马逊关停排行榜、优步设下 1500 美元上限、微软取消许可——企业 AI 的采用确实在经历一次真实的修正。但这不会是最后一次。那么，对那些已经把 AI 嵌进现有流程的公司来说，怎样在不牺牲生产力的前提下管住结构性风险？
从已有的模式看，几条主线正在浮现：

**度量结果，而非活动量。** 最有力的信号，是亚马逊已经从谈论原始 token 数转向“标准化部署”——也就是真正发布上线的 AI 辅助代码行数。**重点不是你的工程师嚼掉了多少 token，而是有多少有用的代码行进了生产。** 研究结论相当明确：Jellyfish 考察了 7548 名工程师，发现 token 预算更高的人，以 10 倍的成本换来 2 倍的吞吐。GitClear 显示，频繁使用 AI 的工程师代码改动率（code churn）高出 9.4 倍——也就是说，很多 AI 产出的代码行刚写出来没多久就被丢弃了。Waydev 对一万多名工程师的分析显示，初版代码的接受率有 80%–90%，但经过几轮评审和修改后骤降到 10%–30%。***所有证据都表明，token 成本和最终交付的代码行数之间，相关性非常松散。***

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/10.webp)

**按非线性的采用曲线做预算。** 大家犯的都是同一个错：按线性采用做预算，等来的却是指数级采用。Uber 一个月内工程师采用率就从 32% 飙到 84%；线性预算连边都摸不着。但答案不是去限制采用，而是按“病毒式扩散”来做预算，并提前装好熔断器（支出上限、功能分级、高用量需要支出审批）。

**分散你的模型依赖。** 宕机数据在这里很说明问题：***今天把单一 AI 供应商焊进你的工程栈，无异于把生产数据库跑在一台没有故障切换的单机上***。**多模型架构**——根据可用性和成本，在 Claude、GPT、Gemini 或开源模型之间灵活路由工作负载——已经从“最佳实践”变成“入场门槛”。GitHub 基于用量的 AI Credits（2026 年 6 月起）甚至直接支持这一点，因为它允许你把 AI Credits 分配到不同模型上。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/11.webp)

**把增强和依赖区分开。** 这是最要紧的一条，也大概是最难落地的。工程师用 AI 工具加速写代码，和工程师离了工具就写不出代码，是两回事。前者是被增强，后者是被绑定。依赖会形成一个脆弱的环节——工具一倒，人也跟着倒。***公司应在关键工作流里保有一条人工基线能力，*** *并不时拉出来做压力测试：这个团队在 48 小时没有 AI 工具的情况下，还能不能发一个版本？* 如果不能，你就已经在工程组织里埋下了结构性的脆弱。

**盯紧定价模型的转轨。** 我们正经历定价模型的范式转移。Claude Code 按席位 + 按用量收费，GitHub Copilot 也将转向 AI Credits 的用量模式。趋势已定：纯席位费在大多数场景里出局，按用量计费在所有场景里进场。企业 AI 由此从一条稳定的 Opex（运营支出）项，变成一条随用量浮动的可变成本项——这正是云计算一直以来的玩法，对开发者工具领域却是新鲜事。那些早就练好云成本优化的企业，可以把现成的本事直接搬来管 AI 工具的开销。而那些一向把云当成“别人家的事”的公司，看到 AI 工具账单时，会再吃一次同样的惊吓。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/12.webp)

说句实在话：“tokenmaxxing”不只是工程师在排行榜上“钻空子”的写照。**它是 AI 工具定价方式和企业预算方式之间结构性错配的第一次显形。**定价问题是一个会卷土重来的真问题。而依赖问题——这道等式里脆弱的那一半——要阴险得多，被谈论得也远远不够。幽灵 GDP 则提醒我们：哪怕企业被飞涨的 AI 账单压得喘不过气，经济数字大概率还是低估了正在发生的财富创造。
那些做对了的企业，会像成熟的成功公司管理云服务一样去管理 AI 工具——盯结果导向的指标，带着良好的治理去消费工具，靠多供应商策略建冗余，并养出内部成熟度，分得清哪些工具是在增强人，哪些是在替代人。
那些做不对的，会继续撞上 Uber 那种局面——事实证明，四个月用光全年预算，正是“奖励活动而非结果”的必然结果。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@0c782f6fc26d50ed1d565562b277733bf1a273b0/ai-insights/2026-06/09/images/the-token-trap/13.webp)

如果你喜欢这篇文章，欢迎点赞 👏👏，并分享给更多人。

### 参考文献及延伸阅读：

1. 亚马逊 KiroRank 关闭 — [金融时报](https://www.ft.com/)（2026 年 5 月 29 日）；[HR 杂志](https://www.hcamag.com/us/specialization/hr-technology/amazon-shuts-down-ai-leaderboard-after-tokenmaxxing/577189)；[商业内幕](https://www.businessinsider.com/)（2026 年 5 月）
2. Uber 用光 Claude Code 预算——[雅虎财经 / The Verge](https://finance.yahoo.com/sectors/technology/articles/uber-coo-andrew-macdonald-says-130036457.html)（2026 年 5 月）；[AI Magazine](https://aimagazine.com/news/why-uber-has-already-burned-through-its-ai-budget)（2026 年 4 月）；[彭博社](https://www.bloomberg.com/)——每月 1500 美元上限（2026 年 6 月）
3. 微软取消 Claude Code 许可 — [The Verge](https://www.theverge.com/)（2026 年 5 月）；[TechRadar](https://www.techradar.com/)；[Windows Forum 分析](https://windowsforum.com/threads/microsoft-cancels-internal-claude-code-licenses-pushes-copilot-cli-by-2026.418482/)
4. Claude 2026 年的宕机事件——[HR Executive](https://hrexecutive.com/claude-outage-highlights-hrs-growing-ai-risks/)（2026 年 3 月）；[Thoughtworks](https://www.thoughtworks.com/en-us/insights/blog/generative-ai/claude-outage-june-2026)（2026 年 6 月）；[Cloud Wars](https://cloudwars.com/ai/cio-insight-business-continuity-lessons-from-anthropic-claude-outage/)（2026 年 3 月）
5. SemiAnalysis — [“AI Dark Output: The Visible Cost of Invisible Output”](https://newsletter.semianalysis.com/p/ai-dark-output-the-visible-cost-of)（2026 年 5 月）；幽灵 GDP 与 1.5 万亿美元隐形价值估算
6. GitHub 基于使用量的计费方式转型——AI Credits，自 2026 年 6 月 1 日起生效
7. 《纽约时报》——[“More! More! More! Tech Workers Max Out Their A.I. Use”](https://www.nytimes.com/)（2026 年 3 月 20 日）——tokenmaxxing 一词由此进入主流
8. Meta “Claudeonomics”排行榜 — [Medium / Adnan Masood](https://medium.com/@adnanmasood/tokenmaxxing-the-productivity-paradox-of-generative-ai-consumption-ddfe72cae8d5)（2026 年 4 月）；30 天内 2810 亿 token
9. Jellyfish——一项针对 7548 名工程师的研究：吞吐量 2 倍、token 成本 10 倍；GitClear——AI 用户代码改动率高出 9.4 倍；Waydev——对一万多名工程师的分析，初始接受率 80%–90%，修改后降至 10%–30%。来源：[EarlyTerms](https://earlyterms.com/term/tokenmaxxing)
10. Chen, Zhang, He, Stoica, Zaharia, Zou — [“价格反转现象：当更便宜的推理模型最终反而成本更高”](https://arxiv.org/html/2603.23971v2) (斯坦福大学、加州大学伯克利分校、卡内基梅隆大学、微软研究院 — arXiv，2026 年 5 月)
11. MS&E 435：人工智能超级周期的经济学——[课程网站](https://mse435.stanford.edu/)（斯坦福大学，2026 年春季）
12. FinOps 基金会 — [2026 年 FinOps 现状](https://data.finops.org/)
