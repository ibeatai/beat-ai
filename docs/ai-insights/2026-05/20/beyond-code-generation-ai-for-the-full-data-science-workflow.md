---
title: "超越代码生成：让 AI 覆盖完整的数据科学工作流"
author: Yu Dong
url: https://medium.com/data-science-collective/beyond-code-generation-ai-for-the-full-data-science-workflow-ef875dce8453
translated: 2026-05-20
tags:
  - Artificial Intelligence
excerpt: 最近我一直被一种挥之不去的 AI FOMO 笼罩。每天都能看到有人分享 AI 技巧、晒出自己造的新智能体和技能，还有各种凭感觉写出来的应用。我越来越意识到，对今天的数据科学家来说，快速适应 AI 已经成了保持竞争力的硬性要求。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/01.webp
---

# 超越代码生成：让 AI 覆盖完整的数据科学工作流

## 用 Codex 和 MCP 在一个真实工作流中打通 Google Drive、GitHub、BigQuery 与分析

最近我一直被一种挥之不去的 AI FOMO 笼罩。每天都能看到有人分享 AI 技巧、晒出自己造的新智能体和技能，还有各种凭感觉写出来的应用。我越来越意识到，对今天的数据科学家来说，快速适应 AI 已经成了保持竞争力的硬性要求。

但我说的不只是和 ChatGPT 头脑风暴、用 Cursor 生成代码，或者用 Claude 润色报告。更大的转变在于：**AI 现在能参与一个端到端得多的数据科学工作流**。

为了把这个想法落到实处，我用自己的 Apple Health 数据在一个真实项目上试了一把。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/01.webp)
*作者用 ChatGPT 生成的图片*

## 一个简单的例子——Apple Health 分析

### 背景

自 2019 年起，我每天都佩戴 Apple Watch 来记录健康数据，比如心率、消耗的能量、睡眠质量等。这些数据攒下了多年来关于我日常生活的行为信号，但 Apple Health 应用基本只用简单的趋势视图把它呈现出来。

六年前我曾试着分析过一份两年期的 Apple Health 导出数据，结果它变成了那种永远做不完的副项目……这次我的目标是借助 AI，快速从原始数据里挖出更多洞见。

### 我手头有什么

我能用上的相关资源有：

1.  **原始的 Apple Health 导出数据**：1.85GB 的 XML，已上传到我的 Google Drive。
2.  六年前放在我 [GitHub 仓库](https://github.com/yudong-94/Apple-Watch-Data-Exploration)里的**示例代码，用于把原始导出数据解析成结构化数据集**，不过这些代码可能已经过时了。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/02.webp)
*作者拍摄的原始 XML 数据截图*

### 没有 AI 的工作流

不用 AI 的标准工作流，看起来会和我六年前做的差不多：检查 XML 结构，写 Python 把它解析成结构化的本地数据集，用 Pandas 和 Numpy 做 EDA，最后总结洞见。

我相信每位数据科学家都熟悉这个过程——它算不上什么高深的东西，但搭起来要花时间。要做出一份打磨过的洞见报告，**至少得花上一整天**。这也正是那个六年前的仓库至今还标着 WIP 的原因……

### AI 端到端工作流

我用 AI 之后的新工作流是这样的：

1.  AI 在我的 Google Drive 里定位原始数据并下载下来。
2.  AI 参考我旧的 GitHub 代码，写一个 Python 脚本来解析原始数据。
3.  AI 把解析后的数据集上传到 Google BigQuery。当然，分析也完全可以在本地做、不用 BigQuery，但我特意这样设置，是为了更贴近真实的工作环境。
4.  AI 针对 BigQuery 运行 SQL 查询来做分析，并编出一份分析报告。

本质上，从数据工程到分析，几乎每一步都由 AI 处理，我更多地扮演审阅者和决策者的角色。

### AI 生成的报告

下面就来看看，在我的指导和几轮来回沟通下，Codex **在 30 分钟内**能生成出什么——这还不算搭建环境和工具链的时间。

我之所以选 Codex，是因为工作中我主要用 Claude Code，所以想换个工具探索一下。我趁这个机会从零搭建自己的 Codex 环境，好更全面地评估它需要的全部投入。

可以看到，这份报告结构清晰、视觉上也很精致。它把年度趋势、锻炼一致性以及旅行对活动量的影响等有价值的洞见都总结了出来，还给出了建议，并写明了局限性和假设。最让我印象深刻的不只是速度，而是它的输出多快就开始像一份面向利益相关者的分析，而不是一个粗糙的 notebook。

请注意，这份报告出于我的数据隐私考虑做了脱敏处理。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/03.webp)

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/04.webp)

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/05.webp)
*Codex 生成的报告（数字出于数据隐私考虑做了调整，作者截图）*

