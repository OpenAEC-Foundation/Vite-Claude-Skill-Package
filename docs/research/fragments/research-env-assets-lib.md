# Vite Research: Environment API, Assets, Library Mode, Dependency Pre-bundling

> **Source**: Official Vite documentation at https://vite.dev (fetched 2026-03-19)
> **Covers**: Vite 5.x and 6.x

---

## 1. Environment API (Vite 6)

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
- `define` — Define global constant replacements
- `resolve` — Module resolution options
- `optimizeDeps` — Dependency pre-bundling options
- `consumer` — `'client'` | `'server'`
- `dev` — Dev-specific nested options
- `build` — Build-specific nested options

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

## 2. Static Asset Handling

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

#### `?url` — Explicit URL import
For assets not in the auto-detected list:
```js
import workletURL from 'extra-scalloped-border/worklet.js?url'
CSS.paintWorklet.addModule(workletURL)
```

#### `?raw` — Import as string
```js
import shaderString from './shader.glsl?raw'
```

#### `?inline` / `?no-inline` — Control inlining behavior
```js
import imgUrl1 from './img.svg?no-inline'
import imgUrl2 from './img.png?inline'
```

#### `?worker` / `?sharedworker` — Import as web worker
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
- Does NOT work with SSR — `import.meta.url` has different semantics in browsers vs Node.js

### Public Directory

Place assets in `<root>/public` (configurable via `publicDir` option) for files that:
- Are never referenced in source code (e.g., `robots.txt`)
- Must retain exact file names without hashing
- Don't require prior imports for URL access

Assets served at root `/` during development, copied to dist root unchanged.

Reference as absolute paths: `public/icon.png` → `/icon.png` in code.

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
- Arguments MUST be string literals — no variables or expressions

---

## 3. Library Mode

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

## 4. Dependency Pre-Bundling

### Why Pre-Bundling Exists

When you run `vite` for the first time, Vite pre-bundles your project dependencies before loading your site locally. Two main reasons:

#### 1. CommonJS/UMD to ESM Conversion
During development, Vite serves all code as native ESM. Dependencies published as CommonJS or UMD must be converted.

"Vite performs smart import analysis so that named imports to CommonJS modules will work as expected even if the exports are dynamically assigned."

```javascript
// works as expected even though React is CJS
import React, { useState } from 'react'
```

#### 2. Performance — Reducing Module Count
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

## 5. Additional Features

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
Vite only **transpiles** `.ts` files — no type checking. Type checking assumed handled by IDE or separate process.

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

**`isolatedModules` — MUST be `true`:**
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
Partially supported — full support requires TypeScript compiler type inference.

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

### Browser Compatibility (Build Targets)

**Default target (Baseline Widely Available):**
- Chrome >= 111
- Edge >= 111
- Firefox >= 114
- Safari >= 16.4

**Minimum ESM target (es2015):**
- Chrome >= 64
- Firefox >= 67
- Safari >= 11.1
- Edge >= 79

Configure via `build.target`. Vite only handles syntax transforms, NOT polyfills. Use `@vitejs/plugin-legacy` for older browsers.

### Public Base Path

Set `base` config for nested deployments. All asset paths rewritten accordingly. Access at runtime via `import.meta.env.BASE_URL`. Supports relative paths using `"./"` or `""`.

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

### Load Error Handling

Listen for `vite:preloadError` events for failed dynamic imports:
```js
window.addEventListener('vite:preloadError', (event) => {
  // handle error — e.g., reload page
})
```
Occurs when user assets are outdated after deployments.

### HTML as Entry Points

HTML files serve as project entry points. Files in project root are directly accessible:
- `<root>/index.html` → `http://localhost:5173/`
- `<root>/about.html` → `http://localhost:5173/about.html`

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

### Content Security Policy (CSP)

**Nonce-based:**
Set `html.cspNonce` to add nonce attribute to all `<script>`, `<style>`, and `<link>` tags. Vite injects `<meta property="csp-nonce" nonce="PLACEHOLDER" />`. Replace placeholder per request.

**Data URIs:**
Allow `data:` for relevant directives, or disable inlining via `build.assetsInlineLimit: 0`. NEVER allow `data:` for `script-src`.

### License Generation (Vite 6)

```js
export default {
  build: {
    license: true, // or { fileName: 'license.md' }
  },
}
```

Generates `.vite/license.md` with all dependency licenses.

### HMR (Hot Module Replacement)

Vite provides HMR API over native ESM. First-party integrations:
- Vue SFC: `@vitejs/plugin-vue`
- React Fast Refresh: `@vitejs/plugin-react`
- Preact: `@prefresh/vite`

Pre-configured in `create-vite` templates.

---

## Key Version Differences: Vite 5 vs Vite 6

| Feature | Vite 5 | Vite 6 |
|---------|--------|--------|
| Environment API | Implicit client + ssr only | Explicit multi-environment support |
| Pre-bundling tool | esbuild | Rolldown |
| `optimizeDeps` bundler options | `optimizeDeps.esbuildOptions` | `optimizeDeps.rolldownOptions` |
| Build internals | Rollup-based | Rolldown-based (with `rolldownOptions`) |
| License generation | Not built-in | `build.license: true` |
| SSR handling | `server.ssrLoadModule()` | ModuleRunner API (planned migration) |
| Plugin hooks | Implicit environment handling | Moving toward explicit per-environment hooks |

---

## Configuration Quick Reference

### Asset-Related Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `assetsInlineLimit` | `number` | `4096` (4kb) | Assets smaller than this are inlined as base64 |
| `assetsInclude` | `string \| RegExp \| (string \| RegExp)[]` | — | Additional file types to treat as assets |
| `publicDir` | `string \| false` | `'public'` | Directory for static assets served at root |
| `base` | `string` | `'/'` | Public base path |
| `build.assetsDir` | `string` | `'assets'` | Directory for generated assets (relative to outDir) |

### Build Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `build.target` | `string \| string[]` | `'modules'` | Browser compatibility target |
| `build.outDir` | `string` | `'dist'` | Output directory |
| `build.lib` | `object \| false` | — | Library mode configuration |
| `build.sourcemap` | `boolean \| 'inline' \| 'hidden'` | `false` | Source map generation |
| `build.cssCodeSplit` | `boolean` | `true` | Enable CSS code splitting |
| `build.cssTarget` | `string \| string[]` | Same as `build.target` | CSS browser target |
| `build.cssMinify` | `boolean \| 'lightningcss'` | Same as `build.minify` | CSS minification |
| `build.watch` | `WatcherOptions \| null` | `null` | Watch mode config |
| `build.license` | `boolean \| object` | `false` | License file generation (Vite 6) |

### CSS Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `css.modules` | `CSSModulesOptions` | — | CSS Modules configuration |
| `css.modules.localsConvention` | `string` | — | Class name convention (e.g., `'camelCaseOnly'`) |
| `css.transformer` | `'postcss' \| 'lightningcss'` | `'postcss'` | CSS transformer engine |
| `css.lightningcss` | `LightningCSSOptions` | — | Lightning CSS configuration |

### Dependency Pre-Bundling Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `optimizeDeps.include` | `string[]` | — | Force pre-bundle these dependencies |
| `optimizeDeps.exclude` | `string[]` | — | Exclude from pre-bundling |
| `optimizeDeps.force` | `boolean` | `false` | Force re-bundle ignoring cache |
| `optimizeDeps.rolldownOptions` | `object` | — | Rolldown options for pre-bundling (Vite 6) |
| `optimizeDeps.esbuildOptions` | `object` | — | esbuild options for pre-bundling (Vite 5) |
