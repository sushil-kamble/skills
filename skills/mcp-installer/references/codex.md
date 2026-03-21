# Codex

Read this file only when installing or updating MCP servers for Codex.

## Intent Guardrails

- Create one install job per explicit `{server, agent, level}` request.
- Only install for Codex when the user explicitly names Codex or clearly includes it in a multi-agent request.
- Do not copy the same server to Claude Code or OpenCode unless the user asked for that too.
- Do not add tool filters, callback overrides, headers, or extra timeouts unless the server docs require them or the user asked for them.

## Scopes

Two scopes; full precedence chain (highest to lowest): CLI flags > profile > project config (closest CWD wins) > user config > system config (`/etc/codex/config.toml`)

| Scope | Storage | Who sees it |
|-------|---------|-------------|
| `user` | `~/.codex/config.toml` | You only, all projects. **Default target for `codex mcp add`.** |
| `project` | `.codex/config.toml` at repo root | All users of the repo; safe to commit. Only active in trusted projects. |

### Scope selection rules

- **Default (no scope specified):** use `~/.codex/config.toml` (user scope).
- If the user says `global` or `user`, edit `~/.codex/config.toml`.
- If the user says `project`, edit `.codex/config.toml` at the current repo root.
- **⚠️ No gitignored per-project scope.** Codex has no equivalent to Claude Code's `local` scope (a private config that is per-project but not committed). If the user asks for a gitignored per-project install, warn them: _"Codex does not support a private per-project scope. The closest option is user scope (`~/.codex/config.toml`), which is private but available across all projects. Alternatively, project scope (`.codex/config.toml`) is per-project but committed to git. Which do you prefer?"_

## Transport Choice

- Supported transports: local `stdio` and streamable `http`.
- Prefer the transport shown in the server's official install docs.
- For remote servers, use bearer token or OAuth fields only when the official docs say to.

## Preferred Install Method

- Use `codex mcp add` for simple stdio servers.
- Edit `config.toml` directly for remote servers or whenever you need fields beyond the basic CLI add flow.
- Codex CLI and IDE share the same MCP config, so a single config change covers both clients.

Simple stdio add:

```bash
codex mcp add <server-name> --env KEY=value -- <command> [args...]
```

## Config Shape

Stdio server:

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
```

Remote server:

```toml
[mcp_servers.figma]
url = "https://mcp.figma.com/mcp"
bearer_token_env_var = "FIGMA_OAUTH_TOKEN"
http_headers = { "X-Figma-Region" = "us-east-1" }
```

Remote server with env-backed header:

```toml
[mcp_servers.stitch]
url = "https://stitch.googleapis.com/mcp"
env_http_headers = { "X-Goog-Api-Key" = "STITCH_API_KEY" }
```

## Supported Fields

- Stdio: `command`, `args`, `env`, `env_vars`, `cwd`
- Remote: `url`, `bearer_token_env_var`, `http_headers`, `env_http_headers`
- Shared: `startup_timeout_sec`, `tool_timeout_sec`, `enabled`, `required`, `enabled_tools`, `disabled_tools`

Prefer `env_http_headers` over literal `http_headers` when the header value is a secret that already exists in the environment.

## OAuth Notes

- For OAuth-capable HTTP servers, install the config first and then tell the user to run `codex mcp login <server-name>` if auth is still required.
- Add top-level `mcp_oauth_callback_port` or `mcp_oauth_callback_url` only when the server docs explicitly require a fixed callback.

## Verification

- Re-read the target `config.toml`.
- Use `codex mcp --help` if you need to confirm available MCP subcommands.
- In the Codex TUI, `/mcp` shows active MCP servers.
