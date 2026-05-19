# Changelog

All notable changes to the Tailwind CSS Skill Package.

Format follows [Keep a Changelog](https://keepachangelog.com/).

## [1.0.0] : 2026-05-19

### Added
- 30 deterministic Claude skills across 5 categories (core 3, syntax 10, impl 11, errors 5, agents 1)
- Dual coverage of Tailwind CSS v3.4 and v4.0+ per skill where versions diverge (Decision D-008)
- 9 lessons learned (L-001..L-009) covering v4 directive removal, @reference scoped trap, removed config options, variant stacking order flip, default visual changes, dynamic-class anti-pattern, tmux worker context overflow, validator folder-prefix convention, and Keywords-regex period-cut behaviour
- Compliance audit report at 100% (4/4 validation checks passing)
- INDEX.md with full skill catalog and dependency graph
- package.json `agents.skills[]` manifest (30 entries)
- agents/openai.yaml for OpenAI Codex skill discovery
- README with installation paths, quality guarantees, companion-package matrix

### Methodology
- 7-phase research-first methodology executed end-to-end
- Phase 2 deep research : 5668 words, 30 sources WebFetched + verified
- Phase 3 masterplan refinement : 12 decisions backed by research
- Phase 5 skill creation : tmux-orchestration with 3 persistent skill-builder workers across 10 batches
- Phase 6 validation : 100% audit score, all skills pass frontmatter / line-count / structure / language / em-dash checks

### Tooling
- CI/CD : .github/workflows/quality.yml validates frontmatter, line count, structure, language, em-dash on push and PR
- Skill-Package-Workflow-Template scripts used for all validation and manifest generation

## [Unreleased]

(no unreleased changes)
