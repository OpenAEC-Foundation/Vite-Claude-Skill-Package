# Vite Claude Skill Package — Skill Index

> 22 deterministic skills for Vite 6.x, 7.x, and 8.x

## Core Skills (2)

| Skill | Description |
|-------|-------------|
| [vite-core-architecture](skills/source/vite-core/vite-core-architecture/) | Dev server vs build pipeline, internal tools evolution (esbuild→Oxc, Rollup→Rolldown), module graph, browser targets |
| [vite-core-environment-api](skills/source/vite-core/vite-core-environment-api/) | Vite 6+ Environment API, per-environment config, custom providers, shared vs per-environment plugins |

## Syntax Skills (8)

| Skill | Description |
|-------|-------------|
| [vite-syntax-config](skills/source/vite-syntax/vite-syntax-config/) | vite.config.ts, defineConfig(), conditional/async config, shared options (root, base, mode, define, plugins) |
| [vite-syntax-server](skills/source/vite-syntax/vite-syntax-server/) | All server.* options: proxy, HTTPS, CORS, HMR, fs restrictions, middleware mode, warmup |
| [vite-syntax-build](skills/source/vite-syntax/vite-syntax-build/) | All build.* options: target, rolldownOptions/rollupOptions, minify, manifest, library mode, multi-page |
| [vite-syntax-resolve-css](skills/source/vite-syntax/vite-syntax-resolve-css/) | resolve.alias, CSS modules, preprocessors (Sass/Less/Stylus), PostCSS, Lightning CSS, JSON/HTML options |
| [vite-syntax-plugin-api](skills/source/vite-syntax/vite-syntax-plugin-api/) | Complete plugin API: all hooks, ordering, virtual modules, hook filters, client-server communication |
| [vite-syntax-hmr-api](skills/source/vite-syntax/vite-syntax-hmr-api/) | HMR client API: accept/dispose/prune/invalidate, custom events, boundary concept, full reload triggers |
| [vite-syntax-assets](skills/source/vite-syntax/vite-syntax-assets/) | Static assets, ?url/?raw/?inline/?worker, glob import, JSON imports, WebAssembly, Web Workers |
| [vite-syntax-env-vars](skills/source/vite-syntax/vite-syntax-env-vars/) | import.meta.env, .env files, VITE_ prefix, modes, TypeScript IntelliSense, HTML replacement |

## Implementation Skills (7)

| Skill | Description |
|-------|-------------|
| [vite-impl-project-setup](skills/source/vite-impl/vite-impl-project-setup/) | create-vite, framework templates, TypeScript config, CLI commands, package.json scripts |
| [vite-impl-library-mode](skills/source/vite-impl/vite-impl-library-mode/) | build.lib, output formats (ES/CJS/UMD/IIFE), externals, CSS, DTS generation, package.json exports |
| [vite-impl-ssr](skills/source/vite-impl/vite-impl-ssr/) | SSR dev workflow, ssrLoadModule, externals, build commands, SSR manifest, production setup |
| [vite-impl-backend-integration](skills/source/vite-impl/vite-impl-backend-integration/) | Backend frameworks, manifest.json, dev/prod HTML, ManifestChunk traversal, React preamble |
| [vite-impl-optimization](skills/source/vite-impl/vite-impl-optimization/) | Dependency pre-bundling, optimizeDeps, monorepo linked deps, caching, browser cache |
| [vite-impl-javascript-api](skills/source/vite-impl/vite-impl-javascript-api/) | Programmatic API: createServer, build, preview, resolveConfig, mergeConfig, utilities |
| [vite-impl-migration](skills/source/vite-impl/vite-impl-migration/) | v5→v6→v7→v8 migration guides, breaking changes per version, config migration patterns |

## Error Skills (3)

| Skill | Description |
|-------|-------------|
| [vite-errors-build](skills/source/vite-errors/vite-errors-build/) | Build failures, Rolldown errors, chunk warnings, target issues, v8 BundleError, minifier problems |
| [vite-errors-dependency](skills/source/vite-errors/vite-errors-dependency/) | Pre-bundling failures, CJS/ESM issues, cache problems, monorepo deps, slow startup |
| [vite-errors-dev-server](skills/source/vite-errors/vite-errors-dev-server/) | HMR failures, proxy errors, CORS, HTTPS, fs.strict, port conflicts, WebSocket issues |

## Agent Skills (2)

| Skill | Description |
|-------|-------------|
| [vite-agents-review](skills/source/vite-agents/vite-agents-review/) | 50-check validation checklist for Vite projects: config, plugins, HMR, env vars, build, SSR, security |
| [vite-agents-project-scaffolder](skills/source/vite-agents/vite-agents-project-scaffolder/) | Generate complete Vite projects: React, Vue, Svelte, vanilla with TypeScript, env setup, framework plugins |

---

## Version Coverage

| Vite Version | Internal Tools | Coverage |
|-------------|---------------|----------|
| Vite 6.x | esbuild + Rollup | Full |
| Vite 7.x | esbuild + Rollup | Full |
| Vite 8.x | Oxc + Rolldown | Full (primary target) |
| Vite 5.x | esbuild + Rollup | Migration only |

## Skill Statistics

- **Total skills**: 22
- **Categories**: 5 (core, syntax, impl, errors, agents)
- **Reference files**: 88 (4 per skill)
- **Language**: English only
- **Quality**: All skills <500 lines, deterministic language, WebFetch-verified
