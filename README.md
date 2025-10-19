# Claude Code Workflows & Agents

A comprehensive production-ready system combining **specialized AI agents**, **multi-agent workflow orchestrators**, and **development tools** for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Overview

This unified repository provides everything needed for intelligent automation and multi-agent orchestration across modern software development:

- **9 Specialized Agents** - Domain experts with deep knowledge across database architecture, API design, backend/frontend development, testing, code review, DevOps, documentation, and Speckit management
- **1 Workflow Orchestrator** - Multi-agent coordination system for API-first full-stack development with proper task sequencing and delegation
- **Development Tools** - Focused utilities for specific tasks including API scaffolding, security scanning, test automation, and infrastructure setup

## Installation

Add this marketplace to Claude Code:

```bash
/plugin marketplace add https://github.com/bodis/agents
```

Then browse and install plugins using:

```bash
/plugin
```

### Available Plugins

#### Getting Started

**mvp-dev-stack** - MVP Development Stack (Python, React&Next.js, Supabase)
```bash
/plugin install mvp-dev-stack
```

[documentation about the stack](documentations/mvp-development-plugin.md)

## MVP Stack & Agentic Workflow

### Tech Stack
The MVP Development Stack consists of:
- **Backend**: Python/FastAPI with Pydantic v2
- **Frontend**: React/Next.js with TypeScript & shadcn/ui
- **Database**: Supabase (PostgreSQL with RLS)
- **API Design**: OpenAPI 3.0.0 specification-first
- **Deployment**: GCP (Cloud Run + Cloud Storage)

### API-First Agentic Development Flow

```mermaid
flowchart TD
    Start([User Request]) --> Orchestrator[implementation-orchestrator<br/>Delegates Only - No File Modifications]

    Orchestrator --> DB[1. supabase-architect<br/>Database Schema]
    DB --> DBFiles[Creates:<br/>•supabase/migrations/*.sql<br/>•docs/database/README.md]

    DBFiles --> API[2. api-designer<br/>API Contracts]
    API --> APIFiles[Creates:<br/>•docs/openapi.yaml<br/>•REST + SSE specs]

    APIFiles --> Backend[3. backend-developer<br/>FastAPI Implementation]
    Backend --> BackendFiles[Creates:<br/>•backend/src/**/*.py<br/>•Services & endpoints]

    APIFiles --> Frontend[4. frontend-developer<br/>React/Next.js UI]
    Frontend --> FrontendFiles[Creates:<br/>•frontend/src/**/*<br/>•Components & hooks]

    BackendFiles --> Tests[5. test-engineer<br/>Test Suites]
    FrontendFiles --> Tests
    Tests --> TestFiles[Creates:<br/>•backend/tests/**<br/>•frontend/tests/**<br/>•e2e/**]

    TestFiles --> Review[6. code-reviewer<br/>Quality Gate]
    Review --> Docs[7. documentation-writer<br/>Documentation Updates]
    Docs --> DocsFiles[Updates:<br/>•README.md<br/>•CLAUDE.md if relevant<br/>•docs/documentation.md]

    DocsFiles --> Speckit[8. speckit-manager<br/>Status Updates]
    Speckit --> SpeckitFiles[Updates:<br/>•specs/**/* task status<br/>Only if Speckit exists]

    SpeckitFiles --> Complete([Feature Complete])

    style Orchestrator fill:#3b3b3b,stroke-width:4px
    style DB fill:#3b3b3b,stroke-width:4px
    style API fill:#3b3b3b,stroke-width:4px
    style Backend fill:#3b3b3b,stroke-width:4px
    style Frontend fill:#3b3b3b,stroke-width:4px
    style Tests fill:#3b3b3b,stroke-width:4px
    style Review fill:#3b3b3b,stroke-width:4px
    style Docs fill:#3b3b3b,stroke-width:4px
    style Speckit fill:#3b3b3b,stroke-width:4px
    style Complete fill:#3b3b3b,stroke-width:4px
```

**Key Principles:**
- **Sequential Flow**: Each step depends on the previous one - no parallel execution
- **Orchestrator Delegates Only**: implementation-orchestrator cannot modify any files, only coordinates other agents
- **Clear Ownership**: Each agent can only modify their designated files
- **API-First**: Database → API Spec → Implementation → Testing → Documentation → Status Updates
- **Separation of Concerns**: Each agent has a single, focused responsibility
- **Documentation Last**: documentation-writer runs after implementation to update docs and CLAUDE.md if relevant
- **Status Management**: speckit-manager runs last to update task completion status (if Speckit exists)

### Example: Building a Notification System

