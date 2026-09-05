# Peer-to-Peer Agent Communication

Everything in `subagent-lifecycle.md` and `multi-agent-hierarchy.md` is hub-and-spoke: sub-agents report to whichever agent spawned them, never to each other, and the root/lead stays the only integrator. That stays the default. This file covers the narrower case where two or more agents genuinely need to talk *to each other*, directly, across multiple rounds — a distinct capability (e.g. Claude Code's Agent Teams: named teammates, each with its own context window, messaging each other through a mailbox, plus a shared task list), not a paradigm shift in how you orchestrate.

## It doesn't reduce context bloat — it costs more, on purpose

Don't reach for this expecting token savings. Every peer agent is a full separate context window; total spend scales with how many are talking, well above a single session's cost. The actual payoff is narrower than "less context": it keeps any *one* agent's own context clean via isolation — the same benefit plain delegation already gives you — while letting two specialized agents resolve something directly across several exchanges, instead of every exchange being forced through a summarize-and-relay hop via the root. That second part is what a hub-and-spoke brief-and-report can't do: a relay degrades information each hop (the "telephone game" problem), and repeated relaying for what's really one ongoing conversation between two agents costs more round-trips, not fewer.

If a one-shot handoff would do — one agent's output feeds another's input, no back-and-forth needed — that's still an ordinary brief-and-report. Peer messaging is for when the back-and-forth itself is the point.

## When it earns its keep

- **Competing-hypothesis debugging.** Root cause is unclear; spawn several investigators, each committed to a different theory, and have them actively try to disprove each other's — not just report independently. Sequential or isolated investigation anchors on whichever theory got explored first; live cross-challenge surfaces the one that actually survives scrutiny.
- **Live parallel review.** Independent reviewers (security/performance/tests, say) benefit from seeing and challenging each other's in-flight findings, not only a final synthesis you produce after all of them finish separately.
- **Cross-layer coordination with real renegotiation.** Frontend/backend/test teammates whose interface needs several actual rounds of adjustment — not a contract you can pin once up front. If the interface *can* be pinned up front and built against, that's still `task-intake-and-tracking.md`'s contract-first pattern with a single hub-mediated handoff — reach for peer messaging only when it genuinely can't be pinned in advance.

## When it actively hurts

Some of this skill's existing patterns work *because* agents don't talk to each other — don't retrofit peer messaging onto them:

- The high-risk-change reviewer (`multi-agent-hierarchy.md`) is deliberately handed the requirement and the diff but not the implementer's reasoning, so its judgment isn't anchored to the implementer's own blind spots. Letting it message the implementer directly defeats that.
- The ambiguous-design-decision pattern (independent candidate plans + a judge) works the same way — candidates are independent precisely so they don't converge on each other's early assumptions.
- Routine delegation with a stable contract, disjoint file ownership, or a simple one-shot handoff: peer messaging adds coordination overhead and cost for no benefit there.

## Mechanics (check what's actually available before relying on it — this is experimental and evolves)

- **A full team**: the root becomes the team lead; named teammates each get their own context, message each other and the lead directly, and share a task list. The lead still never abdicates integration or final verification just because teammates can resolve things among themselves — same role as always (`multi-agent-hierarchy.md`'s role hierarchy applies unchanged).
- **Lighter weight**: an ordinary named sub-agent can often be messaged again later without standing up a full team — reach for that before a full team when what's needed is occasional follow-up, not sustained multi-round exchange.
- **Cross-session**: independent sessions you run yourself can pass messages to each other — for genuinely separate workstreams (e.g. per-repo leads checking in), not a single coordinated team.
- No nested teams/peer groups: a peer agent doesn't stand up its own peer group on its own initiative, the same nesting discipline as `subagent-lifecycle.md`'s one-level default.

## Failure modes to guard against

Inter-agent misalignment is a real, named failure category (not just a hub-and-spoke one): agents can talk past each other, fail to actually update on what the other just said, or converge on a false consensus that neither one would reach alone. Apply the same discipline as everywhere else in this skill:

- **Verify, don't just relay** — the root/lead checks actual diffs and outputs before trusting a reported "we agree," the same as any other sub-agent claim.
- **Shared-worktree safety still applies** (`multi-agent-hierarchy.md`) — peer messaging doesn't replace disjoint file ownership; two teammates can still clobber each other's edits.
- **Size the team to the task.** Coordination overhead grows with headcount; a few focused peers that actually need to talk beats a larger group that mostly doesn't.
