# PRD Examples — Annotated

> Extracted from the AI-Optimized PRD Framework. Three example PRDs for the same product (URL shortener),
> graded Perfect / Mid / Terrible, plus a side-by-side comparison table.

---

## SECTION 3: THREE EXAMPLE PRDs — ANNOTATED

The following three PRDs specify the **same product**: a URL shortener API service called "Shrink." Each PRD appears roughly similar in length and structure at first glance. They are annotated with `<!-- GRADE: ... -->` comments explaining what makes each requirement perfect, mid-grade, or terrible.

Read all three sequentially. The differences are in precision, not in word count.

---

### ══════════════════════════════════════════
### EXAMPLE A: THE PERFECT PRD
### ══════════════════════════════════════════

```markdown
# Shrink — URL Shortener API — PRD v1.0

## Objective
Shrink is a REST API that accepts a long URL and returns a shortened
URL. When a user visits the short URL, they are redirected to the
original. The API tracks click counts per link.

<!-- GRADE: PERFECT — One sentence. States the WHAT (shorten, redirect,
track) and the WHO (any user via REST). No ambiguity about the product
boundary. An AI agent reading this knows exactly what to build and — 
critically — what NOT to build. -->

## Success Criteria
- [ ] POST /shorten accepts a valid URL and returns a 7-character slug
      in < 200ms at p95
- [ ] GET /:slug returns HTTP 301 to the original URL in < 50ms at p95
- [ ] GET /stats/:slug returns JSON with total click count and
      created_at timestamp
- [ ] All endpoints return structured JSON errors with appropriate
      HTTP status codes (400, 404, 429, 500)
- [ ] The system handles 1,000 requests/second on a single 2-vCPU
      instance

<!-- GRADE: PERFECT — Every criterion is measurable, testable, and
bounded. Latency targets are specific (p95, not "fast"). Throughput has
a concrete number. Error responses are enumerated with status codes.
An AI agent can write tests that verify every single line here. -->

## Tech Stack
- Runtime: Node.js 22 LTS
- Framework: Fastify 5.x (NOT Express — Fastify for performance)
- Database: PostgreSQL 16 via pg-native (NOT an ORM)
- Cache: Redis 7.x for slug lookups
- Language: TypeScript 5.6, strict mode, no `any` types
- Testing: Vitest 2.x
- Linting: Biome

<!-- GRADE: PERFECT — Exact versions. Negative constraints ("NOT
Express," "NOT an ORM," "no any types") eliminate the most common
AI default choices that would violate the design intent. An agent
cannot misinterpret "Fastify 5.x (NOT Express)" — the parenthetical
costs 3 words and prevents a mass rewrite. -->

## Commands
```bash
npm install                # Install dependencies
npm run dev                # Start dev server on port 3000
npm test                   # Run Vitest suite — expect 0 failures
npm run lint               # Run Biome — expect 0 errors
npm run build              # Compile TypeScript to dist/
npm run db:migrate         # Run PostgreSQL migrations
npm run db:seed            # Seed with 100 test URLs
```

<!-- GRADE: PERFECT — Every command is present, executable, and has an
expected outcome ("expect 0 failures"). The agent can run each command
and verify success programmatically. db:seed gives the agent test data
to validate against. Compare to a mid-PRD that says "standard npm
scripts" — the agent would guess what those are. -->

## Project Structure
```
src/
├── server.ts              # Fastify instance + plugin registration
├── routes/
│   ├── shorten.ts         # POST /shorten
│   ├── redirect.ts        # GET /:slug
│   └── stats.ts           # GET /stats/:slug
├── services/
│   ├── slug.ts            # Slug generation (nanoid, 7 chars, a-z0-9)
│   └── url-validator.ts   # URL validation (protocol, reachability)
├── db/
│   ├── pool.ts            # pg-native connection pool (max 20 conns)
│   └── migrations/
│       └── 001_create_urls.sql
├── cache/
│   └── redis.ts           # Redis client + slug lookup cache
├── middleware/
│   ├── rate-limiter.ts    # 100 req/min per IP via Redis
│   └── error-handler.ts   # Structured JSON error responses
├── types/
│   └── index.ts           # Shared TypeScript interfaces
└── tests/
    ├── shorten.test.ts
    ├── redirect.test.ts
    ├── stats.test.ts
    └── rate-limiter.test.ts
