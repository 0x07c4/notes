# OpenAI Codex Harness 与 Solo 笔记

## Metadata

| Field | Value |
| --- | --- |
| Language | `zh-CN` |
| Type | `research-note` |
| Status | `active` |
| Summary | 从 OpenAI 对 Codex harness 和 App Server 的描述里，提炼出对 Solo 有用的 runtime、事件流和审批结构判断。 |
| Topics | `Codex`, `agent runtime`, `Solo`, `app server` |
| Keywords | `Codex`, `harness`, `App Server`, `thread`, `turn`, `item`, `approval`, `event stream` |
| Related | [solo-runtime-boundary.md](./solo-runtime-boundary.md), [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md), [agent-interaction-formalism.md](./agent-interaction-formalism.md) |
| Last Updated | `2026-03-29` |

## 一句话判断

- OpenAI 真正在做的，不是“再包一层聊天 UI”，而是把 agent runtime 做成一个可复用的 `App Server`。
- `Thread -> Turn -> Item` 不是文档上的抽象，而是 Codex 协议和实现里的真实一等对象。
- Solo 现在已经有 `方向卡 / 预览卡 / 用户确认` 的产品骨架，但底层状态模型仍然偏 `ChatMessage + ToolProposal`，还没升级成统一事件流。

## 1. 两篇文章讲的重点

### 1.1 Harness engineering: leveraging Codex in an agent-first world

发布时间：2026-02-11  
链接：https://openai.com/index/harness-engineering/

核心结论：

- OpenAI 把工程重点从“人亲自写代码”转向“设计 agent 能稳定工作的环境、工具、文档和反馈回路”。
- `harness` 的价值不在单次 prompt，而在整套工程外壳：工具接入、文档组织、可观测性、验证流程、审批流程。
- `AGENTS.md` 更适合做索引，不适合变成一份大而全的总手册。
- 高吞吐 agent 开发会改变 PR、review、合并和修复节奏，但长期架构一致性仍是问题。

### 1.2 Unlocking the Codex harness: how we built the App Server

发布时间：2026-02-04  
链接：https://openai.com/index/unlocking-the-codex-harness/

核心结论：

- 这篇更偏系统设计。重点不是模型能力，而是把 Codex 背后的 agent loop、状态和交互，封成一个稳定的 `App Server`。
- `App Server` 通过双向 JSON-RPC 暴露线程、回合、事件、审批和工具调用，让 Web、CLI、VS Code 之类的客户端复用同一套运行时。
- OpenAI 没把 MCP 当唯一主抽象，因为复杂 agent 交互不只是“调用工具”，还包括流式事件、diff、审批、线程恢复、分叉、review 等能力。

## 2. 从 Codex 源码确认到的结构

### 2.1 App Server 是正式的一层，不是临时适配层

参考：

- `codex/codex-rs/app-server/README.md`
- `codex/codex-rs/docs/codex_mcp_interface.md`

从源码和文档可以确认：

- Codex 真的有独立的 `app-server` crate，不是 CLI 里的一段薄封装。
- 对外协议明确区分：
  - `thread/start` / `thread/resume` / `thread/fork` / `thread/read` / `thread/list`
  - `turn/start` / `turn/steer` / `turn/interrupt`
  - `review/start`
  - `command/exec`
  - `fs/*`
  - `tool/requestUserInput`
- 这说明 OpenAI 把“会话生命周期”“agent 回合生命周期”“审批”“文件/命令副作用”“用户插话”都放到了统一协议里。

### 2.2 `Thread -> Turn -> Item` 是核心模型

参考：

- `codex/codex-rs/app-server/README.md`
- `codex/codex-rs/app-server-protocol/src/protocol/v2.rs`

Codex 的核心原语不是 message，而是：

- `Thread`
  - 带 `id`、`preview`、`ephemeral`、`status`、`cwd`、`source`、`git_info`
  - 可以 `resume`、`fork`、`archive`、`rollback`
- `Turn`
  - 一次具体回合
  - 可以 `start`、`steer`、`interrupt`
