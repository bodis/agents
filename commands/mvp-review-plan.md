---
name: review-plan
description: Review a Speckit plan before implementation
usage: /review-plan [feature-number]
---

# Review Plan Command

Review a Speckit implementation plan for completeness and clarity.

## Usage

```bash
/review-plan 003
```

## What It Does

```
Please review the implementation plan at specs/$ARGUMENTS-*/plan.md

Check for:
1. Clear task definitions
2. Proper task sequencing and dependencies
3. Missing technical details
4. Ambiguous requirements
5. Testing requirements
6. Potential technical challenges

Provide:
- Summary of the plan
- Identified issues or gaps
- Suggested improvements
- Estimated complexity
- Recommendation: READY / NEEDS_WORK
```
