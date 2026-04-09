# Codebase Recon Skill — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a standalone, cross-agent skill plugin that analyzes git history to produce a structured codebase health report.

**Architecture:** Single SKILL.md file containing all instructions for the two-phase execution model (probe then parallel analysis). No scripts or external dependencies — the skill is pure markdown instructions that any agent interprets and executes via shell commands.

**Tech Stack:** Markdown (SKILL.md), git commands, shell pipelines. No runtime dependencies.

**Spec:** `docs/specs/2026-04-09-codebase-recon-design.md`

---

## File Structure

| File | Responsibility |
|------|---------------|
| `skills/codebase-recon/SKILL.md` | Main skill — frontmatter + full instructions |
| `README.md` | Installation, usage, attribution, examples |
| `LICENSE` | MIT license |

---

### Task 1: Initialize Git Repo

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Initialize git repo**

Run:
```bash
cd ~/Projects/codebase-recon-skill
git init
```

Expected: `Initialized empty Git repository`

- [ ] **Step 2: Create .gitignore**

```gitignore
.DS_Store
*.swp
*~
```

- [ ] **Step 3: Initial commit**

Run:
```bash
git add .gitignore docs/specs/2026-04-09-codebase-recon-design.md
git commit -m "chore: add design spec and gitignore"
```

Expected: commit succeeds with 2 files

---

### Task 2: Create LICENSE

**Files:**
- Create: `LICENSE`

- [ ] **Step 1: Create MIT license file**

```
MIT License

Copyright (c) 2026 Jiachen Yu

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 2: Commit**

Run:
```bash
git add LICENSE
git commit -m "chore: add MIT license"
```

---

### Task 3: Create Claude Code Plugin Config

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Create plugin.json**

```json
{
  "name": "codebase-recon",
  "version": "1.0.0",
  "description": "Analyze git history to understand a codebase before reading any code. Reveals hotspots, risk areas, team structure, and development momentum.",
  "author": {
    "name": "Jiachen Yu",
    "url": "https://github.com/yujiachen-y"
  },
  "repository": "https://github.com/yujiachen-y/codebase-recon-skill",
  "license": "MIT",
  "keywords": [
    "git",
    "codebase",
    "recon",
    "onboarding",
    "code-health",
    "bus-factor",
    "hotspots"
  ]
}
```

- [ ] **Step 2: Create marketplace.json**

```json
{
  "name": "codebase-recon-skill",
  "owner": {
    "name": "Jiachen Yu",
    "url": "https://github.com/yujiachen-y"
  },
  "plugins": [
    {
      "name": "codebase-recon",
      "source": "./",
      "description": "Analyze git history to understand a codebase before reading any code",
      "version": "1.0.0"
    }
  ]
}
```

- [ ] **Step 3: Commit**

Run:
```bash
git add .claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "chore: add Claude Code plugin and marketplace config"
```

---

### Task 4: Write SKILL.md — Frontmatter and Overview

**Files:**
- Create: `skills/codebase-recon/SKILL.md`

This is the core deliverable. The SKILL.md must use generic instruction language (no Claude-specific tool names) per the cross-agent compatibility requirement.

- [ ] **Step 1: Create directory and write frontmatter + overview section**

```markdown
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
```

- [ ] **Step 2: Commit**

Run:
```bash
git add skills/codebase-recon/SKILL.md
git commit -m "feat: add SKILL.md with frontmatter and overview"
```

---

### Task 5: Write SKILL.md — Phase 1 (Probe)

**Files:**
- Modify: `skills/codebase-recon/SKILL.md`

- [ ] **Step 1: Append Phase 1 instructions**

Append the following to SKILL.md after the overview:

```markdown

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
```

- [ ] **Step 2: Commit**

Run:
```bash
git add skills/codebase-recon/SKILL.md
git commit -m "feat: add Phase 1 probe instructions"
```

---

### Task 6: Write SKILL.md — Phase 2 (Parallel Analysis)

**Files:**
- Modify: `skills/codebase-recon/SKILL.md`

- [ ] **Step 1: Append Phase 2 instructions**

Append the following to SKILL.md:

```markdown

## Phase 2: Parallel Analysis

Run all 7 commands in parallel (they are independent). Substitute `WINDOW` and `N` from Phase 1. For small repos, omit `--since` flags entirely.

### 2a. Code Hotspots

Most-changed files in the analysis window:

```sh
git log --format=format: --name-only --since="WINDOW" | sort | uniq -c | sort -nr | head -N
```

### 2b. Bus Factor

All-time contributor ranking by commit count:

```sh
git shortlog -sn --no-merges
```

### 2c. Bug Magnets

