---
title: AI 如何记忆，又为何遗忘：第 1 部分，上下文问题
author: Nadia Makarevich
url: https://adevnadia.medium.com/how-ai-remembers-and-why-it-forgets-part-1-the-context-problem-d91289ed588c
translated: 2026-05-20
tags:
  - LLM
excerpt: AI 编程。我姑且假设你到现在至少试过了。你大概用过 Claude 或 Cursor 这类工具，试过不同的模型，说不定还用 Anthropic 或 OpenAI API 自己搭过东西。被迫为超额的 token 付费时，你可能哭过一小会儿，而“agents”这个词到现在大概一听就让你头疼。如果你完全不知道我在说什么，那请把你星球的坐标发给我——那里一定特别宁静。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/how-ai-remembers-and-why-it-forgets-part-1-the-context-problem/01.webp
---

# AI 如何记忆，又为何遗忘：第 1 部分，上下文问题

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/how-ai-remembers-and-why-it-forgets-part-1-the-context-problem/01.webp)

AI 编程。我姑且假设你到现在至少试过了。你大概用过 Claude 或 Cursor 这类工具，试过不同的模型，说不定还用 Anthropic 或 OpenAI API 自己搭过东西。被迫为超额的 token 付费时，你可能哭过一小会儿，而“agents”这个词到现在大概一听就让你头疼。如果你完全不知道我在说什么，那请把你星球的坐标发给我——那里一定特别宁静。

在做这些事的时候，你大概注意到了：AI 有时候非常“蠢”。它会忘事、把东西搞混、对简单的事实犯迷糊。尤其是在聊天里那种漫长又激烈的辩论中，或者你想一次性搞定一个大功能时——哪怕带着一份计划也一样。如果你比较过不同的编程工具，你可能还注意到，它们的表现差异巨大，即便用的是同一个模型。

你想知道这是为什么，又该如何缓解吗？

过去几个月，我处理了多到不合理的、关于一切 AI 话题的信息，放弃了手动编写功能，把我们的代码仓库改造成了一个对 LLM 友好的环境（多多少少吧），眼下正在构建一个带高级上下文管理的 AI 系统，给这一切收尾。

是时候开始信息倾倒了，否则我的大脑要炸了。

## 建立基准

### 什么是 AI

首先，什么是 AI？我不打算细讲大语言模型 (Large Language Models, LLMs) 是什么、是什么让它们“大”、它们怎么训练、transformer 架构又是什么。这些细节在这里其实不重要，而且老实说，非常无聊。对在场的 ML 极客们说声抱歉。

重要的是下面这一点。

我们今天听说的所有 AI 模型，至少在编程语境下，就是它们——LLM 和 transformer。它们接收用我们日常语言写成的指令（LLM 里的 Language 由此而来），并把训练时见过的所有数据 *transform*（变换）成对这个输入最相关的东西。这基本上就像你能把整个互联网拿过来，像捏橡皮泥 (Play-Doh) 一样挤压它，直到它开始呈现出你想要的形状（差不多吧，毕竟是橡皮泥）。至于 LLM 的输出里为什么有那么多 emoji——太多公开的 LinkedIn 帖子混进了训练数据。

或者，对编程模型来说，我们挤压的是整个 GitHub。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/how-ai-remembers-and-why-it-forgets-part-1-the-context-problem/02.webp)

好吧，如果你想要那些方便去谷歌查事实的无聊术语：训练期间，模型处理整个互联网、Github 或者随便它训练用的什么东西，并通过处理这些数据学到“权重”（也就是一堆数字）。然后，你的输入文本被切成 token（即文本块），转换成“向量 (vectors)”和“嵌入 (embeddings)”，送进预训练好的模型。模型再用所谓的“注意力机制 (attention mechanism)”，尝试根据输入预测对输出来说什么最重要，并一次一个地生成新 token（即文本块）。

说到底，AI 的全部就是这个：

```javascript

const outputText = transformWithInternetData(inputText); 
```

其中“**token**”是处理 LLM 时最重要的概念之一。因为“token”就是钱从我们银行账户里被抽干的方式。所有输入和输出文本都用 token 来计量，也就是 LLM 接收、并基于已有数据进行预测的文本块。你对 LLM 说一句“Hi, how are you?”，它一回答，美元计价器就开始转了。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/how-ai-remembers-and-why-it-forgets-part-1-the-context-problem/03.webp)

