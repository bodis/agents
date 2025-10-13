# Claude Code - Main Documentation

## Overview

Claude Code is an agentic coding tool that lives in your terminal, understands your codebase, and helps you code faster through natural language commands. Created by Anthropic, it integrates directly with your development environment.

## Installation

```bash
# Install Claude Code
npm install -g @anthropic-ai/claude-code

# Navigate to your project
cd your-awesome-project

# Start coding with Claude
claude

# You'll be prompted to log in on first use
```

### Prerequisites
- Node.js 18 or newer
- A Claude.ai (recommended) or Claude Console account

### Update Claude Code
```bash
claude update
```

## What Claude Code Does

### Core Capabilities
- **Build features from descriptions**: Tell Claude what you want to build in plain English. It will make a plan, write the code, and ensure it works.
- **Debug and fix issues**: Describe a bug or paste an error message. Claude Code will analyze your codebase, identify the problem, and implement a fix.
- **Navigate any codebase**: Ask anything about your team's codebase. Claude Code maintains awareness of your entire project structure.
- **Automate tedious tasks**: Fix lint issues, resolve merge conflicts, and write release notes.

### Why Developers Love Claude Code
- **Works in your terminal**: Not another chat window or IDE. Meets you where you work.
- **Takes action**: Directly edits files, runs commands, and creates commits.
- **Unix philosophy**: Composable and scriptable. `tail -f app.log | claude -p "Slack me if you see anomalies"` works.
- **Enterprise-ready**: Use the Claude API, or host on AWS or GCP.

## Core Concepts

### Models
Claude Code supports multiple models from the Claude 4 family:
- **Claude Sonnet 4.5**: Default model, smartest and most efficient for everyday use
- **Claude Opus 4.1**: Maximum capability with enhanced coding and debugging performance
- **Claude Haiku**: Fastest and most cost-effective for simple tasks

### Context Management
- **CLAUDE.md files**: Special files that Claude automatically pulls into context when starting a conversation
- **Context window**: The maximum amount of text a model can consider at once
- **Compaction**: Automatic summarization when approaching context limits

### Permission Modes
- **Default mode**: Claude asks permission for each action
- **Auto-accept mode**: Toggle with Shift+Tab to let Claude work autonomously
- **Plan mode**: Toggle with Shift+Tab twice for structured planning without execution
- **Skip permissions**: Use `--dangerously-skip-permissions` flag (use with caution)

## Project Structure

### Configuration Files
```
your-project/
├── .claude/
│   ├── settings.json          # Project-level settings (committed)
│   ├── settings.local.json    # Local settings (not committed)
│   ├── commands/              # Custom slash commands
│   ├── agents/                # Custom subagents
│   └── hooks/                 # Hook scripts
├── CLAUDE.md                  # Project documentation for Claude
└── .mcp.json                  # Project-level MCP server config
```

### User-Level Files
```
~/.claude/
├── settings.json              # User-level settings
├── commands/                  # User-level commands
├── agents/                    # User-level agents
└── projects/                  # Session transcripts
```

## CLAUDE.md Files

CLAUDE.md is a special file that Claude automatically pulls into context. It should contain:
- Project architecture overview
- Dependencies and tech stack
- Coding conventions and style guides
- Common commands (build, test, lint)
- Development workflows
- Known issues or gotchas

Example:
```markdown
# Project: E-commerce Platform

## Architecture
- Frontend: React 18 with Vite
- Backend: Node.js/Express
- Database: PostgreSQL
- State: Redux Toolkit

## Commands
- `npm run dev` - Start development server
- `npm test` - Run Jest tests
- `npm run lint` - Run ESLint

## Conventions
- Use functional components
- Follow Airbnb style guide
- Place tests in __tests__ directory
```

## Basic Usage

### Interactive REPL
```bash
# Start interactive session
claude

# Continue previous conversation
claude -c

# Resume specific session
claude -r <session-id> "Continue working on that feature"
```

### One-Shot Commands
```bash
# Execute and exit
claude -p "Explain this function"

# Pipe content
cat logs.txt | claude -p "Summarize errors"

# Process files
claude -p "Add tests to all components in src/components"
```

### File References
```bash
# Use @ to reference files
claude "Review @src/auth.ts for security issues"

# Use tab completion for file paths
claude "Fix the bug in @<TAB>
```

### Extended Thinking
Use specific phrases to trigger deeper analysis:
- `think` - Basic extended thinking
- `think hard` - More computation time
- `think harder` - Even more analysis
- `ultrathink` - Maximum thinking budget

