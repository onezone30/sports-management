# Data Model

This is the current baseline, not a final spec — expected to expand as the project goes. See "Not yet modeled" at the bottom for known gaps. See `mvp-scope.md` for why these entities exist and `architecture-rules.md` for how they're used across layers.

## Entities

- **`sports`** (id, name) — admin-created freely, not a fixed list.
- **`sport_stat_type_templates`** (id, sport_id, name) — default Stat Types for a sport, admin-defined. Copied into a League's own `stat_types` when that league is created; editing a template afterward does not affect leagues that already copied it.
- **`leagues`** (id, sport_id, name) — a league is tied to one sport, and multiple leagues can run concurrently, including multiple leagues of the same sport.
- **`seasons`** (id, league_id, name, start_date, end_date, status)
- **`stat_types`** (id, league_id, name) — a league's own copy of its stat categories, independently editable per league after creation.
- **`teams`** (id, season_id, name, logo_path nullable)
- **`players`** (id, team_id, name, jersey_number) — roster only, no stats attached to a player in MVP.
- **`games`** (id, season_id, home_team_id, away_team_id, scheduled_at, venue, status, home_score nullable, away_score nullable) — score is a first-class field, not a Stat Type (see rationale below); nullable until the game has a result.
- **`game_stats`** (id, game_id, team_id, stat_type_id, value) — team-level only, no `player_id` in MVP. See "Not yet modeled" for how this extends to player-level later.

## Hierarchy

```
Sport
  -> Sport Stat Type Template
  -> League
       -> Stat Type (copied from Sport template, then independent)
       -> Season
            -> Team
                 -> Player (roster)
            -> Game (home_score, away_score)
                 -> Game Stat (per team, per Stat Type)
```

## Key decisions and why

**Score is a real column on `games`, not a Stat Type.** Considered making it fully generic (every number, including score, stored as a `game_stats` row with a Stat Type flagged "primary"), but rejected: score is read on nearly every screen (schedule, standings, game cards), while other stats are supplementary detail. Forcing the single most-accessed field in the app behind a join, purely to keep one mechanism for everything, optimizes for schema purity over actual usage. Standings are computed directly from `home_score`/`away_score` — simple column comparison, no join needed.

**Stat Types are per-league, seeded from a per-sport template ("Both" option).** Pure per-sport (shared, no override) would mean two leagues of the same sport can never track different things. Pure per-league (no template) means re-creating "Goals, Assists, Cards" from scratch every time a new same-sport league is created. The template approach gets fast setup for a new league *and* full independence afterward — editing one league's Stat Types never affects another league, even of the same sport.

**Sport is admin-created, not a fixed list.** Required for "flexible across all sports" to actually be true — nothing about soccer or basketball is hardcoded anywhere in the schema.

**No `standings` table.** Standings are computed on the fly from `games` (wins/losses/ties from `home_score` vs `away_score`), not stored — avoids standings ever drifting out of sync with actual game results.

**Venue is a plain string on `games`**, not its own entity — MVP doesn't need a managed, reusable Venue.

## Not yet modeled

- **Player-level stats** — deferred from MVP. Extends cleanly later via a nullable `player_id` column added to `game_stats` (existing team-level rows keep `player_id = null`); no rework of what's built now.
- **Team Manager / Coach role and per-team authorization** — deferred; will need Policies (see `architecture-rules.md`) once it exists, since it's a per-record permission check.
- **Divisions/Conferences within a season** — deferred; additive later as a layer between Season and Team.
- **Registration/payments** — not modeled at all yet, own subsystem.
- **Full Venue entity** — currently just a string; would need its own table if venues need to be managed/reused/looked up later.
- **Game events / play-by-play** — not modeled; current model only captures final per-game, per-team totals.
- **Whatever else comes up while building** — this section is the landing spot for it.
