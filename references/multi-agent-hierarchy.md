# Multi-Agent Hierarchy and Shared-Worktree Safety

`subagent-lifecycle.md` covers the basics of one-off delegation. This file is for when there's genuinely more than a couple of sub-agents in flight at once — a real fan-out, not a single one-off task — where role clarity and filesystem safety start to matter on their own.

## Role hierarchy

| Role | Owns | May edit | May delegate |
|---|---|---|---|
| **Root coordinator** (you, talking to the user) | The user's actual request, decomposition, plan/status, an ownership map, integration, final verification and response | Integration points and unassigned files | Yes — creates independent workstreams and remains accountable for every result |
| **Workstream lead** | One repo, subsystem, or Story subtask with a clear deliverable | Its explicitly assigned file/domain boundary | Only when your brief explicitly permits it and a real independent child task exists |
| **Specialist** | One bounded investigation or implementation unit | Only named files/domain, preferably disjoint from other agents | No, by default |
| **Reviewer** | Independent correctness/security/test/design review | Read-only unless separately asked to prepare an isolated fix | No |

This expresses responsibility, not seniority. You (root) never abdicate integration or verification to a lead. A lead that spawns a child becomes responsible for checking that child's actual diff/output before reporting upward — the same "verify, don't just relay" rule from `subagent-lifecycle.md`, one level down.

Keep the tree shallow: root → specialist/reviewer is the default. Root → lead → specialist is reserved for a genuinely multi-part workstream, not a default. Never create a layer of delegation merely to relay a message.

## Task routing

| Task shape | Recommended topology |
|---|---|
| One answer, grep, file read, tiny localized edit | You handle it directly; no fan-out |
| Broad research/audit across independent domains | One specialist per domain; you synthesize and dedupe |
| Localized implementation | One implementation specialist; you review the diff and run integration checks |
| Several independent implementation units in one repo | One specialist per disjoint file/domain boundary; you own shared interfaces and merge points |
| Cross-repo feature | One workstream lead per repo; you pin the contract and branch for each before dispatch |
| Story with independent subtasks | One agent per ready subtask, respecting dependency order and available concurrency; each updates only its own durable status artifact |
| High-risk change (money movement, auth, migrations, infrastructure writes) | Implementation agent plus a separate read-only reviewer; you resolve findings and validate |
| CI failure or review-comment batch | Separate diagnostics by failing job/comment cluster only when independent; a single agent owns each eventual file edit |
| Visual/browser validation | Implementation and visual-review can be separate workstreams; the reviewer records evidence and doesn't silently redesign |

Don't split by arbitrary file count. Split where inputs, outputs, and ownership can be stated independently of each other.

## Concurrency and nesting

Available concurrency is session/harness-dependent — check what's actually live before spawning more, and don't hardcode a number from a prior session.

- Parallelize independent reads, investigations, tests, repos, or genuinely disjoint implementations.
- Serialize tasks with interface or data dependencies on each other.
- A child may spawn its own child only when your brief explicitly authorizes that lead role for that specific workstream — not on the child's own initiative. Nested agents still consume the same overall concurrency budget as everything else.
- Don't override the model/reasoning tier unless the task or the user actually requires a non-default one — inherited default is correct far more often than not.
- Brief with exact paths, constraints, and expected output rather than handing over full conversation history "just in case" — that defeats the context-hygiene point of delegating in the first place.

If more independent workstreams are ready than your available concurrency supports, run them in waves — start the next wave as slots free up rather than firing everything in one batch and letting the excess queue silently.

## Shared-worktree safety

Multiple agents working the same machine/filesystem share it — Git provides no lock, so ownership has to be a convention everyone actually follows.

Before fan-out, record an ownership map: agent, repo/branch, writable paths, read-only dependencies, deliverable. Two active agents must never edit the same file or generated-output family concurrently. Shared interfaces, migrations, lockfiles, route registries, generated code, and central exports belong to one named owner or to your own integration phase — never to "whoever gets there first."

Every agent must preserve pre-existing user changes and check the current diff before editing — don't assume a clean starting state. If unexpected overlapping edits appear mid-task, stop editing that area and escalate to the owner/root rather than resolving it unilaterally. **Never reset, revert, stash, or clean another agent's (or the user's) work** — an agent that finds unexpected changes should investigate and report, not erase them to get a clean slate.

For concurrent implementations with genuinely unavoidable overlap, use separate git worktrees or serialize the work instead of trusting convention alone. Cross-repo work naturally has separate histories but still needs an explicit contract (per `task-intake-and-tracking.md`'s Tech Design contracts) so the pieces actually fit together.

## Brief contract

Every delegated task brief should state:

- objective and explicit non-goals;
- the target repo path, branch, and current Story/subtask path when applicable;
- required project docs/instructions the sub-agent needs (by reference — path, not pasted content, see below);
- writable ownership boundary, and which files/domains are read-only for this agent;
- known dirty-worktree context that must be preserved;
- what validation/evidence is expected back, including any environment requirements (e.g. must run via a specific container/venv, needs a real vs. mocked external call);
- the requested return shape: findings, files changed, checks run, risks/blockers, and any durable-doc updates made.

**Reference source documents by path instead of pasting their full rules into the brief.** Repeat inline only the one or two constraints whose omission would make the brief actually unsafe (a target branch that isn't the default, a hard file boundary, "testnet/sandbox only" for anything touching real external state) — not the whole policy document. A brief that pastes everything in duplicates content that will drift out of sync with the source the next time it's edited.

## Communication and handoff

Send follow-up context to the existing owner of a workstream rather than spawning a duplicate agent for work already in flight. Agents should report blockers early and clearly distinguish observed facts from inferences.

A completion handoff should state:

1. Outcome and scope actually completed (vs. what was asked).
2. Files changed or evidence inspected.
3. Checks run, and the exact failures/limitations if any.
4. Risks, unresolved questions, and integration assumptions made.
5. Durable plan/Story/status notes updated, if any.

You verify the actual diffs and outputs, integrate in dependency order, run the broadest proportionate final check, and only then mark the durable work complete — an agent's own summary is a claim, not proof.

## Fan-out patterns

**Read-only audit.** You define one question and partition it by subsystem. Specialists return findings with file/line evidence. A separate reviewer may challenge high-severity findings before you trust them. You deduplicate and write durable findings to wherever the project records known issues.

**Single-repo build.** You own contracts and shared registries. Specialists own disjoint modules/tests/docs. You integrate shared exports/migrations/generated files after specialists finish, then run repo-level checks.

**Cross-repo feature.** You record the contract (API/event/schema) in the Story's Tech Design first. One lead owns each repo and branch. Backend contract work precedes dependent frontend validation unless mocks are explicitly part of the agreed contract. Each repo gets its own commit and its own validation evidence — never one commit spanning repos.

**High-risk change.** One agent implements; a separate reviewer receives the requirement and the diff, but not the implementer's reasoning, so it isn't anchored to the implementer's own blind spots. The reviewer stays read-only and focuses on invariants, failure recovery, idempotency, authorization, and test gaps. You adjudicate rather than forwarding either side's conclusion uncritically.

## Anti-patterns

- Spawning agents without a disjoint deliverable or ownership boundary.
- Letting multiple agents edit the same working-tree area concurrently and hoping it works out.
- Asking every agent to re-read the entire project's docs instead of the specific paths it needs.
- Pasting shared policy into every brief until the copies drift out of sync with the source.
- Treating an agent's summary as proof that its edits or checks actually succeeded.
- Letting a leaf agent expand scope, commit, push, or mutate an external system without explicit authority to do so.
- Marking a Story/subtask done before integration and verification actually happened.
