# Task Intake and Tracking

Durable, multi-session work needs a file a *different* session or a different agent can pick up with no conversation history — not a plan that only lives in one transcript. This section assumes a markdown-file-based store (a docs vault, a plain `docs/plans/` folder in-repo); adapt the status-field mechanics to whatever the project actually uses if it's a ticket tracker instead. If the store is an Obsidian vault, see `obsidian-backend.md` for the concrete commands.

## Flat plan vs. Story

**Flat plan (the default):** a single file for a self-contained, single-workstream task — a new feature, a fix/improvement, or an architecture decision worth recording. Generic templates: `assets/templates/New Feature.md`, `assets/templates/Improvement or Fix.md`, `assets/templates/Architecture Decision.md`.

**Story (the escalation path):** use instead of a flat plan when the work needs real design, spans more than one repo/service, or splits naturally into parallel workstreams. A Story is a folder — `Story.md` (overview + subtask table), `Tech Design.md` (contracts + implementation notes), an optional `ADR.md`, and one `Subtask - <Name>.md` per parallel workstream. Templates: `assets/templates/Story.md`, `assets/templates/Story Tech Design.md`, `assets/templates/Story Subtask.md`.

**Why one file per subtask.** It's what lets N sub-agents work concurrently without stepping on each other's edits — each subtask file is a single agent's exclusive write surface. `Story.md`'s subtask table stays the one place to see the whole story's state at a glance, kept in sync with each subtask file's own status.

**Contract-first is what unblocks parallelism.** `Tech Design.md`'s Contracts section should pin down each new/changed interface (endpoint, function signature, event shape — whatever the boundary is) specifically enough that one workstream can build against it and another can implement it, independently and before either side is done. If a contract needs to change mid-flight, edit it in `Tech Design.md` (single source of truth) and flag every subtask referencing it — don't let each side re-derive it independently.

## Working a Story as a managing agent

1. Read `Story.md`'s subtask table for current status.
2. For each subtask still open/in-progress, spawn one sub-agent per subtask **in parallel** (a single batch of calls, not sequential). Brief each with its subtask file's path, the relevant Contracts section, the target repo/service, and the instruction to flip *only its own* subtask file's status to in-progress immediately on starting and update its Implementation Notes as it goes — not just at the end.
3. As each sub-agent reports back — not only once the whole batch is done — apply "verify, don't just relay" (`subagent-lifecycle.md`) before flipping its subtask to done: check the actual diff, not just the summary.
4. Sync `Story.md`'s subtask table right after each verified update — don't batch the sync until every subtask finishes. Anyone opening `Story.md` mid-session should see current reality.

## Status lifecycle

`open` → `in_progress`/`blocked` while worked → `done`/`cancelled` when finished (add a `completed: <date>` field at that point). Applies to both flat plans and every file within a Story.

## Real-time task tracking

Track status as work happens, not after the fact. Two scopes:

- **In-session** (your harness's own task-list/progress-tracking mechanism, if it has one — e.g. Claude Code's `TaskCreate`/`TaskUpdate` tools): the current conversation's own step-by-step progress, visible to the user live. Mark a step complete the moment it's done — don't batch updates to the end of a turn or session.
- **Cross-session** (the plan/Story file itself): durable state a different session or agent needs to reconstruct where things stand with no conversation history. Same rule: flip a subtask to in-progress when a sub-agent starts it and to done as soon as its work is verified — as part of that sub-agent's own brief, not as a batch sync you perform once at the end.

The failure mode this avoids: a plan file that only reflects reality once, at the very end. That's useless to anyone checking in mid-flight, and it's one dropped or interrupted session away from silently losing track of work that was actually finished and verified.

## Pause and resume

Because state lives in the plan/Story files themselves (each file's status field plus its Implementation Notes log), any session — resumed hours or days later, or a different agent entirely — reconstructs where things stand by reading the Story plus the relevant subtask files. No dependency on conversation history.

## Archiving

Once a flat plan (or every subtask and the Story itself) is done/cancelled, add a `completed: <date>` field and move the file (or the whole Story folder) into an archive location — background reference only, not current behavior. Keep archived work separate from active plans so a fresh read of the active folder reflects only what's actually in flight.
