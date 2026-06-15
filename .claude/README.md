# Claude Code config

Portable Claude Code setup. `CLAUDE.md` and `skills/` are **symlinked** into `~/.claude/` (single source of truth); `settings.json` is **filled from a template** per machine because it holds machine-specific secrets.

## Files

- `CLAUDE.md` — global preferences (loaded every session) — symlink into `~/.claude/`
- `skills/` — custom skills: `git-commit`, `git-commit-lite`, `review-pr` — symlink into `~/.claude/skills/`
- `settings.json.template` — model/permissions defaults, the Prettier-on-edit hook, enabled plugins. Copy to `~/.claude/settings.json` and fill in machine-specific bits (see Telemetry). The real `settings.json` is gitignored, so secrets never get committed.

## New-machine setup

```bash
mkdir -p ~/.claude/skills
ln -sfn "$PWD/.claude/CLAUDE.md" ~/.claude/CLAUDE.md
for s in .claude/skills/*/; do ln -sfn "$PWD/$s" ~/.claude/skills/"$(basename "$s")"; done

# settings.json holds machine-specific secrets — copy the template (don't symlink) and fill it in
cp .claude/settings.json.template ~/.claude/settings.json
```

Then:

1. Install the Claude desktop app and **sign in** — account connectors return automatically.
2. Install Superpowers: **+ → Plugins → Add plugin → superpowers** (or `/plugin install superpowers@claude-plugins-official`).
3. Add the portable MCP server: `claude mcp add -s user context7 -- npx -y @upstash/context7-mcp@latest`
4. Machine-specific MCP servers (per machine): a local Bitbucket MCP server, and the Figma desktop app's local server.
5. Prereqs: Node ≥ 22, `jq` (the format hook needs it), per-project Prettier.

## Telemetry (optional)

Telemetry config lives only in your **local** `~/.claude/settings.json` (gitignored) — never in this repo. The desktop app reads it from `settings.json` (not your shell), so it has to go there. To enable it, merge this `env` block into the JSON and fill in your values (copy them from an existing machine's `~/.claude/settings.json` or your onboarding):

```json
"env": {
  "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
  "OTEL_EXPORTER_OTLP_ENDPOINT": "<your-otel-endpoint>",
  "OTEL_EXPORTER_OTLP_PROTOCOL": "http/json",
  "OTEL_LOGS_EXPORTER": "otlp",
  "OTEL_METRICS_EXPORTER": "otlp",
  "OTEL_METRIC_EXPORT_INTERVAL": "60000",
  "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer <your-token>",
  "OTEL_RESOURCE_ATTRIBUTES": "user.email=<you@example.com>"
}
```

The token stays in this local, uncommitted file — never in the repo.

## Not included (by design)

`~/.claude.json` (auth / history / MCP credentials), project memories under `~/.claude/projects/*/memory/`, the filled-in `settings.json` (gitignored), and any real tokens.
