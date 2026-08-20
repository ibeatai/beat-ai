---
title: 拥有你自己的 Harness
author: Sau Sheong
url: https://sausheong.com/own-your-harness-2f5299a855a7
translated: 2026-05-21
tags:
  - AI Agent
excerpt: 过去几个月里，我写了几篇关于 agent harness 的文章，也写了为什么所有人似乎都聚焦于它。一句话总结就是：它是新的前沿。没错，大家仍在模型这条战线上厮杀（比如 DeepSeek 刚发布了 v4，OpenAI 发布了 GPT-5.5，Anthropic 拿 Mythos 把所有人吓得不轻），但真正的战斗将发生在 harness 上。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/21/images/own-your-harness/01.webp
---

# 拥有你自己的 Harness

## Claude Code 和 OpenClaw 中的 compaction，揭示了关于 agent 基础设施的什么

![图片来源：Nano Banana 2](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/21/images/own-your-harness/01.webp)

*免责声明：这篇文章可能相当技术化，即便它并不展示代码。如果这是你读起来不太舒服的内容，那就只读开头和结尾。要点都在那两处。*

过去几个月里，我写了几篇关于 agent harness 的文章，也写了为什么所有人似乎都聚焦于它。一句话总结就是：它是新的前沿。没错，大家仍在模型这条战线上厮杀（比如 DeepSeek 刚发布了 v4，OpenAI 发布了 GPT-5.5，Anthropic 拿 Mythos 把所有人吓得不轻），但真正的战斗将发生在 harness 上。

OpenAI 已经超越了模型本身，正在构建一套完整的 agent 系统。它现在提供能跨工具执行多步任务并保留上下文的工作区 agent，一套把工具、记忆和编排方式标准化的 Agents SDK，以及用于大规模管理这些 agent 的 Frontier。Anthropic 走的是一条平行路线。它不做一个通用型 harness，而是创造一批领域专属的环境，比如 Claude Code、Claude Cowork 和 Claude Design，每一个都基于同一个模型，但针对某一类特定工作做了调优。

微软和谷歌则把这个趋势进一步推向企业基础设施。微软正把 Copilot 变成一个系统层，Agent 365 加上了身份、治理、编排和生命周期管理，让 agent 成为一等公民。谷歌在 Cloud Next 2026 上把这一转向说得很明确，推出了 Gemini Enterprise Agent Platform，把 agent 定位为在整个组织中连接数据、工具和执行的那一层。然后还有 xAI。它没有纯粹在模型层竞争，而是转向收购 Cursor——一个 AI 原生的编码环境——实际上就是在买下工作流、分发渠道和一个开发者执行循环。换句话说，就是 harness。

这个模式现在已经清晰得无可辩驳。在每一个主要玩家那里，模型正在变成众多组件中的一个。真正的产品是 harness——那个掌管上下文、工具、执行、身份和集成的系统，竞争正在向那里转移。

我也被问过好几次了：harness 能复杂到哪去？它不就是模型之上的某个软件层吗？重要的部分是模型，其他一切都只是花架子。连 Anthropic 自己都这么说过。打造了 Claude Code 的 Boris Cherny，在 2025 年 5 月的 Latent Space 播客上把它称为"对模型尽可能薄的一层包装"，还补了一句"所有的秘方都在模型里"。既然这样，那我们干脆从别人那买一个 harness，了事就好。我们不想把资源捆绑在一个属于商品、又只是模型之上薄薄一层包装的东西上。

![由 ChatGPT 生成](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/21/images/own-your-harness/02.webp)

我认为很多人仍然没有理解 harness 能有多复杂，也没有理解不同策略之间的差异有多重要。这是构建真正强大的 agent 和构建只会依赖模型能力的平庸 agent 之间的区别。别误会我的意思，我不是说模型不重要。我想说的是，harness 将成为那个决定性的差异点。正如 Shangru 在另一篇文章里贴切地说的，模型是引擎，但 harness 是车辆。两者都重要，但除非你有海量的钱，否则你拥有不了自己的模型，所以你应该努力去拥有自己的 harness。否则你就只是个乘客。

在这篇文章里，我想更深入地剖析当下最流行的两个 agent：Claude Code 和 OpenClaw。命运使然，Anthropic 不小心泄露了他们的 Claude Code 源代码，而且网上有大量仓库，让我们能窥探这个（截至目前）最流行的编码 agent 的内部构造。OpenClaw 的代码当然是开源的，可以让我随手翻看。它们的代码库都相当庞大，所以我只聚焦于其中一个部分——compaction 策略。

