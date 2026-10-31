# agent-toolkit

My personal Claude Code configuration: global instructions and custom skills.

## Contents

| Path | What it is |
|------|------------|
| `CLAUDE.md` | Global instructions loaded into every Claude Code session (working style, permission rules, git/PR conventions, Task Action Plans). |
| `tap/` | `/tap` skill: manage a **Task Action Plan**, a short, ordered, confidence-scored list of work items. Subcommands: `go`, `look`. |
| `pr-review/` | `/pr-review` skill: step-by-step PR review of the checked-out branch, with parallel security / logic / tech-debt review agents. `--compiled` produces a single report instead of pausing between steps. |

## Install

Symlink into your Claude Code config so edits here take effect immediately:

```sh
git clone git@github.com:drewstaylor/agent-toolkit.git
cd agent-toolkit

ln -s "$PWD/CLAUDE.md" ~/.claude/CLAUDE.md
mkdir -p ~/.claude/skills
ln -s "$PWD/tap"       ~/.claude/skills/tap
ln -s "$PWD/pr-review" ~/.claude/skills/pr-review
```

Back up any existing `~/.claude/CLAUDE.md` first.

## Task Action Plans (TAP)

A TAP is the human-facing summary of a plan: ordered work items, the files each one touches, and a confidence score for completing it autonomously. Work doesn't start below 90% confidence, and open questions must be resolved first, not deferred. See `CLAUDE.md` for the full rules.

## License

[MIT](LICENSE)
