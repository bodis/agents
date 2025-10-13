# Claude Code - Settings & Configuration Documentation

Complete reference for configuring Claude Code through settings files and environment variables.

## Configuration Overview

Claude Code uses a hierarchical configuration system with three levels:

### Configuration Hierarchy (Priority Order)

1. **Project Settings** (`.claude/settings.json`) - Highest priority
   - Shared with team via version control
   - Project-specific configurations
   - Overrides user settings

2. **Local Project Settings** (`.claude/settings.local.json`)
   - Personal overrides for the project
   - Not committed to version control
   - Overrides user settings but lower than project settings

3. **User Settings** (`~/.claude/settings.json`)
   - Global personal preferences
   - Applies to all projects
   - Lowest priority

### Configuration Methods

1. **Interactive Configuration**: `/config` command
2. **Direct File Editing**: Edit settings.json files
3. **Environment Variables**: Runtime configuration
4. **CLI Flags**: Session-specific overrides

## Settings File Location

### User Level
```bash
~/.claude/settings.json
```

### Project Level
```bash
# Committed to git (shared with team)
.claude/settings.json

# Not committed (personal)
.claude/settings.local.json
```

### Project Structure
```
your-project/
├── .claude/
│   ├── settings.json         # Shared project settings
│   ├── settings.local.json   # Personal project settings
│   ├── commands/             # Custom commands
│   ├── agents/               # Custom agents
│   └── hooks/                # Hook scripts
├── .mcp.json                 # MCP server config (project)
└── CLAUDE.md                 # Project documentation
```

## Interactive Configuration

### Open Settings Interface
```bash
/config
```

Opens tabbed interface with:
- **General**: Model, permissions, directories
- **Tools**: Allowed/disallowed tools
- **Agents**: Subagent management
- **MCP**: MCP server configuration
- **Plugins**: Plugin management
- **Hooks**: Hook configuration
- **Advanced**: Status line, output styles

## Complete Settings Schema

### Basic Structure
```json
{
  "model": "claude-sonnet-4-5-20250929",
  "allowedTools": ["Read", "Write", "Bash"],
  "disallowedTools": ["Edit"],
  "mcpServers": {},
  "hooks": {},
  "statusLine": {},
  "enabledPlugins": {},
  "extraKnownMarketplaces": {}
}
```

## Model Configuration

### Setting Default Model
```json
{
  "model": "claude-sonnet-4-5-20250929"
}
```

### Available Models
```json
{
  "model": "claude-sonnet-4-5-20250929",  // Default, balanced
  "model": "claude-opus-4-1-20250805",    // Maximum capability
  "model": "claude-haiku-3-5-20250514"    // Fastest, cost-effective
}
```

### Short Names
```json
{
  "model": "sonnet",  // Latest Sonnet
  "model": "opus",    // Latest Opus
  "model": "haiku"    // Latest Haiku
}
```

### Via Environment Variable
```bash
export ANTHROPIC_MODEL="claude-sonnet-4-5-20250929"
```

### Via CLI Flag
```bash
claude --model opus
claude --model claude-opus-4-1-20250805
```

## Tool Permissions

### Allowed Tools
Tools that don't require permission prompts.

```json
{
  "allowedTools": [
    "Read",
    "Write",
    "Edit",
    "Bash",
    "Grep",
    "Glob"
  ]
}
```

### Disallowed Tools
Tools that are blocked entirely.

```json
{
  "disallowedTools": [
    "Write",
    "Edit"
  ]
}
```

### Bash Command Patterns
```json
{
  "allowedTools": [
    "Bash(git *)",           // All git commands
    "Bash(npm test)",        // Specific command
    "Bash(npm *)",           // All npm commands
    "Read",
    "Grep"
  ],
  "disallowedTools": [
    "Bash(rm *)",            // Block rm commands
    "Bash(sudo *)"           // Block sudo
  ]
}
```

### MCP Tools
```json
{
  "allowedTools": [
    "mcp__github__*",                    // All GitHub MCP tools
    "mcp__github__create_issue",         // Specific tool
    "mcp__*"                             // All MCP tools
  ]
}
```

### Configuration via /permissions
```bash
/permissions
```

Interactive interface to:
- View current permissions
- Add/remove allowed tools
- Add/remove disallowed tools
- Add domain allowlist (web_fetch)

## MCP Server Configuration

### In settings.json
```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
    }
  }
}
```

### In .mcp.json (Project Level)
```json
{
  "mcpServers": {
    "project-db": {
      "command": "npx",
      "args": ["-y", "@company/db-mcp-server"],
      "env": {
        "DB_URL": "postgresql://localhost/mydb"
      }
    }
  }
}
```

See MCP Documentation for complete reference.

