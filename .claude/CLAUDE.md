# Personal Claude Code setup

Cross-project notes that apply in every repo. A specific project's `CLAUDE.md` or memory overrides these on conflict.

## Tooling installed (2026-06-15)

- **Context7 MCP** (user-global). For any question about a library, framework, SDK, or API, call `mcp__context7__resolve-library-id` then `mcp__context7__query-docs` for current docs — prefer this over training knowledge or web search. Send only library names + topic strings; never proprietary code or PHI (queries go to Upstash).
- **Prettier-on-edit hook** (`~/.claude/settings.json`, PostToolUse). Edited files are auto-formatted with Prettier **only when the file's project has its own Prettier config**. No need to re-run Prettier manually on files just edited in a configured project.
- **Superpowers** (user-global plugin, v5.x). Use its `brainstorming` / `writing-plans` skills for substantial multi-file features or refactors; skip the full workflow for trivial edits. Its red/green/refactor TDD core only pays off in repos with a real test suite — elsewhere, lean on the planning/brainstorming skills.

## How I want you to work

### Workflow
- **Ask clarifying questions before assuming.** Surface ambiguous requirements and edge cases before implementing, not after.
- **Commits:** single-line conventional-commit subject; never add a `Co-Authored-By` or Claude attribution trailer.

### Code quality & style
- **JSDoc on every function** (exported and local): document params, returns, and edge cases. (The "no WHAT / don't restate the callsite" rule applies only to inline `//` comments, not JSDoc.)
- **Named exports only** — `export { foo } from '...'`, never `export *`; prefer named consts over default exports.
- **Arrow functions for new utilities** (`export const foo = () => {}`). Exception: saga generators stay `function*`.
- **Use shared/enum constants instead of string literals.**
- **Search for existing code before creating new**; keep helpers feature-local by default, lift to a shared location only when actually reused.
- **Don't suppress linter rules without justification** — if you must, inline-disable the specific rule with a reason comment.

### React / UI
- **PropTypes on every new component** (not linter-enforced); use specific validators, never `PropTypes.any`.
- Name the `useDispatch()` result `dispatch` (drop aliases like `reduxDispatch` unless there's a real collision).
- **Accessibility:** handle keyboard navigation, focus management, and ARIA for interactive components — beyond what static analysis catches.
