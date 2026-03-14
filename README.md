# file-tracker

A CLI tool that shows which files your [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plans will edit, and checks for conflicts between them.

When you're working with multiple Claude Code plans and want to run them in parallel, `file-tracker` tells you if they touch the same files — so you know whether it's safe.

## How it works

1. Reads plan files from `~/.claude/plans/`
2. Extracts file paths referenced in each plan
3. Finds the relevant git repo and filters to only plans created **since the last commit**
4. Compares file lists and reports conflicts

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

PLANS since last commit (1h ago)

  [A] Fix Muscle Balance card  (3m ago, 1 file)
      Sources/WorkoutTrackerApp/Views/Tracking/TrackingView.swift

  [B] Multi-Set Exercises  (1h ago, 2 files)
      WorkoutRepository.swift
      WorkoutView.swift

──────────────────────────────────────────

  Plan [A] + Plan [B]: No shared files — safe to run in parallel.
```

When plans share files:

```
  Plan [A] + Plan [B]: CONFLICT
    Fix layout  +  Add feature
    Shared: WorkoutRepository.swift
```

## How plans are filtered

Only plans created after the last git commit in their project are shown. This means after you commit and start fresh, old plans disappear automatically. Plans that can't be matched to a git repo fall back to a 24-hour window.
