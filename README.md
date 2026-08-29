# obsidian-agentic-workflow

A skill for [Claude Code](https://claude.com/claude-code) and [Codex](https://developers.openai.com/codex/) covering how an orchestrating coding agent should manage multi-step engineering work:

- **Sub-agent delegation** — when to spawn, how to brief, rotation for long-running sub-agents, verify-don't-relay.
- **Model/reasoning tier selection** — cheapest-that-works, with current mappings for both Claude and Codex.
- **Cross-tool delegation** — using Claude and Codex as each other's independent second opinion or parallel workstream.
- **Task intake and tracking** — flat plans vs. multi-workstream Stories, status lifecycle, contract-first parallelism, real-time sync, archiving.
- **Continuous documentation** — what's worth writing back to project reference docs after a task.
- **Deterministic multi-agent workflows** — recognizing workflow-shaped work, the opt-in gate, rate-limit pacing.
- **Obsidian as a docs/task-intake backend** — how the task-intake and documentation patterns above map onto an Obsidian vault, when that's how a project stores its docs and plans. For the actual vault layout and CLI mechanics, see the companion [`obsidian-vault-toolkit`](https://github.com/KonievM/obsidian-vault-toolkit) (`obsidian-vault-structure` + `obsidian-vault-maintenance`).

## Install

**Claude Code:** copy (or clone) this folder into `~/.claude/skills/obsidian-agentic-workflow` (user-level) or `<project>/.claude/skills/obsidian-agentic-workflow` (project-level).

**Codex:** copy (or clone) this folder into `~/.codex/skills/obsidian-agentic-workflow` (or `$CODEX_HOME/skills/obsidian-agentic-workflow`).

The skill is self-contained — `SKILL.md` plus `references/` (loaded on trigger) and `assets/templates/` (generic starter templates for flat plans and Stories). See `SKILL.md` for the full structure.

## Per-project setup

This skill is general doctrine, not tied to any specific project. A project using it should pin down its own concrete details — where its plans/Stories actually live, its current model lineup if it differs from the defaults in `references/model-tiers.md`, and which reference docs it has — in its own agent-instructions file (`CLAUDE.md`, `AGENTS.md`, etc.). See `SKILL.md`'s "Per-project setup" section.

## License

MIT — see `LICENSE`.
