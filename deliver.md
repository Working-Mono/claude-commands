---
description: Full Orchestrate → Build → Verify delivery loop. Use for any non-trivial implementation work.
argument-hint: <what to build or change>
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent
---

# Deliver: Orchestrate → Build → Verify

This command enforces the full PI agent delivery loop within a Claude Code session.
Three phases, strict sequencing. Do not skip phases or combine them.

---

## PHASE 1: ORCHESTRATE

You are the orchestrator. You route, plan, and gate. You do NOT build in this phase.

### 1a. Classify

Assess the request: $ARGUMENTS

- **Complexity 1-2** (toggle a setting, quick lookup, single-file fix): Skip to Phase 2 directly. No spec needed.
- **Complexity 3+** (multi-file, multi-system, new feature, integration work): Continue with full planning.

### 1b. Explore

Investigate in parallel before planning:
- **Codebase**: Search for existing patterns, similar implementations, reusable code
- **Context**: Read relevant config, context files, requirements
- **Impact**: What systems are affected? What could break? What's the blast radius?

Use the Agent tool with subagent_type "Explore" to run these in parallel.

### 1c. Write the Verification Spec

**THIS IS THE MOST IMPORTANT STEP. Do it BEFORE building.**

Write a verification spec that defines exactly what "done" looks like. The spec must contain concrete, testable assertions. Not vibes. Not "it should work." Actual checks.

Format:

```
## Verification Spec: {title}

### Assertions

1. [TYPE] {description}
   - Check: {exact command, query, or inspection to run}
   - Expected: {exact expected result}

2. [TYPE] {description}
   - Check: {exact command or query}
   - Expected: {exact expected result}

...

### Manual Review (if any)
- {Things that require human eyes: visual layout, tone, UX}
```

Assertion types:
- `[FILE]` - File exists with expected content
- `[BUILD]` - Code compiles/builds without errors
- `[TEST]` - Tests pass
- `[API]` - API endpoint returns expected response
- `[DB]` - Database query returns expected rows/values
- `[SLACK]` - Slack message exists with expected content
- `[ATTIO]` - CRM record has expected field values
- `[HTTP]` - URL is reachable and returns expected status
- `[IDEMPOTENT]` - Running twice doesn't create duplicates
- `[CROSS-SYSTEM]` - Values match between two systems

**Present the verification spec to the user for approval before proceeding to Phase 2.**

### 1d. Write the Task Breakdown

Break the work into precise, granular tasks (same rules as `/plan`):
- Each task completable in 2-15 minutes
- Exact file paths and function names
- Each task references which verification assertions it satisfies

---

## PHASE 2: BUILD

You are the builder. You implement, deploy, trigger. You do NOT verify in this phase.

### 2a. Dispatch Builder

Launch a builder subagent in an isolated worktree:

```
Agent(
  subagent_type: "general-purpose",
  isolation: "worktree",
  prompt: """
  You are a BUILDER agent. You implement code changes and deploy them.

  YOUR CONSTRAINTS:
  - You CANNOT claim the work is done. You report what you changed and what you triggered.
  - You CANNOT skip steps in the task list.
  - You CANNOT modify the verification spec.

  VERIFICATION SPEC (you are building to pass this):
  {paste the verification spec from Phase 1}

  TASKS:
  {paste the task breakdown from Phase 1}

  INSTRUCTIONS:
  1. Work through each task in order
  2. For each task, implement the change, then confirm it with evidence
  3. After all tasks, run the self-review checklist:
     - No hardcoded values that should be in config/env
     - No credentials or secrets in code
     - Timeouts on all HTTP calls (30s default, 60s for bulk)
     - No print() debugging left in code
     - Imports clean (no unused, no missing)
     - No unrelated changes in diff
     - Idempotent where applicable
  4. Report back:
     - What files were changed (list with brief description)
     - What was deployed (if anything)
     - What was triggered (if anything)
     - Any concerns or open questions
     - The git branch name with your changes

  DO NOT say "done" or "complete". Report what you did and let the verifier check it.
  """
)
```