- `Item`
  - 真正的工作单元
  - 包括 `userMessage`、`agentMessage`、`plan`、`reasoning`、`commandExecution`、`fileChange`、`mcpToolCall`、`collabToolCall`、`enteredReviewMode`、`exitedReviewMode`

这点对 Solo 很重要，因为它意味着：

- UI 上看到的“消息”“预览卡”“命令卡”“review 卡”本质都可以只是 `Item` 的不同投影。
- 线程历史不等于聊天文本历史，而是完整的结构化事件历史。

### 2.3 Codex 把流式事件当一等能力

参考：

- `codex/codex-rs/app-server/README.md`

Codex 的事件流不是“最后给一段完整回复”，而是：

- `turn/started`
- `item/started`
- `item/.../delta`
- `item/completed`
- `turn/diff/updated`
- `turn/plan/updated`
- `turn/completed`

其中最值得 Solo 借鉴的有 3 个：

- `turn/diff/updated`
  - 给的是整轮聚合后的最新 diff，不要求客户端自己拼所有 file change
- `turn/plan/updated`
  - 计划是独立流，不需要从自然语言里硬解析
- `item/agentMessage/delta`
  - 文本流和工具流并存，而不是互相覆盖

### 2.4 审批是协议内建能力，不是 UI 侧额外补丁

参考：

- `codex/codex-rs/app-server/README.md`
- `codex/codex-rs/app-server-protocol/src/protocol/v2.rs`

Codex 里审批流是完整协议：

- 先有 `item/started`，把待执行命令或待应用改动渲染出来
- 然后 server 主动发 `requestApproval`
- 客户端返回 `accept / decline / cancel`
- server 再发 `serverRequest/resolved`
- 最终以 `item/completed` 收口

而且它不止支持单次 accept，还支持：

- `acceptForSession`
- 规则层 amendment
- network policy amendment
- 更细粒度的 `AskForApproval::Granular`

这说明 OpenAI 的心智模型不是“先聊天，再临时弹个确认框”，而是“审批本身就是 turn 内的一个状态机”。

### 2.5 运行态状态和持久化历史是分开的

参考：

- `codex/codex-rs/app-server/src/thread_state.rs`
- `codex/codex-rs/app-server-protocol/src/protocol/v2.rs`

从 `thread_state.rs` 可以确认几件事：

- Codex 为每个 thread 单独维护订阅连接、当前 turn 历史、pending interrupt、listener generation
- 当前活跃状态在内存里管理
- 持久化历史在 `thread/read` / `thread/resume` / `thread/fork` 时再读回

这个分层很关键：

- 运行时事件流负责“当前正在发生什么”
- 持久化线程负责“历史和恢复”
- 客户端连接负责“谁订阅哪些事件”

Solo 目前还没有这个分层，更多是把持久态和 UI 态直接揉在一起。

### 2.6 Codex 已经考虑了“恢复、分叉、回滚、压缩”

参考：

- `codex/codex-rs/app-server/README.md`

Codex 已经内建：

- `thread/resume`
- `thread/fork`
- `thread/rollback`
- `thread/compact/start`
- `persistExtendedHistory`

这背后的判断很清楚：

- agent 会话不是一次性聊天
- 长线程一定会越来越重
- 用户会需要“从这里分一条支线继续”
- 用户也会需要“撤回最后几轮，把上下文剪短”

这比普通聊天产品更接近真正的工作流系统。

### 2.7 客户端连接本身也是协议对象

参考：

- `codex/codex-rs/app-server/src/message_processor.rs`
- `codex/codex-rs/app-server/README.md`

Codex 会在每条连接上维护：

- 是否已 `initialize`
- 是否开启 experimental API
- 客户端名称和版本
- `optOutNotificationMethods`

这个设计很实用，因为它允许：

- 同一个 runtime 同时服务不同前端
- 不同客户端按需订阅事件
- 重流量事件按连接粒度裁剪

对 Solo 来说，这意味着以后如果有桌面端、网页端、侧边栏面板，不一定要每端各自维护一套 agent 状态机。

## 3. Solo 当前已经有的基础

参考：

- `solo/src-tauri/src/models.rs`
- `solo/src-tauri/src/lib.rs`
- `solo/src-tauri/src/storage.rs`
- `solo/src/App.jsx`
- `solo/src/components/InspectorPane.jsx`

