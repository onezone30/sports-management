# Architecture Rules

Backend flow: **Controller → Action → Repository → Model**, plus Form Requests and Resources at the edges. Each layer has exactly one job. See `folder-structure.md` for where files live and `naming-conventions.md` for what they're called.

## Controller

- Job: translate HTTP in, HTTP out. Nothing else.
- Calls one Form Request (validation), one Action (the operation), returns one Resource (response shape).
- Must never: contain business logic, query the database directly, or call a Repository directly.

## Form Request

- Job: validate the *shape* of the input (required, string, image, max size, etc.).
- Must never: contain business rules (e.g. "is registration open" is not a shape question — that's a Model/Action concern).

## Action

- Job: one business operation. Orchestrates the sequence of what needs to happen.
- One public method: `handle(...)`. If it needs to do more than one thing, split it into another Action.
- Calls Repositories for persistence, calls Model methods for self-contained rules, may call other Actions for composition (e.g. combining "create team" + "upload logo").
- Framework abstractions that are already sufficient (like `Storage`) are used directly in the Action — don't wrap them in another custom layer for the same reason Eloquent doesn't need a Repository-like wrapper around `Storage`.
- Must never: contain raw Eloquent queries — that belongs in the Repository.

## Model

- Job: hold self-contained facts/rules about its own data — e.g. `Season::isOpenForTeamRegistration()`, `Game::hasResult()`.
- A rule belongs here if it only needs that one record's own data to answer true/false.
- Must never: orchestrate other models, call a Repository, or reach outside its own data to answer a question.

## Repository

- Job: persistence only — read and write data, nothing else.
- Interface (`Contracts/`) + Eloquent implementation (`Eloquent/`), bound in `RepositoryServiceProvider`.
- Methods stay generic and Eloquent-flavored: `all()`, `find($id)`, `create($data)`, `update($id, $data)`, `delete($id)`.
- Must never: leak Eloquent-specific objects (query builders, relationship loaders) through its interface. If a caller needs eager loading or a scope, that's a sign the Repository method needs a clearer, purpose-built method (e.g. `findWithRoster($id)`) — not a raw builder passed through.
- Must never: contain business rules — a Repository doesn't know *why* it's saving something, only *how*.

## Resource

- Job: shape the final JSON returned to the frontend, including any transforms (e.g. converting a stored file path into a public URL via `Storage::url()`).
- Must never: contain business logic — only presentation/formatting.

## Error handling

- Actions throw domain-specific exceptions for business-rule failures (e.g. `GameAlreadyHasResultException`), defined in `app/Exceptions/`.
- Controllers never catch these individually — exceptions bubble up to Laravel's central exception handler, which maps each exception type to an HTTP status + consistent JSON error shape (`{ "message": "..." }`). This keeps error handling in one place instead of repeated try/catch blocks per Controller.
- Each custom exception maps to a specific status: business-rule conflicts (e.g. game already has a result) → 409, not found → 404, validation → 422 (handled automatically by Form Requests), unauthorized → 403.

## Authorization

- MVP has one loggable-in role (Admin) with no per-record distinction — every Admin can do everything. Public and Admin share the same resource URLs (e.g. `/api/leagues`), with `auth:sanctum` applied per HTTP verb: `GET` routes stay open (public read), `POST`/`PATCH`/`DELETE` require auth. A separate `/api/admin/*` namespace was considered and rejected — it would just be a second URL for the same resource. See `api-design.md` for the full route list.
- Policies (per-record checks, e.g. `TeamPolicy::update(User $user, Team $team)`) are intentionally deferred — there's nothing for them to differentiate yet. Add Policies when the Team Manager role ships (needs "can only edit their own team," which middleware can't express).
- Gates (broad, non-record capability checks in a service provider) aren't needed yet either, for the same reason — one role, no branching.

## Testing per layer

