# Lessons Learned

Observations and findings captured during skill package development.

---

## L-001: Vite 5 to 6 Migration is Environment API-Centric

- **Date**: 2026-03-19
- **Context**: Determining scope of version coverage for the skill package.
- **Finding**: The primary breaking change between Vite 5 and Vite 6 is the introduction of the Environment API, which allows per-environment (client, SSR, custom) configuration. Skills that cover SSR, dev server internals, or plugin hooks MUST account for this API change. Most other Vite 5 patterns remain valid in Vite 6.

---

## L-002: Vite is NOT Rollup (Plugin API Distinction)

- **Date**: 2026-03-19
- **Context**: Claude frequently confuses Vite plugin hooks with Rollup plugin hooks.
- **Finding**: While Vite's plugin API is Rollup-compatible, Vite adds its own hooks (`configResolved`, `configureServer`, `transformIndexHtml`, `handleHotUpdate`) that have no Rollup equivalent. Skills MUST clearly distinguish Vite-specific hooks from Rollup-compatible hooks to prevent Claude from hallucinating incorrect hook signatures.

---

## L-003: Single Technology Simplifies Package Structure

- **Date**: 2026-03-19
- **Context**: Unlike multi-technology packages (Blender: 4 technologies), Vite is one tool.
- **Finding**: No need for per-technology separation. `skills/source/vite-{category}/` is sufficient. Simpler dependency graph. Proven pattern from the Tauri 2 package.
