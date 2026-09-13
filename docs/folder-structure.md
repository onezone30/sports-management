# Folder Structure

Monorepo: `backend/` (Laravel) and `frontend/` (React + TypeScript) side by side.

```
sports-management/
├── backend/                    Laravel (JSON API only)
│   ├── app/
│   │   ├── Http/
│   │   │   ├── Controllers/Api/
│   │   │   ├── Requests/
│   │   │   └── Resources/
│   │   ├── Models/
│   │   ├── Repositories/
│   │   │   ├── Contracts/       interfaces
│   │   │   └── Eloquent/        implementations
│   │   ├── Actions/             one class per business operation, grouped by domain
│   │   │   ├── League/
│   │   │   ├── Season/
│   │   │   ├── Team/
│   │   │   └── Game/
│   │   └── Providers/
│   ├── config/
│   ├── database/
│   │   ├── factories/
│   │   ├── migrations/
│   │   └── seeders/
│   ├── routes/
│   │   └── api.php
│   ├── tests/
│   │   ├── Feature/
│   │   └── Unit/
│   ├── .env.example
│   └── composer.json
│
├── frontend/                   React + TypeScript SPA (Vite)
│   ├── src/
│   │   ├── components/
│   │   │   ├── ui/               shadcn/ui components (button.tsx, dialog.tsx, table.tsx, ...)
│   │   │   ├── layout/           Navbar.tsx, AdminLayout.tsx, PublicLayout.tsx
│   │   │   ├── ProtectedRoute.tsx  checks AuthContext's role against a route's allowed roles
│   │   │   └── (shared domain components: StandingsTable.tsx, GameCard.tsx, TeamBadge.tsx)
│   │   ├── pages/
│   │   │   ├── public/           LeaguesPage.tsx, SeasonPage.tsx, StandingsPage.tsx, TeamPage.tsx
│   │   │   └── admin/            LoginPage.tsx, DashboardPage.tsx, LeaguesAdminPage.tsx,
│   │   │                         SeasonsAdminPage.tsx, TeamsAdminPage.tsx, ScheduleAdminPage.tsx,
│   │   │                         GameResultPage.tsx
│   │   ├── hooks/                useLeagues.ts, useSeasons.ts, useTeams.ts, useGames.ts, useStandings.ts
│   │   │                         (each wraps an axios call + its own loading/error state)
│   │   ├── context/              AuthContext.tsx (current admin session, provided at app root)
│   │   ├── lib/                  api.ts (shared axios instance, withCredentials: true for Sanctum)
│   │   ├── types/                league.ts, season.ts, team.ts, game.ts, stat.ts
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── tests/
│   ├── index.html
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   ├── tsconfig.json
│   └── package.json
│
├── docs/
│   ├── tech-stack.md
│   ├── mvp-scope.md
│   ├── folder-structure.md
│   ├── naming-conventions.md
│   ├── architecture-rules.md
│   ├── data-model.md
│   ├── api-design.md
│   ├── git-rules.md
│   ├── RULES.md
│   └── activity-log.md
│
├── CLAUDE.md
└── .gitignore
```
