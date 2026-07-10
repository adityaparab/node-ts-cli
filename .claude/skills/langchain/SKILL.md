---
name: langchain
description: >-
  How to write LangChain code in this project (JS/TS, langchain 1.x +
  @langchain/core 1.x). Use when building agents, tools, models, or structured
  output with LangChain. Key rule: use `createAgent` from `langchain` — the old
  `createReactAgent` is deprecated. Defaults to Claude models.
---

# Writing LangChain code in this project

Installed: **`langchain@1.x`**, **`@langchain/core@1.x`**, **`zod@4`**. This is the
LangChain **1.x** API — much of the pre-1.0 material online is out of date.

Follow the [typescript](../typescript/SKILL.md) skill for TS/ESM rules (`.js`
import specifiers, `import type`, yarn). LangGraph orchestration lives in the
[langgraph](../langgraph/SKILL.md) skill.

## Agents — use `createAgent`, not `createReactAgent`

`createReactAgent` from `@langchain/langgraph/prebuilt` is **deprecated**. Build
agents with `createAgent` from the `langchain` package. It returns a `ReactAgent`
whose compiled graph is reachable via `.graph` (usable as a LangGraph subgraph).

```ts
import { createAgent, tool } from 'langchain';
import { z } from 'zod';

const getWeather = tool(
  ({ location }) => `Sunny in ${location}`,
  {
    name: 'get_weather',
    description: 'Get the weather for a location.',
    schema: z.object({ location: z.string().describe('City name') }),
  },
);

const agent = createAgent({
  model: 'anthropic:claude-opus-4-8',   // provider:model string, or a chat-model instance
  systemPrompt: 'You are a helpful assistant.',
  tools: [getWeather],
});

const result = await agent.invoke({ messages: [{ role: 'user', content: '...' }] });
```

Key `createAgent` params: `model` (string or model instance), `tools`,
`systemPrompt` (string or `SystemMessage`), `responseFormat`, `stateSchema`,
`middleware`, `name`, `description`, `checkpointer`.

## Models — default to Claude

- Requires a provider package. For Claude: **`yarn add @langchain/anthropic`**
  (not yet installed — add it when wiring models).
- **Default to `claude-opus-4-8`** unless the user names another model. Pass it
  either as a provider string `"anthropic:claude-opus-4-8"` (resolved by
  `initChatModel`) or as an instance: `new ChatAnthropic({ model: 'claude-opus-4-8' })`.
- Skills stay **model-agnostic**: never construct a model inside a tool/skill
  definition. Accept the model (string or instance) and inject it at the edge.

## Tools

- Define with `tool(fn, { name, description, schema })` from `langchain`.
- `schema` is a **zod v4** object; describe every field — the model relies on it.
- Prefer prescriptive descriptions ("Call this when …"), not just what it does.

## Structured output

Pass a zod schema as `responseFormat` on `createAgent`. The result carries a
`structuredResponse` field matching the schema:

```ts
const schema = z.object({ title: z.string(), items: z.array(z.string()) });
const agent = createAgent({ model, systemPrompt, responseFormat: schema });
const { structuredResponse } = await agent.invoke({ messages: [...] });
```

## Messages

Import message classes from `@langchain/core/messages`
(`HumanMessage`, `AIMessage`, `SystemMessage`, `ToolMessage`, `BaseMessage`).

## Gotchas

- `responseFormat` typing: pass the zod schema as an **object literal** at the
  `createAgent` call site so the zod overload resolves (a pre-typed
  `CreateAgentParams` variable defaults to the JSON-schema branch and errors).
- Under `exactOptionalPropertyTypes`, don't pass `tools: undefined` — omit the key
  (conditional spread). See the [typescript](../typescript/SKILL.md) skill.
- For provider-string models, the provider package must be installed or resolution
  throws at runtime.
