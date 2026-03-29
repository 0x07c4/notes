# Agent Architecture DSL

## Metadata

| Field | Value |
| --- | --- |
| Language | `zh-CN` |
| Type | `dsl-note` |
| Status | `draft` |
| Summary | 为 agent 系统定义一套偏协议化的软件架构符号，重点表达状态、权限、副作用、审批和投影，而不是只画模块关系图。 |
| Topics | `DSL`, `formalism`, `agent runtime`, `architecture` |
| Keywords | `DSL`, `protocol`, `state machine`, `capability`, `invariant`, `projection`, `approval` |
| Related | [agent-interaction-formalism.md](./agent-interaction-formalism.md), [solo-runtime-boundary.md](./solo-runtime-boundary.md), [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md) |
| Last Updated | `2026-03-30` |

## 一句话判断

- 对 agent 系统来说，真正需要的不是“画组件图的符号”，而是“表达协议、权限和提交边界的符号”。

## 目标

这套 DSL 不是为了替代所有软件建模方法。

它只解决一个更具体的问题：

- 如何形式化表达一个 agent system 的交互协议、状态转换、副作用边界和审批语义

换句话说，它关心的不是：

- 机器有几台
- 服务怎么部署
- 模块怎么分包

它关心的是：

- 谁能做什么
- 在什么状态下能做
- 哪些动作需要审批
- 哪些规则永远不能被打破
- runtime 怎么投影到 UI / log / eval

## 设计原则

### 1. 先表达时序，再表达结构

传统架构图更擅长表达：

- 依赖关系
- 模块边界
- 部署结构

但 agent system 最容易出问题的地方其实是：

- 时序
- 提交点
- 权限边界
- 回滚点
- 审批流

所以 DSL 要优先表达协议，而不是模块。

### 2. 先表达副作用，再表达文本

agent system 的风险不在“它说了什么”，而在“它做了什么”。

所以 DSL 的核心原语必须能表达：

- `command`
- `file_change`
- `approval`
- `apply`
- `rollback`

而不是只围绕 `message`。

### 3. 自然语言可以解释，但不能定真相

DSL 里可以保留：

- label
- description
- notes

但真正决定系统语义的，应该是：

- state
- transition
- capability
- invariant

## 最小符号集合

我建议最小只保留 6 类符号。

### 1. `role`

表示参与者。

例如：

```txt
role User
role Agent
role Runtime
role Workspace
role Tool
```

### 2. `entity`

表示系统里的一等对象。

例如：

```txt
entity Thread
entity Turn
entity Item
entity Approval
```

### 3. `state`

表示可观察的运行状态。

例如：

```txt
state Idle
state Drafted
state Previewed
state Approved
state Applied
state Rejected
state Failed
```

### 4. `transition`

表示状态如何转移。

例如：

```txt
Idle      --user.request-->   Drafted
Drafted   --agent.preview-->  Previewed
Previewed --user.approve-->   Approved
Previewed --user.reject-->    Rejected
Approved  --agent.apply-->    Applied
```

### 5. `capability`

表示谁在什么条件下可以触发什么副作用。

例如：

```txt
Agent.read(path)   when path in visible_scope
Agent.write(diff)  when state == Approved
Agent.exec(cmd)    when policy.allows(cmd)
```

### 6. `invariant`

表示永远不能被破坏的规则。

例如：

```txt
always apply -> approved_before
always write -> within_scope
always approval -> tied_to_item
```

## 补充符号

如果要让 DSL 真正服务产品，而不是只服务 runtime 实现，还应该补两类。

### 7. `projection`

表示同一套 runtime 如何投影成不同视图。

例如：

```txt
projection ui(Turn)
projection log(Turn)
projection eval(Thread)
```

这层很关键，因为很多系统不是没有结构，而是 UI 直接读 provider 细节，没有独立 projection。

### 8. `effect`

如果要把 capability 再收严一点，可以把副作用种类单独抽出来。

例如：

```txt
effect ReadFile
effect WriteFile
effect ExecCommand
effect RequestNetwork
```

然后 capability 可以写成：

```txt
Agent can ExecCommand when policy.allows(cmd)
Agent can WriteFile when state == Approved
```

## 一套最小 DSL 草案

