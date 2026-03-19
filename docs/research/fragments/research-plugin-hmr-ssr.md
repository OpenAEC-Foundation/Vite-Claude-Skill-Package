# Vite Research: Plugin API, HMR API, SSR, JavaScript API

> **Source**: Official Vite documentation at https://vite.dev (redirected from vitejs.dev)
> **Fetched**: 2026-03-19
> **Covers**: Vite 5.x, 6.x, references to 8.x (latest docs)

---

## 1. Plugin API

**Source**: https://vite.dev/guide/api-plugin

### 1.1 Plugin Structure and Conventions

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

### 1.2 Plugin Ordering (enforce property)

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

### 1.3 Conditional Application (apply property)

```js
// Apply only during build or serve
apply: 'build', // or 'serve'

// Or use function for precise control
apply(config, { command }) {
  return command === 'build' && !config.build.ssr
}
```

### 1.4 Plugin Arrays and Conditional Plugins

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

### 1.5 Rollup-Compatible (Universal) Hooks

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

### 1.6 ALL Vite-Specific Hooks

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

**Post-middleware injection** — return a function to add middleware after internal ones:

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

### 1.7 Virtual Modules Pattern

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

### 1.8 Hook Filters (Vite 6.3.0+)

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

### 1.9 Plugin Context Meta

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

### 1.10 Output Bundle Metadata

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

### 1.11 Client-Server Communication

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

### 1.12 Path Normalization and Filtering Utilities

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

### 1.13 Rollup Plugin Compatibility Notes

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

## 2. HMR API (Client)

**Source**: https://vite.dev/guide/api-hmr

### 2.1 Complete TypeScript Interface

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

### 2.2 import.meta.hot Guard Pattern

All HMR code MUST be wrapped in a conditional guard for tree-shaking in production:

```js
if (import.meta.hot) {
  // HMR code here
}
```

### 2.3 hot.accept() — Self-Accepting Modules

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

### 2.4 hot.accept(deps, cb) — Accepting Dependencies

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

### 2.5 hot.dispose(cb) — Cleanup Side Effects

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

### 2.6 hot.prune(cb) — Module Pruning

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

### 2.7 hot.invalidate(message?: string) — Force Propagation

Forces HMR to propagate upward to importers when the module cannot handle its own update:

```js
import.meta.hot.accept((module) => {
  if (cannotHandleUpdate(module)) {
    import.meta.hot.invalidate()
  }
})
```

**Pattern**: ALWAYS call `accept()` before `invalidate()` for proper intent communication.

### 2.8 hot.data — Persistent Data Across Updates

The data object persists across module updates:

```js
// Correct: mutate properties
import.meta.hot.data.someValue = 'hello'

// Incorrect: reassignment NOT supported
import.meta.hot.data = { someValue: 'hello' }
```

### 2.9 hot.on() / hot.off() — Listening to Events

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

### 2.10 hot.send(event, data) — Custom Events to Server

Send custom events to Vite's dev server. Data is buffered if WebSocket is not yet connected.

### 2.11 HMR Boundary Concept

An **HMR boundary** is a module that "accepts" hot updates via `import.meta.hot.accept()`. It marks where updates stop propagating up the dependency chain.

Key detail: Vite's HMR does NOT actually swap the originally imported module. Re-exports must use `let` (not `const`) and the boundary module handles update propagation.

### 2.12 When Full Reload Happens vs HMR Update

**Full page reloads occur when:**
- No HMR boundary exists for a changed module (update propagates to root)
- The boundary's accept handler fails
- `hot.invalidate()` is called and propagates all the way up

**HMR update occurs when:**
- A changed module has an HMR boundary (self-accepting or accepted by a parent)
- The accept callback successfully handles the update

### 2.13 TypeScript Setup

Add to `tsconfig.json` for HMR type definitions:

```json
{
  "compilerOptions": {
    "types": ["vite/client"]
  }
}
```

---

## 3. SSR (Server-Side Rendering)

**Source**: https://vite.dev/guide/ssr

### 3.1 SSR Dev Workflow

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

### 3.2 ssrLoadModule()

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

### 3.3 ssrFixStacktrace()

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

### 3.4 SSR Externals Configuration

| Option | Purpose |
|--------|---------|
| `ssr.noExternal` | Dependencies requiring Vite's transformation pipeline (array or `true` to bundle everything) |
| `ssr.external` | Dependencies to externalize despite being linked |
| `ssr.target` | SSR environment: `'node'` (default) or `'webworker'` |

- `ssr.noExternal: true` bundles everything into single JS file (useful for webworker target)
- `ssr.target` affects package entry resolution

### 3.5 SSR Build Config and Commands

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

