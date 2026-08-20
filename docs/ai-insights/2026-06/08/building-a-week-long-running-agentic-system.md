---
title: 如何构建持续运行一周的智能体系统
author: Fareed Khan
url: https://levelup.gitconnected.com/building-a-week-long-running-agentic-system-2ad79f8190bb
translated: 2026-06-08
excerpt: 持久性是一种工程特性，而非模型性能。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/01.thumb.webp
---

# 构建一个持续运行一周的智能体系统

持久性是一种工程特性，而非模型性能。


大多数AI 智能体都是一个单循环程序，一旦进程重启、上下文窗口填满或某个 API 调用失败，它就会立即终止运行。这对于快速聊天来说尚可接受，但对于需要运行数天甚至数周的长期任务来说就显得力不从心了。在本博客中，我们将构建一个能够持续运行整个任务周期的智能体。它可以自主规划工作，构建并测试实际软件，并从失败中学习，从而在下次运行中不断改进自身。为此，模型本身并不需要变得特别智能，因为其周围的系统已经承担了大部分繁重的工作。

![运行一周的智能体系统（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/01.webp)

这是构建之前的整体架构。人可以通过持久化信号进行审批和引导。持久化控制平面负责调度工作。智能体团队负责执行工作，包括一名首席工程师和几名助手。实际状态存储在 Git 和分层记忆中，只有确定性验证器才能判定某项工作已完成。

1. 我想看看我们能把事情往另一个方向推进到什么程度。
2. 如果我们把长时自主性视为一个工程问题而不是一个模型能力，会怎么样？
3. 该模型以短时脉冲式思维运行，系统通过出色地完成三件枯燥乏味的事情来持续运行数周。它将真实状态置于模型之外，记录每一步操作，并通过实际测试验证进度。

在本博客中，我将围绕一个具体的任务展开，以确保思路清晰。这个任务是从零开始构建一个小型 Python 语言服务器（LSP 服务器），整个系统大约需要一周时间才能完成。下面的跟踪记录是该任务的典型运行示例，读完本博客后，您将能够理解其中的每一行代码。

```
2026-06-02T08:44:51.880Z INFO  mission.complete items=14/14 cycles=615 head=9d8c7b6 status=DONE
2026-06-02T08:44:51.900Z INFO  governor.final cum_usd=178.60 ceiling=400.00 crashes_survived=1
                               loop_trips=1 reviewer_blocking=2 scope_cuts=1 skills_learned=2
```

这项任务在主机重启后依然运行良好，触发了一次循环检测器，被独立审核员否决了两次，由于计划过大而自行缩减了范围，并且在此过程中掌握了两项可复用的技能。这一切仅花费了 178 美元 60 美分 token，而上限为 400 美元。让我们一步一步地构建出能够生成这条生产线的系统吧。

## 代码概述

所有代码都位于一个名为 `lra`（全称 Long Running Agents）的 Python 包中。您可以在 GitHub 上找到完整的项目：

代码库非常庞大，所以我们不会把整个代码树都列出来，而是列出一些关键部分的结构图。每个文件夹代表一项职责，我们将大致按照这个顺序逐一介绍。

```
src/lra/
├── durable/        
├── state/          
├── contracts/      
├── agent/          
├── agents/         
├── coordination/   
├── model/          
├── execution/      
├── safety/         
├── verify/         
├── governor/       
├── hitl/           
├── memory/         
└── evals/          
```

![lra 包的职责组织方式（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/02.webp)

模型层是可插拔的，因此您可以零成本运行整个系统。本地 Ollama 模型或免费云模型都是免费的，而测试则使用“存根”模型。只有当您将其指向 Claude 等付费 API 时，才需要支付令牌费用。以下是快速入门指南：

```

uv sync
uv run lra version
uv run lra config        
uv run lra mission --task "Create a hello.py that prints hello and a test for it" \
                   --workdir .lra/workspaces/demo
```

`lra mission` 命令会将任务规划成一个清单，然后逐项执行，直到所有项目都得到验证。在命令和日志中，你会看到这被称为“任务”（mission），例如 `lra mission`，这只是系统对一次任务运行的称呼。它执行的所有操作都会生成真实的 Git 历史记录，因此你可以使用 `git -C .lra/workspaces/demo log --oneline` 命令来查看后续的工作。现在，让我们来了解一下系统为何如此构建。


## 核心思想：假设会被打断

在编写任何代码之前，我们需要一个核心理念，其他一切都以此为基础。如果一个智能体需要运行一周，那么在这一周内它必然会受到中断。机器会重启，API 会限制访问速率，上下文窗口会溢出，工作进程会被部署操作终止。中断在这里并非特殊情况，而是常态。因此，我们从一开始就考虑到了这一点，我们遵循的原则很简单：假设会中断。

### 上下文窗口是一种有损缓存

最常见的错误是将模型的上下文窗口视为智能体的记忆。事实并非如此。上下文窗口是一个容量很小且有损的缓存。它最多只能存储几十万个标记，当它被填满时，数据会被汇总或截断，并在进程重启时完全消失。如果智能体对“我在此任务中的位置”的理解仅仅存在于上下文窗口中，那么第一次中断就会将其清除。

![上下文窗口是一个小型有损缓存，会在中断时消失（创建者：）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/03.webp)

因此，经过数千次迭代后，上下文窗口将会被填满，而崩溃或重启会将其清除。当这种情况发生时，一个缺乏经验的智能体将完全失去对当前状态的感知。它不知道自己已经写入了哪些文件，哪些测试已经通过，或者已经做出了哪些设计决策。这正是大多数长时间运行的智能体最终崩溃的原因。我们将通过将真实状态转移到一个不会消失的地方来解决这个问题。

### 窗外的房地产世界

解决方法是将任务的真实状态保存在模型之外，放在一个不受任何影响的地方。我们使用一个 Git 仓库，并在其中存放一些结构化文件。模型在每个周期开始时读取这个状态以进行更新，执行少量工作，然后将结果提交回去。由于状态永远不会出现在上下文窗口中，因此上下文窗口就变成了可丢弃的。

![实际状态保存在 Git 中，每个周期都会重新读取并提交。（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/04.webp)

这就是全部思路的概括。实际状态是一个 Git 仓库加上一小部分已保存的状态文件。模型每个周期都会读取最新的状态，因此一个全新的、上下文窗口为空的模型可以在几秒钟内重建完整的感知。然后，它会将结果提交回相同的状态。周期之间的崩溃不会造成任何损失，因为唯一持久的状态是我们已经提交的部分。现在，让我们来构建这个已保存的状态。

## 已保存状态

保存的状态是一组包含所有重要信息的文件。它位于我们正在开发的代码仓库中的 `.lra/` 文件夹中，并与实际的代码更改一起提交到 Git。这样，一次提交就能同时记录工作内容和工作进度。我们先来看数据，因为数据就是设计。

### 清单

已保存状态的核心是一个清单。清单上的每一项都是一个小的工作单元，并且只能处于四种状态之一：`待办`、`进行中`、`阻塞`或`已完成`。一条重要的规则（我们稍后会强制执行）是：只有经过实际测试验证后，清单才能切换到`已完成`状态。以下是用 Pydantic 编写的数据模型。

```python
ItemStatus = Literal["todo", "in_progress", "blocked", "done"]class ChecklistItem(BaseModel):
    """One mission work-unit; flips to `done` only after deterministic verification."""    id: str
    description: str
    status: ItemStatus = "todo"
    
    verified_by: list[str] = Field(default_factory=list)
    depends_on: list[str] = Field(default_factory=list)  
    attempts: int = 0                                     
    notes: str = ""
    schema_version: int = 1                                   @property
    def is_open(self) -> bool:
        return self.status in ("todo", "in_progress", "blocked")
```

每个项目都有自己的依赖项列表和尝试计数器。`verified_by` 字段是一个审计跟踪，它记录了验证项目完成情况的检查名称，以便我们之后能够用具体的答案而不是猜测来回答“我们如何知道它有效？”这个问题。现在，检查清单本身就是这些项目的有序列表，外加一个非常重要的方法。

```python
class Checklist(BaseModel):
    items: list[ChecklistItem] = Field(default_factory=list)
    schema_version: int = 1    def next_actionable(self) -> ChecklistItem | None:
        """Return the first open item whose dependencies are all done, else None."""
        done = {i.id for i in self.items if i.status == "done"}
        for item in self.items:
            if item.is_open and all(dep in done for dep in item.depends_on):
                return item
        return None    @property
    def all_done(self) -> bool:
        return bool(self.items) and all(i.status == "done" for i in self.items)
```

`next_actionable` 方法让智能体始终知道下一步该做什么。它会遍历列表，并返回第一个依赖项全部完成的未完成项。这使得一个没有记忆的新智能体能够直接从上一个智能体停止的地方继续执行。它不需要记住任何内容，只需查询清单即可。

![清单会选择下一个依赖项已完成的未完成项（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/05.webp)

### 决策与事件

清单会告诉你还剩下哪些工作要做，但它不会告诉你之前做出选择的原因，也不会提供详细的操作历史记录。为此，我们在清单旁边维护了两个仅追加记录的日志。一个用于记录决策，另一个用于记录事件。

```python
class DecisionRecord(BaseModel):
    """A durable, never-compacted record of an implicit design decision (anti-drift)."""    decision: str               
    rationale: str
    alternatives_rejected: str = ""
    affected: list[str] = Field(default_factory=list)   
    cycle_id: str = ""class EventRecord(BaseModel):
    """One entry in the append-only episodic event log."""    kind: str                   
    cycle_id: str = ""
    payload: dict[str, object] = Field(default_factory=dict)
    
    payload_ref: str | None = None
```

决策日志的重要性远超表面。当上下文窗口被压缩时，最先被丢弃的是一些细小的隐式决策，例如“我们选择直接读取字节而不是使用`input()`”。这些决策恰恰是导致后续代码漂移的罪魁祸首，因为未来的代码循环会悄然与过去的代码循环相矛盾。通过将这些决策写入一个永不压缩的仅追加文件中，我们可以始终将新的工作与旧的决策进行比对。

![决策和事件是仅追加的日志，永远不会被压缩删除（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/06.webp)

### 阅读现状

现在是关键操作。在每个周期开始时，智能体会根据这些文件以及 Git 日志重建其完整的感知信息。这相当于将“假设中断”规则转化为一个方法。以下是 `GitMissionAnchor` 类的读取路径。

```python
def _read_sync(self) -> SituationSnapshot:
    checklist = self._read_checklist()
    progress = ""
    progress_path = self._path(PROGRESS_FILE)
    if progress_path.exists():
        progress = progress_path.read_text(encoding="utf-8")    
    active = next((i for i in checklist.items if i.status == "in_progress"), None)
    if active is None:
        active = checklist.next_actionable()    return SituationSnapshot(
        head_sha=git_ops.head_sha(self.workdir),
        recent_commits=git_ops.log_oneline(self.workdir, 10),
        progress_summary=progress,
        open_items=[i for i in checklist.items if i.is_open],
        last_decisions=self._read_recent_decisions(5),
        active_item=active,
        is_complete=checklist.all_done,
    )
```

