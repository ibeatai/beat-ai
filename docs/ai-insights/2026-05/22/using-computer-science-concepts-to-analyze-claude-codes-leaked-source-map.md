---
title: 用计算机科学概念解读 Claude Code 泄露的 source map
author: Kaitai Dong
url: https://ai.gopubby.com/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map-7717dbdfb2de
translated: 2026-05-22
tags:
  - Artificial Intelligence
  - LLM
excerpt: 这个故事现在大家都知道了。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/01.webp
---

# 用计算机科学概念解读 Claude Code 泄露的 source map

## 一次代码级拆解：剖析它的 harness、记忆层级与能力边界。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/01.webp)

## 一次不寻常的代码泄露

这个故事现在大家都知道了。

2026 年 3 月 31 日，Anthropic 在一次 Claude Code 的 npm 更新中意外地把一个 `.map` source map 文件一并发布了出去。几分钟内，这件事就被人发现并迅速传开。

那 60 万行代码被人镜像、分析、移植到 Python 和其他语言，并上传到去中心化服务器上。其中最热门的一个分支是 GitHub 上的 [`claw-code`](https://github.com/instructkr/claw-code)，由 @realsigridjin 创建。

大多数源代码泄露之所以有意思，是因为它们揭露了秘密。而 Claude Code 的 source map 泄露之所以有意思，原因几乎相反：它让这个产品看起来*没那么*神奇，却*更加*令人印象深刻。泄露出来的不只是实现细节，而是**对模型周围运行时（runtime）的一次窥视**：**工具编排**、**prompt 布局**、**上下文压缩**、**记忆维护**、**子智能体模式**，以及**能力边界**。这正是它在众多工程师中如此迅速地掀起一轮又一轮讨论的原因。

> 泄露出来的不是一段神奇的 prompt，而是一套关于如何构建编程智能体的、行得通的理论。

最让我震撼的是——这就是一个严肃的编程智能体在你不再把 LLM 当作产品本身、而开始把它当作**一个更大系统中的一个组件**之后应有的样子。

Claude Code 读起来不像"一个带 shell 访问权限的聊天机器人"，它读起来像一个**运行时**。一旦你这样去读它，代码就会更快地变得说得通。

## 仓库的形态本身就告诉了你 Claude Code 是什么

在深入具体细节之前，Claude Code 的目录结构本身就说明了很多东西。

被还原出来的仓库目录树，以及那份架构指南，把 Claude Code 拆分成了一些可辨认的部分：工具、服务、记忆/上下文、协调器/子智能体、hook、流式输出，以及 UI 层。这是一个强烈的信号，说明这个产品不是围绕"一个超级 prompt"组织的，而是围绕一个管理**执行**、**状态**和**能力边界**的运行时来组织的。

```
ported-claude-code/src/
    ├── main.tsx      
    ├── tools/        
    ├── commands/     
    ├── services/     
    ├── utils/        
    ├── context/      
    ├── coordinator/  
    ├── assistant/    
    ├── buddy/        
    ├── remote/       
    ├── plugins/      
    ├── skills/       
    ├── voice/        
    ├── vim/          
    ...
```

这一点之所以重要，是因为你常常能通过**复杂度所在的位置**来判断一个产品*究竟是什么*。在较弱的智能体系统里，复杂度往往坍缩进 prompt 和临时拼凑的编排逻辑中。

而在这里，复杂度存在于模型周围的支撑结构里：

-   工作是如何被调度的
-   上下文是如何被裁剪的
-   工具是如何被暴露出来的
-   记忆是如何被存储的
-   权限是如何被强制执行的

这也是为什么 Claude Code 在实际使用中感觉超级扎实的原因之一。

这个 Python 版 `src/` 移植在这里很有用，相当于一张精简版的 X 光片。它并不是泄露出来的完整实现，也不该被当作完整实现来看待。但它确实足够干净地暴露出了系统的*形态*，让架构更容易讨论：一个查询引擎、一个 transcript 存储、一组显式的工具定义，以及一个独立的权限上下文。如果这个真实产品确实是一个运行时而非 prompt 包装器，那你正会期望看到这类文件。

![图 1：Claude Code 结构及其各组件概览 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/02.webp)

## harness 就是产品

如果说有一个想法比其他任何东西都更能解释清楚 Claude Code，那就是这个：**harness 就是产品**。

模型当然重要。但让 Claude Code 感觉强大的，是**模型被嵌入在一个控制预算、回合、transcript 状态、工具可见性、重试和压缩的运行时里**，而不是只把一个巨大的 prompt 丢给模型然后听天由命。这种"运行时优先"的解读，也正是其他分析反复指向的方向。

这个 Python 移植以一种非常紧凑的方式让这一点变得可见。`QueryEngineConfig` 定义了 `max_turns`、`max_budget_tokens`、`compact_after_turns`、`structured_output` 和 `structured_retry_limit`。光是这一点就已经是一条线索了。

这不是"调用模型并打印结果"的那种配置面，而是**一个在资源和格式约束下运行的调度器的配置面**。

```python
@dataclass(frozen=True)
class QueryEngineConfig:
    max_turns: int = 8
    max_budget_tokens: int = 2000
    compact_after_turns: int = 12
    structured_output: bool = False
    structured_retry_limit: int = 2
```

> **模型提供推理能力，harness 提供可治理性。**

再看 `submit_message()`。在它真正"作答"之前，它会检查 `mutable_messages` 是否已经达到 `max_turns`；如果达到了，它就会带着一个停止条件提前返回。在函数后面，它会计算用量、携带匹配到的命令和工具、跟踪权限拒绝、追加到 transcript 状态，并发出一个 `stop_reason`。

回合结束后，引擎可以调用 `compact_messages_if_needed()` 来收缩工作集。即便在这个极简的移植里，这也足以清楚地展现出运行时的本能：**每个回合都被当作一次有边界的操作，带有计量、状态变更和失败语义**。

```python
def submit_message(
    self,
    prompt: str,
    matched_commands: tuple[str, ...] = (),
    matched_tools: tuple[str, ...] = (),
    denied_tools: tuple[PermissionDenial, ...] = (),
) -> TurnResult:
    if len(self.mutable_messages) >= self.config.max_turns:
        output = f'Max turns reached before processing prompt: {prompt}'                return TurnResult(
            prompt=prompt,
            output=output,
            matched_commands=matched_commands,
            matched_tools=matched_tools,
            permission_denials=denied_tools,
            usage=self.total_usage,
            stop_reason='max_turns_reached',
        )
```

流式路径强化了同样的观点。`stream_submit_message()` 不只是产出"一些输出"，它发出的是**结构化的生命周期事件**，比如 `message_start`、命令和工具匹配、权限拒绝、增量（delta），以及 `message_stop`。这意味着**流式输出**被当作**运行时协议的一部分**，而不是事后才拼上去的 UI 糖衣。

```python
def stream_submit_message(
    self,
    prompt: str,
    matched_commands: tuple[str, ...] = (),
    matched_tools: tuple[str, ...] = (),
    denied_tools: tuple[PermissionDenial, ...] = (),
):
    yield {'type': 'message_start', 'session_id': self.session_id, 'prompt': prompt}
    if matched_commands:
        yield {'type': 'command_match', 'commands': matched_commands}
    if matched_tools:
        yield {'type': 'tool_match', 'tools': matched_tools}
    if denied_tools:
        yield {'type': 'permission_denial', 'denials': [denial.tool_name for denial in denied_tools]}
    result = self.submit_message(prompt, matched_commands, matched_tools, denied_tools)
    yield {'type': 'message_delta', 'text': result.output}
    yield {
        'type': 'message_stop',
        'usage': {'input_tokens': result.usage.input_tokens, 'output_tokens': result.usage.output_tokens},
        'stop_reason': result.stop_reason,
        'transcript_size': len(self.transcript_store.entries),
    }
```

到这里，"智能体工程"开始看起来更像系统工程了。

难的部分不是让一个 LLM 输出某种看似合理的东西。

难的部分是去治理：

-   它**何时**应该**行动**
-   它应该**携带多少上下文**
-   它**被允许触碰什么**
-   它应该**如何失败**
-   系统应该**如何恢复**

Claude Code 看起来强大，是因为这些工作中有很大一部分似乎都投入到了 harness 里。

> Claude Code 令人印象深刻，不是因为它调用了一个 LLM，而是因为它治理着这个 LLM 被允许运行的那些条件。

## 用 Gall 定律解读 Claude Code

Gall 定律是解读这类系统最干净利落的方式之一。

![图 2：系统设计中 Gall 定律的定义 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/03.webp)

它通常的表述是：**一个能运转的复杂系统，无一例外，都是从一个能运转的简单系统演化而来的**。这条定律的实际含义很重要。当你用 Gall 定律去读代码时，你寻找的不是完美的对称性，而是**系统为了应对真实瓶颈而逐层累积**的证据。这是一种很不一样的评判架构的方式。

Claude Code 几乎完美地契合这个模式。它有一条演化路径，与一段时间内所做的改进密切吻合：

-   prompt 前缀变得对缓存友好（cache-aware），因为重复 token 太贵了；
-   上下文控制演变成了一条多阶段的压缩流水线，因为长会话变得不稳定；
-   记忆获得了维护机制，因为只追加（append-only）的记忆变成了噪声；
-   工具获得了更明确的边界，因为与世界的交互有风险；
-   编排扩展到了单个智能体之外，因为有些任务需要专门化和隔离；
-   功能门控（feature gate）变得更硬，因为仅限内部的能力需要更强的边界。

这些都不像是装饰性的，而像是压力驱动的演化。

> Claude Code 看起来不像一个一次性设计出来的系统，它看起来像一个不断在解决真实瓶颈的系统。

![图 3：一段时间内为这个 agentic 系统所做的改进与增强领域 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/04.webp)

### Gall 定律深入 1：上下文控制最简单的可用版本

Python 移植里最有"Gall 定律味道"的代码是 `compact_messages_if_needed()`。

它简单到几乎无聊：如果 `len(self.mutable_messages)` 大于 `self.config.compact_after_turns`，就把消息列表切片，只保留最后 `compact_after_turns` 项，然后调用 `self.transcript_store.compact(...)`。

`TranscriptStore.compact()` 同样朴素：如果 `entries` 比 `keep_last` 长，就只保留最后 `keep_last` 个条目。

![图 4：`compact_messages_if_needed()` 与 `TranscriptStore.compact()` 的并排代码片段 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/05.webp)

为什么这有意思？因为它正是 Gall 定律的微缩版。

在你拥有一套完整的层级体系——带有工具结果预算、微压缩（micro-compact）、上下文坍缩（context collapse）、自动压缩（auto-compact）和后台记忆精炼——之前，你通常先有的是一个能用的简单策略：保留近期历史，丢弃其余。

### Gall 定律深入 2：有边界的失败是一种成熟的本能

另一个能看出演化感的地方，是有边界的失败。`QueryEngineConfig` 包含 `structured_retry_limit`，而 `_render_structured_output()` 只会按这个配置的次数循环尝试，之后就放弃。`submit_message()` 也强制执行一个 `max_turns` 边界，而配置里还包含一个 `max_budget_tokens` 上限。

这些都是小细节，但它们揭示出一种成熟的本能：**不要让格式化重试、长会话或预算漂移无限制地跑下去**，只因为模型强大到可以一直跑。

![图 5：重试行为被显式地设了边界，而不是任由其无限制运行。 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/06.webp)

这些具体的 Python 函数和那个更丰富的 `autoCompact` 失败断路器并不相同，但它们显然出自同一族设计选择：**给重试设边界**、**给成本设边界**，并在维护路径开始变得浪费时显式地停止工作。

### Gall 定律深入 3：prompt 设计变成了基础设施

泄露代码里最有力的例子之一，是**静态/动态 prompt 拆分**。

这里的思路是把系统 prompt 拆成两类，即静态类（稳定的工具定义、人设和策略归属于此）和动态类（会话状态和不断变化的用户上下文归属于此），二者由一个清晰的边界 `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` 分隔。

稳定的部分永远不会变，所以这部分可以放进 prompt 缓存。每次需要调用它时，它都会命中 Anthropic 的 prompt 缓存，从而降低延迟和 token 成本。这就是为什么在接入智能体时，要始终**把静态部分放在最前面，把动态部分留在最后**。

这正是 Gall 定律所预言的那种层次：一旦 token 成本和延迟成为硬约束，prompt 布局就不再只是措辞，而成了基础设施。

![图 6：一旦稳定前缀被共享、被缓存、像运行时资源一样被计费，prompt 布局就变成了基础设施。 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/07.webp)

换句话说：Claude Code 似乎把 prompt 中昂贵、可复用的部分放在前面，把会话专属、会破坏缓存的内容推到一个显式边界之后。

### Gall 定律深入 4：代码看起来像打了补丁，因为系统看起来"住过人"

这是一个更宽泛的观点。Claude Code 看起来不像某人一次性设计出来的、一尘不染的全新架构。

它看起来像一个不断撞上真实瓶颈、然后一个一个打补丁的产品：重复的 prompt 开销太多、上下文增长太多、记忆熵太多、环境中的能力（ambient capability）太多、隐藏的失败成本太多。

这通常是个好兆头。那些经受住真实使用考验而存活下来的系统，往往恰恰会累积起这种实用主义式的不均匀感。

所以 Gall 定律不只是一句贴在 Claude Code 上的好引言，它是一种有用的方式，用来解释为什么这个架构既有层次感、又令人信服。

## 用最小授权原则解读 Claude Code

如果说 Gall 定律解释了这类系统是***如何***成长起来的，那么**最小授权原则**（Principle of Least Authority，PoLA）就解释了为什么它的某些边界感觉如此有纪律。

用大白话说，PoLA 很简单：**每个组件都只应获得它实际所需的最小授权**。在普通软件里，这能缩小爆炸半径。而在智能体系统里，它更为重要，因为规划者（planner）强大、带概率性、而且常常过度自信。你**不会**想让一个模型拥有宽泛的、弥漫式的权力，然后只靠一句礼貌的指令来约束它。

![图 7：最小授权原则（PoLA）的概念 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/08.webp)

在这个视角下，Claude Code 的工具设计就变得很说得通了。

### PoLA 深入 1：工具是显式的能力，不是弥漫式的权力

在 Python 移植里，`Tool.py` 定义了一个冻结的 `ToolDefinition` 数据类，带有 `name` 和 `purpose`，而 `DEFAULT_TOOLS` 显式地枚举了可用的工具，比如 `port_manifest`。这看起来可能很微不足道，但从概念上说很重要。运行时把工具建模成一个个有名字的能力，而不是让模型去假定一个模糊的"我能做点事"的能力面。

![图 8：工具面是显式的而非弥漫式的，以及能力在被调用之前如何先被命名。 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/09.webp)

这让我想起 *syscall*（系统调用）的工作方式。在操作系统里，一个进程不会直接伸手去改变世界，它要通过**定义好的接口**来发起请求。Claude Code 似乎遵循同样的本能：

> 工具不是为了方便而存在的辅助函数；它们是运行时与外部世界之间被中介过的连接。

### PoLA 深入 2：权限检查与工具是否存在被分开建模

下一步甚至更重要。在 `permissions.py` 里，`ToolPermissionContext` 存储 `deny_names` 和 `deny_prefixes`，暴露一个 `from_iterables()` 辅助方法来规范化它们，并通过同时检查精确拒绝项和基于前缀的拒绝项来实现 `blocks(tool_name)`。这意味着**授权**没有被熔合进工具定义本身，它存在于一个**独立的授权对象**里。

![图 9：权限逻辑作为架构存在，而不只是作为 prompt 里的一句建议。 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/10.webp)

这种分离是 Python 移植里最清晰的最小授权信号之一。一个工具可以存在，但当前上下文是否应被允许使用它，要由另一个关注点更狭窄的对象来中介。

小小的模式，大大的想法！

### PoLA 深入 3：最强的授权边界是"不存在"

这就是公开泄露分析超出 Python 移植的地方了。

"告诉模型不要使用某项能力"和"把这项能力从构建产物里移除，让模型连提都提不出来"之间，应该有一道强烈的区分。

还存在一些仅限内部的功能，以及被死代码消除（dead-code elimination）处理掉的隐藏功能。这是纯粹的最小授权思维：**最安全的能力不是模型被警告远离的那种，而是模型根本看不到的那种**。

这也是为什么编译期消除是如此有力的设计动作。prompt 期的禁止仍然让能力在概念上存在着，而构建期的移除则在模型开始推理之前就缩小了它的行动空间。

## 记忆层级很可能才是真正的"独门秘方"

如果说 harness 解释了为什么 Claude Code 感觉受控，最小授权解释了为什么它感觉有纪律，那么**记忆架构则解释了为什么它能在更长的工作流中保持有用**，而不会坍缩成一摊烂泥。我认为这部分大多数人仍然低估了。一个好的设计不是"把所有东西都存下来"，而是结构化的、带宽感知的、并且刻意持怀疑态度的。

第一个大想法是：记忆是一个**索引**，不是存储。**始终加载的那一层**是**轻量的、像指针一样的**。真正的知识活在那条热路径之外，只在需要时才被取回。如果你把指针和数据负载（payload）混在一起，热路径几乎会立刻臃肿起来。Claude Code 的架构似乎避开了这个陷阱。

第二个大想法是**三层设计**：

-   一个极小的、始终加载的索引，
-   按需加载的主题文件，
-   不会被整体注入、只在需要时才被检索的 transcript。

> 让热层保持微小，让较冷的层远离关键路径，只在有具体理由时才把信息提升上来。

![图 10：Claude Code 的记忆系统。热记忆保持微小；较冷的知识只在有正当理由时才被取回。 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/11.webp)

第三个大想法是**严格的写入纪律**。系统先写入一个正规的文件，然后再更新索引，而不是把内容直接灌进索引本身。这是一个有力的设计选择。它让索引保持像指针一样，**保留了摘要与源之间的分离**，并减缓了那种害死许多智能体记忆系统的熵增。

第四个大想法是**陈旧性（staleness）**。如果记忆不再与现实相符，那记忆就是错的。这听起来理所当然，但它意味着一件许多系统并不做的事：**避免持久化那些可以从代码这类权威来源重新推导出来的事实**。Claude Code 避免存储从代码派生的事实，并**把记忆当作一种提示（hint）**，而非真相。这或许是整个架构里最棒的一个洞见。

它们**不**存什么，才是这里真正的设计信号。没有一大本调试轨迹的剪贴簿，没有一大堆可推导的代码结构，也没有仅仅因为模型碰巧注意到了什么就不加区分地持久化下来。这种克制是很难假装出来的。

## autoDream 是记忆的 GC，而不只是"记忆"

一旦你这样去读这套记忆系统，autoDream 就不再像一个古怪的功能，而开始看起来像智能体记忆的垃圾回收（garbage collection）。它可以被描述为后台整合：**合并重叠的记忆**、**对重复的论断去重**、**消解矛盾**、**把模糊的措辞规范化为更绝对的形式**，以及**激进地剪枝**。这不是记笔记，这是维护。

![图 11：autoDream 流水线看起来不像"存得更多"，更像"持续重写记忆，让它保持有用"。 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/12.webp)

这里的关键洞见是：**只追加的记忆会腐烂**。一个只会累积笔记的系统，最终会用残渣替换掉可用的知识。所以，让 Claude Code 的记忆架构感觉精巧的，并不是持久化本身，而是**愿意去重写和删除**。如果你从不重写记忆，那你其实并没有记忆，你有的是日志。

而且，最小授权在这里又一次出现了。据称，后台维护路径被隔离在一个分叉出来的、工具受限的子智能体里。这是一个很好的例子，展示了两个设计目标被同时解决：**保持主上下文干净**，以及**让维护路径的范围被严格收窄**。

## 最有意思的安全教训关乎来源（provenance），而非 prompt

现在我们来谈谈来源（provenance）。一旦一个系统开始对上下文进行摘要、压缩和重写，记忆设计就成了信任边界的一部分。风险不只是不受信任的数据进入系统，风险在于**不受信任的数据稍后带着被升级过的授权回来**——就好像它是用户偏好、稳定记忆，或受信任的指令一样。

藏在一个外部文件里的一段字符串是一回事。同样这段字符串被压缩进一份"摘要"里、现在读起来像是策略或用户反馈，则是另一回事。这个 bug 不只出在输出上，它出在决定了哪些内容被提升、以何种授权被提升的那套记忆布局和来源规则上。

![图 12：危险的不只是坏数据进入系统，而是坏数据稍后带着被升级过的授权回来。 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/13.webp)

这正是 Claude Code 变得有意思、超出对产品的赞叹之处。它暗示着，未来某些智能体安全问题，看起来将不那么像经典的 prompt 注入，而更像上下文管理流水线内部的来源失效（provenance failure）。

## 为什么 Claude Code 感觉快，同样是一个系统的故事

值得明说的是，Claude Code 的响应速度不只是一个模型速度的故事。Amdahl 定律在这里很有帮助：**如果模型周围的串行开销占了主导，那么单靠一个更强的模型并不会让系统感觉戏剧性地变快**。Claude Code 似乎正是在攻击那些开销——prompt 前缀复用、schema/工具复用、对琐碎情况的基于规则的处理、有边界的重试、流式输出，以及上下文控制。

所以当 Claude Code 感觉快时，你所感受到的有一部分不是"模型思考得更快"，而是"运行时在做更少的重复工作"。这是一个很不一样的性能故事，而且是一个持久得多的故事。

![图 13：Claude Code 在何处、以何种方式节省时间和 token \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/14.webp)

## 构建者应该从中"偷"走什么

研究 Claude Code 最好的理由，不是对某一个产品的好奇。这次 source map 泄露暴露出了一份对严肃 agentic 系统很有用的设计手册。

**把 prompt 当作基础设施**，而不是文案。如果 prompt 的某些部分是稳定的，就把它们布局成保持稳定、可缓存的样子。

**把上下文当作一个被管理的记忆层级**，而不是一份无限增长的 transcript。让热路径保持小，按需取回较冷的知识，并**像设计记住一样精心地设计遗忘**。

**把 harness 当作产品**。预算、回合、重试、transcript 策略、事件流、压缩，以及子智能体隔离，都不是打磨，它们就是运行时。

**把工具当作能力**。让能力面变得显式，把授权单独建模，并移除那些不该存在的东西——而不是寄望于 prompt 里的一句警告就够了。

**把来源当作安全的一部分**。一旦上下文可以被摘要和重写，无论你是否为此做过规划，你的记忆流水线都已是你安全边界的一部分。

![图 14：Claude Code 的关键要点 \[作者绘图\]](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/using-computer-science-concepts-to-analyze-claude-codes-leaked-source-map/15.webp)

*感谢你阅读这篇博客。欢迎随意留下任何评论！*