## compaction 到底是什么

那么……什么是 compaction？

当一个 agent 和它的模型进行长对话时，对话历史在每一轮都会增长。每一次工具调用、每一次文件读取、每一条错误消息、每一段模型输出，都会被重新缝回下一轮发送的 prompt 里。模型有一个上下文窗口。对于现代前沿模型来说，那通常是 20 万 token，如果你愿意付费享受这个特权，有时能到一百万。最终你会撞上那堵墙。compaction 是一个总称，涵盖 agent 为了让对话不溢出边界所做的一切。

你可以用便宜的方式做：丢掉旧的工具结果，或裁剪掉一段嘈杂的 bash 日志。你也可以用昂贵的方式做：再次调用模型来总结整段对话。你可以在问题出现之前就做（预测式），也可以在那堵墙已经在视野里时才做（反应式）。你可以用一份摘要替换历史，也可以把重要事实写进一个持久文件，让 agent 之后可以搜索。大多数真实的 agent 会同时用上好几种，而顺序、阈值、prompt，以及每种技术与 prompt 缓存交互的方式——所有这些加起来，构成了一个设计选择比你以为的要多得多的技术栈。

Claude Code 和 OpenClaw 都有这样一个技术栈。它们是分层的、防御性的，而且相当有主见。它们在几乎每一件事上都意见相左。

## 两种相反的策略

第一个、也是最大的区别，是每个系统决定何时进行 compaction。

Claude Code 是反应式的。它让对话一直增长，直到预计的 token 用量接近上下文窗口，到那时才进行总结。触发公式位于 `autoCompact.ts` 中。当运行中的 token 计数距离有效上下文窗口还有 13,000 token 时，自动 compaction 触发；手动 `/compact` 则在还剩 3,000 token 时被阻止。还有一个真正的反应式兜底机制：如果所有其他机制不知怎地都漏掉了阈值，它会捕获 API 返回的 `prompt_too_long` 413 错误。模型被信任可以一直跑到那堵墙跟前。

OpenClaw 是先发式的。在每一次 LLM 调用之前，它都会估算下一个请求的总 token 成本，乘以 1.2 的安全余量，然后问自己这次调用能否装得下。如果装不下，它就路由到最便宜的可行策略。那可能只是截断一个超大的工具结果（`truncate_tool_results_only`），或者一次完整总结（`compact_only`），或者两者都来（`compact_then_truncate`）。在这之上还有一个 `transformContext` 守卫，如果一个请求预计会超过窗口的 90%，它会直接抛出"Context overflow"。

这是两种非常不同的策略。Claude Code 假设模型通常会在撞上限制之前就完成工作，所以它让上下文跑得火热，只在被迫时才支付总结的成本。OpenClaw 假设在每次调用前估算 token（成本本质上为零）是值得做的，可以避免意外，并且能在抵达那个昂贵选项之前，优雅地逐级升级走过较便宜的选项。

两者都不是显然正确的。反应式更简单，在顺利路径上更快。先发式更具防御性，在情况变得诡异时更灵活（连续多个长工具结果、一次出乎意料臃肿的文件读取、一个产出了比平常长得多的回复的模型）。两个工程水平都很高的 agent，最终落在这条光谱的两个相反端点上——这件事本身就告诉你，这块设计空间还有多么悬而未决。

## 每一轮都跑的那些便宜手段

两个 agent 都有一层不断运行、并且完全不调用 LLM 的小技巧。这是人们听到"compaction"时通常不会想到的部分。

Claude Code 拥有这一堆里最"作弊"的技巧，而它之所以可用，只因为 Anthropic 拥有自己的 API。有一个叫 `cache_edits` 的 beta 功能，让客户端可以告诉 API 服务器从缓存的 prompt 中删除特定的工具结果。本地的消息列表保持完整，但缓存的前缀被精准地编辑掉，过时的工具结果不再消耗 token，同时又不让 prompt 缓存失效。对于一段 20 万 token 的缓存前缀，一次失效会让下一轮真的花掉钱，所以 `cache_edits` 让你既能驱逐内容，又不必承受缓存未命中。Claude Code 把这个叫做"cached microcompact"，并对一个固定白名单上的工具（Read、Bash、Grep、Glob、Edit、Write、WebSearch、WebFetch）每一轮都运行它。甚至还有一个基于时间的姊妹变体，当用户空闲超过 60 分钟时清掉过时的工具结果——因为到那个时候，服务器端的缓存已经冷过期了，再也没什么可损失的。

