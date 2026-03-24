---
name: review-pr
description: Review a pull request and draft efficient, findings-first feedback before taking any review action. Use when the user provides a PR URL, says "review PR", "review this pull request", "check this PR", or shares a github.com pull request link or bitbucket.org pull-requests link.
allowed-tools: "Bash(gh:*) Bash(git:*) Read WebFetch"
---

# Review PR

Review exactly one pull request and draft efficient, high-signal feedback before taking any review action.

CRITICAL: NEVER modify either branch in the PR. NEVER merge, decline, or close. NEVER apply code changes, create commits, or push. NEVER create worktrees. NEVER fabricate findings. NEVER call any external tool when the PR input is missing, ambiguous, or malformed.

## Step 1 — Validate Input

Before calling any external tool, extract the PR reference from the user's request and validate it locally.

Accepted input forms:

- Full GitHub PR URL: `https://github.com/OWNER/REPO/pull/NUMBER`
- Full Bitbucket Cloud PR URL: `https://bitbucket.org/WORKSPACE/REPO/pull-requests/NUMBER`
- Explicit identity: `github OWNER/REPO#NUMBER` or `bitbucket WORKSPACE/REPO#NUMBER`
- Repo-qualified identity without provider: `OWNER/REPO#NUMBER` only if the provider can be inferred unambiguously from `git remote get-url origin`

Validation rules:

- If there is **no PR URL or parseable PR identity**, stop immediately and report that the review cannot start because no PR input was provided.
- If there are **multiple PR references**, ask the user which single PR to review and stop.
- If the input does not parse into exactly one supported tuple of provider + owner/workspace + repository + PR number, stop and report that the PR input is invalid.
- Treat malformed URLs, unsupported hosts, missing repo names, and non-numeric PR numbers as invalid.
- If the user provides only `#123`, `PR 123`, or another ambiguous shorthand, treat it as incomplete unless the repo and provider can be inferred safely from `git remote get-url origin`.

ALWAYS validate before proceeding. Only continue after you have one validated PR target.

## Step 2 — Resolve the Provider and Review Backend

Determine the review backend based on the validated provider.

### GitHub (preferred path)

If the PR is on GitHub, use the `gh` CLI as the primary backend. Verify `gh` is available by running `gh --version`. If not available, stop and explain that the `gh` CLI is required for GitHub PR reviews.

### Non-GitHub (MCP fallback)

If the PR is on Bitbucket, GitLab, or another provider:

1. Check if an MCP server is available that supports the provider (e.g., GitKraken, a provider-native server).
2. If an MCP server is available and exposes the required review tools, use it. Read the tool schema/descriptor before every MCP tool invocation.
3. If the MCP server exposes `mcp_auth`, invoke that first before any other calls.
4. If no suitable MCP server is available, stop and explain what setup is needed.

IMPORTANT: Do NOT use any tool that creates a worktree, checks out code, or otherwise changes local or remote branches. This skill is review-only.

## Step 3 — Gather Pull Request Context

### GitHub via `gh` CLI

```
gh pr view <URL> --json title,body,baseRefName,headRefName,author,state,reviewRequests,labels,number
gh pr diff <URL>
gh api repos/{owner}/{repo}/pulls/{number}/reviews
gh api repos/{owner}/{repo}/pulls/{number}/comments
```

### Non-GitHub via MCP

Use the selected MCP backend to retrieve equivalent context: PR metadata, diffs, and existing review threads.

### Required context

1. **PR metadata**: title, description, source branch, target branch, author, current state, reviewers, and linked tickets if available.
2. **Changed files and diffs** for the PR, or the best available file-level change summary.
3. **Existing PR comments and review threads** to avoid duplicate feedback and understand prior discussion.

If the backend cannot return enough change context to perform a credible review, stop and explain the limitation instead of guessing.

## Step 4 — Gather Contribution and Review Scaffolding

Review the target project's guidance before drafting feedback.

### GitHub via `gh` CLI

Attempt to fetch these files from the PR's base branch using:

```
gh api repos/{owner}/{repo}/contents/{path}?ref={base_branch}
```

If the local repo has the target branch checked out or available, use `Read` instead.

### Files to check (in precedence order)

- `CONTRIBUTING.md`, `CONTRIBUTING`
- `docs/CONTRIBUTING.md`
- `DEVELOPMENT.md`, `docs/DEVELOPMENT.md`
- `HACKING.md`
- `CODEOWNERS`, `.github/CODEOWNERS`
- `.github/CONTRIBUTING.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/PULL_REQUEST_TEMPLATE/` (if the backend can enumerate templates)
- `.bitbucket/CONTRIBUTING.md`
- `.bitbucket/PULL_REQUEST_TEMPLATE.md`
- `README.md`

