# Claude Code - CLI Reference

Complete reference for Claude Code command-line interface, including commands and flags.

## Basic Commands

| Command | Description | Example |
|---------|-------------|---------|
| `claude` | Start interactive REPL | `claude` |
| `claude "query"` | Start REPL with initial prompt | `claude "explain this project"` |
| `claude -p "query"` | Query via SDK, then exit | `claude -p "explain this function"` |
| `cat file \| claude -p "query"` | Process piped content | `cat logs.txt \| claude -p "explain"` |
| `claude -c` | Continue most recent conversation | `claude -c` |
| `claude -c -p "query"` | Continue via SDK | `claude -c -p "Check for type errors"` |
| `claude -r "<session-id>" "query"` | Resume session by ID | `claude -r "abc123" "Finish this PR"` |
| `claude update` | Update to latest version | `claude update` |
| `claude mcp` | Configure MCP servers | See MCP documentation |

## Command Flags

### Directory & Access Control

#### --add-dir
Add additional working directories for Claude to access (validates each path exists as a directory).

```bash
claude --add-dir ../apps ../lib
```

### Tool Permission Control

#### --allowedTools
A list of tools that should be allowed without prompting the user for permission, in addition to settings.json.

```bash
claude --allowedTools "Bash(git log:*)" "Bash(git diff:*)" "Read"
```

Supports:
- Exact tool names: `"Read"`, `"Write"`
- Bash patterns: `"Bash(git log:*)"`
- Wildcards: `"Bash(*)"`

#### --disallowedTools
A list of tools that should be blocked (takes precedence over allowedTools).

```bash
claude --disallowedTools "Bash(git log:*)" "Bash(git diff:*)" "Edit"
```

### Model & Execution Control

#### --print, -p
Execute query and exit (non-interactive mode).

```bash
claude -p "query"
```

#### --append-system-prompt
Append additional instructions to the system prompt (only works with --print).

```bash
claude --append-system-prompt "Custom instruction"
```

#### --model
Select Claude model (short name like `sonnet` or `opus`) or full model name.

```bash
claude --model claude-sonnet-4-20250514
claude --model opus
```

Available models:
- `sonnet` - Claude Sonnet 4.5 (default, most balanced)
- `opus` - Claude Opus 4.1 (maximum capability)
- `haiku` - Claude Haiku (fastest, most cost-effective)

#### --max-turns
Limit the number of conversation turns.

```bash
claude -p --max-turns 3 "query"
```

### Output & Format Control

#### --output-format
Specify output format: `text` (default), `json`, or `stream-json`.

```bash
claude -p "query" --output-format json
```

**JSON output format**:
```json
{
  "response": "Claude's response text",
  "tool_calls": [...],
  "usage": {
    "input_tokens": 100,
    "output_tokens": 200
  }
}
```

**Stream JSON format**: Line-delimited JSON for streaming responses.

#### --input-format
Specify input format: `text` (default) or `stream-json`.

```bash
claude -p --output-format json --input-format stream-json
```

#### --include-partial-messages
Include partial streaming messages in stream-json output (only with --print and --output-format=stream-json).

```bash
claude -p --output-format stream-json --include-partial-messages "query"
```

### Session Management

#### --resume
Resume a specific session by ID (alternative to -r).

```bash
claude --resume abc123 "query"
```

#### --continue
Continue most recent conversation (alternative to -c).

```bash
claude --continue
```

### Permission Modes

#### --permission-mode
Set permission handling mode: `ask` (default) or `plan`.

```bash
claude --permission-mode plan
```

Modes:
- `ask` - Prompt for each permission
- `plan` - Plan mode (Shift+Tab twice in interactive)

#### --permission-prompt-tool
Specify tools that require explicit permission prompt.

```bash
claude -p --permission-prompt-tool mcp_auth_tool "query"
```

#### --dangerously-skip-permissions
Skip all permission checks (dangerous - use only in isolated environments).

```bash
claude --dangerously-skip-permissions
```

**⚠️ WARNING**: This flag bypasses all safety checks. Use only in:
- Docker containers without network access
- Trusted, isolated environments
- Automated workflows with known safe commands

### Debug & Logging

#### --verbose
Enable verbose logging with detailed diagnostic information.

```bash
claude --verbose
```

Shows:
- API request/response details
- Tool invocations
- Context management
- Hook execution
- MCP server communication

## Custom Agents (--agents Flag)

Define custom subagents inline using JSON.

### Format
```bash
claude --agents '{
  "agent-name": {
    "description": "When to use this agent",
    "prompt": "System prompt for agent",
    "tools": ["Tool1", "Tool2"],
    "model": "sonnet"
  }
}'
```

### Example
```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  },
  "debugger": {
    "description": "Debugging specialist for errors and test failures.",
    "prompt": "You are an expert debugger. Analyze errors, identify root causes, and provide fixes.",
    "tools": ["Read", "Bash", "Edit"],
    "model": "opus"
  }
}'
```

