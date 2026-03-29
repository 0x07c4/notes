# Agent 交互约束的形式化方法

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

## 最后一句

适合软件架构、且能约束 `agent` 交互的形式化，不是单个图形符号，而是：

- `protocol + capabilities + invariants`

如果只能先选一个主抽象，就选：

- `state machine as protocol`

然后再在外面叠一层：

- `bounded effects`
