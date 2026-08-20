---
title: Agent Harness 剖析
author: Avi Chawla
url: https://blog.dailydoseofds.com/p/the-anatomy-of-an-agent-harness
translated: 2026-05-27
excerpt: 一个 ReAct loop、几个工具、一段写得不错的 system prompt——demo 阶段往往能撑得出奇地远。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/01.webp
---

# Agent Harness 剖析

### [Canvas Framework：把 AI agent 推上生产环境的结构化方法](https://fandf.co/41QzChe)

在基础模型出现之前，要做一个 AI 功能，意味着先收集并标注训练数据、从零训练一个定制模型，然后才能把它集成进产品。一个团队要走完这套流程才能验证用户到底想不想要这个功能，往往得花上几个月，还得砸进一大笔算力。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/01.webp)

基础模型把这一瓶颈拆掉了——它们已经预训练好，可以直接通过 API 调用。如今团队只要调一下 GPT-4 或 Claude，用 zero-shot 或 few-shot prompt 就能在几天内交付一个 MVP，先验证用户需求，再决定要不要为 RAG 或微调投入数据整理工作。

但对 agentic 系统来说，中间还缺了一层。

agent 设计必须紧跟在产品定义之后——因为 agent 的能力、工作流、记忆需求，决定了下游它需要哪些知识、哪些模型提供方才合理。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/02.webp)

MongoDB 围绕这条顺序发布过一份详细拆解，叫 **[Canvas Framework](https://fandf.co/41QzChe)**。它用两张规划画布。

-   POC 画布有 8 个方格，覆盖产品验证、agent 设计（能力、自主性边界、记忆需求）、数据需求（知识源、更新频率、反馈回路）以及模型集成（提供方选型、prompt 策略、成本验证）

-   生产画布在此基础上加了 11 个方格，针对扩展场景：容错、多 agent 协作、跨应用存储/向量检索/agent 记忆的统一数据架构，再加上安全加固与治理。


**[完整拆解在这里 →](https://fandf.co/41QzChe)**

*感谢 MongoDB 与我们今天的合作！*

* * *

### Agent Harness 剖析

一个 **[ReAct loop](https://www.dailydoseofds.com/ai-agents-crash-course-part-10-with-implementation/)**、几个工具、一段写得不错的 system prompt——demo 阶段往往能撑得出奇地远。

但任务一旦需要走 10 步以上，问题就全冒出来：模型忘了三步前自己干过什么，tool call 静默失败，context window 被一堆垃圾塞满。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/03.webp)

问题不在模型，而在围绕模型的那一整圈东西。

LangChain 验证过这一点：只改了包在 LLM 外面的基础设施（同一个模型、同一份权重），他们在 TerminalBench 2.0 上的排名从 30 名开外直接跳到了第 5 名。

另一项独立研究项目让 LLM 自己优化这层基础设施，pass rate 直接冲到 76.4%，超过了人工设计的系统。

这层基础设施现在有了名字：agent harness。

#### 什么是 Agent Harness？

这个术语在 2026 年初被正式定名，但概念本身早就存在。

harness 是包裹在 LLM 外面的完整软件基础设施，包括编排循环、工具、记忆、context 管理、状态持久化、错误处理和 guardrail。

Anthropic 的 Claude Code 文档说得很直白：SDK 就是「为 Claude Code 提供动力的 agent harness」。

LangChain 的 Vivek Trivedy 给出了一个我们很喜欢的经典公式：「如果你不是那个模型，你就是 harness。」

换个说法：「agent」是涌现出来的行为——一个有目标、会用工具、能自我纠错的实体，用户直接打交道的就是它；而 harness 是产出这一行为的那套机器。有人说「我做了一个 agent」时，其实是做了一个 harness，再把它指向某个模型。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/04.webp)

Beren Millidge 在 2023 年的一篇文章里把这个类比讲得很精准：

-   一个裸 LLM 就像一个没有 RAM、没有硬盘、没有 I/O 的 CPU。

-   context window 充当 RAM 的角色（快但有限）。

-   外部数据库充当硬盘（大但慢）。

-   工具集成则相当于设备驱动。


harness 就是操作系统。

#### 工程的三个层次

围绕模型，有三个同心圆般的工程层次：

-   prompt 工程负责打磨模型收到的指令。

-   context 工程负责管理模型什么时候看到什么。

-   harness 工程则把以上两者都包含进来，再加上整套应用基础设施：工具编排、状态持久化、错误恢复、验证循环、安全约束、生命周期管理。


harness 不是套在 prompt 外面的一层壳，而是让自主 agent 行为得以发生的那整套系统。

#### 生产级 Harness 的 11 个组件

综合 Anthropic、OpenAI、LangChain 以及更广泛的从业者社区的经验，一个生产级 agent harness 有 11 个明确的组件。逐个看一下。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/05.webp)

