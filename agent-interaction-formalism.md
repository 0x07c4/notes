# Agent 交互约束的形式化方法

## Metadata

| Field | Value |
| --- | --- |
| Language | `zh-CN` |
| Type | `architecture-note` |
| Status | `evergreen` |
| Summary | 讨论如何用协议、能力边界和不变量约束 `agent` 交互，并界定自然语言在系统里的正确位置。 |
| Topics | `formalism`, `AI engineering`, `agent runtime` |
| Keywords | `protocol`, `capability`, `invariant`, `state machine`, `natural language`, `Dijkstra`, `Brooks` |
| Related | [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md), [solo-runtime-boundary.md](./solo-runtime-boundary.md) |
| Last Updated | `2026-03-29` |

## 一句话判断

- 适合软件架构的，不是一个万能图形符号，而是一套可执行协议。
- 要约束 `agent`，核心不是把流程“画出来”，而是把交互顺序、权限边界和安全不变量同时形式化。
- 最实用的组合是：
  - `state machine / typestate` 做主干
  - `capability / effect system` 管副作用
  - 少量 `temporal logic` 管全局安全规则

## 为什么单一符号不够

如果只用一种形式化表示，通常会丢掉一部分关键信息：

- 只画流程图，约束不了权限
- 只写权限模型，约束不了交互顺序
- 只写逻辑公式，工程上又太重，不适合表达日常状态流

所以更合理的做法不是找“唯一正确符号”，而是分层：

- `Protocol`
  - 谁先说什么
  - 下一步允许什么
  - 哪些分支合法
- `Capability`
  - 当前参与方能做什么副作用
  - 哪些资源能读
  - 哪些动作必须审批
- `Invariant`
  - 哪些规则永远不能被破坏

## 适合的三个形式化方向

### 1. Session Types / Protocol Types

适合描述：

- `user`
- `agent`
- `tool`
- `workspace`

之间的消息顺序与合法分支。

最像“对话协议类型系统”。

如果目标是限制 `agent` 在某个架构里乱跳步骤，这类形式化很合适。

### 2. State Machine / Typestate / Statecharts

适合描述：

- 当前处于哪个状态
- 在这个状态下允许哪些动作
- 哪些动作会把系统推进到下一个状态

比如：

- `Drafted -> Previewed -> Approved -> Applied`

只要没有进入 `Approved`，`Apply` 就不合法。

这类抽象最适合直接落到产品和 runtime。

### 3. Temporal Logic / TLA+

适合表达全局安全性质，例如：

- `always(write -> approved)`
- `always(apply -> previewed_before)`
- `always(no_mode_switch_without_explicit_user_action)`

这类能力强，但成本也高。

更适合用来验证少量核心规则，而不是日常把所有流程都写成形式化证明。

## 对软件架构最实用的组合

主干用：

- `状态机 + capability/effect system`

关键安全规则用：

- 少量时序逻辑

一句话说，就是：

- `agent architecture` 不该抽象成“自由 planner + tools”
- 更稳的抽象是 `protocol machine + bounded effects`

## 一个最小 DSL 轮廓

```txt
protocol WorkspaceEdit {
  roles: User, Agent, Workspace

  states:
    Idle, Drafted, Previewed, Approved, Applied, Rejected

  transitions:
    Idle      --user.request-->      Drafted
    Drafted   --agent.preview-->     Previewed
    Previewed --user.approve-->      Approved
    Previewed --user.reject-->       Rejected
    Approved  --agent.apply-->       Applied

  capabilities:
    Agent.read(path)     if path in visible_scope
    Agent.search(query)  if query in allowed_sources
    Agent.write(diff)    only_in Approved
    Agent.exec(cmd)      if policy.allows(cmd)

  invariants:
    always preview_before_apply
    always no_write_without_approval
    always no_mode_switch_without_explicit_user_action
}
```

这类 DSL 的价值不在文档性，而在它能被编译成真实约束。

## 这套形式化应该落到哪里

### 1. Orchestrator

根据协议判断：

- 当前状态是什么
- 下一步是否合法
- 是否需要等待用户动作

### 2. Tool Executor

根据 capability 判断：

- 这个 `tool call` 是否越权
- 是否需要审批 token
- 是否超过工作区可见范围

### 3. UI Projection

根据状态机决定前端只显示什么：

- 建议
- 预览
- 审批
- 已应用

而不是让 UI 直接猜 agent 现在“想干嘛”。

### 4. Eval / Replay

根据 invariant 检查轨迹：

