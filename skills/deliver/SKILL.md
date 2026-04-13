---
name: deliver
description: "TRIGGER automatically when the user asks to build, implement, create, add, or deploy something that touches 2+ files or involves integration work. This includes: new features, new automations, new connectors, API integrations, data pipelines, multi-step implementation, or any request with complexity >= 3. Do NOT trigger for: questions, exploration, lookups, single-file edits, config changes, or quick fixes."
---

# Deliver: Orchestrate → Build → Verify

This skill enforces a structured delivery loop. Three phases, strict sequencing. Do not skip phases or combine them.

## When This Triggers

You should follow this workflow when the user's request involves:
- Building something new (feature, automation, connector, pipeline)
- Multi-file implementation work
- Deploying to external systems
- Integration between two or more systems
- Any work where "did it actually work?" is a non-trivial question

Skip this for simple tasks: single-file edits, config changes, lookups, questions.

---

## PHASE 1: ORCHESTRATE

You plan and define "done." You do NOT build in this phase.

### 1a. Classify Complexity

Assess the request:
- **Complexity 1-2** (single-file fix, config change, quick lookup): Handle directly. No spec needed. Still self-review before claiming done.
- **Complexity 3+** (multi-file, multi-system, new feature): Continue with full spec.

### 1b. Explore

Before planning, investigate in parallel:
- **Codebase**: Existing patterns, similar implementations, reusable code
- **Context**: Config files, requirements, constraints
- **Impact**: Systems affected, blast radius, what could break

### 1c. Write the Verification Spec

**Write this BEFORE building. This is the single most important step.**

The spec defines exactly what "done" looks like with concrete, testable assertions.

```
## Verification Spec: {title}

### Assertions

1. [TYPE] {description}
   - Check: {exact command, query, or inspection}
   - Expected: {exact expected result}

2. [TYPE] {description}
   - Check: {exact command or query}
   - Expected: {exact expected result}

### Manual Review (if any)
- {Things requiring human eyes}
```

Assertion types: `[FILE]`, `[BUILD]`, `[TEST]`, `[API]`, `[DB]`, `[SLACK]`, `[ATTIO]`, `[HTTP]`, `[IDEMPOTENT]`, `[CROSS-SYSTEM]`

**Present the spec to the user for approval before Phase 2.**

### 1d. Task Breakdown

Break work into precise tasks:
- Each completable in 2-15 minutes
- Exact file paths and function names
- Each task references which assertions it satisfies

---

## PHASE 2: BUILD

You implement. You do NOT verify completeness in this phase.

### For complexity 3+: Dispatch a builder subagent

Launch in an isolated worktree so the builder has a clean workspace:

```
Agent(
  isolation: "worktree",
  prompt: "You are a BUILDER. You implement and report. You CANNOT claim done.

  VERIFICATION SPEC (build to pass this):
  {spec from Phase 1}

  TASKS:
  {task breakdown from Phase 1}

  INSTRUCTIONS:
  1. Implement each task in order
  2. Self-review before reporting:
     - No hardcoded values that belong in config/env
     - No credentials in code
     - Timeouts on all HTTP calls
     - No print() debugging left
     - No unrelated changes in diff
  3. Report what you changed, deployed, and triggered. Do NOT claim done."
)
```

### For complexity 1-2: Build directly

Implement the change, but still self-review before moving on.

### Review builder output

- Did it address every task?
- Any concerns to resolve before verification?

---

## PHASE 3: VERIFY

You are adversarial. You assume things are broken until proven otherwise.

### Dispatch a verifier subagent

```
Agent(
  prompt: "You are a VERIFIER. You FIND PROBLEMS. You are adversarial.

  CONSTRAINTS:
  - You CANNOT edit files or write code
  - You CANNOT deploy anything
  - You CANNOT fix problems, only report them
  - You CANNOT say 'looks good' without running every assertion

  MINDSET:
  - Assume broken until proven otherwise
  - HTTP 200 does not mean correct
  - 'No errors' does not mean 'works correctly'

  VERIFICATION SPEC:
  {spec from Phase 1}

  BUILDER REPORT:
  {output from Phase 2}

  INSTRUCTIONS:
  Run EVERY assertion. For each one:
  1. Execute the exact check
  2. Record actual result
  3. Mark PASS, FAIL, or SKIP (with reason)
  4. For FAILs: explain what's wrong vs expected
  5. DO NOT suggest fixes

  Report as a table:
  | # | Type | Assertion | Result | Evidence |
  Then list failures and manual review items.
  Overall: PASS or FAIL"
)
```

### Gate the result

- **All PASS**: Report to user with evidence. Flag manual review items.
- **Any FAIL (attempt 1-2)**: Return to Phase 2 with specific failures. Builder fixes, verifier re-runs.
- **3x FAIL**: STOP. Escalate to user with what was tried and what keeps failing.

---

## RULES

1. **Spec before code.** Never build without defining what "done" looks like first.
2. **No self-verification.** The context that built it should not verify it. Use separate subagents.
3. **Evidence over claims.** Every assertion needs a command run and output shown.
4. **Three strikes.** Three failed verification cycles = escalate, don't loop.
5. **Human gates the spec.** User approves the verification spec before building starts.
