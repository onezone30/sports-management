# Tech Stack

Sports-management is a rec-league app for managing teams, schedules, and stats, built to be flexible across different sports (not locked to one sport's data shape).

## Stack

| Layer | Choice |
|---|---|
| Backend | Laravel (JSON API only, no Blade/Inertia) |
| Frontend | React + TypeScript SPA (separate codebase, built with Vite) |
| Frontend routing | React Router |
| Database | MySQL |
| Auth | Laravel Sanctum, SPA mode (cookie-based session) |
| HTTP client | axios |
| Frontend state | Plain `useState` / `useContext` (no query-caching library) |
| Styling | Tailwind CSS |
| Components | shadcn/ui (Tailwind + Base UI primitives, copy-paste ownership) |
| Backend testing | Pest |
| Frontend testing | Vitest |
| Hosting | Not decided yet, local dev only for now |

## Why this shape

**TypeScript over plain JavaScript.** Chosen despite adding a second learning curve alongside React and Laravel — the type layer catches prop/shape mismatches at write-time instead of runtime (especially useful for matching whatever the Laravel API actually returns), and it's the default in most real-world React codebases, so better to build the habit from the start than retrofit it later.

**Laravel API + React SPA (decoupled), not Inertia.** Chose the fully separate setup over Inertia.js on purpose, even though Inertia is less work: this forces real practice with API design, request/response contracts, and auth across a network boundary, rather than Laravel and React being glued together in one process. More moving parts, more learning surface.

**MySQL.** Laravel's default, most documentation assumes it, no reason to fight the grain while still learning Laravel fundamentals.

**Sanctum SPA mode over token auth.** Cookie-based session auth is simpler and safer by default for a first-party SPA talking to its own backend (no token storage/XSS surface in the browser). Trade-off: frontend and backend need to share a top-level domain (or be configured as Sanctum "stateful domains") — this applies in local dev too, not just production.

**Plain `useState`/`useContext`, no TanStack Query.** Deliberately skipping a data-fetching library so loading states, error states, and refetch-after-mutation logic get written by hand. This is intentional friction for learning, not an oversight — expect it to feel repetitive by the 4th or 5th CRUD screen (teams, schedules, stats, players, standings). That repetition, when it shows up, is the natural signal for introducing a fetching library later — not a sign the current approach is wrong.

**Tailwind + shadcn/ui.** shadcn/ui ships as copied-in component source (not an opaque npm dependency), so components can be read and modified directly — fits learning by understanding internals rather than treating UI as a black box. Traditional component libraries (MUI, Chakra, Mantine) were ruled out because they bring their own styling engine and conflict with Tailwind.

**Pest + Vitest chosen, not yet prioritized.** Tooling decided up front so there's no future "which test framework" detour, but writing tests is not a current priority — expected to be picked up once the app's shape stabilizes.

**Hosting deferred.** No hosting decision needed yet; this stack (Laravel API + static-buildable React SPA + MySQL) doesn't lock in any particular host, so the decision can wait until there's something worth deploying.

## Local dev notes

- Two apps, one per directory: `backend/` for Laravel, `frontend/` for React+Vite — see `docs/folder-structure.md`.
- Sanctum's stateful-domain config needs to include whatever host:port the Vite dev server runs on (e.g. `localhost:5173`) for session cookies to work locally, matching how it'll need to work in production (shared top-level domain).
- CORS must be configured on the Laravel side to allow the SPA's dev origin.

## Open / deferred

- Data model for "flexible across sports" (how teams/schedules/stats stay generic instead of hardcoded to one sport) — not covered here, needs its own design pass before building the schema.
- Hosting target.
- When to introduce a data-fetching library on the frontend.
