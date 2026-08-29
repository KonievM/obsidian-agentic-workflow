# Continuous Documentation

After completing any non-trivial task, check whether anything learned during it is worth writing back to the project's reference docs (architecture notes, conventions, an agent-instructions file like `CLAUDE.md`/`AGENTS.md`).

**Worth persisting:** undocumented structure you had to discover by reading code, a module's non-obvious purpose, a key convention that isn't derivable by skimming, an architectural decision future work should know about, a gotcha that cost real exploration time to find.

**Not worth persisting:** anything already derivable by reading the code, anything that only matters to this one conversation, anything the repo's own history/commits already record.

**Where it goes:**

- A specific reference doc (architecture, schema, conventions, per-subsystem notes) — update it directly if you discovered something that belongs there. If the project's reference docs live in an Obsidian vault, see `obsidian-backend.md` for the concrete commands.
- An agent-instructions file (`CLAUDE.md`, `AGENTS.md`) — only for genuine agent rules/constraints ("always X when touching Y"), not documentation. Documentation belongs in the reference docs, not duplicated into the instructions file.

This is distinct from an agent's own private/session memory (a personal preferences or past-corrections store, if your harness has one) — that's about how *you* should behave across sessions with this user; this is about what the *project* needs recorded so any future agent or teammate doesn't have to re-derive it. Don't conflate the two stores.

Only add something if it's genuinely non-obvious and would save real exploration time in a future session. When in doubt, don't add it — a reference doc that accumulates marginal notes becomes as hard to trust as no doc at all.
