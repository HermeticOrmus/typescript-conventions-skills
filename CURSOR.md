# Using this repo with Cursor

This project includes a Cursor project rule scoped to TypeScript files.

## In this repository

1. Open the folder in Cursor.
2. The rule [`.cursor/rules/typescript-conventions.mdc`](.cursor/rules/typescript-conventions.mdc) is scoped to `**/*.ts`, `**/*.tsx`, `**/tsconfig*.json`.
3. Confirm under **Settings → Rules**.

## Use the same discipline elsewhere

**Cursor**: Copy `.cursor/rules/typescript-conventions.mdc` into that project's `.cursor/rules/`.

**Other AI tools**: Copy [`CLAUDE.md`](CLAUDE.md) to the project root.

## Pair with `tsc --noEmit` in CI

```yaml
- run: tsc --noEmit
- run: eslint .
```
