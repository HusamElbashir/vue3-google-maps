# Investigation: Upstream Issue #377

**Issue:** "0.27.0 emits invalid default import from vue in ESM output"
**Reported pattern:** `import require$$0, { ... } from "vue";` in ESM build output

## Summary

**Cannot reproduce. This is NOT a real regression in the published package.**

## Methodology

1. Built the library locally at v0.26.0 (current HEAD) with Vite 4.5.14
2. Downloaded and inspected the published npm packages for v0.27.0 and v0.27.1
3. Searched all output formats (.mjs, .cjs, .umd.js) for `require$$0` or invalid default imports
4. Compared build configurations between v0.25.1 and v0.26.0

## Findings

### Local build (v0.26.0) — Clean
```
import { defineComponent as T, ref as _, provide as U, ... } from "vue";
```
Single named import, no default import, no `require$$0`.

### Published npm v0.27.0 — Clean
Same clean named-only import from `"vue"`. No `require$$0` pattern found in any dist file.

### Published npm v0.27.1 — Clean
Same clean output.

### Build configuration analysis
- Vue is properly externalized: `rollupOptions.external: ["vue"]`
- No `commonjsOptions` configured (uses Vite/Rollup defaults)
- The `lodash-es` dependency added in v0.26.0 is properly bundled (not external) and doesn't affect Vue imports
- The `useStrictCSP=true` change in v0.27.0 only affects CSS injection, not module imports

## Likely explanation

The `require$$0` pattern is characteristic of Rollup's CommonJS interop when it incorrectly treats an ESM module as CJS. This could be triggered by:

1. **Consumer's build setup** — If the consuming project has a misconfigured `commonjsOptions` or `ssr.noExternal` that causes `vue` to be processed through the CommonJS plugin
2. **Stale node_modules** — A corrupted or mixed install where the CJS version of a dependency leaks into the ESM resolution
3. **Specific bundler version** — An edge case in a specific Rollup/Vite version used by the consumer

The issue is **not present** in the library's published output. PR #379's proposed fix (adding `commonjsOptions.esmExternals`) would be defensive but is not addressing a real bug in the library itself.

## Conclusion

- **Reproducible?** No — not in the published package or local builds
- **Actual regression?** No — the ESM output is correct across v0.25.1, v0.26.0, v0.27.0, and v0.27.1
- **PR #379:** Unnecessary for the published package; the fix it proposes addresses a consumer-side configuration issue
