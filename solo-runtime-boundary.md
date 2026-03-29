# Solo Runtime Boundary

## Metadata

| Field | Value |
| --- | --- |
| Language | `zh-CN` |
| Type | `boundary-note` |
| Status | `active` |
| Summary | 说明 Solo 应该自己掌握哪些 runtime 语义，哪些部分只该作为 provider 接入层或 projection 细节存在。 |
| Topics | `Solo`, `runtime boundary`, `provider adapter`, `projection` |
| Keywords | `runtime boundary`, `adapter`, `provider`, `projection`, `codex_cli`, `approval` |
| Related | [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md), [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md) |
| Last Updated | `2026-03-29` |

## 一句话结论

- `codex_cli` 可以继续当 access layer
- `codex_cli` 不能继续定义 Solo 的 runtime / protocol

## 长期会偏的 3 个信号

### 1. 新功能只能靠改 prompt 和解析新代码块实现

如果每加一个能力，都要靠新增一段 prompt 规则，再解析一个新的 `solo-*` 代码块，说明 Solo 的内部协议还没立起来。

短期这能跑，长期会变成：

- 能力越来越碎
- 规则越来越难维护
- provider 一换就全崩

### 2. 前端越来越依赖 `codex_cli` 的中间状态文案

如果方向卡、预览卡、审批态越来越依赖：

- `stage`
- `detail`
- `level`
- 某些 CLI 特有状态文案

那说明前端读的是 provider 细节，不是 Solo 自己的 runtime projection。

### 3. 一旦换 provider，主交互就要重写

如果未来从 `codex_cli` 换成：

- `app-server`
- OpenAI API
- 其他 provider

结果方向卡、预览卡、审批流都要跟着重写，那就说明 Solo 目前还没有真正拥有自己的运行时协议。

## 不偏的正确边界

### 1. Solo 自己定义内部原语

真正该由 Solo 定义的，是这些：

- `session/thread`
- `turn`
- `item`
- `approval state`
- `projection`

这层一旦明确，底层 provider 就只是输入来源，不该决定产品结构。

### 2. provider 只负责接入，不负责定义产品语义

provider 应该只负责：

- 接收请求
- 返回流式文本 / 状态 / 工具结果
- 被映射成 Solo 内部事件

provider 不该直接决定：

- 方向卡怎么归组
- 预览卡怎么生成
- 审批流怎么走
- 工作区协作怎么表达

### 3. UI 只读 Solo projection

主区、Inspector、审批区最终都应该读 Solo 自己投影出来的数据，而不是直接读底层 provider 的原始状态。

这样未来换底层时：

- adapter 可以变
- 主交互不用重写

## 现在该怎么做

### 短期

- 继续保留 `codex_cli`
- 不急着切 provider
- 不为了“去 CLI”而提前做大重构

### 中期

- 先把 Solo 自己的 runtime 状态机立起来
- 把 `messages + proposals` 降成 projection
- 把 turn 和 item 变成内部真相

### 长期

- 再决定是否接 `app-server`
- 或 OpenAI API
- 或多个 provider 并存

这一步应该发生在 Solo 内部协议稳定之后，而不是之前。

## 产品边界

### 1. 不照搬 Codex UI

值得借的是 runtime 和协议，不是把产品长成另一个 Codex。

Solo 的核心表达仍然应该是：

- 对话
- 工作区协作
- 建议
- 预览
- 用户确认

### 2. 不继续把 prompt hack 当长期协议

`solo-choice`、`solo-write`、`solo-command` 现在有价值，但长期只能算过渡层。

以后每加一个能力，都应该先问：

- 这是 Solo 的内部 primitive
- 还是又一层 prompt patch

如果是后者，就该警惕继续偏离。

### 3. 不让审批退化成 provider 驱动的附属功能

审批是 Solo 产品核心，不该变成某个 provider 顺手给的副作用确认框。

审批应该始终属于 Solo 自己的状态机。

## 替换底层的验收标准

只有满足下面这些，才算真正不依赖某个 provider：

- 换 provider 时，不需要重写主交互
- 只需要改 adapter，不需要改方向卡 / 预览卡 / 审批流
- 前端继续读同一套 projection
- 现有工作区协作语义不变

## 最后一句

正确路线不是“先替换 `codex_cli`，再赌 Solo 协议自然长出来”。

正确路线是：

- 先内建 Solo 协议
- 再替换底层接入层
