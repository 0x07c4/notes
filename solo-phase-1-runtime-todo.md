# Solo Phase 1 Runtime TODO

## 目标

Phase 1 不重做 UI，不替换 `codex_cli`，只做一件事：

- 在 Solo 内部建立最小可用的 `thread/session -> turn -> item` 运行时结构
- 保留现有 `方向卡 / 预览卡 / 用户确认` 交互
- 让 `messages + proposals` 退到 projection，而不是继续当底层真相

一句话边界：

- `codex_cli` 现在可以继续当接入层
- 但不能继续当 Solo 的长期 runtime 协议层

## 当前结论

### 1. 现在没有明显偏离，但继续这样下去会偏

当前接法是：

- 默认 provider 是 `codex_cli`，见 `solo/src-tauri/src/models.rs`
- 启动时还会强制把 provider 补回 `codex_cli`，见 `solo/src/App.jsx`
- Rust 侧通过 `codex exec --sandbox read-only --json` 起一个子进程拿流式输出，见 `solo/src-tauri/src/lib.rs`
- 然后把回复里的 `solo-choice` / `solo-write` / `solo-command` 代码块解析成本地 proposal，见 `solo/src-tauri/src/lib.rs`

这条路短期没问题，因为：

- 登录态现成
- 流式输出现成
- 仍然是 Solo 在掌控预览和审批

但它的长期问题也很直接：

- Solo 当前很多核心协议还活在 prompt 和文本块里
- `proposal` 是解析产物，不是 runtime 原生对象
- `turn` 的边界、事件顺序、审批状态，都没有成为一等结构

结论：

- 短期保留 `codex_cli`
- 中期把 Solo 自己的 runtime 状态机立起来
- 之后再决定是否接 `app-server`、OpenAI API、或者其他 provider

## 当前真实结构

### 1. 底层状态

当前主要状态对象：

- `Settings`
- `ChatSession`
- `ChatMessage`
- `ToolProposal`
- `Workspace`

现状问题：

- `ChatSession` 里只有 `messages` 和 `pending_approvals`，没有显式 `turns`
- `ToolProposal` 独立于消息流存在
- `pending_approvals` 挂在 session 上，不挂在具体 turn 或 item 上

对应文件：

- `solo/src-tauri/src/models.rs`

### 2. 运行流

当前工作区协作主流程大致是：

1. UI 发起 `chat_send`
2. Rust 侧拼 prompt
3. 调 `codex exec --json`
4. 收流式 token / status
5. 从最终文本里解析 `solo-*` 结构化块
6. 生成本地 `ToolProposal`
7. 用户再确认 proposal

现状问题：

- turn 边界隐式存在，但没有单独建模
- proposal 是回合后的解析副产物，不是 turn 内 item
- 命令、写文件、方向选择不是统一 item 生命周期

对应文件：

- `solo/src-tauri/src/lib.rs`

### 3. UI 侧

UI 已经有很好的投影层雏形：

- 工作区模式与对话模式分离
- `choice / preview / auto` 三种 intent
- 主区方向卡、预览卡
- Inspector 里 diff / command / context 分栏

这部分不该先推倒。

对应文件：

- `solo/src/App.jsx`
- `solo/src/components/InspectorPane.jsx`

## 主要阻塞点

### 阻塞点 1：没有显式 Turn

现在很多“这一轮”的概念都存在，但没有真实 `Turn` 结构：

- 本轮用户输入
- 本轮流式状态
- 本轮产生的 proposal
- 本轮审批
- 本轮最终结果

后果：

- 无法稳定表达一轮里的事件顺序
- 很难做历史恢复、分叉、回滚

### 阻塞点 2：`ToolProposal` 不是 runtime item

当前 `write` / `command` / `choice` 都被落成 `ToolProposal`，但这更像 UI 产物，不像底层执行对象。

后果：

- proposal 和 message 是两套平行历史
- 很难表达 “先 plan，再 diff，再 approval，再 completed”

### 阻塞点 3：审批状态太扁

现在审批主要是：

- `pending`
- `approved`
- `applied`
- `rejected`
- `failed`

但缺两层关键语义：

- 请求是否已被用户处理
- item 是否已经以最终执行状态收口

这会让命令执行中、请求取消、多个审批并发时很快变乱。

### 阻塞点 4：协议还活在文本解析里

`solo-choice` / `solo-write` / `solo-command` 现在很有用，但它们仍是：

- prompt 约束
- 文本块协议
- 解析后再变成本地对象

短期可接受，长期不够稳。

## Phase 1 最小改造顺序

### Step 1：补 `TurnRecord`

先在 Rust 数据模型里新增一层最小 `TurnRecord`：

- `id`
- `session_id`
- `created_at`
- `status`
- `intent`
- `user_message_id`
- `assistant_message_id`
- `item_ids`
- `summary`

要求：

- 不替换现有 `messages`
- 不替换现有 `proposals`
- 先把它加成底层真相层，旧 UI 暂时继续读老数据

目的：

- 先让“这一轮”有可落盘、可追踪的边界

### Step 2：补 `TurnItem`

新增统一 item 模型，先只覆盖最小集合：

- `user_message`
- `agent_message`
- `choice`
- `write`
- `command`
- `plan`

要求：

