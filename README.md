# Notes

[简体中文](./README.md) | [English](./README.en.md)

> 一个长期生长的思考仓库，用来保存值得反复回看的判断、问题和结构。

## 介绍

这个仓库不是某个单一主题的资料夹，也不是一次性项目文档。

它更像一个长期笔记本：

- 记录思考
- 沉淀判断
- 收纳结构化问题
- 保留以后还会回来看的东西

这里会有 AI engineering、agent runtime、软件架构、产品边界、`Solo` 相关内容。  
以后也会有别的主题。

重点不是“现在有哪些文章”，而是把未来持续产生的思考，放进一个能积累、能回看、能继续展开的地方。

## 这个仓库在做什么

它不是当前文件列表的快照。

它更像一个会持续生长的思考仓库，用来存放：

- 架构笔记
- 产品判断
- 工程研究
- runtime 和 protocol 想法
- 设计约束
- 概念模型
- 其他值得保留和回看的内容

当前内容刚好比较集中在四条线上：

| 主题 | 内容 |
| --- | --- |
| Agent Runtime | `thread / turn / item`、审批、事件流、运行时边界 |
| AI Engineering | 模型写代码之后，什么变了，什么依然难 |
| Formalism | 如何用协议、能力边界和不变量约束 `agent` |
| Solo | 面向 human-in-the-loop 的 AI 工作台相关判断 |

## 当前入口

如果你想快速进入这个仓库当前的内容，建议从这里开始：

1. `agent-interaction-formalism.md`
   看 `state machine + bounded effects + invariants` 这条线
2. `solo-runtime-boundary.md`
   看 Solo 自己的 runtime 和 provider adapter 的边界
3. `solo-phase-1-runtime-todo.md`
   看如何从 chat-shaped state 迁移到 runtime-shaped state
4. `openai-codex-harness-solo-notes.md`
   看 Codex App Server 方向和 Solo 架构的连接点

## 当前笔记

### `agent-interaction-formalism.md`

讨论为什么软件架构应该用可执行协议约束 agent，而不是依赖自由文本提示。

涉及：

- `protocol + capability + invariant`
- 为什么自然语言应该停在 `intent layer`
- 为什么 `LLM` 不是银弹，而更像 `accidental-complexity compressor`

### `solo-runtime-boundary.md`

一篇短的边界文档，讨论 Solo 应该自己拥有哪部分 runtime 语义，哪部分只该是 adapter 关心的事。

涉及：

- 为什么 `codex_cli` 可以是 access layer，但不该是 Solo 的长期 runtime
- 为什么 UI 应该读 Solo projection，而不是 provider 原生状态

### `solo-phase-1-runtime-todo.md`

一份具体实现计划，目标是在不提前重写产品表层的前提下，引入最小可用 runtime 模型。

涉及：

- `TurnRecord`
- `TurnItem`
- item 级审批状态
- dual-write 迁移策略

### `openai-codex-harness-solo-notes.md`

围绕 OpenAI Codex 写作和实现方向整理的笔记，重点是 runtime 结构，而不是 UI 模仿。

涉及：

- `Thread -> Turn -> Item`
- event streams
- approval as protocol
- live runtime state 和 persisted history 的分层

这些只是当前入口，不是这个仓库未来的边界。

## 写作风格

这些笔记会尽量保持：

- 简洁
- 结构化
- 偏架构判断
- 偏可复用工程结论
- 以中文为主，必要时保留英文技术术语

它们不一定是精修文章。  
但应该值得以后回来再看。

## License

本仓库使用 Apache 2.0 许可证，见 [`LICENSE`](./LICENSE)。
