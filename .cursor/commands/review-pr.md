# Review PR

Review exactly one pull request and draft efficient, high-signal feedback before taking any review action. Constraints: never modify either branch in the PR, never merge, never decline, never close, never apply code changes, never create commits, never push, never create worktrees, never fabricate findings, and never call an MCP server when the PR input is missing, ambiguous, or malformed.

## Step 1 - Validate Input Before Any MCP Calls

Before inspecting or calling any MCP server, extract PR references from the user's request and validate them locally.

Accepted input forms:

- Full GitHub PR URL: `https://github.com/<owner>/<repo>/pull/<number>`
- Full Bitbucket Cloud PR URL: `https://bitbucket.org/<workspace>/<repo>/pull-requests/<number>`
- Explicit identity: `github <owner>/<repo>#<number>` or `bitbucket <workspace>/<repo>#<number>`
- Repo-qualified identity without provider: `<owner>/<repo>#<number>` only if the provider can be inferred unambiguously from the current repo remote via `git remote get-url origin`

Validation rules:

- If there is **no PR URL or parseable PR identity**, stop immediately and report that the review cannot start because no PR input was provided.
- If there are **multiple PR references**, do not call any MCP server. Ask the user which single PR to review and stop.
- If the input does not parse into exactly one supported tuple of `provider + organization/workspace + repository + PR number`, stop immediately and report that the PR input is invalid.
- Treat obvious malformed URLs, unsupported hosts, missing repo names, and non-numeric PR numbers as invalid input.
- If the user provides only `#123`, `PR 123`, or another ambiguous shorthand, treat it as incomplete input unless the repo and provider can be inferred safely and unambiguously without MCP calls.

Only continue after you have one validated PR target.

## Step 2 - Resolve the Provider and Review Backend

After input validation succeeds, determine which MCP backend to use.

1. Inspect the available MCP server descriptors and tool schemas before calling any MCP tool.
2. Prefer a provider-native MCP server for the resolved provider if one is installed and exposes the required review tools.
3. If no provider-native server is available, fall back to the GitKraken MCP server.
4. If the selected server exposes `mcp_auth`, invoke that first before any other provider tool calls.
5. Before every MCP tool invocation, read the corresponding tool schema/descriptor first.

Required read capabilities:

- PR detail retrieval
- PR comment retrieval
- Repository file content retrieval for contribution guidance

Required write capabilities vary by action:

- Comment review
- Approval review
- Request-changes review
- Optional inline comments or code suggestions

If the selected MCP server cannot support the minimum read workflow, stop and explain what setup is missing.

Important constraint:

- Do **not** use any MCP tool that creates a worktree, checks out code, or otherwise changes local or remote branches. In particular, do **not** use review flows that create a dedicated worktree or mutate Git state. This command is review-only.

## Step 3 - Gather Pull Request Context

Use the selected MCP backend to retrieve the review context for the single validated PR:

1. PR metadata: title, description, source branch, target branch, author, current state, reviewers, and linked tickets if available.
2. Changed files and diffs for the PR, or the best available file-level change summary if full diffs are not available.
3. Existing PR comments and review threads so you can avoid duplicate feedback and understand prior discussion.

If the MCP cannot return enough change context to perform a credible review, stop and explain the limitation instead of guessing.

## Step 4 - Gather Contribution and Review Scaffolding

Review the target project's guidance before drafting feedback. Use the PR's target branch or base branch as the preferred ref when fetching repository files.

Attempt to retrieve all of the following files; use every one that is available. When guidance conflicts across files, the order below indicates precedence:

- `CONTRIBUTING.md`, `CONTRIBUTING`
- `docs/CONTRIBUTING.md`
- `DEVELOPMENT.md`, `docs/DEVELOPMENT.md`
- `HACKING.md`
- `CODEOWNERS`, `.github/CODEOWNERS`
- `.github/CONTRIBUTING.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/PULL_REQUEST_TEMPLATE/` if the backend can enumerate and fetch templates
- `.bitbucket/CONTRIBUTING.md`
- `.bitbucket/PULL_REQUEST_TEMPLATE.md`
- `README.md`

Extract any guidance on:

- Required review focus areas
- Testing expectations
- Style or architectural constraints that matter during review
- Ownership boundaries from `CODEOWNERS`
- PR template sections that imply what reviewers should verify

Treat these files as scaffolding, not absolute truth. If none are available, continue with standard review best practices.

## Step 5 - Evaluate MCP Action Capabilities

Map the available review actions before drafting the final recommendation.

- If the active MCP explicitly supports **approve**, you may use approval after user confirmation.
- If the active MCP explicitly supports **request changes**, you may use that action after user confirmation when blocking issues exist.
- If the active MCP explicitly supports **inline comments** or **code suggestions**, you may prepare line-specific feedback for submission after user confirmation.
- If the active MCP only supports a summary review comment, keep line comments and code suggestions as draft-only text for the user.
- Never assume that `approve: false` means `request changes`. Only use `request changes` when the MCP schema or provider documentation explicitly supports it.

Current GitKraken constraint in this environment:

- `pull_request_create_review` exposes review text plus an approval flag, but does not expose explicit fields for line comments, code suggestions, or request-changes reviews.
- Therefore, when using GitKraken, capability-gate those richer actions instead of pretending they are available.

