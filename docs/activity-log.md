# Activity Log

Chronological log of decisions and progress, for future reference. Not a duplicate of the docs themselves — see the linked doc for full detail on any entry.

## 2026-09-14

- Chose tech stack: Laravel (API-only) + React/TypeScript SPA, MySQL, Sanctum SPA auth, Tailwind + shadcn/ui, Pest + Vitest (tooling only, testing not yet prioritized). See `tech-stack.md`.
- Defined MVP scope: Admin + Public Viewer roles only, League->Season->Team->Game->Stat Type hierarchy, multi-league/multi-sport concurrent support, team-level stats (player-level deferred), Divisions/Conferences deferred. See `mvp-scope.md`.
- Decided monorepo layout: `backend/` + `frontend/` (renamed from `api/`/`web/`). See `folder-structure.md`.
- Wrote `naming-conventions.md` and `RULES.md` (learning-project rules: Claude doesn't write/edit code or run scaffolding, gives options with reasoning, walks through debugging instead of naming root causes).
- Researched and decided backend pattern: Repository (kept deliberately, despite it being non-idiomatic Laravel, for the general design-pattern learning value) + Action (one class per business operation, replacing an earlier Services approach). See `architecture-rules.md` for full layer responsibilities (Controller/Form Request/Action/Model/Repository/Resource on the backend; Page/Component/Hook/Context/lib/types on the frontend), plus error handling, authorization (middleware now, Policies deferred until Team Manager role exists), testing-per-layer, and route-guarding conventions.
- Designed the data model: `sports`, `sport_stat_type_templates`, `leagues`, `seasons`, `stat_types`, `teams`, `players`, `games`, `game_stats`. Key calls: score is a real column on `games` (not a generic Stat Type, since it's the most-read field in the app); Stat Types are per-league, seeded from a per-sport template; Sport is admin-created freely, not a fixed list; no stored standings table (computed on the fly). See `data-model.md`.
