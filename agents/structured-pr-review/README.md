# Claude Code PR-review subagent

A CLI wrapper that fetches a real GitHub PR, pins its head revision, invokes a
no-tools Claude Code reviewer, and prints a structured Markdown comment.
It never posts, merges, pushes, checks out or runs the target repository.

## Setup

1. Install Python 3.10+, GitHub CLI and Claude Code; sign in to `gh` and Claude.
2. Put this directory on PATH and run `chmod +x claude-review` (or invoke
   `bash /path/to/claude-review`). The Python implementation has no dependencies.
3. Run `claude-review --pr https://github.com/OWNER/REPO/pull/123`.

Use `--evidence-dir ./new-run` to retain the pinned packet, structured review,
rendered Markdown and hashes. The directory must be new so previous evidence is
not overwritten. `--collect-only` fetches the actual diff without model usage.
`--model`, `--timeout`, `--max-diff-bytes` and `--max-budget-usd` make limits
explicit. The default model budget is $1 per model invocation. A review uses two
invocations (draft and source-check), so their configured budgets can total $2.
The timeout is also per invocation, not the total wall-clock duration. These
options do not purchase credit or top up an account.

## Review contract

Summary (2-3 sentences), identified risks, improvement suggestions, and a
Low/Medium/High confidence level. The subagent receives all changed-file patches
provided by GitHub, with metadata, pinned head/base and explicit missing-patch
limitations. It distinguishes demonstrated defects from context-dependent risks.
Findings must cite a collected file and, when supplied, an actual new-side hunk
line. No findings are manufactured just to populate a list.

GitHub files are paginated; changed HEADs and incomplete counts abort the run.
Diff size over the explicit limit aborts rather than silently omitting code.
Binary/missing patches are declared, not treated as reviewed source. A diff-only
review cannot establish runtime correctness; output always says no tests ran.

Claude starts in a temporary empty cwd with no tools, no project/user settings,
no hooks, no MCP connections and no saved session. The script's GitHub reads are
performed by `gh`, outside the model. It does not weaken the owner's existing
Claude configuration or change credentials. No token values are put in prompts.
For private PRs, the chosen Claude provider receives the diff: use this only when
you have authorization for that provider to process the repository.

The printed Markdown is ready to copy into a PR comment after review. Posting
is deliberately a separate action. `samples/` holds actual executed examples
and labels their validation scope. Deterministic tests use fake process outputs;
they do not masquerade as live Claude results.

```sh
python3 -B -m unittest discover -s tests -v
```

References: https://code.claude.com/docs/en/sub-agents and
https://code.claude.com/docs/en/cli-reference (checked 2026-09-17).

## Current validation status

The runner has 34 passing tests and actual two-pass Claude Code executions on
Click PRs #3782 and #3860. Both use local qwen3.5:4b, not Anthropic inference.
The additional verification pass removed the false suggestions in #3782, but
#3860 still has an inaccurate return-type description and speculative advice.
A larger PR timed out on this local backend. This remains a draft pending
review-quality and performance validation. Read SAMPLE-QUALITY.md before using sample comments. The tool never
posts them automatically. One JSON code fence is accepted; extra prose is not.
