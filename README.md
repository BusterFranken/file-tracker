# file-tracker

A CLI tool that shows which files your [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plans will edit, and groups the plans you can safely run together.

When you're working with multiple Claude Code plans and want to run them in parallel, `file-tracker` groups the ones that touch no common files — so you can see at a glance which plans you can run together.

## How it works

1. Reads plan files from `~/.claude/plans/`
2. Extracts file paths referenced in each plan
3. Matches each plan to a git repo and retires any plan whose work has already landed in a commit
4. Groups plans that share no files into sets you can run together in parallel

## Install

```bash
# Copy the script to your PATH
cp file-tracker ~/.local/bin/
chmod +x ~/.local/bin/file-tracker
```

Requires Python 3 and `git`.

## Usage

```bash
file-tracker
```

Example output:

```
FILE TRACKER
==========================================

OPEN PLANS  (3 completed plans hidden)

  [A] Fix Muscle Balance card  (3m ago, 1 file)
      Sources/WorkoutTrackerApp/Views/Tracking/TrackingView.swift

  [B] Multi-Set Exercises  (1h ago, 2 files)
      WorkoutRepository.swift
      WorkoutView.swift

──────────────────────────────────────────

CAN RUN TOGETHER  (grouped — no shared files)

  ● [A] [B]  (2 plans)
      Fix Muscle Balance card · Multi-Set Exercises
```

Plans are grouped into **maximal sets that can all run together** — every plan in a group touches no file used by any other plan in that group. If `[A]`, `[B]` and `[C]` are all mutually compatible, they collapse into a single group instead of three pairwise lines:

```
  ● [A] [B] [C]  (3 plans)
      Fix layout · Add feature · Update docs
```

A plan that shares a file with *every* other plan can't join any group, so it's dropped from the overview with a one-line note:

```
  Dropped — must run alone (share files with all others): [D]
```

## How plans are filtered

A plan is considered **done** — and hidden — once a single commit (made after the plan was written) changes at least 60% of the files that plan references. This retires a plan when its work actually lands, instead of wiping every plan the moment you make *any* commit in the repo.

The threshold lives in `COMMITTED_COVERAGE` at the top of the script. It uses the *best single commit* rather than the union of all commits, so shared files (`worker.ts`, `webhook.ts`, …) that get touched by unrelated commits don't prematurely retire a plan whose own work never shipped.

Plans that can't be matched to a git repo fall back to a 24-hour window.
