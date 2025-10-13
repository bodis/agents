# Claude Code - Agents & Subagents Documentation

Complete reference for creating and managing specialized AI subagents in Claude Code.

## What are Subagents?

Subagents are specialized AI assistants that Claude Code can delegate tasks to. Each subagent:
- Has its own **dedicated context window** (separate from main conversation)
- Includes a **custom system prompt** that guides its behavior
- Can have **specific tool permissions** for security and focus
- Can use a **different model** (Sonnet, Opus, or Haiku)
- Operates **independently** and returns results to the main agent

### Benefits
- **Improved context management**: Separate context prevents pollution
- **Specialized expertise**: Task-specific configurations and prompts
- **Parallel workflows**: Multiple agents can work simultaneously
- **Better focus**: Keep main conversation high-level
- **Team collaboration**: Share agents across projects

## Core Concepts

### Automatic Delegation
Claude automatically invokes appropriate subagents based on:
- Task description in your prompt
- Agent's `description` field
- Current context and available tools
- Agent's expertise and configuration

### Context Isolation
Each subagent operates in its own context window:
- Prevents context pollution between tasks
- Allows larger tasks without overwhelming main agent
- Maintains focus on specific objectives
- Enables parallel execution

### Tool Permissions
Subagents can have restricted tool access:
- Inherit all tools by default
- Or specify exact tools needed
- Includes MCP server tools
- Configurable via `/agents` command

## Creating Subagents

### Method 1: Interactive Creation with /agents

```bash
# Open agent management interface
/agents

# Select "Create new agent"
# Follow prompts to configure:
# - Name
# - Description (when to use)
# - System prompt
# - Tools (optional)
# - Model (optional)
```

### Method 2: File-Based Creation

#### Project-Level Agents
```bash
# Create project agent
mkdir -p .claude/agents
touch .claude/agents/agent-name.md
```

#### User-Level Agents
```bash
# Create user agent
mkdir -p ~/.claude/agents
touch ~/.claude/agents/agent-name.md
```

### Method 3: CLI Flag (Inline Definition)

```bash
claude --agents '{
  "agent-name": {
    "description": "When to invoke this agent",
    "prompt": "System prompt defining behavior",
    "tools": ["Tool1", "Tool2"],
    "model": "sonnet"
  }
}'
```

## Agent File Format

Agents are defined in Markdown files with YAML frontmatter:

```markdown
---
name: agent-name
description: Brief description of when to use this agent
tools: Tool1, Tool2, Tool3  # Optional
model: sonnet               # Optional: sonnet, opus, haiku
---

# Agent System Prompt

You are a specialized agent for [specific purpose].

## Your Role
[Define the agent's role and expertise]

## When Invoked
[What triggers this agent]

## Key Practices
- [Best practice 1]
- [Best practice 2]
- [Best practice 3]

## Approach
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Output Format
[How to structure responses]
```

### Required Fields

| Field | Description |
|-------|-------------|
| `name` | Unique identifier for the agent |
| `description` | When this agent should be invoked (critical for automatic delegation) |

### Optional Fields

| Field | Default | Description |
|-------|---------|-------------|
| `tools` | All tools | Comma-separated list of allowed tools |
| `model` | Inherits | `sonnet`, `opus`, or `haiku` |

## Example Agents

### Code Reviewer
```markdown
---
name: code-reviewer
description: Expert code reviewer. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: opus
---

You are a senior code reviewer focusing on code quality, security, and best practices.

## When Invoked
- After significant code changes
- Before pull request creation
- When user requests code review
- Proactively when new code is written

## Review Focus
1. **Security**: Authentication, authorization, input validation
2. **Quality**: Naming, structure, complexity, documentation
3. **Performance**: Bottlenecks, memory leaks, optimizations
4. **Architecture**: SOLID principles, patterns, scalability

## Output Format
```

### Test Engineer
```markdown
---
name: test-runner
description: Use proactively to run tests and fix failures
tools: Bash, Read, Write, Edit
model: sonnet
---

You are a test automation expert. When you see code changes, proactively run the appropriate tests.

## Responsibilities
1. Identify which tests to run based on changes
2. Execute test suites
3. Analyze failures
4. Fix tests while preserving original intent
5. Ensure full test coverage

## Test Priority
- Unit tests for modified functions
- Integration tests for related features
- E2E tests for critical paths

## On Failure
1. Read error messages carefully
2. Understand root cause
3. Fix issue (code or test)
4. Re-run to verify
5. Report results
```