OpenClaw 做不了这些当中的任何一项，因为它不拥有 API。它有的，是一个通用的 `tool-result-context-guard`，会缩减任何单个超过半个上下文窗口的工具结果，并在总量越过 90% 时抛出溢出错误。它更粗糙，但它对任何提供商都管用。OpenClaw 还有明确的"工具对修复"（tool-pair repair）代码，在历史裁剪之后移除孤立的 `tool_result` 块——因为 Anthropic 的 API 会拒绝一个其匹配的 `tool_use` 已被丢弃的 `tool_result` 请求。Claude Code 不需要这个，因为它有针对性的 `cache_edits` 操作不会留下孤儿。

两者都有滑动窗口逻辑，在上下文开始膨胀时丢掉中间的消息。Claude Code 把它叫做"snip"，OpenClaw 把它叫做 `limitHistoryTurns` 和 `pruneHistoryForContextShare`（后者持续丢掉最旧的，直到保留下来的上下文最多只占窗口的一半）。两者都不新颖。两者都有必要。

要点在于：相当一部分工作发生在任何基于 LLM 的总结进入画面之前。如果你能把这些便宜手段做得足够聪明，你就能把那次昂贵的调用一次次往后推。Claude Code 推得最远，因为它拥有别人都接触不到的 API 功能。

## 当驱逐还不够用时

便宜手段终究会用尽，你不得不进行总结。

Claude Code 的总结路径是一次单独的 LLM 调用。整段对话被送进去（图片和文档被替换成占位符，以便为总结器节省 token），一份很长的结构化摘要被产出来。输出被封顶在 20,000 token，这个尺寸对标的是历史 compact 摘要长度的 p99.99（根据代码里一条注释，大约是 17,387 token）。这次调用是流式的，所以用户能看到文本一边生成一边出现，而不是干等一个阻塞式的整体响应。这里有一个值得点出来的巧妙优化。总结器通过 `runForkedAgent` 运行，它继承了父 agent 的 prompt 缓存前缀，所以这次总结调用虽然是一个全新的请求，却基本上是一次缓存命中。`compact.ts` 里的一条注释指出，不带前缀共享地运行它会是"98% 的缓存未命中，花掉整个机群 cache_creation 的约 0.76%（每天约 380 亿 token）"。那是一种你只有在 agent 大规模运行时才看得到的细节。

OpenClaw 的路径要精巧得多。总结调用被委派给底层的 `pi-coding-agent` SDK，它可以为 compaction 调用使用一个单独的、更便宜的模型（通过 `agents.defaults.compaction.model` 配置）。当对话大到无法一次总结完时，OpenClaw 在工具调用的边界上把它切开（这样一个助手的 `tool_use` 和它匹配的 `tool_result` 总是待在一起），用一次单独的 LLM 调用总结每一块，然后跑一次最终的合并调用，把各块的摘要缝合到一起。这条合并 prompt 值得完整引用。

```
Merge these partial summaries into a single cohesive summary.MUST PRESERVE:
- Active tasks and their current status (in-progress, blocked, pending)
- Batch operation progress (e.g., '5/17 items completed')
- The last thing the user requested and what was being done about it
- Decisions made and their rationale
- TODOs, open questions, and constraints
- Any commitments or follow-ups promisedPRIORITIZE recent context over older history. The agent needs to know
what it was doing, not just what was discussed.
```

"5/17 items completed" 这个例子很有意思。写这条 prompt 的人显然亲眼见过模型把批处理进度计数器从摘要里丢掉，于是学会了专门点名要求保留它们。结尾那句话才是精华。agent 需要知道它当时在做什么，而不只是讨论过什么。那一句话就道出了 compaction 的全部要点。

OpenClaw 还有一个 Claude Code 没有的东西，那就是一个确定性的审计步骤。摘要生成之后，`auditSummaryQuality` 会检查所有 5 个必需的章节标题是否按顺序出现、从最后 10 条消息中提取出的每一个标识符是否都在摘要文本里留存下来、以及最新的用户提问是否与摘要正文有 token 重叠。如果这些当中有任何一项失败，摘要就会带着结构化的反馈被重新生成（`missing_section: ## Decisions`、`missing_identifiers: abc123, ...`）。agent 自己给自己的作业打分，然后重交。

Claude Code 一概没有这些。它信任模型在第一次尝试时就遵循 schema，而模型通常确实会，但这里没有任何程序化的安全网。

