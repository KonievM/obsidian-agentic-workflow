---
status: open
---

> Copy alongside `Story.md` as `Tech Design.md`. This is the single source of truth for interface contracts — every subtask that implements or consumes a contract should link to a specific heading here rather than re-describing it.

### Overview

One paragraph: what's being built, at the design level.

### Contracts

One subsection per new/changed interface (endpoint, function signature, event/message shape). Pin down enough detail that one workstream can build against it while another implements it, independently, before either is done.

#### <Interface name>

- Shape:
- Callers / consumers:
- Error cases:

### Implementation Notes

Running log of design decisions made during implementation that change or clarify what's above — update as work progresses, not just at the start.
