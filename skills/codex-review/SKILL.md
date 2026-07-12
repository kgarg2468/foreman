---
name: codex-review
description: Ask Codex CLI (gpt-5.6) for an independent code review of uncommitted changes, a branch diff, or a specific implementation. Use when the user asks for a Codex or GPT review, when the CLAUDE.md routing calls for an extra independent review perspective, or when a diff should be audited for bugs and regressions by a different model family. For a review by Claude itself, use the normal review process instead.
---

# Codex Review

Use Codex as an independent reviewer when a second-pass review is wanted or a change is broad enough that another model family's perspective is useful.

Prefer the normal Claude review for small local checks. Do not delegate review just to avoid reading the code yourself. Treat Codex's output as evidence, not authority.

## Workflow

1. Identify the review target: uncommitted changes, base branch, or specific files.
2. Create a temp artifact directory for the report.
3. Run `codex review` with a focused prompt.
4. Read the report and verify important claims against the code before presenting them.

```bash
ARTIFACT_DIR="$(mktemp -d "${TMPDIR:-/tmp}/codex-review.XXXXXX")"
REPORT="$ARTIFACT_DIR/report.md"
PROMPT="$ARTIFACT_DIR/prompt.md"

# Review staged, unstaged, and untracked changes
codex -C "$PWD" review --uncommitted - < "$PROMPT" > "$REPORT"

# Review current branch against a base branch
codex -C "$PWD" review --base main - < "$PROMPT" > "$REPORT"
```

- Model/effort: default review runs on the config default model; for a heavier audit pass, add `-c model="gpt-5.6-sol" -c model_reasoning_effort="high"`.
- The prompt file must begin with `ROLE: WORKER` (disables the codex-side orchestrator contract) and state what the change was supposed to do, known risk areas, and what to prioritize (correctness > regressions > style — style feedback from GPT is low-value; taste review stays with Fable/Opus).

## Presenting results

- Cross-check each concrete claim (line numbers, alleged bugs) against the actual code; drop anything that doesn't hold up.
- Merge findings into your own review verdict — attribute independent findings to Codex so the user knows the source.
