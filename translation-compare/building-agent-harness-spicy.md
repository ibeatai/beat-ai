---
title: "打造一套生产级 Agent Harness：把 Claude Code 改造成多 agent 工程流水线"
author: Messi Li
url: https://licaomeng.medium.com/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline-1db4e242d08a
translated: 2026-06-03
tags:
  - LLM
  - AI Agent
---

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/01.jpg)

作者开篇就甩出一句很狠的比喻：单独一个 coding agent，不过是「泡在罐子里的大脑」。它能思考、能生成代码、能调函数——但它没法在凌晨三点回你的 Slack 私信，没法重跑挂掉的 CI，没法去修刚刚冒出来的 merge 冲突，也记不住「审查者昨天问的问题到现在还没答」。这话听上去像段子，可它精准点出了 demo 级 agent 和生产系统之间那道天堑。

让大脑真正有用的，是 harness——围绕 LLM 的运行时脚手架，给它装上感官、双手和记忆：事件接入、agent 编排、持久化状态、自愈循环、可观测性，再加一个给人类操作者的控制面。作者这套系统不打算做研究 demo，而是要做一个真正盯着某个 Slack 频道、对内部五个 repo 提 MR、隔夜处理审查者评论、CI 一红就悄悄自愈的生产系统。把这些词排在一起容易让人以为是 PPT，但他们摆出来的细节确实是被生产环境磨出来的。

