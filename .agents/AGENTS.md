# Agent Instructions

## General Coding Guidelines
- **Clean & Professional**: Produce code that is clean, professional, readable, and highly testable.
- **Single Responsibility Principle (SRP)**: Strictly adhere to SRP. Each module, class, and function should have one and only one reason to change.
- **Cognitive Load**: Organize code well to avoid overloading the developer's cognitive abilities. Keep functions short, use meaningful names, and modularize complex logic.

## Package Management
- **Yarn**: Always use `yarn` as the package manager.
- **Node Commands**: For running node-related commands or scripts, always use `yarn`.

## Skills
Technology-specific instructions for producing code in this project. Read the
relevant skill before writing code with that technology (each links to its
canonical, fuller version under `.claude/skills/<name>/SKILL.md`):
- [TypeScript](./skills/typescript.md) — strict ESM/nodenext conventions, import rules, strict-flag traps, build/test via yarn.
- [LangChain](./skills/langchain.md) — agents (`createAgent`), tools, models (default Claude), structured output.
- [LangGraph](./skills/langgraph.md) — `StateGraph` workflows, nodes/edges, embedding agents as subgraph nodes.
