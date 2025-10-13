# Claude Code - Hooks Documentation

Complete reference for implementing event-driven automation in Claude Code, with multi-agent workflow patterns.

## What are Hooks?

Claude Code hooks are user-defined shell commands that execute at various points in Claude Code's lifecycle. They provide deterministic control over Claude's behavior, ensuring certain actions always happen rather than relying on the LLM to choose to run them.

### Benefits
- **Deterministic control**: Actions happen automatically without Claude choosing
- **Multi-agent coordination**: Automate workflows between specialized agents
- **Code quality**: Auto-format code after edits or enforce conventions
- **Smart execution**: Run only relevant checks based on what changed
- **Audit trail**: Log all executed commands for security and debugging
- **Workflow integration**: Connect Claude Code to existing development tools

---

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

---

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

---

## Multi-Agent Hooks Pattern

### Key Principle: Agent-Awareness via File Paths

**Hooks are GLOBAL** - they run for ALL agents. You can't directly target specific agents, but you can use **file path conventions** to achieve agent-specific behavior.

**Pattern:**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'case $FILE in backend/**/*.py) cd backend && uv run black ${FILE#backend/};; frontend/**/*.ts*) cd frontend && npx prettier --write ${FILE#frontend/};; supabase/migrations/*.sql) supabase gen types typescript --local > src/types/database.types.ts;; esac'"
          }
        ]
      }
    ]
  }
}
```

**Logic:**
- `backend/**/*.py` → backend-developer creates these
- `frontend/**/*.ts*` → frontend-developer creates these  
- `supabase/migrations/*.sql` → supabase-architect creates these
- `docs/openapi.yaml` → api-designer creates this

### Available Environment Variables

| Variable | Available In | Description |
|----------|--------------|-------------|
| `$FILE` | `PreToolUse`, `PostToolUse` | File being edited (via tool_input.file_path) |
| `$CLAUDE_PROJECT_DIR` | All | Absolute path to project root |
| `$CLAUDE_FILE_PATHS` | `PostToolUse` | File paths from tool |

**Note:** Extract `$FILE` from JSON in hooks:
```bash
FILE=$(jq -r '.tool_input.file_path // empty' <<< "$STDIN")
```

---

## Recommended Multi-Agent Hooks

### Complete hooks.json for Multi-Agent Projects

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'FILE=$(jq -r \".tool_input.file_path // empty\"); case $FILE in backend/**/*.py) cd backend && uv run black ${FILE#backend/};; frontend/**/*.ts*) cd frontend && npx prettier --write ${FILE#frontend/};; docs/openapi.yaml) npx @apidevtools/swagger-cli validate $FILE || true;; supabase/migrations/*.sql) supabase gen types typescript --local > src/types/database.types.ts 2>/dev/null || true;; esac'",
            "timeout": 30
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json, sys, re; data=json.load(sys.stdin); cmd=data.get('tool_input',{}).get('command',''); patterns=['rm\\\\s+.*-[rf]','sudo\\\\s+rm','chmod\\\\s+777','DROP\\\\s+DATABASE','TRUNCATE']; sys.exit(2 if any(re.search(p, cmd, re.I) for p in patterns) else 0)\"",
            "timeout": 5
          }
        ]
      },
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json, sys; data=json.load(sys.stdin); path=data.get('tool_input',{}).get('file_path',''); sys.exit(2 if any(p in path for p in ['.env', 'package-lock.json', '.git/', 'node_modules/', 'uv.lock']) else 0)\"",
            "timeout": 5
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'CHANGED=$(git diff --name-only 2>/dev/null || echo \"\"); if echo \"$CHANGED\" | grep -q \"supabase/migrations/\"; then echo \"⚠️  Migration created - Update docs/database/README.md\"; fi; if echo \"$CHANGED\" | grep -q \"docs/openapi.yaml\"; then echo \"⚠️  OpenAPI updated - backend-developer should implement endpoints\"; fi'",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### Smart Test Runner (Pre-Commit Alternative)

