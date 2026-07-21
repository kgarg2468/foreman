---
name: dispatch-workers
description: Dispatch worker agents (gpt-5.6-sol via codex exec, or Claude opus-4.8 / fable-5 via claude -p) for bounded implementation, refactors, bulk analysis, discovery, taste, or judgment escalate, per AGENTS.md. Use when routing execution work to workers.
---

# Dispatch Workers

Workers are stateless: give full context, expect a summary back, then review the diff yourself.

## Model + effort selection

GPT — only `gpt-5.6-sol`:

- `low`: bulk reads, summaries, extraction, mechanical edits
- `xhigh`: DEFAULT scoped implementation and debugging
- `max`: rare — one tightly-coupled hard problem
- `ultra`: rare — large task that truly parallelizes

Claude:

- `opus-4.8` + `xhigh`: default taste / cross-review (UI, copy, public APIs, significant diffs)
- `fable-5` + `high`: rare judgment escalate (architecture, synthesis, taste ceiling). Never ultracode. Never security/cyber/bio.

Never omit GPT effort — this machine's config default is `ultra`. Never use terra or luna.

## Command shapes

```bash
# GPT worker, read-only / bulk
codex exec -m gpt-5.6-sol --sandbox read-only --skip-git-repo-check \
  -c model_reasoning_effort="low" \
  -o /tmp/worker-1.md "PROMPT" < /dev/null

# GPT worker, implementation
codex exec -m gpt-5.6-sol --sandbox workspace-write --skip-git-repo-check \
  -c model_reasoning_effort="xhigh" --cd /path/to/repo \
  -o /tmp/worker-1.md "PROMPT" < /dev/null

# Opus taste / review worker
claude -p --model opus --effort xhigh "PROMPT" < /dev/null

# Fable judgment escalate (rare)
claude -p --model fable --effort high "PROMPT" < /dev/null
```

- Always close stdin (`< /dev/null`) — codex hangs waiting on it otherwise.
- Long specs: write to a temp markdown file and tell the worker to read it.
- Long runs: explicit shell timeout, or background and poll the `-o` report file.
- Parallel implementation workers: separate git worktrees (`git worktree add`), distinct `-o` files.

## Prompt template (all six parts required)

1. `ROLE: WORKER` — verbatim first line. This disables orchestration for the worker and prevents delegation loops (required for Opus and Fable too — they load CLAUDE.md).
2. Goal — one paragraph, self-contained (the worker sees nothing else).
3. File allowlist — exact paths the worker may modify.
4. Non-goals — verbatim: "Change ONLY the listed files. Do not refactor, rename, add validation, or fix adjacent issues — flag them in your summary instead."
5. Stop condition — what done looks like (tests pass, report written).
6. Output format — summary + files changed + flagged adjacent issues.

## After the worker returns

- Read the report file, then diff-gate: any file outside the allowlist, or a diff far larger than the task warrants, gets rejected and re-dispatched with the flags.
- Verify the stop condition actually holds (run the tests); don't trust the worker's claim.
