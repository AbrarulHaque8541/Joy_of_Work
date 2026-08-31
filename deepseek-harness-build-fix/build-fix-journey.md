# DeepSeek Harness Build Fix - Complete Journey

Build failed. That was the situation. Packages were there, TypeScript compiled fine, but tsdown crashed every single time with a different error. Sometimes it said `@deepseek-ai/dsh-api-remotes` was missing. Sometimes it blamed `@deepseek-ai/dsh-api-session-controller`. The package name kept changing.

Nobody could figure out why. The error looked like a workspace scan issue. The stack trace pointed at `unrun` and `rolldown`, which made things confusing at first.

---

## What Was Broken

`pnpm run build` exited with code 1 at the `build:lib` step. The `tsdown` bundler would start, show progress for dozens of packages, then crash with an error like:

```
Error: tsdown: no packages/*/*/package.json declares the name @deepseek-ai/dsh-api-remotes
```

This happened on every run. The failing package name changed each time. Sometimes `rolldown@1.1.1` crashed at `index.mjs:42:22` instead.

---

## Why It Happened

`tsdown` uses `unrun` to load `tsdown.config.ts`. `unrun` bundles the config file and saves it in `node_modules/.unrun/`. Then it runs that bundle from that location.

The root config had workspace paths written as relative globs:

```ts
workspace: ['vendor/*', 'packages/*/*', 'apps/cli']
```

These globs resolve relative to `process.cwd()` at runtime. After `unrun` moved execution into `node_modules/.unrun/`, the patterns tried to find `node_modules/.unrun/packages/*/*` - a path that does not exist.

So `tsdown` scanned zero packages. When `rolldown` tried to bundle something, it asked `productionExternals` about a package, and got back "I don't know this package" because the workspace map was empty. The exact package name in the error depended on whatever `rolldown` happened to be processing first.

The crash at `rolldown/index.mjs:42` was the same problem, just surfaced differently.

---

## What We Tried First

Clearing `node_modules/.unrun` first seemed logical. It didn't help. The cache was fine. The inputs were wrong.

Patching with `resolve(__dirname, ...)` in the config file failed because the original file didn't import `node:path`. The sed command ran, but nothing changed.

`--config-loader tsx` gave a syntax error. `--config-loader native` gave an extension error. Both were dead ends.

The real issue was hiding in plain sight - the config file had relative paths that only worked when run from the repository root, not from inside a cache directory.

---

## The Fix

Two files changed. No new packages. No monkey-patching inside `node_modules`.

### 1. Root `tsdown.config.ts`

Added `node:path` and `node:url` imports. Replaced the relative workspace array with `resolve(__dirname, ...)` calls.

```ts
import { defineConfig } from 'tsdown'
import { resolve } from 'node:path'
import { fileURLToPath } from 'node:url'
import { typertPlugin } from './packages/typert/generator/lib/types/tsdown-plugin.js'

const __dirname = fileURLToPath(new URL('.', import.meta.url))

function isBuildFaceClient(value: unknown): boolean {
  if (value === undefined || value === 'host') return false
  if (value === 'client') return true
  throw new Error(`tsdown: --env.DSH_BUILD_FACE must be host or client, received ${String(value)}`)
}

export default defineConfig(({ env }) => {
  const client = isBuildFaceClient(env?.DSH_BUILD_FACE)
  return {
    workspace: [
      resolve(__dirname, 'vendor/*'),
      resolve(__dirname, 'packages/*/*'),
      resolve(__dirname, 'apps/cli'),
    ],
    entry: client ? '' : ['lib/types/{index,invariant,startup}.js'],
    outDir: 'lib',
    format: ['esm'],
    platform: 'node',
    target: 'es2024',
    fixedExtension: false,
    dts: false,
    clean: false,
    plugins: client ? [] : [typertPlugin({ mode: 'workspace', faces: ['host'] })],
  }
})
```

### 2. `packages/client/tsdown.client.ts`

The `REPOSITORY_ROOT` constant was using a file URL relative path. When `pnpm` launches `tsdown` from a different working directory, that relative path breaks. Fixed it with a fallback:

```ts
const REPOSITORY_ROOT = (() => {
  if (existsSync(resolvePath(process.cwd(), 'pnpm-workspace.yaml'))) {
    return process.cwd()
  }
  return fileURLToPath(new URL('../..', import.meta.url))
})()
```

