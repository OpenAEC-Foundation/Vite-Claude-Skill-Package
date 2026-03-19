# Vite Skill Package — Definitive Masterplan

## Status

Phase 3 complete. Finalized from raw masterplan after vooronderzoek review.
Date: 2026-03-19

---

## Decisions Made During Refinement

| # | Decision | Rationale |
|---|----------|-----------|
| D-01 | **Updated** version scope from v5/v6 to v6/v7/v8 | Research revealed Vite 8 is current (Rolldown+Oxc). v5 is legacy. |
| D-02 | **Merged** `vite-syntax-resolve` and `vite-syntax-css` into `vite-syntax-resolve-css` | Both are config subsections, neither large enough alone. Combined keeps skills focused. |
| D-03 | **Added** `vite-core-environment-api` | Research revealed Environment API (v6+) is significant enough for dedicated coverage. |
| D-04 | **Added** `vite-impl-javascript-api` | Programmatic API (createServer, build, preview) deserves its own skill. |
| D-05 | **Added** `vite-syntax-env-vars` | Environment variables (.env, modes, VITE_ prefix) are distinct from config — separate skill. |
| D-06 | **Merged** multi-page app content into `vite-syntax-build` | Multi-page is just `rolldownOptions.input` — too thin for own skill. |
| D-07 | **Removed** `vite-impl-multi-page` | Absorbed into build syntax skill. |
| D-08 | **Reordered** batches: core first, then syntax, impl, errors, agents last | Standard dependency ordering proven in Tauri package. |

**Result**: 25 raw skills → **22 definitive skills** (1 merge, 3 additions, 2 removals/absorptions).

---

## Definitive Skill Inventory (22 skills)

### vite-core/ (2 skills)

| Name | Scope | Key APIs | Research Input | Complexity | Dependencies |
|------|-------|----------|----------------|------------|-------------|
| `vite-core-architecture` | How Vite works; dev server vs build; esbuild→Oxc and Rollup→Rolldown evolution; module graph; dependency pre-bundling concept; index.html as entry; browser support | Dev server, build pipeline, pre-bundling | Fragments §1 (Architecture) | M | None |
| `vite-core-environment-api` | Vite 6+ Environment API; per-environment config; EnvironmentOptions; custom environment providers; shared vs per-environment plugins; migration from v5 | `environments`, `EnvironmentOptions`, `DevEnvironment` | Fragment 3 §1 | M | core-architecture |

### vite-syntax/ (8 skills)

| Name | Scope | Key APIs | Research Input | Complexity | Dependencies |
|------|-------|----------|----------------|------------|-------------|
| `vite-syntax-config` | vite.config.ts structure; defineConfig(); conditional/async config; loadEnv(); shared options (root, base, mode, define, plugins, publicDir, cacheDir, logLevel, appType, envPrefix, envDir) | `defineConfig`, `loadEnv`, shared config keys | Fragment 1 §2 | L | core-architecture |
| `vite-syntax-server` | All server.* options: host, port, proxy, cors, https, hmr, fs (strict/allow/deny), warmup, middlewareMode, open, headers, watch, forwardConsole | `server.*` config namespace | Fragment 1 §3 | M | syntax-config |
| `vite-syntax-build` | All build.* options: target, outDir, assetsDir, assetsInlineLimit, cssCodeSplit, sourcemap, rolldownOptions (v8)/rollupOptions (v6), minify (oxc/terser/esbuild), manifest, ssr, lib overview, multi-page input, watch, license | `build.*` config namespace | Fragment 1 §2 (build section) | L | syntax-config |
| `vite-syntax-resolve-css` | resolve.* (alias, dedupe, conditions, mainFields, extensions, preserveSymlinks, tsconfigPaths) + css.* (modules, postcss, preprocessorOptions, preprocessorMaxWorkers, devSourcemap, transformer, lightningcss) + json.* + html.* | `resolve.*`, `css.*`, `json.*`, `html.*` | Fragment 1 §2 (resolve+css) | M | syntax-config |
| `vite-syntax-plugin-api` | Plugin structure; Vite-specific hooks (config, configResolved, configureServer, configurePreviewServer, transformIndexHtml, handleHotUpdate); Rollup-compatible hooks; enforce/apply; virtual modules; hook filters (v6.3+); plugin context meta; client-server communication; output bundle metadata | All plugin hooks, `Plugin` type | Fragment 2 §1 | L | core-architecture |
| `vite-syntax-hmr-api` | ViteHotContext interface; import.meta.hot guard; accept (self/deps); dispose; prune; invalidate; hot.data; on/off/send; built-in events; HMR boundary concept; full reload triggers | `import.meta.hot.*`, `ViteHotContext` | Fragment 2 §2 | M | core-architecture |
| `vite-syntax-assets` | Static asset imports; ?url/?raw/?inline/?worker suffixes; new URL() pattern; public directory; JSON imports; glob import (import.meta.glob) with all options; WebAssembly; Web Workers (constructor + query patterns) | Asset imports, `import.meta.glob`, `?url`, `?raw` | Fragment 3 §2, Fragment 1 §6 | M | syntax-config |
| `vite-syntax-env-vars` | import.meta.env built-in constants; .env file loading order; VITE_ prefix security; variable expansion; modes; TypeScript IntelliSense (ImportMetaEnv); HTML env replacement | `import.meta.env`, `.env` files, `loadEnv()` | Fragment 1 §4 | S | syntax-config |

### vite-impl/ (7 skills)

| Name | Scope | Key APIs | Research Input | Complexity | Dependencies |
|------|-------|----------|----------------|------------|-------------|
| `vite-impl-project-setup` | create-vite scaffolding; framework templates; project structure; CLI commands (dev/build/preview); TypeScript setup (tsconfig, vite-env.d.ts); package.json scripts | `create-vite`, `vite` CLI | Fragment 1 §6, §8, §9 | M | syntax-config |
| `vite-impl-library-mode` | build.lib configuration; entry/name/fileName/formats/cssFileName; external dependencies; CSS handling; multiple entry points; DTS generation; package.json exports | `build.lib`, `rolldownOptions.external` | Fragment 3 §3 | M | syntax-build |
| `vite-impl-ssr` | SSR dev workflow; middleware mode + Express; ssrLoadModule; ssrFixStacktrace; SSR externals (noExternal/external/target); SSR build commands; SSR manifest; production build steps; SSR conditional logic; plugin SSR detection | `ssrLoadModule`, `ssrFixStacktrace`, `ssr.*` config | Fragment 2 §3 | L | syntax-server, syntax-plugin-api |
| `vite-impl-backend-integration` | Backend framework integration; manifest.json structure; dev mode HTML setup; production rendering; ManifestChunk traversal; modulepreload polyfill; React preamble | `build.manifest`, `ManifestChunk` | Fragment 1 §5 | M | syntax-build |
| `vite-impl-optimization` | Dependency pre-bundling; why pre-bundling exists; CJS→ESM conversion; performance optimization; optimizeDeps.include/exclude/force; monorepo linked deps; caching behavior; browser cache; force re-bundling | `optimizeDeps.*`, pre-bundling | Fragment 3 §4 | M | syntax-config |
| `vite-impl-javascript-api` | Programmatic API; createServer(); ViteDevServer interface; build(); preview(); PreviewServer; resolveConfig(); mergeConfig(); loadConfigFromFile(); loadEnv(); transformWithOxc; utility functions; version constants | `createServer`, `build`, `preview`, `resolveConfig` | Fragment 2 §4 | L | syntax-config, syntax-server |
| `vite-impl-migration` | v5→v6 breaking changes; v6→v7 breaking changes; v7→v8 breaking changes; Rollup→Rolldown migration; esbuild→Oxc migration; Environment API adoption; Node.js requirements per version | Migration APIs, deprecated options | Fragment 1 §7 | M | core-architecture |

