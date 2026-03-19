# Vooronderzoek Vite — Consolidated Research Document

> Generated: 2026-03-19
> Sources: Official Vite documentation at https://vite.dev (fetched via WebFetch)
> Covers: Vite 6.x, 7.x, 8.x (current)
> Research fragments: docs/research/fragments/research-*.md

---

## 1. Architecture Overview

### How Vite Works: Dev vs Build

Vite consists of **two major components**:

1. **Dev Server**: Serves source code over native ES modules (ESM) with extremely fast Hot Module Replacement (HMR). No bundling during development -- modules are served as close to original source as possible.
2. **Build Command**: Bundles code for production using **Rolldown** (v8+), pre-configured to output highly optimized static assets.

### Evolution of Internal Tools

| Version | Dev Transform | Dev Pre-bundling | Production Bundler | CSS Minification |
|---------|--------------|------------------|--------------------|------------------|
| Vite 5  | esbuild      | esbuild          | Rollup             | esbuild          |
| Vite 6  | esbuild      | esbuild          | Rollup             | esbuild/lightningcss |
| Vite 7  | esbuild      | esbuild          | Rollup             | esbuild/lightningcss |
| Vite 8  | **Oxc**      | **Rolldown**     | **Rolldown**       | **Lightning CSS** |

### index.html as Entry Point

Unlike traditional bundlers, `index.html` is front-and-central -- it IS the entry point:
- `<root>/index.html` maps to `http://localhost:5173/`
- `<root>/about.html` maps to `http://localhost:5173/about.html`
- Vite resolves `<script type="module" src="...">` and processes inline modules/CSS

### Dependency Pre-Bundling

Pre-bundling serves two purposes:

1. **CommonJS/UMD to ESM conversion**: Vite serves all code as native ESM during dev, so dependencies must be converted.
2. **Performance**: Dependencies with many internal modules (e.g., lodash-es has 600+ modules) are bundled into a single module to avoid hundreds of HTTP requests.

```javascript
// Works correctly thanks to smart import analysis
import React, { useState } from 'react'
```

**How it works**:
- Vite crawls source code to discover bare imports from `node_modules`
- Pre-bundling runs via Rolldown (v8) / esbuild (v5-7) on first `vite` execution
- Results cached in `node_modules/.vite`
- Re-bundling triggers: lockfile changes, vite.config.js changes, NODE_ENV changes, patches folder changes

**Monorepos and linked deps**: Linked packages are treated as source code (not pre-bundled) unless they are non-ESM, in which case add to `optimizeDeps.include`:

```javascript
export default defineConfig({
  optimizeDeps: {
    include: ['linked-dep'],
  },
})
```

**Browser caching**: Pre-bundled deps use `max-age=31536000,immutable` HTTP headers with version query strings for cache invalidation.

### Browser Support

- **Development**: Targets esnext (latest JavaScript/CSS features), Baseline Newly Available browsers
- **Production (v8 default)**: `'baseline-widely-available'` = `['chrome111', 'edge111', 'firefox114', 'safari16.4']`
- **Legacy**: Use `@vitejs/plugin-legacy` for older browser support

### Node.js Requirements

- **Vite 7+**: Node.js 20.19+ or 22.12+ (Node 18 EOL support dropped)
- **Vite 5/6**: Node.js 18+

---

## 2. Configuration System

### Config File

Vite automatically resolves `vite.config.js` (or `.ts`, `.mjs`, `.mts`, `.cjs`, `.cts`).

```typescript
// vite.config.ts
import { defineConfig } from 'vite'

export default defineConfig({
  // config options
})
```

**Conditional config** (based on command/mode):

```typescript
export default defineConfig(({ command, mode, isSsrBuild, isPreview }) => {
  if (command === 'serve') {
    return { /* dev config */ }
  } else {
    return { /* build config */ }
  }
})
```

**Async config**:

```typescript
export default defineConfig(async ({ command, mode }) => {
  const data = await asyncFunction()
  return { /* config */ }
})
```

**Loading env variables in config**:

```typescript
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '')
  return {
    define: {
      __APP_ENV__: JSON.stringify(env.APP_ENV),
    },
  }
})
```

### Shared Options

#### root
- **Type**: `string`
- **Default**: `process.cwd()`
- Project root directory (where `index.html` is located)

#### base
- **Type**: `string`
- **Default**: `/`
- Public base path. Accepts absolute URL pathname (`/foo/`), full URL (`https://bar.com/foo/`), or empty string/`./` for embedded deployment

#### mode
- **Type**: `string`
- **Default**: `'development'` for serve, `'production'` for build
- Overrides default mode for both serve and build. Also overridable via `--mode` CLI flag

#### define
- **Type**: `Record<string, any>`
- Global constant replacements. Values must be JSON-serializable strings or single identifiers.

```javascript
export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify('v1.0.0'),
    __API_URL__: 'window.__backend_api_url',
  },
})
```

#### plugins
- **Type**: `(Plugin | Plugin[] | Promise<Plugin | Plugin[]>)[]`
- Array of plugins. Falsy plugins ignored, arrays flattened.

#### publicDir
- **Type**: `string | false`
- **Default**: `"public"`
- Static assets directory. Files served at `/` during dev, copied to `outDir` root during build, always served as-is without transform.

#### cacheDir
- **Type**: `string`
- **Default**: `"node_modules/.vite"`
- Cache directory for pre-bundled deps and other Vite-generated cache files.

### resolve.* Options

#### resolve.alias
- **Type**: `Record<string, string> | Array<{ find: string | RegExp, replacement: string }>`

```javascript
// Object format
resolve: {
  alias: {
    '@': '/src',
    utils: '../../../utils',
  }
}

// Array format (supports regex)
resolve: {
  alias: [
    { find: 'utils', replacement: '../../../utils' },
    { find: /^(.*)\.js$/, replacement: '$1.alias' }
  ]
}
```

#### resolve.dedupe
- **Type**: `string[]`
- Force Vite to resolve listed dependencies to the same copy. Useful for monorepos with duplicate dependencies.

#### resolve.conditions
- **Type**: `string[]`
- **Default**: `['module', 'browser', 'development|production']` (client), `['module', 'node', 'development|production']` (SSR)
- Additional conditions for resolving Conditional Exports. `'development|production'` replaced based on NODE_ENV.

#### resolve.mainFields
- **Type**: `string[]`
- **Default**: `['browser', 'module', 'jsnext:main', 'jsnext']`
- Fields in `package.json` to try when resolving entry points. Lower precedence than `exports` field.

#### resolve.extensions
- **Type**: `string[]`
- **Default**: `['.mjs', '.js', '.mts', '.ts', '.jsx', '.tsx', '.json']`
- File extensions to try for imports without extensions. NOT recommended to omit extensions for custom types (e.g., `.vue`).

#### resolve.preserveSymlinks
- **Type**: `boolean`
- **Default**: `false`
- Determine file identity by original path (without following symlinks).

#### resolve.tsconfigPaths
- **Type**: `boolean`
- **Default**: `false`
- Enable tsconfig `paths` resolution. TypeScript team discourages this due to performance costs.

### css.* Options

#### css.modules
- **Type**: Object with `getJSON`, `scopeBehaviour`, `globalModulePaths`, `exportGlobals`, `generateScopedName`, `hashPrefix`, `localsConvention`
- Configure CSS module behavior. Passed to postcss-modules. No effect with Lightning CSS.

#### css.postcss
- **Type**: `string | (postcss.ProcessOptions & { plugins?: postcss.AcceptedPlugin[] })`
- Inline PostCSS config or path to config directory.

#### css.preprocessorOptions
- **Type**: `Record<string, object>`
- Options for CSS preprocessors (sass/scss, less, stylus).

```javascript
export default defineConfig({
  css: {
    preprocessorOptions: {
      less: {
        math: 'parens-division',
      },
      scss: {
        additionalData: `$injectedColor: orange;`,
      },
    },
  },
})
```

#### css.preprocessorMaxWorkers
- **Type**: `number | true`
- **Default**: `true` (CPUs minus 1)
- Max threads for CSS preprocessors. `0` disables workers.

#### css.devSourcemap
- **Type**: `boolean`
- **Default**: `false`
- **Experimental**: Enable sourcemaps during dev.

#### css.transformer
- **Type**: `'postcss' | 'lightningcss'`
- **Default**: `'postcss'`
- **Experimental**: CSS processing engine selection.

#### css.lightningcss
- **Type**: Object (targets, include, exclude, drafts, nonStandard, pseudoClasses, unusedSymbols, cssModules)
- **Experimental**: Lightning CSS configuration. Use `css.lightningcss.cssModules` instead of `css.modules` when using Lightning CSS.

### html.* Options

#### html.cspNonce
- **Type**: `string`
- Nonce placeholder for script/style tags. Also generates a `<meta>` tag with the nonce value.

### json.* Options

#### json.namedExports
- **Type**: `boolean`
- **Default**: `true`
- Support named imports from `.json` files.

#### json.stringify
- **Type**: `boolean | 'auto'`
- **Default**: `'auto'`
- Convert imported JSON to `JSON.parse()` for performance. `'auto'` stringifies data larger than 10kB.

### oxc (v8+, replaces esbuild)
- **Type**: `OxcOptions | false`
- Configure Oxc transformer for JSX and file transformation. Applied to `ts`, `jsx`, `tsx` by default.

```javascript
export default defineConfig({
  oxc: {
    jsx: {
      runtime: 'classic',
      pragma: 'h',
      pragmaFrag: 'Fragment',
    },
  },
})
```

### esbuild (deprecated in v8)
- **Type**: `ESBuildOptions | false`
- **Deprecated**: Converted to `oxc` internally. Use `oxc` option instead.

### assetsInclude
- **Type**: `string | RegExp | (string | RegExp)[]`
- Additional file types to treat as static assets.

```javascript
export default defineConfig({
  assetsInclude: ['**/*.gltf'],
})
```

### logLevel
- **Type**: `'info' | 'warn' | 'error' | 'silent'`
- **Default**: `'info'`

### customLogger
- **Type**: Logger interface

```typescript
import { createLogger, defineConfig } from 'vite'

const logger = createLogger()
const loggerWarn = logger.warn
logger.warn = (msg, options) => {
  if (msg.includes('vite:css') && msg.includes(' is empty')) return
  loggerWarn(msg, options)
}

export default defineConfig({
  customLogger: logger,
})
```

### clearScreen
- **Type**: `boolean`
- **Default**: `true`
- Prevent Vite from clearing the terminal screen.

### envDir
- **Type**: `string | false`
- **Default**: `root`
- Directory for `.env` files. `false` disables `.env` loading.

