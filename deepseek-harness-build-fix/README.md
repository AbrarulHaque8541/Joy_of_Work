# DeepSeek Harness - Build Fix Documentation

A complete record of fixing the `tsdown` + `unrun` workspace resolution bug that broke the build pipeline.

---

## Quick Start

If you're here because your build is failing with errors like:

```
Error: tsdown: no packages/*/*/package.json declares the name @deepseek-ai/dsh-...
```

Then apply these two patches and rebuild:

### Patch 1 - Root `tsdown.config.ts`

```ts
// Add at the top of the file
import { resolve } from 'node:path'
import { fileURLToPath } from 'node:url'

const __dirname = fileURLToPath(new URL('.', import.meta.url))

// Change the workspace array from:
workspace: ['vendor/*', 'packages/*/*', 'apps/cli']

// To:
workspace: [
  resolve(__dirname, 'vendor/*'),
  resolve(__dirname, 'packages/*/*'),
  resolve(__dirname, 'apps/cli'),
]
```

### Patch 2 - `packages/client/tsdown.client.ts`

Find this line:

```ts
const REPOSITORY_ROOT = fileURLToPath(new URL('../..', import.meta.url))
```

Replace it with:

```ts
const REPOSITORY_ROOT = (() => {
  if (existsSync(resolvePath(process.cwd(), 'pnpm-workspace.yaml'))) {
    return process.cwd()
  }
  return fileURLToPath(new URL('../..', import.meta.url))
})()
```

### Rebuild

```bash
rm -rf node_modules/.unrun
pnpm run build
```

---

## Files in This Directory

| File | Description |
|---|---|
| `build-fix-journey.md` | Full story - what broke, what we tried, what worked |
| `002-tsdown-unrun-workspace-fix.md` | Technical postmortem with root cause analysis |

---

## What Was Fixed

The build failed because `tsdown` uses `unrun` to load its config file. `unrun` saves a bundled copy to `node_modules/.unrun/` and runs it from there. The workspace paths in the config were relative strings like `'packages/*/*'`, which resolved to `node_modules/.unrun/packages/*/*` - a path that does not exist.

The workspace scanner found zero packages. When `rolldown` tried to bundle anything, it asked about packages that weren't in the map, and got errors.

The fix uses `resolve(__dirname, ...)` to convert relative paths to absolute paths before `rolldown` sees them.

---

## Related Repositories

- [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) - the main harness repository
- [tsdown](https://github.com/tsdown/tsdown) - the bundler
- [unrun](https://github.com/unjs/unrun) - the config loader

---

## Searchable Keywords

These terms will help you find this page:

- tsdown build fails
- unrun workspace resolution
- packages not found tsdown
- rolldown crash index.mjs
- tsdown no packages declared
- pnpm workspace glob pattern
- node_modules/.unrun tsdown
- dsh build pipeline
- deepseek harness build error