当前 Solo 已经做出来的东西，并不弱：

- 会话有 `interaction_mode`
  - `conversation`
  - `workspaceCollaboration`
- turn intent 已经分成：
  - `auto`
  - `choice`
  - `preview`
- 模型输出已经被约束成结构化块：
  - `solo-choice`
  - `solo-write`
  - `solo-command`
- 方向卡可以被选择后继续展开成更具体的预览
- 写文件提案带：
  - `relative_path`
  - `base_hash`
  - `diff_text`
  - `next_content_preview`
- 命令提案也有单独审批和输出流
- 工作区有文件树、文件预览、附件、`.ignore` 过滤

所以 Solo 缺的不是“预览确认”这层产品表达，而是底下那层统一状态模型。

## 4. 我对 Solo 最有用的几个判断

### 4.1 不要照抄 Codex UI，要照抄它的 runtime 抽象

对 Solo 最有价值的不是把页面做成 Codex 的样子，而是把内部数据模型升级成：

- `Session/Thread`
- `Turn`
- `Item/Event`

然后把现有 UI 继续保留成这些事件的投影：

- 对话消息 = `agentMessage` / `userMessage`
- 方向卡 = `choice` 类 item 的产品投影
- 预览卡 = `fileChange` / `commandExecution` 的产品投影
- 审批动作 = `requestApproval -> resolved -> completed`

这样可以保住 Solo 现在“视觉化决策流”的外壳，又不被当前 `messages + proposals` 的数据模型卡死。

### 4.2 `ToolProposal` 不该只是附着在 session 上的列表，而该变成 turn 内 item

现在 Solo 的 `ToolProposal` 是独立存储的，和聊天消息并列存在。这个做法短期够用，但会带来几个问题：

- 同一轮里多个提案的因果顺序不清晰
- 很难表达“一个 turn 里先看代码，再给 plan，再给 diff，再请求审批”
- 恢复历史时只能拼接消息和提案，很难还原完整过程

更稳的做法是：

- 把 `write` / `command` / `choice` 都视为 `TurnItem`
- `pending_approvals` 变成 item 状态，而不是 session 上的单独数组
- Inspector 和主区卡片只是在读某些 item 子集

### 4.3 Solo 非常适合补一个 `turn/plan/updated`

Solo 现在已经有“方向建议”和“具体预览”，但中间缺一个稳定层：

- 方向太高层
- 预览太落地

中间最该补的就是“这一轮打算怎么走”的结构化 plan：

- 当前要看哪些文件
- 先确认什么
- 预计产出哪些预览卡
- 哪一步正在进行

这会比现在只靠自然语言说明更稳，也更适合做视觉化决策流。

### 4.4 Solo 也应该有整轮聚合 diff，而不是只盯单文件预览

Codex 的 `turn/diff/updated` 很值得学。

Solo 当前的 `write` 提案以单文件为中心，这没错，但一旦一个方向涉及多个文件，就会出现：

- 用户不知道“整体会改成什么样”
- 多张预览卡之间缺少整轮视角

建议补两层：

- 单文件卡：继续保留当前预览卡
- 整轮聚合 diff：给用户一眼看整体范围

这能明显提高用户做确认决策时的把握感。

### 4.5 方向选择最适合做成“分叉 turn”，而不是只改 proposal 状态

现在 Solo 的 `proposal_choose` 会把方向提案标记为 `selected`，然后自动发一条 follow-up user message 去展开预览。

这已经接近 `fork` 的感觉了，但还差一步：

- 现在只是“在原会话里继续”
- 更合理的是允许把某个方向显式变成一条新支线

对 Solo 的直接启发：

- 方向卡不一定只是“选中”
- 可以增加“沿这个方向继续”
- 后台可以把它做成新 turn，甚至以后做成 thread fork

这会让 Solo 更像“决策流”，而不是“聊天历史里插一条选择消息”。

### 4.6 `request_user_input` 很适合 Solo，而不是让模型反复追问自然语言

Codex 的 `tool/requestUserInput` 很重要，因为很多时候 agent 卡住不是因为不会做，而是缺一个结构化选择。

