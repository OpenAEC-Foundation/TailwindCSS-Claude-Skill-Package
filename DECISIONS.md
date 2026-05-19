# Architectural Decisions

Numbered decisions (D-XXX) with rationale. Immutable once recorded — new decisions can supersede but never delete old ones.

---

## D-001: English-Only Skills

- **Date**: 2026-05-19
- **Decision**: All skill content MUST be in English only.
- **Rationale**: Skills are instructions FOR Claude, not for end users. Claude reads English and responds in the user's language. Bilingual skills double maintenance with zero functional benefit. Proven in ERPNext (28 skills) and Blender-Bonsai (73 skills).
- **Consequence**: No translations needed. All descriptions, code comments, and documentation in English.

---

## D-002: MIT License

- **Date**: 2026-05-19
- **Decision**: Project uses MIT License.
- **Rationale**: Most permissive license, maximizes adoption. Consistent with OpenAEC Foundation standards.
- **Consequence**: No commercial restrictions. Community-friendly.

---

## D-003: SKILL.md Under 500 Lines

- **Date**: 2026-05-19
- **Decision**: SKILL.md files MUST be under 500 lines.
- **Rationale**: Keeps main skill focused on decision trees and quick reference. Heavy content belongs in references/ directory. Proven optimal in ERPNext (180-427 lines per skill).
- **Consequence**: Complex topics split between SKILL.md (quick reference + patterns) and references/ (complete API, examples, anti-patterns).

---

## D-004: 7-Phase Research-First Methodology

- **Date**: 2026-05-19
- **Decision**: Follow the 7-phase research-first development methodology.
- **Rationale**: Proven in ERPNext (28 skills), Blender-Bonsai (73 skills), and Tauri 2 (27 skills). Research prevents hallucination. Deterministic skills require deep understanding.
- **Consequence**: No skill creation without prior deep research. Phases are sequential with defined exit criteria.

---

## D-005: ROADMAP.md as Single Source of Truth

- **Date**: 2026-05-19
- **Decision**: ROADMAP.md is the ONLY place for project status.
- **Rationale**: Multiple status locations cause drift and "which is current?" confusion. Single source of truth enables reliable session recovery.
- **Consequence**: Never duplicate status in CLAUDE.md or other files. All status references point to ROADMAP.md.

---

## D-006: WebFetch for Research Verification

- **Date**: 2026-05-19
- **Decision**: Use WebFetch to verify all code examples against latest official documentation.
- **Rationale**: Technology APIs evolve. Training data may be stale. WebFetch ensures latest official docs are consulted, not outdated cached knowledge.
- **Consequence**: All code examples must be verified against current official documentation before inclusion in skills.

---

## D-007: GitHub Publication Under OpenAEC Foundation

- **Date**: 2026-05-19
- **Decision**: Publish all skill packages under the OpenAEC Foundation GitHub organization.
- **Rationale**: Centralized, consistent branding. Community ownership. Discoverability.
- **Consequence**: All repos follow OpenAEC naming conventions and include social preview banners with OpenAEC branding.

---

## D-008: Dual v3/v4 Coverage Per Skill

- **Date**: 2026-05-19
- **Decision**: Every skill where Tailwind v3.4 and v4.0 diverge MUST present explicit v3 column and v4 column (tables or labeled snippets), never a single shared snippet that hides version differences.
- **Rationale**: Phase 2 research (L-001 through L-005) showed v4 is more divergent from v3 than the raw masterplan assumed. `@tailwind` directives removed, JS config no longer auto-loaded, `corePlugins`/`safelist`/`separator` removed without replacement, variant stacking order flipped, default border-color + ring-width changed. A skill that picks one version silently misleads users on the other.
- **Consequence**: 30-skill inventory includes v4-only skills (functional-utilities, 3d-transforms, modern-utilities) and v3-only context in `impl-config-v3`. All shared skills use side-by-side tables.

---

## D-009: Phase 4 Topic-Research Selectively Skipped

- **Date**: 2026-05-19
- **Decision**: For Tailwind pkg, Phase 4 topic-research is SKIPPED for skills where vooronderzoek-tailwind.md already covers the topic in sufficient depth (most skills), and EXECUTED only for skills needing deeper drill-down (`impl-plugins-custom`, `impl-tailwind-merge`, `impl-migration-v3-v4`, `errors-dynamic-classes`).
- **Rationale**: BOOTSTRAP-RUNBOOK §6.3 skip-criteria (Docker L-001) authorises this when vooronderzoek is >40 doc-pages-equivalent and well-cited. Tailwind vooronderzoek is 5668 words with 30 verified sources covering every skill's scope directly.
- **Consequence**: 26 skills go straight to tmux-worker with vooronderzoek-only reference. 4 skills get topic-research first. Total time saved ~1.5h.

---

## D-010: tmux-orchestration For Phase 5

- **Date**: 2026-05-19
- **Decision**: Phase 5 skill creation uses the `tmux-orchestration` skill with 3 persistent `skill-builder` workers, batch-loop with quality-gate per worker reply.
- **Rationale**: 30 skills is above the 15-skill threshold from BOOTSTRAP-RUNBOOK §P-012. Persistent workers with file-scope-isolation + QG-loop is the proven pattern (Blender-Bonsai 73 skills, Frappe 61 skills).
- **Consequence**: Workers spawn in VS Code panels, reply-language Nederlands, QG cadence "every worker reply". File-scope updated per batch via re-instruct.