If the work is simple (complexity 1-2), skip the subagent and build directly, but still follow the self-review checklist.

### 2b. Review Builder Output

Read the builder's report. Check:
- Did it address every task in the breakdown?
- Are there concerns or open questions to resolve before verification?
- Does the branch exist and contain the expected changes?

If the builder reported concerns, address them before moving to Phase 3.

---

## PHASE 3: VERIFY

You are the verifier. You are adversarial. You assume things are broken until proven otherwise.

### 3a. Dispatch Verifier

Launch a verifier subagent (NOT in a worktree, it reads the builder's changes):

```
Agent(
  subagent_type: "general-purpose",
  prompt: """
  You are a VERIFIER agent. You are adversarial. You FIND PROBLEMS.

  YOUR CONSTRAINTS:
  - You CANNOT edit any files or write any code.
  - You CANNOT deploy anything.
  - You CANNOT fix problems. You can only report them.
  - You CANNOT say "looks good" without running every assertion.

  YOUR MINDSET:
  - Assume everything is broken until you prove otherwise.
  - HTTP 200 does not mean correct. Data could be wrong, duplicated, or missing.
  - "No errors" does not mean "works correctly."
  - The builder has completion bias. Your job is to counter it.

  VERIFICATION SPEC:
  {paste the verification spec from Phase 1}

  BUILDER REPORT:
  {paste the builder's output from Phase 2}

  INSTRUCTIONS:
  1. Go through EVERY assertion in the verification spec, one by one
  2. For each assertion:
     a. Run the exact check specified
     b. Record the actual result
     c. Mark PASS, FAIL, or SKIP (with reason)
  3. For any FAIL: explain exactly what's wrong and what was expected
  4. DO NOT suggest fixes. That's the builder's job.
  5. Report:

  ## Verification Results

  | # | Type | Assertion | Result | Evidence |
  |---|---|---|---|---|
  | 1 | [TYPE] | {description} | PASS/FAIL | {what you observed} |
  | 2 | [TYPE] | {description} | PASS/FAIL | {what you observed} |

  ### Failures (if any)
  - Assertion {N}: {what's wrong, what was expected vs actual}

  ### Manual Review Items
  - {items that require human review, flagged for the user}

  ### Overall: PASS / FAIL
  """
)
```

### 3b. Gate the Result

Based on verifier output:

**All assertions PASS:**
- Report to user: work is complete, here's the evidence
- List any manual review items that need human eyes

**Any assertions FAIL (attempt 1 or 2):**
- Show the user the failures
- Return to Phase 2: dispatch builder again with the specific failures
- The builder receives: the original spec + the verifier's failure report
- After builder fixes, return to Phase 3: dispatch verifier again

**Failures persist after 3 attempts:**
- STOP. Escalate to the user.
- Report: what was tried, what keeps failing, what you think the blocker is
- Do not keep retrying. Something fundamental needs human judgment.

---

## FLOW SUMMARY

```
ORCHESTRATE          BUILD              VERIFY

classify ──→ spec ──→ builder ──→ verifier
  │            │        │            │
  │            │        │       PASS? ──→ DONE (report with evidence)
  │            │        │            │
  │            │        │       FAIL? ──→ back to builder (max 3x)
  │            │        │            │
  │            │        │       3x FAIL? → escalate to human
  │            │
  │      (user approves spec
  │       before building starts)
  │
  simple? ──→ build directly, self-review, report
```

---

## RULES

1. **Spec before code.** Never start building without a verification spec (unless complexity 1-2).
2. **No self-verification.** The agent that built it does not verify it. Use separate subagents.
3. **Evidence over claims.** Every assertion needs a command run and output shown.
4. **Three strikes.** Three failed verification cycles = escalate, don't loop.
5. **Human gates the spec.** Present the verification spec for approval before building starts.