请仔细阅读该方法，因为它是整个系统的核心。它会打开清单文件，读取进度描述，找到当前活动项，拉取最近十次 Git 提交和最近五个决策，并将所有内容打包成一个 `SituationSnapshot`。这个过程只需几毫秒即可完成，而且即使是从未执行过此任务的全新进程也能正常运行。正因如此，即使进程崩溃也几乎不会造成任何影响。下一个工作进程只需读取快照即可继续执行。

![当前状态由 git 重建，状态文件在毫秒内生成（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/07.webp)

### 提交检查点

另一半是写入。当一个周期结束时，我们会写入更新后的文件，并通过一次原子性的 Git 提交将其提交。如果提交成功，则实际状态会更新。如果在提交之前发生任何崩溃，则实际状态不会更新，下一个周期会重试相同的操作。不存在半成品状态。

```python
def _commit_sync(self, checkpoint: Checkpoint) -> str:
    self.anchor.mkdir(parents=True, exist_ok=True)
    self._write_checklist(checkpoint.checklist)              
    if checkpoint.progress_summary:
        self._path(PROGRESS_FILE).write_text(checkpoint.progress_summary, encoding="utf-8")
    if checkpoint.decisions:                                 
        with self._path(DECISIONS_FILE).open("a", encoding="utf-8") as fh:
            for decision in checkpoint.decisions:
                fh.write(decision.model_dump_json() + "\n")
    if checkpoint.events:                                    
        with self._path(EVENTS_FILE).open("a", encoding="utf-8") as fh:
            for event in checkpoint.events:
                fh.write(event.model_dump_json() + "\n")
    message = checkpoint.commit_message or f"lra: checkpoint {checkpoint.cycle_id}"
    return git_ops.commit_all(self.workdir, message)         
```

请注意，决策和事件是追加的，而不是覆盖的，而清单则会被完全重写。整个过程以一次 `git commit` 操作结束。由于提交信息是确定性的且易于理解的，因此 Git 历史记录本身就成为了一份易于阅读的进度报告。以下是一个包​​含三个小任务完成后生成的历史记录示例。

```bash

$ git -C .lra/workspaces/demo log --oneline
9d8c7b6 lra: complete 03 (task 3)
b3c4d5e lra: complete 02 (task 2)
a1f2e3d lra: complete 01 (task 1)
0c1d2e3 lra: initialize mission anchor
```

每个已完成的项目对应一个提交，再加上创建已保存状态的初始提交。你可以直接从 Git 日志中读取整个任务，无需任何特殊工具。这就是将实际状态放在 Git 中的好处。现在我们有了存储进度的地方。接下来我们需要一个循环来实际实现这些进度。

![检查点会写入状态文件，并在一次原子提交中提交它们（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/08.webp)

## 智能体周期

我们已经有了存储实际状态的地方。现在我们需要一个循环来读取状态，执行一些操作，然后将结果写回。我把这个工作单元称为一个周期。周期特意设计得很小。它处理一个待办事项清单项目，运行模型几个步骤，验证结果，然后提交。保持单元小是系统弹性的关键，因为即使系统崩溃，我们最多也只会损失一个未完成的周期。

### 思考、行动、验证、检查点

每个循环都遵循相同的四个步骤。它从 Git 读取数据状态，运行模型及其工具进行操作，通过实际检查验证结果，并将结果检查点保存回 Git。然后重复此过程。

![一个周期：读取状态、使用工具执行操作、验证，然后创建检查点（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/09.webp)

这是循环的开头部分，来自 `AgentLoop` 类。它首先读取快照，如果任务已经完成，则立即返回。这种提前返回使得整个循环具有幂等性。

```
async def run_cycle(self, *, ctx, mission_id, cycle_id, anchor_text, checks=None):
    checks = checks or []
    snapshot = await self._anchor.read_situational_awareness()
    
    if snapshot.is_complete or snapshot.active_item is None:
        return CycleOutcome(
            item_id=None, advanced=False, verified=False,
            is_complete=True, head_sha=snapshot.head_sha, tool_calls=0, turns=0,
        )    item = snapshot.active_item  
    messages = build_messages(
        anchor_text=anchor_text, snapshot=snapshot, item=item,
        specs=self._dispatcher.specs(),   
    )
```

该循环会从快照中选择一个项目（即当前活动项目），并构建一个提示，其中包含当前进度、未完成的项目以及可以调用的工具。请注意，它从不依赖自身记忆来决定处理哪些项目，而是始终查询最新读取的快照。现在，它进入了真正思考和行动的阶段。

### 工具循环

在一个循环中，模型和工具轮流执行任务。模型会发出工具调用信号或“完成”信号。如果是工具调用信号，我们会运行该工具，将结果作为观测值反馈给模型，然后再次询问模型。这个过程会重复进行，直到达到预设的循环次数，因此即使模型出错，也不会在单个循环中无限循环。

![模型和工具轮流运行，直到模型发出完成信号或达到循环次数上限为止。（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/10.webp)

```
tool_calls = 0
for turns in range(1, self._max_turns + 1):
    result = await self._model.complete(messages)     
    if self._ledger is not None:                      
        self._ledger.record(
            cycle_id=cycle_id, usage=result.usage,
            usd=self._model.estimate_cost_usd(result.usage),
        )    action = parse_action(result.text, result.tool_calls)
    if action.done or action.tool is None:            
        messages.append(ModelMessage(role="assistant", content=result.text))
        break    
    call = ToolCall(id=f"{cycle_id}-{turns}", name=action.tool, arguments=action.arguments)
    tool_result = await self._dispatcher.dispatch(call, ctx)
    tool_calls += 1
    observation = (tool_result.content or tool_result.error or "")[:4000]
    messages.append(ModelMessage(role="assistant", content=result.text))
    messages.append(ModelMessage(role="user", content=f"OBSERVATION ({action.tool}): {observation}"))
```

每个模型回合都会将其实际的 token 使用情况记录到成本账本中，我们稍后将使用该账本来控制预算。我们反馈的观测数据上限为四千个字符，因此庞大的工具输出不会破坏上下文。当模型最终发出完成信号时，我们会退出并进入验证阶段。以下是实际 LSP 任务的一个周期示例，其中首席工程师编写传输文件并运行测试。

```

07:30:04 INFO  lead.turn cycle=c1 model=claude-opus-4-8 tok_in=8112 tok_out=905 cache_read=5900 cost=0.0712
07:30:16 INFO  tool.exec cycle=c1 name=write_file path=pylsp_mini/transport.py bytes=812
08:01:59 INFO  tool.exec cycle=c1 name=run_command cmd="uv run pytest tests/test_initialize.py -q" -> exit=1
    tests/_fakeclient.py:55: in _read_response
        header = self._read_until(b"\r\n\r\n")
    E   Failed: Timeout >10.0s
08:02:10 WARN  verify.fail cycle=c1 attempt=1 reason=timeout tests=0/1
```

一次模型迭代，一个文件写入，一次测试运行，然后因超时失败。这是一个完全正常的周期。由于工作未通过，因此该项目不会被标记为已完成。相反，它会被记录为一次失败的尝试，下一个周期将再次尝试。这是系统完全按照设计运行的，我们稍后将追踪此特定错误直至最终解决。

### 上下文压缩

一个循环可以执行多次扫描，每次工具调用和观察都会增加消息列表。如果一个循环探索了很多次，该列表可能会变得非常长，以至于窗口不堪重负。因此，我们对其进行压缩。我们保留系统提示和最近几次扫描的原始信息，并将所有更早的信息汇总到一个简洁的笔记中。

![压缩功能使系统能够及时显示最近的回合数，并汇总较早的回合数。（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/11.webp)

```
async def compact_messages(*, model, messages, keep_last=4):
    """Return system + a summary-of-older-turns + the last `keep_last` turns."""
    system = [m for m in messages if m.role == "system"][:1]    
    body = [m for m in messages if m.role != "system"]
    if len(body) <= keep_last:
        return messages                                             to_summarize = body[:-keep_last]                            
    recent = body[-keep_last:]                                  
    transcript = "\n".join(f"{m.role}: {m.content}" for m in to_summarize)[:12000]
    result = await model.complete([
        ModelMessage(role="system", content=_SUMMARY_INSTRUCTIONS),
        ModelMessage(role="user", content=transcript),
    ])
    summary = ModelMessage(role="user", content=f"[compacted summary]\n{result.text[:4000]}")
    return [*system, summary, *recent]
```

摘要指令非常明确。它告诉模型要保留任务、关键决策及其原因、发现结果（包括失败的方法）以及后续步骤。系统提示信息永远不会被压缩，因为它至关重要。这样既能保证长时间的探索过程在窗口内进行，又不会丢失重要的信息。值得注意的是，这种上下文压缩类似于 Git 中的持久状态，但速度更快、规模更小。Git 保存的状态可以跨周期甚至在崩溃后仍然存在，而压缩只是保持一个长时间周期的整洁。

## 模型层

循环调用了 `self._model.complete(...)`，但它并不关心具体是哪个模型。这种间接性是故意的。整个系统只与一个接口通信，我们只需更改一个配置即可切换其背后的实际模型。这使得同一段代码在测试环境中的本地模型上可以零成本运行，而在生产环境中的 Claude 上则可以正常运行。

### 一个界面，多个后端

该接口是一个小型协议。一个模型接收消息列表，并返回包含实际令牌使用情况的实际结果。就是这样。

![一个支持可互换的 Stub、Ollama、OpenAI 和 Claude 后端的 ModelProvider 接口（创建者：）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/12.webp)

```python
@runtime_checkable
class ModelProvider(Protocol):
    """A pluggable LLM backend. Implementations live in src/lra/model/."""    name: str    async def complete(self, messages, *, tools=None, max_tokens=None) -> "TurnResult":
        """Run one real model turn and return its real output + usage."""
        ...    def estimate_cost_usd(self, usage: "Usage") -> float:
        """Compute real USD cost from real token counts (0.0 for local/free backends)."""
        ...class Usage(BaseModel):
    """Real token accounting as reported by the provider."""    input_tokens: int = 0
    output_tokens: int = 0
    cache_read_input_tokens: int = 0
    cache_creation_input_tokens: int = 0
    model: str = ""
```

此接口共有四种实现方式。`stub` 模型是确定性的且免费的，用于测试，以便在无需网络或密钥的情况下运行整个系统。`ollama` 和 `openai_compat` 后端与本地或免费云层模型通信。`claude` 后端与 Anthropic 的 Messages API 通信。它们都返回相同的 `TurnResult` 和真实的 `Usage`，因此系统的其他部分不会感知到任何差异。Claude 后端甚至会单独报告提示缓存令牌，这对于长期成本控制至关重要。

```
_PRICES: dict[str, tuple[float, float]] = {
    "claude-opus-4-8": (15.0, 75.0),     
    "claude-sonnet-4-6": (3.0, 15.0),
    "claude-haiku-4-5-20251001": (1.0, 5.0),
}
```

### 真实 token，真实成本

