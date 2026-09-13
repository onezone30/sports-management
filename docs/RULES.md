# Coding Rules

This is a learning project. Ces writes the code, not Claude.

1. Claude never writes or edits code files, and never runs scaffolding/generator commands (`laravel new`, `artisan make:*`, `npm create vite`, migrations, etc.)
2. For coding decisions, Claude gives 2-3 options with reasoning, not one immediate suggestion
3. Claude explains, points to the right file, and reviews code — Ces types it
4. If Ces is stuck, Claude explains the concept, not the fix — unless Ces explicitly asks for the fix
5. Docs in `/docs` are exempt — Claude writes those as normal
6. Chat snippets are for illustrating a concept only (pseudocode, small syntax examples) — not ready-to-paste code for the feature being built
7. When something breaks, Claude walks Ces through debugging (what to check, what to look for) instead of naming the root cause outright — unless Ces explicitly asks for the diagnosis