- **Actions** — unit tests with a mocked/faked Repository (no real database). Tests "given the repository returns X, does the Action produce the right result or throw the right exception." This is the actual payoff of coding to a Repository interface.
- **Repositories** — feature tests against a real test database (Pest + `RefreshDatabase`) — verify `create()`/`find()` actually persist/return correctly.
- **Model rule-methods** — unit tests directly, no DB needed — construct a model with specific attributes, assert the rule method returns the right boolean.
- **Controllers** — feature tests hitting the real HTTP endpoint (`postJson('/api/teams', [...])`), checking status code + response shape. This is the one test that exercises the whole stack together.
- **Frontend (Vitest)** — component tests render with given props and assert output; hook tests mock the axios call and assert loading/error/data state transitions.

## Files/images

- Never stored as binary data in the database — only a path/reference string is stored (e.g. `logo_path`).
- The actual file is handled via Laravel's `Storage` facade directly inside the Action; the Repository just persists the resulting path like any other field.

---

# Frontend (`frontend/`)

Flow: **Page → Hook → `lib/api.ts` (axios)**, with Components as reusable rendering, Context for global state, Types as shared contracts. See `folder-structure.md` for layout.

## Page

- Job: compose components for one route/screen, call hooks to get data, handle page-level layout.
- Must never: call axios directly — always through a hook. A Page not going through a hook means the loading/error handling for that data isn't reusable anywhere else.

## Component

- Job: render UI from props. Two flavors: `ui/` (shadcn primitives) and shared domain components (`StandingsTable.tsx`, `GameCard.tsx`) that take data as props and display it.
- Must never: fetch data itself (no axios, no hooks that hit the network) inside a shared/reusable component — it receives data from the Page that renders it. Keeps a component usable in any context without caring where its data came from.

## Hook

- Job: own the data-fetching lifecycle for one resource — call `lib/api.ts`, hold that call's own loading/error/data state, expose it to whichever Page uses it (e.g. `useTeams.ts`).
- Since there's no query-caching library, each hook is responsible for its own refetch-after-mutation logic (e.g. refetch the roster after adding a player).
- Must never: contain JSX/rendering.

## Context

- Job: global, cross-page state only — currently just `AuthContext.tsx` (is an admin logged in, and their role).
- Must never: become a dumping ground for state that only one Page/Component needs — that stays local (`useState`) in whatever Page owns it.

## `lib/`

- Job: framework-agnostic shared code — the axios instance (`api.ts`, `withCredentials: true` for Sanctum), small pure helper functions.
- Must never: contain React-specific code (no hooks, no JSX) — anything React-specific belongs in `hooks/` or `components/`.

## `types/`

- Job: TypeScript interfaces mirroring what the API returns, one file per domain concept (`team.ts` exports `Team`).
- Must never: contain logic — types only.

## Business rules

- The backend is the source of truth for business rules (e.g. "is registration open"). The frontend may re-check the same rule for UX (disabling a button, showing a message early) but must not treat that client-side check as authoritative — the real enforcement happens in the Action on the backend, since a client-side-only check can always be bypassed.

## Route guarding

- `ProtectedRoute.tsx` wraps a route, reads the current role from `AuthContext`, and checks it against that route's allowed roles (e.g. `<ProtectedRoute allowedRoles={['admin']}>`) — redirects to `/login` if the check fails. This is the standard RBAC pattern for React (role declared once per route, not inferred from file location), not something specific to this project.
- Each route's allowed roles are declared where the route itself is defined — a single, explicit source of truth for "who can access this," instead of relying on which folder a page happens to sit in.
- This is UX only, not security. It runs as JavaScript in the browser and can be bypassed (disabled JS, devtools, or calling the API directly with curl/Postman). The actual security boundary is the backend's `auth:sanctum` middleware (see Authorization above) — `ProtectedRoute` must never be treated as the real access control.
- MVP has only one role (Admin), so today every `ProtectedRoute` just checks "is logged in." The `allowedRoles` list is what makes this extend cleanly to a second role (e.g. Officials) later without restructuring anything — see `folder-structure.md`.
