# CLAUDE.md

## Project overview

Auto-Context is a GitHub Action that automatically keeps AI-agent context files (CLAUDE.md, AGENT.md, AGENTS.md, .cursorrules) in sync with codebases. It runs on PR and main-branch push events, uses Claude to analyze diffs, and proposes or applies updates to context files.

Published on GitHub Marketplace as `Neon-Data/auto-context`.

## Tech stack

- **Runtime:** GitHub Actions composite action (pure bash, no Node.js/Docker build step)
- **AI:** Anthropic Claude via `anthropics/claude-code-action@v1`
- **Prompt templating:** `envsubst` (shell-native variable substitution)
- **License:** MIT

## Repository structure

| Path | Purpose |
|------|---------|
| `action.yml` | Main composite action definition — all action logic lives here |
| `prompts/update-context.md` | Prompt template sent to Claude (uses `${VAR}` placeholders) |
| `examples/workflow-pr.yml` | Example consumer workflow for PR triggers |
| `examples/workflow-main.yml` | Example consumer workflow for main-branch push triggers |
| `README.md` | User-facing documentation |
| `LICENSE` | MIT license |
| `.gitignore` | Ignores node_modules, .DS_Store, swap files |

There is no `src/`, `dist/`, or build output. The entire action is a single `action.yml` with inline bash.

## Architecture

### Sandboxed execution model

Claude only operates on a local checkout of the repository — it never has access to the remote. The pipeline works in two phases:

1. **Claude phase** (step 4): Claude reads diffs and edits context files in the checked-out working tree. Its tools are restricted to read-only git commands (`git diff`, `git log`, `git show`) and file operations. It cannot push, create branches, open PRs, or call GitHub APIs.
2. **Apply phase** (step 6): Deterministic bash handles all git and GitHub operations — committing, pushing, creating PRs, and posting comments. This runs after Claude's session ends.

This separation ensures Claude can never modify the remote repository directly. All changes go through the action's controlled commit/push logic, which includes loop prevention, proper labeling, and configurable modes.

### Action pipeline (6 steps in `action.yml`)

1. **detect** — Determines trigger type (PR vs push), extracts PR number, head branch, before SHA
2. **should-run** — Loop prevention: skips if commit message contains `[auto-context]` or PR has `auto-context` label
3. **gather** — Computes diff range, lists changed files, renders the prompt template via `envsubst`
4. **run-claude** — Invokes `anthropics/claude-code-action@v1` with the rendered prompt; Claude reads diffs dynamically using `git diff`, `git log`, `git show`, `git ls-files`, `git blame`
5. **check-changes** — Checks if Claude modified any files
6. **apply** — Commits and pushes/PRs/comments based on mode settings

### Trigger modes

| Trigger | Modes | Default |
|---------|-------|---------|
| PR (`pull_request`) | `comment`, `auto-pr`, `auto-commit` | `comment` |
| Main branch (`push`) | `pr`, `direct-commit` | `pr` |

### Loop prevention layers

1. `github.actor != 'github-actions[bot]'` — workflow-level skip
2. `!contains(commit.message, '[auto-context]')` — commit message tag
3. `committer.username != 'web-flow'` — skip PR merges on main
4. `GITHUB_TOKEN` pushes don't trigger `pull_request` events (GitHub built-in)
5. PRs created by the action get `auto-context` label; action skips labeled PRs

### Prompt template variables

The prompt at `prompts/update-context.md` uses these `envsubst` variables:

| Variable | Source |
|----------|--------|
| `${CONTEXT_FILES_LIST}` | User input or `(auto-discover)` |
| `${DIFF_RANGE}` | Git range for the changes (e.g., `abc123..HEAD`) |
| `${CHANGED_FILES}` | Full `git diff --name-only` output (no truncation) |
| `${MAIN_BRANCH}` | Repository default branch |
| `${CURRENT_COMMIT}` | Current HEAD SHA |
| `${INIT_MODE}` | Whether to bootstrap CLAUDE.md if none exists |

## Action inputs

| Input | Required | Default |
|-------|----------|---------|
| `anthropic-api-key` | Yes | — |
| `github-token` | No | `${{ github.token }}` |
| `model` | No | `claude-opus-4-6` |
| `context-files` | No | `""` (auto-discover) |
| `pr-mode` | No | `comment` |
| `main-mode` | No | `pr` |
| `init` | No | `true` |
| `debug` | No | `false` |

## Development

There is no build step, test suite, or package manager. The action is a single composite YAML file.

### Testing locally

There is no local test harness. Test by pushing to a branch and triggering the action in a consumer repo.

### Key conventions

- All logic in `action.yml` inline bash — no external scripts
- Commit messages for auto-generated commits use the format: `docs: update context files [auto-context]`
- Branch naming: `auto-context/pr-{NUMBER}-{EPOCH}-{SHORT_SHA}` or `auto-context/main-{EPOCH}-{SHORT_SHA}` (epoch timestamp prevents collisions on re-runs for the same commit)
- Git author for bot commits: `auto-context[bot] <auto-context[bot]@users.noreply.github.com>`
- Claude reads diffs dynamically via git tools — only the file list and diff range are embedded in the prompt
- Claude's Bash tools are restricted to read-only git commands (`diff`, `log`, `show`, `ls-files`, `blame`); file operations (`Read`, `Write`, `Edit`) are available for editing context files locally
- When context files exceed ~200 lines, Claude splits sections into `context/TOPIC_NAME.md` files and links them from the main file using `[Topic Name](context/TOPIC_NAME.md)`
