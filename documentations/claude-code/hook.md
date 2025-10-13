# Claude Code - Hooks Documentation

Complete reference for implementing event-driven automation in Claude Code through hooks.

## What are Hooks?

Claude Code hooks are user-defined shell commands that execute at various points in Claude Code's lifecycle. They provide deterministic control over Claude's behavior, ensuring certain actions always happen rather than relying on the LLM to choose to run them.

### Benefits
- **Deterministic control**: Actions happen automatically without Claude choosing
- **Notification integration**: Get alerts about tool usage and permission requests
- **Code quality**: Auto-format code after edits or enforce conventions
- **Audit trail**: Log all executed commands for security and debugging
- **Workflow integration**: Connect Claude Code to existing development tools

## Hook Events

Claude Code provides several hook events that run at different points in the workflow:

| Event | When it fires | Use cases |
|-------|---------------|-----------|
| `UserPromptSubmit` | When user submits a prompt (before Claude processes it) | Validate prompts, add context, security filtering |
| `PreToolUse` | Before tool calls (can block them) | Block dangerous commands, validate inputs, logging |
| `PostToolUse` | After tool calls complete | Auto-format code, run tests, notify completion |
| `Notification` | When Claude sends notifications | Custom notification systems, alerts |
| `Stop` | When Claude finishes responding | Commit changes, update docs, cleanup |
| `SubagentStop` | When subagent tasks complete | Multi-agent coordination, result aggregation |
| `PreCompact` | Before context compaction | Save important context, logging |
| `SessionStart` | When Claude Code session starts | Initialize environment, load configs |
| `SessionEnd` | When Claude Code session ends | Cleanup, save state, reports |

## Configuration

### Configuration Files

Hooks are configured in settings files with hierarchical precedence:

1. **Project settings** (`.claude/settings.json`) - Highest priority, shared with team
2. **Local project settings** (`.claude/settings.local.json`) - Personal overrides
3. **User settings** (`~/.claude/settings.json`) - Global personal hooks

### Basic Syntax

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

### Configuration with /hooks Command

```bash
# Interactive hook configuration
/hooks

# Select event type
# Choose tool matcher (for PreToolUse/PostToolUse)
# Enter command
# Configuration is added to settings.json
```

## Hook Event Details

### UserPromptSubmit

Fires immediately when user submits a prompt, before Claude processes it.

**Input JSON**:
```json
{
  "hook_event_name": "UserPromptSubmit",
  "prompt": "user's prompt text",
  "session_id": "abc123...",
  "timestamp": "2024-01-20T15:30:45.123Z"
}
```

**Control Options**:
```json
{
  "continue": false,
  "stopReason": "Prompt blocked due to security policy",
  "additionalContext": "Context to prepend to prompt"
}
```

**Use Cases**:
- Validate prompts for security
- Add project context automatically
- Block dangerous patterns
- Log all prompts
- Enhance prompts with metadata

**Example**:
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.claude/hooks/validate-prompt.py"
          }
        ]
      }
    ]
  }
}
```

### PreToolUse

Fires before Claude uses any tool (file operations, bash commands, etc.).

**Input JSON**:
```json
{
  "hook_event_name": "PreToolUse",
  "session_id": "abc123...",
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf /tmp/data",
    "description": "Clean temporary files"
  }
}
```

**Control Options**:
```json
{
  "decision": "block",
  "reason": "Dangerous command blocked",
  "permissionDecision": "deny"
}
```

Exit codes:
- `0` - Allow (continue normally)
- `1` - Error (show error to user)
- `2` - Block (prevent tool execution)

**Use Cases**:
- Block dangerous commands
- Validate file paths
- Log tool usage
- Check permissions
- Security filtering

**Examples**:

Block dangerous patterns:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json, sys, re; data=json.load(sys.stdin); cmd=data.get('tool_input',{}).get('command',''); patterns=['rm\\\\s+.*-[rf]','sudo\\\\s+rm','chmod\\\\s+777']; sys.exit(2 if any(re.search(p, cmd, re.I) for p in patterns) else 0)\""
          }
        ]
      }
    ]
  }
}
```

Block sensitive files:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json, sys; data=json.load(sys.stdin); path=data.get('tool_input',{}).get('file_path',''); sys.exit(2 if any(p in path for p in ['.env', 'package-lock.json', '.git/']) else 0)\""
          }
        ]
      }
    ]
  }
}
```

Log commands:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '\"\\(.tool_input.command) - \\(.tool_input.description // \\\"No description\\\")\"' >> ~/.claude/bash-command-log.txt"
          }
        ]
      }
    ]
  }
}
```

### PostToolUse

Fires after successful tool completion.

**Input JSON**:
```json
{
  "hook_event_name": "PostToolUse",
  "session_id": "abc123...",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "src/utils.ts",
    "instructions": "Add type annotations"
  },
  "tool_response": {
    "success": true,
    "message": "File edited successfully"
  }
}
```

