# Skill: LangGraph

Canonical, full detail: [.claude/skills/langgraph/SKILL.md](../../.claude/skills/langgraph/SKILL.md).

Project uses **@langchain/langgraph 1.x** (JS/TS) with `@langchain/core` 1.x + `zod@4`. Most points:

- Core: `StateGraph`, `Annotation` / `MessagesAnnotation`, `START` / `END`, `Command` / `Send` / `interrupt` — all from `@langchain/langgraph`.
- **Build:** `new StateGraph(schema).addNode(...).addEdge(START, ...).addEdge(..., END).compile()`; call `invoke`/`stream` on the **compiled** graph.
- **Nodes** are `(state) => Partial<State>`; return only changed channels (reducers merge).
- **Embed a LangChain agent as a subgraph node:** `graph.addNode('agent', agent.graph)` (`agent` from `createAgent`).
- **State:** `MessagesAnnotation` for chat; `Annotation.Root({...})` (or a zod schema) for custom channels with reducers/defaults.
- **Routing:** `addConditionalEdges`. **Memory:** `.compile({ checkpointer })` + `thread_id` in config.
- Keep node functions pure/injectable so graphs test with a fake model.

Follow [typescript.md](./typescript.md) for TS/ESM rules; [langchain.md](./langchain.md) for agents/tools/models.
