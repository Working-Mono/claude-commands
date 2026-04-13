---
name: systematic-debug
description: "TRIGGER automatically when the user reports something is broken, failing, erroring, not working, returning wrong results, or behaving unexpectedly. Also trigger when investigating bugs, errors, exceptions, or unexpected behavior. Do NOT jump to fixing. Follow the hypothesis-driven sequence."
---

# Systematic Debugging

Do NOT jump to fixing. Isolate the root cause first.

## Step 1: Gather Evidence

- The exact error message or incorrect output
- When it last worked (git log, deploy history, logs)
- What changed between then and now

## Step 2: Form Hypotheses (max 3)

Based on evidence, rank by likelihood:
1. **Most likely**: [hypothesis] because [evidence]
2. **Possible**: [hypothesis] because [evidence]
3. **Less likely**: [hypothesis] because [evidence]

Present these to the user before proceeding.

## Step 3: Test Hypotheses (cheapest test first)

For each, find the cheapest confirming/ruling-out test:
- Reproduce in isolation?
- Query the external service directly?
- Compare working vs broken output?
- Check the service's status page or changelog?

Run cheapest first. If it rules out #1, move to #2.

## Step 4: Approach Abandonment

If an approach fails twice with the same error:
1. STOP. Re-read the error. Actually read it.
2. Is there a fundamentally different approach?
3. Did the API/service/dependency change?
4. Do NOT retry the same thing hoping for different results.

## Step 5: Fix and Verify

After identifying root cause:
1. Implement minimal fix
2. Test in smallest scope possible
3. Confirm the hypothesis was correct (not just that the symptom disappeared)
4. Verify with evidence before claiming fixed

## Step 6: Report

- **Root cause**: What was actually wrong
- **Hypothesis path**: What was tested and ruled out
- **Fix**: What changed
- **Evidence**: Proof it works
- **Prevention**: What to change to prevent recurrence