## Step 6 - Review the Pull Request

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
- Avoid repeating feedback that already exists on the PR unless you are adding new evidence or a clearer explanation.
- When a suggestion is optional, label it clearly as non-blocking.
- If no substantive issues are found, explicitly say that no blocking findings were identified and mention any residual testing gaps or assumptions.
- Call out a notable strength or thoughtful change when it adds signal, but keep praise concise.

## Step 7 - Draft the Review and Stop for Approval

Draft the review before taking any write action.

Choose the proposed review outcome:

- `approve` only when there are no blocking issues and the active MCP supports approval
- `request changes` only when there are blocking issues and the active MCP explicitly supports that action
- `comment` for non-blocking feedback or whenever a stronger action is unsupported

Present the draft using the following standard format. Omit any section that has no content.

---

**Decision badge** — open the draft with exactly one of the following as a top-level heading. Remove the others entirely:

- `## ✅ APPROVE`
- `## ⚠️ REQUEST CHANGES`
- `## 💬 COMMENT`
- `## 📋 DRAFT ONLY`

**Summary** — one sentence immediately after the badge describing what the PR does and the overall assessment.

---

**Sections** — use these headings and emojis exactly. Each finding bullet must be prefixed with a category label in backticks chosen from: `[CORRECTNESS]`, `[SECURITY]`, `[PERFORMANCE]`, `[RELIABILITY]`, `[TESTS]`, `[DESIGN]`, `[MAINTAINABILITY]`, `[DOCS]`, `[STYLE]`. Category labels map to the review priorities in Step 6 in that order.

```
### ⛔️ Blocking — Must fix before merge
- `[CATEGORY]` **Title** — Explanation. File/line reference. Suggested path to resolution.

### 🤔 Suggestions — Non-blocking improvements
- `[CATEGORY]` **Title** — Explanation.

### ⛏️ Nits — Optional polish
- `[CATEGORY]` **Title** — Explanation.

### 💪 Strengths
- Concise note. Only include when it adds meaningful signal.

### 🚧 Residual Risks
- Testing gaps, unverified edge cases, assumptions, rollback considerations.

---

### 📝 Submission Plan
**Action:** `approve` / `request changes` / `comment`
- **Will submit:** description of what MCP action will be taken
- **Cannot submit (MCP limitation):** any items that must remain draft-only
```

---

If the backend supports inline comments or code suggestions, include a short draft list of those items under the relevant section. If not, convert them into plain-text review feedback and state that they will remain draft-only.

Present the full draft in Markdown. **STOP - do not call any MCP write tool yet. End your response here and wait for the user's explicit approval or edits.**

## Step 8 - Submit Only After Explicit User Approval

Only proceed when the user explicitly approves the drafted review or asks for specific edits and then approves the revised draft.

Submission rules:

- If the user does not explicitly approve, do not submit anything.
- If the user asks for an unsupported action, explain the limitation and do not submit a substitute action unless the user approves the revised plan.
- Submit only the single approved review for the single validated PR.
- Never submit a review to a different PR, repository, or provider than the validated target.

Action mapping:

- **Comment review**: submit only if the active MCP clearly supports a non-approving review or review comment action.
- **Approve review**: submit only when approval is supported and the user approves that outcome.
- **Request changes**: submit only when the active MCP explicitly supports it. Otherwise keep it as draft-only text and tell the user that the current backend cannot send a formal request-changes review.
- **Inline comments / code suggestions**: submit only when explicitly supported by the active MCP and approved by the user.

After submission:

- Confirm exactly what action was sent.
- Summarize any drafted but unsubmitted feedback that the MCP could not post.
- Do not take any further action on the PR.

## Rules

- **One PR at a time.** Never review more than one PR in a single run.
- **No MCP calls on bad input.** Missing, ambiguous, multi-PR, or malformed input must be handled before any MCP call.
- **Never mutate branches.** No commits, pushes, rebases, checkouts, merges, suggestions applied to code, or worktree creation.
- **Never merge or decline.** This command may only comment, approve, or request changes when explicitly supported.
- **Ask, do not assume.** If the diff or project guidance leaves intent unclear, ask the user a targeted clarification question before drafting the review.
- **Do not fabricate context.** Only reference files, diffs, comments, and capabilities you actually retrieved.
- **Respect project guidance.** If contribution docs or review templates exist, use them to shape what you focus on.
- **Stay concise and useful.** Prioritize the smallest set of comments that most improves the PR.

## Decision Format

Whenever presenting the user with a choice, always use this exact structure:

**[Question in a single sentence]**

- **a)** [Short label] - [One-sentence description]
- **b)** [Short label] - [One-sentence description]
- _(additional options as needed)_
- **or** tell me what you'd like to do instead.

Reply with **a**, **b**, _(etc.)_, or describe your preference.

Rules: lowercase single-letter keys (a, b, c, ...); labels 2-4 words; cap at 5 options; if one option is the obvious default, list it first (as option a) and mark it with "(recommended)"; always end with the open-ended fallback line; always include the "Reply with..." footer. Accept the letter in any casing and with or without trailing punctuation. If the user responds with freeform text, treat it as a valid instruction and continue - do not re-prompt for a letter.
