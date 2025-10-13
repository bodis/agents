# Claude Code - Plugins Documentation

Complete reference for the Claude Code plugin system, including creation, distribution, and management.

## What are Plugins?

Plugins are lightweight packages that extend Claude Code functionality by bundling:
- **Slash commands**: Custom shortcuts for operations
- **Subagents**: Specialized AI assistants
- **MCP servers**: External tool integrations
- **Hooks**: Event-driven automation

### Benefits
- **Easy sharing**: One command installation
- **Team consistency**: Standardized workflows
- **Modularity**: Enable/disable as needed
- **Version control**: Track plugin updates
- **Community**: Access shared best practices

## Quick Start

### Install Plugin
```bash
# Interactive installation
/plugin

# Install from marketplace
/plugin install [plugin-name]@[marketplace]

# Example
/plugin install security-auditor@team-tools
```

### Enable/Disable Plugins
```bash
# Interactive management
/plugin

# Toggle plugin on/off
# Reduces context when disabled
```

### List Installed Plugins
```bash
/plugin list
```

## Plugin Structure

### Directory Layout
```
my-plugin/
├── plugin.json              # Plugin manifest (required)
├── commands/                # Slash commands
│   ├── review.md
│   └── deploy.md
├── agents/                  # Subagents
│   ├── reviewer.md
│   └── tester.md
├── hooks/                   # Event hooks
│   └── hooks.json
├── .mcp.json               # MCP servers
└── README.md               # Documentation
```

### Minimal Plugin
```
minimal-plugin/
├── plugin.json
└── commands/
    └── hello.md
```

## Plugin Manifest (plugin.json)

### Basic Format
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Plugin description",
  "author": {
    "name": "Your Name",
    "email": "[email protected]"
  }
}
```

### Complete Schema
```json
{
  "name": "enterprise-tools",
  "version": "2.1.0",
  "description": "Enterprise workflow automation tools",
  
  "author": {
    "name": "Enterprise Team",
    "email": "[email protected]"
  },
  
  "homepage": "https://docs.company.com/plugins/enterprise-tools",
  "repository": "https://github.com/company/enterprise-plugin",
  "license": "MIT",
  "keywords": ["enterprise", "workflow", "automation"],
  "category": "productivity",
  
  "commands": [
    "./commands/core/",
    "./commands/enterprise/",
    "./commands/experimental/preview.md"
  ],
  
  "agents": [
    "./agents/security-reviewer.md",
    "./agents/compliance-checker.md"
  ],
  
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{
          "type": "command",
          "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
        }]
      }
    ]
  },
  
  "mcpServers": {
    "enterprise-db": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  },
  
  "strict": false
}
```

### Required Fields

| Field | Description |
|-------|-------------|
| `name` | Unique plugin identifier |
| `version` | Semantic version (X.Y.Z) |
| `description` | Brief plugin description |

### Optional Fields

| Field | Default | Description |
|-------|---------|-------------|
| `author` | - | Author information (name, email) |
| `homepage` | - | Plugin documentation URL |
| `repository` | - | Source code repository |
| `license` | - | License identifier (MIT, Apache, etc.) |
| `keywords` | [] | Searchable keywords |
| `category` | - | Plugin category |
| `commands` | - | Paths to command files/directories |
| `agents` | - | Paths to agent files |
| `hooks` | - | Hook configuration inline or path |
| `mcpServers` | - | MCP configuration inline or path |
| `strict` | true | Fail on validation errors |

## Plugin Components

### 1. Commands

Commands are Markdown files with optional frontmatter.

**Location**: `commands/` directory

**File format**:
```markdown
---
description: Command description for help menu
argument-hint: "<what to provide>"
---

# Command Implementation

Task description: $ARGUMENTS

[Instructions for Claude]
```

**Example** (`commands/review.md`):
```markdown
---
description: Comprehensive code review
argument-hint: "<files or scope>"
---

Perform a comprehensive code review of: $ARGUMENTS

