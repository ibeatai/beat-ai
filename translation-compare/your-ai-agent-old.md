---
title: 你的 AI agent 能读你的代码库，却不了解你的产品
author: Gregory Muryn-Mukha
url: https://uxdesign.cc/your-ai-agent-can-read-your-codebase-it-doesnt-know-your-product-b5ea0cd77989
translated: 2026-05-23
tags:
  - Artificial Intelligence
---

# 你的 AI agent 能读你的代码库，却不了解你的产品

## **如何把代码里没有的品牌、模式和视觉语言喂给 AI 编码 agent。**

![](/docs/ai-insights/2026-05/23/images/your-ai-agent-can-read-your-codebase-it-doesnt-know-your-product/01.webp)
*一本摊开的代码书旁边放着一沓扇形展开的设计样本，这是一个产品的两类制品，而 agent 只能读懂其中一种。由 OpenAI 生成。*

> ***TL;DR.*** *AI 编码 agent 可以 grep 你的代码库。但它们仍然会产出通用 SaaS 味的东西，因为它们缺少你产品的* 设计上下文 *：产品如何运作、它原则上拒绝哪些交互模式、是什么让它感觉像你的产品。我把这些写成了一个结构化的 Claude Code skill。第一次输出就不再那么泛泛。下面是这个系统、每个部分为什么存在，以及构建它难在哪里。*

我让我的 AI agent 为我们的产品头脑风暴新功能。不到一分钟，它返回了一份干净的清单：通知中心、活动流、带分析小组件的仪表盘、入门引导向导。都是合理的功能。却没有一个适合我们。

我们的核心交互是对话，而不是向导加仪表盘的流程。通知中心默认有一堆被动消息需要分拣。入门引导向导默认存在一个线性的首次使用流程，而我们刻意没有做这个。每个建议都悄悄打破了我们已经决定的原则。

agent 可以完整访问代码库。问题不在这里。

问题是另一种形态的知识：从设计角度看，*产品是什么*。它如何运作。我们原则上拒绝哪些交互模式。它怎么说话。"符合品牌" 对一个圆角或一个错误状态来说到底意味着什么。准确地说，是什么让一个界面感觉像**我们**，而不是像 YC 批次里冒出来的任何一个 B2B SaaS。

这些都不在代码里。它们存在于品味里，存在于 Figma 文件里，存在于上周二我们在 Slack 里如何决定包装某个功能，存在于半打人的脑子里。

![](/docs/ai-insights/2026-05/23/images/your-ai-agent-can-read-your-codebase-it-doesnt-know-your-product/02.webp)
*四张样本卡散落在桌面上。四种模式都被审视过；没有一种适合。由 OpenAI 生成。*

这就是 agent 缺失的那类知识。可以称之为产品的**设计上下文**。

问题不在于 agent 能读什么，而在于代码没有说什么。

## **对 AI 来说，一个产品到底是什么**

思考如何把这些喂给 agent 时，我最后梳理出了七种不同形态的知识。每一种都是不同类型的东西。没有一种适合塞进 README。

1.  **架构。** 服务、数据存储、鉴权、数据流、硬约束。
2.  **功能。** 端到端流程、领域实体模型、产品界面。
3.  **技术栈约定。** yarn/pnpm、monorepo 布局、完成前检查、导入规则。
4.  **品牌声音。** 语气维度（正式度、创造性、坚定程度、emoji）、CTA 词汇、文案模式、反模式。产品实际如何对用户说话。
5.  **视觉识别。** 字体、字重、圆角族、阴影色相、颜色使用规则，等等。
6.  **交互原则。** 通用 agent 会漏掉的产品级 UX 立场。
7.  **定位。** 产品是什么，不是什么，差异化是什么，它与哪些竞争者可以区分开。

这七项合在一起，构成了**设计上下文**。在这里，设计和工程之间的边界变得模糊。一个懂产品的设计师会同时知道两边，agent 也应该如此。