```
👤 User Request: "Add notifications with real-time updates"

┌──────────────────────────────────────────────────────┐
│         🎯 implementation-orchestrator               │
│         Coordinates & Delegates (No File Edits)      │
└────────────────┬─────────────────────────────────────┘
                 │
Step 1 ──────────▼─────────────────────────────────────┐
│ 🗄️ supabase-architect                                │
│ Task: Create notifications table with RLS            │
│ ✅ Output: supabase/migrations/001_notifications.sql │
│           docs/database/README.md updated            │
└──────────────────────────────────────────────────────┘
                 │
Step 2 ──────────▼─────────────────────────────────────┐
│ 📝 api-designer                                      │
│ Task: Define REST endpoints + SSE stream             │
│ ✅ Output: docs/openapi.yaml with:                   │
│           POST /api/notifications                    │
│           GET /api/stream/notifications              │
└──────────────────────────────────────────────────────┘
                 │
Step 3 ──────────▼─────────────────────────────────────┐
│ ⚙️ backend-developer                                 │
│ Task: Implement FastAPI notification service         │
│ ✅ Output: backend/src/services/notifications.py     │
│           backend/src/api/v1/notifications.py        │
└──────────────────────────────────────────────────────┘
                 │
Step 4 ──────────▼─────────────────────────────────────┐
│ 🎨 frontend-developer                                │
│ Task: Build notification UI components               │
│ ✅ Output: frontend/src/components/NotificationBell  │
│           frontend/src/hooks/useNotifications        │
└──────────────────────────────────────────────────────┘
                 │
Step 5 ──────────▼─────────────────────────────────────┐
│ 🧪 test-engineer                                     │
│ Task: Write comprehensive test suite                 │
│ ✅ Output: backend/tests/test_notifications.py       │
│           e2e/notifications.spec.ts                  │
└──────────────────────────────────────────────────────┘
                 │
Step 6 ──────────▼─────────────────────────────────────┐
│ 🔍 code-reviewer                                     │
│ Task: Security audit & quality check                 │
│ ✅ Output: Approval with feedback                    │
└──────────────────────────────────────────────────────┘
                 │
Step 7 ──────────▼─────────────────────────────────────┐
│ 📚 documentation-writer                              │
│ Task: Update project documentation                   │
│ ✅ Output: docs/documentation.md updated             │
│           CLAUDE.md updated (SSE pattern added)      │
│           README.md (no changes needed)              │
└──────────────────────────────────────────────────────┘
                 │
Step 8 ──────────▼─────────────────────────────────────┐
│ ✓ speckit-manager                                    │
│ Task: Update Speckit task completion status          │
│ ✅ Output: specs/notifications/tasks/* marked done   │
│           specs/notifications/plan.md updated        │
└──────────────────────────────────────────────────────┘
                 │
                 ▼
         ✅ Feature Complete
```

## Agent Categories

### Workflow Coordination

**implementation-orchestrator**
- **Role**: Workflow coordinator and task delegator
- **Capabilities**: Plans execution, sequences tasks, delegates to specialized agents via natural language
- **Critical**: Cannot modify any files - only coordinates and delegates
- **Tools**: Read, Bash, Grep, Glob, TodoWrite
- **Delegation**: Uses explicit language to route work to specialized agents (framework handles invocation)
- **Activates**: When implementing complete features or multi-step tasks

### Database Layer

**supabase-architect**
- **Role**: Database schema designer and migration manager
- **Capabilities**: Creates migrations, manages RLS policies, generates TypeScript types
- **Modifies**: `supabase/migrations/*`, `docs/database/README.md`, `docs/datamodel.md`
- **Tools**: Read, Write, Edit, Bash, Grep, Glob
- **Activates**: For database structure changes, migrations, RLS policies

### API Design Layer

**api-designer**
- **Role**: OpenAPI 3.0.0 specification specialist
- **Capabilities**: Defines REST and SSE endpoint contracts, documents request/response schemas
- **Modifies**: `docs/openapi.yaml`
- **Tools**: Read, Write, Edit, Bash, Grep, Glob
- **Activates**: For API endpoint design, documentation, contract definition

### Implementation Layer

**backend-developer**
- **Role**: Python FastAPI implementation specialist
- **Capabilities**: Implements FastAPI endpoints, creates services and business logic, integrates with Supabase
- **Modifies**: `backend/src/**/*.py`
- **Tools**: Read, Write, Edit, Bash, Grep, Glob
- **Activates**: For Python/FastAPI code, API implementation, business logic

**frontend-developer**
- **Role**: React/Next.js UI specialist
- **Capabilities**: Builds React components, implements forms with Zod validation, integrates with backend APIs
- **Modifies**: `frontend/src/**/*`
- **Tools**: Read, Write, Edit, Bash, Grep, Glob, Playwright
- **Activates**: For React components, UI/UX, client-side logic

