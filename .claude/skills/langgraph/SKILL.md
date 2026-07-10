---
name: langgraph
description: >-
  How to build LangGraph workflows in this project (@langchain/langgraph 1.x, JS/TS).
  Use when creating StateGraph pipelines, nodes, edges, conditional routing,
  state channels, or embedding agents as subgraph nodes. Pairs with the langchain skill.
---

# Writing LangGraph code in this project

Installed: **`@langchain/langgraph@1.x`** with `@langchain/core@1.x` and `zod@4`.
This is LangGraph **1.x** for JavaScript/TypeScript.

Follow the [typescript](../typescript/SKILL.md) skill for TS/ESM rules and the
[langchain](../langchain/SKILL.md) skill for agents/tools/models.

## Core building blocks

- **`StateGraph`** — the graph. Construct with a state schema (an `Annotation`
  root or a zod schema).
- **`Annotation` / `MessagesAnnotation`** — define state channels. Use the
  prebuilt `MessagesAnnotation` for chat-style graphs (a `messages` channel with
  the `addMessages` reducer).
- **`START` / `END`** — entry/exit sentinels for edges.
- **`Command`, `Send`, `interrupt`** — dynamic routing, fan-out, human-in-the-loop.

Import everything from `@langchain/langgraph`:
```ts
import { StateGraph, MessagesAnnotation, START, END } from '@langchain/langgraph';
```

## A minimal graph

```ts
const graph = new StateGraph(MessagesAnnotation)
  .addNode('respond', async (state) => {
    // state.messages is available; return a partial state update
    return { messages: [/* new messages */] };
  })
  .addEdge(START, 'respond')
  .addEdge('respond', END)
  .compile();

const result = await graph.invoke({ messages: [{ role: 'user', content: 'hi' }] });
```

Nodes are `(state) => Partial<State> | Promise<Partial<State>>`. Return only the
channels you're updating; reducers (e.g. `addMessages`) merge them.

## Embedding an agent as a subgraph node

A `createAgent` result (from the [langchain](../langchain/SKILL.md) skill) exposes
its compiled graph via `.graph`, which drops straight into a parent graph:

```ts
import { createAgent } from 'langchain';

const agent = createAgent({ model, systemPrompt, tools });
const graph = new StateGraph(MessagesAnnotation)
  .addNode('agent', agent.graph)          // subgraph node
  .addEdge(START, 'agent')
  .addEdge('agent', END)
  .compile();
```

Alternatively wrap `agent.invoke` in a plain node function when the parent state
differs from the agent's.

## Custom state schemas

Define channels with `Annotation` when you need more than `messages`:

```ts
import { Annotation } from '@langchain/langgraph';

const State = Annotation.Root({
  topic: Annotation<string>(),
  drafts: Annotation<string[]>({
    reducer: (a, b) => [...a, ...b],
    default: () => [],
  }),
});
new StateGraph(State).addNode(/* ... */);
```

zod schemas are also accepted as state (LangGraph 1.x, via the `@langchain/langgraph/zod` interop) — zod is `zod@4` here.

## Conditional routing

```ts
graph.addConditionalEdges('classify', (state) =>
  state.done ? END : 'refine',
);
```

## Persistence / memory

Pass a checkpointer to `.compile({ checkpointer })` (e.g. `MemorySaver` from
`@langchain/langgraph`) and invoke with a `thread_id` in config to persist state
across turns.

## Gotchas

- `.compile()` returns the runnable graph — call graph methods (`invoke`,
  `stream`) on the compiled object, not the builder.
- Node names are strings and must be unique; the agent-as-subgraph node name is
  what you pass to `addNode`, independent of the agent's own `name`.
- Keep node functions pure and side-effect-light so the graph is testable with a
  fake model (see the [typescript](../typescript/SKILL.md) skill's testing notes).