### envPrefix
- **Type**: `string | string[]`
- **Default**: `VITE_`
- Env variables starting with this prefix are exposed via `import.meta.env`.
- **Security**: NEVER set to `''` -- would expose ALL env variables.

### appType
- **Type**: `'spa' | 'mpa' | 'custom'`
- **Default**: `'spa'`
- `'spa'`: HTML middlewares + SPA fallback; `'mpa'`: HTML middlewares only; `'custom'`: no HTML middlewares.

### future
- **Type**: `Record<string, 'warn' | undefined>`
- Enable future breaking changes for smooth migration.

### Server Options (server.*)

#### server.host
- **Type**: `string | boolean`
- **Default**: `'localhost'`
- Set to `0.0.0.0` or `true` to listen on all addresses (LAN + public).

#### server.allowedHosts
- **Type**: `string[] | true`
- **Default**: `[]`
- Hostnames permitted to respond. Localhost and IP addresses allowed by default.

#### server.port
- **Type**: `number`
- **Default**: `5173`
- Automatically tries next port if occupied.

#### server.strictPort
- **Type**: `boolean`
- Exit if port is in use instead of trying next available.

#### server.https
- **Type**: `https.ServerOptions`
- Enable TLS + HTTP/2. Options passed to `https.createServer()`.

#### server.open
- **Type**: `boolean | string`
- Auto-open browser. String value used as URL pathname.

#### server.proxy
- **Type**: `Record<string, string | ProxyOptions>`
- Custom proxy rules (extends `http-proxy-3`).

```javascript
export default defineConfig({
  server: {
    proxy: {
      // String shorthand
      '/foo': 'http://localhost:4567',
      // With options
      '/api': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
})
```

#### server.cors
- **Type**: `boolean | CorsOptions`
- **Default**: Allows localhost origins (`/^https?:\/\/(?:(?:[^:]+\.)?localhost|127\.0\.0\.1|\[::1\])(?::\d+)?$/`)
- Set to `true` for any origin.

#### server.headers
- **Type**: `OutgoingHttpHeaders`
- Custom server response headers.

#### server.hmr
- **Type**: `boolean | { protocol?, host?, port?, path?, timeout?, overlay?, clientPort?, server? }`
- Disable or configure HMR connection.

#### server.forwardConsole (v8+)
- **Type**: `boolean | { unhandledErrors?, logLevels? }`
- **Default**: Auto-detected (true when AI agent detected)
- Forward browser runtime events to server console.

#### server.warmup
- **Type**: `{ clientFiles?: string[], ssrFiles?: string[] }`
- Pre-transform and cache files for faster initial page load.

```javascript
export default defineConfig({
  server: {
    warmup: {
      clientFiles: ['./src/components/*.vue'],
      ssrFiles: ['./src/server/*.ts'],
    },
  },
})
```

#### server.watch
- **Type**: `object | null`
- Chokidar file system watcher options.

#### server.middlewareMode
- **Type**: `boolean`
- **Default**: `false`
- Create Vite as middleware (for custom HTTP server integration).

```javascript
import express from 'express'
import { createServer as createViteServer } from 'vite'

async function createServer() {
  const app = express()
  const vite = await createViteServer({
    server: { middlewareMode: true },
    appType: 'custom',
  })
  app.use(vite.middlewares)
}
```

#### server.fs.strict
- **Type**: `boolean`
- **Default**: `true`
- Restrict serving files outside workspace root.

#### server.fs.allow
- **Type**: `string[]`
- Directories allowed for `/@fs/` serving.

#### server.fs.deny
- **Type**: `string[]`
- **Default**: `['.env', '.env.*', '*.{crt,pem}', '**/.git/**']`
- Blocklist for sensitive files (picomatch patterns).

#### server.origin
- **Type**: `string`
- Origin for generated asset URLs during development.

#### server.sourcemapIgnoreList
- **Type**: `false | (sourcePath: string, sourcemapPath: string) => boolean`
- **Default**: `(sourcePath) => sourcePath.includes('node_modules')`

### Build Options (build.*)

#### build.target
- **Type**: `string | string[]`
- **Default**: `'baseline-widely-available'` (v7+: `['chrome111', 'edge111', 'firefox114', 'safari16.4']`)
- Can use `'esnext'` for minimal transpiling. Custom targets like `es2015`, `chrome58`.

#### build.modulePreload
- **Type**: `boolean | { polyfill?: boolean, resolveDependencies?: Function }`
- **Default**: `{ polyfill: true }`

#### build.outDir
- **Type**: `string`
- **Default**: `'dist'`

#### build.assetsDir
- **Type**: `string`
- **Default**: `'assets'`
- Not used in Library Mode.

#### build.assetsInlineLimit
- **Type**: `number | ((filePath: string, content: Buffer) => boolean | undefined)`
- **Default**: `4096` (4 KiB)
- Assets smaller than this are inlined as base64.

#### build.cssCodeSplit
- **Type**: `boolean`
- **Default**: `true`
- CSS extracted into separate chunks for async imports.

#### build.cssTarget
- **Type**: `string | string[]`
- **Default**: same as `build.target`
- Separate browser target for CSS minification.

#### build.cssMinify
- **Type**: `boolean | 'lightningcss' | 'esbuild'`
- **Default**: same as `build.minify` for client; `'lightningcss'` for SSR

#### build.sourcemap
- **Type**: `boolean | 'inline' | 'hidden'`
- **Default**: `false`
- `true` = separate file, `'inline'` = data URI, `'hidden'` = no comments.

#### build.rolldownOptions (v8+)
- **Type**: `RolldownOptions`
- Direct Rolldown bundle config customization.

#### build.rollupOptions (deprecated in v8)
- **Type**: `RolldownOptions`
- Use `build.rolldownOptions` instead.

#### build.lib
- **Type**: `{ entry, name?, formats?, fileName?, cssFileName? }`
- Library mode configuration.

```javascript
build: {
  lib: {
    entry: ['src/main.js'],
    fileName: (format, entryName) => `my-lib-${entryName}.${format}.js`,
    cssFileName: 'my-lib-style',
  },
}
```

Supported formats: `'es' | 'cjs' | 'umd' | 'iife'`

#### build.manifest
- **Type**: `boolean | string`
- **Default**: `false`
- Generates `.vite/manifest.json` mapping source files to hashed outputs.

#### build.ssrManifest
- **Type**: `boolean | string`
- **Default**: `false`
- Generates `.vite/ssr-manifest.json`.

#### build.ssr
- **Type**: `boolean | string`
- **Default**: `false`

#### build.minify
- **Type**: `boolean | 'oxc' | 'terser' | 'esbuild'`
- **Default**: `'oxc'` for client (v8+); `false` for SSR
- Oxc is 30-90x faster than Terser.

#### build.write
- **Type**: `boolean`
- **Default**: `true`
- Set `false` for programmatic builds needing post-processing.

#### build.emptyOutDir
- **Type**: `boolean`
- **Default**: `true` if `outDir` is inside `root`

#### build.copyPublicDir
- **Type**: `boolean`
- **Default**: `true`

#### build.reportCompressedSize
- **Type**: `boolean`
- **Default**: `true`
- Disable for large projects to speed up builds.

#### build.chunkSizeWarningLimit
- **Type**: `number`
- **Default**: `500` (KiB, uncompressed)

#### build.watch
- **Type**: `WatcherOptions | null`
- **Default**: `null`
- Set to `{}` to enable watch mode for builds.

#### build.license (v8+)
- **Type**: `boolean | { fileName?: string }`
- **Default**: `false`
- Generate `.vite/license.md` with bundled dependency licenses.

### Dependency Optimization Options (optimizeDeps.*)

#### optimizeDeps.entries
- **Type**: `string | string[]`
- **Default**: auto-detected from `.html` files or `build.rollupOptions.input`
- Custom entry points for dependency crawling (tinyglobby patterns). In v7+, always receives globs, not literal paths.

#### optimizeDeps.exclude
- **Type**: `string[]`
- Dependencies to exclude from pre-bundling. CommonJS deps should NOT be excluded.

#### optimizeDeps.include
- **Type**: `string[]`
- Force pre-bundling of specific packages. Supports deep import globs: `'my-lib/components/**/*.vue'`

#### optimizeDeps.rolldownOptions (v8+)
- **Type**: `Omit<RolldownOptions, 'input' | 'logLevel' | 'output'> & { output?: ... }`
- Customize Rolldown for dep scanning and optimization.

#### optimizeDeps.esbuildOptions (deprecated in v8)
- Converted internally to `rolldownOptions`.

#### optimizeDeps.force
- **Type**: `boolean`
- **Default**: `false`
- Force re-bundling, ignoring cache.

#### optimizeDeps.noDiscovery
- **Type**: `boolean`
- **Default**: `false`
- Disable automatic discovery; only optimize `include` list.

#### optimizeDeps.holdUntilCrawlEnd
- **Type**: `boolean`
- **Default**: `true`
- **Experimental**: Hold optimization until all static imports crawled.

#### optimizeDeps.needsInterop
- **Type**: `string[]`
- **Experimental**: Force ESM interop for listed dependencies.

---

## 3. Environment Variables

### Built-in Constants (import.meta.env)

| Property | Type | Description |
|----------|------|-------------|
| `import.meta.env.MODE` | `string` | Current mode |
| `import.meta.env.BASE_URL` | `string` | Base URL from `base` config |
| `import.meta.env.PROD` | `boolean` | Running in production |
| `import.meta.env.DEV` | `boolean` | Running in development |
| `import.meta.env.SSR` | `boolean` | Running server-side |

### .env File Loading Order (priority low to high)

1. `.env` -- all cases
2. `.env.local` -- all cases, gitignored
3. `.env.[mode]` -- specific mode only
4. `.env.[mode].local` -- specific mode, gitignored

Existing OS environment variables have highest priority.

### VITE_ Prefix Requirement

Only variables prefixed with `VITE_` are exposed to client code:

```
VITE_SOME_KEY=123     # Exposed: import.meta.env.VITE_SOME_KEY === "123"
DB_PASSWORD=foobar    # NOT exposed: import.meta.env.DB_PASSWORD === undefined
```

**Security**: `VITE_*` variables should NOT contain sensitive info (API keys, secrets).

### Variable Expansion (dotenv-expand)

```
KEY=123
NEW_KEY1=test$foo     # test
NEW_KEY2=test\$foo    # test$foo
NEW_KEY3=test$KEY     # test123
```

### Modes

| Command | NODE_ENV | Mode |
|---------|----------|------|
| `vite build` | `"production"` | `"production"` |
| `vite build --mode development` | `"production"` | `"development"` |
| `NODE_ENV=development vite build` | `"development"` | `"production"` |