每个模型回合都会返回一个真实的 token 数量，我们将其转换为真实的美元金额并添加到账本中。关键在于这个数字是真实的。我们从不估算或伪造成本。成本数据直接来自服务提供商的响应。账本非常小。

![每回合都会将实际的 token 使用情况记录到管理者读取的成本账簿中（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/13.webp)

```python
class CostLedger(BaseModel):
    entries: list[CostEntry] = Field(default_factory=list)    def record(self, *, cycle_id: str, usage: Usage, usd: float) -> CostEntry:
        entry = CostEntry(
            cycle_id=cycle_id, model=usage.model,
            input_tokens=usage.input_tokens, output_tokens=usage.output_tokens, usd=usd,
        )
        self.entries.append(entry)
        return entry    @property
    def total_usd(self) -> float:
        return sum(e.usd for e in self.entries)    def mean_usd_per_cycle(self) -> float:
        cycles = {e.cycle_id for e in self.entries}
        return self.total_usd / len(cycles) if cycles else 0.0
```

账本会记录累计总成本和每个周期的平均成本。正是这个平均值让预算管理程序能够在下一步运行之前预测其成本。本地和免费后端会记录一个真正的零值，同时仍然跟踪 token，因此即使在免费运行中，成本机制也会被执行。以下是跟踪中几个模型回合的示例，其中缓存读取执行了实际工作。

```

07:30:45 INFO  planner.turn cycle=replan-1 model=claude-opus-4-8 tok_in=7740 tok_out=1410 cache_read=6100 cost=0.0731
07:31:40 INFO  lead.turn   cycle=c201    model=claude-opus-4-8 tok_in=9340 tok_out=1190 cache_read=8100 cost=0.0841
11:24:40 INFO  governor.tick cum_usd=103.90 tests=9/14
```

看看这次领先回合的 `cache_read=8100`。大部分输入都来自提示缓存，这比获取新的输入令牌要便宜得多。在一周的循环中，由于会重复读取大量相同的上下文，缓存决定了运行成本的高低。`governor.tick` 行显示了正在跟踪的总成本，目前为 103 美元 90 美分，14 个项目中有 9 个已完成。

### 故障转移

一周的时间足够长，足以让一个服务提供商出现故障。因此，我们可以将多个服务提供商封装在一个 `FailoverModel` 中。它会按顺序尝试这些服务提供商，并返回第一个成功的服务提供商。只有当所有服务提供商都失败时，才会抛出异常。

![故障转移按顺序尝试提供程序，并返回第一个成功提供程序（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/14.webp)

```python
class FailoverModel(ModelProvider):
    """Try providers in order; return the first success, raise only if all fail."""    def __init__(self, providers: list[ModelProvider]) -> None:
        if not providers:
            raise ValueError("FailoverModel needs at least one provider")
        self._providers = providers
        self.name = "failover:" + ",".join(p.name for p in providers)
```

这段代码虽然很小，但却能确保即使某个 API 出现短暂故障，也不会导致持续一周的任务终止。任务会继续使用备用模型运行，而且由于模型层只是一个接口，系统中的其他组件无需感知到故障的发生。此外，还有一种运行模式：如果所有模型都彻底宕机，任务会进入持久睡眠状态，并通知人工处理，而不是进行多次重试。我们很快就会看到这种持久睡眠模式。

## 工具和沙盒

模型无法直接接触外部世界。它只能请求运行某个工具，而每个这样的请求都必须经过一道安全门。所有安全机制都存在于这道安全门中，它位于模型底层的代码中，而不是隐藏在提示信息中，因此不会被巧妙的输入绕过。

### 允许列表调度程序

`AllowListDispatcher` 是模型请求到实际操作的唯一路径。它会在执行任何操作之前检查四件事：工具是否已知、工具是否在允许列表中、如果该工具会修改模型，是否允许修改模型，以及如果该工具会访问网络，是否允许出口访问。默认情况下，出口访问被拒绝。

![每个工具调用都必须先经过允许列表调度程序才能运行（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/15.webp)

```
async def dispatch(self, call: ToolCall, ctx: ToolContext) -> ToolResult:
    tool = self._tools.get(call.name)
    if tool is None:
        return ToolResult.failure(f"unknown tool: {call.name!r}")
    if not self._permitted(call.name):
        return ToolResult.failure(f"tool not allowed: {call.name!r}")
    if tool.spec.mutating and not self._allow_mutating:
        return ToolResult.failure(f"mutating tools are disabled: {call.name!r}")
    if tool.spec.egress and not self._allow_egress:
        return ToolResult.failure(f"egress is disabled (default-deny): {call.name!r}")    missing = _missing_required(tool.spec.parameters, call.arguments)
    if missing:
        return ToolResult.failure(f"missing required args for {call.name!r}: {sorted(missing)}")    try:
        return await tool.run(call.arguments, ctx)
    except Exception as exc:  
        return ToolResult.failure(f"{type(exc).__name__}: {exc}")
```

有两点需要指出。首先，这个门禁是在模型下方的代码中强制执行的。提示注入无法给自己授予工具，因为模型根本不知道存在被禁止的工具。其次，最后的 `try/except` 语句意味着抛出异常的工具不会导致循环崩溃。它会返回一个失败结果，该结果会成为观察值，模型可以对此做出反应。从工具端来看，这个循环是不可中断的。不同的角色会分配不同的调度器，因此只读研究员绝对不可能获得任何具有变异性的工具。

### 沙盒

执行命令的工具并非运行在主机上，而是运行在沙箱中。沙箱本质上是另一个接口，它有三种不同的实现方式：用于开发的本地子进程沙箱、用于持续集成的 Docker 沙箱以及用于生产的 E2B 微型虚拟机沙箱。智能体循环与这些接口通信，但始终无法得知底层运行的是哪个接口。

![一个包含本地、Docker 和 E2B 实现以及快照的沙箱接口（由……创建）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/16.webp)

```python
@runtime_checkable
class Sandbox(Protocol):
    name: str    async def open(self, *, workdir: str, snapshot_id: str | None = None) -> "SandboxSession":
        """Open (or restore from snapshot_id) a session rooted at workdir."""
        ...    async def snapshot(self, session: "SandboxSession") -> "Snapshot":
        """Persist the session's state and return a restorable handle."""
        ...
```

这里有趣的方法是“快照”。沙箱会话可以被冻结，之后可以通过句柄恢复。这对于长达一周的运行至关重要，因为这意味着工作环境本身（而不仅仅是 Git 状态）可以在崩溃或长时间休眠后暂停和恢复。我们之前在主机重启时在跟踪日志中看到了 `sandbox.restore` 这行。通过对工具进行门控和沙箱化，我们可以让模型执行操作。但是执行操作并不等同于成功。为此，我们需要一个门控来决定什么才算完成。

## 确定性验证

这是博客中最重要的一部分，所以我打算放慢速度。成千上万步下来，小错误会不断累积。如果每一步的可靠性都达到 99%，那么几百步之后，你的模型几乎肯定就出问题了。阻止这种累积效应的唯一方法就是永远不要相信模型自身关于某个方法有效的声明。我们应该相信真正的测试结果。

### 退出代码决定一切

`DeterministicVerifier` 将每个检查作为实际子进程运行，并查看其退出代码。只有当所有门控检查的退出代码均为零时，才会执行某个项目。这并非取决于模型的指示，而是取决于退出代码的指示。

![只有实际的退出代码（而非模型的判断）才能将项目状态更改为“已完成”（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/17.webp)

```python
class DeterministicVerifier:
    async def verify(self, workdir: str, checks: list[Check]) -> VerificationResult:
        results: list[CheckResult] = []
        for check in checks:
            outcome = await run_proc(check.command, cwd=workdir, timeout_s=self._timeout)
            results.append(
                CheckResult(name=check.name, passed=outcome.ok, exit_code=outcome.exit_code)
            )
        
        all_green = all(
            result.passed
            for check, result in zip(checks, results, strict=True)
            if check.gating
        )
        return VerificationResult(all_green=all_green, results=results)
```

Python 项目的默认门控是三个实际命令，你已经在跟踪中看到了它们。

```python
def default_python_checks() -> list[Check]:
    return [
        Check(name="ruff", command=["uv", "run", "ruff", "check", "."]),
        Check(name="mypy", command=["uv", "run", "mypy"]),
        Check(name="pytest", command=["uv", "run", "pytest", "-q"]),
    ]
```

回到智能体循环中，这个验证器的结果是决定项目命运的唯一因素。如果 `all_green` 为真，则项目状态变为 `done`，并且其 `verified_by` 字段会填充检查名称。否则，项目状态变为 `blocked`，并且尝试计数器会增加。在这里，我们一直在追踪的传输错误终于通过了。

```

09:10:21 INFO  tool.exec cycle=c201 name=run_command cmd="uv run pytest tests/test_framing.py -q" -> exit=0
    collected 7 items
    tests/test_framing.py .......                                            [100%]
    7 passed in 0.13s
09:10:21 INFO  verify.ok cycle=c201 checks=[pytest]   <- FIRST GREEN (cycle 201, ~2.5 days in)
09:11:02 INFO  git.commit head=a1f2e3d msg="lra: transport framing (isolated, byte-tested)"
```

退出代码为零，七项测试通过，验证器这才显示“verify.ok”。这是整个任务的第一个绿色标记，它出现在第 201 个周期，大约两天半之后。紧随其后的提交是第一个真正可靠的进展。在此之前的所有尝试都只是尝试。

### 信任大门

这里有个不易察觉的陷阱。只有当测试真正覆盖了所有代码时，绿色才代表正确。一个没有测试的代码库虽然显示绿色，但也毫无价值。因此，在我们过分依赖测试之前，尤其是在允许并行写入之前，我们需要评估测试的可靠性。我们使用两个指标：代码覆盖率和变异分数。

![覆盖率和变异分数决定了对验证门的信任程度（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/18.webp)

```python
class TrustBootstrap:
    def __init__(self, *, coverage_threshold: float = 0.6, timeout_s: int = 900) -> None:
        self._threshold = coverage_threshold    async def measure(self, workdir: str, *, command=None) -> "VerifierTrust":
        cmd = command or ["uv", "run", "pytest", "--cov", "--cov-report=term-missing", "-q"]
        result = await run_proc(cmd, cwd=workdir, timeout_s=self._timeout)
        combined = f"{result.stdout}\n{result.stderr}"
        pct = parse_coverage_pct(combined)
        return VerifierTrust(coverage_pct=pct, trusted=pct >= self._threshold, raw_tail=combined[-2000:])
```

代码覆盖率告诉我们哪些代码行执行了。变异测试更进一步，它会故意在代码中引入错误，并检查测试是否能捕获到这些错误。较高的变异分数是门控机制能够检测到回归的最有力证据。以下是 LSP 任务后期的一次变异测试结果。

```

09:36:02 INFO  run_command cmd="uv run pytest --cov -q"  -> exit=0  "coverage: 91%"
09:48:55 INFO  run_command cmd="uv run mutmut run"       -> exit=0  "killed 89/106 (score 0.84)"
09:49:00 INFO  gate.trusted coverage=0.91 mutation=0.84
```

