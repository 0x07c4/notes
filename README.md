# Notes

<p align="center">
  <a href="./README.md">简体中文</a> · <a href="./README.en.md">English</a>
</p>

<p align="center">
  <img alt="Language" src="https://img.shields.io/badge/language-Simplified%20Chinese%20%2F%20English-111827?style=flat-square">
  <img alt="Type" src="https://img.shields.io/badge/type-knowledge%20entry%20%2B%20long--running%20notebook-0f766e?style=flat-square">
  <img alt="Focus" src="https://img.shields.io/badge/focus-personal%20thinking%20%7C%20architecture%20%7C%20Solo-7c3aed?style=flat-square">
</p>

<p align="center">
  一个面向人和 AI 检索的长期思考入口，用来保存值得反复回看的判断、问题和结构。
</p>

## 入口说明

这个仓库不只是笔记堆放处，更是一个知识入口。

后面不管主题怎么扩展，它都应该能承担两件事：

- 给人一个能快速理解和导航的入口
- 给 AI 一个能检索、定位、串联个人思考的入口

所以这里不会只强调“展示”，而会强调：

- 主题索引
- 文件入口
- 检索线索
- 稳定命名
- 可持续扩展

## 仓库画像

这个仓库依然是一个会持续生长的 notebook，只是它现在更明确地朝“知识库入口”方向组织。

| 维度 | 说明 |
| --- | --- |
| 定位 | 个人思考的知识入口 |
| 形态 | 笔记、研究记录、架构判断、概念模型 |
| 导航层 | `README` 负责总入口，`INDEX.md` 负责主题索引 |
| 当前焦点 | AI engineering、agent runtime、formalism、`Solo` |
| 长期范围 | 不设主题上限，只保留值得回看和检索的内容 |

## 怎么使用这个仓库

如果你是第一次进入这个仓库：

1. 先看这个 `README`
2. 再看 [INDEX.md](./INDEX.md)
3. 然后按主题或文件继续往下钻

如果你是来检索某个想法：

- 先看主题索引
- 再看文件名和关键词
- 最后进入具体笔记

如果以后这个仓库接到 AI 检索链路，优先入口也应该是：

- `README.md`
- `INDEX.md`
- 具体主题文件

## 检索提示

为了让后续检索更稳，这个仓库会尽量保持这些约束：

- 一个文件尽量围绕一个清晰主题
- 文件名尽量直接反映主题
- 标题尽量直接说结论或问题
- 笔记头部尽量保持统一 `Metadata` 结构
- 重要文件之间保持显式链接
- 索引文件优先放在仓库根目录
- 新笔记优先从 [NOTE_TEMPLATE.md](./NOTE_TEMPLATE.md) 开始

## 当前主题

当前内容刚好比较集中在四条线上：

| 主题 | 内容 |
| --- | --- |
| Agent Runtime | `thread / turn / item`、审批、事件流、运行时边界 |
| AI Engineering | 模型写代码之后，什么变了，什么依然难 |
| Formalism | 如何用协议、能力边界和不变量约束 `agent` |
| Solo | 面向 human-in-the-loop 的 AI 工作台相关判断 |

这些只是当前聚类，不是这个仓库未来的边界。

## 主题索引

更完整的当前主题入口见 [INDEX.md](./INDEX.md)。

如果你想快速进入现在的内容，建议从这里开始：

1. [agent-interaction-formalism.md](./agent-interaction-formalism.md)
   `state machine + bounded effects + invariants`
2. [solo-runtime-boundary.md](./solo-runtime-boundary.md)
   Solo 自己的 runtime 和 provider adapter 的边界
3. [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md)
   从 chat-shaped state 迁移到 runtime-shaped state 的最小路径
4. [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md)
   Codex App Server 方向和 Solo 架构的连接点

## 当前入口文件

| 文件 | 作用 | 关键词 |
| --- | --- |
| [INDEX.md](./INDEX.md) | 当前知识图谱和主题入口 | index, map, topics, retrieval |
| [NOTE_TEMPLATE.md](./NOTE_TEMPLATE.md) | 新笔记的统一起点模板 | template, metadata, structure |
| [agent-interaction-formalism.md](./agent-interaction-formalism.md) | 用协议、能力和不变量约束 `agent` 交互 | protocol, capability, invariant, formalism |
| [solo-runtime-boundary.md](./solo-runtime-boundary.md) | 划清 Solo runtime 和 provider 适配层的边界 | runtime boundary, adapter, projection |
| [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md) | 给运行时改造提供一条最小实现路线 | turn, item, approval, migration |
| [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md) | 把 Codex 的 runtime 结构抽成对 Solo 有用的判断 | Codex, App Server, thread, turn, item |

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