```

<!-- GRADE: PERFECT — Every file is named. Every file has a one-line
purpose comment. The agent doesn't decide where to put things — the
PRD decides. Implementation details (nanoid, 7 chars, a-z0-9) appear
inline as parentheticals so the agent encounters them at exactly the
right moment. The test files mirror the routes 1:1, making coverage
gaps impossible to miss. -->

## Non-Goals / Out of Scope
- No user accounts, authentication, or API keys in v1
- No custom slug selection (all slugs are randomly generated)
- No link expiration or TTL
- No analytics beyond total click count (no referrer, geo, UA)
- No web UI or dashboard — API only
- No bulk URL creation endpoint
- No webhook or callback support

<!-- GRADE: PERFECT — Seven explicit exclusions. Each one prevents a
specific feature an AI would likely infer ("most URL shorteners have
auth" → No. "most have custom slugs" → No.) The specificity of
"no analytics beyond total click count (no referrer, geo, UA)" shows
the author thought about WHAT the agent might add and preempted it.
Without this section, a quality-oriented AI agent will add at minimum
3–4 of these features. -->

## Boundaries
- ALWAYS: Run `npm test` and `npm run lint` before reporting task done
- ALWAYS: Use parameterized SQL queries — never string concatenation
- ASK FIRST: Before adding any new npm dependency
- ASK FIRST: Before modifying the database schema after initial migration
- NEVER: Use console.log for production logging (use Fastify's pino)
- NEVER: Store or log the full original URL in plaintext error messages

<!-- GRADE: PERFECT — Three-tier boundaries (ALWAYS / ASK FIRST / NEVER)
give the agent clear autonomy zones. The NEVER rules prevent specific
security antipatterns (SQL injection, URL leakage). "ASK FIRST before
adding dependencies" prevents package hallucination — the #1 AI-specific
security vulnerability. -->

## Phase 1: Core Shortening
### Requirements
1. POST /shorten SHALL accept JSON body `{ "url": "<string>" }`
2. The url field SHALL be validated: must start with http:// or https://,
   must be ≤ 2,048 characters, must contain a valid TLD
3. WHEN url is valid, the system SHALL generate a 7-character slug using
   nanoid with alphabet `a-z0-9` (lowercase only)
4. The slug + original URL SHALL be stored in PostgreSQL table `urls`
   with columns: id (bigserial PK), slug (varchar(7) UNIQUE INDEX),
   original_url (text NOT NULL), created_at (timestamptz DEFAULT now()),
   click_count (integer DEFAULT 0)
5. The response SHALL be HTTP 201 with JSON:
   `{ "short_url": "http://localhost:3000/<slug>", "slug": "<slug>" }`
6. WHEN url is missing or invalid, the response SHALL be HTTP 400 with:
   `{ "error": "invalid_url", "message": "<human-readable reason>" }`
7. WHEN a duplicate original_url is submitted, the system SHALL return
   the existing slug (idempotent), not create a new entry

<!-- GRADE: PERFECT — EARS keywords (SHALL, WHEN). Exact field names,
types, and constraints. The response JSON is shown literally. Req #7
handles the duplicate edge case explicitly — without it, the agent would
either create duplicates (wasting storage) or crash on unique constraint
violations. The validation rules in #2 are specific (protocol, length,
TLD) not vague ("valid URL"). -->

### Acceptance Criteria
- [ ] `curl -X POST localhost:3000/shorten -H 'Content-Type: application/json' -d '{"url":"https://example.com"}' | jq .slug` returns a 7-char string
- [ ] Submitting the same URL twice returns the same slug
- [ ] Submitting `{"url":"not-a-url"}` returns 400
- [ ] Submitting `{"url":"ftp://example.com"}` returns 400 (ftp not allowed)
- [ ] Submitting a URL longer than 2,048 chars returns 400

<!-- GRADE: PERFECT — Literally copy-pasteable curl commands. The agent
can run these as integration tests. Edge cases (duplicate, bad protocol,
too long) are explicit. No ambiguity about pass/fail criteria. -->

### Verification
```bash
npm test -- --grep "shorten"  # Expect: all shorten tests pass
curl -s -o /dev/null -w "%{http_code}" -X POST localhost:3000/shorten \
  -H 'Content-Type: application/json' \
  -d '{"url":"https://example.com"}'