Review criteria:
1. Code quality and maintainability
2. Security vulnerabilities
3. Performance implications
4. Test coverage
5. Documentation completeness

Output format:
## Summary
[Overview]

## Issues
### Critical
[List]

### Suggestions
[List]
```

**Invocation**:
```bash
/review src/auth.ts
```

### 2. Agents

Agents are Markdown files with YAML frontmatter.

**Location**: `agents/` directory

**Format**: See Agents documentation for complete format.

**Example** (`agents/security-reviewer.md`):
```markdown
---
name: security-reviewer
description: Security vulnerability scanner
tools: Read, Grep, Bash
model: opus
---

You are a security expert specializing in vulnerability detection.

[System prompt details...]
```

**Plugin agents**:
- Automatically available when plugin enabled
- Can be invoked explicitly: `"Use the security-reviewer agent from security-plugin"`
- Managed through `/agents` interface

### 3. Hooks

Hooks respond to Claude Code lifecycle events.

**Location**: `hooks/hooks.json` or inline in plugin.json

**Format**:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/report.sh"
          }
        ]
      }
    ]
  }
}
```

**Plugin hooks**:
- Merged with user/project hooks
- Multiple hooks can respond to same event
- Use `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths

### 4. MCP Servers

MCP servers connect to external tools and data sources.

**Location**: `.mcp.json` or inline in plugin.json

**Format**:
```json
{
  "mcpServers": {
    "plugin-database": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "DB_PATH": "${CLAUDE_PLUGIN_ROOT}/data"
      }
    },
    "plugin-api": {
      "command": "npx",
      "args": ["@company/mcp-server", "--plugin-mode"],
      "cwd": "${CLAUDE_PLUGIN_ROOT}"
    }
  }
}
```

**Integration**:
- Available when plugin enabled
- Configurable through `/mcp` command
- Use environment variables for secrets

## Creating Plugins

### Step 1: Initialize Structure
```bash
mkdir my-plugin
cd my-plugin
mkdir -p commands agents hooks
```

### Step 2: Create Manifest
```bash
cat > plugin.json << 'EOF'
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My awesome plugin",
  "author": {
    "name": "Your Name"
  }
}
EOF
```

### Step 3: Add Components

Add commands:
```bash
cat > commands/hello.md << 'EOF'
---
description: Say hello
---

Say hello to: $ARGUMENTS
EOF
```

Add agents:
```bash
cat > agents/helper.md << 'EOF'
---
name: helper
description: General purpose helper
---

You are a helpful assistant.
EOF
```

### Step 4: Test Locally

```bash
# Link plugin locally for testing
/plugin add local-dev --source local --path /path/to/my-plugin

# Test command
/hello world

# Test agent
"Use the helper agent"
```

### Step 5: Documentation
```bash
cat > README.md << 'EOF'
# My Plugin

Description of what the plugin does.

## Installation

\`\`\`bash
/plugin install my-plugin@marketplace
\`\`\`

## Commands

- `/command-name` - Description

## Agents

- `agent-name` - Description

## Configuration

[Setup instructions]
EOF
```

## Environment Variables

### ${CLAUDE_PLUGIN_ROOT}
Absolute path to plugin directory. Use for plugin-relative paths.

**Example**:
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh"
      }]
    }]
  }
}
```

## Plugin Configuration

### settings.json
```json
{
  "enabledPlugins": {
    "plugin-name@marketplace-name": true,
    "another-plugin@marketplace": false
  },
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": "github",
      "repo": "company/claude-plugins"
    }
  }
}
```

### Scope Levels

1. **User** (`~/.claude/settings.json`)
   - Personal plugin preferences
   - Always available

2. **Project** (`.claude/settings.json`)
   - Project-specific plugins
   - Shared with team

3. **Local** (`.claude/settings.local.json`)
   - Per-machine overrides
   - Not committed

## Plugin Marketplaces

### What are Marketplaces?

Marketplaces are catalogs of available plugins that provide:
- **Centralized discovery**: Browse plugins from multiple sources
- **Version management**: Track and update plugins
- **Team distribution**: Share required plugins
- **Flexible sources**: Git repos, GitHub, local paths, npm

### Add Marketplace
```bash
# GitHub marketplace
/plugin marketplace add user-or-org/repo-name

