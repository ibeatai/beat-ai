---
title: "把 Claude Code 改造成多 agent 工程流水线：一套跑了几个月的生产级 agent harness 实录"
author: Messi Li
url: https://licaomeng.medium.com/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline-1db4e242d08a
translated: 2026-06-03
tags:
  - LLM
  - AI Agent
---

# 把 Claude Code 改造成多 agent 工程流水线：一套跑了几个月的生产级 agent harness 实录

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/01.jpg)

一个孤立的编码 agent，本质上是泡在罐子里的大脑。它会思考、会生成代码、会调函数——但它没法在凌晨三点回你的 Slack 私信，没法重跑一个挂掉的 CI，没法处理刚冒出来的 merge 冲突，也记不住昨天某位 reviewer 提的问题至今还没人答。

让这颗大脑真正有用的，是 **harness**：套在 LLM 外面的运行时脚手架，给它装上感官、双手和记忆。它要管的东西包括：

- 事件接入（event ingestion）
- agent 编排（orchestration）
- 持久化状态
- 自愈循环（self-healing loops）
- 可观测性（observability）
- 给人类操作者的控制面

这篇文章讲的就是怎么造一套这样的 harness——不是研究 demo，而是一套真正跑在生产里的系统：它盯着一个 Slack 频道，对内部五个仓库提 MR，整夜处理 reviewer 的评论，CI 变红时还能悄悄自愈。

