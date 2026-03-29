# Notes

[简体中文](./README.md) | [English](./README.en.md)

> A long-running notebook for ideas worth keeping.

## Introduction

This repository is not a folder for a single topic, and it is not one-off project documentation.

It is closer to a long-lived notebook:

- capturing ideas
- distilling judgments
- organizing structured questions
- keeping things worth revisiting later

Some notes will be about AI engineering, agent runtimes, software architecture, product boundaries, and `Solo`.  
Some will be about other things entirely.

The point is not to freeze the repository around its current files.  
The point is to keep thoughts that remain useful after the conversation is over.

## What This Repository Is For

This is not just a snapshot of the current file list.

It is meant to become a durable thinking repository for:

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
| Agent Runtime | `thread / turn / item`, approvals, event streams, and runtime boundaries |
| AI Engineering | What changes when models write code, and what still remains fundamentally hard |
| Formalism | How to constrain `agent` behavior with protocol, capability, and invariants |
| Solo | Product and architecture notes for a human-in-the-loop AI workspace |

## Current Entry Points

If you want the fastest path into the repository as it exists today, start here:

1. `agent-interaction-formalism.md`
   For the `state machine + bounded effects + invariants` line of thought
2. `solo-runtime-boundary.md`
   For the boundary between Solo's own runtime and any provider adapter
3. `solo-phase-1-runtime-todo.md`
   For the migration path from chat-shaped state to runtime-shaped state
4. `openai-codex-harness-solo-notes.md`
   For the connection between Codex App Server ideas and Solo's architecture direction

## Current Notes

### `agent-interaction-formalism.md`

Why software architecture should constrain agents with an executable protocol instead of relying on free-form prompting.

Topics:

- `protocol + capability + invariant`
- why natural language should stop at the `intent layer`
- why `LLM` is not a silver bullet, but more like an `accidental-complexity compressor`

### `solo-runtime-boundary.md`

A short boundary document about what Solo should own as runtime semantics, and what should remain an adapter concern.

Topics:

- why `codex_cli` can be an access layer but not Solo's long-term runtime
- why UI should read Solo projections rather than provider-native state

### `solo-phase-1-runtime-todo.md`

A concrete implementation plan for introducing a minimal runtime model without prematurely rewriting the product surface.

Topics:

- `TurnRecord`
- `TurnItem`
- item-level approval state
- dual-write migration strategy

### `openai-codex-harness-solo-notes.md`

Notes extracted from OpenAI's Codex direction, with emphasis on reusable runtime structure rather than UI imitation.

Topics:

- `Thread -> Turn -> Item`
- event streams
- approval as protocol
- the separation between live runtime state and persisted history

These are current entry points, not the long-term boundary of the repository.

## Writing Style

These notes aim to stay:

- concise
- structured
- architecture-oriented
- biased toward reusable engineering judgments
- mostly in Chinese, while keeping English technical terms where they are clearer

They do not need to be polished essays.  
They do need to stay worth rereading.

## License

This repository is licensed under Apache 2.0. See [`LICENSE`](./LICENSE).
