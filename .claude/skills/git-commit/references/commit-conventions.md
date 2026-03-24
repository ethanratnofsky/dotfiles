# Commit Convention Detection Reference

Detailed lookup tables for Step 4 of the git-commit skill. Consult this file when detecting the project's commit message format.

## Contribution Guideline Files

Check for these files in the repository root and common subdirectories. Check **platform-specific directories** based on the detected remote.

**General (any platform):**

- `CONTRIBUTING.md`, `CONTRIBUTING`
- `docs/CONTRIBUTING.md`
- `CODE_OF_CONDUCT.md` (sometimes contains commit conventions)
- `DEVELOPMENT.md`, `docs/DEVELOPMENT.md`
- `HACKING.md`

**GitHub-specific:**

- `.github/CONTRIBUTING.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/PULL_REQUEST_TEMPLATE/` (directory of multiple templates)
- `.github/COMMIT_CONVENTION.md`

**Bitbucket-specific:**

- `.bitbucket/CONTRIBUTING.md`
- `.bitbucket/PULL_REQUEST_TEMPLATE.md`
- `bitbucket-pipelines.yml` (pipeline stages may hint at required commit prefixes)
- `COMMIT_MSG_TEMPLATE` or `.gitmessage` (local commit template via `git config commit.template`)

**Extract from these files:** required commit message format, allowed types/scopes, required prefixes, issue/ticket references, sign-off lines, trailers, maximum subject line length, body and footer conventions.

## Config-Based Convention Files

- `.commitlintrc`, `.commitlintrc.json`, `.commitlintrc.yml`, `commitlint.config.js`, `commitlint.config.ts` — Conventional Commits with specific rules
- `.czrc`, `.cz.json`, or `config.commitizen` key in `package.json` — Commitizen usage
- `package.json` `scripts` containing `commitlint`, `husky`, or `lint-staged` — enforced commit hooks
- `.releaserc`, `release.config.js` — semantic-release implies Conventional Commits
- `git config --get commit.template` — local or global commit template file
- `bitbucket-pipelines.yml` — branch patterns like `feature/*`, `bugfix/*`, `hotfix/*` suggest matching commit prefixes
- `jira.config`, `.jira.yml`, or Jira integration references in `bitbucket-pipelines.yml` — Jira issue keys expected in commits

## Commit History Pattern Detection

When running `git log --oneline --no-merges -20`, look for:

- `type(scope): description` → Conventional Commits
- `PROJ-123: description` or `[PROJ-123] description` → Jira-style keys (common in Bitbucket + Jira)
- `#42` → GitHub issue reference
- `bitbucket:#7` or `issue #7` → Bitbucket Cloud issue reference
- Jira smart-commit footers: `PROJ-123 #done`, `PROJ-123 #comment ...`, `PROJ-123 #time 2h`
- Imperative mood (`Add feature`) vs past tense (`Added feature`)
- Capitalized vs lowercase subjects
- Consistent max length

## Conventional Commits Fallback

Default format when no project convention is detected:

```
type(optional scope): short imperative description

optional body — explain what and why, not how

optional footer(s) — e.g., Closes #123, Fixes PROJ-456, BREAKING CHANGE: ...
```

**Allowed types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

**Rules:**

- Subject line: imperative mood, lowercase start, no trailing period, max 72 characters
- Body: wrap at 80 characters, separated from subject by a blank line
- Footer: reference issues/tickets using platform-appropriate syntax

## Platform Footer Reference

| Platform / Tracker     | Subject-line prefix         | Footer syntax                                                         |
| ---------------------- | --------------------------- | --------------------------------------------------------------------- |
| GitHub                 | —                           | `Closes #42`, `Fixes #42`, `Refs #42`                                 |
| GitLab                 | —                           | `Closes #42`, `Relates to #42`                                        |
| Bitbucket Cloud issues | —                           | `Closes #7`, `Fixes #7`, `References #7`                              |
| Bitbucket + Jira       | `PROJ-123:` or `[PROJ-123]` | `PROJ-123 #done`, `PROJ-123 #comment text`, `PROJ-123 #time duration` |
| Standalone Jira        | `PROJ-123:` or `[PROJ-123]` | `Resolves PROJ-123`, `See PROJ-123`                                   |
| Unknown                | —                           | Include any ticket/issue ID the user provides                         |

If Bitbucket + Jira integration is detected, prefer the **Jira smart-commit** footer format so that Bitbucket automatically transitions issues, adds comments, and logs work in Jira.