# Expect: 201
```

<!-- GRADE: PERFECT — Two verification methods: unit test and manual
smoke test. Expected outputs are commented inline. -->

## Phase 2: Redirect
### Requirements
1. GET /:slug SHALL look up the slug in Redis cache first, then
   PostgreSQL if cache misses
2. WHEN slug is found, the system SHALL return HTTP 301 with
   Location header set to the original URL
3. WHEN slug is found, the system SHALL increment click_count in
   PostgreSQL (async, non-blocking — do not delay the redirect)
4. WHEN slug is found and served from PostgreSQL (cache miss), the
   system SHALL populate the Redis cache with key `slug:<slug>`
   and value `<original_url>`, TTL 1 hour
5. WHEN slug is NOT found, the system SHALL return HTTP 404 with:
   `{ "error": "not_found", "message": "Slug not found" }`
6. The slug parameter SHALL be validated: exactly 7 characters,
   lowercase alphanumeric only. Malformed slugs SHALL return 400,
   NOT 404

<!-- GRADE: PERFECT — Req #3 specifies "async, non-blocking" which
prevents the most common performance mistake (synchronous DB write
blocking the redirect). Req #4 specifies exact cache key format and
TTL. Req #6 distinguishes 400 (bad input) from 404 (valid format but
doesn't exist) — a subtle correctness detail AI would certainly miss
without guidance. -->

### Acceptance Criteria
- [ ] Visiting a valid short URL redirects (301) to the original
- [ ] The redirect response has no body (just the Location header)
- [ ] Visiting an unknown slug returns 404 JSON
- [ ] Visiting `/ABCDEFG` (uppercase) returns 400, not 404
- [ ] After redirect, click_count in DB has incremented by 1
- [ ] Second visit to same slug serves from Redis (verify via Redis CLI)

### Verification
```bash
npm test -- --grep "redirect"
SLUG=$(curl -s -X POST localhost:3000/shorten \
  -H 'Content-Type: application/json' \
  -d '{"url":"https://example.com"}' | jq -r .slug)
curl -s -o /dev/null -w "%{http_code}" "localhost:3000/$SLUG"
# Expect: 301
redis-cli GET "slug:$SLUG"
# Expect: https://example.com
```

## Phase 3: Stats + Rate Limiting
### Requirements
1. GET /stats/:slug SHALL return JSON:
   `{ "slug": "<slug>", "original_url": "<url>", "click_count": <int>, "created_at": "<ISO8601>" }`
2. WHEN slug is not found, return HTTP 404 (same format as redirect 404)
3. All endpoints SHALL be rate-limited to 100 requests per minute per
   client IP, tracked in Redis with key `ratelimit:<ip>`, TTL 60 seconds
4. WHEN rate limit is exceeded, return HTTP 429 with:
   `{ "error": "rate_limited", "message": "Too many requests", "retry_after_seconds": <int> }`
5. The 429 response SHALL include a `Retry-After` HTTP header (seconds)

### Acceptance Criteria
- [ ] Stats endpoint returns correct click_count after N redirects
- [ ] Stats endpoint returns ISO8601 created_at
- [ ] Sending 101 requests in 60 seconds from same IP triggers 429
- [ ] 429 response includes both JSON retry_after_seconds and Retry-After header
- [ ] Rate limiter does not count 429 responses toward the limit

### Verification
```bash
npm test -- --grep "stats|rate"
# Load test: 101 rapid requests, expect last one to be 429
for i in $(seq 1 101); do
  curl -s -o /dev/null -w "%{http_code}\n" localhost:3000/stats/aaaaaaa
