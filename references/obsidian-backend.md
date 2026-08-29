# Obsidian as the Docs/Task-Intake Backend

When a project keeps its reference docs and task-intake plans in an Obsidian vault (rather than a plain in-repo `docs/` folder), Obsidian's CLI is the concrete backend for `task-intake-and-tracking.md` and `continuous-documentation.md` — read those two first for the pattern; this file is the mechanics of applying it through Obsidian specifically. The three work as one package: Obsidian is the storage/link layer, the other two files are the workflow that uses it.

## Prefer the CLI over raw filesystem access

The vault is Obsidian-managed, and an `obsidian` CLI is commonly installed alongside it (check `which obsidian`; if absent, this file doesn't apply — fall back to direct filesystem reads/writes on the vault path, which still work since it's just files on disk). Prefer the CLI over `Read`/`Glob`/raw `cat` on vault paths when it's available — it goes through the running Obsidian app, so it respects the vault's actual link graph, properties (including dropdown/typed properties from plugins like Metadata Menu), and tags, and can search/write, not just read.

Requires the Obsidian app to be running with the target vault open (`obsidian vaults` lists known vaults). Starting it yourself (`open -a Obsidian` on macOS, bundle id `md.obsidian`) is safe and not a risky/destructive action — do it without a confirmation round-trip if the app isn't already open, then retry. If the vault still doesn't appear after a few seconds, or you don't know the vault name, ask the user rather than guessing.

Every command needs `vault=<name>`. Paths are vault-relative (`Folder/Note.md`, no leading slash; `.md` optional for `file=` lookups, required for `path=`).

## Common commands

| Need | Command |
|---|---|
| Read a note | `obsidian vault=<name> read path="Folder/Note.md"` |
| Full-text search (optionally scoped to a folder) | `obsidian vault=<name> search query="term" path="Folder" limit=5` |
| Search with matching line context | `obsidian vault=<name> search:context query="term" path="Folder"` |
| List files in a folder | `obsidian vault=<name> files folder="Folder"` |
| List headings in a note | `obsidian vault=<name> outline path="Folder/Note.md"` |
| Backlinks / outgoing links | `obsidian vault=<name> backlinks path="..."` / `links path="..."` |
| Append to a note (e.g. logging a decision, updating Implementation Notes) | `obsidian vault=<name> append path="..." content="..."` |
| Prepend to a note | `obsidian vault=<name> prepend path="..." content="..."` |
| Create or overwrite a note | `obsidian vault=<name> create path="..." content="..." overwrite` (omit `overwrite` to fail instead of clobbering an existing file) |
| **Flip a status property** (e.g. a plan/Story/subtask from `open` to `in_progress`/`done`) | `obsidian vault=<name> property:set path="..." name="status" value="in_progress"` — use this, not `create ... overwrite`, for a status change: it edits just the one property in place instead of rewriting (and risking dropping/corrupting) the whole note |
| Read a single property value | `obsidian vault=<name> property:read path="..." name="status"` |
| Remove a property | `obsidian vault=<name> property:remove path="..." name="..."` |
| Toggle/set a markdown checkbox task (e.g. a Story Subtask's Done Checklist) | `obsidian vault=<name> task path="..." line=<n> done` (or `todo`/`toggle`) |
| Move/rename a file (archiving a completed plan/Story) | `obsidian vault=<name> move path="Folder/Note.md" to="Archive/Note.md"` or `rename ... name="New Name"` |
| Delete a file | `obsidian vault=<name> delete path="..."` (add `permanent` to skip trash) |

Use `\n`/`\t` inside a `content=` value for newlines/tabs — the CLI doesn't accept a file as input, so keep `create`/`append` payloads to what's comfortable to build inline; for long-form content, either build it up with a few `append` calls or fall back to a direct filesystem `Write`/`Edit` on the vault's absolute path (still valid, you just lose the link-graph-aware guardrails for that one write).

Fall back to direct filesystem access only if the CLI still errors after trying `open -a Obsidian` (app slow to start, or genuinely unavailable) — `Read`/`Write` on the absolute vault path still works.

## How the generic patterns map onto a vault

- **Flat plans and Stories** (`task-intake-and-tracking.md`): live as notes/folders in whatever the project's plans path is (check the project's own agent-instructions file for it). A Story's `status` field is typically a dropdown property via the Metadata Menu plugin rather than free text — same lifecycle (`open`/`in_progress`/`blocked`/`done`/`cancelled`), just enforced by the vault's schema instead of convention alone. Flip it with `property:set`, per the real-time-tracking rule in `task-intake-and-tracking.md` (the moment a sub-agent starts/finishes its subtask, not batched to the end).
- **Continuous documentation** (`continuous-documentation.md`): "the project's reference docs" means the vault's reference notes for that project/subsystem — update the specific note via `append`/`create overwrite`, not a general project index.
- **Archiving**: use `move` to relocate a completed plan or Story folder to its archive path — a real file move, not a separate archive API.
- **Cross-links**: use wikilinks (`[[Note Name]]`) between notes the way the templates in `assets/templates/` already do (`Story.md` linking to `Tech Design.md`, a subtask linking to its parent Story) — that's what makes `backlinks`/`links` useful for reconstructing state later.

Not every project uses Obsidian — if a project's docs live in plain in-repo markdown instead, skip this file; `task-intake-and-tracking.md` and `continuous-documentation.md` already describe the pattern in backend-agnostic terms.