### vite-errors/ (3 skills)

| Name | Scope | Key APIs | Research Input | Complexity | Dependencies |
|------|-------|----------|----------------|------------|-------------|
| `vite-errors-build` | Build failures; Rolldown errors; chunk size warnings; target compatibility; CSS minification issues; sourcemap errors; library mode build issues; v8 BundleError handling | Build error types | Fragment 1 §7 (breaking changes), §2 (build) | M | syntax-build |
| `vite-errors-dependency` | Pre-bundling failures; CJS/ESM conversion issues; optimizeDeps troubleshooting; monorepo linked dep errors; cache invalidation; missing dependencies; browser cache issues | `optimizeDeps` errors | Fragment 3 §4 | M | impl-optimization |
| `vite-errors-dev-server` | Dev server failures; HMR not working; full reload instead of HMR; proxy errors; CORS issues; HTTPS setup problems; module resolution errors; fs.strict violations; port conflicts | Dev server error patterns | Fragment 1 §3, Fragment 2 §2 | M | syntax-server, syntax-hmr-api |

### vite-agents/ (2 skills)

| Name | Scope | Key APIs | Research Input | Complexity | Dependencies |
|------|-------|----------|----------------|------------|-------------|
| `vite-agents-review` | Validation checklist for Vite config/code; config correctness; plugin hook signatures; HMR pattern validation; version-specific API usage; anti-pattern detection; build optimization checks | All validation rules | All fragments §anti-patterns | M | ALL syntax + impl skills |
| `vite-agents-project-scaffolder` | Generate complete Vite project; vite.config.ts with plugins; TypeScript setup; env variables; package.json; framework detection; plugin selection decision tree | All scaffolding patterns | Fragment 1 §8, §9 | L | ALL core + syntax skills |

---

## Batch Execution Plan (DEFINITIVE)

| Batch | Skills | Count | Dependencies | Notes |
|-------|--------|-------|-------------|-------|
| 1 | `core-architecture`, `syntax-config` | 2 | None | Foundation skills, no dependencies |
| 2 | `core-environment-api`, `syntax-server`, `syntax-build` | 3 | Batch 1 | Core v6 API + primary config namespaces |
| 3 | `syntax-resolve-css`, `syntax-plugin-api`, `syntax-hmr-api` | 3 | Batch 1 | Config subsections + plugin/HMR APIs |
| 4 | `syntax-assets`, `syntax-env-vars`, `impl-project-setup` | 3 | Batch 1-2 | Assets, env vars, project setup |
| 5 | `impl-library-mode`, `impl-ssr`, `impl-backend-integration` | 3 | Batch 2-3 | Build workflows |
| 6 | `impl-optimization`, `impl-javascript-api`, `impl-migration` | 3 | Batch 1-3 | Advanced impl skills |
| 7 | `errors-build`, `errors-dependency`, `errors-dev-server` | 3 | Batch 2-6 | All error skills |
| 8 | `agents-review`, `agents-project-scaffolder` | 2 | ALL above | Agent skills last |

**Total**: 22 skills across 8 batches.

---

## Per-Skill Agent Prompts

### Constants

```
PROJECT_ROOT = C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package
RESEARCH_DIR = C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\docs\research\fragments
RESEARCH_1 = C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\docs\research\fragments\research-core-config.md
RESEARCH_2 = C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\docs\research\fragments\research-plugin-hmr-ssr.md
RESEARCH_3 = C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\docs\research\fragments\research-env-assets-lib.md
REQUIREMENTS_FILE = C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\REQUIREMENTS.md
REFERENCE_SKILL = C:\Users\Freek Heijting\Documents\GitHub\Tauri-2-Claude-Skill-Package\skills\source\tauri-core\tauri-core-architecture\SKILL.md
```

---

### Batch 1

#### Prompt: vite-core-architecture

```
## Task: Create the vite-core-architecture skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-core\vite-core-architecture\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/internals.md (version evolution table, module graph, build pipeline)
3. references/examples.md (minimal project, CLI commands, package.json)
4. references/anti-patterns.md (common architecture mistakes)

### Reference Format
Read and follow the structure of:
C:\Users\Freek Heijting\Documents\GitHub\Tauri-2-Claude-Skill-Package\skills\source\tauri-core\tauri-core-architecture\SKILL.md

### YAML Frontmatter
---
name: vite-core-architecture
description: "Guides Vite application architecture including dev server vs build pipeline, native ESM serving, dependency pre-bundling concept, module graph, internal tools evolution (esbuild→Oxc, Rollup→Rolldown), index.html as entry point, browser support targets, and Node.js requirements. Activates when creating new Vite projects, understanding Vite internals, or reasoning about dev vs production behavior."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- How Vite works: dev server (native ESM) vs build (bundler)
- Internal tools evolution table: v5 (esbuild+Rollup) → v6/v7 (esbuild+Rollup) → v8 (Oxc+Rolldown)
- Module graph and dependency pre-bundling concept (why, not how — how is in impl-optimization)
- index.html as application entry point
- Browser support targets (dev: esnext, production: baseline-widely-available)
- Node.js requirements per version (v5/v6: 18+, v7+: 20.19+/22.12+)
- CLI commands overview (vite, vite build, vite preview)
- Official plugins list (@vitejs/plugin-react, -vue, -legacy, etc.)
- create-vite templates overview

### Research Sections to Read
From research-core-config.md:
- Section 1: Architecture Overview
- Section 8: CLI Commands
- Section 9: Framework Templates
- Section 10: Official Plugins

### Quality Rules
- English only
- SKILL.md < 500 lines; heavy content goes in references/
- Use ALWAYS/NEVER deterministic language
- Include version evolution table prominently
- Note v6/v7/v8 differences where applicable
- Include Critical Warnings section
```

#### Prompt: vite-syntax-config

