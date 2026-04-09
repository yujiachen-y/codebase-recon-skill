# Codebase Recon — Skill Design Spec

## Attribution

Inspired by ["The Git Commands I Run Before Reading Any Code"](https://piechowski.io/post/git-commands-before-reading-code/) by Ally Piechowski.

## Overview

A Claude Code skill that analyzes git history to reveal project health, risk areas, team structure, and development momentum — before reading any code. Invoked explicitly via `/codebase-recon`.

**Target audience**: Anyone starting work in an unfamiliar repo — developers onboarding, tech leads auditing, consultants evaluating.

**Distribution**: Standalone plugin repo (`codebase-recon-skill`), installable via:
- **Claude Code**: plugin system (`/install-skill` or marketplace)
- **skills.sh**: `npx skills add <owner>/codebase-recon-skill` — auto-listed via install telemetry, making it available to 20+ coding agents (Cline, Cursor, GitHub Copilot, Gemini CLI, etc.)

**Cross-agent compatibility**: The skill follows the [Agent Skills Specification](https://agentskills.io/specification). It uses only `SKILL.md` frontmatter (`name`, `description`, `compatibility`) and standard git/shell commands — no Claude-specific tool names in the instructions. This ensures any agent that supports the Agent Skills spec can execute it.

## Execution Model: Two-Phase

### Phase 1 — Probe

A single Bash call to determine repo scale and calibrate analysis parameters.

Collected vitals:
- Total commit count (`git rev-list --count HEAD`)
- First and most recent commit dates
- Branch count (`git branch -a | wc -l`)

Calibration table:

| Repo Size | Commits | `--since` Window | `--head` Count |
|-----------|---------|------------------|----------------|
| Small     | <500    | entire history   | 10             |
| Medium    | 500–10k | 1 year           | 20             |
| Large     | >10k    | 6 months         | 30             |

### Phase 2 — Parallel Analysis

7 independent Bash calls dispatched in a single message. `WINDOW` and `N` are set by Phase 1 calibration.

| # | Dimension | Source | Command |
|---|-----------|--------|---------|
| 1 | Code hotspots | Article | `git log --format=format: --name-only --since=WINDOW \| sort \| uniq -c \| sort -nr \| head -N` |
| 2 | Bus factor | Article | `git shortlog -sn --no-merges` |
| 3 | Bug magnets | Article | `git log -i -E --grep="fix\|bug\|broken" --name-only --format='' --since=WINDOW \| sort \| uniq -c \| sort -nr \| head -N` |
| 4 | Team momentum | Article | `git log --format='%ad' --date=format:'%Y-%m' \| sort \| uniq -c` |
| 5 | Firefighting | Article | `git log --oneline --since=WINDOW \| grep -iE 'revert\|hotfix\|emergency\|rollback'` |
| 6 | Recently added files | New | `git log --diff-filter=A --since=WINDOW --name-only --format='' \| sort \| uniq -c \| sort -nr \| head -N` |
| 7 | Active vs total contributors | New | `git shortlog -sn --no-merges --since="3 months ago" \| wc -l` (compared against total from #2). Note: the 3-month window is fixed regardless of repo size — it measures "who's here now", not historical activity. |

## Report Structure

Terminal output, structured as follows:

```
═══ Codebase Recon Report ═══

Repo Vitals
   Age: X | Commits: N | Branches: N | Analysis window: WINDOW

1. Code Hotspots (most-changed files)
   [raw ranked list]

2. Bug Magnets (files with fix/bug commits)
   [raw ranked list]

3. High-Risk Files (appear in BOTH hotspots AND bug magnets)
   [cross-referenced list with rank in each dimension]

4. Bus Factor
   [contributor ranking]
   Active (last 3 months): X of Y total contributors

5. Team Momentum
   [monthly commit frequency]
   Trend: [rising / stable / declining / erratic]

6. Firefighting Frequency
   [revert/hotfix/emergency commits]
   Rate: N emergency commits out of M total (X%)

7. Recently Added Files
   [new files — where active development is happening]

8. Recommendations
   - "Start reading: [top 3 high-risk files]"
   - "Talk to: [top contributor for high-risk areas]"
   - "Watch out: [trend warnings if applicable]"
```

## Cross-Referencing Logic

- Intersect hotspots (#1) with bug magnets (#2) to produce **High-Risk Files** (#3)
- For each high-risk file, identify the primary owner via `git shortlog -sn -- <file>` — feeds into Recommendations
- Compare active vs total contributors — flag bus factor risk if ratio is low

## Post-Report Interaction

After printing the terminal report, Claude asks:

> "Want me to save this report to a markdown file? (e.g., `docs/codebase-recon-report.md`)"

If yes, write the same content as markdown. Do not commit — let the user decide.

## Skill Metadata

Per [Agent Skills Specification](https://agentskills.io/specification):

```yaml
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
```

### Cross-Agent Compatibility Notes

The SKILL.md body must use **generic instruction language**, not Claude-specific tool names:
- Say "run this shell command" not "use the Bash tool"
- Say "read the output" not "use the Read tool"
- Say "search for files" not "use the Grep tool"

This ensures any agent (Claude Code, Cline, Cursor, Copilot, Gemini CLI, etc.) can interpret and execute the instructions via its own tool set.

## Repo Structure

```
codebase-recon-skill/
  .claude-plugin/
    plugin.json           # Plugin metadata (name, version, author, keywords)
    marketplace.json      # Self-hosted marketplace config
  skills/
    codebase-recon/
      SKILL.md            # Main skill file
  README.md               # Installation & usage instructions, attribution
  LICENSE                  # MIT
  docs/
    specs/
      2026-04-09-codebase-recon-design.md   # This file
```
