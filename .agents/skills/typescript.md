# Skill: TypeScript

Canonical, full detail: [.claude/skills/typescript/SKILL.md](../../.claude/skills/typescript/SKILL.md).

Rules that trip up generated code in this project (**TypeScript 7**, strict ESM):

- **Package manager is `yarn`.** Build `yarn build`, run `yarn start`, dev `yarn dev`, test `yarn test`.
- **Relative imports end in `.js`** (`nodenext`): `import { x } from './x.js'`, not `'./x'`.
- **`verbatimModuleSyntax`:** use `import type` for type-only imports.
- **`exactOptionalPropertyTypes`:** never assign `undefined` to an optional prop — omit the key (conditional spread).
- **`noUncheckedIndexedAccess`:** indexed access is `T | undefined`; guard it.
- **SRP, short functions, testable** (see [AGENTS.md](../AGENTS.md)). Tests are `node:test` files named `*.test.ts`, kept pure.