Custom modes: `vite build --mode staging` with `.env.staging` file.

### TypeScript IntelliSense

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string
  // more env variables...
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

### HTML Env Replacement

```html
<h1>Vite is running in %MODE%</h1>
<p>Using data from %VITE_API_URL%</p>
```

Non-existent constants are silently ignored in HTML (unlike JS where they become `undefined`).

---

## 4. Plugin API

**Source**: https://vite.dev/guide/api-plugin

### 4.1 Plugin Structure and Conventions

A Vite plugin is a **factory function** returning a plugin object with a required `name` property:

```js
export default function myPlugin(options = {}) {
  return {
    name: 'my-plugin', // required, appears in warnings/errors
    // hooks follow...
  }
}
```

**Naming Conventions:**
- Rolldown plugins: `rolldown-plugin-` prefix with keywords `rolldown-plugin` and `vite-plugin`
- Vite-only plugins: `vite-plugin-` prefix with keyword `vite-plugin`
- Framework-specific: `vite-plugin-vue-`, `vite-plugin-react-`, `vite-plugin-svelte-` prefixes

**Configuration:**

```js
// vite.config.js
import vitePlugin from 'vite-plugin-feature'
import rollupPlugin from 'rollup-plugin-feature'

export default defineConfig({
  plugins: [vitePlugin(), rollupPlugin()],
})
```

Falsy plugins are ignored. Arrays are flattened, enabling plugin presets.

### 4.2 Plugin Ordering (enforce property)

The `enforce` property controls application order:

```js
return {
  name: 'my-plugin',
  enforce: 'pre', // or 'post'
}
```

**Resolution order:**
1. Alias
2. User plugins with `enforce: 'pre'`
3. Vite core plugins
4. User plugins (no enforce value)
5. Vite build plugins
6. User plugins with `enforce: 'post'`
7. Vite post build plugins

### 4.3 Conditional Application (apply property)

```js
// Apply only during build or serve
apply: 'build', // or 'serve'

// Or use function for precise control
apply(config, { command }) {
  return command === 'build' && !config.build.ssr
}
```

### 4.4 Plugin Arrays and Conditional Plugins

Falsy values are ignored, arrays are flattened:

```js
export default defineConfig({
  plugins: [
    vitePlugin(),
    condition ? optionalPlugin() : null, // falsy ignored
    [pluginA(), pluginB()],              // arrays flattened
  ],
})
```

### 4.5 Rollup-Compatible (Universal) Hooks

**Called on dev server start:**
- `options(options: InputOptions): InputOptions | null`
- `buildStart(this: PluginContext): void | Promise<void>`

**Called per module request:**
- `resolveId(this: PluginContext, id: string, importer?: string, options?: ...): string | null | Promise<...>`
- `load(this: PluginContext, id: string): string | null | Promise<...>`
- `transform(this: PluginContext, code: string, id: string): TransformResult | null | Promise<...>`

**Called on server close:**
- `buildEnd(this: PluginContext): void | Promise<void>`
- `closeBundle(): void | Promise<void>`

**IMPORTANT**: `moduleParsed` hook is NOT called during dev for performance reasons.

### 4.6 ALL Vite-Specific Hooks

#### `config` Hook

**Type:** `(config: UserConfig, env: { mode: string, command: string }) => UserConfig | null | void`
**Kind:** async, sequential

Modifies Vite config before resolution. Returns partial config object (recommended) or mutates directly:

```js
const partialConfigPlugin = () => ({
  name: 'return-partial',
  config: () => ({
    resolve: { alias: { foo: 'bar' } },
  }),
})

const mutateConfigPlugin = () => ({
  name: 'mutate-config',
  config(config, { command }) {
    if (command === 'build') {
      config.root = 'foo'
    }
  },
})
```

**IMPORTANT:** User plugins resolve before this hook, so injecting plugins here has no effect.

#### `configResolved` Hook

**Type:** `(config: ResolvedConfig) => void | Promise<void>`
**Kind:** async, parallel

Called after Vite config is resolved. Store final config for use in other hooks:

```js
const examplePlugin = () => {
  let config

  return {
    name: 'read-config',
    configResolved(resolvedConfig) {
      config = resolvedConfig
    },
    transform(code, id) {
      if (config.command === 'serve') {
        // dev: unbundled
      } else {
        // build: bundled
      }
    },
  }
}
```

Command values: `'serve'` (dev) or `'build'`.

#### `configureServer` Hook

**Type:** `(server: ViteDevServer) => (() => void) | void | Promise<...>`
**Kind:** async, sequential

Configure dev server by adding custom middlewares:

```js
const myPlugin = () => ({
  name: 'configure-server',
  configureServer(server) {
    server.middlewares.use((req, res, next) => {
      // custom middleware logic
    })
  },
})
```

**Post-middleware injection** -- return a function to add middleware after internal ones:

```js
configureServer(server) {
  return () => {
    server.middlewares.use((req, res, next) => {
      // runs after internal middlewares
    })
  }
}
```

**Storing server access for use in other hooks:**

```js
const myPlugin = () => {
  let server
  return {
    name: 'configure-server',
    configureServer(_server) {
      server = _server
    },
    transform(code, id) {
      if (server) { /* use server */ }
    },
  }
}
```

**Note:** Not called during production build.

#### `configurePreviewServer` Hook

**Type:** `(server: PreviewServer) => (() => void) | void | Promise<...>`
**Kind:** async, sequential

Identical to `configureServer` but for the preview server.

#### `transformIndexHtml` Hook

**Type:** `IndexHtmlTransformHook | { order?: 'pre' | 'post', handler: IndexHtmlTransformHook }`
**Kind:** async, sequential

Transform HTML entry files (like `index.html`):

```js
const htmlPlugin = () => ({
  name: 'html-transform',
  transformIndexHtml(html) {
    return html.replace(
      /<title>(.*?)<\/title>/,
      `<title>Title replaced!</title>`,
    )
  },
})
```

**Full type signature:**

```ts
type IndexHtmlTransformHook = (
  html: string,
  ctx: {
    path: string
    filename: string
    server?: ViteDevServer
    bundle?: OutputBundle
    chunk?: OutputChunk
  },
) => IndexHtmlTransformResult | void | Promise<IndexHtmlTransformResult | void>

type IndexHtmlTransformResult =
  | string
  | HtmlTagDescriptor[]
  | { html: string; tags: HtmlTagDescriptor[] }

interface HtmlTagDescriptor {
  tag: string
  attrs?: Record<string, string | boolean>
  children?: string | HtmlTagDescriptor[]
  injectTo?: 'head' | 'body' | 'head-prepend' | 'body-prepend'
}
```

**Order property:**
- `order: 'pre'`: Before HTML processing
- `order: 'post'` (default): After HTML processing

**Tag injection example:**

```js
transformIndexHtml: {
  order: 'pre',
  handler(html) {
    return [
      { tag: 'meta', attrs: { name: 'theme-color', content: '#fff' } },
      { tag: 'script', attrs: { src: 'path/to/script.js' }, injectTo: 'body' },
    ]
  },
}
```

#### `handleHotUpdate` Hook

**Type:** `(ctx: HmrContext) => Array<ModuleNode> | void | Promise<...>`
**Kind:** async, sequential

Custom HMR update handling:

```ts
interface HmrContext {
  file: string
  timestamp: number
  modules: Array<ModuleNode>
  read: () => string | Promise<string>
  server: ViteDevServer
}
```

**Full reload after invalidation:**

```js
handleHotUpdate({ server, modules, timestamp }) {
  const invalidatedModules = new Set()
  for (const mod of modules) {
    server.moduleGraph.invalidateModule(
      mod,
      invalidatedModules,
      timestamp,
      true
    )
  }
  server.ws.send({ type: 'full-reload' })
  return []
}
```

**Custom HMR event:**

```js
handleHotUpdate({ server }) {
  server.ws.send({
    type: 'custom',
    event: 'special-update',
    data: {}
  })
  return []
}
```

Client-side listener:

```js
if (import.meta.hot) {
  import.meta.hot.on('special-update', (data) => {
    // perform custom update
  })
}
```

### 4.7 Virtual Modules Pattern

Create virtual modules for passing build-time info via ESM imports:

```js
import { exactRegex } from '@rolldown/pluginutils'

export default function myPlugin() {
  const virtualModuleId = 'virtual:my-module'
  const resolvedVirtualModuleId = '\0' + virtualModuleId

  return {
    name: 'my-plugin',
    resolveId: {
      filter: { id: exactRegex(virtualModuleId) },
      handler() {
        return resolvedVirtualModuleId
      },
    },
    load: {
      filter: { id: exactRegex(resolvedVirtualModuleId) },
      handler() {
        return `export const msg = "from virtual module"`
      },
    },
  }
}
```

User-facing import:

```js
import { msg } from 'virtual:my-module'
console.log(msg)
```

**Convention:** Prefix virtual modules with `virtual:` for users. Internally prefix IDs with `\0` to prevent other plugin processing. During dev, `\0{id}` encodes as `/@id/__x00__{id}` in browser URLs.

### 4.8 Hook Filters (Vite 6.3.0+)

Modern hook syntax with performance optimization:

```js
transform: {
  filter: {
    id: jsFileRegex, // or exactRegex(), prefixRegex()
  },
  handler(code, id) {
    if (!jsFileRegex.test(id)) return null
    return { code: transformCode(code), map: null }
  },
}
```

### 4.9 Plugin Context Meta

Access additional Vite metadata in plugin hooks via `this.meta`:

```js
buildStart() {
  const viteVersion = this.meta.viteVersion // e.g., "8.0.0"
  if (this.meta.rolldownVersion) {
    // Rolldown-powered Vite (v8+)
  } else {
    // Rollup-powered Vite
  }
}
```

### 4.10 Output Bundle Metadata

During build, Vite augments output with `viteMetadata`:

```ts
// Available on RenderedChunk, OutputChunk, OutputAsset
viteMetadata: {
  importedCss: Set<string>
  importedAssets: Set<string>
}
```

Usage example:

```ts
function outputMetadataPlugin(): Plugin {
  return {
    name: 'output-metadata-plugin',
    generateBundle(_, bundle) {
      for (const output of Object.values(bundle)) {
        const css = output.viteMetadata?.importedCss
        const assets = output.viteMetadata?.importedAssets
        if (!css?.size && !assets?.size) continue

        console.log(output.fileName, {
          css: css ? [...css] : [],
          assets: assets ? [...assets] : [],
        })
      }
    },
  }
}
```

### 4.11 Client-Server Communication

**Server to Client:**

