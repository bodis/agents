# Claude Code Workflows & Agents

A comprehensive production-ready system combining **specialized AI agents**, **multi-agent workflow orchestrators**, and **development tools** for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Overview

This unified repository provides everything needed for intelligent automation and multi-agent orchestration across modern software development:

- **5 Specialized Agents** - Domain experts with deep knowledge across architecture, languages, infrastructure, quality, data/AI, documentation, business operations, and SEO
- **1 Workflow Orchestrators** - Multi-agent coordination systems for complex operations like full-stack development, security hardening, ML pipelines, and incident response
- **3 Development Tools** - Focused utilities for specific tasks including API scaffolding, security scanning, test automation, and infrastructure setup

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

Our agents follow a strict API-first workflow with clear ownership boundaries:

1. **supabase-architect** → Owns database schema (`supabase/migrations/*`, `docs/database/README.md`)
2. **api-designer** → Owns API contracts (`docs/openapi.yaml`)
3. **backend-developer** → Owns backend implementation (`backend/src/*`)
4. **frontend-developer** → Owns UI components (`frontend/src/*`)
5. **test-engineer** → Owns test suites (`*/tests/*`)
6. **code-reviewer** → Read-only quality gate
7. **implementation-orchestrator** → Coordinates the workflow

Each agent has defined file ownership - they can only modify their designated files and read others for reference.

### Example Flow: Adding User Notifications

```
User: "Add a notification system with real-time updates"

implementation-orchestrator:
  → Delegates to supabase-architect: "Create notifications table"
    ✓ Creates migration, updates docs/database/README.md

  → Delegates to api-designer: "Define notification endpoints"
    ✓ Creates REST + SSE specs in docs/openapi.yaml

  → Delegates to backend-developer: "Implement notification service"
    ✓ Builds FastAPI endpoints matching OpenAPI spec

  → Delegates to frontend-developer: "Create notification UI"
    ✓ Builds React components using API endpoints

  → Delegates to test-engineer: "Write tests"
    ✓ Creates pytest + Playwright tests

  → Delegates to code-reviewer: "Final review"
    ✓ Security & quality check

Result: Feature complete with proper separation of concerns
```

## Agent Categories

### xxxx

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