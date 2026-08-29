# Model / Reasoning Tiers

Pick the cheapest tier that reliably does the task correctly — not the most capable one available. Four tiers, defined by role rather than a specific model name (lineups change; the roles don't):

| Tier | Role | Example task shapes |
|---|---|---|
| **Fast/cheap** | Well-specified, checkable transformations — pattern-matching quality, not deep reasoning | Proofreading, text generation, translation/localization, copy adaptation, mechanical retrieval/formatting before handing off to a costlier reviewer |
| **Balanced/default** | The coding & reasoning workhorse; codebase-grounded work across many files; orchestration itself | Feature design, implementation, most architectural review, sub-agent management/synthesis |
| **Max-reasoning** | Reserved for real tradeoff ambiguity or wide blast radius, not a default upgrade | High-ambiguity feature design, system-wide-blast-radius decisions |
| **Independent second opinion** | A *different model lineage* from your own default — the value isn't raw capability, it's uncorrelated blind spots | A second, independent panelist on an expensive-to-reverse decision (architecture decision, cross-repo contract change), or reviewing a diff you yourself wrote |

## Task type → tier

| # | Task type | Tier | Why |
|---|---|---|---|
| 1 | Proofreading, text generation, translation/localization, copy adaptation | Fast/cheap | Big cost/speed win, negligible quality loss for this task shape |
| 2 | Feature design (breaking a feature into steps, evaluating approaches, writing a design doc) | Balanced/default; escalate to max-reasoning only for real ambiguity or wide blast radius | Needs codebase-grounded reasoning across many files — most feature design doesn't need the ceiling |
| 3 | Architectural review & decisions | Balanced/default primary; add an independent-second-opinion pass for high-stakes/irreversible calls | Don't default every review to a two-model panel — reserve it for calls that actually warrant the extra cost |
| 4 | Sub-agent management/orchestration (coordinating other agents, synthesizing output, deciding what to spawn next) | Balanced/default — in practice, just inherit the managing session's own model rather than hardcoding | Control-flow and integration work, not generation — reliable instruction-following matters more than max reasoning depth. Keep the expensive tiers for the leaf work being orchestrated, not the orchestrator |
| 5 | Preparing data/context for review by a costly tier (gathering, filtering, summarizing before handing off) | Fast/cheap default; step up to balanced only when *deciding what's relevant* is itself a hard call | Mechanical retrieval/formatting doesn't need top-tier reasoning |

**Default for unclassified tasks:** balanced/default — the safe general-purpose choice.

## Current model mappings

Re-derive these when they look stale — model lineups change faster than this doc does.

**Claude:** fast/cheap = Haiku (current generation); balanced/default = Sonnet (current generation); max-reasoning = Opus (current generation); independent second opinion = a distinct model lineage available in your environment (a differently-trained model, not just a different size of the same family) *or*, when reachable, Codex — see `cross-tool-delegation.md`.

**Codex:** fast/cheap and max-reasoning are the low and high ends of Codex's own current agentic-coding lineup — check `~/.codex/models_cache.json` for the live list, filtered to `visibility == "list"`; the highest-effort model in the newest generation is your max-reasoning tier, the fastest/cheapest is your fast/cheap tier, everything else at default effort is balanced/default. Don't target an internal auto-review-only model directly (e.g. one marked `visibility: hide` in that cache) — those aren't meant to be picked by a caller. Codex's own lineup has no built-in "different lineage" slot — that's what Claude fills when reachable, see `cross-tool-delegation.md`.