done | tail -1
# Expect: 429
```
```

---

### ══════════════════════════════════════════
### EXAMPLE B: THE MID PRD
### ══════════════════════════════════════════

```markdown
# Shrink — URL Shortener

## Overview
Build a URL shortener service. Users can submit a long URL and get
a short one back. The short URL redirects to the original. Track
how many times each link is clicked.

<!-- GRADE: MID — Communicates the idea, but lacks precision. "Users"
is undefined (API? Web? Both?). "Track how many times" gives no detail
on how that data is exposed. An AI agent reading this will make its own
decisions about the interface (probably adds a web UI). Compare to
Perfect PRD: "REST API that accepts a long URL and returns a shortened
URL." That single word "API" eliminates an entire class of ambiguity. -->

## Success Criteria
- URLs can be shortened and redirected
- Click tracking works
- The system is fast and reliable
- Proper error handling

<!-- GRADE: MID — None of these are measurable. "Fast" means what?
50ms? 5 seconds? "Proper error handling" is completely subjective.
An AI agent will implement whatever error handling it deems "proper"
which likely means try/catch with console.error. Compare to Perfect:
"< 200ms at p95" and "HTTP 400, 404, 429, 500 with structured JSON."
These criteria cannot be tested — they can only be argued about. -->

## Tech Stack
- Node.js with TypeScript
- Some kind of database (PostgreSQL preferred)
- Redis for caching would be nice
- Use a modern testing framework

<!-- GRADE: MID — No versions. "Some kind of database (PostgreSQL
preferred)" invites the agent to choose — and it might choose SQLite
for simplicity, or Prisma as an ORM, both of which may conflict with
performance requirements. "Would be nice" is a suggestion, not a
requirement — the agent may skip Redis entirely. "Modern testing
framework" could mean Vitest, Jest, Mocha, or node:test. The agent
will pick whichever it has seen most in training data (probably Jest),
which may or may not be what you wanted. -->

## Project Structure
Standard Node.js project layout with src directory, tests, and
configuration files.

<!-- GRADE: MID — This tells the agent nothing. "Standard" according
to whom? The agent will create its own structure, which might be
deeply nested (src/modules/url/controllers/shortenController.ts) or
flat (src/index.ts with everything). Compare to Perfect PRD which
names every file and its purpose. You will spend more time reorganizing
the code than you saved by not specifying the structure. -->

## API Endpoints

### POST /shorten
Takes a URL in the request body and returns a shortened URL. Should
validate that the input is a real URL.

<!-- GRADE: MID — "Takes a URL in the request body" — what format?
JSON? Form data? Query param? "Returns a shortened URL" — what HTTP
status? What response shape? "Should validate" — the word "should"
is ambiguous in specs (it means "recommended but not required" in
RFC 2119). "Real URL" — does ftp:// count? mailto:? data:? What
about localhost URLs? The agent will make reasonable-seeming choices
for all of these that may conflict with your expectations. -->

### GET /:slug
Redirects to the original URL.

<!-- GRADE: MID — Which redirect code? 301 (permanent) or 302
(temporary)? This matters for SEO and caching. What happens if the
slug doesn't exist — 404 page? JSON error? HTML error? What if the
slug format is wrong? The agent will implement a 302 redirect (the
default in most frameworks) and probably return an HTML 404 page,
neither of which match the API-first design intent. -->

### GET /stats/:slug
Returns click statistics for a shortened URL.

<!-- GRADE: MID — What fields are in the response? Total clicks only?
Clicks over time? Referrer data? Geographic data? The agent will
decide. It might build something simple (click_count) or something
elaborate (time-series analytics with charts). Both are reasonable
interpretations of "click statistics." -->

## Additional Requirements
- Add rate limiting to prevent abuse
- Handle errors gracefully
- Use caching where appropriate
- Write tests for the main functionality

<!-- GRADE: MID — Every one of these is a vague directive that the
agent must interpret. "Rate limiting" — what limit? Per IP? Per API
key? What response on limit? "Gracefully" is meaningless to an AI.
"Where appropriate" delegates the caching architecture entirely to
the agent. "Main functionality" — does that mean 60% coverage? 90%?
Just happy paths? These read fine in a human PRD because humans ask
follow-up questions. AI agents don't — they just decide. -->

## Nice to Have
- Custom slug support
- Link expiration
- Bulk URL creation

<!-- GRADE: MID — This is a trap. "Nice to have" is ambiguous to AI
agents. Some will build all three. Some will build none. Some will
build partial implementations. You now have no idea what you'll get
and no clear acceptance criteria for any of it. Compare to Perfect
PRD: these three features are in the NON-GOALS section, explicitly
excluded. That's the correct place for features you've considered
but decided against for v1. -->
```

