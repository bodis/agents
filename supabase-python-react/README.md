# Supabase Python React Stack Plugin

AI-driven MVP development stack with Speckit integration, sequential agent orchestration, and comprehensive testing for building SaaS applications.

## Overview

This Claude Code plugin provides a complete development workflow using specialized agents that work together to implement features following API-first principles. The stack combines Supabase (database), Python FastAPI (backend), and React/Next.js (frontend) with automated testing, code review, and documentation.

**Version**: 0.2.0
**Author**: Tamas Bodis
**License**: MIT

## Key Features

- **Sequential Agent Orchestration**: Enforced workflow ensures proper task ordering
- **API-First Development**: Database → API Spec → Backend → Frontend → Tests
- **Speckit Integration**: Task and plan management with automatic status tracking
- **Comprehensive Testing**: Backend unit tests (mandatory), E2E tests (conditional)
- **Auto-Formatting**: Black for Python, Prettier for TypeScript
- **OpenAPI Validation**: Automatic spec validation on changes
- **Smart Test Runner**: Runs only relevant tests based on changed files

## Critical: Agent Naming Convention

### Plugin Prefix Requirement

All agents in this plugin are automatically prefixed with `supabase-python-react-stack:` when installed. This is a **mandatory requirement** for agent delegation to work correctly.

### Correct Usage

When using the Task tool to delegate work to agents, **ALWAYS use the full prefixed name**:

✅ **Correct**:
```markdown
Task tool with subagent_type="supabase-python-react-stack:backend-developer"
```

❌ **Incorrect**:
```markdown
Task tool with subagent_type="backend-developer"
```

### Agent Names

| Short Name | Full Prefixed Name |
|-----------|-------------------|
| `supabase-architect` | `supabase-python-react-stack:supabase-architect` |
| `api-designer` | `supabase-python-react-stack:api-designer` |
| `backend-developer` | `supabase-python-react-stack:backend-developer` |
| `frontend-developer` | `supabase-python-react-stack:frontend-developer` |
| `test-engineer` | `supabase-python-react-stack:test-engineer` |
| `code-reviewer` | `supabase-python-react-stack:code-reviewer` |
| `devops-engineer` | `supabase-python-react-stack:devops-engineer` |
| `documentation-writer` | `supabase-python-react-stack:documentation-writer` |
| `speckit-manager` | `supabase-python-react-stack:speckit-manager` |
| `implementation-orchestrator` | `supabase-python-react-stack:implementation-orchestrator` |

## Architecture & Workflow

### API-First Development Workflow

The plugin enforces a strict sequential workflow:

```
1. Database Design (supabase-architect)
   ↓ Creates: migrations, docs/datamodel.md

2. API Specification (api-designer)
   ↓ Creates: docs/openapi.yaml

3. Backend Implementation (backend-developer)
   ↓ Creates: backend/src/**/*.py

4. Frontend Implementation (frontend-developer)
   ↓ Creates: frontend/src/**/*

5. Testing (test-engineer)
   ↓ Creates: backend/tests/**, frontend tests (conditional)

6. Code Review (code-reviewer)
   ↓ Reviews: security, quality, patterns

7. Documentation (documentation-writer)
   ↓ Updates: README.md, CLAUDE.md, docs/

8. Speckit Status (speckit-manager) [if applicable]
   ↓ Updates: specs/**/* task status
```

### Documentation Dependencies

Critical documentation files that agents depend on:

| File | Maintained By | Consumed By | Purpose |
|------|--------------|-------------|---------|
| `specs/**/*` | speckit-manager (ONLY) | All agents (read-only) | Feature specifications, plans |
| `docs/datamodel.md` | supabase-architect | ALL agents | Data model (single source of truth) |
| `docs/database/README.md` | supabase-architect | backend, api-designer | Migration details, RLS policies |
| `docs/openapi.yaml` | api-designer | backend, frontend | API specifications |
| `docs/external_apis.md` | Manual | api-designer, backend, frontend | External system integrations |
| `CLAUDE.md` | documentation-writer | All agents | Project patterns, decisions |
| `docs/CHANGELOG.md` | documentation-writer | Reference only | Feature history |

## Agent Responsibilities

### Database Layer

**supabase-python-react-stack:supabase-architect**
- Creates/modifies database schemas via migrations
- Manages Row Level Security (RLS) policies
- Maintains `docs/datamodel.md` and `docs/database/README.md`
- Generates TypeScript types from schema
- **Must run before**: api-designer

### API Design Layer

