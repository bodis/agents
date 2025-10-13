---
name: implement
description: Execute Speckit implementation plan using orchestrated agents following API-first workflow
usage: /implement [feature-number]
---

# Implementation Command

Execute a Speckit plan using the implementation-orchestrator agent to coordinate our specialized MVP stack agents.

## Usage

```bash
/implement 003
```

This will:
1. Load specs/003-*/plan.md
2. Activate implementation-orchestrator
3. Execute API-first workflow with specialized agents
4. Validate each step before proceeding
5. Report completion with full test coverage

## Available Agents for MVP Stack

The orchestrator will delegate to these specialized agents:
- **supabase-architect**: Database schemas, migrations, RLS policies
- **api-designer**: OpenAPI specifications, REST/SSE endpoints
- **backend-developer**: FastAPI implementation (Python)
- **frontend-developer**: React/Next.js UI components
- **test-engineer**: Test suites (pytest, Playwright)
- **code-reviewer**: Security and quality review
- **devops-engineer**: CI/CD and deployment configuration

## What It Does

```
Please activate the implementation-orchestrator agent to execute the implementation plan following our API-first workflow.

Feature: $ARGUMENTS
Plan file: specs/$ARGUMENTS-*/plan.md

Follow the API-First Development Flow:

STEP 1: Database Layer
- If database changes needed, delegate to supabase-architect
- Creates migrations in supabase/migrations/
- Updates docs/database/README.md

STEP 2: API Specification
- Delegate to api-designer
- Creates/updates docs/openapi.yaml
- Validates specification with openapi-spec-validator

STEP 3: Backend Implementation
- Delegate to backend-developer
- Implements FastAPI endpoints matching OpenAPI spec
- Reads from docs/openapi.yaml and docs/database/README.md

STEP 4: Frontend Implementation
- Delegate to frontend-developer
- Builds React components consuming API endpoints
- Uses shadcn/ui and follows docs/openapi.yaml

STEP 5: Testing
- Delegate to test-engineer
- Backend tests are MANDATORY (>80% coverage)
- Frontend tests for critical components only
- E2E tests for critical user flows

STEP 6: Code Review
- Delegate to code-reviewer
- Final security and quality check
- Ensures adherence to specifications

Important Rules:
- Execute tasks SEQUENTIALLY (never parallel)
- Each agent owns specific files - respect boundaries
- Documentation BEFORE implementation
- Backend tests are MANDATORY
- Only proceed when current step is complete
- Ask for input on ambiguity or blockers
- Provide clear progress updates at each step

Expected Output:
- Task-by-task execution status
- Files created/modified by each agent
- Test results after implementation
- Final summary with all deliverables
```