```
## Task: Create the vite-syntax-config skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-syntax\vite-syntax-config\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/config-options.md (all shared config options with types and defaults)
3. references/examples.md (conditional config, async config, loadEnv, mergeConfig)
4. references/anti-patterns.md (config mistakes)

### YAML Frontmatter
---
name: vite-syntax-config
description: "Guides vite.config.ts/js configuration including defineConfig(), conditional and async config functions, loadEnv() for environment variables in config, and all shared options (root, base, mode, define, plugins, publicDir, cacheDir, logLevel, clearScreen, envDir, envPrefix, appType, future). Activates when creating or editing vite.config.ts, setting up project configuration, or configuring shared Vite options."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- vite.config.ts/js/mjs/mts/cjs/cts file resolution
- defineConfig() for type safety
- Conditional config: ({ command, mode, isSsrBuild, isPreview }) => config
- Async config: async ({ command, mode }) => config
- loadEnv(mode, envDir, prefixes) usage in config
- All shared options with types, defaults, and usage:
  - root, base, mode, define, plugins, publicDir, cacheDir
  - logLevel, customLogger, clearScreen, envDir, envPrefix, appType
  - future (migration helpers)
  - assetsInclude
- Plugin array handling (falsy ignored, arrays flattened)
- Security: NEVER set envPrefix to '' (exposes all env vars)

### Research Sections to Read
From research-core-config.md:
- Section 2: Configuration System (all shared options)

### Quality Rules
- English only
- SKILL.md < 500 lines; heavy content goes in references/
- Use ALWAYS/NEVER deterministic language
- Include complete options table with types and defaults
- Include conditional config and async config patterns
- Include Critical Warnings (envPrefix security)
```

---

### Batch 2

#### Prompt: vite-core-environment-api

```
## Task: Create the vite-core-environment-api skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-core\vite-core-environment-api\

### Files to Create
1. SKILL.md (<500 lines)
2. references/config-options.md (EnvironmentOptions interface, per-environment config keys)
3. references/examples.md (multi-environment config, custom provider, framework usage)
4. references/anti-patterns.md (environment config mistakes)

### YAML Frontmatter
---
name: vite-core-environment-api
description: "Guides the Vite 6+ Environment API for multi-environment configuration including per-environment build and dev settings, EnvironmentOptions interface, custom environment providers, shared vs per-environment plugins, configuration inheritance, and migration from Vite 5 implicit environments. Activates when configuring SSR environments, edge workers, custom environments, or migrating to Vite 6+ environment model."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x or later."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- What the Environment API is and why it exists
- Backward compatibility (SPAs unchanged)
- Per-environment configuration: environments.server, environments.edge, etc.
- Configuration inheritance rules (top-level → environment)
- EnvironmentOptions interface: define, resolve, optimizeDeps, consumer, dev, build
- Custom environment providers
- Shared plugins vs per-environment plugins
- Target audiences: end users, plugin authors, framework authors, runtime providers
- Release status (RC phase)
- Migration from Vite 5 implicit environments

### Research Sections to Read
From research-env-assets-lib.md:
- Section 1: Environment API (Vite 6)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include EnvironmentOptions interface
- Include multi-environment config example
- Note this is Vite 6+ only
```

#### Prompt: vite-syntax-server

```
## Task: Create the vite-syntax-server skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-syntax\vite-syntax-server\

### Files to Create
1. SKILL.md (<500 lines)
2. references/config-options.md (all server.* options with types, defaults)
3. references/examples.md (proxy, HTTPS, middleware mode, warmup, fs)
4. references/anti-patterns.md (server config mistakes)

### YAML Frontmatter
---
name: vite-syntax-server
description: "Guides all Vite dev server configuration options including server.host, server.port, server.proxy, server.cors, server.https, server.hmr, server.fs (strict/allow/deny), server.warmup, server.middlewareMode, server.open, server.headers, server.watch, server.forwardConsole, and server.allowedHosts. Activates when configuring the Vite dev server, setting up proxy rules, enabling HTTPS, restricting file system access, or using middleware mode."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- All server.* options with types, defaults, and code examples
- server.host: localhost vs 0.0.0.0
- server.port: default 5173, strictPort
- server.proxy: string shorthand, options object, rewrite
- server.cors: default behavior, custom origins
- server.https: TLS + HTTP/2 setup
- server.hmr: WebSocket config, overlay, clientPort
- server.fs: strict mode, allow/deny lists, security
- server.warmup: clientFiles, ssrFiles
- server.middlewareMode: custom HTTP server integration
- server.open, server.headers, server.watch
- server.allowedHosts (security)
- server.forwardConsole (v8+)
- server.origin, server.sourcemapIgnoreList

### Research Sections to Read
From research-core-config.md:
- Section 2: Server Options (server.*)
- Section 3: Dev Server

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include complete options table
- Include proxy configuration examples
- Include middleware mode pattern
- Note v8-specific options (forwardConsole)
```

#### Prompt: vite-syntax-build

```
## Task: Create the vite-syntax-build skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-syntax\vite-syntax-build\

### Files to Create
1. SKILL.md (<500 lines)
2. references/config-options.md (all build.* options with types, defaults)
3. references/examples.md (production build, multi-page, watch mode, library mode overview)
4. references/anti-patterns.md (build config mistakes)

### YAML Frontmatter
---
name: vite-syntax-build
description: "Guides all Vite build configuration options including build.target, build.outDir, build.assetsDir, build.assetsInlineLimit, build.cssCodeSplit, build.cssTarget, build.cssMinify, build.sourcemap, build.rolldownOptions (v8)/build.rollupOptions (v6-v7), build.lib, build.manifest, build.ssr, build.ssrManifest, build.minify (oxc/terser/esbuild), build.write, build.emptyOutDir, build.reportCompressedSize, build.chunkSizeWarningLimit, build.watch, and build.license. Activates when configuring production builds, optimizing output, setting build targets, or configuring chunk splitting."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- All build.* options with types, defaults, code examples
- build.target: baseline-widely-available (v7+), esnext, custom targets
- build.rolldownOptions (v8) vs build.rollupOptions (v6-v7)
- build.minify: oxc (v8 default) vs esbuild vs terser
- build.lib overview (detail in impl-library-mode)
- build.manifest and ManifestChunk structure
- build.ssr and build.ssrManifest
- build.sourcemap options (true, inline, hidden)
- build.cssCodeSplit, build.cssTarget, build.cssMinify
- build.license (v8+)
- Multi-page app: rolldownOptions.input with multiple HTML entries
- build.watch for rebuild on changes
- Build optimizations: CSS code splitting, preload directives, async chunk loading

### Research Sections to Read
From research-core-config.md:
- Section 2: Build Options (build.*)
From research-env-assets-lib.md:
- Section 5: Build Optimizations

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include complete options table with version annotations
- MUST note rolldownOptions vs rollupOptions per version
- MUST note minify options per version (oxc vs esbuild)
- Include multi-page input example
```

---

### Batch 3

#### Prompt: vite-syntax-resolve-css

