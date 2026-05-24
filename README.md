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


---

## Part of the Libre Open-Source Stack for Claude Code

This repository is part of a growing family of open-source toolkits for Claude Code.

### Libre suite — comprehensive plugin bundles

- [LibreUIUX-Claude-Code](https://github.com/HermeticOrmus/LibreUIUX-Claude-Code) — UI/UX development (152 agents, 70 plugins, 76 commands, 74 skills)
- [LibreArch-Claude-Code](https://github.com/HermeticOrmus/LibreArch-Claude-Code) — Software architecture and system design
- [LibreCopy-Claude-Code](https://github.com/HermeticOrmus/LibreCopy-Claude-Code) — Technical writing and documentation engineering
- [LibreDevOps-Claude-Code](https://github.com/HermeticOrmus/LibreDevOps-Claude-Code) — DevOps engineering and infrastructure automation
- [LibreEmbed-Claude-Code](https://github.com/HermeticOrmus/LibreEmbed-Claude-Code) — Embedded systems, firmware, and IoT development
- [LibreFinTech-Claude-Code](https://github.com/HermeticOrmus/LibreFinTech-Claude-Code) — Financial technology development
- [LibreGEO-Claude-Code](https://github.com/HermeticOrmus/LibreGEO-Claude-Code) — AI-search optimization (ChatGPT, Perplexity, Gemini, Google AI Overviews)
- [LibreGameDev-Claude-Code](https://github.com/HermeticOrmus/LibreGameDev-Claude-Code) — Game development across Godot, Unity, Unreal
- [LibreMLOps-Claude-Code](https://github.com/HermeticOrmus/LibreMLOps-Claude-Code) — ML engineering and AI operations
- [LibreMobileDev-Claude-Code](https://github.com/HermeticOrmus/LibreMobileDev-Claude-Code) — Mobile app development (Flutter, React Native, native iOS, native Android)
- [LibreSecOps-Claude-Code](https://github.com/HermeticOrmus/LibreSecOps-Claude-Code) — Security operations

### Skills mini-repos — single CLAUDE.md drop-ins

- [vibe-engineer-skills](https://github.com/HermeticOrmus/vibe-engineer-skills) — Direct AI codegen well (hypothesis → scope → validate → reject working-but-wrong)
- [markdown-discipline-skills](https://github.com/HermeticOrmus/markdown-discipline-skills) — Strip AI-slop from markdown (no em dashes, no marketing fluff)
- [shell-safety-skills](https://github.com/HermeticOrmus/shell-safety-skills) — `set -euo pipefail` discipline + 15 failure-mode examples
- [commit-standard-skills](https://github.com/HermeticOrmus/commit-standard-skills) — Ormus Commit Standard v1.0 + commit-msg hook + commitlint
- [unwoke-skills](https://github.com/HermeticOrmus/unwoke-skills) — Strip AI theater (ten sins to eliminate, symmetric engagement)
- [python-conventions-skills](https://github.com/HermeticOrmus/python-conventions-skills) — Modern Python 3.11+ (types, pathlib, async, ruff, mypy, uv)
- [hermetic-laws-skills](https://github.com/HermeticOrmus/hermetic-laws-skills) — Seven Hermetic Principles applied to engineering
- [riper-workflow-skills](https://github.com/HermeticOrmus/riper-workflow-skills) — Research / Innovate / Plan / Execute / Review systematic dev
- [six-day-cycle-skills](https://github.com/HermeticOrmus/six-day-cycle-skills) — Sustainable shipping cadence with mandatory rest
- [token-optimization-skills](https://github.com/HermeticOrmus/token-optimization-skills) — Claude Code token + context optimization
- [osint-skills](https://github.com/HermeticOrmus/osint-skills) — OSINT research methodology (multi-wave investigative spiral)
- [calcinate-skills](https://github.com/HermeticOrmus/calcinate-skills) — Stage 1 of the Magnum Opus (burn project bloat)
- [claude-md-overhaul-skills](https://github.com/HermeticOrmus/claude-md-overhaul-skills) — Audit CLAUDE.md and MEMORY.md against caps
- [session-handoff-skills](https://github.com/HermeticOrmus/session-handoff-skills) — Session handoff + pickup discipline
- [naming-skills](https://github.com/HermeticOrmus/naming-skills) — Product naming methodology (mine the brand's vocabulary)
- [magnum-opus-skills](https://github.com/HermeticOrmus/magnum-opus-skills) — Seven-stage alchemy applied to project transformation

### Template source

- [andrej-karpathy-skills](https://github.com/HermeticOrmus/andrej-karpathy-skills) — the canonical single-file CLAUDE.md pattern (fork of jiayuan_jy's original)

Star the family, not just one — that's how the suite stays coherent.
