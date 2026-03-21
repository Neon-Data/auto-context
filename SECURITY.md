# Security Policy

## Supported Versions

| Version | Supported |
|---------|-----------|
| 1.x     | Yes       |

## How This Action Handles Sensitive Data

- **API keys** are passed via GitHub Actions secrets and never logged or stored by this action.
- **Repository code** is sent to the Anthropic API for analysis. No code is sent to any other third party.
- **Claude's tools are sandboxed** to read-only git commands (`git diff`, `git log`, `git show`, `git ls-files`, `git blame`) and local file operations (`Read`, `Write`, `Edit`). Claude cannot execute arbitrary shell commands, push to remotes, or call GitHub APIs.
- **GitHub token** permissions should be scoped to `contents: write` and `pull-requests: write` only. Use the default `GITHUB_TOKEN` rather than a PAT when possible.

## Reporting a Vulnerability

If you discover a security vulnerability, please report it responsibly:

1. **Do not** open a public GitHub issue.
2. Email **mezoistvan1@gmail.com** with a description of the vulnerability.
3. Include steps to reproduce, if possible.
4. You will receive a response within 72 hours.

We appreciate responsible disclosure and will credit reporters in the fix release notes (unless anonymity is preferred).
