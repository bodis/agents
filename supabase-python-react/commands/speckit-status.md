---
name: btstack.status
description: Check MVP stack implementation status and agent workflow progress
usage: /status
---

# Status Command

Check current implementation progress across the MVP stack layers and agent workflow.

## Usage

```bash
/status
```

## What It Does

```
Please report the current MVP stack implementation status:

## 1. Active Implementation Check

Check for any active implementation:
- Look for recent orchestrator activity
- Check if any agent is currently working
- Identify which stage of API-first workflow we're in

## 2. Stack Layer Status

### Database Layer (supabase-architect)
Check `supabase/migrations/` directory:
- Latest migration file and timestamp
- Check if `docs/database/README.md` exists and is up-to-date
- Count total tables defined

### API Layer (api-designer)
Check `docs/openapi.yaml`:
- File exists and last modified
- Count of endpoints defined
- Validation status (if openapi-spec-validator installed)

### Backend Layer (backend-developer)
Check `backend/` directory:
- FastAPI app status (src/main.py exists?)
- Number of service files in `backend/src/services/`
- Number of route files in `backend/src/api/`
- Test coverage if available

### Frontend Layer (frontend-developer)
Check `frontend/` directory:
- Next.js app status (app or pages directory?)
- Number of components in `frontend/src/components/`
- shadcn/ui components installed

### Testing Layer (test-engineer)
Check test directories:
- Backend tests: Count in `backend/tests/`
- Frontend tests: Count in `frontend/tests/`
- E2E tests: Count in `e2e/` or `tests/e2e/`
- Last test run results if available

### Infrastructure (devops-engineer)
Check deployment configs:
- `.github/workflows/` CI/CD pipelines count
- Docker files present (Dockerfile, docker-compose.yml)
- Environment files (.env.example)

## 3. Workflow Progress Report

If implementation is active:

📊 API-First Workflow Status
─────────────────────────────
Current Feature: [name from specs/]
Overall Progress: [X/6 stages complete]

Stage Status:
✅ 1. Database Schema    - Completed by supabase-architect
✅ 2. API Specification   - Completed by api-designer
🔄 3. Backend Implementation - IN PROGRESS (backend-developer)
⏸️ 4. Frontend Implementation - Pending
⏸️ 5. Testing              - Pending
⏸️ 6. Code Review          - Pending

Current Task: [specific task description]
Active Agent: [agent-name]
Files Being Modified: [list]

## 4. Documentation Sync Check

Verify critical documentation alignment:
- Database schema in `docs/database/README.md` matches migrations? [YES/NO]
- API spec in `docs/openapi.yaml` matches database schema? [YES/NO]
- Backend implementation matches API spec? [YES/NO]
- Frontend consuming correct endpoints? [YES/NO]

## 5. Recent Activity

Last 5 modifications by agents:
1. [timestamp] - [agent] modified [file]
2. [timestamp] - [agent] modified [file]
3. [timestamp] - [agent] modified [file]
4. [timestamp] - [agent] modified [file]
5. [timestamp] - [agent] modified [file]

## 6. Available for Implementation

Speckit plans ready to implement:
- specs/001-[name]/plan.md - [READY/NEEDS_REVIEW]
- specs/002-[name]/plan.md - [READY/NEEDS_REVIEW]
- specs/003-[name]/plan.md - [IN_PROGRESS/READY/NEEDS_REVIEW]

## 7. Test Coverage Summary

Backend (Python/FastAPI):
- Unit Tests: [X tests, Y% coverage]
- Integration Tests: [X tests]
- Status: [PASSING/FAILING/NOT_RUN]

Frontend (React/Next.js):
- Component Tests: [X tests]
- E2E Tests: [X tests]
- Status: [PASSING/FAILING/NOT_RUN]

## Output Format

=================================
       MVP STACK STATUS
=================================

🏗️ Implementation: [ACTIVE/IDLE]
📁 Current Feature: [name or N/A]
👤 Active Agent: [agent-name or N/A]
📈 Progress: [X/6 workflow stages]

Stack Health:
─────────────
✅ Database:  [migrations] migrations, [tables] tables
✅ API:       [endpoints] endpoints defined
✅ Backend:   [services] services, [routes] routes
✅ Frontend:  [components] components
✅ Tests:     [total] tests ([backend] backend, [frontend] frontend)
✅ CI/CD:     [pipelines] pipelines configured

Documentation Sync:
──────────────────
[✅/❌] Database schema documented
[✅/❌] API specification validated
[✅/❌] Backend matches spec
[✅/❌] Frontend integration aligned

Next Actions:
────────────
• [Current task if active]
• [Or next recommended action]

================================
```