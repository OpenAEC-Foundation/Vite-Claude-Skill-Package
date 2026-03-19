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

---

## L-004: Always WebFetch Before Assuming Version State

- **Date**: 2026-03-19
- **Context**: CLAUDE.md assumed Vite 5/6 were current. WebFetch revealed Vite 8 (Rolldown+Oxc) is current.
- **Finding**: NEVER rely on training data for version information. The gap between training data and reality can be multiple major versions. WebFetch MUST be the first step in any research phase. This single check changed the entire scope of the project.

---

## L-005: Domain Migrations Happen Silently

- **Date**: 2026-03-19
- **Context**: All SOURCES.md URLs used vitejs.dev. WebFetch revealed 301 redirect to vite.dev.
- **Finding**: Official documentation domains can migrate between training data and now. Always verify canonical URLs via WebFetch and update SOURCES.md immediately.

---

## L-006: Phases 1-3 Can Be Compressed When Research Is Strong

- **Date**: 2026-03-19
- **Context**: Traditional 7-phase methodology separates raw masterplan, research, and refinement into distinct phases.
- **Finding**: When research (Phase 2) is thorough and done early, the raw masterplan (Phase 1) and refinement (Phase 3) can be done in a single pass. The research fragments serve as both vooronderzoek AND topic-specific research, allowing Phase 4 to be embedded in Phase 5 agent prompts.

---

## L-007: Commit Per Phase, Not Per Session

- **Date**: 2026-03-19
- **Context**: Compliance audit revealed all phases (2-6) were committed in a single git commit.
- **Finding**: Even when executing multiple phases in one session, ALWAYS commit after each phase completes. This creates traceable git history that matches ROADMAP.md phase boundaries. A single monolithic commit makes it impossible to verify phase ordering and prevents meaningful `git bisect` or phase-level rollback.

---

## L-008: YAML Frontmatter Must Use Folded Block Scalar From Day One

- **Date**: 2026-03-19
- **Context**: Compliance audit found all 22 skills using quoted strings instead of `>` folded block scalar for descriptions.
- **Finding**: The masterplan agent prompts contained quoted string descriptions, which agents copied verbatim. Ensure masterplan prompts ALREADY use the `>` format so agent output matches the standard without post-hoc migration. Also ensure descriptions follow the "Use when... / Prevents... / Keywords:..." structure.

---

## L-009: Masterplan Refinement Decisions Must Sync to DECISIONS.md

- **Date**: 2026-03-19
- **Context**: Compliance audit found masterplan-local decisions (D-01 to D-08) not reflected in project-level DECISIONS.md.
- **Finding**: When the masterplan includes refinement decisions (merge, remove, reorder skills), these MUST be recorded in DECISIONS.md with project-level numbering (D-013+). The masterplan is an execution document; DECISIONS.md is the authoritative record.
