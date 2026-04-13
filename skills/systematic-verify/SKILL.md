---
name: systematic-verify
description: "TRIGGER automatically before ANY completion claim. When you are about to say work is done, fixed, deployed, passing, complete, or ready, this skill MUST fire first. Also trigger when the user asks to check, verify, or confirm something works."
---

# Verification Before Completion

**No completion claims without fresh verification evidence.**

This fires automatically before you claim anything is done.

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
| "Deployed" | Live endpoint returns expected response |
| "Rows created" | SELECT query with row count and sample |
| "Message posted" | API query showing the message content |
| "Record updated" | API query showing field values |
| "Tests pass" | Test output showing 0 failures |
| "Build succeeds" | Build command with exit 0 |
| "Bug fixed" | Original symptom confirmed gone |
| "Migration applied" | Schema query confirming changes |

## Red Flags (STOP)

You are about to violate this if you catch yourself:
- Saying "should work now", "probably deployed", "seems correct"
- Reporting success based on HTTP 200 alone
- Relying on a previous run's output after making changes
- About to say "Great!", "Perfect!", "Done!" before running checks
- Thinking "I'm confident" (confidence is not evidence)

## Self-Review

Before reporting, also check:
- No hardcoded values that should be in config/env
- No credentials or secrets in code
- No print() debugging left
- No unrelated changes in diff
- Changes scoped to the task

## Then report with evidence attached.