Files most associated with bug-fix commits:

```sh
git log -i -E --grep="fix|bug|broken" --name-only --format='' --since="WINDOW" | sort | uniq -c | sort -nr | head -N
```

### 2d. Team Momentum

Commit frequency by month (all time):

```sh
git log --format='%ad' --date=format:'%Y-%m' | sort | uniq -c
```

### 2e. Firefighting Frequency

Emergency/revert commits in the analysis window:

```sh
git log --oneline --since="WINDOW" | grep -iE 'revert|hotfix|emergency|rollback'
```

### 2f. Recently Added Files

New files added in the analysis window:

```sh
git log --diff-filter=A --since="WINDOW" --name-only --format='' | sort | uniq -c | sort -nr | head -N
```

### 2g. Active vs Total Contributors

Count of contributors active in the last 3 months (fixed window — measures "who's here now"):

```sh
git shortlog -sn --no-merges --since="3 months ago" | wc -l
```

Compare this count against the total from 2b.
```

- [ ] **Step 2: Commit**

Run:
```bash
git add skills/codebase-recon/SKILL.md
git commit -m "feat: add Phase 2 parallel analysis commands"
```

---

### Task 7: Write SKILL.md — Cross-Referencing and Report

**Files:**
- Modify: `skills/codebase-recon/SKILL.md`

- [ ] **Step 1: Append cross-referencing logic and report template**

Append the following to SKILL.md:

```markdown

## Cross-Referencing

After collecting all Phase 2 results, perform these cross-references before presenting the report:

1. **High-Risk Files**: Intersect code hotspots (2a) with bug magnets (2c). Files appearing in both lists are highest-risk.
2. **Risk Ownership**: For each high-risk file, run `git shortlog -sn -- <file>` to identify the primary owner.
3. **Bus Factor Risk**: If active contributors (2g) are less than 30% of total contributors (2b), flag this as a bus factor concern.
4. **Momentum Trend**: Analyze the monthly commit counts (2d):
   - Compare the average of the last 3 months to the average of the 3 months before that.
   - Rising: last 3 months average > prior 3 months average by 20%+
   - Declining: last 3 months average < prior 3 months average by 20%+
   - Erratic: month-over-month variance exceeds 50%
   - Stable: otherwise

## Report Template

Present the report in the terminal using this structure:

```
═══ Codebase Recon Report ═══

Repo Vitals
  Age: [first commit] to [latest commit] | Commits: N | Branches: N | Analysis window: WINDOW

1. Code Hotspots (most-changed files)
   [ranked list: count  filepath]

2. Bug Magnets (files with fix/bug/broken commits)
   [ranked list: count  filepath]

3. High-Risk Files (appear in BOTH hotspots AND bug magnets)
   [list with: filepath — hotspot rank #X, bug magnet rank #Y, primary owner: NAME]
   If none overlap, state: "No files appear in both lists — good sign."

4. Bus Factor
   [top 10 contributors: count  name]
   Active (last 3 months): X of Y total contributors
   [If active < 30% of total: "Warning: low active contributor ratio — knowledge concentration risk"]

5. Team Momentum
   [monthly commit counts, most recent 12 months or all if fewer]
   Trend: [rising / stable / declining / erratic]

6. Firefighting Frequency
   [list of revert/hotfix/emergency commits, or "None found"]
   Rate: N emergency commits out of M total in window (X%)

7. Recently Added Files
   [ranked list: count  filepath]

8. Recommendations
   - Start reading: [top 3 high-risk files, or top 3 hotspots if no high-risk files]
   - Talk to: [primary owner of the #1 high-risk or hotspot file]
   - Watch out: [any trend warnings — declining momentum, low bus factor, high firefighting rate]
```

After printing the report, ask:

> "Want me to save this report to a markdown file? (e.g., `docs/codebase-recon-report.md`)"

If yes, write the same content as a markdown file. Do not commit — let the user decide.
```

- [ ] **Step 2: Commit**

Run:
```bash
git add skills/codebase-recon/SKILL.md
git commit -m "feat: add cross-referencing logic and report template"
```

---

### Task 8: Write README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README with installation, usage, attribution, and example**

```markdown
# codebase-recon-skill

A coding agent skill that analyzes git history to understand a codebase before reading any code. Reveals project health, risk areas, team structure, and development momentum.