```js
export default defineConfig({
  plugins: [
    {
      configureServer(server) {
        server.ws.on('connection', () => {
          server.ws.send('my:greetings', { msg: 'hello' })
        })
      },
    },
  ],
})
```

**Client to Server:**

```ts
// Client side
if (import.meta.hot) {
  import.meta.hot.send('my:from-client', { msg: 'Hey!' })
}
```

```js
// Server side plugin
configureServer(server) {
  server.ws.on('my:from-client', (data, client) => {
    console.log('Message from client:', data.msg)
    client.send('my:ack', { msg: 'Hi! I got your message!' })
  })
}
```

**TypeScript Custom Events:**

```ts
// events.d.ts
import 'vite/types/customEvent.d.ts'

declare module 'vite/types/customEvent.d.ts' {
  interface CustomEventMap {
    'custom:foo': { msg: string }
  }
}
```

Then use `InferCustomEventPayload<T>` for type inference:

```ts
type CustomFooPayload = InferCustomEventPayload<'custom:foo'>
import.meta.hot?.on('custom:foo', (payload) => {
  // payload type inferred as { msg: string }
})
```

### 4.12 Path Normalization and Filtering Utilities

```js
import { normalizePath } from 'vite'
normalizePath('foo\\bar') // 'foo/bar'
```

```js
import { createFilter } from 'vite'

const filter = createFilter(
  ['src/**/*.vue'], // include
  ['node_modules/**'] // exclude
)
```

### 4.13 Rollup Plugin Compatibility Notes

Rollup plugins work in Vite if they:
- Don't use `moduleParsed` hook
- Don't rely on Rolldown-specific options
- Don't tightly couple bundle-phase and output-phase hooks

Rollup-only plugins can be scoped:

```js
import example from 'rolldown-plugin-example'

export default defineConfig({
  plugins: [
    { ...example(), enforce: 'post', apply: 'build' },
  ],
})
```

---

## 5. HMR API

**Source**: https://vite.dev/guide/api-hmr

### 5.1 Complete TypeScript Interface

```typescript
interface ViteHotContext {
  readonly data: any

  accept(): void
  accept(cb: (mod: ModuleNamespace | undefined) => void): void
  accept(dep: string, cb: (mod: ModuleNamespace | undefined) => void): void
  accept(
    deps: readonly string[],
    cb: (mods: Array<ModuleNamespace | undefined>) => void,
  ): void

  dispose(cb: (data: any) => void): void
  prune(cb: (data: any) => void): void
  invalidate(message?: string): void

  on<T extends CustomEventName>(
    event: T,
    cb: (payload: InferCustomEventPayload<T>) => void,
  ): void
  off<T extends CustomEventName>(
    event: T,
    cb: (payload: InferCustomEventPayload<T>) => void,
  ): void
  send<T extends CustomEventName>(
    event: T,
    data?: InferCustomEventPayload<T>,
  ): void
}
```

### 5.2 import.meta.hot Guard Pattern

All HMR code MUST be wrapped in a conditional guard for tree-shaking in production:

```js
if (import.meta.hot) {
  // HMR code here
}
```

### 5.3 hot.accept() -- Self-Accepting Modules

A module can accept its own updates:

```js
export const count = 1

if (import.meta.hot) {
  import.meta.hot.accept((newModule) => {
    if (newModule) {
      console.log('updated: count is now ', newModule.count)
    }
  })
}
```

**CRITICAL requirement**: Vite requires that the call appears as `import.meta.hot.accept(` (whitespace-sensitive) for static analysis to work properly.

### 5.4 hot.accept(deps, cb) -- Accepting Dependencies

Accept updates from direct dependencies without reloading self:

```js
import { foo } from './foo.js'

foo()

if (import.meta.hot) {
  // Single dependency
  import.meta.hot.accept('./foo.js', (newFoo) => {
    newFoo?.foo()
  })

  // Multiple dependencies
  import.meta.hot.accept(
    ['./foo.js', './bar.js'],
    ([newFooModule, newBarModule]) => {
      // Only updated modules are non-null; empty array on syntax error
    },
  )
}
```

### 5.5 hot.dispose(cb) -- Cleanup Side Effects

Clean up persistent side effects before a module is replaced:

```js
function setupSideEffect() {}

setupSideEffect()

if (import.meta.hot) {
  import.meta.hot.dispose((data) => {
    // cleanup side effect
  })
}
```

### 5.6 hot.prune(cb) -- Module Pruning

Registers a callback when the module is no longer imported on the page:

```js
function setupOrReuseSideEffect() {}

setupOrReuseSideEffect()

if (import.meta.hot) {
  import.meta.hot.prune((data) => {
    // cleanup side effect
  })
}
```

### 5.7 hot.invalidate(message?: string) -- Force Propagation

Forces HMR to propagate upward to importers when the module cannot handle its own update:

```js
import.meta.hot.accept((module) => {
  if (cannotHandleUpdate(module)) {
    import.meta.hot.invalidate()
  }
})
```

**Pattern**: ALWAYS call `accept()` before `invalidate()` for proper intent communication.

### 5.8 hot.data -- Persistent Data Across Updates

The data object persists across module updates:

```js
// Correct: mutate properties
import.meta.hot.data.someValue = 'hello'

// Incorrect: reassignment NOT supported
import.meta.hot.data = { someValue: 'hello' }
```

### 5.9 hot.on() / hot.off() -- Listening to Events

Built-in HMR events:

| Event | When |
|-------|------|
| `'vite:beforeUpdate'` | Update about to apply |
| `'vite:afterUpdate'` | Update applied |
| `'vite:beforeFullReload'` | Full reload imminent |
| `'vite:beforePrune'` | Modules about to be pruned |
| `'vite:invalidate'` | Module invalidated |
| `'vite:error'` | Error occurred (e.g., syntax error) |
| `'vite:ws:disconnect'` | WebSocket connection lost |
| `'vite:ws:connect'` | WebSocket (re-)established |

Custom events from plugins also dispatch via this mechanism.

### 5.10 hot.send(event, data) -- Custom Events to Server

Send custom events to Vite's dev server. Data is buffered if WebSocket is not yet connected.

### 5.11 HMR Boundary Concept

An **HMR boundary** is a module that "accepts" hot updates via `import.meta.hot.accept()`. It marks where updates stop propagating up the dependency chain.

Key detail: Vite's HMR does NOT actually swap the originally imported module. Re-exports must use `let` (not `const`) and the boundary module handles update propagation.

### 5.12 When Full Reload Happens vs HMR Update

**Full page reloads occur when:**
- No HMR boundary exists for a changed module (update propagates to root)
- The boundary's accept handler fails
- `hot.invalidate()` is called and propagates all the way up

**HMR update occurs when:**
- A changed module has an HMR boundary (self-accepting or accepted by a parent)
- The accept callback successfully handles the update

### 5.13 TypeScript Setup

Add to `tsconfig.json` for HMR type definitions:

```json
{
  "compilerOptions": {
    "types": ["vite/client"]
  }
}
```

---

## 6. SSR (Server-Side Rendering)

**Source**: https://vite.dev/guide/ssr

### 6.1 SSR Dev Workflow

Vite SSR uses **middleware mode** with a custom server (e.g., Express):

```javascript
import fs from 'node:fs'
import path from 'node:path'
import express from 'express'
import { createServer as createViteServer } from 'vite'

async function createServer() {
  const app = express()

  const vite = await createViteServer({
    server: { middlewareMode: true },
    appType: 'custom'
  })

  // Use Vite's connect instance as middleware
  app.use(vite.middlewares)

  app.use('*all', async (req, res, next) => {
    const url = req.originalUrl

    try {
      // 1. Read index.html
      let template = fs.readFileSync(
        path.resolve(import.meta.dirname, 'index.html'),
        'utf-8',
      )

      // 2. Apply Vite HTML transforms
      template = await vite.transformIndexHtml(url, template)

      // 3. Load server entry
      const { render } = await vite.ssrLoadModule('/src/entry-server.js')

      // 4. Render the app HTML
      const appHtml = await render(url)

      // 5. Inject into template
      const html = template.replace(`<!--ssr-outlet-->`, () => appHtml)

      // 6. Send rendered HTML
      res.status(200).set({ 'Content-Type': 'text/html' }).end(html)
    } catch (e) {
      vite.ssrFixStacktrace(e)
      next(e)
    }
  })

  app.listen(5173)
}

createServer()
```

**index.html placeholder:**

```html
<div id="app"><!--ssr-outlet--></div>
<script type="module" src="/src/entry-client.js"></script>
```

Update `package.json`: change `"dev": "vite"` to `"dev": "node server"`.

### 6.2 ssrLoadModule()

```javascript
const { render } = await vite.ssrLoadModule('/src/entry-server.js')
```

- Transforms ESM source to Node.js-compatible code WITHOUT bundling
- Provides HMR-like invalidation for development
- Returns the module's exports directly

**Full signature on ViteDevServer:**

```typescript
ssrLoadModule(
  url: string,
  options?: { fixStacktrace?: boolean },
): Promise<Record<string, any>>
```

### 6.3 ssrFixStacktrace()

```javascript
try {
  // server logic
} catch (e) {
  vite.ssrFixStacktrace(e)
  next(e)
}
```

Maps error stack traces back to original source code for debugging.

**Signature:**

```typescript
ssrFixStacktrace(e: Error): void
```

### 6.4 SSR Externals Configuration

| Option | Purpose |
|--------|---------|
| `ssr.noExternal` | Dependencies requiring Vite's transformation pipeline (array or `true` to bundle everything) |
| `ssr.external` | Dependencies to externalize despite being linked |
| `ssr.target` | SSR environment: `'node'` (default) or `'webworker'` |

- `ssr.noExternal: true` bundles everything into single JS file (useful for webworker target)
- `ssr.target` affects package entry resolution

### 6.5 SSR Build Config and Commands

```json
{
  "scripts": {
    "build:client": "vite build --outDir dist/client",
    "build:server": "vite build --outDir dist/server --ssr src/entry-server.js"
  }
}
```

The `--ssr` flag indicates SSR-specific build output.

**SSR Manifest for preload directives:**

```
"build:client": "vite build --outDir dist/client --ssrManifest"
```

Generates `dist/client/.vite/ssr-manifest.json` mapping module IDs to chunks/assets.

### 6.6 SSR Conditional Logic

```javascript
if (import.meta.env.SSR) {
  // server-only code
}
```

Statically replaced during build, enabling tree-shaking of unused branches.

### 6.7 CJS/ESM Handling in SSR

- `ssrLoadModule` automatically transforms ESM to Node.js-compatible code -- no bundling required
- For production, directly import the pre-built server bundle: `import('./dist/server/entry-server.js')`

