# Skill: LangChain

Canonical, full detail: [.claude/skills/langchain/SKILL.md](../../.claude/skills/langchain/SKILL.md).

Project uses **langchain 1.x** + `@langchain/core` 1.x + `zod@4` (JS/TS). Most points:

- **Build agents with `createAgent` from `langchain`** — `createReactAgent` is **deprecated**. It returns a `ReactAgent`; its compiled graph is `.graph`.
- Params: `model`, `tools`, `systemPrompt`, `responseFormat` (zod → `result.structuredResponse`), `stateSchema`, `middleware`, `name`.
- **Tools:** `tool(fn, { name, description, schema })`, `schema` is a zod v4 object; describe every field.
- **Models default to Claude `claude-opus-4-8`** (string `"anthropic:claude-opus-4-8"` or a `ChatAnthropic` instance). Needs `@langchain/anthropic` (add when wiring models). Keep skills **model-agnostic** — inject the model, never construct it inside a tool/skill.
- Messages from `@langchain/core/messages`.
- Pass `responseFormat`'s zod schema as an object literal at the call site (overload resolution).

Follow [typescript.md](./typescript.md) for TS/ESM rules; [langgraph.md](./langgraph.md) for orchestration.