# Example
/plugin marketplace add company/claude-plugins
```

### Marketplace Format

**Location**: `.claude-plugin/marketplace.json` in repository

**Structure**:
```json
{
  "name": "team-tools",
  "version": "1.0.0",
  "description": "Team plugin marketplace",
  "plugins": {
    "code-formatter": {
      "source": "github",
      "repo": "team/formatter-plugin",
      "version": "2.1.0",
      "description": "Automatic code formatting"
    },
    "security-scanner": {
      "source": "git",
      "url": "https://internal.git/security-plugin.git",
      "version": "1.5.0",
      "description": "Security vulnerability scanner"
    },
    "local-tools": {
      "source": "local",
      "path": "/shared/plugins/tools",
      "version": "1.0.0",
      "description": "Local development tools"
    }
  }
}
```

### Plugin Sources

#### GitHub
```json
{
  "source": "github",
  "repo": "username/plugin-repo"
}
```

#### Git Repository
```json
{
  "source": "git",
  "url": "https://git.example.com/plugin.git"
}
```

#### Local Path
```json
{
  "source": "local",
  "path": "/absolute/path/to/plugin"
}
```

### Creating a Marketplace

1. **Create repository**:
```bash
mkdir claude-plugins
cd claude-plugins
mkdir .claude-plugin
```

2. **Create marketplace.json**:
```bash
cat > .claude-plugin/marketplace.json << 'EOF'
{
  "name": "my-marketplace",
  "version": "1.0.0",
  "description": "My plugin marketplace",
  "plugins": {}
}
EOF
```

3. **Add plugins**:
```json
{
  "plugins": {
    "plugin-name": {
      "source": "github",
      "repo": "user/plugin-repo",
      "version": "1.0.0",
      "description": "Plugin description",
      "category": "productivity",
      "keywords": ["keyword1", "keyword2"]
    }
  }
}
```

4. **Publish**:
```bash
git add .
git commit -m "Initial marketplace"
git push origin main
```

5. **Share**:
```bash
# Users add with:
/plugin marketplace add username/claude-plugins
```

## Plugin Management

### List Plugins
```bash
/plugin list
```

Shows:
- Installed plugins
- Enabled/disabled status
- Version information
- Marketplace source

### Enable/Disable
```bash
/plugin
# Select plugin
# Toggle enabled state
```

**Why disable?**
- Reduce system prompt context
- Temporarily remove functionality
- Test without uninstalling

### Update Plugin
```bash
/plugin update [plugin-name]
/plugin update --all
```

### Remove Plugin
```bash
/plugin remove [plugin-name]
```

### Inspect Plugin
```bash
/plugin get [plugin-name]
```

Shows:
- Configuration
- Components (commands, agents, hooks, MCP)
- File paths
- Enabled state

## Advanced Features

### Conditional Loading

Plugins can use hooks to load conditionally:

```json
{
  "hooks": {
    "SessionStart": [{
      "hooks": [{
        "type": "command",
        "command": "if [ -f .use-plugin ]; then exit 0; else exit 1; fi"
      }]
    }]
  }
}
```

### Plugin Dependencies

Document dependencies in README:

```markdown
## Dependencies

