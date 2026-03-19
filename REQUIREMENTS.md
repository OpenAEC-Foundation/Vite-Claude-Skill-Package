# REQUIREMENTS

## What This Skill Package Must Achieve

### Primary Goal
Enable Claude to write correct, version-aware Vite configuration and plugin code for both Vite 5 and Vite 6 — without hallucinating APIs or using deprecated patterns.

### What Claude Should Do After Loading Skills
1. Recognize Vite context from user requests (build tool, dev server, HMR, bundling)
2. Select the correct skill(s) automatically based on the request
3. Write correct TypeScript/JavaScript configuration and plugin code
4. Avoid known anti-patterns and common AI mistakes (e.g., confusing Webpack/Rollup APIs with Vite)
5. Follow best practices documented in the skill references

### Quality Guarantees
| Guarantee | Description |
|-----------|-------------|
| Version-correct | Code MUST specify Vite 5.x or 6.x target and note differences |
| API-accurate | All config options and plugin hooks verified against official docs |
| Framework-aware | Skills cover Vite with React, Vue, Svelte, and vanilla setups |
| Anti-pattern-free | Known mistakes are explicitly documented and avoided |
| Deterministic | Skills use ALWAYS/NEVER language, not suggestions |
| Self-contained | Each skill works independently without requiring other skills |

---

## Per-Area Requirements

### 1. Configuration
| Requirement | Detail |
|-------------|--------|
| Core config | `vite.config.ts` structure, `defineConfig()`, conditional config |
| Server options | `server.proxy`, `server.https`, `server.cors`, `server.hmr` |
| Build options | `build.target`, `build.outDir`, `build.rollupOptions`, chunk splitting |
| Resolve options | `resolve.alias`, `resolve.extensions`, `resolve.conditions` |
| CSS options | `css.modules`, `css.preprocessorOptions`, `css.postcss` |
| Critical | Conditional config with `mode` and environment variables |

### 2. Plugin API
| Requirement | Detail |
|-------------|--------|
| Hook lifecycle | `configResolved`, `transformIndexHtml`, `transform`, `load`, `resolveId` |
| Plugin ordering | `enforce: 'pre'` / `enforce: 'post'`, plugin execution order |
| Virtual modules | `resolveId` + `load` pattern for virtual module creation |
| HMR in plugins | `handleHotUpdate`, custom HMR via `import.meta.hot` |
| Critical | Correct hook signatures and return types for Vite 5 AND 6 |

### 3. HMR (Hot Module Replacement)
| Requirement | Detail |
|-------------|--------|
| Client API | `import.meta.hot.accept()`, `import.meta.hot.dispose()`, self-accepting modules |
| Boundary patterns | HMR boundaries, propagation, full-reload triggers |
| Custom events | `import.meta.hot.send()`, `import.meta.hot.on()` |
| Critical | Correct HMR guard pattern (`if (import.meta.hot)`) |

### 4. SSR (Server-Side Rendering)
| Requirement | Detail |
|-------------|--------|
| SSR config | `ssr.external`, `ssr.noExternal`, `ssr.target` |
| Module loading | `ssrLoadModule()`, `ssrFixStacktrace()` |
| Externalization | Dependency externalization rules, CJS/ESM handling |
| Critical | SSR-specific build configuration and module resolution |

### 5. Library Mode
| Requirement | Detail |
|-------------|--------|
| Build config | `build.lib` options: `entry`, `formats`, `fileName`, `name` |
| Externals | `build.rollupOptions.external` for peer dependencies |
| Type generation | DTS plugin integration, `.d.ts` output |
| Critical | Correct UMD/ESM/CJS output configuration |

### 6. Environment API (Vite 6)
| Requirement | Detail |
|-------------|--------|
| Environments | `ssr`, `client`, custom environments |
| Dev config | Per-environment dev configuration |
| Build config | Per-environment build configuration |
| Critical | Vite 6 breaking changes from Vite 5 environment handling |

### 7. Asset Handling
| Requirement | Detail |
|-------------|--------|
| Static assets | Import patterns, `?url`, `?raw`, `?inline`, `?worker` suffixes |
| Public directory | `public/` directory behavior, `publicDir` config |
| JSON handling | `json.stringify`, named imports from JSON |
| Critical | Correct asset URL resolution in dev vs production |

### 8. Dependency Optimization
| Requirement | Detail |
|-------------|--------|
| Pre-bundling | `optimizeDeps.include`, `optimizeDeps.exclude`, `optimizeDeps.force` |
| CJS to ESM | Automatic CJS-to-ESM conversion, edge cases |
| Monorepo deps | Linked dependencies, workspace packages |
| Critical | When and why to force re-optimization |

---

## Critical Requirements (apply to ALL skills)

- All config code MUST work with Vite 6.x/7.x/8.x and note differences per version
- All plugin code MUST use correct hook signatures for the target version
- Config examples MUST use `defineConfig()` for type safety
- Environment variable examples MUST use `import.meta.env` (not `process.env` in client code)
- Code examples MUST be verified against official documentation

---

## Structural Requirements

### Skill Format
- SKILL.md < 500 lines (heavy content in references/)
- YAML frontmatter with name and description (including trigger words)
- English-only content
- Deterministic language (ALWAYS/NEVER, imperative)

### Skill Categories
| Category | Purpose | Must Include |
|----------|---------|--------------|
| syntax/ | How to write it | Config options, API signatures, code patterns |
| impl/ | How to build it | Decision trees, workflows, step-by-step guides |
| errors/ | How to handle failures | Error patterns, diagnostics, recovery |
| core/ | Cross-cutting | Architecture overview, dev server internals, build pipeline |
| agents/ | Orchestration | Validation checklists, auto-detection |

---

## Research Requirements (before creating any skill)

1. Official documentation MUST be consulted and referenced
2. Source code MUST be checked for accuracy when docs are ambiguous
3. Anti-patterns MUST be identified from real issues (GitHub issues, Discord, forums)
4. Code examples MUST be verified (not hallucinated)
5. Version accuracy MUST be confirmed via WebFetch (D-012)

---

## Non-Requirements (explicitly out of scope)

- No Vite 5.x or older coverage except migration (Vite 6, 7, 8 — D-008)
- No deep-dive into specific frontend frameworks (React/Vue/Svelte internals)
- No Webpack/Rollup migration guides
- No deployment platform tutorials (Vercel, Netlify specifics)
- No testing framework integration deep-dives (Vitest has its own ecosystem)
