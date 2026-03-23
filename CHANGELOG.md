# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-03-21

### Added

- Composite GitHub Action for automatically updating AI-agent context files
- PR trigger modes: `auto-commit` (default), `comment`, `auto-pr`
- Main branch trigger modes: `pr` (default), `direct-commit`
- Auto-discovery of context files (`CLAUDE.md`, `AGENT.md`, `AGENTS.md`, `.cursorrules`)
- Init mode: bootstrap a `CLAUDE.md` from scratch when no context files exist
- Multi-layer loop prevention to avoid infinite re-triggering
- Prompt template with `envsubst` variable substitution
- Example workflows for PR and main branch triggers
