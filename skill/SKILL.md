---
name: codex-lanes
description: "Launch, steer, inspect and stop Codex app-server lanes from Claude Code. Use when a Codex task needs mid-turn steering or follow-up turns on the same thread."
---

# Codex lanes

## Setup

Install the repository's `codex-lane` executable on PATH. Install this `skill/`
directory as `~/.claude/skills/codex-lanes/`.
Run `codex-lane --help` for requirements and environment settings.
Authenticate Codex before starting a lane. The default `CODEX_HOME` is `~/.codex`.
Lanes run with full filesystem access and approvals disabled. Use a trusted brief
and a checkout whose files the lane may change.

## Scope the task

Read the relevant source before writing a brief. Name the goal, owned files,
forbidden paths, contract, verification commands and expected report.
Assign separate owned paths when several lanes share a checkout.
Choose the model through `--model MODEL` or `CODEX_LANE_MODEL`, or use Codex's
configured model. Set `CODEX_LANE_SERVICE_TIER` to override its configured tier.
Set `CODEX_LANE_PREAMBLE` to text that should precede every initial brief.
The preamble does not apply to steer messages or later turns.

## Start or resume

```bash
codex-lane start review /path/to/repo /path/to/brief.md --model MODEL
codex-lane status review
```

Use `resume` to adopt an existing Codex thread. Its transcript stays available.

```bash
codex-lane resume review THREAD_ID /path/to/repo /path/to/follow-up.md
```

State, logs, events and the last message live under
`~/.local/state/codex-lane/`. Override this with `CODEX_LANE_DIR`.
The supervisor forks into the background. Session cleanup may still kill it;
process lifetime beyond the launching session is not guaranteed.

## Steer and inspect

```bash
codex-lane steer review "Check the error path too."
codex-lane steer review @/path/to/update.md
codex-lane status
codex-lane last review
codex-lane watch review
```

`steer` injects input into a running turn, or starts a follow-up turn when idle.
`watch` streams turn completion, errors, failed commands, process exit and new
commits. It starts at the end of existing logs. Run it under the caller's
available process-monitoring tool when notifications are needed.

`status` reports process liveness, turn count and the age of worktree changes.
Turn notifications can lag. Read the lane's `.log` and `.log.stderr` files when
status and worktree activity disagree. Use `last` for the latest completed agent
message.

## Finish

Read the result and inspect the changed files. Run the brief's verification
commands on the real entry point. Report unconfirmed claims explicitly.

```bash
codex-lane interrupt review
codex-lane stop review
```

`interrupt` ends the active turn. `stop` terminates the lane's app-server.
