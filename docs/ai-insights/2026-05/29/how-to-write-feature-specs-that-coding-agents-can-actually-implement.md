---
title: 怎样写出 coding agent 真正能落地的功能规格
author: Rico Fritzsche
url: https://levelup.gitconnected.com/how-to-write-feature-specs-that-coding-agents-can-actually-implement-c7cd84e33cdc
translated: 2026-05-29
excerpt: 为什么 agentic coding 靠的是确定性契约，而不是模糊需求加 prompt 撞大运
tags:
  - Programming
  - Artificial Intelligence
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@50756e864d1372dfaaa4100bc21289f6c418a4c0/ai-insights/2026-05/29/images/how-to-write-feature-specs-that-coding-agents-can-actually-implement/01.thumb.webp
---

# 怎样写出 coding agent 真正能落地的功能规格

为什么 agentic coding 靠的是确定性契约，而不是模糊需求加 prompt 撞大运

![图片来源：Compagnons on Unsplash](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@50756e864d1372dfaaa4100bc21289f6c418a4c0/ai-insights/2026-05/29/images/how-to-write-feature-specs-that-coding-agents-can-actually-implement/01.jpg)

coding agent 正在改变工程精力最值钱的位置。如今，杠杆点更多落在动手实现之前。结果好不好，靠的不再是敲代码的速度，而是功能定义得多清楚、行为规定得多精确、留给"自由发挥"的空间有多小。

这是个好转向。它把注意力推向了那些本来就该被重视的东西：明确的契约、确定的行为、清晰的边界，以及一套能证明"功能真的做到了它该做的"的验证。在这种环境下，功能规格不再是一份软绵绵的规划文档，它本身就是工程界面的一部分。

