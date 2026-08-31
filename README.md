# Joy of Work

Documentation and fixes for developer tooling setup.

---

## Projects

### DeepSeek Harness Build Fix

Fixed the `tsdown` + `unrun` workspace resolution bug that broke the build pipeline.

**Problem:** Build failed with errors about missing package declarations, even though all packages existed.

**Root Cause:** `unrun` runs the config from `node_modules/.unrun/`, so relative glob patterns like `'packages/*/*'` resolve to wrong paths.

**Fix:** Use `resolve(__dirname, ...)` for absolute paths.

See [deepseek-harness-build-fix/](deepseek-harness-build-fix/) for full documentation.
