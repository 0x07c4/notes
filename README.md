# Notes

> A long-running notebook for ideas worth keeping.

> 一个长期生长的思考仓库，用来保存值得反复回看的判断、问题和结构。

## 中文介绍

这个仓库不是某个单一主题的资料夹，也不是一次性项目文档。

它更像一个长期笔记本：

- 记录思考
- 沉淀判断
- 收纳结构化问题
- 保留以后还会回来看的东西

这里会有 AI engineering、agent runtime、软件架构、产品边界、`Solo` 相关内容。  
以后也会有别的主题。

重点不是“现在有哪些文章”，而是把未来持续产生的思考，放进一个能积累、能回看、能继续展开的地方。

This repository is the home for ongoing thinking.

Some notes will be about AI engineering, agent runtimes, software architecture, and `Solo`.  
Some will be about other things entirely.

The point of this repo is not to freeze a topic.  
The point is to keep thoughts that are still useful after the conversation is over.

## What This Repo Is About

This is not just a snapshot of the current files.

It is intended to become a durable thinking repository:

- architecture notes
- product judgments
- engineering research
- runtime and protocol ideas
- design constraints
- conceptual models
- anything else worth preserving and revisiting

The current notes happen to cluster around four threads:

| Theme | What it covers |
| --- | --- |
| Agent Runtime | How to model `thread / turn / item`, approvals, event streams, and runtime boundaries |
| AI Engineering | What changes when models write code, and what still remains fundamentally hard |
| Formalism | How to constrain `agent` behavior with protocol, capability, and invariants |
| Solo | Product and architecture notes for a human-in-the-loop AI workspace |

## Current Entry Points

If you want the fastest path into the repository as it exists today, start here:

1. `agent-interaction-formalism.md`
   For the protocol-oriented view: `state machine + bounded effects + invariants`
2. `solo-runtime-boundary.md`
   For the boundary between Solo's own runtime and any provider-specific adapter
3. `solo-phase-1-runtime-todo.md`
   For the concrete migration path from chat-shaped state to runtime-shaped state
4. `openai-codex-harness-solo-notes.md`
   For notes connecting Codex's App Server model to Solo's architecture direction

## Current Notes

### `agent-interaction-formalism.md`

Why software architecture should constrain agents with an executable protocol instead of relying on free-form prompting.

Topics:

- `protocol + capability + invariant`
- why natural language should stop at the `intent layer`
- why `LLM` is not a silver bullet, but an `accidental-complexity compressor`

### `solo-runtime-boundary.md`

A short boundary document about what Solo should own as runtime semantics, and what should remain just an adapter concern.

Topics:

- why `codex_cli` can be an access layer but not Solo's long-term runtime
- why UI should read Solo projections, not provider-native state

### `solo-phase-1-runtime-todo.md`

A concrete implementation plan for introducing a minimal runtime model without prematurely rewriting the product surface.

Topics:

- `TurnRecord`
- `TurnItem`
- item-level approval state
- dual-write migration strategy

### `openai-codex-harness-solo-notes.md`

Notes extracted from OpenAI's Codex writing and implementation direction, with emphasis on reusable runtime structure rather than UI imitation.

Topics:

- `Thread -> Turn -> Item`
- event streams
- approval as protocol
- separation between live runtime state and persisted history

These are current entry points, not the long-term boundary of the repository.

## Working Style

These notes are intentionally:

- concise
- architectural
- biased toward reusable engineering judgments
- written mostly in Chinese, with technical terms kept in English where that is clearer

They do not need to be polished essays.  
They do need to stay worth rereading.

## License

This repository is licensed under Apache 2.0. See [`LICENSE`](./LICENSE).
