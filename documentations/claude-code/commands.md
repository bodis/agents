# Claude Code - Commands Documentation

Complete reference for slash commands and custom command creation in Claude Code.

## Overview

Commands in Claude Code come in two types:
1. **Built-in slash commands**: System commands prefixed with `/`
2. **Custom commands**: User-defined shortcuts in Markdown files

## Built-in Slash Commands

### Session Management

#### /clear
Reset the conversation history and context.

```bash
/clear
```

**When to use**:
- Starting a new task
- Context is too large
- Switching topics
- After completing a feature

**Effect**:
- Clears conversation history
- Resets context window
- Preserves settings and configuration
- Maintains tool permissions

**Best practice**: Use frequently between tasks to maintain focus and reduce token usage.

#### /compact
Summarize conversation to reduce context usage.

```bash
/compact [instructions]
```

**Examples**:
```bash
/compact

/compact Focus on preserving authentication implementation and database schema

/compact Keep only the last 5 messages about the API design
```

**When to use**:
- Approaching context limit
- Long conversations
- Before complex tasks
- To preserve specific information

**Note**: Claude Sonnet 4.5 has 1 million token context window, but compaction still helps maintain focus.

### Model Management

#### /model
Switch between Claude models.

```bash
/model
```

**Interactive menu**:
- Claude Opus 4.1 - Maximum capability
- Claude Sonnet 4.5 - Balanced (default)
- Claude Haiku - Fastest, most cost-effective

**Via prompt**:
```bash
"Switch to Opus"
"Use Sonnet for this"
```

**Model selection tips**:
- **Opus**: Complex architecture, security audits, critical fixes
- **Sonnet**: General coding, most tasks (default)
- **Haiku**: Simple tasks, iteration, cost optimization

### Configuration

#### /config
Open interactive settings interface.

```bash
/config
```

**Tabs**:
- **General**: Model, permissions, directories
- **Tools**: Allowed/disallowed tools
- **Agents**: Subagent management
- **MCP**: MCP server configuration
- **Plugins**: Plugin management
- **Hooks**: Hook configuration
- **Advanced**: Status line, output styles

**Use cases**:
- View current configuration
- Modify settings visually
- Check merged settings from all levels

#### /permissions
Manage tool permissions.

```bash
/permissions
```

**Actions**:
- View current allowed/disallowed tools
- Add tools to allowlist
- Add tools to blocklist
- Add domains to web_fetch allowlist
- View permission decisions

**Examples**:
```bash
# Interactive
/permissions
# Select "Add allowed tool"
# Choose from list or enter pattern

# Via prompt
"Allow all git commands"
"Block rm commands"
```

#### /init
Create CLAUDE.md file for project.

```bash
/init
```

**What it does**:
- Creates `CLAUDE.md` in project root
- Documents project architecture
- Lists dependencies
- Documents common commands
- Adds coding conventions

**Generated content**:
```markdown
# Project: [Project Name]

## Architecture
[Auto-detected structure]

## Dependencies
[From package.json, requirements.txt, etc.]

## Commands
- Build: [detected]
- Test: [detected]
- Lint: [detected]

## Conventions
[Language-specific defaults]
```

**Customization**: Edit CLAUDE.md after generation to add project-specific information.

### Agent Management

#### /agents
Manage subagents.

```bash
/agents
```

**Actions**:
- List all agents (project + user)
- Create new agent
- Edit existing agent
- Delete agent
- Configure agent tools
- Test agent invocation

**Interface**:
```
Available Agents:
1. code-reviewer (project)
2. test-runner (project)
3. security-auditor (user)
4. data-scientist (user)

Actions:
[C] Create new agent
[E] Edit agent
[D] Delete agent
[T] Configure tools
[Q] Quit
```

See Agents Documentation for complete reference.

### Plugin Management

#### /plugin
Manage plugins.

```bash
/plugin
```

**Actions**:
- List installed plugins
- Install new plugin
- Enable/disable plugin
- Update plugin
- Remove plugin
- Manage marketplaces

**Subcommands**:
```bash
/plugin list
/plugin install [name]@[marketplace]
/plugin remove [name]
/plugin update [name]
/plugin marketplace add [repo]
```

