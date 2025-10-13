# Claude Code - Model Context Protocol (MCP) Documentation

Complete reference for integrating Claude Code with external tools and data sources through MCP.

## What is MCP?

The Model Context Protocol (MCP) is an open standard for connecting AI assistants to external tools and data sources. It provides a universal, open standard for connecting AI systems with data sources, replacing fragmented integrations with a single protocol.

### Key Benefits
- **Unified integration**: One protocol for all external tools
- **Security**: Proper authentication and permission handling
- **Extensibility**: Easy to add new capabilities
- **Community**: Growing ecosystem of MCP servers

## MCP Architecture

```
┌──────────────┐
│ Claude Code  │ ← Host/Client
│   (MCP Client) │
└──────┬───────┘
       │
       │ MCP Protocol (JSON-RPC 2.0)
       │
┌──────┴───────────────────────┐
│     MCP Servers              │
├──────────────────────────────┤
│ • Filesystem                 │
│ • GitHub                     │
│ • Google Drive               │
│ • Slack                      │
│ • Database                   │
│ • Custom Tools               │
└──────────────────────────────┘
```

### Components
- **Host**: Claude Code (the application running the AI)
- **MCP Client**: Connector within Claude Code
- **MCP Server**: External tool or data source
- **Tools**: Capabilities exposed by MCP servers
- **Resources**: Data available through MCP servers

## Quick Start

### List Configured Servers
```bash
claude mcp list
```

### Add MCP Server
```bash
# Basic syntax
claude mcp add <name> -- <command> [args...]

# With scope
claude mcp add <name> -s user|project -- <command>

# With environment variables
claude mcp add <name> -e KEY=value -- <command>

# With transport
claude mcp add <name> --transport http|sse <url>
```

### Common MCP Servers

#### Filesystem Access
```bash
claude mcp add filesystem -s user -- npx -y @modelcontextprotocol/server-filesystem ~/Documents ~/Projects
```

#### GitHub Integration
```bash
claude mcp add github -s user -e GITHUB_TOKEN=$GITHUB_TOKEN -- npx -y @modelcontextprotocol/server-github
```

#### Google Drive
```bash
claude mcp add gdrive -s user -- npx -y @modelcontextprotocol/server-gdrive
```

#### Slack
```bash
claude mcp add slack -s user -e SLACK_BOT_TOKEN=$SLACK_BOT_TOKEN -- npx -y @modelcontextprotocol/server-slack
```

## Configuration

### Configuration Locations

MCP servers can be configured at three levels (in order of precedence):

1. **Project-scoped** (`.mcp.json` in project root)
   - Committed to version control
   - Shared with team
   - Highest priority

2. **Project-local** (`.claude/settings.local.json`)
   - Not committed
   - Personal overrides
   - Medium priority

3. **User-level** (`~/.claude/settings.json`)
   - Global personal settings
   - Lowest priority

### .mcp.json Format

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@package/server"],
      "env": {
        "API_KEY": "your-key"
      }
    }
  }
}
```

### settings.json Format

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@package/server"],
      "env": {
        "API_KEY": "your-key"
      }
    }
  }
}
```

## MCP Server Types

### Transport Types

#### stdio (Standard Input/Output)
Most common transport for local servers.

```json
{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@package/server"]
}
```

#### HTTP
For remote HTTP-based servers.

```bash
claude mcp add --transport http stripe https://mcp.stripe.com
```

#### SSE (Server-Sent Events)
For streaming remote servers.

```bash
claude mcp add --transport sse asana https://mcp.asana.com/sse
```

## Official MCP Servers

### Development Tools

#### GitHub
Manage repositories, issues, and pull requests.
```bash
claude mcp add github -s user -e GITHUB_TOKEN=$GITHUB_TOKEN -- npx -y @modelcontextprotocol/server-github
```

Capabilities:
- Create and manage issues
- Review pull requests
- Search repositories
- Manage branches

#### GitLab
Similar to GitHub for GitLab users.
```bash
claude mcp add gitlab -s user -e GITLAB_TOKEN=$GITLAB_TOKEN -- npx -y @modelcontextprotocol/server-gitlab
```

### File & Cloud Storage

#### Filesystem
Direct file manipulation beyond @ references.
```bash
claude mcp add filesystem -s user -- npx -y @modelcontextprotocol/server-filesystem ~/Documents ~/Projects
```

