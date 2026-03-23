# Auto-Context

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Auto--Context-purple?logo=github)](https://github.com/marketplace/actions/auto-context)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Keep your AI-agent context files (CLAUDE.md, AGENT.md, etc.) in sync with your codebase — automatically.

> **New here?** Read [Why Auto-Context exists](WHY.md) for the problem this solves and the thinking behind it.

---

## How It Works

Auto-Context runs on PR events and main branch pushes. It uses Claude to analyze recent code changes, checks if any context files reference stale information, and proposes or applies updates.

| Trigger | What happens |
|---------|-------------|
| PR opened or updated | Checks context files against the PR diff, applies changes based on `pr-mode` |
| Push to main branch | Checks context files against the push diff, applies changes based on `main-mode` |

### PR modes

| Mode | Behavior |
|------|----------|
| `comment` | Posts a PR comment showing proposed changes with a link to create a merge PR |
| `auto-pr` | Creates a PR targeting the current PR branch with the proposed changes |
| `auto-commit` (default) | Commits the changes directly to the PR branch |

### Main branch modes

| Mode | Behavior |
|------|----------|
| `pr` (default) | Opens a PR with the proposed changes targeting the main branch |
| `direct-commit` | Commits the changes directly to the main branch |

### Safety model

Claude only operates on a local checkout of the repository — it never has access to the remote. The action runs in two distinct phases:

1. **Claude phase:** Claude reads diffs and edits context files in the checked-out working tree. Its tools are restricted to read-only git commands (`git diff`, `git log`, `git show`, `git ls-files`, `git blame`) and local file operations. It cannot push, create branches, open PRs, or call GitHub APIs.
2. **Apply phase:** After Claude's session ends, deterministic bash handles all git and GitHub operations — committing, pushing, creating PRs, and posting comments.

This separation means Claude can never modify the remote repository directly. All changes flow through the action's controlled commit/push logic, which includes loop prevention and configurable delivery modes.

---

## Quick Start

### 1. Add your API key

Go to **Settings > Secrets and variables > Actions** and add `ANTHROPIC_API_KEY` with your Anthropic API key.

### 2. Add workflows

Create `.github/workflows/auto-context-pr.yml`:

```yaml
name: "Auto-Context: PR Update"

on:
  pull_request:
    types: [opened, synchronize]

concurrency:
  group: auto-context-${{ github.head_ref }}
  cancel-in-progress: true

permissions:
  contents: write
  pull-requests: write

jobs:
  update-context:
    runs-on: ubuntu-latest
    if: github.actor != 'github-actions[bot]'
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          ref: ${{ github.head_ref }}

      - uses: Neon-Data/auto-context@beta
        with:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

Create `.github/workflows/auto-context-main.yml`:

```yaml
name: "Auto-Context: Main Branch Update"

on:
  push:
    branches:
      - main

concurrency:
  group: auto-context-main
  cancel-in-progress: false

permissions:
  contents: write
  pull-requests: write

jobs:
  update-context:
    runs-on: ubuntu-latest
    if: >-
      github.actor != 'github-actions[bot]'
      && !contains(github.event.head_commit.message, '[auto-context]')
      && github.event.head_commit.committer.username != 'web-flow'
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: Neon-Data/auto-context@beta
        with:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `anthropic-api-key` | Yes | — | Anthropic API key |
| `github-token` | No | `${{ github.token }}` | GitHub token for PR comments and creating PRs |
| `model` | No | `claude-opus-4-6` | Claude model to use |
| `context-files` | No | `""` (auto-discover) | Comma-separated context file names/globs. When empty, Claude discovers them automatically |
| `pr-mode` | No | `auto-commit` | How to apply changes on PRs: `comment`, `auto-pr`, or `auto-commit` |
| `main-mode` | No | `pr` | How to apply changes on main pushes: `pr` or `direct-commit` |
| `init` | No | `true` | Create a minimal `CLAUDE.md` if no context files exist yet |
| `debug` | No | `false` | Show full Claude output in the GitHub Actions logs |

**Note:** The main/default branch is detected automatically from the repository settings. No configuration needed.

## Outputs

| Output | Description |
|--------|-------------|
| `context-updated` | `true` if context files were changed |
| `pr-url` | URL of created PR (when applicable) |

---

## Permissions

Your workflow needs these permissions:

```yaml
permissions:
  contents: write       # Push branches and commits
  pull-requests: write  # Post comments and create PRs
```

---

## Context File Discovery

By default, Auto-Context discovers context files automatically. It looks for:

- `CLAUDE.md` at the repo root and in subdirectories
- `AGENT.md` and `AGENTS.md` at the repo root and in subdirectories
- `.cursorrules` at the repo root
- Files in a `context/` folder (see [Scaling large context files](#scaling-large-context-files) below)
- Any other markdown files that serve as AI-agent context

To restrict which files are checked, use the `context-files` input:

```yaml
- uses: Neon-Data/auto-context@beta
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    context-files: "CLAUDE.md,src/api/CLAUDE.md"
```

---

## Bootstrapping a New Repo

When Auto-Context runs for the first time on a repo with no context files, it automatically creates a `CLAUDE.md` and populates it with real information about your project — tech stack, project structure, development commands, and conventions — all inferred by Claude from the actual codebase.

This means you can add Auto-Context to any repo and get a useful context file on the very first run, with no manual setup.

To disable this behavior:

```yaml
- uses: Neon-Data/auto-context@beta
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    init: false
```

---

## Examples

### PR with comment mode

Post a comment with proposed changes instead of committing directly:

```yaml
- uses: Neon-Data/auto-context@beta
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    pr-mode: comment
```

### PR with auto-PR

Create a separate PR with the context updates targeting the PR branch:

```yaml
- uses: Neon-Data/auto-context@beta
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    pr-mode: auto-pr
```

### Main branch with direct commit

Push context updates directly to main:

```yaml
- uses: Neon-Data/auto-context@beta
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    main-mode: direct-commit
```

### Using a different model

```yaml
- uses: Neon-Data/auto-context@beta
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    model: claude-sonnet-4-6
```

---

## Scaling Large Context Files

As a project grows, a single `CLAUDE.md` can become too long for agents to scan quickly. When Auto-Context detects that a context file is getting unwieldy, it will automatically break sections into separate markdown files inside a `context/` folder and link them back from the main file:

```
CLAUDE.md               ← concise index with links
context/
  ARCHITECTURE.md       ← detailed architecture docs
  CONVENTIONS.md        ← coding conventions
  API.md                ← API reference
  ...
```

Links in the main file use the format:

```markdown
- [Architecture](context/ARCHITECTURE.md)
- [Conventions](context/CONVENTIONS.md)
```

This keeps the root context file small and scannable while preserving all the detail in linked pages that agents can follow when they need deeper information.

---

## When Does It Run?

Auto-Context is designed around two triggers:

| Trigger | Runs on | Skips |
|---------|---------|-------|
| **PR** | Every PR open/update | Bot-authored PRs |
| **Main branch** | Direct pushes only | PR merges (already handled by the PR trigger) |

PR merges to main are intentionally skipped because context files were already checked (and updated if needed) when the PR was open. The `web-flow` committer filter in the example workflow handles this — `web-flow` is GitHub's merge-commit author for PRs merged via the UI.

### Loop prevention

Multiple layers prevent infinite re-triggering when Auto-Context pushes commits or creates PRs:

| Layer | Mechanism | Where |
|-------|-----------|-------|
| 1 | `github.actor != 'github-actions[bot]'` | Consumer workflow `if` condition |
| 2 | `!contains(commit.message, '[auto-context]')` | Consumer workflow `if` condition (main branch) |
| 3 | `committer.username != 'web-flow'` | Consumer workflow `if` condition (main branch) |
| 4 | `GITHUB_TOKEN` pushes don't trigger `pull_request` events | GitHub built-in behavior |
| 5 | PRs created by the action get `auto-context` label, action skips labeled PRs | Action internal check |

Layer 1 is the cheapest (prevents the workflow from starting). The remaining layers are failsafes.

---

## FAQ

**Does it work with monorepos?**
Yes. Auto-Context discovers component-level context files (e.g., `packages/api/CLAUDE.md`) and updates them independently.

**Does it work with private repos?**
Yes. The action runs in your GitHub Actions environment. No code leaves the runner except what is sent to the Anthropic API.

**What if I don't have context files yet?**
By default (`init: true`), Auto-Context uses Claude to analyze your repository and create a comprehensive `CLAUDE.md` covering tech stack, project structure, development commands, and conventions. Set `init: false` if you prefer to create context files manually.

**What happens when my context file gets really long?**
Auto-Context will automatically split it into a `context/` folder with separate topic files (e.g., `context/ARCHITECTURE.md`) and turn the main file into a concise index linking to them. See [Scaling Large Context Files](#scaling-large-context-files).

**What about branch protection on main?**
If using `main-mode: direct-commit`, the bot needs push access. Either allow the GitHub Actions bot to bypass protection rules, or use a PAT with admin access. The default `main-mode: pr` avoids this issue by creating a PR instead.

**Can I use it with self-hosted runners?**
Yes. The action installs the `@anthropic-ai/claude-code` CLI via npm at runtime. Ensure your runner has Node.js 18+, npm, and network access to the Anthropic API.

---

## License

[MIT](LICENSE)