Solo 很适合把这能力产品化成：

- 1 到 3 个非常短的问题
- 明确互斥选项
- 直接写进当前 turn 的状态机

典型场景：

- 要改哪个文件作为第一步
- 这轮是先做方向建议还是直接出预览
- 某个命令是否允许本次会话内持续放行

这会比“模型在消息里再问一段自然语言问题”更适合协作。

### 4.7 审批流应该补齐“请求已解决”和“执行已收口”两个阶段

当前 Solo 审批已经有 `pending / approved / applied / rejected / failed`，但还缺两个非常重要的语义：

- 请求本身是否已经被用户处理
- 执行结果是否已经作为 turn 的终态收口

Codex 的流程是：

- request arrives
- request resolved
- item completed

Solo 如果补齐这层，会更容易处理：

- 用户确认后仍在运行中的命令
- 请求被取消但 turn 继续
- 一个 turn 里多个审批并发存在

### 4.8 运行时事件流和持久化展示层要拆开

我的判断：

- Solo 当前最容易被拖慢的地方，不是 UI，而是数据层会越来越难维护
- 一旦以后加入更多 item 类型，比如 plan、review、sub-agent、structured input，现在的 `messages + proposals + event listeners` 会开始发散

所以更好的路线是：

- 底层持久化 event log
- 中间层做 projection
- UI 读 projection

这样你可以继续保留：

- 主区方向卡
- 主区预览卡
- 右侧 Inspector
- 文件预览

但底下会变得稳很多。

## 5. 对 Solo 的建议路线

### Phase 1: 先重构数据模型，不重做 UI

- 给 session 引入显式 `turns`
- 给每个 turn 引入 `items`
- 把现有 `ToolProposal` 映射进 item
- 把 `ChatMessage` 也映射进 item
- UI 先继续读 projection，不强迫前端一次性重写

目标：

- 保住当前产品表达
- 先解决数据模型和因果顺序问题

### Phase 2: 补 plan、聚合 diff、审批状态机

- 增加 `plan` item
- 增加 turn 级聚合 diff
- 审批增加 `request -> resolved -> completed`
- 命令和写文件都收口到统一 item 生命周期

目标：

- 让“建议 -> 预览 -> 确认 -> 执行”真正变成一个连续 turn

### Phase 3: 补 branch / rollback / review

- 方向卡支持继续成新支线
- 会话支持回滚到前几轮
- review 作为独立 mode，而不是普通 assistant reply

目标：

- 把 Solo 从“有预览卡的聊天 UI”推到“真正可操作的协作 runtime”

### Phase 4: 再考虑更重的能力

- request_user_input
- 更细粒度权限策略
- 多客户端订阅
- 实时音频或更复杂的 transport

这些都不是第一优先级，不要过早把 Solo 做成沉重的伪 IDE。

## 6. 一个明确的边界

我认为 Solo 不该照抄 Codex 的地方也很清楚：

- 不要把底层 item 直接原样暴露给用户
- 不要先上复杂 sub-agent 协作再补基础状态模型
- 不要让审批退化成“自动执行 + 事后通知”

Solo 的优势仍然应该是：

- 对话
- 工作区协作
- 方向卡
- 预览卡
- 用户确认

Codex 值得借鉴的是 runtime 和协议，不是把产品气质做成另一个 Codex。

## 7. 参考入口

### OpenAI 文章

- https://openai.com/index/harness-engineering/
- https://openai.com/index/unlocking-the-codex-harness/

### Codex 源码

- `codex/codex-rs/app-server/README.md`
- `codex/codex-rs/docs/codex_mcp_interface.md`
- `codex/codex-rs/app-server-protocol/src/protocol/v2.rs`
- `codex/codex-rs/app-server/src/thread_state.rs`
- `codex/codex-rs/app-server/src/message_processor.rs`

### Solo 源码

- `solo/src-tauri/src/models.rs`
- `solo/src-tauri/src/lib.rs`
- `solo/src-tauri/src/storage.rs`
- `solo/src/App.jsx`
- `solo/src/components/InspectorPane.jsx`
