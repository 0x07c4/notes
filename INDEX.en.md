# Index

## Metadata

| Field | Value |
| --- | --- |
| Language | `en` |
| Type | `index` |
| Status | `active` |
| Summary | The current topic map and retrieval entry for finding ideas in this repository by theme, file, and keywords. |
| Topics | `index`, `retrieval`, `knowledge entry` |
| Keywords | `index`, `topics`, `retrieval`, `map`, `keywords` |
| Related | [README.en.md](./README.en.md), [INDEX.md](./INDEX.md) |
| Last Updated | `2026-03-29` |

[Back to README](./README.en.md) | [中文索引](./INDEX.md)

> The current topic map and retrieval entry for this repository.

## How To Use This Index

This is not a complete directory listing. It is the current knowledge entry.

It is useful for three things:

1. understanding what this repository is mainly about right now
2. finding files by topic
3. providing a stable entry layer for AI retrieval or search systems

## Current Topic Map

### 1. AI Engineering

Focus:

- software engineering judgments in the LLM era
- accidental complexity vs. essential complexity
- engineering boundaries for agent systems

Related files:

- [agent-interaction-formalism.md](./agent-interaction-formalism.md)
- [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md)

Retrieval keywords:

- `AI engineering`
- `no silver bullet`
- `Dijkstra`
- `Brooks`
- `natural language programming`

### 2. Agent Runtime

Focus:

- `thread / turn / item`
- event streams
- approval
- runtime boundaries

Related files:

- [solo-runtime-boundary.md](./solo-runtime-boundary.md)
- [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md)
- [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md)

Retrieval keywords:

- `turn`
- `item`
- `approval`
- `runtime`
- `projection`
- `app server`

### 3. Formalism

Focus:

- protocolizing agent interaction
- capability / effect boundaries
- invariants
- the boundary between natural language and formal structure

Related files:

- [agent-interaction-formalism.md](./agent-interaction-formalism.md)

Retrieval keywords:

- `protocol`
- `capability`
- `effect system`
- `state machine`
- `invariant`

### 4. Solo

Focus:

- human-in-the-loop
- conversation + workspace collaboration
- the boundary between runtime and UI projection

Related files:

- [solo-runtime-boundary.md](./solo-runtime-boundary.md)
- [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md)
- [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md)

Retrieval keywords:

- `Solo`
- `workspace collaboration`
- `preview`
- `approval`
- `projection`

## File Index

| File | Core question | Keywords |
| --- | --- | --- |
| [agent-interaction-formalism.md](./agent-interaction-formalism.md) | How to constrain `agent` interaction with formal structure instead of relying only on prompts | protocol, capability, invariant, natural language |
| [solo-runtime-boundary.md](./solo-runtime-boundary.md) | Which runtime semantics Solo should own, and which concerns belong only to a provider adapter | runtime boundary, adapter, provider, projection |
| [solo-phase-1-runtime-todo.md](./solo-phase-1-runtime-todo.md) | How to evolve from the current chat-shaped state into `thread / turn / item` | turn, item, approval state, migration |
| [openai-codex-harness-solo-notes.md](./openai-codex-harness-solo-notes.md) | What Solo can learn from Codex's App Server and runtime model | Codex, harness, app server, thread, turn, item |

## Expansion Rules

If this repository keeps evolving into a retrievable personal thinking base, future content should aim to preserve:

- filenames that directly reflect the topic
- titles that state the problem or judgment clearly
- timely index updates when new themes appear
- explicit cross-links between important notes
- files that still stand on their own when read in isolation
- new notes should start from [NOTE_TEMPLATE.en.md](./NOTE_TEMPLATE.en.md)
