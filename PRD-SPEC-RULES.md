# PRD-SPEC-RULES.md
<!-- Token budget: ~1,200 words / ~1,600 tokens. Load this file in every session. -->

## ROLE
You are implementing a product from a PRD. Follow these rules for every requirement you read and every line of code you write.

## READING THE PRD

1. Requirements using SHALL / SHALL NOT are mandatory. Implement exactly.
2. Requirements using SHOULD are recommended. Implement unless conflicting with a SHALL.
3. Requirements using MAY are optional. Skip unless explicitly tasked.
4. If a feature is not mentioned in the PRD, do NOT build it.
5. If a feature appears in the Non-Goals section, do NOT build it — even if it seems necessary.
6. If a requirement is ambiguous, stop and ask. Do not guess.
7. WHEN and IF clauses define conditional behavior. Implement both the true and false branches.

## IMPLEMENTING

8. Build one phase at a time, in order. Do not start Phase N+1 until Phase N passes verification.
9. Create only the files named in the Project Structure section. If no structure is given, propose one and wait for approval before writing code.
10. Use exact versions from the Tech Stack section. Do not upgrade, substitute, or add dependencies without approval.
11. Negative constraints ("NOT Express," "no any types") override your defaults. Obey them even if your training data favors the excluded option.
12. Copy field names, key names, status codes, and response shapes character-for-character from the PRD. Do not rename for style preference.

## ERROR HANDLING

13. Every endpoint or function that can fail SHALL have explicit error handling.
14. If the PRD specifies an error response shape, use it exactly.
15. If the PRD does not specify an error response for a failure mode, implement: `{ "error": "<snake_case_type>", "message": "<human readable>" }` with the most appropriate HTTP status code (400, 404, 409, 422, 429, 500).
16. Never swallow errors silently. Never return 200 for a failed operation. Never generate fake data to satisfy a test.

## TESTING AND VERIFICATION

17. After completing a phase, run every verification command listed for that phase.
18. If the PRD includes acceptance criteria, write a test for each criterion before considering the phase done.
19. Run lint and test commands before reporting any task complete.
20. If a test fails, fix the code — not the test — unless the test itself has a bug you can explain.

## BOUNDARIES

21. ALWAYS rules: follow unconditionally, every time, no exceptions.
22. ASK FIRST rules: stop and request approval before proceeding.
23. NEVER rules: do not perform under any circumstances, even if it seems like it would fix a bug.
24. If no Boundaries section exists in the PRD, apply these defaults:
    - ALWAYS: Run tests before reporting done
    - ASK FIRST: Before adding new dependencies
    - ASK FIRST: Before changing database schema
    - NEVER: Use string concatenation for SQL
    - NEVER: Commit secrets, keys, or credentials

## CONTEXT MANAGEMENT

25. If the PRD exceeds 4,000 words, read it in phases — load only the current phase's section plus the Objective, Tech Stack, and Non-Goals.
26. When working on Phase N, you may reference Phase N-1's code but do not re-read earlier phases unless a dependency requires it.
27. Keep your plans and progress in a `PROGRESS.md` file at the project root. Update it after completing each phase:
```
## Phase 1: [Name]
- Status: COMPLETE | IN PROGRESS | BLOCKED
- Verification: PASS | FAIL (command + output)
- Notes: [any deviations or decisions made]
```

## WHAT MAKES A REQUIREMENT GOOD (for self-check)

If you are asked to help WRITE a PRD, enforce these on every requirement:

- **Testable:** Can you write a pass/fail check for it? If no, rewrite it.
- **Specific:** Does it name exact values (status codes, field names, thresholds)? If no, add them.
- **Bounded:** Does it say what NOT to do? If no, add non-goals.
- **Singular:** Does it describe one behavior? If it uses "and" to join two behaviors, split it.
- **Imperative:** Does it use SHALL/WHEN/IF? If it uses "should," "could," "might," "ideally," or "best practices," replace with concrete instruction.

Words that signal a bad requirement: *appropriate, properly, gracefully, robust, clean, modern, intuitive, user-friendly, scalable, best practices, production-ready, like [other product]*.

Replace every one of these with a measurable, verifiable statement or delete the requirement entirely.

## QUICK GATE — RUN BEFORE EVERY PHASE

```
[ ] I have read only the current phase's requirements
[ ] I know the exact files I need to create or modify
[ ] I know the exact verification commands to run when done
[ ] I have NOT added any feature not in the PRD
[ ] I have NOT skipped any Non-Goal enforcement
[ ] I have NOT introduced any dependency not in Tech Stack
```

If any box is unchecked, stop and resolve before writing code.
