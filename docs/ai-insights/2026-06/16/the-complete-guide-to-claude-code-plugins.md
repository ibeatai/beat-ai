---
title: Claude Code 插件完全指南
author: zhaozhiming
url: https://ai.gopubby.com/the-complete-guide-to-claude-code-plugins-3e24ed7b8f7d
translated: 2026-06-16
excerpt: 深入剖析 Claude Code 的插件系统，包括插件概念、安装和使用、市场、局限性以及与其他 Coding Agent 插件系统的比较。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@45b2e72da3fb2ab31aed589bfd6a779314c7faff/ai-insights/2026-06/16/images/the-complete-guide-to-claude-code-plugins/01.thumb.webp
---

# Claude Code 插件完全指南

深入剖析 Claude Code 的插件系统，包括插件概念、安装和使用、市场、局限性以及与其他 Coding Agent 插件系统的比较。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@45b2e72da3fb2ab31aed589bfd6a779314c7faff/ai-insights/2026-06/16/images/the-complete-guide-to-claude-code-plugins/01.jpg)

在本系列文章中，我们介绍了几种自定义 Claude Code 的方法：使用 CLAUDE.md 文件预先解释项目背景和协作限制；使用斜杠命令和技能将重复性任务转化为可靠的工作流；以及使用钩子在执行过程中的关键节点强制执行行为边界。这些工具共同扩展了 Claude Code 的功能，使其能够更好地理解你的项目并适应你的工作方式。但这些工具目前大多各自独立运行。它们就像钢铁侠、蜘蛛侠和雷神：各自强大，但只有复仇者联盟齐心协力才能拯救世界。Claude Code 是否有类似的功能可以将这些强大的功能整合为一个整体呢？答案是肯定的：插件。插件将 Claude Code 的各项功能打包成一个可分发、可安装的单元，使其更易于重用和共享。

## 插件简介

让我们先来看看 Claude Code 对插件的最初官方定义：

> *插件是一种轻量级的方式，可以打包和共享以下任意组合：斜杠命令、子代理、MCP 服务器和钩子。*

官方文档还指出：

> *插件将成为我们打包和共享 Claude Code 自定义功能的标准方式，随着我们添加更多扩展点，我们将不断改进这种格式。*

换句话说，插件被定位为分发自定义功能的标准方式，其格式会随着更多扩展点的加入而不断演变。事实上，LSP、监控、主题和其他功能都是后来添加的。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@45b2e72da3fb2ab31aed589bfd6a779314c7faff/ai-insights/2026-06/16/images/the-complete-guide-to-claude-code-plugins/02.jpg)

插件之所以必要，核心原因在于重用。在单个项目中为自己配置 Claude Code 时，将命令、技能和钩子直接放在 `.claude/` 目录下无疑是最简单的方法。但一旦更换机器、启动新项目或想与同事共享配置，依赖手动复制就变得非常麻烦。手动复制没有版本控制，容易遗漏文件，甚至可能导致某些组件无法使用。例如，如果某个技能目录依赖于必须通过插件机制自动注册的命令、钩子或子代理，那么仅仅复制该目录是不够的。插件可以将一系列功能转换为一个命名清晰、版本明确、结构清晰的单元，从而使共享、安装、更新和移除变得易于管理和可重复。

因此，用户无需了解插件内部是通过命令还是钩子实现的。插件清单中的版本号也方便团队成员清晰地识别每个人使用的确切版本，并且插件可以安装在项目范围内，以便与代码仓库共享。此外，插件中的每个组件都使用插件名称作为其命名空间。例如，命令是 `/team-plugin:hello` 而不是 `/hello`，即使来自不同来源的多个插件包含同名组件，也能避免冲突。

## 安装和使用插件

要了解安装过程，首先需要区分 Marketplace 和插件。插件是你最终安装的功能包，而 Marketplace 是一个列出众多插件的目录。Marketplace 本身并不包含功能；它只是告诉 Claude Code 哪些插件可供安装。因此，安装插件通常包含两个步骤：首先添加 Marketplace，以便 Claude Code 可以浏览其内容；然后从中选择并安装特定的插件。官方文档中的比喻很贴切：Marketplace 就像一个应用商店。添加商店后，你可以浏览其货架，但你仍然需要选择并下载单个应用（插件）。