```
## Task: Create the vite-syntax-resolve-css skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-syntax\vite-syntax-resolve-css\

### Files to Create
1. SKILL.md (<500 lines)
2. references/config-options.md (all resolve.*, css.*, json.*, html.* options)
3. references/examples.md (aliases, CSS modules, preprocessors, Lightning CSS)
4. references/anti-patterns.md (resolve and CSS mistakes)

### YAML Frontmatter
---
name: vite-syntax-resolve-css
description: "Guides Vite module resolution and CSS configuration including resolve.alias, resolve.dedupe, resolve.conditions, resolve.extensions, resolve.tsconfigPaths, css.modules, css.postcss, css.preprocessorOptions (Sass/Less/Stylus), css.transformer (PostCSS/Lightning CSS), css.lightningcss, css.devSourcemap, json.namedExports, json.stringify, and html.cspNonce. Activates when configuring path aliases, CSS modules, CSS preprocessors, PostCSS, Lightning CSS, or module resolution behavior."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- resolve.alias (object and array formats, regex support)
- resolve.dedupe, resolve.conditions, resolve.mainFields, resolve.extensions
- resolve.preserveSymlinks, resolve.tsconfigPaths
- css.modules (scopeBehaviour, localsConvention, generateScopedName, etc.)
- css.postcss (inline config or path)
- css.preprocessorOptions (scss, less, stylus — additionalData, math, etc.)
- css.preprocessorMaxWorkers
- css.devSourcemap
- css.transformer ('postcss' | 'lightningcss')
- css.lightningcss configuration
- json.namedExports, json.stringify ('auto')
- html.cspNonce
- CSS Modules: .module.css pattern, camelCase convention
- Preprocessor installation (sass-embedded, less, stylus — NO Vite plugins needed)
- PostCSS auto-detection

### Research Sections to Read
From research-core-config.md:
- Section 2: resolve.* Options, css.* Options, json.* Options, html.* Options
From research-env-assets-lib.md:
- Section 5: CSS Features

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include both resolve and CSS options tables
- Include alias examples (object + array format)
- Include CSS preprocessor setup examples
```

#### Prompt: vite-syntax-plugin-api

```
## Task: Create the vite-syntax-plugin-api skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-syntax\vite-syntax-plugin-api\

### Files to Create
1. SKILL.md (<500 lines)
2. references/hooks.md (ALL hook signatures — Vite-specific and Rollup-compatible)
3. references/examples.md (complete plugin template, virtual modules, tag injection, client-server communication)
4. references/anti-patterns.md (plugin development mistakes)

### YAML Frontmatter
---
name: vite-syntax-plugin-api
description: "Guides the complete Vite plugin API including plugin structure, naming conventions, Vite-specific hooks (config, configResolved, configureServer, configurePreviewServer, transformIndexHtml, handleHotUpdate), Rollup-compatible hooks (resolveId, load, transform, buildStart, buildEnd), plugin ordering (enforce pre/post), conditional application (apply), virtual modules pattern, hook filters (v6.3+), plugin context meta, client-server WebSocket communication, output bundle metadata, and path normalization utilities. Activates when writing Vite plugins, understanding hook lifecycle, creating virtual modules, or customizing the build pipeline."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- Plugin structure: factory function returning object with name property
- Naming conventions: vite-plugin-*, rolldown-plugin-*, framework-specific
- ALL Vite-specific hooks with COMPLETE signatures:
  - config(config, env) → UserConfig | null | void
  - configResolved(config) → void
  - configureServer(server) → (() => void) | void
  - configurePreviewServer(server) → (() => void) | void
  - transformIndexHtml(html, ctx) → string | HtmlTagDescriptor[] | { html, tags }
  - handleHotUpdate(ctx) → ModuleNode[] | void
- ALL Rollup-compatible hooks: options, buildStart, resolveId, load, transform, buildEnd, closeBundle
- Plugin ordering: enforce 'pre' | 'post', execution order
- Conditional apply: 'build' | 'serve' | function
- Virtual modules: virtual: prefix + \0 internal prefix
- Hook filters (v6.3+): filter.id with regex
- Plugin context: this.meta.viteVersion, this.meta.rolldownVersion
- Client-server communication: server.ws.send/on, import.meta.hot.send
- Output bundle metadata: viteMetadata.importedCss, importedAssets
- Utility: normalizePath, createFilter

### Research Sections to Read
From research-plugin-hmr-ssr.md:
- Section 1: Plugin API (complete)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- MUST include ALL hook signatures with types in references/
- Include complete plugin template
- Include virtual modules pattern
- Distinguish Vite-specific from Rollup-compatible hooks clearly
- Note v6.3+ hook filters
```

#### Prompt: vite-syntax-hmr-api

```
## Task: Create the vite-syntax-hmr-api skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-syntax\vite-syntax-hmr-api\

### Files to Create
1. SKILL.md (<500 lines)
2. references/api-reference.md (ViteHotContext interface, all method signatures, built-in events)
3. references/examples.md (self-accepting, dep-accepting, dispose+data, custom events)
4. references/anti-patterns.md (HMR mistakes)

### YAML Frontmatter
---
name: vite-syntax-hmr-api
description: "Guides the Vite HMR (Hot Module Replacement) client API including ViteHotContext interface, import.meta.hot guard pattern, hot.accept() for self-accepting and dependency-accepting modules, hot.dispose() for cleanup, hot.prune() for module pruning, hot.invalidate() for forced propagation, hot.data for persistent state, hot.on()/hot.off()/hot.send() for custom events, built-in HMR events, HMR boundary concept, and when full reload occurs vs HMR update. Activates when implementing custom HMR handling, debugging HMR issues, writing HMR-aware modules, or understanding why full reloads happen."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- ViteHotContext full interface with ALL method signatures
- import.meta.hot guard pattern (CRITICAL: whitespace-sensitive for static analysis)
- hot.accept(): self-accepting modules
- hot.accept(dep, cb) / hot.accept(deps, cb): dependency accepting
- hot.dispose(cb): cleanup side effects
- hot.prune(cb): module pruning
- hot.invalidate(message?): force propagation (MUST call accept() first)
- hot.data: persistent data across updates (mutate properties, NOT reassign)
- hot.on/off: built-in events (vite:beforeUpdate, vite:afterUpdate, vite:beforeFullReload, vite:beforePrune, vite:invalidate, vite:error, vite:ws:disconnect, vite:ws:connect)
- hot.send: custom events to server
- HMR boundary concept: what it is, how updates propagate
- When full reload happens vs HMR update
- TypeScript setup: vite/client types

### Research Sections to Read
From research-plugin-hmr-ssr.md:
- Section 2: HMR API (complete)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include ViteHotContext interface
- Include built-in events table
- CRITICAL WARNING: import.meta.hot.accept( is whitespace-sensitive
- CRITICAL WARNING: hot.data cannot be reassigned, only mutated
```

---

### Batch 4

#### Prompt: vite-syntax-assets

