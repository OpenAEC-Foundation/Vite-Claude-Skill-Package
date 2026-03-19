# Vite Research: Core Concepts, Configuration, Dev Server

> **Sources**: All data fetched via WebFetch from official Vite documentation (vite.dev) on 2026-03-19.
> **Note**: The official domain has migrated from vitejs.dev to vite.dev (301 redirect).
> **Current stable**: Vite 8.x (Rolldown-based). Vite 6/7 docs available at v6.vite.dev / v7.vite.dev.

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

## 3. Dev Server

### HMR Mechanism

Vite provides an HMR API over native ESM. First-party integrations:
- Vue Single File Components
- React Fast Refresh
- Preact via @prefresh/vite

HMR delivers instant, precise updates without page reload or state loss.

### Middleware Mode

Create Vite as middleware for custom HTTP servers:

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

  app.use('*', async (req, res) => {
    // Custom server logic, SSR rendering, etc.
  })

  app.listen(5173)
}
createServer()
```

### Proxy Configuration

```javascript
export default defineConfig({
  server: {
    proxy: {
      // String shorthand: /foo -> http://localhost:4567/foo
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

### HTTPS Setup

```javascript
export default defineConfig({
  server: {
    https: {
      key: fs.readFileSync('path/to/key.pem'),
      cert: fs.readFileSync('path/to/cert.pem'),
    },
  },
})
```

Enables TLS + HTTP/2.

### File System Serving (server.fs)

- `server.fs.strict: true` (default) -- restricts serving files outside workspace root
- `server.fs.allow: string[]` -- whitelist directories for `/@fs/` serving
- `server.fs.deny: ['.env', '.env.*', '*.{crt,pem}', '**/.git/**']` -- blocklist sensitive files

### Server Warmup

Pre-transform files for faster initial load:

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

---

## 4. Environment Variables

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

## 5. Backend Integration

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

## 6. Key Features Reference

### TypeScript Support

- Transpile-only (no type checking) -- 20-30x faster than `tsc`
- Run `tsc --noEmit` separately for type checking
- Critical tsconfig options: `isolatedModules: true` (required), `useDefineForClassFields: true` for ES2022+
- Client types: add `"vite/client"` to `compilerOptions.types` or use `/// <reference types="vite/client" />`

### CSS Support

- **CSS Modules**: Files ending in `.module.css`
- **Preprocessors**: Sass (`sass-embedded`/`sass`), Less (`less`), Stylus (`stylus`) -- install only, no Vite plugins needed
- **PostCSS**: Auto-detected from config files
- **Lightning CSS**: Experimental alternative via `css.transformer: 'lightningcss'`
- **`?inline` query**: Prevents CSS injection into page

### Static Assets

```javascript
import imgUrl from './img.png'          // URL
import raw from './shader.glsl?raw'     // String content
import assetUrl from './asset.js?url'   // URL only
import Worker from './worker.js?worker' // Web Worker
```

### Glob Import

```javascript
const modules = import.meta.glob('./dir/*.js')
// Lazy: { './dir/foo.js': () => import('./dir/foo.js'), ... }

const eager = import.meta.glob('./dir/*.js', { eager: true })
// Eager: { './dir/foo.js': Module, ... }

const named = import.meta.glob('./dir/*.js', { import: 'setup', eager: true })
// Named + eager: tree-shakeable
```

### JSON Import

```javascript
import json from './example.json'          // Full object
import { field } from './example.json'     // Named export
```

### WebAssembly

```javascript
import init from './example.wasm?init'
init().then((instance) => {
  instance.exports.test()
})
```

### Web Workers

```typescript
// Constructor syntax (recommended)
const worker = new Worker(new URL('./worker.js', import.meta.url))
const moduleWorker = new Worker(new URL('./worker.js', import.meta.url), {
  type: 'module',
})

// Query suffix syntax
import MyWorker from './worker?worker'
const worker = new MyWorker()
```

---

## 7. Migration Guides

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

## 8. CLI Commands

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

## 9. Framework Templates (via create-vite)

vanilla, vanilla-ts, vue, vue-ts, react, react-ts, react-swc, react-swc-ts, preact, preact-ts, lit, lit-ts, svelte, svelte-ts, solid, solid-ts, qwik, qwik-ts

## 10. Official Plugins

- `@vitejs/plugin-vue` -- Vue SFC support
- `@vitejs/plugin-vue-jsx` -- Vue JSX support
- `@vitejs/plugin-react` -- React support (Babel)
- `@vitejs/plugin-react-swc` -- React support (SWC)
- `@vitejs/plugin-rsc` -- React Server Components
- `@vitejs/plugin-legacy` -- Legacy browser support
