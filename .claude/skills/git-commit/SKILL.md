---
name: git-commit
description: Analyze the Git working tree, detect the project's commit message conventions, stage files, and create a well-structured local commit. Use when the user wants to commit changes, write a commit message, stage and commit files, or says "commit", "create a commit", or "write commit message".
allowed-tools: "Bash(git:*) Read"
---

# Git Commit

Analyze the current Git working tree, determine the right commit scope and message format, then create a well-structured commit.

CRITICAL: NEVER push to a remote. NEVER commit secrets, API keys, tokens, or credentials. NEVER force-add ignored files. NEVER fabricate file names or changes — only reference files and diffs that actually exist in command output.

## Step 1 — Gather the Current Git State

Run the following commands and capture their output:

```
git status --porcelain=v1
git diff --stat
git diff --cached --stat
```

- If there are **no changes at all** (nothing staged, nothing unstaged, no untracked files), stop and report: "Nothing to commit — working tree is clean."
- Otherwise, summarize the state for the user:
  - Which files are **staged** (ready to commit).
  - Which files are **unstaged** (modified but not added).
  - Which files are **untracked** (new, not yet tracked by Git).

## Step 2 — Ask the User What to Include

If there are only staged changes and nothing else, skip this step and proceed directly.

Otherwise, present the following options conversationally:

- **(a) Stage all** _(recommended)_ — run `git add -A` to include all modified and untracked files, then commit.
- **(b) Staged only** — commit the currently staged changes and leave everything else alone.
- **(c) Select files** — list unstaged/untracked files and let the user choose which ones to add before committing.
- Or the user can describe a different preference.

Accept the user's choice in any format — a letter, a label, or freeform instructions. Do not re-prompt for a letter if freeform text is provided.

## Step 3 — Stage Files According to the User's Choice

- Staged only: do nothing extra; use the existing index.
- Stage all: run `git add -A`.
- Select files: run `git add <files>` for each file the user selected.

After staging, run `git diff --cached` to capture the **full diff of what will be committed**. This is the primary input for writing the commit message.

## Step 4 — Determine the Commit Message Format

IMPORTANT: Before checking individual sources, consult `references/commit-conventions.md` for the full list of files and configs to inspect.

Investigate multiple sources **in priority order** to discover how this project structures its commit messages. Use the **first source that provides clear guidance**; combine sources if one fills gaps left by another.

Detect the hosting platform by running:

```
git remote get-url origin 2>/dev/null
```

- URLs containing `github.com` → **GitHub**
- URLs containing `bitbucket.org` → **Bitbucket** (also check for Jira integration)
- URLs containing `gitlab.com` → **GitLab**
- Anything else → **Unknown / generic**

### 4a. Check for Contribution Guidelines

Look for contribution and commit convention files as listed in `references/commit-conventions.md`. If any exist, read them and extract required format, types, scopes, prefixes, issue references, sign-off lines, and length limits.

### 4b. Check for Config-Based Conventions

Look for tool configuration that enforces commit formats — commitlint, commitizen, semantic-release, git commit templates, and pipeline configs. See `references/commit-conventions.md` for the full list.

### 4c. Analyze Recent Commit History

Run:

```
git log --oneline --no-merges -20
```

Study the output and identify the **dominant pattern**: Conventional Commits, Jira keys, GitHub issue refs, tense, casing, max length. Adopt the dominant style so the new commit blends in with the project history.

### 4d. Fallback — Conventional Commits

If none of the above sources yield a clear convention, default to **Conventional Commits** as detailed in `references/commit-conventions.md`.

## Step 5 — Confirm Ticket Number

Before drafting the commit message, determine whether a ticket or issue number is already available:

1. **Branch name** — run `git branch --show-current`. Names like `feature/PROJ-123-description` or `fix/42-fix-bug` often embed a ticket key.
2. **User's original message** — the user may have mentioned a ticket or issue number explicitly.
3. **Commit history** — if every recent commit in Step 4c includes a ticket reference, one is likely expected here too.

If a ticket number **is** identifiable from any source above, carry it forward to Step 6.

If no ticket number can be determined, ALWAYS ask the user before proceeding:

- **(a) Yes** — provide the ticket or issue number (e.g., `PROJ-123`, `#42`).
- **(b) No ticket** — there is no associated ticket or issue for this change.

## Step 6 — Draft the Commit Message

Using the staged diff from Step 3, the format from Step 4, and the ticket number from Step 5:

1. **Determine the type** (feat, fix, refactor, etc.) by analyzing what the diff actually does.
2. **Determine the scope** (if the project uses scopes) from the primary area of change.
3. **Write the subject line** — a concise imperative summary. NEVER use vague messages like "update code" or "fix stuff."
4. **Write the body** (if warranted) — explain the motivation, trade-offs, or anything a future reader needs. Skip the body for trivial one-file changes.
5. **Add footers** (if applicable) — issue references, breaking-change notices, co-authors. Match the reference style to the project's platform per `references/commit-conventions.md`.

If the staged changes clearly contain multiple unrelated concerns, suggest splitting them into separate commits and help the user do so.

Present the **complete draft message** to the user inside a fenced code block.

CRITICAL: **STOP here. Do NOT run `git commit` or any other command. End your response and wait for the user's explicit approval before proceeding to Step 7.**

## Step 7 — Commit

CRITICAL: Only execute this step after the user has explicitly approved the draft message (e.g., "yes", "looks good", "LGTM", or they provide edits to incorporate). If the user has not yet replied, do not run any commands.

Once the user approves (or after incorporating their edits), commit using a heredoc to preserve multi-line formatting:

```
git commit -F- <<'COMMIT_MSG'
<final message here>
COMMIT_MSG
```

If the commit succeeds, show the output of `git log --oneline -1` as confirmation.

If the commit fails (e.g., pre-commit hooks reject it), show the error output and help the user resolve it.

## Rules

- NEVER push. This skill only creates local commits.
- NEVER force-add ignored files. Respect `.gitignore` at all times.
- NEVER commit secrets. If the diff contains anything that looks like a key, token, password, or secret, warn the user and do not commit until it is resolved.
- NEVER fabricate file names or changes. Only reference files and diffs that actually exist in command output.
- ALWAYS ask rather than assume. If the purpose of a change is ambiguous from the diff alone, ask the user to clarify before writing the message.
- ALWAYS keep it atomic. If staged changes contain multiple unrelated concerns, suggest splitting into separate commits.