Security: Only scoped to specified directories.

#### Google Drive
Access and manage Google Drive files.
```bash
claude mcp add gdrive -s user -- npx -y @modelcontextprotocol/server-gdrive
```

### Communication

#### Slack
Integrate with Slack workspaces.
```bash
claude mcp add slack -s user -e SLACK_BOT_TOKEN=$BOT_TOKEN -- npx -y @modelcontextprotocol/server-slack
```

Capabilities:
- Read channels
- Send messages
- Search history
- Manage threads

#### Gmail
Email management and automation.
```bash
claude mcp add gmail -s user -- npx -y @modelcontextprotocol/server-gmail
```

### Databases

#### PostgreSQL
Query and manage PostgreSQL databases.
```bash
claude mcp add postgres -s user -e DATABASE_URL=$DB_URL -- npx -y @modelcontextprotocol/server-postgres
```

#### SQLite
Local SQLite database access.
```bash
claude mcp add sqlite -s user -- npx -y @modelcontextprotocol/server-sqlite --db-path ./data.db
```

### Project Management

#### Asana
Task and project management.
```bash
claude mcp add --transport sse asana https://mcp.asana.com/sse
```

#### Jira & Confluence (Atlassian)
Issue tracking and documentation.
```bash
claude mcp add --transport sse atlassian https://mcp.atlassian.com/v1/sse
```

#### Linear
Modern issue tracking.
```bash
claude mcp add linear -s user -e LINEAR_API_KEY=$KEY -- npx -y @linear/mcp-server
```

### Monitoring & Observability

#### Sentry
Error tracking and debugging.
```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

#### Jam
Debug with recordings, console logs, and network requests.
```bash
claude mcp add --transport http jam https://mcp.jam.dev/mcp
```

### Design & Media

#### Figma
Design file integration.
```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

Local server option also available - see developers.figma.com.

#### Cloudinary
Media asset management.
```bash
# See Cloudinary documentation for specific server URLs
```

#### Canva
Design browsing and generation.
```bash
claude mcp add --transport http canva https://mcp.canva.com/mcp
```

### Business Services

#### Stripe
Payment processing integration.
```bash
claude mcp add --transport http stripe https://mcp.stripe.com
```

#### invideo
Video creation capabilities.
```bash
claude mcp add --transport sse invideo https://mcp.invideo.io/sse
```

### Browser Automation

#### Puppeteer
Browser automation and web scraping.
```bash
claude mcp add puppeteer -s user -- npx -y @modelcontextprotocol/server-puppeteer
```

### Development Utilities

#### Context7
Real-time, version-specific library documentation.
```bash
claude mcp add context7 -s user -- npx -y @upstash/context7-mcp@latest
```

#### Perplexity
AI-powered search integration.
```bash
claude mcp add perplexity -s user -e PERPLEXITY_API_KEY=$KEY -- npx -y perplexity-mcp
```

## Usage Examples

### Query Databases
```bash
# After configuring postgres MCP
claude "Find emails of 10 random users who used feature ENG-4521 from our Postgres database"
```

### Create Issues
```bash
# After configuring GitHub/Linear MCP
claude "Add the feature described in Linear issue ENG-4521 and create a PR on GitHub"
```

### Analyze Monitoring Data
```bash
# After configuring Sentry MCP
claude "Check Sentry to analyze errors for feature ENG-4521"
```

### Integrate Designs
```bash
# After configuring Figma MCP
claude "Update our email template based on the new Figma designs posted in Slack"
```

### Automate Communication
```bash
# After configuring Gmail MCP
claude "Create Gmail drafts inviting 10 users to a feedback session"
```

## Advanced Configuration

### Environment Variables
```json
{
  "mcpServers": {
    "api-server": {
      "command": "node",
      "args": ["server.js"],
      "env": {
        "API_KEY": "your-key",
        "API_URL": "https://api.example.com",
        "DEBUG": "true"
      }
    }
  }
}
```

### Multiple Instances
```json
{
  "mcpServers": {
    "github-work": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "work-token"
      }
    },
    "github-personal": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "personal-token"
      }
    }
  }
}
```

