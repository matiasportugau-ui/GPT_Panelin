# Claude Code settings

> Comprehensive configuration through a scope system for Claude Code.

## Configuration Scopes

| Scope | Location | Who it affects | Shared with team |
|-------|----------|----------------|------------------|
| **Managed** | System-level `managed-settings.json` | All users on machine | Yes (deployed by IT) |
| **User** | `~/.claude/` directory | You, across all projects | No |
| **Project** | `.claude/` in repository | All collaborators | Yes (committed to git) |
| **Local** | `.claude/*.local.*` files | You, in this repository only | No (gitignored) |

### Scope Precedence (High to Low)

1. Managed (highest - cannot be overridden)
2. Command line arguments
3. Local project settings
4. Shared project settings
5. User settings (lowest)

## Settings Files

### Location Structure

* **User**: `~/.claude/settings.json`
* **Project**: `.claude/settings.json`
* **Local**: `.claude/settings.local.json`
* **Managed** (system-wide):
  * macOS: `/Library/Application Support/ClaudeCode/`
  * Linux/WSL: `/etc/claude-code/`
  * Windows: `C:\Program Files\ClaudeCode\`

### Example settings.json

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp"
  }
}
```

## Available Settings

| Key | Description | Example |
|-----|-------------|---------|
| `apiKeyHelper` | Custom script to generate auth value | `/bin/generate_temp_api_key.sh` |
| `cleanupPeriodDays` | Sessions inactive period before deletion (default: 30) | `20` |
| `companyAnnouncements` | Announcements displayed at startup | `["Welcome to Acme Corp!"]` |
| `env` | Environment variables for every session | `{"FOO": "bar"}` |
| `attribution` | Customize git commit/PR attribution | `{"commit": "Generated", "pr": ""}` |
| `permissions` | Allow/deny/ask tool permissions | See Permission Settings |
| `hooks` | Custom commands before/after tool execution | See [Hooks](hooks.md) |
| `disableAllHooks` | Disable all hooks | `true` |
| `allowManagedHooksOnly` | (Managed only) Prevent user/project hooks | `true` |
| `model` | Override default model | `"claude-sonnet-4-5-20250929"` |
| `statusLine` | Custom status line context | `{"type": "command", "command": "..."}` |
| `respectGitignore` | Respect `.gitignore` in file picker (default: true) | `false` |
| `outputStyle` | Adjust system prompt output style | `"Explanatory"` |
| `forceLoginMethod` | Restrict login to `claudeai` or `console` | `claudeai` |
| `enableAllProjectMcpServers` | Auto-approve all MCP servers in `.mcp.json` | `true` |
| `alwaysThinkingEnabled` | Enable extended thinking by default | `true` |
| `language` | Preferred response language | `"japanese"` |
| `autoUpdatesChannel` | Release channel: `"stable"` or `"latest"` | `"stable"` |

## Permission Settings

### Structure

```json
{
  "permissions": {
    "allow": ["Bash(npm run *)", "Read(~/.config)"],
    "ask": ["Bash(git push *)"],
    "deny": ["WebFetch", "Bash(curl *)", "Read(./.env)"],
    "additionalDirectories": ["../docs/"],
    "defaultMode": "acceptEdits",
    "disableBypassPermissionsMode": "disable"
  }
}
```

### Permission Rule Syntax

**Matching all uses:**

* `Bash` or `Bash(*)` - matches all bash commands
* `WebFetch` - matches all web fetch requests
* `Read` - matches all file reads

**Wildcard patterns:**

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git commit *)",
      "Bash(git * main)",
      "Bash(* --version)"
    ],
    "deny": [
      "Bash(git push *)"
    ]
  }
}
```

### Rule Evaluation Order

1. **Deny** rules (checked first)
2. **Ask** rules
3. **Allow** rules (checked last)

First matching rule determines behavior. Deny always takes precedence.

## Hook Configuration

See [Hooks reference](hooks.md) for detailed hook configuration.

## Environment Variables

Key environment variables:

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | API key for Claude SDK |
| `ANTHROPIC_AUTH_TOKEN` | Custom Authorization header value |
| `ANTHROPIC_MODEL` | Override default model |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | Enable OpenTelemetry (set to `1`) |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max output tokens (default: 32000, max: 64000) |
| `CLAUDE_CODE_SHELL` | Override shell detection |
| `DISABLE_TELEMETRY` | Opt out of Statsig telemetry |
| `DISABLE_AUTOUPDATER` | Disable automatic updates |
| `DISABLE_ERROR_REPORTING` | Opt out of Sentry error reporting |
| `MAX_THINKING_TOKENS` | Override extended thinking budget |

## Related resources

* [Claude Code on the web](claude-code-on-the-web.md)
* [Hooks reference](hooks.md)
* [Security](security.md)
* [Data usage](data-usage.md)
