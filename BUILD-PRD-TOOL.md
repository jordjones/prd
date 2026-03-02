# BUILD-PRD-TOOL.md
# 1-Shot Claude Code Prompt + PRD for the /prd Slash Command Skill

> **What this file is:** Hand this file to a fresh Claude Code instance alongside
> `AI_PRD_FRAMEWORK.md` and `PRD-SPEC-RULES.md` from this project. Claude Code
> will read this file and build the entire `/prd` slash command skill.
>
> **What this file is NOT:** This is not the skill itself. This is the blueprint
> that tells Claude Code how to build the skill.

---

## ═══════════════════════════════════════════════
## PART 1: THE 1-SHOT PROMPT
## ═══════════════════════════════════════════════

Copy-paste the block below (between the `>>>` markers) as your first message
to Claude Code in a new project directory.

>>> START PROMPT >>>

You are building an open-source Claude Code skill called `prd-generator` that
creates a `/prd` slash command. This command helps users write AI-optimized
Product Requirements Documents.

**Read these files first, in order, before writing any code:**
1. `BUILD-PRD-TOOL.md` — This file. Contains the full PRD for what you're building. Read PART 2 completely.
2. `AI_PRD_FRAMEWORK.md` — The research-backed framework the /prd command teaches and enforces. This is reference material the skill will point users to, not something to embed in full.
3. `PRD-SPEC-RULES.md` — The operational rules file. Core rules from this file get embedded in the skill. The full file ships as a reference.

**After reading all three files, execute the PRD in PART 2 of BUILD-PRD-TOOL.md phase by phase.**

Do not skip phases. Do not start Phase N+1 until Phase N passes its verification. Ask me before adding any dependency not specified in the PRD.

>>> END PROMPT >>>

---

## ═══════════════════════════════════════════════
## PART 2: THE PRD
## ═══════════════════════════════════════════════

---

# prd-generator — PRD v1.0

## Objective

`prd-generator` is a Claude Code skill that registers a `/prd` slash command.
When invoked, `/prd` guides the user through a hybrid intake process (quick
structured questions → full PRD generation → self-audit → refinement loop),
then outputs an AI-optimized Product Requirements Document and cross-agent
context files (AGENTS.md for Codex, .cursor/rules/ for Cursor). The tool
adapts its depth and technical language based on the user's expertise level,
detected during intake.

## Success Criteria

- [ ] Running `/prd` in Claude Code launches the hybrid intake flow
- [ ] Running `/prd My cool app idea` passes the argument as the initial product description, skipping the first intake question
- [ ] The intake phase completes in 5–8 questions, adapting based on user responses
- [ ] A single `PRD-<workstream>.md` file is generated matching the template structure
- [ ] The generated PRD passes self-audit: 0 requirements flagged as untestable, unspecific, or unbounded
- [ ] `AGENTS.md` is generated as a Codex-compatible derivative
- [ ] `.cursor/rules/prd.mdc` is generated as a Cursor-compatible derivative
- [ ] If existing `PRD*.md` files are found, the user is prompted to update, overwrite, or create new
- [ ] The skill loads in < 2 seconds and consumes < 1,600 tokens of context budget
- [ ] A README.md with installation instructions, usage examples, and extension guide is included

## Tech Stack

- Platform: Claude Code skill (SKILL.md with YAML frontmatter)
- Language: Markdown prompts (no compiled code — this is a prompt-based skill)
- File format: `.md` for all skill files, `.mdc` for Cursor rules
- Standard: Agent Skills open standard (compatible with Claude Code, Codex, Cursor, Gemini CLI)
- License: MIT

## Commands

```bash
# Installation (user runs these)
mkdir -p ~/.claude/skills/prd-generator
cp -r prd-generator/* ~/.claude/skills/prd-generator/

# Verification (after installation)
# In any Claude Code session, type:
/prd
# Expect: intake flow begins with project scan offer

# Alternative: project-scoped installation
mkdir -p .claude/skills/prd-generator
cp -r prd-generator/* .claude/skills/prd-generator/
```

## Project Structure

