---
name: git-commit-lite
description: Create a well-structured local commit quickly with automatic convention detection. Use when the user says "quick commit", "commit lite", "fast commit", or wants a streamlined commit without verbose step-by-step explanation.
allowed-tools: "Bash(git:*) Read"
---

# Git Commit Lite

Create a well-structured local commit. NEVER push, NEVER commit secrets, NEVER force-add ignored files, NEVER fabricate changes. ALWAYS ask rather than assume when a change's purpose is ambiguous.

## Workflow

Run `git status --porcelain=v1`, `git diff --stat`, and `git diff --cached --stat`. If clean, stop. Otherwise show staged vs unstaged vs untracked.

If there are unstaged or untracked files, present the staging choice: **(a)** staged only, **(b)** `git add -A` first _(recommended)_, or **(c)** select files. Skip if only staged changes exist. Then run `git diff --cached` for the full diff.

Detect platform via `git remote get-url origin` — github.com / bitbucket.org / gitlab.com / other. Then determine commit message format from the first source with clear guidance:

1. **Contribution docs** — `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `DEVELOPMENT.md`, `HACKING.md`, `docs/CONTRIBUTING.md`, `.github/CONTRIBUTING.md`, `.github/PULL_REQUEST_TEMPLATE.md`, `.github/COMMIT_CONVENTION.md`, `.bitbucket/CONTRIBUTING.md`, `.bitbucket/PULL_REQUEST_TEMPLATE.md`, `COMMIT_MSG_TEMPLATE`, `.gitmessage`. Extract format, types, scopes, required refs.
2. **Tool config** — commitlint, `.czrc`, commitizen in `package.json`, `.releaserc`, `git config --get commit.template`, `bitbucket-pipelines.yml` branch patterns, `jira.config`, `.jira.yml`.
3. **History** — `git log --oneline --no-merges -20`. Match dominant pattern: Conventional Commits, Jira keys (`PROJ-123`), GitHub `#N`, Bitbucket `issue #N`, smart-commits (`PROJ-123 #done`), tense, casing.
4. **Fallback** — Conventional Commits: `type(scope): description` + body + footers. Types: `feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert`. Subject max 72 chars, imperative, lowercase, no period.

Platform footer reference:

| Platform         | Subject prefix             | Footer                                                           |
| ---------------- | -------------------------- | ---------------------------------------------------------------- |
| GitHub           | —                          | `Closes #42`, `Fixes #42`, `Refs #42`                            |
| GitLab           | —                          | `Closes #42`, `Relates to #42`                                   |
| Bitbucket issues | —                          | `Closes #7`, `Fixes #7`, `References #7`                         |
| Bitbucket + Jira | `PROJ-123:` / `[PROJ-123]` | `PROJ-123 #done`, `PROJ-123 #comment text`, `PROJ-123 #time dur` |
| Standalone Jira  | `PROJ-123:` / `[PROJ-123]` | `Resolves PROJ-123`, `See PROJ-123`                              |
| Unknown          | —                          | Include any ticket/issue ID provided                             |

Before drafting, check for a ticket or issue number from these sources in order: (1) branch name via `git branch --show-current` (e.g. `feature/PROJ-123-...`), (2) the user's message, (3) the commit history pattern from step 3. If a ticket number is found, carry it forward. If none is found, ALWAYS ask the user — **(a)** provide a ticket/issue number or **(b)** confirm there is no ticket — before proceeding.

Draft the commit message from the diff + detected format. Determine type, scope, subject, body (if non-trivial), footers. Include the ticket reference in the footer if one was provided; omit it if the user confirmed there is none. If changes are unrelated, suggest splitting. Present the draft in a fenced code block.

CRITICAL: **STOP — do NOT run `git commit` or any other command. End your response and wait for the user's explicit approval before committing.**

Only after the user explicitly approves, commit using a heredoc:

```
git commit -F- <<'COMMIT_MSG'
<final message here>
COMMIT_MSG
```

Show `git log --oneline -1` on success. On failure, show error and help resolve.