他们这套东西是内部系统——一条搭在 [Claude Code](https://claude.ai/code) 之上的多 agent 流水线，已经连续运行了好几个月，从 Slack 接信号、派发真实工程任务。这篇文章是自顶向下的视角：它干什么、组件怎么咬合、以及那些塑造设计的「生产疤痕」。值得一提的是「连续运行数月」这个词——很多 agent 项目活不过一次 demo，能在真实负载下熬几个月，本身就是一种背书。

**本文目录：**

-   [为什么要 harness，而不只是一个 agent](#why-a-harness-not-just-an-agent)
-   [这套 harness 到底干什么](#what-our-harness-does)
-   [harness 必须填平的三道沟](#three-gaps-a-harness-must-close)
-   [第 1 层 — 事件接入](#layer-1--event-ingestion)
-   [第 2 层 — agent 编排](#layer-2--agent-orchestration)
    ↳ [自主 agent 循环细节](#the-autonomous-agent-loop-in-detail)
    ↳ [自我改进层](#the-self-improvement-layer)
-   [第 3 层 — 持久化状态](#layer-3--persistent-state)
-   [第 4 层 — 自愈循环](#layer-4--self-healing-loops)
-   [第 5 层 — 可观测性](#layer-5--observability)
-   [第 6 层 — 人在环路中的控制](#layer-6--human-in-the-loop-control)
-   [六层之外](#beyond-the-six-layers-the-pieces-that-make-it-production)
-   [端到端走查](#end-to-end-walkthrough-one-ticket-one-mr)
-   [与其他 harness 的对比](#comparison-to-other-harnesses)

## Why a Harness, Not Just an Agent

2023 年 ChatGPT plugins 出现时，很多团队都干了同一件「显而易见」的事：在 LLM 前面套个聊天 UI，配几个 function-call 工具，然后就管它叫「工程 agent」。作者说得很不留情：这套路在 demo 里几乎能用，在生产里从来没用过。第一周之内就会暴露三种失败模式：

1.  **操作者一关标签页，上下文就没了。** 记忆必须活得比单次会话更久。一个真实工程任务跨越好几天：建 JIRA、拉分支、起草、评审、处理评审意见、CI 通过、合并。没有任何一次 LLM 调用能扛住这种状态。
2.  **外部一变，它没法反应。** 审查者下午 4 点贴了条评论，第三次提交 CI 挂了，队友在 thread 里回复了。agent 必须被这些事件唤醒，而不是永远轮询、或者干等用户重新输入。
3.  **它没法从自己的失败里恢复。** agent 推了个 commit，CI 崩了，操作者第二天还得回来重新解释一遍失败原因。或者 agent 的 Okta（一个身份管理平台）会话半路过期，整条流水线悄无声息地死掉。这两件事每天都在发生，代价就是工程师晚上、周末被 page，去处理本该由 agent 搞定的活。

作者把账算得很清楚：这些问题正是 harness 层该解决的。harness 才是 `claude.ai` 聊天页和 Claude Code 本体、Cursor 的 background agents、Cognition 的 Devin，乃至他们这套系统之间的真正分界线。这里有一句话颇值得划重点——**大脑（模型）在这些系统里基本是可互换的，变的是包在它外面的 harness。** 这等于把行业里「模型为王」的叙事掀翻了一半：你换不换模型差别没那么大，你有没有把 harness 做扎实，差别是天和地。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/02.webp)

大脑只能通过 harness 跟世界打交道。换句话说，让 LLM 表现得像个系统的那些工程活，几乎全在 harness 里完成。模型只是泡在罐子里那颗脑子，真正干脏活累活的是外面这层脚手架。

## What Our Harness Does

他们从一个非常具体的问题起步：一个 Slack 频道——就叫它 `#support` 吧——每周大约堆积 30 张工单，每张几乎都要先做一次调查，然后（通常）落到五个 repo 之一里的一处代码改动。每张工单跨越数天，审查者会在 MR 上留评论，CI 会抽风，有时一番调查到头来发现是上个月某张工单的重复。扛这摊活的团队晚上周末被 page，而且活儿还没越做越少——同样形态的工单每个周期都卷土重来。这段描述任何被 on-call 折磨过的工程师都会心头一紧：它没有夸张，它就是日常。

这套 harness 有三项职责，按干预深度排序：

1.  **调查（Investigate）**——`#support` 里出现新工单时，顺着 thread 往下追，跨 repo 收集上下文，把结构化分析贴回同一个 thread。
2.  **修复（Fix）**——操作者批准后，开一个 MR（Merge Request）带上拟议改动，处理审查者评论，盯 CI，CI 崩了就修，一路推到合并。
3.  **自我改进（Self-improve）**——每个结案都会反哺：一旦出现反复出现的模式（比如同一区域的五张工单都动了同一份 config），就生成 gap 并对 harness 自身提出改动。

第三项才是把它和「花哨的工单机器人」区分开的东西。作者的措辞很到位：harness 的底料就是它自己的源码，而 LLM 通过走人类评审的 PR，对这份源码拥有写权限。这是整篇文章里最有野心的设定——一个会改自己的系统。能不能管得住它不把自己改坏，后文见分晓。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/03.webp)

六个逻辑层，下面逐一拆。

## Three Gaps a Harness Must Close

整套 harness 围绕一个洞见构建：**一个无状态、被动反应的 LLM 不是生产系统。** 六个层的存在，是为了填平裸 LLM 自己填不平的三道沟。这种「先讲清楚为什么要这么多层」的写法，比直接堆架构图厚道得多。

第一道沟是**反应性（reactivity）**：LLM 没法自己醒过来。第 1 层（事件接入）来填——Slack mention、GitLab CI 结果、PagerDuty page 全部汇入一个统一的派发队列。LLM 从不轮询；它被调用。

第二道沟是**持久性（persistence）**：LLM 在会话之间会遗忘，跨机器会丢上下文，而且分不清「我处理过那个」和「那个 commit 真的落到 origin 上了」。第 3 层（持久化状态）来填——进程内做去重、本地 JSON 存运营映射、git 同步的工作区存持久案例状态。跨机器会话连续性（后文细讲）是同一思路的延伸：状态活在 git 里，而不是某个进程里。「分不清『我处理过』和『真落地了』」这一句，戳中的正是 agent 系统最阴险的一类幻觉。

第三道沟是**质量（quality）**：没有结构约束的话，LLM 推理循环会从薄弱证据里写出自信满满的结论，会在没有穷尽可达信源时就宣布调查完成，会生成第一句话就「一股 AI 味」的 MR 描述。第 2 层（agent 编排）和调查循环来填这道沟——结构化输出契约、每轮迭代上的质量闸门、循环之后的对抗式评审，以及把「改代码 → CI → 合并」这个外层周期闭合的自愈循环。

把这些层粘合在一起的，是**复利属性（compounding property）**：每一个结案都让下一个更快。一次被批准的调查变成知识库条目；一个被批准的 MR 变成 gap 分析器可以引用的模式；一个最终化的案例对 harness 自身提出一个改进 MR。系统每过一个周期，就把自己处理得糟糕的那一类工单收窄一点。这个「复利」叙事是全文的灵魂，但也是最该被怀疑的地方——复利方向一旦反了，错误也会复利。作者后面确实没回避这点。

## Layer 1 — Event Ingestion

harness 需要被三种外部信号源唤醒：

-   **Slack 消息**——频道 mention、thread 回复、自我 DM 的 admin 命令。
-   **GitLab 活动**——MR 评审评论、CI pipeline 结果、pipeline 失败。
-   **PagerDuty 告警**——当某条 oncall page 引用了 harness 见过的工单时，把上下文浮现出来。

每种信号的延迟和可靠性特征都不同，所以他们给每种用了不同的入口。这一点很务实——很多人会偷懒搞一个统一接入然后被各家 API 的脾气教做人。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/04.webp)

这里有两件教科书描述不会告诉你的实战细节，也正是这一节最有价值的部分。

**最终一致性。** Slack 的 `conversations.history` 接口会滞后 1–30 秒。如果他们的 poller 看到空结果就傻乎乎地推进游标，那么任何 `ts` 落在这个窗口里的消息都会被永久跳过。他们的做法是**在空轮询时冻结游标**，从同一个最旧时间戳重试，直到至少有一条消息回来；跨重试的重复处理则由「按 message-ts 去重」的集合过滤掉。这是把「最终一致性」这个抽象名词，翻译成了一行实打实的工程纪律。

[**Socket Mode**](https://api.slack.com/apis/socket-mode) **+ poller，双保险。** Socket Mode 给 thread 回复带来亚秒级延迟，但 WebSocket 连接会断。poller 就是安全网——它用一个共享的 `socket-dedup` 文件兜住 Socket Mode 漏掉的任何东西，保证同一条消息不会被派发两次。没有这层，连接断开期间到达的消息会无声消失，而操作者根本收不到「漏了」的信号。注意他们对「无声失败」的执念——这是贯穿全文的安全观。

mr-monitor 的 cron 循环则是另一种性格。GitLab 的评审评论不会把事件推进流水线，只能去轮询。它最难的问题居然是**轮询本身的扩展性**：随着 harness 接手越来越多长期存活的 MR，它那 10 分钟的轮询循环会攒下一堆「死 thread 游标」，每个游标每周期都要烧掉一次 Slack API 调用。不治理的话，这些游标会制造持续的 429 限流风暴，把真正的轮询活活饿死。他们的解法是：当上游 API 返回 `thread_not_found` 时自动驱逐游标——下个周期的循环里它就消失了。修复前：每周期 12 次重试，持续数小时；修复后：归零。一个看似无关痛痒的「死游标」能演化成限流风暴，这正是「能 demo」和「能长跑」之间最典型的差距。

## Layer 2 — Agent Orchestration

这是 LLM 真正跑起来的地方。harness 把 worker 当作短命的 [systemd](https://systemd.io/) 单元来 spawn，每个 worker 携带一份 JSON payload，里面指定了一个 workflow + 一个 case\_dir + 一份 thread 上下文。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/05.webp)

每条 pipeline 都是 Claude Code（那颗大脑）的一次长跑调用，配上一份精心编排的 prompt 和一个可写的 `case_dir`。case\_dir 是 agent 的草稿空间：里面有该案例的 `TASK.md`、不断累积的 `followup_transcript_*.md` 文件、生成的 `gap_report*.md` 文件，以及中间 JSON 产物。

编排（choreography）很关键。作者强调他们**不是**用一个万能 agent 包办一切，而是有好几个专职 agent，各自在明确定义的交接点上串成链：

-   `**oncall_run**` 跑初始调查——最多 **20 轮推理迭代**、带质量闸门——结尾有一个显式的自我批判步骤。产出：一份 `ANALYSIS.md`（哪里出了错）和一份 `TASK.md`（该怎么做）。
-   `**case_followup**` 是长期存活的聊天 agent。操作者可以在一个案例 thread 上 DM 回复好几天；每次回复都用完整对话历史重新调起 case\_followup。关键在于，case\_followup 每 N 次回复还会把 `gap_analyzer` 作为子 agent 调起一次*（自我改进循环后文细讲）*——正是这个循环产出了那些指向 harness 自身代码的 `gap_report*.md` 产物。
-   `**finalize_case**` 是整合者。当一个案例闲置四小时、或操作者运行 `admin: finalize <case>` 时，它收集每一份 gap\_report，过滤掉误报，开**一个 MR** 带上幸存下来的 gap 修复。这个 MR 瞄准的是 harness 自己的代码库，而不是原工单的 repo。这就是自我改进循环的具象形态。
-   `**dev-agent**` 是干活的。它在每案独立的 [git worktree](https://git-scm.com/docs/git-worktree) 里运行，改代码、推送、盯 CI、CI 崩了就修、处理审查者评论。三种任务模式：`address_review`、`fix_ci` 和 `task`（最初的实现）。

关于这张编排图，有两点不那么显而易见，作者特意点出：

第一：**一切都发生在 worktree 里**，而非主 repo 检出。多个案例可以同时在飞，彼此不踩脚。poller / dev-agent 调度器在 payload 里把 worktree 路径传下去；LLM 永远看不到主检出。这是后文「worktree 不是实现细节」的伏笔，作者显然把它当成承重墙。

第二：**case\_dir 是单一事实来源。** `~/.harness/` 里的状态文件（下一节就见）把一个 Slack thread 映射到一个 case\_dir，agent 永远相对 case\_dir 来操作。这让系统其余部分简单得多：重命名 case\_dir，只要映射更新了，每个 workflow 都跟着走。这种「一切相对某个根」的设计，省下的不是代码而是后半夜的排障时间。

上面的编排图说的是*什么*在跑，下一节解释的是*怎么*跑——把一条 Slack 命令变成一次完成的调查，背后的机制。

## The Autonomous Agent Loop in Detail

「这玩意儿到底干啥」的一句话答案是：**操作者敲一条 Slack 命令，然后一个由迭代推理、自我批判和工具调用组成的闭环就不再需要任何干预地跑下去，直到要么活干完了，要么系统诚实地承认自己进行不下去了。** 作者自己也承认，这句话藏掉了几乎所有有意思的决定。这一节就是来拆这句话的。「诚实地承认进行不下去」是个很高的标准——大多数 agent 宁可编一个自信的错误结论，也不愿说「我卡住了」。

harness 内部跑着四种 agent 循环，每种都针对不同形状的问题做了调校：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/06.webp)

这些是形状。下面是实质——每个机制按它运行的顺序讲。上面那张图对应全部这些；读完之后图上的节点就讲得通了。*（闸门代号：* [*G1–G4、N1–N3、N5、A1–A3*](#quality-gates-the-structure-that-makes-confidence-mean-something) *·* [*D1–D15*](#adversarial-review-the-red-team-inside)*）*

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/07.webp)

## Before the Loop: The Pre-Commitment Lock

在第一次 Claude 调用开火之前，有两道保险先跑。第一道是结构性的。

**循环开始之前：预承诺锁（pre-commitment lock）。** on-call 调查 pipeline 做的第一件事——在 Claude 跑任何一轮调查迭代之前——是让 Claude 承诺一份 `hypothesis_slate`：对该问题至少六条候选解释，外加四到五条用于在它们之间取舍的评估标准。这些被冻结进 `kickoff_precommitment.json`，并在调查的余下过程中被当作不可变。

作者把这称为「系统里最重要、也最不显眼的方法论保险」，这个判断站得住。没有它，LLM 推理循环会朝着「早期证据最支持的那个假设」收敛，然后回溯性地把后续所有证据都框成对它的印证——这正是确认偏误的机器版。预承诺锁并不阻止收敛，它强制调查从一个完整的假设空间出发，并把「从这个空间走到结论」的路径记录下来。对抗式评审者（Phase B）检查的是：给定证据，这个结论是否就是初始假设清单本该预测出的那一个，而不只是结论自己内部自洽。把「不许马后炮」做成一个被冻结的 JSON 文件，这一手相当聪明——它不靠 prompt 里的「请客观」，而是用不可变状态把退路堵死。

**同样在调查循环开始前：经验锚（empirical anchor，Phase C）。** 仅在第一轮迭代时，harness 跑一组不涉及 LLM 的 OS 级命令——`journalctl` 错误尾巴、磁盘和内存快照、日志文件发现。输出作为原始证据注入第一份调查 prompt。这给了 Claude 一份在它形成任何假设之前的、事实性的系统状态快照。它是非阻塞的（出错就静默吞掉），耗时不到一秒；它的价值在于让 Claude 的第一步推理从地面真相出发，而不只是从问题描述出发。先喂事实再让模型猜，顺序对了，幻觉就少一截。

## Structured Output: The Completion Report Schema

每一轮迭代都以一份可解析的 JSON 契约收尾——`completion_report`——harness 用它来决定下一步发生什么。这个结构是固定的：

```json
{
  "status": "IN_PROGRESS" | "BLOCKED" | "COMPLETE",
  "confidence": 78,
  "open_questions": ["..."],
  "unchecked_sources": [{"name": "...", "access_status": "..."}],
  "contradiction_register": ["..."],
  "assumption_register": ["..."],
  "adjacent_problems": [{"summary": "...", "status": "...", "blocker": "..."}],
  "draft_response": "the user-facing Slack message"
}
```

这块 JSON 前后可以有自由格式的散文，但被解析出来的那份报告才是闸门评估的对象、才是状态文件持久化的内容。作者把这条点出来作为整套系统里最大的杠杆点：**用一点散文自由度，换一份可解析的契约。** 没有这个 schema，每个下游步骤都得从散文里重新抽事实；有了它，闸门就能作为确定性的 Python 跑起来。一句话戳破了无数 agent 项目的痛：让模型「随便说」很爽，但下游为了解析它的话能愁白头。

## Quality Gates: The Structure That Makes Confidence Mean Something

如果 Claude 可以不给任何支撑就写下 `confidence: 95`，那这个置信度数字就只是营销噪声。harness 通过一组**质量闸门（quality gates）**强制一份结构契约——这些闸门在每一轮迭代的输出上运行，除非证据和置信度对得上，否则不放行。这句话本身就是对一切「模型说自己 95% 确定」的当头棒喝。

每次 Claude 出招后，有三族闸门运行：

-   **G 族（逻辑一致性）** 强制推理不在跨轮之间自相矛盾。最清楚的例子：如果这一轮 `open_questions` 变多了但 `confidence` 反而上升，G1 就触发——分母扩大就使任何向上的移动失效。这族还检查：散文里的「张力措辞」是否登记为一条矛盾，以及相邻问题是否用有效的 status 对象（而非占位字符串）来追踪。共四道闸门。
-   **N 族（结构完整性）** 检查调查产物是否完整成形、而非半成品。N1：每个可执行的修复动作必须配一个 `verify_action`，带上描述、预期结果和时机。提一个修复却不说明你会怎么验证它，被当作未完成的工作。共三道闸门。
-   **A 族（断言上限）** 施加 Claude 没法靠散文绕过的机械限制。A1 算一个硬置信度天花板：`1.0 − (open_questions × 0.08) − (unchecked_sources × 0.05)`——不管 Claude 写什么，闸门都用算术把它压下去。其他 A 闸门会在相邻问题仍未解决时拦住 `COMPLETE` 状态、要求假设登记册非空、并强制至少一个拟议动作直接映射到根因。共七道闸门。

还有两条额外的抽查：**EQ1** 要求根因节点要么引用硬证据（非 INFERRED），要么接受 60% 的封顶。**P7** 每有一个对抗维度被标 FAIL，就把天花板降 7%，下限 40%。

整个闸门框架包含 19 个函数。G 族盯*Claude 写了什么*，N 族盯*Claude 漏了什么*，A 族盯*Claude 声称了什么*。这一句总结得相当漂亮——它把「怎么管住一个爱吹牛的 LLM」拆成了三个可执行的维度。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/08.webp)

闸门违规会变成 `**pending_guard_notes**`，被前置到*下一轮*迭代 prompt 的 `## SYSTEM QUALITY GUARD NOTES (from previous iteration)` 之下。指令很明确：继续之前先处理掉所有 note。这就是那个强制机制——闸门不去拦 Claude，它们把 Claude 上一轮的违规变成它下一轮读到的第一样东西。作者补了一句很冷峻的话：模型有多礼貌无关紧要，真正推动调查前进的是结构性强制。这等于直接否定了「靠 prompt 礼貌请求模型自律」的天真路线。

此外还有一个**上下文钉死（context-pinning）**步骤（A7），在每次 prompt 构建结尾运行，与是否有违规无关：当前报告里未解决的矛盾和未验证的假设，会被逐字重新贴到下一份 prompt 底部。这是为了防那个被反复记录过的「注意力衰减」毛病——模型在第 3 轮承认了一个复杂情况，到第 7 轮又悄悄把它丢了。给模型的记忆「钉钉子」，这是把已知缺陷当工程约束来对待，而不是指望下一代模型自己变好。

## Confidence as a Concrete Number

最重要的设计决定是：**agent 在每一轮迭代的结构化输出里给自己打一个** `**confidence: <int>**`，0–100 分制。这个数字不是摆设——它真的在闸控真实行为：

-   `**< 70**`：不能退出调查循环。流水线强制最多再跑三轮，注入一条 guard note：「你说你完事了，但置信度在线以下」。
-   `**≥ 70**` **并且** `**status: COMPLETE**` **并且没有** `**open_questions**`：闸门打开，进入**对抗式评审**阶段（红队/蓝队）。
-   `**≥ 95**`：闸门打开，允许 agent **自动执行非破坏性动作**（Jira 评论、状态流转、工单分配）。低于 95，这些动作就只作为草稿建议留给操作者。
-   **连续两轮跌幅 > 10%**：循环以 `degrading` 为由退出——agent 在倒退，停下来告诉操作者。

每个案例的完整置信度轨迹会作为一条 sparkline 写进 `ANALYSIS.md`：`45% → 62% → 78% → 80%`。操作者就靠这条 sparkline 来分诊「要不要信这个结果」。它同时也是 harness 自身自我改进循环在判断「哪些过往案例值得换个 prompt 重跑」时读的东西。把置信度从「一个口头数字」升级成「一个被算术封顶、又驱动真实分支的状态量」，这是这套系统区别于普通 agent 最硬的一块。

## Three Force-Continue Mechanisms

上面那张图里有三条分支：退出检查触发了 `ready_for_adversarial` 或 `blocked`，但循环并没有退出。每条对应一个强制继续条件——系统在这些情形里覆盖退出信号，转而注入一条 guard note：

-   **CONFIDENCE\_FORCE**：`status=COMPLETE` + `open_questions=[]` 但 `confidence < 70`——模型嘴上说完事了，却用数字悄悄标了不确定。强制最多再跑 3 轮（这些轮数算在 20 轮预算之内，不是额外加在上面）。
-   **CAUSAL\_CHAIN\_FORCE**：`status=COMPLETE` + `confidence ≥ 70` 但 `causal_chain_complete != true`——一个症状层的发现，却没追到根因。强制最多再跑 3 轮（同样算在 20 轮预算之内，不额外叠加）。
-   **G\_BLOCKER\_FORCE**：`status=BLOCKED` 但可达的技术信源被跳过、或被「靠推断」标成不可用而非真出了错。调查不能在没有真正穷尽它能够到的东西之前就以 blocked 退出。最多 3 次续跑（同样算在 20 轮预算之内，不额外叠加）。

作者总结说，这三条加在一起防住了最常见的三种过早退出：听起来很自信的不确定、症状层的「完成」、以及幻觉出来的「不可用」。第三条尤其辣——它等于直接假设模型会偷懒撒谎说「我够不着」，然后用机制逼它去够。这种「默认不信任 agent 的自述」的姿态，是整篇文章最一以贯之的工程哲学。

## Adversarial Review: The Red Team Inside

一旦 on-call 调查 pipeline 以置信度 ≥ 70 宣布自己完成，控制权就交给**另一次扮演对抗式评审者角色的 Claude 调用**。这次评审内部发生什么，有一棵决策树。

评审者看不到和调查者相同的 prompt——它拿到的是分析文本，外加一份明确的 15 个评估维度清单（D1 到 D15：假设有效性、证据链、因果推理、备择假设等等）。它产出一份结构化裁决：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/09.webp)

对抗式评审有三个退出条件：

1.  **发现关键问题** → 关键 note 作为 `pending_guard_notes` 注入*下一轮*调查迭代，循环重启。调查者现在拿到了明确的「评审者认为你漏了 X」的反馈。
2.  **只有次要问题、第一次过** → 带着次要批评再回环一轮。
3.  **连续两次只有次要问题、或一次干净通过** → 退出 `complete`，调查发布。

把对抗式评审做成**一次用不同 prompt 的独立 Claude 调用**，是个刻意的选择。同一个模型扮演不同角色，能抓住单 prompt 迭代会漏掉的相当一部分自我吹捧。作者给了个有分量的实例：在同一个案例上，他们见过置信度从调查阶段的 90 被对抗式评审压到 65，评审者精确地指向了调查者一笔带过的那个假设。这个「90 → 65」不是修辞，是能复现的现象——它给「让模型自我批判」这件常被质疑的事提供了真实证据。

还有第二个、结构上完全不同的评审与它并行：**红队评审（G5）**。对抗式评审者读的是完整调查叙事，而红队评审者只看两样东西：原始问题陈述和最终结论。没有中间推理、没有证据轨迹、没有此前各轮。它独立产出一个假设，并标出「调查发现的东西」与「一个冷读者本会预测的东西」之间的任何缺口。

其目的是认知隔离，而非覆盖率。对抗式评审检查调查是否结构完整（D1-D15）；红队检查的是：不靠调查自己的框定，这个结论是否仍然可达——推理是真扎根于问题，还是调查锚定在了某个早期发现上、结论顺着这个锚而非顺着证据长出来。两者都作为独立 Claude 调用运行；两者都必须通过，案例才标记 `COMPLETE`。用「只给问题和结论、掐掉全部中间过程」来做隔离，这一招比单纯多跑一遍评审高明——它专治「锚定偏误」，而锚定偏误恰恰是 agent 调查里最隐蔽的病。

## Five Honest Exit Reasons

调查循环可以因五种原因结束，到底是哪一种被记录下来并展示给操作者：

```
exit=complete       confidence=89%  iterations=12  elapsed=13min
exit=blocked        confidence=62%  iterations=4   elapsed=4min    (cannot proceed without info)
exit=stalled        confidence=58%  iterations=7   elapsed=22min   (two rounds, zero new facts)
exit=degrading      confidence=42%  iterations=5   elapsed=8min    (dropped >15% twice)
exit=timeout        confidence=73%  iterations=18  elapsed=50min
```

这其中，`blocked` 是最可供操作者行动的：agent 已经在 `unchecked_sources` 或 `adjacent_problems` 里登记了明确的阻塞点，需要人类判断来解锁。其余四种是技术性失败模式，harness 的处理方式是把它们浮现出来，而不是盲目重试。这一节标题里那个「honest」用得很到位——大多数系统的退出原因只有「成功」和「失败」，而这里把「停滞」「退化」「超时」分得清清楚楚，等于承认 agent 会以多种方式失败，并拒绝用一句「出错了」糊弄操作者。

## Tool Access: What Claude Can Reach From Inside

工具边界在这里很重要，因为置信度闸门部分依赖于 Claude 实际能够到什么——一个说「检查 unchecked\_sources」的闸门，只有在「检查它们的工具确实可用」时才有意义。每条 pipeline 用 `--permission-mode bypassPermissions` 启动一个 Claude Code 子进程。prompt 里**没有内联的工具白名单**；边界由 [MCP（Model Context Protocol）](https://modelcontextprotocol.io/) catalog 里实际接线了什么来设定。这个 catalog 覆盖五个外部系统：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/10.webp)

有几条约束值得专门点出，因为它们对行为的塑造比 model card 还大：

-   `**Bash**` **在调查 pipeline 里被显式禁用。** 调查循环以无 Bash 访问（`--disallowedTools Bash`）运行 Claude，以防 git 竞态——两个 Claude 进程并发修改同一个工作树会把状态搞坏。Bash 在 dev-agent pipeline 里可用，因为它跑在隔离的 worktree 里。
-   **Slack 写工具被屏蔽。** `send_message_to_self` 和 `edit_message_to_self` 在每一次 pipeline 调用里都被禁用。Claude 不能直接往 Slack 发；所有对外通信都走结构化完成报告里的 `draft_response` 字段，由 harness 在校验过退出条件之后再发。这意味着模型永远不能直接「开口」，它只能把话写进一个字段，等系统替它说——这是把「人在环路」做进了工具层。
-   `**bypassPermissions**` **在实践中 ≠ 不受限。** 每条 pipeline 的框定 prompt 编码了硬规则：`ANALYSIS.md` 可编辑但 `message.json` 不可变；run 之后通过 git diff 自动还原 `case_dir` 之外的写入；PROD DB 写入必须按命令逐次等待操作者显式确认。安全是在框定层 + 一道 run 后边界检查上做的，而不是在 tool-call 层。
-   **级联回退编码在 prompt 里，而非硬编码。** 当 `search_datadog_services` 返回 permission-denied 时，prompt 指示 agent 转去用 `mcp__glean_default__search`、拿 Jira 关键词当代理。这条回退链是 SOP（标准作业程序）被执行的方式的一部分，而不是运行时的一部分。
-   **只读工具并行跑，写工具串行跑。** 流式模式子进程在 `tool_use` 事件发生时就解析它们，对读操作（Datadog 查询、Glean 搜索、文件读取）并发派发，最多 10 个一批。任何写操作——带副作用的 Bash、编辑、MR 评论、DB 写入——一次只跑一个。这让单轮调查迭代能对日志并行做六次查询，同时仍维持写入顺序。读并行、写串行这条线划得干净利落——既要速度又要不踩状态，这是被并发坑过的人才会写下的规矩。

## The Same Pattern, Four Sizes

harness 里每条 pipeline 都用同一套骨架的某种变体：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/11.webp)

on-call 调查 pipeline 跑这套骨架最多 20 轮。task analyze 阶段最多跑 8 轮。implement 阶段跑一次，但把*build* 步骤循环三次。dev-agent 每次 spawn 跑一次、没有内层循环，但 mr-monitor 实际上把迭代外化了——每次 CI 失败就 spawn 一个全新的 dev-agent，连续最多三次，之后升级给人类。

这种一致性是有意为之。四个循环之所以不同，是因为它们解决不同的问题，但它们共享同一套词汇：置信度是一个数字、退出是一个有名字的原因、自我批判是一次独立的 Claude 调用、升级是发进原始 thread 的一条 Slack DM。操作者学会一种形状，就能在到处认出它。这条「四种尺寸、同一套词汇」的设计，省下的是操作者的认知负担——对一个要长期被人盯着用的系统，这比多写几个功能更值钱。

## The Self-Improvement Layer

这里闭合了前面引入的那个循环：harness 修改自己的底料，由人类评审。有两个循环在做这件事，运行在不同时间尺度上。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/12.webp)

**循环 1 — 每案结构性修复（gap → PR）。** 每次调查结案后，`gap_analyzer` 找出 harness 表现欠佳之处：跳过的数据源、选错的工具、流水线 bug、缺失的测试、MR 描述里的格式失败。gap 按类别和置信度过滤，再传给 `finalize_case`。案例闲置四小时后，`finalize_case` 在一个专用 worktree 里 spawn `patch_suggester`。结果是对 harness 自己代码库的一个 PR——改代码文件、prompt 模板或 SOP 章节。这个 PR 需要人类合并，但不需要人类去写。「不需要人类去写、但需要人类合并」这一句，是整个自改进设定的安全阀所在——它把生成权交给机器，把决策权留给人。

**循环 2 — 跨案行为强化（failure mode → SOP 建议）。** 每个案例的 `auto_retrospective` 步骤会登记哪些失败模式触发了——按类型分类：数据收集失败（调查是否跳过了可达信源？）、质量闸门违规（置信度声明是否与证据矛盾？）、以及分析完整性失败（根因是否一路追到底了？）。这些按项目累积进案例注册表。一个每周 cron（`knowledge-updater`）统计最近十个案例里各模式的频率，当某个模式越过阈值时生成 SOP 建议：10 次里 3 次触发一条建议，6 次触发一条告警。这些建议很具体：*「『跳过直接 DB 查询』本月触发了 3 次——SOP 已更新：在推断 schema 之前，总是先跑直接的表查询。」* 由人类评审并应用。这个循环刻意不自动应用——在自修改系统里，会复利错误的自动决策是一类已知的失败。这条克制堪称全文最清醒的一笔：它清楚地承认「让系统自动改自己最危险的部分」会把错误也复利掉。

**为什么这让 harness 变得个性化。** SOP 是 harness 对「你的团队怎么做调查」的建模。它一开始很通用。随着案例累积、失败模式重复，kb-updater 逐渐建立起这个项目特有失败模式的画像。20–30 个案例之后，一个项目的 SOP 看上去和通用模板完全不一样——它反映了这个代码库里那些总被跳过的特定数据源、总被误用的工具、总是打偏的调查框定。每个项目有自己的 SOP。A 团队的失败教会的是 A 团队的 SOP，不会污染 B 团队的。把「踩过的坑」沉淀成一份会随项目演化的活文档，这比任何「记忆」噱头都更接近真正的组织学习。

这正是让学习变得持久的原因。一个上下文窗口只能活过一次会话；一个被合并的代码改动是有版本、被评审、被测试且永久的；一次 SOP 更新是经操作者评审、并被带进每一次未来调查的。底料在改进；下一个案例跑在一个比上一个更好的 harness 上。这一段把「context window 活一次会话 vs 合并的代码永久存在」对照着写，是对「靠超长上下文当记忆」这条流行路线的有力反驳。

## Layer 3 — Persistent State

agent 编排是按事件跑的东西；持久化状态是进程重启、机器重启、或操作者从笔记本切到 CVM 时还活着的东西。harness 的状态活在三层里，特征截然不同：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/13.webp)

**内存缓存**又小又可重建。它们活在 poller / keepalive 进程里，重启时从本地状态重建。

**本地状态**（`~/.harness/*.json`）是运营地图。关键文件有：

-   `task-thread-map.json` —— Slack thread\_ts → JIRA id + 绝对 `task_dir` 路径。
-   `oncall_thread_map.json` —— 同样形状，但用于 on-call 调查案例。
-   `monitored-mrs.json` —— harness 开过的 MR 列表 + 它们的 CI 状态。
-   `patch-pr-state.json` —— 每案 finalize 状态（`merged`、`no_gaps`、`failed`……）。
-   `oncall-state.json` —— kill switch 标志（第 6 层细讲）。
-   `socket-dedup` —— Socket Mode 处理过的近期消息 ts。这一层是机器本地的、不共享；重启安全，但机器一死，这层就没了。

**Git 同步状态**是持久层。案例工作区（`case-workspaces/<ticket-id>/`）在每次有意思的状态变更时 commit 并 push 到一个私有 git repo。第二台机器克隆同一个 repo 就能恢复同一段对话上下文。这就是他们拿到跨机器会话连续性的方式——操作者可以从笔记本或远程开发 VM 跟 harness 聊天而不丢上下文。

他们还给 git 同步状态加了第三种角色：**记忆固化（memory consolidation）。** 长对话会攒下几十个 `followup_transcript_*.md` 文件。活动到一定程度后，一个子 agent 把它们全读一遍、重写 `TASK.md` 把蒸馏后的状态纳入，之后 workflow 就不再把单个 transcript 喂给后续轮次。没有这步，prompt 会涨过模型的上下文窗口。**有了**这步，又真有过度压缩、丢信息的实在风险——而且他们真丢过一次，把一份 14KB 的文档压成了 587B。正是那次事故催生了现在的三层防护：一个「UPDATE not REWRITE」的 prompt 模板、一个拒绝「写入小于输入 50%」的输出尺寸守卫，以及一个防并发固化跑互相竞争的每案锁文件。14KB 压成 587B 这个数字本身就是最好的辣评——它诚实地暴露了「让模型自己压缩记忆」会出什么洋相，也解释了为什么后面要堆三层保险。

从单个 Slack thread 到案例 `TASK.md` 的查找图虽小，但值得一看：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/14.webp)

两个状态文件，同样的 value 形状：一个指向案例工作区的绝对路径。每个 workflow 最终都解析到一个 `case_dir`，然后相对它操作。重命名工作区根目录——他们项目中途真干过一次——只是一个迁移脚本的事，它和目录移动同步地把两张映射里的绝对路径一起重写。

持久化状态是地基。当建在这地基之上的操作失败时会怎样？作者抛出一句很有态度的话：harness 不把失败浮现给操作者——它从失败里恢复。这句话顺势把读者推进了下一层。

## Layer 4 — Self-Healing Loops

这是 harness 从「通知器」变成「自主体」的地方。三个循环，每个都在没有操作者干预的情况下闭合。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/15.webp)

前两个是完全自主的。第三个刻意不是——OKTA 刷新需要一个硬件绑定的 MFA 提示，所以 agent 把自己停下来，在 DM 里附上一份 30 秒的 runbook 请求人类操作。这是设计上的人在环路，而非能力上的缺口。作者特意区分这两者，是为了堵住「你这不是没做到吗」的质疑——有些事不是做不到，是不该让机器单独做。

这些循环有一个微妙属性：**重试预算与自我识别。** CI 自动修复有每 MR 的上限（连续 3 次失败 → 升级给操作者）。审查者自动回复用**作者感知去重**——如果 bot 在一段讨论里的上一条 note 比审查者的更新，循环就不再开火（bot 已经回过了）。OKTA 助手有一个 15 分钟的批准窗口。没有这些，循环偶尔会进入螺旋——项目早期，一个带 flaky 集成测试的 MR 在一个下午里跑出了 80 个空操作的「fix CI」commit，他们才逮住。这个「80 个空 commit」是全文最有画面感的疤痕——它把「自动化一旦失控会有多滑稽」具象成了一个数字，也解释了为什么每个自愈循环都得配一个重试预算。

## Three-Layer Self-Heal: When the Deterministic Rule Isn’t Enough

上面那两个 CI 自动修复和审查者回复循环是端到端的恢复流。在这些流之下，每个可能失败的原子操作（push、rebase、MR 评论、branch reset）都坐在另一套更小的 3 层架构上，做**每操作的自恢复**。作者特意提醒：这与第 6 层里的 3 层**安全**模型是不同的轴——那一套讲的是克制（agent 被允许做什么），这一套讲的是恢复（一个被允许的操作失败时怎么办）。容易混，值得分清。这种主动澄清「别把我两套三层搞混」的写法，说明他自己也知道命名碰车容易让读者犯晕。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/16.webp)

**L1 — 确定性守卫。** 亚秒级、基于规则、永远在线。对 push 而言就是 `_pre_push_rebase(repo_path, branch)`：每次 harness 即将 push，先 `git fetch origin` 再 `git rebase origin/<branch>`。rebase 干净就继续 push；有冲突或 fetch 失败，L1 就干净地中止、枚举冲突文件、交棒给 L2。L1 不能做任何有创造性的事——这正是重点。规则小到能推理得清，且有界到可以在每次操作上跑而不花 token 成本。

**L2 — 代理式自愈循环。** 当 L1 搞不定时，harness spawn 一个受约束的 Claude 会话（`_self_heal_push_loop`），喂它完整的失败上下文：git 状态、冲突文件、促成这次 push 的原始任务，以及一组收得很紧的红线（绝不对受保护分支 force-push、绝不丢未推送的 commit、绝不盲选一边冲突）。三条约束让它安全：

1.  **有界重试。** 每个事件 ≤3 次尝试，每个 Claude turn 4 分钟超时。每次 Claude 动作之后 harness 重跑真正的 `git push`——Claude 自认为冲突是否解决从不被信任；`git push` exit 0 才是地面真相。Claude 想喊「修好了」就喊，但只要下一次 push 返回非零，循环就把那次记成失败尝试。
2.  **置信度闸门 ≥ 70。** 逐字借自 on-call 调查 pipeline 的 `oncall_run` 配置：Claude 每个 turn 自评 0–100，低于 70 就必须放弃而不是动手。低于下限的自我报告碰不到工作树。
3.  **真实性不变量被保住。** L2 成功意味着一个真实 commit 落到了 origin。下游回复闸门（给审查者那条「Done — updated」的回复）仍然硬接在 `git push` exit 0 上，而不接在 Claude 关于自身成功的任何说法上。L2 不能绕过回复闸门，它只能喂给它一次真实的 push。

**L3 — 操作者升级。** L2 放弃了（低置信度）或耗尽了尝试 → harness 把**完整的 L2 轨迹**DM 给操作者：每一次尝试、Claude 采取的每一个动作、每一个置信度评分、最终的 `git status`。操作者从一个被充分埋点的状态接手，而不是从「出了点问题」接手。时间尺度：小时级，不是秒级。

两条值得拎出来的原则：

-   **L2 是每事件的，不是永远在线的。** L1 在每次 push 上跑是因为它便宜；L2 烧 token，所以只在 L1 已经失败时才开火——成本被罕见情形闸住，而非被常见情形闸住。
-   **真相在线上（Ground truth on the wire）。** 不管 Claude 怎么相信自己的工作，校验都重跑那个真实操作。对 push 是 `git push` 退出码；对测试修复是跑真正的测试；对 MR 回复是回复闸门检查 push 退出。agent 的自我报告永远不是地面真相——harness 总是对着线上重新核验。

作者说这个模式可以推广到 push 之外：任何 harness 操作，只要有一条清晰的确定性恢复（L1）、一个受约束的代理式恢复（L2）、和一个干净的升级面（L3），就能用同样的方式包起来。正是这个形状，区分了「能从已知失败里恢复的自主循环」和「世界一变就要操作者上场的循环」。把「永不信任 agent 自述、永远对线上重核」做成一条贯穿全系统的不变量，这是这套设计里最硬核、也最值得抄走的一条。

## Layer 5 — Observability

harness 往四个可观测性面写数据，每个面向不同消费者调校：

-   **实时 Slack 更新**——一个 agent 在跑时，案例 thread 里一条占位消息会循环过一串状态消息（`Reading TASK.md` → `Searching codebase` → `Drafting patch`），最后落成 `Done` 或 `Failed: <reason>`。这是流式上报器；它编辑同一条 Slack 消息，所以 thread 不会被刷屏。
-   **结构化日志**——`~/.harness/logs/agent.log` 和 `~/.harness/logs/error.log`。每次 spawn、每次链式派发、每次重试尝试都拿到一条结构化事件。这是他们诊断时 grep 的东西。
-   **终态事件的 Slack DM**——MR opened、MR merged、OKTA expired、CI escalation。这些是一等公民的操作者通知，适用时挂在案例线程下。
-   **系统日志**——`~/.harness/logs/system.log`，记跨切面事件：部署完成、watchdog 动作、同步状态。

他们做了一个明确选择：**不**建 dashboard。Slack 和 grep 就是控制台。作者给的理由很扎心：dashboard 多出第三个要维护新鲜度的面，而它一旦过时，就是工程师最先停止阅读的东西。流式 Slack 消息这个模式提供的运营信号已经够用，他们从没想过要在上面再加 UI。这是一个反潮流但很诚实的决定——它承认「漂亮的看板大多沦为没人看的摆设」，并把节省下来的维护成本算进了系统的可长跑性。

## MCP Health Monitoring

harness 依赖五个外部 MCP（Slack、Datadog、Jira/Confluence、Glean、GitLab）。每个有自己的 OAuth token、自己的刷新节奏、自己不透明的失败模式。不治理的话，MCP token 过期会表现为：一个 Claude 会话莫名其妙地返回「我没有访问那个工具的权限」，而操作者那边看不到任何错误。

一个小服务——**mcp-watchdog**——堵上这个缺口。每 10 分钟它轮询每个 MCP 的健康端点、尝试一次空操作的工具调用，如果失败：

1.  尝试一次静默 token 刷新。
2.  刷新失败，就给操作者发条 Slack DM：「Atlassian MCP 需要重新认证——点这里。」
3.  连续两次断连之后（`DISCONNECT_CONFIRM_THRESHOLD`），用最近一张工单上的一个 `:warning:` reaction 升级。

作者点出这类失败有多折磨：没有这个循环，一个过期 token 会表现成「agent 今天变蠢了」——这一类失败极难诊断，因为 agent.log 里没有任何一句说「token 过期了」，它说的是「工具返回了空结果」。把认证健康做成一等公民的可观测性面，是他们「本该更早就建」的一步。这句自我检讨很值钱——「agent 今天变蠢了」恰恰是无数 agent 系统被冤枉的真相，根因往往不在模型而在那条没人盯的 token。

## Layer 6 — Human-in-the-Loop Control

这是用得最多的面，也是他们一开始最没建好的面。所有操作者交互都走单一渠道——自我 DM——组织成六个命令族：

-   **Task**——`admin: task <jira>` · `--auto` 用于无人值守执行。触发完整的 analyze → implement → MR 流水线。
-   **On-call**——`admin: oncall-run <url>` 从一个 Slack thread 启动调查。`oncall-toggle` 开关自动监控。
-   **MR control**——`admin: mr rescan` 重新轮询某个特定 MR。`mr pause/resume` 闸控该 MR 上的自动回复循环。
-   **Pause**——`admin: oncall pause [duration]` 让自动派发静音。`oncall resume/status` 解除或显示当前状态。
-   **Finalize**——`admin: finalize <case>` 整合 gap 报告、对 harness 开改进 PR。
-   **Registry**——`admin: register-thread` 手动把一个 Slack thread 关联到一个案例。`close-case` 标记它完成。

最难调对的模式是**暂停 kill switch**。第一版把所有东西都暂停了——包括 admin 命令本身。结果就是：那个把系统暂停了的操作者，反而没法把它恢复回来。这个 bug 黑色幽默到位——它精准复现了「一刀切」式开关最经典的翻车现场。第二版暂停得太窄，操作者的意图（「别吵了」）没被尊重。当前版本被校准到一个精确的区分：**暂停让由他人活动驱动的自动派发静音**，但绝不让操作者自己的 admin 命令或线程内交互静音。poller 里的派发点干净地分成两组：代表他人驱动的自动活动的（暂停时被闸住），和代表操作者自身意图的（永远在线）。

这个校准是吃了亏才学会的：第一次部署暂停后不久，一条审查者评论的自动回复在另一个案例上悄悄开了火，而他们没注意到，因为操作者的暂停看起来正在生效。当前设计把暂停当作**定向静默**——安抚系统的同时保住操作者的能动性。

一张图把这个划分讲清楚：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/17.webp)

规则是：**操作者自己的动作绝不静音；他人的活动在暂停时被闸住。** `admin: oncall pause` 控制的就这一件事。一个看似只是「加个暂停按钮」的小功能，能演化成三版迭代，这恰恰印证了文章的母题——「人在环路」的控制面，最难的从来不是有没有，而是边界划在哪。

## Three Layers of Safety

暂停 kill switch 是最显眼的安全面，但不是唯一的。还有两层在没有任何运行时配置的情况下强制安全——一层在代码里，一层在部署惯例里：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/18.webp)

Slack 频道白名单比听上去重要得多。项目早期他们有过一次险些出事的近失（near-miss）：一个配置错误的 Slack token 拿到了对内部团队频道的写权限——意味着 agent 本可以直接贴给队友、完全绕过操作者，而团队没有任何办法把它和真人消息区分开。这之后，每一次对外的 `chat.postMessage` 在发送前都会拿目标频道去对白名单校验。唯一被允许的目的地是操作者的自我 DM（操作者与 bot 之间那条 DM 的频道 ID——一个硬编码的单一值）。如果 agent 哪天想直接发消息给队友，它必须把内容浮现回操作者，由操作者选择是否转发。作者把这条上升成一个反复回到的架构选择：**agent 只能通过操作者的信箱对外发声。** 「agent 本可以冒充人类直接给队友发消息」这个近失，是全文最让人后背发凉的一个，也最有力地论证了为什么对外通信必须收口到一个硬编码的单一出口。

## Beyond the Six Layers: The Pieces That Make It Production

上面的六层是承重结构。还有几个机制坐在层级分类之外，却同样关键——按作者的经验，正是这些部分，区分了「demo 好看」的 harness 和「能不间断真跑」的 harness。这等于预告：真正让系统活下来的，往往不是那张漂亮的分层图，而是图外这些没人愿意写进 PPT 的脏活。

## Multi-Project Routing via a Single YAML

harness 今天服务两个项目，每个有不同的运营规则，而加第三个需要零代码改动——只要在 `agent-config/projects.yaml` 里加一条：

```yaml
projects:
  team-a:
    sop: "team-a/SOP.md"
    kb: "team-a/KNOWLEDGE-BASE.md"
    dfr_threads_dir: "team-a/threads"
    task_dir: "task-workspaces"
    slack_channels: ["support-channel-a", "support-channel-b"]
    jira_prefixes: ["AAA", "BBB", "CCC"]
  team-b:
    sop: "team-b/SOP.md"
    kb: "team-b/knowledge-base/kb.md"
    dfr_threads_dir: "team-b/threads"
    task_dir: "task-workspaces"
    slack_channels: []
    jira_prefixes: ["XX", "YY"]
```

一个 `resolve_project()` 函数按优先级路由每条进来的信号：**Jira prefix > Slack channel > default**。每个项目拿到自己的 SOP 文件、KB 文件和目录树。调查 prompt 的组装会根据信号解析到哪个项目，自动挑对的 SOP 和 KB。这就是让单个 harness 实例服务两支规则迥异团队的那个把戏。

这个模式可以推广。任何在同一套 harness 上跑多个产品团队的组织都能用同样的办法：声明式配置，没有按项目分叉的代码。它还逼出一种干净的纪律——任何特定于某个项目的东西都必须活在它的 SOP 文件里、而不是代码里，因为代码是跨项目共享的。「项目特异性必须沉进配置、不许污染共享代码」这条纪律，比那段 YAML 本身更值得学——它把多租户的混乱挡在了代码之外。

## Cross-Machine Session Continuity

操作者在两台机器上工作：一台笔记本和一台数据中心里长期存活的 CVM。同一个 Claude 会话需要能从任意一台恢复。这很难，因为 Claude Code 默认把会话状态持久化到本地磁盘——从另一台机器重启，会话就是不可见的。

harness 用一个 **git 支持的会话存储**来解决：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/19.webp)

机制分三部分：

1.  **会话被 git 跟踪。** 每个会话的 `.jsonl` + `.meta` 文件在每次有意思的状态变更时被 commit 并 push 到一个私有 repo。repo 是事实来源。
2.  **一个 5 秒的后台拉取器**在每台机器上跑，保持本地副本是最新的。这预拉上下文，好让 Claude 会话恢复不必在每次按键上付网络延迟。
3.  **按 CWD 查会话 UUID。** Claude 在某目录里启动时，`session-start.sh` hook 扫描本地 `.meta` 文件，找出 CWD 匹配的那个、恢复对应的 UUID。同一个会话 UUID 会在操作者所在的那台机器上恢复。

同步 hook（`user-prompt-submit.sh`，每次 Claude prompt 之后开火）按严格顺序做三件操作，用 `flock` 串行化：

1.  **Fetch + reset。** `git fetch origin main && git reset --hard`——现在本地 `active.jsonl` 装的是*另一台*机器的 transcript。
2.  **在覆写之前抽取增量。** 读水位线（我们已经从另一台机器注入了多少行），只抽取从那以后的新行，存到一个临时文件。这一步必须发生在第 3 步*之前*——早先一个 bug 把它俩顺序调反了，导致注入总是把本机自己的 transcript 又读回给自己。
3.  **Push 自己的 transcript。** 把本地 Claude 会话 transcript 复制到 `active.jsonl`，commit、push。然后把水位线更新成本地行数。

flock 释放后，第 2 步的增量作为一条系统消息注入 Claude：「另一台机器自我们上次同步以来加了这 N 条消息。」下一份 prompt 就同时看到两台机器的上下文。

一个 `.linecount` 文件把真实总行数和文件大小分开追踪，因为当 transcript 为了 push 被截断时 `wc -l` 给的答案是错的（push 前他们截到 1,000 行，以避开 25 MiB 的 git 限制）。没有 `.linecount`，水位线会在每次截断时复位，导致注入反复重投成千上万行已见过的内容。

这里值得抬走的更广的想法是「git 即状态存储」。他们不只把 git 当代码仓库，而是当作一个 [**CRDT**](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type)**式的、agent 状态的持久化日志**，操作者的两台机器是它最终一致的读者。把 git 重新定义成「agent 状态的 CRDT 日志、机器当读者」是个相当大胆且优雅的隐喻——它把一个被无数人用烂的工具，用出了一个完全不同的语义。

还有一个机制和跨机器连续性密切相关：记忆固化。memory\_consolidate 步骤（把 transcript 重写成干净的 TASK.md/ANALYSIS.md）不按固定计时器开火。一个独立服务——`**consolidate_trigger**`——拿每个 turn 对照四个信号评估：transcript 里出现 commit SHA 或决策动词、一个刚关闭的评审 thread、一个累积 transcript 超过 5KB 的案例工作区、或离上次固化至少过了一小时。任一信号触发固化。其原则是：信息在一个决策做出之后就停止累积，而非在固定时长之后——这四个信号合起来近似「这段对话此刻是否处在一个自然的歇脚点」，而不需要操作者去手动标记它。用「决策做出了没」而非「过了多久」来判断该不该固化记忆，这个思路比定时器聪明——它抓的是对话的语义节律，而不是钟表。

## Worktrees Are Not an Implementation Detail

前面说过「一切都发生在 worktree 里」。这值得展开，因为它代表的架构选择是真·承重墙。作者亲自把它从「实现细节」里拎出来，标题就摆明态度。

最初的设计是让 agent 吐出一个 diff，harness 在主 repo 检出里调 `git apply`。这有两种反复咬他们的失败模式：

1.  **脆弱的 diff 头。** Claude 会吐一个带行号的 diff，这些行号在生成时是对的，但在一个并发 commit 落地之后就错了。`git apply` 会拒掉整个 patch。
2.  **主树卡死。** 如果 `git apply --check` 中途失败，主 repo 会被留在半应用状态。下一个在同一 repo 上跑的案例会继承这个脏树。

修复方案是 `worktree_manager` 服务。在任何需要改代码的 agent 跑之前，manager 在 `~/.worktrees/case-<case_id>-<stamp>/` 创建一个沙箱化 worktree，从一个干净分支检出。Claude 在那个 worktree 里用它原生的 Edit/Write 工具操作——没有 `git apply`，没有 diff 文本的往返。agent 工作的方式就和人类工程师一样：开一个新分支、在新检出里编辑、commit、push。「让 agent 像人类工程师那样开分支干活，而不是吐 diff 让系统去贴」——这个转变看似朴素，却一举消灭了两类反复咬人的失败，是把「对齐人类工作流」当成可靠性手段的范例。

worktree 在 MR 合并或关闭时被拆掉。如果 agent 中途失败，worktree 是唯一受影响的东西——主检出和其他并发案例毫发无损。

这正是在单台机器上让**安全的并发 agent**成为可能的根本。没有每案 worktree，同一 repo 上的两个案例会在文件系统状态上竞态；有了它们，它们由构造上就是隔离的。

## Dogfooding: The Harness Fixes Its Own Pull Requests

harness 的源码活在一个私有 GitHub repo 里。当 gap-report → finalize → MR 循环对 harness 自己产出一个改进 MR 时，那个 MR 有它自己的 CI：pytest、linter、Python 语法检查。如果一个 harness 改进 MR 上 CI 挂了，一个独立服务——**pr-ci-fixer**——会用 mr-monitor 处理客户 MR 失败的同一方式把它接过来。

结果是：**harness 不只自愈它为工单写的代码，也自愈它为自己写的代码。** 那个加了一个新重试预算的 MR 可能因为预算破坏了一个已有测试而 CI 挂掉；pr-ci-fixer 注意到、在一个 worktree 里 spawn 一个 Claude 会话、打补丁、重推，然后人类评审者看到一个绿色的、可以直接合的 MR。他们不止一次看着这个循环在真实的 harness 改动上闭合。

在 harness 自己源码上 dogfood 这个循环，是「这玩意儿到底是不是生产级」的承重测试。作者给了一个相当有说服力的判据：如果 agent 能在自己的 pull request 上保持自己的 CI 绿，那它大概也能被信任去处理客户的 PR。这个「让系统先在自己身上证明自己」的标准，比任何 benchmark 跑分都更接近「生产可信」的本质——它要求系统对自己也下得了手、也修得回来。

## Test Architecture: Five Layers for Five Failure Classes

harness 有跨 126 个文件的 4,656 个测试函数。作者自己提醒：这个数字脱离了「每个测试抓什么」的框架就有误导性，因为不是所有测试都等价。难得有人在甩出测试数量后立刻自我泼冷水，这份克制比那个大数字更可信。

L0–L4 框架按每个测试隔离什么，把它分到五层之一：

-   **L0（纯边界）：** 无 mock、无文件系统、无 IO。测纯逻辑——闸门函数、JSON 抽取、置信度天花板数学。一个在 L0 通过的测试，无论部署环境如何都保证正确。
-   **L1（文件隔离）：** tmpdir 里的真实文件系统、真实线程、对 pathlib 不打 mock。抓 L0 按设计会漏掉的并发写竞态和状态文件原子性 bug。
-   **L2（服务隔离）：** 真实子进程执行 + 对外部 API（Slack、GitLab、Claude）打 mock。抓服务组件之间的集成 bug——派发路由、payload schema、状态文件状态迁移——这些是 L1 漏掉的，因为 L1 只隔离文件系统行为。
-   **L3（沙箱）：** 真实 git repo（经子进程）、假 Slack 和 Claude 响应。对着真实 git 对象模型测完整 workflow 序列——任务注册、implement 阶段启动、待确认。
-   **L4（集成矩阵）：** 六个子类，覆盖 actor 契约（L4a）、admin × role 矩阵（L4b）、跨 run 状态一致性（L4c）、服务间链（L4d）、外部 actor 角色（L4e）、方法论缺口（L4f）、以及时序排序（L4h/L4i）。
-   **L4g（生产回放）：** 13 个测试，把特定的生产事故复现为回归测试。这些是在生产里发现一个 bug *之后*写的——测试编码了触发该 bug 的确切 actor、上下文和状态，并被永久保留以防回归。

L4g 测试写起来最贵、留着最值。它们编码了关于失败模式的「机构记忆」，而这些失败模式在复发之前是看不见的。把生产事故一条条钉成永久回归测试，等于把每一次踩坑都变成系统的免疫记忆——这才是 4,656 这个数字真正有分量的地方，而不是数字本身。

## The Anti-AI-Flavor Style Guide

harness 开的每个 MR 在提交前都过最后一道滤网：一份专门为了让输出**看上去不像 AI 生成**而存在的 style SOP。规则很直白：

-   **零 AI 署名。** 没有 `Co-Authored-By: Claude`、没有「Generated with Claude Code」、commit 消息里没有 `[bot]` 后缀。这个 MR 是团队的，不是模型的。
-   **不用填空式标题。** 「What this MR does / why we need it」这类模板被禁。MR 描述写成连续散文，配一个具体的 `Changes:` 项目符号列表，就像一个工程师手写的那样。
-   **不用加粗内联标签结构。** `**Root cause:**` 后面跟一段，被禁。`**Fix:**`、`**Risk:**` 也禁。标签式加粗是最可靠的「AI 味」露馅信号。
-   **工作证明必须命令可复现。** 不是「本地已验证」——而是 `pytest tests/test_foo.py::test_bar -v`，带上真实命令和相关的 3 行输出。

这份 SOP 在 MR 描述被组织的那个点注入补丁生成 prompt。它独立于 prompt 模板存在，因为它是**操作者策展的**：SOP 可以随时编辑，下一个 MR 就拾起新规则，无需重启 daemon、无需改代码。Gap 循环甚至可以提议对 SOP 自身的编辑——关于「agent 该怎么写 MR」的元规则，最终会变成对那个元规则文件的 MR。

作者说这件事比听上去更重要，理由很现实：没有什么能比「团队发现每个 MR 描述都一股 AI 味」更快地杀死对一条自动化 MR 流水线的信任。味儿一旦出来，合并门槛就抬高，agent 真正的信号被埋掉，操作者开始想手动重写描述——这就把整件事的意义打回原形。style SOP 正是让 MR 描述融进人写的那一堆里的东西，而这正是让循环保持可用的东西。这一节很有反差感——一套以「机器自主」为荣的系统，居然要花专门的力气让机器藏起自己的机器味。但作者的逻辑成立：信任是这条流水线的命根子，而「一股 AI 味」就是信任的头号杀手。

## End-to-End Walkthrough: One Ticket, One MR

一张真实工单到达时会发生什么。时间是近似值。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/20.webp)

端到端，这个循环对一张中小型工单通常在 **30–90 分钟的墙钟时间**里完成。操作者的总主动投入通常在 5 分钟以内：确认分析、可选地引导有歧义的决定、可选地点一下合并。把人类的「主动投入」压到 5 分钟以内，而不是把人类彻底踢出环路，这个度拿捏得相当克制——它卖的不是「全自动」，而是「省下你 95% 的时间还把决定权留给你」。

调查本身会累积。30 个案例之后，harness 有 30 个 `gap_report*.md` 文件，每个都指向 LLM 认为可以改进的某处。一个双周一次的 `admin: gap-patterns` run 把这些聚类成反复出现的失败模式，下一次 finalize 把头部那些转成对 harness repo 的 MR，于是 harness 改进自己。这就是自我改进循环的具象。

## Comparison to Other Harnesses

好几个其他系统共享这个 harness 形状。作者摆出他们这套与之重叠在哪、又分岔在哪：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/21.webp)

他们这套**异常深**的地方：

-   **预承诺假设锁**——Claude 在跑任何一轮调查迭代之前承诺 ≥6 条候选假设，防止事后合理化。他们找到的商业 harness 里没有等价物。
-   **19 个函数的质量闸门框架 + 机械置信度天花板**——置信度被算成一个硬算术封顶（`1.0 − open_q×0.08 − unchecked×0.05`），而不只是一个自报数字。三族闸门（逻辑一致性、结构完整性、断言上限）加上逐维度的对抗惩罚。他们找到的商业 harness 里没有这个。
-   **会修改 harness 自己代码库的自我改进**——gap\_reports → 过滤后对 harness 自己代码/prompt/SOP 的 PR。最接近的学术等价物（SICA，arXiv 2504）拿到了 benchmark 增益，但它是离线研究；没有生产系统出货「自主修改 harness 代码」这件事。
-   **项目范围的 SOP 个性化**——每个项目的 SOP 基于它自己的失败模式频率演化。20–30 个案例之后，SOP 反映领域特有的调查失败。没有别的生产 harness 实现这个。
-   **带显式交接的多 agent 编排**——`oncall_run` → `case_followup` → `gap_analyzer` → `finalize_case` → `dev-agent`，每个有定义好的交接 schema 和隔离的 worktree，而不是一个 agent 包办一切。
-   **跨机器状态同步**——经 git 支持的水位线注入，同一个 Claude 会话在笔记本或远程 VM 上带完整上下文恢复，无需手动同步。
-   **内建 CI 自动修复 + 审查者自动回复循环**——只有 Devin（商业）出货可比的内建自愈；其他每个框架都要自定义接线。

他们这套**刻意做薄**的地方：

-   没有正式的**评估 harness**。他们不在 benchmark 上给 LLM 打分。生产反馈就是评估。
-   没有 **dashboard**。Slack + 日志。
-   没有**成本计**。token 用量被松散追踪，但不作为 dashboard 浮现。成本由人在环路的确认步骤来兜底。

这份对比的可信度恰恰来自后半截——一个肯把「我们没做什么、为什么不做」摊开讲的团队，比只列卖点的更让人相信前半截的「我们异常深」。尤其「生产反馈就是评估」这句，是对当下 benchmark 崇拜的一记冷静反问：跑分高，真能代表它在你的工单上靠谱吗？

## Outlook

harness 仍在演化——自主循环里的覆盖缺口、成本计量、一个正式的评估 harness——这些都不是拦路虎。系统已经做到了它被造出来要做的事：把 Slack 信号转成合并的代码、隔夜处理审查者反馈、自愈 CI、把自己的改进反哺进自己的下一个版本。

更深的教训是那个元循环（meta-loop）。harness 里每一道疤，都是因为一张真实工单去碰了它才被抓住的。构建循环和运营循环是同一个循环，只是在不同时间尺度上。生产信号回流到 harness 自己的源码；agent 提一个修复；人类评审并合并；下一类 bug 就离被修剪掉近了一步。「build loop 和 operate loop 是同一个循环」这个收束很漂亮——它把全文那些零碎的疤痕统一成了一条主线。

系统没做完。作者甩出一句很有态度的判断：一个自以为做完了的生产 harness，是一个停止累积疤痕的 harness——换句话说，是一个不再跑在真实负载上的 harness。

**先把循环造出来，再让它自主。** 顺序很重要，因为一个没有反馈通路回流进自己底料的自主系统，不过是一种更快地、规模化地出货同一套错误的方式。这句收尾辣味十足，也立得住——它把整篇文章的方法论压成一句可执行的告诫：别急着炫自主，先确认你的系统能从自己的错里学回来，否则你只是给翻车装了个涡轮增压。

## 原文链接

> **Building a Production Agent Harness: Turning Claude Code Into a Multi-Agent Engineering Pipeline**
>
> 来源：Medium | 作者：Messi Li
>
> 一篇罕见地把「能 demo」和「能在生产里长跑数月」之间的全部脏活摊开讲清楚的实战长文：用预承诺锁、19 个质量闸门、对抗式评审、git 即状态存储、三层自愈和会改自己代码的自我改进循环，把 Claude Code 攒成一条真正盯 Slack、提 MR、自愈 CI 的多 agent 流水线。
>
> 👉 <a href="https://licaomeng.medium.com/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline-1db4e242d08a" target="_blank" rel="noopener noreferrer">点击阅读原文</a>