## 我实际是怎么做的

既然已经看到了 AI 在 30 分钟内能交出的成果，下面就让我把它拆开，把我为实现这一切所走的每一步都展示出来。这次实验我用的是 [Codex](https://chatgpt.com/codex)。和 Claude Code 一样，它可以在桌面应用、IDE 或 CLI 中运行。

### 1\. 设置 MCP

要让 Codex 能访问 Google Drive、GitHub、Google BigQuery 等工具，下一步就是配置 Model Context Protocol (MCP) 服务器。

设置 MCP 最简单的办法，就是让 Codex 替你做。比如我让它配置 Google Drive MCP 时，它很快就配好了本地文件，还给出了清晰的后续步骤，告诉我怎么在 Google Cloud Console 里创建一个 OAuth 客户端。

它并不总是一次就成功，但坚持下去是有用的 :) 我让它配置 BigQuery MCP 时，它在连接成功前至少失败了 10 次。不过每一次，它都给了我清晰的说明，告诉我怎么测试、哪些信息有助于排查问题。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/06.webp)

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/07.webp)
*作者拍摄的 Codex MCP 设置截图*

### 2\. 用 Plan Mode 制定计划

设置好 MCP 之后，我转向了实际的项目。对于一个涉及多个数据源、工具和问题的复杂项目，我通常会先用 Plan Mode 来敲定实现步骤。在 Claude Code 和 Codex 里，都可以用 `/plan` 启用 Plan Mode。它的工作方式是这样的：你勾勒出任务和大致的计划，模型则提出澄清性的问题，再给你一份更详细的实现计划供你审阅和细化。在下面的截图里，可以看到我用它做的第一轮迭代。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/08.webp)

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/09.webp)

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/10.webp)
*作者拍摄的 Plan Mode 截图*

### 3\. 执行与迭代

在我点下「Yes, implement this plan」之后，Codex 就按步骤自行执行起来。它干了 13 分钟，生成了下面这第一份分析。它在不同工具之间切换得很快，但因为在 BigQuery MCP 上又遇到了更多问题，它把分析放在了本地做。再经过一轮排查，它终于能正常上传数据集并在 BigQuery 中运行查询了。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/11.webp)
*作者拍摄的第一份分析输出截图*

不过，第一遍的输出还是偏浅，于是我用追问引导它深入下去。比如，我的 Google Drive 里存着过去旅行的机票和行程计划，我让它找出这些文件，分析我在旅途中的活动模式。它成功定位了那些文件，提取出我的旅行天数，并完成了分析。

经过几轮迭代之后，它在 30 分钟内生成了一份全面得多的报告，就是我开头分享的那份。你可以在[这里](https://github.com/yudong-94/apple-health-analysis-with-codex)找到它的代码。这大概是这次练习中最重要的一课：**AI 跑得很快，但深度仍然来自迭代和更好的问题。**

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/20/images/beyond-code-generation-ai-for-the-full-data-science-workflow/12.webp)
*Codex 定位我过去的旅行日期（作者截图）*

## 给数据科学家的几点要点

### AI 改变了什么

以上是一个小例子，展示了我如何用 Codex 和 MCP **在不手动写一行代码的情况下跑完一次端到端的分析**。那么对工作中的数据科学家来说，要点是什么？

1.  **思考要超越编码辅助。** 与其只把 AI 用在编码和写作上，不如把它的角色拓展到整个数据科学生命周期。在这个例子里，我用 AI 在 Google Drive 中定位原始数据，又把解析后的数据集上传到 BigQuery。围绕数据流水线和模型部署，还有更多 AI 用例可以挖掘。
2.  **上下文成为力量倍增器。** 正是 MCP 让这个工作流强大了许多。Codex 扫描我的 Google Drive 来定位旅行日期，读取我旧的 GitHub 代码来找示例解析代码。同样地，你也可以启用其他**经公司批准的** MCP，帮你的 AI（和你自己）更好地理解上下文。例如：
    \- 接入 Slack MCP 和 Gmail MCP，搜索过去相关的对话。
    \- 用 Atlassian MCP，访问 Confluence 上的表文档。
    \- 配置 Snowflake MCP，探索数据 schema 并运行查询。