```
## Task: Create the vite-syntax-assets skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-syntax\vite-syntax-assets\

### Files to Create
1. SKILL.md (<500 lines)
2. references/import-patterns.md (all import suffixes, glob options, worker patterns)
3. references/examples.md (assets, glob, JSON, WebAssembly, workers)
4. references/anti-patterns.md (asset handling mistakes)

### YAML Frontmatter
---
name: vite-syntax-assets
description: "Guides Vite static asset handling including asset imports (resolved URLs), query suffixes (?url, ?raw, ?inline, ?no-inline, ?worker, ?sharedworker), new URL() pattern with import.meta.url, public directory behavior, JSON imports with named exports, glob imports (import.meta.glob) with eager/lazy/named/query options, WebAssembly ?init imports, and Web Worker patterns (constructor and query syntax). Activates when importing static assets, using glob imports, configuring the public directory, loading Web Workers, or handling WebAssembly modules."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- Asset imports: dev vs production URL resolution
- Query suffixes: ?url, ?raw, ?inline, ?no-inline, ?worker, ?sharedworker, ?worker&inline, ?worker&url
- new URL(path, import.meta.url) pattern (limitations: static strings, no SSR)
- Public directory: publicDir config, behavior, when to use
- JSON imports: full object, named exports (tree-shakeable), json.stringify
- Glob import (import.meta.glob): lazy/eager, multiple patterns, negative patterns, named imports, custom queries, base option
- Glob caveats: Vite-only, string literals only, tinyglobby matching
- WebAssembly: ?init suffix, importObject, URL import for multiple instances
- Web Workers: constructor pattern (new Worker(new URL(...))), query pattern (?worker)
- assetsInlineLimit: base64 inlining threshold
- vite-ignore attribute for HTML opt-out

### Research Sections to Read
From research-env-assets-lib.md:
- Section 2: Static Asset Handling
From research-core-config.md:
- Section 6: Key Features Reference (assets, glob, JSON, workers)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include all query suffixes in a table
- Include glob import options table
- Include Web Worker decision tree (constructor vs query)
- CRITICAL WARNING: import.meta.glob arguments must be string literals
```

#### Prompt: vite-syntax-env-vars

```
## Task: Create the vite-syntax-env-vars skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-syntax\vite-syntax-env-vars\

### Files to Create
1. SKILL.md (<500 lines)
2. references/config-options.md (env-related config, .env file precedence, modes)
3. references/examples.md (env files, TypeScript IntelliSense, HTML replacement, custom modes)
4. references/anti-patterns.md (env variable mistakes)

### YAML Frontmatter
---
name: vite-syntax-env-vars
description: "Guides Vite environment variables including import.meta.env built-in constants (MODE, BASE_URL, PROD, DEV, SSR), .env file loading order and precedence, VITE_ prefix security requirement, dotenv-expand variable expansion, custom modes, NODE_ENV vs mode distinction, TypeScript IntelliSense for custom env vars (ImportMetaEnv), HTML template env replacement (%VAR%), and loadEnv() for config access. Activates when configuring environment variables, using .env files, creating custom modes, or debugging env variable issues."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- import.meta.env built-in constants: MODE, BASE_URL, PROD, DEV, SSR
- .env file loading order: .env → .env.local → .env.[mode] → .env.[mode].local
- VITE_ prefix requirement (security: NEVER set envPrefix to '')
- Variable expansion (dotenv-expand syntax)
- Modes: development, production, custom (--mode flag)
- NODE_ENV vs mode distinction (table showing combinations)
- TypeScript IntelliSense: ImportMetaEnv interface in vite-env.d.ts
- HTML env replacement: %VITE_VAR% syntax
- loadEnv() in config files
- envDir and envPrefix config options
- Security: VITE_* variables MUST NOT contain secrets

### Research Sections to Read
From research-core-config.md:
- Section 4: Environment Variables

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include .env loading order table
- Include NODE_ENV vs mode table
- Include TypeScript IntelliSense setup
- CRITICAL WARNING: envPrefix security
- CRITICAL WARNING: no secrets in VITE_ vars
```

#### Prompt: vite-impl-project-setup

```
## Task: Create the vite-impl-project-setup skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-impl\vite-impl-project-setup\

### Files to Create
1. SKILL.md (<500 lines)
2. references/templates.md (all create-vite templates, framework setup)
3. references/examples.md (new project workflow, TypeScript setup, package.json)
4. references/anti-patterns.md (project setup mistakes)

### YAML Frontmatter
---
name: vite-impl-project-setup
description: "Guides Vite project setup including create-vite scaffolding with all framework templates, project structure conventions, CLI commands (vite dev, vite build, vite preview), TypeScript configuration (tsconfig.json requirements, vite-env.d.ts, isolatedModules), package.json scripts, and framework plugin integration. Activates when creating a new Vite project, adding Vite to an existing project, configuring TypeScript, or setting up the development workflow."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- create-vite workflow and all templates (vanilla, react, vue, svelte, preact, lit, solid, qwik — with -ts variants)
- Project structure: index.html, src/, public/, vite.config.ts
- CLI: vite (dev), vite build, vite preview with common flags
- TypeScript setup: tsconfig requirements (isolatedModules: true), vite-env.d.ts, vite/client types
- package.json scripts pattern
- Framework plugin setup (@vitejs/plugin-react, @vitejs/plugin-vue, etc.)
- JSX/TSX configuration (oxc in v8, esbuild in v6/v7)
- CSS preprocessor installation (no Vite plugins needed)
- Source control: what to commit, what to gitignore

### Research Sections to Read
From research-core-config.md:
- Section 1: Architecture (project structure)
- Section 6: TypeScript Support, JSX
- Section 8-9: CLI, Framework Templates
From research-env-assets-lib.md:
- Section 5: TypeScript Support

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include step-by-step project creation workflow
- Include TypeScript tsconfig requirements
- Include framework plugin table
```

---

### Batch 5

#### Prompt: vite-impl-library-mode

```
## Task: Create the vite-impl-library-mode skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-impl\vite-impl-library-mode\

### Files to Create
1. SKILL.md (<500 lines)
2. references/config-options.md (build.lib options, package.json exports patterns)
3. references/examples.md (single entry, multiple entries, CSS handling, DTS)
4. references/anti-patterns.md (library mode mistakes)

### YAML Frontmatter
---
name: vite-impl-library-mode
description: "Guides Vite library mode for building reusable npm packages including build.lib configuration (entry, name, fileName, formats, cssFileName), external dependencies via rolldownOptions.external, output formats (es, cjs, umd, iife), CSS handling in libraries, multiple entry points, DTS generation with vite-plugin-dts, and recommended package.json exports field structure. Activates when building a library, creating an npm package, configuring UMD/ESM/CJS output, or setting up package.json exports."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- build.lib configuration: entry, name, fileName, formats, cssFileName
- Default formats: single entry (es+umd), multiple entries (es+cjs)
- External dependencies: rolldownOptions.external + output.globals (for UMD)
- CSS handling: separate CSS file, cssFileName, package.json CSS export
- Multiple entry points: object format entry
- DTS generation: vite-plugin-dts integration
- Recommended package.json: type, files, main, module, exports, types
- UMD global variable name requirement

### Research Sections to Read
From research-env-assets-lib.md:
- Section 3: Library Mode (complete)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include complete package.json example
- Include build.lib options table
- Include format decision tree
- Note rolldownOptions (v8) vs rollupOptions (v6/v7)
```

