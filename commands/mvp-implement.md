---
name: implement
description: Execute Speckit implementation plan using orchestrated agents
usage: /implement [feature-number]
---

# Implementation Command

Execute a Speckit plan using the implementation-orchestrator agent.

## Usage

```bash
/implement 003
```

This will:
1. Load specs/003-*/plan.md
2. Activate implementation-orchestrator
3. Execute tasks sequentially with appropriate agents
4. Run tests after each task
5. Report completion

## Arguments

- `feature-number`: The feature number from specs/ directory (e.g., 003)
- If omitted, will look for most recent feature

## What It Does

```
Please activate the implementation-orchestrator agent to execute the implementation plan.

Feature: $ARGUMENTS
Plan file: specs/$ARGUMENTS-*/plan.md

Follow these steps:
1. Read and parse the implementation plan
2. Present task breakdown to me for confirmation
3. Execute each task sequentially using the appropriate specialized agent
4. Ensure tests are written and passing after each backend task
5. Do NOT proceed to next task until current task is complete
6. Ask for my input if you encounter ambiguity or blockers
7. Provide clear progress updates
8. Report completion summary when all tasks done

Remember:
- Execute tasks ONE AT A TIME (never parallel)
- Backend tasks MUST have tests
- Only proceed when tests pass
- Stop and ask if anything is unclear
```
