# PRD Template Skeleton

> Use this skeleton as a starting point. Delete sections that don't apply.

```markdown
# [Product Name] — PRD v[X.Y]

## Objective
[One sentence: what this product does and for whom.]

## Success Criteria
- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]

## Tech Stack
- Language: [e.g., TypeScript 5.4]
- Framework: [e.g., Next.js 14.2 with App Router]
- Database: [e.g., PostgreSQL 16 via Prisma 5.x]
- Hosting: [e.g., Vercel]

## Commands
- Install: `npm install`
- Dev: `npm run dev`
- Test: `npm test`
- Lint: `npm run lint`
- Build: `npm run build`

## Project Structure
src/
├── app/          # Next.js routes
├── lib/          # Shared utilities
├── components/   # React components
├── db/            # Prisma schema + migrations
└── types/         # TypeScript interfaces

## Non-Goals / Out of Scope
- [Capability 1 that SHALL NOT be built]
- [Capability 2 that SHALL NOT be built]

## Boundaries
- ALWAYS: [Things the agent must always do]
- ASK FIRST: [Things requiring human approval]
- NEVER: [Things the agent must never do]

## Phase 1: [Name]
### Requirements
1. [Requirement using EARS format]
2. ...

### Acceptance Criteria
- [ ] [Testable criterion]

### Verification
`[command to run that proves phase is complete]`

## Phase 2: [Name]
...
```
