# claude-orchestrator

Global orchestration setup for Claude Code (and anything that runs it: Conductor, T3 Code). Fable 5 acts as the manager — planning, routing, and reviewing — while execution is dispatched to GPT-5.6 workers (sol/terra/luna) through the OpenAI Codex CLI, which bills a separate flat subscription.

## What's in here

| Path | Installs to | Purpose |
|---|---|---|
| `CLAUDE.md` | `~/.claude/CLAUDE.md` | Global orchestrator contract: model roster, routing rules, effort matrix, scope fencing |
| `skills/codex-implementation/` | `~/.claude/skills/codex-implementation/` | How to dispatch Codex workers (command shapes, prompt template, diff gate) |
| `skills/codex-review/` | `~/.claude/skills/codex-review/` | Independent code review via `codex review` |

## Setup on a new machine

Paste this to Claude Code (or any agent with shell access) on the target machine:

```text
Set up my Claude orchestrator config from https://github.com/kgarg2468/claude-orchestrator

1. Clone the repo to ~/Documents/Projects/claude-orchestrator (private repo — use my
   authenticated gh CLI: `gh repo clone kgarg2468/claude-orchestrator`).

2. Verify prerequisites, and tell me about anything missing instead of working around it:
   - `claude` CLI installed and logged in
   - `codex` CLI installed (`npm i -g @openai/codex`) and authenticated (`codex login`)
   - `git` available

3. Install by symlinking (so future `git pull` updates apply automatically):
   - Back up any existing ~/.claude/CLAUDE.md to ~/.claude/CLAUDE.md.bak first
   - ln -sf <repo>/CLAUDE.md ~/.claude/CLAUDE.md
   - mkdir -p ~/.claude/skills
   - ln -sfn <repo>/skills/codex-implementation ~/.claude/skills/codex-implementation
   - ln -sfn <repo>/skills/codex-review ~/.claude/skills/codex-review

4. Smoke-test the worker path (should print READY in a few seconds):
   codex exec -m gpt-5.6-luna --sandbox read-only --skip-git-repo-check \
     -c model_reasoning_effort="low" "Reply with exactly one word: READY" < /dev/null

5. Smoke-test that the contract loads:
   claude -p --model haiku "In one sentence: per your global instructions, which models
   handle bulk mechanical work?" < /dev/null
   Expected: an answer mentioning luna/terra.

6. Report what you installed, the results of both smoke tests, and anything that failed.
```

## Updating

Edit files here, commit, push. Machines using symlinks pick up changes on `git pull`. If you edit `~/.claude/CLAUDE.md` directly on a symlinked machine, the change lands in the repo clone — commit and push it from there.

## Notes

- Codex model/effort defaults live in `~/.codex/config.toml` per machine and are intentionally NOT managed by this repo — the contract instructs workers to always set model and effort explicitly, so local config defaults don't leak into dispatches.
- On a work machine, remember this routes code through your personal OpenAI subscription — check your employer's data policy first.