### Plugin Environment Variables
Use `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths:

```json
{
  "mcpServers": {
    "plugin-server": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/custom",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "DATA_PATH": "${CLAUDE_PLUGIN_ROOT}/data"
      }
    }
  }
}
```

## Output Management

### Token Limits
MCP tools can produce large outputs. Claude Code manages this:

- **Warning threshold**: 10,000 tokens
- **Default limit**: 25,000 tokens
- **Configurable**: Set `MAX_MCP_OUTPUT_TOKENS` environment variable

```bash
export MAX_MCP_OUTPUT_TOKENS=50000
claude
```

### When to Increase Limits
- Database queries returning many rows
- File system operations listing large directories
- API responses with extensive data
- Document retrieval with large content

## Security Considerations

### API Keys & Credentials
- **Never commit** API keys to version control
- Use environment variables
- Store in `.claude/settings.local.json` (not committed)
- Use secrets management for teams

### Trust & Review
- **Review MCP servers** before installation
- Check source code for third-party servers
- Be cautious with servers that fetch untrusted content
- Understand what data each server can access

### Prompt Injection Risk
- MCP servers fetching external content can expose you to prompt injection
- Review server security practices
- Use trusted, official servers when possible

### Access Control
- Limit MCP server scope to necessary directories
- Use least privilege principle
- Review permissions in `/permissions` menu
- Monitor MCP server activity with `--verbose`

## Debugging MCP

### List Servers & Status
```bash
claude mcp list
```

### Get Server Details
```bash
claude mcp get <server-name>
```

### Debug Mode
```bash
claude --mcp-debug
```

Shows:
- MCP server initialization
- Tool invocations
- Communication errors
- Authentication issues

### Test MCP Server Manually
```bash
# Test server directly
echo '{"jsonrpc":"2.0","method":"initialize","params":{"protocolVersion":"2024-11-05"},"id":1}' | npx @modelcontextprotocol/server-github
```

### Check Logs
MCP server logs are typically in:
```bash
~/.claude/logs/mcp-server-<name>.log
```

## Creating Custom MCP Servers

### Basic Structure
```typescript
import { Server } from '@modelcontextprotocol/sdk/server';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio';

const server = new Server({
  name: 'my-custom-server',
  version: '1.0.0'
});

// Add tools
server.setRequestHandler('tools/list', async () => ({
  tools: [
    {
      name: 'my_tool',
      description: 'What this tool does',
      inputSchema: {
        type: 'object',
        properties: {
          param: { type: 'string' }
        }
      }
    }
  ]
}));

// Handle tool calls
server.setRequestHandler('tools/call', async (request) => {
  // Implementation
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Publishing
1. Package as npm module
2. Document configuration requirements
3. Include security considerations
4. Submit to MCP server directory
5. Share in community

## MCP in Hooks

Hooks can target MCP tools using the pattern `mcp__<server>__<tool>`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__github__*",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'About to use GitHub MCP tool'"
          }
        ]
      }
    ]
  }
}
```

Examples:
- `mcp__memory__create_entities` - Memory server
- `mcp__filesystem__read_file` - Filesystem server
- `mcp__github__search_repositories` - GitHub server

See Hooks documentation for more details.

## Best Practices

### Configuration Management
- Use project `.mcp.json` for team-shared servers
- Use `.claude/settings.local.json` for personal credentials
- Document required environment variables in README
- Version control safe configurations

### Performance
- Configure appropriate output limits
- Use pagination for large datasets
- Implement caching when appropriate
- Monitor token usage

### Security
- Regular security audits of custom servers
- Keep servers updated
- Use HTTPS for remote servers
- Implement proper authentication

### Team Collaboration
- Document MCP setup in project README
- Provide setup scripts for common configurations
- Share marketplace configurations
- Maintain internal server documentation

## Docker MCP Toolkit

For containerized MCP server management:

```bash
# See Docker Desktop MCP Toolkit
# Provides one-click deployment
# 220+ pre-built servers
# Automatic credential handling
```

Benefits:
- No dependency conflicts
- Consistent environments
- Easy updates
- Security isolation

## Resources

- MCP Specification: https://spec.modelcontextprotocol.io
- Official Servers: https://github.com/modelcontextprotocol/servers
- Community Servers: https://www.claudemcp.com
- Docker MCP Toolkit: Docker Desktop integration

## Related Documentation
- [Main documentation](README.md)
- [CLI Reference](cli.md)
- [Hooks Documentation](hook.md)
- [Plugins Documentation](plugin.md)
- [Settings Documentation](settings.md)
