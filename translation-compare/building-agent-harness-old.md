---
title: "搭建生产级 Agent Harness：把 Claude Code 改造成多 Agent 工程流水线"
author: Messi Li
url: https://licaomeng.medium.com/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline-1db4e242d08a
translated: 2026-06-03
excerpt: 单独一个编码 agent 就是一颗泡在罐子里的大脑。能思考、能生成代码、能调用函数——但回不了你凌晨三点发来的 Slack 私信，重试不了一个失败的 CI 任务，修不了它自己那个刚开的 MR 上冒出来的合并冲突，也记不住昨天某位评审提的问题至今还没人答。
tags:
  - LLM
  - AI Agent
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/01.thumb.webp
---

# 搭建生产级 Agent Harness：把 Claude Code 改造成多 Agent 工程流水线

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/01.jpg)

单独一个编码 agent 就是一颗泡在罐子里的大脑。能思考、能生成代码、能调用函数——但回不了你凌晨三点发来的 Slack 私信，重试不了一个失败的 CI 任务，修不了它自己那个刚开的 MR 上冒出来的合并冲突，也记不住昨天某位评审提的问题至今还没人答。

让这颗大脑真正有用的，是 **harness**：包在 LLM 外面的运行时脚手架，给它装上感官、双手和记忆。事件接入、agent 编排、持久化状态、自愈循环、可观测性，再加上一块给人类操作员的控制面。本文讲的就是怎么搭这么一套——不是研究演示，而是一套生产系统：盯着一个 Slack 频道，对五个内部仓库开 MR，通宵处理评审意见，CI 跑红了还能悄悄自愈。