coding agent 需要的东西不止是一个功能名加一个请求结构。它需要一条清晰的命令边界。规格必须把决策、相关事实上下文、验收条件，以及成功时产出的事实，全部定义出来。[Command Context Consistency](https://ricofritzsche.me/simplicity-wins-command-context-consistency/) 把这件事变成一份精确的工程界面：边界就落在决策本身，而不是某个含糊的用例描述里。边界一旦明确，功能就不会再化进共享逻辑里消失不见，实现也直接得多。

这篇文章里，我会聊聊：什么样的功能规格才算"可执行"，动手实现前有哪些部分必须显式写死，以及 prompt 应当怎样指向这些决策，而不是自己扛着这些决策走。这才是 agentic coding 里最实在的一面：不是生成更多代码，而是在代码生成之前做更多工程。

## 什么样的功能规格才算可执行

功能规格之所以可执行，是因为它把行为定下来了——否则这些行为就得在实现阶段被现编出来。它定义入口、定义合法输入、定义非法输入、定义要做的决策、定义这个决策依赖的上下文、定义成功决策的结果、定义失败情形，以及验证契约的测试。

定义必须精确。规格要写清什么事件触发功能、什么样的结果算成功；还要把请求形状写得足够精确，让校验本身是确定性的。必填字段、可选字段、trim、归一化、格式、取值范围、语义检查，都得写进契约。这些规则不显式定义，实现就只能从猜测开始。

规格要写明：这个功能被允许做哪些决策、哪些既有事实跟这个决策相关、决策成功时产出什么事实或结果。它还得定义什么算"重复输入"——重新创建、拒绝、当作幂等重复来处理，都是完全不同的行为。这部分行为定义清楚了，功能才算完整规约。

![可执行功能规格的内核，是一条固定路径：触发、契约、上下文、决策、已定义的结果。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@50756e864d1372dfaaa4100bc21289f6c418a4c0/ai-insights/2026-05/29/images/how-to-write-feature-specs-that-coding-agents-can-actually-implement/02.webp)

对外的失败契约同样要写明。畸形请求、业务输入非法、冲突、内部运行时失败——这些是不同的结果，必须有不同的含义。一旦写下来，实现路径就大大收窄。验证也一样。一份强的功能规格本身就暗示了它的证明：成功、非法输入、冲突或重试行为、边界情况。

## 一个简单例子：open\_account

open\_account 这份规格之所以可执行，是因为它在实现通常开始走偏的几个点上把行为钉死了。第一点是边界。

```
## Interface shape
- Method and path: POST /accounts
- Success statuses: 201 Created
- Failure statuses: 400 Bad Request, 422 Unprocessable Entity, 500 Internal Server Error
```

这就够阻止功能扩散成多个端点或意料外的输出。操作从一个 HTTP 边界开始，返回一组已定义的状态码。实现不必再去琢磨"这个功能该怎么暴露"。

第二点是输入契约。

```
#### date_of_birth
- required
- ISO date format: YYYY-MM-DD
- must be parseable as a calendar date
- must be a past date
- applicant must be at least 18 years old
#### residential_address.country_code
- required
- exactly 2 alphabetic characters after trimming
- stored uppercase
```

到这一步，功能不再只是粗糙的需求描述，而是一份确定性契约。规则定义了语法分析、语义有效性、归一化、存储格式。coding agent 在年龄、日期处理、国家代码归一化这些点上根本没有自由发挥的余地。

第三点是决策与重试行为。

```

1. The system generates the account_id.
2. The account opens in status open.
3. The account opens with currency EUR.
4. The initial balance is zero.

This feature is not idempotent.
The request is not deduplicated by applicant name, date of birth, address, or government_id.
In this example, repeated valid requests may open multiple accounts.
```

这一段把那些经常被默认带过的行为讲清楚了。调用方不指定账户 ID，而是由功能生成。重复同一个请求不会触发什么隐藏的去重规则，反而会再开一个账户。这些决定一旦写下来，实现就不用再猜"重复输入到底是什么意思"。

第四点是失败契约。

```
### 400 Bad Request
- INVALID_JSON
- INVALID_FIELD_TYPE
- INVALID_DATE_FORMAT### 422 Unprocessable Entity
- FIRST_NAME_REQUIRED
- LAST_NAME_REQUIRED
- DATE_OF_BIRTH_INVALID
- APPLICANT_MUST_BE_ADULT
- ADDRESS_REQUIRED
- COUNTRY_CODE_INVALID
- GOVERNMENT_ID_REQUIRED### 500 Internal Server Error
- INTERNAL_ERROR
```

这就把畸形输入、业务输入非法、内部运行时失败彻底分开，对外行为变小变清晰。实现照契约走，而不是边写边发明错误映射。

最后一点是验证。

```
## Test cases
1. Happy path: valid applicant data returns 201 and a generated account_id.
2. Validation failure: blank first_name returns 422 FIRST_NAME_REQUIRED.
3. Validation failure: malformed date_of_birth returns 400 INVALID_DATE_FORMAT.
4. Edge case: a second identical request creates a second account because this feature is not idempotent.
5. Internal failure: simulated persistence failure returns 500 INTERNAL_ERROR.
```

这些测试就是契约的一部分，定义了实现必须证明的东西。这就是这份规格可执行的原因：在代码生成之前，边界、输入规则、决策行为、失败契约和证明路径都已经钉死了。

## prompt 不该扛着功能走

规格把行为定下来之后，prompt 就可以保持"操作性"。它的工作不再是描述功能本身，而是把 agent 指向契约、指向当前的真相源、指向本地的实现边界、指向必要的验证。这是一个小得多、也可靠得多的角色。

这里用的 prompt 模板正是这么干的。它告诉 agent：先检查 [factstore 仓库](https://github.com/ricofritzsche/factstore)，把当前 API 当成真相源，避免引入 wrapper 和 CRUD 抽象，把功能保持在本地，输入输出保持显式，用常规的 Rust 检查来验证结果。prompt 自己不扛去重策略、字段规则、幂等行为或响应语义——这些属于规格。

来看一个 prompt 例子：

```
Use $feature-slice-rust and $factstore-usage to implement the feature slice: <feature-name>.Inspect the factstore repository first and use its current API as the source of truth. Do not invent wrappers or CRUD abstractions.Implement one small end-to-end feature in the existing Rust service. Keep it local under `src/features/<feature-name>/`, keep IO explicit, use a small internal split if the feature has multiple concerns, avoid HTTP-verb file names, and avoid generic technical roles.Use factstore directly for writes. Keep startup/config changes minimal. Do not add speculative abstractions.Verify with:
- cargo check
- cargo test
- cargo fmt 
- cargo clippy 
```

这种切分很重要。一旦 prompt 试图覆盖整个功能，它就会变成一份压缩版的、更弱的第二规格。结果就是两个真相源：一个在功能规格里，一个在 prompt 里。它们会很快漂移开，而实现最后会跟着更容易满足的那个走。所以 prompt 应当窄而流程化，由规格去描述行为。

项目规则同样在加固这条边界。feature-slice skill 让工作保持在本地，避免泛化的技术角色，让读、写、边界代码可见。factstore-usage skill 让写侧契约显式，防止 agent 把 factstore 藏到 repository 层或可变的 CRUD 模型背后。这些指令是结构性的，塑造的是功能如何被实现，而不是重新定义功能要做什么。

skill 定义实现必须遵守的结构规则；规格定义功能本身；prompt 定义如何在这个项目里执行。这三部分各管各的，agent 即兴发挥的空间就小得多，第一遍生成出来的代码也会更小、更贴近预期行为。

## 审计在收紧规格

一份功能规格可以已经很强，但仍在边角处留有漂移空间。这一点在 open\_account 这个例子里立刻就暴露了。主行为已经清楚到能落地实现，但第一遍里对外行为在某些地方还是开着口子——契约没有完全钉死。最大长度规则是有的，但这些情况下的失败映射没写得够明确。内部失败行为也得被纳入对外契约，而不是停留在"实现细节"。

重点不是一次就拿出完美规格，而是把残余的歧义挪到一个能看见、能复审、容易收紧的位置。一旦实现按写下来的契约被审计，规格里的薄弱处就藏不住——再也无法躲在"能跑的代码"背后。

这也改变了审计的角色。审计不只看代码能不能编、端点在不在、测试过不过，它检查的是实现行为是不是和书面契约完全一致。这包括没文档的状态码、没文档的错误码、缺失的测试、薄弱的边界情况定义，以及那些因为规格没先把它定下来、所以代码不得不自己发明行为的地方。

这是 agentic coding 里最有用的转变之一。实现不再把模糊的工程决策藏在 code review 或后续清理里，审计会把这些东西推回契约里——它们本来就该在那里。这一步做到了，下一遍实现就更小、更直接，对解释的依赖也少得多。

所以，功能规格通过实现和审计不断改进，直到契约紧到代码几乎没有即兴发挥的余地。这就是工程工作真正"上移"的那个点。代码跟着契约走，契约扛住所有难决策。

## 结语

coding agent 把工程精力往上推。写代码这件事本身价值变小了，把功能定义得足够紧、让实现可以照契约走而不是发明行为，这件事价值变大了。这就是可执行功能规格能给你的：固定的边界、确定性规则、明确的结果、一条证明路径。

这些决策一旦写下来，prompt 就可以保持小巧、实现可以保持本地、审计可以保持严格。代码不必再扛那些悬而未决的行为，契约已经把这份活干掉了。

功能规约得越精确，实现需要猜的就越少。

*Cheers*！

完整例子在这里：[https://github.com/ricofritzsche/agentic-feature-slice-templates/tree/main/examples/banking](https://github.com/ricofritzsche/agentic-feature-slice-templates/tree/main/examples/banking)
