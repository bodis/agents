---
name: speckit-review-plan
description: Review Speckit plan for compatibility with stack agents and tech stack
usage: /speckit-review-plan [feature-number]
---

# Speckit Review Plan Command

Review a Speckit implementation plan to ensure it can be executed by our stack agents and aligns with our tech stack.

## Usage

```bash
/speckit-review-plan 003
```

## What It Does

```
Please review the implementation plan at specs/$ARGUMENTS-*/plan.md for compatibility with our stack.

## 1. Tech Stack Compatibility Check

Verify the plan aligns with:
- **Backend**: Python/FastAPI with Pydantic v2
- **Frontend**: React/Next.js with TypeScript & shadcn/ui
- **Database**: Supabase (PostgreSQL with RLS)
- **API Design**: OpenAPI 3.0.0 specification
- **Real-time**: SSE (Server-Sent Events), NOT WebSockets
- **Deployment**: GCP (Cloud Run + Cloud Storage)

Flag any requirements that don't match our stack.

## 2. Agent Capability Validation

For each task in the plan, verify we have the right agent:

Database Tasks → supabase-architect can handle:
- PostgreSQL table creation/modification
- RLS policies
- Database migrations
- Indexes and constraints

API Tasks → api-designer can handle:
- REST endpoint definitions
- SSE endpoint specifications
- OpenAPI 3.0.0 schemas
- Request/response validation rules

Backend Tasks → backend-developer can handle:
- FastAPI endpoint implementation
- Pydantic models
- Supabase client integration
- SSE implementation
- Business logic in Python

Frontend Tasks → frontend-developer can handle:
- React/Next.js components
- shadcn/ui integration
- React Query for data fetching
- Zod validation
- TypeScript implementations

Testing Tasks → test-engineer can handle:
- pytest for backend
- Jest/Vitest for frontend
- Playwright for E2E
- Test coverage requirements

Infrastructure Tasks → devops-engineer can handle:
- GitHub Actions CI/CD
- Docker configurations
- GCP deployments
- Environment configurations

## 3. Workflow Feasibility

Check if the plan follows API-first development:
1. Database schema defined first?
2. API specification before implementation?
3. Clear dependencies between tasks?
4. Testing requirements specified?
5. Sequential execution possible?

## 4. Identify Gaps & Blockers

Look for:
- Tasks requiring technologies outside our stack
- Missing technical specifications
- Ambiguous requirements
- Tasks no agent can handle
- Circular dependencies
- Missing test requirements
- Authentication/authorization gaps
- Performance considerations not addressed

## 5. Provide Assessment

READY TO IMPLEMENT if:
✅ All tasks map to available agents
✅ Tech stack requirements match
✅ API-first workflow possible
✅ Clear task dependencies
✅ Testing requirements defined

NEEDS CLARIFICATION if:
⚠️ Ambiguous task definitions
⚠️ Missing technical details
⚠️ Unclear agent assignments

CANNOT IMPLEMENT if:
❌ Requires tech outside our stack
❌ Tasks beyond agent capabilities
❌ Incompatible with API-first flow
❌ Critical information missing

## Output Format

1. **Compatibility Summary**
   - Tech stack alignment: [COMPATIBLE/ISSUES]
   - Agent coverage: [X/Y tasks covered]

2. **Task-to-Agent Mapping**
   - Task 1: [agent-name] ✅/⚠️/❌
   - Task 2: [agent-name] ✅/⚠️/❌
   - ...

3. **Identified Issues**
   - [List specific problems]

4. **Recommendations**
   - [Specific fixes needed]

5. **Final Verdict**
   - READY TO IMPLEMENT / NEEDS CLARIFICATION / CANNOT IMPLEMENT
   - Confidence: [High/Medium/Low]
```