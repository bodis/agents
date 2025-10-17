---
name: btstack.implement
description: Coordinate feature implementation using implementation-orchestrator agent
argument-hint: [feature-id] or description
---

**CRITICAL: You MUST immediately delegate this entire task to the implementation-orchestrator agent.**

**DO NOT attempt to implement, plan, or coordinate this yourself. Your ONLY job is to invoke the Task tool with subagent_type="supabase-python-react-stack:implementation-orchestrator" and pass the user's request.**

## Delegation Instructions

Invoke the Task tool with:
- **subagent_type**: `"supabase-python-react-stack:implementation-orchestrator"`
- **description**: `"Implement feature: $ARGUMENTS"`
- **prompt**: Use the template below

### Prompt Template

```
The user wants to implement: $ARGUMENTS

Context:
- If this is a feature ID (e.g., "003" or "001-feature-name"), check the specs/ directory for existing Speckit plans
- If this is a description, treat it as a new feature request
- The project follows API-first development workflow (Database → API → Backend → Frontend → Tests → Review → Docs)

Your task:
1. Analyze the request and load relevant context (CLAUDE.md, specs/, current implementation)
2. Create and present an execution plan showing all required phases
3. **Immediately execute** the plan by delegating to specialized agents sequentially (do NOT wait for approval)
4. Validate each phase before moving to the next
5. Report progress and completion status

Execute autonomously following your defined workflow. Only ask for user input when truly blocked.
```

## Notes

- The orchestrator agent knows all workflow rules, available agents, and quality gates
- No need to repeat instructions here - the orchestrator's agent definition is the single source of truth
- Just pass the user's request and let the orchestrator handle everything
