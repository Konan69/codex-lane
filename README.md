# codex-lane

Steer a Codex agent in the middle of its turn.

`codex exec` runs one prompt to the end, so a wrong assumption in the brief costs the whole run. A lane keeps one `codex app-server` alive behind a small supervisor. `steer` adds to the turn that is running, or starts the next turn on the same thread when the lane is idle. Works from a shell or from another agent such as Claude Code.

![Explainer: codex-lane start launches a supervisor and codex app-server; a steer during turn 1 travels over review.sock and becomes turn/steer inside the running turn; a steer on the idle lane becomes turn/start for turn 2 on the same thread; codex-lane stop ends it](docs/media/hero.gif)

## Install

Linux, Python 3.9+, and a signed-in `codex` CLI on `PATH`. No dependencies.

```bash
install -m 755 codex-lane ~/.local/bin/codex-lane
mkdir -p ~/.claude/skills/codex-lanes && cp skill/SKILL.md ~/.claude/skills/codex-lanes/   # optional: lets Claude Code drive lanes
```

## Use

```bash
codex-lane start review ~/src/my-repo brief.md   # start a lane from a brief
codex-lane steer review "Check the error path too."
codex-lane steer review @update.md              # steer from a file
codex-lane status                               # every lane: status, turns, last worktree change
codex-lane last review                          # the last agent message
codex-lane watch                                # one line per finished turn, error or new commit
codex-lane stop review
```

Lanes run with full filesystem access and auto-approve everything, so point them at a checkout they are allowed to change. `codex-lane --help` lists every command.

## Configuration

| Variable | Default | |
| --- | --- | --- |
| `CODEX_HOME` | `~/.codex` | Codex config and auth |
| `CODEX_LANE_DIR` | `~/.local/state/codex-lane` | Lane state, logs and sockets |
| `CODEX_LANE_MODEL` | Codex's default | Model for new lanes (`--model` overrides) |
| `CODEX_LANE_SERVICE_TIER` | Codex's default | Passed as `-c service_tier=...` |
| `CODEX_LANE_PREAMBLE` | empty | Text put before the first brief only |

## Limits

- Linux only (`fork`, `/proc`, Unix sockets).
- The control socket has no authentication. Anyone who can write to it can steer or stop the lane.
- `status` can show `starting` with 0 turns while the lane is working, because the app-server does not always send turn notifications. The `.log` files tell the truth.
- `steer`, `interrupt` and `stop` on a lane that is not running exit with a Python traceback.

How the supervisor, socket and lane files work: [docs/internals.md](docs/internals.md).

## License

MIT
