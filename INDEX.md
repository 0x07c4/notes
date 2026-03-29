# Index

## Metadata

| Field | Value |
| --- | --- |
| Language | `zh-CN` |
| Type | `index` |
| Status | `active` |
| Summary | 当前仓库的主题地图与检索入口，用来按主题、文件和关键词定位个人思考。 |
| Topics | `index`, `retrieval`, `knowledge entry` |
| Keywords | `index`, `topics`, `retrieval`, `map`, `keywords` |
| Related | [README.md](./README.md), [INDEX.en.md](./INDEX.en.md) |
| Last Updated | `2026-03-29` |

[返回首页](./README.md) | [English Index](./INDEX.en.md)

> 当前仓库的主题地图与检索入口。

## 怎么看这个索引

这个文件不是完整目录，而是当前知识入口。

适合三种用法：

1. 快速了解这个仓库现在主要在记什么
2. 按主题找到对应文件
3. 给 AI 或检索系统提供一层稳定入口

## 当前主题地图

### 1. AI Engineering

关注：

- LLM 时代的软件工程判断
- accidental complexity 与 essential complexity
- agent system 的工程边界

相关文件：

- [agent-interaction-formalism.md](./agent-interaction-formalism.md)
- [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md)

检索关键词：

- `AI engineering`
- `no silver bullet`
- `Dijkstra`
- `Brooks`
- `natural language programming`

### 2. Agent Runtime

关注：

- `thread / turn / item`
- event streams
- approval
- runtime boundary

相关文件：

- [solo-runtime-boundary.md](./solo-runtime-boundary.md)
- [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md)
- [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md)

检索关键词：

- `turn`
- `item`
- `approval`
- `runtime`
- `projection`
- `app server`

### 3. Formalism

关注：

- 协议化 agent 交互
- capability / effect boundary
- invariants
- 自然语言与形式化边界

相关文件：

- [agent-interaction-formalism.md](./agent-interaction-formalism.md)

检索关键词：

- `protocol`
- `capability`
- `effect system`
- `state machine`
- `invariant`

### 4. Solo

关注：

- human-in-the-loop
- 对话 + 工作区协作
- runtime 和 UI projection 的边界

相关文件：

- [solo-runtime-boundary.md](./solo-runtime-boundary.md)
- [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md)
- [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md)

检索关键词：

- `Solo`
- `workspace collaboration`
- `preview`
- `approval`
- `projection`

## 文件索引

| 文件 | 核心问题 | 关键词 |
| --- | --- | --- |
| [agent-interaction-formalism.md](./agent-interaction-formalism.md) | 如何用形式化结构约束 `agent` 交互，而不是只靠 prompt | protocol, capability, invariant, natural language |
| [solo-runtime-boundary.md](./solo-runtime-boundary.md) | Solo 应该自己拥有什么 runtime 语义，什么只该是 provider adapter | runtime boundary, adapter, provider, projection |
| [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md) | 如何从现有 chat-shaped state 演进到 `thread / turn / item` | turn, item, approval state, migration |
| [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md) | Codex 的 App Server 和 runtime 模型能给 Solo 带来什么启发 | Codex, harness, app server, thread, turn, item |

## 后续扩展原则

如果这个仓库继续往“可检索的个人思考库”发展，后续内容最好尽量满足：

- 文件名直接反映主题
- 标题直接说结论或问题
- 新主题进来时及时更新索引
- 重要笔记彼此交叉链接
- 优先让单篇文件能独立成立
- 新笔记优先从 [NOTE_TEMPLATE.md](./NOTE_TEMPLATE.md) 复制开始