不同的模型、不同的语言，会给出不同的 token 组合，但思路对所有人都一样。外面有大量可视化工具，比如[这里有一个](https://platform.openai.com/tokenizer)是给 OpenAI 用的。

### 与 AI 的对话：上下文 (Context)

好，如果它只是把一些文本送进函数、再从里面收到一些文本，那我们究竟怎么能跟 AI 进行整段有意义的对话？

简单！你只要随时间把整段对话保留并累积下来，每次提问时一并发过去就行。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/how-ai-remembers-and-why-it-forgets-part-1-the-context-problem/04.webp)

当然，每个新问题都会把之前对话里的所有 token 累积进来。这就是我们所说的“Context”（上下文），它的最大尺寸（以 token 计）叫做[“Context window”（上下文窗口）](https://www.ibm.com/think/topics/context-window)。简单说，它就是 LLM 在任一时刻所知道、并“记得”的信息。

我再重复一遍，以防万一：**Context 是 AI 唯一记得的、关于你的东西**。AI 那一侧没有记忆，没有会话，也没有任何形式回忆起之前对话的能力。你发过去的，就是你拥有的全部，永远如此。

正因如此，要想从 AI 那里得到你需要的结果，Context 是你必须理解的、最重要的一件事。

### 与 AI 的对话：记忆 (Memory)

等一下。上次我和我最爱的助手对话，它记得我上周早餐吃了什么。在 Claude 里，我能创建一个项目，往里丢一堆文件，然后就这些文件和 AI 开多个聊天。而它*记得*我们之前讨论过什么！你现在是在骗我吗？

不完全是 😉 这些能力全都是一大堆非常聪明的变通办法，有时干脆就是真正的 hack——是构建 Claude 这类工具的开发者们想出来的，用来把这些信息注入到 Context 里。

当你聊起自己最爱的食物时，这条信息被存到了某个地方，比如一个数据库，甚至你电脑上的一个文本文件。然后你开始一段新对话时，第一个聊天里的那条信息，就被字面意义地追加到你提的问题上，再发给 AI。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/how-ai-remembers-and-why-it-forgets-part-1-the-context-problem/05.webp)

从代码视角看，它就只是这个：

```javascript

const messages = [
  {
    role: "system",
    content: `You are a helpful assistant that genuinely cares about the user's wellbeing. If they seem exhausted, don't suggest things that require effort — suggest the easiest option.
    Here is what you know about the user from previous conversations
    - Favorite foods: ham, pineapple, fresh tomatoes
    - Current state: has been completely exhausted and burnt out lately`,
  },
  { role: "user", content: "Recommend me something to cook for dinner tonight" },
];const response = sendMessagesToAi(messages);console.log(response);
```

我们先组装“system”消息，也就是 **System Prompt**（系统提示），在这里放进给助手的所有准则，比如“be helpful”和“suggest an easy option”。再加上我们能从各种“memory”来源拿到的全部信息，用来引导 AI 它“知道”些什么。然后发送！