代码也许承载了其中 40%，足以推断 token、布局约定、实体名、API 形状。另外 60% 存在于代码之外：设计文件、营销文案、各种决定。

一个只读代码的 agent 会把剩下的部分编出来。"把剩下的编出来" 正是为什么头脑风暴清单看起来合理、通用，却不像你的产品。

这就是 [Simon Willison](https://simonwillison.net/)、[Andrej Karpathy](https://karpathy.ai/) 和其他人一直称为 *context engineering* 的 AI 编码辅助部分：明确决定你的 agent 在做任何事之前应该知道什么。[Martin Fowler 的团队在 2026 年初写道](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html)，context engineering 正在成为使用编码 agent 交付的团队所需的 "the defining AI skill"。它没有 prompt engineering 那么光鲜，却更承重。

## **README 承载不了什么**

一份好的 README 是系统地图。设计上下文是系统的*性格*。这两者装不进同一个文档。

我可以在 README 里描述我们的架构。我可以列出我们的实体。我可以指向我们的组件库。但我没法把一整段 token 偏好、断点特例、字重别名，以及团队在一年设计评审里积累下来的 "绝不要用 X" 禁令，可信地放进 README。

这些都不属于入职文档。但 agent 在写一个界面之前需要知道所有这些。最接近的类比，是成熟设计系统发布的内容和声音文档。[Shopify Polaris](https://polaris.shopify.com/content) 有，[Atlassian Design System](https://atlassian.design/content) 也有。

[Brad Frost](https://bradfrost.com/) 和其他人多年来一直在写：文档决定了一个设计系统到底能活下来还是死掉；没有文档，代码里存在的系统永远不是团队真正使用的那个系统。给 agent 的 skill 也是同一个想法，只是被压缩并分诊成机器可加载的形式。

## Claude Code skill

我把设计上下文变成了一个 [Claude Code skill](https://www.anthropic.com/claude-code)：一个在你处理产品时会自动加载的目录。这个目录包含：

```
<product>-context/
├── SKILL.md                      
├── design.md                     
├── pointers.md                   
├── quickref/
│   ├── component-map.md          
│   ├── tokens.md                 
│   └── commands.md               
└── references/
    ├── overview.md               
    ├── functionality.md          
    ├── system-analysis.md        
    ├── typescript.md             
    ├── python.md                 
    ├── figma-component-mapping.md
    └── frontend/
        ├── ui-package.md
        ├── styles-package.md
        └── styles.md
```

1.  **SKILL.md** 是路由器。它的工作是根据当前任务决定要加载其他哪些文件。写 UI 文案？你需要 `design.md` §2 Brand voice。头脑风暴一个功能？你需要 `functionality.md` 和 `design.md`。路由器让上下文使用保持克制。[Katherine Yeh 今年早些时候在 Bootcamp 撰文](https://medium.com/design-bootcamp/a-designers-guide-to-organizing-ai-skills-and-tools-in-claude-code-f87477c35b82)，谈到设计师的 Claude Code 设置时，也更广泛地指出了同一点：当越来越多 skill 进入工作流后，困难的问题不再是每个 skill 做什么，而是知道哪个 skill 适合当前任务。
2.  **design.md** 是皇冠上的宝石。它有 11 个章节，覆盖产品身份、品牌声音、视觉基础、布局、UX 模式、组件编写、样式编写、Figma 约定、落地页模式、定位哲学，以及审美*感觉*。这是无法自动生成的文件，也是最重要的文件。
3.  **pointers.md** 是索引："当你需要 X，读取 Y。" 它是为 agent 的分诊写的。
4.  **quickref** 是从更重的参考文档里提取出来的、面向任务的紧凑速查表。agent 先加载 quickref，只有 quickref 解决不了时才打开重文档。
5.  **references** 是规范上下文文档的快照。它们放在 skill 内部，所以 skill 是自包含的。

## **我是怎么做到的**

我不是一次坐下来写完七个文件的。顺序很重要。

### **第一遍：架构和功能**

这两个文件是 agent 基本可以从代码里推导出来的。我让一个 sub-agent 看着代码库："写一份架构概览、服务地图、鉴权模型、数据流和已知约束。" 然后让第二个 sub-agent 处理功能："端到端流程、实体模型、界面范围。"

每个大约两小时，再轻度手工编辑。agent 很擅长这个。你基本是在让它总结代码里已经存在的东西。

### **第二遍：技术栈约定**

`typescript.md`、`python.md`、前端专项文档。这些也大多可以推导出来："用哪个包管理器，哪个 linter，组件仪式是什么，绝对不要做什么。"

agent 会根据你的配置和少数组件写出草稿。你再编辑那些配置里看不见、但对团队很承重的 "绝不" 规则。

### **第三遍：最难的 design.md**

这是 agent 无法自动生成的部分。

agent 可以枚举你的 token。它无法告诉你 "我们的主 CTA 是唯一的强调 CTA"。它可以读取你的通知文案。它无法告诉你 "我们不用装饰性 emoji；`useTranslation.ts` 里的那些 ✌️ 字符串是遗留的，不符合品牌。"

我分层写了 *design.md*。

![](/docs/ai-insights/2026-05/23/images/your-ai-agent-can-read-your-codebase-it-doesnt-know-your-product/03.webp)
*设计上下文的五层书册。由 OpenAI 生成。*

1.  **Tokens。** 颜色、字体字重和字号体系、间距、圆角、阴影。机械性内容。
2.  **UX 模式。** 把聊天作为指令、警告只做建议、空状态约定，等等。这些约定存在于代码里，但除非有人把它们标出来，否则并不明显。
3.  **品牌声音。** 语气维度。CTA 词汇。空状态三件套。通知三件套。反模式：触碰时需要清除的遗留文案、已经上线的拼写错误、早期时代留下来的 emoji。我把每条规则都锚定在从代码库逐字拉出来的真实字符串上。
4.  **审美。** 这是我花时间最多的一层。审美层是发生在 token *之上*的一切。构图原则。超越 token 表的颜色哲学（中性色是舞台，强调色是演员）。字体*感觉*（层级由字重承载，而不是字号）。标志性细节（对生成文本做句子级内联重写、在消费 CTA 上展示可见的单位成本、可编辑字段旁边放状态徽章）。
5.  **明确的反模式。** "*我们不是 neobrutalist，不是 glassmorphic，不是 maximalist，不是 playful，不是 austere-enterprise grey，也不是 AI-chatbot-glow。*" 还有一份自测：一个界面要跑过的十个 yes/no 问题。  
    Token 是 agent 能读到的。构图哲学只能被告知。

### **第四遍：路由器**

SKILL.md 不超过 150 行。带有触发短语的 YAML frontmatter。description 包含常见 prompt 词（"figma"、"component"、"design"、"brand"），以便自动调用 skill。

路由器正文是任务到文件的路由表、适用于每次修改的硬约束、"永远不要做 X" 的前端规则，以及逐字写入的完成前检查命令。我还加入了与其他 skill 叠加使用的组合说明：做创意工作时先运行一个头脑风暴 skill，再用这个 skill 来落地。

### **第五遍：持续发生的 refinement**

这个 skill 立刻就有用了，但大多数改进都来自实际使用它并观察它失败。

Inter 而不是 Graphik，是我读 Figma 渲染时抓到的 bug。"句号规则太粗暴" 这个细微点，则来自我注意到真实上线的通知正文并不以句号结尾；只有多句正文才会这样。

三族圆角体系，是我在代码库里发现五组不同 border-radius 组合后意识到的：它们并不是随意的。

每次 agent 做错什么，我都会问："什么上下文可以避免这个错误？" 然后把那段上下文写进 skill。几轮之后，agent 就不再犯那些错误了。

## **skill 何时产生回报**

四个用例，按回报有多明显从高到低排列。

### **Figma 到代码查找**

粘贴一个 Figma frame 名称或 URL。agent 会落到正确的功能目录，找到正确的基础组件。"代码里的 Project Builder modal 在哪？" 会在一轮内返回 `apps/web/src/features/projects/project-builder/ProjectBuilderModal.tsx`，不需要打开一份 136 KB 的映射文档。  
紧凑 quickref 完成解析；重文档只是 fallback。

### **组件编写**

写一个新的 `ProjectSidebarFilter` 组件，产出的代码第一次运行就通过 typecheck + lint，使用项目的 UI primitives（不是通用 div），使用 token（不是 hex），遵循 `memo` + `displayName` 仪式，并按照项目的 BEM 约定和按断点区分的圆角族来写 SCSS。

agent 不会猜这些仪式。它们在 skill 里。[Nick Babich 3 月在 UX Planet 调研了同一领域](https://uxplanet.org/must-have-ux-ui-design-skills-for-claude-code-364e93e3a614)：表现最好的设计专项 skill，都是那些明确编码项目约定的 skill，这样 agent 就不会退回到 Inter 加蓝色渐变的默认值。

### **产品头脑风暴**

"面向团队负责人有什么新功能？" 会扎根于真实实体模型。想法会使用真实 primitives（Project、Member、Workflow、Milestone）。重要的产品级约束（按使用量计费、警告只做建议、把聊天作为指令）会过滤这些想法。你不会得到 "要不我们做个 CRM"。那会打破定位。你得到的是适合的想法。

### **落地页和定位工作。**

"为我们的新 integrations 页面起草一个 hero" 会产出符合真实品牌声音的文案：使用真实差异点，提到产品的三个用户原型，并且用的是它们的真实名称。不是通用的 "unlock your potential" SaaS。

### **审计**

"这个界面感觉像我们吗？" 会运行 §11 自测，并逐项给出诚实的 yes/no。当周围没人时，它可以像一个还不错的设计评审替身。

## **技术上它如何工作**

这个 skill 是 `.claude/skills/<product>-context/` 下的一个目录。harness 会在会话开始时自动发现它。`SKILL.md` 有 YAML frontmatter，其中 `description` 字段包含触发短语。当用户 prompt 包含其中某个短语时，skill 就会加载。

路由器正文是任务到文件的表。不是每个任务都加载每个文件。加载是渐进式的：先打开路由器，然后打开与任务相关的 quickref，再打开 `design.md` 的章节，只有 quickref 解决不了时才打开完整参考文档。这样可以让上下文使用量足够低，以至于那份重型映射文档（136 KB）很少需要被加载。

参考文档要么放在 skill 里（作为 `references/*.md` 快照），要么通过路径引用（指向主仓库的规范文件）。两种我都试过。

按路径引用能让 skill 很小，并把规范文件保留为 single source of truth，代价是需要这个 repo。快照则让 skill 自包含。在哪里都能用，不需要 repo。最后我选择了快照，并放了一个 `references/README.md`，说明规范原件在哪里，并要求每季度重新同步。

## **什么有效，什么困难**

### **有效的地方**

-   第一版输出就是对的。迭代循环从 "写 → 修通用 SaaS 错误 → 再试一次" 变成 "写 → 小改 → 发布"。对一个设计很重的产品来说，这大约能把视觉工作的迭代轮次减半。
-   skill 是团队制品。它提交进 repo。队友安装后会自动获得同一份上下文。
-   它自带文档属性。读这个 skill 本身就是一种不错的入职方式。
-   Skill 可以组合。创意工作先跑一个头脑风暴 skill，再用这个 skill 做落地。Figma 到代码的 skill 负责转换，这个 skill 负责约定。把它们叠起来。
-   冷启动有效。一个加载了 skill 的新会话，会立刻产出感觉正确的输出。不需要先说 "让我先给你补一下我们的代码库背景"。

### **困难的地方**

-   写 `design.md` 要花好几个小时。架构和功能文件基本可以自动生成；设计文档不行。你需要一个有品味的人，也就是具备 [Paul Graham 在 "Taste for Makers" 中描述的能力](https://paulgraham.com/taste.html)：能区分好作品和坏作品，并知道为什么，才能把审美层说清楚。如果你的团队里没人能说 "我们不是 neobrutalist，原因是……"，那 skill 也说不出来。
-   skill 会漂移。一个快速交付的产品会演化自己的声音、界面和 token 尺度。skill 也必须演化。每季度复审是最低要求。
-   不是每个产品的设计上下文权重都一样。如果你的产品是 CLI 工具，`design.md` 的重要性会低一些。如果它是营销落地页，`system-analysis.md` 的重要性会低一些。投入要匹配你的产品实际是什么。
-   上下文预算是真实存在的。带 references 的完整 skill 可能很大。quickref/ 模式和渐进加载，才让它能放进 agent 的上下文窗口。
-   实现细节是 Claude Code 专属的。SKILL.md frontmatter 格式带有 Anthropic 风格。同样的*内容*可以用于 Cursor、Windsurf 或其他 agent，但你需要重构触发机制。

![](/docs/ai-insights/2026-05/23/images/your-ai-agent-can-read-your-codebase-it-doesnt-know-your-product/04.webp)
*一件作品被审阅并批准后的标记。品味变成了签名。由 OpenAI 生成。*

有几件事，这种方法解决不了。skill 捕捉的是规则，而不是品味。仍然需要有品味的人来写这些规则。对大规模或快速变化的设计系统来说，从 Figma 做实时检索胜过手工维护快照。skill 会漂移；骨架比具体内容更长寿，这正是它值得维护的原因。

## **元观点**

我的 agent 比我更擅长读我的代码库。它找到正确文件的时间，只够我开始打字。它记得 exports。它打字比我快。

它没有的是*观点*，无论是我的观点，还是产品的观点。skill 是一种方法，把那种观点从我的脑子里转移到 agent 的工作上下文里，一次转移一个文件。

每当 agent 产出感觉很通用的东西时，我都会问："为了避免这个错误，我会给一个新设计师或新开发者什么上下文？" 然后把那段上下文写进 skill。

## **模板**

一个通用结构起点：[**templates.zip**](https://murynmukha.com/downloads/templates.zip)  
把它放进 `.claude/skills/`，让一个 agent 读取你的代码库和设计文件，然后迭代。

## 参考资料

-   [面向编码 Agent 的 Context Engineering](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html)，Martin Fowler 的团队
-   [设计师组织 Claude Code 中 AI Skills 和工具的指南](https://medium.com/design-bootcamp/a-designers-guide-to-organizing-ai-skills-and-tools-in-claude-code-f87477c35b82)，Katherine Yeh，Bootcamp
-   [Claude Code 必备 UX/UI 设计 Skills](https://uxplanet.org/must-have-ux-ui-design-skills-for-claude-code-364e93e3a614)，Nick Babich，UX Planet
-   [Taste for Makers](https://paulgraham.com/taste.html)，Paul Graham
-   [Context engineering](https://simonwillison.net/2025/jun/27/context-engineering/)，Simon Willison
-   [On context engineering over prompt engineering](https://x.com/karpathy/status/1937902205765607626)，Andrej Karpathy
-   [维护设计系统（Atomic Design，第 5 章）](https://atomicdesign.bradfrost.com/chapter-5/)，Brad Frost

*如果你也做过类似的东西，比如产品上下文 skill、给 agent 用的设计品味文档，或者任何位于 token 之上的东西，我很想看看。可以在下面回复，或者通过* [*murynmukha.com*](https://murynmukha.com/) *找到我。*