覆盖率达到 91%，测试杀死了 106 个突变体中的 89 个，得分为 0.84。此时，系统会记录“gate.trusted”，这意味着绿色对勾现在具有实际意义。这便是获得信任该网关的依据。

### 不稳定的隔离

最后一个验证问题。不稳定的测试（即随机通过和失败的测试）会干扰流程。如果不稳定的检查会阻碍流程，那么绿色状态就会变得不可靠，智能体会浪费资源去追逐虚假结果。因此，我们会隔离已知的不稳定检查。为了保证可见性，它们仍然会运行，但会被强制设置为非门控状态。

![为了确保可见性，不稳定的检查仍然会运行，但会被强制设为非门控模式（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/19.webp)

```python
class FlakyQuarantine:
    def __init__(self) -> None:
        self._flaky: set[str] = set()    def mark_flaky(self, name: str) -> None:
        self._flaky.add(name)    def partition(self, checks: list[Check]) -> tuple[list[Check], list[Check]]:
        """Split into (gating, quarantined); quarantined checks are forced non-gating."""
        gating, quarantined = [], []
        for check in checks:
            if check.name in self._flaky:
                quarantined.append(check.model_copy(update={"gating": False}))
            else:
                gating.append(check)
        return gating, quarantined
```

隔离的检查项会被复制，并设置 `gating=False`，这样它就不会阻塞任何项目，但它仍然会运行，以便我们跟踪其随时间推移的不稳定率。这保证了“绿色”的含义清晰明确。现在我们有了一个完整的内部引擎。它读取真实值，通过沙箱中的门控工具执行操作，并且只有在可信的测试通过时才会标记进度。下一个问题是如何让这个引擎运行一周。

## 坚固的脊梁

内部循环本身运行良好，但它仍然只是一个 Python 进程。如果该进程崩溃，循环也会随之终止。为了保证程序运行一周，我们将循环封装在一个持久执行引擎中。我们使用 Temporal。Temporal 的核心思想是将控制流拆分为两种代码：一种是工作流代码，它必须是确定性的，并且会记录日志以便重放；另一种是活动代码，它负责所有繁琐的非确定性工作，其结果会被记录一次，然后在重放时从缓存中获取。正是这种拆分方式让我们能够免费实现崩溃恢复。

### 确定性调度器

`MissionWorkflow` 是调度器，它特意设计得非常小巧简洁。它不包含任何模型调用、文件 I/O 操作，也不涉及任何随机性。它所做的只是每次调度一个周期活动，并统计已运行的活动数量。

![MissionWorkflow 是一个确定性调度器，每个周期调度一个活动（创建者：）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/20.webp)

```python
@workflow.defn
class MissionWorkflow:
    @workflow.run
    async def run(self, inp: MissionInput, state: MissionState | None = None) -> MissionResult:
        state = state or MissionState()
        self._cycles_done = state.cycles_done        while True:
            if state.cycles_done >= inp.max_cycles:
                return await self._terminal(inp, state, completed=False)            
            result = await workflow.execute_activity(
                run_agent_cycle,
                CycleInput(
                    mission_id=inp.mission_id, workdir=inp.workdir,
                    cycle_id=f"c{state.cycles_done + 1}", check_commands=inp.check_commands,
                ),
                start_to_close_timeout=timedelta(minutes=10),
                retry_policy=_CYCLE_RETRY,           
            )
            state.cycles_done += 1
            self._cycles_done = state.cycles_done            if result.is_complete:
                return MissionResult(mission_id=inp.mission_id, completed=True,
                                     cycles=state.cycles_done, head_sha=result.head_sha,
                                     items_done=result.items_done, items_total=result.items_total)            if state.cycles_done % inp.cycles_before_can == 0:
                workflow.continue_as_new(args=[inp, state])    
```

这就是整个调度器。仔细阅读，注意其中缺少什么。它没有业务逻辑。工作流只决定何时运行下一个周期以及何时停止。由于工作流是确定性的，Temporal 可以随时重放其历史记录并达到完全相同的状态。重试策略意味着，如果某个周期活动失败，Temporal 会使用退避机制重试最多十次，然后才会放弃。真正的智能都蕴藏在这个活动中。

### 记录活动和回放

该活动负责运行所有非确定性操作。它运行着我们之前构建的真正的 `AgentLoop`。其关键特性是幂等性。它始终从 Git 读取最新状态并推进下一个可执行项，因此运行两次是安全的。

![崩溃会触发重放；已完成的活动来自缓存，因此不会重复工作（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/21.webp)

```
async def _execute_cycle(inp: CycleInput) -> CycleResult:
    """Advance the mission by one verified item via the real agent loop; idempotent."""
    anchor = GitMissionAnchor(inp.workdir)
    snapshot = await anchor.read_situational_awareness()       if snapshot.is_complete or snapshot.active_item is None:
        
        checklist = await anchor.read_checklist()
        done = sum(1 for i in checklist.items if i.status == "done")
        return CycleResult(item_id=None, advanced=False, head_sha=snapshot.head_sha,
                           is_complete=True, items_done=done, items_total=len(checklist.items),
                           note="nothing actionable")    session = await LocalSandbox().open(workdir=inp.workdir)
    loop = AgentLoop(model=build_provider(), dispatcher=AllowListDispatcher(default_local_tools()),
                     verifier=DeterministicVerifier(), anchor=anchor)
    
```

现在观察一下主机在运行过程中重启时会发生什么。这是第三天的 LSP 跟踪记录，当时主机操作系统更新导致机器在运行过程中重启。

```

13:48:12 ERROR worker.disconnect reason="connection reset (host OS update reboot)" inflight_cycle=c190
13:55:40 WARN  temporal.replay wf=mission:mission_4c7e21a9 events=631 (completed activities served from cache; 0 tokens re-spent; 0 double commits)
13:55:41 INFO  reconcile.in_flight ticket=01 branch=feat/transport -> adopt (branch present, unmerged)
13:55:42 DEBUG orchestrator.note "replay restored state EXACTLY -> a still-RED transport. durability protects WORK, not CORRECTNESS."
```

这就是持久性保证的体现。工作进程终止，一个新的工作进程启动，Temporal 重放了 631 个事件。所有已完成的活动都从缓存中恢复，因此没有重复使用任何令牌，也没有重复提交。任务从它之前的周期恢复执行。我喜欢跟踪信息中的最后一行。

持久性保护的是工作成果，而非正确性。重放操作恢复了测试仍然失败的传输过程，因为那才是真实状态。系统不会假装故障已修复，它只是拒绝丢失已完成的工作。

### 持久睡眠，如新

还有两个技巧能让这长达一周的运行真正有效。第一个是持久睡眠。当没有任务需要处理时（例如夜间），工作流会根据一个持久计时器进入睡眠状态。这不会消耗任何资源，而且重启后仍然有效。第二个是“以新状态继续执行”。否则，长达一周的运行会积累庞大的事件历史记录，因此每隔一定周期，工作流就会滚动执行一次，每次只保留一个精简的、很小的状态。

![持久睡眠无需任何成本，而“继续为新”功能则能确保事件历史记录的完整性（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/22.webp)

```

if state.cycles_done % inp.cycles_before_can == 0:
    
    
    workflow.continue_as_new(args=[inp, state])
```

被继承的 `MissionState` 只是一个计数器，即 `cycles_done`。其他所有数据都存储在 Git 仓库中。因此，数据滚动更新的成本很低，新的执行从一个干净、简洁的历史记录开始。以下是跟踪记录中一天结束时的正常状态。

```

18:32:04 INFO  sleep.durable until=2026-05-27T07:30:00Z reason=idle-eod
18:32:04 INFO  consolidate episodes=12 facts=5 ; governor.tick cum_usd=0.094
...
07:30:00 INFO  wake active=01 reground=git+anchor dur=0.041s
```

该任务从晚上一直休眠到第二天早上，完全不消耗任何资源，然后唤醒并从 Git 仓库重新获取自身状态，耗时 41 毫秒。这个“重新获取状态”的步骤正是我们之前提到的“读取情境感知”操作。智能体进入休眠状态，世界运转，醒来后它仍然清楚地知道自己身处何处。这就是一周大部分时间都在睡眠中度过时的实际时间。

### 索赔检查编解码器

一周内，工作流会记录数千个步骤，其中一些有效载荷很大，例如大型模型响应或冗长的测试日志。如果我们直接记录这些原始数据，Temporal 的历史记录将会膨胀。因此，我们采用了声明检查模式。任何超过阈值的有效载荷都会存储在对象存储中，只有简短的内容寻址键会被写入日志。

![大型有效载荷会进入对象存储，并且只会记录一个短键值（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/23.webp)

```python
class ClaimCheckCodec(PayloadCodec):
    async def encode(self, payloads):
        encoded = []
        for payload in payloads:
            if len(payload.data) > self._threshold:        
                key = await self._store.put(payload.data)   
                encoded.append(Payload(
                    metadata={"encoding": _CLAIMCHECK_ENCODING,
                              "lra-orig-encoding": payload.metadata.get("encoding", b"")},
                    data=key.encode("utf-8"),               
                ))
            else:
                encoded.append(payload)
        return encoded
```

对象存储键是内容的 SHA256 哈希值，因此相同的有效负载会自动存储一次。一个 10 兆字节的响应在历史记录中会变成一个 64 个字符的哈希值。您可以在跟踪的第一行看到这一点，其中数据转换器配置了一个声明检查阈值。

```

09:00:01 INFO  temporal.connect addr=localhost:7233 ns=default build_id=lra-7f1c
               data_converter=ClaimCheckCodec(threshold=32768)
```

编解码器集成在数据转换器层，因此活动代码根本察觉不到它的存在。它只是简单地传递对象，而较大的对象则会被透明地卸载。这使得日志文件足够小，即使经过一周的工作也能快速重放。

### 传奇与补偿

最后一个持久性机制是针对那些涉及 Git 仓库外部的操作，例如创建分支、发起拉取请求或合并。这些操作都是多步骤的，如果中途失败，就会造成混乱。因此，每个执行步骤在运行之前都会注册一个撤销操作，如果失败，我们会按相反的顺序执行撤销操作。

![每向前一步都会记录一次撤销操作；如果失败，则按最新操作优先的顺序运行（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/24.webp)

```python
class Saga:
    def add(self, name: str, undo: Callable[[], Awaitable[None]]) -> None:
        self._compensations.append(Compensation(name=name, undo=undo))    async def compensate(self) -> list[str]:
        """Run compensations newest-first (LIFO); record any undo failures, never stop."""
        ran = []
        for comp in reversed(self._compensations):
            try:
                await comp.undo()
                ran.append(comp.name)
            except Exception as exc:
                self.failures.append(f"{comp.name}: {type(exc).__name__}: {exc}")
        return ran
```

因此，如果我们创建一个分支并提交一个拉取请求，然后合并失败，那么流程会按顺序关闭拉取请求并删除分支。需要注意的是，像生产环境部署这样真正不可逆的操作不会被模拟回滚，而是会转交给人工处理，我们稍后会详细介绍。有了持久化的基础架构，我们的单个智能体现在可以运行数周。下一个问题是，单个智能体是否足够。

