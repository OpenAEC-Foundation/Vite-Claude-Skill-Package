# Approved Sources Registry

All skills in this package MUST be verified against these approved sources. No other sources are authoritative.

## Official Documentation

| Source | URL | Purpose |
|--------|-----|---------|
| Vite Documentation | https://vite.dev/ | Primary reference for all Vite content |
| Vite Guide | https://vite.dev/guide/ | Getting started, features, concepts |
| Vite Config Reference | https://vite.dev/config/ | Complete configuration options reference |
| Vite Plugin API | https://vite.dev/guide/api-plugin | Plugin hook signatures and lifecycle |
| Vite HMR API | https://vite.dev/guide/api-hmr | Hot Module Replacement client API |
| Vite JavaScript API | https://vite.dev/guide/api-javascript | Programmatic `createServer()` and `build()` |
| Vite Environment API | https://vite.dev/guide/api-environment | Vite 6 Environment API (multi-environment) |
| Vite SSR Guide | https://vite.dev/guide/ssr | Server-Side Rendering guide |
| Vite Library Mode | https://vite.dev/guide/build#library-mode | Library build configuration |
| Vite Static Asset Handling | https://vite.dev/guide/assets | Asset import patterns and public directory |
| Vite Env Variables | https://vite.dev/guide/env-and-mode | `import.meta.env`, `.env` files, modes |
| Vite Dependency Pre-Bundling | https://vite.dev/guide/dep-pre-bundling | esbuild pre-bundling, optimization |
| Vite Backend Integration | https://vite.dev/guide/backend-integration | Integration with backend frameworks |

## Source Code

| Source | URL | Purpose |
|--------|-----|---------|
| Vite Core Repository | https://github.com/vitejs/vite | Main monorepo (core, plugin-legacy, plugin-react, create-vite) |
| Vite Source (packages/vite) | https://github.com/vitejs/vite/tree/main/packages/vite | Core source code |
| Vite Plugins | https://github.com/vitejs/vite/tree/main/packages | Official plugins (React, Legacy) |
| Awesome Vite | https://github.com/vitejs/awesome-vite | Community plugins and resources |

## Package Docs

| Source | URL | Purpose |
|--------|-----|---------|
| vite (npm) | https://www.npmjs.com/package/vite | npm package info |
| @vitejs/plugin-react | https://www.npmjs.com/package/@vitejs/plugin-react | React plugin |
| @vitejs/plugin-vue | https://www.npmjs.com/package/@vitejs/plugin-vue | Vue plugin |
| @vitejs/plugin-legacy | https://www.npmjs.com/package/@vitejs/plugin-legacy | Legacy browser support |
| vite-plugin-svelte | https://www.npmjs.com/package/@sveltejs/vite-plugin-svelte | Svelte plugin |

## Migration & Release Notes

| Source | URL | Purpose |
|--------|-----|---------|
| Vite 5 to 6 Migration | https://vite.dev/guide/migration | Migration guide between major versions |
| Vite Blog | https://vite.dev/blog/ | Release announcements |
| GitHub Releases | https://github.com/vitejs/vite/releases | All version changelogs |

## Community (for Anti-Pattern Research)

| Source | URL | Purpose |
|--------|-----|---------|
| GitHub Issues | https://github.com/vitejs/vite/issues | Real-world error patterns |
| GitHub Discussions | https://github.com/vitejs/vite/discussions | Q&A, patterns |
| Vite Discord | https://chat.vitejs.dev/ | Community help |

## Claude / Anthropic (Skill Development Platform)

| Source | URL | Purpose |
|--------|-----|---------|
| Agent Skills Standard | https://agentskills.io | Open standard |
| Agent Skills Spec | https://github.com/agentskills/agentskills | Specification |
| Agent Skills Best Practices | https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices | Authoring guide |

## OpenAEC Foundation

| Source | URL | Purpose |
|--------|-----|---------|
| ERPNext Skill Package | https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package | Methodology template |
| Tauri 2 Skill Package | https://github.com/OpenAEC-Foundation/Tauri-2-Claude-Skill-Package | Proven 27-skill package |
| Blender Skill Package | (reference only) | Proven 73-skill package |

## Source Verification Rules

1. **Primary sources ONLY** — Official docs, official repos, official npm docs.
2. **NEVER trust random blog posts** — Even popular ones may be outdated or wrong for v5/v6.
3. **Verify code against official docs** — Every code snippet in a skill MUST match current API.
4. **Note when source was last verified** — Track in the table below.
5. **Cross-reference if docs are sparse** — When official docs lack detail, verify against source code in the monorepo.

## Last Verified

| Technology | Date | Action | Notes |
|------------|------|--------|-------|
| Vite | 2026-03-19 | Deep research complete | All primary docs verified via WebFetch. Domain migrated vitejs.dev→vite.dev. Current stable: v8 (Rolldown+Oxc). v6 docs at v6.vite.dev. |