### 6.8 Production Build Steps

1. Read `dist/client/index.html` (not root) as template
2. Replace `ssrLoadModule()` with `import('./dist/server/entry-server.js')`
3. Move Vite dev server creation behind `process.env.NODE_ENV` checks
4. Add static file serving from `dist/client`

### 6.9 SSR-Specific Vite Config Options

| Option | Purpose |
|--------|---------|
| `ssr.resolve.conditions` | Customize package entry resolution for SSR builds (overrides `resolve.conditions`) |
| `ssr.resolve.externalConditions` | Additional conditions for externalized dependencies |
| `ssr.noExternal` | Array of dependencies to transform through Vite |
| `ssr.external` | Array of dependencies to externalize despite being linked |
| `ssr.target` | `'node'` (default) or `'webworker'` |

### 6.10 Plugin SSR-Specific Logic

```javascript
export function mySSRPlugin() {
  return {
    name: 'my-ssr',
    transform(code, id, options) {
      if (options?.ssr) {
        // perform ssr-specific transform
      }
    },
  }
}
```

**Version note:** Before Vite 2.7, this was informed to plugin hooks with a positional `ssr` param instead of using the `options` object.

---

## 7. JavaScript API (Programmatic)

**Source**: https://vite.dev/guide/api-javascript

### 7.1 createServer()

```typescript
async function createServer(inlineConfig?: InlineConfig): Promise<ViteDevServer>
```

**Example:**

```typescript
import { createServer } from 'vite'

const server = await createServer({
  configFile: false,
  root: import.meta.dirname,
  server: {
    port: 1337,
  },
})
await server.listen()
server.printUrls()
server.bindCLIShortcuts({ print: true })
```

**Key notes:**
- When using `createServer` and `build` in the same Node.js process, set `process.env.NODE_ENV` or the `mode` config option consistently
- In middleware mode with WebSocket proxy, provide the parent HTTP server in `middlewareMode`

### 7.2 ViteDevServer Interface (Complete)

```typescript
interface ViteDevServer {
  config: ResolvedConfig
  middlewares: Connect.Server
  httpServer: http.Server | null
  watcher: FSWatcher
  ws: WebSocketServer
  pluginContainer: PluginContainer
  moduleGraph: ModuleGraph
  resolvedUrls: ResolvedServerUrls | null

  transformRequest(
    url: string,
    options?: TransformOptions,
  ): Promise<TransformResult | null>

  transformIndexHtml(
    url: string,
    html: string,
    originalUrl?: string,
  ): Promise<string>

  ssrLoadModule(
    url: string,
    options?: { fixStacktrace?: boolean },
  ): Promise<Record<string, any>>

  ssrFixStacktrace(e: Error): void
  reloadModule(module: ModuleNode): Promise<void>
  listen(port?: number, isRestart?: boolean): Promise<ViteDevServer>
  restart(forceOptimize?: boolean): Promise<void>
  close(): Promise<void>
  bindCLIShortcuts(options?: BindCLIShortcutsOptions<ViteDevServer>): void
  waitForRequestsIdle(ignoredId?: string): Promise<void>
}
```

**Key properties:**
- `config`: The resolved configuration object
- `moduleGraph`: Tracks import relationships and HMR state
- `middlewares`: Connect app for attaching custom middleware
- `watcher`: Chokidar file system watcher
- `ws`: WebSocket server for HMR communication
- `pluginContainer`: Plugin container for running plugin hooks

### 7.3 build()

```typescript
async function build(
  inlineConfig?: InlineConfig,
): Promise<RollupOutput | RollupOutput[]>
```

**Example:**

```typescript
import path from 'node:path'
import { build } from 'vite'

await build({
  root: path.resolve(import.meta.dirname, './project'),
  base: '/foo/',
  build: {
    rollupOptions: {
      // ...
    },
  },
})
```

### 7.4 preview()

```typescript
async function preview(inlineConfig?: InlineConfig): Promise<PreviewServer>
```

**Example:**

```typescript
import { preview } from 'vite'

const previewServer = await preview({
  preview: {
    port: 8080,
    open: true,
  },
})

previewServer.printUrls()
previewServer.bindCLIShortcuts({ print: true })
```

**PreviewServer interface:**

```typescript
interface PreviewServer {
  config: ResolvedConfig
  middlewares: Connect.Server
  httpServer: http.Server
  resolvedUrls: ResolvedServerUrls | null
  printUrls(): void
  bindCLIShortcuts(options?: BindCLIShortcutsOptions<PreviewServer>): void
}
```

### 7.5 resolveConfig()

```typescript
async function resolveConfig(
  inlineConfig: InlineConfig,
  command: 'build' | 'serve',
  defaultMode = 'development',
  defaultNodeEnv = 'development',
  isPreview = false,
): Promise<ResolvedConfig>
```

The `command` parameter is `serve` for dev/preview, `build` for production builds.

### 7.6 mergeConfig()

```typescript
function mergeConfig(
  defaults: Record<string, any>,
  overrides: Record<string, any>,
  isRoot = true,
): Record<string, any>
```

Deeply merges two Vite configs. Set `isRoot` to `false` when merging nested options like `build`.

### 7.7 Important TypeScript Types

**InlineConfig:** Extends `UserConfig` with additional properties:
- `configFile`: Specify config file path; set to `false` to disable auto-resolving

**ResolvedConfig:** Has all `UserConfig` properties (resolved and non-undefined) plus utilities:
- `config.assetsInclude`: Function checking if an ID is an asset
- `config.logger`: Internal logger object

### 7.8 Utility Functions

**searchForWorkspaceRoot():**

```typescript
function searchForWorkspaceRoot(
  current: string,
  root = searchForPackageRoot(current),
): string
```

**loadEnv():**

```typescript
function loadEnv(
  mode: string,
  envDir: string,
  prefixes: string | string[] = 'VITE_',
): Record<string, string>
```

**normalizePath():**

```typescript
function normalizePath(id: string): string
```

### 7.9 Transform Functions

**transformWithOxc() (Current):**

```typescript
async function transformWithOxc(
  code: string,
  filename: string,
  options?: OxcTransformOptions,
  inMap?: object,
): Promise<Omit<OxcTransformResult, 'errors'> & { warnings: string[] }>
```

**transformWithEsbuild() (Deprecated -- use transformWithOxc):**

```typescript
async function transformWithEsbuild(
  code: string,
  filename: string,
  options?: EsbuildTransformOptions,
  inMap?: object,
): Promise<ESBuildTransformResult>
```

### 7.10 loadConfigFromFile()

```typescript
async function loadConfigFromFile(
  configEnv: ConfigEnv,
  configFile?: string,
  configRoot: string = process.cwd(),
  logLevel?: LogLevel,
  customLogger?: Logger,
): Promise<{ path: string; config: UserConfig; dependencies: string[] } | null>
```

### 7.11 preprocessCSS() (Experimental)

```typescript
async function preprocessCSS(
  code: string,
  filename: string,
  config: ResolvedConfig,
): Promise<PreprocessCSSResult>

interface PreprocessCSSResult {
  code: string
  map?: SourceMapInput
  modules?: Record<string, string>
  deps?: Set<string>
}
```

### 7.12 Version Constants

| Constant | Description |
|----------|-------------|
| `version` | Current Vite version string (e.g., "8.0.0") |
| `rolldownVersion` | Rolldown version used (e.g., "1.0.0") |
| `esbuildVersion` | For backward compatibility only |
| `rollupVersion` | For backward compatibility only |

---

## 8. Environment API -- Vite 6

### What the Environment API Is

The Environment API, introduced in Vite 6, formalizes how Vite handles different execution contexts. Previously, Vite implicitly supported only `client` and optional `ssr` environments. Vite 6 allows explicit configuration of **multiple named environments** to match production setups more closely (e.g., browser, Node server, edge workers).

The API solves "Closing the Gap Between Build and Dev" by running server code in matching runtimes during development.

### Backward Compatibility

For simple SPA/MPA apps, no new APIs are required. Vite internally applies options to a `client` environment without exposing the concept. The existing server API remains compatible.

### Per-Environment Configuration

**Basic (unchanged for SPAs):**
```javascript
export default defineConfig({
  build: { sourcemap: false },
  optimizeDeps: { include: ['lib'] }
})
```

**Multi-environment apps:**
```javascript
export default {
  build: { sourcemap: false },
  optimizeDeps: { include: ['lib'] },
  environments: {
    server: {},
    edge: {
      resolve: { noExternal: true }
    }
  }
}
```

### Configuration Inheritance

Environments inherit top-level options by default. However, some options like `optimizeDeps` apply **only** to `client` environments and will NOT propagate to server environments automatically.

### EnvironmentOptions Interface

Includes:
- `define` -- Define global constant replacements
- `resolve` -- Module resolution options
- `optimizeDeps` -- Dependency pre-bundling options
- `consumer` -- `'client'` | `'server'`
- `dev` -- Dev-specific nested options
- `build` -- Build-specific nested options

`UserConfig` extends `EnvironmentOptions` and adds:
- `environments: Record<string, EnvironmentOptions>`

### Custom Environment Providers (Runtime Support)

Custom environments can be provided by runtime providers. Example with Cloudflare:
```javascript
import { customEnvironment } from 'vite-environment-provider'

export default {
  environments: {
    ssr: customEnvironment({
      build: { outDir: '/dist/ssr' }
    })
  }
}
```

### Shared Plugins vs Per-Environment Plugins

- **Shared plugins**: Applied across all environments by default (existing behavior)
- **Per-environment plugins**: Can be scoped to specific environments
- Planned deprecation: moving from implicit to explicit per-environment plugin hooks

### Release Status

The Environment API is in **release candidate** phase. Stability maintained between major releases, but some specific APIs remain experimental. The ecosystem should experiment before stabilization in a future major version.

### Target Audiences

| Audience | What They Do |
|----------|-------------|
| End Users | Use basic config for SPAs; explicitly configure environments for SSR/complex apps |
| Plugin Authors | Access more consistent APIs for environment interaction |
| Framework Authors | Programmatic environment configuration capabilities |
| Runtime Providers | Offer custom environments for their platforms |

### Migration from Vite 5

- SSR will move to ModuleRunner API
- Per-environment plugin hooks will become explicit (currently implicit)
- Existing top-level config continues to work unchanged for single-environment apps

---

## 9. Static Asset Handling

### Importing Assets as URLs

When you import a static asset, Vite returns a resolved public URL:

```js
import imgUrl from './img.png'
document.getElementById('hero-img').src = imgUrl
```

- **Development**: `imgUrl` resolves to `/src/img.png`
- **Production**: becomes `/assets/img.2d8efhg.png` (with content hash)

