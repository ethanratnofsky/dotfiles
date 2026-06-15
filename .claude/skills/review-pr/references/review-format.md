# Review Output Format

Standard template for drafting PR reviews in Step 7. Present the full draft in Markdown using this structure. Omit any section that has no content.

## Decision Badge

Open the draft with exactly one of the following as a top-level heading. Remove the others entirely:

- `## APPROVE`
- `## REQUEST CHANGES`
- `## COMMENT`
- `## DRAFT ONLY`

Use `DRAFT ONLY` when the backend cannot submit the review and the output is for the user's reference only.

## Summary

One sentence immediately after the badge describing what the PR does and the overall assessment.

## Sections

Use these headings exactly. Each finding bullet must be prefixed with a category label in backticks chosen from: `[CORRECTNESS]`, `[SECURITY]`, `[PERFORMANCE]`, `[RELIABILITY]`, `[TESTS]`, `[DESIGN]`, `[MAINTAINABILITY]`, `[DOCS]`, `[STYLE]`.

Category labels map to the review priorities (Step 6) in the order listed there.

Anchor findings to a `file:line` reference whenever one applies, so they can be posted as inline comments by backends that support them (see Guidelines).

```
### Blocking — Must fix before merge
- `[CATEGORY]` **Title** — Explanation. File/line reference. Suggested path to resolution.

### Suggestions — Non-blocking improvements
- `[CATEGORY]` **Title** — Explanation.

### Nits — Optional polish
- `[CATEGORY]` **Title** — Explanation.

### Strengths
- Concise note. Only include when it adds meaningful signal.

### Residual Risks
- Testing gaps, unverified edge cases, assumptions, rollback considerations.

---

### Submission Plan
**Action:** `approve` / `request changes` / `comment`
- **Will submit:** description of what action will be taken
- **Inline comments:** file/line-anchored findings that will be posted inline (when the backend supports them)
- **Cannot submit (backend limitation):** any items that must remain draft-only
```

## Guidelines

- Omit any section that has no findings.
- Keep each bullet concise and actionable.
- Inline file-level comments are supported by the available backends — GitHub via `gh api`, and the Bitbucket MCP backend via `create_pull_request_comment` with `file_path` (and optional `line`). Post file/line-anchored findings as inline comments rather than burying them in the review body, and list them under **Inline comments** in the Submission Plan. Only convert a finding to plain-text review feedback (noted as draft-only) when it has no specific file/line anchor or the backend genuinely cannot post it.
- The Submission Plan section ALWAYS appears last and summarizes what will actually be sent to the PR platform versus what remains as local draft text.
