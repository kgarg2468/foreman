---
name: dispatch-workers
description: Dispatch worker agents (gpt-5.6 sol/terra/luna via codex exec, or Claude opus-4.8 via claude -p) for bounded implementation, refactors, bulk analysis, or discovery, per the orchestrator contract in AGENTS.md. Use when routing execution work to workers.
---

# Dispatch Workers

Workers are stateless: give full context, expect a summary back, then review the diff yourself.

## Model + effort selection

- `gpt-5.6-terra` + `high`: default for scoped implementation and debugging.
- `gpt-5.6-sol` + `high`: hard shards only (long-horizon, multi-file, gnarly). `xhigh`/`max` for one genuinely hard problem; `ultra` when one large task splits into parallel streams (sol self-spawns subagents — prefer this over a hand-rolled sol swarm).
- `gpt-5.6-luna` + `medium`: bulk reads, summaries, extraction, mechanical edits (`low` for renames/formatting).
- `opus-4.8`: taste-critical implementation (UI, copy, public APIs). Bills the Claude subscription — use deliberately.
- Never omit effort — this machine's config default is `ultra`.

## Command shapes

```bash
# GPT worker, read-only discovery / analysis
codex exec -m gpt-5.6-luna --sandbox read-only --skip-git-repo-check \
  -c model_reasoning_effort="medium" \
  -o /tmp/worker-1.md "PROMPT" < /dev/null

# GPT worker, implementation (writes allowed in workspace)
codex exec -m gpt-5.6-terra --sandbox workspace-write --skip-git-repo-check \
  -c model_reasoning_effort="high" --cd /path/to/repo \
  -o /tmp/worker-1.md "PROMPT" < /dev/null

# Claude opus worker (taste lane)
claude -p --model opus "PROMPT" < /dev/null
```

- Always close stdin (`< /dev/null`) — codex hangs waiting on it otherwise.
- Long specs: write to a temp markdown file and tell the worker to read it.
- Long runs: explicit shell timeout, or background and poll the `-o` report file.
- Parallel implementation workers: separate git worktrees (`git worktree add`), distinct `-o` files.

## Prompt template (all six parts required)

1. `ROLE: WORKER` — verbatim first line. This disables orchestration for the worker and prevents delegation loops.
2. Goal — one paragraph, self-contained (the worker sees nothing else).
3. File allowlist — exact paths the worker may modify.
4. Non-goals — verbatim: "Change ONLY the listed files. Do not refactor, rename, add validation, or fix adjacent issues — flag them in your summary instead."
5. Stop condition — what done looks like (tests pass, report written).
6. Output format — summary + files changed + flagged adjacent issues.

## After the worker returns

- Read the report file, then diff-gate: any file outside the allowlist, or a diff far larger than the task warrants, gets rejected and re-dispatched with the flags.
- Verify the stop condition actually holds (run the tests); don't trust the worker's claim.