For git-based projects, run tests only for changed files:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'CHANGED=$(git diff --name-only 2>/dev/null || echo \"\"); BACKEND_CHANGED=$(echo \"$CHANGED\" | grep -E \"^backend/.*\\\\.py$|^supabase/migrations/\"); FRONTEND_CHANGED=$(echo \"$CHANGED\" | grep -E \"^frontend/\"); if [ -n \"$BACKEND_CHANGED\" ]; then cd backend && uv run pytest tests/ -x || echo \"⚠️  Backend tests failed\"; fi; if [ -n \"$FRONTEND_CHANGED\" ]; then cd frontend && npm test || echo \"⚠️  Frontend tests failed\"; fi'",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

**Why this is better:** Only runs Python tests when backend/migrations change, only runs frontend tests when frontend changes. Saves time!

---

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

**Multi-Agent Use Case**: Add project context automatically
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'echo \"{\\\"additionalContext\\\":\\\"Project Stack: uv+FastAPI+Next.js+Supabase. Read CLAUDE.md for patterns. Check docs/ for current state.\\\"}\"'"
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

**Multi-Agent Use Cases**:

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
            "command": "python3 -c \"import json, sys, re; data=json.load(sys.stdin); cmd=data.get('tool_input',{}).get('command',''); patterns=['rm\\\\s+.*-[rf]','sudo\\\\s+rm','chmod\\\\s+777','DROP\\\\s+DATABASE']; sys.exit(2 if any(re.search(p, cmd, re.I) for p in patterns) else 0)\""
          }
        ]
      }
    ]
  }
}
```

Block protected files (prevent agents from modifying):
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json, sys; data=json.load(sys.stdin); path=data.get('tool_input',{}).get('file_path',''); protected=['.env','package-lock.json','.git/','uv.lock','node_modules/']; sys.exit(2 if any(p in path for p in protected) else 0)\""
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

**Multi-Agent Workflow Automation**:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "comment": "Format code based on agent workspace",
            "command": "bash -c 'FILE=$(jq -r \".tool_input.file_path // empty\"); case $FILE in backend/**/*.py) cd backend && uv run black ${FILE#backend/};; frontend/**/*.ts*) cd frontend && npx prettier --write ${FILE#frontend/};; esac'"
          },
          {
            "type": "command",
            "comment": "Auto-generate types after migration",
            "command": "bash -c 'FILE=$(jq -r \".tool_input.file_path // empty\"); if [[ $FILE == supabase/migrations/*.sql ]]; then supabase gen types typescript --local > src/types/database.types.ts 2>/dev/null || true; echo \"✅ TypeScript types regenerated\"; fi'"
          },
          {
            "type": "command",
            "comment": "Validate OpenAPI spec",
            "command": "bash -c 'FILE=$(jq -r \".tool_input.file_path // empty\"); if [[ $FILE == docs/openapi.yaml ]]; then npx @apidevtools/swagger-cli validate $FILE || echo \"⚠️  OpenAPI validation failed\"; fi'"
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

**Multi-Agent Coordination Examples**:

Agent workflow reminders:
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'CHANGED=$(git diff --name-only 2>/dev/null || echo \"\"); if echo \"$CHANGED\" | grep -q \"supabase/migrations/\"; then echo \"📝 Migration created by supabase-architect\"; echo \"   ➡️  Next: backend-developer implement endpoints\"; echo \"   ➡️  Update: docs/database/README.md\"; fi; if echo \"$CHANGED\" | grep -q \"docs/openapi.yaml\"; then echo \"📝 API spec updated by api-designer\"; echo \"   ➡️  Next: backend-developer implement endpoints\"; fi'"
          }
        ]
      }
    ]
  }
}
```

Smart test runner (run only changed):
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'CHANGED=$(git diff --name-only 2>/dev/null); if echo \"$CHANGED\" | grep -qE \"^backend/.*\\\\.py$|^supabase/migrations/\"; then cd backend && uv run pytest tests/ -x; fi; if echo \"$CHANGED\" | grep -qE \"^frontend/\"; then cd frontend && npm test; fi'",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

### SubagentStop

Fires when subagent tasks complete.

