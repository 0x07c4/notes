# Notes

<p align="center">
  <a href="./README.md">简体中文</a> · <a href="./README.en.md">English</a>
</p>

<p align="center">
  <img alt="Language" src="https://img.shields.io/badge/language-Simplified%20Chinese%20%2F%20English-111827?style=flat-square">
  <img alt="Type" src="https://img.shields.io/badge/type-long--running%20notebook-0f766e?style=flat-square">
  <img alt="Focus" src="https://img.shields.io/badge/focus-architecture%20%7C%20AI%20engineering%20%7C%20Solo-7c3aed?style=flat-square">
</p>

<p align="center">
  一个长期生长的思考仓库，用来保存值得反复回看的判断、问题和结构。
</p>

## 仓库画像

这个仓库不是某个单一主题的资料夹，也不是一次性项目文档。

它更像一个会持续生长的 notebook：

- 记录思考
- 沉淀判断
- 收纳结构化问题
- 保留以后还会回来看的东西

重点不是“现在有哪些文章”，而是把未来持续产生的思考，放进一个能积累、能回看、能继续展开的地方。

| 维度 | 说明 |
| --- | --- |
| 定位 | 长期思考仓库 |
| 形态 | 笔记、研究记录、架构判断、概念模型 |
| 当前焦点 | AI engineering、agent runtime、formalism、`Solo` |
| 长期范围 | 不设主题上限，只保留值得回看的内容 |

## 当前主题

当前内容刚好比较集中在四条线上：

| 主题 | 内容 |
| --- | --- |
| Agent Runtime | `thread / turn / item`、审批、事件流、运行时边界 |
| AI Engineering | 模型写代码之后，什么变了，什么依然难 |
| Formalism | 如何用协议、能力边界和不变量约束 `agent` |
| Solo | 面向 human-in-the-loop 的 AI 工作台相关判断 |

这些只是当前聚类，不是这个仓库未来的边界。

## 从哪里开始

如果你想快速进入这个仓库现在的内容，建议从这里开始：

1. [agent-interaction-formalism.md](./agent-interaction-formalism.md)
   `state machine + bounded effects + invariants`
2. [solo-runtime-boundary.md](./solo-runtime-boundary.md)
   Solo 自己的 runtime 和 provider adapter 的边界
3. [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md)
   从 chat-shaped state 迁移到 runtime-shaped state 的最小路径
4. [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md)
   Codex App Server 方向和 Solo 架构的连接点

## 当前入口文件

| 文件 | 作用 |
| --- | --- |
| [agent-interaction-formalism.md](./agent-interaction-formalism.md) | 用协议、能力和不变量约束 `agent` 交互 |
| [solo-runtime-boundary.md](./solo-runtime-boundary.md) | 划清 Solo runtime 和 provider 适配层的边界 |
| [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md) | 给运行时改造提供一条最小实现路线 |
| [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md) | 把 Codex 的 runtime 结构抽成对 Solo 有用的判断 |

## 写作方式

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
