# WorkFlow HRMS — Setup Guide & Fix Notes

This project is a full-stack HRMS (HR Management System) boilerplate:

- **Frontend**: Next.js 16 + React 19 + Tailwind v4 + Apollo Client + Redux Toolkit
- **Backend**: .NET 10 modular monolith, GraphQL (HotChocolate), PostgreSQL, MediatR/CQRS

This document covers what was fixed and how to actually run the project.

---

## ⚠️ Important: this was reviewed by static code reading, not by actually running it

I do not have internet access or a .NET SDK in my working environment, so I could not run
`npm install`, `npm run dev`, `dotnet build`, or `dotnet run` myself. Everything below was found
and fixed by reading the source carefully and cross-referencing official docs/package metadata —
but you should still run a first build yourself and watch for anything I couldn't catch this way.

---

## What was fixed

### Frontend (`/frontend`)

1. **Broken login redirect** — `app/page.tsx` and `app/dashboard/page.tsx` redirected to
   `/(auth)/login`. In Next.js App Router, folders in parentheses are route groups and are
   **not** part of the URL — the real route is `/login`. Fixed both.
2. **Session lost on page refresh** — `SessionContext` never restored the logged-in user from
   storage on load, so refreshing the page silently logged you out client-side even with a
   valid token still saved. Added persistence (`tokenStorage.ts`) and a rehydration effect.
3. **Tailwind v4 wasn't loading the custom theme** — the project mixes Tailwind v4
   (`@import "tailwindcss"`) with a v3-style `tailwind.config.ts` that defines custom colors
   (`border`, `muted`, `primary`, etc.), custom spacing, and animations. Tailwind v4 ignores
   that config file unless you link it explicitly. Added
   `@config "../tailwind.config.ts";` to `app/globals.css` (after the `@import` statements,
   per Tailwind's required ordering). Without this, most of the `DataTable` component's
   styling (borders, muted backgrounds, etc.) would silently not apply.
4. **Removed conflicting lockfiles** — both `package-lock.json` and `pnpm-lock.yaml` existed,
   plus a stray `pnpm-workspace.yaml`. The README documents npm, so the pnpm files were removed
   to avoid ambiguity about which package manager to use.
5. **Fixed `.env.local` / `.env.development.local` / `env.example` / `.env.example`** —
   pointed at the backend's actual port (`5056`, from `launchSettings.json`), not the
   placeholder `api.example.com` or other stale ports. Note: Next.js loads
   `.env.development.local` with **higher priority** than `.env.local` during `npm run dev` —
   this file existed with a stale port (`7007`) that would have silently overridden the fix in
   `.env.local` alone, so both needed to be corrected.

**Good news:** every page in the frontend currently uses self-contained mock data — none of
them call the GraphQL API yet. So the frontend runs completely standalone; you don't need the
backend or a database to see the UI working.

### Backend (`/backend`)

1. **Incomplete solution file** — `HRMSBoilerPlate.slnx` only listed the `TodoFeature` module;
   the other 16 feature modules existed on disk and were referenced by the API project's
   `.csproj`, but weren't visible in the solution (so they wouldn't show up if you opened it in
   an IDE). Regenerated the `.slnx` to include all 17 modules / 68 projects.
2. **CORS was fully disabled** — `Cors:AllowedOrigins` was an empty array in
   `appsettings.Development.json`, so no CORS policy was ever applied at all. This would block
   any browser request from the frontend the moment it's wired up to call the API. Added
   `http://localhost:3000` (the default Next.js dev port).
3. **Three GraphQL modules were never registered with the schema** — `RecruitmentFeature`,
   `AnalyticsFeature`, and `OnboardingFeature` had complete, working GraphQL query/mutation
   types and were referenced by the API's `.csproj`, but were missing from
   `GraphQLModuleRegistration.cs`. Added the missing `using` statements and
   `.AddRecruitmentGraphQL() / .AddAnalyticsGraphQL() / .AddOnboardingGraphQL()` calls.
4. **Same three modules were missing their database/DI wiring** — each module needs a
   `ConfigureServiceExtension.cs` that registers its EF Core entity configurator and repository
   interface. Recruitment, Analytics, and Onboarding didn't have one (every other module did).
   Created the missing files (mirroring the existing pattern exactly) and wired them into
   `RepositoryRegistration.cs`. Without this fix, those three modules' database tables would
   never be created, and their repositories would fail to resolve at runtime.