## prompt 讲述了一个故事

总结 prompt 本身，是窥探每个项目如何思考 compaction 的最清晰窗口。

Claude Code 的主总结 prompt（`src/services/compact/prompt.ts` 中的 `BASE_COMPACT_PROMPT`）要求一份 9 个章节的叙述式摘要，涵盖：主要请求与意图、关键技术概念、文件与代码段、错误与修复、问题求解、所有用户消息（没错，全部）、待办任务、当前工作，以及一个可选的下一步。下一步那个章节有一个不寻常的约束。如果存在一个下一步，模型被告知要包含"来自最近对话的直接引用，确切地展示你当时在做什么任务、做到了哪里。这应当是逐字的，以确保对任务的理解不发生偏移。"这是一个针对一种众所周知的模型行为的巧妙防御——模型在总结之后会微妙地偏离用户的真实请求。通过强制逐字引用，compaction 之后的 agent 被锚定到它当时所依据的确切措辞上。

同一条 prompt 以一段不寻常地强硬的"禁用工具"开场白开头。根据同一文件里的一条注释，原因是：用于缓存共享的 fork 继承了父 agent 的完整工具集（它必须如此，否则缓存键就会改变）。带自适应思考的 Sonnet 4.6，在面对那条较旧、较弱的 prompt 时，有 2.79% 的情况会在 compaction 期间尝试一次工具调用，而 Sonnet 4.5 是 0.01%。在 prompt 最顶端加上那句明确的"工具调用将被拒绝，并会浪费你唯一的一轮，你会任务失败"，就是团队对那次 4.6 回退的回应。那是一种你只有在拥有数百万 agent 会话的遥测数据时才有机会做的、经验驱动的 prompt 工程。

OpenClaw 的 prompt 在风格上非常不同。如果说 Claude Code 读起来像一位资深工程师在写一份谨慎的交接笔记，OpenClaw 读起来则像一份要求 LLM 去实现的合同规范。它要求的摘要结构是五个简洁的标题（Decisions、Open TODOs、Constraints/Rules、Pending user asks、Exact identifiers）。还有一条单独的指令——"逐字保留所有不透明标识符，包括 UUID、哈希、ID、主机名、IP、端口、URL 和文件名"——并带有 `strict`/`off`/`custom` 三种策略模式，而那个审计步骤会真的去检查合规性。这种审美差异自始至终都很一致。Claude Code 用第二人称和解释性的散文（"特别留意具体的用户反馈"）。OpenClaw 用祈使式的块（"MUST PRESERVE …… PRIORITIZE recent context"）。

Claude Code 的 prompt 更温暖、更灵活，这大概有助于模型产出一份更丰富的叙述。OpenClaw 的 prompt 更严格、更可验证，这意味着那个审计循环有具体的东西可以对照检查。schema 的选择直接映射到验证方式的选择。

## 记忆作为补充

两个 agent 都有一个单独的持久记忆层，作为 compaction 的补充。它们都判定，对话历史不是状态唯一该存在的地方。它们在如何做这件事上意见相左。

Claude Code 的会话记忆是一个单独的 Markdown 文件，有 10 个固定的章节（Session Title、Current State、Task specification、Files and Functions、Workflow、Errors & Corrections、Codebase and System Documentation、Learnings、Key results、Worklog）。这个文件在整段对话中由一个 fork 出来的子 agent 增量更新，那个子 agent 的工具权限被收紧到"只能 Edit 这一个文件，别的什么都不行"。更新在一组 token 阈值（10K 时初始化，更新间隔 5K）和工具调用阈值（更新间隔 3 次调用）的组合下触发。当 compaction 的时刻到来，如果会话记忆文件存在且不为空，它就被作为 compaction 后的摘要用户消息注入，并完全替代由 LLM 驱动的总结。不进行任何总结调用。那个持久文件就是摘要。

OpenClaw 的记忆是一个只追加（append-only）的 Markdown 文件日志，用带 FTS5 和嵌入向量的 SQLite 建索引以供检索。写入发生在一个"记忆刷新"（memory flush）轮次中，它就在 compaction 之前运行——同一个 agent（处于一个只允许向某个特定路径做追加写入的工具受限模式中）被要求提取出持久的事实。compaction 之后，刷新出的内容不会被自动注入回上下文。取而代之的是，下一次 agent 运行会拿到一条指向工作区的刷新 prompt，而记忆内容会在需要时通过一个记忆搜索工具被惰性地取回。

