# Claude Code

Read this file only when installing or updating MCP servers for Claude Code.

## Intent Guardrails

- Create one install job per explicit `{server, agent, level}` request.
- Only touch Claude Code when the user explicitly asked for Claude Code or clearly included it in a multi-agent request.
- Keep each agent's scope independent. If the user wants project scope for Codex and user scope for Claude Code, do exactly that.
- Do not switch to plugin MCP, managed MCP, channels, or tool restrictions unless the user explicitly asked for those features.

## Scopes

Three scopes, highest to lowest precedence: **local > project > user**

| Scope | Flag | Storage | Who sees it |
|-------|------|---------|-------------|
| `local` | `--scope local` (or omit `--scope`) | `~/.claude.json` under the project path | You only, current project only. **Default when no `--scope` is passed.** Previously called "project" in older Claude Code versions. |
| `project` | `--scope project` | `.mcp.json` at repo root | Everyone on the team; committed to git. Claude Code always asks for confirmation before using project-scoped servers. |
| `user` | `--scope user` | `~/.claude.json` (top-level) | You only, all projects on this machine. Previously called "global" in older Claude Code versions. |

### Scope selection rules

- **Default (no scope specified):** use `local` — it is private, per-project, and never touches shared files.
- If the user says `global` or `user`, use `--scope user`.
- If the user says `project` or `shared`, use `--scope project` and confirm the `.mcp.json` will be committed.
- If the user says `local`, use `--scope local` (or omit `--scope`).
- Do not silently downgrade `project` to `local`; they are distinct scopes with different storage and visibility.

## Transport Choice

- Prefer `http` for remote servers.
- Use `sse` only when the official docs provide SSE and no HTTP option.
- Use `stdio` for local command-based servers.

## Preferred Install Method

Prefer the CLI because it applies the right schema and scope automatically.

HTTP:

```bash
claude mcp add --transport http [--scope <scope>] <name> <url>
```

SSE:

```bash
claude mcp add --transport sse [--scope <scope>] <name> <url>
```

Stdio:

```bash
claude mcp add --transport stdio [--scope <scope>] [--env KEY=value] <name> -- <command> [args...]
```

## Important Option Rule

- Put all Claude flags before the server name.
- Use `--` only once, to separate Claude flags from the stdio server command.
- Do not let server flags leak before `--`; keep the boundary exact.

## When to Use `add-json`

Use `claude mcp add-json` when:
- the official docs already provide JSON config
- you need an `oauth` object
- you need `authServerMetadataUrl`

For project-scoped HTTP servers with secret headers, prefer editing `.mcp.json` to use env var expansion instead of passing a literal secret via `--header`.

Project-scoped file shape:

```json
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

Project-scoped HTTP server with env-backed header:

```json
{
  "mcpServers": {
    "stitch": {
      "type": "http",
      "url": "https://stitch.googleapis.com/mcp",
      "headers": {
        "X-Goog-Api-Key": "${STITCH_API_KEY}"
      }
    }
  }
}
```

## Environment Variable Expansion

- `.mcp.json` supports `${ENV_VAR}` expansion, including inside HTTP `headers`.
- Prefer `${ENV_VAR}` in shared project config when the secret already exists as an env var and the user wants project scope without committing the literal secret.
- `local` scope is still the private per-project option when the user does not want the shared project config to reference the secret at all.

## OAuth Notes

- Prefer HTTP over SSE when both are available.
- If the server needs OAuth, install it first and then tell the user to finish auth through `/mcp`.
- If the docs require pre-registered credentials, use `--client-id`, `--client-secret`, and `--callback-port` exactly as documented by that server.
- If the user only asked to install the server, do not start adding optional OAuth metadata overrides or channels config.

## Verification

- `claude mcp get <name>`
- `claude mcp list`
- Inside Claude Code, `/mcp` is the follow-up step for auth and status checks.

## Platform Note

- On native Windows, stdio servers launched with `npx` need `cmd /c npx ...`.
