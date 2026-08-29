# Task Intake and Tracking

Durable, multi-session work needs a file a *different* session or a different agent can pick up with no conversation history — not a plan that only lives in one transcript. This section assumes a markdown-file-based store (a docs vault, a plain `docs/plans/` folder in-repo); adapt the status-field mechanics to whatever the project actually uses if it's a ticket tracker instead. If the store is an Obsidian vault, see `obsidian-backend.md` for the concrete commands.

## Flat plan vs. Story

**Flat plan (the default):** a single file for a self-contained, single-workstream task — a new feature, a fix/improvement, or an architecture decision worth recording. Generic templates: `assets/templates/New Feature.md`, `assets/templates/Improvement or Fix.md`, `assets/templates/Architecture Decision.md`.

**Story (the escalation path):** use instead of a flat plan when the work needs real design, spans more than one repo/service, or splits naturally into parallel workstreams. A Story is a folder — `Story.md` (overview + subtask table), `Tech Design.md` (contracts + implementation notes), an optional `ADR.md`, and one `Subtask - <Name>.md` per parallel workstream. Templates: `assets/templates/Story.md`, `assets/templates/Story Tech Design.md`, `assets/templates/Story Subtask.md`.

**Why one file per subtask.** It's what lets N sub-agents work concurrently without stepping on each other's edits — each subtask file is a single agent's exclusive write surface. `Story.md`'s subtask table stays the one place to see the whole story's state at a glance, kept in sync with each subtask file's own status.

**Contract-first is what unblocks parallelism.** `Tech Design.md`'s Contracts section should pin down each new/changed interface (endpoint, function signature, event shape — whatever the boundary is) specifically enough that one workstream can build against it and another can implement it, independently and before either side is done. If a contract needs to change mid-flight, edit it in `Tech Design.md` (single source of truth) and flag every subtask referencing it — don't let each side re-derive it independently. *(Ontology engineering in miniature — a shared, explicit schema every workstream reasons against instead of each re-deriving its own.)*

## Write it directly, don't draft-then-ask

When a request names or clearly maps to one of these artifact shapes — "research X and write a plan," "create a story for Y," "document this as an architecture decision" — write the actual content directly into the plan store, in the same turn, using the matching template. Don't draft the full deliverable in your chat response and ask afterward whether to formalize it into a durable file — that inverts where the durable copy is supposed to live, produces a chat wall of text that's one summarization pass away from being lost, and leaves the store that's supposed to be authoritative empty in the meantime. Your reply in that case should be short: what was created, where (a path/link), and a one-paragraph summary — not the deliverable's full content restated in chat.

This applies to your own output, not just a sub-agent's synthesis, and it applies to a plan produced via a host tool's own built-in plan-approval flow too — if that plan is worth resuming later, its durable copy belongs in the plan store, not only in whatever ephemeral, tool-internal location the approval flow defaults to. Write it to the store in the same turn the plan is approved, before or alongside starting implementation.

Reserve "ask before creating a new file" for genuinely ambiguous cases — unclear which template fits, or unclear whether something is Story-sized vs. flat-plan-sized. A named or obviously-matching artifact type isn't ambiguous.

## Working a Story as a managing agent

1. Read `Story.md`'s subtask table for current status and each subtask's declared dependencies.
2. Determine the next ready wave from that table's *current* state, not from a dependency list memorized at the start of the session — a subtask is ready the moment the table shows its blockers as `done`, whether that happened just now or in an earlier session. Don't hold the schedule in your own head; re-read the table each time.
3. For each subtask that's ready and still open/in-progress, spawn one sub-agent per subtask **in parallel** (a single batch of calls, not sequential). Brief each with its subtask file's path, the relevant Contracts section, the target repo/service, and the instruction to flip *only its own* subtask file's status to in-progress immediately on starting and update its Implementation Notes as it goes — not just at the end.
4. As each sub-agent reports back — not only once the whole batch is done — apply "verify, don't just relay" (`subagent-lifecycle.md`) before flipping its subtask to done: check the actual diff, not just the summary.
5. Sync `Story.md`'s subtask table right after each verified update — don't batch the sync until every subtask finishes. Anyone opening `Story.md` mid-session should see current reality, and the next wave (step 2) reads it fresh.

