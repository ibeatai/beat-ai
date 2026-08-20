---
excerpt: 七个框架试完，答案不在框架里。
---

# 从 Google ADK 到 OpenFang，我试了七个 Agent 框架，最后被一个教程点醒了

> 七个框架试完，答案不在框架里。

**一个开发者花一年多试了七个 Agent 框架，最后被一个 84 行代码的教程点醒：你纠结的"选框架"，本质是在纠结"选哪个壳"。**

## 七个坑

Google ADK，骨子里为 Gemini 服务，换模型处处别扭，想脱离 Google Cloud 就是在和框架较劲。阿里 AgentScope，理念认同但社区太小，遇到问题搜不到答案，需要读完源码才能用——那和没有框架有什么区别？

LangGraph 花时间最多，生态最大，checkpoint 做得好。但大量时间花在学框架本身而不是写 Agent。有一次排查问题，最后发现是框架内部状态序列化的 bug，跟业务逻辑一点关系都没有。这个抽象层到底是在帮你还是在拦你？

Claude Agent SDK 设计克制，给工具不给框架。但"克制"意味着状态管理、错误恢复、多 Agent 编排全得自己搭。OpenClaw 三天 6 万 star，很酷，但它解决的是"个人助手"场景，不是业务流程里稳定跑任务的 Agent。OpenHands 做编码 Agent 很能打，但拿来做非编码场景改动太大。OpenFang 技术硬核但 3 月刚开源，生产验证约等于零。

## 84 行代码的真相

直到看到 Learn Claude Code 项目——12 节课从零搭 Agent，第一课的核心实现就是一个 while 循环：用户发消息，模型判断要不要调工具，调了就执行，结果喂回去，继续循环。84 行，一个完整能跑的 Agent。

那七个框架，底层都是这同一个循环。LangGraph 包了图引擎，CrewAI 包了角色扮演，Google ADK 包了 Vertex AI 生态。它们的差异本质上是在争论"循环外面应该包什么壳"。但循环本身根本不是问题，Anthropic 自己都说了："很多模式用几行代码就能实现。"

## 框架是壳，工具和上下文是命

真正卡脖子的是两件事。第一，给 Agent 的工具好不好用——接口设计、错误信息可读性、返回结果结构化程度，直接决定 Agent 能不能理解下一步该干什么。Princeton 研究发现，同一个模型换一套工具接口，成功率差异巨大，改善工具设计比改善架构本身提升还大。

第二，跑到第 100 步时它还清醒吗。上下文窗口溢出，关键信息被挤掉，Agent 忘了自己在干嘛——这才是生产环境真正会杀死你的问题。2026 年 Agent 工程的核心挑战不是 prompt engineering，是 context engineering。

社区管这叫"框架幻灭曲线"：用框架快速原型，撞上框架的墙，用裸 API 重写，最后自己造一套薄薄的工具层。别在选框架上花太久，选一个能读懂源码的，赶紧把精力放到工具接口和上下文管理上。那才是 Agent 能不能落地的分水岭。

---

## 原文链接

> **从 Google ADK 到 OpenFang，我试了七个 Agent 框架，最后被一个教程点醒了**
>
> 来源：AutomatorRunner
>
> 七个 Agent 框架实测对比，揭示框架选择背后的真正问题
>
> 👉 <a href="https://mp.weixin.qq.com/s/B8xiTGVIZ63Q27y_gFc4Cg" target="_blank" rel="noopener noreferrer">点击阅读原文</a>

---

**#AIAgent #框架选择 #AgentEngineering #ContextEngineering #LLM**