##### 1\. 编排循环（Orchestration Loop）

这是 harness 的心跳。它实现的是 Thought-Action-Observation（TAO）循环，也叫 ReAct loop。循环步骤是：拼 prompt、调 LLM、解析输出、执行 tool call、把结果回填、重复，直到结束。

机制上往往就是一个 while 循环。复杂度不在循环本身，而在循环所管理的一切。Anthropic 把它们的运行时形容成「dumb loop」——所有智能都在模型里，harness 只管「下一轮、再下一轮」。

##### 2\. 工具（Tools）

工具是 agent 的双手。它们以 schema（名称、描述、参数类型）的形式定义，被注入 LLM 的 context，让模型知道有哪些手段可用。工具层负责注册、schema 校验、参数提取、沙箱执行、结果捕获，以及把结果格式化成 LLM 能读的 observation。

Claude Code 提供六大类工具：文件操作、搜索、执行、Web 访问、代码智能、subagent 派生。OpenAI 的 Agents SDK 支持 function tool（通过 `function_tool`）、托管工具（WebSearch、CodeInterpreter、FileSearch）和 MCP server 工具。

##### 3\. 记忆（Memory）

记忆在多个时间尺度上运作。短期记忆是单次会话内的对话历史；长期记忆要跨会话保留：Anthropic 用 `CLAUDE.md` 项目文件和自动生成的 `MEMORY.md` 文件；LangGraph 用按命名空间组织的 JSON Store；OpenAI 支持基于 SQLite 或 Redis 的 Sessions。

Claude Code 实现了三级层次结构：一份轻量索引（每条约 150 字符，始终加载）、按需拉取的详细主题文件，以及只能通过搜索访问的原始 transcript。

##### 4\. context 管理（Context management）

很多 agent 在这一环上悄无声息地崩掉。核心问题叫 context rot：关键内容一旦掉在 context window 的中段位置，模型表现会下降 30% 以上。

哪怕是百万 token 量级的 window，随着 context 增长，指令遵循能力也会退化。

生产环境里的几种应对策略：

-   compaction：在接近上限时对对话历史做摘要（Claude Code 保留架构决策和未解决的 bug，丢掉冗余的工具输出）

-   observation masking：JetBrains 的 Junie 会把旧的工具输出隐藏掉，但保留 tool call 可见

-   just-in-time 检索：维护一份轻量级标识符，按需动态加载数据（Claude Code 用 grep、glob、head、tail，而不是直接整文件加载）

-   subagent 派遣：每个 subagent 大量探索，但只把 1000 到 2000 token 的压缩摘要返回上来


Anthropic 的 context 工程指南把目标讲得很清楚：找到一组尽可能小、又能最大化预期结果概率的高信噪 token。

##### 5\. prompt 构造（Prompt construction）

这一步拼出模型每一步实际看到的内容。结构是层级化的：system prompt、工具定义、记忆文件、对话历史，再加上当前的用户消息。

OpenAI 的 Codex 用了一套严格的优先级栈：服务端控制的 system message（最高优先级）、工具定义、developer 指令、user 指令（层层级联的 `AGENTS.md`，上限 32 KiB），最后才是对话历史。

##### 6\. 输出解析（Output parsing）

现代 harness 依赖的是原生 tool calling——模型返回的是结构化的 `tool_calls` 对象，而不是要再去解析的自由文本。

harness 检查一下：有没有 tool call？有就执行并继续循环；没有就给出最终答案。

对于结构化输出，OpenAI 和 LangChain 都通过 Pydantic 模型支持 schema 约束的响应。

至于像 RetryWithErrorOutputParser 这类老办法（把原始 prompt、失败的补全和解析错误一起回喂给模型），在边缘场景里仍然可用。

##### 7\. 状态管理（State management）

LangGraph 把状态建模为流经图节点的类型化字典，由 reducer 来合并更新。

checkpoint 发生在 super-step 边界上，从而支持中断后恢复以及 time-travel 调试。

OpenAI 提供四种互斥的策略：应用层记忆、SDK Sessions、服务端 Conversations API，或者轻量级的 previous\_response\_id 串接。Claude Code 走的是另一条路：把 git commit 当 checkpoint，把 progress 文件当结构化便签。

##### 8\. 错误处理（Error handling）

为什么这一环要紧？一个 10 步的流程，哪怕每步成功率高达 99%，端到端成功率也只剩约 90.4%——这是叠乘出来的。

LangGraph 区分四类错误：瞬时错误（带退避重试）、LLM 可恢复错误（把错误作为 ToolMessage 返回，让模型自行调整）、用户可修复错误（中断流程等待人工输入）、未预期错误（向上抛出供调试）。Anthropic 在 tool handler 内部就捕获失败，把它们作为错误结果返回，让循环继续跑。Stripe 的生产 harness 把重试上限设在 2 次。

