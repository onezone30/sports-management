# Naming Conventions

## Laravel (`backend/`)

Mostly fixed by the framework, not a real choice:

- Classes: `StudlyCase` — `Team.php`, `TeamController.php`
- Database tables: `snake_case`, plural — `teams`, `stat_types`
- Database columns: `snake_case` — `created_at`, `season_id`
- Migrations: Laravel's own timestamped format, generated via `artisan make:migration`
- Routes/URIs: `kebab-case`, plural nouns — `/api/teams`, `/api/stat-types`

### Repositories

- Interface: `{Model}RepositoryInterface.php` in `app/Repositories/Contracts/` — e.g. `TeamRepositoryInterface.php`
- Implementation: `{Model}Repository.php` in `app/Repositories/Eloquent/` — e.g. `TeamRepository.php`
- Methods: generic, Eloquent-flavored — `all()`, `find($id)`, `create(array $data)`, `update($id, array $data)`, `delete($id)`. A repository's job is data access only, nothing sport/business-specific.
- Interfaces are bound to their Eloquent implementation in `app/Providers/RepositoryServiceProvider.php`, registered in `config/app.php` — this is what lets a Controller type-hint the interface and get the real implementation injected.

### Actions

- `{Verb}{Noun}Action.php` in `app/Actions/{Domain}/` — one class per business operation, grouped by domain folder — e.g. `app/Actions/Team/RegisterTeamForSeasonAction.php`, `app/Actions/Game/SubmitGameResultAction.php`
- One public method: `handle(...)`. No other public methods — if an Action needs to do more than one thing, split it into another Action.
- Class name describes the business action, not CRUD — `RegisterTeamForSeasonAction`, not `TeamCreateAction`.
- Actions orchestrate: call Repositories for persistence, call Model methods for self-contained rules (e.g. `$season->isOpenForTeamRegistration()`), and may call other Actions for composition.

## React + TypeScript (`frontend/`)

- Components: `PascalCase.tsx` — `TeamPage.tsx`, `StandingsTable.tsx` (JSX convention, not optional)
- Hooks: `camelCase.ts`, must start with `use` — `useTeams.ts` (React rule, not optional)
- Types, lib utils, context files: `camelCase.ts` — `apiClient.ts`, chosen for consistency with the variable/function names inside those files
- Type files: singular per domain concept — `team.ts` exports `interface Team`, not `teams.ts`

## Cross-cutting

- API endpoints: plural nouns, matching the DB table names — `GET /api/teams`, `POST /api/teams`, `GET /api/stat-types`