```txt
protocol WorkspaceEdit {
  roles:
    User, Agent, Runtime, Workspace

  entities:
    Thread, Turn, Item, Approval

  states:
    Idle, Drafted, Previewed, Approved, Applied, Rejected, Failed

  transitions:
    Idle      --user.request-->   Drafted
    Drafted   --agent.preview-->  Previewed
    Previewed --user.approve-->   Approved
    Previewed --user.reject-->    Rejected
    Approved  --agent.apply-->    Applied

  capabilities:
    Agent.read(path)   when path in visible_scope
    Agent.write(diff)  when state == Approved
    Agent.exec(cmd)    when policy.allows(cmd)

  invariants:
    always apply -> approved_before
    always write -> within_scope
    always approval -> tied_to_item

  projections:
    ui(Turn)
    log(Turn)
    eval(Thread)
}
```

这套东西的重点不在“好看”，而在它可以被编译成：

- runtime checks
- UI gating
- approval protocol
- eval rules

## 一套更接近 Solo 的示例

```txt
protocol SoloWorkspaceTurn {
  roles:
    User, Agent, Runtime, Workspace

  entities:
    Thread, Turn, Item, Preview, Approval

  states:
    Idle, Planning, Previewing, WaitingApproval, Applying, Completed, Failed

  transitions:
    Idle            --user.request-->        Planning
    Planning        --agent.propose-->       Previewing
    Previewing      --user.request_change--> Planning
    Previewing      --user.approve-->        WaitingApproval
    WaitingApproval --runtime.prepare-->     Applying
    Applying        --runtime.success-->     Completed
    Applying        --runtime.failure-->     Failed

  capabilities:
    Agent.read(path)      when path in workspace.scope
    Agent.propose(change) when turn.state in {Planning, Previewing}
    Runtime.apply(diff)   when turn.state == WaitingApproval
    Runtime.exec(cmd)     when turn.state == WaitingApproval and policy.allows(cmd)

  invariants:
    always apply -> preview_exists
    always apply -> approval_resolved_accept
    always command -> policy_checked
    always workspace_write -> within_scope

  projections:
    ui(turn) -> cards, preview, approval
    log(turn) -> events
    eval(thread) -> trace, violations
}
```

这版更能看出：

- 人不是“外部打断者”
- 人是协议参与方
- approval 是 commit protocol 的一部分

## 最小 `Model + Protocol` 应该怎么定义

如果现在就要开始做，不要先追求完整 DSL。

更现实的顺序是：

1. 先定最小 `model`
2. 再定最小 `protocol`
3. 最后再决定 notation 怎么写

因为 notation 可以改，model 一旦错了，后面全会漂。

### 最小 `model`

我建议最小只保留 5 个核心对象。

#### 1. `Role`

表示谁在参与协议。

最小集合：

```txt
Role ::= User | Agent | Runtime | Workspace
```

`Tool` 和 `Provider` 先不要急着做成一等对象，可以先作为 Runtime 下面的 effect target。

#### 2. `Thread`

表示一条可恢复、可分叉、可回放的长期工作链。

最小定义：

```txt
Thread ::= Turn*
```

没有 `Thread`，你就没有：

- resume
- fork
- replay
- 长期历史

#### 3. `Turn`

表示一次明确的事务边界。

最小定义：

```txt
Turn ::= {
  input,
  items,
  state
}
```

这里最关键的不是字段多少，而是：

- 系统终于有了“一轮”的真对象

没有 `Turn`，你就只能在 message 流里猜“这一轮到底是什么”。

#### 4. `Item`

表示 turn 里的原子工作单元。

最小种类：

```txt
Item ::= Message | Plan | Preview | Approval | Command | FileChange
```

为什么必须有 `Item`：

- 不然 message、proposal、command、diff 就会分散成多套平行历史
- runtime 没法统一审计和投影

#### 5. `Approval`

表示一条显式的提交决议。

最小定义：

```txt
Approval ::= {
  target_item,
  resolution
}
```

其中：

```txt
Resolution ::= Pending | Accept | Reject
```

为什么它必须单独存在：

- 因为 approval 不是 UI 小部件
- 它是 commit protocol 的一部分

### 为什么这 5 个是最小集合

这 5 个对象刚好对应 5 件系统级问题：

| 对象 | 解决的问题 |
| --- | --- |
| `Role` | 谁在参与、谁对谁负责 |
| `Thread` | 历史如何连续存在 |
| `Turn` | 一次交互的边界是什么 |
| `Item` | 这一轮里到底发生了哪些原子动作 |
| `Approval` | 哪些动作真正被允许提交 |

如果再往下砍，系统会马上退化。

比如：

