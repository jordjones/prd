---
name: prd
description: "Generate AI-optimized Product Requirements Documents. Use for: product requirements, PRD, specification, spec, requirements document, write a PRD, create requirements, spec out, plan a feature."
argument-hint: "[optional product description or workstream name]"
---

You are generating an AI-optimized PRD. Follow this workflow exactly.

## Step 1: Detect Existing PRDs

Scan the current directory and immediate children for files matching `PRD*.md` or `*-PRD.md`.

- **IF files found:** Present a numbered list and ask: "I found these existing PRDs: [list]. Would you like to: (1) Update one of these, (2) Create a new PRD for a different workstream, or (3) Start fresh and replace one?"
- **IF no files found OR user chooses new:** Proceed to Step 2.

## Step 2: Project Scan

Ask: "Would you like me to scan your project files (package.json, CLAUDE.md, src/, etc.) to auto-detect your tech stack and structure? [Y/n]"

**WHEN accepted**, read (never modify): `package.json`, `tsconfig.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`, `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/`, and top-level directory listing. Extract: language, framework, dependencies, build/test/lint commands, project structure.

## Step 3: Expertise Detection

Detect user expertise from responses throughout intake:
- **Technical:** Mentions specific technologies, uses precise terminology (API, schema, endpoint), references architecture patterns.
- **Non-technical:** Describes features as user stories ("I want users to..."), focuses on outcomes over implementation.
- **Expert:** Specifies versions, names constraints, mentions edge cases unprompted.

Adapt question depth accordingly — skip basics for technical users, ask about goals/outcomes for non-technical, ask only gaps for experts.

## Step 4: Hybrid Intake (5-8 Questions Max)

Ask questions in this order. Skip any already answered by `$ARGUMENTS` or project scan:

1. "Describe your product in 1-2 sentences. What does it do and for whom?" *(skip if argument provided)*
2. "What are the 3-5 core actions a user takes?"
3. "What should this product explicitly NOT do in v1?" *(critical — non-goals)*
4. "What's your tech stack?" *(skip if detected via project scan)*
5. "How will you verify it works? Any specific test scenarios?"
6-8. Adaptive follow-ups based on gaps: edge cases, error handling, phasing, performance targets, security model.

**WHEN all required information is gathered before 8 questions**, proceed directly to generation. Do NOT pad with unnecessary questions.

## Step 5: Generate PRD

Generate `PRD-<workstream>.md` where `<workstream>` is a slug from the product name (e.g., `PRD-url-shortener.md`).

Read `references/PRD-TEMPLATE.md` for the skeleton structure. The generated PRD SHALL use this exact section order:

```
# [Product Name] — PRD v1.0
## Objective
## Success Criteria
## Tech Stack
## Commands
## Project Structure
## Non-Goals / Out of Scope
## Boundaries
## Phase 1: [Name]
### Requirements
### Acceptance Criteria
### Verification
## Phase 2: [Name]
...
```

**Generation rules — enforce every one:**

1. Every requirement SHALL use EARS keywords: SHALL, SHALL NOT, WHEN, IF. NEVER use "should," "could," "might," "ideally," "appropriate," "proper," "clean," "modern," or "best practices" without a concrete definition.
2. Every endpoint or data-displaying component SHALL include: success response with exact JSON shape, at least one error response with HTTP status and JSON shape, input validation rules.
3. Every phase SHALL have: numbered requirements (max 15 per phase), checkbox acceptance criteria with testable conditions, a verification section with at least one executable command.
4. Non-Goals SHALL contain 5+ explicit exclusions. Infer likely non-goals from the product type (API → "No web UI," MVP → "No analytics dashboard," internal tool → "No public registration").
5. Boundaries SHALL include all three tiers: ALWAYS (2+, always including "run tests before reporting done"), ASK FIRST (2+, always including "before adding new dependencies"), NEVER (2+, always including "use parameterized queries for all database operations").
6. Tech Stack SHALL include exact version numbers. Use detected versions, stated versions, or latest stable LTS with "(verify version)" annotation.
7. Decompose into 3-6 phases. Phase 1 is always the minimal viable core. Each subsequent phase builds on the previous.
8. WHEN scope implies >6 phases, inform the user: "This scope suggests [N] phases. I recommend splitting into multiple PRDs by workstream. Shall I generate a PRD for just [core workstream] first?"

