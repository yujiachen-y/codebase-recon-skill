---
name: codebase-recon
description: >
  Use when entering an unfamiliar codebase, onboarding to a new project,
  or wanting to assess codebase health before reading code — analyzes git
  history to reveal hotspots, risk areas, team structure, and development
  momentum
compatibility: Requires git. Works in any terminal-based coding agent.
metadata:
  author: yujiachen-y
  version: "1.0"
  inspired-by: https://piechowski.io/post/git-commands-before-reading-code/
---

# Codebase Recon

Analyze git history to understand a codebase before reading any code. Reveals project health, risk areas, team structure, and development momentum.

Inspired by ["The Git Commands I Run Before Reading Any Code"](https://piechowski.io/post/git-commands-before-reading-code/) by [Ally Piechowski](https://github.com/grepsedawk).

## Phase 1: Probe

Before running analysis, determine repo scale to calibrate time windows and result counts.

Run this single shell command to collect repo vitals:

```sh
echo "COMMITS=$(git rev-list --count HEAD)" && \
echo "FIRST_COMMIT=$(git log --reverse --format='%ad' --date=short | head -1)" && \
echo "LATEST_COMMIT=$(git log --format='%ad' --date=short | head -1)" && \
echo "BRANCHES=$(git branch -a | wc -l | tr -d ' ')"
```

Use the commit count to set parameters for Phase 2:

| Repo Size | Commits | `WINDOW` (--since) | `N` (--head) |
|-----------|---------|---------------------|--------------|
| Small     | <500    | (omit --since)      | 10           |
| Medium    | 500-10k | `1 year ago`        | 20           |
| Large     | >10k    | `6 months ago`      | 30           |

Print the Repo Vitals line immediately:

```
Repo Vitals: Age: [FIRST_COMMIT to LATEST_COMMIT] | Commits: [COMMITS] | Branches: [BRANCHES] | Analysis window: [WINDOW or "all time"]
```
