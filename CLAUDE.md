# CLAUDE.md

TypeScript conventions. Apply to every `.ts`, `.tsx`, and `tsconfig*.json`. Project-specific overrides win — these are defaults.

**Tradeoff**: bias toward strict-but-modern. For one-off scripts or legacy projects with much existing JS, use judgment.

## Type discipline

- **`strict: true` always.** If a project's tsconfig doesn't have it, enabling it is the right opening move.
- **Never `any`.** If you genuinely don't know the shape, use `unknown` and narrow.
- **Discriminated unions over optional fields** for state machines. `{ status: 'idle' } | { status: 'loading' } | { status: 'error', message: string }` beats `{ status: string, message?: string }`.
- **Type-only imports** (`import type { Foo } from '...'`) for types — keeps bundler/runtime trees clean.
- **Result types over throw** for failure modes the caller is expected to handle. Reserve `throw` for programmer errors and unrecoverables.

## Patterns

- Prefer `as const` over `enum` for string-literal sets — better tree-shaking, no runtime cost
- Prefer `satisfies` over type assertion (`as`) for object literals — preserves narrowing while validating shape
- `readonly` on arrays and props that shouldn't mutate — catches a class of bugs at zero cost
- Brand types for IDs (`type UserId = string & { readonly __brand: unique symbol }`) when mixing IDs is a real risk

## Async

- `async/await` over raw `.then()` chains
- Always handle the rejection path — never `await fn()` without a try/catch or a result-type wrapper at API boundaries
- `Promise.all` for parallel; `Promise.allSettled` when partial success is meaningful
- Never `Promise.race` without a timeout — accidental dropped work

## Anti-patterns

- `any` (use `unknown` + narrow)
- `// @ts-ignore` (use `// @ts-expect-error` so it errors when the issue is fixed)
- Unsafe type assertions (`x as Foo`) where `satisfies` or a guard would work
- Mutating function parameters
- `enum` (use union literals + `as const`)
- Catching errors and swallowing them silently (`} catch {}`)

## Project setup

Standard `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true
  }
}
```

The extras beyond `strict: true`:

- `noUncheckedIndexedAccess` — `arr[0]` is `T | undefined`, not `T`. Catches off-by-one + missing-key bugs.
- `noImplicitOverride` — must write `override` explicitly. Catches accidental override of unrelated methods.
- `exactOptionalPropertyTypes` — `{ x?: string }` does NOT accept `{ x: undefined }`. Distinguishes "not provided" from "provided as undefined".

## Tooling

- **`tsc --noEmit`** as the type-check pass in CI
- **`eslint`** with `@typescript-eslint` for linting
- **`prettier`** for formatting (or eslint's formatting rules if you prefer one tool)
- **`vitest`** or `jest` for tests; vitest is faster + ESM-native
- **`tsx`** for running TypeScript directly without a build step

---

**License**: MIT.
