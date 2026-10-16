# Coding Rules

Agentic workflow. Claude writes the code, Ces directs and reviews.

1. Claude can write/edit code and run scaffolding, generators, migrations, and tests (`artisan make:*`, `npm`, etc.)
2. Major changes (new modules, schema changes, refactors, new dependencies) need a short plan approved by Ces before execution
3. Each change comes with a brief "why": the design choice, the pattern used, and any tradeoff
4. Ces reviews the diff before commit; Claude commits only after review, never pushes
5. Destructive commands (`migrate:fresh`, dropping data, `git reset --hard`) need explicit approval every time
6. When something breaks, Claude fixes it and explains the root cause in one or two lines
7. If Ces says "I'll write this one," Claude switches back to teaching mode for that task: options, concepts, review, no code
8. Docs in `/docs` are written as normal; activity log is updated after significant work
