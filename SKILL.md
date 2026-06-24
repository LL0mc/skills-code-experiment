---
name: code-experiment
description: "Run safe code experiments on an isolated Git branch. Use when the user wants to try a risky optimization, prototype different implementations and compare results, benchmark alternative approaches, or iterate on improvements with clear success/failure criteria. Do NOT use for: refactoring (needs regression tests, not experiments), documentation, deployment, CI setup, writing brand-new modules from scratch, migration projects, or simple bug fixes that don't need comparative evaluation. Trigger on: '试试', '实验', '尝试新方案', '优化', '能不能提升', 'prototype', 'spike', 'POC', 'sandbox', 'proof of concept', '看看效果', '测一下性能', '对比', '基线', '分支上试试'."
license: MIT
---

# Code Experiment

A structured workflow for running safe, isolated code experiments without risking production code. The experiment lives entirely on a Git branch — no copies, no feature flags, no scaffolding.

The whole point: **make the risky change, measure honestly, keep only what works, and leave no trace of what doesn't.**

## Phase 1: Setup

### 1a. Propose and proceed

When the user asks to try something, don't start coding. First propose the experiment framework concisely:
1. Work on a separate branch
2. Define success criteria together
3. Take a baseline measurement
4. Iterate with clear stop conditions
5. Let them decide the outcome

Then **proceed immediately** — the proposal IS the starting point. Do not wait for approval. The user can object at any time if they disagree. In automated/self-contained scenarios (subagents, CI), propose inline and continue without blocking.

### 1b. Create the experiment branch

```
git checkout -b experiment/<short-description>
```

The branch name is the container for the entire experiment. When the experiment ends, either the branch merges or it's deleted — no leftover artifacts.

### 1c. Define success criteria with the user

Ask the user explicitly. Record the answers.

| Question | Purpose |
|---|---|
| What metric are we optimizing? | Latency, bundle size, LOC, test coverage, readability score... Must be measurable |
| What's the current (baseline) value? | If unknown, we'll measure it next |
| What's the minimum improvement to call it a success? | e.g., "20% faster", "10% less memory", "half the code" |
| How do we measure? | Exact command or script. Must be the same every time |
| How many measurements per run? | For noisy metrics, run 3-5 times and take median |
| Where can we change code? | Specific files/modules/functions |
| Where can we NOT change? | API contracts, public interfaces, config schemas, etc. |
| Consecutive failures before switching direction? | Default: 3 |
| Total failures before stopping? | Default: 5 (across all directions) |

### 1d. Run baseline evaluation

Run the agreed-upon evaluation on the current (unchanged) code. This is the **baseline**.

If there's no existing benchmark script, write one as part of the experiment and commit it to the branch. The benchmark script lives at the project root as `benchmark-<experiment-name>.sh` (or `.py`, `.js`, etc.).

Save raw output to `.experiment-results/000-baseline/`.

### 1e. Write EXPERIMENT.md

Create `EXPERIMENT.md` in the project root (see the template in `references/experiment-template.md`). Fill in:
- Hypothesis and direction
- All success criteria from 1c
- Scope (can/cannot modify)
- Baseline value and evaluation command
- Fallback settings

### 1f. First commit

```
git add -A
git commit -m "experiment: init <name>"
git tag experiment/<name>/baseline
git tag experiment/<name>/best  # same commit initially
```

## Phase 2: Iteration Loop

### The loop

```
best = baseline  (tag: experiment/<name>/best)

loop:
  1. Make ONE focused change to the code
  2. Run the SAME evaluation command as baseline
  3. Save raw output to .experiment-results/attempt-NNN/
  4. Compare result with best so far
  5. Record in EXPERIMENT.md and commit
  6. If improved:
       git tag -f experiment/<name>/best  # move to current commit
       consecutive_failures = 0
       continue on this code (iterate further)
     If NOT improved:
       git checkout experiment/<name>/best  # restore best code
       consecutive_failures += 1
       if consecutive_failures >= N:
         switch direction (different approach)
         consecutive_failures = 0
       if total_failures >= M:
         STOP → go to Phase 3
     If code doesn't compile / tests fail:
       git checkout experiment/<name>/best  # restore best code
       record as BROKEN (not counted as failure, just noise)
  7. Repeat
```

