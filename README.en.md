# Notes

<p align="center">
  <a href="./README.md">简体中文</a> · <a href="./README.en.md">English</a>
</p>

<p align="center">
  <img alt="Language" src="https://img.shields.io/badge/language-Chinese%20%2F%20English-111827?style=flat-square">
  <img alt="Type" src="https://img.shields.io/badge/type-knowledge%20entry%20%2B%20long--running%20notebook-0f766e?style=flat-square">
  <img alt="Focus" src="https://img.shields.io/badge/focus-personal%20thinking%20%7C%20architecture%20%7C%20Solo-7c3aed?style=flat-square">
</p>

<p align="center">
  A long-running entry point for human and AI retrieval of personal thinking.
</p>

## Entry Notes

This repository is not just a place to store notes. It is also meant to work as a knowledge entry.

Over time it should do two jobs well:

- give humans a clear place to start and navigate
- give AI systems a stable place to retrieve, locate, and connect personal thinking

So the repository is optimized less for display, and more for:

- topic indexing
- file-level entry points
- retrieval hints
- stable naming
- long-term extensibility

## Repository Snapshot

| Dimension | Description |
| --- | --- |
| Positioning | A knowledge entry for personal thinking |
| Format | Notes, research fragments, architectural judgments, conceptual models |
| Navigation Layer | `README` for entry, `INDEX.md` for topic indexing |
| Current Focus | AI engineering, agent runtime, formalism, `Solo` |
| Long-Term Scope | Open-ended, as long as the content is worth revisiting |

## How To Use This Repository

If you are entering the repository for the first time:

1. start with this `README`
2. then read [INDEX.en.md](./INDEX.en.md)
3. then drill down by topic or file

If you are here to retrieve a specific idea:

- start from the topic map
- then use filenames and keywords
- then open the specific note

If this repository is later connected to an AI retrieval flow, the preferred top-level entry should be:

- `README.md` / `README.en.md`
- `INDEX.md` / `INDEX.en.md`
- topic files

## Retrieval Hints

To make later retrieval more stable, this repository should try to preserve:

- one clear theme per file where possible
- filenames that directly reflect the topic
- titles that state the judgment or question clearly
- a consistent `Metadata` header near the top of notes
- explicit links across important notes
- root-level index files as stable entry points
- new notes should start from [NOTE_TEMPLATE.en.md](./NOTE_TEMPLATE.en.md)

## Current Themes

The current notes happen to cluster around four threads:

| Theme | What it covers |
| --- | --- |
| Agent Runtime | `thread / turn / item`, approvals, event streams, and runtime boundaries |
| AI Engineering | What changes when models write code, and what still remains fundamentally hard |
| Formalism | How to constrain `agent` behavior with protocol, capability, and invariants |
| Solo | Product and architecture notes for a human-in-the-loop AI workspace |

These are just the current clusters, not the long-term boundary of the repository.

## Topic Index

For a fuller topic map, see [INDEX.en.md](./INDEX.en.md).

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

| File | Why it matters | Keywords |
| --- | --- |
| [INDEX.en.md](./INDEX.en.md) | The current knowledge map and topic entry layer | index, map, topics, retrieval |
| [NOTE_TEMPLATE.en.md](./NOTE_TEMPLATE.en.md) | The standard starting template for new notes | template, metadata, structure |
| [agent-interaction-formalism.md](./agent-interaction-formalism.md) | Constraining `agent` interaction with protocol, capabilities, and invariants | protocol, capability, invariant, formalism |
| [solo-runtime-boundary.md](./solo-runtime-boundary.md) | Separating Solo runtime semantics from provider integration concerns | runtime boundary, adapter, projection |
| [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md) | A minimal implementation path for runtime refactoring | turn, item, approval, migration |
| [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md) | Extracting reusable runtime lessons from Codex for Solo | Codex, App Server, thread, turn, item |

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