## Hooks Configuration

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'About to run command' >> /tmp/log.txt"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $file",
            "timeout": 30
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "git add -A && git commit -m 'Auto-commit'"
          }
        ]
      }
    ]
  }
}
```

See Hooks Documentation for complete reference.

## Status Line Configuration

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

See Status Line Documentation for complete reference.

## Plugin Configuration

### Enabled Plugins
```json
{
  "enabledPlugins": {
    "code-formatter@team-tools": true,
    "security-scanner@team-tools": true,
    "experimental-plugin@community": false
  }
}
```

### Plugin Marketplaces
```json
{
  "extraKnownMarketplaces": {
    "team-tools": {
      "source": "github",
      "repo": "company/claude-plugins"
    },
    "internal": {
      "source": "git",
      "url": "https://git.internal.com/plugins.git"
    }
  }
}
```

See Plugins Documentation for complete reference.

## Directory Configuration

### Additional Working Directories
```json
{
  "additionalWorkingDirectories": [
    "../shared-lib",
    "../common-utils"
  ]
}
```

### Via CLI Flag
```bash
claude --add-dir ../apps ../lib
```

## Advanced Settings

### Output Styles
```json
{
  "outputStyle": "default"
}
```

Available styles:
- `default` - Standard output
- `compact` - Minimal output
- `verbose` - Detailed output

### Permission Mode
```json
{
  "permissionMode": "ask"
}
```

Modes:
- `ask` - Prompt for each permission (default)
- `plan` - Plan mode, no execution without approval

### Max Turns
```json
{
  "maxTurns": 10
}
```

Limit conversation turns (useful for cost control).

## Complete Example Settings

### User Settings (~/.claude/settings.json)
```json
{
  "model": "claude-sonnet-4-5-20250929",
  
  "allowedTools": [
    "Read",
    "Grep",
    "Glob",
    "Bash(git *)",
    "Bash(npm *)"
  ],
  
  "disallowedTools": [
    "Bash(rm -rf *)",
    "Bash(sudo *)"
  ],
  
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  },
  
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs input\"'"
          }
        ]
      }
    ]
  },
  
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh"
  },
  
  "enabledPlugins": {
    "my-tools@personal": true
  }
}
```

### Project Settings (.claude/settings.json)
```json
{
  "allowedTools": [
    "Read",
    "Write",
    "Edit",
    "Bash(npm test)",
    "Bash(npm run build)",
    "Bash(git *)"
  ],
  
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\""
          },
          {
            "type": "command",
            "command": "npx eslint --fix \"$CLAUDE_FILE_PATHS\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "npm test"
          }
        ]
      }
    ]
  },
  
  "enabledPlugins": {
    "team-standards@company": true,
    "security-tools@company": true
  }
}
```

### Local Settings (.claude/settings.local.json)
```json
{
  "model": "claude-opus-4-1-20250805",
  
  "mcpServers": {
    "local-db": {
      "command": "npx",
      "args": ["@company/db-server"],
      "env": {
        "DB_URL": "postgresql://localhost:5432/dev_db",
        "DB_PASSWORD": "my-local-password"
      }
    }
  },
  
  "allowedTools": [
    "Bash(*)"
  ]
}
```

## Environment Variables

### API Configuration
```bash
# API Key (alternative to claude.ai login)
export ANTHROPIC_API_KEY="sk-ant-..."

# Default Model
export ANTHROPIC_MODEL="claude-sonnet-4-5-20250929"

# API Base URL (for proxies)
export ANTHROPIC_BASE_URL="https://api.anthropic.com"
```

### MCP Configuration
```bash
# Maximum MCP tool output tokens
export MAX_MCP_OUTPUT_TOKENS=50000
```

### Project Paths
```bash
# Available in hooks
$CLAUDE_PROJECT_DIR      # Absolute path to project root
$CLAUDE_FILE_PATHS       # Files modified (in PostToolUse hooks)
```

### Plugin Paths
```bash
# Available in plugin hooks
${CLAUDE_PLUGIN_ROOT}    # Plugin directory path
```

## Configuration Management

### View Current Configuration
```bash
/config
```

Shows:
- Active settings from all levels
- Merged configuration
- Setting sources (user/project/local)

### Validate Configuration
```bash
# Check for errors
cat .claude/settings.json | jq .
```

### Reset Configuration
```bash
# Remove user settings
rm ~/.claude/settings.json

# Remove project settings
rm .claude/settings.json
rm .claude/settings.local.json
```

### Backup Configuration
```bash
# Backup user settings
cp ~/.claude/settings.json ~/.claude/settings.json.backup

