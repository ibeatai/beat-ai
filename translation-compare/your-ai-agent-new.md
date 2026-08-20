---
title: 你的 AI agent 能读懂代码库，却不懂你的产品
author: Gregory Muryn-Mukha
url: https://uxdesign.cc/your-ai-agent-can-read-your-codebase-it-doesnt-know-your-product-b5ea0cd77989
translated: 2026-05-23
tags:
  - Claude Code
  - Product Design
  - Artificial Intelligence
  - Design Systems
  - UX
---

# 你的 AI agent 能读懂代码库，却不懂你的产品

## **如何把品牌、模式、视觉语言这些代码里没有的东西，喂给 AI 编码 agent。**

![](/docs/ai-insights/2026-05/23/images/your-ai-agent-can-read-your-codebase-it-doesnt-know-your-product/01.webp)
*一本翻开的代码之书旁边，是一摞展开成扇形的设计样张——产品的两份"出土文物"，agent 只读得懂其中一份。由 OpenAI 生成。*

> ***TL;DR.*** *AI 编码 agent 可以 grep 你的整个代码库，但产出依然透着一股通用 SaaS 的味道——因为它缺的是你产品的* design context *：产品是怎么表现自己的、有哪些交互模式被你从原则上拒掉、是什么让这个产品看起来像「你的」。我把这些写成了一个结构化的 Claude Code skill，第一遍输出就不再"通用"了。下面是这套系统、每一块为什么存在，以及搭建它时哪里最难。*

我让 AI agent 给我们的产品头脑风暴一些新功能。不到一分钟它就甩回一份整齐的清单：通知中心、动态流、带分析 widget 的 dashboard、新手引导向导。功能本身都挺合理，但没有一个对得上我们的产品。

我们的核心交互是一段对话，不是"向导加 dashboard"那一套流程。通知中心默认了用户在被动处理一堆需要分类的消息；onboarding 向导默认了一段线性的首次使用流程——但这正是我们刻意不做的东西。每一条建议都不声不响地踩破了我们早就立下的某条原则。

agent 拿到的是整个代码库的访问权限。问题完全不出在这里。

问题出在另一种形态的知识——从设计的视角看，*这个产品到底是什么*。它怎么表现自己。哪些交互模式我们从原则上拒绝。它怎么开口说话。一个圆角半径、一个错误状态，怎么样才算"on brand"。到底是什么，让一屏看起来像**我们**，而不是像每一家从 YC batch 里走出来的 B2B SaaS。

这些东西一项都没躺在代码里。它们躺在 taste 里、躺在某个 Figma 文件里、躺在我们上周二在 Slack 里如何措辞讨论某个功能的那段对话里、躺在团队里六七个人的脑袋里。

![](/docs/ai-insights/2026-05/23/images/your-ai-agent-can-read-your-codebase-it-doesnt-know-your-product/02.webp)
*桌面上散开的四张样本卡。四种模式都审过了，没有一种对得上。由 OpenAI 生成。*

这就是 agent 缺的那种形态的知识。我们把它叫做产品的 **design context**。

问题不在 agent 能读到什么，而在代码本身没有把什么写下来。

## **一个产品（对 AI 而言）到底是什么**

想清楚怎么把这些东西喂给 agent 之后，我落到了七种不同形态的知识上。每一种都是不一样的东西，没有一种放得进 README。

1.  **架构。** 服务、数据存储、auth、数据流、硬约束。
2.  **功能。** 端到端流程、领域实体模型、产品的各个界面（surface）。
3.  **技术栈惯例。** yarn/pnpm、monorepo 布局、pre-completion 检查、import 规则。
4.  **品牌口吻。** 语气的几个维度（正式度、创造性、坚定程度、emoji 用不用），CTA 词汇、文案套路、反模式。也就是产品实际上是怎么跟用户说话的。
5.  **视觉识别。** 字体、字重、几套圆角体系、阴影色调、用色规则等等。
6.  **交互原则。** 那些通用 agent 会忽略掉的、属于产品层面的 UX 立场。
7.  **定位。** 产品是什么、不是什么、有哪些差异点、能和哪些竞品明确区分开来。