## Keyboard Shortcuts

### Navigation & Control
- **Escape**: Stop Claude during any phase
- **Escape twice**: Jump back in history to edit previous prompt
- **Shift+Tab**: Toggle auto-accept mode
- **Shift+Tab twice**: Toggle plan mode
- **Ctrl+C**: Exit Claude Code
- **Ctrl+R**: Search command history
- **Ctrl+B**: Move Bash command to background

### File Operations
- **Shift + Drag**: Reference file in prompt (don't open in tab)
- **Ctrl+V**: Paste images from clipboard (not Command+V)

## Environment Variables

```bash
# Set model
export ANTHROPIC_MODEL="claude-sonnet-4-5-20250929"

# Set API key (if not using Claude.ai login)
export ANTHROPIC_API_KEY="your-api-key"

# Set MCP output token limit
export MAX_MCP_OUTPUT_TOKENS=50000

# Project directory (available in hooks)
$CLAUDE_PROJECT_DIR
```

## Common Workflows

### Feature Development
1. Ask Claude to make a plan: `think about how to implement user authentication`
2. Review and approve the plan
3. Have Claude implement: `implement the authentication system`
4. Ask Claude to commit and create PR

### Debugging
1. Paste error message: `I'm getting this error: [paste error]`
2. Claude analyzes the codebase and identifies root cause
3. Claude implements fix
4. Claude verifies the fix works

### Code Review
1. `Review the changes in my last commit`
2. Claude analyzes changes for:
   - Code quality issues
   - Security vulnerabilities
   - Performance concerns
   - Best practice violations

### Refactoring
1. Use plan mode: `think hard about refactoring the payment module`
2. Review comprehensive strategy
3. Implement in phases with checkpoints

## Best Practices

### Context Optimization
- Use `/clear` frequently between tasks to reset context
- Store common patterns in CLAUDE.md
- Use specific file references with @
- Keep CLAUDE.md files focused and current

### Collaboration
- Be an active collaborator, don't just supervise
- Course-correct Claude at any time
- Provide thorough explanations upfront
- Use checkpoints for complex tasks

### Safety
- Review hooks before enabling them
- Use plan mode for unfamiliar tasks
- Keep sensitive files in .gitignore
- Use --dangerously-skip-permissions only in containers

## Advanced Features

### Multi-Directory Support
```bash
claude --add-dir ../shared-lib ../common-utils
```

### Output Formats
```bash
# JSON output for scripting
claude -p "List all TODO comments" --output-format json

# Stream JSON
claude -p "query" --output-format stream-json
```

### Custom System Prompts
```bash
claude --append-system-prompt "Always use TypeScript strict mode"
```

### Max Turns Limit
```bash
claude -p --max-turns 3 "refactor this file"
```

## Security Considerations

### What Claude Code Can Access
- Files in current directory and subdirectories
- Environment variables in your shell
- Network access (if not blocked)
- Commands you've allowed via permissions

### What It Cannot Do Without Permission
- Execute shell commands
- Edit files
- Read sensitive files (.env, package-lock.json)
- Access files outside project directory

### Best Practices
- Review permissions carefully
- Use hooks to block dangerous operations
- Run in containers for untrusted tasks
- Keep API keys in environment variables, not code

## Troubleshooting

### Common Issues
- **Claude not responding**: Check network connection, API status
- **Permission errors**: Review allowed tools in `/permissions`
- **Context overflow**: Use `/clear` or `/compact`
- **Hooks not firing**: Review hooks with `/hooks` command

### Debug Mode
```bash
claude --verbose  # Detailed logging
claude --debug    # Hook execution details
```

### Getting Help
```bash
/help          # List all commands
/doctor        # Diagnose issues
/bug           # Report bug
```

## Resources

- Official Documentation: https://docs.claude.com/en/docs/claude-code
- GitHub Repository: https://github.com/anthropics/claude-code
- Community Discord: Join Claude Developers Discord
- Best Practices: https://www.anthropic.com/engineering/claude-code-best-practices

## Related Documentation Files

For detailed information on specific topics, see:
- [CLI Reference](cli.md)
- [MCP Documentation](mcp.md)
- [Hooks Documentation](hook.md)
- [Agents Documentation](agent.md)
- [Plugins Documentation](plugin.md)
- [Settings Documentation](settings.md)
- [Commands Documentation](commands.md)
- [Status Line Documentation](status-line.md)