```
prd-generator/
├── SKILL.md                    # Main skill file → registers /prd command
├── references/
│   ├── PRD-SPEC-RULES.md       # Operational rules (copied from source)
│   ├── PRD-TEMPLATE.md         # Skeleton template for generation
│   └── EXAMPLES.md             # Annotated perfect/mid/terrible examples
├── README.md                   # Open-source docs: install, use, extend
├── LICENSE                     # MIT license
└── CHANGELOG.md                # Version history
```

## Non-Goals / Out of Scope

- No web UI, dashboard, or browser-based interface
- No MCP server — this is a local skill only
- No integration with project management tools (Jira, Linear, etc.)
- No automatic code generation from the PRD (the PRD is the output, not code)
- No multi-user collaboration or real-time editing
- No cloud storage, sync, or account system
- No telemetry, analytics, or usage tracking
- No modification of existing source code in the user's project
- No installation of npm packages, pip packages, or any runtime dependencies

## Boundaries

- ALWAYS: Read existing PRD files before generating new ones
- ALWAYS: Run self-audit on generated PRD before presenting to user
- ALWAYS: Ask before overwriting any existing file
- ASK FIRST: Before creating files outside the current working directory
- ASK FIRST: Before scanning project files (offer, don't assume)
- NEVER: Modify source code, config files, or any non-PRD file in the user's project
- NEVER: Make assumptions about tech stack if the user hasn't stated one and project scan was declined
- NEVER: Generate a PRD with requirements that use "should," "could," "appropriate," "proper," or "best practices" without concrete definitions

---

## Phase 1: SKILL.md — The Core Slash Command

### Requirements

1. SKILL.md SHALL have YAML frontmatter with:
   - `name: prd`
   - `description:` a 2–3 sentence trigger description that includes phrases: "product requirements," "PRD," "specification," "spec," "requirements document," "write a PRD," "create requirements," "spec out," "plan a feature"
   - `argument-hint: [optional product description or workstream name]`

2. WHEN `/prd` is invoked, the skill SHALL first check for existing `PRD*.md` and `*-PRD.md` files in the current directory and its immediate children

3. WHEN existing PRD files are found, the skill SHALL present them as a numbered list and ask:
   "I found these existing PRDs: [list]. Would you like to: (1) Update one of these, (2) Create a new PRD for a different workstream, or (3) Start fresh and replace one?"

4. WHEN no existing PRDs are found OR the user chooses to create new, the skill SHALL ask:
   "Would you like me to scan your project files (package.json, CLAUDE.md, src/, etc.) to auto-detect your tech stack and structure? [Y/n]"

5. WHEN project scan is accepted, the skill SHALL read (not modify): `package.json`, `tsconfig.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`, `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/`, and the top-level directory structure. It SHALL extract: language, framework, existing dependencies, build/test/lint commands, and project structure.

6. The skill SHALL detect user expertise level from intake responses using these signals:
   - **Technical indicators:** mentions specific technologies, uses precise terminology (API, schema, endpoint, middleware), references architecture patterns
   - **Non-technical indicators:** describes features in user-story language ("I want users to be able to..."), focuses on outcomes over implementation, asks what things mean
   - **Expert indicators:** specifies versions, names constraints, mentions edge cases unprompted

7. The skill SHALL adapt its intake questions based on detected expertise:
   - **For technical users:** skip basics, ask about architecture constraints, edge cases, performance targets, security model
   - **For non-technical users:** ask about user goals, core user actions, what success looks like, what the product should NOT do
   - **For expert users:** ask only about non-goals, boundaries, phasing strategy, and verification approach — assume they'll provide the rest

8. The hybrid intake flow SHALL proceed in this order:
   - Q1: "Describe your product in 1–2 sentences. What does it do and for whom?" (skip if argument was provided)
   - Q2: "What are the 3–5 core actions a user takes?" (adapt phrasing to expertise)
   - Q3: "What should this product explicitly NOT do in v1?" (critical — non-goals)
   - Q4: Tech stack question (skip if detected via project scan)
   - Q5: "How will you verify it works? Any specific test scenarios?" (adapt to expertise)
   - Q6–Q8: Adaptive follow-ups based on gaps detected in answers (edge cases, error handling, phasing)

9. The skill SHALL complete intake in no more than 8 questions total. WHEN all required information is gathered before 8 questions, it SHALL proceed directly to generation.

10. After intake, the skill SHALL generate a `PRD-<workstream>.md` file where `<workstream>` is a slug derived from the product name or feature area (e.g., `PRD-url-shortener.md`, `PRD-auth-service.md`, `PRD-onboarding-flow.md`)

### Acceptance Criteria

- [ ] `/prd` launches intake flow in Claude Code
- [ ] `/prd Build a REST API for task management` skips Q1 and uses the argument
- [ ] Existing PRD detection finds `PRD-auth.md` in project root when present
- [ ] Project scan correctly identifies tech stack from `package.json`
- [ ] Intake completes in 5–8 questions
- [ ] Generated filename follows `PRD-<slug>.md` convention

### Verification

```bash
# After creating SKILL.md, verify frontmatter parses correctly:
head -10 prd-generator/SKILL.md
# Expect: valid YAML between --- markers with name: prd

# Verify the skill appears in Claude Code:
# In a Claude Code session, type /prd and press Tab
# Expect: /prd appears in autocomplete with description
```

---

## Phase 2: PRD Generation Engine (within SKILL.md)

### Requirements

1. The generation logic SHALL be embedded in the SKILL.md body as structured prompt instructions (not external scripts or code files)

2. The generated PRD SHALL follow this exact section order:
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

3. Every generated requirement SHALL use EARS keywords: SHALL, SHALL NOT, WHEN, IF. The generation logic SHALL NOT produce requirements using "should," "could," "might," "ideally," or "best practices."

4. Every generated endpoint or data-displaying component SHALL include:
   - Success response with exact JSON shape
   - At least one error response with HTTP status code and JSON shape
   - Input validation rules with specific criteria

5. Every generated phase SHALL include:
   - A numbered requirements list (max 15 requirements per phase)
   - Acceptance criteria as a checkbox list with testable conditions
   - A verification section with at least one executable command

6. The generated Non-Goals section SHALL contain at least 5 explicit exclusions. The skill SHALL infer likely non-goals from the product type (e.g., "API project" → "No web UI," "MVP" → "No analytics dashboard," "internal tool" → "No public registration")

7. The generated Boundaries section SHALL include all three tiers:
   - ALWAYS: at least 2 rules (always including "run tests before reporting done")
   - ASK FIRST: at least 2 rules (always including "before adding new dependencies")
   - NEVER: at least 2 rules (always including "use parameterized queries for all database operations")

8. The generated Tech Stack section SHALL include exact version numbers. WHEN detected via project scan, use the actual installed versions. WHEN manually specified, use the user's stated versions. WHEN neither, use the latest stable LTS version at time of generation and note "(verify version)" beside it.

9. The generated PRD SHALL be broken into 3–6 phases. Phase 1 SHALL always be the minimal viable core. Each subsequent phase SHALL build on the previous. The skill SHALL not generate more than 50 requirements per phase.

10. WHEN the product description implies a broad scope (more than 6 phases worth of work), the skill SHALL inform the user: "This product scope suggests [N] phases. I recommend splitting into multiple PRDs by workstream. Shall I generate a PRD for just [core workstream] first?"

### Acceptance Criteria

- [ ] Generated PRD contains all required sections in correct order
- [ ] Zero requirements use "should," "could," or "best practices"
- [ ] Every endpoint has success + error response shapes
- [ ] Non-Goals section has 5+ exclusions
- [ ] Boundaries section has ALWAYS / ASK FIRST / NEVER tiers
- [ ] Tech stack includes version numbers
- [ ] Work is phased into 3–6 phases with verification per phase

### Verification

```bash
# After generating a test PRD, verify structure:
grep -c "^## " PRD-test.md
# Expect: 8+ (Objective, Success, Tech, Commands, Structure, Non-Goals, Boundaries, Phase 1+)

grep -ci "should\|could\|best practices\|appropriate\|proper" PRD-test.md
# Expect: 0 (or only in Non-Goals negations like "should NOT")

grep -c "SHALL" PRD-test.md
# Expect: 15+ (at least one per requirement)

grep -c "ALWAYS\|ASK FIRST\|NEVER" PRD-test.md
# Expect: 6+ (at least 2 per tier)
```

---

## Phase 3: Self-Audit Engine

### Requirements

1. After generating the PRD and before presenting it to the user, the skill SHALL perform a self-audit pass against these five quality gates:

   **Gate 1 — Testable:** For each requirement, ask: "Can I write a pass/fail check for this?" Flag any requirement where the answer is no.

   **Gate 2 — Specific:** For each requirement, check: Does it contain at least one concrete value (a number, a field name, a status code, a file path, a format)? Flag any requirement that is purely qualitative.

   **Gate 3 — Bounded:** Check: Does the Non-Goals section exist and contain 5+ items? Does each phase have ≤ 50 requirements? Flag violations.

   **Gate 4 — Singular:** For each requirement, check: Does it describe exactly one behavior? Flag any requirement containing "and" that joins two distinct behaviors (e.g., "validate the input AND store it in the database").

   **Gate 5 — Imperative:** Check: Does every requirement use SHALL/WHEN/IF? Flag any using "should," "could," "might," "ideally," "appropriate," "proper," "clean," "modern," "best practices," or "like [other product]."

2. WHEN the self-audit finds 0 flags, the skill SHALL present the PRD with: "PRD generated and passed all quality gates. [show summary table]"

3. WHEN the self-audit finds 1+ flags, the skill SHALL present the PRD with the flags listed, grouped by gate, and offer: "I found [N] requirements that could be stronger. Want me to fix them, or would you prefer to review the PRD as-is first?"

4. WHEN the user asks to fix flagged requirements, the skill SHALL rewrite ONLY the flagged requirements (not regenerate the entire PRD) and re-run the audit on the rewrites.

5. IF `PRD-SPEC-RULES.md` exists in the project root or in `references/`, the skill SHALL additionally check the generated PRD against every numbered rule in that file during the audit. Flag any violations with the rule number.

6. The self-audit results SHALL be presented as a brief summary table:
   ```
   Quality Gate     | Status | Flags
   -----------------|--------|------
   Testable         | PASS   | 0
   Specific         | WARN   | 2 (Reqs 3.4, 3.7)
   Bounded          | PASS   | 0
   Singular         | PASS   | 0
   Imperative       | PASS   | 0
   SPEC-RULES (R1–R8)| PASS | 0
   ```

### Acceptance Criteria

- [ ] Self-audit runs automatically after generation, before user sees the PRD
- [ ] A deliberately vague requirement ("handle errors properly") is flagged by Gate 1 (Testable) and Gate 5 (Imperative)
- [ ] A compound requirement ("validate and store") is flagged by Gate 4 (Singular)
- [ ] A PRD missing Non-Goals is flagged by Gate 3 (Bounded)
- [ ] Clean PRDs show all-PASS summary table
- [ ] Fix mode rewrites only flagged requirements, not the entire document
- [ ] If PRD-SPEC-RULES.md is present, its rules are checked and violations cited by rule number

### Verification

```bash
# Generate a PRD from a deliberately vague description:
# /prd Build me something like Spotify
# Expect: self-audit flags multiple requirements across gates 1, 2, and 5

# Generate a PRD from a precise description with tech stack:
# /prd REST API for task management. TypeScript, Fastify, PostgreSQL. CRUD on tasks with due dates and priority levels. No auth, no UI, no notifications.
# Expect: self-audit shows all-PASS or near-PASS
```

---

## Phase 4: Cross-Agent Output

### Requirements

1. After the PRD passes self-audit (or user accepts as-is), the skill SHALL generate two additional files alongside the `PRD-<workstream>.md`:

2. **AGENTS.md** (Codex-compatible) SHALL be generated by transforming the PRD into Codex's expected format:
   - Project overview extracted from Objective
   - Coding conventions extracted from Tech Stack + Boundaries
   - Key commands extracted from Commands section
   - Testing requirements extracted from Verification sections
   - The format SHALL follow the AGENTS.md open standard (sections: Overview, Conventions, Commands, Testing, Constraints)

3. **`.cursor/rules/prd-<workstream>.mdc`** (Cursor-compatible) SHALL be generated with:
   - Frontmatter containing `description`, `globs` (set to `**/*`), and `alwaysApply: true`
   - Body containing: tech stack constraints, boundary rules (ALWAYS/NEVER only), and phase-awareness instructions
   - Total length SHALL NOT exceed 100 lines (Cursor rules should be concise)

4. WHEN generating cross-agent files, the skill SHALL inform the user:
   "Generated 3 files:
   - `PRD-<workstream>.md` — Full PRD (primary spec)
   - `AGENTS.md` — Codex/OpenAI compatible context
   - `.cursor/rules/prd-<workstream>.mdc` — Cursor compatible rules
   
   The PRD is the source of truth. AGENTS.md and Cursor rules are derived — regenerate them by running `/prd` again if you update the PRD manually."

5. WHEN an `AGENTS.md` already exists, the skill SHALL ask: "An existing AGENTS.md was found. Should I (1) append this workstream's section, (2) replace it entirely, or (3) skip AGENTS.md generation?"

6. The generated AGENTS.md SHALL include a header comment: `<!-- Generated by /prd from PRD-<workstream>.md. Regenerate with /prd, do not edit manually. -->`

### Acceptance Criteria

- [ ] Three files are created after PRD generation
- [ ] AGENTS.md follows the open standard section structure
- [ ] `.cursor/rules/prd-<workstream>.mdc` has valid Cursor frontmatter
- [ ] Cursor rules file is ≤ 100 lines
- [ ] Existing AGENTS.md triggers the append/replace/skip prompt
- [ ] All derived files contain generation source comments

### Verification

```bash
# After generating from a test product:
ls -la PRD-*.md AGENTS.md .cursor/rules/prd-*.mdc
# Expect: all three files exist

wc -l .cursor/rules/prd-*.mdc
# Expect: ≤ 100

head -1 AGENTS.md
# Expect: <!-- Generated by /prd from PRD-...
```

---

## Phase 5: Refinement Loop

### Requirements

1. After presenting the generated PRD and audit results, the skill SHALL offer a refinement menu:
   "Your PRD is ready. You can:
   (1) Accept as-is — save all files
   (2) Refine — tell me what to change and I'll update
   (3) Add a phase — extend the PRD with another phase
   (4) Regenerate — start the intake over from scratch
   (5) Audit only — re-run quality gates on the current draft"

2. WHEN the user selects "Refine," the skill SHALL accept natural language instructions and apply targeted edits to the PRD. It SHALL NOT regenerate the entire document — only modify the sections the user references.

3. WHEN the user selects "Add a phase," the skill SHALL determine the next phase number, ask what the phase covers, and append it to the PRD following the same format (Requirements + Acceptance Criteria + Verification). It SHALL re-run the self-audit on the new phase only.

4. WHEN the user selects "Audit only," the skill SHALL re-run all five quality gates plus SPEC-RULES check on the current draft and present updated results.

5. After any refinement action, the skill SHALL re-present the refinement menu. The loop continues until the user selects "Accept."

6. WHEN the user selects "Accept," the skill SHALL:
   - Write `PRD-<workstream>.md` to the project root
   - Write or update `AGENTS.md`
   - Write `.cursor/rules/prd-<workstream>.mdc`
   - Confirm with: "PRD saved. Start building with: 'Implement Phase 1 of PRD-<workstream>.md'"

### Acceptance Criteria

- [ ] Refinement menu appears after generation
- [ ] "Refine" applies targeted edits without regenerating the full PRD
- [ ] "Add a phase" appends correctly numbered phase with proper format
- [ ] "Audit only" re-runs all gates and shows updated table
- [ ] Loop continues until "Accept"
- [ ] "Accept" writes all three files and shows the implementation prompt

### Verification

```bash
# Generate a PRD, then select "Add a phase" and add Phase 4
# Verify the phase was appended:
grep "## Phase 4" PRD-test.md
# Expect: match found

# Select "Audit only" and verify table displays:
# Expect: quality gate table with PASS/WARN/FAIL per gate
```

---

## Phase 6: Reference Files, README, and Packaging

### Requirements

1. `references/PRD-SPEC-RULES.md` SHALL be a copy of the `PRD-SPEC-RULES.md` source file provided in the project, with no modifications.

2. `references/PRD-TEMPLATE.md` SHALL contain the skeleton template from Section 2 of `AI_PRD_FRAMEWORK.md` (the code block between the "PRD TEMPLATE SKELETON" markers), extracted as a standalone file.

3. `references/EXAMPLES.md` SHALL contain the three annotated example PRDs from Section 3 of `AI_PRD_FRAMEWORK.md` (Perfect, Mid, Terrible), extracted as a standalone file with the section header and comparison table.

4. `README.md` SHALL contain:
   - Project name and one-line description
   - A "Quick Start" section with installation commands for both global (`~/.claude/skills/`) and project-scoped (`.claude/skills/`) installation
   - A "Usage" section showing `/prd` invocation with and without arguments
   - A "What It Generates" section listing the three output files with descriptions
   - A "How It Works" section with a brief flow diagram (ASCII or Mermaid):
     `Invoke → Detect existing PRDs → Offer project scan → Intake questions → Generate → Self-audit → Refine loop → Save files`
   - A "Extending" section explaining how to modify the template, add quality gates, or customize the intake questions
   - A "Cross-Agent Compatibility" section explaining AGENTS.md and Cursor rules output
   - A "Philosophy" section with a 3–4 sentence summary of why AI-optimized PRDs differ from traditional PRDs, linking to `AI_PRD_FRAMEWORK.md` for the full research
   - A "License" section: MIT

5. `LICENSE` SHALL contain the MIT license text with year 2025 and copyright holder set to "prd-generator contributors"

6. `CHANGELOG.md` SHALL contain a single entry:
   ```
   ## [1.0.0] - <today's date>
   ### Added
   - Initial release of /prd slash command
   - Hybrid intake flow with adaptive expertise detection
   - Self-audit engine with 5 quality gates
   - Cross-agent output (PRD.md, AGENTS.md, .cursor/rules/)
   - Multi-PRD support with workstream naming
   - Reference docs: template, rules, annotated examples
   ```

### Acceptance Criteria

- [ ] All files in the project structure exist and are non-empty
- [ ] README.md installation instructions are copy-pasteable and correct
- [ ] README.md contains all 8 required sections
- [ ] references/ contains 3 files matching source content
- [ ] LICENSE is valid MIT
- [ ] CHANGELOG.md has 1.0.0 entry

### Verification

```bash
# Verify complete file tree:
find prd-generator -type f | sort
# Expect:
# prd-generator/CHANGELOG.md
# prd-generator/LICENSE
# prd-generator/README.md
# prd-generator/SKILL.md
# prd-generator/references/EXAMPLES.md
# prd-generator/references/PRD-SPEC-RULES.md
# prd-generator/references/PRD-TEMPLATE.md

# Verify README has required sections:
grep -c "^## " prd-generator/README.md
# Expect: 8+

# Verify no empty files:
find prd-generator -type f -empty
# Expect: no output
```

---

## PART 3: FILE MANIFEST FOR THE 1-SHOT SESSION

When starting the Claude Code session, ensure these files are in the working directory:

| File | Purpose | Source |
|------|---------|--------|
| `BUILD-PRD-TOOL.md` | This file — the 1-shot prompt + PRD | You're reading it |
| `AI_PRD_FRAMEWORK.md` | Research report + framework + annotated examples | Generated in prior chat |
| `PRD-SPEC-RULES.md` | Operational rules for every session | Generated in prior chat |

Claude Code will read all three, then build the `prd-generator/` skill directory
from scratch following the phased PRD above.

---

## PART 4: POST-BUILD TESTING CHECKLIST

After Claude Code completes all 6 phases, manually verify:

```
[ ] /prd appears in Claude Code autocomplete
[ ] /prd with no args starts intake at Q1
[ ] /prd "Build a REST API for invoices" starts intake at Q2
[ ] Project scan detects package.json when present
[ ] Existing PRD detection finds PRD-*.md files
[ ] Generated PRD has all required sections
[ ] Generated PRD uses SHALL/WHEN/IF (no "should"/"could")
[ ] Self-audit table appears before PRD is presented
[ ] Deliberately vague input triggers audit flags
[ ] "Refine" edits without full regeneration
[ ] "Add a phase" appends correctly
[ ] "Accept" writes all 3 output files
[ ] AGENTS.md follows open standard format
[ ] .cursor/rules/ file is ≤ 100 lines with valid frontmatter
[ ] Running /prd again detects the PRD just created
```