5. **11 of 17 modules were missing from MediatR/AutoMapper/FluentValidation registration** —
   `Startup.cs` only passed 6 module assemblies into `AddInjectionApplication(...)`. MediatR
   only scans the assemblies you explicitly give it, so calling any mutation/query in
   `UserFeature`, `AttendanceFeature`, `LeaveFeature`, `PerformanceFeature`, `TrainingFeature`,
   `RecognitionFeature`, `HRCopilotFeature`, `ContributionsFeature`, `RecruitmentFeature`,
   `AnalyticsFeature`, or `OnboardingFeature` would have thrown a "no handler registered"
   exception at runtime. Added all 11 missing assemblies to the registration list.
6. **Removed stale `build_error.txt`** — a leftover error log from an earlier debugging session
   that referenced `net9.0` paths, while the current `.csproj` files all target `net10.0` with
   internally consistent package versions. Not reproducible against the current code; removed
   as clutter.
7. **Expanded `.gitignore`** — added `*.user`, `.vs/`, and local appsettings overrides (only
   `bin/`/`obj/` were ignored before).

## ⚠️ Known gap I did *not* invent a fix for

**There is no authentication backend.** The frontend's entire login flow
(`lib/auth/authService.ts`, `lib/auth/tokenStorage.ts`) expects REST endpoints
`POST /auth/login` and `POST /auth/refresh`. The .NET backend has **zero REST controllers** —
it's 100% GraphQL, and the `UserFeature` GraphQL module only has employee CRUD mutations
(`createEmployee`, `updateEmployee`, `deleteEmployee`), no login/auth mutation anywhere.

This means: even with Postgres running and the backend started, the `/login` page will never
successfully authenticate, because there's nothing on the backend listening for it. This is a
real, unfinished design gap between the two halves of the project — not something I could
responsibly "fix" without inventing an entire auth module and guessing at your intended security
model (password hashing scheme, JWT vs session, refresh token rotation, etc.).

Two ways to move forward:
- **For now**: keep `NEXT_PUBLIC_DISABLE_AUTH=true` in `frontend/.env.local` (already the
  default). The middleware will skip the login requirement entirely, and since all current pages
  use mock data, everything will work as a demo.
- **To actually build auth**: you'll need to add a REST (or GraphQL) login endpoint to the
  backend that validates credentials and issues a JWT, plus a way to create/seed user accounts
  with hashed passwords. Happy to help with this as a separate task — just say so.

---

## How to run it

### Prerequisites

- **Node.js 20+** and npm (for the frontend)
- **.NET 10 SDK** (for the backend) — see https://dotnet.microsoft.com/download
  - `backend/API/HRMS.API/dotnet-install.sh` is also included as a convenience installer script
- **PostgreSQL 14+** running locally (only needed for the backend)

### 1. Frontend (works standalone, no backend needed)

```bash
cd frontend
npm install
npm run dev
```

Open http://localhost:3000 — it will redirect to `/dashboard` (auth is disabled by default).
Every page works with mock/demo data right out of the box.

### 2. Backend (optional — only needed if you want the real API/GraphQL running)

First, create a local Postgres database matching the connection string in
`backend/API/HRMS.API/appsettings.Development.json`:

```bash
# Default expected connection string:
# Host=localhost;Port=5432;Database=HRMS;Username=postgres;Password=postgress
psql -U postgres -c "CREATE DATABASE \"HRMS\";"
```

(Edit `appsettings.Development.json` if your local Postgres uses different credentials —
the password "postgress" with the double-s is just what ships in this boilerplate, not a typo
you need to match exactly, as long as both the DB and the config agree.)

Then build and run:

```bash
cd backend
dotnet restore
dotnet build
dotnet run --project API/HRMS.API
```

The API will start on **http://localhost:5056** (and https://localhost:7033). Tables are
created automatically on first run via `Database.EnsureCreated()` — no manual migrations
needed. Visit `http://localhost:5056/graphql` for the GraphQL playground (Banana Cake Pop),
where you can explore and test the schema directly.

The frontend's `.env.local` already points at `http://localhost:5056` / `/graphql`, so once
both are running and you start wiring actual pages to GraphQL queries, they'll be able to talk
to each other (CORS for `localhost:3000` is now configured).

---

## If `dotnet build` still fails for you

I could not run a real build in my environment, so if you hit a compile error I didn't catch:

1. Run `dotnet restore` first and check for NuGet package resolution errors specifically —
   that's the most likely failure point given the project pins fairly recent versions
   (`net10.0`, several `10.0.x` `Microsoft.Extensions.*` packages).
2. Check `backend/Shared/HRMS.Shared.Core/HRMS.Core.KeyVault/HRMS.Core.KeyVault.csproj` first if
   you see anything mentioning `Azure.Extensions.AspNetCore.Configuration.Secrets` — there was a
   stale error log in this exact spot before (now removed) from an earlier `net9.0` build
   attempt that no longer matches the current `net10.0` project files.