(Steps 1–2 are the same stigmergic-coordination idea as real-time tracking above, applied to sequencing itself: wave readiness comes from re-reading the shared file's current state rather than a schedule root holds in its head — cellular automata.)

## Status lifecycle

`open` → `in_progress`/`blocked` while worked → `done`/`cancelled` when finished (add a `completed: <date>` field at that point). Applies to both flat plans and every file within a Story. A small fixed vocabulary rather than free-text status is itself an ontology-engineering move — anyone reading a subtask file (a person, root, a resumed session) gets an unambiguous, machine-checkable state instead of parsing prose.

## Real-time task tracking

Track status as work happens, not after the fact. Two scopes:

- **In-session** (your harness's own task-list/progress-tracking mechanism, if it has one — e.g. Claude Code's `TaskCreate`/`TaskUpdate` tools): the current conversation's own step-by-step progress, visible to the user live. Mark a step complete the moment it's done — don't batch updates to the end of a turn or session.
- **Cross-session** (the plan/Story file itself): durable state a different session or agent needs to reconstruct where things stand with no conversation history. Same rule: flip a subtask to in-progress when a sub-agent starts it and to done as soon as its work is verified — as part of that sub-agent's own brief, not as a batch sync you perform once at the end.

The failure mode this avoids: a plan file that only reflects reality once, at the very end. That's useless to anyone checking in mid-flight, and it's one dropped or interrupted session away from silently losing track of work that was actually finished and verified.

This is cellular-automata-style stigmergic coordination, not hub-and-spoke chat: sub-agents write status into the shared Story/plan file directly, and anyone reading it — root, a resumed session, another sub-agent — gets current state from that shared surface instead of a relayed report.

## Pause and resume

Because state lives in the plan/Story files themselves (each file's status field plus its Implementation Notes log), any session — resumed hours or days later, or a different agent entirely — reconstructs where things stand by reading the Story plus the relevant subtask files. No dependency on conversation history.

## Archiving

Once a flat plan (or every subtask and the Story itself) is done/cancelled, add a `completed: <date>` field and move the file (or the whole Story folder) into an archive location — background reference only, not current behavior. Keep archived work separate from active plans so a fresh read of the active folder reflects only what's actually in flight.

**Archiving is not the same as documenting — do both before considering the work finished.** Moving a folder to an archive location makes the *build history* (subtask-by-subtask decisions, evidence gathered along the way, bugs found in passing) inert — nobody browses the archive to learn how something currently works, and any link into the pre-archive path silently goes stale the moment the folder moves. A completed Story/plan being archived means the durable, current-state knowledge from it now needs to live somewhere a future session will actually look — the project's general reference docs, not the archived folder itself. Concretely, as part of the same archiving pass, not a follow-up:

1. Decide whether the completed work needs its own reference note, or just a section/paragraph added to an existing one — judge by whether it has enough of its own data model/lifecycle/surface area to be genuinely worth its own note.
2. Extend every reference doc whose scope the work actually touched — check the plan/Story's own scope/affected-files section and match each touched area to whatever documents it, rather than guessing which apply.
3. Carry forward any real, still-open gaps (an explicitly deferred non-goal, a known unimplemented edge case, a bug found but not fixed) into the relevant note's own known-gaps section — don't let a Story's Open Questions be the only place these live, since that section stops being read once the folder is archived.
4. Fix every link that pointed at the plan/Story by its pre-archive path — search for it before moving anything, and either repoint each hit to the new reference-doc location (preferred) or update it to the archive path if the link is genuinely about build history rather than current behavior. A dangling link is usually silent — nothing errors on it — and won't surface again until someone happens to follow it.

Treat "archived" and "documented" as two separate, both-required checkboxes — a Story that's been archived but never folded into the general docs is a real state this kind of system can land in, not a hypothetical failure mode.