Key behaviors:
- CSS `url()` references are handled the same way
- Vue plugin automatically converts asset references in templates to imports
- Common image, media, and font file types are automatically detected as assets
- Assets smaller than `assetsInlineLimit` become base64 data URLs
- Git LFS placeholders are excluded from inlining

### URL Query Suffixes

#### `?url` -- Explicit URL import
For assets not in the auto-detected list:
```js
import workletURL from 'extra-scalloped-border/worklet.js?url'
CSS.paintWorklet.addModule(workletURL)
```

#### `?raw` -- Import as string
```js
import shaderString from './shader.glsl?raw'
```

#### `?inline` / `?no-inline` -- Control inlining behavior
```js
import imgUrl1 from './img.svg?no-inline'
import imgUrl2 from './img.png?inline'
```

#### `?worker` / `?sharedworker` -- Import as web worker
```js
import Worker from './shader.js?worker'
const worker = new Worker()
```

Combined suffixes: `?worker&inline` inlines workers as base64 strings.

### `new URL(url, import.meta.url)` Pattern

Uses native ESM features:
```js
const imgUrl = new URL('./img.png', import.meta.url).href
document.getElementById('hero-img').src = imgUrl
```

Supports dynamic URLs via template literals:
```js
function getImageUrl(name) {
  return new URL(`./dir/${name}.png`, import.meta.url).href
}
```

Vite transforms this during builds to maintain correct paths after bundling and hashing.

**Constraints:**
- URL strings must be **static** so they can be analyzed (or code remains unchanged)
- Does NOT work with SSR -- `import.meta.url` has different semantics in browsers vs Node.js

### Public Directory

Place assets in `<root>/public` (configurable via `publicDir` option) for files that:
- Are never referenced in source code (e.g., `robots.txt`)
- Must retain exact file names without hashing
- Don't require prior imports for URL access

Assets served at root `/` during development, copied to dist root unchanged.

Reference as absolute paths: `public/icon.png` -> `/icon.png` in code.

**Recommendation**: "Prefer importing assets unless you specifically need the guarantees provided by the public directory."

### JSON Imports

JSON files can be directly imported; named imports are supported:
```js
// import the entire object
import json from './example.json'

// import a named field (tree-shakeable)
import { field } from './example.json'
```

### Glob Import (`import.meta.glob`)

#### Basic Usage
```js
const modules = import.meta.glob('./dir/*.js')
// Transforms to:
const modules = {
  './dir/bar.js': () => import('./dir/bar.js'),
  './dir/foo.js': () => import('./dir/foo.js'),
}
```

Iterate:
```js
for (const path in modules) {
  modules[path]().then((mod) => {
    console.log(path, mod)
  })
}
```

#### Eager Loading
```js
const modules = import.meta.glob('./dir/*.js', { eager: true })
// Transforms to:
import * as __vite_glob_0_0 from './dir/bar.js'
import * as __vite_glob_0_1 from './dir/foo.js'
const modules = {
  './dir/bar.js': __vite_glob_0_0,
  './dir/foo.js': __vite_glob_0_1,
}
```

#### Multiple Patterns
```js
const modules = import.meta.glob(['./dir/*.js', './another/*.js'])
```

#### Negative Patterns (Exclusion)
```js
const modules = import.meta.glob(['./dir/*.js', '!**/bar.js'])
// Results in only foo.js
```

#### Named Imports
```ts
const modules = import.meta.glob('./dir/*.js', { import: 'setup' })
// Transforms to:
const modules = {
  './dir/bar.js': () => import('./dir/bar.js').then((m) => m.setup),
  './dir/foo.js': () => import('./dir/foo.js').then((m) => m.setup),
}
```

With eager for tree-shaking:
```ts
const modules = import.meta.glob('./dir/*.js', {
  import: 'setup',
  eager: true,
})
// Transforms to:
import { setup as __vite_glob_0_0 } from './dir/bar.js'
import { setup as __vite_glob_0_1 } from './dir/foo.js'
```

Use `import: 'default'` for default exports.

#### Custom Queries
```ts
const moduleStrings = import.meta.glob('./dir/*.svg', {
  query: '?raw',
  import: 'default',
})
const moduleUrls = import.meta.glob('./dir/*.svg', {
  query: '?url',
  import: 'default',
})
```

Custom queries for plugin consumption:
```ts
const modules = import.meta.glob('./dir/*.js', {
  query: { foo: 'bar', bar: true },
})
```

#### Base Path Option
```ts
const modulesWithBase = import.meta.glob('./**/*.js', {
  base: './base',
})
// Transforms to:
const modulesWithBase = {
  './dir/foo.js': () => import('./base/dir/foo.js'),
  './dir/bar.js': () => import('./base/dir/bar.js'),
}
```

Base can only be a directory path relative to importer or absolute against project root. Aliases and virtual modules are NOT supported.

#### Caveats
- Vite-only feature, NOT a web/ES standard
- Glob patterns must be relative (`./`), absolute (`/`), or alias paths
- Matching via `tinyglobby`
- Arguments MUST be string literals -- no variables or expressions

---

## 10. Library Mode

### Why Library Mode

Library Mode allows building reusable libraries (npm packages) with Vite instead of full applications. Entry point is your library code, not an HTML file.

### `build.lib` Configuration

#### Single Entry
```javascript
// vite.config.js
import { resolve } from 'path'
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    lib: {
      entry: resolve(import.meta.dirname, 'lib/main.js'),
      name: 'MyLib',        // global variable name for UMD/IIFE
      fileName: 'my-lib',   // output file name (without extension)
    },
    rolldownOptions: {
      external: ['vue'],
      output: {
        globals: {
          vue: 'Vue'
        }
      }
    }
  }
})
```

#### Multiple Entry Points
```javascript
build: {
  lib: {
    entry: {
      'my-lib': resolve(import.meta.dirname, 'lib/main.js'),
      secondary: resolve(import.meta.dirname, 'lib/secondary.js'),
    },
    name: 'MyLib',
  }
}
```

### Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `entry` | `string \| string[] \| Record<string, string>` | Entry point(s) for the library |
| `name` | `string` | Global variable name for UMD/IIFE builds |
| `fileName` | `string \| ((format, entryName) => string)` | Output file name (auto-adds format extension) |
| `formats` | `string[]` | Output formats (default: `['es', 'umd']` for single entry, `['es', 'cjs']` for multiple) |
| `cssFileName` | `string` | Custom CSS output filename |

### Default Output Formats

| Entry Type | Default Formats |
|-----------|-----------------|
| Single entry | `es` and `umd` |
| Multiple entries | `es` and `cjs` |

### External Dependencies

Dependencies that should NOT be bundled into the library (e.g., framework peer dependencies):

```javascript
build: {
  rolldownOptions: {
    external: ['vue'],
    output: {
      globals: {
        vue: 'Vue'   // required for UMD builds
      }
    }
  }
}
```

### CSS Handling in Library Mode

CSS is bundled as a separate file (e.g., `dist/my-lib.css`). Customize filename with `build.lib.cssFileName`.

Export in `package.json`:
```json
{
  "exports": {
    "./style.css": "./dist/my-lib.css"
  }
}
```

### Recommended `package.json` Exports

```json
{
  "name": "my-lib",
  "type": "module",
  "files": ["dist"],
  "main": "./dist/my-lib.umd.cjs",
  "module": "./dist/my-lib.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.js",
      "require": "./dist/my-lib.umd.cjs"
    },
    "./style.css": "./dist/my-lib.css"
  }
}
```

### DTS Generation

Vite does NOT generate TypeScript declarations natively. Use `vite-plugin-dts` for `.d.ts` generation:
```bash
npm add -D vite-plugin-dts
```

---

## 11. Dependency Pre-bundling

### Why Pre-Bundling Exists

When you run `vite` for the first time, Vite pre-bundles your project dependencies before loading your site locally. Two main reasons:

#### 1. CommonJS/UMD to ESM Conversion
During development, Vite serves all code as native ESM. Dependencies published as CommonJS or UMD must be converted.

"Vite performs smart import analysis so that named imports to CommonJS modules will work as expected even if the exports are dynamically assigned."

```javascript
// works as expected even though React is CJS
import React, { useState } from 'react'
```

#### 2. Performance -- Reducing Module Count
"Vite converts ESM dependencies with many internal modules into a single module to improve subsequent page load performance."

Example: `lodash-es` has over 600 internal modules. Without pre-bundling, `import { debounce } from 'lodash-es'` triggers 600+ HTTP requests. After pre-bundling, it becomes a single module.

### Build Tool: Rolldown

Pre-bundling uses **Rolldown** (in Vite 6; previously esbuild in Vite 5). "The pre-bundling is performed with Rolldown so it's typically very fast."

> **Version note**: Vite 5 used esbuild for pre-bundling; Vite 6 switched to Rolldown.

### Automatic Dependency Discovery

Vite crawls source code to identify bare imports automatically. "If a new dependency import is encountered that isn't already in the cache, Vite will re-run the dep bundling process and reload the page if needed."

### `optimizeDeps` Configuration

#### Basic Structure
```javascript
export default defineConfig({
  optimizeDeps: {
    include: ['linked-dep'],
  },
})
```

#### `optimizeDeps.include`
Force pre-bundling of specific dependencies. Use when:
- Automatic discovery fails (imports come from plugin transforms)
- Large ESM dependencies with many internal modules
- CommonJS dependencies that need conversion

```javascript
optimizeDeps: {
  include: ['linked-dep', 'lodash-es']
}
```

#### `optimizeDeps.exclude`
Exclude specific dependencies from pre-bundling. Use for:
- Small, already-valid ESM packages

#### `optimizeDeps.force`
Force re-bundling, ignoring previously cached results.

#### `optimizeDeps.rolldownOptions`
Customize the Rolldown bundler used for pre-bundling. Allows adding plugins or modifying build targets.

> **Version note**: In Vite 5 this was `optimizeDeps.esbuildOptions`.

### Monorepo and Linked Dependencies

Vite automatically detects linked packages not from `node_modules`. "It will not attempt to bundle the linked dep, and will analyze the linked dep's dependency list instead."

**Requirements:**
- Linked dependencies MUST export ESM
- If they don't, add them to `optimizeDeps.include`
- For changes to linked deps, restart the dev server with `--force` flag

### Caching Behavior

#### File System Cache
Vite caches pre-bundled dependencies in `node_modules/.vite`. Re-bundling occurs when:

| Trigger | Description |
|---------|-------------|
| Package manager lockfile | `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, etc. |
| Patches folder | Modification time changes |
| `vite.config.js` fields | Relevant config fields change |
| `NODE_ENV` value | Environment variable changes |

**Force re-bundling:**
- CLI: `vite --force` or `vite dev --force`
- Manual: Delete `node_modules/.vite` directory

#### Browser Cache
Resolved dependency requests use HTTP headers: `max-age=31536000,immutable` for aggressive caching. "Once cached, these requests will never hit the dev server again. They are auto invalidated by the appended version query if a different version is installed."

**Debugging locally edited dependencies:**
1. Temporarily disable browser cache via DevTools Network tab
2. Restart dev server with `--force` flag
3. Reload the page

### Important Note
"Dependency pre-bundling only applies in development mode." In production builds, `@rollup/plugin-commonjs` is used instead (Vite 5) or Rolldown handles it natively (Vite 6).

---

## 12. Additional Features

### CSS Features

#### CSS Modules
Files ending with `.module.css` are treated as CSS Modules:

```css
/* example.module.css */
.red {
  color: red;
}
```

```js
import classes from './example.module.css'
document.getElementById('foo').className = classes.red
```

Configure via `css.modules` option. With `localsConvention: 'camelCaseOnly'`, named imports work:
```js
import { applyColor } from './example.module.css'
```

#### CSS Preprocessors
Built-in support (install preprocessor only, no Vite plugins needed):

```bash
# Sass
npm add -D sass-embedded  # or sass

# Less
npm add -D less

# Stylus
npm add -D stylus
```

Supported extensions: `.scss`, `.sass`, `.less`, `.styl`, `.stylus`

Features:
- `@import` resolving respects Vite aliases
- URL references in different directories auto-rebase
- Combine with `.module` extension: e.g., `style.module.scss`

#### PostCSS
Auto-applies valid PostCSS config (formats supported by `postcss-load-config`). CSS minification runs after PostCSS using `build.cssTarget` option.

#### Lightning CSS (Experimental)
Available since Vite 4.4. Enable:

```bash
npm add -D lightningcss
```

```js
// vite.config.js
export default defineConfig({
  css: {
    transformer: 'lightningcss',
    lightningcss: {
      // Lightning CSS options
    }
  },
  build: {
    cssMinify: 'lightningcss'
  }
})
```

#### `@import` Inlining and Rebasing
Pre-configured via `postcss-import`. Vite aliases respected in `@import`. All CSS `url()` references auto-rebase for correctness.

#### CSS Injection Disabling
Use `?inline` query to prevent automatic injection:
```js
import './foo.css'                         // injected into page
import otherStyles from './bar.css?inline' // NOT injected, returned as string
```

### TypeScript Support

#### Transpile Only
Vite only **transpiles** `.ts` files -- no type checking. Type checking assumed handled by IDE or separate process.

Vite uses esbuild for TS transpilation: 20-30x faster than `tsc`. HMR updates reflect in under 50ms.

**For type checking:**
- Production: run `tsc --noEmit` alongside `vite build`
- Development: run `tsc --noEmit --watch` separately, or use `vite-plugin-checker`

#### Type-Only Imports
Prevent incorrect bundling:
```ts
import type { T } from 'only/types'
export type { T }
```

#### Required `tsconfig.json` Settings

**`isolatedModules` -- MUST be `true`:**
```json
{
  "compilerOptions": {
    "isolatedModules": true
  }
}
```
esbuild performs transpilation without type info; doesn't support `const enum` or implicit type-only imports.

**`useDefineForClassFields`:**
- Defaults to `true` if target is ES2022+/ESNext
- Defaults to `false` otherwise

**`target`:**
- Vite IGNORES `tsconfig.json` `target` value
- Use `esbuild.target` in dev (defaults to `esnext`)
- `build.target` takes priority in production builds

**`emitDecoratorMetadata`:**
Partially supported -- full support requires TypeScript compiler type inference.

**`paths` (path aliases):**
Enable with `resolve.tsconfigPaths: true` in Vite config.

#### Client Types
Add to `tsconfig.json`:
```json
{
  "compilerOptions": {
    "types": ["vite/client"]
  }
}
```

Or use triple-slash directive in `vite-env.d.ts`:
```ts
/// <reference types="vite/client" />
```

Provides type shims for:
- Asset imports (`.svg`, `.png`, etc.)
- `import.meta.env` types
- HMR API types on `import.meta.hot`

### JSX and TSX Support

`.jsx` and `.tsx` supported out of the box via esbuild transpilation.

Framework plugins (e.g., `@vitejs/plugin-vue-jsx`, `@vitejs/plugin-react`) configure JSX automatically with HMR.

Custom JSX configuration (non-framework):
```js
export default defineConfig({
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment',
  },
})
```

Auto-inject JSX helpers:
```js
export default defineConfig({
  esbuild: {
    jsxInject: `import React from 'react'`,
  },
})
```

### Web Workers

#### Constructor Pattern (Recommended)
```ts
const worker = new Worker(new URL('./worker.js', import.meta.url))
```

With module worker option:
```ts
const worker = new Worker(new URL('./worker.js', import.meta.url), {
  type: 'module',
})
```

**Constraint**: Worker detection requires `new URL()` directly inside `new Worker()`. All option parameters must be static values.

#### Query Suffix Pattern
```js
import MyWorker from './worker?worker'
const worker = new MyWorker()
```

Inline as base64:
```js
import MyWorker from './worker?worker&inline'
```

Retrieve as URL only:
```js
import MyWorker from './worker?worker&url'
```

Worker scripts can use ESM `import` (native in dev, compiled away for production).

### WebAssembly

Pre-compiled `.wasm` files imported with `?init`:
```js
import init from './example.wasm?init'

init().then((instance) => {
  instance.exports.test()
})
```

Pass `importObject`:
```js
init({
  imports: {
    someFunc: () => { /* ... */ },
  },
}).then(() => { /* ... */ })
```

Production: `.wasm` files smaller than `assetInlineLimit` inline as base64; otherwise treated as static asset.

For multiple instantiations, use explicit URL import:
```js
import wasmUrl from 'foo.wasm?url'

const main = async () => {
  const responsePromise = fetch(wasmUrl)
  const { module, instance } =
    await WebAssembly.instantiateStreaming(responsePromise)
}
main()
```

### Build Optimizations

#### CSS Code Splitting
Vite auto-extracts CSS from async chunks into separate files. CSS auto-loads via `<link>` tag before async chunk evaluation (prevents FOUC).

Disable: `build.cssCodeSplit: false`

#### Preload Directives Generation
Vite auto-generates `<link rel="modulepreload">` directives for entry chunks and direct imports in built HTML.

#### Async Chunk Loading Optimization
When async chunk A imports common chunk C, Vite rewrites to fetch both in parallel:
```
Entry ---> (A + C)  [parallel fetch]
```
Eliminates roundtrips regardless of import depth.

### Dynamic Import
```ts
const module = await import(`./dir/${file}.js`)
```

Variables represent file names one level deep only. Must follow rules:
- Start with `./` or `../`
- End with file extension
- For same-directory: requires prefix pattern like `./prefix-${foo}.js`

### Content Security Policy (CSP)

**Nonce-based:**
Set `html.cspNonce` to add nonce attribute to all `<script>`, `<style>`, and `<link>` tags. Vite injects `<meta property="csp-nonce" nonce="PLACEHOLDER" />`. Replace placeholder per request.

**Data URIs:**
Allow `data:` for relevant directives, or disable inlining via `build.assetsInlineLimit: 0`. NEVER allow `data:` for `script-src`.

### Load Error Handling

Listen for `vite:preloadError` events for failed dynamic imports:
```js
window.addEventListener('vite:preloadError', (event) => {
  // handle error -- e.g., reload page
})
```
Occurs when user assets are outdated after deployments.

### HTML as Entry Points

HTML files serve as project entry points. Files in project root are directly accessible:
- `<root>/index.html` -> `http://localhost:5173/`
- `<root>/about.html` -> `http://localhost:5173/about.html`

Supported HTML asset elements:
- `<script type="module" src>`
- `<link href>` (stylesheets)
- `<img src>`, `<img srcset>`
- `<video src>`, `<video poster>`
- `<audio src>`, `<source src>`
- `<embed src>`, `<object data>`
- `<track src>`, `<input src>`
- `<meta content>` (for specific name/property attributes)

Opt-out of asset processing: add `vite-ignore` attribute to the element.

### Multi-Page App

Specify multiple HTML entry points:
```javascript
build: {
  rolldownOptions: {
    input: {
      main: resolve(import.meta.dirname, 'index.html'),
      nested: resolve(import.meta.dirname, 'nested/index.html'),
    }
  }
}
```

### Watch Mode (Rebuild on Changes)

Enable with `vite build --watch` or configure `build.watch` with WatcherOptions.

---

## 13. Version Differences (Consolidated)

### Internal Tooling Evolution

| Version | Dev Transform | Dev Pre-bundling | Production Bundler | CSS Minification |
|---------|--------------|------------------|--------------------|------------------|
| Vite 5  | esbuild      | esbuild          | Rollup             | esbuild          |
| Vite 6  | esbuild      | esbuild          | Rollup             | esbuild/lightningcss |
| Vite 7  | esbuild      | esbuild          | Rollup             | esbuild/lightningcss |
| Vite 8  | **Oxc**      | **Rolldown**     | **Rolldown**       | **Lightning CSS** |

### API and Configuration Differences

| Area | v5/v6 | v8+ (latest docs) |
|------|-------|--------------------|
| Bundler | Rollup-based | Rolldown-based |
| Transform | esbuild (`transformWithEsbuild`) | OXC (`transformWithOxc`) |
| Hook filters | Not available | `filter` property on hooks (v6.3.0+) |
| Plugin utils | `@rollup/pluginutils` | `@rolldown/pluginutils` (with `exactRegex`, `prefixRegex`) |
| Plugin context meta | `this.meta.viteVersion` | Also `this.meta.rolldownVersion` |
| SSR plugin params | `options.ssr` on options object | Same (changed from positional in v2.7) |
| Domain | vitejs.dev | vite.dev (301 redirect) |

### Feature Differences: Vite 5 vs Vite 6

| Feature | Vite 5 | Vite 6 |
|---------|--------|--------|
| Environment API | Implicit client + ssr only | Explicit multi-environment support |
| Pre-bundling tool | esbuild | Rolldown |
| `optimizeDeps` bundler options | `optimizeDeps.esbuildOptions` | `optimizeDeps.rolldownOptions` |
| Build internals | Rollup-based | Rolldown-based (with `rolldownOptions`) |
| License generation | Not built-in | `build.license: true` |
| SSR handling | `server.ssrLoadModule()` | ModuleRunner API (planned migration) |
| Plugin hooks | Implicit environment handling | Moving toward explicit per-environment hooks |