这种对比是结构性的。Claude Code 把记忆当作 compaction 的替代品——一个有固定 schema、被持续编辑的就地文件。OpenClaw 把记忆当作 compaction 的前奏——一份只追加的、为搜索建了索引的日志，在 compaction 前的爆发中写入。两者似乎都合理。两者谁都没有收敛到对方的做法，这说明正确答案在很大程度上取决于你跑的是哪种 agent、以及会话往往有多长。

## 当它出问题时会发生什么

这正是这两个系统分歧最尖锐的地方。

Claude Code 的恢复模型是"断路器加有损兜底"。这里有三个重试预算。`MAX_COMPACT_STREAMING_RETRIES` 是 2，用于瞬时的流式失败。`MAX_PTL_RETRIES` 是 3，用于 compaction 请求本身触发了 `prompt_too_long` 的情形。当那种情况发生时，`truncateHeadForPTLRetry` 丢掉最旧的消息组，在前面加上一个 "[earlier conversation truncated for compaction retry]" 标记，然后再试一次。它是故意有损的。一条代码注释把它称为"针对先发式/手动路径的、笨但安全的兜底"。第三个预算是 `MAX_CONSECUTIVE_AUTOCOMPACT_FAILURES`，设为 3，而为它辩护的那条注释是整个代码库里最有启示性的东西。"有 1,279 个会话在单个会话里出现了 50 次以上的连续失败（最多达 3,272 次），在全球范围内每天浪费约 25 万次 API 调用。"这个断路器之所以存在，是因为 Anthropic 有人看了日志，看到真金白银被卡在 compaction 循环里的 agent 烧掉了。

OpenClaw 的恢复模型是"快照加取消"。在 pi 运行 `session.compact()` 之前，OpenClaw 把磁盘上的会话文件复制成一个 `.checkpoint.<uuid>.jsonl` 的兄弟文件。成功时，这个检查点被保留下来（这样用户可以回滚）。失败时，它被清理掉。那个总结保障机制，如果它产不出一份干净的摘要，就返回 `{ cancel: true }`，让底层的 SDK 带着未经修改的转录继续。那条渐进式兜底链是最巧妙的部分。`summarizeWithFallback` 先尝试完整总结，然后退回到"只总结小消息，并对大的那些做个标注"，再退回到"只标注一下当时有什么"。一个 OOM 的块不会搞死整次 compaction。

两种哲学都说得通，但它们优化的是不同的东西。Claude Code 优化的是快速的用户反馈。如果 compaction 失败，就静默重试，退回到有损截断，如果那也不管用，就放弃，停止浪费 API 调用。OpenClaw 优化的是转录的保全。如果 compaction 不能安全地完成，那就别做，保留原始对话，让用户再试一次。当 agent 是用户的主要工具、解除阻塞比保真度更重要时，Claude Code 的做法就是你想要的。当对话历史本身就是一份不该被静默改动的宝贵成果物时，OpenClaw 的做法就是你想要的。

## 成本、延迟，以及真实的取舍

许多关于 compaction（或者真的说，任何架构设计）的讨论，完全跳过了运营这个维度。我们来快速看一眼数字。

在成本上，Claude Code 在热路径上每次触发严格来说更便宜。四种机制里有三种（`cachedMicrocompact`、基于时间的 MC、API 端的上下文管理）都是零 LLM 的。那次完整的 compact，当它确实触发时，由于 fork agent 的技巧，前缀部分是缓存热的。没有更便宜模型这个选项，因为 compact 用的就是主循环正在用的那个模型。OpenClaw 可能会很昂贵。分块总结的路径发出 N+1 次 LLM 调用（每块一次加上一次合并），而每次调用都把不同的块作为用户消息送出去，所以前缀缓存的复用基本为零。为了抵消这一点，OpenClaw 支持为 compaction 调用使用一个单独的（更便宜的）模型，这是 Claude Code 没有的。

在延迟上，Claude Code 是流式的。用户在 compaction 期间能看到输出逐步出现，而且没有明确的上限超时。OpenClaw 是顺序的。各个块用朴素的 `await` 循环一个接一个地总结，没有 `Promise.all` 并行，所以一次 4 块的 compaction 大致要付出 4 倍的单块延迟再加上一次合并调用。这里有一个硬性的 15 分钟安全超时，如果整个过程卡住就会中止它。