**Input JSON**:
```json
{
  "hook_event_name": "SubagentStop",
  "session_id": "abc123...",
  "agent_name": "supabase-architect",
  "agent_result": {
    "status": "success",
    "output": "Migration created: 20250110_add_users.sql"
  }
}
```

**Multi-Agent Coordination**:
```json
{
  "hooks": {
    "SubagentStop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'AGENT=$(jq -r \".agent_name // empty\"); if [ \"$AGENT\" = \"supabase-architect\" ]; then echo \"🔄 DB agent finished - regenerating types...\"; supabase gen types typescript --local > src/types/database.types.ts 2>/dev/null || true; fi'"
          }
        ]
      }
    ]
  }
}
```

### Other Events

**Notification** - Custom notification systems  
**PreCompact** - Save context before compaction  
**SessionStart** - Initialize environment  
**SessionEnd** - Cleanup and reporting

See official documentation for detailed JSON schemas.

---

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
| `"mcp__*"` | All MCP tools |

### Multi-Agent Matcher Examples

```json
// Match file editing tools (any agent)
{"matcher": "Edit|MultiEdit|Write"}

// Match database operations (supabase-architect)
{"matcher": "Bash(supabase *)"}

// Match git commands (version control)
{"matcher": "Bash(git *)"}
```

---

## Agent-Specific Workflow Patterns

### Pattern 1: File Path Based Agent Detection

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "comment": "supabase-architect workflow",
            "command": "bash -c 'FILE=$(jq -r \".tool_input.file_path // empty\"); if [[ $FILE == supabase/migrations/*.sql ]]; then echo \"=== supabase-architect workflow ===\"; supabase gen types typescript --local > src/types/database.types.ts; echo \"⚠️  Update docs/database/README.md\"; fi'"
          },
          {
            "type": "command",
            "comment": "api-designer workflow",
            "command": "bash -c 'FILE=$(jq -r \".tool_input.file_path // empty\"); if [[ $FILE == docs/openapi.yaml ]]; then echo \"=== api-designer workflow ===\"; npx @apidevtools/swagger-cli validate $FILE; echo \"⚠️  backend-developer implement endpoints\"; fi'"
          },
          {
            "type": "command",
            "comment": "backend-developer workflow",
            "command": "bash -c 'FILE=$(jq -r \".tool_input.file_path // empty\"); if [[ $FILE == backend/src/**/*.py ]]; then echo \"=== backend-developer workflow ===\"; cd backend && uv run black ${FILE#backend/}; fi'"
          }
        ]
      }
    ]
  }
}
```

### Pattern 2: Conditional Execution by File Type

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'FILE=$(jq -r \".tool_input.file_path // empty\"); case $FILE in backend/**/*.py) cd backend && uv run black ${FILE#backend/} && uv run ruff check ${FILE#backend/};; frontend/**/*.ts*) cd frontend && npx prettier --write ${FILE#frontend/} && npx eslint --fix ${FILE#frontend/};; docs/openapi.yaml) npx @apidevtools/swagger-cli validate $FILE;; supabase/migrations/*.sql) supabase gen types typescript --local > src/types/database.types.ts;; *) echo \"No hook for $FILE\";; esac'"
          }
        ]
      }
    ]
  }
}
```

---

## Best Practices for Multi-Agent Workflows

### 1. Keep Hooks Fast
```bash
# ❌ Slow - blocks workflow
uv run pytest tests/  # Could take minutes

# ✅ Fast - only run on changed files
if echo "$CHANGED" | grep -qE '^backend/'; then cd backend && uv run pytest tests/ -x; fi
```

### 2. Use Silent Failures for Optional Checks
```bash
# ✅ Don't block on non-critical errors
supabase gen types typescript --local > types.ts 2>/dev/null || true
```

### 3. File Pattern Specificity
```bash
# ❌ Too broad - matches any Python file
if [[ $FILE == *.py ]]; then black $FILE; fi

# ✅ Specific to project structure
if [[ $FILE == backend/**/*.py ]]; then cd backend && uv run black ${FILE#backend/}; fi
```

### 4. Provide Agent Workflow Feedback
```bash
# ✅ Tell user what happened and what's next
echo "✅ Migration created by supabase-architect"
echo "➡️  Next: backend-developer implement endpoints"
echo "📝 Update: docs/database/README.md"
```

