---
name: mcp-installer
description: Install or update MCP server configuration for Codex, Claude Code, and OpenCode. Use when the user asks to add an MCP server for one or more coding agents at project, local, global, or user scope; when they paste install docs and want the right config applied; or when an MCP server should be added only if it is missing from an agent config.
---

# MCP Installer

Install MCP servers for coding agents with minimal, idempotent config changes.

Typical requests:
- `Install Sanity MCP for Codex on project level.`
- `Install Context7 MCP for Claude Code globally and for OpenCode on project level.`

## Workflow

1. Parse the request into one or more install jobs:
   - `{server, agent, level}`
   - Support multiple agents and different levels in one turn.
2. Resolve the server definition before changing config:
   - Prefer install docs pasted by the user in the current turn.
   - Otherwise use primary sources only: the server's official docs, official package page, official README, or official registry entry.
   - Capture only the fields needed for installation: transport, name, command or URL, args, env vars or headers, and whether OAuth is required.
3. Normalize the requested level:
   - `global` or `user` means user-wide personal config.
   - `project` means repo-scoped config committed to or stored with the current project.
   - `local` means a private per-project config that is **not** committed to git (only Claude Code supports this as `--scope local` / the default scope).
   - **Default (no level specified):** use the personal/user-level scope for all agents. This is the safest default: private to the user and never modifies shared files.
   - **Unsupported scope warning:** if the user requests a gitignored per-project scope (i.e., `local`) for Codex or OpenCode, warn them explicitly that those tools do not support it and ask whether they want user scope (private, all projects) or project scope (committed, current project only) instead. Do not silently substitute one for the other.
4. Read only the relevant agent reference before editing:
   - Codex: `references/codex.md`
   - Claude Code: `references/claude-code.md`
   - OpenCode: `references/opencode.md`
5. Inspect the target config first:
   - If the same server already exists with the same transport and command or URL, do not reinstall it.
   - Only patch missing fields such as env vars, headers, timeout, or `enabled`.
   - If the server needs a secret, also inspect the current project's env conventions first (for example `.env`, `.env.example`, or existing config env var names) before deciding whether to wire an env var or use a literal value.
6. Apply the smallest possible change:
   - Preserve unrelated config and comments.
   - Reuse the existing file format.
   - Prefer the agent's native env-based secret pattern over hardcoded tokens whenever the target config format supports it.
7. Verify after writing:
   - Re-read the target config.
   - Run the documented CLI verification command when available.
   - Report any manual auth step that still must be completed.

## Rules

- Do not guess install commands, URLs, headers, or env var names.
- Prefer HTTP over SSE when official docs offer both.
- Use stdio when the official install method is a local command.
- For OAuth servers, install the config first and then tell the user how to authenticate.
- If a required secret is missing, wire the config to the documented env var name instead of inventing a value.
- For remote HTTP servers with secret headers, prefer agent-supported env wiring over literal header values: Codex `env_http_headers`, Claude Code `${ENV_VAR}` expansion in `.mcp.json`, and OpenCode `{env:VAR}` placeholders.
- Use the current repository root for project-scoped installs.
- Keep server names stable. If docs provide a canonical name, use it. Otherwise slugify once and reuse it across edits and verification.
- For OpenCode, install only the MCP server unless the user also asks to enable or disable its tools per agent.

## Output

Report:
- what was installed or updated
- which config file or CLI command was used
- whether the server already existed
- any follow-up auth or env setup still required
