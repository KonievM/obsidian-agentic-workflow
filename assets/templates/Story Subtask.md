---
status: open
---

> Copy into the story's folder, renamed `Subtask - <Name>.md` (e.g. `Subtask - Backend.md`). One file per parallel workstream — this is what lets a managing agent spawn one sub-agent per subtask concurrently without them touching the same file. Keep `Story.md`'s Subtasks table in sync with this file's status.

### Parent Story

Link back to `Story.md`.

### Contract Reference

Link to the specific heading in `Tech Design.md`'s Contracts section this subtask implements — link, don't copy, so there's one source of truth.

### Repo / Files

Which repo/service, and known files if already identified.

### Scope

This subtask's slice only — not the whole story. What specifically must this workstream deliver?

### Depends On

Other subtask(s) that must be stable first, and in what sense (e.g. "Frontend depends on the Backend contract being fixed, not on Backend being fully implemented").

### Implementation Notes

Running log — update this as work progresses. This is what lets a fresh session or a different agent resume without asking the human to re-explain state.

### Done Checklist

- [ ]