## 实力不济的球队

到目前为止，所有工作都由一个智能体完成。显而易见的下一步想法是投入大量智能体来解决问题。但我对此持反对意见，因为智能体越多并不一定越好。智能体越多，成本也就越高，我们应该只在它真正带来好处时才付出代价。

### 为什么多智能体是一种成本

当多个开发者同时编写代码时，他们会做出相互冲突的假设，互相干扰彼此的文件，导致设计失去连贯性。并行开发有利于真正独立的工作，例如阅读代码库的不同部分或以全新的视角审查已完成的代码。但它不利于耦合性强的开发，因为在这种开发模式下，每个决策都依赖于前一个决策。因此，团队成员的构成往往存在不均衡的情况。

![一位负责人撰写；研究人员和审稿人仅独立开展工作（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/25.webp)

其结构如下：一位主控工程师负责所有耦合代码的编写，从而确保设计决策的一致性。临时辅助人员仅负责真正可以并行化的两项工作。研究人员并行阅读代码，审阅者则在全新的上下文中检查代码。只有独立的集成人员才能向主分支写入代码。这不是一个集群式开发团队，而是一个精干的小团队，其中成本最高的部分——代码的一致性——保持串行执行。

### 角色和模型层级

每个角色都由一个简短的规范来描述，其中说明了它使用哪个模型层级以及它被允许执行哪些操作。这就是成本控制的关键所在。我们只在需要高杠杆决策时才使用昂贵的 Opus 模型，而对于批量搜索，则使用低成本的模型。

![角色与模型层级相对应，因此成本与判断的价值成正比（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/26.webp)

```python
class RoleSpec(BaseModel):
    name: str
    tier: ModelTier             
    system_prompt: str
    allow_mutating: bool = False
    allow_egress: bool = False
    max_turns: int = 8
"lead": RoleSpec(
    name="lead",
    tier=ModelTier.OPUS,
    system_prompt=(
        "You are the Lead Engineer and the SOLE writer to the integration line. Keep design "
        "decisions coherent. Mark an item done only when the deterministic checks are green."
    ),
    allow_mutating=True,
),
```

以下是层级与角色的对应关系。高判断力角色使用最强大的模型，而低成本的并行工作则使用较小的模型。

- 策划者、负责人和审阅者都使用 Opus，因为他们的判断决定一切。
- 集成商和测试商使用 Sonnet 进行中等规模的工作。
研究人员使用俳句，因为阅读和总结成本低廉，而且可以重复多次。

这样一来，整个团队的成本就与决策的价值成正比，而不是与计算量成正比。运行五十次的科研人员几乎不需要任何成本，而做出连贯设计选择的负责人则能获得真正值得付费的模型。

### 规划师

任务从规划器开始。规划器接收任务描述，并将其转化为一系列有序、可独立验证的小步骤清单。它向模型请求一个 JSON 数组，然后对其进行防御性解析，因为模型并非总是返回干净的 JSON 数据。

![规划器将任务分解成一个有序的、可验证的清单（创建者：）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/27.webp)

```python
class Planner:
    async def plan(self, *, title: str, description: str, acceptance: str = "") -> Checklist:
        messages = [
            ModelMessage(role="system", content=ROLES["planner"].system_prompt),
            ModelMessage(role="user", content=(
                f"Mission: {title}\n\n{description}\n\n"
                f"Definition of done: {acceptance or '(use your judgment)'}\n\n"
                f"{_PLANNER_INSTRUCTIONS}")),
        ]
        result = await self._model.complete(messages)
        items = parse_checklist(result.text)
        if not items:
            
            items = [ChecklistItem(id="01", description=description.strip() or title.strip())]
        return Checklist(items=items)
```

最后一步的回退机制至关重要。如果模型返回的是无意义的结果，程序不会崩溃。我们会根据描述生成一个包含一个项目的清单，这样任务总能启动。以下是规划器处理 LSP 任务的运行情况，我想请你注意输出结果中一些不寻常的地方。

```

09:47:55 INFO  checklist.write items=31 sha256=4b1c... (01 transport, 02 init, 03 hover,
               04 completion, 05 definition, 06 diagnostics, 07 formatting, 08 documentSymbol, 09 rename, ...)
09:47:55 INFO  ownership.map lead=[transport,dispatch,index] implementers=[per-feature]
```

规划器生成了 31 个项目。结果发现数量太多了。在核心服务器还没运行之前就规划整个服务器，这注定会造成数天的浪费，我们稍后会看到后果。规划器功能强大，但也可能过度规划，系统需要其他机制来纠正这种情况。这本身就是一个很好的教训。计划只是一种假设，而不是一份合同。

### 单文件所有权

当我们允许并行实现时，我们通过一个简单的不变式来避免冲突。每个文件都只有一个写入者。规划器会将文件分配给写入者，而某些共享文件（例如构建清单和 `__init__.py`）只能属于主导者。

![每个文件只有一个写入者；共享文件始终属于主作者（创建者）。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/28.webp)

```python
class FileOwnershipMap(BaseModel):
    owners: dict[str, str] = Field(default_factory=dict)       def permits(self, *, writer: str, path: str) -> bool:
        """Whether `writer` may write `path` under the single-writer-per-file invariant."""
        norm = _normalize(path)
        if is_shared(norm):
            return writer == LEAD               
        owner = self.owners.get(norm)
        if owner is None:
            return writer == LEAD               
        return writer == owner
```

规则很简单：如果文件是共享的，只有发起者才能写入；如果文件是私有的，只有所有者才能写入；任何未分配的文件默认由发起者写入。此规则在 Git 层进行检查，因此任何意外的写入操作都会被明确告知失败，而不是悄无声息地破坏其他实现者的切片。如果写入者发现自己需要使用一个不属于自己的文件，则需要提交租约请求，而不是直接写入。这就是并行写入（在极少数情况下允许并行写入）如何保证安全性的原理。

### 研究者扇出

阅读是一项可以完美并行开展的工作，因为它没有任何副作用。当负责人需要了解背景信息时，组织者会同时安排几位研究人员，每人负责一个不同的问题，然后收集他们的简报。

![研究人员并行开展工作，并将只读简报返回给项目负责人（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/29.webp)

```
async def research_fanout(*, model, dispatcher, ctx, queries, role=None) -> list[SubAgentResult]:
    """Investigate `queries` in parallel; return one brief per query (failures filtered out)."""
    role = role or ROLES["researcher"]
    agents = [SubAgent(role=role, model=model, dispatcher=dispatcher) for _ in queries]
    results = await asyncio.gather(
        *(agent.run(objective=q, ctx=ctx) for agent, q in zip(agents, queries, strict=True)),
        return_exceptions=True,
    )
    return [r for r in results if isinstance(r, SubAgentResult)]   
```

研究人员是只读的，因此他们使用的调度器禁用了变异功能。他们并发运行，任何失败的进程都会从结果中剔除。在 LSP 任务的第一天，主导者不知道协议的线路格式，因此它的第一步是进行研究，而不是猜测。

```

09:00:08 INFO  research.fanout n=4 task_queue=lra-research rule_of_two=ok caps=2
09:00:20 INFO  research.brief id=1 "FRAME ON BYTES. Content-Length = len(utf8(body)). headers ASCII + CRLF;
               body=utf-8 JSON-RPC 2.0. POSITIONS: 'character' is UTF-16 CODE UNITS (HAZARD-1)."
09:00:36 INFO  research.brief id=2 "pygls: read header lines off sys.stdin.BUFFER until blank; read(n) EXACTLY;
               write to sys.stdout.BUFFER. NEVER print()/input(). pylsp setmode(O_BINARY) on Windows."
```

四位研究人员并行开展研究，并提交了简洁明了的简报。第一份简报甚至指出了一个几天后才会显现的风险：UTF-16 列编码。这就是低成本并行读取的优势。只需花费几分钱的 Haiku token，研究人员就能在掌握问题轮廓的基础上，着手开展真正的研究工作。这项研究没有编写任何代码，只是收集了事实。

### Fresh-Context 评论员

第二个并行化的环节是代码审查。单一主体的弊端在于作者偏见。编写代码的人往往是最不了解代码的人。因此，审查者是一个独立的主体，拥有全新的视角。它只能看到代码差异和验收标准，而看不到主导者的思路或之前的尝试。

![评论者以全新的视角看待差异，因此能够发现作者遗漏之处（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/30.webp)

```python
class Reviewer:
    async def review(self, *, diff: str, criteria: str, ctx: ToolContext) -> ReviewResult:
        result = await self._agent.run(
            objective=("Review the diff against the acceptance criteria. Identify correctness, "
                       "security, and scope issues. Prefix any blocking issue with 'BLOCK:'."),
            ctx=ctx,
            extra_context=f"Acceptance criteria: {criteria}\n\nDIFF:\n{diff[:8000]}",
        )
        blocking = "block:" in result.brief.lower() or "blocking" in result.brief.lower()
        return ReviewResult(brief=result.brief, blocking=blocking, tool_calls=result.tool_calls)
```

关键在于提供全新的背景信息。因为审稿人从未亲眼目睹项目进展的艰辛，所以他们对所选方法并不认同。当 LSP 任务在运输环节卡住数日时，正是审稿人最终指出了问题的症结所在。

```

10:15:40 INFO  reviewer.finding sev=blocking kind=architecture
    "you are solving THREE problems in one file and they keep colliding: (1) framing,
     (2) text-vs-binary streams, (3) concurrency. SEPARATE them. Make framing a pure byte-level
     pair read_message/write_message and UNIT-TEST it in isolation. Use sys.std*.buffer only.
     A minimal server needs NO concurrency: a synchronous read->dispatch->write loop."
10:15:40 INFO  reviewer.finding sev=advisory kind=scope
    "a broken core + 31 planned items is the real problem; cut scope until the core is green."
```

那一次评审就让整个项目彻底失败了。项目负责人之前一直焦头烂额，因为他把三个问题混杂在一起。评审员与之前十二次失败的尝试都没有任何瓜葛，所以他一眼就看出了问题所在，并建议将它们分开，缩小范围。这种阻碍性的意见会暂停工作，迫使人们重新思考。这是整个项目过程中最清晰的例子，说明了为什么聘请独立评审员是值得的。

### 决策日志

我们之前已经接触过决策记录这种数据类型。下面介绍一下它在实践中的应用。所有重要的设计决策都会被添加到一个永不压缩的日志中，以便后续工作可以对照检查。

![设计决策会附加到从未压缩的日志中，以便捕获矛盾（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/31.webp)

```python
class DecisionLog:
    """A JSONL file of DecisionRecords. Append-only; never rewritten."""    def append(self, record: DecisionRecord) -> None:
        self._path.parent.mkdir(parents=True, exist_ok=True)
        with self._path.open("a", encoding="utf-8") as fh:
            fh.write(record.model_dump_json() + "\n")    def read(self) -> list[DecisionRecord]:
        if not self._path.exists():
            return []
        lines = [ln for ln in self._path.read_text(encoding="utf-8").splitlines() if ln.strip()]
        return [DecisionRecord.model_validate_json(ln) for ln in lines]
```