### Database Analyst
```markdown
---
name: data-scientist
description: Use proactively for data analysis tasks and queries
tools: Bash, Read, Write
model: sonnet
---

You are a data scientist specializing in SQL and BigQuery analysis.

## When Invoked
1. Understand the data analysis requirement
2. Write efficient SQL queries
3. Use BigQuery command line tools (bq) when appropriate
4. Analyze and summarize results
5. Present findings clearly

## Key Practices
- Write optimized SQL queries with proper filters
- Use appropriate aggregations and joins
- Include comments explaining complex logic
- Format results for readability
- Provide data-driven recommendations

## For Each Analysis
- Explain the query approach
- Document any assumptions
- Highlight key findings
- Suggest next steps based on data

Always ensure queries are efficient and cost-effective.
```

### Security Auditor
```markdown
---
name: security-auditor
description: Security vulnerability scanning and threat analysis
tools: Read, Grep, Bash
model: opus
---

You are a security expert specializing in vulnerability detection and threat analysis.

## Audit Scope
1. **Authentication & Authorization**: JWT, OAuth, sessions, RBAC
2. **Input Validation**: SQL injection, XSS, CSRF
3. **Data Protection**: Encryption, PII handling, secrets management
4. **Dependencies**: Known vulnerabilities, outdated packages
5. **Configuration**: Security headers, CORS, environment variables

## Analysis Process
1. Scan codebase systematically
2. Identify security patterns and anti-patterns
3. Assess risk level (Critical, High, Medium, Low)
4. Provide specific remediation steps
5. Reference security standards (OWASP, CWE)

## Report Format
### Critical Issues
[List with specific file:line, vulnerability, and fix]

### Recommendations
[Prioritized security improvements]
```

### Documentation Writer
```markdown
---
name: doc-writer
description: Technical documentation and API documentation generation
tools: Read, Write, Grep
model: sonnet
---

You are a technical writer specializing in developer documentation.

## Documentation Types
- README files
- API documentation
- Code comments
- Architecture docs
- User guides

## Writing Principles
- Clear and concise
- Code examples for APIs
- Diagrams for complex systems
- Consistent formatting
- Up-to-date information

## Structure
1. Overview/Purpose
2. Prerequisites
3. Installation/Setup
4. Usage examples
5. API reference
6. Troubleshooting
7. Contributing guidelines
```

## Invoking Agents

### Automatic Invocation
Claude automatically selects agents based on task and description:

```bash
# Claude will use code-reviewer agent if configured
"Review the authentication module for security issues"

# Claude will use test-runner agent if configured
"The tests are failing, please investigate"
```

### Explicit Invocation
```bash
# Explicitly request specific agent
"Use the security-auditor subagent to scan for vulnerabilities"

"Have the code-reviewer subagent look at my recent changes"

"Ask the data-scientist subagent to analyze the user metrics"
```

### Proactive Usage
Encourage agents to act proactively by including phrases in description:
- "Use PROACTIVELY after code changes"
- "MUST BE USED for security scans"
- "Automatically run after file edits"

## Managing Agents

### List Agents
```bash
/agents
```

Shows:
- All configured agents
- Project vs user-level
- Tool permissions
- Model assignments

### Edit Agent
```bash
# Interactive editing
/agents
# Select agent
# Choose "Edit"

# Or edit file directly
code .claude/agents/agent-name.md
```

### Delete Agent
```bash
# Interactive deletion
/agents
# Select agent
# Choose "Delete"

# Or delete file
rm .claude/agents/agent-name.md
```

### Configure Tools
```bash
# Interactive tool selection
/agents
# Select agent
# Choose "Configure tools"
# Select from available tools
```

The `/agents` command shows all available tools including:
- Claude Code built-in tools
- MCP server tools
- Custom tools

## Tool Configuration

### Default (Inherit All Tools)
```markdown
---
name: my-agent
description: General purpose agent
---
```

Inherits:
- All Claude Code tools
- All configured MCP tools
- Full capabilities

### Restricted Tools
```markdown
---
name: my-agent
description: Read-only agent
tools: Read, Grep, Glob
---
```

Only has access to specified tools.

### MCP Tools
Agents can access MCP tools when tools field is omitted:

```markdown
---
name: github-agent
description: GitHub operations agent
# No tools field = inherits all including MCP
---
```

If tools are specified, include MCP tools explicitly:
```markdown
---
name: github-agent
description: GitHub operations agent
tools: Read, mcp__github__create_issue, mcp__github__search_repositories
---
```

## Model Selection

### Available Models

| Model | Use Case | Cost | Speed |
|-------|----------|------|-------|
| `sonnet` | General purpose, balanced | Medium | Fast |
| `opus` | Complex reasoning, quality | High | Slower |
| `haiku` | Simple tasks, speed | Low | Fastest |

