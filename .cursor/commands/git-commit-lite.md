# Git Commit Lite

Create a well-structured local commit. Constraints: never push, never commit secrets, never force-add ignored files, never fabricate changes, ask rather than assume when a change's purpose is ambiguous.

Run `git status --porcelain=v1`, `git diff --stat`, and `git diff --cached --stat`. If clean, stop. Otherwise show staged vs unstaged vs untracked. If there are unstaged or untracked files, present the staging choice using the Decision Format below: **(a)** commit staged only, **(b)** `git add -A` first _(recommended)_, or **(c)** select files. Skip if only staged changes exist. Then run `git diff --cached` for the full diff.

Detect platform via `git remote get-url origin` → github.com / bitbucket.org / gitlab.com / other. Then determine commit message format from the first source with clear guidance:

1. **Contribution docs** — `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `DEVELOPMENT.md`, `HACKING.md`, `docs/CONTRIBUTING.md`, `.github/CONTRIBUTING.md`, `.github/PULL_REQUEST_TEMPLATE.md`, `.github/COMMIT_CONVENTION.md`, `.bitbucket/CONTRIBUTING.md`, `.bitbucket/PULL_REQUEST_TEMPLATE.md`, `COMMIT_MSG_TEMPLATE`, `.gitmessage`. Extract format, types, scopes, required refs.
2. **Tool config** — commitlint, `.czrc`, commitizen in `package.json`, `.releaserc`, `git config --get commit.template`, `bitbucket-pipelines.yml` branch patterns, `jira.config`, `.jira.yml`, Jira integration references in `bitbucket-pipelines.yml`.
3. **History** — `git log --oneline --no-merges -20`. Match dominant pattern: Conventional Commits, Jira keys (`PROJ-123`), GitHub `#N`, Bitbucket `issue #N`, smart-commits (`PROJ-123 #done`), tense, casing.
4. **Fallback** — Conventional Commits: `type(scope): description` + body + footers. Types: `feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert`. Subject ≤72 chars, imperative, lowercase, no period.

Platform footer reference:
| Platform | Subject prefix | Footer |
|-|-|-|
| GitHub | — | `Closes #42`, `Fixes #42`, `Refs #42` |
| GitLab | — | `Closes #42`, `Relates to #42` |
| Bitbucket issues | — | `Closes #7`, `Fixes #7`, `References #7` |
| Bitbucket + Jira | `PROJ-123:` / `[PROJ-123]` | `PROJ-123 #done`, `PROJ-123 #comment <text>`, `PROJ-123 #time <dur>` |
| Standalone Jira | `PROJ-123:` / `[PROJ-123]` | `Resolves PROJ-123`, `See PROJ-123` |
| Unknown | — | include any ticket/issue ID provided |

Before drafting, check for a ticket or issue number from these sources in order: (1) branch name via `git branch --show-current` (e.g. `feature/PROJ-123-...`), (2) the user's message, (3) the commit history pattern from step 3. If a ticket number is found, carry it forward. If none is found, **always ask the user** using the Decision Format below — **(a)** provide a ticket/issue number or **(b)** confirm there is no ticket — before proceeding.

Draft the commit message from the diff + detected format. Determine type, scope, subject, body (if non-trivial), footers. Include the ticket reference in the footer if one was provided; omit it if the user confirmed there is none. If changes are unrelated, suggest splitting. Present the draft in a fenced code block. **STOP — do NOT run `git commit` or any other tool. End your response and wait for the user's explicit approval before committing.**

Only after the user explicitly approves in their reply, commit via temp file:

```
echo '<msg>' > /tmp/cursor_commit_msg.txt
git commit -F /tmp/cursor_commit_msg.txt
rm /tmp/cursor_commit_msg.txt
```

Show `git log --oneline -1` on success. On failure, show error and help resolve.

## Decision Format

Whenever presenting the user with a choice, always use this exact structure:

**[Question in a single sentence]**

- **a)** [Short label] — [One-sentence description]
- **b)** [Short label] — [One-sentence description]
- _(additional options as needed)_
- **or** tell me what you'd like to do instead.

Reply with **a**, **b**, _(etc.)_, or describe your preference.

Rules: lowercase single-letter keys (a, b, c, ...); labels 2–4 words; cap at 5 options; if one option is the obvious default, list it first (as option a) and mark it with "(recommended)"; always end with the open-ended fallback line; always include the "Reply with..." footer. Accept the letter in any casing and with or without trailing punctuation. If the user responds with freeform text, treat it as a valid instruction and continue — do not re-prompt for a letter.