# Backup project settings
cp .claude/settings.json .claude/settings.json.backup
```

## Team Collaboration

### Shared Configuration (Commit)
```json
// .claude/settings.json
{
  "allowedTools": ["Read", "Write", "Bash(npm *)"],
  
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit",
      "hooks": [{
        "type": "command",
        "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/format.sh"
      }]
    }]
  },
  
  "enabledPlugins": {
    "team-standards@company": true
  }
}
```

### Personal Configuration (Don't Commit)
```json
// .claude/settings.local.json
{
  "model": "claude-opus-4-1-20250805",
  
  "allowedTools": ["Bash(*)"],
  
  "mcpServers": {
    "personal-notes": {
      "command": "npx",
      "args": ["-y", "@me/notes-server"]
    }
  }
}
```

### .gitignore Setup
```bash
# Add to .gitignore
.claude/settings.local.json
.claude/logs/
```

Claude Code automatically configures git to ignore `settings.local.json` when created.

## Security Best Practices

### API Keys
```json
// ❌ Bad - hardcoded
{
  "mcpServers": {
    "github": {
      "env": {
        "GITHUB_TOKEN": "ghp_hardcoded_token"
      }
    }
  }
}

// ✅ Good - environment variable
{
  "mcpServers": {
    "github": {
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### Tool Permissions
```json
// ❌ Dangerous
{
  "allowedTools": ["Bash(*)"]
}

// ✅ Safe
{
  "allowedTools": [
    "Read",
    "Bash(git *)",
    "Bash(npm test)"
  ],
  "disallowedTools": [
    "Bash(rm *)",
    "Bash(sudo *)"
  ]
}
```

### File Access
```json
// Use hooks to block sensitive files
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "python3 -c \"import json, sys; data=json.load(sys.stdin); path=data.get('tool_input',{}).get('file_path',''); sys.exit(2 if any(p in path for p in ['.env', 'secrets.yml']) else 0)\""
      }]
    }]
  }
}
```

## Debugging Configuration

### Check Effective Configuration
```bash
claude --verbose
```

Shows:
- Which settings file is being used
- Merged configuration
- Permission decisions
- Tool invocations

### Validate JSON
```bash
# Check syntax
jq . ~/.claude/settings.json

# Check syntax and format
jq . .claude/settings.json | sponge .claude/settings.json
```

### Common Issues

**Settings not applied**:
- Check file location
- Validate JSON syntax
- Check hierarchy (project > user)
- Restart Claude Code session

**Permission errors**:
- Review `allowedTools` and `disallowedTools`
- Check hook blocking in PreToolUse
- Use `/permissions` to diagnose

**Hooks not firing**:
- Direct edits require `/hooks` menu review
- Check executable permissions
- Validate hook configuration
- Test manually

## Configuration Templates

### Minimal Configuration
```json
{
  "model": "claude-sonnet-4-5-20250929",
  "allowedTools": ["Read", "Grep", "Glob"]
}
```

### Development Configuration
```json
{
  "model": "claude-sonnet-4-5-20250929",
  
  "allowedTools": [
    "Read", "Write", "Edit",
    "Bash(npm *)", "Bash(git *)",
    "Grep", "Glob"
  ],
  
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit",
      "hooks": [{
        "type": "command",
        "command": "prettier --write $file && eslint --fix $file"
      }]
    }]
  }
}
```

### Production Configuration
```json
{
  "model": "claude-sonnet-4-5-20250929",
  
  "allowedTools": [
    "Read", "Grep", "Glob",
    "Bash(git log *)",
    "Bash(git diff *)"
  ],
  
  "disallowedTools": [
    "Write", "Edit",
    "Bash(rm *)", "Bash(sudo *)"
  ],
  
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/audit.sh"
      }]
    }]
  }
}
```

### Team Configuration
```json
{
  "allowedTools": [
    "Read", "Write", "Edit",
    "Bash(npm test)",
    "Bash(npm run lint)",
    "Bash(git *)"
  ],
  
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [
        {
          "type": "command",
          "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/format.sh"
        },
        {
          "type": "command",
          "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/lint.sh"
        }
      ]
    }]
  },
  
  "enabledPlugins": {
    "code-standards@company": true,
    "testing-suite@company": true,
    "documentation@company": true
  }
}
```

## Migration & Updates

### Migrating Configuration
```bash
# Export current configuration
cp ~/.claude/settings.json ~/claude-config-backup.json

# Import to new machine
cp ~/claude-config-backup.json ~/.claude/settings.json
```

### Updating for New Features
When Claude Code adds new features, update configuration:

```bash
# Review release notes
claude update

# Update settings to use new features
code ~/.claude/settings.json
```

## Related Documentation
- [Main documentation](README.md)
- [CLI Reference](cli.md)
- [Hooks Documentation](hook.md)
- [MCP Documentation](mcp.md)
- [Plugins Documentation](plugin.md)
- [Agents Documentation](agent.md)
- [Status Line Documentation](status-line.md)
