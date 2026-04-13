---
description: Systematic hypothesis-driven debugging. Use when something is broken.
argument-hint: [description of what's broken]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent
---

# Systematic Debugging

Do NOT jump to fixing. Follow this sequence to isolate the root cause first.

## Step 1: Identify What's Broken

Locate the relevant code/system from $ARGUMENTS or current context.

## Step 2: Gather Evidence

Read error messages, logs, and recent output. Write down:
- The exact error message or incorrect output
- When it last worked correctly (check git log, deploy history, logs)
- What changed between then and now (code, config, API, environment)

## Step 3: Form Hypotheses (max 3)

Based on evidence, list up to 3 possible causes ranked by likelihood:

1. **Most likely**: [hypothesis] because [evidence]
2. **Possible**: [hypothesis] because [evidence]
3. **Less likely**: [hypothesis] because [evidence]

Present these to the user before proceeding.

## Step 4: Test Hypotheses (cheapest test first)

For each hypothesis, identify the cheapest test that would confirm or rule it out:
- Can you reproduce the issue in isolation?
- Can you query the external API/service directly to rule out upstream issues?
- Can you compare working vs broken output to isolate the diff?
- Can you check the service's status page or changelog?

Run the cheapest test first. If it rules out hypothesis 1, move to hypothesis 2. Do not skip ahead.

## Step 5: Approach Abandonment Rule

If an approach fails twice with the same error:
1. STOP. Re-read the error message. Actually read it.
2. Ask: is there a fundamentally different approach?
3. Check if the API/service/dependency changed
4. Do NOT retry the same command hoping for a different result.

## Step 6: Fix and Verify

After identifying the root cause:
1. Implement the minimal fix
2. Test in the smallest scope possible first
3. Verify the original hypothesis was correct (not just that the symptom disappeared)
4. Run `/verify` before claiming it's fixed

## Step 7: Report

Summarize:
- **Root cause**: What was actually wrong
- **Hypothesis path**: Which hypotheses were tested and ruled out
- **Fix**: What was changed
- **Verification**: Evidence the fix works
- **Prevention**: Should anything change to prevent recurrence?
