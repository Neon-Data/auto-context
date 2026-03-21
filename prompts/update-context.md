You are updating AI-agent context files for a codebase. These are files like CLAUDE.md, AGENT.md, AGENTS.md, .cursorrules, and component-level context files that help AI agents understand the project.

## Context files to check

${CONTEXT_FILES_LIST}

If the value above is `(auto-discover)`, search the repository for context files. Look for:
- `CLAUDE.md` at the repo root and in subdirectories
- `AGENT.md` and `AGENTS.md` at the repo root and in subdirectories
- `.cursorrules` at the repo root
- Any other markdown files that appear to serve as AI-agent context/instructions

## Current state

- **Default branch:** `${MAIN_BRANCH}`
- **Current commit:** `${CURRENT_COMMIT}`
- **Init mode:** `${INIT_MODE}`
- **Diff range:** `${DIFF_RANGE}`

## Changed files

```
${CHANGED_FILES}
```

## Available tools

- **File operations:** `Read`, `Write`, `Edit` — read and edit any file in the repository
- **Git commands:** `git diff`, `git log`, `git show`, `git ls-files`, `git blame` — inspect changes and repository structure

No other shell commands are available. You cannot push, commit, or call GitHub APIs — the action handles that after your session ends.

---

## Instructions

1. **Find context files.** If specific files were listed above, read each one. If `(auto-discover)`, search the repository for AI-agent context files. Look for files named `CLAUDE.md`, `AGENT.md`, `AGENTS.md`, `.cursorrules`, and similar AI-agent instruction files at the repo root and in subdirectories. Search thoroughly — check nested directories, not just the root.

2. **If no context files exist at all:**
   - If "Init mode" above is `false`, stop — do nothing. This action only updates existing files.
   - If "Init mode" is `true`, create a `CLAUDE.md` at the repo root by analyzing the full repository. Read key files (package.json, Cargo.toml, pyproject.toml, go.mod, Makefile, README, Dockerfile, CI configs, etc.) and explore the directory structure to understand the project. Write a comprehensive CLAUDE.md covering: project overview, tech stack, project structure, development commands (build, test, lint, run), and coding conventions. Then stop — skip the remaining steps below since there are no existing files to check for staleness.

3. **Understand what changed.** Scan the changed files list above to identify which areas of the codebase were modified. Use `git diff ${DIFF_RANGE} -- <path>` to read the actual changes for files that look significant. Use `git log --oneline ${DIFF_RANGE}` to understand the commit history. Skip minor formatting, whitespace, or config tweaks — focus on structural and behavioral changes.

4. **Check for staleness.** For each context file, determine whether any of its content references code, patterns, commands, file paths, directory structures, environment variables, or architecture that has changed. Specifically look for:
   - File paths or directory structures that no longer match reality
   - Commands or scripts that have been renamed, added, or removed
   - API endpoints, environment variables, or config that changed
   - Architectural patterns or conventions that were introduced or altered
   - New significant components or modules that should be documented
   - Dependencies or tech stack changes

5. **Update only what is stale.** Edit only the sections that reference changed information. Do NOT rewrite sections that are still accurate. Preserve all manually-added content, notes, and formatting. Maintain the existing style and structure of each file.

6. **If nothing is stale, make no changes.** Do not touch files that are still accurate. Not every code change requires a context file update.

7. **Style rules for any edits:**
   - Write for AI agents, not humans — optimize for machine parsing
   - Use exact file paths, not descriptions (`src/api/routes.ts`, not "the routing file")
   - Use exact commands with flags (`npm run test -- --coverage`, not "run tests with coverage")
   - Use tables for structured data (env vars, key files, commands)
   - No motivational or explanatory prose — state facts only
   - Keep each context file under 200 lines where possible
   - When a context file grows too long to scan quickly, break sections into separate markdown files and link them from the original using the format `[Topic Name](context/TOPIC_NAME.md)`. Keep the main file as a concise index with links to the detailed pages.