### Rules for clean signal

- **One change per attempt.** Each attempt modifies exactly one thing. If you want to try two ideas, they get two separate attempts. This way you know what caused the result.
- **Same evaluation, same conditions.** Always use the exact same command. Run on the same machine. Close other programs. Don't warmup differently.
- **Commit after every evaluation.** Even failed attempts get committed. The commit graph is the experiment's raw timeline. Tag every attempt: `git tag experiment/<name>/attempt-NNN` for easy reference.
- **For noisy metrics**, run the evaluation 3-5 times and report median + min/max. Don't cherry-pick the best run.
- **If a change introduces a new bug** (tests fail, code doesn't compile), immediately restore best and record as BROKEN. Don't iterate on broken code.

### Recording in EXPERIMENT.md

Each attempt gets a log entry:

```markdown
### attempt-003

**Change:** Replaced bubble sort with quicksort in sort_users()
**Hypothesis:** O(n log n) will be faster than O(n²) for lists > 100 items

**Evaluation command:** `npm run benchmark:sort`
**Result:** 142ms (was 310ms at best)
**vs best:** +54% (improved ✓)
**Status:** SUCCESS
**Tags:** experiment/sort-optimization/attempt-003, experiment/sort-optimization/best

**Notes:** Benchmarked 5 runs, all within 2% variance. Clean improvement.
```

### Branch by abstraction for structural changes

If the experiment requires a fundamentally different internal implementation that would be hard to revert by checkout alone (e.g., replacing a module's core algorithm), use **branch by abstraction**:

1. Identify the interface boundary around what you're changing
2. Keep the original implementation as the default
3. Add the new implementation behind the same interface
4. Wire a switch (config, env var, or simple if/else) to select which impl runs
5. The switch is temporary — it exists only inside this branch and is removed before merge

This lets you A/B within a single commit without messy file copies.

## Phase 3: Wrap-up

### 3a. Present results

Update `EXPERIMENT.md` with a summary section at the top:

```markdown
## Summary

**Hypothesis:** [original hypothesis]
**Baseline:** [value]
**Best result:** [value] (attempt-NNN)
**Change vs baseline:** [+/-X%]

**Total attempts:** N
**Successful:** N
**Failed:** N
**Broken:** N

**What worked:** [description]
**What didn't:** [description]

**Conclusion:** [clear statement of findings]
```

Present the summary to the user. Include the key numbers and the diff.

### 3b. Export outputs

Copy all experiment artifacts to a designated output location so they persist regardless of what happens to the branch:
- `EXPERIMENT.md` (the structured record)
- Modified source files
- Raw data from `.experiment-results/` (if not cleaned)
- `summary.txt` with a quick overview

In subagent scenarios where a parent agent expects results in a outputs directory, always write there.

### 3c. Let the user decide

Ask: "这里是最新的实验报告摘要。你觉得这个结果值得合并到主分支吗？如果不需要合并，这个分支可以直接丢弃。"

Do NOT push for merge. The user decides.

- If they want to merge: `git checkout main && git merge experiment/<name>`
- If they want to discard: the experiment branch can be deleted

### 3d. Cleanup

Ask: "实验产生的中间文件（.experiment-results/ 里的 benchmark 数据）需要清理掉吗？"

If yes, delete `.experiment-results/`. The `EXPERIMENT.md` on main (if merged) or on the branch (if kept for reference) contains all the structured information.

## Key principles

- **The branch is the sandbox.** Source isolation comes from Git, not directory copies. No `experiments/` folder, no copied files, no scaffolding.
- **Honest measurement.** Run the same thing the same way every time. Document the command. Don't cherry-pick.
- **One change at a time.** If you can't describe the single change you made, you made too many changes.
- **Rollback is a `git checkout`.** Always know which commit is the best so far. Tag it.
- **Let the user decide success.** Your job is to generate data, analysis, and a clear diff — not to make the call.
