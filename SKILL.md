---
name: obsidian-agentic-workflow
description: Operating procedure for an orchestrating coding agent (Claude Code or Codex) managing multi-step engineering work — when and how to delegate to sub-agents, which model/reasoning tier to use for a task, how to delegate to the other tool (Claude and Codex, whichever one isn't the current agent) as an independent second opinion or a parallel workstream, how to track and hand off durable multi-session work (flat plans and multi-workstream Stories, including ones stored in an Obsidian vault), when to propose a deterministic multi-agent workflow, when unit tests and API checks aren't enough to call a feature done, and how to keep project reference docs current. Use whenever a task is large enough to delegate, spans more than one session, needs a durable plan/story file to resume from, involves choosing a model tier, touches an Obsidian vault used for docs/plans, is about to be declared done based on tests/review alone, or should update the project's reference docs afterward.
---

# Agentic Workflow

Operating procedure for a managing agent — the session talking to the user — running engineering work of real size. Applies equally whether the managing agent is Claude Code or Codex; where the two diverge (tool names, MCP wiring), the relevant reference file says so explicitly.

## Philosophy

Orchestrate, not execute. Decompose the task, decide what to delegate, brief sub-agents, read and verify their results, integrate. Once a piece of work is delegable, doing it yourself instead is a cost: it burns your own context on detail nobody downstream needs (raw exploration, draft iterations, file dumps), and it serializes work that could run in parallel.

Don't over-apply this — a single grep, a single file read, a two-line fix doesn't need a sub-agent; the delegation overhead (spin-up, briefing, reading the result back) costs more than doing it inline. Delegate when a task is well-scoped *and* either parallelizable, would pollute your own context, or is simply a large enough self-contained chunk on its own.

**Resist mid-task scope creep back into execution.** The common failure isn't over-delegating, it's the reverse: a new constraint or fix-up lands mid-task and it feels like less overhead to just do it inline than to re-brief. That's exactly how a manager's context re-fills with the exploration/drafts it was supposed to stay clear of. Treat new instructions the same as the original ask — delegate them too. See `references/subagent-lifecycle.md`.

## Where these patterns come from

Several patterns below aren't ad hoc — each traces to a named paradigm from ongoing R&D research into agent design approaches: shared state that sub-agents read/write directly instead of relayed chat (**cellular automata** — stigmergic coordination), branching into multiple candidates and verifying/judging instead of trusting a single pass (**computable search**), and shared typed contracts/vocabularies instead of free-form prose between agents (**ontology engineering**). Each reference file names the paradigm inline at the specific point it applies; the underlying research notes are kept in a private vault outside this repo.

## Public-repo hygiene

This skill's own files are versioned in a public GitHub repo. When editing anything here:

- Never hardcode an absolute personal path (home directory, personal vault/notes location, machine-specific path) into skill content. Reference the *idea* or a relative/generic path instead of where it happens to live on one machine.
- Before committing, check the diff itself for `/Users/<name>`, `/home/<name>`, or similar — don't rely on remembering not to add one.
- Before pushing, check that no commit trailer or footer a harness auto-appends (e.g. a session-URL trailer) is going out with it — strip it if the target repo is public.
- If something personal does land in a public repo's history, fixing only the latest commit isn't enough — it needs a history rewrite (e.g. `git filter-repo`) and a force-push, not just a follow-up commit.

## Quick checklist

1. Can this be answered in 1–2 direct tool calls? → just do it.
2. Is it well-scoped and parallelizable, would it bloat your context, or is it just a big self-contained chunk? → delegate. Pick a tier from `references/model-tiers.md`. Size alone is a reason to delegate.
3. New instructions landing mid-task? → delegate those too (re-brief or spawn) — don't fold them into your own direct execution.
4. Is the work complex, cross-cutting, or parallelizable enough to need a multi-file Story instead of a flat plan? → set one up per `references/task-intake-and-tracking.md` before spawning sub-agents against it.
5. Is a single sub-agent's task big enough to span multiple milestones? → checkpoint and rotate per `references/subagent-lifecycle.md`, don't let one sub-agent run indefinitely.
6. More than a couple of sub-agents genuinely in flight at once? → cap concurrency and wave the rest (`references/subagent-lifecycle.md`), and use `references/multi-agent-hierarchy.md` for role clarity and shared-worktree safety once it's a real fan-out, not a one-off.
7. Brief every sub-agent with full context — it starts with none of this conversation.
8. When it returns: verify, don't just relay. Check the actual diff/output, not only the summary. Sync any durable plan status immediately — don't batch it to the end (`references/task-intake-and-tracking.md`).
9. Durable output goes where it belongs — a plan/Story file, a repo commit, a project reference doc — never left stranded only in a transcript. If a request names or clearly matches one of these artifact shapes, write it there directly, in the same turn — don't draft it in chat and ask afterward (`references/task-intake-and-tracking.md`).
10. Does the task look like 3+ dependent/parallel agent calls (a big audit, migration, or review)? → propose a deterministic multi-agent workflow instead of hand-chaining sub-agent calls yourself, per `references/workflows-and-pacing.md` — but still wait for explicit user opt-in before running one.
11. About to launch a large fan-out while another is still active, or seeing repeated rate/error signals? → pace it per `references/workflows-and-pacing.md`.
12. Want an independent second opinion, or a genuinely separate parallel workstream? → check whether the other tool (Claude ↔ Codex) is reachable and off cooldown, per `references/cross-tool-delegation.md`.
13. Do two or more agents need genuinely repeated back-and-forth with each other — debate, live cross-challenge, multi-round interface renegotiation — not just a one-shot brief-and-report? → direct peer-to-peer messaging (e.g. Claude Code's Agent Teams) can be worth its extra cost; see `references/peer-communication.md`. It costs *more* tokens than hub-and-spoke, not less — don't reach for it expecting context savings, and don't use it where a pattern below deliberately keeps agents isolated from each other.
14. Finishing a non-trivial task? → check whether anything learned is worth persisting to the project's reference docs, per `references/continuous-documentation.md`.
15. About to declare a feature done based on unit tests / code review / a direct API call alone? → if it touches a client/server seam, a real external integration, or combinatorial UI states, that's not enough — see `references/live-validation.md`.

## Reference files

- `references/model-tiers.md` — cheapest-that-works tier framework, task-type → tier table, current model mappings for Claude and Codex.
- `references/subagent-lifecycle.md` — when to spawn, concurrency cap, briefing, nesting limit, rotation for long sub-agent runs, who verifies, where results live.
- `references/multi-agent-hierarchy.md` — for a real fan-out (not just one-off delegation): role hierarchy, task-routing table, shared-worktree ownership/safety, a brief-contract checklist, structured handoff format, fan-out-pattern recipes, and anti-patterns.
- `references/cross-tool-delegation.md` — using the other tool (Claude ↔ Codex) as an independent-model delegate: availability check, when to reach for it, usage-limit cooldown handling.
- `references/peer-communication.md` — direct agent-to-agent messaging (e.g. Agent Teams) as an escalation from hub-and-spoke: what it actually buys (not context savings — it costs more), when it earns its keep, when it breaks patterns that deliberately keep agents isolated, and the failure modes to guard against.
- `references/task-intake-and-tracking.md` — flat plan vs. multi-workstream Story, status lifecycle, contract-first parallelism, real-time tracking, archiving. Generic starter templates for both live in `assets/templates/`.
- `references/continuous-documentation.md` — what's worth writing back to project reference docs after a task, and what isn't.
- `references/live-validation.md` — why unit tests and direct API calls both miss integration/contract bugs, when a real browser/app click-through against a real stack is worth the cost, and how to do it without touching real user data.
- `references/obsidian-backend.md` — when the project's docs/plans live in an Obsidian vault, how this workflow's patterns (status lifecycle, archiving, cross-links) map onto one. Skip if the project doesn't use Obsidian. For the actual CLI commands, see the sibling `obsidian-vault-maintenance` skill; for how the vault should be laid out, see `obsidian-vault-structure`.
- `references/workflows-and-pacing.md` — recognizing workflow-shaped work, the opt-in gate, rate-limit pacing for large fan-outs.

## Related skills

If the project's docs/plans live in an Obsidian vault: `obsidian-vault-structure` covers how the vault itself should be laid out (map of content, per-topic entry points, where the plans/templates/archive folders sit), and `obsidian-vault-maintenance` covers the CLI mechanics to create and keep that layout current. This skill covers the workflow that runs on top of both — it doesn't duplicate their content, just points into them where relevant (`references/obsidian-backend.md`).

## Per-project setup

This skill is the general doctrine. A specific project should pin down, in its own agent-instructions file (`CLAUDE.md`, `AGENTS.md`, etc.), the concrete details this doctrine references but doesn't hardcode: where durable plans/Stories actually live (a docs vault, a `docs/plans/` folder, an issue tracker), what its current model lineup maps to each tier if it differs from the defaults in `model-tiers.md`, and which of its own reference docs exist and what topic each covers. Read that project config first — it's what turns this generic procedure into an executable one for that repo.
