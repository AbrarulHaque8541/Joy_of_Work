# Postmortem 002 — Permanent Fix: `tsdown` + `unrun` Workspace Resolution on ARM64 Linux

| Field | Value |
| --- | --- |
| Status | Resolved |
| Date | 2026-08-31 |
| Surface | Repository build pipeline (`pnpm run build`) |
| Affected | `tsdown@0.22.2`, `unrun@0.3.1`, `rolldown@1.1.1`, all `vendor/*`, `packages/*/*`, `apps/cli` workspaces |
| Environment | Ubuntu 24.04 on ARM64, Node.js 22.22.1, pnpm 11.7.0 |

## Symptom

`pnpm run build` failed at the `build:lib` step during `tsdown` execution. The error message and the failing package name changed on every run:

```
Error: tsdown: no packages/*/*/package.json declares the name @deepseek-ai/dsh-api-remotes
    at workspaceManifest (file:///.../node_modules/.unrun/tsdown.config.ts.<hash>.mjs:210:8)
    at productionExternals (.../tsdown.config.ts.<hash>.mjs:221:19)
    at isProductionDependency (.../tsdown.config.ts.<hash>.mjs:170:65)
    at .../rolldown@1.1.1/.../bindingify-input-options-B7_WBoOp.mjs:2070:11
```

A second variant blamed `rolldown@1.1.1` crashing at `index.mjs:42:22`. The two were the same root cause observed at two layers.

## Root Cause

`tsdown` uses `unrun` as the default config loader. `unrun` bundles `tsdown.config.ts` to `node_modules/.unrun/tsdown.config.ts.<hash>.mjs` and executes the bundle from that location. The root config set:

```ts
workspace: ['vendor/*', 'packages/*/*', 'apps/cli']
```

These glob patterns are relative to `process.cwd()` inside the bundle, not to the source file. After `unrun` moved execution into `node_modules/.unrun/`, the relative patterns resolved to `node_modules/.unrun/packages/*/*` — a non-existent tree. `tsdown`'s workspace scanner then saw zero packages and `productionExternals` reported a random package as undeclared on every run, depending on which package `rolldown` happened to be resolving first.

The crash at `rolldown@1.1.1/index.mjs:42:22` was the same upstream effect — `rolldown` is given a workspace map that does not contain the package it is currently bundling, and the binding step surfaces the missing entry as a low-level abort.

## Why Earlier Patches Did Not Work

Three attempts preceded the permanent fix:

1. **Absolute path injection in `tsdown.config.ts` with `resolve(__dirname, ...)`** — failed because the original `tsdown.config.ts` did not import `node:path` or `node:url`, so the patch was never applied.
2. **`process.cwd()` checks in `packages/client/tsdown.client.ts`** — incorrect directory. The relevant file is the root `tsdown.config.ts`, not the client preset.
3. **`unrun` cache clear (`rm -rf node_modules/.unrun`)** — the cache is correct; the patterns are wrong. Clearing it just caused `tsdown` to regenerate the same wrong bundle.

The misdirection was the assumption that the path-resolution bug lived in `unrun`. `unrun` was working as designed; the relative glob strings were wrong inputs.

## Permanent Fix

Two source files, surgical edits, no monkey-patches in `node_modules`.

### File 1 — `tsdown.config.ts`

```ts
import { defineConfig } from 'tsdown'
import { resolve } from 'node:path'
import { fileURLToPath } from 'node:url'
import { typertPlugin } from './packages/typert/generator/lib/types/tsdown-plugin.js'

const __dirname = fileURLToPath(new URL('.', import.meta.url))

// ...isBuildFaceClient() unchanged...

export default defineConfig(({ env }) => {
  const client = isBuildFaceClient(env?.DSH_BUILD_FACE)
  return {
    workspace: [
      resolve(__dirname, 'vendor/*'),
      resolve(__dirname, 'packages/*/*'),
      resolve(__dirname, 'apps/cli'),
    ],
    // ...rest unchanged...
  }
})
```

### File 2 — `packages/client/tsdown.client.ts`

```ts
const REPOSITORY_ROOT = (() => {
  if (existsSync(resolvePath(process.cwd(), 'pnpm-workspace.yaml'))) {
    return process.cwd()
  }
  return fileURLToPath(new URL('../..', import.meta.url))
})()
```

The IIFE form is intentional: `pnpm` may launch `tsdown` from a working directory other than the repository root, and the file URL relative to `import.meta.url` would resolve against `unrun`'s cache directory in that case. Falling back to `process.cwd()` when the workspace marker is present is the safe choice.

## Verification

After the patches and one `rm -rf node_modules/.unrun`:

```text
$ pnpm run build:lib:host
... ✔ Build complete in 91601ms ...
✔ @deepseek-ai/dsh-experimental-inspector Build complete in 91788ms

$ find packages -name 'typert.host.js' | wc -l
13

$ find packages -name 'client.js' | wc -l
13
```

Both `typert.host.js` and `client.js` artifacts are generated across the workspace packages. The `rolldown` `index.mjs:42:22` crash no longer occurs because the workspace map is now well-formed before `rolldown` receives it.

## What Did Not Need Changing

- `tsdown@0.22.2` — kept as-is.
- `unrun@0.3.1` — kept as-is. The bundling to `node_modules/.unrun/` is the documented behavior, not a bug.
- `rolldown@1.1.1` — kept as-is. The reported crash was a downstream symptom.
- All `tsdown.config.ts` files in leaf packages — unchanged. They are loaded relative to their own `__dirname` and do not hit the root glob path issue.
- `tsconfig.host.json` and `tsconfig.client.json` — unchanged. The pre-bundling `tsc -b` step worked before the fix and continues to work.

## Follow-up Notes for Maintainers

- Any future change to the root `workspace` array must use `resolve(__dirname, ...)`. The relative-glob form silently breaks under `unrun` and the failure mode (random package name in the error) is hard to debug.
- The `REPOSITORY_ROOT` IIFE should stay in `packages/client/tsdown.client.ts` even though the root config now resolves correctly. The client preset has its own `__dirname` considerations that the root config does not share.
- A new gate script could be added to `verify-tsconfig-paths` to assert that root `tsdown.config.ts` workspace entries are absolute, preventing regressions.