**Examples**:
```bash
/plugin install security-tools@company
/plugin list
/plugin update --all
```

See Plugins Documentation for complete reference.

### MCP Management

#### /mcp
Manage MCP servers.

```bash
/mcp
```

**Actions**:
- List configured MCP servers
- Add new MCP server
- Remove MCP server
- Get server details
- Import from Claude Desktop
- Test server connection

**Subcommands**:
```bash
/mcp list
/mcp add [name] [options] -- [command]
/mcp remove [name]
/mcp get [name]
/mcp import
```

**Examples**:
```bash
/mcp add github -s user -e GITHUB_TOKEN=$TOKEN -- npx -y @modelcontextprotocol/server-github

/mcp list

/mcp remove github
```

See MCP Documentation for complete reference.

### Hook Management

#### /hooks
Configure hooks.

```bash
/hooks
```

**Actions**:
- List configured hooks
- Add new hook
- Edit hook
- Delete hook
- Test hook execution

**Interactive flow**:
1. Select event type (PreToolUse, PostToolUse, etc.)
2. Choose tool matcher (if applicable)
3. Enter command to execute
4. Set timeout (optional)
5. Configuration saved to settings.json

**Example**:
```bash
/hooks
# Select "PreToolUse"
# Matcher: "Bash"
# Command: "echo 'Running command' >> /tmp/log.txt"
# Saves to settings.json
```

**Note**: Direct edits to hooks in settings.json require review in `/hooks` menu to take effect (security feature).

See Hooks Documentation for complete reference.

### Status Line

#### /statusline
Configure custom status line.

```bash
/statusline [instructions]
```

**Examples**:
```bash
/statusline

/statusline show the model name in orange

/statusline add git branch and current time
```

**What it does**:
- Creates/updates status line script
- By default, reproduces terminal prompt
- Can customize with specific instructions
- Saves configuration to settings.json

See Status Line Documentation for complete reference.

### Terminal Setup

#### /terminal-setup
Install terminal key bindings.

```bash
/terminal-setup
```

**Installs**:
- Shift+Enter for newlines (iTerm2, VS Code)
- Proper terminal integration
- Clipboard support configuration

**Supported terminals**:
- iTerm2 (macOS)
- VS Code integrated terminal
- Standard terminals with ANSI support

### Debugging & Help

#### /help
List all available commands.

```bash
/help
```

**Output**:
```
Available commands:
/clear          - Clear conversation history
/model          - Switch Claude model
/config         - Open settings
/permissions    - Manage tool permissions
/agents         - Manage subagents
/plugin         - Manage plugins
/mcp            - Manage MCP servers
/hooks          - Configure hooks
...
```

#### /doctor
Diagnose issues with Claude Code.

```bash
/doctor
```

**Checks**:
- Installation status
- Configuration validity
- MCP server connectivity
- Hook execution
- File permissions
- API connectivity

**Output example**:
```
✓ Claude Code installed correctly
✓ API connection working
✓ Configuration valid
✗ MCP server 'github' not responding
  → Check GITHUB_TOKEN environment variable
✓ Hooks configured correctly
```

#### /bug
Report a bug.

```bash
/bug
```

**What it does**:
- Opens bug report interface
- Collects diagnostic information
- Submits to Claude Code team
- Includes session context (if permitted)

### Vim Mode

#### /vim
Toggle Vim-style editing.

```bash
/vim
```

**Enables**:
- Vim key bindings for input
- Modal editing (normal/insert)
- Vim navigation commands

**Modes**:
- Normal mode: ESC
- Insert mode: i, a, o
- Visual mode: v

**Disable**: `/vim` again to toggle off

## Custom Commands

### What are Custom Commands?

Custom commands are user-defined shortcuts stored as Markdown files. They:
- Start with `/` in conversation
- Contain prompts and instructions for Claude
- Support arguments via `$ARGUMENTS`
- Can be project or user-level
- Are shared via version control (project-level)

### Command Locations

#### User Commands
```bash
~/.claude/commands/
```

Available in all projects.

#### Project Commands
```bash
.claude/commands/
```

Shared with team via git.

### Command File Format

```markdown
---
description: Brief description for help menu
argument-hint: "<what to provide>"
---

# Command Instructions

Task description using $ARGUMENTS

[Detailed instructions for Claude]
```