在审稿人介入后，LSP 任务的决策清晰地展现了方法是如何改变的。

```

07:31:02 INFO  decisions.append id=3 "narrow->14; transport-first TDD; SYNC loop; binary streams (sys.*.buffer)"
```

那一行记录了转折点。任务从三十一项缩减到十四项，首先确定了传输层的测试驱动开发，并选择了使用二进制流的同步循环。由于这些内容被记录下来且从未被压缩，因此未来的任何迭代周期都不会悄然滑回混乱的异步方法。决策日志是团队对“为什么”的长期记忆，而不仅仅是“做了什么”。

## 既要保证安全又要控制预算

一个能够运行一周、编写代码、执行命令并连接到网络的智能体功能强大，而强大的事物需要防护措施。本节将介绍防止长时间运行的智能体悄无声息地烧钱、无限循环或做出危险行为的防护措施。这些防护措施并非存在于提示符中，而是存在于模型无法理解的代码中。

### 预算州长

第一道防线是资金。如果预计支出会超出预算上限，州长会在下一轮预算周期开始前就拒绝启动。他不会等到预算超支才采取行动，而是根据运行平均值预测下一步的成本，并提前停止预算。

![如果下一轮选举周期会突破预算上限，州长会在选举开始前拒绝启动该周期。（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/32.webp)

```python
class BudgetGovernor:
    """Refuses the next step BEFORE it runs if it would breach the spend/iteration ceiling."""    def authorize_next(self, ledger, *, cycles_done, projected_usd=None) -> GovernorDecision:
        spent = ledger.total_usd
        
        estimate = projected_usd if projected_usd is not None else ledger.mean_usd_per_cycle()
        projected_total = spent + estimate        if cycles_done >= self._max_cycles:
            return self._deny("max cycles reached", projected_total, spent)
        if projected_total > self._ceiling:
            return self._deny("projected spend exceeds ceiling", projected_total, spent)
        return GovernorDecision(allow=True, reason="within budget",
                                projected_usd=projected_total, spent_usd=spent,
                                ceiling_usd=self._ceiling)
```

智能体循环在每个周期开始时调用 `authorize_next`，如果决策结果不是 `allow`，则任务会干净利落地停止，而不是超支。这里，控制程序会授权一个周期，然后在运行的稍后阶段更新累计总数。

```

07:30:01 INFO  governor.authorize cycle=c1 allow=true projected_usd=0.06 spent=0.094
...
17:48:02 INFO  governor.tick cum_usd=31.29 tests=0/1 active=01 cycles_today=1 attempts=12
```

第一行显示，由于预计总额远低于上限，因此该周期获得批准。第二行（时间较晚）显示，在运输任务失败的一天后，累计支出为 31.29 美元。州长最终无需干预这项任务，因为最终支出为 178 美元，而上限为 400 美元。但这正是为什么一个陷入僵局的任务不会悄无声息地累积上千美元的原因。

### 回路检测器

第二个防护措施是时间。智能体可能会陷入反复尝试同一失败操作的困境。循环检测器会监视相同的状态和动作特征是否重复出现，一旦超过阈值，它就会触发并升级事件。

![当相同的状态和动作重复过于频繁时，循环检测器会触发（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/33.webp)

```python
class LoopDetector:
    """Flags oscillation: the same (state, action) signature recurring too often."""    def __init__(self, *, threshold: int = 3) -> None:
        self._threshold = threshold
        self._counts: Counter[str] = Counter()    def observe(self, signature: str) -> bool:
        """Record a signature; return True once it has looped past the threshold."""
        self._counts[signature] += 1
        return self._counts[signature] >= self._threshold
```

正是这一点避免了 LSP 任务在传输过程中无限期地卡住。在同样的编辑操作不断失败后，检测器先是发出警告，然后触发警报，最终将这项工作上报给了我们之前遇到的那位审核员。

```

17:48:02 WARN  loop_detector signature="01:transport:edit" count=3 action=warn
...
10:15:09 ERROR loop_detector signature="01:transport:rewrite" count=4 action=TRIP
10:15:09 INFO  orchestrator.pause item=01 ; escalate=reviewer reason=loop-trip
```

签名“01:transport:rewrite”重复出现了四次，检测器触发了警报。为了避免主控端继续失败，协调器暂停了该项任务并请求审核员介入。正是这次升级处理产生了架构发现，从而解决了所有问题。循环检测器并没有修复这个错误。它注意到智能体无法自行修复，因此请求了帮助。

### 默认拒绝出口

现在说说危险的部分——网络。出口策略默认是拒绝访问。除非明确列入允许列表，否则任何主机都无法访问。而且智能体永远不会持有真实的凭据。它看到的只是占位符，真正的密钥会在最后一刻，也就是在出口边界处，由代理服务器替换掉。

![出口默认拒绝访问，智能体仅在边界注入真实密钥（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/34.webp)

```python
class EgressPolicy(BaseModel):
    """Allow-list of permitted egress hosts (default-deny: empty = nothing allowed)."""    allow_hosts: set[str] = Field(default_factory=set)    def permits(self, url: str) -> bool:
        host = urlparse(url).hostname
        return host is not None and host in self.allow_hostsclass CredentialBroker:
    """Maps placeholder tokens to real secrets, injected only at egress."""    def resolve(self, value: str) -> str:
        for placeholder, secret in self._secrets.items():
            value = value.replace(placeholder, secret)   
        return value
```

好处显而易见。即使智能体被恶意网页劫持，最多也只会泄露一个无用的占位符字符串，因为真正的密钥永远不会进入其上下文。而且它只能访问我们授权的主机。在跟踪记录中，每次获取操作都会在其旁边记录出口决策。

```

09:00:12 INFO  tool.exec agent=researcher-1 name=web_fetch
               url=microsoft.github.io/.../specification/
```

获取请求成功，日志显示 `egress=allow(microsoft.github.io)`，这意味着该主机在允许列表中。任何不在列表中的主机的请求都会在离开机器之前就被策略拒绝。网络访问权限是智能体被严格授予的特权，并非默认权限。

### 二元法则

这里蕴含着更深层次的安全理念，它是一种结构性的安全措施，而非单一的检查。Meta 将其称为“二选一”规则。存在三种危险的能力：摄入不受信任的内容、访问私人数据以及与外部通信。同时拥有这三种能力是极其危险的组合，会导致快速注入攻击造成灾难性后果。因此，单个会话最多只允许拥有这三种能力中的两种。

![一个会话最多可以包含以下三者中的两项：不受信任的内容、私有数据和外部通信（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/35.webp)

```python
class Capability(str, Enum):
    UNTRUSTED_CONTENT = "untrusted_content"   
    PRIVATE_DATA = "private_data"             
    EXTERNAL_COMMS = "external_comms"         def check_rule_of_two(capabilities: set[Capability]) -> None:
    """Raise if a session would hold the full lethal trifecta (all three at once)."""
    if len(capabilities) >= 3:
        raise RuleOfTwoViolation(
            "session would hold untrusted content + private data + external comms "
            "(the lethal trifecta); split capabilities across sessions"
        )
```

编排器会在授予会话相应权限之前检查这一点。如果一项工作需要全部三种权限，则会将其拆分到不同的会话中执行。您在研究的扇出跟踪中看到了这一点的证明，其中一行显示为 `rule_of_two=ok caps=2`。研究人员可以读取不受信任的网页内容并访问网络（这需要两种权限），但他们无法访问私有数据。系统在允许他们运行之前强制执行了此限制。这是基于结构的安全保障，而不是寄希望于模型的行为。

### 人机交互门

有些操作风险过高，例如合并到受保护的分支或部署，因此绝不能自动执行。对于这些操作，系统会暂停并请求人工干预。该机制具有持久性，可以等待数天而不会产生任何成本，更重要的是，它设置了超时机制并带有默认操作，因此即使无人干预，任务也不会无限期地挂起。

![不可逆操作会在具有超时和默认操作的持久门处暂停（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/36.webp)

```python
class AutoPolicyGate:
    """Resolves gates by policy alone (no human)."""    async def request(self, req: GateRequest) -> GateResolution:
        if req.risk is RiskTier.REVERSIBLE and self._auto_approve_reversible:
            return GateResolution(gate_id=req.gate_id, decision=GateDecision.APPROVE,
                                  resolved_by="auto-policy")
        
        return GateResolution(gate_id=req.gate_id, decision=req.default_action,
                              resolved_by="default", defaulted=True)
```

该门的持久版本存在于工作流本身，并使用信号进行处理。当需要人工干预时，工作流会将其状态设置为“等待人工干预”，并停留在该状态，直到有人发出决策信号或超时为止。一个典型的状态查询如下所示。

```bash

$ uv run lra mission-status mission_4c7e21a9
status=WAITING_ON_HUMAN cycles=190$ uv run lra mission-approve mission_4c7e21a9 --decision approve
sent decision 'approve' to mission mission_4c7e21a9
```

一个健康的已停放任务会显示“SLEEPING”（睡眠中），而“WAITING_ON_HUMAN”（等待人工处理）则表示存在待处理事项，需要有人处理。批准命令会通知工作流，工作流会立即恢复。如果在超时前无人响应，则会执行默认操作，对于不可逆操作而言，默认操作是停止运行。因此，默认情况下，无人值守运行是安全的。它不会悄无声息地执行危险操作，也不会悄无声息地挂起。

## 能持续数周的记忆

我们有一个可以安全运行一周的系统。但是，安全运行一周和在一周内学习是两回事。保存的状态使智能体能够完美地回忆起当前任务。而记忆赋予它更多能力：将经验教训跨周期、最终跨任务地应用。正是在这里，系统不再只是执行任务，而是开始不断改进。

### 记忆的三层结构

我们将记忆分为三个层次，借鉴了人们谈论记忆的方式。情景记忆记录的是已经发生的事情，也就是原始的事件日志。语义记忆是关于代码和领域的提炼事实。程序性记忆是可重用的技能，是经过验证的实际代码及其适用条件。

![记忆分为三个层次：情景事件、语义事实和程序性技能（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/37.webp)

```python
class MemoryRecord(BaseModel):
    id: str
    kind: str           
    text: str
    metadata: dict[str, str] = {}
    embedding_model: str | None = None
    embedding_version: str | None = None
    valid: bool = True  @runtime_checkable
class SemanticIndex(Protocol):
    """Stores records and retrieves them by semantic similarity."""    async def add(self, records: list[MemoryRecord]) -> None: ...
    async def query(self, text: str, *, k: int = 5) -> list["RetrievalHit"]: ...
```

请注意 `valid` 标志以及 `embedding_model` 和 `embedding_version` 字段。`valid` 标志表示我们不会直接删除记忆，而只是将其标记为无效，稍后会详细说明。`model` 和 `version` 字段的作用是防止一个不易察觉的长期错误：比较由不同嵌入模型生成的向量。几周后，您可能会更改嵌入器，而悄悄地混合来自两个模型的向量会导致垃圾回收。