### 5. Smart Test Execution
```bash
# ✅ Only test what changed
CHANGED=$(git diff --name-only)
if echo "$CHANGED" | grep -qE '^backend/'; then 
  cd backend && uv run pytest tests/ -x
fi
```

---

## Security Considerations

### Risks in Multi-Agent Workflows
- Hooks run automatically for ALL agents
- Agents can trigger multiple hooks in sequence
- Complex workflows increase attack surface

### Mitigations
- **Block protected files** - .env, lock files, .git/
- **Block dangerous commands** - rm -rf, DROP DATABASE, etc.
- **Validate agent outputs** - OpenAPI validation, type checking
- **Log agent activity** - Track which agent did what
- **Use PreToolUse blocks** - Prevent dangerous operations before they execute

**Example security hook:**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "comment": "Block dangerous operations",
            "command": "python3 -c \"import json, sys, re; data=json.load(sys.stdin); cmd=data.get('tool_input',{}).get('command',''); dangerous=['rm\\\\s+.*-[rf]','DROP\\\\s+DATABASE','TRUNCATE','chmod\\\\s+777','sudo\\\\s+rm']; sys.exit(2 if any(re.search(p, cmd, re.I) for p in dangerous) else 0)\""
          }
        ]
      }
    ]
  }
}
```

---

## Debugging Multi-Agent Hooks

### Enable Debug Mode
```bash
claude --debug
claude -d --verbose
```

### Log Agent Activity
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'TOOL=$(jq -r \".tool_name // empty\"); FILE=$(jq -r \".tool_input.file_path // empty\"); echo \"[$(date \"+%H:%M:%S\")] Tool: $TOOL, File: $FILE\" >> .claude/activity.log'"
          }
        ]
      }
    ]
  }
}
```

### Test Hook Manually
```bash
# Simulate PostToolUse for migration
echo '{
  "hook_event_name": "PostToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "supabase/migrations/20250110_test.sql"
  }
}' | bash -c 'FILE=$(jq -r ".tool_input.file_path"); echo "File: $FILE"'
```

---

## Common Multi-Agent Hook Issues

### Issue: Hook Runs for Wrong Agent
**Cause:** File path pattern too broad

**Solution:**
```bash
# ❌ Matches any Python file
if [[ $FILE == *.py ]]

# ✅ Only backend directory
if [[ $FILE == backend/**/*.py ]]
```

### Issue: Tests Run Unnecessarily
**Cause:** Not checking what actually changed

**Solution:**
```bash
# ✅ Only run tests for changed parts
CHANGED=$(git diff --name-only)
if echo "$CHANGED" | grep -qE '^backend/'; then
  cd backend && uv run pytest
fi
```

### Issue: Type Generation Fails
**Cause:** Supabase not running or wrong directory

**Solution:**
```bash
# ✅ Silent failure with fallback
supabase gen types typescript --local > src/types/database.types.ts 2>/dev/null || true
```

---

## Summary

### Multi-Agent Hooks Enable:
✅ **Automatic workflows** - supabase-architect creates migration → types auto-generate  
✅ **Smart execution** - Only run tests for changed parts  
✅ **Agent coordination** - Reminders about next steps  
✅ **Quality enforcement** - Auto-format, validate, lint  
✅ **Security** - Block dangerous operations  

### Key Patterns:
1. **File path detection** - Infer which agent by file location
2. **Git-aware conditionals** - Run only on changed files  
3. **Silent failures** - Don't block on optional operations
4. **Workflow reminders** - Tell user what's next
5. **Security blocks** - Prevent dangerous operations

### Recommended Setup:
Use the complete multi-agent hooks.json provided above for:
- Format on save (agent-aware)
- Type generation (supabase-architect)
- OpenAPI validation (api-designer)
- Smart test runner (only changed parts)
- Security blocks (dangerous commands/files)
- Workflow coordination (agent reminders)

---

## Related Documentation
- [Agents & Subagents](agents.md)
- [CLI Reference](cli.md)
- [Settings Documentation](settings.md)
- [MCP Documentation](mcp.md)