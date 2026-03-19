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

## D-007: Vite 5 and 6 Coverage
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Vite has two current major versions (5.x and 6.x) with the Environment API as the key differentiator
**Decision**: Skills cover both Vite 5 and Vite 6. Vite 4.x and older are out of scope. Where APIs differ between v5 and v6, skills MUST note the difference explicitly.
**Rationale**: Vite 5 is still widely deployed. Vite 6 introduces the Environment API and other breaking changes. Covering both ensures the package is useful for the majority of active Vite projects. The Environment API is the most significant new feature in v6 and warrants dedicated coverage.