- 是否发生未审批写入
- 是否跳过预览直接应用
- 是否发生未显式触发的模式切换

## 工程实现上的关键判断

只有形式化描述还不够，必须变成 `runtime enforcement`。

否则它只是文档，不是约束。

最小闭环应该是：

1. 用协议描述允许的交互路径
2. 用 capability 描述允许的副作用
3. 用审批 token 或状态门控保护高风险动作
4. 所有工具调用先过 policy check，再执行
5. 所有执行轨迹可记录、可回放、可做 eval

## 对 Solo 这类产品的启发

如果产品强调：

- `human-in-the-loop`
- `建议 + 预览 + 用户确认`

那最适合的建模方式不是“自治 agent”，而是：

- 一个严格受协议驱动的状态机
- 一个被 capability 限制的副作用执行层

也就是：

- `agent` 负责生成候选动作
- `protocol machine` 负责决定这些动作能否进入下一步

这比把架构中心放在“planner 是否足够聪明”上更稳。

## 自然语言该停在哪一层

`EWD667` 的核心警告到今天仍然成立：

- 自然语言不适合充当最终可执行语义

但在 LLM 时代，不能把这个结论误读成“自然语言没用”。

更准确的分层应该是：

### 1. Intent Layer

自然语言适合放在这里：

- 用户目标
- 任务背景
- 偏好与约束
- 验收方向

这一层允许模糊，因为它本来就是在表达人类意图。

### 2. Deliberation Layer

自然语言也适合放在这里：

- agent 的计划草案
- 风险说明
- 候选方案比较
- review 评论

这一层的价值是帮助人和 agent 对齐，而不是直接驱动副作用。

### 3. Protocol Layer

从这里开始，就不该主要靠自然语言了。

这层应该形式化为：

- 状态机
- 事件类型
- 审批流
- 角色边界
- 可恢复的 turn/item 历史

否则系统会退化成“看起来在协作，其实全靠 prompt 猜”。

### 4. Effect Layer

这一层必须更硬：

- tool schema
- 参数校验
- 权限检查
- scope 限制
- diff / command / file write 的显式表示

这里不能让自由文本直接决定执行。

### 5. Correctness Layer

这一层最不能靠自然语言：

- 类型系统
- 结构化输出
- 测试
- invariants
- policy checks
- evals

自然语言可以描述“想要什么”，但不能单独证明“已经正确”。

## 一个实用边界

可以把自然语言放在：

- `specification draft`
- `intent capture`
- `steering`
- `review`

不要把自然语言放在：

- 权限判定
- 最终状态转移
- 副作用授权
- 数据结构真相
- 正确性判定

一句话说：

- 自然语言适合做 `intent layer`
- 不适合做 `correctness layer`

## Brooks《No Silver Bullet》的补充判断

Brooks 的核心区分到今天仍然非常有用：

- `accidental complexity`
  - 来自工具、语言、流程、环境、重复劳动
- `essential complexity`
  - 来自问题本身、需求歧义、概念边界、状态爆炸、协作协调

他的核心判断不是“工具没价值”，而是：

- 不要指望某一种新技术在十年内把软件开发整体生产率、可靠性、简单性提升一个数量级

### 放到 LLM 时代，哪部分还是对的

大体还是对的。

因为今天被明显压缩的，主要还是：

- 脚手架生成
- 样板代码
- API 粘合
- 文档整理
- 测试补全
- 重复性迁移

这些多数属于 `accidental complexity`。

而真正难的部分依旧存在：

- 到底要建什么
- 边界怎么划
- 状态机怎么定义
- 权限和审批怎么建模
- 系统如何长期保持一致性
- 多人 / 多 agent 协作下如何避免语义漂移

这些更接近 `essential complexity`。

### 哪部分需要修正

如果把 Brooks 的原话机械搬到今天，也不够准确。

因为 LLM 确实已经在局部任务上带来了接近数量级的提速，尤其在：

- 初始实现
- 探索式编码
- 跨栈切换
- 小修小补
- 测试与重构辅助

所以今天更准确的说法不是：

- “没有任何地方出现银弹”

而是：

- “没有一个单独技术能在系统级消灭软件工程的本质复杂度”

### 对 AI engineering 的直接启发

这意味着：

- 不要把模型能力当银弹
- 真正的工程杠杆在 `harness`
- 提升来自一整套组合，而不是一个神模型

如果用更工程化的话说：

- 模型主要在吃掉 `accidental complexity`
- 架构、协议、权限、评估、产品边界，仍然要由系统设计去处理 `essential complexity`

