# MVP Development Stack Plugin

AI-driven development workflow with Speckit integration, sequential agent orchestration, and comprehensive testing.

## Features

- 🎯 **Speckit Integration**: Define features with Speckit, implement with AI agents
- 🤖 **Intelligent Orchestration**: Sequential task execution with specialized agents
- ✅ **Test-First Development**: Mandatory backend tests, conditional frontend tests
- 🔄 **Quality Gates**: Automated code review, security checks, test coverage
- 🚀 **CI/CD Ready**: GitHub Actions, Supabase deployments

## Installation

### Option 1: Project-Level (Recommended)

1. **Clone or copy this plugin to your project**:
```bash
mkdir -p .claude-plugins
cp -r mvp-dev-stack .claude-plugins/
```

2. **Create `.claude/settings.json`**:
```json
{
  "marketplaces": [
    {
      "name": "local-plugins",
      "url": "file://./.claude-plugins"
    }
  ],
  "plugins": ["mvp-dev-stack@local-plugins"]
}
```

3. **Create `.env` with required keys**:
```env
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_SERVICE_KEY=xxx
STRIPE_SECRET_KEY=sk_test_xxx
```

4. **Trust your repository folder** when Claude Code asks

### Option 2: Global Installation

```bash
# Copy to global marketplace
mkdir -p ~/.claude/marketplaces/my-plugins
cp -r mvp-dev-stack ~/.claude/marketplaces/my-plugins/

# In Claude Code
/plugin marketplace add file://~/.claude/marketplaces/my-plugins
/plugin install mvp-dev-stack
```

## Usage

### 1. Define Feature with Speckit

```bash
# In Claude Code, manually run:
/speckit.specify    # Define feature requirements
/speckit.analyze    # Analyze and break down
/speckit.plan       # Create implementation plan
```

This creates: `specs/NNN-feature-name/plan.md`

### 2. Implement Feature

```bash
# Hand off to orchestration
/implement 003

# Or be explicit
"Use implementation-orchestrator to execute specs/003-*/plan.md"
```

The orchestrator will:
- ✅ Read and parse the plan
- ✅ Break into sequential tasks
- ✅ Assign each task to specialized agent
- ✅ Ensure tests pass before proceeding
- ✅ Ask for input when needed
- ✅ Report completion

### 3. Check Progress

```bash
/status  # See current implementation status
```

### 4. Review Before Implementing

```bash
/review-plan 003  # Get plan review and recommendations
```

## Agents

### Implementation Orchestrator
**Role**: Main coordinator
**When**: Always for `/implement` command
**Responsibility**: Sequential task execution, agent delegation

### Backend Developer
**Role**: Python/FastAPI specialist
**When**: API endpoints, business logic, database operations
**Responsibility**: TDD, type-safe code, Supabase integration

### Frontend Developer
**Role**: React/Next.js specialist  
**When**: UI components, pages, client-side logic
**Responsibility**: TypeScript, responsive design, API integration

### Test Engineer
**Role**: Testing specialist
**When**: Writing tests, debugging test failures
**Responsibility**: Unit tests (backend required), E2E tests (conditional)

### Code Reviewer
**Role**: Quality assurance
**When**: Final gate before feature completion
**Responsibility**: Security, best practices, code quality

### DevOps Agent
**Role**: CI/CD and deployment
**When**: Infrastructure, pipelines, deployments
**Responsibility**: GitHub Actions, Supabase migrations

## MCP Servers

This plugin configures four MCP servers:

- **Supabase MCP**: Database operations, schema management
- **Playwright MCP**: E2E testing automation
- **Ref MCP**: Up-to-date documentation lookup
- **Stripe MCP**: Payment integration (if used)

## Project Structure Expected

```
your-project/
├── src/                    # Python backend
│   ├── api/                # FastAPI endpoints
│   ├── models/             # Pydantic models
│   ├── services/           # Business logic
│   └── core/               # Config, dependencies
├── tests/                  # Backend tests
│   ├── unit/               # Required
│   └── integration/        # Required for APIs
├── frontend/               # React/Next.js
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   └── api/
│   └── tests/              # Conditional
├── specs/                  # Speckit specifications
├── .claude/                # Claude Code config
├── .claude-plugins/        # This plugin
└── .github/workflows/      # CI/CD
```

## Configuration

### Required Files

**`.env`**:
```env
SUPABASE_URL=xxx
SUPABASE_SERVICE_KEY=xxx
STRIPE_SECRET_KEY=xxx
```

**`constitution.md`** (project root):
Your project principles and constraints.

**`CLAUDE.md`** (project root):
Living document of patterns, learnings, conventions.

## Testing Requirements

### Backend (Python)
- ✅ **REQUIRED**: Unit tests for all business logic
- ✅ **REQUIRED**: Integration tests for API endpoints  
- ✅ **REQUIRED**: >80% coverage
- ✅ **REQUIRED**: Tests pass before proceeding

### Frontend (React)
- ⚠️ **CONDITIONAL**: Tests for complex components
- ⚠️ **CONDITIONAL**: E2E tests for critical flows
- ✅ **ALWAYS**: Visual review for UI changes

## Workflow Philosophy

1. **Sequential, Not Parallel**: Tasks execute one at a time
2. **Test-First**: Write tests before implementation
3. **Stop on Failure**: Never proceed with failing tests
4. **User Involvement**: Ask when ambiguous, don't assume
5. **Clear Communication**: Progress updates after each task

## Examples

### Simple Feature Implementation

```bash
# 1. Define with Speckit
/speckit.specify "Add user notification system"

# 2. Review plan
/review-plan 003

# 3. Implement
/implement 003

# Orchestrator executes:
# → backend-developer: Create models
# → test-engineer: Write unit tests
# → backend-developer: Create service
# → backend-developer: Create API endpoints
# → test-engineer: Integration tests
# → frontend-developer: Create UI components
# → test-engineer: E2E tests (if needed)
# → code-reviewer: Final review
```

### Complex Multi-Step Feature

```bash
/implement 005

# Orchestrator recognizes complexity
# Breaks into smaller tasks
# Asks for clarification on ambiguous parts
# Executes with proper dependencies
# Reports blockers immediately
```

## Troubleshooting

### Plugin Not Loading
```bash
# Check settings
cat .claude/settings.json

# Verify plugin structure
ls -la .claude-plugins/mvp-dev-stack/

# Restart Claude Code
```

### MCP Servers Not Working
```bash
# Check environment variables
cat .env

# Test Supabase connection
supabase status

# Verify MCP config
cat .claude-plugins/mvp-dev-stack/.mcp.json
```

### Tests Failing
```bash
# Backend
pytest tests/ -v --tb=short

# Frontend  
npm test -- --verbose
```

## Contributing

1. Fork/copy this plugin
2. Modify agents in `agents/` directory
3. Update `CLAUDE.md` with your patterns
4. Test with real features
5. Share improvements!

## License

MIT

## Support

For issues with:
- **Plugin**: Check this README, review agent prompts
- **Speckit**: See Speckit documentation
- **Claude Code**: See https://docs.claude.com/claude-code
- **MCP Servers**: Check individual MCP documentation

## Version History

- **1.0.0** (2025-10-11): Initial release
  - 6 specialized agents
  - Speckit integration
  - Sequential orchestration
  - 4 MCP servers configured