### 嵌入

为了按含义搜索记忆，我们将文本转换为向量。当然，嵌入器是另一个可替换的接口。一个是调用付费语义 API 的正式版本，另一个是免费的确定性离线版本，用于测试和本地运行。

![嵌入来自可交换嵌入器，并提供离线哈希版本，运行次数为 0 美元（创建者：）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/38.webp)

```python
class HashEmbedder:
    """A deterministic, offline embedder (lexical hashing). For dev/CI/offline only."""    def __init__(self, dim: int = 256) -> None:
        self.name, self.version, self.dim = "hash", "1", dim    async def embed(self, texts: list[str]) -> list[list[float]]:
        return [self._embed_one(text) for text in texts]    def _embed_one(self, text: str) -> list[float]:
        vec = [0.0] * self.dim
        for token in _TOKEN.findall(text.lower()):
            bucket = int(hashlib.sha1(token.encode("utf-8")).hexdigest(), 16) % self.dim
            vec[bucket] += 1.0                       
        norm = math.sqrt(sum(v * v for v in vec)) or 1.0
        return [v / norm for v in vec]               
```

哈希嵌入器并非语义化的，它只是词法哈希，但它是确定性的且免费的，因此整个记忆子系统无需网络和密钥即可运行和测试。在生产环境中，只需在同一接口后方替换真正的 Voyage 嵌入器即可。关键在于，系统的其他部分保持不变。

### 混合检索

纯向量搜索会遗漏精确的关键词匹配，而纯关键词搜索则会遗漏语义信息。因此，我们同时运行这两种方法，并将排名结果融合。我们使用互惠排名融合技术，将密集语义搜索与词汇 BM25 搜索相结合，这种方法无需两个排名列表的分数处于同一尺度即可进行融合。

![密集搜索和词汇搜索与互惠排序融合，然后重新排序（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/39.webp)

```python
def reciprocal_rank_fusion(rankings: list[list[str]], *, k: int = 60) -> list[tuple[str, float]]:
    """Fuse ranked id-lists via RRF: score = sum of 1/(k + rank)."""
    fused: dict[str, float] = {}
    for ranking in rankings:
        for rank, doc_id in enumerate(ranking, start=1):
            fused[doc_id] = fused.get(doc_id, 0.0) + 1.0 / (k + rank)
    return sorted(fused.items(), key=lambda pair: pair[1], reverse=True)class HybridRetriever:
    async def query(self, text: str, *, k: int = 5, candidate_k: int = 20) -> list[RetrievalHit]:
        dense = [h.record.id for h in await self._semantic.query(text, k=candidate_k)]
        lexical = [doc_id for doc_id, _ in self._bm25.query(text, k=candidate_k)]
        fused = reciprocal_rank_fusion([dense, lexical])     
        
```

这种融合方式非常巧妙。每个列表都会贡献一个分数，分数等于 1 加上一个常数和排名，因此，如果一个项目在两种方法中都排名很高，它就会排在首位；而即使只有一个方法找到该项目，它仍然有机会被其他方法找到。这是标准的两阶段检索方案，而且完全是用 Python 编写的，所以运行成本为零，并且结果完全确定。此外，还有一个可选的交叉编码器重排序器用于最终优化，但默认的无操作重排序器是免费的。

### 合并

零散的事件会迅速堆积。我们不想永远保存上万个原始事件。因此，在持久休眠期内，一个类似图书管理员的角色会将最近的事件整合为几个持久的语义事实。这相当于模型反思自身的工作，并将值得保留的内容记录下来。

![睡眠期间，图书管理员会将最近发生的事件提炼成持久的语义事实（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/40.webp)

```
async def consolidate(*, model, episodes: list[str], mission_id=None) -> list[MemoryRecord]:
    """Distill `episodes` into semantic MemoryRecord facts (model-driven, safe fallback)."""
    if not episodes:
        return []
    joined = "\n".join(f"- {e}" for e in episodes[-200:])
    result = await model.complete([
        ModelMessage(role="system", content="You are the Librarian; consolidate memory."),
        ModelMessage(role="user", content=f"{_INSTRUCTIONS}\n\nEpisodes:\n{joined}"),
    ])
    facts = _parse_facts(result.text)
    if not facts:                                       
        facts = list(dict.fromkeys(episodes))[-10:]     
    return [MemoryRecord(id=new_id("fact"), kind="semantic", text=f, metadata=...) for f in facts]
```

遗忘机制是刻意保守的。我们绝不让模型直接改写事实，因为那样会将幻觉变成记忆中的真相。我们只对事实进行软性无效化标记。以下是 LSP 任务中最艰难的两天之后进行的巩固步骤。

```

07:00:05 INFO  librarian.consolidate window=2d episodes=29 -> lessons=4
07:00:05 INFO  librarian.lesson L1 "wire-protocol framing is byte-level + stream-mode-sensitive -> isolate & unit-test FIRST"
07:00:05 INFO  librarian.lesson L2 "never fix 3 entangled problems in one file; split them"
07:00:05 INFO  librarian.lesson L3 "stream.read(n) on a pipe can short-read -> loop until n bytes"
```

一夜之间，二十九个原始片段被提炼成了四个精辟的教训。请仔细阅读这些教训。它们正是系统在艰难时期痛苦摸索后总结出的宝贵经验。第二天早上，这些教训被运用到重新规划中，这正是第二次尝试取得巨大成功的原因。系统经过反思和沉淀，醒来后更加成熟。

### 技能库

最高层级是程序性记忆，它是一个可重用技能的实际库。技能本身就是代码加上其适用的前提条件。关键规则是，只有经过验证的技能才能被收录到技能库中，也就是说，它在学习时必须通过确定性测试。

![技能只有通过确定性测试关卡才能进入技能库（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/41.webp)

```python
class InMemorySkillStore:
    async def add(self, skill: Skill) -> None:
        if not skill.verified:
            raise SkillNotVerifiedError(
                f"refusing to admit unverified skill {skill.id!r} (must pass the test gate first)"
            )
        self._by_id[skill.id] = skill
        await self._index.add([MemoryRecord(
            id=skill.id, kind="procedural", text=skill.description,
            metadata={"namespace": skill.namespace, "name": skill.name},
        )])
```

商店断然拒绝接受未经验证的技能。正是这一点防止了技能库被看似合理实则谬误的内容所充斥。技能还带有来源和有效期，因此课程不会永远停留在原地。请注意 LSP 跟踪中的技能门控机制，它先是拒绝，然后才接受。

```

18:35:10 INFO  skill.admit count=0 reason="nothing passed the gate; the system records no progress it didn't earn"
...
09:11:02 INFO  skill.confirm name=lsp-framing precond=[python,stdio] proven_by=tests/test_framing.py
11:50:02 INFO  skill.admit name=lsp-utf16-offsets proven_by=tests/test_positions_unicode.py
```

整个过程中我最喜欢的就是第一行。经过一整天的失败尝试，没有任何技能被采纳，因为没有一项技能通过审核。系统不会记录任何它没有获得的进度。然后，当框架最终通过测试后，技能“lsp-framing”被确认，并以测试文件作为证据。之后，UTF-16 修复也获得了自己的技能。库中只包含经过验证有效的内容。这使得技能可以安全地在下一个任务中重复使用。

## 让教训永存

我们刚刚构建的记忆在执行任务时很有帮助。但最有趣的学习发生在任务之间。任务结束后，系统可以回顾之前的失败，改进自身的提示，从而使下一个同类型任务进行得更顺利。这就是自我改进，而且必须非常谨慎地进行，因为如果贸然让智能体重写自己的指令，将会非常危险。

### 提出更好的提示

第一步完全离线进行，在任务间隙完成，绝不会在实时运行期间执行。演化器读取已完成任务的失败跟踪信息，并为某个角色提出改进的系统提示。这是一种反思性优化，符合 GEPA 的理念。重要的是，输出结果只是一个候选方案，尚未进行任何更改。

![离线状态下，演化器读取故障跟踪信息并提出候选提示（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/42.webp)

```python
class PromptEvolver:
    """Proposes an improved prompt for a role from its failure traces (offline)."""    async def propose(self, *, role, current_prompt, failure_traces) -> PromptCandidate:
        traces = "\n".join(f"- {t}" for t in failure_traces[-50:]) or "(no traces)"
        result = await self._model.complete([
            ModelMessage(role="system", content=(
                "You optimize an agent's SYSTEM PROMPT by reflecting on its failures "
                "(GEPA-style). Output ONLY the improved prompt text, no preamble.")),
            ModelMessage(role="user", content=(
                f"Role: {role}\n\nCurrent prompt:\n{current_prompt}\n\n"
                f"Failure traces:\n{traces}\n\nReturn an improved prompt.")),
        ])
        candidate = result.text.strip() or current_prompt
        return PromptCandidate(role=role, prompt=candidate,
                               rationale="reflective optimization over failure traces (eval-gated)")
```

演化器会分析出错的原因，并提出改进方案。例如，在 LSP 任务中，它发现由于协议本身存在问题，且范围过大，导致耗时数天，因此提出了一些规则来避免下次出现类似情况。

```

22:14:40 INFO  evolve.candidate diff="+ for stdio/wire-protocol: implement+unit-test byte-level framing
    in ISOLATION first (incl. a short-read stream + a multibyte body); use binary streams; start
    synchronous. + if an unproven core has >~10 dependent items, cut scope until the core is green."
```

这位候选人就像是首席工程师经历了一周痛苦的教训，最终浓缩成几条必须遵守的规则。但候选人仅仅是一个提案，我们目前还不能完全信任它，它需要时间来证明自己的价值。

### 评估工具和法官

为了判断候选题目是否真的更优，我们会将其与一组预先准备好的标准题目进行对比，并对结果进行评分。评分过程中，我们会聘请一位独立评委，并根据评分标准进行评判，因为许多评判标准都比较模糊，并非简单的合格或不合格。

![候选人根据金牌案例由独立智能体（作为评委）进行评分（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/43.webp)

```
async def run_eval_suite(*, model: ModelProvider, cases: list[EvalCase]) -> EvalReport:
    """Judge each case and aggregate into a report (pass rate + mean score)."""
    judge = AgentAsJudge(model)
    results = []
    for case in cases:
        verdict = await judge.judge(candidate=case.candidate, rubric=case.rubric)
        results.append(EvalCaseResult(name=case.name, passed=verdict.passed,
                                      score=verdict.score, rationale=verdict.rationale))
    return EvalReport(results=results)
```

评判者刻意与它评判的智能体分开，并且默认情况下，如果输出无法解析则判定失败，因此它绝不会默默地通过任何判断。以下是评估结果，比较了旧提示和改进后的提示。

```

22:14:55 INFO  eval.run set=wire-protocol-gold n=20 baseline=0.71 candidate=0.85
22:15:30 INFO  judge.verdict model=claude-opus-4-8 promote=true delta=+0.14 regressions=0
```

在二十个黄金案例中，基准提示得分为 0.71，改进后的提示得分为 0.85。评委建议晋升，得分提高了 0.14，且没有出现任何倒退。现在候选人有了证据支持。但我们仍然不会贸然发布。