3.  **规则和可复用的技能很重要**。虽然这个例子里我没有明确演示，但你应该定制规则、创建技能，来引导你的 AI 并扩展它的能力。这些话题值得下次单独写一篇文章 :)

### **数据科学家的角色将如何演变**

但这是否意味着 AI 会取代数据科学家？这个例子也让我们看清了数据科学家的角色未来会如何转向。

1.  **更少手动执行，更多问题解决**。在上面的例子里，Codex 生成的初始分析非常基础。AI 生成的分析质量，很大程度上取决于你界定问题的质量。你需要把问题定义清楚，拆解成可执行的任务，找出对路的方法，并把分析推向更深处。
2.  **领域知识至关重要**。要正确解读结果、给出建议，领域知识仍然非常必要。比如 AI 注意到我的活动量从 2020 年起明显下降，它找不到有说服力的解释，只能说：「*可能的原因包括日常变化、工作安排、生活方式转变、受伤、动力不足，或训练不那么规律，但这些都是推断，不是发现*。」可背后真正的原因，你大概已经猜到了——是疫情。我从 2020 年初开始在家办公，自然就消耗更少卡路里了。这是一个很简单的例子，说明为什么领域知识依然重要：即便 AI 能访问你公司里过去所有的文档，也不代表它能理解所有业务上的微妙之处，而这正是你的竞争优势。
3.  这个例子相对来说还算直接，但仍有不少类别的工作，我今天不会放心让 AI 独立去做，尤其是**那些需要更强技术和统计判断的项目**，比如因果推断。

### 重要的注意事项

最后但同样重要的是，使用 AI 时有几点考量你必须记牢：

1.  **数据安全**。我相信这一点你已经听过很多遍了，但请允许我再说一次：使用 AI 的数据安全风险是真实存在的。对于个人副项目，我可以随心所欲地设置、自担风险（说实话，给 AI 完全访问 Google Drive 的权限感觉是步险棋，所以这更多是出于演示目的）。但在工作中，一定要遵循公司关于哪些工具可以安全使用、该怎么用的指引，并且在点「approve」之前务必把每一条命令都通读一遍。
2.  **复核代码**。在我这个简单的项目里，AI 写出准确的 SQL 毫无问题。但在更复杂的业务场景中，我还是会时不时看到 AI 在代码里犯错。有时它会连接粒度不同的表，导致扇出和重复计数；有时它又会漏掉关键的过滤条件。
3.  **AI 很方便，但它在完成你的请求时可能带来意想不到的副作用**……让我用一个有趣的故事来收尾。今天早上我打开笔记本电脑，看到一条磁盘空间已用尽的警报——我用的是一台 512GB SSD 的 MacBook Pro，而我相当确定自己只占用了大约一半的存储空间。由于昨晚一直在玩 Codex，它成了我的头号嫌疑人。于是我真的去问了它：「*嘿你做了什么吗？我的「系统数据」一夜之间涨了 150GB*」。它回答：「*没有，Codex 只占用 xx MB*」。然后我翻了翻文件，看到一个 142GB 的「bigquery-mcp-wrapper.log」……很可能是 Codex 在排查 BigQuery MCP 设置问题时建立了这个日志文件，后来在实际的分析任务中，它膨胀成了一个庞然大物。所以是的，这台神奇的许愿机器是要付出代价的。

这段经历很好地为我总结了其中的取舍：AI 能极大地压缩从原始数据到有用分析之间的距离，但要把它的价值榨到最大，仍然需要判断力、监督，以及一份愿意去调试工作流本身的耐心。

这篇文章最初发表于 [Towards Data Science](https://towardsdatascience.com/beyond-code-generation-ai-for-the-full-data-science-workflow/)。

如果你喜欢这篇文章，欢迎关注我，也欢迎看看我关于数据科学、分析和 AI 的其他文章。