### 和 Dijkstra 放在一起看

这两篇其实能拼成一个很强的判断：

- Dijkstra 提醒你：不要让模糊表示充当最终语义
- Brooks 提醒你：不要幻想某个新工具会消灭本质复杂度

合起来就是：

- 不能因为 LLM 会说话、会写代码，就把它当成 runtime
- 也不能因为 LLM 提升巨大，就误以为架构、协议和验证突然不重要了

## 对今天的结论

如果硬压成一句话：

- `LLM 不是 silver bullet，而是 accidental-complexity compressor`

它非常强，但它压缩的主要不是“系统为什么难”，而是“把系统做出来时那些烦人的摩擦成本”。

## 为什么 Agent Runtime 会变成新基础设施层

如果模型只能补全代码，那工程中心还在编辑器、编译器和 CI。

但当模型开始：

- 连续多步工作
- 调工具
- 读写工作区
- 需要审批
- 需要恢复和追溯

系统的关键问题就不再只是“模型够不够强”，而变成：

- 这一轮在做什么
- 哪些副作用允许发生
- 当前状态能不能恢复
- 用户在哪里介入
- 不同客户端怎么共享同一轮运行态

这时候就会自然长出一层新的基础设施：

- `thread`
- `turn`
- `item`
- `approval`
- `diff`
- `plan`
- `event stream`
- `replay`

这层东西既不是 UI，也不是模型本身，更不是单个工具。

它本质上就是：

- `agent runtime`

### 为什么它会变成基础设施，而不是产品细节

#### 1. 状态管理会跨客户端复用

同一个运行中的任务，可能会被：

- Web
- CLI
- IDE
- review 面板

同时消费。

如果没有统一 runtime，每个前端都会各自拼一套“当前发生了什么”的解释层。

最后一定漂。

#### 2. 副作用治理不能留给 prompt

只要 agent 会：

- 执行命令
- 改文件
- 发请求
- 调外部工具

就必须有独立于自然语言的：

- capability
- policy
- approval
- scope

这套东西天然属于 runtime，不属于 prompt。

#### 3. 可恢复、可分叉、可回放不是附属功能

真正进入工作流之后，用户一定会要：

- 恢复 thread
- 从某一轮 fork
- 看上一次 agent 到底做了什么
- 追查哪一步开始偏

这说明历史和事件流必须是一等对象，而不是聊天记录副产物。

#### 4. Eval 和审计需要结构化轨迹

如果系统只留下自由文本，你几乎没法稳定回答：

- 这轮有没有越权
- 有没有跳过审批
- 哪一步失败
- 哪个模型或哪个 prompt 改坏了行为

要做这些，就必须有 runtime 轨迹。

#### 5. Provider 会换，runtime 不能跟着漂

模型、SDK、接入层都会变。

如果产品语义也跟着变，说明你没有 runtime，只有 provider patch。

真正稳定的一层应该是：

- provider 可替换
- runtime 不漂
- UI 读 projection

### 所以 runtime 的真实位置是什么

它在架构里的位置，应该介于：

- 上层产品交互
- 下层模型 / tool / provider

之间。

它负责把这些东西收成一个稳定系统：

- 用户意图
- agent 计划
- 工具调用
- 审批
- 文件改动
- 历史
- 恢复
- 评估

一句话说：

- `agent runtime` 是 AI system 的 transaction layer

## 为什么大多数“Agent”还没到 Runtime 这一层

现在很多所谓 agent，其实更接近：

- prompt orchestration
- tool wrapper
- workflow script

而不是完整 runtime。

它们常见的特征是：

- 只有 message，没有 `turn / item`
- 工具调用能跑，但没有统一 event model
- 有审批弹窗，但审批不在协议里
- 有历史，但历史只是文本日志
- 能“继续”，但不能稳定恢复到结构化状态
- 能演示，但很难 replay、fork、audit

这类系统能做 demo，也能做局部自动化。

但它们一旦往真正的工作流系统走，就会撞到同一堵墙：

- 状态不稳
- 权限不稳
- UI 解释层太重
- provider 一换就漂

所以很多人现在谈的 “agent”，本质上还是：

- `LLM + tools + prompt glue`

还不是：

- `runtime + protocol + bounded effects`

## 最后一句

适合软件架构、且能约束 `agent` 交互的形式化，不是单个图形符号，而是：

- `protocol + capabilities + invariants`

如果只能先选一个主抽象，就选：

- `state machine as protocol`

然后再在外面叠一层：

- `bounded effects`