**Use Cases**:
- Auto-format code after edits
- Run linters
- Execute tests
- Update documentation
- Notify team members

**Examples**:

Auto-format TypeScript:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | { read file_path; if echo \"$file_path\" | grep -qE '\\.(ts|tsx)$'; then npx prettier --write \"$file_path\"; fi; }"
          }
        ]
      }
    ]
  }
}
```

Run type checking:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_FILE_PATHS\" =~ \\.(ts|tsx)$ ]]; then npx tsc --noEmit --skipLibCheck \"$CLAUDE_FILE_PATHS\" || echo '⚠️  TypeScript errors detected'; fi"
          }
        ]
      }
    ]
  }
}
```

Format Python with Black:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | { read file_path; if echo \"$file_path\" | grep -q '\\.py$'; then black \"$file_path\"; fi; }"
          }
        ]
      }
    ]
  }
}
```

### Notification

Fires when Claude sends notifications or requests permissions.

**Input JSON**:
```json
{
  "hook_event_name": "Notification",
  "session_id": "abc123...",
  "message": "Awaiting your input"
}
```

**Use Cases**:
- Custom notification systems
- Desktop notifications
- Team alerts (Slack, email)
- Sound notifications

**Examples**:

macOS notification:
```json
{
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs input\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

Linux notification:
```json
{
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Awaiting your input'"
          }
        ]
      }
    ]
  }
}
```

### Stop

Fires when Claude finishes responding and stops.

**Input JSON**:
```json
{
  "hook_event_name": "Stop",
  "session_id": "abc123...",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "workspace": {
    "current_dir": "/current/dir",
    "project_dir": "/project/root"
  },
  "cost": {
    "total_cost_usd": 0.01234,
    "total_duration_ms": 45000,
    "total_lines_added": 156,
    "total_lines_removed": 23
  }
}
```

**Use Cases**:
- Auto-commit changes
- Update documentation
- Run tests
- Generate reports
- Track usage/costs

**Examples**:

Auto-commit with prompt message:
```bash
#!/bin/bash
# Extract last prompt from transcript
prompt=$(jq -r 'select(.role=="user") | .content' ~/.claude/transcript.jsonl | tail -1)
git add -A
git commit -m "$prompt"
```

Notification with sound:
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude has finished!\" with title \"✅ Claude Done\" sound name \"Glass\"'"
          }
        ]
      }
    ]
  }
}
```

Track usage:
```bash
#!/usr/bin/env python3
import json, sys
data = json.load(sys.stdin)
cost = data.get('cost', {})
with open('~/.claude/usage.log', 'a') as f:
    f.write(f"Session: {data['session_id']}, Cost: ${cost.get('total_cost_usd', 0):.4f}\n")
```

### SubagentStop

Fires when subagent tasks complete.

**Input JSON**:
```json
{
  "hook_event_name": "SubagentStop",
  "session_id": "abc123...",
  "agent_name": "code-reviewer",
  "agent_result": {
    "status": "success",
    "output": "Review complete"
  }
}
```

**Use Cases**:
- Multi-agent coordination
- Result aggregation
- Logging subagent activity
- Chain subagent tasks

### PreCompact

Fires before Claude Code runs a compact operation to summarize context.

**Input JSON**:
```json
{
  "hook_event_name": "PreCompact",
  "session_id": "abc123...",
  "current_tokens": 95000,
  "target_tokens": 50000
}
```

**Use Cases**:
- Save important context before compaction
- Log compaction events
- Notify about context cleanup

### SessionStart

Fires when Claude Code session starts.

**Input JSON**:
```json
{
  "hook_event_name": "SessionStart",
  "session_id": "abc123...",
  "workspace": {
    "project_dir": "/path/to/project"
  }
}
```

**Use Cases**:
- Initialize environment
- Load configuration
- Start services
- Log session start

### SessionEnd

Fires when Claude Code session ends.

**Input JSON**:
```json
{
  "hook_event_name": "SessionEnd",
  "session_id": "abc123...",
  "duration_ms": 1234567
}
```

**Use Cases**:
- Cleanup resources
- Save session state
- Generate reports
- Stop services

## Matchers

Matchers filter when hooks should fire (only applicable for PreToolUse and PostToolUse).

### Syntax

```json
{
  "matcher": "ToolPattern"
}
```

### Patterns

| Pattern | Matches |
|---------|---------|
| `"Write"` | Exact match - only Write tool |
| `"Edit\|MultiEdit\|Write"` | Any of these tools (pipe-separated) |
| `"*"` or `""` | All tools |
| `"Bash"` | All Bash tool invocations |
| `"Bash(git *)"` | Bash commands starting with "git" |
| `"Bash(git log:*)"` | Bash commands matching "git log" |
| `"mcp__*"` | All MCP tools |
| `"mcp__github__*"` | All GitHub MCP tools |

### Examples

```json
// Match file editing tools
{"matcher": "Edit|MultiEdit|Write"}

