---
description: Verify your work before claiming completion. Evidence before assertions.
allowed-tools: Bash, Read, Glob, Grep
---

# Verification Before Completion

**No completion claims without fresh verification evidence.**

Before you claim anything is done, fixed, deployed, or passing, run this checklist.

## The Gate

For EVERY claim you're about to make:

1. **IDENTIFY**: What command or query proves this claim?
2. **RUN**: Execute it fresh right now (not from earlier in the session)
3. **READ**: Full output. Check status codes and actual values.
4. **VERIFY**: Does the output confirm what you're about to claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence attached

## Evidence Requirements

| You Claim | You Must Show |
|---|---|
| "Deployed edge function" | Live endpoint returns expected response |
| "Supabase rows created" | SELECT query with row count and sample |
| "Slack message posted" | Slack API query showing the message |
| "Attio record updated" | Attio API query showing field values |
| "Webhook triggered" | Response body or log entry from the handler |
| "Migration applied" | Schema query confirming new columns/tables |
| "Tests pass" | Test command output showing 0 failures |
| "Build succeeds" | Build command with exit 0 |
| "Bug fixed" | Reproduce original symptom, confirm it's gone |

## Red Flags (STOP)

You are about to violate this rule if you catch yourself:
- Using "should work now", "probably deployed", "seems correct"
- Reporting success based on HTTP 200 alone
- Trusting that a deploy command succeeded without checking the live endpoint
- Relying on a previous run's output after making changes
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!")
- Using "I'm confident" (confidence is not evidence)

## Rationalization Prevention

| You Think | You Must Do Instead |
|---|---|
| "Deploy succeeded, I saw no errors" | Query the live endpoint |
| "I ran the sync, it should be done" | Count the rows in Supabase |
| "Slack message sent, got 200" | Read the channel to confirm content |
| "This fix should resolve the issue" | Run the test/check that would fail if it didn't |
| "Linter passed so it compiles" | Run the actual build |
| "Agent said it completed" | Check the VCS diff for actual changes |

## Self-Review Checklist

Before reporting completion, also check:

**Code Quality:**
- No hardcoded values that should be in config/env
- No credentials, tokens, or secrets in code
- Timeouts set on all HTTP calls (30s default, 60s for bulk)
- No print() debugging left in code
- Imports are clean (no unused, no missing)

**Diff Review:**
- No unrelated changes in the diff
- No files accidentally deleted or overwritten
- Changes are scoped to the task (nothing extra)

## Run It

Now verify your work. Start with the most important claim and work through each one.
