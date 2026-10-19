# CareerOps Phase 0 — Foundations Summary

**Date:** 2026-09-07  
**Status:** In Progress — Infrastructure & Setup

---

## Project Overview

CareerOps is a flagship portfolio project: an intelligent career discovery, application, outreach, and analytics platform built hands-on by Rob. Claude acts strictly as a senior-engineer mentor — architecture guidance, code review, teaching — not as the primary developer.

**Operating mode source of truth:** `D:\Work\AI\Prompts\GitHub\career-tool`  
**Project knowledge/decisions:** `D:\Work\Obsidian-Brain\BRAIN\01 - PROJECTS\CareerOps\`

---

## Phase 0 Objective

Get the project to a state where building a feature means "add code to a real system," not "start from nothing every session."

**Exit criteria:**
- Local dev environment works end-to-end (API + web + DB connectivity proven)
- Repository scaffold matches GitHub portfolio standards
- CI skeleton in place (lint + typecheck + test on push)
- All major stack/architecture decisions documented as ADRs
- First real feature (Phase 1: Discovery) has an obvious place to land

---

## Phase 0 Decisions (Locked In)

### 1. Single-User Architecture
- **Decision:** No auth/multi-tenancy in Phases 0–5. Single-user only.
- **Why:** CareerOps is a personal tool, not a SaaS product. Adding multi-user complexity now is overengineering for a non-existent use case.
- **Consequence:** Simpler schema (no `user_id` columns). If retroactively opened to other users later, tenant isolation must be retrofitted — accepted tradeoff.
- **ADR:** `Decisions/2026-09-06-single-user-architecture-no-multi-tenancy-in-phase-0-5.md`

### 2. Stack: Supabase + Node/TS + Prisma/Drizzle
- **Database:** Supabase (managed Postgres)
  - Free tier covers single-user scope
  - Network-reachable from day one (supports live-demo-for-interviews plan)
  - Rob already knows Supabase from Valerie project (reduces new-tool overhead)
  
- **Backend API:** Node/TypeScript with Express or Fastify
  - TS end-to-end with frontend (no context-switching languages)
  - ORM: Prisma or Drizzle (both work well with Postgres and single-developer workflow)
  - Deploys to Render or Fly.io (Netlify won't run a persistent backend process)
  
- **Frontend:** React + TypeScript with Vite
  - Deploys to Netlify
  - Calls out to API for dynamic data
  
- **Why this stack:** Postgres is the right category (relational data: Users, Opportunities, Applications, SearchProfiles). Supabase is the right provider (managed, known, free tier sufficient). TS is the right choice across both (one language, less cognitive load while learning architecture). Node/Express/Fastify is straightforward for a first serious backend project.

- **ADR:** `Decisions/2026-09-07-stack-supabase-postgres-node-typescript-prisma-drizzle.md`

### 3. Flat Repository Structure (Not Monorepo)
- **Decision:** `apps/api`, `apps/web` as top-level app folders. Shared code as plain modules imported by relative path — no npm workspaces, no `packages/` yet.
- **Why:** Monorepo tooling (workspace config, cross-package builds) adds real overhead with no payoff until there's a second real consumer of shared code. That's not the case today.
- **Future:** Migrating to a monorepo later (if/when `packages/` genuinely need independent versioning) is a cheap, well-understood refactor — not locked out, just deferred.
- **ADR:** `Decisions/2026-09-07-flat-repository-structure-over-monorepo-workspace-tooling.md`

### 4. CI / Version Control
- **CI Tool:** GitHub Actions
- **What runs:** lint + typecheck + test on every push to main branches
- **Why:** Cheap to set up now; expensive to retrofit once there's real code to bring into compliance
- **Setup:** Happens in next task (repo restructure + initial app scaffolds)

---

## Phase 0 Tasks (In Order)

### ✅ Task 1: ADR Logging & Roadmap Update
**Status:** DONE (2026-09-07)
- ADRs created for all three major decisions
- Roadmap updated with Phase 0 exit criteria and resolved decisions
- Obsidian project notes up to date

### 🔄 Task 2: Create Supabase Project (IN PROGRESS)
**What you're doing:** Provisioning a managed Postgres database.

**Steps:**
1. Go to supabase.com, sign in, create a **new project** (don't reuse Valerie's)
2. Name: `careerops`
3. Region: closest to your location
4. **Save the database password** to your password manager (NOT in Obsidian, NOT in a file)
5. Once provisioned, note:
   - Connection string (direct Postgres connection)
   - Project URL and API keys (if you ever need the REST/realtime APIs)
6. Test connectivity with a Postgres client (e.g., `psql` from terminal)

**Open decision:** Connection pooling mode — for a single-user API process, direct connection is simpler and sufficient. Flag if you're planning serverless instead.

**Do NOT commit the connection string yet** — no `.env` file exists until the next task.

### ⏳ Task 3: Repository Restructure + App Scaffolds
**What you're doing:** Initialize both `apps/api` and `apps/web` with minimal "hello world" end-to-end wiring.

**Steps:**
1. Remove empty `packages/job-engine`, `packages/matching-engine`, `packages/shared` folders (nothing lost)
2. In `apps/api`:
   - `npm init`, add TypeScript, Fastify (or Express), one `GET /health` route
   - Create `.env.example` with placeholder `DATABASE_URL=...`
   - Create `.env` (real Supabase connection string, gitignored)
   - Add `tsconfig.json`, `.gitignore`
3. In `apps/web`:
   - Scaffold with Vite: `npm create vite@latest . -- --template react-ts`
   - Add a simple fetch to `/api/health` on mount
   - Display the API's response on the page
4. Add `.gitignore` at repo root (ignore `node_modules`, `.env`)
5. Test: `npm run dev` in both folders, verify web app displays "API is healthy" from the running API
6. Stop API, reload web app — should show error state (not silent blank)

**Expected result:** Proof that the wiring works end-to-end before touching any real feature logic.

**Git workflow:**
- Commit 1: Restructure (remove empty packages/)
- Commit 2: API scaffold + health endpoint
- Commit 3: Web scaffold + API client
- Commit 4: `.gitignore` update + `.env` setup

### ⏳ Task 4: GitHub Actions CI Setup
**What you're doing:** Add workflows to run lint + typecheck + test on push.

**Steps:**
1. Create `.github/workflows/ci.yml`
2. Define:
   - `npm run lint` (ESLint or Biome)
   - `npm run typecheck` (tsc --noEmit for both apps)
   - `npm run test` (placeholder until tests exist)
3. Runs on push to `main`, `develop`, and all `feature/*` branches
4. Fails the build if any step fails

### ⏳ Task 5: Documentation Skeleton
**What you're doing:** Create minimal but real docs that future phases will expand.

**Files:**
- `docs/architecture/README.md` — brief architecture overview (updates as phases progress)
- `docs/security/README.md` — placeholder for threat modelling (Phase 4+ concern)
- Top-level `README.md` — project identity (what this is, why, how to run locally)

**Exit criteria for Phase 0:** All above complete, repo is usable template for Phase 1 work.

---

## Open Decisions (Still TBD)

1. **Secrets handling strategy**
   - Current plan: `.env` file (gitignored) with `.env.example` committed as template
   - Alternatives: vault/secret manager (overkill for single-user personal tool)
   - Confirm this approach before treating as settled

2. **Source strategy for job ingestion** (Phase 1)
   - Official job board APIs vs scraping
   - Requires research before implementation (ToS, rate limits, auth)
   - Not a Phase 0 blocker, but flag early

3. **ORM choice** (Prisma vs Drizzle)
   - Both solid; pick one and note reasoning in a commit message
   - Not a full ADR-level decision (routine choice)

---

## Repository Structure (After Phase 0)

```
careerops/
├── apps/
│   ├── api/                    # Node/TS backend
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   ├── services/
│   │   │   ├── types/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── .env.example       # Template, committed
│   │   └── .env               # Real secrets, gitignored
│   │
│   └── web/                    # React frontend
│       ├── src/
│       │   ├── components/
│       │   ├── pages/
│       │   └── App.tsx
│       ├── package.json
│       ├── tsconfig.json
│       └── vite.config.ts
│
├── docs/
│   ├── architecture/           # ADRs + system design
│   ├── security/               # Threat model, secrets handling
│   └── README.md               # Development guide
│
├── .github/
│   └── workflows/
│       └── ci.yml              # Lint + typecheck + test
│
├── .gitignore
├── README.md                   # Project identity & how to run
└── PHASE_0_SUMMARY.md          # This file
```

---

## Key Resources

- **Operating prompt (governs mentoring):** `D:\Work\AI\Prompts\GitHub\career-tool`
- **Obsidian project hub:** `D:\Work\Obsidian-Brain\BRAIN\01 - PROJECTS\CareerOps\`
- **ADR index:** `CareerOps/Decisions/📖 ADR Index.md`
- **Roadmap:** `CareerOps/Planning/🗺️ Phased Build Roadmap.md`
- **Session logs:** `CareerOps/Logs/` (create dated notes as you work)

---

## What Comes After Phase 0

Once Phase 0 is complete, Phase 1 (Discovery) begins:

- Integrate first job source (API or scraper, research first)
- Normalize opportunities to a consistent schema
- Implement SearchProfile model (keywords, regions, roles, exclusions)
- Wire up Prisma/Drizzle migrations and DB schema
- Build basic UI for profile switching and seeing result differences

**MVP boundary:** Phase 0 + Phase 1 + deterministic Phase 2 (non-AI scoring) = a real, usable tool worth demoing.

---

## Notes for Future Sessions

- **Secrets:** Database password is in your password manager. Connection string goes in `.env` (gitignored). Never put any of this in Obsidian.
- **Decisions:** If something feels like a meaningful choice, create an ADR (use `mcp__obsidian__log_decision`). Routine choices (ORM, CSS library) just get noted in commit messages.
- **Scope:** Phase 0 is *only* infrastructure. No job-scanning, no matching, no UI beyond "hello world" proving the wiring works. Focus is on removing unknowns from the plumbing.
- **Testing:** Tests don't exist yet (placeholder `npm run test` in CI). Add unit tests for Phase 1 logic once you're building actual features.

---

**Last updated:** 2026-09-07  
**Current phase:** Phase 0 (In Progress)  
**Next task:** Supabase project creation