**Note**: The current official docs (vite.dev) describe Vite 8.x which uses Rolldown. For Vite 5.x/6.x projects, the Rollup-based APIs apply. The hook signatures and plugin structure remain compatible across versions -- the main differences are the underlying bundler engine and transform tool.

---

## 14. Migration Guides

### Vite 5 to 6 Breaking Changes

1. **Environment API**: New experimental Environment API with internal refactoring
2. **Vite Runtime API removed**: Replaced by Module Runner API
3. **resolve.conditions defaults changed**: Now explicitly `['module', 'browser', 'development|production']`
4. **json.stringify default**: Changed to `'auto'` (stringifies only large files >10kB)
5. **HTML asset processing extended**: More HTML elements process asset references
6. **PostCSS**: Updated to postcss-load-config v6 (requires `tsx`/`jiti` for TS configs, not `ts-node`)
7. **Sass**: Modern API is now default; legacy via `api: 'legacy'`
8. **Library mode CSS**: Filename uses `package.json` name instead of `style.css`
9. **build.cssMinify**: Enabled by default for SSR
10. **CommonJS strictRequires**: Now `true` by default (was `'auto'`)

### Vite 6 to 7 Breaking Changes

1. **Node.js 20.19+ / 22.12+ required** (18 dropped)
2. **build.target default**: Updated to `'baseline-widely-available'` (Chrome 107, Edge 107, Firefox 104, Safari 16.0)
3. **Sass legacy API removed**: Only modern API supported
4. **Removed**: `splitVendorChunkPlugin`, hook-level `enforce`/`transform` for `transformIndexHtml`
5. **optimizeDeps.entries**: Always receives globs, not literal paths

### Vite 7 to 8 Breaking Changes

1. **Rolldown replaces Rollup**: `build.rolldownOptions` replaces `build.rollupOptions`
2. **Oxc replaces esbuild**: `oxc` config replaces `esbuild` config
3. **Lightning CSS default**: CSS minification uses Lightning CSS by default
4. **Minifier**: Oxc Minifier replaces esbuild (30-90x faster than Terser)
5. **CommonJS interop**: Consistent handling of `default` imports from CJS modules
6. **Module resolution**: Removed format sniffing heuristic between `browser`/`module` fields
7. **esbuild optional**: No longer direct dependency; install manually if plugins need it
8. **build.target default**: Updated to Chrome 111, Edge 111, Firefox 114, Safari 16.4
9. **`import.meta.url` in UMD/IIFE**: No longer polyfilled
10. **Removed**: `build.rollupOptions.watch.chokidar`, object form of `output.manualChunks`
11. **Plugin authors**: Must add `moduleType: 'js'` when converting non-JS content in load/transform hooks
12. **build() API**: Throws `BundleError` instead of raw error

```javascript
// v8: Handling build errors
try {
  await build()
} catch (e) {
  if (e.errors) {
    for (const error of e.errors) {
      console.log(error.code)
    }
  }
}
```

---

## 15. Backend Integration

### Configuration for Backend Integration

```javascript
// vite.config.js
export default defineConfig({
  server: {
    cors: {
      origin: 'http://my-backend.example.com',
    },
  },
  build: {
    manifest: true,
    rollupOptions: {
      input: '/path/to/main.js',
    },
  },
})
```

Include modulepreload polyfill:
```javascript
import 'vite/modulepreload-polyfill'
```

### Development Mode HTML

```html
<!-- Vite dev server scripts -->
<script type="module" src="http://localhost:5173/@vite/client"></script>
<script type="module" src="http://localhost:5173/main.js"></script>
```

**React-specific preamble** (before above scripts):
```html
<script type="module">
  import RefreshRuntime from 'http://localhost:5173/@react-refresh'
  RefreshRuntime.injectIntoGlobalHook(window)
  window.$RefreshReg$ = () => {}
  window.$RefreshSig$ = () => (type) => type
  window.__vite_plugin_react_preamble_installed__ = true
</script>
```

### Production: manifest.json

After `vite build`, `.vite/manifest.json` is generated:

```json
{
  "views/foo.js": {
    "file": "assets/foo-BRBmoGS9.js",
    "name": "foo",
    "src": "views/foo.js",
    "isEntry": true,
    "imports": ["_shared-B7PI925R.js"],
    "css": ["assets/foo-5UjPuW-k.css"]
  }
}
```

**ManifestChunk properties**: `src`, `file`, `css`, `assets`, `isEntry`, `name`, `isDynamicEntry`, `imports`, `dynamicImports`

**Rendering order for each entry**:
1. `<link rel="stylesheet">` for entry CSS
2. Recursively include `<link rel="stylesheet">` for imported chunks' CSS
3. `<script type="module">` for the entry file
4. Optional `<link rel="modulepreload">` for imported JS chunks

```html
<link rel="stylesheet" href="assets/foo-5UjPuW-k.css" />
<link rel="stylesheet" href="assets/shared-ChJ_j-JJ.css" />
<script type="module" src="assets/foo-BRBmoGS9.js"></script>
<link rel="modulepreload" href="assets/shared-B7PI925R.js" />
```

### TypeScript Helper for Manifest Traversal

```typescript
import type { Manifest, ManifestChunk } from 'vite'

export default function importedChunks(
  manifest: Manifest,
  name: string,
): ManifestChunk[] {
  const seen = new Set<string>()

  function getImportedChunks(chunk: ManifestChunk): ManifestChunk[] {
    const chunks: ManifestChunk[] = []
    for (const file of chunk.imports ?? []) {
      const importee = manifest[file]
      if (seen.has(file)) continue
      seen.add(file)
      chunks.push(...getImportedChunks(importee))
      chunks.push(importee)
    }
    return chunks
  }

  return getImportedChunks(manifest[name])
}
```

---

## Key Patterns for Skill Creation

### Pattern: Complete Plugin Template

```ts
import type { Plugin, ResolvedConfig } from 'vite'

export default function myPlugin(options = {}): Plugin {
  let config: ResolvedConfig

  return {
    name: 'vite-plugin-my-plugin',
    enforce: 'pre', // optional: 'pre' | 'post'
    apply: 'serve', // optional: 'serve' | 'build' | function

    config(userConfig, { command, mode }) {
      // Modify config before resolution
      return { /* partial config */ }
    },

    configResolved(resolvedConfig) {
      config = resolvedConfig
    },

    configureServer(server) {
      // Add middleware, WebSocket handlers
    },

    transformIndexHtml(html, ctx) {
      // Transform HTML, inject tags
      return html
    },

    resolveId(id) {
      // Resolve virtual modules
    },

    load(id) {
      // Load virtual modules
    },

    transform(code, id) {
      // Transform source code
    },

    handleHotUpdate(ctx) {
      // Custom HMR handling
    },
  }
}
```

### Pattern: SSR-Aware Plugin

```ts
export default function ssrAwarePlugin(): Plugin {
  return {
    name: 'ssr-aware',
    transform(code, id, options) {
      const isSSR = options?.ssr
      if (isSSR) {
        // Server-specific transform
      } else {
        // Client-specific transform
      }
    },
  }
}
```

### Pattern: Full HMR Module

```ts
let state = initState()

function render() {
  // Use state to render
}

render()

if (import.meta.hot) {
  import.meta.hot.accept((newModule) => {
    if (newModule) {
      // Update with new module
    }
  })

  import.meta.hot.dispose((data) => {
    data.state = state // Preserve state
  })

  if (import.meta.hot.data.state) {
    state = import.meta.hot.data.state // Restore state
  }
}
```

---

## CLI Commands

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

Additional flags: `--port`, `--open`, `--mode`, `--force` (re-bundle deps).

## Framework Templates (via create-vite)

vanilla, vanilla-ts, vue, vue-ts, react, react-ts, react-swc, react-swc-ts, preact, preact-ts, lit, lit-ts, svelte, svelte-ts, solid, solid-ts, qwik, qwik-ts

## Official Plugins

- `@vitejs/plugin-vue` -- Vue SFC support
- `@vitejs/plugin-vue-jsx` -- Vue JSX support
- `@vitejs/plugin-react` -- React support (Babel)
- `@vitejs/plugin-react-swc` -- React support (SWC)
- `@vitejs/plugin-rsc` -- React Server Components
- `@vitejs/plugin-legacy` -- Legacy browser support

---

## Configuration Quick Reference

### Asset-Related Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `assetsInlineLimit` | `number` | `4096` (4kb) | Assets smaller than this are inlined as base64 |
| `assetsInclude` | `string \| RegExp \| (string \| RegExp)[]` | -- | Additional file types to treat as assets |
| `publicDir` | `string \| false` | `'public'` | Directory for static assets served at root |
| `base` | `string` | `'/'` | Public base path |
| `build.assetsDir` | `string` | `'assets'` | Directory for generated assets (relative to outDir) |

### Build Options Quick Reference

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `build.target` | `string \| string[]` | `'modules'` | Browser compatibility target |
| `build.outDir` | `string` | `'dist'` | Output directory |
| `build.lib` | `object \| false` | -- | Library mode configuration |
| `build.sourcemap` | `boolean \| 'inline' \| 'hidden'` | `false` | Source map generation |
| `build.cssCodeSplit` | `boolean` | `true` | Enable CSS code splitting |
| `build.cssTarget` | `string \| string[]` | Same as `build.target` | CSS browser target |
| `build.cssMinify` | `boolean \| 'lightningcss'` | Same as `build.minify` | CSS minification |
| `build.watch` | `WatcherOptions \| null` | `null` | Watch mode config |
| `build.license` | `boolean \| object` | `false` | License file generation (Vite 6) |

### CSS Options Quick Reference

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `css.modules` | `CSSModulesOptions` | -- | CSS Modules configuration |
| `css.modules.localsConvention` | `string` | -- | Class name convention (e.g., `'camelCaseOnly'`) |
| `css.transformer` | `'postcss' \| 'lightningcss'` | `'postcss'` | CSS transformer engine |
| `css.lightningcss` | `LightningCSSOptions` | -- | Lightning CSS configuration |

### Dependency Pre-Bundling Options Quick Reference

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `optimizeDeps.include` | `string[]` | -- | Force pre-bundle these dependencies |
| `optimizeDeps.exclude` | `string[]` | -- | Exclude from pre-bundling |
| `optimizeDeps.force` | `boolean` | `false` | Force re-bundle ignoring cache |
| `optimizeDeps.rolldownOptions` | `object` | -- | Rolldown options for pre-bundling (Vite 6) |
| `optimizeDeps.esbuildOptions` | `object` | -- | esbuild options for pre-bundling (Vite 5) |