### 3.6 SSR Conditional Logic

```javascript
if (import.meta.env.SSR) {
  // server-only code
}
```

Statically replaced during build, enabling tree-shaking of unused branches.

### 3.7 CJS/ESM Handling in SSR

- `ssrLoadModule` automatically transforms ESM to Node.js-compatible code — no bundling required
- For production, directly import the pre-built server bundle: `import('./dist/server/entry-server.js')`

### 3.8 Production Build Steps

1. Read `dist/client/index.html` (not root) as template
2. Replace `ssrLoadModule()` with `import('./dist/server/entry-server.js')`
3. Move Vite dev server creation behind `process.env.NODE_ENV` checks
4. Add static file serving from `dist/client`

### 3.9 SSR-Specific Vite Config Options

| Option | Purpose |
|--------|---------|
| `ssr.resolve.conditions` | Customize package entry resolution for SSR builds (overrides `resolve.conditions`) |
| `ssr.resolve.externalConditions` | Additional conditions for externalized dependencies |
| `ssr.noExternal` | Array of dependencies to transform through Vite |
| `ssr.external` | Array of dependencies to externalize despite being linked |
| `ssr.target` | `'node'` (default) or `'webworker'` |

### 3.10 Plugin SSR-Specific Logic

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

## 4. JavaScript API (Programmatic)

**Source**: https://vite.dev/guide/api-javascript

### 4.1 createServer()

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

### 4.2 ViteDevServer Interface (Complete)

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

### 4.3 build()

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

### 4.4 preview()

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

### 4.5 resolveConfig()

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

### 4.6 mergeConfig()

```typescript
function mergeConfig(
  defaults: Record<string, any>,
  overrides: Record<string, any>,
  isRoot = true,
): Record<string, any>
```

Deeply merges two Vite configs. Set `isRoot` to `false` when merging nested options like `build`.

### 4.7 Important TypeScript Types

**InlineConfig:** Extends `UserConfig` with additional properties:
- `configFile`: Specify config file path; set to `false` to disable auto-resolving

**ResolvedConfig:** Has all `UserConfig` properties (resolved and non-undefined) plus utilities:
- `config.assetsInclude`: Function checking if an ID is an asset
- `config.logger`: Internal logger object

### 4.8 Utility Functions

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

### 4.9 Transform Functions

**transformWithOxc() (Current):**

```typescript
async function transformWithOxc(
  code: string,
  filename: string,
  options?: OxcTransformOptions,
  inMap?: object,
): Promise<Omit<OxcTransformResult, 'errors'> & { warnings: string[] }>
```

**transformWithEsbuild() (Deprecated — use transformWithOxc):**

```typescript
async function transformWithEsbuild(
  code: string,
  filename: string,
  options?: EsbuildTransformOptions,
  inMap?: object,
): Promise<ESBuildTransformResult>
```

### 4.10 loadConfigFromFile()

```typescript
async function loadConfigFromFile(
  configEnv: ConfigEnv,
  configFile?: string,
  configRoot: string = process.cwd(),
  logLevel?: LogLevel,
  customLogger?: Logger,
): Promise<{ path: string; config: UserConfig; dependencies: string[] } | null>
```

### 4.11 preprocessCSS() (Experimental)

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

### 4.12 Version Constants

| Constant | Description |
|----------|-------------|
| `version` | Current Vite version string (e.g., "8.0.0") |
| `rolldownVersion` | Rolldown version used (e.g., "1.0.0") |
| `esbuildVersion` | For backward compatibility only |
| `rollupVersion` | For backward compatibility only |

---

## 5. Version Differences Summary

| Area | v5/v6 | v8+ (latest docs) |
|------|-------|--------------------|
| Bundler | Rollup-based | Rolldown-based |
| Transform | esbuild (`transformWithEsbuild`) | OXC (`transformWithOxc`) |
| Hook filters | Not available | `filter` property on hooks (v6.3.0+) |
| Plugin utils | `@rollup/pluginutils` | `@rolldown/pluginutils` (with `exactRegex`, `prefixRegex`) |
| Plugin context meta | `this.meta.viteVersion` | Also `this.meta.rolldownVersion` |
| SSR plugin params | `options.ssr` on options object | Same (changed from positional in v2.7) |
| Domain | vitejs.dev | vite.dev (301 redirect) |

**Note**: The current official docs (vite.dev) describe Vite 8.x which uses Rolldown. For Vite 5.x/6.x projects, the Rollup-based APIs apply. The hook signatures and plugin structure remain compatible across versions — the main differences are the underlying bundler engine and transform tool.

---

## 6. Key Patterns for Skill Creation

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
