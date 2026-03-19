# DECISIONS

Architectural and process decisions with rationale. Each decision is numbered and immutable once recorded. New decisions may supersede old ones but old ones are never deleted.

---

## D-001: 7-Phase Research-First Methodology
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Need a structured approach to build high-quality skills
**Decision**: Adopt the 7-phase methodology proven in the ERPNext, Blender-Bonsai, and Tauri Skill Packages
**Rationale**: ERPNext produced 28 skills, Blender-Bonsai produced 73 skills, Tauri produced 27 skills with this approach. Research-first prevents hallucinated content.
**Reference**: https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package/blob/main/WAY_OF_WORK.md

## D-002: Single Technology Package
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Vite is one technology (build tool) unlike multi-technology packages
**Decision**: No per-technology separation needed. All skills share the `vite-` prefix under a single `skills/source/` tree.
**Rationale**: Vite is a single build tool. The plugin API, config system, and dev server are all part of the same framework. No sub-package splitting needed.

## D-003: English-Only Skills
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Team works primarily in Dutch, skills target international audience
**Decision**: ALL skill content in English only
**Rationale**: Skills are instructions for Claude, not end-user documentation. Claude reads English and responds in any language. Bilingual skills double maintenance with zero functional benefit. Proven in ERPNext, Blender, and Tauri projects.

## D-004: Claude Code Agent Tool for Orchestration
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Need to produce ~25 skills efficiently. Windows environment, no oa-cli available.
**Decision**: Use Claude Code Agent tool for parallel execution instead of oa-cli
**Rationale**: Windows environment does not support oa-cli (requires WSL/Linux). Claude Code Agent tool provides native parallelism within the Claude Code session.

## D-005: MIT License
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Need to choose open-source license
**Decision**: MIT License
**Rationale**: Most permissive, maximizes adoption. Consistent with OpenAEC Foundation philosophy.

## D-006: ROADMAP.md as Single Source of Truth
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Need to track project status across multiple sessions and agents
**Decision**: ROADMAP.md is the ONLY place where project status is tracked
**Rationale**: Multiple status locations cause drift and confusion. Single source prevents "which is current?" questions.

## D-007: Vite 6, 7, and 8 Coverage (SUPERSEDED by D-008)
**Date**: 2026-03-19
**Status**: SUPERSEDED
**Context**: Originally assumed Vite 5/6 were current
**Decision**: Skills cover Vite 5 and 6
**Rationale**: Superseded after research revealed Vite 8 is current

## D-008: Vite Version Scope — v6 through v8 (current)
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Research via WebFetch (2026-03-19) revealed Vite is at v8 with Rolldown replacing Rollup and Oxc replacing esbuild. Domain migrated from vitejs.dev to vite.dev.
**Decision**: Skills cover Vite 6, 7, and 8. Vite 5 and older covered ONLY in migration context. v8 is primary target. Where APIs differ between versions (e.g., esbuild→Oxc, Rollup→Rolldown, build.rollupOptions→build.rolldownOptions), skills MUST note version-specific options.
**Rationale**: Vite 8 uses Rolldown+Oxc (completely different build pipeline from v5/v6). v6 introduced Environment API. v7 dropped Node 18. These three versions represent the modern Vite stack. v5 users need migration guidance, not skill coverage.

## D-009: SKILL.md Line Limit
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Skills must be concise and focused
**Decision**: SKILL.md files MUST be under 500 lines. Heavy content goes in references/ subdirectory.
**Rationale**: Proven in ERPNext, Blender, and Tauri packages. Keeps skills scannable and prevents Claude context bloat.

## D-010: Domain Update — vite.dev
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Research revealed vitejs.dev now 301-redirects to vite.dev
**Decision**: All source URLs updated to use vite.dev domain. Historical docs at v6.vite.dev, v7.vite.dev.
**Rationale**: Follow canonical domain to prevent redirect chains.

## D-011: 22 Skills — Definitive Count
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Research completed, skill inventory finalized based on Vite API surface
**Decision**: 22 skills across 5 categories (core: 2, syntax: 8, impl: 7, errors: 3, agents: 2), executed in 8 batches.
**Rationale**: Covers complete Vite surface (config, plugin API, HMR, SSR, library mode, Environment API, assets, optimization, JavaScript API) without overlap. Sized appropriately for single-technology package.

## D-012: WebFetch Verification Mandatory
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Training data may be outdated (originally assumed v5/v6, reality is v8)
**Decision**: ALL code examples and API references MUST be verified via WebFetch against official documentation before inclusion in skills.
**Rationale**: Vite v8 introduced breaking changes (Rolldown, Oxc, Lightning CSS default). Training data contains stale information about esbuild and Rollup-based Vite.

## D-013: Merge resolve and CSS into single skill
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Masterplan refinement — resolve.* and css.* are both config subsections, neither large enough for a standalone skill.
**Decision**: Merge `vite-syntax-resolve` and `vite-syntax-css` into `vite-syntax-resolve-css`.
**Rationale**: Both are configuration subsections under vite.config.ts. Combined skill stays well under 500 lines and avoids thin standalone skills.

## D-014: Merge multi-page into build syntax skill
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Masterplan refinement — multi-page app setup is just `rolldownOptions.input` configuration.
**Decision**: Absorb `vite-impl-multi-page` into `vite-syntax-build`. Remove multi-page as standalone skill.
**Rationale**: Multi-page configuration amounts to a single config option (input object). Too thin for a standalone skill.

## D-015: Batch execution order — core first, agents last
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Masterplan refinement — need to determine optimal batch ordering.
**Decision**: Execute batches as: core → syntax → impl → errors → agents. Foundation skills built first, agent skills last (depend on all others).
**Rationale**: Standard dependency ordering proven in Tauri package. Agents need to reference all other skill patterns.

## D-016: Reference file naming — context-specific over template defaults
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: SKILL.md template specifies `methods.md`, `examples.md`, `anti-patterns.md` as reference files. Skills use context-specific names (config-options.md, api-reference.md, hooks.md, etc.).
**Decision**: Context-specific reference file names are acceptable and preferred when they improve clarity. The template provides defaults, not mandates.
**Rationale**: `config-options.md` is more discoverable than `methods.md` for a configuration skill. All reference links are valid and correctly linked from SKILL.md.

## D-017: Phase 4 Topic Research Embedded in Phase 5 Agent Prompts
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Phase 4 (Topic-Specific Research) normally produces per-skill research documents in `docs/research/topic-research/`. For this package, comprehensive research fragments from Phase 2 already covered all skill topics.
**Decision**: Skip standalone Phase 4 topic-research files. Instead, embed topic-specific research sections directly in the Phase 5 agent prompts, referencing the Phase 2 research fragments.
**Rationale**: The 3 research fragments (3138 lines total) already provide per-topic coverage. Duplicating this into separate topic-research files would add overhead without new information. See also L-006.
