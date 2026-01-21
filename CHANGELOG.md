# Changelog

All notable changes to the dev-flow plugin documented here.

## [Unreleased]

## [1.1.0] - 2026-01-21

### Changed
- Improved skill activation clarity with explicit trigger phrases in description
- Optimized SKILL.md token efficiency (225 lines, down from 288)
- Strengthened Quick Start with prominent error prevention warnings
- Reorganized Common Scenarios as concise patterns with copy-paste CHANGELOG examples
- Enhanced Critical Rules section with table format for team clarity
- Improved bracket system explanation with simplified table and "why frozen?" context

### Fixed
- Clarified Step 4 version file search with specific grep command
- Added explicit guardrails for non-amendable commits
- Added explicit "DO NOT" guidance for changelog version bump documentation

## [1.0.0] - 2026-01-21

### Added
- Release-process skill with semantic versioning guidance using bracket-based freezing
- Support for single-component, multi-component (monorepo), and plugin project structures
- Bracket system preventing version thrashing (PATCH/MINOR/MAJOR freezing)
- Changelog best practices with critical rules for user-facing documentation
- Multi-component version coordination (parent version ≥ highest child)
- Pre-release version support (alpha, beta, rc)
- Detailed reference guides covering:
  - Bracket system deep dive with state transitions
  - Changelog guidelines with section explanations and common mistakes
  - Project structure patterns (single, monorepo, plugin, skill)
  - Step-by-step workflows for common release scenarios
  - Scope creep handling and multi-component releases
