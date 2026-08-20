---
title: 构建一套 agent harness
author: Heeki Park
url: https://heeki.medium.com/building-an-agent-harness-31942331d605
translated: 2026-05-22
excerpt: Prompt engineering、context engineering，现在轮到 harness engineering。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/building-an-agent-harness/01.webp
---

# 构建一套 agent harness

Prompt engineering、context engineering，现在轮到 harness engineering。

Prompt engineering 随着 2022 年底 ChatGPT 的发布进入大众视野，它定义了人与大语言模型高效互动的方式。Context engineering 在去年（2025 年）走到台前，因为开发者们都想在不突破上下文上限的前提下，往 prompt 里塞进尽可能多的有用信息。而 harness engineering 成了今年的热门话题——开发者着手优化围绕模型的脚手架，确保 agent 能处理好一个任务请求、并返回尽可能好的结果。

LangChain [写过](https://www.langchain.com/blog/improving-deep-agents-with-harness-engineering)一篇关于用 harness engineering 改进 deep agent 的文章，展示了如何在保持底层模型不变的情况下，通过调整系统提示词、工具调用和 middleware 来优化 agent 表现。其中那一层 middleware 尤其值得一提：它注入目录上下文以加快 agent 上手速度、校验输出以做质量验证、并强制执行时间预算以确保工作能在合理时间内完成。这说明 agent 开发者可以站在前沿实验室的肩膀上，做出更出色的结果，而不是单纯等着下一个模型版本来提升质量。

Birgitta Böckeler [写过](https://martinfowler.com/articles/harness-engineering.html)一篇面向编码 agent 用户的 harness engineering 文章，她用一个简单的等式来定义它：`agent = model + harness`。她这样解释一套构建良好的 harness 想达成的目标：

> 一套构建良好的外层 harness 服务于两个目标：它提高了 agent 第一次就把事情做对的概率，并且它提供了一个反馈回路，能在尽可能多的问题到达人类眼前之前就自我纠正掉。

AWS 刚刚为 AgentCore [推出](https://aws.amazon.com/blogs/machine-learning/get-to-your-first-working-agent-in-minutes-announcing-new-features-in-amazon-bedrock-agentcore/)了一套全新的托管式 agent harness，用一个简单的、基于配置的部署方式取代了原先繁琐的 agent 构建流程。

在这篇博客里，我会讲一讲自己在 agent 平台原型 Loom 中构建 agent harness 的持续探索过程。我还会分享把 AgentCore 的新 harness 集成进来、作为构建 agent 的又一个选项的体验。

### 深入 Loom 的引擎盖之下

在我介绍 Loom 的[那篇文章](https://heeki.medium.com/introducing-loom-an-agent-platform-66e7db019cdb)里，我列出了客户挑战 #4：软件部署需要严格的测试，对 AI 生成的代码尤其如此。正因如此，我使用了一个预先写好的 agent，它本质上是借助 feature flag 来开启与 memory、工具和其他 agent 的集成。这样就能走标准的代码扫描流程，也避免了为不受信任的代码执行专门搭一个隔离环境的需要。

![通过配置注入完成的预写式 agent 构建](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/building-an-agent-harness/01.webp)

为做到这一点，我注入了一个 `AGENT_CONFIG_JSON` 环境变量，其载荷如下：

```json
{
  "system_prompt": "persona, instructions, guidelines",
  "model_id": "us.amazon.nova-2-lite-v1:0",
  "max_tokens": 16384,
  "integrations": {
    "mcp_servers": [
      {
        "name": "utilities",
        "enabled": true,
        "transport": "streamable_http",
        "endpoint_url": "https://example.heeki.cloud/mcp",
        "auth": {
          "type": "oauth2",
          "credential_provider_name": "<agentcore-credential-provider-for-utilities>",
          "well_known_endpoint": "https://cognito-idp.<region>.amazonaws.com/<cognito-pool-id>/.well-known/openid-configuration",
          "scopes": "utilities/invoke ..."
        }
      }
    ],
    "a2a_agents": [
      {
        "name": "agentforce-assistant",
        "enabled": true,
        "endpoint_url": "https://api.salesforce.com/einstein/ai-agent/a2a/<agent-identifier>",
        "auth": {
          "type": "oauth2",
          "credential_provider_name": "<agentcore-credential-provider-for-agentforce-assistant>",
          "well_known_endpoint": "https://<organization-id>.develop.my.salesforce.com/.well-known/openid-configuration"
        }
      }
    ],
    "memory": {
      "enabled": true,
      "resources": [
        {
          "name": "demo_employee_assistant",
          "memory_id": "demo_employee_assistant-RNDrSHC2P5",
          "arn": "arn:aws:bedrock-agentcore:<region>:<account-id>:memory/demo_employee_assistant-RNDrSHC2P5"
        }
      ]
    }
  }
}
```

`system_prompt` 参数把 persona、instructions、guidelines 这几个独立的输入合并起来，用于初始化 agent。

```
agent = Agent(
    system_prompt=config.system_prompt,
    model=model,
    tools=tools,
    hooks=hooks,
)
```

`model_id` 和 `max_tokens` 参数用于配置 Bedrock 模型。将来我打算更新这部分，以支持其他第三方模型供应商。

```
model = BedrockModel(
    model_id=config.model_id,
    max_tokens=config.max_tokens,
    streaming=True,
)
```

`integrations.mcp_servers` 和 `integrations.a2a_agents` 参数用于有条件地定义 agent 的工具列表。下面的代码我为了简洁略作了简化。

```
if config.integrations.mcp_servers:
    enabled_servers = [s for s in config.integrations.mcp_servers if s.enabled]
    if enabled_servers:
        mcp_clients = build_mcp_clients(config.integrations.mcp_servers)
        tools.extend(mcp_clients)
        logger.info("Loaded %d MCP tool client(s)", len(mcp_clients))if config.integrations.a2a_agents:
    enabled_agents = [a for a in config.integrations.a2a_agents if a.enabled]
    if enabled_agents:
        a2a_clients = create_a2a_clients(config.integrations.a2a_agents)
        if a2a_clients:
            tools.extend(a2a_clients)
            logger.info("Loaded %d A2A client(s)", len(a2a_clients))
```

`integrations.memory` 参数用于定义在生命周期事件中是否需要一个 hook 来与 memory 交互。

```
if config.integrations.memory.enabled:
    memory_store_id = None
    if config.integrations.memory.resources:
        memory_store_id = config.integrations.memory.resources[0].memory_id
    memory_hook = MemoryHook(memory_store_id=memory_store_id)
    hooks.append(memory_hook)
    logger.info("Enabled AgentCore Memory hook (store_id=%s)", memory_store_id)
```

这种做法让我可以在部署时完成 agent 配置，并根据用例需求，按需选择性地设置上述每一项配置。

但如果我想在运行时修改 agent 配置呢？让终端用户挑选自己偏好的模型是很常见的需求。在与 agent 交互的过程中追加工具集成，同样很常见。

### 权衡部署时配置与运行时配置

那这要怎么实现？部署时的配置充当默认设置，而运行时配置则允许用户用自己的选择去覆盖这些默认值。

![默认的部署配置与用于覆盖的运行时配置](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/building-an-agent-harness/02.webp)

invoke 载荷中包含了在运行时覆盖模型、以及添加 connector（工具或其他 agent）的选项。这部分通过 `InvokeRequest` 类[来定义](https://github.com/heeki/loom/blob/bd0234ae69983429fa4734c7383880522c4ac41b/backend/app/routers/invocations.py#L46)。

```json
{
  "prompt": "What is the weather in New York?",
  "session_id": "existing-session-id-or-null",
  "model_id": "us.amazon.nova-2-lite-v1:0",
  "credential_id": 1,
  "bearer_token": "<your-bearer-token>",
  "connector_ids": [3, 7]
}
```

-   `prompt`（必填）：用户消息
-   `session_id`：复用一个已有会话，省略则创建一个新会话
-   `model_id`：覆盖 agent 的默认模型，必须在 `allowed_model_ids` 之内
-   `credential_id`：用于生成 token 的内部 authorizer 标识符
-   `bearer_token`：手动生成的 bearer token
-   `connector_ids`：为本次调用挂载的内部 connector id

举个例子，我可以为某个特定 agent 配置一组允许使用的模型，终端用户就从这组里挑选。

![管理员视图：为终端用户选定允许使用的模型](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/building-an-agent-harness/03.webp)

接着终端用户就能去选他偏好的模型。

![终端用户视图：选择偏好的模型](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/building-an-agent-harness/04.webp)

如果用户选了另一个模型，那这个选择就会随 invoke 请求一起传过去。agent 随后[会用](https://github.com/heeki/loom/blob/bd0234ae69983429fa4734c7383880522c4ac41b/agents/strands_agent/src/handler.py#L173)被选中的那个模型。

### 在运行时添加工具

在运行时更新模型相当容易，因为那只是 agent 里多处理一个参数而已。但在运行时添加新工具有个曲折之处，需要额外当心。

![运行时 connector 启用视图](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/building-an-agent-harness/05.webp)

在我那篇介绍性文章里，我列出了客户挑战 #6：身份传播需要在层层委派的 actor 链中得到管理。我提出过一个简化方案——刻意把架构约束在单跳（single hop）以内。这也是当今许多 agent 用户界面采用的做法：用户可以按需启用某个连接。

这种做法的关键好处，是在用户切换开启一个新 connector 的那一刻就捕获其凭据。我推测，在这个时间点捕获凭据带来的用户授权疲劳是*最小*的，因为此时用户对与该集成交互这件事意图非常明确。

把凭据的捕获拖到真正需要时才做并不合理，因为用户可能已经为了一个长耗时任务走开很久了。打个比方，那就好比你在 Claude Code 里启动一个长耗时任务、起身离开，回来却发现它在你走后 10 秒就暂停了执行，正请求你授权它进入某个目录或读取某个文件——体验是一样的糟糕。

要注意，这是对 agent 身份与权限委派问题的一种*简化*，而不是解决方案。RFC 8693 是一个 token exchange 提案，它通过在 JWT 的 actor claim 中携带信息来帮助处理委派。

在上面那张示例截图里，我用 exa.ai/mcp 注册了一个账号并创建了一个 API key，免费档每月给 1000 次请求。对基础原型开发来说够用了。虽然我可以在管理员层级配置 MCP server，但这样做实际上会把那个 API key 设给每一个启用该 connector 的用户。在某些场景下，这没问题。在另一些场景下，按用户区分 API key 才说得通。

![MCP connector 的按用户 API key](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@d636560ddb58a0d75173d1977cf7a323f1319997/ai-insights/2026-05/22/images/building-an-agent-harness/06.webp)

我选了后一条路。所以当用户启用 connector 时，会弹出一个对话框，要求用户输入个人 API key。这个个人 key 随后会在向 MCP server 发请求时被用上。

要注意，在撰写本文时，AgentCore credential provider 还不支持按用户区分的 API key。因此我最终是通过 agent 请求来注入这个 header 的，而没有走 credential provider。

### 把托管式 agent harness 集成进 Loom

为了走到现在这个状态，实现过程经历了相当多的迭代。一方面，自己构建一套 agent harness 让我学到很多东西，也给了我极大的灵活性，比如为遥测和成本估算加上自定义日志。另一方面，对一个 agent 平台来说，这或许已经超出了一个人愿意投入的工程量。

AgentCore 的 harness 刚刚推出，目的就是简化 agent 的部署、以及与各类 context provider 的集成。

我尤其欣赏这套托管式 agent harness 的一点，是它在模型选择和工具集成上同时提供了部署时和运行时的选项。比如在部署时，我有下面这些选项，这里用 CLI 来举例说明配置的可能性：

```
aws bedrock-agentcore-control create-harness \
  --region us-east-1 \
  --harness-name "loom-harness-example" \
  --execution-role-arn "arn:aws:iam::${ACCOUNT_ID}:role/loom-harness-role" \
  --system-prompt "persona, instructions, guidelines" \
  --model "us.amazon.nova-2-lite-v1:0" \
  --max-iterations 75 \
  --max-tokens 16384 \
  --tools '[{"type": "remote_mcp", "name": "exa", "config": {"remoteMcp": {"url": "https://mcp.exa.ai/mcp"}}}]' \
  --memory '{"optionalValue": {"agentCoreMemoryConfiguration": {"arn": "arn:aws:bedrock-agentcore:${REGION}:${ACCOUNT_ID}:memory/${MEMORY_ID}"}}}'
```

类似地，在运行时，我有下面这些选项来覆盖模型和工具：

```
response = client.invoke_harness(
    harnessArn=HARNESS_ARN,
    runtimeSessionId=SESSION_ID,
    messages=[
        {
            "role": "user",
            "content": [
                {"text": "What's the weather in New York City?"}
            ],
        }
    ],
    model={
        "bedrockModelConfig": {
            "modelId": "us.anthropic.claude-sonnet-4-6"
        }
    },
    tools=[
        {
            "type": "remote_mcp",
            "name": "exa",
            "config": {"remoteMcp": {"url": "https://mcp.exa.ai/mcp"}}
        }
    ]
)
```

对于构建 agent 这件事，一套完全托管、无服务器的体验能省下大量时间！而且它的部署体验快得令人愉悦。用我自己的自定义 agent 大约要花 1 分钟，而 harness 部署只要约 20 秒！