Extract guidance on: required review focus areas, testing expectations, style or architectural constraints, ownership boundaries from `CODEOWNERS`, PR template sections that imply what reviewers should verify.

Treat these files as scaffolding, not absolute truth. If none are available, continue with standard review best practices.

## Step 5 — Evaluate Review Action Capabilities

Map the available review actions before drafting.

### GitHub via `gh` CLI

`gh pr review` natively supports:

- `--approve` — approve the PR
- `--request-changes` — request changes with a review body
- `--comment` — leave a comment-only review
- `--body "text"` — the review body text

All three review actions are fully supported. Inline file-level comments are not directly supported via `gh pr review` but can be posted via `gh api` if needed.

### Non-GitHub via MCP

Capability-gate actions based on what the MCP schema explicitly supports. NEVER assume that `approve: false` means `request changes`. Only use `request changes` when the MCP schema explicitly supports it.

## Step 6 — Review the Pull Request

Review the changes with a findings-first mindset. Focus on the highest-value feedback first:

1. `[CORRECTNESS]` — Correctness and behavioral regressions
2. `[SECURITY]` — Security and privacy risks
3. `[PERFORMANCE]` / `[RELIABILITY]` — Data loss, performance, and reliability risks
4. `[TESTS]` — Missing or weak tests for risky changes
5. `[DESIGN]` / `[MAINTAINABILITY]` — Design and maintainability issues that materially affect future work
6. `[DOCS]` — Missing or incorrect documentation for meaningful changes
7. `[STYLE]` — Style or naming issues only when they violate project guidance or materially reduce clarity

Review heuristics:

- Prefer concise, actionable feedback over long essays.
- Separate **blocking findings** from **non-blocking suggestions**.
- Use `request changes` only for genuine must-fix problems.
- Avoid repeating feedback that already exists on the PR.
- When a suggestion is optional, label it clearly as non-blocking.
- If no substantive issues are found, explicitly say so and mention any residual testing gaps or assumptions.
- Call out a notable strength when it adds signal, but keep praise concise.

## Step 7 — Draft the Review and Stop for Approval

Draft the review before taking any write action. Consult `references/review-format.md` for the standard output template.

Choose the proposed review outcome:

- `approve` — no blocking issues and the backend supports approval
- `request changes` — blocking issues exist and the backend explicitly supports it
- `comment` — non-blocking feedback or whenever a stronger action is unsupported

Present the full draft in Markdown using the format from `references/review-format.md`.

CRITICAL: **STOP here. Do NOT submit the review or call any write tool. End your response and wait for the user's explicit approval or edits.**

## Step 8 — Submit Only After Explicit User Approval

CRITICAL: Only proceed when the user explicitly approves the drafted review or asks for specific edits and then approves the revised draft.

### GitHub submission via `gh` CLI

```
gh pr review <URL> --approve --body "review text"
gh pr review <URL> --request-changes --body "review text"
gh pr review <URL> --comment --body "review text"
```

### Non-GitHub submission via MCP

Use the appropriate MCP tool as determined in Step 5.

### Submission rules

- If the user does not explicitly approve, do not submit anything.
- If the user asks for an unsupported action, explain the limitation and do not submit a substitute unless the user approves the revised plan.
- Submit only the single approved review for the single validated PR.
- NEVER submit a review to a different PR, repository, or provider than the validated target.

After submission:

- Confirm exactly what action was sent.
- Summarize any drafted but unsubmitted feedback that the backend could not post.
- Do not take any further action on the PR.

## Rules

- **One PR at a time.** NEVER review more than one PR in a single run.
- **No external calls on bad input.** Missing, ambiguous, multi-PR, or malformed input must be handled before any tool call.
- **NEVER mutate branches.** No commits, pushes, rebases, checkouts, merges, or worktree creation.
- **NEVER merge or decline.** This skill may only comment, approve, or request changes when explicitly supported.
- **Ask, do not assume.** If the diff or project guidance leaves intent unclear, ask the user a targeted clarification question before drafting the review.
- **Do not fabricate context.** Only reference files, diffs, comments, and capabilities you actually retrieved.
- **Respect project guidance.** If contribution docs or review templates exist, use them to shape what you focus on.
- **Stay concise and useful.** Prioritize the smallest set of comments that most improves the PR.