#### Prompt: vite-impl-ssr

```
## Task: Create the vite-impl-ssr skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-impl\vite-impl-ssr\

### Files to Create
1. SKILL.md (<500 lines)
2. references/api-reference.md (ssrLoadModule, ssrFixStacktrace, SSR config options)
3. references/examples.md (Express SSR dev server, build commands, production server)
4. references/anti-patterns.md (SSR mistakes)

### YAML Frontmatter
---
name: vite-impl-ssr
description: "Guides Vite Server-Side Rendering including SSR dev workflow with Express middleware mode, ssrLoadModule() for development, ssrFixStacktrace() for error debugging, SSR externals configuration (ssr.noExternal, ssr.external, ssr.target), SSR build commands, SSR manifest for preload directives, SSR conditional logic (import.meta.env.SSR), production build steps, and plugin SSR detection via options.ssr. Activates when implementing SSR, configuring SSR externals, building SSR applications, or debugging SSR errors."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- SSR dev workflow: middleware mode + Express
- ssrLoadModule(url, options?) → module exports
- ssrFixStacktrace(e) for source-mapped errors
- SSR externals: ssr.noExternal (array or true), ssr.external, ssr.target ('node' | 'webworker')
- SSR build: --ssr flag, separate client/server builds
- SSR manifest: --ssrManifest flag, .vite/ssr-manifest.json
- SSR conditional: import.meta.env.SSR (tree-shakeable)
- Production build steps: template from dist/client, import from dist/server
- Plugin SSR detection: options?.ssr in transform hook
- index.html placeholder pattern: <!--ssr-outlet-->
- ssr.resolve.conditions, ssr.resolve.externalConditions

### Research Sections to Read
From research-plugin-hmr-ssr.md:
- Section 3: SSR (complete)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include complete Express SSR dev server example
- Include build commands for client + server
- Include production server setup
```

#### Prompt: vite-impl-backend-integration

```
## Task: Create the vite-impl-backend-integration skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-impl\vite-impl-backend-integration\

### Files to Create
1. SKILL.md (<500 lines)
2. references/manifest-reference.md (ManifestChunk interface, traversal algorithm)
3. references/examples.md (dev HTML, production rendering, React preamble)
4. references/anti-patterns.md (backend integration mistakes)

### YAML Frontmatter
---
name: vite-impl-backend-integration
description: "Guides Vite integration with backend frameworks including build.manifest configuration, .vite/manifest.json structure and ManifestChunk properties, development mode HTML setup with Vite client script, production HTML rendering from manifest, CSS and JS chunk resolution order, React preamble for HMR, modulepreload polyfill, and manifest traversal algorithm. Activates when integrating Vite with a backend framework, rendering Vite assets from server-side templates, or setting up dev/production HTML serving."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- Vite config for backend integration: build.manifest, server.cors
- Development mode HTML: @vite/client script, module entry script
- React-specific preamble (RefreshRuntime injection)
- manifest.json structure: ManifestChunk properties (src, file, css, assets, isEntry, imports, dynamicImports)
- Rendering order: CSS links → imported CSS → script → modulepreload
- Manifest traversal algorithm (TypeScript implementation)
- modulepreload polyfill: 'vite/modulepreload-polyfill'
- CORS configuration for dev server

### Research Sections to Read
From research-core-config.md:
- Section 5: Backend Integration (complete)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include dev mode HTML template
- Include manifest.json example
- Include rendering algorithm
- Include React preamble
```

---

### Batch 6

#### Prompt: vite-impl-optimization

```
## Task: Create the vite-impl-optimization skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-impl\vite-impl-optimization\

### Files to Create
1. SKILL.md (<500 lines)
2. references/config-options.md (all optimizeDeps.* options)
3. references/examples.md (include/exclude, monorepo, force re-bundle, custom entries)
4. references/anti-patterns.md (dependency optimization mistakes)

### YAML Frontmatter
---
name: vite-impl-optimization
description: "Guides Vite dependency pre-bundling and optimization including why pre-bundling exists (CJS-to-ESM conversion, HTTP request reduction), optimizeDeps.include/exclude/force configuration, automatic dependency discovery, monorepo linked dependency handling, caching behavior (node_modules/.vite), cache invalidation triggers, browser cache headers, Rolldown (v8) vs esbuild (v5-v7) for pre-bundling, and debugging dependency issues. Activates when configuring dependency pre-bundling, troubleshooting slow dev server starts, handling monorepo dependencies, or forcing cache invalidation."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- Why pre-bundling: CJS→ESM, performance (module count reduction)
- Pre-bundling tools: Rolldown (v8), esbuild (v5-v7)
- optimizeDeps.include: when to use, deep import globs
- optimizeDeps.exclude: when to use, CJS deps should NOT be excluded
- optimizeDeps.force: force re-bundling
- optimizeDeps.entries: custom entry points
- optimizeDeps.noDiscovery: disable auto-discovery
- optimizeDeps.rolldownOptions (v8) / esbuildOptions (v5-v7)
- optimizeDeps.holdUntilCrawlEnd, needsInterop
- Automatic dependency discovery
- Monorepo linked deps: must export ESM, add to include if not
- Caching: node_modules/.vite, invalidation triggers
- Browser cache: max-age=31536000,immutable + version query
- Debugging: --force flag, DevTools network tab

### Research Sections to Read
From research-env-assets-lib.md:
- Section 4: Dependency Pre-Bundling (complete)
From research-core-config.md:
- Section 2: optimizeDeps.* Options

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include cache invalidation triggers table
- Include monorepo decision tree
- Note Rolldown vs esbuild per version
```

#### Prompt: vite-impl-javascript-api

