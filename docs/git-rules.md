# Git Rules

Builds on the Version Control section already in the project's root `CLAUDE.md` (commit after significant changes, keep commits focused/atomic, no auto-push, don't auto-commit activity logs/docs) — this doc covers what that one doesn't.

## Commit messages

- Free-form, descriptive sentences — no required prefix (no Conventional Commits format). A clear sentence describing the change is enough, matching `CLAUDE.md`'s existing "clear messages" rule without adding a new convention on top.

## Branching

- Trunk-based: commit directly to `main`. No feature branches for this solo project — there's no team to coordinate with and no PR/CI gate waiting on them, so a branch/merge cycle would be overhead without a protection benefit.

## Who commits

- Ces commits all code changes — consistent with `RULES.md` (Ces writes the code, Ces commits it, building the commit-message-writing habit himself).
- Claude may commit doc-only changes (files under `/docs`) when explicitly asked to, per `RULES.md`'s exemption for planning/design docs. This is not the same as `CLAUDE.md`'s "don't auto-commit activity logs and docs" rule — that rule is about Claude never committing docs *unprompted*; committing when Ces explicitly asks is still fine.