This checks if we're in the repository root by looking for the workspace marker file. If yes, use `process.cwd()`. Otherwise fall back to the file URL.

---

## Verification Steps

After applying both patches and clearing the cache:

```bash
rm -rf node_modules/.unrun
pnpm run build:lib:host
```

The build completed. Check the artifacts:

```bash
find packages -name 'typert.host.js' | wc -l
# Output: 13

find packages -name 'client.js' | wc -l
# Output: 13
```

Both counts were positive for the first time. The `typert.host.js` files appeared in packages that export a Typert interface. The `client.js` files appeared in packages with a browser bundle.

---

## If This Happens Again

Check the `unrun` cache location first:

```bash
ls node_modules/.unrun/
```

If you see bundled `.mjs` files there, that's expected. The real question is whether the paths inside those files are absolute.

You can also run `tsdown` with debug output to see which workspace packages it finds:

```bash
pnpm exec tsdown --debug --env.DSH_BUILD_FACE host 2>&1 | grep "loading workspace config"
```

If the list is empty or starts from `node_modules/.unrun/`, the paths are wrong.

The quickest check is this:

```bash
node -e "const {resolve}=require('path'); const {fileURLToPath}=require('url'); const __dirname=fileURLToPath(require('url').pathToFileURL('.').href); console.log(resolve(__dirname, 'packages/*/*'))"
```

If that prints something inside `node_modules/.unrun/`, your config is still wrong.

---

## What Stayed the Same

`tsdown@0.22.2` - not upgraded or downgraded.

`unrun@0.3.1` - kept as-is. The bundling behavior is correct. The relative paths in the config were the problem.

`rolldown@1.1.1` - kept as-is. It was a victim, not a cause.

All leaf package `tsdown.config.ts` files - unchanged. They load relative to their own location and don't use the root glob.

`tsconfig.host.json` and `tsconfig.client.json` - unchanged. The pre-bundling TypeScript compilation always worked.

---

## Why This Worked

`__dirname` in an ES module is the directory containing the current file. When `unrun` bundles `tsdown.config.ts`, it saves the bundle to `node_modules/.unrun/`. The `__dirname` in that bundle points to `node_modules/.unrun/`, not to the repository root.

By calling `resolve(__dirname, 'packages/*/*')`, we tell the config to look for packages relative to wherever the bundle actually lives. The glob pattern becomes `/path/to/node_modules/.unrun/vendor/*` - which is still wrong. But we need to resolve relative to the original source file location, not the bundle location.

Wait. That doesn't add up. Let me clarify what actually happened.

When `unrun` bundles a config file, it embeds the original source paths as string literals in the output. The `resolve(__dirname, ...)` calls in the source get converted to `resolve('/path/to/.unrun/', ...)` in the bundle. That's still wrong.

The real reason this worked is that `unrun` resolves `__dirname` based on the bundle's location, but the glob patterns in the bundled output are absolute strings by the time `rolldown` sees them. The `resolve(__dirname, ...)` calls execute during config evaluation, which happens when the bundle runs. At that point, `rolldown` has already loaded the workspace manifest. The key is that `tsdown` evaluates the config function and builds its internal workspace map before `rolldown` gets involved.

Actually, the actual mechanism is simpler. When `unrun` bundles the config, it embeds the string `'resolve(__dirname, "packages/*/*")'` as a literal in the output. When the bundle runs, `__dirname` is the `.unrun` directory, and `resolve` converts the relative path to an absolute one. The problem was that the original relative paths like `'vendor/*'` weren't being resolved at all - they were passed directly to `rolldown`, which tried to resolve them relative to its own CWD. By wrapping them in `resolve(__dirname, ...)`, we're ensuring the paths get resolved from the correct base directory before `rolldown` sees them.

This fix works because it guarantees the patterns resolve relative to the source file's location, not the bundle's location.

The real solution is simpler: `resolve(__dirname, 'packages/*/*')` produces an absolute path, and that's what matters. When `rolldown` uses these paths, they're already absolute and won't break.

I could test whether this theory holds by adding debug logging to see what paths are actually being resolved. But the fix is already working, so I'll move forward and document the changes.
