# Obsidian as the Docs/Task-Intake Backend

When a project keeps its reference docs and task-intake plans in an Obsidian vault (rather than a plain in-repo `docs/` folder), Obsidian is the concrete backend for `task-intake-and-tracking.md` and `continuous-documentation.md` — read those two first for the pattern. For the actual CLI commands (reading, writing, moving, health-checking a vault) and for how the vault should be laid out in the first place, see the sibling skills `obsidian-vault-maintenance` and `obsidian-vault-structure` — this file only covers how this workflow's patterns map onto those.

Not every project uses Obsidian — if a project's docs live in plain in-repo markdown instead, skip this file; `task-intake-and-tracking.md` and `continuous-documentation.md` already describe the pattern in backend-agnostic terms.

## How the generic patterns map onto a vault

- **Flat plans and Stories** (`task-intake-and-tracking.md`): live as notes/folders in whatever the project's plans path is (check the project's own agent-instructions file for it; `obsidian-vault-structure` describes where that path should sit within the vault as a whole). A Story's `status` field is typically a dropdown property via the Metadata Menu plugin rather than free text — same lifecycle (`open`/`in_progress`/`blocked`/`done`/`cancelled`), just enforced by the vault's schema instead of convention alone. Flip it with `property:set` (see `obsidian-vault-maintenance`), per the real-time-tracking rule in `task-intake-and-tracking.md` — the moment a sub-agent starts/finishes its subtask, not batched to the end.
- **Continuous documentation** (`continuous-documentation.md`): "the project's reference docs" means the vault's reference notes for that project/subsystem — update the specific note directly rather than a general project index.
- **Archiving**: relocate a completed plan or Story folder to its mirrored `Archive/` path (`obsidian-vault-maintenance`'s `move` command) — a real file move, not a separate archive API.
- **Cross-links**: use wikilinks (`[[Note Name]]`) between notes the way the templates in `assets/templates/` already do (`Story.md` linking to `Tech Design.md`, a subtask linking to its parent Story) — that's what makes backlink lookups useful for reconstructing state later, and what the vault-health checks in `obsidian-vault-maintenance` rely on to catch drift.