### 晋级之门

最后一关最为严格。候选程序只有在评估中达到基准水平才能晋升，并且还必须通过与已记录任务历史记录的比对，以确保其不会破坏确定性。即便如此，最终也需要以拉取请求的形式提交，并由人工审核批准。实时智能体的提示信息绝不会进行热补丁更新。

![候选人只有在经过评估、无误的复盘以及人工审核后才能晋升。（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/44.webp)

```
async def evolve_and_gate(*, model, role, current_prompt, failure_traces,
                          baseline_cases, candidate_cases, min_improvement=0.0) -> EvolutionResult:
    """Propose an improved prompt and promote it ONLY if it beats baseline by min_improvement."""
    candidate = await PromptEvolver(model).propose(
        role=role, current_prompt=current_prompt, failure_traces=failure_traces)
    baseline = await run_eval_suite(model=model, cases=baseline_cases)
    evolved = await run_eval_suite(model=model, cases=candidate_cases)
    promoted = evolved.pass_rate >= baseline.pass_rate + min_improvement
    return EvolutionResult(promoted=promoted, candidate=candidate,
                           baseline_pass_rate=baseline.pass_rate,
                           candidate_pass_rate=evolved.pass_rate,
                           reason=("candidate improved" if promoted else "rejected"))
```

晋级流程由一系列独立的检查、评估、裁判和复审组成。这是从追踪结果来看的最终晋级步骤。

```

22:15:31 INFO  replay.histories n=9 result=9/9 clean
22:15:31 INFO  evolve.promote target=prompts/lead.md mode=PR human_approval=required (never hot-patched a live run)
```

九条记录的历史记录都根据新的提示进行了干净利落地重现，之后才将更改作为拉取请求提交，并需要人工批准。这句话的结尾体现了系统自我改进的整个安全理念。系统从不在运行中直接进行热补丁。它会提出改进方案，验证其有效性，并征求意见，只有这样，改进才会生效。下一个线路协议任务将省去大部分浪费数天的时间，因为经验教训现在已经融入到负责人的既定指令中。这个循环已经物有所值。

## 一周的任务

我们已经构建了所有组件。现在，让我们观察它们在一周内如何协同工作，完成我一直引用的那个任务。这个任务是从零开始构建一个小型 Python 语言服务器。语言服务器通过标准输入输出使用特定的网络协议进行通信，而这最终却是一个出乎意料的棘手问题。本节将完整记录整个运行过程，从开始到结束，以便您了解我们构建的各个组件在实际压力下的表现。

### 零日：研究与计划

任务始于一条命令。工作流启动，保存的状态被初始化，控制器读取其上限。然后，主导者没有猜测协议，而是明智地将第一步用于研究。

![第零天：研究成果分发给一项计划，该计划的范围超出预期，涵盖 31 个项目（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/45.webp)

```bash

$ uv run lra mission-start --title pylsp-mini --workdir .lra/workspaces/pylsp-mini --task @mission.txt
09:00:01 INFO  workflow.started wf=mission:mission_4c7e21a9 task_queue=lra-mission
09:00:01 INFO  governor.config ceiling_usd=400.00 max_cycles=2000 stall_limit=5
09:00:02 DEBUG planner.thinking "I do not have the LSP wire format memorised. Guessing will burn cycles.
               Spend on READ-research first: base protocol framing, a real server's transport, exact JSON."
09:47:55 INFO  checklist.write items=31  (01 transport, 02 init, 03 hover, 04 completion, ...)
```

最终的计划包含了三十一项，这太多了。在传输机制尚未建立之前就规划整个服务器，这正是我们之前指出的过度设计，也是后续工作浪费的根源。但系统当时并不知道这一点。它执行了计划并开始运行。这很正常。优秀的工程师也会过度规划。关键在于系统能否从中恢复，而它确实可以。

### 难点

传输层是第一个问题，而且非常棘手。项目负责人编写了一个传输层，运行测试，结果失败了。他再次尝试，结果失败的原因却不一样。整整一天下来，他尝试了十二次，每次都修复了上一次的问题，同时又暴露出一个新的问题。先是缺少头部信息的裸 JSON，然后是 Windows 换行符转换导致的两字节不匹配，接着是管道读取数据过短，最后是 asyncio 死锁。

![十二次运输尝试，每次都解决了一个症状，但又暴露了下一个症状（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/46.webp)

```

09:20:38 verify.fail attempt=3 reason=byte-mismatch delta=+2  
10:11:40 verify.fail attempt=6 reason=JSONDecodeError "Unterminated string"  
11:11:02 verify.fail attempt=10 reason=Timeout >12s  
17:48:02 WARN  loop_detector signature="01:transport:edit" count=3 action=warn
```

这些故障中的每一个都是真实的 bug，即使是人类在 Windows 上编写语言服务器传输程序时也会遇到。关键在于最后一行。循环检测器已经开始计数。系统注意到同一个项目以相同的方式不断失败。它目前还没有介入，但它正在监视，而这种监视即将发挥作用。

### 崩溃与重玩

第三天，传输状态仍然为红色，主机在周期进行中途重启进行操作系统更新。这足以让普通智能体崩溃。但在这里，这却不成问题。

![周期中途重启可通过重放恢复，从而恢复到之前的状态（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/47.webp)

```

13:48:12 ERROR worker.disconnect reason="connection reset (host OS update reboot)" inflight_cycle=c190
13:55:40 WARN  temporal.replay events=631 (completed activities served from cache; 0 tokens re-spent; 0 double commits)
13:55:41 INFO  reconcile.in_flight ticket=01 branch=feat/transport -> adopt (branch present, unmerged)
13:55:42 DEBUG orchestrator.note "replay restored state EXACTLY -> a still-RED transport."
```

工作进程崩溃后，一个新的工作进程启动了。Temporal 重放了日志，从缓存中提供了已完成的工作，任务从它之前的周期继续执行。没有重复使用令牌，也没有重复提交。请注意，它恢复了一个仍然失败的传输，因为那才是真实的状态。持久化骨干网保护了工作，并没有假装错误已经修复。任务在崩溃前就卡住了，崩溃后也卡住了，这完全正确。

### 摆脱困境

最终打破僵局的并非一次偶然的修改，而是循环检测器触发，并将问题上报给审核人员，从而产生了我们之前看到的架构缺陷。一夜之间，管理员将这些缺陷总结成经验教训。第二天早上，规划人员根据这些经验教训重新规划，缩小了项目范围，项目负责人则将传输层重建为一个独立的、经过测试的单元。

![经验教训加上审阅者的发现促使重新制定计划，并取得了第一个绿色成果（由……创建）。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/48.webp)

```

07:00:06 INFO  reflection.write item=01 "the 12 attempts were the same 3 bugs reshuffled. stop editing
               transport.py in place; build framing as a tested unit, sync, binary."
07:31:02 INFO  state.migrate checklist v1->v2 items=14 (dropped 06-09, plus 4 sub-items)
09:10:21 INFO  verify.ok cycle=c201 checks=[pytest]   <- FIRST GREEN (cycle 201, ~2.5 days in)
```

阅读这段文字。反思指出了真正的问题所在：那十二次尝试其实只是三个 bug 换了个位置而已。检查清单从三十一项缩减到十四项。在第 201 个周期，大约两天半之后，传输终于通过了测试。这是整个任务的第一个“绿色”标志。在此之前的所有尝试都只是碰运气，而在此之后的一切都建立在坚实的基础之上。这次恢复得益于系统自身的机制：循环检测器、审查员、管理员和重新规划器，它们各自尽职尽责地完成了自己的工作。

### 扇形展开和最终门

核心部分通过验证后，其余部分进展迅速。各个功能彼此独立，因此可以并行分发给不同的实现者。之前研究人员在项目启动之初就发现的一个老问题——UTF-16 列编码——最终被证实有效并得到修复。之后，整个系统顺利通过最终审核。

![核心变为绿色后，功能展开，UTF-16 错误得到修复，最终通过验证。（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/49.webp)

```

07:30:03 INFO  child.spawn implementer:0 ticket={feature:hover, write_set:[features/hover.py, tests/test_hover.py]}
07:30:03 INFO  child.spawn implementer:1 ticket={feature:completion, write_set:[features/completion.py, ...]}
08:58:12 INFO  child.done implementer-0 branch=feat/hover verify=ok skill_reused=lsp-framing
09:31:45 INFO  reviewer.finding sev=blocking "HAZARD-1 from day0: LSP 'character' = UTF-16 CODE UNITS;
               you used Python str indices. convert at the boundary."
08:43:02 INFO  run_command cmd="make check" -> exit=0   "ruff: clean; mypy: clean; pytest: 58 passed"
08:44:51 INFO  mission.complete items=14/14 cycles=615 head=9d8c7b6 status=DONE
```

这些代码行里有很多值得称道的地方。实现者们并行处理不相交的文件，其中一位还重用了在最难的部分学到的 `lsp-framing` 技能，因此无需重新查找传输方式。审核人员发现了从一开始就存在的 UTF-16 编码问题。最后一道关卡 `make check` 顺利通过，58 项测试全部通过。整个任务在 615 个循环中完成了全部 14 个项目。这就是我们博客的开篇之句，现在，它中的每一个字都应该变得清晰易懂了。

## 已验证的与前沿的

明确说明这套系统的功能和局限性非常重要，因为很容易在演示中夸大其词。以下是清晰的划分。

以下部件均经过验证和测试。它们是工程设计，并且目前仍在使用。

- 持久性和崩溃恢复能力。运行过程中重启不会丢失任何工作，也不会重新消耗任何 token。
- 状态数据存在于模型之外。它保存在 Git 仓库中，因此任何重启都会在几秒钟内重建状态数据。
- 确定性验证。所有进展都由实际测试而非模型的判断来把关。
- 安全边界。工具门、默认拒绝出口、预算控制和循环检测器都在代码中强制执行。
- 分级记忆和技能重用，技能只有通过门槛才能被允许使用。

这就是前沿领域，我不会夸大其词。完全无人干预、持续数周的自主运行，即模型无需任何人工干预即可可靠地完成复杂任务，这并非已解决的问题，目前还没有任何模型能够可靠地做到这一点。我们目前能够保证高可靠性的任务周期仍然远低于一天。

![哪些是经过验证和检验的，哪些仍处于前沿领域（创建者）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@ae8908e3bd59743a2a2b81a238180b0c8149e3ea/ai-insights/2026-06/08/images/building-a-week-long-running-agentic-system/50.webp)

因此，这套系统的宣称非常精准。它可以连续运行数周。它会休眠、会恢复运行、会重新启动，并且始终不会丢失正在执行的任务，因为持久性和可验证性是系统的工程特性，而非我们希望模型具备的能力。模型以短时、经过验证的突发模式驱动工作，并由人工控制不可逆操作。这正是本博客的标题。持久性是工程特性，而非模型能力。我们并没有让模型变得更智能。我们构建的是一个系统，它能让现有的模型在一周内高效运行而不会崩溃。
