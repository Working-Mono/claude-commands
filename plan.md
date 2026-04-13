---
description: Plan implementation with precise, granular tasks before building
argument-hint: <what to build or change>
allowed-tools: Bash, Read, Glob, Grep, Agent
---

# Plan Before Building

Break down $ARGUMENTS into precise, independently executable tasks.

## Step 1: Explore

Before writing any plan, investigate:

1. **Existing patterns**: Search the codebase for similar implementations
2. **Files involved**: Identify every file that will be created or modified
3. **Dependencies**: What systems, APIs, or data does this touch?
4. **Risks**: What could go wrong? What's the blast radius?

Use parallel exploration where possible.

## Step 2: Write Tasks

Each task MUST:

- Be completable in a single focused session (2-15 minutes)
- Name the exact files to create or modify
- Specify the function/class/block to change (not just "update the connector")
- Include acceptance criteria (what proves it's done)
- Be independently testable where possible

**"implement the sync" is too vague.**
**"add `fetch_page()` to `connectors/attio.py` returning paginated company records with cursor-based pagination and 1s backoff on rate limits" is correct.**

If a task can't be described at this granularity, break it down further.

## Step 3: Sequence

Order tasks by:
1. Foundation first (schemas, types, configs)
2. Core logic second (main functions, connectors)
3. Integration third (wiring, triggers, entry points)
4. Verification last (what proves the whole thing works)

Flag tasks that MUST be sequential vs tasks that can run in parallel.

## Step 4: Present

Output the plan as a numbered task list:

```
## Plan: {title}

### Context
{What exists, what we're building, key constraints}

### Tasks

1. **{Task name}** [complexity: 2]
   - Files: `path/to/file.py`
   - Do: {Precise description}
   - Done when: {Acceptance criteria}
   - Depends on: none

2. **{Task name}** [complexity: 3]
   - Files: `path/to/file.py`, `path/to/other.py`
   - Do: {Precise description}
   - Done when: {Acceptance criteria}
   - Depends on: Task 1

### Verification
{How to verify the whole thing works end-to-end}
```

Ask the user to review and approve before executing.