**supabase-python-react-stack:api-designer**
- Creates OpenAPI 3.0.0 specifications in `docs/openapi.yaml`
- Defines REST and SSE endpoint contracts
- Documents request/response schemas
- Validates specifications
- **Must run before**: backend-developer, frontend-developer
- **Depends on**: supabase-architect (for datamodel)

### Implementation Layer

**supabase-python-react-stack:backend-developer**
- Implements FastAPI endpoints from OpenAPI specs
- Creates Python services and business logic
- Integrates with Supabase using defined schemas
- Writes SSE endpoints for real-time features
- **Depends on**: api-designer, supabase-architect

**supabase-python-react-stack:frontend-developer**
- Builds React/Next.js UI components
- Uses shadcn/ui component library
- Implements forms with Zod validation
- Integrates with backend APIs
- **Depends on**: api-designer (for endpoints)

### Quality & Testing Layer

**supabase-python-react-stack:test-engineer**
- Creates unit tests (mandatory for backend)
- Writes integration tests for APIs
- Implements E2E tests for critical flows
- Ensures > 80% backend coverage

**supabase-python-react-stack:code-reviewer**
- Reviews code for security and quality
- Checks adherence to patterns
- Final approval before completion
- **Runs after**: all implementation

### Infrastructure Layer

**supabase-python-react-stack:devops-engineer**
- Sets up local development environment
- Configures CI/CD pipelines (GitHub Actions)
- Manages Docker configurations
- Handles GCP deployments

### Documentation Layer

**supabase-python-react-stack:documentation-writer**
- Maintains README.md, CLAUDE.md, docs/documentation.md
- Updates CLAUDE.md with new patterns and learnings
- Documents features after implementation
- **Runs after**: code-reviewer

**supabase-python-react-stack:speckit-manager**
- Updates Speckit task/plan status in specs/ directory ONLY
- Marks tasks as completed based on implementation results
- Cannot modify anything outside specs/ directory
- **Runs LAST** after documentation-writer

### Orchestration Layer

**supabase-python-react-stack:implementation-orchestrator**
- Coordinates sequential feature implementation
- Delegates tasks to specialized agents
- Cannot modify files directly (only delegates)
- Enforces API-first workflow
- Tracks progress and validates each phase

## Key Architectural Decisions

### 1. Speckit Content Protection (2025-10-15, commit c0a9b5c)

**Decision**: Only `speckit-manager` agent can modify `specs/**/*` directory.

**Rationale**: Prevents other agents from accidentally modifying feature specifications during implementation. Ensures specs remain the single source of truth for requirements.

**Impact**: All agents READ specs but cannot write to them. Speckit-manager runs LAST in workflow to update task completion status.

### 2. API-First Development (commit dfa5530)

**Decision**: Enforce strict sequence: Database → API → Backend → Frontend.

**Rationale**: Prevents implementation inconsistencies and ensures all layers work from the same contracts.

**Impact**:
- Backend cannot implement endpoints not in OpenAPI spec
- Frontend cannot create mock APIs
- All must stop if dependencies incomplete

### 3. Agent Prefix Requirement (2025-10-16)

**Decision**: Use full prefixed agent names in all delegations.

**Rationale**: Claude Code installs plugin agents with namespace prefix to avoid naming conflicts. Unprefixed names cause Task tool failures.

**Impact**:
- All agent markdown files updated with prefixed references
- Orchestrator enforced to use full names
- Prevents delegation failures and improper workarounds

### 4. Orchestrator Constraints (2025-10-16)

**Decision**: Implementation orchestrator has NO file modification tools.

**Rationale**: Prevents orchestrator from implementing code itself when delegation fails. Forces proper error reporting.

**Impact**:
- Orchestrator can only READ files and delegate work
- If Task tool fails, must STOP and report (no workarounds)
- Forbidden from using bash pipes/redirection for file creation

### 5. Auto-Formatting Hooks (commit c9eb860)

**Decision**: Automatic formatting after file edits (Black for Python, Prettier for TypeScript).

**Rationale**: Ensures consistent code style without manual intervention.

**Impact**:
- `post_edit` hook formats Python with Black
- `post_edit` hook formats TypeScript/TSX with Prettier
- OpenAPI spec validated automatically on change
- Database types regenerated after migrations

### 6. Smart Test Runner (commit c9eb860)

**Decision**: Run only relevant tests based on changed files.

**Rationale**: Speeds up pre-commit checks by avoiding full test suite runs.

**Impact**:
- `pre_commit` hook detects changed files
- Runs backend tests only if backend/Python files changed
- Runs frontend tests only if frontend files changed
- Fails fast (-x flag) on first test failure

## Commands

### /supabase-python-react-stack:mvp-implement [feature-id] or description
Coordinate feature implementation using implementation-orchestrator agent.

