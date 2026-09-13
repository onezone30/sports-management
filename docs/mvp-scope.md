# MVP Scope

Defines what ships in v1 of sports-management, and what's deliberately deferred. See `docs/tech-stack.md` for the stack this is built on.

## Roles

- **Admin** — runs the leagues, only role with write access.
- **Public Viewer** — no auth, read-only access to schedules/standings/stats.

No other roles (Team Manager, Referee, Player accounts) exist in MVP. See Deferred section for why.

## Data hierarchy

```
League (has a sport)
  -> Season
       -> Team
       -> Game (schedule + result)
       -> Stat Type (per-sport: goals/assists/cards for soccer, points/rebounds/fouls for basketball, etc.)
```

Stat Types are per-league (scoped to that league's sport), not global — this is the mechanism that makes the app sport-agnostic instead of hardcoded to one sport's stat categories.

## MVP features

**Admin can:**
- Create/edit leagues (each tied to a sport)
- Create/edit seasons within a league
- Create/edit teams within a season
- Add players to a team roster (name + jersey number only — no photos, bios, or player accounts)
- Build a schedule: create games with date/time, opponents, and a venue as a plain text field (not a managed entity)
- Enter game results and stats after a game, at team level (e.g. "Team A: 3 goals" — not attributed to individual players)

**Public Viewer can:**
- Browse leagues and seasons
- View a season's schedule
- View standings (computed automatically from game results — not manually entered, to avoid standings drifting out of sync with results)
- View team stats

**Multi-league/multi-sport:** the app supports multiple leagues running concurrently, each potentially a different sport, from day one — this isn't deferred, it's core to the hierarchy above.

## Deferred (not MVP)

Each of these is a real, likely-needed feature — just not required for a working first season.

- **Team Manager/Coach role** — self-service login so a coach manages their own team's roster/results instead of the admin doing all data entry. Deferred because it adds per-team authorization logic (a coach can only touch their own team) on top of everything else.
- **Referee/Scorekeeper role** — a role solely for entering live results. For a rec league this is usually the same person as admin or a coach; not worth a separate role until that's no longer true.
- **Player accounts/logins** — individual players don't get accounts; public standings/schedule pages cover "can I see my team's info" without needing per-player auth.
- **Player-level stats** — stats start at team level only (e.g. final score). Attributing individual stats to a specific player (who scored, who got the rebound) roughly doubles data-entry work and depends on the roster being reliable first. Top candidate for the next round of features after MVP, but not committed to being "the very next thing" — priority after MVP ships will depend on what actually turns out to matter once a season is running.
- **Divisions/Conferences within a season** — grouping teams into subgroups for standings/scheduling. Only matters once a season has enough teams that one flat standings table stops making sense (e.g. 20+ teams). Deferred because it's additive later (a new layer on top of Season -> Team) rather than something that reshapes the core data model, unlike player-level stats.
- **Registration/payments** — players/teams paying league fees. A subsystem of its own.
- **Game events / play-by-play** — granular in-game event logging instead of final stats.
- **Announcements/news, team logos/photos** — presentation/content features, not core to running a league.
- **Full Venue entity** — a managed, reusable Venue with its own admin screen. MVP just stores venue as free text on a Game.

## Why this scope

The guiding cut was: what's the minimum needed to actually run one full season of a rec league, across any sport, without hardcoding assumptions that would need reworking later? Stat Types and basic rosters are in MVP specifically because leaving them out would mean the "flexible for all sports" premise isn't actually true yet — everything else deferred is real product value that doesn't block a working season.