### Agent Configuration Fields

| Field | Required | Description |
|-------|----------|-------------|
| `description` | Yes | When this agent should be invoked |
| `prompt` | Yes | System prompt defining agent behavior |
| `tools` | No | Array of tool names (inherits all if omitted) |
| `model` | No | Model to use: `sonnet`, `opus`, or `haiku` |

## MCP Commands

### claude mcp
Manage Model Context Protocol servers.

```bash
# List MCP servers
claude mcp list

# Add MCP server
claude mcp add <name> [options] -- <command> [args...]

# Remove MCP server
claude mcp remove <name>

# Get server details
claude mcp get <name>

# Import from Claude Desktop
claude mcp import
```

### MCP Add Examples

```bash
# Add filesystem server
claude mcp add filesystem -s user -- npx -y @modelcontextprotocol/server-filesystem ~/Documents

# Add with environment variables
claude mcp add github -s user -e GITHUB_TOKEN=$GITHUB_TOKEN -- npx -y @modelcontextprotocol/server-github

# Add with transport
claude mcp add --transport http stripe https://mcp.stripe.com
```

See MCP documentation for detailed information.

## Environment Variables

### API Configuration
```bash
export ANTHROPIC_API_KEY="sk-ant-..."     # API key (if not using claude.ai login)
export ANTHROPIC_MODEL="claude-sonnet-4-5-20250929"  # Default model
```

### Claude Code Paths
```bash
$CLAUDE_PROJECT_DIR    # Absolute path to project root (available in hooks)
```

### MCP Configuration
```bash
export MAX_MCP_OUTPUT_TOKENS=50000  # Maximum tokens for MCP tool output (default: 25000)
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Permission denied (hooks) |
| 130 | Interrupted (Ctrl+C) |

## Scripting Examples

### JSON Output Parsing
```bash
# Get TODO list as JSON
result=$(claude -p "List all TODO comments" --output-format json)
echo "$result" | jq -r '.response'
```

### CI/CD Integration
```bash
# Automated code review in CI
claude -p "Review this PR for security issues" \
  --output-format json \
  --max-turns 1 \
  > review-results.json
```

### Piped Input
```bash
# Analyze logs
tail -n 100 app.log | claude -p "Summarize errors and suggest fixes"

# Process git diff
git diff | claude -p "Write release notes from this diff"
```

### Automated Translation
```bash
# In CI pipeline
claude -p "If there are new text strings, translate them into French and raise a PR for @lang-fr-team to review"
```

### Monitoring
```bash
# Alert on anomalies
tail -f app.log | claude -p "Slack me if you see any anomalies appear in this log stream"
```

## Interactive Mode Commands

When in interactive REPL, use slash commands:

```bash
/help          # List all commands
/clear         # Clear conversation history
/model         # Switch model
/permissions   # Manage tool permissions
/hooks         # Configure hooks
/agents        # Manage subagents
/plugin        # Manage plugins
/mcp           # Manage MCP servers
/config        # Open settings
/bug           # Report bug
/doctor        # Diagnose issues
```

See Commands documentation for complete slash command reference.

## Advanced Usage Patterns

### Multi-Agent Workflows
```bash
# Define specialized agents for complex tasks
claude --agents '{
  "architect": {
    "description": "System architecture and design",
    "prompt": "Expert architect for system design"
  },
  "implementer": {
    "description": "Code implementation",
    "prompt": "Senior engineer for implementation"
  },
  "tester": {
    "description": "Test creation and validation",
    "prompt": "QA engineer for comprehensive testing"
  }
}' "Design and implement user authentication system"
```

### Safe Automation
```bash
# Run in container with limited permissions
docker run -v $(pwd):/workspace -w /workspace \
  claude-container \
  claude --dangerously-skip-permissions -p "Fix all lint errors"
```

### Performance Monitoring
```bash
# Track API usage
claude --verbose -p "analyze codebase" 2>&1 | tee claude-session.log
```

## Tips & Best Practices

### Performance
- Use `--max-turns` to limit API calls for simple queries
- Use `--output-format json` for parsing and automation
- Pipe specific content instead of analyzing entire codebases

### Safety
- Never use `--dangerously-skip-permissions` in production environments
- Always review tool permissions with `/permissions`
- Test in isolated environments first

### Productivity
- Create shell aliases for common patterns:
  ```bash
  alias claude-review='claude -p "Review last commit for issues"'
  alias claude-test='claude -p "Generate tests for"'
  ```
- Use `--add-dir` for monorepo workflows
- Leverage `--append-system-prompt` for consistent behavior

## Related Documentation
- [Main documentation](README.md)
- [Commands Documentation](commands.md)
- [MCP Documentation](mcp.md)
- [Agents Documentation](agent.md)
- [Hooks Documentation](hook.md)
- [Settings Documentation](settings.md)