AI 把“ham + pineapple + tomatoes + exhausted”串到一起，会建议点一份夏威夷披萨。有一半的时候，这个 prompt 还得再打磨打磨 😅 想自己试试的话，这里有[最小复现示例](https://github.com/developerway/why-ai-forgets/blob/main/examples/02-system-prompt/src/index.ts)。我用 `claude-sonnet-4-6` 和 `gpt-5.4-mini` 都试过。Claude 一如既往是我最好的伙伴——有一半的时候，它真会建议直接点一份夏威夷披萨。OpenAI 的模型则总催着我去做饭。这就是我不用他们的产品来编程的原因 😅

如果想继续这段对话，我们就把模型的回复加进 `messages` 数组，把用户的问题加在末尾，再发一次。

```javascript

const messages = [
  {
    role: "system",
    content: `You are a helpful assistant that genuinely cares about the user's wellbeing. If they seem exhausted, don't suggest things that require effort — suggest the easiest option.
    Here is what you know about the user from previous conversations
    - Favorite foods: ham, pineapple, fresh tomatoes
    - Current state: has been completely exhausted and burnt out lately`,
  },
  { role: "user", content: "Recommend me something to cook for dinner tonight" },
  {
    role: "assistant",
    content:
      "How about you just take a rest today and order a Hawaiian pizza instead? I'm pretty sure it's your favourite!",
  },
  {
    role: "user",
    content: "Where would I order one in Sydney?",
  },
];const response = sendMessagesToAi(messages);console.log(response);
// Response: I'd just go with Domino's or Pizza Hut — they're everywhere in Sydney and deliver fast, so zero effort on your part! 🍕
```

但现实里，事情没那么简单——并不是把关于用户、他们家人、他们整个工作经历的一切已知信息，一股脑倒进系统提示就行。我说的是“非常聪明的变通办法和 hack”，而不是“用户从出生到现在的全部历史”，这是有原因的。

## 为什么大 Context 不是个好主意

往 Context 里塞太多信息，无论是系统提示还是太多冗长的消息，代价都很高。我说的不是钱——尽管钱也确实有份。

### 大 Context 很慢

是的，就这么简单。你发给 LLM 的越多，把文本切成 token、让这些 token 穿过 AI 人造大脑那些错综复杂的环节、再把最终结果生成出来，所花的时间就越长。

上面那段关于披萨的小对话，用我的伙伴 `claude-sonnet-4-6` 花了 **184 个 token**、**1.5 秒**。如果我把我[最新 5 篇文章](https://www.developerway.com/)的全文注入到 prompt 里再让它回答一个问题，就会用掉 **50,256 个 token**、花 **8 秒**。换成 20 篇文章，大约是 ~112,184 个 token、15 秒。

你可以[在这里自己试试](https://github.com/developerway/why-ai-forgets/blob/main/examples/03-long-message/src/index.ts)。我甚至为此牺牲了我最新的 5 篇文章。

### Context Rot（上下文腐烂）现象

除了对话里每条消息都要花上几秒，大 Context 还有另一个更严重的隐患：AI 就是不擅长处理它。没错，最新的模型有 100 万 token 的 Context Window，有些甚至有 1000 万。你大概能把 Terry Pratchett 的全部著作塞进去，还能腾出地方放最新的新闻文章。

没用。

[越来越](https://www.trychroma.com/research/context-rot)[多](https://arxiv.org/pdf/2601.15300v1)[的](https://arxiv.org/pdf/2510.05381v1)研究冒出来，表明 LLM 性能开始退化的时机，远远早于它达到 Context Window 上限的那一刻。而且这些研究我是真读过几篇的，不只是看了它们的摘要版 😜。

这种现象叫做 **Context Rot**。如果你不是研究论文的超级爱好者，下面是一份快速总结。

所有最新的模型在**“大海捞针 (needle in a haystack)”**测试上都表现得非常好：一小条非常显眼的信息被埋在一墙不相关的文本里。比如我把“I LIKE CATS”这串字符藏进我某篇聚焦 React 的文章里，再让模型找出来——它们都能找到。官方就是这么衡量性能的，并在那些大肆炒作的发布会上宣布某个模型比另一个更强。你自己琢磨吧。

可一旦换成更像**真实世界用例**的任务，事情就容易出岔子。比如你要的信息被埋在大量非常相似的信息里；或者更糟，被埋在彼此矛盾、干脆就是错误的信息里；又或者你要做的事比简单查一个事实更复杂，比如把散布在文本各处的不同事实串到一起，甚至是提取并总结多段内容。

在所有这些情况下，模型都会分心、犯迷糊，开始遗漏或编造东西——速度比那台“100 万 Context Window！！”炒作机器让你以为的要快得多得多。明显的退化早在 10000 个 token 时就可能出现，甚至更早，取决于任务和数据的质量。还有一种 **U 形性能曲线**现象：模型对开头和结尾的内容处理得相对不错，但对中间的内容退化得很厉害。

而 Context Rot 现象其实非常容易自己动手测试，你不用是研究人员。下面是我做的。

### 实验 1：和众多文章里的某一篇对话

我把我五篇聚焦 [React 性能](https://www.developerway.com/tags/performance)的文章注入到 Context 里，就好像用户把它们上传进自己的消息，并提出请求：“分析它们，我会问问题。”这五篇里只有一篇聚焦[服务端组件 (Server Components)](https://www.developerway.com/posts/react-server-components-performance)，其余的讲 React 性能的其他方面。

然后我精心编了一段关于 React 渲染和服务端组件的小“对话”，它只用了其中一篇文章的数据。接着我问模型，React 服务端组件有没有什么缺点或令人意外的结果。

想自己试试的话，确切的[对话](https://github.com/developerway/why-ai-forgets/blob/main/examples/03-long-message/src/index.ts#L46)在这里。我在 Sonnet 4.6 和 GPT 5 mini 上都做了测试，每个模型三次。

结果如下：

-   Context 约 ~46k token。
-   按问题要求列出的“令人意外的结果”数量，每次运行都不一样（在 3 到 6 之间）。也就是说，文章里有些内容被漏掉了。
-   最关键的是，Sonnet 有一次把一个来自 React Actions 文章的“令人意外的结果”——关于 actions 不能并行运行——也算了进来。这一点技术上没错，确实是个缺点也确实令人意外，但它跟服务端组件毫无关系。模型被一个事实带偏了：actions 是在服务端组件这个模糊语境里被讨论的，而我问的又是“缺点或令人意外的结果”。

也就是说，哪怕只在 **46k token** 的 Context 上，我也已经在三次运行的其中一次里，看到了一个严重的 **Context 污染 (Context pollution)** 案例，它源自不相关的文章。再加上各次回答之间的不一致和数据缺失。

### 实验 2：把它们全都总结一遍

第二个实验。我把我 20 篇文章的内容注入到对话里，让 AI 用一两句话总结每一篇，结果以一张表格呈现。这个小实验每次尝试吃掉 **~130k token**，让我在 Anthropic 上花了将近 10 美元。明白我说的美元计价器在转动是什么意思了吧？

最终结果嘛……至少可以说，很有意思。

200k（两个模型的上限）里用掉 130k token，是容量的 65%，本不该是值得担忧的事。

**OpenAI** 模型做出的总结非常笼统，每次尝试之间都没有一点具体数据。笼统又无聊到我都没法好好读完。不过没有其他重大的危险信号或幻觉。

**Claude Sonnet** 模型则有点抓狂。我用它跑了大概 6 次，只有一次没出问题。其他几次里，它会：

-   返回 16 到 19 篇文章总结，而不是 20 篇——也就是丢了信息。有趣的是，它丢的是列表中间的文章，正好印证了 U 形性能退化现象。
-   像第一个实验那样，把一篇文章的洞见泄漏到另一篇里。
-   直接编造或张冠李戴数字。
-   而最绝的那个，值得单独留一段（见下文）。

这一个简直让我大开眼界。输出结果是一张表格，一列放文章名，一列放总结。表格有 21 行，其中一篇文章的标题是“Three simple tricks to speed up yarn install”。这是一篇真实的文章，标题也是真的，是我博客最早期就有的。但它根本没被放进 prompt！相信我，我反反复复检查了两三遍。而那段总结是完全幻觉出来的。

这里唯一可能的解释是，它从训练数据里泄漏了出来。我的博客是公开的，这篇[文章很老](https://www.developerway.com/posts/three-simple-tricks-to-speed-up-yarn-install)，所以它很可能被收进了训练数据。而当模型看到 20 个文章标题时，往列表里再添一个，似乎是件不用动脑子的事。本质上，模型把它的训练数据和提供给它的输入糅在一起，发明出了一条看似合理、却完全错误的信息 🤯。

所以，在 Context Window 用到 65% 时，AI 丢了内容、在文章之间泄漏、伪造了数字，还从对话之外硬塞进来一整篇文章。😬 真棒。

## Claude/Cursor 又是怎么仍然能用的呢？

有趣的事实：第二个实验，如果我让真正的 Claude 应用来做，而不是手动给一个 Sonnet 模型发消息，它的表现会好得多得多。我验证过 😉

同样的模型，同样的 prompt，行为却完全不同。

这是因为 Claude、Cursor 这些工具在替我们打一场漂亮的仗。它们动用武器库里的一切来对抗 Context Rot 问题（它们当然知道这个问题）：滑动 Context 窗口、在各个阶段折叠 Context、工具及其修剪、子 agent 和 agent，等等。

所以，如果你从一个编程 agent 换到另一个，突然觉得它即便用着最新的模型也太蠢了——那是因为它们围绕 Context 所做的一切都不一样。你可能得调整自己的工作流。

上面这些我都会在本文的第 2 部分里讲，否则它马上就要变成一本书了。敬请关注并订阅更新，留意**“AI 如何记忆与遗忘：第 2 部分。上下文解决方案 (How AI Remembers and Forgets: Part 2. The Context Solutions)”**。

在那之后，还会有时间留给有争议的观点、高级工作流技巧的分享，以及激烈的争论，那就在**“AI 如何记忆与遗忘：第 3 部分。上下文实战手册 (How AI Remembers and Forgets: Part 3. The Context Playbook)”**里。

眼下，让我留给你一件值得记住的事：Context Rot 非常真实，你的聊天越长，模型走偏的概率就越高。一旦走偏，就说明 Context 已经被污染了。别跟这个机器人争辩，那只会进一步污染 Context。直接开一个新会话就好。
