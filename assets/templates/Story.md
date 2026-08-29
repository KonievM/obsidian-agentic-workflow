---
status: open
---

> Use a Story instead of a flat template when the task needs real design work, spans more than one repo/service, or benefits from splitting into parallel workstreams. For a smaller, single-workstream change, use the flat templates instead.
>
> Copy this file into a new folder `<plans-location>/Stories/<story-slug>/`, renamed to `Story.md`. Alongside it, add `Tech Design.md`, one `Subtask - <Name>.md` per parallel workstream, and optionally `ADR.md` if a real decision needs recording. Same status lifecycle as other templates; on completion, move the whole `Stories/<story-slug>/` folder to an archive location.

### Goal

What are we trying to achieve, and why now?

### Scope

- In scope:
- Out of scope:

### Affected repos / files

Which repos/services are involved. Reference known file paths if you already know them.

### Subtasks

One row per parallel workstream. Keep the Status column in sync with each subtask file's own status — this table is the single place to see the whole story's state at a glance.

| Subtask file | Repo/service | Depends on | Status |
|---|---|---|---|
| Subtask - Backend | — | — | open |
| Subtask - Frontend | — | Backend contract stable | open |

### Design docs

- Tech Design — contracts and implementation notes
- ADR — architecture decision (if applicable; omit this line if there isn't one)

### Open Questions

Anything ambiguous that needs a decision before/during implementation.

### Acceptance Criteria

How do we know the whole story is done? (Per-subtask acceptance criteria live in each subtask file — this is the story-level bar.)