```
## Task: Create the vite-impl-javascript-api skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-impl\vite-impl-javascript-api\

### Files to Create
1. SKILL.md (<500 lines)
2. references/api-reference.md (createServer, ViteDevServer, build, preview, PreviewServer, resolveConfig, mergeConfig, utility functions)
3. references/examples.md (programmatic dev server, build, preview, loadEnv, transformWithOxc)
4. references/anti-patterns.md (programmatic API mistakes)

### YAML Frontmatter
---
name: vite-impl-javascript-api
description: "Guides the Vite programmatic JavaScript API including createServer() with InlineConfig, ViteDevServer interface (moduleGraph, middlewares, ws, transformRequest, ssrLoadModule), build() with RollupOutput, preview() with PreviewServer, resolveConfig(), mergeConfig(), loadConfigFromFile(), loadEnv(), searchForWorkspaceRoot(), transformWithOxc() (v8)/transformWithEsbuild() (deprecated), normalizePath(), createFilter(), and version constants. Activates when using Vite programmatically, building custom dev servers, running builds from scripts, or accessing Vite utilities."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- createServer(inlineConfig?) → ViteDevServer
- ViteDevServer interface: config, middlewares, httpServer, watcher, ws, pluginContainer, moduleGraph, resolvedUrls, transformRequest, transformIndexHtml, ssrLoadModule, ssrFixStacktrace, reloadModule, listen, restart, close, bindCLIShortcuts, waitForRequestsIdle
- build(inlineConfig?) → RollupOutput
- preview(inlineConfig?) → PreviewServer
- PreviewServer interface
- resolveConfig(inlineConfig, command, defaultMode, defaultNodeEnv, isPreview)
- mergeConfig(defaults, overrides, isRoot): deep merge for nested options
- loadConfigFromFile(configEnv, configFile?, configRoot?, logLevel?, customLogger?)
- loadEnv(mode, envDir, prefixes)
- searchForWorkspaceRoot(current, root?)
- transformWithOxc (v8) / transformWithEsbuild (deprecated)
- normalizePath(), createFilter()
- preprocessCSS() (experimental)
- Version constants: version, rolldownVersion

### Research Sections to Read
From research-plugin-hmr-ssr.md:
- Section 4: JavaScript API (complete)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include ViteDevServer interface
- Include createServer + build examples
- Note transformWithOxc (v8) vs transformWithEsbuild
```

#### Prompt: vite-impl-migration

```
## Task: Create the vite-impl-migration skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-impl\vite-impl-migration\

### Files to Create
1. SKILL.md (<500 lines)
2. references/breaking-changes.md (v5→v6, v6→v7, v7→v8 complete lists)
3. references/examples.md (config migration examples per version)
4. references/anti-patterns.md (migration mistakes)

### YAML Frontmatter
---
name: vite-impl-migration
description: "Guides Vite version migration including v5 to v6 breaking changes (Environment API, json.stringify, resolve.conditions, PostCSS, Sass modern API), v6 to v7 breaking changes (Node.js 20+, build.target update, Sass legacy removed), and v7 to v8 breaking changes (Rolldown replaces Rollup, Oxc replaces esbuild, Lightning CSS default, rolldownOptions replaces rollupOptions, moduleType requirement for plugins). Activates when upgrading Vite versions, encountering deprecated APIs, or resolving version-specific errors."
license: MIT
compatibility: "Designed for Claude Code. Covers migration paths from Vite 5 through Vite 8."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- v5→v6: Environment API, Vite Runtime API removed, resolve.conditions defaults, json.stringify 'auto', PostCSS v6, Sass modern API default, library mode CSS filename, build.cssMinify SSR default, CommonJS strictRequires
- v6→v7: Node.js 20.19+/22.12+, build.target baseline-widely-available, Sass legacy removed, splitVendorChunkPlugin removed, hook-level enforce/transform removed for transformIndexHtml, optimizeDeps.entries globs
- v7→v8: Rolldown replaces Rollup (build.rolldownOptions), Oxc replaces esbuild (oxc config), Lightning CSS default minifier, Oxc minifier default, CJS interop changes, module resolution changes, esbuild optional, build.target updated, import.meta.url UMD/IIFE, moduleType: 'js' for plugins, BundleError
- Node.js version requirements per Vite version
- Config migration patterns per version

### Research Sections to Read
From research-core-config.md:
- Section 7: Migration Guides (complete)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include version comparison table
- Include config migration examples (before/after)
- Include Node.js requirements per version
- Organized by migration path (v5→v6, v6→v7, v7→v8)
```

---

### Batch 7

#### Prompt: vite-errors-build

```
## Task: Create the vite-errors-build skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-errors\vite-errors-build\

### Files to Create
1. SKILL.md (<500 lines)
2. references/error-catalog.md (build error patterns with symptoms, causes, fixes)
3. references/examples.md (error scenarios and solutions)
4. references/anti-patterns.md (build configuration mistakes)

### YAML Frontmatter
---
name: vite-errors-build
description: "Diagnoses and resolves Vite build errors including Rolldown/Rollup bundling failures, chunk size warnings, build.target compatibility issues, CSS minification errors, sourcemap problems, library mode build failures, deprecated rollupOptions in v8, v8 BundleError handling, minifier issues (oxc vs esbuild vs terser), and asset processing errors. Activates when encountering build failures, production bundle issues, chunk size warnings, or version-specific build errors."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- Rolldown/Rollup bundling errors
- Chunk size warnings (build.chunkSizeWarningLimit)
- build.target compatibility: syntax not supported by target
- CSS minification failures
- Sourcemap generation issues
- Library mode: missing name for UMD, missing external for peer deps
- v8: build.rollupOptions deprecated → use build.rolldownOptions
- v8: BundleError handling (try/catch with e.errors)
- v8: moduleType: 'js' required in plugins for non-JS content
- Minifier issues: oxc (v8), esbuild, terser
- Asset processing errors: inline limit, public dir issues
- vite:preloadError event for dynamic import failures

### Research Sections to Read
From research-core-config.md:
- Section 2: Build Options
- Section 7: Migration breaking changes (build-related)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Format as diagnostic table: Symptom → Cause → Fix
- Include v8 BundleError handling code
- Include version-specific fixes
```

#### Prompt: vite-errors-dependency

```
## Task: Create the vite-errors-dependency skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-errors\vite-errors-dependency\

### Files to Create
1. SKILL.md (<500 lines)
2. references/error-catalog.md (dependency error patterns)
3. references/examples.md (error scenarios and solutions)
4. references/anti-patterns.md (dependency optimization mistakes)

### YAML Frontmatter
---
name: vite-errors-dependency
description: "Diagnoses and resolves Vite dependency pre-bundling errors including CJS/ESM conversion failures, missing dependencies not auto-discovered, optimizeDeps misconfiguration, monorepo linked dependency issues, cache invalidation problems, browser cache staleness, excluded CJS dependencies breaking, and slow dev server starts from large dependency trees. Activates when encountering pre-bundling errors, dependency resolution failures, stale cache issues, or slow development server startup."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- CJS→ESM conversion failures: named imports not working
- Missing dependency: import from plugin transform not discovered
- optimizeDeps.exclude on CJS dep (breaks it)
- Monorepo: linked dep not exporting ESM
- Cache invalidation: when it triggers, when it doesn't
- Browser cache: stale deps after manual edits
- Slow startup: too many modules (need optimizeDeps.include)
- --force flag: when to use
- Deleting node_modules/.vite manually
- Browser DevTools network tab debugging
- optimizeDeps.needsInterop for forced ESM interop
- Version-specific: esbuildOptions (v5-v7) vs rolldownOptions (v8)

### Research Sections to Read
From research-env-assets-lib.md:
- Section 4: Dependency Pre-Bundling (debugging section)
From research-core-config.md:
- Section 2: optimizeDeps.* Options

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Format as diagnostic table: Symptom → Cause → Fix
- Include debugging flowchart
- Include cache behavior explanation
```