---

### ══════════════════════════════════════════
### EXAMPLE C: THE TERRIBLE PRD
### ══════════════════════════════════════════

```markdown
# URL Shortener

Build me a URL shortener like bit.ly. It should be modern, clean,
and production-ready. Use best practices.

<!-- GRADE: TERRIBLE — "Like bit.ly" is the most dangerous phrase in
AI-assisted development. Bit.ly has: user accounts, custom domains,
branded links, QR codes, team workspaces, enterprise SSO, analytics
dashboards, REST + GraphQL APIs, webhook integrations, link-in-bio
pages, and more. Which parts? The agent doesn't know, so it will
either build a tiny fraction (disappointing) or attempt everything
(disastrous). "Modern, clean, and production-ready" are aesthetic
judgments, not requirements. "Best practices" according to which
framework, which year, which team? This prompt will produce working
code that matches nothing the author imagined. -->

## Tech Requirements
- Use a good tech stack
- Make sure it scales
- Should be secure
- Deploy somewhere easy

<!-- GRADE: TERRIBLE — Four requirements, zero information. "Good tech
stack" is fully delegated to the AI, which will choose based on training
data frequency (likely Express + MongoDB + React — the most common
tutorial stack, not the best production stack). "Scales" to what? 10
users? 10 million? "Secure" against what threats? "Somewhere easy"
probably becomes a Vercel hello-world deploy that doesn't include the
database. The agent will spend tokens reasoning about what you might
have meant instead of building what you actually need. -->

## Features
- Shorten URLs
- Redirect short URLs to long ones
- Track clicks
- User dashboard
- Nice UI
- API access
- Analytics

<!-- GRADE: TERRIBLE — This list mixes core functionality (shorten,
redirect) with major features (user dashboard, analytics) with vague
desires (nice UI) at the same priority level. There is no phasing, no
sequencing, no acceptance criteria. The agent cannot tell that "shorten
URLs" is essential and "user dashboard" is a stretch goal. It will
attempt to build all seven simultaneously, creating a shallow
implementation of everything with a deep implementation of nothing.
The word "analytics" alone could mean anything from a click counter
to a full Mixpanel clone. -->

## Database
Store everything in a database.

<!-- GRADE: TERRIBLE — Which database? What schema? What indexes?
What are the entities and their relationships? "Everything" includes
what — URLs? Users? Sessions? Click events with timestamps? Analytics
aggregations? The agent will design the schema itself, and its choices
will be internally consistent but almost certainly misaligned with
your actual needs. It might create 2 tables or 12. Both are valid
interpretations of "everything." -->

## Error Handling
Handle errors properly.

<!-- GRADE: TERRIBLE — This is not a requirement. It is a wish.
Compare to Perfect PRD's Req #6: "WHEN url is missing or invalid,
the response SHALL be HTTP 400 with: { error: 'invalid_url',
message: '<human-readable reason>' }." That is a requirement. This
is the equivalent of telling a contractor "make the house good." -->

## Testing
Write tests.

<!-- GRADE: TERRIBLE — What tests? Unit? Integration? E2E? What
coverage target? What framework? What scenarios? This requirement
adds zero constraining information. The agent might write 3 tests
or 300. It might test only the happy path or every edge case. It
might use Jest, Vitest, Mocha, tap, or node:test. Without knowing
what "tests" means to the author, the AI will do whatever requires
the least reasoning — typically a handful of happy-path unit tests
with no edge case coverage and no integration tests. -->

## Design
Make it look professional and modern. Take inspiration from
popular URL shorteners.

<!-- GRADE: TERRIBLE — The Perfect PRD explicitly says "No web UI or
dashboard — API only." This PRD requests a full frontend with
aesthetic requirements ("professional and modern") that are
unverifiable by an AI agent. "Take inspiration from" may cause the
agent to attempt to replicate copyrighted designs. The AI will spend
the majority of its tokens on UI components that may not even be
needed. A common failure mode: the agent builds a beautiful React
dashboard with Tailwind and Framer Motion, and the API behind it
returns hardcoded test data because it ran out of context budget
before implementing the backend. -->

## Deployment
Set it up so I can deploy it easily.

<!-- GRADE: TERRIBLE — "Easily" is relative. To whom? A DevOps
engineer or a non-technical founder? The agent might generate a
Dockerfile, a docker-compose.yml, a Terraform config, a Kubernetes
manifest, a Railway.toml, a Vercel config, or just a README that
says "run npm start." All of these are "easy" deployment setups
by some definition. Without specifying the target platform and the
deployer's skill level, this requirement is essentially random. -->

## Other Notes
- Make it fast
- Should work well
- Keep the code clean
- Add comments where needed

<!-- GRADE: TERRIBLE — The final section perfectly captures everything
wrong with this PRD. Four statements that apply to literally every
software project ever built. They constrain nothing, guide nothing,
and verify nothing. An AI agent processing these will either ignore
them entirely (they add no information) or over-index on "add comments"
and produce code where every line has a redundant comment like
"// increment the counter" above `counter++`. The signal-to-noise
ratio of this PRD is approximately zero. -->
```

