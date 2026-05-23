# TypeScript Conventions Skills

> A single `CLAUDE.md` for TypeScript conventions. `strict: true` always, never `any`, discriminated unions over optional fields, Result types over throw, `satisfies` over `as`. Drop into any TS project.

## What it covers

| Layer | Rule |
|---|---|
| Types | strict: true, never any, discriminated unions, type-only imports, Result types |
| Patterns | as const over enum, satisfies over as, readonly, brand types for IDs |
| Async | async/await, handle rejections, Promise.all/allSettled |
| Anti-patterns | any, @ts-ignore, unsafe assertions, mutating params, enum, silent catch |
| Tsconfig extras | noUncheckedIndexedAccess, noImplicitOverride, exactOptionalPropertyTypes |
| Tooling | tsc --noEmit, eslint + @typescript-eslint, prettier, vitest, tsx |

Full content: [`CLAUDE.md`](CLAUDE.md). Code-level before/after: [`EXAMPLES.md`](EXAMPLES.md).

## Install

### As a project CLAUDE.md

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/HermeticOrmus/typescript-conventions-skills/main/CLAUDE.md
```

### As a Claude Code skill

The same content as an installable skill: [`skills/typescript-conventions/`](skills/typescript-conventions/).

### In Cursor

See [`CURSOR.md`](CURSOR.md). Rule at [`.cursor/rules/typescript-conventions.mdc`](.cursor/rules/typescript-conventions.mdc).

## Why this exists

TypeScript has a wide range of styles. Some projects ship with `strict: false`, `any` scattered, runtime enum overhead. Others use modern idioms (discriminated unions, Result types, branded types). This file installs the modern set as the default.

AI-generated TypeScript often defaults to the looser style — `any` when the AI doesn't know the shape, `Foo as Bar` assertions, enum because that's what the corpus has. This CLAUDE.md overrides those defaults.

## Particularly opinionated takes

- **Result types over throw**. For failures the caller is supposed to handle, use `Result<T, E>` (a discriminated union: `{ ok: true, value: T } | { ok: false, error: E }`). Reserve `throw` for programmer errors and unrecoverables.
- **`as const` over `enum`**. Enums have runtime overhead and don't tree-shake well. `as const` arrays + `Literal` unions give you the same type safety with zero runtime cost.
- **`satisfies` over `as`**. `as` is a type assertion that bypasses checking; `satisfies` checks the value against the type without widening. Almost always the right choice for object literals.
- **`noUncheckedIndexedAccess: true`**. Makes `arr[0]` typed as `T | undefined`. Initially annoying, prevents a class of bugs.

If you disagree, override at the project level. These are baselines, not religion.

## See also

- [`python-conventions-skills`](https://github.com/HermeticOrmus/python-conventions-skills) — sister repo for Python
- [`shell-safety-skills`](https://github.com/HermeticOrmus/shell-safety-skills) — companion for bash
- [`andrej-karpathy-skills`](https://github.com/HermeticOrmus/andrej-karpathy-skills) — general coding discipline
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/) — book-length resource
- [Effective TypeScript](https://effectivetypescript.com/) — the canonical book

## Contributing

PRs welcome — especially React-specific patterns, Node/server patterns, Deno patterns, framework conventions (Next.js, Remix, SvelteKit, Astro), and worked migration examples (any → unknown, JS → TS).

## License

MIT.