### Configuration
```markdown
---
name: my-agent
description: Agent description
model: opus
---
```

### Best Practices
- Use `opus` for: Code review, architecture, security
- Use `sonnet` for: General coding, testing, documentation
- Use `haiku` for: Simple tasks, fast iteration, cost savings

## Agent Scope

### Project-Level (.claude/agents/)
- Shared with team
- Version controlled
- Project-specific expertise
- Consistent across team members

### User-Level (~/.claude/agents/)
- Personal agents
- Not shared
- Personal workflows
- Available in all projects

### Priority
1. **Project agents** (highest)
2. **User agents**
3. **CLI-defined agents** (--agents flag)

When naming conflicts occur, project-specific agents override global ones.

## Multi-Agent Workflows

### Parallel Development
```bash
"Implement user authentication:
- Use backend-architect to design API
- Use frontend-developer to create UI
- Use test-engineer to write tests
- Use security-auditor to review implementation"
```

### Sequential Workflows
```bash
"Refactor payment module:
1. Use code-reviewer to analyze current code
2. Use architect to design improvements
3. Use implementer to make changes
4. Use test-runner to verify functionality"
```

### Agent Coordination
Agents can reference each other's work:
```bash
"Use debugger agent to investigate error,
then have test-engineer create regression test"
```

## Advanced Patterns

### Agent Templates

Create reusable agent templates:

```bash
.claude/agent-templates/
├── backend-expert.md
├── frontend-expert.md
├── qa-specialist.md
└── devops-engineer.md
```

Copy and customize for specific projects.

### Agent Marketplaces

Share agents via plugins and marketplaces:
- GitHub repositories
- Team plugin marketplaces
- Community collections

### Agent Composition

Combine multiple agents for complex tasks:

```markdown
---
name: full-stack-developer
description: Complete feature development
---

You coordinate multiple specialized agents:
- backend-architect for API design
- frontend-developer for UI
- test-engineer for testing
- security-auditor for security
- doc-writer for documentation

Delegate tasks appropriately and synthesize results.
```

## Best Practices

### Description Field
- **Be specific** about when to invoke
- **Include keywords** relevant to tasks
- **Mention "proactively"** for automatic usage
- **Describe scope** clearly

Good: "Use proactively after TypeScript file edits to run type checking"
Bad: "TypeScript helper"

### System Prompts
- **Define role clearly**
- **Specify approach** and methodology
- **Include examples** when helpful
- **Set quality standards**
- **Define output format**

### Tool Selection
- **Minimize tools** for focused agents
- **Security**: Restrict tools for security-sensitive agents
- **Performance**: Fewer tools = faster decisions
- **Clarity**: Explicit tools show intent

### Model Choice
- **Don't over-engineer**: Sonnet handles most tasks
- **Reserve Opus**: For complex reasoning only
- **Use Haiku**: For simple, repetitive tasks
- **Cost awareness**: Monitor usage patterns

## Debugging Agents

### Check Agent Status
```bash
/agents
```

### Verbose Mode
```bash
claude --verbose
```

Shows:
- Which agent was invoked
- Why it was selected
- Tool calls made
- Results returned

### Test Agent Manually
Create minimal test case:
```bash
claude "Use [agent-name] to [specific task]"
```

### Common Issues

**Agent not invoked**:
- Check description field matches task
- Verify agent file is in correct location
- Check naming conflicts (project vs user)

**Agent fails**:
- Verify tool permissions
- Check model availability
- Review system prompt clarity

**Wrong agent selected**:
- Make descriptions more specific
- Use explicit invocation
- Review multiple agent descriptions for conflicts

## Security Considerations

### Agent Permissions
- Review tool access carefully
- Use principle of least privilege
- Restrict sensitive operations
- Audit agent actions

### Prompt Injection
- Agents process untrusted input
- Validate agent outputs
- Monitor for unusual behavior
- Use hooks for validation

### Team Sharing
- Review agents before enabling
- Document security requirements
- Use code review for agent changes
- Test in isolated environment first

## Resources & Examples

### Community Agents
- awesome-claude-code-agents (GitHub)
- wshobson/agents (83+ production agents)
- VoltAgent/awesome-claude-code-subagents

### Plugin Marketplaces
- Search for "claude-code agents" on GitHub
- Check team plugin repositories
- Community Discord channels

## Related Documentation
- [Main documentation](README.md)
- [Hooks Documentation](hook.md)
- [Plugins Documentation](plugin.md)
- [CLI Reference](cli.md)
- [Settings Documentation](settings.md)