- `write` / `command` / `choice` 先和现有 `ToolProposal` 双写
- UI 继续读 `ToolProposal`
- 底层开始写 `TurnItem`

目的：

- 先建立新 runtime，不强迫前端一次性切换

### Step 3：把审批状态挂到 item 上

增加 item 级审批状态，建议最小集合：

- `none`
- `requested`
- `resolved_accept`
- `resolved_reject`
- `completed`
- `failed`

要求：

- session 上的 `pending_approvals` 暂时保留，只当 projection
- 真正审批状态开始以 item 为准

目的：

- 让审批从“全局列表”变成“turn 内状态机”

### Step 4：补 turn 级 projection

从 `TurnRecord + TurnItem` 投影出旧 UI 继续使用的数据：

- `ChatMessage`
- `ToolProposal`
- `pending_approvals`

要求：

- 先只做单向 projection
- 不要一开始就删老结构

目的：

- 低风险迁移
- 让 UI 基本不动

### Step 5：补最小 plan item

这一阶段不要上复杂计划系统，只补一个极简 `plan` item：

- 本轮目标
- 当前步骤
- 下一步

来源可以先是：

- prompt 约束下的结构化文本
- 或后端显式生成的最小计划对象

目的：

- 在 `choice` 和 `preview` 之间补一层稳定的中间态

## 第一批具体切入口

### 1. 先把 `ChatSession` 固定为 thread

Phase 1 不要改这个判断：

- `ChatSession == thread`
- 先补 `turn`
- 不先重写 `sessions / workspaces / approvals` 三块存储

这样最稳，因为现在真正的根容器已经是 session，只是缺 turn 和 item。

### 2. 优先补 `turn_id`

第一批真正要加的，不是完整 event sourcing，而是 `turn_id`。

建议最小落点：

- `ChatMessage` 增加 `turn_id`
- `ToolProposal` 增加 `turn_id`
- 新增 `TurnRecord`

然后把下面几个入口统一打上 `turn_id`：

- `chat_send`
- `proposal_choose`
- `manual_import_assistant_reply`
- `create_reply_proposals`
- `create_write_proposal`
- `create_command_proposal`

这样先解决“消息和 proposal 无法稳定归组”的问题。

### 3. 把“选方向 -> 展开预览”改成显式 turn transition

现在这条链路本质上是：

- 把 choice proposal 标成 `selected`
- 塞一条伪 user message
- 再跑一轮 `process_chat_turn`

Phase 1 不一定要立刻做 thread fork，但至少要改成：

- 创建一个新 `TurnRecord`
- 把“已选择方向，展开具体预览”挂到这个 turn
- 让后续 preview proposal 都归到这个 turn

这一步非常关键，因为它能把 Solo 最核心的“方向 -> 预览”链条先变成稳定 runtime。

### 4. 统一一层 item runtime 事件，但保留旧事件

现在不要立刻删这些旧事件：

- `chat-stream-status`
- `chat-stream-token`
- `chat-stream-done`
- `approval-updated`
- `command-output`
- `command-finished`

更稳的做法是：

- 在同一批 emit 点上，额外发一层统一的 `turn-item-updated`
- 旧 UI 继续吃旧事件
- 新 runtime 逐步改成吃新事件

这样不会一上来把 `App.jsx` 的状态拼接全打碎。

### 5. 前端先加 adapter，不先重写页面

现在前端已经有足够好的投影层，不该先推翻。

Phase 1 的前端目标只应该是：

- 开始按 `turn_id` 聚合方向卡和预览卡
- 开始让 `pending_approvals` 退成 projection
- 不重做主区和 Inspector 结构

换句话说：

- 先改数据读取方式
- 不先改页面表达方式

## Phase 1 暂不做

这些都先别碰：

- 替换 `codex_cli`
- 接 `codex app-server`
- thread fork / rollback / compact
- sub-agent / collab tool call
- realtime audio
- 更复杂的权限策略
- 多客户端订阅

原因：

- 当前最缺的不是 transport，而是 Solo 自己的 runtime 结构
- 现在先补这些，只会把系统变重

## 建议改造位置

### 优先改

- `solo/src-tauri/src/models.rs`
  - 加 `TurnRecord` / `TurnItem` / item 级审批状态
- `solo/src-tauri/src/storage.rs`
  - 加 turns/items 的持久化与 projection
- `solo/src-tauri/src/lib.rs`
  - 改 chat turn 流，把 proposal 生成并入 turn item 流

### 尽量少改

- `solo/src/App.jsx`
- `solo/src/components/InspectorPane.jsx`

原则：

- Phase 1 不做 UI 大改
- 只在 UI 需要的地方补从新模型读取 projection 的桥接

## 验收标准

做到下面几点，就算 Phase 1 成功：

- 每次 `chat_send` 都会产生一个明确 turn
- 每条 proposal 都能追溯到具体 turn
- 审批状态能挂回具体 item，而不是只靠 session 列表
- 旧 UI 基本不需要重写
- `codex_cli` 仍可继续工作，但不再定义 Solo 的底层状态模型

## 后续 Phase 2

Phase 2 再做这些：

- 聚合 diff
- 更正式的 plan 流
- 方向选择 -> 新 turn / 支线
- review mode
- rollback

但这些都建立在 Phase 1 先把 runtime 骨架立起来。
