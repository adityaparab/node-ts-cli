---
name: typescript
description: >-
  Conventions and gotchas for writing TypeScript in this project. Use whenever
  producing or editing .ts code here — covers the strict ESM/nodenext setup,
  import rules (.js specifiers, `import type`), the strict-flag traps
  (exactOptionalPropertyTypes, noUncheckedIndexedAccess), and build/test/run via yarn.
---

# Writing TypeScript in this project

This project compiles with **TypeScript 7** (the native compiler) as strict ESM.
Match the setup exactly — several of these rules produce compile errors if ignored.

## Toolchain (do not deviate)

- **Package manager: `yarn`.** Never `npm`/`pnpm`. Run all scripts with yarn.
- **Build:** `yarn build` (`tsc` → `dist/`). **Run:** `yarn start`. **Dev:** `yarn dev` (ts-node).
- **Module system:** ESM (`"type": "module"`), `module: "nodenext"`, `target: "esnext"`.
- Source lives in `src/`, output in `dist/`. Don't edit `dist/`.

## Import rules — these are enforced by the config

1. **Relative imports MUST end in `.js`**, even though the source file is `.ts`
   (`nodenext` resolution). Import the compiled name:
   ```ts
   import { foo } from './foo.js';        // ✅ resolves to ./foo.ts
   import { foo } from './foo';           // ❌ nodenext error
   ```
   Package imports (`langchain`, `@langchain/core/messages`, `zod`) take no extension.

2. **`verbatimModuleSyntax` is on — use `import type` for type-only imports.**
   Mixing a value and a type in one statement is fine only if the type is marked:
   ```ts
   import { createAgent } from 'langchain';
   import type { ReactAgent } from 'langchain';
   import { z } from 'zod';               // value import (z is used at runtime)
   import type { Skill } from './types.js';
   ```

## Strict-flag traps (all enabled)

- **`exactOptionalPropertyTypes`:** you cannot assign `undefined` to an optional
  property. Don't write `{ tools: undefined }` for `tools?: T`. Omit the key —
  build it conditionally with a spread:
  ```ts
  const params = {
    model,
    ...(opts.tools ? { tools: opts.tools } : {}),
  };
  ```
- **`noUncheckedIndexedAccess`:** indexed access is `T | undefined`. Guard array
  and record lookups (`arr[0]` is `T | undefined`) before use.
- **`strict`** (all sub-flags), **`isolatedModules`**, **`noUncheckedSideEffectImports`**
  are on. No implicit `any`; null/undefined are tracked.

## Style (see [.agents/AGENTS.md](../../../.agents/AGENTS.md))

- **SRP:** one responsibility per module/class/function. Keep functions short.
- Clean, readable, testable. Meaningful names. Modularize complex logic.
- Prefer small, single-purpose files; a barrel `index.ts` re-exports the public API.

## Testing

- Tests use the built-in **`node:test`** runner + `node:assert/strict`, named
  `*.test.ts` beside the code. Keep unit tests pure (no network/model calls).
- Run with `yarn test` (builds, then `node --test "dist/**/*.test.js"`).
- Prefer pure, injectable logic so it's testable without live services — e.g. keep
  model construction out of business logic and inject it.

## Verifying a change

After a nontrivial edit, run `yarn build` (catches every type error thanks to
strict mode) and `yarn test`. A clean `tsc` is the primary correctness gate here.