Read `references/PRD-SPEC-RULES.md` for additional operational rules to enforce.

## Step 6: Self-Audit

**Before presenting the PRD**, run it through these 5 quality gates:

| Gate | Check |
|------|-------|
| **Testable** | Can a pass/fail check be written for each requirement? Flag if no. |
| **Specific** | Does each requirement contain at least one concrete value (number, field name, status code, file path, format)? Flag purely qualitative reqs. |
| **Bounded** | Does Non-Goals exist with 5+ items? Does each phase have <=50 requirements? Flag violations. |
| **Singular** | Does each requirement describe exactly one behavior? Flag any using "and" to join two distinct behaviors. |
| **Imperative** | Does every requirement use SHALL/WHEN/IF? Flag any using "should," "could," "might," "ideally," "appropriate," "proper," "clean," "modern," "best practices," or "like [other product]." |

**IF `PRD-SPEC-RULES.md` exists** in the project root or in `references/`, also check every numbered rule from that file. Flag violations with rule number.

**Present results as:**
```
Quality Gate        | Status | Flags
--------------------|--------|------
Testable            | PASS   | 0
Specific            | PASS   | 0
Bounded             | PASS   | 0
Singular            | PASS   | 0
Imperative          | PASS   | 0
SPEC-RULES (R1-R27) | PASS   | 0
```

- **0 flags:** "PRD generated and passed all quality gates."
- **1+ flags:** List flagged requirements by gate. Offer: "I found [N] requirements that could be stronger. Want me to fix them, or review the PRD as-is first?"
- **Fix mode:** Rewrite ONLY flagged requirements, then re-audit the rewrites only.

## Step 7: Cross-Agent Output

After the PRD passes audit (or user accepts as-is), generate two additional files:

**AGENTS.md** (Codex-compatible):
- First line: `<!-- Generated by /prd from PRD-<workstream>.md. Regenerate with /prd, do not edit manually. -->`
- Sections: Overview (from Objective), Conventions (from Tech Stack + Boundaries), Commands (from Commands), Testing (from Verification sections), Constraints (from Non-Goals + Boundaries)
- IF existing AGENTS.md found, ask: "(1) Append this workstream's section, (2) Replace entirely, or (3) Skip AGENTS.md generation?"

**.cursor/rules/prd-<workstream>.mdc** (Cursor-compatible):
- Frontmatter: `description`, `globs: **/*`, `alwaysApply: true`
- Body: tech stack constraints, ALWAYS/NEVER boundary rules, phase awareness
- Max 100 lines total

**Notify user:**
> Generated 3 files:
> - `PRD-<workstream>.md` — Full PRD (primary spec)
> - `AGENTS.md` — Codex/OpenAI compatible context
> - `.cursor/rules/prd-<workstream>.mdc` — Cursor compatible rules
>
> The PRD is the source of truth. AGENTS.md and Cursor rules are derived — regenerate them by running `/prd` again if you update the PRD manually.

## Step 8: Refinement Loop

Present this menu after generation:

> Your PRD is ready. You can:
> 1. **Accept as-is** — save all files
> 2. **Refine** — tell me what to change and I'll update
> 3. **Add a phase** — extend the PRD with another phase
> 4. **Regenerate** — start the intake over from scratch
> 5. **Audit only** — re-run quality gates on the current draft

**Behaviors:**
- **Accept:** Write all 3 files to the project. Confirm with: "PRD saved. Start building with: 'Implement Phase 1 of PRD-<workstream>.md'"
- **Refine:** Accept natural language instructions. Apply targeted edits to referenced sections only — do NOT regenerate the full document.
- **Add a phase:** Determine next phase number, ask what it covers, append using same format (Requirements + Acceptance Criteria + Verification). Re-audit the new phase only.
- **Regenerate:** Restart from Step 4 (intake).
- **Audit only:** Re-run all 5 gates + SPEC-RULES check. Present updated table.

**Loop continues until the user selects Accept.**