### Creating Commands

#### Method 1: Ask Claude
```bash
"Create a /review command that performs comprehensive code review"
```

Claude will:
1. Create `.claude/commands/review.md`
2. Write command prompt
3. Test the command

#### Method 2: Manual Creation
```bash
mkdir -p .claude/commands
cat > .claude/commands/review.md << 'EOF'
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

Provide specific file:line references for issues.
EOF
```

#### Method 3: Plugin Commands
Install commands via plugins:
```bash
/plugin install code-review-suite@marketplace
```

### Command Examples

#### Code Review
```markdown
---
description: Comprehensive code review
argument-hint: "<scope>"
---

Perform a comprehensive code review of: $ARGUMENTS

## Review Focus
1. **Security**: Authentication, authorization, input validation, SQL injection, XSS
2. **Quality**: Naming, structure, complexity, documentation, error handling
3. **Performance**: N+1 queries, memory leaks, algorithmic complexity
4. **Testing**: Test coverage, test quality, edge cases
5. **Architecture**: SOLID principles, design patterns, maintainability

## Output Format
### Critical Issues
[file:line] - Description and fix

### High Priority
[file:line] - Description and suggestion

### Suggestions
[file:line] - Improvement ideas
```

**Usage**:
```bash
/review src/auth/
/review @src/payment.ts
```

#### Test Generation
```markdown
---
description: Generate comprehensive tests
argument-hint: "<files to test>"
---

Generate comprehensive tests for: $ARGUMENTS

## Test Requirements
- Use Jest and React Testing Library (for React)
- Place in `__tests__` directory
- Mock external dependencies
- Test all major functionality
- Include edge cases and error scenarios
- Test user interactions
- Verify state changes
- Aim for >80% code coverage

## Test Structure
- Clear test descriptions
- AAA pattern (Arrange, Act, Assert)
- One assertion per test
- Proper cleanup in afterEach
```

**Usage**:
```bash
/test src/components/Button.tsx
/test all components in src/auth/
```

#### Documentation Generator
```markdown
---
description: Generate API documentation
argument-hint: "<files>"
---

Generate comprehensive API documentation for: $ARGUMENTS

## Documentation Requirements
- Function signatures with type information
- Parameter descriptions with types and constraints
- Return value documentation
- Usage examples
- Error cases and handling
- Related functions
- Performance notes (if relevant)

## Format
Use JSDoc/TSDoc format for inline documentation.
Create separate README.md for module overview.
```

#### Debugging Assistant
```markdown
---
description: Debug issue step-by-step
argument-hint: "<error description>"
---

Debug the following issue: $ARGUMENTS

## Debug Process
1. Understand the error message and stack trace
2. Identify the root cause
3. Find all related code
4. Analyze the bug's context
5. Propose 2-3 possible fixes
6. Recommend the best approach
7. Implement the fix
8. Add tests to prevent regression

Ask questions if you need more information.
```

#### Refactoring Command
```markdown
---
description: Refactor code for better quality
argument-hint: "<files or scope>"
---

Refactor the following code: $ARGUMENTS

## Refactoring Goals
1. Improve readability and maintainability
2. Reduce complexity
3. Remove code duplication
4. Apply design patterns appropriately
5. Improve naming
6. Add missing error handling
7. Optimize performance where beneficial

## Constraints
- Maintain existing functionality
- Keep tests passing
- Preserve API contracts
- Document breaking changes

## Output
Show before/after comparisons for significant changes.
```

#### Deploy Command
```markdown
---
description: Deploy to environment
argument-hint: "<staging|production>"
---

Deploy to: $ARGUMENTS

## Deployment Checklist
1. Run all tests
2. Check for uncommitted changes
3. Verify environment variables
4. Build production bundle
5. Run database migrations (if needed)
6. Deploy to environment
7. Verify deployment
8. Update deployment log
9. Notify team

Stop if any step fails.
```

### Command with Subfolders

```bash
.claude/commands/
├── core/
│   ├── review.md
│   └── test.md
├── deploy/
│   ├── staging.md
│   └── production.md
└── docs/
    └── generate.md
```

**Usage**:
```bash
/core/review src/
/deploy/staging
/docs/generate
```

