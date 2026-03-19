# Vite Claude Skill Package

<p align="center">
  <strong style="font-size: 1.5em; color: #646CFF;">22 Deterministic Claude AI Skills for Vite 6, 7 & 8</strong>
</p>

![Claude Code Ready](https://img.shields.io/badge/Claude_Code-Ready-blue?style=flat-square&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZD0iTTEyIDJDNi40OCAyIDIgNi40OCAyIDEyczQuNDggMTAgMTAgMTAgMTAtNC40OCAxMC0xMFMxNy41MiAyIDEyIDJ6IiBmaWxsPSIjZmZmIi8+PC9zdmc+)
![Vite 6-8](https://img.shields.io/badge/Vite-6_%7C_7_%7C_8-646CFF?style=flat-square&logo=vite&logoColor=white)
![TypeScript + JavaScript](https://img.shields.io/badge/TypeScript_+_JavaScript-Full_Coverage-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Skills](https://img.shields.io/badge/Skills-22-brightgreen?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

**Deterministic Claude AI skills for Vite — next-generation frontend build tool with instant HMR, optimized builds, and plugin ecosystem.**

Built on the [Agent Skills](https://agentskills.org) open standard.

---

## Why This Exists

Without skills, Claude generates incorrect or outdated Vite patterns:

```typescript
// Wrong — Webpack-style config that won't work in Vite
module.exports = {
  entry: './src/index.js',
  output: { path: path.resolve(__dirname, 'dist') },
  module: { rules: [{ test: /\.tsx?$/, use: 'ts-loader' }] }
};
```

With this skill package, Claude produces correct, version-aware Vite configuration:

```typescript
// Correct — Vite 8 config with defineConfig and Rolldown
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    target: 'baseline-widely-available',
    outDir: 'dist',
    minify: 'oxc',  // v8 default, 30-90x faster than Terser
    rolldownOptions: {  // v8: rolldownOptions, not rollupOptions
      output: {
        manualChunks: { vendor: ['react', 'react-dom'] }
      }
    }
  }
})
```

---

## Skills (22)

| Category | Count | Skills |
|----------|:-----:|--------|
| **[Core](skills/source/vite-core/)** | 2 | Architecture, Environment API (v6+) |
| **[Syntax](skills/source/vite-syntax/)** | 8 | Config, Server, Build, Resolve+CSS, Plugin API, HMR API, Assets, Env Vars |
| **[Implementation](skills/source/vite-impl/)** | 7 | Project Setup, Library Mode, SSR, Backend Integration, Optimization, JavaScript API, Migration |
| **[Errors](skills/source/vite-errors/)** | 3 | Build Errors, Dependency Errors, Dev Server Errors |
| **[Agents](skills/source/vite-agents/)** | 2 | Code Review (50-check checklist), Project Scaffolder |

See [INDEX.md](INDEX.md) for the complete skill catalog with descriptions.

## Version Coverage

| Vite Version | Internal Tools | Status |
|-------------|---------------|--------|
| **Vite 8.x** | Oxc + Rolldown | Primary target |
| **Vite 7.x** | esbuild + Rollup | Full coverage |
| **Vite 6.x** | esbuild + Rollup | Full coverage |
| Vite 5.x | esbuild + Rollup | Migration guide only |

All skills note version-specific differences (e.g., `build.rolldownOptions` v8 vs `build.rollupOptions` v6/v7).

## Installation

### Claude Code

```bash
# Option 1: Clone the full package
git clone https://github.com/OpenAEC-Foundation/Vite-Claude-Skill-Package.git
cp -r Vite-Claude-Skill-Package/skills/source/ ~/.claude/skills/vite/

# Option 2: Add as git submodule
git submodule add https://github.com/OpenAEC-Foundation/Vite-Claude-Skill-Package.git .claude/skills/vite
```

### Claude.ai (Web)

Upload individual SKILL.md files as project knowledge.

## Coverage Areas

| Area | Key Topics |
|------|-----------|
| **Configuration** | `defineConfig()`, conditional config, all shared/server/build options |
| **Plugin API** | All hooks (Vite-specific + Rollup-compatible), virtual modules, hook filters |
| **HMR** | `import.meta.hot`, accept/dispose/prune/invalidate, custom events, boundaries |
| **SSR** | Middleware mode, ssrLoadModule, externals, build commands, SSR manifest |
| **Library Mode** | Multi-format output (ES/CJS/UMD/IIFE), externals, DTS, package.json exports |
| **Environment API** | Vite 6+ multi-environment, per-env config, custom providers |
| **Assets** | Import suffixes (?url/?raw/?inline/?worker), glob import, Web Workers, WASM |
| **Optimization** | Pre-bundling, CJS-to-ESM, monorepo linked deps, caching |
| **Migration** | v5→v6→v7→v8, Rollup→Rolldown, esbuild→Oxc, breaking changes per version |
| **JavaScript API** | createServer, build, preview, resolveConfig, mergeConfig, utilities |

## Quality Guarantees

| Guarantee | How |
|-----------|-----|
| **Version-correct** | All code specifies Vite 6/7/8 target with version-specific annotations |
| **API-accurate** | All config options and hooks verified via WebFetch against official docs |
| **Deterministic** | ALWAYS/NEVER language, not suggestions — skills are instructions |
| **Anti-pattern-free** | Known mistakes documented with WRONG/CORRECT examples |
| **Self-contained** | Each skill works independently without requiring other skills |
| **Concise** | All SKILL.md files under 500 lines; heavy content in references/ |

## Documentation

| Document | Purpose |
|----------|---------|
| [INDEX.md](INDEX.md) | Complete skill catalog |
| [ROADMAP.md](ROADMAP.md) | Project status (single source of truth) |
| [REQUIREMENTS.md](REQUIREMENTS.md) | Quality guarantees and per-area requirements |
| [DECISIONS.md](DECISIONS.md) | Architectural decisions with rationale |
| [SOURCES.md](SOURCES.md) | Official reference URLs and verification rules |
| [WAY_OF_WORK.md](WAY_OF_WORK.md) | 7-phase development methodology |
| [LESSONS.md](LESSONS.md) | Lessons learned during development |

## Methodology

Built using the **7-phase research-first methodology**, proven across 4 skill packages:

1. **Setup** — Project structure and governance
2. **Deep Research** — WebFetch against official Vite docs (vite.dev)
3. **Masterplan** — 22 skills, 8 execution batches, full agent prompts
4. **Topic Research** — Embedded in skill creation prompts
5. **Skill Creation** — Parallel agent execution (3 per batch)
6. **Validation** — Structural + content checks
7. **Publication** — GitHub release

## Related Projects

| Project | Skills | Technology |
|---------|:------:|-----------|
| [ERPNext Skill Package](https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package) | 28 | ERPNext/Frappe |
| [Blender-Bonsai Skill Package](https://github.com/OpenAEC-Foundation/Blender-Bonsai-ifcOpenshell-Sverchok-Claude-Skill-Package) | 73 | Blender, Bonsai, IfcOpenShell, Sverchok |
| [Tauri 2 Skill Package](https://github.com/OpenAEC-Foundation/Tauri-2-Claude-Skill-Package) | 27 | Tauri 2 desktop apps |
| [OpenAEC Foundation](https://github.com/OpenAEC-Foundation) | — | Parent organization |

## License

[MIT](LICENSE)

---

Part of the [OpenAEC Foundation](https://github.com/OpenAEC-Foundation) ecosystem.