#### Prompt: vite-errors-dev-server

```
## Task: Create the vite-errors-dev-server skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-errors\vite-errors-dev-server\

### Files to Create
1. SKILL.md (<500 lines)
2. references/error-catalog.md (dev server error patterns)
3. references/examples.md (error scenarios and solutions)
4. references/anti-patterns.md (dev server configuration mistakes)

### YAML Frontmatter
---
name: vite-errors-dev-server
description: "Diagnoses and resolves Vite dev server errors including HMR not working (full reload instead), proxy configuration failures, CORS issues, HTTPS setup problems, module resolution errors, server.fs.strict violations, port conflicts, WebSocket connection failures, file watcher issues, and middleware mode problems. Activates when encountering dev server startup failures, HMR issues, proxy errors, CORS blocks, or module not found errors during development."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- HMR not working: missing HMR boundary, full reload triggers, accept() whitespace issue
- HMR WebSocket: connection failures, server.hmr config
- Proxy: incorrect rewrite, changeOrigin missing, target format
- CORS: blocked requests, server.cors config
- HTTPS: certificate errors, server.https setup
- Module resolution: alias not working, extension not found
- server.fs.strict: file outside workspace root
- Port conflicts: server.strictPort, next port fallback
- File watcher: chokidar issues, server.watch config
- Middleware mode: missing appType: 'custom'
- server.allowedHosts: DNS rebinding protection
- Slow initial load: use server.warmup

### Research Sections to Read
From research-core-config.md:
- Section 2: Server Options
- Section 3: Dev Server
From research-plugin-hmr-ssr.md:
- Section 2: HMR (boundary concept, full reload triggers)

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Format as diagnostic table: Symptom → Cause → Fix
- Include HMR debugging flowchart
- CRITICAL: import.meta.hot.accept( whitespace requirement
```

---

### Batch 8

#### Prompt: vite-agents-review

```
## Task: Create the vite-agents-review skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-agents\vite-agents-review\

### Files to Create
1. SKILL.md (<500 lines)
2. references/checklist.md (complete validation checklist organized by area)
3. references/examples.md (good code vs bad code examples)
4. references/anti-patterns.md (consolidated anti-patterns from all skills)

### YAML Frontmatter
---
name: vite-agents-review
description: "Validates generated Vite code and configuration by checking config correctness, plugin hook signatures, HMR patterns, version-specific API usage, build optimization, environment variable security, SSR configuration, and known anti-patterns. Run this validation checklist against any Vite project to ensure correctness. Activates when reviewing Vite configuration, validating a Vite project, checking for common mistakes, or auditing Vite code before deployment."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- Config validation: defineConfig used, correct option names for target version
- Version check: rolldownOptions (v8) vs rollupOptions (v6/v7), oxc vs esbuild
- Plugin validation: correct hook signatures, enforce/apply usage, virtual module convention
- HMR validation: guard pattern, accept whitespace, dispose cleanup, data mutation (not reassignment)
- Env vars: VITE_ prefix, no secrets, TypeScript IntelliSense setup
- Build: target matches deployment, sourcemaps configured, CSS code splitting
- SSR: externals configured, middleware mode with appType: 'custom'
- Library mode: name for UMD, externals for peer deps, package.json exports
- Security: envPrefix not empty, server.fs.strict, server.allowedHosts
- Anti-pattern consolidated list from all error skills

### Research Sections to Read
All research fragments — anti-pattern sections

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Structure as runnable checklist with pass/fail criteria
- Group by area (config, plugins, HMR, env, build, SSR, library, security)
```

#### Prompt: vite-agents-project-scaffolder

```
## Task: Create the vite-agents-project-scaffolder skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\Vite-Claude-Skill-Package\skills\source\vite-agents\vite-agents-project-scaffolder\

### Files to Create
1. SKILL.md (<500 lines)
2. references/templates.md (file templates for each project type)
3. references/examples.md (complete scaffold output for React-TS, Vue-TS, vanilla)
4. references/anti-patterns.md (scaffolding mistakes)

### YAML Frontmatter
---
name: vite-agents-project-scaffolder
description: "Generates complete Vite project structure including vite.config.ts with appropriate plugins, TypeScript configuration (tsconfig.json, vite-env.d.ts), package.json with scripts and dependencies, index.html entry point, source directory structure, environment variable setup (.env, .env.example), and framework-specific configurations. Supports React, Vue, Svelte, and vanilla TypeScript projects. Activates when generating a new Vite project from scratch, adding Vite to an existing app, or scaffolding a production-ready Vite setup."
license: MIT
compatibility: "Designed for Claude Code. Requires Vite 6.x, 7.x, or 8.x."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT)
- Project structure generation: all required files and directories
- vite.config.ts: defineConfig, framework plugin, resolve.alias, build options
- TypeScript: tsconfig.json (isolatedModules, types, target), vite-env.d.ts
- package.json: scripts, dependencies, devDependencies per framework
- index.html: entry point with script module tag
- Environment setup: .env.example, .env.local in .gitignore
- .gitignore: node_modules, dist, .env.local, *.local
- Framework detection and plugin selection decision tree
- Vite version detection: v6/v7/v8 appropriate config

### Research Sections to Read
From research-core-config.md:
- Section 1: Architecture (project structure)
- Section 6: TypeScript, CSS
- Section 8-9: CLI, Templates

### Quality Rules
- English only, <500 lines SKILL.md, ALWAYS/NEVER language
- Include complete file templates
- Include framework plugin selection decision tree
- All generated code MUST follow patterns from other skills
```

---

## Appendix: Skill Directory Structure

```
skills/source/
├── vite-core/
│   ├── vite-core-architecture/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── internals.md
│   │       ├── examples.md
│   │       └── anti-patterns.md
│   └── vite-core-environment-api/
│       ├── SKILL.md
│       └── references/
├── vite-syntax/
│   ├── vite-syntax-config/
│   ├── vite-syntax-server/
│   ├── vite-syntax-build/
│   ├── vite-syntax-resolve-css/
│   ├── vite-syntax-plugin-api/
│   ├── vite-syntax-hmr-api/
│   ├── vite-syntax-assets/
│   └── vite-syntax-env-vars/
├── vite-impl/
│   ├── vite-impl-project-setup/
│   ├── vite-impl-library-mode/
│   ├── vite-impl-ssr/
│   ├── vite-impl-backend-integration/
│   ├── vite-impl-optimization/
│   ├── vite-impl-javascript-api/
│   └── vite-impl-migration/
├── vite-errors/
│   ├── vite-errors-build/
│   ├── vite-errors-dependency/
│   └── vite-errors-dev-server/
└── vite-agents/
    ├── vite-agents-review/
    └── vite-agents-project-scaffolder/
```