// Match all Bash commands
{"matcher": "Bash"}

// Match git commands only
{"matcher": "Bash(git *)"}

// Match all MCP tools
{"matcher": "mcp__*"}

// Match specific MCP server
{"matcher": "mcp__github__*"}
```

## Control Flow

### Exit Codes

| Code | Meaning | Effect |
|------|---------|--------|
| `0` | Success | Continue normally |
| `1` | Error | Show error to user, continue |
| `2` | Block | Prevent action (PreToolUse only) |

### JSON Output

Hooks can return structured JSON for advanced control:

```json
{
  "continue": true,
  "suppressOutput": false,
  "stopReason": "string",
  "decision": "approve|block",
  "reason": "Explanation"
}
```

#### UserPromptSubmit Control
```json
{
  "continue": false,
  "stopReason": "Prompt blocked",
  "additionalContext": "Project: MyApp\nContext: Production environment"
}
```

#### PreToolUse Control
```json
{
  "decision": "block",
  "reason": "Dangerous operation detected",
  "permissionDecision": "deny"
}
```

#### Stop Control
```json
{
  "decision": "block",
  "reason": "Tests failing, cannot complete"
}
```

## Environment Variables

### Available Variables

```bash
$CLAUDE_PROJECT_DIR     # Absolute path to project root
$CLAUDE_FILE_PATHS      # File paths from tool (PostToolUse)
```

### Usage

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

## Advanced Patterns

### Multi-Hook Chains

Execute multiple hooks for the same event:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {"type": "command", "command": "prettier --write $file"},
          {"type": "command", "command": "eslint --fix $file"},
          {"type": "command", "command": "jest --findRelatedTests $file"}
        ]
      }
    ]
  }
}
```

### Conditional Execution

```bash
#!/bin/bash
# Only run on TypeScript files
if echo "$file" | grep -qE '\.(ts|tsx)$'; then
  npx prettier --write "$file"
fi
```

### Python Hooks

```python
#!/usr/bin/env python3
import json, sys, re

# Read hook input
data = json.load(sys.stdin)

# Process
tool_input = data.get('tool_input', {})
command = tool_input.get('command', '')

# Check for dangerous patterns
dangerous = [
    r'rm\s+.*-[rf]',
    r'sudo\s+rm',
    r'chmod\s+777'
]

for pattern in dangerous:
    if re.search(pattern, command, re.IGNORECASE):
        print(json.dumps({
            "decision": "block",
            "reason": f"Blocked dangerous pattern: {pattern}"
        }))
        sys.exit(0)

# Allow if safe
sys.exit(0)
```

### GitButler Integration

Automatic branch management:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [{"type": "command", "command": "but claude pre-tool"}]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [{"type": "command", "command": "but claude post-tool"}]
      }
    ],
    "Stop": [
      {
        "hooks": [{"type": "command", "command": "but claude stop"}]
      }
    ]
  }
}
```

## Best Practices

### Security
- **Always review hooks** before enabling them
- **Use exit code 2** to block dangerous operations
- **Validate inputs** in PreToolUse hooks
- **Never trust user input** without sanitization
- **Log security events** for audit trail

### Performance
- **Keep hooks fast** (< 1 second when possible)
- **Use timeouts** to prevent hanging
- **Run expensive operations** in background
- **Cache results** when appropriate

### Reliability
- **Handle errors gracefully**
- **Provide informative error messages**
- **Test hooks thoroughly**
- **Use logging for debugging**

### Maintainability
- **Document your hooks**
- **Use external scripts** for complex logic
- **Share hooks with team** via project settings
- **Version control hooks** in .claude/

## Debugging

### Enable Debug Mode
```bash
claude --debug
claude -d
```

### Check Hook Execution
```bash
# Verbose output
claude --verbose

# MCP and hook debugging
claude --mcp-debug --verbose
```

### Test Hooks Manually
```bash
# Simulate hook input
echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | ./your-hook.sh
```

### Common Issues

**Hook not firing**:
- Check matcher pattern
- Verify file permissions (chmod +x)
- Review configuration with `/hooks`
- Check settings precedence

**Hook timeout**:
- Increase timeout in configuration
- Move slow operations to background
- Optimize hook script

**Permission denied**:
- Make script executable
- Check file paths
- Verify environment variables

## Security Considerations

### Risks
- Hooks run automatically with your credentials
- Malicious hooks can exfiltrate data
- Hooks can modify files without confirmation
- Hooks can execute arbitrary commands

### Mitigations
- **Review all hooks** before enabling
- **Use version control** for hooks
- **Limit hook scope** with matchers
- **Test in isolated environments**
- **Monitor hook activity** with logging
- **Use containers** for untrusted hooks

## Related Documentation
- [Main documentation](README.md)
- [CLI Reference](cli.md)
- [Settings Documentation](settings.md)
- [MCP Documentation](mcp.md)
- [Hooks Documentation](hook.md)
- [Plugins Documentation](plugin.md)