例如，Anthropic 的官方插件市场 [claude-plugins-official](https://github.com/anthropics/claude-plugins-official) 会在 Claude Code 启动时自动可用。你无需在安装官方插件之前添加该插件市场。要安装 `frontend-design` 插件，请使用 `plugin-name@marketplace-name` 格式：

```
/plugin install frontend-design@claude-plugins-official
```

对于第三方应用市场，你必须先添加该应用市场，然后再安装插件。例如，安装 [planning-with-files](https://github.com/OthmanAdi/planning-with-files) 插件时，其应用市场名称和插件名称均为 `planning-with-files`。添加应用市场时，请使用 GitHub 仓库所有者和仓库名称指定其来源：

```
/plugin marketplace add OthmanAdi/planning-with-files
/plugin install planning-with-files@planning-with-files
```

安装完成后，运行 `/reload-plugins` 以激活它。然后你可以使用命名空间命令，例如 `/planning-with-files:plan`。

你可以从多种来源添加市场。最常用的是 GitHub 的 `owner/repo` 简写形式，如上例所示。你也可以使用任何 Git 仓库的完整 URL，包括 GitLab、Bitbucket 或自托管仓库，并在 `#` 后附加分支或标签。此外，你还可以指向本地目录或远程的 `marketplace.json` 文件。共同的要求是，来源必须包含一个描述市场所列插件的 [.claude-plugin/marketplace.json](https://code.claude.com/docs/en/plugin-marketplaces#create-the-marketplace-file) 文件。

```

/plugin marketplace add https://gitlab.com/my-team/ai-plugins.git
/plugin marketplace add https://gitlab.com/my-team/ai-plugins.git
/plugin marketplace add https://gitlab.com/my-team/ai-plugins.git

/plugin marketplace add /Users/jimmy/work/ai-plugin-marketplace

/plugin marketplace add https://example.com/marketplace.json
```

安装和管理插件有两种方法。在 Claude Code 内部，`/plugin` 命令会打开一个包含四个标签页的交互式界面，方便浏览和选择插件。在 Claude Code 外部，`claude plugin` 命令行工具更适合在终端中执行脚本和批量操作。添加市场或安装插件时，还可以选择其作用域。用户作用域适用于你的所有项目；项目作用域会将配置写入 `.claude/settings.json` 文件，以便与仓库协作者共享；本地作用域仅适用于你当前所在的仓库。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@45b2e72da3fb2ab31aed589bfd6a779314c7faff/ai-insights/2026-06/16/images/the-complete-guide-to-claude-code-plugins/03.webp)

```
# install from outside of Claude Code
claude plugin install planning-with-files@planning-with-files
# select the installation scope, default is user
claude plugin install planning-with-files@planning-with-files 
```

你还可以绕过插件市场，使用 `--plugin-dir` 参数直接加载本地插件目录。这无需设置插件市场：只需在启动时指定目录即可。此功能主要用于开发和调试你自己的插件。现在，它还支持直接加载 `.zip` 压缩包，无需先解压。当使用 `--plugin-dir` 加载的插件与已安装的插件市场插件同名时，本地版本在当前会话中优先，允许你在不卸载已安装版本的情况下测试更改。

```

claude --plugin-dir /Users/jimmy/work/dev-helper

claude --plugin-dir /Users/jimmy/work/dev-helper.zip
```

日常管理非常简单。安装、启用或禁用插件后，请记得运行 `/reload-plugins` 命令来应用更改。你可以使用 `/plugin disable` 命令暂时禁用不需要的插件，或使用 `/plugin uninstall` 命令将其彻底移除。最后一点需要注意：删除插件市场也会卸载你从该市场安装的所有插件，因此在删除插件市场之前，请务必确认你不再需要这些插件。

## 开发插件

开发插件的门槛比你想象的要低。最小的插件就是一个目录，其中包含一个位于 `.claude-plugin/plugin.json` 的清单文件，该文件声明了插件的身份：

```json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

-   `name` 既是执行命令时使用的唯一标识符，也是命名空间
-   `version` 使用语义化版本控制来管理版本发布
-   `description` 显示在插件管理器中

一旦清单文件存在，插件就有了自己的身份标识。剩下的工作就是添加各种功能。

插件功能按照常规目录放置：命令放在 `commands/` 目录下，技能放在 `skills/` 目录下，子代理放在 `agents/` 目录下，钩子放在 `hooks/hooks.json`，MCP 配置放在 `.mcp.json`。如果需要代码智能，还可以添加 `.lsp.json` 文件。需要强调一个常见的陷阱：除了 `plugin.json` 文件之外，所有这些目录和文件都必须放在插件根目录下，而不是 `.claude-plugin/` 目录内，否则 Claude Code 将无法加载它们。整体结构大致如下：

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json     # the only file allowed in this directory
├── commands/
├── skills/
├── agents/
├── hooks/
│   └── hooks.json
└── .mcp.json
```

> ***什么是 LSP？*** *Claude Code 的 LSP 代码智能功能连接到语言服务器，提供项目级的代码理解，包括跳转到定义、查找引用、类型信息和符号分析。这有助于 Claude Code 更准确地读取和修改代码。*

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@45b2e72da3fb2ab31aed589bfd6a779314c7faff/ai-insights/2026-06/16/images/the-complete-guide-to-claude-code-plugins/04.jpg)

另一个需要注意的开发细节是 `${CLAUDE_PLUGIN_ROOT}` 变量。插件安装完成后，Claude Code 会将其复制到缓存目录中。如果 Hook 脚本或配置需要引用插件自带的文件，则不能硬编码相对路径或绝对路径。而应该使用 `${CLAUDE_PLUGIN_ROOT}` 来定位插件根目录。[在关于 Hooks 的文章](https://medium.com/ai-advances/a-complete-guide-to-claude-code-hooks-fcd3b262622d)中讨论的 `superpowers` 插件就采用了这种方法。它的 SessionStart Hook 会调用一个使用 `"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd"` 的脚本，从而确保无论插件安装在哪台机器上或缓存路径下，都能找到该脚本。

编写插件后，调试的最佳方法是什么？最简单的方法是使用 `--plugin-dir` 直接加载本地目录，而无需先将其发布到应用市场。在启动时指定插件目录，并在加载多个插件时重复此选项：

```
claude --plugin-dir ./my-plugin
```

在测试期间，如果你发现问题并修改了插件，请在 Claude Code 中运行 `/reload-plugins` 以重新加载更改而无需重新启动，从而避免重复重启。

如果你已经在 `.claude/` 目录下创建了一系列命令和技能，那么将它们转换为插件也很简单。创建一个包含清单文件的插件目录，将 `.claude/commands/`、`.claude/agents/` 和 `.claude/skills/` 目录的内容复制到该目录中，然后使用相同的格式将 `settings.json` 中的 `hooks` 对象移动到 `hooks/hooks.json` 中。这样就完成了从独立配置到插件的迁移。插件稳定并准备好发布后，添加一个描述市场内容的 `marketplace.json` 文件，将其放置在你的市场仓库中，其他人就可以像安装其他插件一样安装它了。官方网站提供了关于如何组织市场并将其提交到官方市场的[完整说明](https://code.claude.com/docs/en/plugin-marketplaces)，因此我们在此不再赘述。

## 插件的局限性和缺点

插件作为一种分发机制固然方便，但它仍是一项相对早期的功能。一旦开始将其用于团队协作、项目标准和复杂工作流程，就会发现一些明显的局限性。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@45b2e72da3fb2ab31aed589bfd6a779314c7faff/ai-insights/2026-06/16/images/the-complete-guide-to-claude-code-plugins/05.jpg)

### 插件子代理功能降低

第一个限制是插件内部的子代理会失去一些功能。正如之前在[关于钩子的文章](https://medium.com/ai-advances/a-complete-guide-to-claude-code-hooks-fcd3b262622d)中提到的，即使插件子代理的 frontmatter 中包含了 `hooks`、`mcpServers` 和 `permissionMode` 字段，出于安全考虑，它们在运行时也会被忽略。这意味着同一个子代理定义放在你自己的 `.claude/agents/` 目录中可能可以正常工作，但一旦将其打包并作为插件分发，其某些功能可能会悄无声息地失效。

这项限制背后的原因不难理解。子代理本身就是一个权限受限的执行单元。如果允许它在插件内部注册钩子、连接 MCP 服务器或更改权限模式，实际上就等于允许低权限角色修改执行过程的控制规则，从而破坏整个权限模型的安全边界。因此，从安全角度来看，这项限制是合理的，但插件作者需要事先了解这一点。你不能假设插件内部的子代理与本地子代理拥有完全相同的功能，受影响的逻辑必须在其他地方实现。

### 市场缺乏分类，插件内容不透明。

第二个问题同时影响到发现和信任。就发现而言，目前的插件市场就像一个扁平的插件列表，没有系统化的分类、标签或筛选功能。浏览主要依赖于插件名称和描述。在插件数量较少的情况下，这尚可接受，但随着生态系统的发展和更多类似插件的出现，找到合适的插件或比较多个备选方案将变得越来越困难。

内容不透明性需要格外谨慎。安装插件意味着在你的计算机上引入一些可以用你的权限执行任意代码的程序。插件的钩子可以运行 shell 命令，其 MCP 服务器可以建立外部连接，其命令可以将提示注入到上下文中。然而，在安装之前，通常很难清楚地了解软件包中包含的内容。官方文档明确指出，Anthropic 无法控制也无法验证插件中包含的 MCP 服务器、文件或其他软件，因此你必须确保在安装插件之前信任其来源。

### 插件粒度

第三个问题涉及插件的粒度。观察目前已发布的插件，可以发现一个有趣的现象：有些插件非常轻量级，可能只包含一个技能，例如之前介绍的 `planning-with-files` 插件。一个组件却占用了整个插件，仍然需要完整的 `plugin.json` 文件、目录结构和版本管理这一整套流程。实际上，这就像用一个集装箱来运输一个信封。

事实上，业界已经有了一种更轻量级的技能安装方式：`npx skills add <package>`。这款来自 Vercel Labs 的开源 CLI 工具定位为技能包管理器，类似于 npm 之于 JavaScript，只不过它管理的是 AI 智能体的技能包。关键在于，它**与智能体无关**，这意味着同一个技能可以被 Claude Code、Codex、Cursor 和 Gemini CLI 使用，而无需绑定到任何单一工具。

两者之间的根本区别在于它们运行的​​层级不同。技能（Skill）是内容：一个包含可选脚本和资源的 SKILL.md 文件。插件（Plugin）是一个容器，它将多个技能与钩子（Hooks）、子代理（Subagents）和 MCP 服务器协调在一起。`npx skills` 命令从 GitHub 仓库下载 SKILL.md 文件并将其放置在相应的目录中，本质上是一种技能文件传输机制。插件则在 Claude Code 的原生扩展系统中注册一整套功能组件。

那么，何时应该使用插件，何时应该使用 `npx skills` 呢？如果你只有一个独立的技能，`npx skills` 就足够了：它轻量级且简洁，并且同一个技能可以被其他智能体复用。但如果你需要分发一整套协作规范，那么插件才是正确的选择。

## Claude Code 插件与其他 Coding Agent 插件的比较

纵观更广泛的 Coding Agent 生态系统，我们可以发现一个有趣的趋势：进入 2026 年之后，几乎所有主流工具都独立地朝着同一个方向发展，即插件化。Claude Code 相对较早地采用了这种方法，而在 2026 年上半年，Codex、Cursor 和 Gemini CLI 也都添加了各自对应的机制。插件已经从 Claude Code 的一项独特功能演变为这一代 Coding Agent 的标配功能。

Codex 于 2026 年 3 月推出了插件，Cursor 紧随其后，于 2026 年 2 月在 2.5 版本中推出了插件，而 Gemini CLI 则使用扩展（Extensions）来提供类似的功能。以下是它们与 Claude Code 的比较：

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@45b2e72da3fb2ab31aed589bfd6a779314c7faff/ai-insights/2026-06/16/images/the-complete-guide-to-claude-code-plugins/06.webp)

如表格所示，这些插件机制大体相似。大多数都使用清单文件以及类似 `skills/`、`hooks/hooks.json` 和 `.mcp.json` 的结构。然而，它们的产品定位值得我们更仔细地关注：

-   Claude Code 专注于终端中的智能体编码以及捕获可重用的团队工作流程
-   Codex 将应用连接与技能和 MCP 并列，更加强调应用集成和跨服务自动化
-   Cursor 的插件与编辑器深度集成，强调端到端的产品开发工作流程，从阅读 Linear issue、将 Figma 设计转换为代码，到一键部署
-   Gemini CLI 的扩展类似于一种 CLI 扩展包格式，它将上下文和命令封装在 MCP 周围，使工具开箱即用，并与 Google Cloud 生态系统更紧密地集成

尽管这些机制提供类似的功能，但它们并不能直接互换。Coding Agent 插件目前还没有真正统一的标准。钩子就是一个典型的例子：Claude Code 和 Gemini CLI 都将钩子存储在 `hooks/hooks.json` 文件中，但它们的事件名称完全不同。Claude Code 使用 `UserPromptSubmit`、`PostToolUse` 和 `Stop` 等名称，而 Gemini CLI 使用 `BeforeAgent`、`AfterTool` 和 `SessionEnd`。因此，即使你完全照搬插件目录，它也可能无法在其他工具中正常工作。

## 插件的未来

这并非我们第一次看到这样的故事。2023 年底，OpenAI 发布了 [GPTs](https://openai.com/index/introducing-gpts/)，其愿景与如今的插件类似：任何人都可以创建定制的 AI 助手，并通过商店分发或盈利，**以 App Store 改变移动互联网的方式改变 AI**。但我们都看到了之后发生的一切。许多 GPTs 只不过是包装在外壳里的提示，商店缺乏有效的质量过滤和发现机制，而创作者也没有可持续的盈利模式。如今，GPTs 依然存在，但距离当初设想的全新生态系统还相去甚远。

插件显现出几个类似的警告信号：

-   它们的内容严重依赖于提示驱动的技能和命令，带来了相当大的商品化风险
-   市场发现和审查机制仍处于初级阶段
-   开发者缺乏明确的激励，大多数插件都由开源社区贡献，且没有既定的商业化途径

然而，与 GPTs 不同，插件并非旨在满足诸如撰写文案、创建图像或查找信息等通用需求。它们着眼于开发团队的关键生产需求：统一编码标准、获取领域知识以及自动化工作流程。一个已嵌入 `CI/CD` 流程的代码审查插件，不会仅仅因为热度消退就被弃用。这意味着，即使长尾效应降温，领先的高价值工作流程插件仍将拥有坚实的基础，得以继续使用。

插件会重蹈 GPTs 的覆辙吗？我认为答案取决于一个问题：Anthropic 是否仅仅将插件视为功能列表中的又一个勾选项，还是真心愿意持续投入资源，用于发现、质量标准和开发者激励？目前，插件的底层机制已经相当稳固，但生态系统基础设施仍有很长的路要走。

## 结论

插件解决的核心问题是将 Claude Code 分散的自定义功能集合转化为一个可管理、可分发的单元。结合本系列之前的文章，我们现在对 Claude Code 组件的使用和分发方式有了更清晰的了解。Claude Code 插件的设计已经相对完善，拥有成熟的安装、版本控制、市场分发和组件组织流程。然而，在更广泛的 Coding Agent 生态系统中，直接在另一个 Coding Agent 中复用插件仍然十分困难。真正的跨工具互操作性距离实现还有很长的路要走。

## 参考

-   Discover and install prebuilt plugins through marketplaces — Claude Code Docs
-   Create plugins — Claude Code Docs
-   Create and distribute a plugin marketplace — Claude Code Docs
-   Claude Code settings — Claude Code Docs
-   Plugins reference — Claude Code Docs
-   Plugins — OpenAI Codex Docs
-   Plugins — Cursor Docs
-   Gemini CLI extensions

关注我，了解 AI 和软件工程领域的最新进展。欢迎提出问题和发表评论。
</content>