- 没有 `Turn`，就会退回聊天流
- 没有 `Item`，就会退回 message + sidecar proposal
- 没有 `Approval`，就会退回 UI 弹窗

### 最小 `protocol`

有了这 5 个对象以后，最小协议其实可以压得很短。

#### Turn State

```txt
TurnState ::= Idle | Planning | Previewing | WaitingApproval | Applying | Completed | Failed
```

#### Transition

```txt
Idle            --user.request-->        Planning
Planning        --agent.propose-->       Previewing
Previewing      --user.request_change--> Planning
Previewing      --user.approve-->        WaitingApproval
Previewing      --user.reject-->         Completed
WaitingApproval --runtime.prepare-->     Applying
Applying        --runtime.success-->     Completed
Applying        --runtime.failure-->     Failed
```

这里最关键的不是名字，而是顺序关系：

- 先有 propose / preview
- 再有 approval
- 最后才有 apply

也就是：

- preview-before-commit

#### Minimal Capability Rules

```txt
Agent.read(path)      when path in workspace.scope
Agent.propose(change) when turn.state in {Planning, Previewing}
Runtime.apply(diff)   when turn.state == WaitingApproval
Runtime.exec(cmd)     when turn.state == WaitingApproval and policy.allows(cmd)
```

这一步保证：

- 生成可以先发生
- 落地必须后发生

#### Minimal Invariants

```txt
always apply -> approval_resolved_accept
always apply -> preview_exists
always write -> within_scope
always command -> policy_checked
```

这一步保证：

- 不会偷提交
- 不会越权执行

### 为什么我说要先定 `model + protocol`

因为一旦这两层定下来：

- notation 只是写法问题
- schema 只是字段展开问题
- runtime 只是执行器问题
- UI 只是 projection 问题

反过来，如果这两层没定：

- 你写再漂亮的 DSL 也只是语法糖
- 你做再完整的 UI 也只是在猜状态

## 一句收束

对 agent architecture 来说，真正的起点不是 DSL syntax，而是：

- 一组能承载事务边界的对象
- 一组能定义提交顺序的协议

## 和常见形式化方法的关系

### 和 UML 的关系

UML 适合画：

- 类关系
- 组件关系
- 时序图

但对 agent system 来说，它的问题是太分散：

- 状态在一张图
- 时序在一张图
- 权限常常不在图里
- 不变量又在别处

所以它不适合直接当主协议符号。

### 和 TLA+ / Temporal Logic 的关系

时序逻辑适合验证：

- safety
- liveness
- invariants

但它太重，不适合当团队日常的主表示。

更合理的关系是：

- DSL 负责表达系统协议
- temporal logic 负责验证关键规则

### 和 BPMN / Workflow DSL 的关系

Workflow DSL 往往擅长表达：

- 任务流
- 分支
- 人机协同节点

但它们通常不够强地表达：

- capability
- effect boundary
- runtime projection

所以它们只覆盖了一部分。

## 这套 DSL 应该怎么落地

如果真要工程化，我会让它至少能生成 4 类东西。

### 1. Runtime Guards

根据：

- `transition`
- `capability`
- `invariant`

拦截非法动作。

### 2. UI Gates

根据：

- 当前状态
- 可用 capability
- 审批状态

决定：

- 哪些按钮能点
- 该显示 preview 还是 approval

### 3. Audit / Replay Trace

把 turn 和 item 收成结构化日志，支持：

- replay
- rollback analysis
- eval

### 4. Type Definitions

把 DSL 编译成：

- TypeScript types
- Rust enums / structs

这样前后端和 runtime 才能共用一套真实协议。

## 一个现实判断

如果一个 agent 产品未来真想稳定，它迟早会长出类似东西。

区别只在于：

- 是显式设计出来
- 还是隐式散落在 prompt、UI 判断和工具 wrapper 里

后者短期更快，长期更乱。

## 当前建议

如果现在就要开始做，不要一上来追求“大而全符号系统”。

先只做这三层：

1. `state`
2. `capability`
3. `invariant`

这是最小而有用的一组。

其中：

- `state` 定合法流程
- `capability` 定合法副作用
- `invariant` 定系统底线

有了这三层，再往外长：

- projection
- effect taxonomy
- protocol compiler

## 最后一句

对 agent system 来说，最该被形式化的不是“组件长什么样”，而是：

- 哪些动作什么时候能发生

所以最有价值的架构符号不是 component notation，而是：

- protocol notation
