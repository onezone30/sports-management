# API Design

REST JSON API. Public read access, Admin write access — same resource URLs for both, auth enforced per HTTP verb (`GET` open, `POST`/`PATCH`/`DELETE` require `auth:sanctum`). See `architecture-rules.md` for why this replaced an earlier `/api/admin/*` namespace idea, and `naming-conventions.md` for the kebab-case/plural convention these follow.

Nesting: shallow — list/create nested under the parent resource, single-resource operations (show/update/delete) flat by the resource's own ID. Matches how the resources actually depend on each other in `data-model.md`.

## Routes

```
Auth
  POST   /api/login
  POST   /api/logout
  GET    /api/user                                current admin session, or null

Sports (admin creates freely, not a fixed list)
  GET    /api/sports
  POST   /api/sports                              auth
  GET    /api/sports/{sport}
  PATCH  /api/sports/{sport}                      auth
  DELETE /api/sports/{sport}                      auth

Sport Stat Type Templates
  GET    /api/sports/{sport}/stat-type-templates
  POST   /api/sports/{sport}/stat-type-templates  auth
  PATCH  /api/stat-type-templates/{template}      auth
  DELETE /api/stat-type-templates/{template}      auth

Leagues
  GET    /api/leagues
  POST   /api/leagues                             auth — copies the sport's templates into this league's stat-types
  GET    /api/leagues/{league}
  PATCH  /api/leagues/{league}                    auth
  DELETE /api/leagues/{league}                    auth

Stat Types (league-owned, independent after the copy)
  GET    /api/leagues/{league}/stat-types
  POST   /api/leagues/{league}/stat-types         auth
  PATCH  /api/stat-types/{statType}               auth
  DELETE /api/stat-types/{statType}               auth

Seasons
  GET    /api/leagues/{league}/seasons
  POST   /api/leagues/{league}/seasons            auth
  GET    /api/seasons/{season}
  PATCH  /api/seasons/{season}                    auth
  DELETE /api/seasons/{season}                    auth

Teams
  GET    /api/seasons/{season}/teams
  POST   /api/seasons/{season}/teams              auth
  GET    /api/teams/{team}
  PATCH  /api/teams/{team}                        auth
  DELETE /api/teams/{team}                        auth
  POST   /api/teams/{team}/logo                   auth — separate Action from team create/update

Players (roster)
  GET    /api/teams/{team}/players
  POST   /api/teams/{team}/players                auth
  PATCH  /api/players/{player}                    auth
  DELETE /api/players/{player}                    auth

Games
  GET    /api/seasons/{season}/games              the schedule
  POST   /api/seasons/{season}/games              auth — create/schedule
  GET    /api/games/{game}
  PATCH  /api/games/{game}                        auth — reschedule
  DELETE /api/games/{game}                        auth
  POST   /api/games/{game}/result                 auth — submit result + stats (SubmitGameResultAction)

Standings (computed, read-only — see data-model.md)
  GET    /api/seasons/{season}/standings
```

## Response shape

- Single resource: `{ "data": { ...fields } }` (Laravel API Resource default wrapping).
- Collection: `{ "data": [ { ... }, { ... } ] }`.
- `Team`/`Game` etc. responses expose only the fields defined by their Resource (see `architecture-rules.md`) — never raw Eloquent output.
- File fields (e.g. team logo) are returned as a full URL (`logo_url`), not the stored path.

## Error shape

Matches `architecture-rules.md`'s error handling — every error response:

```
{ "message": "Human-readable explanation" }
```

with the HTTP status carrying the meaning:

- `404` — resource not found
- `422` — validation failure (Form Request rejected the input shape)
- `409` — business-rule conflict (e.g. game already has a result)
- `403` — unauthorized (not logged in / not allowed to perform this action)

## Auth flow (Sanctum SPA mode)

1. Frontend calls `GET /sanctum/csrf-cookie` (provided by Sanctum itself) before the first state-changing request, to get a CSRF cookie.
2. `POST /api/login` with credentials — sets a session cookie on success.
3. Subsequent requests include the session cookie automatically (`withCredentials: true` in axios, per `folder-structure.md`'s `lib/api.ts`).
4. `GET /api/user` lets the frontend check on load whether a session is already active (e.g. after a page refresh).
5. `POST /api/logout` clears the session.
