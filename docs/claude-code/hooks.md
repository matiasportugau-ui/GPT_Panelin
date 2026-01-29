# Hooks reference

> This page provides reference documentation for implementing hooks in Claude Code.

For a quickstart guide with examples, see [Get started with Claude Code hooks](https://code.claude.com/docs/en/hooks-guide).

## Hook lifecycle

Hooks fire at specific points during a Claude Code session.

| Hook | When it fires |
|:-----|:--------------|
| `SessionStart` | Session begins or resumes |
| `UserPromptSubmit` | User submits a prompt |
| `PreToolUse` | Before tool execution |
| `PermissionRequest` | When permission dialog appears |
| `PostToolUse` | After tool succeeds |
| `PostToolUseFailure` | After tool fails |
| `SubagentStart` | When spawning a subagent |
| `SubagentStop` | When subagent finishes |
| `Stop` | Claude finishes responding |
| `PreCompact` | Before context compaction |
| `SessionEnd` | Session terminates |
| `Notification` | Claude Code sends notifications |

## Configuration

Claude Code hooks are configured in your [settings files](settings.md):

* `~/.claude/settings.json` - User settings
* `.claude/settings.json` - Project settings
* `.claude/settings.local.json` - Local project settings (not committed)
* Managed policy settings

> **Note:** Enterprise administrators can use `allowManagedHooksOnly` to block user, project, and plugin hooks. See [Hook configuration](settings.md).

### Structure

Hooks are organized by matchers, where each matcher can have multiple hooks:

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ]
  }
}
```

* **matcher**: Pattern to match tool names, case-sensitive (only applicable for `PreToolUse`, `PermissionRequest`, and `PostToolUse`)
  * Simple strings match exactly: `Write` matches only the Write tool
  * Supports regex: `Edit|Write` or `Notebook.*`
  * Use `*` to match all tools. You can also use empty string (`""`) or leave `matcher` blank.
* **hooks**: Array of hooks to execute when the pattern matches
  * `type`: Hook execution type - `"command"` for bash commands or `"prompt"` for LLM-based evaluation
  * `command`: (For `type: "command"`) The bash command to execute (can use `$CLAUDE_PROJECT_DIR` environment variable)
  * `prompt`: (For `type: "prompt"`) The prompt to send to the LLM for evaluation
  * `timeout`: (Optional) How long a hook should run, in seconds, before canceling that specific hook

For events like `UserPromptSubmit`, `Stop`, `SubagentStop`, and `Setup` that don't use matchers, you can omit the matcher field:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/prompt-validator.py"
          }
        ]
      }
    ]
  }
}
```

### Project-Specific Hook Scripts

You can use the environment variable `CLAUDE_PROJECT_DIR` (only available when Claude Code spawns the hook command) to reference scripts stored in your project:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check-style.sh"
          }
        ]
      }
    ]
  }
}
```

## Hook Events

### PreToolUse

Runs after Claude creates tool parameters and before processing the tool call.

**Common matchers:**

* `Task` - Subagent tasks
* `Bash` - Shell commands
* `Glob` - File pattern matching
* `Grep` - Content search
* `Read` - File reading
* `Edit` - File editing
* `Write` - File writing
* `WebFetch`, `WebSearch` - Web operations

### PermissionRequest

Runs when the user is shown a permission dialog. Recognizes the same matcher values as PreToolUse.

### PostToolUse

Runs immediately after a tool completes successfully. Recognizes the same matcher values as PreToolUse.

### Notification

Runs when Claude Code sends notifications. Supports matchers to filter by notification type.

**Common matchers:**

* `permission_prompt` - Permission requests from Claude Code
* `idle_prompt` - When Claude is waiting for user input (after 60+ seconds of idle time)
* `auth_success` - Authentication success notifications
* `elicitation_dialog` - When Claude Code needs input for MCP tool elicitation

### UserPromptSubmit

Runs when the user submits a prompt, before Claude processes it.

### Stop

Runs when the main Claude Code agent has finished responding. Does not run if the stoppage occurred due to a user interrupt.

### SubagentStop

Runs when a Claude Code subagent (Task tool call) has finished responding.

### PreCompact

Runs before Claude Code is about to run a compact operation.

**Matchers:**

* `manual` - Invoked from `/compact`
* `auto` - Invoked from auto-compact (due to full context window)

### Setup

Runs when Claude Code is invoked with repository setup and maintenance flags (`--init`, `--init-only`, or `--maintenance`).

**Matchers:**

* `init` - Invoked from `--init` or `--init-only` flags
* `maintenance` - Invoked from `--maintenance` flag

### SessionStart

Runs when Claude Code starts a new session or resumes an existing session.

**Matchers:**

* `startup` - Invoked from startup
* `resume` - Invoked from `--resume`, `--continue`, or `/resume`
* `clear` - Invoked from `/clear`
* `compact` - Invoked from auto or manual compact

#### Persisting environment variables

SessionStart hooks have access to the `CLAUDE_ENV_FILE` environment variable, which provides a file path where you can persist environment variables for subsequent bash commands.

```bash
#!/bin/bash

if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
  echo 'export API_KEY=your-api-key' >> "$CLAUDE_ENV_FILE"
  echo 'export PATH="$PATH:./node_modules/.bin"' >> "$CLAUDE_ENV_FILE"
fi

exit 0
```

### SessionEnd

Runs when a Claude Code session ends. Useful for cleanup tasks, logging session statistics, or saving session state.

## Hook Input

Hooks receive JSON data via stdin containing session information and event-specific data:

```typescript
{
  // Common fields
  session_id: string
  transcript_path: string  // Path to conversation JSON
  cwd: string              // The current working directory when the hook is invoked
  permission_mode: string  // Current permission mode

  // Event-specific fields
  hook_event_name: string
  ...
}
```

## Hook Output

### Simple: Exit Code

* **Exit code 0**: Success
* **Exit code 2**: Blocking error. `stderr` is used as the error message and fed back to Claude
* **Other exit codes**: Non-blocking error

### Advanced: JSON Output

Hooks can return structured JSON in `stdout` for more sophisticated control. JSON output is only processed when the hook exits with code 0.

## Hook Execution Details

* **Timeout**: 60-second execution limit by default, configurable per command
* **Parallelization**: All matching hooks run in parallel
* **Deduplication**: Multiple identical hook commands are deduplicated automatically
* **Environment**: Runs in current directory with Claude Code's environment
  * `CLAUDE_PROJECT_DIR` - absolute path to the project root directory
  * `CLAUDE_CODE_REMOTE` - indicates whether running in a remote (web) environment (`"true"`) or local CLI environment

## Security Considerations

**USE AT YOUR OWN RISK**: Claude Code hooks execute arbitrary shell commands on your system automatically.

### Security Best Practices

1. **Validate and sanitize inputs** - Never trust input data blindly
2. **Always quote shell variables** - Use `"$VAR"` not `$VAR`
3. **Block path traversal** - Check for `..` in file paths
4. **Use absolute paths** - Specify full paths for scripts
5. **Skip sensitive files** - Avoid `.env`, `.git/`, keys, etc.

## Related resources

* [Claude Code on the web](claude-code-on-the-web.md)
* [Settings reference](settings.md)
* [Security](security.md)
