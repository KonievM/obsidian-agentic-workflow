---
name: obsidian-agentic-workflow
description: Operating procedure for an orchestrating coding agent (Claude Code or Codex) managing multi-step engineering work — when and how to delegate to sub-agents, which model/reasoning tier to use for a task, how to delegate to the other tool (Claude and Codex, whichever one isn't the current agent) as an independent second opinion or a parallel workstream, how to track and hand off durable multi-session work (flat plans and multi-workstream Stories, including ones stored in an Obsidian vault), when to propose a deterministic multi-agent workflow, and how to keep project reference docs current. Use whenever a task is large enough to delegate, spans more than one session, needs a durable plan/story file to resume from, involves choosing a model tier, touches an Obsidian vault used for docs/plans, or should update the project's reference docs afterward.
---

# Agentic Workflow

Operating procedure for a managing agent — the session talking to the user — running engineering work of real size. Applies equally whether the managing agent is Claude Code or Codex; where the two diverge (tool names, MCP wiring), the relevant reference file says so explicitly.

## Philosophy

Orchestrate, not execute. Decompose the task, decide what to delegate, brief sub-agents, read and verify their results, integrate. Once a piece of work is delegable, doing it yourself instead is a cost: it burns your own context on detail nobody downstream needs (raw exploration, draft iterations, file dumps), and it serializes work that could run in parallel.

Don't over-apply this — a single grep, a single file read, a two-line fix doesn't need a sub-agent; the delegation overhead (spin-up, briefing, reading the result back) costs more than doing it inline. Delegate when a task is well-scoped *and* either parallelizable, would pollute your own context, or is simply a large enough self-contained chunk on its own.

**Resist mid-task scope creep back into execution.** The common failure isn't over-delegating, it's the reverse: a new constraint or fix-up lands mid-task and it feels like less overhead to just do it inline than to re-brief. That's exactly how a manager's context re-fills with the exploration/drafts it was supposed to stay clear of. Treat new instructions the same as the original ask — delegate them too. See `references/subagent-lifecycle.md`.

## Quick checklist

1. Can this be answered in 1–2 direct tool calls? → just do it.
2. Is it well-scoped and parallelizable, would it bloat your context, or is it just a big self-contained chunk? → delegate. Pick a tier from `references/model-tiers.md`. Size alone is a reason to delegate.
3. New instructions landing mid-task? → delegate those too (re-brief or spawn) — don't fold them into your own direct execution.
4. Is the work complex, cross-cutting, or parallelizable enough to need a multi-file Story instead of a flat plan? → set one up per `references/task-intake-and-tracking.md` before spawning sub-agents against it.
5. Is a single sub-agent's task big enough to span multiple milestones? → checkpoint and rotate per `references/subagent-lifecycle.md`, don't let one sub-agent run indefinitely.
6. Brief every sub-agent with full context — it starts with none of this conversation.
7. When it returns: verify, don't just relay. Check the actual diff/output, not only the summary. Sync any durable plan status immediately — don't batch it to the end (`references/task-intake-and-tracking.md`).
8. Durable output goes where it belongs — a plan/Story file, a repo commit, a project reference doc — never left stranded only in a transcript.
9. Does the task look like 3+ dependent/parallel agent calls (a big audit, migration, or review)? → propose a deterministic multi-agent workflow instead of hand-chaining sub-agent calls yourself, per `references/workflows-and-pacing.md` — but still wait for explicit user opt-in before running one.
10. About to launch a large fan-out while another is still active, or seeing repeated rate/error signals? → pace it per `references/workflows-and-pacing.md`.
11. Want an independent second opinion, or a genuinely separate parallel workstream? → check whether the other tool (Claude ↔ Codex) is reachable and off cooldown, per `references/cross-tool-delegation.md`.
12. Finishing a non-trivial task? → check whether anything learned is worth persisting to the project's reference docs, per `references/continuous-documentation.md`.

## Reference files

- `references/model-tiers.md` — cheapest-that-works tier framework, task-type → tier table, current model mappings for Claude and Codex.
- `references/subagent-lifecycle.md` — when to spawn, briefing, nesting limit, rotation for long sub-agent runs, who verifies, where results live.
- `references/cross-tool-delegation.md` — using the other tool (Claude ↔ Codex) as an independent-model delegate: availability check, when to reach for it, usage-limit cooldown handling.
- `references/task-intake-and-tracking.md` — flat plan vs. multi-workstream Story, status lifecycle, contract-first parallelism, real-time tracking, archiving. Generic starter templates for both live in `assets/templates/`.
- `references/continuous-documentation.md` — what's worth writing back to project reference docs after a task, and what isn't.
- `references/obsidian-backend.md` — when the project's docs/plans live in an Obsidian vault, the CLI commands and conventions that implement the two files above concretely (dropdown status properties, wikilinks, archiving via `move`). Skip if the project doesn't use Obsidian.
- `references/workflows-and-pacing.md` — recognizing workflow-shaped work, the opt-in gate, rate-limit pacing for large fan-outs.

## Per-project setup

This skill is the general doctrine. A specific project should pin down, in its own agent-instructions file (`CLAUDE.md`, `AGENTS.md`, etc.), the concrete details this doctrine references but doesn't hardcode: where durable plans/Stories actually live (a docs vault, a `docs/plans/` folder, an issue tracker), what its current model lineup maps to each tier if it differs from the defaults in `model-tiers.md`, and which of its own reference docs exist and what topic each covers. Read that project config first — it's what turns this generic procedure into an executable one for that repo.
