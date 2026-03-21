# OpenCode

Read this file only when installing or updating MCP servers for OpenCode.

## Intent Guardrails

- Create one install job per explicit `{server, agent, level}` request.
- Only install for OpenCode when the user explicitly names OpenCode or clearly includes it in a multi-agent request.
- Do not add tool enablement or deny rules unless the user explicitly asked for them.
- Keep the requested scope separate from other agents in the same prompt.

## Scopes

Three tiers, lowest to highest precedence (configs are **merged**, not replaced — a higher-precedence config only wins on conflicting keys):

| Scope | Storage | Who sees it |
|-------|---------|-------------|
| `remote` | `.well-known/opencode` on the org server | Everyone in the org; read-only managed defaults. |
| `global` | `~/.config/opencode/opencode.json` | You only, all projects. **Default when no scope is specified.** |
| `project` | `opencode.json` at the project root | All users of the repo; safe to commit. Highest precedence. |

### Scope selection rules

- **Default (no scope specified):** use `~/.config/opencode/opencode.json` (global/user scope).
- If the user says `global`, `user`, or gives no scope, edit `~/.config/opencode/opencode.json`.
- If the user says `project`, edit `opencode.json` at the current repo root.
- **⚠️ No gitignored per-project scope.** OpenCode has no equivalent to Claude Code's `local` scope. If the user asks for a gitignored per-project install, warn them: _"OpenCode does not support a private per-project scope. The closest option is global scope (`~/.config/opencode/opencode.json`), which is private but available across all projects. Alternatively, project scope (`opencode.json`) is per-project but committed to git. Which do you prefer?"_

## File Rules

- OpenCode supports both JSON and JSONC. Preserve the file format that already exists.
- Configs are **merged**: adding a server at project scope does not remove servers defined in global scope.
- Edit only the `mcp` object unless the user explicitly asks for tool enablement rules.
- Use the current directory or nearest Git root for project-scoped installs.

## Config Shape

Local stdio server:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "my-local-mcp": {
      "type": "local",
      "command": ["npx", "-y", "my-mcp-command"],
      "enabled": true,
      "environment": {
        "MY_ENV_VAR": "value"
      }
    }
  }
}
```

Remote server:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "context7": {
      "type": "remote",
      "url": "https://mcp.context7.com/mcp"
    }
  }
}
```

Remote server with env-based auth:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "my-remote-mcp": {
      "type": "remote",
      "url": "https://mcp.example.com/mcp",
      "headers": {
        "Authorization": "Bearer {env:MY_API_KEY}"
      },
      "oauth": false
    }
  }
}
```

For remote headers that carry secrets, prefer `{env:VAR_NAME}` placeholders over literal values in project config.

## OAuth Notes

- Remote servers can authenticate automatically when OAuth is supported.
- If needed, trigger auth with `opencode mcp auth <name>`.
- Use `{env:VAR_NAME}` placeholders for secrets in config.

## Verification

- `opencode mcp list`
- `opencode mcp debug <name>` for remote auth or connectivity issues
- `opencode mcp logout <name>` removes stored credentials
