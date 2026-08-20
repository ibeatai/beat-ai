# 1.1 Agent Harness 剖析

> 原文：[The Anatomy of an Agent Harness](https://blog.dailydoseofds.com/p/the-anatomy-of-an-agent-harness)，作者：Avi Chawla

深入解读 Anthropic、OpenAI、Perplexity 与 LangChain 究竟在构建什么。

一个 **ReAct loop**、几个工具，加上写得不错的 system prompt，做出 demo 时往往能走得相当远。

但任务一旦需要 10+ 步，事情就会崩塌：模型忘了三步之前做过什么，工具调用悄无声息地失败，context window 被垃圾塞满。

![图1](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/learn-ai/agent-harness/imgs/anatomy/01.png)

问题不在模型本身，而在模型周围的一切。

LangChain 用一次实验证明了这点：只改 LLM 外层的基础设施（模型不变、权重不变），TerminalBench 2.0 的排名就从前 30 名开外冲进第 5 名。

另一项研究让 LLM 自己去优化这层基础设施，通过率拉到 76.4%，超过了人工设计的方案。

这层基础设施现在有了名字：agent harness。

## 什么是 Agent Harness？

这个词是在 2026 年初被正式定下来的，但概念早已存在。

harness 就是包裹 LLM 的整套软件基础设施，包括编排循环、工具、记忆、上下文管理、状态持久化、错误处理和安全护栏。

Anthropic 的 Claude Code 文档说得更直白：这个 SDK 就是"驱动 Claude Code 的 agent harness"。

LangChain 的 Vivek Trivedy 给了一个特别经典的概括："不是模型的部分，就是 harness。"

换种说法："agent" 是涌现出来的行为——用户面对的那个会朝目标推进、会用工具、会自我纠错的实体。harness 才是产生这种行为的机器。有人说"我做了一个 agent"，意思其实是搭了一个 harness，然后把它指向某个模型。

Beren Millidge 在 2023 年的文章里把这个类比说得很到位：

* 原始 LLM 是一颗没有 RAM、没有磁盘、没有 I/O 的 CPU。

* context window 是 RAM（快但容量有限）。

* 外部数据库是磁盘（大但慢）。

* 工具集成是设备驱动。

harness 就是操作系统。

![图2](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/learn-ai/agent-harness/imgs/anatomy/02.png)

## 工程的三个层级

围绕模型有三个同心的工程层级：

* Prompt 工程负责打磨模型收到的指令。

* Context 工程负责决定模型在什么时候看到什么。

* Harness 工程把前两者全包进去，再加上整套应用层基础设施：工具编排、状态持久化、错误恢复、校验回路、安全策略执行、生命周期管理。

harness 不是 prompt 外面套一层壳，而是让自主 agent 行为成为可能的那一整套系统。

![图3](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/learn-ai/agent-harness/imgs/anatomy/03.png)

## 生产级 Harness 的 11 个组件

综合 Anthropic、OpenAI、LangChain 以及更广泛的工程实践社区，一个生产级 agent harness 由 11 个相互独立的组件构成。下面逐个看。

### 1. 编排循环

这是整个 harness 的心跳。它实现的是 Thought-Action-Observation（TAO）循环，也就是常说的 ReAct loop。每一轮的流程是：组装 prompt → 调 LLM → 解析输出 → 执行其中的工具调用 → 把结果喂回去 → 重复，直到完成。

从机制上看，它通常就是一个 while 循环。复杂度都堆在循环管理的那些东西上，而不在循环本身。Anthropic 把自家 runtime 形容为"笨循环"——所有智能都在模型里，harness 只负责管 turn。

### 2. 工具

工具是 agent 的手。每个工具都被定义成 schema（名称、描述、参数类型），注入到 LLM 的 context 里，让模型知道有哪些手可用。工具层负责注册、schema 校验、参数提取、沙箱执行、抓结果，再把结果格式化成 LLM 能读懂的 observation。

Claude Code 提供六类工具：文件操作、搜索、执行、网络访问、代码理解、subagent 创建。OpenAI 的 Agents SDK 支持三类：函数工具（通过 `function_tool`）、托管工具（WebSearch、CodeInterpreter、FileSearch）、MCP 服务工具。

### 3. 记忆

记忆运作在多个时间尺度上。短期记忆是单次会话内的对话历史；长期记忆能跨会话留存：Anthropic 用项目级的 `CLAUDE.md` 文件和自动生成的 `MEMORY.md` 文件；LangGraph 用按命名空间组织的 JSON Store；OpenAI 用 Session，后端是 SQLite 或 Redis。

Claude Code 走的是三层结构：一份轻量索引（每条约 150 字符，常驻加载）、按需拉取的主题详细文件、以及只能通过搜索访问的原始 transcript。

### 4. 上下文管理

很多 agent 就是死在这一环——还死得悄无声息。核心问题叫 context rot：关键内容落到 context window 中段时，模型表现会下降 30%+。

哪怕是百万 token 的窗口，随着 context 变长，指令遵循能力也会下滑。

生产环境常用的应对策略包括：

* Compaction：快触顶时把对话历史压成摘要（Claude Code 会保留架构决策和未解决的 bug，把冗余的工具输出丢掉）

* Observation masking（观察值遮蔽）：JetBrains 的 Junie 会隐藏旧的工具输出，但保留工具调用本身可见

* 即时检索（just-in-time retrieval）：只保留轻量标识符，需要时再动态加载数据（Claude Code 用 grep、glob、head、tail，而不是直接把整个文件加载进来）

* Subagent 分派：每个 subagent 自己去广泛探索，但只回传 1,000 到 2,000 token 的压缩摘要

Anthropic 的 context 工程指南把目标说得很明确：找到尽可能小的高信号 token 集合，让目标结果出现的概率最大化。

### 5. Prompt 组装

这一步负责把模型每一轮实际看到的内容拼起来。结构是分层的：system prompt、工具定义、记忆文件、对话历史、当前用户消息。

OpenAI 的 Codex 走的是严格的优先级栈：服务端控制的系统消息（最高优先级）→ 工具定义 → 开发者指令 → 用户指令（层叠的 `AGENTS.md` 文件，上限 32 KiB）→ 对话历史。

### 6. 输出解析

现代 harness 都依赖原生工具调用——模型直接返回结构化的 `tool_calls` 对象，而不是需要再去解析的自由文本。

harness 只判断一件事：有没有工具调用？有就执行、继续循环；没有就给出最终答案。

至于结构化输出，OpenAI 和 LangChain 都支持通过 Pydantic 模型对响应做 schema 约束。

像 RetryWithErrorOutputParser 这种老办法——把原始 prompt、失败的输出和解析错误一起再喂回模型——也还留着，应对边缘场景。

### 7. 状态管理

LangGraph 把状态建模成带类型的字典，在图节点之间流动，更新由 reducer 合并。

Checkpoint 发生在 super-step 边界上，支持中断后续跑，也支持时间回溯式调试。

OpenAI 提供四种互斥的策略：应用层内存、SDK Session、服务端 Conversations API，或者用轻量的 previous_response_id 串起来。Claude Code 走的是另一条路：用 git commit 作 checkpoint，用 progress 文件作结构化草稿纸。

### 8. 错误处理

为什么这一项重要：一个 10 步的流程，每步成功率 99%，端到端成功率算上复合效应也只有约 90.4%。

LangGraph 把错误分成四类：临时性错误（带退避地重试）、模型可恢复错误（作为 ToolMessage 把错误返回给模型，让它自己调整）、用户可修复错误（中断、等人工输入）、以及非预期错误（往上抛出来给开发者调试）。Anthropic 的做法是在工具 handler 内部捕获失败、以 error result 返回，让循环继续跑。Stripe 的生产 harness 把重试次数最高限制在两次。

### 9. 护栏与安全

OpenAI 的 SDK 实现了三层护栏：输入护栏（在第一个 agent 上运行）、输出护栏（在最终输出上运行）、工具护栏（每次工具调用时运行）。

触发时，"绊线"机制会立即叫停 agent。

Anthropic 在架构上把权限执行和模型推理拆开：模型决定要尝试什么，工具系统决定什么被允许。Claude Code 对约 40 项工具能力分别独立设闸门，分三个阶段：加载项目时建立信任、每次工具调用前做权限检查、高风险操作必须用户显式确认。

### 10. 校验回路

玩具 demo 和生产级 agent 的分水岭就在这里。Anthropic 推荐三种做法：规则化反馈（测试、linter、类型检查器）、视觉反馈（UI 任务通过 Playwright 截图）、LLM-as-judge（让另一个 subagent 来评判输出）。

Claude Code 的作者 Boris Cherny 说过：给模型一种校验自己工作成果的手段，能把质量提升 2 到 3 倍。

### 11. Subagent 编排

Claude Code 支持三种执行模型：Fork（父 context 的逐字节副本）、Teammate（独立的终端面板，通过文件邮箱通信）、Worktree（自己一个 git worktree，每个 agent 一条独立分支）。

OpenAI 的 SDK 支持两种：agents-as-tools（让专家 agent 处理有限范围的子任务）和 handoff（把全部控制权交给专家 agent）。LangGraph 则把 subagent 实现为嵌套的状态图。

![图4](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/learn-ai/agent-harness/imgs/anatomy/04.png)

## 一次循环逐步拆解

知道了这些组件，再看它们在一个循环里如何协作。

* 第 1 步（组装 Prompt）：harness 拼出完整输入——system prompt + 工具 schema + 记忆文件 + 对话历史 + 当前用户消息。重要 context 放在 prompt 的头部和尾部（这是 "Lost in the Middle" 研究的结论）。

* 第 2 步（LLM 推理）：拼好的 prompt 发给模型 API，模型生成输出 token：文本、工具调用请求，或两者皆有。

* 第 3 步（输出分类）：模型只产出文本、没有工具调用 → 循环结束；请求了工具调用 → 进入执行；请求了 handoff → 切换当前 agent 并重启循环。

* 第 4 步（工具执行）：对每个工具调用，harness 校验参数、检查权限、在沙箱里执行、抓取结果。只读操作可以并发，会改状态的操作串行执行。

* 第 5 步（结果打包）：工具结果被格式化为 LLM 能读的消息。错误也会捕获并作为 error result 返回，让模型自纠。

* 第 6 步（更新 Context）：结果追加进对话历史。如果接近 context window 上限，harness 触发 compaction。

* 第 7 步（循环）：回到第 1 步，重复直到终止。

终止条件是分层的：模型给出不带工具调用的回复、超过最大 turn 上限、token 预算耗尽、护栏绊线触发、用户中断，或者返回了安全策略的拒答。一个简单问题可能 1 到 2 个 turn 搞定；一个复杂的重构任务则可能跨几十个 turn、串起几十次工具调用。

对于跨多个 context window 的长任务，Anthropic 设计了一种两阶段的 "Ralph Loop" 模式。

先由一个 Initializer Agent 搭好环境（init 脚本、progress 文件、特性清单、初始 git commit）；之后每个会话里的 Coding Agent 都去读 git log 和 progress 文件给自己定位、挑出优先级最高的未完成特性、动手做、commit、写摘要。

文件系统提供了跨 context window 的连续性。

## 各家框架的实现方式

![图5](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/learn-ai/agent-harness/imgs/anatomy/05.png)

Anthropic 的 Claude Agent SDK 通过一个 `query()` 函数把 harness 暴露出来：该函数创建 agent 循环，返回一个流式吐消息的 async iterator。

这个 runtime 就是一个"笨循环"，所有智能都在模型里。Claude Code 跑的是 Gather-Act-Verify 循环：收集 context（搜索文件、读代码）→ 采取行动（改文件、跑命令）→ 校验结果（跑测试、查输出）→ 重复。

OpenAI 的 Agents SDK 通过 Runner 类实现 harness，提供三种模式：async、sync、streamed。

这个 SDK 是"代码优先"——工作流逻辑直接用原生 Python 写，而不是用图式 DSL。Codex harness 在此基础上做了三层架构扩展：Codex Core（agent 代码 + runtime）、App Server（双向 JSON-RPC API）、客户端表面层（CLI、VS Code、Web 应用）。所有表面层共享同一个 harness——这就是为什么"Codex 模型在 Codex 客户端里的体感比在通用聊天窗口里要好"。

LangGraph 把 harness 显式建模成状态图：两个节点（`llm_call` 和 `tool_node`）由一条条件边相连——有工具调用就路由到 tool_node，没有就路由到 END。

LangGraph 是从 LangChain 的 AgentExecutor 演化来的，后者在 v0.2 中被废弃——理由是难扩展、缺多 agent 支持。LangChain 的 Deep Agents 直接用了 "agent harness" 这个词，里面有内置工具、规划（write_todos 工具）、用于 context 管理的文件系统、subagent 创建、持久化记忆。

CrewAI 走的是基于角色的多 agent 架构：Agent（LLM 外面的 harness，由角色、目标、背景故事、工具来定义）、Task（工作单元）、Crew（agent 的集合）。CrewAI 的 Flows 层补了一层"确定性骨架，在该智能的地方智能"——它管路由和校验，Crew 则负责自主协作。

## 脚手架的比喻

![图6](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/learn-ai/agent-harness/imgs/anatomy/06.png)

建筑脚手架是一种临时性的基础设施，让工人能去够本来够不着的位置。它自己不参与施工，但少了它，工人上不去高层。

关键洞察是：楼盖好了，脚手架就拆掉。模型变强了，harness 复杂度就应该往下减。Manus 在六个月里重写了五次，每次都在减复杂度——复杂的工具定义换成通用 shell 执行，"管理 agent" 换成简单的结构化 handoff。

这指向一个共同演化原则：现在模型在 post-training 阶段就已经把具体的 harness 纳入回路。Claude Code 的模型学的就是它训练时配的那个 harness。换工具实现可能会让性能下降——耦合就是这么紧。

![图7](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/learn-ai/agent-harness/imgs/anatomy/07.png)

检验 harness 设计是否"未来友好"有个判据：模型越强、性能越好，而 harness 复杂度不需要加——能做到这点，设计就站得住。

![图8](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/learn-ai/agent-harness/imgs/anatomy/08.png)

## Harness 设计的七个抉择

每个 harness 架构师都要面对七个选择：

1. 单 agent vs. 多 agent。Anthropic 和 OpenAI 都建议先把单 agent 压榨到极致。多 agent 系统额外开销不小（路由需要额外 LLM 调用、handoff 时会丢 context）。只有当工具数量重叠超过约 10 个、或任务领域明显分离时，才拆。

2. ReAct vs. plan-and-execute。ReAct 在每一步交错进行推理和行动（灵活，但单步成本更高）；plan-and-execute 则把规划和执行拆开。LLMCompiler 报告的提速是顺序 ReAct 的 3.6 倍。

3. Context window 管理策略。生产环境的五种做法：按时间清理、对话摘要、observation masking、结构化记录、subagent 分派。ACON 的研究表明：把推理轨迹的优先级抬到原始工具输出之上，能减 26% 到 54% 的 token，同时把准确率保住 95%+。

4. 校验回路设计。计算式校验（测试、linter）给出确定性真值；推理式校验（LLM-as-judge）能抓语义问题但增加延迟。Martin Fowler 在 Thoughtworks 的团队把这一对概念叫做：guide（前馈，行动前引导）vs. sensor（反馈，行动后观察）。

5. 权限与安全架构。宽松式（快但有风险，绝大多数操作自动放行）vs. 严格式（安全但慢，每个操作都要批准）。选哪种取决于部署场景。

6. 工具范围策略。工具越多，性能往往越差。Vercel 从 v0 里砍掉了 80% 的工具，效果反而更好。Claude Code 靠 lazy loading 让 context 减少了 95%。原则就一条：只暴露当前这一步真正需要的最小工具集。

7. Harness 厚度。多少逻辑放在 harness 里、多少留给模型。Anthropic 押的是薄 harness + 模型进化；图式框架押的是显式控制。新模型版本一旦把规划能力内化，Anthropic 就会把 Claude Code harness 里相应的规划步骤删掉。

## harness 就是产品

两个用同一个模型的产品，仅凭 harness 设计的差异，表现就可能天差地别。TerminalBench 的证据很清楚：只改 harness，agent 排名就动了 20+ 位。

harness 既没被攻克、也不是商品化的薄层。真正难的工程都在这里：把 context 当稀缺资源来管理、设计能在错误复合之前就抓住的校验回路、做出连续却不幻觉的记忆系统，以及在"搭多少脚手架 vs. 留多少给模型"之间做架构押注。

模型变强，harness 整体趋势是越来越薄。但 harness 本身不会消失——再强的模型，也得有人去管它的 context window、执行它的工具调用、持久化它的状态、校验它的输出。

下次你的 agent 翻车时，别先怪模型，先去看 harness。