---

## SECTION 4: SIDE-BY-SIDE COMPARISON — WHY THE DIFFERENCES MATTER

| Dimension | Perfect PRD | Mid PRD | Terrible PRD |
|-----------|------------|---------|--------------|
| **Objective** | 1 sentence, names interface type (REST API), lists 3 exact capabilities | 3 sentences, omits interface type, vague on exposure of data | 1 sentence referencing another product ("like bit.ly") |
| **Success Criteria** | 5 measurable targets with latency, throughput, status codes | 4 unmeasurable desires ("fast," "proper") | None |
| **Tech Stack** | Exact versions, negative constraints ("NOT Express") | General preferences, no versions, hedging language ("would be nice") | Fully delegated ("good tech stack") |
| **Structure** | Every file named with purpose comments | "Standard layout" (undefined) | Not mentioned |
| **Non-Goals** | 7 explicit exclusions preventing feature creep | 3 "Nice to Haves" (ambiguous priority) | Not mentioned (everything is in scope) |
| **Requirements** | EARS keywords, exact field names/types, edge cases | Natural language, missing edge cases, ambiguous verbs | Feature names only, no details |
| **Verification** | Copy-pasteable commands with expected output | "Write tests" (unspecified) | "Write tests" (2 words) |
| **Boundaries** | ALWAYS / ASK FIRST / NEVER tiers | Not mentioned | Not mentioned |
| **Token Efficiency** | ~1,800 words of high-signal content | ~600 words, ~40% is filler or ambiguity | ~250 words, ~90% is noise |
| **Predicted Outcome** | Exactly what was specified, verifiable | 60–70% of intent, requires 2–3 correction rounds | Unrecognizable; rebuild from scratch |

---