**Usage**:
```
/supabase-python-react-stack:mvp-implement 001-user-authentication
/supabase-python-react-stack:mvp-implement Add user profile management
```

### /supabase-python-react-stack:mvp-review-plan
Review Speckit plan for compatibility with MVP stack agents and tech stack.

### /supabase-python-react-stack:speckit-status
Check MVP stack implementation status and agent workflow progress.

## Hooks

### Post-Edit Hooks
- **format-python**: Auto-format Python code with Black
- **format-typescript**: Auto-format TypeScript/TSX with Prettier
- **validate-openapi**: Validate OpenAPI spec on change
- **generate-db-types**: Regenerate TypeScript types after migration changes

### Pre-Commit Hooks
- **smart-test-runner**: Run only relevant tests based on changed files

### Pre-Tool-Use Hooks
- **log-agent-action**: Log which agent is performing actions (debugging)

### Post-Tool-Use Hooks
- **check-db-doc-sync**: Remind to update DB docs after migration

### User-Prompt-Submit Hooks
- **load-context**: Load project context for agents

## Tech Stack

### Backend
- **Python 3.11+** with `uv` package manager
- **FastAPI** + **Pydantic v2** for APIs
- **Supabase** Python client for database
- **Server-Sent Events (SSE)** for real-time features
- **pytest** for testing
- **Black** for formatting

### Frontend
- **React 18+** with hooks and Server Components
- **Next.js 14+** with App Router
- **TypeScript** strict mode
- **shadcn/ui** component library
- **React Query** for data fetching
- **Zod** for validation
- **Playwright** for E2E testing
- **Prettier** for formatting

### Database
- **Supabase** (PostgreSQL)
- Row Level Security (RLS) policies enforced
- Migration-based schema management

### DevOps
- **Docker** for containerization
- **GitHub Actions** for CI/CD
- **GCP** for cloud deployment

## Best Practices for Creating New Plugins

Based on lessons learned from this plugin:

### 1. Agent Naming
- ✅ Document the full prefixed name for all agents
- ✅ Use prefixes in all agent markdown files
- ✅ Include naming convention in README
- ❌ Don't assume agents can reference each other by short names

### 2. Agent Constraints
- ✅ Clearly define which agents can modify which files
- ✅ Use tool restrictions to enforce boundaries
- ✅ Explicitly forbid workarounds when delegation fails
- ❌ Don't give orchestrators file modification tools

### 3. Documentation
- ✅ Maintain clear documentation dependencies
- ✅ Define single source of truth for each concept
- ✅ Document which agent maintains each file
- ❌ Don't allow duplication of documentation

### 4. Workflow Enforcement
- ✅ Define strict sequential phases
- ✅ Add pre-flight checks for dependencies
- ✅ Include validation steps in workflow
- ❌ Don't allow agents to skip required phases

### 5. Testing & Quality
- ✅ Make backend tests mandatory
- ✅ Use smart test runners to save time
- ✅ Auto-format on save
- ❌ Don't skip validation steps

### 6. Error Handling
- ✅ Agents must STOP when dependencies missing
- ✅ Report exact errors with clear resolution steps
- ✅ Never proceed with assumptions
- ❌ Don't work around missing specifications

## Troubleshooting

### Agent Delegation Fails

**Error**: `Agent type 'backend-developer' not found`

**Solution**: Use the full prefixed name:
```
supabase-python-react-stack:backend-developer
```

### Orchestrator Creating Files Directly

**Symptom**: Orchestrator uses bash pipes or echo to create files.

**Solution**: This indicates delegation failed. Check:
1. Are you using the full prefixed agent name?
2. Is the target agent available in the plugin?
3. Review orchestrator constraints to ensure it can't work around failures

### Specs Directory Modified by Wrong Agent

**Symptom**: Specs modified by agents other than speckit-manager.

**Solution**: Check agent markdown files ensure they list `specs/**/*` as read-only. Only speckit-manager should have write access.

### Tests Not Running

**Symptom**: Smart test runner not detecting changes.

**Solution**: Check `.github/workflows/` and verify `pre_commit` hook configured correctly.

## Contributing

When extending this plugin:

1. Follow the existing agent naming convention
2. Update agent dependencies in markdown files
3. Maintain documentation consistency
4. Add any new architectural decisions to this README
5. Test delegation between agents
6. Verify hooks work correctly

## License

MIT License - see LICENSE file for details.

## Links

- GitHub: https://github.com/bodis/agents
- Plugin Marketplace: Claude Code Plugins
- Documentation: See individual agent markdown files in `agents/`
