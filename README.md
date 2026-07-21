# foreman

Global orchestration setup for both manager families, across every harness that runs them (Claude Code, Codex CLI, Conductor, T3 Code):

- **Claude side**: Fable 5 manages at **high** effort (locked), dispatching `gpt-5.6-sol` workers (low / xhigh / max / ultra) and Opus at xhigh for taste.
- **Codex side**: GPT-5.6 Sol manages, dispatching the same Sol effort ladder, Opus at xhigh for default taste/review, and Fable at high for rare judgment escalate.

Which contract is active depends only on which model family you launch. Every dispatched worker prompt starts with a `ROLE: WORKER` sentinel, which drops the worker out of manager mode — one level of hierarchy, no delegation loops.

## Worker ladder

| Lane | Model + effort |
|---|---|
| Bulk / mechanical | sol + low |
| Default implement / debug / GPT review | sol + xhigh |
| Hardest single-thread | sol + max (rare) |
| Parallel mega-task | sol + ultra (rare) |
| Default taste / cross-review | opus + xhigh |
| Judgment escalate (Codex side) | fable + high (rare) |
| Manager (Claude side) | fable + high (locked) |

No Terra, Luna, Sonnet, Haiku, or Opus ultracode for dispatched workers.

## What's in here

| Path | Installs to | Purpose |
|---|---|---|
| `CLAUDE.md` | `~/.claude/CLAUDE.md` | Claude-side orchestrator contract (Fable as manager) |
| `skills/codex-implementation/` | `~/.claude/skills/codex-implementation/` | Claude-side: dispatching Sol workers |
| `skills/codex-review/` | `~/.claude/skills/codex-review/` | Claude-side: independent review via `codex review` |
| `codex/AGENTS.md` | `~/.codex/AGENTS.md` | Codex-side orchestrator contract (Sol as manager) |
| `codex/skills/dispatch-workers/` | `~/.codex/skills/dispatch-workers/` | Codex-side: dispatching Sol / Opus / Fable workers |
| `codex/skills/cross-review/` | `~/.codex/skills/cross-review/` | Codex-side: cross-family review |

## Setup on a new machine

Paste this to Claude Code (or any agent with shell access) on the target machine:

```text
Set up my foreman orchestrator config from https://github.com/kgarg2468/foreman

1. Clone the repo to ~/Documents/Projects/foreman:
   `gh repo clone kgarg2468/foreman`

2. Verify prerequisites, and tell me about anything missing instead of working around it:
   - `claude` CLI installed
   - `claude-opus` and `claude-fable` wrappers on PATH (Bedrock entrypoints; used by Codex
     dispatches — never plain `claude --model opus|fable`)
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
   codex exec -m gpt-5.6-sol --sandbox read-only --skip-git-repo-check \
     -c model_reasoning_effort="low" "Reply with exactly one word: READY" < /dev/null

5. Smoke-test that both contracts load:
   # Claude-side contract (use your normal Fable/Opus Claude Code session, or):
   claude -p --effort high "In one sentence: per your global instructions, which model
   and effort handle bulk mechanical work, and what is your manager effort?" < /dev/null
   Expected: sol at low for bulk; manager effort high (locked).
   codex exec -m gpt-5.6-sol --sandbox read-only --skip-git-repo-check \
     -c model_reasoning_effort="low" "In one sentence: per your global instructions,
     what is your role and which CLI handles default taste review?" < /dev/null
   Expected: orchestrator; claude-opus at xhigh for taste.

5b. Smoke-test Bedrock wrappers (Codex's Claude path):
   claude-opus -p --effort xhigh "Reply with exactly one word: READY" < /dev/null
   Expected: READY (bills Bedrock, not Anthropic subscription).

6. Smoke-test the worker sentinel: rerun the sol command from step 5 with the prompt
   prefixed by the line "ROLE: WORKER". Expected: it now says it is a worker and may
   not delegate.

7. Report what you installed, the results of all smoke tests, and anything that failed.
```

## Updating

Edit files here, commit, push. Machines using symlinks pick up changes on `git pull`. If you edit `~/.claude/CLAUDE.md` directly on a symlinked machine, the change lands in the repo clone — commit and push it from there.

## Notes

- Codex model/effort defaults live in `~/.codex/config.toml` per machine and are intentionally NOT managed by this repo — the contract instructs workers to always set model and effort explicitly, so local config defaults don't leak into dispatches.
- Claude taste/judgment workers from Codex must go through `claude-opus` / `claude-fable` (Bedrock wrappers on PATH). Plain `claude --model opus|fable` bills the wrong account.
- Those wrappers use isolated config dirs (`~/.claude-opus`, `~/.claude-fable`), so they do not load `~/.claude/CLAUDE.md`. Worker behavior is enforced by the `ROLE: WORKER` prompt prefix.
- On a work machine, remember Sol dispatches route code through your personal OpenAI subscription — check your employer's data policy first.
