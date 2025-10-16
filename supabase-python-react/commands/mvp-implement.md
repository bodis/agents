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

## Execution Requirements

**MANDATORY for orchestrator:**

1. **MUST present execution plan** showing ALL phases before starting
2. **MUST execute phases sequentially** (no parallel execution)
3. **MUST validate outputs** after each phase before proceeding
4. **MUST justify any skipped phases** explicitly
5. **MUST delegate to ALL relevant agents** - do not skip agents unless explicitly justified

### Phase Validation Gates

The orchestrator MUST verify between phases:
- After supabase-architect → Check docs/datamodel.md exists and is updated
- After api-designer → Check docs/openapi.yaml exists with new endpoints
- After backend-developer → Check backend files created, validate against openapi.yaml
- After frontend-developer → Check frontend files created
- After test-engineer → Verify tests exist and pass
- After code-reviewer → Verify review approval received
- After documentation-writer → Verify docs updated
- After speckit-manager → Verify specs/ updated (if applicable)

### Hard Rules

❌ **NEVER proceed to backend-developer without docs/openapi.yaml**
❌ **NEVER proceed to api-designer without docs/datamodel.md (if DB changes)**
❌ **NEVER skip documentation-writer**
❌ **NEVER skip code-reviewer**
❌ **NEVER skip test-engineer for backend implementations**

## Critical Rules

- **NEVER modify files directly** - Always delegate to specialized agents
- **Execute tasks sequentially** - Respect dependencies
- **Follow API-first** - Design before implementation
- **Validate quality gates** - Tests must pass before proceeding
- **Ask when unclear** - Don't assume requirements

Arguments provided: $ARGUMENTS