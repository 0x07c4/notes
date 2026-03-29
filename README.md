# Notes

> Working notes on AI engineering, agent runtimes, and the architecture direction behind `Solo`.

This repository collects a small set of opinionated notes about building software in the LLM era.

It is not a general knowledge base.  
It is a focused record of architectural judgments, runtime boundaries, protocol ideas, and design constraints that are likely to matter again.

## What This Repo Is About

The current notes cluster around four threads:

| Theme | What it covers |
| --- | --- |
| Agent Runtime | How to model `thread / turn / item`, approvals, event streams, and runtime boundaries |
| AI Engineering | What changes when models write code, and what still remains fundamentally hard |
| Formalism | How to constrain `agent` behavior with protocol, capability, and invariants |
| Solo | Product and architecture notes for a human-in-the-loop AI workspace |

## Reading Map

If you want the fastest path through the repository, start here:

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

## Working Style

These notes are intentionally:

- concise
- architectural
- biased toward reusable engineering judgments
- written mostly in Chinese, with technical terms kept in English where that is clearer

They are not meant to be polished essays.  
They are meant to be useful.

## License

This repository is licensed under Apache 2.0. See [`LICENSE`](./LICENSE).
