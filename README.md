# Vite Claude Skill Package

<p align="center">
  <strong style="font-size: 1.5em; color: #646CFF;">Deterministic Claude AI Skills for Vite 5 & 6</strong>
</p>

![Claude Code Ready](https://img.shields.io/badge/Claude_Code-Ready-blue?style=flat-square&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZD0iTTEyIDJDNi40OCAyIDIgNi40OCAyIDEyczQuNDggMTAgMTAgMTAgMTAtNC40OCAxMC0xMFMxNy41MiAyIDEyIDJ6IiBmaWxsPSIjZmZmIi8+PC9zdmc+)
![Vite 5 & 6](https://img.shields.io/badge/Vite-5_%26_6-646CFF?style=flat-square&logo=vite&logoColor=white)
![TypeScript + JavaScript](https://img.shields.io/badge/TypeScript_+_JavaScript-Full_Coverage-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Skills](https://img.shields.io/badge/Skills-In_Progress-orange?style=flat-square)
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

With this skill package, Claude produces correct Vite configuration:

```typescript
// Correct — Vite config with defineConfig for type safety
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    target: 'es2020',
    outDir: 'dist',
    rollupOptions: {
      output: {
        manualChunks: { vendor: ['react', 'react-dom'] }
      }
    }
  }
})
```

---

## Current Progress

| Phase | Name | Status |
|-------|------|--------|
| 1 | Setup + Raw Masterplan | IN PROGRESS (50%) |
| 2 | Deep Research | NOT STARTED |
| 3 | Masterplan Refinement | NOT STARTED |
| 4 | Topic-Specific Research | NOT STARTED |
| 5 | Skill Creation | NOT STARTED |
| 6 | Validation | NOT STARTED |
| 7 | Publication | NOT STARTED |

## Planned Skill Categories

| Category | Purpose |
|----------|---------|
| **vite-core/** | Architecture, dev server, build pipeline, environment API |
| **vite-syntax/** | Config options, plugin API hooks, HMR API, SSR API |
| **vite-impl/** | Project setup, library mode, SSR integration, monorepo |
| **vite-errors/** | Build errors, dependency issues, HMR failures, plugin conflicts |
| **vite-agents/** | Config validation, project scaffolding |

## Coverage Areas

| Area | Topics |
|------|--------|
| **Configuration** | `defineConfig()`, conditional config, environment variables, mode |
| **Plugin API** | Hook lifecycle, virtual modules, HMR handling, enforce ordering |
| **HMR** | `import.meta.hot`, accept/dispose, custom events, boundaries |
| **SSR** | Module loading, externalization, build config, streaming |
| **Library Mode** | Multi-format output, externals, type generation |
| **Environment API** | Vite 6 multi-environment, per-env config, custom environments |
| **Asset Handling** | Import suffixes, public directory, URL resolution |
| **Dep Optimization** | Pre-bundling, CJS-to-ESM, monorepo linked deps |
| **Build** | Chunk splitting, targets, Rollup options, CSS code-split |

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

## Version Compatibility

| Technology | Versions | Notes |
|------------|----------|-------|
| Vite | **5.x, 6.x** | Primary targets, differences noted per skill |
| Node.js | 18+ | Required for Vite 5/6 |
| TypeScript | 5.x | Config and plugin type safety |

## Methodology

This package is developed using the **7-phase research-first methodology**, proven across multiple skill packages:

1. **Setup + Raw Masterplan** — Project structure and governance files
2. **Deep Research** — Comprehensive source analysis of Vite documentation, source code, and community resources
3. **Masterplan Refinement** — Skill inventory refinement based on research findings
4. **Topic-Specific Research** — Deep-dive per skill topic
5. **Skill Creation** — Deterministic skill files following Agent Skills standard
6. **Validation** — Correctness, completeness, and consistency checks
7. **Publication** — GitHub release and documentation

## Documentation

| Document | Purpose |
|----------|---------|
| [ROADMAP.md](ROADMAP.md) | Project status (single source of truth) |
| [REQUIREMENTS.md](REQUIREMENTS.md) | Quality guarantees and per-area requirements |
| [DECISIONS.md](DECISIONS.md) | Architectural decisions with rationale |
| [SOURCES.md](SOURCES.md) | Official reference URLs and verification rules |
| [WAY_OF_WORK.md](WAY_OF_WORK.md) | 7-phase development methodology |
| [LESSONS.md](LESSONS.md) | Lessons learned during development |
| [CHANGELOG.md](CHANGELOG.md) | Version history |

## Related Projects

| Project | Description |
|---------|-------------|
| [ERPNext Skill Package](https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package) | 28 skills for ERPNext/Frappe development |
| [Blender-Bonsai Skill Package](https://github.com/OpenAEC-Foundation/Blender-Bonsai-ifcOpenshell-Sverchok-Claude-Skill-Package) | 73 skills for Blender, Bonsai, IfcOpenShell & Sverchok |
| [Tauri 2 Skill Package](https://github.com/OpenAEC-Foundation/Tauri-2-Claude-Skill-Package) | 27 skills for Tauri 2 desktop development |
| [OpenAEC Foundation](https://github.com/OpenAEC-Foundation) | Parent organization |

## License

[MIT](LICENSE)

---

Part of the [OpenAEC Foundation](https://github.com/OpenAEC-Foundation) ecosystem.
