---
name: implement
description: Coordinate feature implementation using implementation-orchestrator agent
argument-hint: [feature-id] or description
---

Use the **implementation-orchestrator** agent to coordinate feature development following API-first workflow.

## Instructions

1. **Analyze the request**:
   - If given a feature ID (e.g., `003`): Check `specs/` directory for existing plan
   - If given a description: Understand what needs to be built
   - If no arguments: Ask the user what they want to implement

2. **Load context**:
   - Read `CLAUDE.md` for project patterns
   - If Speckit plan exists: Load `plan.md` and `tasks.md`, identify what's done
   - Review current project state to understand existing implementation

3. **Create execution plan**:
   - Present clear execution order following API-first workflow
   - Show: Database → API → Backend → Frontend → Tests → Review → Documentation
   - Ask user to approve plan before proceeding

4. **Execute sequentially**:
   - Delegate each task to the appropriate specialized agent using: "Use the [agent-name] to [specific task]"
   - Wait for completion before proceeding to next task
   - Validate quality gates between tasks

5. **Report progress**:
   - After each task: Report what was created/modified
   - At completion: Summarize all changes and next steps

## Available Agents for Delegation

- `supabase-architect` - Database schemas, migrations, RLS
- `api-designer` - OpenAPI specifications
- `backend-developer` - FastAPI implementation
- `frontend-developer` - React/Next.js components
- `test-engineer` - Test suites
- `code-reviewer` - Code quality review
- `documentation-writer` - Update docs and CLAUDE.md
- `speckit-manager` - Update specs/ status (if applicable)

## Critical Rules

- **NEVER modify files directly** - Always delegate to specialized agents
- **Execute tasks sequentially** - Respect dependencies
- **Follow API-first** - Design before implementation
- **Validate quality gates** - Tests must pass before proceeding
- **Ask when unclear** - Don't assume requirements

Arguments provided: $ARGUMENTS