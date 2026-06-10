---
description: Simple code review of the current git diff
model: sonnet
#disable-model-invocation: true
#user-invocable: false
---

Review the current git diff and give short, actionable feedback.

## Steps

1. Run `git diff HEAD` to get the current changes. If empty, try `git diff --staged`.
2. Read any changed files that need more context.
3. Output a review with two sections:

**Bugs** — correctness issues, wrong logic, missing error handling, security problems. Only real issues, no style nitpicks.

**Improvements** — simplifications, redundant code, readability wins. Top 2-3 only.

If a section has nothing to flag, omit it. Keep the entire review under 20 lines.