### Command Arguments

#### Using $ARGUMENTS
```markdown
Process these files: $ARGUMENTS
```

**Invocation**:
```bash
/command src/file1.ts src/file2.ts
```

**Result**: Claude sees "Process these files: src/file1.ts src/file2.ts"

#### Multiple Argument Variables (Not Supported)
Commands only support `$ARGUMENTS` as a single variable.

For complex argument parsing, use a wrapper script:
```markdown
---
description: Complex command
---

Run this command with arguments: $ARGUMENTS

Parse arguments:
- First word: action (build/test/deploy)
- Second word: environment
- Remaining: files

Then execute the appropriate workflow.
```

### Command Best Practices

#### Clear Descriptions
```markdown
---
description: Generate comprehensive test suite with Jest
argument-hint: "<files to test>"
---
```

#### Structured Instructions
```markdown
## Step 1: Analysis
[Instructions]

## Step 2: Implementation
[Instructions]

## Step 3: Verification
[Instructions]
```

#### Include Examples
```markdown
## Example Usage
Given the following component:
```typescript
function Button({ onClick, label }) { ... }
```

Generate tests that cover:
- Click events
- Disabled state
- Accessibility
```

#### Define Output Format
```markdown
## Output Format
```typescript
// Test file structure
describe('ComponentName', () => {
  describe('Feature', () => {
    it('should do something', () => {
      // Test implementation
    });
  });
});
```
```

#### Use Context
```markdown
Reference the project's CLAUDE.md for:
- Testing framework (Jest/Vitest)
- Style guide
- File naming conventions
```

### Command Organization

#### Logical Grouping
```bash
.claude/commands/
├── testing/
│   ├── unit.md
│   ├── integration.md
│   └── e2e.md
├── docs/
│   ├── api.md
│   ├── readme.md
│   └── changelog.md
└── quality/
    ├── review.md
    ├── lint.md
    └── security.md
```

#### Team Standards
Create commands for team workflows:
```markdown
# .claude/commands/team/pr-checklist.md
---
description: Pre-PR checklist
---

Run our standard PR checklist:

1. Run linter: `npm run lint`
2. Run tests: `npm test`
3. Check coverage: `npm run coverage`
4. Update documentation
5. Review your own changes
6. Write meaningful commit message
7. Update CHANGELOG.md
8. Request review from 2 team members

Verify each step passes before proceeding.
```

### Sharing Commands

#### Via Git (Project Commands)
```bash
git add .claude/commands/
git commit -m "Add custom commands for team"
git push
```

Team members get commands automatically when they pull.

#### Via Plugins
Package commands as plugins:
```json
{
  "name": "my-commands",
  "commands": [
    "./commands/review.md",
    "./commands/test.md"
  ]
}
```

See Plugins Documentation for details.

#### Via Documentation
Share in README:
```markdown
## Custom Commands

Copy these to `.claude/commands/`:
- [review.md](link)
- [test.md](link)
```

## Command Tips & Tricks

### Tab Completion
Commands support tab completion:
```bash
/rev<TAB>    # Completes to /review
/test/u<TAB> # Completes to /test/unit
```

### File References
Commands can reference files:
```bash
/review @src/auth.ts @src/user.ts
```

### Chaining Commands
Use multiple commands in sequence:
```bash
/test src/auth.ts
# Wait for completion
/review src/auth.ts
```

### Command History
Use up arrow to access previous commands:
```bash
↑ # Shows last command
↑↑ # Shows command before that
```

## Troubleshooting

### Command Not Found
- Check file exists in `.claude/commands/` or `~/.claude/commands/`
- Verify filename ends with `.md`
- Restart Claude Code session
- Use `/help` to list available commands

### Command Not Working
- Check Markdown syntax
- Verify frontmatter format
- Test `$ARGUMENTS` substitution manually
- Review command output for errors

### Command Conflicts
If user and project commands have same name:
- Project commands take priority
- Rename one command
- Use subfolder to organize

## Related Documentation
- [Main Documentation](README.md)
- [Plugins Documentation](plugin.md)
- [Agents Documentation](agent.md)
- [Settings Documentation](settings.md)
- [CLI Reference](cli.md)
