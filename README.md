# obsidian-agentic-workflow

A skill for [Claude Code](https://claude.com/claude-code) and [Codex](https://developers.openai.com/codex/) that gives an orchestrating coding agent a consistent operating procedure for engineering work bigger than a single edit — delegating to sub-agents, picking a model tier, tracking multi-session work, and keeping project docs current.

## The problem

An agent left to improvise on real-sized work tends to fail in a few predictable ways: it does everything inline instead of delegating, filling its own context with exploration and drafts nobody downstream needed; it defaults to the most expensive model out of habit, or the cheapest one for a call that actually needed depth; a plan or task list only reflects reality once, at the very end, so an interrupted session loses track of what was actually finished; and things learned mid-task evaporate with the conversation instead of getting written down anywhere durable. This skill is a written-down answer to those failure modes, not a new capability — it doesn't add tools, it tells an agent that already has sub-agents, multiple models, and file access how to use them without the usual drift.

## What it covers

| Area | What it gives you |
|---|---|
| Sub-agent delegation | Spawn criteria, a briefing checklist, rotation for long-running sub-agents, "verify the diff, don't just relay" — instead of ad hoc, under-briefed sub-agents |
| Multi-agent hierarchy | Root/lead/specialist/reviewer roles, task routing, shared-worktree ownership — instead of agents colliding on the same files with no one accountable for integration |
| Model/reasoning tier selection | A cheapest-that-works tier framework, mapped to current Claude and Codex lineups — instead of defaulting to the priciest (or cheapest) model out of habit |
| Cross-tool delegation | An availability check, when to actually reach for the other tool, a usage-limit cooldown convention — instead of not knowing Claude/Codex can check each other, or hammering a tool past its limit |
| Task intake and tracking | Flat plans vs. Stories, one file per subtask, real-time status sync, write-it-directly instead of draft-then-ask — instead of a plan that only reflects reality at the end |
| Live validation | When a real client/browser click-through against a real stack is worth the cost, and how to do it safely — instead of calling a feature done from unit tests and API checks alone |
| Continuous documentation | A worth-persisting line and where each finding goes, archiving vs. documenting as separate steps — instead of discoveries that evaporate, or an archive nobody folds back into the docs |
| Deterministic multi-agent workflows | Recognizing workflow-shaped work, proposing (never auto-running) a structured fan-out, pacing rules — instead of hand-chaining calls yourself or firing an oversized fan-out |
| Obsidian as a docs/task-intake backend | How the patterns above map onto vault notes/properties — instead of knowing the patterns exist but not how to apply them |

For the vault layout and CLI mechanics themselves (not covered here), see the companion [`obsidian-vault-toolkit`](https://github.com/KonievM/obsidian-vault-toolkit) (`obsidian-vault-structure` + `obsidian-vault-maintenance`).

## Install

**Claude Code** (user-level; swap the destination for `<project>/.claude/skills/obsidian-agentic-workflow` to install it project-scoped instead):

```bash
git clone https://github.com/KonievM/obsidian-agentic-workflow ~/.claude/skills/obsidian-agentic-workflow
```

**Codex:**

```bash
git clone https://github.com/KonievM/obsidian-agentic-workflow "${CODEX_HOME:-$HOME/.codex}/skills/obsidian-agentic-workflow"
```

No further setup — the skill triggers automatically once its folder is in place. It's self-contained: `SKILL.md` plus `references/` (loaded on trigger) and `assets/templates/` (generic starter templates for flat plans and Stories). See `SKILL.md` for the full structure.

## Per-project setup

This skill is general doctrine, not tied to any specific project. A project using it should pin down its own concrete details — where its plans/Stories actually live, its current model lineup if it differs from the defaults in `references/model-tiers.md`, and which reference docs it has — in its own agent-instructions file (`CLAUDE.md`, `AGENTS.md`, etc.). See `SKILL.md`'s "Per-project setup" section.

## License

MIT — see `LICENSE`.