##### 9\. guardrail 与安全

OpenAI 的 SDK 实现了三层 guardrail：输入 guardrail（在第一个 agent 上运行）、输出 guardrail（在最终输出上运行）、工具 guardrail（在每一次工具调用上运行）。

还有一个「tripwire」机制——一旦触发，agent 立刻停下。

Anthropic 在架构层面把权限执行和模型推理分开。模型负责决定要尝试什么，工具系统决定哪些被允许。Claude Code 把大约 40 个独立的工具能力分别加了门控，分三个阶段：项目加载时建立信任、每次 tool call 前做权限检查、高风险操作明确要求用户确认。

##### 10\. 验证循环（Verification loops）

这一环把玩具 demo 和生产级 agent 分开。Anthropic 推荐三种做法：基于规则的反馈（测试、linter、类型检查器）、视觉反馈（针对 UI 任务，用 Playwright 截图）、LLM-as-judge（让另一个 subagent 评估输出）。

Claude Code 的作者 Boris Cherny 指出：给模型一种验证自身工作的途径，能把质量提高 2 到 3 倍。

##### 11\. subagent 编排

Claude Code 支持三种执行模型：Fork（与父级 context 字节级一致的副本）、Teammate（独立的终端面板，通过文件式邮箱通信）、Worktree（独立的 git worktree，每个 agent 一个隔离的分支）。

OpenAI 的 SDK 支持 agents-as-tools（专才 agent 处理一个有边界的子任务）和 handoff（专才 agent 接管全部控制权）。LangGraph 把 subagent 实现为嵌套的状态图。

#### 一次完整循环的逐步走读

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/06.webp)

知道了组件，再来追踪一下它们是怎么在单次循环里协同工作的。

-   步骤 1（Prompt 拼装）：harness 拼出完整输入——system prompt + 工具 schema + 记忆文件 + 对话历史 + 当前用户消息。重要的 context 放在 prompt 的开头和结尾（也就是「Lost in the Middle」研究的发现）。

-   步骤 2（LLM 推理）：拼好的 prompt 发到模型 API。模型生成输出 token：文本、tool call 请求，或者两者皆有。

-   步骤 3（输出分类）：如果模型只产出了文本、没有 tool call，循环结束；如果请求了 tool call，进入执行环节；如果请求 handoff，更新当前 agent 并重启。

-   步骤 4（工具执行）：对每个 tool call，harness 校验参数、检查权限、在沙箱环境内执行、捕获结果。只读操作可以并发跑，写操作只能串行。

-   步骤 5（结果打包）：把工具结果格式化成 LLM 能读的消息。错误被捕获并作为错误结果返回，让模型可以自我纠正。

-   步骤 6（context 更新）：结果被追加到对话历史。如果接近 context window 上限，harness 触发 compaction。

-   步骤 7（循环）：回到步骤 1。重复直至终止。


终止条件是分层的：模型产出无 tool call 的响应、超过最大轮次上限、token 预算耗尽、guardrail tripwire 触发、用户中断，或者返回了一个安全拒答。一个简单问题可能 1 到 2 轮就结束，一个复杂的重构任务则可能跨越很多轮、串联起几十次 tool call。

对于跨多个 context window 的长程任务，Anthropic 设计了一种两阶段的「Ralph Loop」模式。

它先用一个 Initializer Agent 把环境准备好（init 脚本、progress 文件、特性清单、初始 git commit），后续每个会话都由一个 Coding Agent 接手：读取 git log 和 progress 文件来定位自己当前在哪、挑出优先级最高的未完成特性、动手做、提交、写总结。

文件系统提供了跨 context window 的连续性。

#### 各框架是怎么实现这一模式的

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/07.webp)

Anthropic 的 Claude Agent SDK 把 harness 暴露成一个 `single query()` 函数——它会创建出 agentic loop，并返回一个异步迭代器流式输出消息。

它的运行时就是「dumb loop」，所有智能都在模型里。Claude Code 走的是 Gather-Act-Verify 循环：收集 context（搜索文件、读代码）、采取行动（编辑文件、跑命令）、验证结果（跑测试、检查输出）、再循环。

OpenAI 的 Agents SDK 通过 Runner 类来实现 harness，支持三种模式：async、sync、streamed。

这个 SDK 是「code-first」的——workflow 逻辑用原生 Python 表达，而不是某种图 DSL。Codex harness 在此基础上加了一套三层架构：Codex Core（agent 代码 + 运行时）、App Server（双向 JSON-RPC API），以及客户端形态（CLI、VS Code、Web 应用）。所有形态共享同一个 harness——所以「Codex 的模型在 Codex 的界面上感觉就是比在普通聊天窗口里更顺手」。