Claude Code 的 compaction 技术栈在热路径上更便宜、更快，因为它拥有推理层，能用别人都用不了的 API 功能。OpenClaw 的技术栈更具防御性、更可验证、也更可移植，因为它必须跨越那些它不掌控的提供商工作。两种选择都源自每个 agent 在供应链中所处的位置。

## 为什么你应该拥有自己的 harness

先停下来咂摸一下这个反讽。Anthropic 自己把 Claude Code 称为"对模型尽可能薄的一层包装"。然后他们交付了一个有八种不同机制的 compaction 技术栈、一个由沙盒化的 fork 子 agent 更新的会话记忆子系统、一个针对卡死会话的生产遥测数据校准过的断路器、一个专门为了让驱逐不会破坏 prompt 缓存而发明的 beta API 功能，以及经验充分到能把游离工具调用率追踪到 0.01% 的 prompt 调优。如果那就是尽可能薄的一层包装，那这个"薄"承担的活儿可不少。

那么回到我开头提的那个问题。为什么不干脆从别人那买你的 harness，把它当成商品化的基础设施？

这番对比把答案说得相当清楚。compaction 只是 harness 的一个切片，而仅仅在这一个切片上，地球上最流行的两个编码 agent 就构建出了在每一个层面几乎都相反的技术栈（反应式对先发式、叙述式对结构式、记忆替代摘要对记忆补充摘要、失败时有损对失败时快照）。两者每天都交付给真实用户。两者都管用。在这些选择底下，并没有一个等着被发现的"正确"答案。

想想看，如果你从货架上买你的 harness，那意味着什么。你继承的是别人的策略，包括：是让上下文跑得火热还是每一轮都估算、摘要该遵循什么 schema、在解除阻塞对保全这条取舍上落在哪一侧、以及用什么阈值、重试预算和兜底链。这些当中没有一个被记录在 README 里，因为它们不是功能。它们是那看不见的脚手架，决定了你的 agent 是为你的领域里的你的用户工作，还是只为这个 harness 最初被构建时所针对的那些用户和那个领域工作。如果你的 agent 是给资深工程师用的编码助手，而那个 harness 是为做小项目的新手用户调优的，你会在一些你不容易调试的地方感受到那种错配。

然后还有校准这个问题。Claude Code 代码库里的一条注释为一个重试预算辩护："有 1,279 个会话在单个会话里出现了 50 次以上的连续失败（最多达 3,272 次），在全球范围内每天浪费约 25 万次 API 调用。"那个数字不是来自一次架构评审，它来自生产日志。同样的模式也出现在那个观察里——Sonnet 4.6 在 compaction 期间有 2.79% 的时候尝试不想要的工具调用，而 Sonnet 4.5 是 0.01%。这个差异直接导致了那段"禁用工具"开场白被重写。同样地，这不是从某份 prompt 工程指南里推导出来的。它来自大规模运营那个 agent、并测量它的行为。

这两个数字，以及两个代码库里数十个像它们一样的数字，都是端到端拥有 harness、并且能够审视自己遥测数据的产物。如果别人拥有你的 harness，他们也就拥有了你的遥测数据，而他们做的那些优化会针对他们的机群校准，不是你的。你将永远是别人路线图上的一名乘客。

而 compaction 只是看得见的那一点。在工具路由、子 agent 编排、流式协议、错误分类、缓存管理、计划持久化、成本核算里，都有类似的设计深度。那些层里的每一个都有同样的性质。两个讲道理的工程师会对它意见相左，而对你而言的正确答案取决于你的模型、你的用户、你的延迟预算和你的失败容忍度。你没法靠购买 harness 来从这些决策里抽身。你只能从亲自做这些决策里抽身，而那是一件不同的、糟糕得多的事。

平心而论，你大概不该在第一天就从零开始写你的 harness。OpenClaw 所做的那种一块一块的复用（包装 `pi-coding-agent` 并提供宿主的粘合层）是一个合理的起点，如果你正在快速推进的话。但你一开始撞上诡异的生产行为，或者你的延迟画像与上游默认值发生偏离，或者你的用户在意某件上游没有去优化的事——那一刻，你就会回到这里读源代码，并希望你和他们之间的边界能再多偏向你一点点。你越早带着意图去划定那些边界，这件事就越便宜。

harness 是车辆。买一辆车之前，你会先好好想一想——你不知道刹车是怎么工作的、方向盘手感如何，而 harness 也一样。如果你的 agent 对你的业务很重要，你就应该拥有那个决定它在压力之下如何表现的部分。