我们把它搭成了一套内部系统——基于 [Claude Code](https://claude.ai/code) 的多 agent 流水线，至今已连续运行好几个月，从 Slack 接入信号、分派真实的工程活。本文是自上而下的视角：它做什么、各组件如何咬合，以及那些塑造了设计的生产伤疤。

**本文目录：**

-   [为什么要 harness，而不只是一个 agent](#why-a-harness-not-just-an-agent)
-   [我们的 harness 做什么](#what-our-harness-does)
-   [Harness 必须补上的三道缺口](#three-gaps-a-harness-must-close)
-   [第 1 层 —— 事件接入](#layer-1--event-ingestion)
-   [第 2 层 —— Agent 编排](#layer-2--agent-orchestration)  
    ↳ [自主 agent 循环细节](#the-autonomous-agent-loop-in-detail)  
    ↳ [自我改进层](#the-self-improvement-layer)
-   [第 3 层 —— 持久化状态](#layer-3--persistent-state)
-   [第 4 层 —— 自愈循环](#layer-4--self-healing-loops)
-   [第 5 层 —— 可观测性](#layer-5--observability)
-   [第 6 层 —— 人在环路的控制](#layer-6--human-in-the-loop-control)
-   [六层之外](#beyond-the-six-layers-the-pieces-that-make-it-production)
-   [端到端走查](#end-to-end-walkthrough-one-ticket-one-mr)
-   [与其它 harness 的对比](#comparison-to-other-harnesses)

## 为什么要 harness，而不只是一个 agent

2023 年 ChatGPT 插件刚出来时，很多团队试了最显而易见的玩法：给 LLM 套个聊天界面，配几个 function-call 工具，就管它叫"工程 agent"。这套路在演示里勉强能跑，到生产环境就从没成过。三种失败模式头一周就会冒出来：

1.  **操作员一关标签页，上下文就丢了。** 记忆得熬过单次聊天会话。一个真实的工程任务跨好几天：建 JIRA、拉分支、起草、评审、处理评审意见、CI 通过、合并。没有哪一次 LLM 调用扛得住这么多状态。
2.  **外部一变化，它没法响应。** 评审下午四点贴了条评论。CI 在第三次提交时挂了。队友在串里回了话。Agent 得在这些事件上醒来，而不是无限轮询，或者干等用户重新发提示。
3.  **它没法从自己的失败里恢复。** Agent 推了个提交；CI 崩了；操作员第二天还得回来重新解释一遍这次失败。又或者 agent 的 Okta（一个身份管理平台）会话半路过期，整条流水线无声无息地死掉。这两种每天都在发生。代价就是：工程师在晚上和周末被叫起来，处理本该 agent 搞定的活。

解决这些就是 harness 层的活。Harness 正是 `claude.ai` 聊天标签页和这类系统之间的分水岭——Claude Code 本身、Cursor 的后台 agent、Cognition 的 Devin，还有我们做的这套。大脑（模型）在这几者之间基本可以互换——变的是包在它外面的 harness。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/02.webp)

大脑只通过 harness 跟世界对话。让 LLM 表现得像个系统的工程活，几乎全是 harness 在干。

## 我们的 harness 做什么

起点是个具体问题：一个 Slack 频道——就叫它 `#support`——每周积压约 30 条工单，每条都需要先调查、再（通常）在五个仓库之一里改代码。每条工单都跨好几天。评审在 MR 上留评论。CI 抽风。有时一番调查下来才发现是上个月某条的重复。扛这摊活的团队晚上周末都在被呼叫，而活儿本身没见好转——同样形态的工单每个周期都卷土重来。

Harness 有三项职责，按介入深度排列：

1.  **调查**——`#support` 里出现新工单时，顺着串往下读，跨仓库收集上下文，把结构化分析贴回同一条串。
2.  **修复**——操作员批准后，开一个 MR（Merge Request）提出改动，处理评审意见，盯着 CI，CI 挂了就修，一路推到合并。
3.  **自我改进**——每个结案都会反哺：一旦出现反复出现的模式（比如同一区域的五条工单都动到同一份配置），就生成缺口分析，并对 harness 自身提出改动方案。

第三项才是把它和花哨的工单机器人区分开的地方。Harness 的底料就是它自己的源代码，而 LLM 通过走人工评审的 PR，对这份底料有写权限。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/03.webp)

六个逻辑层，下面逐一讲。

## Harness 必须补上的三道缺口

整套 harness 围绕一个洞见：**一个无状态、被动响应的 LLM，不是生产系统。** 这六层之所以存在，是为了补上裸 LLM 自己补不了的三道缺口。

第一道缺口是**响应性**：LLM 没法自己醒来。第 1 层（事件接入）补上它——Slack 提及、GitLab CI 结果、PagerDuty 呼叫，全都汇入一条统一的分派队列。LLM 从不轮询，它是被调用的。

第二道缺口是**持久性**：LLM 在会话之间会忘事，在机器之间会丢上下文，也分不清"我已经处理了"和"那个提交真的落到 origin 上了"。第 3 层（持久化状态）补上它——内存负责进程内去重，本地 JSON 存运维映射，git 同步的工作区存可持久化的案件状态。跨机器的会话连续性（后文细讲）是同一思路的延伸：状态活在 git 里，而不是活在某个进程里。

第三道缺口是**质量**：没有结构约束，一个 LLM 推理循环会从薄弱证据里写出笃定的结论，在没有穷尽可达来源的情况下就宣布调查完成，还会生成第一句话就一股 AI 味的 MR 描述。第 2 层（agent 编排）和调查循环一起补上这道缺口——结构化输出契约、每轮迭代上的质量门、循环之后的对抗式评审，以及把"改代码 → CI → 合并"这条外环闭合的自愈循环。

把这几层串起来的，是**复利特性**：每个结案都让下一个更快。一份批准的调查会变成知识库条目。一个批准的 MR 会变成缺口分析器能引用的模式。一个结清的案件会对 harness 自身开一个改进 MR。系统一个周期一个周期地，收窄那些它处理得不好的工单类别。

## 第 1 层 —— 事件接入

Harness 需要在三类外部信号源上醒来：

-   **Slack 消息**——频道提及、串内回复、自我私信的管理命令。
-   **GitLab 活动**——MR 评审评论、CI 流水线结果、流水线失败。
-   **PagerDuty 告警**——当一次值班呼叫引用到 harness 见过的工单时，把上下文浮现出来。

三者的延迟和可靠性特征各不相同，所以每一类我们都用不同的入口。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/04.webp)

这里有两件教科书描述不会展示、但很实用的事。

**最终一致性。** Slack 的 `conversations.history` 端点会滞后 1–30 秒。如果我们的轮询器看到空结果、就天真地往前推游标，那么任何 `ts` 落在那个窗口里的消息都会被永久跳过。我们的做法是**空轮询时冻结游标**，从同一个最旧的时间戳一直重试，直到至少有一条消息回来。重试之间的重复处理，靠一个按 message-ts 去重的集合滤掉。

[**Socket Mode**](https://api.slack.com/apis/socket-mode) **+ 轮询器双保险。** Socket Mode 给串内回复带来亚秒级延迟。但 WebSocket 连接会断。轮询器就是安全网——它用一个共享的 `socket-dedup` 文件，兜住 Socket Mode 漏掉的任何东西，保证同一条消息不会被分派两次。没有这层，连接断开期间到达的消息会无声消失——操作员根本收不到它们被漏掉的信号。

mr-monitor 这条 cron 循环则是另一种脾气。GitLab 评审评论不会把事件推进我们的流水线，只能靠我们去轮询。它最棘手的问题原来是**轮询本身的扩展性**：随着 harness 接手越来越多长寿命的 MR，它那条 10 分钟的轮询循环会积累一堆死串游标，每个游标每个周期都要花一次 Slack API 调用。不处理的话，这些会引发持续的 429 限流风暴，把真正的轮询给饿死。我们加了游标自动驱逐：当上游 API 返回 `thread_not_found` 时，该游标就从下一周期的循环里消失。（修复前：每周期 12 次重试，持续好几个小时；修复后：零。）

## 第 2 层 —— Agent 编排

这是 LLM 真正跑起来的地方。Harness 把 worker 拉起成一次性的 [systemd](https://systemd.io/) 单元，每个都携带一份 JSON 载荷，里面指明了一个 workflow + 一个 case\_dir + 一份串上下文。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/05.webp)

每条流水线都是一次对 Claude Code（大脑）的长跑式调用，配上一份精心准备的提示和一个可写的 `case_dir`。case\_dir 是 agent 的草稿空间：里面放着该案件的 `TASK.md`、累积的 `followup_transcript_*.md` 文件、生成的 `gap_report*.md` 文件，以及中间的 JSON 产物。

编排很关键。我们没有一个什么都干的 agent——而是有好几个专门的，每个都在一个明确的交接点上链接起来：

-   `**oncall_run**` 跑初始调查——最多 **20 轮推理迭代**、带质量门——末尾还有一个明确的自我批判步骤。输出：一份 `ANALYSIS.md`（哪里出了问题）和一份 `TASK.md`（要做什么）。
-   `**case_followup**` 是长寿命的对话 agent。操作员可以在某条案件串上私信回复好几天；每次回复都会带着完整对话历史重新调起 case\_followup。关键在于，case\_followup 每隔 N 次回复还会把 `gap_analyzer` 作为子 agent 调起一次 *（自我改进循环见下文）*——正是这条循环产出指向 harness 自身代码的 `gap_report*.md` 产物。
-   `**finalize_case**` 是合并器。当一个案件闲置满四小时，或者操作员跑了 `admin: finalize <case>`，它就收集每一份 gap\_report，滤掉误报，开**一个 MR** 把幸存下来的缺口修复打包进去。这个 MR 针对的是 harness 自己的代码库，而不是原始工单所在的仓库。这就是自我改进循环的具象形态。
-   `**dev-agent**` 是动手的那个。它在每个案件专属的 [git worktree](https://git-scm.com/docs/git-worktree) 里跑，改代码、推送、盯 CI、CI 挂了就修、处理评审意见。三种任务模式：`address_review`、`fix_ci` 和 `task`（最初的实现）。

这张图里有两点并不直观。

第一：**一切都在 worktree 里发生**，而不是在主仓库检出里。多个并发案件可以同时在飞，互不踩脚。轮询器 / dev-agent 分派器在载荷里传 worktree 路径；LLM 从来看不到主检出。

第二：**case\_dir 是唯一真相来源。** `~/.harness/` 里的状态文件（下面就会看到）把一条 Slack 串映射到一个 case\_dir。Agent 永远相对 case\_dir 来操作。这让系统其余部分简单了很多：把 case\_dir 改个名，只要映射跟着更新，每条 workflow 都会跟上。

上面那张编排图展示的是*跑的是什么*。下面这一节解释*怎么跑*——把一条 Slack 命令变成一份完成的调查，靠的是哪些机制。

## 自主 agent 循环细节

"这玩意儿到底干什么"的一句话回答是：**操作员敲一条 Slack 命令，然后一个由迭代推理、自我批判、工具调用组成的闭环就跑起来，无需再干预，直到活儿干完，或者系统老实承认自己进行不下去。** 这句话藏起了几乎所有有意思的决策。本节把它拆开。

Harness 内部跑着四种 agent 循环，各自针对一种不同形态的问题调过参：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/06.webp)

这些是形态。下面是实质——每个机制按运行顺序排开。上面那张图覆盖了它们全部；读完之后那些节点就讲得通了。*（门控代号：* [*G1–G4、N1–N3、N5、A1–A3*](#quality-gates-the-structure-that-makes-confidence-mean-something) *·* [*D1–D15*](#adversarial-review-the-red-team-inside)*）*

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/07.webp)

## 循环之前：预承诺锁

第一次 Claude 调用打响之前，先跑两道防护。第一道是结构性的。

**循环开始前：预承诺锁。** 值班调查流水线做的第一件事——在 Claude 跑任何一轮调查迭代之前——就是要求 Claude 锁定一份 `hypothesis_slate`：至少六个候选解释，外加四到五条用于在它们之间取舍的评估准则。这些被冻结进 `kickoff_precommitment.json`，在整场调查的余下部分都被当成不可变的。

这是整个系统里最重要、也最不显眼的方法论防护。没有它，一个 LLM 推理循环会向"早期证据最支持的那个假设"收敛，然后回溯性地把后续所有证据都框成在印证它。预承诺锁并不阻止收敛——它逼着调查从一个完整的假设空间出发，并记录从那个空间走到某个结论的路径。对抗式评审者（阶段 B）检查的是：给定证据，这个结论是不是最初那份候选清单本会预测出的那个，而不只是结论自身内部是否自洽。

**同样在调查循环开始前：经验锚点（阶段 C）。** 仅在第一轮迭代时，harness 跑一组操作系统级命令，全程不让 LLM 介入——`journalctl` 错误尾巴、磁盘和内存快照、日志文件发现。输出作为原始证据注入第一份调查提示。这让 Claude 在形成任何假设之前，先拿到一份事实性的系统状态快照。它非阻塞（出错就默默吞掉），耗时不到一秒；它的价值在于让 Claude 的第一步推理从真实地面状态出发，而不只是从问题描述出发。

## 结构化输出：完成报告 schema

每轮迭代都以一份可解析的 JSON 契约收尾——`completion_report`——harness 用它来决定接下来发生什么。形状是固定的：

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

这个块前后允许写自由格式的散文，但门控评估的、状态文件持久化的，都是解析出来的那份报告。这是我们找到的、让 LLM 输出在流水线里保持可用的最大杠杆点：**用一点散文自由度，换一份可解析的契约。** 没有这份 schema，下游每一步都得从散文里重新抽取事实。有了它，门控就能作为确定性的 Python 跑起来。

## 质量门：让置信度真正有意义的结构

如果 Claude 不用拿出依据、随手写个 `confidence: 95` 就行，那这个置信度数字就是营销噪音。Harness 通过一组**质量门**强制一份结构性契约，这些门在每一轮迭代的输出上跑，除非证据和置信度对得上，否则不放行。

每次 Claude 出招后，三族门会跑：

-   **G 门（逻辑一致性）** 强制推理在各轮之间不自相矛盾。最清楚的例子：如果这一轮 `open_questions` 涨了、`confidence` 却跟着涨，G1 就触发——分母扩大会让任何上行动作失效。这一族还检查散文里的张力措辞是否登记了一条矛盾，以及相邻问题是否用有效的状态对象（而不是占位字符串）来跟踪。共四道门。
-   **N 门（结构完整性）** 检查调查产物是否完整成形、而非半成品。N1：每个可执行的修复动作都必须配一个 `verify_action`，带上描述、预期结果和时机。提了修复却不说明你怎么验证它，会被当成未完成的活。共三道门。
-   **A 门（断言上限）** 施加 Claude 用散文无法绕过的机械限制。A1 算出一个硬性置信度上限：`1.0 − (open_questions × 0.08) − (unchecked_sources × 0.05)`——不管 Claude 写什么，这道门都用算术把它压住。其它 A 门会在相邻问题仍未关闭时拦下 `COMPLETE` 状态，要求假设登记表非空，并强制至少有一个提出的动作直接对应到根因上。共七道门。

另有两项抽查：**EQ1** 要求根因节点引用硬证据（非 INFERRED），否则接受 60% 的上限。**P7** 每有一个对抗维度被标 FAIL，就把上限降 7%，下限 40%。

门控框架包含 19 个函数。G 门盯*Claude 写了什么*，N 门盯*Claude 漏了什么*，A 门盯*Claude 声称了什么*。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/08.webp)

门控违规会变成 `**pending_guard_notes**`，被前置到*下一*轮迭代提示的开头，挂在 `## SYSTEM QUALITY GUARD NOTES (from previous iteration)` 之下。指令写得很明白：继续之前先处理掉所有这些笔记。这就是那个倒逼机制——门不拦 Claude，而是把它上一轮的违规变成下一轮它第一眼读到的东西。模型礼不礼貌无所谓；推着调查往前走的是结构性强制。

还有一道**上下文钉住**步骤（A7），它在每次提示构建的末尾跑，独立于违规之外：把当前报告里未解决的矛盾和未验证的假设，一字不差地重新挂到下一份提示的底部。这防住了那个有据可查的"注意力衰退"模式——模型在第 3 轮承认了某个复杂之处，到第 7 轮却悄悄把它丢了。

## 把置信度做成一个具体的数字

最重要的设计决策是：**agent 在每轮迭代的结构化输出里给自己打一个** `**confidence: <int>**`，0–100 分制。这个数字不是摆设——它门控真实行为：

-   `**< 70**`：不能退出调查循环。流水线强制最多三轮追加，注入一条守护笔记："你说你做完了，但置信度在标准线以下"。
-   `**≥ 70**` **且** `**status: COMPLETE**` **且没有** `**open_questions**`：门打开，进入**对抗式评审**阶段（红队 / 蓝队）。
-   `**≥ 95**`：门打开，允许 agent **自动执行非破坏性动作**（Jira 评论、状态流转、工单指派）。低于 95，这些动作就停在草稿建议状态，留给操作员。
-   **连续两轮跌幅 > 10%**：循环以 `degrading` 为由退出——agent 在往后退，停下来告诉操作员。

每个案件完整的置信度轨迹都会以 sparkline 的形式写进 `ANALYSIS.md`：`45% → 62% → 78% → 80%`。操作员就看这条 sparkline 来分诊该不该信这个结果。Harness 自己的自我改进循环在决定哪些过往案件值得换个提示重跑时，读的也是它。

## 三种强制续跑机制

上面那张图里有三条分支，退出检查那里触发了 `ready_for_adversarial` 或 `blocked`，但循环并不退出。每条都对应一个强制续跑条件——系统覆盖退出信号、转而注入一条守护笔记的情形：

-   **CONFIDENCE\_FORCE**：`status=COMPLETE` + `open_questions=[]`，但 `confidence < 70`——模型嘴上宣布做完，却通过数字悄悄标了不确定。强制最多再迭代 3 轮（这几轮算在 20 轮预算之内，不是额外叠加）。
-   **CAUSAL\_CHAIN\_FORCE**：`status=COMPLETE` + `confidence ≥ 70`，但 `causal_chain_complete != true`——一个症状层的发现，没有追到根因。强制最多再迭代 3 轮（这几轮算在 20 轮预算之内，不是额外叠加）。
-   **G\_BLOCKER\_FORCE**：`status=BLOCKED`，但可达的技术来源被跳过、或被靠推断（而非真实报错）标成了不可用。调查不能在没真正穷尽可达之物的情况下就以 blocked 退出。最多续 3 轮（算在 20 轮预算之内，不是额外叠加）。

这三者合起来，挡住了最常见的几种过早退出：听起来笃定的不确定、症状层就收工、以及凭空捏造的不可用。

## 对抗式评审：内部红队

值班调查流水线一旦以置信度 ≥ 70 宣布自己做完，控制权就交给**另一次扮演对抗式评审者角色的 Claude 调用**。这次评审内部发生什么的决策树：

评审者看不到和调查者同一份提示——它拿到的是分析文本，外加一份明确的、15 个评估维度的清单（D1 到 D15：假设有效性、证据链、因果推理、备选假设等等）。它产出一份结构化裁决：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/09.webp)

对抗式评审有三个退出条件：

1.  **发现关键问题** → 关键笔记作为 `pending_guard_notes` 注入*下一*轮调查迭代，循环重启。调查者现在拿到了明确的"评审者认为你漏了 X"反馈。
2.  **只有次要问题、且是第一遍** → 带着次要批评再回环一轮。
3.  **连续两遍只有次要问题，或者一遍干净通过** → 以 `complete` 退出。调查发布。

把对抗式评审做成**一次用不同提示的独立 Claude 调用**，是有意为之。同一个模型扮演不同角色，能逮住单提示迭代会漏掉的相当一部分自我吹捧。我们见过同一个案件上，置信度声明在调查和它的对抗式评审之间从 90 被下调到 65，评审者精准地指向了调查者一笔带过的那个假设。

跟它并排跑的还有第二种、结构上不同的评审：**红队评审（G5）**。对抗式评审者读的是完整的调查叙事，而红队评审者只看两样东西：原始问题陈述，和最终结论。没有中间推理，没有证据链，没有先前各轮。它产出一个独立假设，并标出"调查发现的"和"一个冷读者本会预测的"之间的任何缺口。

目的是认知隔离，不是覆盖率。对抗式评审检查的是调查在结构上是否完整（D1-D15）。红队检查的是：脱离调查的框定，这个结论本身是否还够得着——推理是不是真扎根在问题上，还是说调查锚定在某个早期发现上、结论是从那个锚点而不是从证据推出来的。两者都作为独立的 Claude 调用跑；案件被标 `COMPLETE` 之前，两者都必须通过。

## 五条诚实的退出理由

调查循环可以因为五种理由结束，到底是哪一种会被记下来并回示给操作员：

```
exit=complete       confidence=89%  iterations=12  elapsed=13min
exit=blocked        confidence=62%  iterations=4   elapsed=4min    (cannot proceed without info)
exit=stalled        confidence=58%  iterations=7   elapsed=22min   (two rounds, zero new facts)
exit=degrading      confidence=42%  iterations=5   elapsed=8min    (dropped >15% twice)
exit=timeout        confidence=73%  iterations=18  elapsed=50min
```

这几种里，`blocked` 对操作员最有行动价值：agent 已经在 `unchecked_sources` 或 `adjacent_problems` 里登记了明确的阻塞物，需要人来判断如何解阻。另外四种是技术性的失败模式，harness 的处理方式是把它们浮现出来，而不是盲目重试。

## 工具访问：Claude 从内部能够到什么

工具边界在这里很要紧，因为置信度门部分取决于 Claude 实际能够到什么——一道说"检查 unchecked\_sources"的门，只有在真有工具能去检查它们时才有意义。每条流水线启动一个带 `--permission-mode bypassPermissions` 的 Claude Code 子进程。提示里**没有内联的工具白名单**；边界由 [MCP（Model Context Protocol）](https://modelcontextprotocol.io/) 目录里实际接通了什么来设定。这份目录覆盖五个外部系统：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/10.webp)

有几条约束值得点出来，因为它们对行为的塑造，比模型卡片更大：

-   `**Bash**` **在调查流水线里被显式禁用。** 调查循环跑的 Claude 不带 Bash 访问（`--disallowedTools Bash`），以防 git 竞态——两个 Claude 进程并发修改同一棵工作树会损坏状态。Bash 在 dev-agent 流水线里可用，那条流水线跑在隔离的 worktree 里。
-   **Slack 写工具被屏蔽。** `send_message_to_self` 和 `edit_message_to_self` 在每次流水线调用里都被禁。Claude 没法直接往 Slack 发东西；所有对外沟通都走结构化完成报告里的 `draft_response` 字段，由 harness 在校验过退出条件之后再发出去。
-   `**bypassPermissions**` **在实践中 ≠ 无限制。** 每条流水线的框定提示编码了硬规则：`ANALYSIS.md` 可编辑，但 `message.json` 不可变；写到 `case_dir` 之外的内容，会在跑完后通过 git diff 自动回滚；PROD DB 写操作必须等到每条命令实例的明确操作员确认。安全做在框定层 + 一道跑后边界检查，而不是做在工具调用层。
-   **级联回退编码在提示里，不是硬编码。** 当 `search_datadog_services` 返回权限拒绝时，提示指示 agent 转向 `mcp__glean_default__search`，用 Jira 关键词作代理。这条回退链是 SOP（标准作业程序）如何被强制执行的一部分，而不是运行时的一部分。
-   **只读工具并行跑；写工具串行跑。** 流式模式的子进程一边解析 `tool_use` 事件、一边并发分派，读操作最多同时 10 个（Datadog 查询、Glean 搜索、文件读取）。任何会写的东西——带副作用的 Bash、编辑、MR 评论、DB 写——一次只跑一个。这让单轮调查迭代能对日志做六个并行查询，同时还保住写顺序。

## 同一套模式，四种尺寸

Harness 里的每条流水线都用同一副骨架的某个变体：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/11.webp)

值班调查流水线把这套跑最多 20 轮。任务分析阶段跑最多 8 轮。实现阶段跑一次，但把*构建*步骤循环三遍。dev-agent 每次拉起跑一次、没有内层循环，但 mr-monitor 实际上把迭代外化了——每次 CI 失败它就拉起一个全新的 dev-agent，连续最多三次，然后升级给人。

这种一致性是刻意的。四种循环之所以不同，是因为它们解决不同的问题，但它们共享同一套词汇：置信度是个数字，退出是个有名字的理由，自我批判是一次独立的 Claude 调用，升级是一条发回原始串的 Slack 私信。操作员学一遍这个形态，到处都认得出来。

## 自我改进层

这里闭合了前面引入的那个循环：harness 修改自己的底料，由人来评审。两条循环干这事，运行在不同的时间尺度上。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/12.webp)

**循环 1 —— 单案件结构性修补（gap → PR）。** 每次调查结案后，`gap_analyzer` 找出 harness 在哪儿表现不佳：跳过的数据源、选错的工具、流水线 bug、缺失的测试、MR 描述里的格式失败。缺口按类别和置信度过滤，再传给 `finalize_case`。案件闲置四小时后，`finalize_case` 在一个专属 worktree 里拉起 `patch_suggester`。结果是对 harness 自己代码库的一个 PR——编辑代码文件、提示模板，或 SOP 章节。这个 PR 需要人来合并。但它不需要人来写。

**循环 2 —— 跨案件行为强化（失败模式 → SOP 建议）。** 每个案件的 `auto_retrospective` 步骤盘点触发了哪些失败模式——按类型归类：数据收集失败（调查是否跳过了可达来源？）、质量门违规（置信度声明是否和证据矛盾？）、分析完整性失败（根因是否一路追到底？）。这些按项目累积进案件登记表。一条每周 cron（`knowledge-updater`）统计最近十个案件里各模式的频率，任一模式越过阈值就生成 SOP 建议：10 次里 3 次触发一条建议，10 次里 6 次触发一条警报。建议很具体：*"'跳过了直接 DB 查询'这个月触发了 3 次——SOP 已更新：推断 schema 之前，总是先跑直接表查询。"* 人来评审并采纳。这条循环刻意不自动应用——会把错误复利的自动决策，是自修改系统里一类已知的失败。

**为什么这让 harness 变得个性化。** SOP 是 harness 对"你们团队怎么做调查"建立的模型。它一开始很泛。随着案件累积、失败模式重复，kb-updater 逐渐拼出这个项目特有的失败模式图景。20–30 个案件之后，一个项目的 SOP 跟泛用模板已经毫无相似之处——它反映的是具体哪些数据源老被跳过、哪些工具老被用错、哪些调查框定在这个代码库里始终不得要领。每个项目有自己的 SOP。A 团队的失败教出 A 团队的 SOP；它们不会污染 B 团队的。

这就是让学习能持久的原因。一个上下文窗口只熬过一次会话。一个合并的代码改动是带版本、经评审、有测试、永久的。一次 SOP 更新经操作员评审、被带进往后每一次调查。底料在改进；下一个案件跑在一套比上一个更好的 harness 上。

## 第 3 层 —— 持久化状态

Agent 编排是按事件跑的东西。持久化状态是进程重启、机器重启、或操作员从笔记本切到 CVM 之后还活着的东西。Harness 的状态分三层，特征截然不同：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/13.webp)

**内存缓存**很小、可重建。它们活在轮询器 / keepalive 进程内部，重启时从本地状态重建。

**本地状态**（`~/.harness/*.json`）是运维映射。关键文件：

-   `task-thread-map.json`——Slack thread\_ts → JIRA id + 绝对 `task_dir` 路径。
-   `oncall_thread_map.json`——同样的形状，但用于值班调查案件。
-   `monitored-mrs.json`——harness 开过的 MR 列表 + 它们的 CI 状态。
-   `patch-pr-state.json`——每个案件的 finalize 状态（`merged`、`no_gaps`、`failed`……）。
-   `oncall-state.json`——kill switch 标志（第 6 层细讲）。
-   `socket-dedup`——Socket Mode 近期处理过的消息 ts。这一层是机器本地的、不共享。重启安全，但机器一死，这层就没了。

**Git 同步状态**是可持久化的那层。案件工作区（`case-workspaces/<ticket-id>/`）在每次有意思的状态变更时提交并推到一个私有 git 仓库。另一台机器克隆同一个仓库，就能续上同一段对话上下文。我们就是这样拿到跨机器会话连续性的——操作员可以从笔记本、也可以从远程开发 VM 跟 harness 聊，不丢上下文。

我们给 git 同步状态加了第三个角色：**记忆固结。** 长对话会积累几十个 `followup_transcript_*.md` 文件。活动够多之后，一个子 agent 把它们全读一遍，重写 `TASK.md` 把蒸馏后的状态收进去，往后 workflow 就不再把单个逐字稿喂给后续轮次。没有这步，提示会涨过模型的上下文窗口。**有了**这步，又有个真实风险：我们会过度压缩、丢信息——而且确实发生过一次，把一份 14KB 的文档压到了 587B。那次事故让现在有了三层保护：一个"UPDATE 而非 REWRITE"的提示模板，一道输出大小守卫（拒绝小于输入 50% 的写入），以及一个每案件的锁文件，防并发固结跑互相竞态。

从单条 Slack 串到案件 TASK.md 的查找图很小，但值得展示一下：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/14.webp)

两个状态文件，同一种值形态：一条指向案件工作区的绝对路径。每条 workflow 最终都解析到一个 `case_dir`，然后相对它操作。重命名工作区根——我们项目中途干过一次——就是一个迁移脚本，把两个映射里的绝对路径与目录移动一步到位地重写。

可持久化状态是地基。当建在这层之上的操作失败时会怎样？Harness 不把失败浮给操作员——它从失败里恢复。

## 第 4 层 —— 自愈循环

这里 harness 不再只是个通知器，开始变得自主。三条循环，每条都不靠操作员干预就闭合。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/15.webp)

前两条完全自主。第三条刻意不是——OKTA 刷新需要一次硬件绑定的 MFA 提示，所以 agent 自己停下来，在私信里附一份 30 秒的操作手册请人动手。这是设计上的人在环路，不是能力差距。

这些循环有个微妙的属性：**重试预算与自我识别**。CI 自动修复有个每 MR 的上限（连续失败 3 次 → 升级给操作员）。评审者自动回复用**作者感知去重**——如果机器人在某条讨论里的上一条笔记比评审者的更新，循环就不再触发（机器人已经回过了）。OKTA 助手有个 15 分钟的批准窗口。没有这些，循环偶尔会陷入螺旋——项目早期，一个带 flaky 集成测试的 MR，在我们逮住它之前，一个下午跑了 80 次空操作的"修 CI"提交。

## 三层自愈：当确定性规则不够用时

上面的 CI 自动修复和评审者回复循环是端到端的恢复流。在那些流之下，每个可能失败的原子操作（push、rebase、MR 评论、分支重置）都坐落在一套独立、更小的三层架构上，做**逐操作自恢复**。这跟第 6 层那个三层**安全**模型是不同的轴——那个讲的是克制（agent 被允许做什么），这个讲的是恢复（一个被允许的操作失败时怎么办）。容易混，值得分清楚。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/16.webp)

**L1 —— 确定性守卫。** 亚秒级、基于规则、永远在线。对 push 来说，那就是 `_pre_push_rebase(repo_path, branch)`：每次 harness 要 push 之前，先 `git fetch origin` 再 `git rebase origin/<branch>`。rebase 干净就 push；有冲突或 fetch 失败，L1 就干净利落地中止，列出冲突文件，交给 L2。L1 做不了任何有创造性的事——这正是要点。规则小到足以推理，也有界到足以在每个操作上跑而不花 token。

**L2 —— agentic 自愈循环。** 当 L1 修不了局面，harness 拉起一个受约束的 Claude 会话（`_self_heal_push_loop`），带上完整的失败上下文：git 状态、冲突文件、当初触发这次 push 的原始任务，以及一组紧绷的红线（绝不强推到受保护分支、绝不丢未推送的提交、绝不盲选某一侧冲突）。三条约束让它安全：

1.  **有界重试。** 每次事故 ≤3 次尝试，每次 Claude 出招超时 4 分钟。每次 Claude 动作之后，harness 重跑真实的 `git push`——Claude 自己对冲突是否解决的判断永远不被信任；`git push` 退出 0 才是地面真相。Claude 想怎么宣称"修好了"都行，但只要下一次 push 返回非零，循环就把那次算作一次失败尝试。
2.  **置信度门 ≥ 70。** 一字不差从值班调查流水线的 `oncall_run` 配置借来：Claude 每招给自己打 0–100 分，低于 70 就必须撤退、不能动手。低于下限的自报不许碰工作树。
3.  **真实性不变量得以保留。** L2 成功意味着真有一个提交落到了 origin。下游那道回复门（给评审者发的"已搞定——已更新"回复）依然硬连到 `git push` 退出 0，而不是 Claude 关于自己成功的任何说法。L2 绕不过回复门；它只能给回复门喂一次真实的 push。

**L3 —— 操作员升级。** L2 撤退了（低置信度）或者尝试耗尽 → harness 给操作员发私信，附上**完整的 L2 轨迹**：每次尝试、Claude 走的每一步、每个置信度评分、最终的 `git status`。操作员从一个全程留痕的状态接手，而不是从"出了点问题"接手。时间尺度：小时，不是秒。

有两条原则值得浮出来：

-   **L2 是逐事故触发的，不是永远在线。** L1 在每次 push 上跑，因为它便宜。L2 烧 token，所以只在 L1 已经失败时才触发——成本由罕见情形门控，不由常见情形门控。
-   **以连线上的真实为地面真相。** 不管 Claude 怎么认为自己的活，校验都重跑真实操作。对 push，那是 `git push` 退出码。对测试修复，那是跑真实的测试。对 MR 回复，那是回复门检查 push 退出码。Agent 的自报从来不是地面真相——harness 拿连线重新核对。

这套模式可以推广到 push 之外。任何有明确确定性恢复（L1）、受约束 agentic 恢复（L2）和干净升级面（L3）的 harness 操作，都能照样包起来。正是这个形态，划开了"能从自己已知失败里恢复的自主循环"和"世界一变就立刻要操作员的循环"。

## 第 5 层 —— 可观测性

Harness 往四个可观测性面写东西，每个都针对一类不同的消费者调过：

-   **Slack 实时更新**——agent 跑的时候，案件串里有一条占位消息在状态消息间轮播（`Reading TASK.md` → `Searching codebase` → `Drafting patch`），最后定格为 `Done` 或 `Failed: <reason>`。这是流式汇报器；它编辑同一条 Slack 消息，所以串不会被刷屏。
-   **结构化日志**——`~/.harness/logs/agent.log` 和 `~/.harness/logs/error.log`。每次拉起、每次链路分派、每次重试尝试，都有一条结构化事件。诊断时我们 grep 的就是它。
-   **针对终态事件的 Slack 私信**——MR 开了、MR 合了、OKTA 过期、CI 升级。这些是一等公民级的操作员通知，必要时挂在案件之下成串。
-   **系统日志**——`~/.harness/logs/system.log`，记跨切面事件：部署完成、看门狗动作、同步状态。

我们做了个明确的选择：**不**搭仪表盘。Slack 和 grep 就是控制台。仪表盘多出第三个要保持时效的面，而且一旦过时，它就是工程师第一个停止去看的东西。流式 Slack 消息这套模式提供的运维信号已经够了，我们从没想要在上面再加个 UI。

## MCP 健康监控

Harness 依赖五个外部 MCP（Slack、Datadog、Jira/Confluence、Glean、GitLab）。每个都有自己的 OAuth token、自己的刷新节奏、自己一套不透明的失败模式。不处理的话，MCP token 过期会表现为：一个 Claude 会话莫名其妙地返回"我没有访问那个工具的权限"，而操作员看不到任何错误。

一个小服务——**mcp-watchdog**——补上这道缺口。每 10 分钟它轮询每个 MCP 的健康端点，试一次空操作的工具调用，失败了就：

1.  尝试一次静默 token 刷新。
2.  刷新失败，就给操作员发条 Slack 私信："Atlassian MCP 需要重新认证——点这里。"
3.  连续两次断连之后（`DISCONNECT_CONFIRM_THRESHOLD`），在最近的工单上加一个 `:warning:` reaction 来升级。

没有这条循环，一个过期 token 浮现出来的样子就是"agent 今天变蠢了"——这类失败诊断起来痛苦得要命，因为 agent.log 里没有一句说"token 过期"；它只说"工具返回空结果"。把认证健康做成一等公民级的可观测性面，是我们本该更早建好的一步。

## 第 6 层 —— 人在环路的控制

这是用得最多的面，也是我们一开始最没建够的那个。所有操作员交互都走单一通道——自我私信——组织成六个命令族：

-   **Task**——`admin: task <jira>` · 带 `--auto` 表示无人值守执行。触发完整的 分析 → 实现 → MR 流水线。
-   **On-call**——`admin: oncall-run <url>` 从一条 Slack 串发起调查。`oncall-toggle` 启用或禁用自动监控。
-   **MR control**——`admin: mr rescan` 重新轮询某个特定 MR。`mr pause/resume` 门控该 MR 上的自动回复循环。
-   **Pause**——`admin: oncall pause [duration]` 让自动分派噤声。`oncall resume/status` 解除它或显示当前状态。
-   **Finalize**——`admin: finalize <case>` 合并缺口报告，对 harness 开改进 PR。
-   **Registry**——`admin: register-thread` 手动把一条 Slack 串关联到一个案件。`close-case` 把它标为完成。

最难调对的模式是**暂停 kill switch**。第一版把一切都暂停了——包括管理命令本身。把系统暂停的那位操作员，接着就没法恢复它了。第二版暂停得太窄，操作员的意图（"别吵了"）没被尊重。当前这版校准到了一个精确的区分：**暂停让别人活动驱动的自动分派噤声**，但从不让操作员自己的管理命令或串内交互噤声。轮询器里的分派点干净地分成两组：那些代表别人活动驱动的自动行为（暂停时门控），和那些代表操作员自身意图的（永远在线）。

这次校准是吃了苦头才学到的：第一版暂停部署后不久，一条评审评论的自动回复无声地在另一个案件上触发了，而我们没察觉，因为操作员的暂停看上去正生效。当前设计把暂停当成**定向静默**——让系统安静下来，同时保住操作员的能动性。

一张图把这个分割讲清楚：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/17.webp)

规则：**操作员自己的动作绝不噤声；别人的活动在暂停时门控。** `admin: oncall pause` 管的就这些。

## 三层安全

暂停 kill switch 是最显眼的安全面，但不是唯一的。另有两层在没有任何运行时配置的情况下强制安全——一层在代码里，一层在部署约定里：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/18.webp)

Slack 频道白名单比听起来要紧。项目早期我们有过一次险情：一个配错的 Slack token 拥有对内部团队频道的写权限——意味着 agent 本可以直接发消息给队友、完全绕过操作员，而团队根本没法把它和真人消息区分开。那之后，每一条对外的 `chat.postMessage` 在发送前都对照白名单校验目标频道。唯一被允许的目标是操作员的自我私信（操作员自己和机器人那条私信的频道 ID——一个写死的单值）。如果 agent 哪天试图直接给队友发消息，它必须把内容浮回给操作员，由操作员决定要不要转发。这是我们反复回到的一个架构选择：**agent 只通过操作员的信箱对外沟通。**

## 六层之外：让它撑得起生产的那些零件

上面六层是承重结构。还有几样机制坐在层级分类之外，但同样关键——以我们的经验，正是这些零件，划开了"演示效果好的 harness"和"真能不间断跑的 harness"。

## 用一份 YAML 做多项目路由

Harness 今天服务两个项目，各有不同的运维规则，加第三个项目零代码改动——只要在 `agent-config/projects.yaml` 里加一条目

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

一个 `resolve_project()` 函数按优先级路由每条进来的信号：**Jira 前缀 > Slack 频道 > 默认**。每个项目拿到自己的 SOP 文件、KB 文件和目录树。调查提示的组装会根据信号解析到哪个项目，自动取用对的 SOP 和 KB。这就是那个让单个 harness 实例能服务两个不同团队、两套不同运维规则的窍门。

这套模式可以推广。任何有多个产品团队跑在同一套 harness 上的组织，都能用同样的办法：声明式配置，没有每项目的代码分支。它还逼出一种干净的纪律——任何某一项目特有的东西，都必须活在它的 SOP 文件里、而不是代码里，因为代码是跨项目共享的。

## 跨机器会话连续性

操作员从两台机器工作：一台笔记本，一台在数据中心的长寿命 CVM。同一段 Claude 会话需要能从任一台续上。这很难，因为 Claude Code 默认把会话状态持久化到本地磁盘——从另一台机器重启，会话就是隐形的。

Harness 用一个 **git 后备的会话存储**解决这个：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/19.webp)

机制分三部分：

1.  **会话被 git 追踪。** 每段会话的 `.jsonl` + `.meta` 文件在每次有意思的状态变更时提交并推到一个私有仓库。仓库就是真相来源。
2.  **一个 5 秒的后台 fetcher** 在每台机器上跑，让本地副本保持最新。它预拉上下文，这样一次 Claude 会话恢复就不必在每次敲键时都付网络延迟。
3.  **按 CWD 查会话 UUID。** Claude 在某个目录里启动时，`session-start.sh` 钩子扫描本地 `.meta` 文件，找一个 CWD 匹配的，并恢复对应的 UUID。同一个会话 UUID 会在操作员所在的那台机器上恢复。

同步钩子（`user-prompt-submit.sh`，在每次 Claude 提示之后触发）按严格顺序做三步操作，靠 `flock` 串行化：

1.  **Fetch + reset。** `git fetch origin main && git reset --hard`——现在本地的 `active.jsonl` 装的是*另一台*机器的逐字稿。
2.  **覆盖之前先抽增量。** 读水位线（我们已经从另一台机器注入了多少行），只抽自那之后的新行，存到一个临时文件。这步必须在第 3 步*之前*发生——早先一个 bug 把它俩弄反了，导致注入总是把机器自己的逐字稿读回给自己。
3.  **推送自己的逐字稿。** 把本地 Claude 会话逐字稿复制到 `active.jsonl`，提交，推送。然后把水位线更新到本地行数。

flock 释放之后，第 2 步的增量作为系统消息注入给 Claude："另一台机器从我们上次同步以来加了这 N 条消息。"下一份提示就同时看到两台机器的上下文。

一个 `.linecount` 文件把真实总行数和文件大小分开追踪，因为逐字稿为推送被截断后，`wc -l` 会给出错误答案（推送前我们把它截到 1,000 行，以避开 25 MiB 的 git 限制）。没有 `.linecount`，水位线会在每次截断时被重置，导致注入把成千上万条已经看过的行重新投递一遍。

`flock` 串行化的 git 操作防住了并发推送造成的 `index.lock` 损坏（我们撞过一次：一个卡住的会话启动撞上一次并发的 meta-enricher 运行）。一个节流阀（可配，默认 30 秒）在快速敲键时跳过完整同步，避开逐字符的 git 延迟。

"git 即状态存储"这个模式，是从这里能拎走的更宏观的点子。我们不把 git 只当代码仓库，而把它当成 [**类 CRDT**](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) **的、可持久化的 agent 状态日志**，操作员那两台机器作为最终一致的读者。

有一个机制和跨机器连续性密切相关：记忆固结。memory\_consolidate 步骤（把逐字稿重写成干净的 TASK.md/ANALYSIS.md）不按固定计时器触发。一个独立服务——`**consolidate_trigger**`——对照四个信号评估每一轮：逐字稿里出现的提交 SHA 或决策动词、一条刚关闭的评审串、一个累积逐字稿超过 5KB 的案件工作区、或者距上次固结至少过了一小时。任一信号触发固结。原则是：信息在一个决策做出之后停止累积，而不是在固定时长之后——这四个信号合起来，近似在问"对话此刻是不是处在一个自然的歇息状态？"，又不必要求操作员去标记它。

## Worktree 不是实现细节

前面说过"一切都在 worktree 里发生"。这值得展开，因为它代表的架构选择是真正承重的。

最初的设计是让 agent 吐出一个 diff，由 harness 在主仓库检出里调 `git apply`。这有两种反复咬我们的失败模式：

1.  **脆弱的 diff 头。** Claude 会吐出一个 diff，里面的行号在生成时是对的，但在一个并发提交落地之后就错了。`git apply` 会拒掉整个补丁。
2.  **主树卡死。** 如果 `git apply --check` 中途失败，主仓库会被留在半应用状态。下一个在同一仓库上跑的案件会继承这棵脏树。

解法是 `worktree_manager` 服务。在任何需要改代码的 agent 跑之前，管理器在 `~/.worktrees/case-<case_id>-<stamp>/` 创建一个沙箱 worktree，从一个干净分支检出。Claude 在那个 worktree 里用它原生的 Edit/Write 工具操作——没有 `git apply`，没有 diff 文本往返。Agent 工作的方式和一个人类工程师一样：开一个新分支、新检出，编辑，提交，推送。

MR 合并或关闭时，worktree 被拆掉。如果 agent 半路失败，受影响的只有那个 worktree——主检出和其它并发案件毫发无损。

正是这个，才让单机上**安全的并发 agent** 成为可能。没有每案件的 worktree，同一仓库上的两个案件会在文件系统状态上竞态。有了它们，二者从构造上就是隔离的。

## 吃自己的狗粮：Harness 修自己的 Pull Request

Harness 的源代码活在一个私有 GitHub 仓库里。当 gap-report → finalize → MR 这条循环对 harness 自身产出一个改进 MR 时，那个 MR 有它自己的 CI：pytest、linter、Python 语法检查。如果一个 harness 改进 MR 的 CI 挂了，一个独立服务——**pr-ci-fixer**——会像 mr-monitor 接手客户 MR 失败那样接手它。

结果就是：**harness 不光自愈它为工单写的代码，也自愈它给自己写的代码**。那个加了一条新重试预算的 MR，可能因为预算破坏了某个现有测试而 CI 挂掉；pr-ci-fixer 察觉到，在一个 worktree 里拉起一个 Claude 会话，打补丁，重推，人类评审者看到的是一个绿的、可以直接合的 MR。这条循环在真实的 harness 改动上闭合，我们不止见过一次。

让循环在 harness 自己的源代码上吃狗粮，是"这玩意儿是不是真的生产级"的承重测试。如果 agent 能在自己的 pull request 上把自己的 CI 保持绿色，那它大概也能被信任去处理客户的了。

## 测试架构：五层对应五类失败

Harness 有跨 126 个文件的 4,656 个测试函数。这个数字在没有一套理解"每个测试逮什么"的框架时是误导性的，因为不是所有测试都等价。

L0–L4 框架按"隔离的是什么"，把每个测试归到五层之一：

-   **L0（纯边界）：** 无 mock、无文件系统、无 IO。测纯逻辑——门控函数、JSON 抽取、置信度上限算术。一个在 L0 通过的测试，无论部署环境如何，都保证正确。
-   **L1（文件隔离）：** 在临时目录里用真实文件系统，真实线程，不对 pathlib 做 mock。逮住 L0 按设计漏掉的并发写竞态和状态文件原子性 bug。
-   **L2（服务隔离）：** 真实子进程执行，外部 API（Slack、GitLab、Claude）被 mock。逮住服务组件之间的集成 bug——分派路由、载荷 schema、状态文件流转——这些是 L1 漏掉的，因为 L1 只隔离文件系统行为。
-   **L3（沙箱）：** 真实 git 仓库（经子进程），假的 Slack 和 Claude 响应。对着一个真实的 git 对象模型，测完整的 workflow 序列——任务注册、实现阶段启动、待确认。
-   **L4（集成矩阵）：** 六个子类，覆盖 actor 契约（L4a）、admin × 角色矩阵（L4b）、跨运行状态一致性（L4c）、服务间链路（L4d）、外部 actor 角色（L4e）、方法论缺口（L4f），以及时序排列（L4h/L4i）。
-   **L4g（生产回放）：** 十三个把特定生产事故复现成回归测试的测试。这些是在生产里发现一个 bug *之后*才写的——测试编码了触发该 bug 的确切 actor、上下文和状态，并永久保留以防回归。

L4g 测试写起来最贵、保留下来最有价值。它们把失败模式的机构记忆编码下来，否则这些模式直到重现才看得见。

## 反 AI 味的风格指南

Harness 开的每个 MR，在提交前还要过一道滤网：一份专门用来让输出**看起来不像 AI 生成**的风格 SOP。规则很直白：

-   **零 AI 署名。** 没有 `Co-Authored-By: Claude`，没有"Generated with Claude Code"，提交信息里没有 `[bot]` 后缀。这个 MR 是团队的，不是模型的。
-   **没有填空式标题。** "这个 MR 做什么 / 我们为什么需要它"这类模板被禁。MR 描述写成连续散文，配一个具体的 `Changes:` bullet 列表，就像一个工程师手写时那样。
-   **没有加粗内联标签结构。** `**Root cause:**` 后面接一段，被禁。`**Fix:**`、`**Risk:**` 也一样。标签式加粗是单个最可靠的 AI 味破绽。
-   **工作量证明必须命令可复现。** 不是"本地已验证"——而是 `pytest tests/test_foo.py::test_bar -v`，附上真实的命令和相关的那 3 行输出。

SOP 在 MR 描述被撰写的那个点注入到补丁生成提示里。它和提示模板分开存在，因为它是**操作员策划**的：SOP 任何时候都能改，下一个 MR 就取用新规则，不重启守护进程，不改代码。缺口循环甚至能对 SOP 本身提出编辑——关于 agent 该怎么写 MR 的元规则，最后变成对那份元规则文件的 MR。

为什么这比听起来更要紧：没有什么比团队意识到"每个 MR 描述都一股 AI 味"更快地杀死对一条自动 MR 流水线的信任。这股味一冒出来，合并标准就往上抬，agent 真正的信号就被埋掉，操作员开始想手动重写描述——而这就把整件事的意义给毁了。风格 SOP 就是那个让 MR 描述混进人写的那堆里去的东西，而这正是让循环保持可用的东西。

## 端到端走查：一条工单，一个 MR

下面是一条真实工单到来时发生的事。时间是大概值。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/20.webp)

端到端看，对一条中小型工单，这条循环通常在 **30–90 分钟的墙钟时间**内完成。操作员的总主动投入一般在 5 分钟以内：确认分析、按需引导模糊决策、按需点合并。

调查本身会累积。30 个案件之后，harness 有了 30 个 `gap_report*.md` 文件，每个都指向 LLM 认为可以改进的某处。一次每两周一回的 `admin: gap-patterns` 运行，把这些聚成反复出现的失败模式，下一次 finalize 把最靠前的那些转成对 harness 仓库的 MR，harness 就改进了自己。这就是自我改进循环的具象。

## 与其它 harness 的对比

另有几个系统共享 harness 这个形态。我们的在哪儿重叠、在哪儿分岔：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@6ba0de93d8387ba8a2088e5789fd96b27cc104e3/ai-insights/2026-06/03/images/building-a-production-agent-harness-turning-claude-code-into-a-multi-agent-engineering-pipeline/21.webp)

我们**异常深**的地方：

-   **预承诺假设锁**——Claude 在跑任何一轮调查迭代之前先锁定 ≥6 个候选假设，防止事后合理化。我们找过的商业 harness 里，没有一个出货同等的东西。
-   **带机械置信度上限的 19 函数质量门框架**——置信度被算成一个硬性算术上限（`1.0 − open_q×0.08 − unchecked×0.05`），不只是个自报数字。三族门（逻辑一致性、结构完整性、断言上限），外加逐维度的对抗式惩罚。我们找过的商业 harness 里，没有一个出货这个。
-   **修改 harness 自身代码库的自我改进**——gap\_reports → 对 harness 自己代码/提示/SOP 的、经过滤的 PR。最接近的学术对应物（SICA，arXiv 2504）拿到了基准增益，但它是离线研究；没有生产系统出货自主的 harness 代码修改。
-   **项目作用域的 SOP 个性化**——每个项目的 SOP 基于它自己的失败模式频率演化。20–30 个案件之后，SOP 反映领域特有的调查失败。没有别的生产 harness 实现这个。
-   **带明确交接的多 agent 编排**——`oncall_run` → `case_followup` → `gap_analyzer` → `finalize_case` → `dev-agent`，每个都有定义好的交接 schema 和隔离的 worktree，而不是一个 agent 什么都干。
-   **跨机器状态同步**，靠 git 后备的水位线注入——同一段 Claude 会话在笔记本或远程 VM 上以完整上下文恢复，无需手动同步。
-   **内建的 CI 自动修复 + 评审者自动回复循环**——只有 Devin（商业）出货可比的内建自愈；其它每个框架都需要自定义接线。

我们**刻意做薄**的地方：

-   没有正式的**评估 harness**。我们不在基准上给 LLM 打分。生产反馈就是评估。
-   没有**仪表盘**。Slack + 日志。
-   没有**成本计量器**。Token 用量被松散地追踪，但不作为仪表盘浮现。成本由人在环路的确认步骤来界定。

## 展望

Harness 仍在演化——自主循环里的覆盖缺口、成本计量、一套正式的评估 harness——没一个是拦路虎。系统已经在做它被造出来要做的事：把 Slack 信号转成合并的代码、通宵处理评审反馈、自愈 CI、把自己的改进反哺进自己的下一个版本。

更深的教训是那个元循环。Harness 里每一道伤疤，都是因为一条真实工单试探到了它才被逮住。构建循环和运行循环是同一个循环，只是在不同的时间尺度上。生产信号回流到 harness 自己的源代码；agent 提出一个修复；人来评审并合并；下一类 bug 就离被修剪掉又近了一步。

系统没有完结。一个自以为完结了的生产 harness，就是一个停止累积伤疤的 harness——这换个说法就是，它不再跑在真实负载上了。

**先把循环建起来，再让它自主。** 顺序要紧，因为一个没有反哺路径通向自己底料的自主系统，不过是一种更快地、规模化地、出货同一套错误的方式罢了。
