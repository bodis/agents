---
name: implement
description: Execute Speckit implementation plan using implementation-orchestrator agent
usage: /implement [feature-number]
---

# Implementation Command

Execute a Speckit plan using the **implementation-orchestrator** agent to coordinate our specialized MVP stack agents.

## Usage

```bash
/implement 003
```

This will:
1. Find the relevant specs inder the specs/
2. Load plan.md from the identified specs director
3. Identify what is done from the plan
4. maybe for better understanding load the tasks.md for identifying the related task list element (BUT YOU HAVE TO RECREATE YOUR OWN TASKS NOT USING THIS DIRECTLY)
5. Execute API-first workflow with specialized agents
6. Validate each step before proceeding
7. Report completion with full test coverage

## Available Agents for the orchestrator

The orchestrator will delegate to these specialized agents:
- **supabase-architect**: Database schemas, migrations, RLS policies
- **api-designer**: OpenAPI specifications, REST/SSE endpoints
- **backend-developer**: FastAPI implementation (Python)
- **frontend-developer**: React/Next.js UI components
- **test-engineer**: Test suites (pytest, Playwright)
- **code-reviewer**: Security and quality review
- **devops-engineer**: CI/CD and deployment configuration
