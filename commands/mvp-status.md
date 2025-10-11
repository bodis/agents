---
name: status
description: Check implementation progress
usage: /status
---

# Status Command

Check current implementation progress.

## Usage

```bash
/status
```

## What It Does

```
Please report current implementation status:

1. Check for any active implementation (look for recent orchestrator activity)
2. If implementation in progress:
   - Which feature is being implemented
   - Current task being worked on
   - Tasks completed vs total
   - Test status
   - Any blockers
3. If no active implementation:
   - Last completed feature
   - Available Speckit plans ready for implementation

Format:
=Ê Implementation Status

Current Feature: [name]
Progress: [X/Y tasks]
Current Task: [description]
Agent: [agent-name]
Tests: [passing/failing]

Recent Completions:
- [Feature 1] - [date]
- [Feature 2] - [date]
```