This plugin requires:
- MCP Server: GitHub
- Environment: `GITHUB_TOKEN`
- Other Plugin: `base-tools@team`
```

### Plugin Composition

Create meta-plugins that bundle others:

```json
{
  "name": "full-stack-suite",
  "description": "Complete full-stack development suite",
  "dependencies": [
    "frontend-tools@marketplace",
    "backend-tools@marketplace",
    "testing-suite@marketplace"
  ]
}
```

## Best Practices

### Plugin Design
- **Single responsibility**: One plugin, one purpose
- **Clear naming**: Descriptive plugin and component names
- **Good documentation**: README with examples
- **Version semantically**: Follow semver (X.Y.Z)
- **Test thoroughly**: Test all components before release

### Security
- **Review code**: Before installing third-party plugins
- **Minimize permissions**: Only necessary tools
- **Validate inputs**: In hooks and commands
- **Secure secrets**: Use environment variables
- **Audit regularly**: Check for updates and vulnerabilities

### Performance
- **Lazy loading**: Only load when needed
- **Efficient hooks**: Fast execution (< 1s)
- **Cache results**: When appropriate
- **Minimal tools**: Only what's needed

### Team Collaboration
- **Version control**: Track plugin configurations
- **Document setup**: Clear installation instructions
- **Share marketplaces**: Team-wide plugin distribution
- **Code review**: Review plugin changes
- **Communicate updates**: Notify team of plugin changes

## Debugging Plugins

### Verbose Mode
```bash
claude --verbose
```

### Check Plugin Status
```bash
/plugin get [plugin-name]
```

### Test Components

**Test command**:
```bash
/command-name test-argument
```

**Test agent**:
```bash
"Use [agent-name] agent to [task]"
```

**Test hook**:
```bash
# Enable debug mode
claude --debug

# Trigger hook event
# Check execution in verbose output
```

### Common Issues

**Plugin not loaded**:
- Check `enabledPlugins` in settings
- Verify plugin.json is valid
- Check file paths exist
- Review marketplace configuration

**Command not found**:
- Ensure plugin is enabled
- Check commands path in plugin.json
- Verify Markdown file exists
- Try `/help` to see available commands

**Agent not available**:
- Check agents path in plugin.json
- Verify agent file format
- Use `/agents` to list available
- Check for naming conflicts

**Hook not firing**:
- Review hook configuration
- Check matcher patterns
- Verify script is executable
- Enable debug mode

## Example Plugins

### Security Plugin
```json
{
  "name": "security-tools",
  "version": "1.0.0",
  "description": "Security scanning and audit tools",
  "commands": ["./commands/scan.md", "./commands/audit.md"],
  "agents": ["./agents/security-auditor.md"],
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/check-dangerous.sh"
      }]
    }]
  }
}
```

### Documentation Plugin
```json
{
  "name": "doc-tools",
  "version": "1.0.0",
  "description": "Documentation generation tools",
  "commands": [
    "./commands/docs/generate.md",
    "./commands/docs/update.md"
  ],
  "agents": ["./agents/doc-writer.md"],
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/update-docs.sh"
      }]
    }]
  }
}
```

### Testing Plugin
```json
{
  "name": "test-suite",
  "version": "1.0.0",
  "description": "Automated testing tools",
  "commands": [
    "./commands/test.md",
    "./commands/coverage.md"
  ],
  "agents": [
    "./agents/test-engineer.md",
    "./agents/test-reviewer.md"
  ],
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/run-tests.sh"
      }]
    }]
  }
}
```

## Community Resources

### Official Examples
- PR review plugin (Anthropic)
- Security guidance plugin (Anthropic)
- Claude Agent SDK plugin (Anthropic)
- Plugin creator meta-plugin (Anthropic)

### Community Marketplaces
- Dan Ávila's marketplace: DevOps, docs, PM, testing
- Seth Hobson's repository: 80+ specialized agents
- awesome-claude-code (GitHub): Curated plugin list

### Finding Plugins
- Search GitHub: "claude-code plugin"
- Community Discord
- r/ClaudeAI subreddit
- awesome-claude-code repository

## Related Documentation
- [Main documentation](README.md)
- [Agents Documentation](agent.md)
- [Hooks Documentation](hook.md)
- [MCP Documentation](mcp.md)
- [Commands Documentation](commands.md)
- [Settings Documentation](settings.md)