我们这套是内部系统，建在 [Claude Code](https://claude.ai/code) 之上，是个多 agent 流水线，至今已经连续跑了好几个月——从 Slack 吃进信号，派发真实的工程任务。下面是自顶向下的视角：它能干什么、各组件怎么拼起来，以及那些塑造了设计的生产踩坑。

**本文目录：**

-   [为什么要 harness，而不只是一个 agent](#why-a-harness-not-just-an-agent)
-   [我们的 harness 到底干什么](#what-our-harness-does)
-   [harness 必须补上的三道缺口](#three-gaps-a-harness-must-close)
-   [Layer 1 — 事件接入](#layer-1--event-ingestion)
-   [Layer 2 — agent 编排](#layer-2--agent-orchestration)  
    ↳ [自主 agent 循环细节](#the-autonomous-agent-loop-in-detail)  
    ↳ [自我改进层](#the-self-improvement-layer)
-   [Layer 3 — 持久化状态](#layer-3--persistent-state)
-   [Layer 4 — 自愈循环](#layer-4--self-healing-loops)
-   [Layer 5 — 可观测性](#layer-5--observability)
-   [Layer 6 — 人在环中的控制](#layer-6--human-in-the-loop-control)
-   [六层之外：让它真正能上生产的那些零件](#beyond-the-six-layers-the-pieces-that-make-it-production)
-   [端到端走一遍](#end-to-end-walkthrough-one-ticket-one-mr)
-   [和别家 harness 的对比](#comparison-to-other-harnesses)

## 为什么要 harness，而不只是一个 agent

2023 年 ChatGPT plugins 出来时，很多团队都试过最显然的玩法：给 LLM 套个聊天 UI，配几个 function-call 工具，就管它叫"工程 agent"。这套路在 demo 里勉强能跑，上了生产就没一次成的。第一周之内，三种翻车模式必然现身：

1.  **操作者一关标签页，上下文就没了。** 记忆得活过单次会话。一个真实工程任务跨度好几天：建 JIRA、拉分支、起草、评审、处理评审意见、CI 过、合并。没有任何一次 LLM 调用能扛住这套状态。
2.  **外部一变化，它没法响应。** reviewer 下午四点丢了条评论；第三次提交把 CI 跑挂了；队友在 thread 里回复了。agent 必须在这些事件上被唤醒，而不是无限轮询，也不能干等用户重新发 prompt。
3.  **它没法从自己的失败里恢复。** agent 推了个 commit，CI 挂了；操作者第二天得回来重新解释一遍失败原因。或者 agent 的 Okta（一个身份管理平台）会话在任务半途过期，整条流水线悄无声息地死掉。这俩每天都在发生。代价就是：本该 agent 处理好的活，工程师却要在晚上和周末被 page 起来。

补上这三点，正是 harness 层的活。harness 才是 `claude.ai` 聊天页和 Claude Code 本体、Cursor 的后台 agent、Cognition 的 Devin 以及我们这套系统之间的分水岭。大脑（模型）在这些系统里基本是可以互换的——真正不同的，是包在它外面的 harness。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/02.webp)

大脑只能透过 harness 跟外界打交道。让 LLM 表现得像个系统的工程活，几乎全在 harness 这一层。

## 我们的 harness 到底干什么

起点是个很具体的问题：一个 Slack 频道——就叫它 `#support`——每周大约堆进 30 张工单，每张都得先调查、然后（通常）在五个仓库之一里改代码。每张工单跨好几天。reviewer 在 MR 上留评论。CI 偶发性抽风。有时候查到一半发现这就是上个月那张的重复件。扛这摊活的团队晚上周末都在被 page，而活本身并没好转——同一类工单每个周期都会再来一遍。

harness 担三件事，按介入深度排序：

1.  **调查（Investigate）**——`#support` 里出现新工单时，顺着 thread 往下走，跨仓库收集上下文，把结构化的分析回贴到同一个 thread 里。
2.  **修复（Fix）**——操作者批准后，提一个带改动方案的 MR（Merge Request），处理 reviewer 评论，盯 CI，CI 挂了就修，一路走到合并。
3.  **自我改进（Self-improve）**——每个结案都会反哺：一旦出现反复出现的模式（比如同一区域的五张工单都动了同一份配置），就生成缺口分析，并对 harness 自身提改进方案。

第三件事，才是这套系统区别于"花哨工单机器人"的地方。harness 的底座就是它自己的源代码，而 LLM 对这份代码有写权限——通过走人类评审的 PR 来落地。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/03.webp)

六个逻辑层，下面逐层讲。

## harness 必须补上的三道缺口

整套 harness 围绕一条洞察展开：**一个无状态、被动响应的 LLM，不是生产系统。** 六个层存在的意义，就是补上裸 LLM 自己补不了的三道缺口。

**缺口一：响应性。** LLM 没法自己醒来。Layer 1（事件接入）补这道——Slack mention、GitLab CI 结果、PagerDuty 告警，统统汇入一个统一的派发队列。LLM 从不轮询，它是被调起来的。

**缺口二：持久性。** LLM 跨会话会忘事、跨机器会丢上下文，也分不清"我处理过那条"和"那个 commit 真的落到 origin 上了"。Layer 3（持久化状态）补这道，分三档：

- 进程内内存——管单进程去重
- 本地 JSON——存运维映射表
- git 同步的工作区——存可持久化的 case 状态

跨机器的会话连续性（后面会讲）是同一思路的延伸：状态活在 git 里，不在进程里。

**缺口三：质量。** 没有结构约束，LLM 推理循环会从薄弱证据里写出自信满满的结论，没把能查的源查完就宣布调查结束，生成的 MR 描述第一句话就一股 AI 味。Layer 2（agent 编排）和调查循环一起补这道，靠的是：

- 结构化的输出契约
- 每轮迭代都过的质量门
- 循环结束后的对抗式评审
- 把"改代码 → CI → 合并"这条外圈闭上的自愈循环

把这些层串起来的，是**复利效应**：每个结案都让下一个更快。一份批准过的调查，变成知识库条目；一个批准过的 MR，变成缺口分析器能引用的模式；一个最终结案，对 harness 自身开出一个改进 MR。系统一个周期接一个周期地，收窄它处理得不好的那类工单。

## Layer 1 — 事件接入

harness 需要在三类外部信号上醒来，而且每类的延迟和可靠性都不同，所以各用一套不同的入口：

| 信号源 | 覆盖内容 | 入口策略 |
|---|---|---|
| **Slack 消息** | 频道 mention、thread 回复、自我私信里的 admin 命令 | Socket Mode + 轮询双保险 |
| **GitLab 活动** | MR 评论、CI 流水线结果、流水线失败 | mr-monitor cron 轮询 |
| **PagerDuty 告警** | oncall page 引用到 harness 见过的工单时，把上下文浮出来 | 告警接入 |

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/04.webp)

教科书式的描述看不到的，是这两个实战细节。

**最终一致性的坑。** Slack 的 `conversations.history` 接口可能滞后 1–30 秒。如果轮询器看到空结果就傻乎乎地把游标往前推，任何 `ts` 落在那个时间窗里的消息都会被永久跳过。我们的做法是：**空轮询时冻住游标**，从同一个最旧时间戳重试，直到至少有一条消息回来为止。重试之间产生的重复处理，由一个按 message-ts 去重的集合滤掉。

[**Socket Mode**](https://api.slack.com/apis/socket-mode) **+ 轮询，做带裤腰带又系背带的双保险。** Socket Mode 给 thread 回复带来亚秒级延迟。但 WebSocket 连接会断。轮询器就是安全网——它靠一个共享的 `socket-dedup` 文件兜住 Socket Mode 漏掉的消息，保证同一条不会被派发两次。没有这层，连接断开期间到的消息会悄无声息地蒸发，操作者根本收不到"漏了"的信号。

mr-monitor 这条 cron 循环性质不一样。GitLab 的评论不会主动把事件推进我们的流水线，只能去轮询。它最难搞的问题，其实是**轮询本身的扩展性**：harness 接的长寿命 MR 越多，它那 10 分钟一轮的轮询循环就攒下越多死 thread 的游标，每个游标每轮都要花一次 Slack API 调用。不处理的话，这些会持续制造 429 限流风暴，把真正该轮询的活给饿死。我们加了游标自动驱逐：上游 API 返回 `thread_not_found` 时，游标在下一轮循环里直接消失。修复前：每轮 12 次重试，连续好几个小时；修复后：归零。

## Layer 2 — agent 编排

这一层才是 LLM 真正跑起来的地方。harness 把 worker 拉起为临时的 [systemd](https://systemd.io/) 单元，每个带一份 JSON payload，里面点名了一个 workflow + 一个 case\_dir + 一份 thread 上下文。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/05.webp)

每条流水线，都是对 Claude Code（大脑）的一次长时调用，配一段精挑过的 prompt 和一个可写的 `case_dir`。case\_dir 就是 agent 的草稿空间，里面装着这个 case 的 `TASK.md`、累积下来的 `followup_transcript_*.md`、生成的 `gap_report*.md`，以及中间态的 JSON 产物。

编排的章法很关键。我们不是用一个 agent 包打天下，而是有好几个各司其职的专用 agent，在明确的交接点上串成链：

-   **`oncall_run`**——跑初始调查，最多 **20 轮推理迭代**，每轮过质量门，结尾有一步显式的自我批判。产出：`ANALYSIS.md`（哪儿出了问题）和 `TASK.md`（该怎么做）。
-   **`case_followup`**——长寿命的聊天 agent。操作者可以在一个 case thread 上私信回复好几天，每次回复都会带着完整对话历史重新调起 case\_followup。关键在于：case\_followup 每隔 N 条回复，还会把 `gap_analyzer` 当子 agent 调起（*自我改进循环下文细讲*）——这条循环产出那些指向 harness 自身代码的 `gap_report*.md`。
-   **`finalize_case`**——收口者。一个 case 闲置满四小时，或者操作者跑 `admin: finalize <case>` 时，它把所有 gap\_report 收集起来，滤掉误报，然后**只开一个 MR**装下存活的缺口修复。这个 MR 打的是 harness 自身的代码库，不是原工单那个仓库。这就是自我改进循环的实体形态。
-   **`dev-agent`**——干活的。它在每个 case 专属的 [git worktree](https://git-scm.com/docs/git-worktree) 里跑，改代码、推送、盯 CI、CI 挂了就修、处理 reviewer 评论。三种任务模式：`address_review`、`fix_ci`、`task`（原始实现任务）。

这张编排图上有两点不那么显眼，但很关键：

**第一：一切都在 worktree 里发生，** 不碰主仓库的 checkout。多个 case 可以同时在飞，互不踩脚。轮询器 / dev-agent 派发器在 payload 里传 worktree 路径，LLM 从头到尾看不到主 checkout。

**第二：case\_dir 是唯一真相源。** `~/.harness/` 里的状态文件（马上就讲）把一个 Slack thread → 一个 case\_dir 做映射。agent 永远相对 case\_dir 来操作。这让系统的其余部分简单得多：把 case\_dir 改个名，只要映射跟着更新，所有 workflow 自动跟上。

上面这张图说的是**跑什么**。下面这节说的是**怎么跑**——把一条 Slack 命令变成一份完成调查的机理。

## 自主 agent 循环细节

"这玩意儿到底干嘛"的一句话答案是：**操作者敲一条 Slack 命令，然后一个闭环——迭代推理、自我批判、工具调用——在无人再插手的情况下一直跑，直到要么活干完了，要么系统诚实地承认自己进行不下去。** 这句话背后藏着几乎每一个有意思的决策，这节就来拆开。

harness 内部跑着四种 agent 循环，各自针对一种不同形状的问题：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/06.webp)

上面是形状，下面是实质——按运行顺序逐个机制讲。上图对应着所有这些机制，读完后图里的节点就都说得通了。*（门控代号：* [*G1–G4、N1–N3、N5、A1–A3*](#quality-gates-the-structure-that-makes-confidence-mean-something) *·* [*D1–D15*](#adversarial-review-the-red-team-inside)*）*

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/07.webp)

## 循环开始之前：预承诺锁

第一次 Claude 调用触发之前，先跑两道保险。第一道是结构性的。

**预承诺锁（pre-commitment lock）。** oncall 调查流水线干的第一件事——在 Claude 跑任何一轮调查迭代之前——就是要求 Claude 承诺一份 `hypothesis_slate`：至少六个对问题的候选解释，外加四到五条用于在它们之间取舍的评判标准。这些被冻进 `kickoff_precommitment.json`，在整个调查的余下过程里被当作不可变。

这是整个系统里最重要、也最不显眼的方法论保险。没有它，LLM 推理循环会朝着早期证据最支持的那个假设收敛，然后回过头把后续所有证据都框成对它的确认。预承诺锁并不阻止收敛——它逼着调查从一个完整的假设空间开始，并记录从那个空间走到结论的路径。对抗式 reviewer（Phase B）检查的是：结论是不是初始那份假设清单在给定证据下本就会预测出来的那个，而不只是检查结论内部自洽。

**调查循环开始前还有一步：经验锚点（Phase C）。** 只在第一轮迭代时，harness 跑一组 OS 级命令，全程不让 LLM 参与——`journalctl` 的错误尾巴、磁盘和内存快照、日志文件发现。输出作为原始证据注入第一轮调查 prompt。这给了 Claude 一份事实性的系统状态快照——在它形成任何假设之前。这步非阻塞（出错就静默吞掉），耗时不到一秒；它的价值在于：让 Claude 的第一步推理从地面真相出发，而不是只从问题描述出发。

## 结构化输出：完成报告 schema

每一轮迭代都以一份可解析的 JSON 契约收尾——`completion_report`——harness 用它来决定下一步发生什么。形状是固定的：

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

这个块前后允许有自由文本，但被解析出来的报告才是各道门评估的对象、才是状态文件持久化的内容。这是我们找到的、保持 LLM 输出在流水线里可用的最大杠杆点：**拿一部分行文自由度，换一份可解析的契约。** 没有这个 schema，每个下游步骤都得从散文里重新抽事实；有了它，各道门就能跑成确定性的 Python。

## 质量门：让 confidence 这个数真的有意义

如果 Claude 可以不给任何支撑就直接写 `confidence: 95`，那这个数字就是营销噪音。harness 通过一组**质量门（quality gates）**强制一份结构契约：每轮迭代的输出都过门，证据和置信度对不上就拦着不让往前走。

每一次 Claude 回合之后，跑三族门：

- **G 族门（逻辑一致性）**——确保推理跨轮不自相矛盾。最清楚的例子：本轮 `open_questions` 变多了但 `confidence` 反而涨了，G1 就触发——分母扩大让任何向上的移动都失效。这族还会检查：散文里的张力措辞有没有登记成一条矛盾、相邻问题是不是用了合法的 status 对象而不是占位字符串。共 4 道门。
- **N 族门（结构完整性）**——检查调查产物是不是完整成形、不是半截桩子。N1：每个可执行的修复动作都必须配一个 `verify_action`，带上描述、预期结果和时机。提了修复却不说清你怎么验证，就被当作没干完的活。共 3 道门。
- **A 族门（断言天花板）**——施加 Claude 没法靠散文绕开的机械上限。A1 算出一个硬置信度天花板：`1.0 − (open_questions × 0.08) − (unchecked_sources × 0.05)`——不管 Claude 写啥，这道门都按算术给它封顶。其他 A 门还会：相邻问题没关完就拦住 `COMPLETE`、要求假设登记表非空、强制至少一个所提动作直接对应根因。共 7 道门。

另有两道点检：**EQ1** 要求根因节点引用硬证据（非 INFERRED），否则接受 60% 封顶。**P7** 每有一个对抗维度被标 FAIL，就把天花板压低 7%，下限 40%。

门框架一共含 19 个函数。一句话记法：

- G 门盯**Claude 写了什么**
- N 门盯**Claude 漏了什么**
- A 门盯**Claude 声称了什么**

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/08.webp)

门违规会变成 **`pending_guard_notes`**，在*下一轮*迭代的 prompt 里被前置到 `## SYSTEM QUALITY GUARD NOTES (from previous iteration)` 标题下。指令写得很明确：往下走之前，先把所有 note 处理掉。这才是逼迫机制——门不拦 Claude，它是把 Claude 上一轮的违规变成它下一轮开头第一眼看到的东西。模型礼不礼貌无所谓，真正推动调查的是这套结构性强制。

还有一步**上下文钉死（context-pinning，A7）**，跑在每个 prompt builder 的末尾，不依赖是否有违规：把当前报告里未解决的矛盾、未验证的假设，逐字重新贴到下一条 prompt 的底部。这就堵住了那个被广泛记录的"注意力衰退"模式——模型在第 3 轮承认了一个麻烦，到第 7 轮就悄悄把它丢了。

## confidence 作为一个具体的数

最重要的设计决策是：**agent 在每轮迭代的结构化输出里给自己打一个** `**confidence: <int>**`，0–100 制。这数字不是摆设——它门控真实行为：

| confidence 取值 | 门控行为 |
|---|---|
| **< 70** | 不准退出调查循环。流水线强制最多再跑三轮，注入一条 guard note："你说你做完了，但置信度没过线。" |
| **≥ 70 且 `status: COMPLETE` 且无 `open_questions`** | 开门进入**对抗式评审**阶段（红队 / 蓝队）。 |
| **≥ 95** | 开门让 agent **自动执行非破坏性动作**（Jira 评论、状态流转、工单指派）。低于 95，这些动作就停在草稿建议态，等操作者。 |
| **连续两轮下跌 > 10%** | 循环以 `degrading` 为原因退出——agent 在倒退，停下来告诉操作者。 |

每个 case 完整的置信度轨迹会以 sparkline 形式写进 `ANALYSIS.md`：`45% → 62% → 78% → 80%`。操作者就看这条 sparkline 来分诊该不该信这个结果。harness 自己的自我改进循环，在决定哪些旧 case 值得换个 prompt 重跑时，读的也是它。

## 三道强制续跑机制

上面那张图里有三个分支：退出检查触发了 `ready_for_adversarial` 或 `blocked`，但循环并没退出。每个都对应一种强制续跑条件——系统覆盖退出信号、改为注入一条 guard note：

-   **CONFIDENCE\_FORCE**：`status=COMPLETE` + `open_questions=[]` 但 `confidence < 70`——模型嘴上宣布做完，却用那个数字悄悄标着不确定。强制最多再跑 3 轮（这几轮算在 20 轮预算之内，不是额外加的）。
-   **CAUSAL\_CHAIN\_FORCE**：`status=COMPLETE` + `confidence ≥ 70` 但 `causal_chain_complete != true`——一个停在症状层、没追到根因的结论。强制最多再跑 3 轮（同样算在 20 轮预算内，不额外加）。
-   **G\_BLOCKER\_FORCE**：`status=BLOCKED`，但可达的技术源被跳过了，或者是靠推断而非真实报错标成"不可用"。调查没真正把它能够到的源穷尽完，就不准以 blocked 退出。最多 3 次续跑（同样算在 20 轮预算内，不额外加）。

这三道合起来，挡住了最常见的三种过早退出：听起来很自信的不确定、停在症状层的"完成"、凭空捏造的"不可用"。

## 对抗式评审：内置的红队

一旦 oncall 调查流水线以置信度 ≥ 70 宣布自己做完，控制权就交给**另一次独立的 Claude 调用，扮演对抗式 reviewer 的角色**。

这位 reviewer 看不到调查者那份 prompt——它拿到的是分析正文，外加一份明确的 15 维评判清单（D1 到 D15：假设有效性、证据链、因果推理、备选假设等等）。它产出一份结构化裁决：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/09.webp)

对抗式评审有三种退出路径：

1.  **发现致命问题** → 致命 note 作为 `pending_guard_notes` 注入*下一轮*调查迭代，循环重启。调查者现在拿到了明确的"reviewer 认为你漏了 X"反馈。
2.  **只有轻微问题、且是第一遍** → 带着这条轻微批评再回跑一轮。
3.  **连续两遍只剩轻微问题，或者一遍干净通过** → 以 `complete` 退出。调查发布。

把对抗式评审做成**一次带不同 prompt 的独立 Claude 调用**，是有意为之。同一个模型换个角色，能抓住单 prompt 迭代漏掉的相当一部分自我吹捧。我们见过同一个 case，从调查到它的对抗式评审之间，置信度声称被下修——从 90 改到 65，reviewer 精准戳中了调查者一笔带过的那个假设。

还有第二种结构上不同的评审，和这个并行跑：**红队评审（G5）**。对抗式 reviewer 读的是完整调查叙事，红队 reviewer 只看两样东西：原始问题陈述，和最终结论。中间推理、证据轨迹、之前的轮次，一概不给。它产出一个独立假设，并标出"调查找到的"与"一个冷读者本会预测的"之间的任何缺口。

目的是认知隔离，不是覆盖率：

- 对抗式评审检查调查是不是结构上完整（D1–D15）
- 红队检查的是：不靠调查的框定，这个结论本来够不够得着——推理是真正扎根于问题，还是调查锚定在某个早期发现上、结论是从那个锚点而非证据里长出来的

两者都是独立的 Claude 调用，两者都必须通过，case 才会被标 `COMPLETE`。

## 五种诚实的退出原因

调查循环可能因五种原因结束，触发了哪一种会被记录、并回显给操作者：

```
exit=complete       confidence=89%  iterations=12  elapsed=13min
exit=blocked        confidence=62%  iterations=4   elapsed=4min    (cannot proceed without info)
exit=stalled        confidence=58%  iterations=7   elapsed=22min   (two rounds, zero new facts)
exit=degrading      confidence=42%  iterations=5   elapsed=8min    (dropped >15% twice)
exit=timeout        confidence=73%  iterations=18  elapsed=50min
```

这五种里，`blocked` 是最需要操作者动手的：agent 已经在 `unchecked_sources` 或 `adjacent_problems` 里登记了明确的阻塞点，需要人来判断解锁。另外四种是技术性失败模式，harness 的处理方式是把它们浮出来，而不是盲目重试。

## 工具访问：Claude 从内部能够到什么

工具边界在这里很要紧，因为置信度门部分依赖于 Claude 真正能够到什么——一道说"去查 unchecked\_sources"的门，只有在能查它们的工具确实可用时才有意义。每条流水线启动一个 Claude Code 子进程，带 `--permission-mode bypassPermissions`。prompt 里**没有内联的工具白名单**；边界由 [MCP（Model Context Protocol）](https://modelcontextprotocol.io/) catalog 里实际接好的东西决定。catalog 覆盖五个外部系统：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/10.webp)

有几条约束值得单独点出来，因为它们对行为的塑造比模型卡片更大：

-   **调查流水线里明确禁用 `Bash`。** 调查循环跑 Claude 时不给 Bash 访问（`--disallowedTools Bash`），为的是防 git 竞态——两个 Claude 进程并发改同一个工作树会把状态搞坏。Bash 在 dev-agent 流水线里可用，因为它跑在隔离的 worktree 里。
-   **Slack 写工具被封。** `send_message_to_self` 和 `edit_message_to_self` 在每次流水线调用里都被禁。Claude 没法直接往 Slack 发；所有对外通信都走结构化完成报告里的 `draft_response` 字段，由 harness 在校验完退出条件后代发。
-   **`bypassPermissions` ≠ 实践中无限制。** 每条流水线的框定 prompt 编进了硬规则：`ANALYSIS.md` 可编辑，但 `message.json` 不可变；写到 `case_dir` 之外的内容，运行后用 git diff 自动回滚；写生产数据库必须等操作者按命令逐次显式确认。安全在框定层 + 运行后的边界检查，不在工具调用层。
-   **级联回退写在 prompt 里，不是硬编码。** `search_datadog_services` 返回权限不足时，prompt 指示 agent 转去用 `mcp__glean_default__search`、拿 Jira 关键词当代理来查。这条回退链是 SOP（标准操作流程）落地的一部分，不是运行时的一部分。
-   **只读工具并行，写工具串行。** streaming 模式的子进程边发生边解析 `tool_use` 事件，对读操作（Datadog 查询、Glean 搜索、文件读）并发派发，最多同时 10 个。任何写——带副作用的 Bash、edit、MR 评论、DB 写——一次一个。这让单轮调查迭代能对日志并行打六个查询，同时仍维持写顺序。

## 同一套骨架，四种尺码

harness 里的每条流水线，用的都是同一套骨架的某个变体：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/11.webp)

| 流水线 | 迭代方式 |
|---|---|
| oncall 调查流水线 | 这套骨架最多跑 20 轮 |
| task 分析阶段 | 最多 8 轮 |
| implement 阶段 | 整体只跑一次，但内部 *build* 步循环三次 |
| dev-agent | 每次拉起跑一次，无内循环；但 mr-monitor 实际上把迭代外置了——每次 CI 挂就拉一个全新的 dev-agent，连续最多三次，再不行就升级给人 |

这种一致性是刻意的。四个循环各不相同，因为它们解决不同问题，但共享同一套词汇：confidence 是个数、退出是个有名字的原因、自我批判是一次独立的 Claude 调用、升级是往原始 thread 里发一条 Slack 私信。操作者学一遍形状，到处都认得出来。

## 自我改进层

这一层把前面引出的那个循环闭上：harness 改自己的底座，由人来评审。两条循环干这事，运行在不同时间尺度上。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/12.webp)

**循环一——单 case 结构修复（gap → PR）。** 每次调查关掉后，`gap_analyzer` 找出 harness 哪儿表现不行：数据源被跳、工具选错、流水线 bug、缺测试、MR 描述格式翻车。缺口按类别和置信度过滤，传给 `finalize_case`。case 闲置满四小时后，`finalize_case` 在专用 worktree 里拉起 `patch_suggester`。结果是一个打向 harness 自身代码库的 PR——改代码文件、prompt 模板，或 SOP 章节。这个 PR 需要人来合并；但不需要人来写。

**循环二——跨 case 行为强化（失败模式 → SOP 建议）。** 每个 case 的 `auto_retrospective` 步会清点触发了哪些失败模式，按类型归类：

- 数据收集失败（调查跳过了可达的源吗？）
- 质量门违规（置信度声称跟证据矛盾了吗？）
- 分析完整性失败（根因有没有一路追到底？）

这些按项目累积进 case registry。一个每周一次的 cron（`knowledge-updater`）统计最近十个 case 里各模式的频率，任一模式越过阈值就生成 SOP 建议：10 次里中 3 次触发一条建议，中 6 次发一条告警。建议很具体：*"'跳过直连数据库查询'本月触发了 3 次——SOP 已更新：推断 schema 之前，总是先跑直连表查询。"* 由人来评审并应用。这条循环刻意不自动应用——会让错误复利的自动决策，是自修改系统里一类已知的失败。

**为什么这让 harness 变得个性化。** SOP 是 harness 对"你团队怎么做调查"的建模。它一开始是通用的。随着 case 累积、失败模式重复，kb-updater 逐渐画出这个项目特有的失败图谱。跑过 20–30 个 case 之后，一个项目的 SOP 长得跟通用模板完全不一样了——它反映的是这个代码库里那些被反复跳过的具体数据源、被反复用错的工具、那些总是没打到点上的调查框定方式。每个项目有自己的 SOP。A 团队的失败教出 A 团队的 SOP，不会污染 B 团队的。

这就是学习能持久的原因：

- 一个上下文窗口只活过一次会话
- 一个合并进去的代码改动，是版本化的、评审过的、测试过的、永久的
- 一次 SOP 更新经操作者评审，被带进未来每一次调查

底座在改进，下一个 case 跑在一套比上一个更好的 harness 上。

## Layer 3 — 持久化状态

agent 编排是逐事件跑的；持久化状态是进程重启、机器重启、或操作者从笔记本切到 CVM 时还活着的东西。harness 的状态分三层，特性差异很大：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/13.webp)

**内存缓存**又小又可重建。活在轮询器 / keepalive 进程里，重启时从本地状态重建。

**本地状态**（`~/.harness/*.json`）是运维映射表。关键文件：

-   `task-thread-map.json`——Slack thread\_ts → JIRA id + 绝对 `task_dir` 路径。
-   `oncall_thread_map.json`——同样形状，但用于 oncall 调查 case。
-   `monitored-mrs.json`——harness 开过的 MR 列表 + 它们的 CI 状态。
-   `patch-pr-state.json`——每个 case 的 finalize 状态（`merged`、`no_gaps`、`failed`……）。
-   `oncall-state.json`——kill switch 标志（Layer 6 细讲）。
-   `socket-dedup`——Socket Mode 近期处理过的消息 ts。

这层是机器本地的、不共享。重启安全，但机器一死就没了。

**git 同步状态**才是可持久层。case 工作区（`case-workspaces/<ticket-id>/`）在每次有意思的状态变更时 commit + push 到一个私有 git 仓库。第二台机器克隆同一个仓库，就能接着同一段对话上下文继续。这就是跨机器会话连续性的来路——操作者从笔记本或远端 dev VM 跟 harness 聊天，都不丢上下文。

我们给 git 同步状态还加了第三个角色：**记忆固化（memory consolidation）。** 长对话会攒下几十个 `followup_transcript_*.md`。活动够多之后，一个子 agent 把它们全读一遍，把蒸馏出的状态重写进 `TASK.md`，之后 workflow 就不再把单个 transcript 喂给后续回合。没有这步，prompt 会涨过模型的上下文窗口。**有了**这步，又冒出一个真实风险：过度压缩、丢信息——我们确实犯过一次，把一份 14KB 的文档压成了 587B。那次事故催生了如今的三道防护：

- 一个 "UPDATE not REWRITE" 的 prompt 模板
- 一个输出体积守卫，拒绝小于输入体积 50% 的写入
- 一个 per-case 锁文件，防并发固化跑成竞态

从单个 Slack thread 查到这个 case 的 TASK.md，这条查找链不长，但值得画出来：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/14.webp)

两个状态文件，同一种 value 形状：一个指向 case 工作区的绝对路径。每个 workflow 最终都收敛到一个 `case_dir`，然后相对它操作。给工作区根目录改名——我们项目中途真干过一次——就是一个迁移脚本，把两张映射表里的绝对路径跟着目录移动一起改写，步调一致。

可持久状态是地基。建在这地基上的操作要是失败了呢？harness 不把失败浮给操作者——它从失败里恢复。

## Layer 4 — 自愈循环

这一层，harness 从"通知器"升级成"自主体"。三条循环，每条都在无操作者介入下闭合。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/15.webp)

前两条完全自主。第三条故意不是——OKTA 刷新要一个绑定硬件的 MFA 提示，所以 agent 把自己停住，在私信里附一份 30 秒 runbook，请人动手。这是设计上的人在环中，不是能力不足的将就。

这些循环有个微妙特性：**重试预算和自我识别。**

- CI 自动修有 per-MR 上限（连续 3 次失败 → 升级给操作者）
- reviewer 自动回复用**作者感知去重**——如果机器人在某条讨论里的最后一条 note 比 reviewer 的更新，循环就不再触发（机器人已经回过了）
- OKTA helper 有 15 分钟审批窗口

没有这些，循环偶尔会进死旋——项目早期，一个挂着抽风集成测试的 MR，在一个下午里走了 80 个空操作的 "fix CI" commit，我们才发现。

## 三层自愈：确定性规则不够用时

上面 CI 自动修和 reviewer 回复这两条，是端到端的恢复流。在这些流的底下，每个可能失败的原子操作（push、rebase、MR 评论、分支重置）都坐落在一套独立的、更小的 3 层架构上，做**逐操作的自恢复**。这跟 Layer 6 那个 3 层**安全**模型是两条不同的轴——那个讲的是克制（agent 被允许做什么），这个讲的是恢复（一个被允许的操作失败了会怎样）。很容易混，值得分清楚。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/16.webp)

**L1——确定性守卫。** 亚秒级、基于规则、永远在线。对 push 来说，就是 `_pre_push_rebase(repo_path, branch)`：每次 harness 要 push 之前，先 `git fetch origin` 再 `git rebase origin/<branch>`。rebase 干净就放行 push；有冲突或 fetch 失败，L1 干净地中止、列出冲突文件、交给 L2。L1 干不了任何有创意的事——这正是要点。规则小到能在脑子里推演，边界紧到每个操作都跑也不花 token 成本。

**L2——agentic 自愈循环。** L1 搞不定时，harness 拉起一个受约束的 Claude 会话（`_self_heal_push_loop`），喂全失败上下文：git 状态、冲突文件、当初要 push 的原始任务，以及一组很紧的红线（绝不强推受保护分支、绝不丢未推送的 commit、绝不盲选某一边冲突）。三条约束让它安全：

1.  **有界重试。** 每次事故 ≤3 次尝试，每个 Claude 回合 4 分钟超时。Claude 每动一次，harness 都重跑真实的 `git push`——Claude 自认为冲突解没解，从不被信；`git push` exit 0 才是地面真相。Claude 想喊多少遍"修好了"都行，但下一次 push 返回非零，循环就把这次记成失败尝试。
2.  **置信度门 ≥ 70。** 一字不改地借自 oncall 调查流水线的 `oncall_run` 配置：Claude 每回合给自己打 0–100，低于 70 就必须撤退、不准动手。低于下限的自评，碰不到工作树。
3.  **诚实性不变量被守住。** L2 成功，意味着一个真实 commit 落到了 origin。下游那道回复门（给 reviewer 的 "Done — updated" 回复）依然硬接在 `git push` exit 0 上，而不接 Claude 对自己成功与否的任何说法。L2 绕不过回复门，它只能给门喂一次真实的 push。

**L3——升级给操作者。** L2 撤退了（低置信度）或尝试用尽 → harness 把**完整的 L2 轨迹**私信给操作者：每次尝试、Claude 的每个动作、每个置信度评分、最终的 `git status`。操作者从一份全程被仪表化的状态接手，而不是从"出了点问题"接手。时间尺度：小时级，不是秒级。

两条值得浮出来的原则：

-   **L2 是逐事故触发，不是永远在线。** L1 每次 push 都跑，因为它便宜。L2 烧 token，所以只在 L1 已经失败时才触发——成本被罕见情形门控，而不是被常见情形。
-   **以线上为地面真相。** 不管 Claude 自认为干得怎样，校验都重跑真实操作。push 看 `git push` 退出码，测试修复看跑真测试，MR 回复看回复门检查 push 退出码。agent 的自评从来不是地面真相——harness 都拿线上结果再核一遍。

这套模式不止用于 push。任何 harness 操作，只要有清晰的确定性恢复（L1）、受约束的 agentic 恢复（L2）、和干净的升级面（L3），都能照同样方式包起来。正是这个形状，决定了一个自主循环是能从已知失败里恢复，还是世界一变就当场要找操作者。

## Layer 5 — 可观测性

harness 往四个可观测性面写东西，每个面对应一类不同的消费者：

| 面 | 内容 | 谁看 / 为什么 |
|---|---|---|
| **实时 Slack 更新** | agent 跑着时，case thread 里一条占位消息轮播状态（`Reading TASK.md` → `Searching codebase` → `Drafting patch`），最后落到 `Done` 或 `Failed: <reason>` | streaming reporter；只编辑同一条 Slack 消息，所以 thread 不会被刷屏 |
| **结构化日志** | `~/.harness/logs/agent.log` 和 `~/.harness/logs/error.log`；每次拉起、每次链派发、每次重试都记一条结构化事件 | 诊断时拿来 grep 的就是它 |
| **终态事件的 Slack 私信** | MR 开了、MR 合了、OKTA 过期、CI 升级 | 一等公民的操作者通知，相关的话挂在 case thread 下 |
| **系统日志** | `~/.harness/logs/system.log`，记横切事件：部署完成、watchdog 动作、同步状态 | 跨切面排查 |

我们明确选择**不**做 dashboard。Slack 和 grep 就是控制台。dashboard 会多出第三个要时刻保鲜的面，而它一旦发霉，就是工程师最先弃读的东西。streaming-Slack-消息这套模式提供的运维信号已经够用，我们从没想在上面再加个 UI。

## MCP 健康监控

harness 依赖五个外部 MCP（Slack、Datadog、Jira/Confluence、Glean、GitLab）。每个有自己的 OAuth token、自己的刷新节奏、自己一套不透明的失败模式。不处理的话，MCP token 过期会表现为一个莫名其妙返回"我没有那个工具的访问权"的 Claude 会话，操作者那边看不到任何报错。

一个小服务——**mcp-watchdog**——补上这道缺口。每 10 分钟它轮询每个 MCP 的健康端点、尝试一次空操作工具调用，失败就：

1.  尝试一次静默 token 刷新。
2.  刷新还失败，就给操作者发 Slack 私信："Atlassian MCP needs reauth — click here."
3.  连续两次断连后（即 `DISCONNECT_CONFIRM_THRESHOLD`），升级——在最近一张工单上加一个 `:warning:` reaction。

没有这条循环，一个过期 token 表现为"agent 今天变蠢了"——这类失败诊断起来痛苦得离谱，因为 agent.log 里没一句说"token 过期"，它只说"工具返回空结果"。把认证健康做成一等公民的可观测性面，是我们本该更早就建的一步。

## Layer 6 — 人在环中的控制

最常用的面，也是我们一开始最没建够的那个。所有操作者交互都走一条通道——自我私信——组织成六个命令族：

| 命令族 | 用法 | 作用 |
|---|---|---|
| **Task** | `admin: task <jira>`，`--auto` 走无人值守执行 | 触发完整的 analyze → implement → MR 流水线 |
| **On-call** | `admin: oncall-run <url>` 从一个 Slack thread 起调查；`oncall-toggle` 开关自动监控 | 启动 / 切换调查 |
| **MR 控制** | `admin: mr rescan` 重轮询某个 MR；`mr pause/resume` 门控那个 MR 的自动回复循环 | 管单个 MR |
| **Pause** | `admin: oncall pause [duration]` 静默自动派发；`oncall resume/status` 解除或查状态 | 暂停 / 恢复 |
| **Finalize** | `admin: finalize <case>` 收口 gap 报告、对 harness 开改进 PR | 收口结案 |
| **Registry** | `admin: register-thread` 手动把 Slack thread 关联到一个 case；`close-case` 标记完成 | 注册 / 结案 |

最久才调对的，是 **pause kill switch**：

- **第一版**：暂停了一切——连 admin 命令自己也暂停了。结果按下暂停的操作者，反倒没法再恢复它。
- **第二版**：暂停得太窄，操作者的意图（"别吵了"）没被尊重到。
- **现在这版**：校准到一个精确区分——**pause 只静默由别人活动驱动的自动派发**，但绝不静默操作者自己的 admin 命令或 thread 内交互。轮询器里的派发点干净地分成两组：代表别人活动的自动行为（暂停时门控），和代表操作者自身意图的（永远在线）。

这个校准是硬碰硬学来的：第一版 pause 部署后没多久，一条 reviewer 评论的自动回复在另一个 case 上悄悄触发了，我们没注意到——因为操作者的 pause 看起来在生效。现在的设计把 pause 当作**定向静默**——让系统安静下来，同时保住操作者的能动性。

一张图把这个切分讲清楚：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/17.webp)

规则就一句：**操作者自己的动作从不被静默；别人的活动暂停时门控。** `admin: oncall pause` 管的就这么点事。

## 三层安全

pause kill switch 是最显眼的安全面，但不是唯一的。另外两层在无任何运行时配置下强制安全——一层在代码里，一层在部署约定里：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/18.webp)

Slack 频道白名单比听起来重要得多。项目早期我们差点出事：一个配错的 Slack token 拥有内部团队频道的写权限——意味着 agent 本可以直接发消息给队友、完全绕过操作者，而团队没法把它跟真人消息区分开。那之后，每一条对外的 `chat.postMessage` 在发送前都会拿目标频道去比白名单。唯一允许的目标，是操作者的自我私信（操作者本人跟机器人的那条 DM 的频道 ID——一个硬编码的单值）。agent 但凡想直接给队友发消息，就必须把内容浮回给操作者，由操作者决定是否转发。这是我们反复回到的一个架构选择：**agent 只通过操作者的信箱对外通信。**

## 六层之外：让它真正能上生产的那些零件

上面六层是承重结构。还有几个机制落在层分类之外，却同样要命——以我们的经验，这些零件正是"demo 好看的 harness"和"真能不间断地跑的 harness"之间的分界。

## 单个 YAML 搞定多项目路由

harness 今天服务两个项目，各有不同的运维规则，加第三个项目要改的代码量是零——只需在 `agent-config/projects.yaml` 里加一条：

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

一个 `resolve_project()` 函数按优先级给每条进来的信号路由：**Jira prefix > Slack channel > 默认**。每个项目有自己的 SOP 文件、KB 文件和目录树。调查 prompt 的组装会根据信号解析到哪个项目，自动挑对应的 SOP 和 KB。正是这个机巧，让单个 harness 实例服务两个不同团队、两套不同的运维规则。

这套模式可推广。任何在同一个 harness 上跑多个产品团队的组织都能照搬：声明式配置，不分项目改代码。它还逼出一种干净的纪律——任何专属于某一项目的东西，都必须活在它的 SOP 文件里、而不是代码里，因为代码是跨项目共享的。

## 跨机器会话连续性

操作者从两台机器上干活：一台笔记本，一台数据中心里的长寿命 CVM。同一个 Claude 会话需要能从任意一台恢复。这很难，因为 Claude Code 默认把会话状态持久化到本地磁盘——从另一台机器重启，这个会话就是隐形的。

harness 用一个 **git 后端会话存储**来解：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/19.webp)

机制分三块：

1.  **会话被 git 追踪。** 每个会话的 `.jsonl` + `.meta` 文件在每次有意思的状态变更时被 commit + push 到一个私有仓库。仓库是真相源。
2.  **一个 5 秒间隔的后台 fetcher** 在每台机器上跑，保持本地副本最新。它预拉上下文，这样 Claude 会话恢复时不必每次敲键都付网络延迟。
3.  **按 CWD 查会话 UUID。** Claude 在某目录启动时，`session-start.sh` hook 扫本地 `.meta` 文件，找 CWD 匹配的那个、恢复对应 UUID。同一个会话 UUID 会在操作者当前所在的那台机器上恢复。

同步 hook（`user-prompt-submit.sh`，每条 Claude prompt 之后触发）按严格顺序做三步操作，用 `flock` 串行化：

1.  **Fetch + reset。** `git fetch origin main && git reset --hard`——现在本地 `active.jsonl` 装的是*另一台*机器的 transcript。
2.  **覆盖前先抽增量。** 读 watermark（我们已经从另一台机器注入了多少行），只抽出此后的新行，存到临时文件。这步必须发生在第 3 步*之前*——早先有个 bug 把两步弄反了，结果注入总是把本机的 transcript 又读回给自己。
3.  **推自己的 transcript。** 把本地 Claude 会话 transcript 拷到 `active.jsonl`，commit、push。然后把 watermark 更新成本地行数。

flock 释放后，第 2 步那份增量作为系统消息注入 Claude："另一台机器自上次同步以来加了这 N 条消息。"下一条 prompt 就同时看到两台机器的上下文。

一个 `.linecount` 文件把真实总行数和文件体积分开追踪，因为 transcript 被截断推送后，`wc -l` 会给错答案（push 前我们截到 1,000 行，以避开 git 25 MiB 上限）。没有 `.linecount`，watermark 会在每次截断时重置，导致注入把上千条已经看过的行又重发一遍。

`flock` 串行化的 git 操作，防住了并发 push 引起的 `index.lock` 损坏（有一次会话启动卡住、又撞上并发的 meta-enricher 跑，我们就中招了）。一个节流（可配，默认 30 秒）在快速敲键时跳过完整同步，避免逐字符的 git 延迟。

"git 当状态存储"这套思路，才是这里最值得拎走的更大点。我们把 git 不只当代码仓库，而是当一个 [**类 CRDT**](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type)**的、可持久的 agent 状态日志**，操作者的两台机器是最终一致的读者。

有个机制跟跨机器连续性紧密相关：记忆固化。memory\_consolidate 这步（把 transcript 重写成干净的 TASK.md/ANALYSIS.md）不按固定计时器触发。一个独立服务——**`consolidate_trigger`**——拿四个信号去评估每一回合，任一命中就触发固化：

- transcript 里出现一个 commit SHA 或一个决策动词
- 一个刚刚关掉的评审 thread
- 一个 case 工作区累积的 transcript 超过 5KB
- 距上次固化已过至少一小时

原则是：信息在一个决策做出之后就停止累积，而不是在固定时长之后。这四个信号合起来近似回答了"对话现在是不是处在一个自然的休止态？"——而不用操作者手动去标。

## worktree 不是实现细节

前面说过"一切都在 worktree 里发生"。这点值得展开，因为它代表的架构选择是真正承重的。

最初的设计是让 agent 吐一份 diff，harness 在主仓库 checkout 里调 `git apply`。这有两种反复咬到我们的失败模式：

1.  **脆弱的 diff 头。** Claude 吐的 diff，行号在生成时是对的，但一个并发 commit 落地后就错了，`git apply` 会拒掉整个补丁。
2.  **主树卡死。** 如果 `git apply --check` 在半途失败，主仓库会停在半应用状态。下一个在同一仓库上跑的 case 会继承这棵脏树。

修法是 `worktree_manager` 服务。任何需要改代码的 agent 跑之前，manager 先在 `~/.worktrees/case-<case_id>-<stamp>/` 建一个沙箱化的 worktree，从一条干净分支 checkout 出来。Claude 在那个 worktree 里用它原生的 Edit/Write 工具操作——没有 `git apply`，没有 diff 文本来回。agent 干活的方式跟人类工程师一模一样：开一条新分支、新 checkout、改、commit、push。

MR 合并或关闭时，worktree 被拆掉。agent 半途失败的话，受影响的只有那个 worktree——主 checkout 和其他并发 case 毫发无损。

这正是在单台机器上**安全并发 agent**得以可能的根本。没有 per-case worktree，同一仓库上的两个 case 会在文件系统状态上竞态；有了它们，隔离是天生的。

## Dogfooding：harness 自己修自己的 PR

harness 的源码住在一个私有 GitHub 仓库。当 gap-report → finalize → MR 这条循环对 harness 自身产出一个改进 MR 时，这个 MR 有它自己的 CI：pytest、linter、Python 语法检查。harness 改进 MR 上的 CI 挂了，一个独立服务——**pr-ci-fixer**——就接手，方式跟 mr-monitor 接手客户 MR 上的失败一模一样。

结果是：**harness 不光自愈它为工单写的代码，也自愈它为自己写的代码。** 一个加新重试预算的 MR，可能因为这预算搞挂了某个现有测试而 CI 失败；pr-ci-fixer 注意到、在 worktree 里拉起一个 Claude 会话、打补丁、重推，人类 reviewer 看到的是一个绿色的、可以直接合的 MR。这条循环在真实的 harness 改动上闭合，我们不止见过一次。

在 harness 自己的源码上 dogfood 这条循环，是"这到底是不是真生产"的承重测试。如果 agent 能让自己的 PR 上自己的 CI 保持绿，那它大概也准备好被托付客户的 PR 了。

## 测试架构：五层对五类失败

harness 有 4,656 个测试函数，分布在 126 个文件里。脱离一个理解框架，这个数字会误导人——因为不是所有测试都等价。

L0–L4 框架按每个测试隔离的对象，把它归到五层之一：

-   **L0（纯边界）：** 无 mock、无文件系统、无 IO。测纯逻辑——门函数、JSON 抽取、置信度天花板算术。L0 通过的测试，保证正确，与部署环境无关。
-   **L1（文件隔离）：** tmpdir 里的真实文件系统、真实多线程，不 mock pathlib。抓 L0 设计上抓不到的并发写竞态和状态文件原子性 bug。
-   **L2（服务隔离）：** 真实子进程执行 + mock 掉的外部 API（Slack、GitLab、Claude）。抓服务组件之间的集成 bug——派发路由、payload schema、状态文件流转——L1 抓不到，因为 L1 只隔离文件系统行为。
-   **L3（沙箱）：** 真实 git 仓库（走子进程）、假的 Slack 和 Claude 响应。针对一个真实的 git 对象模型，测完整 workflow 序列——任务注册、implement 阶段启动、待确认态。
-   **L4（集成矩阵）：** 六个子类，覆盖 actor 契约（L4a）、admin × 角色矩阵（L4b）、跨运行状态一致性（L4c）、服务间链（L4d）、外部 actor 角色（L4e）、方法论缺口（L4f）、以及时序排列（L4h/L4i）。
-   **L4g（生产重放）：** 13 个测试，把具体的生产事故复现成回归测试。它们是在生产里发现一个 bug *之后*才写的——测试编进触发该 bug 的精确 actor、上下文和状态，并永久保留以防回归。

L4g 这些测试写起来最贵，留着最值。它们把那些不复发就看不见的失败模式，编进了机构记忆。

## 反 AI 味的风格指南

harness 开出的每个 MR，提交前还要过最后一道过滤：一份专门为了让输出**不像 AI 生成**而存在的风格 SOP。规则很直白：

-   **零 AI 署名。** 不要 `Co-Authored-By: Claude`，不要 "Generated with Claude Code"，commit 信息里不要 `[bot]` 后缀。MR 是团队的，不是模型的。
-   **不要填空式标题。** "What this MR does / why we need it" 这类模板被禁。MR 描述写成连续散文，配一个具体的 `Changes:` bullet 列表，就像工程师手写的那样。
-   **不要内联加粗标签结构。** `**Root cause:**` 后面跟一段，禁。`**Fix:**`、`**Risk:**` 同禁。标签式加粗是最稳的 AI 味破绽。
-   **工作证据必须可命令复现。** 不是"本地验证过"——而是 `pytest tests/test_foo.py::test_bar -v`，附真实命令和相关的 3 行输出。

这份 SOP 在补丁生成 prompt 里、写 MR 描述的那个点注入。它独立于 prompt 模板，因为它是**操作者策展的**：SOP 可以随时改，下一个 MR 就吃到新规则，不用重启 daemon、不用改代码。Gap 循环甚至能对 SOP 本身提改动——关于"agent 该怎么写 MR"的元规则，最后变成打向那份元规则文件的 MR。

为什么这件事比听起来更要紧：没有什么比团队意识到"每个 MR 描述都一股 AI 味"更快地杀死他们对自动 MR 流水线的信任。这股味一旦在了，合并门槛就抬高，agent 真正的信号被埋掉，操作者开始想手动重写描述——而那就把整件事的意义给毁了。风格 SOP 让 MR 描述跟人写的混在一起难辨真假，而这正是让循环持续可用的关键。

## 端到端走一遍：一张工单，一个 MR

一张真实工单到来时会发生什么。时间是约数。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/20.webp)

端到端，对一张中小型工单，这条循环通常在 **30–90 分钟挂钟时间**内跑完。操作者的总投入一般不到 5 分钟：确认分析、可选地引导一下含糊的决策、可选地点一下合并。

调查本身会累积。跑过 30 个 case 后，harness 攒了 30 个 `gap_report*.md`，每个都指向 LLM 认为可以改进的某处。一个双周一次的 `admin: gap-patterns` 把它们聚成反复出现的失败模式，下一次 finalize 把最靠前的几个转成打向 harness 仓库的 MR，于是 harness 改进自己。这就是自我改进循环的实体形态。

## 和别家 harness 的对比

好几个别的系统也是 harness 这个形状。我们这套跟它们重叠在哪、又分岔在哪：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/21.webp)

我们**做得格外深**的地方：

-   **预承诺假设锁**——Claude 在跑任何一轮调查迭代之前，先承诺 ≥6 个候选假设，防事后合理化。我们找过的商业 harness 里，没有一个出货等价物。
-   **19 函数质量门框架 + 机械置信度天花板**——confidence 是一个硬算术上限（`1.0 − open_q×0.08 − unchecked×0.05`），不只是一个自报的数。三族门（逻辑一致性、结构完整性、断言天花板）外加逐维对抗惩罚。我们找过的商业 harness 里，没有一个出这个。
-   **改 harness 自身代码库的自我改进**——gap\_reports → 过滤后打向 harness 自己代码/prompt/SOP 的 PR。最接近的学术等价物（SICA，arXiv 2504）拿到了基准提升，但它是离线研究；没有生产系统出货自主的 harness 代码修改。
-   **项目级 SOP 个性化**——每个项目的 SOP 基于自己的失败模式频率演化。跑过 20–30 个 case 后，SOP 反映出领域特有的调查失败。没有别的生产 harness 实现这点。
-   **带显式交接的多 agent 编排**——`oncall_run` → `case_followup` → `gap_analyzer` → `finalize_case` → `dev-agent`，每个都有定义好的交接 schema 和隔离的 worktree，而不是一个 agent 包打天下。
-   **跨机器状态同步**，靠 git 后端的 watermark 注入——同一个 Claude 会话在笔记本或远端 VM 上带全上下文恢复，不用手动同步。
-   **内置 CI 自动修 + reviewer 自动回复循环**——只有 Devin（商业）出货可比的内置自愈；其他每个框架都得自己接线。

我们**刻意做得薄**的地方：

-   没有正式的**评估 harness**。我们不在基准上给 LLM 打分。生产反馈就是 eval。
-   没有 **dashboard**。Slack + 日志。
-   没有**成本计**。token 用量松散追踪，但不做成 dashboard 浮出来。成本由人在环中的确认步骤兜住上界。

## 展望

harness 还在演化——自主循环的覆盖缺口、成本计量、正式的 eval harness——没一个是拦路虎。系统已经在做它被造出来要做的事：把 Slack 信号变成合并的代码、整夜处理 reviewer 反馈、自愈 CI、把自己的改进反哺进自己的下一版。

更深的一课是那个元循环。harness 上的每道疤，都是因为一张真实工单去碰它才被抓到的。建造循环和运营循环是同一个循环，只是不同时间尺度。生产信号回流到 harness 自己的源码，agent 提一个修复，人评审并合并，下一类 bug 离被修剪掉又近了一步。

系统没完成。一个自以为完成了的生产 harness，就是一个停止攒疤的 harness——换句话说，它已经不再跑在真实负载上了。

> **先把循环建起来，再让它自主，顺序不能反。** 因为一个没有反馈路径通向自己底座的自主系统，不过是一台以更快速度、更大规模地交付同一套错误的机器。
