# Cross-tool Delegation (Claude ↔ Codex)

Claude Code and Codex can each use the other as an **optional, independent delegate** alongside same-tool sub-agents — not a replacement for them, and not always present. The value isn't raw capability, it's a genuinely different model lineage with uncorrelated blind spots: useful specifically for independent second opinions and for genuinely parallel, isolated workstreams. This section describes the pattern from Claude Code's side (verified concretely); apply the mirror pattern from Codex's side using whatever integration is actually wired into that harness — don't assume a specific reciprocal tool exists without checking.

## Availability check — do this first, every time

Never assume the other tool is reachable; check every time before relying on it.

1. Look for a registered MCP tool pair for the other tool (e.g. from Claude Code, search for `mcp__codex__codex` / `mcp__codex__codex-reply`; from Codex, check `codex mcp list` / `~/.codex/config.toml`'s `[mcp_servers.*]` entries for one wrapping the `claude` CLI). If found, use it directly.
2. If not found, an MCP server only attaches at session start — a mid-session registration won't show up until the next session. Fall back to the other tool's headless CLI instead: from Claude Code, `codex exec "<prompt>" --json -C <target-dir>`; from Codex, `claude -p "<prompt>"` (add `--add-dir <dir>` for tool access outside the cwd). Same delegation, CLI transport, one tier down.
3. If neither works (binary missing, or the tool's own doctor/health command reports auth/connectivity failures), treat it as unavailable for this session and fall back to a same-tool sub-agent instead. Never block a task on the other tool being reachable.

MCP wiring and CLI availability are both environment-specific (which servers are registered, whether the other tool's binary is even installed) — re-check rather than trusting a prior session's finding, and expect it's common to have the CLI-fallback tier available without the MCP tier being set up in either direction.

## When to actually reach for it

The same two slots the independent-second-opinion tier fills generally (`model-tiers.md`), plus one specific to this pairing:

- **Independent second opinion** on a high-stakes/irreversible review, or on a diff you yourself wrote — a different model family shares no correlated blind spots with your own default lineup.
- **Genuinely parallel, isolated workstream** — a separate repo, or a separate git worktree. **Never** point both tools at the same working tree concurrently — neither takes a lock on the other, so their edits collide.

## Model selection within the other tool

Apply the same cheapest-that-works tier logic (`model-tiers.md`) using whichever models are current in the other tool's own lineup — don't assume its tier boundaries match your own tool's. Re-derive the live list rather than trusting a hardcoded name; each tool tends to cache its own current lineup locally (e.g. Codex refreshes `~/.codex/models_cache.json` on use) and updates it independently of this doc.

## Usage-limit cooldown — defer, don't hammer it

Both tools' underlying account plans tend to have account-level usage limits (multi-hour/weekly caps), not per-request token limits — hitting one is not a transient error worth retrying immediately.

- **Detecting it:** a call to the other tool fails with usage/rate-limit wording ("usage limit", "rate limit", "quota exceeded", HTTP 429).
- **On hit:** record the timestamp to a shared marker file (e.g. `date -u +%Y-%m-%dT%H:%M:%SZ > ~/.codex/.claude_cooldown` when Codex is the one that hit its limit) — then fall back to a same-tool sub-agent for that task and finish the turn normally. Don't retry the other tool again in the same session. Keep the marker global to the account, not scoped to one project — the usage limit isn't per-repo.
- **Before delegating at all, check the cooldown:** e.g. `find ~/.codex/.claude_cooldown -mmin -60` — a hit within the last hour means the cooldown is still active; skip straight to the same-tool fallback without attempting the call. No file, or one older than the window, means it's eligible again.
- This is a soft, session-local convention (a file check you make yourself), not an enforced scheduler. A long-running/looping session that wants to actually retry once the cooldown lapses should schedule a wakeup for then (e.g. Claude Code's `ScheduleWakeup`) rather than busy-wait or poll in a tight loop.