LangGraph 把 harness 建模成显式的状态图：两个节点（`llm_call` 和 `tool_node`）由一条条件边连接——有 tool call 就走向 tool\_node，没有就走向 END。

LangGraph 是从 LangChain 的 AgentExecutor 演化出来的。AgentExecutor 在 v0.2 被弃用，原因是难以扩展、又缺乏多 agent 支持。LangChain 的 Deep Agents 明确用了「agent harness」这个说法：内置工具、规划（write\_todos 工具）、用于 context 管理的文件系统、subagent 派生，以及持久化记忆。

CrewAI 实现的是基于角色的多 agent 架构：Agent（包在 LLM 外面的 harness，由角色、目标、背景、工具定义）、Task（工作的基本单位）、Crew（agent 的集合体）。CrewAI 的 Flows 层又加了一条「确定性骨架，关键处保留智能」的脉络——它管路由和校验，Crew 负责自主协作。

#### 脚手架的类比

施工脚手架是一种临时基础设施，让工人能去到原本够不到的位置。它不参与建造，但少了它，工人就上不了高层。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/08.webp)

关键的洞察是：脚手架在建筑完工后会被拆掉。模型越强，harness 的复杂度就应该越低。Manus 在 6 个月内被重写了 5 次，每一次都在剔除复杂度。复杂的工具定义变成了通用的 shell 执行；「管理型 agent」变成了简单的结构化 handoff。

这指向一条 co-evolution 原则：模型如今是带着特定 harness 一起做 post-training 的。Claude Code 的模型学会了使用它训练时所用的那套 harness——换掉工具实现可能让性能下降，就是因为这种紧耦合。

harness 设计的「面向未来检验」是这样一句话：如果换上更强的模型，性能能跟着上去，而不必给 harness 加任何复杂度，那这个设计就是稳的。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/09.webp)

#### Harness 定义的七个决策

每个 harness 架构师都要面对七个选择：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@e558c3134ee7187d876ad177289c91da6685ae56/ai-insights/2026-05/27/images/the-anatomy-of-an-agent-harness/10.webp)

1.  单 agent vs. 多 agent。Anthropic 和 OpenAI 都建议先把单 agent 推到极限。多 agent 系统会带来额外开销（路由要多调 LLM、handoff 过程中 context 丢失）。只有当工具重叠超过 ~10 个、或者明确存在彼此分离的任务域时，才考虑拆开。

2.  ReAct vs. plan-and-execute。ReAct 在每一步都把推理和动作交织在一起（灵活，但每步成本更高）。plan-and-execute 把规划和执行分开。LLMCompiler 的报告显示它比顺序执行的 ReAct 快 3.6 倍。

3.  context window 管理策略。生产环境里有五种思路：按时间清理、对话摘要、observation masking、结构化笔记、subagent 派遣。ACON 的研究表明，通过优先保留推理轨迹而非原始工具输出，可以在保留 95%+ 准确率的同时削减 26% 到 54% 的 token。

4.  验证循环设计。计算式验证（测试、linter）提供确定性的真值；推断式验证（LLM-as-judge）能抓住语义问题，但会增加延迟。Martin Fowler 在 Thoughtworks 的团队把这二者归纳为 guide（前馈，在动作前先做引导）vs. sensor（反馈，在动作后再观察）。

5.  权限与安全架构。宽松型（快但有风险，多数操作自动放行）vs. 严格型（安全但慢，每个操作都要审批）。怎么选取决于部署场景。

6.  工具范围策略。工具越多，性能往往越差。Vercel 把 v0 里 80% 的工具砍掉，结果更好了。Claude Code 借助懒加载实现了 95% 的 context 削减。原则是：只暴露当前这一步所需的最小工具集。

7.  harness 厚度。多少逻辑放在 harness 里、多少留给模型？Anthropic 押的是「薄 harness + 模型进步」；以图为基础的框架押的是「显式控制」。Anthropic 会定期把 Claude Code harness 里的规划步骤删掉，因为新版模型已经把那种能力内化了。


#### Harness 就是产品

两个用了同样模型的产品，性能可能因为 harness 设计不同而天差地别。TerminalBench 的证据非常清楚——只换 harness，agent 的排名就能挪 20 位以上。

harness 不是一个已经解决的问题，也不是某种通用层。真正的硬工程都在这里：把 context 当成稀缺资源来管理、设计能在错误叠乘前就抓住失败的验证循环、构建提供连续性又不带幻觉的记忆系统，以及做出架构上的取舍——到底建多少脚手架、留多少给模型自己来。

整个领域正朝着「更薄的 harness」演进，因为模型在变强。但 harness 本身不会消失。哪怕是最强的模型，也需要有东西来管它的 context window、执行它的 tool call、持久化它的状态、验证它的工作。

下一次 agent 跑挂了，先别怪模型，去看 harness。

感谢阅读！
