# Notes

<p align="center">
  <a href="./README.md">简体中文</a> · <a href="./README.en.md">English</a>
</p>

<p align="center">
  <img alt="Language" src="https://img.shields.io/badge/language-Chinese%20%2F%20English-111827?style=flat-square">
  <img alt="Type" src="https://img.shields.io/badge/type-long--running%20notebook-0f766e?style=flat-square">
  <img alt="Focus" src="https://img.shields.io/badge/focus-architecture%20%7C%20AI%20engineering%20%7C%20Solo-7c3aed?style=flat-square">
</p>

<p align="center">
  A long-running notebook for ideas worth keeping.
</p>

## Repository Snapshot

This repository is not a folder for a single topic, and it is not one-off project documentation.

It is closer to a notebook that keeps growing:

- capturing ideas
- distilling judgments
- organizing structured questions
- keeping things worth revisiting later

The point is not to freeze the repository around its current files.  
The point is to keep thoughts that remain useful after the conversation is over.

| Dimension | Description |
| --- | --- |
| Positioning | A long-running thinking repository |
| Format | Notes, research fragments, architectural judgments, conceptual models |
| Current Focus | AI engineering, agent runtime, formalism, `Solo` |
| Long-Term Scope | Open-ended, as long as the content is worth revisiting |

## Current Themes

The current notes happen to cluster around four threads:

| Theme | What it covers |
| --- | --- |
| Agent Runtime | `thread / turn / item`, approvals, event streams, and runtime boundaries |
| AI Engineering | What changes when models write code, and what still remains fundamentally hard |
| Formalism | How to constrain `agent` behavior with protocol, capability, and invariants |
| Solo | Product and architecture notes for a human-in-the-loop AI workspace |

These are just the current clusters, not the long-term boundary of the repository.

## Where To Start

If you want the fastest path into the repository as it exists today, start here:

1. [agent-interaction-formalism.md](./agent-interaction-formalism.md)
   `state machine + bounded effects + invariants`
2. [solo-runtime-boundary.md](./solo-runtime-boundary.md)
   The boundary between Solo's own runtime and any provider adapter
3. [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md)
   The smallest migration path from chat-shaped state to runtime-shaped state
4. [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md)
   The link between Codex App Server ideas and Solo's architecture direction

## Current Entry Files

| File | Why it matters |
| --- | --- |
| [agent-interaction-formalism.md](./agent-interaction-formalism.md) | Constraining `agent` interaction with protocol, capabilities, and invariants |
| [solo-runtime-boundary.md](./solo-runtime-boundary.md) | Separating Solo runtime semantics from provider integration concerns |
| [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md) | A minimal implementation path for runtime refactoring |
| [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md) | Extracting reusable runtime lessons from Codex for Solo |

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