这七项加在一起，构成了 **design context**。设计和工程的边界，在这里变得模糊。一个真正"懂产品"的设计师两边都得懂——agent 也应该。

代码大概只承载其中 40%，刚好够 agent 推断出 token、布局惯例、实体名、API 形态。剩下 60% 都在代码之外：在设计文件里、在 marketing 文案里、在大大小小的决策里。

只读代码的 agent，会把剩下那 60% 自己"编"出来。所谓"编出来"，就是为什么一份头脑风暴清单看上去既合理、又通用、却怎么也不像你的产品。

这正是 [Simon Willison](https://simonwillison.net/)、[Andrej Karpathy](https://karpathy.ai/) 这些人一直在说的 AI 编码辅助里的那一块：*context engineering*——一门关于"在 agent 动手之前，明确决定它该知道什么"的工程纪律。[Martin Fowler 团队在 2026 年初写道](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html)，对于用 coding agent 交付产品的团队来说，context engineering 正在成为"那项定义性的 AI 技能"。它不像 prompt engineering 那么光鲜，但更承重。

## **README 装不下什么**

一份好的 README 是系统的一张地图；design context 则是这个系统的*性格*。两样东西没法塞进同一份文档。

我可以在 README 里描述我们的架构，可以列出我们的实体，可以指向我们的组件库。但我没法把这种东西放进 README 又不让人发笑：一段关于 token 偏好的段落、那些只在某个 breakpoint 才生效的特例、字重的别名、以及"绝对不用 X"这种禁忌——一个团队会在一年的设计 review 里一点一点攒下来的东西。

这些都不该出现在新人 onboarding 文档里。但 agent 在写一屏界面之前，每一项都得知道。最接近的参照物，是成熟设计系统会发布的那种 content & voice 文档：[Shopify Polaris](https://polaris.shopify.com/content) 有，[Atlassian Design System](https://atlassian.design/content) 也有。

[Brad Frost](https://bradfrost.com/) 等人多年来都在写：设计系统的生死，其实落在文档上；没有文档，代码里那个系统永远不会真的成为团队在用的那个系统。给 agent 写的一个 skill，本质是同样的想法——只是为了机器加载，做了压缩和优先级筛选。

## Claude Code skill

我把 design context 落成了一个 [Claude Code skill](https://www.anthropic.com/claude-code)：一个目录，当你为这个产品工作时自动加载。目录长这样：

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

1.  **SKILL.md** 是一个 router。它的工作是替当前任务决定，要从其他文件里加载哪几份。在写 UI 文案？你需要 `design.md` 的 §2「品牌口吻」。在头脑风暴一个新功能？你需要 `functionality.md` 加 `design.md`。这个 router 让 context 的使用保持克制。[Katherine Yeh 今年早些时候在 Bootcamp](https://medium.com/design-bootcamp/a-designers-guide-to-organizing-ai-skills-and-tools-in-claude-code-f87477c35b82) 谈过一个更宽泛、但本质相同的观察：随着工作流里装入的 skill 越来越多，难点不再是每个 skill 各自做什么，而是知道当前任务该用哪一个。
2.  **design.md** 是镇山之宝。一共 11 节，覆盖产品身份、品牌口吻、视觉基础、布局、UX 模式、组件写法、样式写法、Figma 约定、landing page 套路、定位哲学，以及审美的*手感*。它是整套体系里没法自动生成的那个文件，也是最关键的那个。
3.  **pointers.md** 是一份索引：「你需要 X 时，去读 Y。」是写给 agent 做分诊判断用的。
4.  **quickref** 是从那些更厚的参考文档里抽出来、面向具体任务的紧凑 cheatsheet。agent 先加载 quickref，只有在 quickref 解决不了时才去打开厚文档。
5.  **references** 是那些权威 context 文档的快照。它们住在 skill 内部，让这个 skill 自洽。

## **我是怎么走到这一步的**

我并不是一坐下来就把七个文件写完。次序很重要。

### **第一轮：架构和功能**

这两份是 agent 基本能从代码里推出来的。我把一个 sub-agent 指向代码库：「写一份架构总览、一张服务地图、auth 模型、数据流、已知约束。」然后再起一个 sub-agent 处理功能：「端到端流程、实体模型、整个界面覆盖范围。」

每份大约两小时，再手工润一遍。agent 干这个挺擅长——你其实是在让它把代码里本来就有的东西总结出来。

### **第二轮：技术栈惯例**

`typescript.md`、`python.md`、前端专属文档。基本也都能推：「用哪个包管理器、用哪个 linter、组件的'仪式'是什么、有哪些事是绝对不做的。」

agent 读你的几份配置文件加一小撮组件，先写一稿。你只需要补上一些不在配置里、但对团队真正承重的那些"never"规则。

### **第三轮：那一个难的——design.md**

这一份，agent 是自动生成不了的。

agent 可以把你所有的 token 列出来，但它没法告诉你「我们的主 CTA，是那个唯一带强调色的 CTA」。它可以读你的通知文案，但它没法告诉你「我们不用装饰性 emoji——`useTranslation.ts` 里那几个 ✌️ 是遗留的、不符合品牌」。

我把 *design.md* 一层一层地写。

![](/docs/ai-insights/2026-05/23/images/your-ai-agent-can-read-your-codebase-it-doesnt-know-your-product/03.webp)
*design context 的五层叠卷。由 OpenAI 生成。*

1.  **Token。** 颜色、字重和字号阶梯、间距、圆角、阴影。机械活儿。
2.  **UX 模式。** 「对话即指令」「警告只作提示」、空状态的惯例，等等。这些惯例其实活在代码里，但如果没人贴标签，它们不会显眼。
3.  **品牌口吻。** 语气维度。CTA 词汇。空状态三件套。通知三件套。反模式：碰到就该清掉的遗留文案、已经发布出去的错别字、上个时代留下来的 emoji。每一条规则，我都用代码库里原样拎出来的一段真实字符串来落地。
4.  **审美。** 这一层我花的时间最多。审美层是所有发生在 token *之上* 的东西。构图原则。比 token 表更上层的颜色哲学（中性色是舞台，强调色是演员）。排版的*手感*（用字重承载层级，而不是字号）。标志性的细节（在生成文本上做句子级的就地改写、在花钱的 CTA 上把单价显式露出、在可编辑字段旁边贴状态徽章）。
5.  **明确写出来的反模式。** *"我们不是 neobrutalist、不是 glassmorphic、不是 maximalist、不是 playful、不是企业灰那种 austere-enterprise grey、也不是 AI-chatbot 那种发光风。"* 再加一份自检：十个 yes/no 问题，每一屏都要跑一遍。
    Token agent 自己能读到。构图哲学，只能你亲口告诉它。

### **第四轮：router**

SKILL.md 控制在 150 行以内。YAML frontmatter 里写好触发短语。description 字段里塞了常见的 prompt 词（"figma"、"component"、"design"、"brand"），让这个 skill 被自动唤醒。

router 的主体是一张「任务 → 文件」路由表、一组对每次改动都生效的硬约束、几条前端"绝对不做 X"的规则，以及原样照抄的 pre-completion 校验命令。我还加了几条与其他 skill 叠用时的组合说明：做创作类工作时，先跑一个 brainstorming skill，再跑这个 skill 把它拉回地面。

### **第五轮：打磨，一个还在持续发生的轮次**

skill 立刻就有用了，但大部分的改进，都来自——用它，再看着它失败。

「Inter 而不是 Graphik」是我从一张 Figma 渲染里看出来的 bug。「句号规则太一刀切」这条微妙之处，是我注意到真正发出去的通知正文很少以句号结尾——只有多句正文才会——才意识到的。

那套"三族圆角体系"是怎么浮现的：我先在代码库里翻出五组不同的 border-radius 取值，然后才意识到——这些取值不是随便定的。

每次 agent 出错，我都会问一句：「什么样的 context 本来可以避免这次出错？」然后把那段 context 写进 skill。几轮下来，agent 就不再犯那些错了。

## **这个 skill 在哪些场景下值回票价**

四个使用场景，按"值不值"从高到低排。

### **Figma → 代码 的查找**

把一个 Figma frame 名或者 URL 贴进去。agent 会落到正确的 feature 目录、用上正确的基础组件。问「Project Builder 这个 modal 在代码里哪？」，它一轮就回 `apps/web/src/features/projects/project-builder/ProjectBuilderModal.tsx`，根本不需要打开那份 136 KB 的映射文档。
紧凑的 quickref 负责完成解析；厚文档只是 fallback。

### **写组件**

「写一个新的 `ProjectSidebarFilter` 组件」——一遍就生成出能过 typecheck + lint 的代码：用项目自己的 UI 基础组件（不是裸的 div）、用 token（不是写死的 hex）、走 `memo` + `displayName` 那套"仪式"、SCSS 按项目自己的 BEM 约定写、按 breakpoint 走对应族的圆角。

agent 不用去猜这些"仪式"，它们全在 skill 里。[Nick Babich 三月份在 UX Planet 上扫过同一片地带](https://uxplanet.org/must-have-ux-ui-design-skills-for-claude-code-364e93e3a614)：表现最好的那一类设计相关 skill，都是把项目约定显式编码下来的那种——这样 agent 就不会一退回默认就掉进「Inter + 蓝色渐变」的坑。

### **产品头脑风暴**

「给 team lead 们想一个新功能？」这种问题会落在真实的实体模型上。点子用真实的基础对象（Project、Member、Workflow、Milestone）。真正起作用的那几条产品层面约束（按用量计费、警告只作提示、对话即指令）会先过滤掉一批想法。你不会再得到「要不要加一个 CRM？」——那会打破定位。你拿到的，是对得上产品的点子。

### **Landing page 和定位类工作**

「给我们新的 integrations 页拟一段 hero」——出来的文案是真用品牌口吻写的：用的是真的差异点，提到产品的三类用户时用的是它们真实的名字。不是那种「释放你的潜能」式的通用 SaaS 文案。

### **自审**

「这一屏看上去像我们吗？」它会跑一遍 §11 的自检表，逐条给你诚实的 yes/no。在没人能帮你做设计 review 的时候，这是个不错的替代品。

## **技术上是怎么跑起来的**

这个 skill 是位于 `.claude/skills/<product>-context/` 的一个目录。harness 会在 session 启动时自动发现它。`SKILL.md` 的 YAML frontmatter 里有一个 `description` 字段，里面塞着触发短语。当用户的 prompt 命中其中之一时，这个 skill 就会被加载。

router 的主体就是那张「任务 → 文件」表。不是每个任务都会加载所有文件。加载是渐进的：先打开 router，再打开与任务相关的 quickref，然后是 `design.md` 的某些节，最后只有在 quickref 解决不了时，才打开完整的参考文档。这套机制把 context 占用压得足够低，那份 136 KB 的厚映射文档很少被真正加载。

参考文档要么住在 skill 里（作为 `references/*.md` 的快照），要么以路径引用的方式存在（指向主 repo 里的权威文件）。两种我都试过。

路径引用让 skill 保持小巧，并让权威文件成为唯一真相源——代价是必须带上那个 repo。快照则让 skill 自洽，到哪都能跑、不需要 repo。我最后选了快照，加了一份 `references/README.md`，里面写明权威原始文件在哪，并要求每季度重新 sync 一次。

## **哪里好用，哪里难做**

### **好用的地方**

-   第一遍输出就是对的。迭代循环从「写 → 修通用 SaaS 错 → 再来一次」变成了「写 → 小修一下 → 上线」。对一个设计偏重的产品来说，视觉相关的迭代轮数大约减半。
-   skill 是团队级的产物。它进 repo、commit 进版本控制。队友装好就自动拿到同一份 context。
-   它自带文档属性——读这个 skill，本身就是一种合格的新人 onboarding。
-   多个 skill 可以叠起来用。创作工作上先跑一个 brainstorming skill，然后跑这个把它拉回地面；做 Figma 到代码的翻译时，先跑一个 Figma-to-code skill，再跑这个补约定。叠着用。
-   冷启动可行。一个新 session 把 skill 加载进来，立刻就能产出"感觉对"的输出，不需要先来一段「我先跟你介绍下我们代码库」。

### **难做的地方**

-   写 `design.md` 要花好几个小时。架构和功能那两份文件基本能自动生成；设计这份不能。你需要一个有 taste 的人——[Paul Graham 在《Taste for Makers》里](https://paulgraham.com/taste.html)描述过那种能力：分得清好坏、并能讲清楚为什么——来把审美这一层说出来。如果团队里没人能说出口「我们就 NOT neobrutalist，原因如下」，那这个 skill 也没法替你说出来。
-   skill 会漂移。一个发布节奏快的产品，会持续演化自己的口吻、界面、token 阶梯。skill 也必须跟着演化。每季度 review 一次是底线。
-   不是所有产品的 design context 都同样重要。如果你的产品是一个 CLI 工具，`design.md` 没那么关键；如果是一个 marketing 落地页，`system-analysis.md` 没那么关键。按产品实际样貌分配精力。
-   context 预算是真实存在的限制。一个带 references 的完整 skill 体积可以很大。quickref/ 的模式加上渐进式加载，才让它装得进 agent 的 context window。
-   实现细节是和 Claude Code 绑死的。SKILL.md 的 frontmatter 格式带着浓重的 Anthropic 味道。同样的*内容*在 Cursor、Windsurf 或者别的 agent 上也成立，但触发机制那部分得另外搭。

![](/docs/ai-insights/2026-05/23/images/your-ai-agent-can-read-your-codebase-it-doesnt-know-your-product/04.webp)
*一件作品被审过、被批准的印记。把 taste 做成签名。由 OpenAI 生成。*

这套做法解决不了的事情，也有几件。skill 抓得住规则，抓不住 taste——有 taste 的那个人，仍然得亲手把规则写出来。在规模更大、或者设计系统迭代很快的场景下，从 Figma 实时拉取会胜过手工维护的快照。skill 会漂移；但骨架比那些具体细节活得更久——正是这一点，让维护它仍然值得做。

## **元一点的总结**

我的 agent 在读"我"的代码库这件事上，已经比我自己强了。我刚开始打字，它就找到了对的文件。它记得所有的导出。它打字也比我快。

它唯一缺的，是*视角*——我的视角，或者说产品的视角。这个 skill 就是一种把视角从我脑袋里搬到 agent 工作 context 里的办法——一份文件接一份文件地搬。

每次 agent 出来的东西显得"通用"，我都会问一句：「如果一个新设计师或者新开发者犯了这个错，我会给他怎样的上下文来避免？」然后把那段上下文写进 skill。

## **模板**

一个通用的结构起点：[**templates.zip**](https://murynmukha.com/downloads/templates.zip)
丢进 `.claude/skills/`，让一个 agent 看着你的代码库和设计文件，开始迭代就好。

## 参考资料

-   [Context Engineering for Coding Agents](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html)，Martin Fowler 团队
-   [A Designer's Guide to Organizing AI Skills and Tools in Claude Code](https://medium.com/design-bootcamp/a-designers-guide-to-organizing-ai-skills-and-tools-in-claude-code-f87477c35b82)，Katherine Yeh，Bootcamp
-   [Must-Have UX/UI Design Skills for Claude Code](https://uxplanet.org/must-have-ux-ui-design-skills-for-claude-code-364e93e3a614)，Nick Babich，UX Planet
-   [Taste for Makers](https://paulgraham.com/taste.html)，Paul Graham
-   [Context engineering](https://simonwillison.net/2025/jun/27/context-engineering/)，Simon Willison
-   [On context engineering over prompt engineering](https://x.com/karpathy/status/1937902205765607626)，Andrej Karpathy
-   [Maintaining Design Systems (Atomic Design, Chapter 5)](https://atomicdesign.bradfrost.com/chapter-5/)，Brad Frost

*如果你也搭过类似的东西——一个 product-context skill、一份给 agent 看的"设计品味"文档、任何活在 token 之上的东西——我很想看看。在下方留言，或者来 [murynmukha.com](https://murynmukha.com/) 找我。*