Inspired by ["The Git Commands I Run Before Reading Any Code"](https://piechowski.io/post/git-commands-before-reading-code/) by [Ally Piechowski](https://github.com/grepsedawk).

## Installation

### Via skills.sh (works with 20+ coding agents)

```sh
npx skills add <owner>/codebase-recon-skill
```

Works with Claude Code, Cline, Cursor, GitHub Copilot, Gemini CLI, and any agent supporting the [Agent Skills Specification](https://agentskills.io/specification).

### Via Claude Code plugin system

```
/plugin marketplace add yujiachen-y/codebase-recon-skill
```

Then install the plugin from the marketplace browser via `/plugin`.

## Usage

In your coding agent, invoke:

```
/codebase-recon
```

The skill will:
1. **Probe** the repo to determine its scale (small / medium / large)
2. **Analyze** 7 dimensions in parallel: code hotspots, bus factor, bug magnets, team momentum, firefighting frequency, recently added files, and active contributors
3. **Cross-reference** hotspots with bug magnets to identify high-risk files
4. **Report** findings with actionable recommendations

## What You'll Learn

| Dimension | Question Answered |
|-----------|-------------------|
| Code Hotspots | Which files change the most? |
| Bug Magnets | Which files attract the most bug fixes? |
| High-Risk Files | Which files are both hot AND buggy? |
| Bus Factor | Who knows what? Is knowledge concentrated? |
| Team Momentum | Is development accelerating, stable, or declining? |
| Firefighting | How often are there emergency fixes and reverts? |
| Recently Added | Where is active development happening? |

## Requirements

- `git` (any version)
- A git repository with commit history

## Attribution

This skill is inspired by ["The Git Commands I Run Before Reading Any Code"](https://piechowski.io/post/git-commands-before-reading-code/) by [Ally Piechowski](https://github.com/grepsedawk). The original article describes 5 git commands for codebase reconnaissance. This skill extends the concept with auto-scaling, cross-referencing, and actionable recommendations.

## License

[MIT](LICENSE)
```

- [ ] **Step 2: Commit**

Run:
```bash
git add README.md
git commit -m "docs: add README with installation, usage, and attribution"
```

---

### Task 9: Validate and Final Commit

**Files:**
- Review: `skills/codebase-recon/SKILL.md`

- [ ] **Step 1: Verify SKILL.md frontmatter name matches directory name**

Run:
```bash
# Directory name
basename ~/Projects/codebase-recon-skill/skills/codebase-recon
# Should output: codebase-recon

# Frontmatter name
head -5 ~/Projects/codebase-recon-skill/skills/codebase-recon/SKILL.md
# Should show: name: codebase-recon
```

Expected: both match `codebase-recon`

- [ ] **Step 2: Verify SKILL.md is under 500 lines**

Run:
```bash
wc -l ~/Projects/codebase-recon-skill/skills/codebase-recon/SKILL.md
```

Expected: under 500 lines

- [ ] **Step 3: Verify repo structure matches spec**

Run:
```bash
find ~/Projects/codebase-recon-skill -not -path '*/.git/*' -not -name '.git' | sort
```

Expected output should include:
```
skills/codebase-recon/SKILL.md
README.md
LICENSE
docs/specs/2026-04-09-codebase-recon-design.md
.gitignore
```

- [ ] **Step 4: Test skill locally by installing in Claude Code**

Run:
```bash
cd ~/Projects/codebase-recon-skill
# Dry-run: read the SKILL.md and verify it parses correctly
head -10 skills/codebase-recon/SKILL.md
```

Verify frontmatter YAML is valid (no syntax errors, all required fields present).

- [ ] **Step 5: Create GitHub repo and push**

Run:
```bash
cd ~/Projects/codebase-recon-skill
gh repo create codebase-recon-skill --public --source=. --push --description "A coding agent skill that analyzes git history to understand a codebase before reading any code"
```

Expected: repo created and code pushed

---

### Task 10: Smoke Test on a Real Repo

- [ ] **Step 1: Install the skill locally**

Add the local skill to Claude Code for testing:
```bash
cd ~/Projects/codebase-recon-skill
# Install from local path for testing
```

- [ ] **Step 2: Run `/codebase-recon` on a real repo**

Navigate to a repo with meaningful history (e.g., one of the repos in `~/Projects/` with 100+ commits) and invoke `/codebase-recon`.

Verify:
- Phase 1 correctly identifies repo size
- All 7 Phase 2 commands run and produce output
- Cross-referencing identifies overlapping files (or correctly reports none)
- Report structure matches the template in the spec
- Recommendations section is populated

- [ ] **Step 3: Test the markdown export prompt**

When asked "Want me to save this report?", say yes. Verify the markdown file is written correctly and not auto-committed.

- [ ] **Step 4: Fix any issues found and commit**

If smoke test reveals problems, fix them in SKILL.md and commit:
```bash
git add skills/codebase-recon/SKILL.md
git commit -m "fix: address issues found in smoke test"
git push
```
