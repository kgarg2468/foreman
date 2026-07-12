# claude-orchestrator

Global orchestration setup for both manager families, across every harness that runs them (Claude Code, Codex CLI, Conductor, T3 Code):

- **Claude side**: Fable 5 (or Opus) manages, dispatching GPT-5.6 workers via Codex CLI.
- **Codex side**: GPT-5.6 Sol manages, dispatching terra/luna/sol workers and Opus 4.8 for taste-critical work and cross-family review.

Which contract is active depends only on which model family you launch. Every dispatched worker prompt starts with a `ROLE: WORKER` sentinel, which drops the worker out of manager mode — one level of hierarchy, no delegation loops.

## What's in here

| Path | Installs to | Purpose |
|---|---|---|
| `CLAUDE.md` | `~/.claude/CLAUDE.md` | Claude-side orchestrator contract (Fable/Opus as manager) |
| `skills/codex-implementation/` | `~/.claude/skills/codex-implementation/` | Claude-side: dispatching Codex workers |
| `skills/codex-review/` | `~/.claude/skills/codex-review/` | Claude-side: independent review via `codex review` |
| `codex/AGENTS.md` | `~/.codex/AGENTS.md` | Codex-side orchestrator contract (Sol as manager) |
| `codex/skills/dispatch-workers/` | `~/.codex/skills/dispatch-workers/` | Codex-side: dispatching GPT and Opus workers |
| `codex/skills/cross-review/` | `~/.codex/skills/cross-review/` | Codex-side: cross-family review (Opus or fresh Sol) |

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
   - Back up any existing ~/.claude/CLAUDE.md or ~/.codex/AGENTS.md (if non-empty) to *.bak first
   - ln -sf <repo>/CLAUDE.md ~/.claude/CLAUDE.md
   - mkdir -p ~/.claude/skills ~/.codex/skills
   - ln -sfn <repo>/skills/codex-implementation ~/.claude/skills/codex-implementation
   - ln -sfn <repo>/skills/codex-review ~/.claude/skills/codex-review
   - ln -sf <repo>/codex/AGENTS.md ~/.codex/AGENTS.md
   - ln -sfn <repo>/codex/skills/dispatch-workers ~/.codex/skills/dispatch-workers
   - ln -sfn <repo>/codex/skills/cross-review ~/.codex/skills/cross-review

4. Smoke-test the worker path (should print READY in a few seconds):
   codex exec -m gpt-5.6-luna --sandbox read-only --skip-git-repo-check \
     -c model_reasoning_effort="low" "Reply with exactly one word: READY" < /dev/null

5. Smoke-test that both contracts load:
   claude -p --model haiku "In one sentence: per your global instructions, which models
   handle bulk mechanical work?" < /dev/null
   Expected: an answer mentioning luna/terra.
   codex exec -m gpt-5.6-sol --sandbox read-only --skip-git-repo-check \
     -c model_reasoning_effort="low" "In one sentence: per your global instructions,
     what is your role?" < /dev/null
   Expected: an answer saying it is the orchestrator.

6. Smoke-test the worker sentinel: rerun the sol command from step 5 with the prompt
   prefixed by the line "ROLE: WORKER". Expected: it now says it is a worker and may
   not delegate.

7. Report what you installed, the results of all smoke tests, and anything that failed.
```

## Updating

Edit files here, commit, push. Machines using symlinks pick up changes on `git pull`. If you edit `~/.claude/CLAUDE.md` directly on a symlinked machine, the change lands in the repo clone — commit and push it from there.

## Notes

- Codex model/effort defaults live in `~/.codex/config.toml` per machine and are intentionally NOT managed by this repo — the contract instructs workers to always set model and effort explicitly, so local config defaults don't leak into dispatches.
- On a work machine, remember this routes code through your personal OpenAI subscription — check your employer's data policy first.
