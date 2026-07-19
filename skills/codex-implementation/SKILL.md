---
name: codex-implementation
description: Dispatch OpenAI Codex CLI (gpt-5.6-sol at low/xhigh/max/ultra) as a worker for bounded implementation, refactors, migrations, bulk analysis, or codebase discovery. Use when the orchestrator routes execution work to Codex per CLAUDE.md.
---

# Codex Implementation Worker

Dispatch scoped execution work to gpt-5.6-sol. Workers are stateless: give full context, expect a summary back, then review the diff yourself.

## Model + effort selection

Only model: `gpt-5.6-sol`. Pick effort:

- `low`: bulk reads, summaries, extraction, mechanical edits, renames/formatting
- `xhigh`: DEFAULT for scoped implementation and debugging
- `max`: rare — one tightly-coupled hard problem
- `ultra`: rare — large task that truly parallelizes (Sol self-spawns subagents)

Never omit `-c model_reasoning_effort=...` — the user's config default is `ultra`. Never use terra or luna.

## Command shapes

```bash
# Read-only discovery / bulk analysis
codex exec -m gpt-5.6-sol --sandbox read-only --skip-git-repo-check \
  -c model_reasoning_effort="low" \
  -o /tmp/codex-report.md "PROMPT" < /dev/null

# Implementation (writes allowed in workspace)
codex exec -m gpt-5.6-sol --sandbox workspace-write --skip-git-repo-check \
  -c model_reasoning_effort="xhigh" --cd /path/to/repo \
  -o /tmp/codex-report.md "PROMPT" < /dev/null
```

- Always close stdin (`< /dev/null`) — codex hangs waiting on it otherwise.
- Long specs: write to a temp markdown file and tell the worker to read it; don't inline walls of text.
- Runs can exceed Bash's 10-minute timeout: pass an explicit longer timeout, or background the command and poll for the `-o` report file.
- Parallel implementation workers MUST run in separate git worktrees (`git worktree add`) so edits don't collide. Use distinct `-o /tmp/worker-N.md` files.

## Prompt template (all six parts required)

1. `ROLE: WORKER` — verbatim first line. Disables orchestrator contracts and prevents delegation loops.
2. Goal — one paragraph, self-contained (the worker sees nothing else).
3. File allowlist — exact paths the worker may modify.
4. Non-goals — verbatim: "Change ONLY the listed files. Do not refactor, rename, add validation, or fix adjacent issues — flag them in your summary instead."
5. Stop condition — what done looks like (tests pass, file compiles, report written).
6. Output format — summary + list of files changed + flagged adjacent issues.

## After the worker returns

- Read the report file, then diff-gate: any file outside the allowlist, or a diff far larger than the task warrants, gets rejected and re-dispatched with the flags. Don't silently fix worker output yourself unless it's taste-critical.
- Verify the stop condition actually holds (run the tests); don't trust the worker's claim.
