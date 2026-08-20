---
title: "你的 Claude 配置会随着时间变质：用这条自检提示词，60 秒排毒"
excerpt: 我上周把自己的 Claude 配置砍掉了一半，结果每一次输出都变得更好了。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-03/25/images/claude-self-audit-prompt/cover.jpg
---

![Claude setup 自检提示词主图](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-03/25/images/claude-self-audit-prompt/cover.jpg)

我上周把自己的 Claude 配置砍掉了一半，结果每一次输出都变得更好了。

听起来很反直觉，但 Anthropic 自己最新的 prompting 文档，刚好解释了这件事为什么会发生。

下面这条 prompt 能直接告诉该删掉什么，而且连复制粘贴配置都不用做，Claude 会自己审计这些文件：

附言：如果想每周都在收件箱里收到更多像这样的 AI workflow，可以订阅这份免费邮件，已经有 `3.5 万` 名读者：
[aisolo.beehiiv.com/subscribe](https://aisolo.beehiiv.com/subscribe)

## 每个人最后都会走到这一步

先是某次输出很糟，于是往 `CLAUDE.md` 里加一条规则，也就是那份自定义指令文件。比如：`Be more concise.`

下一周，又来一次糟糕输出。于是再加一条：`Use a casual tone.`

再过一个月，又有别的地方出问题。于是继续加：`Always explain technical terms.`

每一条规则在加进去的当下都很合理。修的都是刚刚真实遇到的问题，所以会觉得这件事很有效率。

但几乎没有人会回头把旧规则删掉。

三个月之后，配置里已经堆了 `89` 条规则。

模型却得在每一次输出里同时满足它们全部，而且不管这些规则这次是否相关。

这就像把一份 `47` 步的菜谱交给厨师，而这道菜其实只需要 `12` 步。

多出来的 `35` 步只会增加混乱。

厨师会开始反复怀疑那些自己本来就会的环节，把心智资源浪费在互相矛盾的步骤协调上，最后做出来的菜，反而不如让他直接去做。

这就是 over-prompting。几乎每个人的配置里都有这个问题。

Anthropic 自己的工程团队最近也在内部 Claude 配置里发现了同样的问题。

他们那套 scaffolding，实际上正在让 AI 变差。

如果连这群业内最聪明的人都会遇到这个问题，那自己的自定义指令大概率也一样。

## 这条 Self-Audit Prompt，直接复制即可

与其一行一行手动翻自己的整套配置文件，反正现实里也没人会这么做，不如直接让 Claude 自己审计自己。

如果平时用的是 Claude 的桌面应用 Cowork，那么 Claude 本来就已经能访问整套配置。

`CLAUDE.md`、skills 文件夹，也就是那些可复用指令文件、context 文件，基本都能直接读，不需要手动粘贴任何东西。

直接打开一个会话，把下面这段原样复制进去：

```text
Read my entire setup before responding. Check my CLAUDE.md, every skill in my skills folder, every file in my context folder, and any other instruction files you can find.
Then go through every rule, instruction, and preference you found. For each one, tell me:
1. Is this something you already do by default without being told?
2. Does this contradict or conflict with another rule somewhere else in my setup?
3. Does this repeat something that's already covered by a different rule or file?
4. Does this read like it was added to fix one specific bad output rather than improve outputs overall?
5. Is this so vague that you'd interpret it differently every time? (ex: "be more natural" or "use a good tone")
Then give me:
- A list of everything you'd cut, with a one-line reason for each
- A list of any conflicts you found between files
- A cleaned-up version of my CLAUDE.md with the dead weight removed
```

就这一条消息。Claude 会把整套配置读完，用这 `5` 个过滤条件逐条检查，然后回来告诉该删什么，以及为什么删。

## 把删减后的配置怎么测

拿到审计结果以后，不要闭着眼把它标出来的东西全删光。流程应该是这样的：

1. 先读 Claude 标记了什么、为什么标记。如果不同意某一条具体删法，就把那条留着。毕竟自己的工作流，还是比 Claude 更清楚。
2. 其他的全部删掉，而且要一次删，不要一条一条试。只有这样才真的能感受到差异。
3. 用删减后的配置跑最常用的 `3` 类任务，也就是每天都会让 Claude 做的那些事情，而不是边缘 case。
4. 如果输出没变，或者更好了，那些被删掉的规则就确认只是负担，继续让它们消失。
5. 如果只有某一件具体事情变差了，就只把那一条规则加回来。

目标是找出自己的 minimum viable setup，也就是那套最小但稳定的配置：用尽可能少的指令，持续拿到想要的输出质量。

## 让它自动跑起来，让 Claude hygiene 一直保持干净

这类审计只有真的去做才有效。而每隔几周手动粘一次这条 prompt，这件事大概率是记不住的。至少我自己记不住。

所以与其指望意志力，不如把它变成一个自动定时任务。

Cowork 内建了调度功能，可以把任何 prompt 设成周期性自动运行。

设置方法如下，整个过程大概只要 `45` 秒：

1. 打开 Cowork。
2. 对它说：

```text
Create a weekly scheduled task called "setup-audit" that runs every Monday at 9am.
The task should: read my entire setup (CLAUDE.md, all skills, all context files, everything), then audit every rule against these 5 filters:
1. Is this something you already do by default?
2. Does it contradict another rule somewhere else?
3. Does it repeat something already covered?
4. Does it look like a bandaid for one bad output?
5. Is it so vague you'd interpret it differently every time?
Then give me a list of what to cut with reasons, any conflicts between files, and a summary of how many rules passed vs got flagged. Don't change any files, just report.
```

Cowork 会创建这个任务，并把它放进侧边栏的 Scheduled 区域。

这样每周一早上，都会自动收到一份新的审计报告，不需要额外做任何事。

读完它，删掉该删的，然后继续过完这一周。

就这样。只做一次设置，后面的 prompt hygiene 就会自动运行。

第一次运行时，它会要求批准工具访问权限。之后它就会在后台静默执行。

自己的 AI 配置，本来就应该随着时间推移变得更简单。

每次模型更新、每增加一个 skill，都很可能让旧指令里的某些内容变得没有必要。

真正拿到最好输出的人，会定期把这些 dead weight 删掉。现在，自己也可以变成这类人。

做减法，反而会有增益。

附言：我很快会办一场深入的 Claude Cowork workshop，专门讲怎么把它用到相当于一支年薪 `50 万美元` 市场团队的输出水平。

上一次有 `180` 人参加。

现在可以先在这里预登记，占个位置，不需要付款：
[https://tally.so/r/LZbxKl](https://tally.so/r/LZbxKl)

---

## 原文链接

> **Your Claude setup rots over time. Detox it in 60 seconds (with this self-audit prompt):**
>
> 来源：X | 作者：Ole Lehmann
>
> 这篇内容的核心不是“多写一点指令”，而是相反：定期删掉已经过时、冲突、重复、模糊的那部分配置，Claude 的输出反而会更稳定。
>
> 👉 <a href="https://x.com/itsolelehmann/status/2036533756572639611" target="_blank" rel="noopener noreferrer">点击阅读原文</a>