### Quality & Testing Layer

**test-engineer**
- **Role**: Testing specialist for comprehensive test coverage
- **Capabilities**: Creates unit tests (mandatory for backend), integration tests, E2E tests
- **Modifies**: `backend/tests/**`, `frontend/tests/**`, `e2e/**`
- **Tools**: Read, Write, Bash, Playwright
- **Activates**: For all testing tasks, test debugging

**code-reviewer**
- **Role**: Code quality and security specialist
- **Capabilities**: Reviews code for security, checks adherence to patterns, final approval
- **Modifies**: None (read-only review)
- **Tools**: Read, Bash
- **Activates**: For final review before feature completion

### Infrastructure Layer

**devops-engineer**
- **Role**: DevOps and infrastructure specialist
- **Capabilities**: Sets up local development, configures CI/CD, manages Docker, handles deployments
- **Modifies**: `.github/workflows/*`, Docker configurations, deployment scripts
- **Tools**: Read, Write, Edit, Bash, Grep, Glob
- **Activates**: For environment setup, deployment, CI/CD tasks

### Documentation Layer

**documentation-writer**
- **Role**: Documentation maintenance specialist
- **Capabilities**: Updates README.md, CLAUDE.md (when relevant), docs/documentation.md
- **Critical**: ONLY agent that can modify CLAUDE.md
- **Modifies**: `README.md`, `CLAUDE.md`, `docs/documentation.md`, `docs/*.md`
- **Tools**: Read, Write, Edit, Bash, Grep, Glob
- **Activates**: After feature implementation for documentation updates

**speckit-manager**
- **Role**: Speckit task status manager
- **Capabilities**: Updates task completion status based on implementation results
- **Critical**: ONLY agent that can modify specs/ directory, runs LAST in workflow
- **Modifies**: `specs/**/*` (task status only)
- **Tools**: Read, Write, Edit, Bash, Grep, Glob
- **Activates**: After documentation-writer, only if Speckit content exists

## Understanding Claude Code's Delegation Architecture

**CRITICAL for plugin developers**: Claude Code has a specific delegation model that fundamentally affects how you structure multi-agent workflows.

### How Delegation Actually Works

Claude Code uses a **single-level delegation model**:

```
Main Agent (default agent)
    ↓ can delegate via Task tool
Specialized Subagents (plugin agents)
    ✗ CANNOT delegate to other subagents
```

**This is the most important architectural constraint to understand when building plugins.**

### What Works ✅

**Main agent delegates to subagents:**
```
Main Agent → Task tool → supabase-architect subagent ✓
Main Agent → Task tool → backend-developer subagent ✓
```

**Slash commands transform main agent into a role:**
```
User: /mvp-implement feature-x
Main Agent (now acting as orchestrator):
  → Task tool → supabase-architect ✓
  → Task tool → backend-developer ✓
  → Task tool → frontend-developer ✓
```

### What Doesn't Work ❌

**Subagents trying to delegate to other subagents:**
```
Main Agent → orchestrator subagent
  → orchestrator tries: Task tool → supabase-architect ✗
  → orchestrator tries: Task tool → backend-developer ✗
```

**What actually happens**: The orchestrator subagent can describe what should be delegated and may return control to the main agent, but it cannot directly invoke other subagents. Success rate is inconsistent (<50%) with unpredictable behavior.

### Architectural Implications

**For plugin authors:**

1. **Use commands, not subagents, for orchestration roles**
   - Put orchestration logic in slash commands (`commands/*.md`)
   - Commands transform the main agent into the orchestrator
   - Main agent then has full Task tool delegation capability

2. **Subagents should be specialists, not coordinators**
   - Each subagent does one specific type of work
   - Subagents should NOT try to delegate to other subagents
   - If coordination is needed, use a command to elevate main agent

3. **Think hierarchically but execute flatly**
   - Design: Orchestrator → Database → API → Backend → Frontend
   - Implementation: Command makes main agent the orchestrator
   - Execution: Main → supabase-architect, Main → api-designer, Main → backend-developer

### Migration Pattern: Subagent to Command

If you have an orchestrator subagent that needs to delegate:

**Before (doesn't work reliably)**:
```
agents/orchestrator.md:
  - Tries to delegate to other agents
  - Describes delegation but can't execute
```

**After (works reliably)**:
```
commands/implement.md:
  - Contains all orchestrator instructions
  - Main agent adopts this role when command invoked
  - Main agent successfully delegates via Task tool
```

**Migration steps**:
1. Copy orchestrator agent's markdown content
2. Create new command file (`commands/your-command.md`)
3. Paste content and adjust to command format
4. Remove orchestrator agent file
5. Update documentation

### Real-World Example

The `supabase-python-react-stack` plugin learned this lesson:

- **v0.1.0**: Had `implementation-orchestrator` as a subagent → unreliable delegation
- **v0.2.0**: Moved orchestration to `/mvp-implement` command → reliable delegation

**Key insight**: Commands extend the main agent's capabilities; subagents are specialists that the main agent coordinates.

## Plugin Development Best Practices

### Agent Naming & Delegation

**Rule**: When agents installed via plugin, they're prefixed (e.g., `plugin-name:agent-name`).

- ✅ Always use full prefixed names when delegating: "Delegating to plugin-name:agent-name"
- ✅ Document full names in plugin README with a table
- ✅ Use prefixed names in all agent markdown files
- ✅ Use explicit, clear language for delegation (framework handles routing)
- ❌ Never assume short names work between agents

### Delegation Architecture (Most Important!)

**Rule**: Use commands for orchestration, subagents for specialization.

- ✅ Use commands (not subagents) for orchestration roles
- ✅ Commands should transform main agent into coordinator role
- ✅ Subagents should be specialists that don't delegate further
- ✅ Use single-level delegation: Main → Subagents (via Task tool)
- ❌ Don't create orchestrator subagents that try to delegate to other subagents

**Why this matters**: Subagents cannot reliably delegate to other subagents in Claude Code. Multi-level delegation fails inconsistently. Always use commands to elevate the main agent into an orchestrator role.

### Agent Boundaries & Constraints

**Rule**: Define clear file ownership; prevent workarounds.

- ✅ Explicitly list which files each agent owns (can modify)
- ✅ Explicitly list which files each agent reads (cannot modify)
- ✅ Use tool restrictions to enforce boundaries
- ✅ Commands that create orchestrator roles should avoid giving file modification tools
- ✅ Document forbidden workarounds (e.g., bash pipes, echo redirection)
- ❌ Don't give coordination roles file modification tools
- ❌ Don't give agents escape hatches when delegation fails

### Documentation Dependencies

**Rule**: Single source of truth; clear ownership.

- ✅ Designate ONE agent owner per documentation file
- ✅ Document which agent maintains each file
- ✅ Define explicit read/write permissions
- ❌ Never duplicate documentation across files
- ❌ Never allow multiple agents to modify the same doc

### Workflow Enforcement

**Rule**: Define strict sequential phases with validation.

- ✅ Document required phase order
- ✅ Add pre-flight checks for dependencies
- ✅ Include validation steps in each phase
- ✅ Agents must STOP when dependencies missing
- ❌ Never skip phases
- ❌ Never proceed with assumptions or incomplete specs

### Error Handling

**Rule**: Fail fast; report clearly; never guess.

- ✅ Agents STOP immediately when blocked
- ✅ Report exact error with resolution steps
- ✅ Reference specific files/agents needed
- ❌ Never work around missing specifications
- ❌ Never assume or infer requirements

### Testing & Quality

**Rule**: Automate validation; make critical tests mandatory.

- ✅ Mark critical tests as mandatory (e.g., backend unit tests)
- ✅ Use hooks for auto-formatting (post_edit)
- ✅ Use hooks for validation (OpenAPI specs, TypeScript checks)
- ✅ Smart test runners (only run relevant tests)
- ❌ Don't skip validation steps
- ❌ Don't make all tests optional

### Speckit Integration

**Rule**: One agent owns specs/; runs last.

- ✅ Only speckit-manager modifies `specs/**/*`
- ✅ All other agents READ specs (never write)
- ✅ Speckit-manager runs LAST in workflow
- ❌ Never let implementation agents modify specs

## Contributing

To add new agents, workflows, or tools:

1. Create a new `.md` file in the appropriate directory with frontmatter
2. Use lowercase, hyphen-separated naming convention
3. Write clear activation criteria in the description
4. Define comprehensive system prompt with expertise areas

### Subagent Format

Each subagent is defined as a Markdown file with frontmatter:

```markdown
---
name: subagent-name
description: Activation criteria for this subagent
model: haiku|sonnet|opus  # Optional: Model selection
tools: tool1, tool2       # Optional: Tool restrictions
---

System prompt defining the subagent's expertise and behavior
```

### Model Selection Criteria

- **haiku**: Simple, deterministic tasks with minimal reasoning
- **sonnet**: Standard development and engineering tasks
- **opus**: Complex analysis, architecture, and critical operations

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Resources

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Subagents Documentation](https://docs.anthropic.com/en/docs/claude-code/sub-agents)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)