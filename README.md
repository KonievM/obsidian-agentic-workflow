# obsidian-agentic-workflow

A skill for [Claude Code](https://claude.com/claude-code) and [Codex](https://developers.openai.com/codex/) that gives an orchestrating coding agent a consistent operating procedure for engineering work bigger than a single edit — delegating to sub-agents, picking a model tier, tracking multi-session work, and keeping project docs current.

## The problem

An agent left to improvise on real-sized work tends to fail in a few predictable ways: it does everything inline instead of delegating, filling its own context with exploration and drafts nobody downstream needed; it defaults to the most expensive model out of habit, or the cheapest one for a call that actually needed depth; a plan or task list only reflects reality once, at the very end, so an interrupted session loses track of what was actually finished; and things learned mid-task evaporate with the conversation instead of getting written down anywhere durable. This skill is a written-down answer to those failure modes, not a new capability — it doesn't add tools, it tells an agent that already has sub-agents, multiple models, and file access how to use them without the usual drift.

## What it covers

| Area | Without this | What the skill provides |
|---|---|---|
| Sub-agent delegation | Sub-agents spun up ad hoc, under-briefed (they start with zero shared context), or run so long their own judgment degrades without anyone noticing | Concrete spawn criteria, a briefing checklist, rotation for long-running sub-agents, and a "verify the diff, don't just relay the summary" rule |
| Model/reasoning tier selection | Reaching for the priciest model as a default, or the cheapest one for a call that needed real judgment | A cheapest-that-works tier framework mapped to current Claude and Codex model lineups |
| Cross-tool delegation | Not knowing Claude and Codex can serve as each other's independent second opinion, or hammering a tool that's hit an account usage limit | An availability-check procedure, when it's actually worth reaching for a second model lineage, and a cooldown convention for usage limits |
| Task intake and tracking | A plan file that only reflects reality once, at the end; multiple sub-agents editing the same file and clobbering each other's work | Flat plans vs. multi-workstream Stories, one-file-per-subtask so parallel sub-agents don't collide, and a real-time (not batched) status-sync rule |
| Continuous documentation | Non-obvious discoveries that lived only in one conversation and are gone once it ends — or the opposite, docs bloated with notes nobody trusts because half of them just restate the code | A concrete worth-persisting/not-worth-persisting line, and where each kind of finding should go |
| Deterministic multi-agent workflows | Hand-chaining sub-agent calls yourself for a big audit/migration/review, which refills your own context with exactly what delegation was supposed to avoid — or the reverse, firing off an oversized fan-out that trips rate limits | Recognizing workflow-shaped work and proposing (never auto-running) a structured fan-out, plus pacing rules for large parallel work |
| Obsidian as a docs/task-intake backend | Knowing the patterns above exist but not how they map onto an actual vault | How task intake and documentation upkeep translate into vault notes/properties, when that's where a project's docs and plans live |

For the vault layout and CLI mechanics themselves (not covered here), see the companion [`obsidian-vault-toolkit`](https://github.com/KonievM/obsidian-vault-toolkit) (`obsidian-vault-structure` + `obsidian-vault-maintenance`).

## Install

**Claude Code:** copy (or clone) this folder into `~/.claude/skills/obsidian-agentic-workflow` (user-level) or `<project>/.claude/skills/obsidian-agentic-workflow` (project-level).

**Codex:** copy (or clone) this folder into `~/.codex/skills/obsidian-agentic-workflow` (or `$CODEX_HOME/skills/obsidian-agentic-workflow`).

The skill is self-contained — `SKILL.md` plus `references/` (loaded on trigger) and `assets/templates/` (generic starter templates for flat plans and Stories). See `SKILL.md` for the full structure.

## Per-project setup

This skill is general doctrine, not tied to any specific project. A project using it should pin down its own concrete details — where its plans/Stories actually live, its current model lineup if it differs from the defaults in `references/model-tiers.md`, and which reference docs it has — in its own agent-instructions file (`CLAUDE.md`, `AGENTS.md`, etc.). See `SKILL.md`'s "Per-project setup" section.

## License

MIT — see `LICENSE`.
