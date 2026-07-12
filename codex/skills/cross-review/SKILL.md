---
name: cross-review
description: Get an independent cross-family review of a diff from Claude opus-4.8 (taste, design) or a fresh gpt-5.6-sol pass (correctness), per the orchestrator contract in AGENTS.md. Use before landing significant changes or when the user asks for a second opinion on a diff.
---

# Cross Review

Cross-family review is the default for significant diffs: you (or your workers) wrote the code, so a different model family reviews it. Treat reviewer output as evidence, not authority. Do not delegate review just to avoid reading the diff yourself.

## Reviewer selection

- `opus-4.8`: taste, API design, naming, UI — the lane GPT is weakest in. Bills the Claude subscription.
- Fresh `gpt-5.6-sol` at `high`: an independent correctness/regression pass when Claude isn't warranted.

## Workflow

1. Produce the diff to review (uncommitted, branch vs base, or specific files).
2. Write a focused prompt: what the change was supposed to do, known risk areas, priorities (correctness > regressions > style).
3. Run the reviewer and capture the report:

```bash
# Opus review of a diff
git diff main... > /tmp/review-diff.patch
claude -p --model opus "Review this diff for taste, API design, and correctness. \
Context: <what it does>. Diff follows:\n$(cat /tmp/review-diff.patch)" < /dev/null > /tmp/review-report.md

# Independent sol review pass
codex review --base main -c model="gpt-5.6-sol" -c model_reasoning_effort="high" \
  - < /tmp/review-prompt.md > /tmp/review-report.md
```

4. Cross-check each concrete claim (line numbers, alleged bugs) against the actual code; drop anything that doesn't hold up.
5. Merge findings into your own verdict, attributing independent findings to the reviewer.
